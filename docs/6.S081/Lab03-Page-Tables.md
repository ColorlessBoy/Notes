# Lab3: Page Tables

这一节实验涉及的是计算机的页表，也就是虚拟内存。我阅读了 `xv6-book` 的第三章内容，写得
非常好，很清楚也简洁，涉及其他章节的复杂内容都会省略，增加了阅读理解的连贯性，非常建议
跟着讲义阅读源码：`kern/memlayout.h`、`kern/vm.c` 和 `kernel/kalloc.c`。

我同时还建议提前阅读第四章和第七章的部分内容，对第二题和第三题有很好的帮助作用。

获取实验代码：

```bash
$ git fetch
$ git checkout pgtbl
$ make clean
```

## 打印页表

!!!题目描述
    实现一个函数 `vmprint(pagetable_t)` 接受一个 `pagetable_t` 打印它的值。
    同时在 `exec` 函数的 `return argc` 前添加 `if(p->pid==1) vmprint(p->pagetable)`，
    来打印第一个进程的 `pagetable`。程序需要通过`make grade`中的`pte printout`测试。

当 `xv6` 启动时需要打印类似如下的信息：

```bash
page table 0x0000000087f6e000
..0: pte 0x0000000021fda801 pa 0x0000000087f6a000
.. ..0: pte 0x0000000021fda401 pa 0x0000000087f69000
.. .. ..0: pte 0x0000000021fdac1f pa 0x0000000087f6b000
.. .. ..1: pte 0x0000000021fda00f pa 0x0000000087f68000
.. .. ..2: pte 0x0000000021fd9c1f pa 0x0000000087f67000
..255: pte 0x0000000021fdb401 pa 0x0000000087f6d000
.. ..511: pte 0x0000000021fdb001 pa 0x0000000087f6c000
.. .. ..510: pte 0x0000000021fdd807 pa 0x0000000087f76000
.. .. ..511: pte 0x0000000020001c0b pa 0x0000000080007000
```

第一行是 `vmprint` 的参数。接下来每一行缩进代表PTE的深度，接着是该PTE在表中的下标、PTE
的地址以及对应的物理地址。不要打印不允许访问的PTE。

提示：

- 可以在 `kernel/vm.c` 中实现；
- 需要用到 `kernel/riscv.h` 最后的宏定义；
- 可以参考 `freewalk`；
- 在 `kernel/defs.h` 中声明 `vmprint` 就可以在 `exec` 中使用它了；
- 使用 `%p` 来打印地址。

???参考答案
    提示已经把一些琐碎的修改点告诉了我们，所以这里就单纯列出一个 `vmprint` 的参考代码吧：

    ```c
    // Recursively print page-table pages information.
    void
    vmprint(pagetable_t pagetable, int deepth)
    {
        if(deepth == 0)
            printf("page table %p\n", pagetable);
        // there are 2^9 = 512 PTEs in a page table.
        for(int i = 0; i < 512; i++){
            pte_t pte = pagetable[i];
            if(pte & PTE_V){
            uint64 child = PTE2PA(pte);

            for(int j = 0; j < deepth; ++j)
                printf(".. ");
            printf("..%d: pte %p pa %p\n", i, pte, child);

            if((pte & (PTE_R|PTE_W|PTE_X)) == 0) 
                // this PTE points to a lower-level page table.
                vmprint((pagetable_t)child, deepth+1);
            }
        }
    }
    ```

!!!思考题
    根据 `xv6-book` 中关于用户内存空间的图3-4来解释第0页与第2页的内容，以及用户模式
    (user mode) 下能否读写第1页的内容。

???参考答案
    `exec` 会先把需要调用的程序复制到进程空间的开头（kernel/exec.c:42:59)，所以前几页
    一定都是程序的代码。但是究竟有几页，我们还是需要继续推理一下。`exec` 会紧接着代码后
    面分配两页内存，分别对应用户空间的`guard page` 和 `stack`(kernel/exec.c:69:76)。
    其中`guard page` 是用来把程序的内容和栈分隔开的安全页，会在`PTE_U`位被标记为不可用
    (kernel/vim.c:348)，而 `PTE_V`还是1。所以 `vmprint` 会打印一个连续的n页程序代码
    数据有关的PTE，紧跟着两页`guard page` 和 `stack`。从实验结果来看，第一个`vmprint`
    开头输出了连续的三页PTE，所以我们知道开始的程序代码占用第0页内容，第1页是`guard page`，
    第2页是`stack`。那么，用户模式下是不可以读写第1页的`guard page`内容的。

## A kernel page table per process

`Xv6` 有一个独立的内核页表用于运行内核程序。而内核页表直接和物理内存地址直接对应，内核
虚拟地址就等于物理地址。`Xv6` 同时也为每一个进程分配独立的用户空间，只包含进程的用户内存，
它从0开始。因为内核页表不包含这些进程页表的映射，用户地址不能被内核直接使用。内核需要使用
用户空间的地址时，就必须先把它转换成物理地址。这节的目标就是让内核直接使用用户指针。

!!!问题描述
    首先修改内核，使得每一个进程使用它子集的拷贝后的内核页表，当该进程进入内核态时使用
    这个私人的内核页表。修改 `struct proc` 来为每一个进程添加一个内核页表，并且修改
    调度器，当切换进程时也修改内核地址。当然，每个进程的内核页表应该和全局的内核页表相同。
    如果 `usertests` 运行通过，就通过了这个实验。

提示：

- 修改结构体 `struct proc` 给每个进程添加内核页表；
- 一种合理的产生内核页表的方式是：实现一个类似 `kvminit` 的函数，创建一个新页表而不是修改
    `kernel_pagetable`。会在 `allocproc` 中调用这个函数。
- 确定每个进程的内核页表中映射了这个函数的内核栈。在未修改的代码里，实现在 `procinit`,
    这里希望移动到 `allocproc` 函数里。
- 修改调度器 `scheduler()` 来修改内核页表在寄存器`satp`里的地址（参考 `kvminithart`)。
    不要忘记在调用 `w_satp()` 后调用 `sfence_vma()`。
- 在 `freeproc` 里释放进程的内核页表。
- 我们在释放进程的内核页表时，不要释放对应物理内存页的叶子节点。
- `vmprint` 能帮助我们`debug` 进程页表。
- 可能需要在 `kernel/vm.c` 和 `kernel/proc.c` 里添加新的函数。
- 如果页表缺失，将会引起页表错误。错误信息包含 `sepc=0x00000000xxxxxxxx`。
    在 `kernel/kernel.asm` 里查看 `xxxxxxxx` 对应哪一段的代码。

> 这一题要求我们为子进程创建一个内核页表，不要求内核页表存在程序页表的映射。对内核页表添加程序页表
> 的功能在下一题完成。这道题目的主要难点是要释放内核页表的时候，先释放进程的内核栈；
> 以及如何修改 `scheduler`。针对如何修改 `scheduler`，
> 这里可以简单阅读一下第七章的内容，`scheduler`里调用了一个`swtch`执行了去运行某个对应进程，
> 所以这个指令前后完成内核页表寄存器 `satp` 的修改。
> 我的理解是，`scheduler` 似乎运行中全局的内核页表下。 `vm.c` 等程序中的变量也声明在全局
> 的内核页表下。

我再次提醒，下面这个答案只能作为思路卡壳时候的提示，生搬硬看是搞不懂的。

???参考答案

    1. 在 `def.h` 对应 `vm.c`的栏中声明函数 `kvmcreate` 和 `kvmfree`，分别用于创建和释放
        内核页表；
        
        ```c
        // Create a kernel page table.
        pagetable_t
        kvmcreate(void)
        {
            pagetable_t kptbl = uvmcreate();

            if(kptbl == 0)
                return 0;

            // uart registers
            if(mappages(kptbl, UART0, PGSIZE, UART0, PTE_R | PTE_W) != 0){
                return 0;
            }

            // virtio mmio disk interface
            if(mappages(kptbl, VIRTIO0, PGSIZE, VIRTIO0, PTE_R | PTE_W) != 0){
                uvmunmap(kptbl, UART0, 1, 0);
                return 0;
            }

            // CLINT
            if(mappages(kptbl, CLINT, 0x10000, CLINT, PTE_R | PTE_W) != 0){
                uvmunmap(kptbl, UART0, 1, 0);
                uvmunmap(kptbl, VIRTIO0, 1, 0);
                return 0;
            }

            // PLIC
            if(mappages(kptbl, PLIC, 0x400000, PLIC, PTE_R | PTE_W) != 0){
                uvmunmap(kptbl, UART0, 1, 0);
                uvmunmap(kptbl, VIRTIO0, 1, 0);
                uvmunmap(kptbl, CLINT, PGROUNDUP(0x10000)/PGSIZE, 0);
                return 0;
            }

            // map kernel text executable and read-only.
            if(mappages(kptbl,KERNBASE, (uint64)etext-KERNBASE, KERNBASE, PTE_R | PTE_X) != 0){
                uvmunmap(kptbl, UART0, 1, 0);
                uvmunmap(kptbl, VIRTIO0, 1, 0);
                uvmunmap(kptbl, CLINT, PGROUNDUP(0x10000)/PGSIZE, 0);
                uvmunmap(kptbl, PLIC, PGROUNDUP(0x400000)/PGSIZE, 0);
                return 0;
            }

            // map kernel data and the physical RAM we'll make use of.
            if(mappages(kptbl, (uint64)etext, PHYSTOP-(uint64)etext, (uint64)etext, PTE_R | PTE_W) != 0){
                uvmunmap(kptbl, UART0, 1, 0);
                uvmunmap(kptbl, VIRTIO0, 1, 0);
                uvmunmap(kptbl, CLINT, PGROUNDUP(0x10000)/PGSIZE, 0);
                uvmunmap(kptbl, PLIC, PGROUNDUP(0x400000)/PGSIZE, 0);
                uvmunmap(kptbl, KERNBASE, PGROUNDUP((uint64)etext-KERNBASE)/PGSIZE, 0);
                return 0;
            }

            // map the trampoline for trap entry/exit to
            // the highest virtual address in the kernel.
            if(mappages(kptbl, TRAMPOLINE, PGSIZE, (uint64)trampoline, PTE_R | PTE_X) != 0){
                uvmunmap(kptbl, UART0, 1, 0);
                uvmunmap(kptbl, VIRTIO0, 1, 0);
                uvmunmap(kptbl, CLINT, PGROUNDUP(0x10000)/PGSIZE, 0);
                uvmunmap(kptbl, PLIC, PGROUNDUP(0x400000)/PGSIZE, 0);
                uvmunmap(kptbl, KERNBASE, PGROUNDUP((uint64)etext-KERNBASE)/PGSIZE, 0);
                uvmunmap(kptbl, (uint64)etext, PGROUNDUP(PHYSTOP-(uint64)etext)/PGSIZE, 0);
            };
            return kptbl;
        }

        // Free a kernel page table.
        void kvmfree(pagetable_t kptbl)
        {
            uvmunmap(kptbl, UART0, 1, 0);
            uvmunmap(kptbl, VIRTIO0, 1, 0);
            uvmunmap(kptbl, CLINT, PGROUNDUP(0x10000)/PGSIZE, 0);
            uvmunmap(kptbl, PLIC, PGROUNDUP(0x400000)/PGSIZE, 0);
            uvmunmap(kptbl, KERNBASE, PGROUNDUP((uint64)etext-KERNBASE)/PGSIZE, 0);
            uvmunmap(kptbl, (uint64)etext, PGROUNDUP(PHYSTOP-(uint64)etext)/PGSIZE, 0);
            uvmunmap(kptbl, TRAMPOLINE, 1, 0);
            uvmfree(kptbl, 0);
        }
        ```

    2. 修改 `kernel/proc.h` 的结构体，添加 `kptbl` 以及内核页表对应的程序栈 `kptbl_kstack`。
        其中 `kptbl_kstack` 是用来表示内核页表对应的程序栈是否分配成功的，实际上和 `kstack`
        内容一模一样。
    
    3. 在 `kernel/proc.c` 中定义 `kstackset` 和 `kstackfree` 两个函数，用来映射内核页
       表的程序栈，以及释放程序栈。分别在 `p->kptbl` 创建后 以及 释放前完成。

        ```c

        // Set kernel stack
        uint64
        kstackset(struct proc *p){
            if(mappages(p->kptbl, p->kstack, PGSIZE, kvmpa(p->kstack), PTE_R | PTE_W) != 0)
                return 0;
            return p->kstack;
        }

        // Free kernel stack
        void
        kstackfree(struct proc *p){
            uvmunmap(p->kptbl, p->kptbl_kstack, 1, 0);
        }
        ```

    4. 修改 `kernel/proc.c` 中的两个函数 `allocproc` 和 `freeproc`，完成第二题。

        ```c
        // Look in the process table for an UNUSED proc.
        // If found, initialize state required to run in the kernel,
        // and return with p->lock held.
        // If there are no free procs, or a memory allocation fails, return 0.
        static struct proc*
        allocproc(void)
        {
            //...
            p->kptbl = kvmcreate();
            if(p->kptbl == 0){
                freeproc(p);
                release(&p->lock);
                return 0;
            }

            p->kptbl_kstack = kstackset(p);
            if(p->kptbl_kstack == 0){
                freeproc(p);
                release(&p->lock);
                return 0;
            }
            //...
        }
        // free a proc structure and the data hanging from it,
        // including user pages.
        // p->lock must be held.
        static void
        freeproc(struct proc *p)
        {
            //...
            if(p->kptbl_kstack)
                kstackfree(p);
            p->kptbl_kstack = 0;

            if(p->kptbl)
                kvmfree(p->kptbl);
            p->kptbl = 0;
            //...
        }
        ```

    5. 修改 `scheduler` 函数

        ```c
        void
        scheduler(void)
        {
            //...
            if(p->state == RUNNABLE) {
                kvminithart1(p->kptbl);
                // Switch to chosen process.  It is the process's job
                // to release its lock and then reacquire it
                // before jumping back to us.
                p->state = RUNNING;
                c->proc = p;
                swtch(&c->context, &p->context);

                // Process is done running for now.
                // It should have changed its p->state before coming back.
                c->proc = 0;

                found = 1;
                kvminithart();
            }
            //...
        }
        ```

## 简化 `copyin/copyinstr`

内核中的函数 `copyin` 读取用户空间的指针。它先把指针转换为物理地址，然后又因为内核地址和
物理地址直接对应，然后内核就可以访问这个地址了。转换过程就是查找进程页表的过程。我们需要
给每个进程的内核页表创建一个映射，来允许 `copyin` 直接使用用户指针。

!!!问题描述
    替换 `kernel/vm.c:copyin()`的实现：间接调用`kernel/vmcopyin.c：copyin_new()`;
    替换 `kernel/vm.c:copyinstr()` 的实现：间接调用`kernel/vmcopy.c:copyinstr_new`;
    映射用户地址到进程内核页表，使得 `copyin()` 和 `copyinstr()` 可用。
    如果 `usertests` 能够正确运行，那么就通过了这个题目。

这个实验依赖于前提：用户虚拟地址和内核的代码和数据地址不重叠。用户空间虚拟地址的起始地址
是0，内核的空闲空间要高得多。但是，这里没有限制用户程序一定要比内核的最小虚拟地址低。
在 `xv6` 中，内核启动后的地址是 `0xC000000`，也就是 `PLIC` 寄存器的地址；具体可以看
`kernel/vm.c:kvminit()` 和 `kernel/memlayout.h`，以及`xv6-book`的图3-4。我们需要
调整`xv6`来避免用户进程超过 `PLIC` 地址。实际上，内核的最低虚拟地址是 `0x2000000`，
保存着`CLINT`寄存器的值。但是`xv6`只是在启动初期会用到这个寄存器的，所以完全可以覆盖它。

提示：

- 先实现 `copyin()`，等成功之后，再实现 `copyinstr`；
- 当内核修改进程的用户映射时，也应该以同样的方式修改进程内核页表。对应要修改 `fork()`，
    `exec()` 和 `sbrk()`;
- 不要忘记包含第一个进程`userinit`的用户页表到它的内核页表中；
- 注意用户指针需要的允许标记。 被标记上 `PTE_U` 的页不能在内核态被访问；
- 不要忘记 `PLIC` 的限制。

Linux使用类似的页表机制。几年前内核在用户空间和内核空间都使用相同的进程页表，同时具有
用户和内核地址映射，以避免用户空间和内核空间之间切换时必须切换页表。但是也导致了
`side-channel`攻击，例如 `Meltdown` 和 `Specture`。

下面的参考答案只能保证测试通过，但是不能保证算法的效率。
我需要将 `grade-lab-pgtbl:65` 关于测试时间的限定增加到 `350` 才能通过测试，不知道是程序效率
的问题，还是电脑不太行。

???参考答案

    1. 首先在 `vm.c` 中定义两个函数（不要忘记在`def.h`中声明）。
    一个函数叫 `kvmdealloc` 用于删除内核页表中对应的某些进程页，另一个叫 `kvmcopy` 用于从
    进程页表中复制某些页到内荷页表中。

        ```c
        // Remove process's page in process's kernel pagetable.
        uint64
        kvmdealloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz)
        {
            if(newsz >= oldsz)
                return oldsz;

            if(PGROUNDUP(newsz) < PGROUNDUP(oldsz)){
                int npages = (PGROUNDUP(oldsz) - PGROUNDUP(newsz)) / PGSIZE;
                uvmunmap(pagetable, PGROUNDUP(newsz), npages, 0);
            }

            return newsz;
        }

        // Copy process's pagetable to process's kernel pagetable.
        int
        kvmcopy(pagetable_t kptbl, pagetable_t pagetable, uint64 start, uint64 end)
        {
            pte_t *pte;
            uint64 pa, i;
            // uint flags;

            start = PGROUNDUP(start);

            if(start >= PLIC || end > PLIC)
                return -1;

            for(i = start; i < end; i += PGSIZE){
                if((pte = walk(pagetable, i, 0)) == 0)
                panic("kvmcopy: pte should exist");
                if((*pte & PTE_V) == 0)
                panic("kvmcopy: page not present");
                pa = PTE2PA(*pte);
                // flags = PTE_FLAGS(*pte);
                if(mappages(kptbl, i, PGSIZE, pa, PTE_R) != 0){
                goto err;
                }
            }
            return 0;

            err:
            uvmunmap(kptbl, start, (end - start) / PGSIZE, 0);
            return -1;
        }
        ```

    2. 修改 `proc.c:fork()`，添加（如果连添加到哪里都不知道，其实先不要看答案）：
    
        ```c
        if(kvmcopy(np->kptbl, np->pagetable, 0, np->sz) < 0){
            freeproc(np);
            release(&np->lock);
            return -1;
        }
        ```

    3. 为系统调用 `sbrk()` 修改函数 `proc.c:growproc()`:
    
        ```c
        int
        growproc(int n)
        {
            uint sz;
            struct proc *p = myproc();

            sz = p->sz;
            if(n > 0){
                if((sz = uvmalloc(p->pagetable, sz, sz + n)) == 0) {
                return -1;
                }
                if(kvmcopy(p->kptbl, p->pagetable, p->sz, sz) < 0) {
                return -1;
                }
            } else if(n < 0){
                sz = uvmdealloc(p->pagetable, sz, sz + n);
                kvmdealloc(p->kptbl, p->sz, sz);
            }

            p->sz = sz;
            return 0;
        }
        ```

    4. 修改 `proc.c:userinit()` ：
    
        ```
        // allocate one user page and copy init's instructions
        // and data into it.
        uvminit(p->pagetable, initcode, sizeof(initcode));
        p->sz = PGSIZE;
        kvmcopy(p->kptbl, p->pagetable, 0, p->sz);
        ```

    5. 修改 `exec.c`，添加：

        ```c
        kvmdealloc(p->kptbl, oldsz, 0);
        if(kvmcopy(p->kptbl, p->pagetable, 0, p->sz) < 0)
            return -1;

        ```

        这时已经给成功给进程中的内核页表添加了进程页表的拷贝了。

    6. 修改 `copyin()` 和 `copyinstr()` 两个函数，在`vm.c`开头添加外部函数`copyin_new()`
        和`copyinstr()`的声明，然后这两个函数分别直接调用这两个新函数。

!!!思考题
    解释一下 `copyin_new()` 中的第三个测试 `srcva + len < srcva` 的作用；
    举出会引发危害的反例来使得 `copyin_new()` 的前两个测试通过，而第三个测试不通过。

    如果没有第三个测试，
    `srcva = 0x0000 0000 0200 0000` 和 `len = 0xffff ffff ffff ffff` 就可以利用
    `uint64` 表达能力有限的特性，访问到内核数据。

## 总结
这次实验做得有些坎坷，是没有理解源码、不会`gdb`调试等原因造成的吧。
有时候也想太多，想着内核页表直接和进程页表公用一张表等等想法，造成了干扰，走了弯路。

其实还是没有特别理解内存，比如 `scheduler` 中定义的那几个局部变量被保存在哪里，在函数中切换
页表寄存器为什么没有报错；放的位置不对，为什么又会报错等等问题。不过每一个细节都搞定了再走下一步
的学习方式效率太低了，先继续走起来吧。
