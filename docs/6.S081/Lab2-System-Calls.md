# System Calls

## 阅读资料

实验指导建议阅读 `xv6-book` 的第二章和第四章的部分内容。我觉得关于 `xv6` 的启动和运行非常重要，
这里可以记录一下。现在的`xv6` 是在 `RISC-V` 体系框架下实现的操作系统，底层的汇编代码完全可以
不求甚解，这样会迷失在代码里。我们就像书里介绍的一样，大概了解即可。

### 启动过程

首先是一个对启动过程的总体描述，这里翻译自 `xv6-book/chapter2`。

使用 `RISC-V` 的CPU分为三个模式：`machine mode`、`supervisor mode` 和 `user mode`。
当 `RISC-V` 计算机启动的时候，它运行一个存储在只读存储器的`boot loader`启动程序。启动
加载程序加载 `xv6` 的内核程序到内存中。再 `machine mode`，CPU 执行 `xv6` 启动程序到
`_entry`(kernel/entry.S:6)。`RISC-V` 启动的时候页面硬件功能还无法使用：虚拟地址直接
映射到物理地址。

`RISC-V` 将 `xv6` 内核加载到物理地址`0x80000000`，因为`0x0`到`0x80000000`包含一些
I/O 设备。

`_entry` 处的指令先设定了一个栈，使 `xv6` 能够运行C语言的程序。`xv6` 声明这个初始栈的
地方是`stack0`(kernel/start.c:11)。而 `_entry` 加载到栈指针寄存器 `sp` 的地址是
`stack0+4096`，栈顶地址，因为`RISC-V`里使用的栈的更新方向是下降的。接着内核调用C语言函
数 `start`(kernel/start.c:21)。

函数 `start` 会注册一些配置，这些指令只能在 `machine mode` 中运行，然后就会切换到
`supervisor mode`。为了进入`supervisor mode`,`RISC-V` 提供了一个指令 `mret`。
这个指令通常是从`machine mode`的指令返回到 `supervisor mode`。但是 `start` 并没有从
这个指令返回，只是设定了一些东西，让CPU认为执行过类似的指令：将保存前一个运行模式的寄存
器 `mstatus` 设定成 `supervisor` 模式，并且把返回地址设定成 `main`(把 `main` 地址保
存到寄存器 `mepc` 中)，在`supervisor mode` 中禁用虚拟内存(向页表寄存器`satp`写入0),
最后将所有的中断和异常都授权给`supervisor mode`。

在真正跳到 `supervisor mode` 前，`start` 还会再多做一件事情：让时钟芯片产生时钟中断。
经过这些琐碎的事情，`start`调用 `mret` 返回到 `supervisor mode`。 内核进入到 `main`
函数(kernel/main.c:11)。

`main` 函数启动一些设备和子程序后，创建它的第一个进程 `userinit`(kernel/proc.c:219)。
第一个进程执行一些用 `RISC-V` 汇编指令编写的程序，`initcode.S`(user/initcode.S:1), 
它会通过运行`exec` 系统调用，重新返回内核。一旦内核完成 `initcode.S` 调用的系统调用，
就会返回到用户空间。 如果需要的话，`Init` (user/init.c:15) 创建了一个新的 `console` 
设备文件，然后打开它作为`0`、`1` 和 `2`文件描述符。最后在 `console` 中运行一个 `shell`
程。系统启动完成。

### 系统调用

接着我们看看一个系统调用的过程，这里翻译自`xv6/chapter4.3,4.4`。

在上文 `initcode.S` 涉及第一个系统调用 `exec` 。我们就来看看 `exec` 在内核中是如何实现的吧。

在`user mode`，程序将 `exec` 使用的参数放在寄存器 `a0` 和 `a1` 中，然后把系统调用的编号放在
`a7` 中。系统调用的编号对应在 `syscalls` 数组里，保存着一系列函数的指针(kernel/syscall.c:108)。
指令 `ecall` 将陷入内核，然后执行 `uservec` 和 `usertrap` 和 `syscall` 三条指令。
(最后这句话我也不知道什么意思，应该是第四章前面部分的内容，这个可以先放一边。)

`syscall`(kernel/syscal.c:133) 从`trapframe`中 的 `a7` 中获得一个系统调用的编号，作为 
`syscalls` 中的下标。 这里涉及的第一个系统调用的编号是 `SYS_exec`(kernel/syscal.h:8)，也
就是会调用系统调用函数 `sys_exec`。

当系统调用函数返回时， `syscall` 会记录它的一个返回值到 `p->trapframe->a0` 中（RISC-V 返
回值就保存在 `a0` 中）。这时，用户空间调用的 `exec()` 将会返回这个值。通常，系统调用通过返回
负值来表示错误，0或正数表示成功。如果系统调用的编号错误，`syscall` 会返回一个错误值 `-1`。

这里 `trapframe` 陷阱到底是什么我们可以先放一边，后面的实验应该会涉及到。

接着我们来了解一下系统调用的参数是如何获取的。RISC-V 将参数放置在寄存器里，而`kernel trap`机制
将会把用户程序的寄存器放在当前进程的 `trapframe` 里，供内核使用。函数 `argint`，`argaddr` 
和 `argfd` 用于从 `trapframe` 里获得相应的参数以及文件描述符表。这些函数都使用 `argraw` 来
获得用户程序寄存器的内容(kernel/syscall.c:35)

一些系统调用的参数是指针，内核需要用这些指针在用户内存中读取或者写入内容。例如，`exec` 系统调用
就向内核传递了一个数组指针，来只想用户空间的字符串变量。这个面临两个挑战：用户程序有可能是有bug
或者有害的，会给内核穿第一个错误的指针，使得内核来访问到了不该访问的内存；第二个是内核的地址映射
表和用户程序不同，所以内核不能使用正常的操作来访问用户程序的内存空间。内核实现了一系列的函数来将
用户的内存中的内容安全地转移到内核空间。`fetchstr` 就是一个例子(kernel/syscall.c:25)。
文件系统调用 `exec` 使用 `fetchstr` 来获取用户空间的文件名。`fetchstr` 和 `copyinstr` 
来做这个困难的事情。

`copyinstr` (kernel/vm.c:406) 从用户页表`pagetable`的虚拟地址 `srcva` 复制了 `max` 
比特和 `dst`。 它使用了 `walkaddr`(调用了 `walk`) 来为 `srcva` 查找物理地址 `pa0`。
内核将物理内存地址全部映射到了一个相同的内核虚拟地址， `copyinstr` 能够直接从 `pa0` 复制字符
到`dst.walkaddr` (kernel/vm.c:95)。它在会检查用户提供的虚拟地址是属于这个进程用户地址空间的,
因此，程序不可能让内核读取其他地址的内存。类似的程序`copyout` 则是将数据从内核空间复制到这个程序
的用户地址空间。

具体在页表上做了什么事情也可以先放一边，后面的实验会涉及到。

## System call tracing

!!!题目描述
    增加一个系统调用 `trace` 帮助我们做后面的debug。它接收一个整数，来确定追踪哪一个系统调用。
    不过特别的是，这个整数应该是个 `mask`，例如，追踪 `fork` 需要输入 `trace(1 << SYS_fork)`
    （定义在 `kernel/syscall.h` 中）。需要修改 `xv6` 内核，使得每一个被追踪的系统调用将要
    返回时，输出一行内容：进程的id，系统调用的名字和返回值。`trace` 应该只需要追踪调用它的进程
    与子进程。

实验程序提供了一个用户可以使用的程序 `trace`（user/trace.c)。当系统调用 `trace` 完成后，
就可以使用这个程序了。

提示：

- 将 `$U/_trace` 加入到 `Makefile` 的 UPROGS 里；
- 在 `user/user.h` 中加入系统调用的声明；在`user/usys.pl`中加入 `stub`;在 `kernel/syscall.h`
中加入系统调用的编号;在 `kernel/sysproc.c` 中添加函数定义。`Makefile` 将使用perl脚本来创
建 `user/usys.S`, 一个真正的系统调用`stubs`，它使用了 RISC-V 的指令 `ecall` 来转换到内核里。
这时应该能通过了编译，但是无法使用 `trace`，因为我们还没有在内核中实现它；
- 将 `sys_trace()` 加入到 `kernel/sysproc.c` 中，将它的参数保存到 `proc` 结构体中
（kernel/proc.h)。使用`kernel/syscall.c` 中的函数来从用户空间获取系统调用的参数，它们的使
用例程在`kernel/sysproc.c` 中；
- 修改 `fork()` （kernel/proc.c) 将 `trace` 的 mask 从父程序复制到子程序；
- 修改 `syscall()`（kernel/syscall.c) 打印 `trace` 的输出。需要一个保存系统调用名字的数组。

???参考答案
    具体修改的地方很琐碎，大部分不需要知道细节，根据提示完全可以猜测出来该怎么做。

    1. `Makefile` 添加 `$U/_trace` 到152行；
    2. `user/user.h` 添加 `int trace(int)` 到26行;
    3. `user/usys.pl` 添加 `entry("trace")` 到 39行；
    4. 在 `sysproc.c` 中添加函数的定义

        ```c
        uint64
        sys_trace(void)
        {
            int n;
            if(argint(0, &n) < 0)
                return -1;
            myproc()->trace_mask = n;
            return 0;
        }
        ```

        对应要修改 `proc.h` 的结构体 `proc`，添加变量 `int trace_mask` 到独占区域
        （不需要加锁）。
        这里结构体 `proc` 实例化了一个全局变量，用于内核程序的通信。

    5. `syscall.h` 中添加声明 `#define SYS_trace  22`；

    6. `syscall.c` 中添加 `extern uint64 sys_trace(void);` 以及补充 `syscalls`;

    7. 修改 `syscall.c` 中的 `syscall` 函数：

        ```c
        void
        syscall(void)
        {
            int num;
            struct proc *p = myproc();

            num = p->trapframe->a7;
            if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
                p->trapframe->a0 = syscalls[num]();
                if(p->trace_mask & (1 << num)) {
                printf("%d: syscall %s -> %d\n", p->pid, syscalls_name[num], p->trapframe->a0);
                }
            } else {
                printf("%d %s: unknown sys call %d\n",
                        p->pid, p->name, num);
                p->trapframe->a0 = -1;
            }
        }
        ```

        对应要创建一个保存系统调用名字的数组 `syscalls_name`。
    
    8. 最后要修改 `proc.c` 中的 `fork` 函数来使得子进程也能追踪：
        添加 `np->trace_mask = p->trace_mask;`。

## Sysinfo

!!!题目描述
    实现一个系统调用 `sysinfo` 来收集一些系统的运行状态信息。`sysinfo` 接收一个指向 `sysinfo`
    结构体的指针(kernel/sysinfo.h)。内核需要填满这个结构体：`freemem` 填入剩余空闲内存；
    `nproc` 填入除了状态为 `UNUSED` 的进程数量。我们提供了一个测试程序 `sysinfotest`，当
    `sysinfo` 正确实现后，运行 `sysinfotest` 会输出 `sysinfotest: OK`。

提示：

- 将 `$U/_sysinfotest` 加入到 `Makefile` 的 `UPROGS` 里；
- 参考上一道题目，添加系统调用的声明，先通过编译。在 `user/user.h` 中添加 `sysinfo()` 时，
    可能需要显示声明结构体 `sysinfo`。这一步先做到编译不报错;
- `sysinfo` 需要拷贝 `struct sysinfo` 到用户空间；参考 `sys_fstat()` (kernel/sysfile.c)
    和 `filestat()`(kernel/file.c) 来看一下如何使用 `copyout()`;
- 向 `kernel/kalloc.c` 里添加一个函数，来获取当前空闲内存的大小；
- 向 `kernel/proc.c` 添加一个函数，来获取进程数量。

???参考答案
    
    1. 在 `Makefile` 中添加 `$U/_sysinfotest` 到 `UPROGS` 中；
    2. 在 `user/usys.pl` 中添加 `entry("sysinfo")`;
    3. 在 `user/user.h` 中添加 `struct sysinfo` 的声明，以及系统调用 
        `int sysinfo(struct sysinfo*)`;
    4. 在 `kernel/syscall.h` 中添加系统调用编号 `#define SYS_trace 23`;
    5. 在 `kernel/syscall.c` 中添加 `extern uint64 sys_sysinfo(void)`，
        并且补充 `syscalls` 和 `syscalls_name` 两个数组；
    6. 在 `kernel/sysproc.c` 中实现 `uint64 sys_sysinfo(void)`函数。
        可以先是一个空函数，这样已经可以通过编译了。实际的参考实现如下：

        ```c
        uint64
        sys_sysinfo(void)
        {
            struct sysinfo info;
            struct proc *p = myproc();
            uint64 addr;
            info.freemem = getfreemem();
            info.nproc = getnproc();
            if(argaddr(0, &addr) < 0 || 
                copyout(p->pagetable, addr, (char *)&info, sizeof(info)) < 0)
                return -1;
            return 0;
        }
        ```

    7. `sys_sysinfo` 调用了两个自定义的函数。
        
        - 在 `kernel/kalloc.c` 中实现读取空闲内存的函数 `getfreemem`:

            ```c
            uint64 getfreemem(void)
            {
                uint64 freemem = 0;
                struct run *r;
                acquire(&kmem.lock);
                r = kmem.freelist;
                for(r = kmem.freelist; r; r = r->next)
                    freemem += PGSIZE;
                release(&kmem.lock);
                return freemem;
            }
            ```

            并且在 `defs.h:66` 中声明函数。
        - 在 `proc.h` 和 `proc.c` 中实现函数 `getnproc` 获取进程数：
            
            ```c
            uint64
            getnproc(void)
            {
                struct proc *p;
                uint64 nproc = 0;
                for(p = proc; p < &proc[NPROC]; p++) {
                    if(p->state != UNUSED) {
                    nproc++;
                    }
                }
                return nproc;
            }
            ```

## 总结

如果干巴巴地抄答案，按照答案来做，那就基本上什么也没搞明白。这个实验通过提示，来促使我们来阅读关于
系统调用的代码，通过代码大概推断出答案的过程也就是搞明白每个文件的作用以及系统调用是如何实现的。