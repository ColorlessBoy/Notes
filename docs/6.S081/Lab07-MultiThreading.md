# Lab7: Multithreading

这个实验是为了熟悉多线程。在实验中我们将实现一个用户层级的多线程调度器，来加速进程的运行速度，并且实现一个`barrier`。 这一章需要阅读`xv6-book`第七章的内容。

切换到本章实验的分支为：

```bash
git betch
git checkout thread
make clean
```

## Uthread: switching between threads

本节要设计一个上下文切换机制（context switch mechanism）来实现一个用户层级的多线程系统。有两个关键的文件`user/uthread.c`和`user/uthread_switch.S`,并且在`Makefile`中构建了一个多线程程序。`uthread.c`包含大部分用户层级的多线程包，并且程序包含三个简单的线程测试程序。现在多线程包缺失了一些代码来创建和切换线程。

!!!问题描述
    补全代码来通过`uthread`测试。首先需要补全`user/thread.c`中的代码`thread_create()`和`thread_schedule()`，以及`user/uthread_switch.S`中的代码`thread_switch`。
    
    - 目标一：保证`thread_schedule()`执行第一次执行线程时，线程调用函数`thread_create()`，创建它自己的栈空间；
    - 目标二：保证`thread_switch`能够在切换线程时正确地保存寄存器，并且正确地恢复寄存器。这里可以修改`struct thread`来保存寄存器。在`thread_schedule`中需要调用`thread_switch`，最终实现从线程`t`切换到线程`next_thread`。

提示：

- `thread_swtich`只需要保存`callee-save`寄存器；
- Debug可以参考`user/uthread.asm`，具体方式是

    ```bash
    (gdb) file user/_uthread
    Reading symbols from user/_uthread...
    (gdb) b uthread.c:60
    ```

    下面是一些[gdb](https://sourceware.org/gdb/current/onlinedocs/gdb/)的命令

    ```bash
    (gdb) p/x *next_thread
    (gdb) x/x next_thread->stack
    (gdb) b thread_switch
    (gdb) c
    (gdb) si
    ```

## Using threads

在这一节将实现一个多线程编程，并且实现一个多线程哈希表。具体是使用UNIX的`pthread`多线程库。`man pthread`可以查看具体文档，或者查看网站文档[here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_lock.html),[here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_init.html),[here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_create.html)。

`notxv6/ph.c`包含一个简单的哈希表，它在单线程运行时是正常的。具体编译方式是（使用本地编译器）

```bash
make ph
./ph 1
```

也可以输入`./ph 2`来使用多核心来运行这个哈希表。

!!!问题1
    为什么多线程会导致原本的程序报错。

!!!问题2
    加入多线程锁的声明

    ```c
    pthread_mutex_t lock;            // declare a lock
    pthread_mutex_init(&lock, NULL); // initialize the lock
    pthread_mutex_lock(&lock);       // acquire lock
    pthread_mutex_unlock(&lock);     // release lock
    ```

    这时`make grade`可以实现`ph_safe`测试。这样只会报`0`关键词缺失。

不要忘记调用`pthread_mutex_init()`。可以尝试为每个哈希桶单独分配一个锁来加速`put()`。

!!!问题3
    修改`put`来保证多线程程序的正确性，运行`make grade`来通过`ph_safe`和`ph_fast`两个测试。

## Barrier

实现一个函数`barrier`来使得所有线程同步到该函数。需要使用`pthread`条件变量，类似于`xv6`中的`sleep`和`wakeup`。本节的测试将在真实计算机上实现。具体执行如下

```bash
make barrier
./barrier 2
```

!!!问题描述
    我们的目标是实现`barrier`。具体包含一些`ph`题目中的锁，以及两个新的请求（具体可以参考[here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_wait.html)和[here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_broadcast.html)）

    ```c
    pthread_cond_wait(&cond, &mutex);  // go to sleep on cond, releasing lock mutex, acquiring upon wake up
    pthread_cond_broadcast(&cond);     // wake up every thread sleeping on cond
    ```

    最终通过`make grade`的测试`barrier`。

已经实现了`barrier_init()`和`struct barrier`。

提示：

- 核心是要同步`bstate.round`；
- 需要注意循环中一个程序已经退出了，另一个程序还在等待的情况。保证进程在当前轮次更新`bstate.nthread`前，上一个轮次没有其他线程更新`bstate.nthread`。


