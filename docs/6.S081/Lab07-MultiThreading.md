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

- `thread_swtich`只需要保存`callee-save`寄存器；（原因请写在`answers-thread.txt`文件中）
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

!!!Note
    我发现这道实验题还需要两个提示：

    - `ra`寄存器是保存返回地址的寄存器，可以用于修改`uthread_shedule()`的返回地址；
    - `sp`寄存器保存的是栈地址，并且riscv的栈指针是从高地址往低地址更新的。

???参考答案

    1. `uthread`结构体的定义，增加一个关于`context`的变量。

        ```c
        struct uthread_context {
            uint64 ra;
            uint64 sp;

            // callee-saved
            uint64 s0;
            uint64 s1;
            uint64 s2;
            uint64 s3;
            uint64 s4;
            uint64 s5;
            uint64 s6;
            uint64 s7;
            uint64 s8;
            uint64 s9;
            uint64 s10;
            uint64 s11;
        };

        struct thread {
            char       stack[STACK_SIZE]; /* the thread's stack */
            int        state;             /* FREE, RUNNING, RUNNABLE */
            struct uthread_context context; /* context for user thread */
        };
        ```

    2. 更新`context`的函数`uthread_switch.S`和内核中的切换函数`swtch`一模一样。

        ```c
        .globl swtch
        swtch:
                sd ra, 0(a0)
                sd sp, 8(a0)
                sd s0, 16(a0)
                sd s1, 24(a0)
                sd s2, 32(a0)
                sd s3, 40(a0)
                sd s4, 48(a0)
                sd s5, 56(a0)
                sd s6, 64(a0)
                sd s7, 72(a0)
                sd s8, 80(a0)
                sd s9, 88(a0)
                sd s10, 96(a0)
                sd s11, 104(a0)

                ld ra, 0(a1)
                ld sp, 8(a1)
                ld s0, 16(a1)
                ld s1, 24(a1)
                ld s2, 32(a1)
                ld s3, 40(a1)
                ld s4, 48(a1)
                ld s5, 56(a1)
                ld s6, 64(a1)
                ld s7, 72(a1)
                ld s8, 80(a1)
                ld s9, 88(a1)
                ld s10, 96(a1)
                ld s11, 104(a1)
                
                ret
        ```

    3. 核心难点就是`thread_create`函数的实现，对寄存器的更新需要一定的先验知识。

        ```c
        void 
        thread_create(void (*func)())
        {
            struct thread *t;

            for (t = all_thread; t < all_thread + MAX_THREAD; t++) {
                if (t->state == FREE) break;
            }
            t->state = RUNNABLE;
            // YOUR CODE HERE
            t->context.ra = (uint64)func;
            t->context.sp = (uint64)(t->stack + STACK_SIZE);
        }
        ```

    4. 对`thread_schedule()`函数的补充就比较简单了，和内核中的多线程一样。

        ```c
        thread_switch((uint64)&t->context, (uint64)&current_thread->context);
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

???参考答案
    1. 两个线程交替运行`put`会造成`table`后面添加元素时后面的操作把前面的操作覆盖，所以造成关键词缺失；
    2. 首先在`user/ph.c:20`处声明线程锁

        ```c
        pthread_mutex_t lock[NBUCKET];            // declare a lock
        ```
    
    3. 接着修改`user/ph.c:put()`函数

        ```c
        static 
        void put(int key, int value)
        {
            int i = key % NBUCKET;

            // is the key already present?
            struct entry *e = 0;

            pthread_mutex_lock(&lock[i]);       // acquire lock
            // ...
            pthread_mutex_unlock(&lock[i]);     // release lock
        }
        ```

    4. 最后在`user/ph.c:main()`中启动锁

        ```c
        for(int i = 0; i < NBUCKET; ++i)
            pthread_mutex_init(&lock[i], NULL); // initialize the lock
        ```

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

!!!Note
    - 本实验题也是需要`pthread_mutex_lock`和`pthread_mutex_unlock`来防止产生死锁。
    - 本实验需要了解`pthread_cond_broadcast`和`pthread_cond_waint`两个函数的作用，尤其是wait，它关系到锁的解除和申请。网上稍微查点例程就搞明白了。

??? 参考答案

    ```c
    static void 
    barrier()
    {
        pthread_mutex_lock(&bstate.barrier_mutex);       // acquire lock
        if(++bstate.nthread == nthread) {
            bstate.nthread = 0;
            bstate.round++;
            pthread_cond_broadcast(&bstate.barrier_cond);
            pthread_mutex_unlock(&bstate.barrier_mutex); 
        } else {
            pthread_cond_wait(&bstate.barrier_cond, &bstate.barrier_mutex);
            pthread_mutex_unlock(&bstate.barrier_mutex); 
        }
    }
    ```