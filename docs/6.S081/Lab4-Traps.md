# Lab4: Traps

这个实验将探究如何使用`trap`来实现系统调用。首先要先完成一个关于栈的热身练习，完成一个
用户层级的陷阱处理器。到目前为止，我感觉6.S081-2020设计的实验还是非常合理的，每一个实验
都正是我正在困扰的内容。

在以前，这个实验的一些题目是放在`Lab1`里的，劝退了很多人吧。现在阅读过一些源码后应该好很多。

在实验前，课程建议我们先阅读一下`xv6-book:ch4`的内容，以及相关源码：

- `kernel/trampoline.S`:负责用户空间和内核空间的转换的汇编程序；
- `kernel/trap.c`:负责处理各种中断信号。

获取实验代码：

```shell
$ git fetch
$ git checkout traps
$ make clean
```

## 阅读材料

`trap`分为三种：`ecall`引发的系统调用；异常；设备中断。用户应该感受不到对中断的处理，
所以内核应该在中断前保存寄存器内容，中断处理完成后恢复寄存器。`xv6`处理中断程序分为四个
步骤：处理器接收中断信号；汇编向量为内核的C代码准备一些东西，内核中陷阱处理器决定该怎么
处理这个陷阱，最后运行系统调用和驱动程序。`xv6`使用三类陷阱处理器：处理用户态的陷阱、
处理内核态的陷阱和时间中断。

### RISC-V 的陷阱机制

处理器使用一系列控制寄存器来完成陷阱机制，这里介绍几个重要的寄存器：

- `stvec`: 内核把陷阱处理器的地址写到这个寄存器里，处理器将跳转到这个地址。
- `sepc`: 记录`pc`, 然后`pc`被`stvec`替换。`sret`会拷贝`stvec`到`pc`。内核能够通过修改
    `sepc`来控制返回。
- `scause`: `trap`的原因。
- `sscratch`: 帮助陷阱处理器的启动的一些有用的值。
- `sstatus`: 一些状态值，`SIE`位控制是否允许设备中断，`SPP`位标志着陷阱是来自用户态
    还是内核态。

处理器处理中断的具体流程是：

- 如果是设备中断，`SIE`清零，程序无限等待设备将它置1。
- 关闭中断 `SIE` 清零。
- 拷贝 `pc` 到 `sepc`。
- 根据当前模式设置`sstatus`中的`SPP`位。
- 把终端原因写入 `scause`。
- 转到 `supervisor` 模式。
- 拷贝 `stvec` 到 `pc`。
- 执行新的`pc`处的指令。

处理器并没有转换内核页，内核栈，以及除了`pc`以外的寄存器。内核程序需要完成这些事情。

### 从用户空间引发陷阱

当陷阱发生在用户空间时，内核开始执行 `uservec`(kernel/trampoline.S:16)，`usertrap`
(kernel/trap.c:37), `usertrapret`(kernel/trap.c:90) 和 `userret`(kernel/trampoline.S:16)。

从内核空间引发陷阱非常具有挑战性，`satp`指向的是用户页表，并不是内核页表，而且栈指针会指向
非法的内容，甚至会造成极大的危害。

因为RISC-V硬件没有转换页表，所以用户页表需要对`stvec`指向的`uservec`做映射。`uservec`
需要从用户页表转换为内核页表，来执行后面的程序。因此在用户页表和内核页表中，需要将`uservec`
映射到相同的虚拟地址，`xv6`通过`trampoline`页来实现这个要求。虚拟地址`TRAMPOLINE`保存
的内容由`trampoline.S`设置，用户模式中`stvec`设置到`uservec`中(kernel/trampoline.S:16)。

当`uservec`开始，所有32个寄存器包含中断代码相关信息。但是`uservec`需要修改一些寄存器并且
保存一些寄存器。RISC-V提供了`sscratch`寄存器来保存帮助信息。比如`trampoline.S:137`
行使用`csrrw`来保存`sscratch`到`a0`。

`uservec`接着要保存用户的寄存器信息。在进入用户空间前，内核设置`sscratch`指向用户的
`trapframe`, 在`trapframe`中保存用户的寄存器(kernel/proc.h:44)。但是`satp`仍然指向
用户页表，`uservec`需要映射`trapframe`。`xv6`给每个进程页表映射一个`trapframe`到虚拟
地址`TRAPFRAME`中，就在`TRAMPOLINE`下面。`p->trapframe`就指向`trapframe`的物理地址，
让内核直接访问这里。 换出`sscratch`后，`a0`包含进程的`trapframe`，然后`uservec`保存用
户的寄存器到`sscratch`。

`trampframe`包含当前进程的`kernel stack`，当前CPU的图标，`usertrap`的地址，内核页表
地址，`uservec`获得这些内容，设置`satp`切换到内核页表，然后调用`usertrap`。

`usertrap`的作用是确定陷阱的原因，执行它，然后返回(kernel/trap.c:37)。它首先改变`stvec`，
所以内核能够开始处理`kernelvec`。然后保存`sepc`，防止进程切换后，`sepc`被重写。
如果陷阱是系统调用，`syscall`接手它；如果是设备中断，`devintr`接手它；如果是异常，
内核将杀死进程。在系统调用中，系统将`pc`加4，使得程序指向`ecall`指令。在结束`usertrap`
前，`usertrap`检查进程是否被杀死，或者应该释放CPU（如果是时钟中断）。

当返回到用户空间后，第一条是调用`usertrapret`(kernel/trap.c:90)。这个函数设置RISC-V
寄存器准备好处理后面的用户空间的陷阱。包括：修改`stvec`指向`uservec`，准备`trapframe`，
设置`sepc`到之前保存的用户程序。最后`usertrapret`调用在trampoline中的`userret`。
`userret`将会转换页表。

`usertrapret`调用`userret`来传递一个指针，指向保存进程用户页表`a1`，和保存TRAPFRAME的`a0`
(kernel/trampoline.S:88)。`userret`转换`satp`到用户页表。`userret`将拷贝`trampframe`
中保存的`a0`来准备后面交换`TRAPFRAME`。`userret`唯一能用的是寄存器内容和`trapframe`中
的内容。然后`userret`从`trapframe`中复原用户程序的寄存器，交换`a0`与`sscratch`中的内容，
恢复用户的`a0`，为下一次陷阱保存`TRAPFRAME`。最后使用`sret`返回用户空间。

> 后面的阅读内容就不翻译了，基本上把用户态处理陷阱的流程走通了，其他流程就非常容易理解了。

## RISC-V 汇编

这里要通过分析`user/call.c`来学习一下RISC-V的汇编语言。 调用`make fs.img`来编译它，
并且生成可读的汇编版本的程序`user/call.asm`。

阅读`call.asm`的函数`g`，`f`和`main`。回答下面的问题:

- 哪些寄存器包含函数的参数?当`main`调用`printf`时，哪个寄存器保存了`13`?
    使用`a0-a7`寄存器来保存函数变量。这里`printf`使用`a2`保存`13`。
- `main`中调用`f`的语句是哪些，调用`g`的语句是哪些?
    被优化了，没有调用这两个函数。
- `printf`的地址是什么？起始地址是`0x630`。
- `main`中`printf`的`jalr`指令运行后，`ra`寄存器保存了什么值？ `ra`保存值是`0x38`。
- 下面的代码将输出什么？输出值取决于RISC-V是大端还是小端。如果是大端，应该改变变量成什么
  才能保证输出不变？57616需要改变吗？

    ```c
    unsigned int i = 0x00646c72;
    printf("H%x Wo%s", 57616, &i);
    ```

    输出的`HE110 World`，如果是大端则`i = 0x726c6400`。

- `printf("x=%d y=%d", 3)` 会输出什么？`y=`后面输出的是`a2`寄存器的内容，但是没有赋值，
    所以要看运行到这里的时候`a2`被其他程序赋值成什么值。

!!!Note
    这里要理解RISC-V函数调用的时候，规定的堆栈变化标准。在课程视频的第五节有专门的讲解，
    这一节 还讲了很多gdb的调试方法，非常值得看一下。
    另外这里有一篇[博客](https://twilco.github.io/riscv-from-scratch/2019/07/28/riscv-from-scratch-4.html)真不错。

## Backtrace

实现一个`gdb`中很有用的指令`backtrace`，当遇到运行出错时，打印函数调用栈上面的指针情况。

!!!题目描述
    在`kernel/printf.c`实现一个`backtrace()`函数。在`sys_sleep`调用这个函数。然后运行
    `bttest`命令，它会呼叫`sys_sleep`。输出应该是：
    
    ```bash
    backtrace:
    0x0000000080002cda
    0x0000000080002bb6
    0x0000000080002898
    ```

    之后退出`qemu`。在终端输入：地址会有一些不同，可以使用`addr2line -e kernel/kernel`，
    或者`riscv64-unknown-elf-addr2line -e kernel/kernel` 然后复制`bttest`输出的结果：

    ```bash
    $ addr2line -e kernel/kernel
    0x0000000080002de2
    0x0000000080002f4a
    0x0000000080002dfc
    Ctrl-D
    ```

    然后我们会看到
    
    ```
    kernel/sysproc.c:74
    kernel/syscall.c:224
    kernel/trap.c:85
    ```

编译器给一片一片的栈区放置一个指针，保存着`caller`的片指针。`backtrace`应该用到这些栈指针
沿着栈去搜索保存着的指针。

一些提示：

- 在`kernel/defs.h`声明`backtrace`，然后就能在`sys_sleep`中调用`backtrace`。
- 在GCC编译器中保存着当前函数的片指针$s0$。在`kernel/riscv.h`中添加函数：

    ```c
    static inline uint64
    r_fp()
    {
        uint64 x;
        asm volatile("mv %0, s0" : "=r" (x));
        return x;
    }    
    ```

    然后再`backtrace`中调用这个函数来读取当前的片指针。
- 在课程笔记中有一张图片画着栈片的结构。返回地址在栈指针的(-8)位置，保存的栈指针在当前
    栈指针的(-16)的位置。
- `Xv6` 给栈分配一个内存页。可以使用`PGROUNDDOWN(fp)`和`PGROUNDUP(fp)`（看`kernel/riscv.h`。
    这个数字有助于`backtrace`结束循环。）

一旦`backtrace`有用，在`kernel/printf.c:panic`中调用它，当内核出错时打印`backtrace`。

???参考答案
    其中用于判断的`0x3fffff9000`是试出来的。

    ```c
    void
    backtrace(void)
    {
        printf("backtrace:\n");
        uint64 *p = (uint64 *)r_fp();
        while(PGROUNDDOWN((uint64)p) == 0x3fffff9000){
            printf("%p\n", p[-1]);
            p = (uint64 *)p[-2];
        }
    }
    ```

## Alarm

!!!题目描述
    为`xv6`增加一个新特性：周期性地提示程序使用的CPU时间。这是一个基础的用户级别的中断/
    错误程序，需要通过`alarmtest`和`usertests`。

需要添加系统调用`sigalarm(interval, handler)`。调用`sigalarm(n, fn)`，用户程序每n个
CPU时钟周期，内核就会调用一下`fn`，当`fn`执行完毕，继续执行用户程序。`sigalarm(0, 0)`
应该不产生中断。

将`user/alarmtest.c`加入到`Makefile`，它需要系统调用`sigalarm`和`sigreturn`。
`alarmtest.c`调用`sigalarm(2, periodic)`。

需要实现的代码可能很短，但是非常巧妙。

> 我建议`sys_sigalarm(void)`和`sys_sigreturn(void)`的实现放到`sysproc.c`中。

### 针对`test0`

提示：

- 将`alarmtest.c`加入到`Makefile`。
- 在`user/user.h`中加入声明：

    ```c
    int sigalarm(int ticks, void (*handler)());
    int sigreturn(void);
    ```

- 更新`user/usys.pl`，`kernel/syscall.h` 和 `kernel/syscall.c` 来创建这两个系统调用。
- 目前`sys_sigreturn`目前只需要返回0。
- `sys_sigalarm()`应该在`proc`(`kernel/proc.h`)结构中保存`interval`和指针。
- 需要记录程序已经用过了多少CPU时钟周期，在`proc`中保存这个内容。在`proc.c/allocproc()`中
    初始化为零。
- 只需要操作时钟周期，例如：`if(which_dev == 0)`。
- 只有进程有未完成的计时器时才会启用警报方程。用户的警告函数地址可能是0（`user/alarmtest.asm`
    里显示`periodic`地址为0）。
- 需要修改`usertrap()`来使得警告中断暂时失效时，运行handler函数。当陷阱返回用户空间时，
    什么指令决定了返回用户空间执行什么指令？
- 当只用一个CPU时比较好使用gdb调试，可以使用`make CPUS=1 qemu-gdb`。
- 这时候应该能成功打印`alarm`。

### 针对`test1/test2`

警告指令的最后需要调用`sigreturn`，参考`alarmtest.c:periodic()`。需要修改`usertrap`
和`sys_sigreturn`来保证程序恢复正常运行。

提示：

- 需要保存一系列的寄存器；
- 当计时器关闭时，`usertrap`在`struct proc`中保存重要的信息，以便`sigreturn`返回正
    确的用户代码。
- 防止程序重入调用——当处理程序还没有返回，内核不应该继续调用它。

???参考答案
    1. 关于添加系统调用等操作在提示里面已经比较到位了。
    2. 针对`struct proc`结构体的修改。添加成员变量

        ```c
        int interval;                // Alarm interval
        uint64 handler;              // Handler function
        int ticks;                   // The number of ticks since last call
        int inhandler;               // inhandler != 0 when the process is running in handler function
        struct trapframe handler_trapframe; // The trapframe before into handler
        ```

    3. 在`trap.c:79`中添加如下程序段，用于调用中断函数。需要注意的是，`trapframe`是指针，
        `handler_trapframe`是结构体。

        ```c
        // give up the CPU if this is a timer interrupt.
        if(which_dev == 2){
            if(p->inhandler == 0) {
                p->ticks += 1;
                if(p->interval > 0 && p->ticks % p->interval == 0){
                    p->handler_trapframe = *(p->trapframe);
                    p->inhandler = 1;
                    p->ticks = 0;
                    p->trapframe->epc = p->handler;
                }
            }
            yield();
        }
        ```

    4. 在`sysproc.c`中定义系统调用。同样需要注意，`trapframe`是指针，`handler_trapframe`
        是结构体。

        ```c
        // alarm system call
        uint64
        sys_sigalarm(void)
        {
            struct proc *p = myproc();
            if(argint(0, &(p->interval)) < 0)
                return -1;
            if(argaddr(1, &(p->handler)) < 0)
                return -1;
            return 0;
        }

        uint64
        sys_sigreturn(void)
        {
            struct proc *p = myproc();
            p->inhandler = 0;
            *(p->trapframe) = p->handler_trapframe;
            return 0;
        }
        ```