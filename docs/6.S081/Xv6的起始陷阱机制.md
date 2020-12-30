# Xv6的起始陷阱机制

## 问题描述：`uservec`是什么时候被设置的？
在前面的实验中，有一些重要的问题没有搞清楚，最终反映到陷阱机制里面，就是第一个`usertrap`怎么来的。

[Xv6-book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)的第四章介绍了如何从用户态产生陷阱，总体流程是：

$$
uservec \rightarrow usertrap \rightarrow usertrapret \rightarrow userret.
$$

但是Xv6启动的整体流程（第二章）是

$$
bookloader \rightarrow \_entry \rightarrow start \rightarrow main \rightarrow userinit,
$$

并且其中最后的主函数`main`中只调用了`trapinithart`函数将中断处理程序寄存器`stvec`设置成了内核态的中断处理程序`kernelvec`，并没有设定用户态的中断处理程序`uservec`。那么`uservec`是什么时候被设置的呢？

## 解释

从代码来看，内核仅仅在`kernel/trap.c:usertrapret()`函数中有代码设置`uservec`：

```c
    w_stvec(TRAMPOLINE + (uservec - trampoline));
```

但是从第四章来看`usertrapret`又是由`uservec`函数产生的，变成了一个 **鸡生蛋蛋生鸡** 的问题。我产生这个困惑的原因就是我没有搞清楚`Xv6`的启动机制。

从代码来看，还有一个地方调用了函数`usertrapret`，那就是`proc.c:forkret()`中。而`forkret()`被`allocproc`设置在了进程结构中。最后回溯发现，`uservec`中调用了`allocproc`。最终破解疑问，其实在`userinit`中隐式地设定了`uservec`。整体流程是

$$
bookloader \rightarrow \_entry \rightarrow start \rightarrow main \rightarrow userinit \rightarrow allocproc \rightarrow forkret \rightarrow usertrapret \rightarrow uservec.
$$

虽然这么简单，但是我还是分析了很久才搞清楚，关键是没有找对分析方法，没有沿着`usertrapret`这条线往前回溯，而是瞎找一通。