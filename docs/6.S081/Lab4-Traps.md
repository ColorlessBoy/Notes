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
- `main`中调用`f`的语句是哪些，调用`g`的语句是哪些?
- `printf`的地址是什么？
- `main`中`printf`的`jalr`指令运行后，`ra`寄存器保存了什么值？
- 下面的代码将输出什么？输出值取决于RISC-V是大端还是小端。如果是打断，应该改变变量成什么
  才能保证输出不变？57616需要改变吗？

    ```c
    unsigned int i = 0x00646c72;
    printf("H%x W0%s", 57616, &i);
    ```

- `printf("x=%d y=%d", 3)` 会输出什么？

