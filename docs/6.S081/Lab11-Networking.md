# Lab11: Networking

在这个实验中我们为xv6实现一个网卡(network interface card, NIC)驱动。
获取代码：

```bash
make clean
git fetch
git checkout net
```

## 背景

可以参考`xv6-book`的第五章内容：中断和设备驱动。

我们需要使用`E1000`网卡驱动来处理网络连接。对于xv6，`E1000`就是一个真实的网卡设备，可以和真实的局域网（Ethernet local area network）。实际上，我们使用的`E1000`设备是由qemu提供的虚拟设备，连接的局域网也是由qemu虚拟的。在这个虚拟的局域网上，xv6的局域网IP地址是`10.0.2.15`。运行qemu的主机在这个虚拟局域网上的IP地址是`10.0.2.2`。当xv6使用E1000发送一个网络包到`10.0.2.2`时，qemu会将这个包发送到运行这个qemu程序的主机上的合适的应用上。

我们将使用QEMU的`user-mode network stack`。QEMU关于用户模式的栈的说明在[这个网站](https://www.qemu.org/docs/master/system/net.html#using-the-user-mode-network-stack)上。实验代码已经修改Makefile支持QEMU的用户模式网络栈和E1000网卡。

Makefile配置的QEMU将所有输入输出的网络包记录在文件`packets.pcap`中。要回顾这些记录来确认xv6成功发送并获得期望获得的正确的网络包。可以使用如下命令来解析网络包：

```bash
tcpdump -XXnr packets.pcap
```

实验代码添加了一些重要的文件，值得我们关注。文件`kernel/e1000.c`包含`E1000`初始化的代码以及清空发送和获得的网络包。它使用了一些定义在`kernel/e1000_dev.h`中的宏定义和声明。具体需要的寄存器、宏等信息在E1000的[软件开发文档](https://pdos.csail.mit.edu/6.828/2020/readings/8254x_GBe_SDM.pdf)。`kernel/net.c`和`kernel/net.h`包含一些简单的网络栈，实现了IP、UDP和ARP协议。这些文件包含一个灵活的数据结构`mbuf`来保存这些包。最后`kernel/pci.c`包含的代码用于在xv6启动的时候查找挂载在PCI主线上的E1000网卡。

## 任务

!!!问题描述
    我们要完善`e1000_transmit()`和`e1000_recv()`，以及`kernel/e1000.c`，使得驱动能够发送和接收网络包。最后通过`make grade`测试。

!!!重要提示
    我们可以参考E100的[软件开发文档](https://pdos.csail.mit.edu/6.828/2020/readings/8254x_GBe_SDM.pdf)。但是文档非常长，这里可以参考如下几个章节：

    - `Section 2`是最重要的，描述了整个设备的大体情况。
    - `Section 3.2` 描述了网络包的接收流程。
    - `Section 3.3`和`Section 3.4` 描述了网络包的发送流程。
    - `Section 13` 列出了E1000提供的寄存器。
    - `Section 14` 能够帮助我们理解实验代码提供的初始化代码。

浏览E100的[软件开发文档](https://pdos.csail.mit.edu/6.828/2020/readings/8254x_GBe_SDM.pdf)的再次说明。QEMU模拟的是`82540EM`。浏览一下`Section 2`的内容来大体了解这些内容。在实现驱动的时候，我们需要熟悉`Section 3`，`Section 14`和`Section 4.1`的内容。可能还需要`Section 13`作为参考。不需要完全搞清楚文档的细节，只需要一个了解一个大概。E1000有非常多的特点，我们只会用到一些非常基础的功能。

在`e1000.c`中实现的`e1000_init()`实直接从RAM中读写信息。这种技术叫做DMA(direct memory access)，表示网卡E1000直接从RAM中读写信息。

由于数据包的突发到达可能比驱动程序处理它们的速度快，e1000_init()为E1000提供了多个缓冲区，E1000可以将数据包写入其中。E1000要求这些缓冲区在RAM中由一组“描述符”来描述;每个描述符在RAM中包含一个地址，E1000可以在其中写一个接收到的包。struct rx_desc描述描述符格式。描述符数组称为接收环，或接收队列。从某种意义上说，它是一个圆环，当卡或驱动程序到达数组的末尾时，它就会回到数组的开头。e1000_init()使用mbufalloc()为E1000分配用于DMA的mbuf包缓冲区。还有一个传输环，驱动程序在其中放置E1000要发送的数据包。e1000_init()配置两个环的大小为RX_RING_SIZE和TX_RING_SIZE。

当net.c中的网络堆栈需要发送数据包时，它使用一个包含要发送的数据包的mbuf调用e1000_transmit()。您的传输代码必须在TX(传输)环的描述符中放置一个指向数据包数据的指针。tx_desc描述描述符格式。您需要确保最终释放每个mbuf，但只有在E1000完成了数据包的传输之后(E1000在描述符中设置E1000_TXD_STAT_DD位来表示这一点)。

当E1000从以太网接收到每个包时，它首先将包dma到下一个RX(接收)环描述符指向的mbuf，然后生成一个中断。您的e1000_recv()代码必须扫描RX环，并通过调用net_rx()将每个新包的mbuf发送到网络堆栈(net.c)。然后，您需要分配一个新的mbuf，并将其放入描述符中，以便当E1000再次到达RX环上的那个点时，它找到一个新的缓冲区，以便DMA新数据包。

除了读写RAM中的描述符环之外，你的驱动程序还需要通过它的内存映射控制寄存器与E1000交互，来检测接收到的数据包何时可用，并通知E1000驱动程序已经用数据包填充了一些TX描述符来发送。全局变量regs保存着一个指向E1000第一个控制寄存器的指针;驱动程序可以通过将regs索引为一个数组来获取其他寄存器。您特别需要使用索引E1000_RDT和E1000_TDT。

要测试驱动程序，请在一个窗口中运行make server，在另一个窗口中运行make qemu，然后在xv6中运行nettests。nettests中的第一个测试尝试向主机操作系统发送UDP数据包，该数据包指向使服务器运行的程序。如果您还没有完成实验，E1000驱动程序实际上不会发送数据包，也不会发生什么事情。

在你完成实验之后，E1000驱动程序将发送数据包，qemu将把它发送到你的主机，make server将看到它，它将发送一个响应数据包，然后E1000驱动程序和nettests将看到响应数据包。但是，在主机发送应答之前，它向xv6发送一个“ARP”请求包，以找出它的48位以太网地址，并期望xv6响应一个ARP应答。一旦你完成了E1000驱动程序的工作，kernel/net.c就会处理这个问题。如果一切顺利，nettests将打印测试ping: OK, make server将打印一条来自xv6!的消息。

命令`tcpdump -XXnr packets.pcap`将会输出如下内容：

```bash
reading from file packets.pcap, link-type EN10MB (Ethernet)
15:27:40.861988 IP 10.0.2.15.2000 > 10.0.2.2.25603: UDP, length 19
        0x0000:  ffff ffff ffff 5254 0012 3456 0800 4500  ......RT..4V..E.
        0x0010:  002f 0000 0000 6411 3eae 0a00 020f 0a00  ./....d.>.......
        0x0020:  0202 07d0 6403 001b 0000 6120 6d65 7373  ....d.....a.mess
        0x0030:  6167 6520 6672 6f6d 2078 7636 21         age.from.xv6!
15:27:40.862370 ARP, Request who-has 10.0.2.15 tell 10.0.2.2, length 28
        0x0000:  ffff ffff ffff 5255 0a00 0202 0806 0001  ......RU........
        0x0010:  0800 0604 0001 5255 0a00 0202 0a00 0202  ......RU........
        0x0020:  0000 0000 0000 0a00 020f                 ..........
15:27:40.862844 ARP, Reply 10.0.2.15 is-at 52:54:00:12:34:56, length 28
        0x0000:  ffff ffff ffff 5254 0012 3456 0806 0001  ......RT..4V....
        0x0010:  0800 0604 0002 5254 0012 3456 0a00 020f  ......RT..4V....
        0x0020:  5255 0a00 0202 0a00 0202                 RU........
15:27:40.863036 IP 10.0.2.2.25603 > 10.0.2.15.2000: UDP, length 17
        0x0000:  5254 0012 3456 5255 0a00 0202 0800 4500  RT..4VRU......E.
        0x0010:  002d 0000 0000 4011 62b0 0a00 0202 0a00  .-....@.b.......
        0x0020:  020f 6403 07d0 0019 3406 7468 6973 2069  ..d.....4.this.i
        0x0030:  7320 7468 6520 686f 7374 21              s.the.host!
```

我们的输出应该还会包含`ARP,Request`,`ARP,Reply`，`UDP`, `a.message.from, xv6`和`this.is.the.host`。

`nettests`执行一些其他测试，最终通过(真实的)Internet向谷歌的名称服务器之一发送DNS请求。你应该确保你的代码通过了所有这些测试，之后你应该看到以下输出:

```bash
$ nettests
nettests running on port 25603
testing ping: OK
testing single-process pings: OK
testing multi-process pings: OK
testing DNS
DNS arecord for pdos.csail.mit.edu. is 128.52.129.126
DNS OK
all tests passed.
```

确保`make grade`通过所有的测试。

## 提示

我们先来实现`e1000_transmit()`和`e1000_recv()`两个函数，运行`make server`以及在xv6中运行`nettests`。我们应该能够看到`nettests`调用的`e1000_transmit()`输出的文字。

实现`e1000_transmit`的提示：

- 首先通过读取E1000_TDT控制寄存器，向E1000请求它期待下一个包的TX环索引。
- 然后检查环是否溢出。如果E1000_TDT索引的描述符中没有设置E1000_TXD_STAT_DD，则E1000还没有完成之前相应的传输请求，因此返回一个错误。
- 否则，使用mbuffree()释放从描述符(如果有的话)传输的最后一个mbuf。
- 然后填写描述符。m->head表示数据包在内存中的内容，m->len表示数据包的长度。设置必要的cmd标志(参见E1000手册中的3.3节)，并保存一个指向mbuf的指针，以便稍后释放。
- 最后，通过向E1000_TDT的TX_RING_SIZE模数中添加1来更新环的位置。
- 如果e1000_transmit()成功地将mbuf添加到环中，则返回0。如果失败(例如，没有可用的描述符来传输mbuf)，返回-1，以便调用者知道释放mbuf。

实现`e1000_recv`的提示：

- 首先，通过获取E1000_RDT控制寄存器并添加一个模RX_RING_SIZE，向E1000请求下一个等待接收包(如果有的话)所在的环索引。 
- 然后通过检查描述符的状态部分中的E1000_RXD_STAT_DD位来检查是否有新的包可用。如果没有,停止。 
- 否则，将mbuf的m->len更新为描述符中报告的长度。使用net_rx()将mbuf交付到网络堆栈。
- 然后使用mbufalloc()分配一个新的mbuf来替换net_rx()所分配的mbuf。将它的数据指针(m->头)写入描述符。将描述符的状态位清除为零。
- 最后，将E1000_RDT寄存器更新为所处理的最后一个环描述符的索引。
- e1000_init()使用mbufs初始化RX环，您将需要了解它是如何实现这一点的，可能还需要借用代码。
- 在某个点上，已经到达的数据包的总数将超过环的大小(16);确保你的代码可以处理这个问题。

我们需要使用锁来应对以下情况:xv6可能从多个进程使用E1000，或者可能在中断到达时在内核线程中使用E1000。

如果想要使用我的`docker`环境来测试，需要修改`Makefile`，相比于其他实验，增加了`make docker-server`命令。需要先执行`make docker-qemu`然后执行`make docker-server`，再执行`nettests`来测试代码。具体`Makefile`添加的内容如下：

```make
## ==================Docker Commands Start======================================
docker: 
	-docker rm -f xv6-labs-2020
	docker run  -it --name "xv6-labs-2020"\
							-w /xv6-labs-2020 -v "$(shell pwd):/xv6-labs-2020" \
							penglingwei/xv6-labs-2020:latest \
							/bin/bash

docker-grade: 
	-docker rm -f xv6-labs-2020
	docker run  -it --name "xv6-labs-2020"\
                -w /xv6-labs-2020 -v "$(shell pwd):/xv6-labs-2020" \
                penglingwei/xv6-labs-2020:latest \
                /bin/bash -c "make grade" 

docker-qemu: 
	-docker rm -f xv6-labs-2020
	docker run  -it --name "xv6-labs-2020"\
                -w /xv6-labs-2020 -v "$(shell pwd):/xv6-labs-2020" \
                penglingwei/xv6-labs-2020:latest \
                /bin/bash -c "make qemu" 

docker-qemu-gdb: 
	-docker rm -f xv6-labs-2020
	docker run  -it --name "xv6-labs-2020"\
                -w /xv6-labs-2020 -v "$(shell pwd):/xv6-labs-2020" \
                penglingwei/xv6-labs-2020:latest \
                /bin/bash -c "make qemu-gdb" 

docker-gdb: .gdbinit
	docker exec -it xv6-labs-2020 /bin/bash -c "gdb-multiarch --command .gdbinit"

docker-server: 
	docker exec -it xv6-labs-2020 /bin/bash -c "make server"

docker-rm:
	docker rm -f xv6-labs-2020
## ==================Docker Commands End========================================
```

???参考答案
    本次实验，我基本上参考了别人的答案。本文列举了非常多的参考文献，没有必要去查看。更重要的是对源码的阅读。我对网卡驱动还是不太了解，陷入参考资料很久也没有进展，索性不好意思地直接参考了[别人的答案](https://www.cnblogs.com/YuanZiming/p/14271553.html)。

    1. 实现函数`kernel/e10000.c：e1000_transmit()`。这里基本上就按照提示加上一些猜测来实现。

        ```c
            int
            e1000_transmit(struct mbuf *m)
            {
                //
                // Your code here.
                //
                // the mbuf contains an ethernet frame; program it into
                // the TX descriptor ring so that the e1000 sends it. Stash
                // a pointer so that it can be freed after sending.
                //
                int idx;
                acquire(&e1000_lock);
                idx = regs[E1000_TDT];
                if((tx_ring[idx].status & E1000_TXD_STAT_DD) == 0){
                release(&e1000_lock);
                return -1;
                }
                if(tx_mbufs[idx] != 0)
                mbuffree(tx_mbufs[idx]);
                tx_mbufs[idx] = m;
                tx_ring[idx].addr = (uint64)m->head;
                tx_ring[idx].length = (uint16)m->len;
                tx_ring[idx].cmd = E1000_TXD_CMD_EOP | E1000_TXD_CMD_RS;
                regs[E1000_TDT] = (idx + 1) % TX_RING_SIZE;
                release(&e1000_lock);
                return 0;
            }
        ```

    2. 实现函数`kernel/e1000.c:e1000_recv()`。这里根据提示的话，我是一下子想不到不用锁的版本。

        ```c
        static void
        e1000_recv(void)
        {
          //
          // Your code here.
          //
          // Check for packets that have arrived from the e1000
          // Create and deliver an mbuf for each packet (using net_rx()).
          //
          int idx;
          while(1) {
              idx = (regs[E1000_RDT] + 1) % RX_RING_SIZE;
              if((rx_ring[idx].status & E1000_RXD_STAT_DD) == 0){
                  break;
              }
              rx_mbufs[idx]->len = rx_ring[idx].length;
              net_rx(rx_mbufs[idx]);
              rx_mbufs[idx] = mbufalloc(0);
              rx_ring[idx].addr = (uint64)rx_mbufs[idx]->head;
              rx_ring[idx].status = 0;
              regs[E1000_RDT] = idx;
          }
        }
        ```