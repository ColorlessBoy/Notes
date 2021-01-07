# Lab8: Locks

这一小节要重新设计锁，增加并行性。多核心机器上操作系统并行性比较差的主要症状是较高的锁争用。我们可以通过修改数据结构和锁的机制来降低锁的争用频率从而提高并行性。我们主要要再xv6的内存分配和高速缓存块上做文章。

需要先阅读xv6-book的第六章内容、第三章第五节物理内存分配有关的内容以及第八章前三节的内容。

获取实验的源代码的指令是

```bash
git fetch
git checkout lock
make clean
```

## Memory Allocator

用户程序`user/kalloctest`压力测试xv6的内存分配程序。三个进程不断地增加缩小它们的地址空间，调用多个`kalloc`、`kfree`，不断地申请`kmem.lock`。如果进程执行`acquire`后等待其他进程，`kalloctest`将会做出统计，并且打印在命令行中。

在分配内存时产生过度的竞争的原因是xv6只用了一个链表来维护闲置物理内存页。我们需要重新设计这一部分的数据结构，包括锁和链表。一个简单的想法是为每一个CPU维护一个链表和锁。虽然增加了平行性，但是难点就是如何同步所有的CPU的链表。

!!!问题描述
    我们需要为每一个CPU实现一个空物理内存链表。当一个CPU的空闲内存链表用完时，需要能够从其他CPU中偷一些闲置页回来。新的锁命名建议都以`kmem`开头。可以运行`usertests sbrkmuch`来测试实验的结果，或者运行`make grade`。

提示：

- 可能需要宏定义 `kernel/param.h:NCPU`；
- 让`freernage`来分配内存，将所有空闲内存页分配给调用`freerange`；
- `cpuid`返回当前CPU核心的编号，只有当中断关闭的时候调用它才是安全的。可以使用函数`push_off()`和`pop_off()`来控制中断的开闭；
- 可以参考`kernel/sprintf.c`的`smprintf`函数来学习字符串规范化输出的方式。

???参考答案
    这道题目做起来比较简单，凭直觉就好了。我们这里只需要修改`kalloc.c`文件。

    1. 修改物理地址链表数据结构，为每一个CPU提供一个双向链表。

        ```c
        struct {
            struct spinlock lock;
            struct run *freelist;
        } kmem[NCPU];
        ```

    2. 修改链表初始函数 `kinit()` 。

        ```c
        void
        kinit()
        {
            char lock_name[10];
            for(int i = 0; i < NCPU; ++i) {
                snprintf(lock_name, 10, "kmem%d", i);
                initlock(&kmem[i].lock, lock_name);
            }
            freerange(end, (void*)PHYSTOP);
        }
        ```

    3. `kfree()`函数也要针对每一个CPU进行操作。

        ```c
        void
        kfree(void *pa)
        {
            struct run *r;
            int icpu;

            if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
                panic("kfree");

            // Fill with junk to catch dangling refs.
            memset(pa, 1, PGSIZE);

            r = (struct run*)pa;

            push_off();
            icpu = cpuid();
            acquire(&kmem[icpu].lock);
            r->next = kmem[icpu].freelist;
            kmem[icpu].freelist = r;
            release(&kmem[icpu].lock);
            pop_off();
        }
        ```

    4. 最后修改分配物理内存的函数`kalloc()`，当前CPU链表为空时，从其他CPU的链表借用。

        ```c
        void *
        kalloc(void)
        {
            struct run *r;
            int icpu;

            push_off();
            icpu = cpuid();
            for(int i = 0; i < NCPU; ++i) {
                acquire(&kmem[(i + icpu)%NCPU].lock);
                r = kmem[(i + icpu)%NCPU].freelist;
                if(r) {
                    kmem[(i + icpu)%NCPU].freelist = r->next;
                }
                release(&kmem[(i + icpu)%NCPU].lock);
                if(r) break;
            }
            pop_off();

            if(r)
                memset((char*)r, 5, PGSIZE); // fill with junk
            return (void*)r;
        }
        ```
## Buffer cache

这道实验题和前面那道完全无关。

多核心处理器的文件系统使用非常频繁，它们使用的锁叫做`bcache.lock`，保护了`kernel/bio.c`中的磁盘缓存。用户程序`bcachetest`创造了一些进程来重复阅读不同的文件，以此来产生代码竞争。

在代码`kernel/bio.c`中，我们能够分析可得`bcache.lock`保护了高速缓存块寄存器，`bache.lock`保护了一些高速缓存器，`refcnt`保存了指向该高速寄存快的个数。高速缓存区的指标为`b->dev`以及`b->blockno`。

!!!题目描述
    修改block cache来降低acquire循环中等待所的轮次。理想情况下，block cache涉及到的锁时间周期接近0，实际比500少就可以。然后修改`bget`和`brelse`使得当前查找和释放不同的高速缓存块几乎不产生冲突（也就是说不申请锁`bcache.lock`）。同时要在多核心下保持数据结构的不变性。

所有的锁的名字建议由`bcache`开始，并且在`initlock`中初始化。

降低`block cache`的争用比`kalloc`要麻烦得多，因为高速缓存块在进程间是共享的。在`kalloc`中只需要给CPU分配不同的物理内存即可，但是`block cache`的分配就不能使用这种方法了。建议使用一个哈希表来保存每个高速缓存块的锁。

下面的场景会导致死锁，但是没有关系：

- 两个进程同时使用一个块；
- 两个进程同时miss高速缓存块；
- 当两个进程使用冲突的块，并且两个块号映射到哈希表的同一个槽的时候，会发生死锁，可以简单增大哈希表，不需要提供解决冲突的机制。

提示：

- 阅读`block cache`的相关描述，在xv6-book的第八章的前三节；
- 可以使用固定的`buckets`并且不动态分配哈希表的大小。使用素数的`buckets`来尽量避免竞争；
- 查找哈希表和添加哈希表必须是原子操作；
- 删除所有`buffer`的链表（`bcache.head`等位置），使用带时间戳的`buffer`（`kernel/trap.c`的`ticks`）。`brelse`不在需要`bcache`锁，`bget`选择最早被使用过的块（也就是很久没有被使用过的块）；
- 可以序列化发射`bget`。当`cache`命中失败时，`bget`可以重新选择一个`buffer`；
- 有些场景可能同时需要两个锁，`bcache`和各个`bucket`的锁，注意避免死锁；
- 在替换`block`时，要将旧的`struct buf`从一个`bucket`移动到另一个`bucket`。但是要避免移动前和移动后的`bucket`是相同的；
- Debug建议：先保留全局的`bcache.lock`，等确保没有竞争操作后再去掉全局锁。也可以运行`make CPUS=1 qemu`来测试单个核心的情况。

!!!Note
    我没有做过题目，所以在翻译题目的时候我也是云里雾里的。做完之后我感觉就是上一道题目的变种，也是分配多个Buckets。

???参考答案

    1. 修改`bcache`。

        ```c
        #define NBUCK 13

        struct {
            struct spinlock lock[NBUCK];
            struct buf buf[NBUF];

            // Linked list of all buffers, through prev/next.
            // Sorted by how recently the buffer was used.
            // head.next is most recent, head.prev is least.
            struct buf head[NBUCK];
        } bcache;
        ```

    2. 修改初始化，将`buf`均匀分给哈希表。

        ```c
        void
        binit(void)
        {
            struct buf *b;
            char lock_name[10];
            int i;

            for(i = 0; i < NBUCK; ++i){
                snprintf(lock_name, 10, "bcache%d", i);
                initlock(&bcache.lock[i], lock_name);

                // Create linked list of buffers
                bcache.head[i].prev = &bcache.head[i];
                bcache.head[i].next = &bcache.head[i];
            }

            for(i = 0, b = bcache.buf; b < bcache.buf+NBUF; b++, i = (i + 1) % NBUCK){
                b->next = bcache.head[i].next;
                b->prev = &bcache.head[i];
                initsleeplock(&b->lock, "buffer");
                bcache.head[i].next->prev = b;
                bcache.head[i].next = b;
            }
        }
        ```

    3. 分配cache，近似LRU。

        ```c
        static struct buf*
        bget(uint dev, uint blockno)
        {
            struct buf *b;
            int i = blockno % NBUCK;

            acquire(&bcache.lock[i]);

            // Is the block already cached?
            for(b = bcache.head[i].next; b != &bcache.head[i]; b = b->next){
                if(b->dev == dev && b->blockno == blockno){
                    b->refcnt++;
                    release(&bcache.lock[i]);
                    acquiresleep(&b->lock);
                    return b;
                }
            }

            // Not cached.
            // Recycle the least recently used (LRU) unused buffer.
            for(b = bcache.head[i].prev; b != &bcache.head[i]; b = b->prev){
                if(b->refcnt == 0) {
                    b->dev = dev;
                    b->blockno = blockno;
                    b->valid = 0;
                    b->refcnt = 1;
                    release(&bcache.lock[i]);
                    acquiresleep(&b->lock);
                    return b;
                }
            }

            // Bucket is full. Stealing.
            for(int j = 1; j < NBUCK; ++j){
                int k = (i + j) % NBUCK;
                acquire(&bcache.lock[k]);
                for(b = bcache.head[k].prev; b != &bcache.head[k]; b = b->prev){
                    if(b->refcnt == 0) {
                        b->next->prev = b->prev;
                        b->prev->next = b->next;
                        release(&bcache.lock[k]);

                        b->next = bcache.head[i].next;
                        b->prev = &bcache.head[i];
                        bcache.head[i].next->prev = b;
                        bcache.head[i].next = b;

                        b->dev = dev;
                        b->blockno = blockno;
                        b->valid = 0;
                        b->refcnt = 1;
                        release(&bcache.lock[i]);

                        acquiresleep(&b->lock);
                        return b;
                    }
                }
                release(&bcache.lock[k]);
            }

            panic("bget: no buffers");
        }
        ```

    4. 对`brelse`、`bpin` 和 `bunpin`修改所需要的锁。

        ```c
        void
        brelse(struct buf *b)
        {
            if(!holdingsleep(&b->lock))
                panic("brelse");

            releasesleep(&b->lock);

            int i = b->blockno % NBUCK;
            acquire(&bcache.lock[i]);
            b->refcnt--;
            if (b->refcnt == 0) {
                // no one is waiting for it.
                b->next->prev = b->prev;
                b->prev->next = b->next;
                b->next = bcache.head[i].next;
                b->prev = &bcache.head[i];
                bcache.head[i].next->prev = b;
                bcache.head[i].next = b;
            }
            release(&bcache.lock[i]);
        }

        void
        bpin(struct buf *b) {
            int i = b->blockno % NBUCK;
            acquire(&bcache.lock[i]);
            b->refcnt++;
            release(&bcache.lock[i]);
        }

        void
        bunpin(struct buf *b) {
            int i = b->blockno % NBUCK;
            acquire(&bcache.lock[i]);
            b->refcnt--;
            release(&bcache.lock[i]);
        }
        ```

