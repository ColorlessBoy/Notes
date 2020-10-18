# Xv6 and Unix utilities

这个实验是要熟悉一下 `xv6` 和它的系统调用。

## 启动 `xv6`

我们需要下载实验用的代码 `xv6-labs-2020`，然后切换到 `util` 分支：

```bash
$ git clone git://g.csail.mit.edu/xv6-labs-2020
Cloning into 'xv6-labs-2020'...
...
$ cd xv6-labs-2020
$ git checkout util
Branch 'util' set up to track remote branch 'util' from 'origin'.
Switched to a new branch 'util'

$
```

然后使用命令 `make qemu` 启动操作系统。

我们先使用第一个系统调用 `ls`，可以看到有一系列文件。在最初的文件系统里，他们被包含在 `user` 文件夹里，很多程序都能使用。

```bash
.              1 1 1024
..             1 1 1024
README         2 2 2059
xargstest.sh   2 3 93
cat            2 4 24256
echo           2 5 23080
forktest       2 6 13272
grep           2 7 27560
init           2 8 23816
kill           2 9 23024
ln             2 10 22880
ls             2 11 26448
mkdir          2 12 23176
rm             2 13 23160
sh             2 14 41976
stressfs       2 15 24016
usertests      2 16 148456
grind          2 17 38144
wc             2 18 25344
zombie         2 19 22408
console        3 20 0
```

`xv6` 没有实现 `ps` 命令，但是可以使用 `ctrl+p` 来打印现在的进程信息。

```bash
$ 
1 sleep  init
2 sleep  sh
```

按 `ctrl+a x` 退出 `qemu`。

## 打分

可以使用 `make grade` 来测试答案。

## sleep

实现 `user/sleep.c` 的功能。

- 要阅读 `xv6 book` 的第一章；
- 查看 `user/echo.c`, `user/grep.c` 和 `user/rm.c` 源文件，看看怎么实现系统调用，其实就是看看C语言怎么获取命令行参数；
- 忘记输入参数的时候需要输出错误信息；
- 输入的参数是字符串，可以使用 `atoi` 来转换类型（查看`user/ulib.c`);
- 使用系统调用 `sleep`;
- 查看 `kernel/sysproc.c` 看看系统调用 `sleep` 是如何实现的(看 `sys_sleep`部分)，`user/user.h` 看C版本的 `sleep` 如何定义，然后看 `user/usys.pl` 看用户调用如何进入系统调用 `sleep` 的。
- 确定 `exit()` 返回正确的值；
- 将 `sleep` 加入到 `Makefile` 的 `UPROGS` 里，然后就可以在 `shell` 里使用这个命令了。

几乎照着提示做就好了，我们使用 `./grade-lab-util sleep` 或者 `make GRADEFLAGS=sleep grade` 来做测试。简单的答案：

??? 参考答案
    ```c
    #include "kernel/types.h"
    #include "kernel/stat.h"
    #include "user/user.h"

    int
    main(int argc, char *argv[])
    {
      int time;
      if(argc < 2){
          fprintf(2, "Usage: sleep NUMBER\n");
          exit(1);
      }
      time = atoi(argv[1]);
      sleep(time);
      exit(0);
    }
    ```

## pingpong

!!!题目描述
    实现一个文件 `user/pingpong.c`:
    要求父程序给子程序发送一个字节，子程序打印`<pid>:received ping`，其中 `<pid>` 是子程序
    的进程号。然后子程序发送给父程序一个字节，并退出。父程序读到子程序发送的字节后，打印 
    `<pid>: received pong` 并退出。

提示：

- 使用 `pipe` 创建管道；
- 使用 `fork` 创建子进程；
- 使用 `read` 从管道读取数据；
- 使用 `write` 向管道写数据；
- 使用 `getpid` 从进程读取进程号；
- 将 `pingpong.c` 加入到 `Makefile` 的 `UPROGS` 中；
- 系统调用都在 `user/user.h` 中，具体怎么调用的还需要再分析。

这道题目几乎是 [xv6-book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)
第一章pipe例程的翻版，只要看懂了这一节内容就能够做成这道题目，其实是比较友好的。

???参考答案
    这里我创建了两个 `pipe`，分别用于 `parent` 向 `child` 发送字符 `a`，以及 `child` 向
    `parent` 发送字符 `b`，保证发送的顺序。

    ```c
    #include "kernel/types.h"
    #include "kernel/stat.h"
    #include "user/user.h"

    int
    main(int argc, char *argv[])
    {
      int p1[2], p2[2], pid;
      char buffer[2]; // one bit
      pipe(p1); pipe(p2);

      // buffers and pids will be different between child and parent.
      if(fork() == 0) {
        //child proc
        close(p1[1]); close(p2[0]);
        read(p1[0], buffer, 1); 
        if(buffer[0] == 'a') {
          pid = getpid();
          printf("%d: received ping\n", pid);
          buffer[0] = 'b';
          write(p2[1], buffer, 1);
        } else {
          fprintf(2, "Child receives a wrong bit!\n");
          exit(1);
        }
        close(p1[0]); close(p2[1]);
      } else {
        // parent proc
        close(p1[0]); close(p2[1]);
        buffer[0] = 'a';
        write(p1[1], buffer, 1);
        read(p2[0], buffer, 1);
        if(buffer[0] == 'b') {
          pid = getpid();
          printf("%d: received pong\n", pid);
        } else {
          fprintf(2, "Parent receives a wrong bit!\n");
          exit(1);
        }
        close(p1[1]); close(p2[0]);
      }
      exit(0);
    }
    ```

## primes

!!!题目描述
    实现文件`user/primes.c`：使用pipes来编写一个素数程序。程序的思路来自Unix的创始者
    Doug McIlroy，可以参考[这个网页](https://swtch.com/~rsc/thread/)。

我们的目标是使用 `pipe` 和 `fork` 来创建pipeline。第一个进程发送 2 到 35 到管道中。
每一个素数对应一个进程。每个进程从左边邻居读入数据，从右边管道发送数据。因为 `xv6` 的
文件描述符和进程数有限，所以第一个进程到 35 就结束。

提示：

- 注意要及时关闭进程不需要的文件描述符，否则 `xv6` 将连 35 都计算不到；
- 开始的主进程需要在所有子进程结束后才能退出；
- `read` 在 `write` 关闭时会返回 0；
- 可以直接将 32-bit 的 `int` 写到 pipes 里， 而不需要转换为ASCII 码再输入输出；
- 仅当有需要时创建进程。

???参考答案

    ```c
    #include "kernel/types.h"
    #include "kernel/stat.h"
    #include "user/user.h"

    int
    main(int argc, char *argv[])
    {
        int i, p[2], p_close[2];
        pipe(p);
        pipe(p_close);

        if(fork() == 0) {
            //child process
            int p2[2], base_prime, num, flag = 0;
            close(p[1]); close(p_close[0]);
            read(p[0], &base_prime, 4);
            printf("prime %d\n", base_prime);
            while(read(p[0], &num, 4) == 4) {
                if(num % base_prime != 0) {
                    if(!flag) {
                        pipe(p2);
                        if(fork() == 0) {
                            close(p[0]); close(p2[1]);
                            p[0] = p2[0];
                            read(p[0], &base_prime, 4);
                            printf("prime %d\n", base_prime);
                        } else {
                            flag = 1;
                            close(p2[0]);
                            write(p2[1], &num, 4);
                        }
                    } else {
                        write(p2[1], &num, 4);
                    }
                }
            }
            close(p2[1]);
            close(p[0]); close(p_close[1]);
        } else {
            //parent process
            close(p[0]);
            close(p_close[1]);
            for(i = 2; i <= 35; ++i) {
                write(p[1], &i, 4);
            }
            close(p[1]);

            // wait for all child process.
            // when p_close[1] is closed, read will get 0x00.
            read(p_close[0], &i, 1);
            close(p_close[0]);
        }
        exit(0);
    }
    ```

如果最初的主程序不等待所有子进程结束，那么 `shell` 会出现bug。比如，将 `read(p_close[0], &i, 1)`
注释掉，最初的主进程提前结束，输出将会如下：

```shell
xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$ primes
pri$m e 2
prime 3
prime 5
prime 7
prime 11
prime 13
prime 17
prime 19
prime 23
prime 29
prime 31
x
```

`x` 表示光标的位置，也就是说 `shell` 会提前输出 `$` 然后卡住，必须要手动按下回车键才行。
另外，因为上面的bug导致打分程序 `grade-lab-util` 会死循环。

## find

!!!问题描述
    实现一个简单版本的 UNIX 的 `find` 程序 `user/find.c`： 找到所有目录树下的文件。

提示：

- 参考 `user/ls.c` 来查看如何读取目录；
- 使用递归形式不断向子目录查找；
- 不要查找 "." 和 ".." 文件夹；
- 对文件系统的更改会一直保存在 `qemu` 的运行过程中；使用 `make clean` 来清除系统文件，然后再
  使用 `make qemu`;
- 使用 C语言 风格的字符串，同时使用 `strcmp()` 来对比字符串；
- 不要忘记把程序添加到 `Makefile` 的 `UPROGS` 里。

这道题几乎可以参考 `user/ls.c`，而且不需要完全了解所有细节，猜测个大概，照猫画虎就能完成。

???参考答案
    ```c
    #include "kernel/types.h"
    #include "kernel/stat.h"
    #include "user/user.h"
    #include "kernel/fs.h"

    char*
    getFilename(char *path)
    {
        char *p;

        // Find first character after last slash.
        for(p=path+strlen(path); p >= path && *p != '/'; p--)
            ;

        p++;
        return p;
    }

    void
    find(char *filename, char *path)
    {
        char buf[512], *p;
        int fd;
        struct dirent de;
        struct stat st;

        if((fd = open(path, 0)) < 0){
            fprintf(2, "ls: cannot open %s\n", buf);
            return;
        }

        if(fstat(fd, &st) < 0){
            fprintf(2, "ls: cannot stat %s\n", buf);
            close(fd);
            return;
        }

        switch(st.type){
        case T_FILE:
            p = getFilename(path);
            if(strcmp(filename, p) == 0) {
                printf("%s\n", path);
            }
            break;

        case T_DIR:
            if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
                printf("ls: path too long\n");
                break;
            }
            strcpy(buf, path);
            p = buf+strlen(buf);
            *p++ = '/';
            while(read(fd, &de, sizeof(de)) == sizeof(de)){
                if(de.inum == 0 
                || strcmp(de.name, ".") == 0
                || strcmp(de.name, "..") == 0)
                    continue;
                memmove(p, de.name, strlen(de.name));
                p[strlen(de.name)] = 0;
                find(filename, buf);
            }
            break;
        }
        close(fd);
        return;
    }

    int
    main(int argc, char *argv[])
    {
        if(argc <= 2){
            fprintf(2, "Usage: find DIRPATH FILENAME\n");
            exit(1);
        }
        find(argv[2], argv[1]);
        exit(0);
    }
    ```

## xargs

!!!题目描述
    实现一个简单的UNIX的 `xargs` 程序：读取几行标准输出，运行一个命令，接受的参数分别是每一行
    的内容。

简单介绍一下 `xargs` 的功能：

```shell
$ echo hello too | xargs echo bye
bye hello too
$
```

这条指令是在 `echo bye` 后面追加 `hello too` 的内容，形成 `echo bye hello too` 命令。
这里只需要完成上面功能即可，也就是直接在后面的命令里追加新的参数。

UNIX 版本的 `xargs` 能够执行更复杂的功能，将左边的内容做更多解析，然后按要求赋值给后面的命令作
为参数, 例如：一次可以为命令提供多个参数。我们这里只需要实现一次赋值一个参数的功能即可。
如果想让其他UNIX上的`find`命令像我们这里需要实现的`find`一样，那么需要在那些`find`命令上添加
变量 `-n 1`。

```shell
$ echo "1\n2" | xargs -n 1 echo line
line 1
line 2
$
```

上面的代码前提是 `echo` 也是UNIX上的 `echo` 命令。 而 `xv6` 上的 `echo` 命令不会处理 `"`
和转义符号 `\` 也就是说，上面的命令要是在 `xv6` 上运行， `echo` 会得到 5 个字符 `1 \ n 2 \n`
(`\n` 是 `xv6` 上的 `echo` 自动添加的符号)。

提示：

- 使用 `fork` 和 `exec` 来执行命令，使用 `wait` 来使得父程序等待子程序结束运行；
- 每次读入一个字符，用`\n`来判断是否是另一个参数；
- `kernel/param.h` 声明了 `MAXARG` 宏定义，或许有用；
- 不要忘记将程序加入到 `UPROGS`。 

???参考答案

    ```c
    #include "kernel/types.h"
    #include "kernel/stat.h"
    #include "kernel/param.h"
    #include "user/user.h"

    int
    main(int argc, char *argv[])
    {
        char buf[MAXARG+1], tmp;
        int index = 0, i;
        
        if(argc < 2) {
            fprintf(2, "Usage: xargs COMMAND [arg1 arg2 ...]\n");
            exit(1);
        }

        for(i = 0; i < argc-1; i++) {
            argv[i] = argv[i+1];
        }
        argv[argc-1] = buf;

        while(read(0, &tmp, 1)) {
            if(tmp == '\n') {
                buf[index] = 0;
                if(fork() == 0) {
                    exec(argv[0], argv);
                    exit(0);
                } else {
                    wait(0);
                }
                index = 0;
            } else {
                buf[index++] = tmp;
            }
        }
        if(index > 0) {
            buf[index++] = 0;
            exec(argv[0], argv);
        }
        exit(0);
    }
    ```

