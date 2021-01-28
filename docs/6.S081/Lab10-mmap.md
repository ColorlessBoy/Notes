# Lab10: mmap

系统调用`mmap`和`munmap`允许UNIX程序对它们的地址空间执行更精致的控制。它们可以被用来在进程间共享内存，将文件映射在进程地址空间以及作为用户级别页错误机制的一部分，例如公开课中讨论的垃圾回收算法。在这个实验中，我们需要添加`mmap`和`munmap`到xv6中，主要是内存映射文件。

实验代码在`mmap`分支上。

```bash
make clean
git fetch
git checkout mmap
```

帮助文档（`man 2 mmap`）说明了`mmap`的具体使用方式。现在人们使用的`mmap`的声明如下：

```c
void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
```

`mmap`有非常多的调用方式，本次实验只需要实现部分功能即可，具体涉及的是内存映射文件的功能。我们可以认为`addr`始终未0，也就是说内核应该决定文件应该映射到虚拟地址的位置。`mmap`在执行成功后返回该虚拟地址，失败时返回`0xffffffffffffffff`。`length`是映射的比特数，它和文件的长度不一定相同。`prot`表示被映射的内存的权限，选项有：是否可读、是否可写、是否可执行，对应的宏有`PROT_READ`和`PROT_WRITE`或者都有。`flag`可以是`MAP_SHARED`(表示内存的修改需要同步到文件中)或者`MAP_PRIVATE`(不同步)，其他位的功能不需要实现。`fd`是打开的文件描述符。我们可以认为`offset`始终为0。

进程映射相同的`MAP_SHARED`文件到不同的物理页是可以的。

`munmap(addr, length)` 应该移除`mmap`映射的地址。如果进程修改了属性为`MAP_SHARED`的文件，那么所做的修改需要更新到文件中。我们可以假设映射的区域只会在开头、结尾或者整个区域，不会有洞。

!!!题目描述
    我们需要实现`mmap`和`munmap`足够多的功能来通过`mmaptest`和`usertests`这两个程序。

提示：

- 首先添加`_mmaptest`到`UPROGS`，然后添加`mmap`和`munmap`系统调用的外壳，实现`user/mmaptest.c`编译通过。(在`kernel/fcntl.h`中已经有一些宏定义`PROT_READ`。) 

???参考答案
    1. `Makefile`中添加：`$U/_mmaptest\`。
    2. `user/user.h`中添加系统调用的声明：

        ```c
        void* mmap(void *, uint, int, int, int, uint);
        int munmap(void *, uint);
        ```

    3. 在`user/usys.pl`中添加：

        ```c
        entry("mmap");
        entry("munmap");
        ```

    4. 在`kernel/syscall.h`中添加：

        ```c
        #define SYS_mmap   22
        #define SYS_munmap 23
        ```

    5. 在`kernel/syscall.c`中添加：

        ```c
        extern uint64 sys_mmap(void);
        extern uint64 sys_munmap(void);
        static uint64 (*syscalls[])(void) = {
        //...
        [SYS_mmap]    sys_mmap,
        [SYS_munmap]  sys_munmap,
        };
        ```
    
    6. 在`kernel/sysfile.c`中添加：

        ```c
        uint64
        sys_mmap(void)
        {
            return 0;
        }

        uint64
        sys_munmap(void)
        {
            return 0;
        }
        ```

- 延迟添加页表，响应页错误。`mmap`不需要分配物理内存也以及读文件。在`usertrap`的页错误处理代码中实现`lazy page`的代码。实现`lazy page`可以让`mmap`处理大文件快一些，并且可以处理大于内存大小的文件。
  
    !!!参考答案
        可以参考 [Lab05 Lazy Allocation](../Lab05-Xv6-Lazy-Allocation/)。

- 保持追踪`mmap`给每个进程映射的内容。定义一个结构体`VMA`(virtual memory area)，记录`mmap`创建的虚拟内存的地址、长度、权限和文件等信息。因为内核没有内存分配器，所以可以声明固定长度的数组给`VMAs`并且从数组中分配需要的空间。例如16的长度已经足够。
- 实现`mmap`：查找进程地址未使用的部分来映射文件，添加VMA到进程的映射区域表中。VMA应该包含一个指向被映射文件的结构体struct file的指针。`mmap`应该要增加文件的`reference`数量，保证文件关闭的时候结构体不会消失。运行`mmaptest`：第一个`mmap`测试应该能成功，但是后面的页表错误还不能处理。
- 添加处理访问`mmap`映射的页引起页错误的代码，用来分配物理内存。读入4096个字节，并且将它映射到用户地址空间。使用`readi`来阅读文件，它接收一个`offset`（注意锁）。不要忘记设置正确的权限。运行`mmaptest`,应该会运行到`munmap`测试。
- 实现`munmap`: 查找VMA来获得地址范围，并且unmap指定页（提示：使用`uvmunmap`）。如果`munmap`移除了所有的页，那么需要降低文件的reference计数器。如果页被修改过，并且文件被标记为`MAP_SHARED`，那么需要更新文件。可以参考`filewrite`。
- 理想情况下，我们只需要被标记为`MAP_SHARED`的页被修改后才写回文件。RISC-V的PTE的脏比特`D`标记页是否被写入。但是`mmaptest`不查看没被修改的页是否被写回，所以本实验可以不考虑理想情况。
- 修改`exit`来删除进程中由`mmap`映射的区域。这时应该能够通过`mmaptest`的`mmap_test`测试，但是无法通过`fork_test`测试。
- 修改`fork`来保证子进程有和父进程一样的`mapped`的区域。不要忘记`VMA`结构体中的`reference`计数器。在子进程中产生页错误后，可以分配一个新的物理内存页来映射，而不一定和父进程的物理内存页相同。虽然保持相同的功能很好，但是需要额外的实现。运行`mmaptest`，能通过`mmap_test`和`fork_test`两个命令。

???参考答案
    1. 创建VMA结构体。具体包括一下几个步骤

        a. 在`kernel/proc.h`中声明：

        ```c
        struct VMA {
        uint64 addr;
        uint length;
        int prot;
        int flags;
        struct file *f;
        uint offset;
        uint used;
        };
        ```
    
        b. 在`kernel/proc.h`中声明宏：`#define NVMAS 16`。

        c. 在`struct proc`中添加`VMA`数组成员变量： 

        ```c
        struct VMA vmas[NVMAS];      // Virtual memory areas
        ```

        d. 在`kernel/defs.h`中添加声明：`struct VMA;`。

    2. 实现`kernel/sysfile.c:sysmmap()`函数。

        ```c
        uint64
        sys_mmap(void)
        {
            uint64 addr;
            uint length, offset;
            int prot, flags, i;
            struct file *f;
            struct proc *p = myproc();

            if(argaddr(0, &addr) < 0 || argint(1, (int*)&length) < 0 || argint(2, &prot) < 0 
            || argint(3, &flags) < 0 || argfd(4, 0, &f) < 0 || argint(5, (int*)&offset) < 0){
                return MAP_FAILED;
            }

            if(f->type != FD_INODE || (!f->writable && (prot & PROT_WRITE) && (flags & MAP_SHARED))){
                return MAP_FAILED;
            }

            if(addr == 0){
                addr = p->sz;
                p->sz += length;
            } else {
                // panic("sys_mmap: addr != 0");
                return MAP_FAILED;
            }

            for(i = 0; i < NVMAS; ++i) {
                if(p->vmas[i].addr == 0) {
                    p->vmas[i].addr = addr;
                    p->vmas[i].length = PGROUNDUP(length);
                    p->vmas[i].prot = prot;
                    p->vmas[i].flags = flags;
                    p->vmas[i].f = filedup(f);
                    p->vmas[i].offset = offset;
                    break;
                }
            }
            if(i == NVMAS)
                return MAP_FAILED;
            
            return addr;
        }
        ```
    
    3. 实现陷阱处理程序`kernel/trap.c:usertrap()`，同时要添加一些头文件。

        ```c
        #include "sleeplock.h"
        #include "fs.h"
        #include "file.h"

        //...

        void
        usertrap(void)
        {
            //...
            }else if(r_scause() == 13 || r_scause() == 15){
                char *mem;
                uint64 va = r_stval();
                if(va >= p->sz){
                    p->killed = 1;
                }else{
                    mem = kalloc();
                    if(mem == 0){
                        // run out of physical memory, 
                        // wait for an empty page.
                        p->killed = 1;
                    }else{
                        int i;
                        memset(mem, 0, PGSIZE);

                        // mmap
                        for(i = 0; i < NVMAS; ++i){
                        if(p->vmas[i].addr != 0 && va >= p->vmas[i].addr 
                            && va < p->vmas[i].addr + p->vmas[i].length){
                            va = PGROUNDDOWN(va);
                            ilock(p->vmas[i].f->ip);
                            if(readi(p->vmas[i].f->ip, 0, (uint64)mem, 
                            va + p->vmas[i].offset - p->vmas[i].addr, PGSIZE) < 0){
                                panic("mmap read failed");
                            }
                            iunlock(p->vmas[i].f->ip);
                            p->vmas[i].used += PGSIZE;
                            if(mappages(p->pagetable, va, PGSIZE, 
                            (uint64)mem, (p->vmas[i].prot << 1)|PTE_U) != 0){
                                kfree(mem);
                                p->killed = 1;
                            }
                            break;
                        }
                        }
                        // not mmap
                        if(i == NVMAS){
                            if(mappages(p->pagetable, PGROUNDDOWN(va), PGSIZE, 
                                (uint64)mem, PTE_W|PTE_X|PTE_R|PTE_U) != 0){
                                kfree(mem);
                                p->killed = 1;
                            }
                        }
                    }
                }
            } else if((which_dev = devintr()) != 0){
            //...
        }
        ```
    
    4. 实现`sys_munmap()`程序，具体包括两个步骤。我的实现可以处理有洞的情形，以`PGSIZE`大小写回，但是`writei`会调用很多次。

        a. 在`kernel/vm.c`中添加辅助函数`uvmmunmap()`。

        ```c
        void
        uvmmunmap(pagetable_t pagetable, uint64 va, uint64 length, struct VMA *vma)
        {
            uint64 a;
            pte_t *pte;

            if((va % PGSIZE) != 0)
                panic("uvmmunmap: not aligned");

            for(a = va; a < va + length && a < va + vma->length; a += PGSIZE){
                if((pte = walk(pagetable, a, 0)) == 0)
                    continue;
                if((*pte & PTE_V) == 0)
                    continue;
                if(PTE_FLAGS(*pte) == PTE_V)
                    panic("uvmunmap: not a leaf");

                if((vma->flags & MAP_SHARED) && vma->f->writable && (vma->prot & PROT_WRITE)){
                    begin_op();
                    ilock(vma->f->ip);
                    writei(vma->f->ip, 1, a, a + vma->offset - vma->addr, PGSIZE);
                    iunlock(vma->f->ip);
                    end_op();
                }

                vma->used -= PGSIZE;
                if(vma->used < 0)
                    panic("uvmmunmap: vma->used < 0");

                uint64 pa = PTE2PA(*pte);
                kfree((void*)pa);
                *pte = 0;
            }
        }
        ```
        
        b. 在`kernel/def.h`中添加`uvmmunmap()`的声明。

        ```c
        void uvmmunmap(pagetable_t, uint64, uint64, struct VMA*);
        ```
        
        c. 实现`kernel/sysfile.c:sys_munmap()`。

        ```c
        uint64
        sys_munmap(void)
        {
            uint64 va;
            uint length;
            struct proc *p = myproc();

            if(argaddr(0, &va) < 0 || argint(1, (int*)&length) < 0){
                return -1;
            }

            va = PGROUNDDOWN(va);

            for(int i = 0; i < NVMAS; ++i){
                if(p->vmas[i].addr != 0 && va >= p->vmas[i].addr 
                && va < p->vmas[i].addr + p->vmas[i].length){
                    uvmmunmap(p->pagetable, va, length, &(p->vmas[i]));
                    if(p->vmas[i].used == 0){
                        fileclose(p->vmas[i].f);
                        p->vmas[i].addr = 0;
                        p->vmas[i].length = 0; 
                        p->vmas[i].prot = 0;
                        p->vmas[i].flags = 0;
                        p->vmas[i].f = 0;
                        p->vmas[i].offset = 0;
                        break;
                    }
                }
            }
            return 0;
        }
        ```
    
    5. 补充`kernel/proc.c:exit()`函数中释放`vma`的代码。

        ```c
        for(int i = 0; i < NVMAS; ++i){
            if(p->vmas[i].addr){
                uvmmunmap(p->pagetable, p->vmas[i].addr, p->vmas[i].length, &(p->vmas[i]));
                fileclose(p->vmas[i].f);
                p->vmas[i].addr = 0;
                p->vmas[i].length = 0; 
                p->vmas[i].prot = 0;
                p->vmas[i].flags = 0;
                p->vmas[i].f = 0;
                p->vmas[i].offset = 0;
            }
        }
        ```
    
    6. 补充`kernel/proc.c:fork()`函数中复制`vma`的代码。

        ```c
        // mmap
        for(i = 0; i < NVMAS; ++i) {
            if(p->vmas[i].addr) {
                np->vmas[i].addr   = p->vmas[i].addr;
                np->vmas[i].length = p->vmas[i].length;
                np->vmas[i].prot   = p->vmas[i].prot;
                np->vmas[i].flags  = p->vmas[i].flags;
                np->vmas[i].f      = filedup(p->vmas[i].f);
                np->vmas[i].offset = p->vmas[i].offset;
            }
        }
        ```