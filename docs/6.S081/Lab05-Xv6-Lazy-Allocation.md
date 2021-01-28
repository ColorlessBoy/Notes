# Lab5: Lazy Page Allocation

Lazy Page能够大大降低用户堆空间的页表交换的硬件IO次数。Xv6使用系统调用`sbrk()`来向内核申请堆
空间。`sbrk()`分配一个物理内存给进程的虚拟地址。内核分配物理地址页的花销非常大。程序经常申请
远远超出它需求的内存空间，却从不使用；或者提前申请内存空间但是过了很久才使用。为了让`sbrk`效率
更高，内核往往使用Lazy Page技术：在程序申请内存时并不真正地分配，直到程序使用它时产生一个缺页
中断，这时内核再分配真实的物理空间。

获取本节实验代码：

```bash
$ git fetch
$ git checkout lazy
$ make clean
```

## Eliminate allocation from sbrk()

!!!问题描述
    删除`sbrk(n)`系统调用中页表分配内容，也就是`sysproc.c:sys_sbrk()`的内容。新的
    `sbrk`应该只增长进程的大小属性（`myproc()->sz`），然后返回旧的进程的大小。它不应该
    分配内存——也就是删除`growproc()`。

修改完毕后，会引起哪里报错呢？修改完后进入qemu，输入指令`echo hi`，我们应该能看到：

```bash
init: starting sh
$ echo hi
usertrap(): unexpected scause 0x000000000000000f pid=3
            sepc=0x0000000000001258 stval=0x0000000000004008
va=0x0000000000004000 pte=0x0000000000000000
panic: uvmunmap: not mapped
```

`usertrap():...`报错指令是来自`trap.c`；确保你理解为什么会引起页错误。`stval=0x0..04008`
表示虚拟地址`0x4008`读取的时候发生错误。

## Lazy allocation
!!!问题描述
    修改`trap.c`中对应的页错误的部分，为用户空间申请一个新的物理内存页到出错误的地址，然后返回
    到用户空间继续用户程序的运行。需要在`printf()`输出`usertrap()`的程序段前完成这个功能。
    修改成功后`echo hi`应该可以工作。

提示：

- 可以通过`r_scause()`来查看报错信息，如果是13或15那就是页错误；
- 调用`r_stval()`返回RISC-V的`stval`寄存器，它包含引起页错误的虚拟地址；
- 借鉴`vm.c:uvmalloc()`的代码。需要使用`kalloc()`和`mappages()`；
- 使用`PGROUNDDOWN(va)`来取整虚拟地址；
- 现在的`uvmunmap()`会报错，修改它；
- 内核报错了可以阅读`kernel/kernel.asm`；
- 使用从`pgtbl`中实现的`vmprint`函数，来打印`page table`的内容；
- 如果遇到报错`incomplete type proc`， 包含`spinlock.h`和`proc.h`头文件。

## Lazytests and Usertests

测试程序要考虑更复杂的情况：

- `sbrk()`是负数的时候该怎么？（我这里直接保留原来的`sbrk()`函数的判断，负数报错，返回-1。)
- 如果虚拟内存访问的本来就是高于进程申请的内存页呢？（在`trap.c`中添加判断。）
- 处理正确处理`fork()`。
- 处理进程进入系统调用`read`和`write`时，系统还没有给用户页表分配内存的情况。
    （应该是修改`copyin`和`copyout`）
- 如果内存占满了，需要正确地杀死进程。
- 处理用户栈下面的非法页的访问。

如果实现正确，那么内核运行`usertests`会出现如下内容：

```bash
$  lazytests
lazytests starting
running test lazy alloc
test lazy alloc: OK
running test lazy unmap...
usertrap(): ...
test lazy unmap: OK
running test out of memory
usertrap(): ...
test out of memory: OK
ALL TESTS PASSED
$ usertests
...
ALL TESTS PASSED
$
```

???参考答案

    1. 首先修改`sysproc.c`：

        ```c
        uint64
        sys_sbrk(void)
        {
            int addr;
            int n;

            if(argint(0, &n) < 0)
                return -1;
            addr = myproc()->sz;

            if(n < 0){
                if(growproc(n) < 0) return -1;
            }
            else{
                myproc()->sz += n;
            }
            return addr;
        }
        ```
    
    2. 修改陷阱函数`trap.c:68`：

        ```c
        else if(r_scause() == 13 || r_scause() == 15){
            char *mem;
            uint64 va = r_stval();
            if(va >= myproc()->sz){
                p->killed = 1;
            }else{
                mem = kalloc();
                if(mem == 0){
                    // run out of physical memory, 
                    // wait for an empty page.
                    p->killed = 1;
                }else{
                    memset(mem, 0, PGSIZE);
                    if(mappages(myproc()->pagetable, PGROUNDDOWN(va), PGSIZE, 
                        (uint64)mem, PTE_W|PTE_X|PTE_R|PTE_U) != 0){
                        kfree(mem);
                        p->killed = 1;
                    }
                }
            }
        }
        ```
    3. 修改`vm.c`来使得系统调用`write`和`read`可用：

        ```c
        //添加头文件
        #include "spinlock.h"
        #include "proc.h"
        //...
        uint64
        walkaddr(pagetable_t pagetable, uint64 va)
        {
            pte_t *pte;
            uint64 pa;

            if(va >= MAXVA)
                return 0;

            // pte = walk(pagetable, va, 0);
            pte = walk(pagetable, va, 0);
            if(pte == 0 || (*pte & PTE_V) == 0){
                struct proc *p = myproc();
                if(va >= p->sz)
                    return 0;

                char *mem = kalloc();
                if(mem == 0){
                    return 0;
                }
                memset(mem, 0, PGSIZE);
                if(mappages(pagetable, PGROUNDDOWN(va), PGSIZE, (uint64)mem, 
                    PTE_W|PTE_X|PTE_R|PTE_U) != 0){
                    kfree(mem);
                    return 0;
                }else{
                    return (uint64)mem;
                }
            }
            if((*pte & PTE_U) == 0)
                return 0;
            pa = PTE2PA(*pte);
            return pa;
        }
        //...
        int
        mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
        {
            //...
            if(*pte & PTE_V){
                return -1;
                // panic("remap");
            }
            //...
        }
        //...
        void
        uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free)
        {
            //...
            if((pte = walk(pagetable, a, 0)) == 0){
                continue;
                // panic("uvmunmap: walk");
            }
            if((*pte & PTE_V) == 0){
                continue;
                // panic("uvmunmap: not mapped");
            }
            //...
        }
        //...
        int
        uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
        {
            //...
            if((pte = walk(old, i, 0)) == 0){
                continue;
            }
            if((*pte & PTE_V) == 0){
                continue;
            }
            //...
        }
        //...
        ```