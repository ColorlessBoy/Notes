# Lab6: Copy-on-Write Fork for xv6

虚拟内存提供了一层间接的级别：内核感知到内存指向的PTEs被标记为非法或者只读时，产生一个页错误， 

Virtual memory provides a level of indirection: the kernel can intercept memory references by marking PTEs invalid or read-only, leading to page faults, and can change what addresses mean by modifying PTEs. There is a saying in computer systems that any systems problem can be solved with a level of indirection. The lazy allocation lab provided one example. This lab explores another example: copy-on write fork.

To start the lab, switch to the cow branch:

$ git fetch
$ git checkout cow
$ make clean