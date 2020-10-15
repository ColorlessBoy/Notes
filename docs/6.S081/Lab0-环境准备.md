# 环境准备

## 前言

> 翻译自[6.S081(Learning by doing)](https://pdos.csail.mit.edu/6.828/2020/overview.html)

这是一门MIT的操作系统实验课，包含：设计和实现操作系统，以及系统编程的基础。主要包含的内
容有：虚拟内存；文件系统；线程；进程切换；内核；终端；系统调用；进程间通信；软件和硬件交互。
通过一个小型的基于RISC-V指令集的操作系统 xv6 来解释这些内容。同时，将通过一系列独立的实验
来扩展 xv6，例如支持更复杂的虚拟内存技术和网络技术。

我们会好奇为什么要学习 xv6 呢？要学习一个重写的 Unix v6 而不是最新的最棒的Linux，
Windows，或者BSD Unix呢？因为 xv6 足够大，足以解释操作系统中的基本的设计与实现理念，
同时又足够的小（比现代的操作系统内核小得多）所以非常容易理解。xv6 有着与现代操作系统非常
类似的结构，所以我们学习 xv6，能够帮助我们理解其他的操作系统内核，例如：Linux。

我为什么要创建这个笔记系列呢？去年我也做过一些这个课的实验，最后也半途而废了。
这段时间我回过头来看，之前做实验直接看别人的笔记，导致我扎进了太多的细节里，这些细节完全
没有必要了解，而且这些细节非常容易使得初学者望而生畏。现在这门课从 6.828 改名叫 6.S081
了，也属于进入了新的阶段，我也应该配合着做一个新的笔记。

我希望这个笔记： 
尽量遵循讲义的要求，而且包含的内容越简单越好，使得知识点是能够被 take away。

## 所需要使用到的工具

> 参考：[Tools Used in 6.S081](https://pdos.csail.mit.edu/6.828/2020/tools.html)

我们需要一系列 RISC-V 版本的工具：QEMU 5.1， GDB 8.3， GCC 和 Binutils。
我是一点都不慌，后面的肯定有讲该怎么安装。我这里使用 Ubuntu 系统，所以我就摘记一下我的
安装过程。可以说新的课程所需要的工具安装起来比之前要方便多了。

首先我们要确定 debian 版本是 `bullseye` 或者 `sid`，使用如下命令检查：

```
cat /etc/debian_version
```

接着我们执行：

```
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu 
```

根据讲义说的，新版的 `qemu-system-misc` 会和实验的内核有冲突，所以需要回退版本。我这里
就不测试是不是又bug了，直接照着讲义来安装旧的版本：

```
sudo apt-get remove qemu-system-misc
sudo apt-get install qemu-system-misc=1:4.2-3ubuntu6
```

非常不幸，我安装旧版本的时候发现浙大源里没有这个包，我先试试现在还有没有这个bug，然后看
看其他源里有没有这个旧版本的包。

