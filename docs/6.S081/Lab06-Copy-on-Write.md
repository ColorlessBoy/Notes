# Lab6: Copy-on-Write Fork for xv6

虚拟内存提供了一层间接的级别：当内核感知到内存指向的PTEs被标记为非法或者只读时，它会产生一个页错
误， 然后中断处理来修改这个PTEs的地址，从而间接寻址的功能。也就是说，在计算机系统里，任何一个系
统错误都可以使用间接的方式来解决。Lazy allocation是这样，Copy-on-Write也是这样。

切换到COW实验的分支：

```bash
$ git fetch
$ git checkout cow
$ make clean
```

## 问题

xv6中的系统调用`fork()`将拷贝父进程所有的用户空间内存到子进程中。如果父进程非常大，那么这种拷贝
要花费大量的时间和空间。通常，我们完全不需要复制所有的用户空间：例如，`fork()`后，子进程紧跟着
执行一个`exec()`指令时，子进程会完全忽略大部分的父进程的用户空间。另一方面，父进程和子进程有时
候会对同一个页表进行写入操作导致冲突，复制页表非常重要。

## 解决方法

`Copy-on-write (COW) fork()` 延迟分配与拷贝物理内存页给子进程，直到拷贝是必须的。
`COW fork()`给子进程创建一个页表指向父进程的物理内存。 `COW fork()` 给父进程和子进程的PTEs
都标记为不可写。当任意一个进程尝试写入某个内存页时，CPU会产生一个页表错误。内核页错误处理handler
会检测并处理这个错误，为这个虚拟页表分配物理内存页表，拷贝原本的内存页到一个新的内存页，然后将报
错的PTE指向这个新的内存页，同时被标记为可写入。这时，页错误handler返回时，程序就能够写入这个虚
拟地址了。

`COW fork()`释放用户空间的物理地址会有一些麻烦。 多个进程的页表会指向同一个物理地址，仅仅当最
后一个进程选择释放这个物理地址后，物理地址才真的被释放。

## 实现Copy-on-Write

!!!问题描述
    实现`Copy-on-Write`来通过`cowtest`和`usertests`。

合理的实现流程：

1. 调整`uvmcopy()`将父进程的物理地址映射给子进程，但是不分配新物理内存页。同时将PTEs的PTE_W
    置0；
2. 调整`usertrap()`来识别页错误。当页错误发生在COW页，那么使用`kalloc()`来分配新的物理内存；
3. 确保物理内存页被正确地释放：当且仅当最后一个指向它的PTE被释放。最好的方法是为每一个物理内存
    页保存一个`reference count`来记录指向它的PTE有多少个。修改`kalloc()`分配这个`reference
    count`。在`fork`中，当父子进程共享物理内存页时，增加`reference count`，在不共享后降低
    `reference count`。`kfree`也要有所修改，仅当物理内存页真正需要被释放时，再放到free list
    后面。可以将这些计数器放在一个固定大小的整数数组中。你需要研究整数数组的下标和大小该如何确定。
    例如，可以直接将物理地址除以4096来作为下标，数组的大小为最大物理地址除以4096。然后再`kinit`
    中声明数组。
4. 修改`copyout()`来使用类似的技术来使用COW技术。

提示：

- 不要基于`lazy allocation`，但是`lazy allocation`实验能够给一些提示；
- 记录每个PTE是否是COW产生的映射会很有用。可以使用PTE中的RSW位（reserved for software），
    目前的xv6没有使用这个标志位。
- `usertests` 和 `cowtest` 的实验并不重合，所以要同时通过两个测试才可以。
- `kernel/riscv.h`中有很多有用的宏定义。
- 如果COW页错误发生，但是没有空余的内存，那么进程应该被杀死。

## 参考答案

1. 首先在`riscv.h：334`中声明超参数`#define PTE_RSW (1L << 8)`用于标记COW页。
2. 在这个实验中，将用到`xv6-book`第六章涉及到的内容——并发锁（concurrency lock）。并发锁用于控制`kernel/kalloc.c`文件中的数据结构`pacnt`用于保存物理内存页被几个虚拟地址所指向。同时需要一个函数`incpacnt()`来更新这个数据结构。(这里我没有使用结构体内部成员函数)

    ```c
    struct {
        struct spinlock lock;
        uint64 cnt[PHYSTOP>>PGSHIFT]; 
    } pacnt; // physical addresses' reference count

    void
    incpacnt(uint64 pa)
    {
        acquire(&pacnt.lock);
        pacnt.cnt[(uint64)pa >> PGSHIFT]++;
        release(&pacnt.lock);
    }
    ```

3. 接着我们为`pacnt`结构体修改`kernel/kalloc.c:kinit()`、`kernel/kalloc.c:kfree()`和`kernel/kalloc.c:kalloc()`三个函数。

    ```c
    void
    kinit()
    {
        initlock(&kmem.lock, "kmem");
        initlock(&pacnt.lock, "pacnt");

        freerange(end, (void*)PHYSTOP);
    }

    void
    kfree(void *pa)
    {
        acquire(&pacnt.lock);
        if(pacnt.cnt[(uint64)pa >> PGSHIFT] > 0)
            pacnt.cnt[(uint64)pa >> PGSHIFT]--;
        if(pacnt.cnt[(uint64)pa >> PGSHIFT] > 0){
            release(&pacnt.lock);
            return;
        }
        release(&pacnt.lock);
        
        //...
    }

    void *
    kalloc(void)
    {
        struct run *r;

        acquire(&kmem.lock);
        r = kmem.freelist;
        if(r)
            kmem.freelist = r->next;
        release(&kmem.lock);

        // update the physical address' reference count
        incpacnt((uint64)r);

        if(r){
            memset((char*)r, 5, PGSIZE); // fill with junk
        }
        return (void*)r;
    }
    ```

4. 修改`kernel/vm.c:uvmcopy()`函数，构造COW页。

    ```c
    int
    uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
    {
        pte_t *pte;
        uint64 pa, i;
        uint flags;
        // char *mem;

        for(i = 0; i < sz; i += PGSIZE){
            if((pte = walk(old, i, 0)) == 0)
                panic("uvmcopy: pte should exist");
            if((*pte & PTE_V) == 0)
                panic("uvmcopy: page not present");

            *pte = ((*pte | PTE_RSW) & (~PTE_W));
            pa = PTE2PA(*pte);
            flags = PTE_FLAGS(*pte);

            incpacnt(pa); //update the physical address's reference count.
            if(mappages(new, i, PGSIZE, pa, flags) != 0){
                goto err;
            }
            
            // if((mem = kalloc()) == 0)
            //   goto err;
            // memmove(mem, (char*)pa, PGSIZE);
            // if(mappages(new, i, PGSIZE, (uint64)mem, flags) != 0){
            //   kfree(mem);
            //   goto err;
            // }
        }
        return 0;

        err:
            uvmunmap(new, 0, i / PGSIZE, 1);
            return -1;
    }
    ```

5. 由此引发的COW页错误的处理程序在`trap.c:usertrap()`中实现。其中为了方便，我在`vm.c`实现了一个分配COW冲突页的函数`cowalloc()`。
   
    ```c
    else if(r_scause() == 13 || r_scause() == 15){
        uint64 va = r_stval();
        if(va >= p->sz){
        p->killed = 1;
        }
        else{
        if(cowalloc(p->pagetable, va) < 0)
            p->killed = 1;
        }
    }
    ```

    ```c
    int
    cowalloc(pagetable_t pagetable, uint64 va){
        pte_t *pte;
        uint64 pa;
        uint flags;
        char *mem;

        if((pte = walk(pagetable, PGROUNDDOWN(va), 0)) == 0)
            return -1;
        if((*pte & PTE_V) == 0 || (*pte & PTE_RSW) == 0)
            return -1;
        
        pa = PTE2PA(*pte);
        flags = ((PTE_FLAGS(*pte) | PTE_W) & (~PTE_RSW));

        if((mem = kalloc()) == 0)
            return -1;
        memmove(mem, (char*)pa, PGSIZE);
        
        *pte = PA2PTE(mem) | flags;

        kfree((char*)pa);
        return 0;
    }
    ```

6. 函数`vm.c:copyout()`也会遇到COW页，需要我们处理相应的情况

    ```c
    int
    copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len)
    {
        \\...
        pa0 = cowwalkaddr(pagetable, va0);
        \\...
    }
    ```

    其中`cowwalkaddr()`会在遇到COW页时，再复制物理地址页

    ```c
    uint64
    cowwalkaddr(pagetable_t pagetable, uint64 va)
    {
        pte_t *pte;
        uint64 pa;

        if(va >= MAXVA)
            return 0;

        pte = walk(pagetable, va, 0);
        if(pte == 0)
            return 0;
        if((*pte & PTE_V) == 0)
            return 0;
        if((*pte & PTE_U) == 0)
            return 0;
        if(*pte & PTE_RSW) {
            if(cowalloc(pagetable, va) < 0)
            return 0;
        }
        pa = PTE2PA(*pte);
        return pa;
    }
    ```

7. 为了能在不同文件中调用新定义的函数，需要再`def.h`中声明这些函数：

    ```c
    // kalloc.c
    // ...
    void            incpacnt(uint64);

    // vm.c
    // ...
    int             cowalloc(pagetable_t, uint64);
    ```