# Lab9: File System

本次实验将添加`large file`和`symbolic link`两个特性到`xv6`中。需要阅读`xv6-book`第八章的内容。实验代码获取：

```bash
git fetch
git checkout fs
make clean
```

## Large files

在这个实验中，需要增加xv6支持的文件最大尺寸。当前的xv6文件限制在268blocks，或者是 268BSIZE 比特（BSIZE 在 xv6 中为 1024)。限制是由 xv6 inode 包含 12 个直接block，以及一个间接block。这个间接block指向256个直接block，所以总共是 $12 + 256 = 268$ blocks。

命令 `bigfile` 是用来检测这个实现的，它会创建一个 65803 blocks。

我们需要实现一个双间接block的功能。我们最终可以实现 $256 \times 256 + 256 + 11$ blocks（原本的12个直接block需要分出一个给双间接block）。

### Preliminaries

程序`mkfs`创建了xv6文件系统所有需要的磁盘镜像，确定了文件系统一共有多少blocks。这个数值由`kernel/param.h:FSSIZE` 中确定。在`make`指令的输出中，有一条

```bash
nmeta 70 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 25) blocks 199930 total 200000
```

介绍了`mkfs/mkfs`给文件系统创建了70个基础块，用于描述文件系统，剩余199930是数据块，总用200000个块。

每次修改文件系统，需要`make clean`来清除`fs.img`。

### What to Look At

我们应该注意定义在`fs.h`中的on-disk的inode结构`struct dinode`。我们应该特别注意`dinode`的几个成员变量`NDIRECT`, `NINDIRECT`, `MAXFILE` 和 `addrs[]`。 `xv6-book`的图8.3展示了xv6的标准inode结构。

第二个应该注意的点是`fs.c`中的函数`bmap()`。确保我们理解了它在做什么。`bmap()`在读写文件时都有用。当写文件的时候，`bmap()`分配新的blocks来保存文件内容，同时如果需要的话，分配间接block来保存blocks的地址。

`bmap()`处理两种块编号。参数`bn`是逻辑块编号——一个关于文件开头的块编号。磁盘的块编号在`ip->addrs[]`中，同时也是`bread()`的参数。我们可以认为`bmap()`是将文件的逻辑块编号映射到磁盘的块编号。

### Your Job

!!!题目描述
    修改`bmap()`来支持双间接块，包括直接块和一个间接块。因此我们将只剩下11个直接块，和一个用于保存间接块地址的双间接块。我们不能修改on-disk的inode的个数。`ip-addrs[]`的前11个项应该是直接块；第12块应该是一个单间接块；第13块是一个双间接块。最终需要通过程序`bigfile`来创建65803个块大小的文件，并且通过测试`usertests`。

提示：

- 确保我们理解了`bmap()`函数。写下`ip->addrs[]`，单间接块和双间接块以及纯单间接块的指向图。确保理解了双间接块如何增加了最大文件的尺寸。
- 想一想如何通过逻辑块编号来确定双间接块的下标，以及它指向的单间接块。
- 如果修改了`NDIRECT`的定义，那就要同时修改`file.h`里的结构体`struct inode`中的`addrs[]`，确保`struct inode`和`struct dinode`的`addrs[]`有相同数量的元素。
- 如果修改`NDIRECT`的定义，那么应该要删除并且创建新的`fs.img`。
- 不要忘记在`bread()`后调用`brelse()`。
- 只在需要的时候分配间接块和双间接块。
- 确保`itrunc`释放文件的所有块，包括双间接块。

???参考答案
    1. 修改宏定义文件`kernel/fs.h`。

        ```c
        #define NDIRECT 11
        #define NINDIRECT (BSIZE / sizeof(uint))
        #define NDINDIRECT ((BSIZE / sizeof(uint))*(BSIZE / sizeof(uint)))
        #define MAXFILE (NDIRECT + NINDIRECT + NDINDIRECT)
        ```
    
    2. 修改`fs.h:struct dinode`和`file.h:struct inode`中的成员变量`addr`。

        ```c
        uint addrs[NDIRECT+2];   // Data block addresses
        ```

    3. 修改文件`kernel/fs.c`中的两个函数`bmap()`和`itrunc`。

        ```c
        static uint
        bmap(struct inode *ip, uint bn)
        {
            uint addr, *a;
            struct buf *bp, *bpp;

            if(bn < NDIRECT){
                if((addr = ip->addrs[bn]) == 0)
                    ip->addrs[bn] = addr = balloc(ip->dev);
                return addr;
            }
            bn -= NDIRECT;

            if(bn < NINDIRECT){
                // Load indirect block, allocating if necessary.
                if((addr = ip->addrs[NDIRECT]) == 0)
                    ip->addrs[NDIRECT] = addr = balloc(ip->dev);
                bp = bread(ip->dev, addr);
                a = (uint*)bp->data;
                if((addr = a[bn]) == 0){
                    a[bn] = addr = balloc(ip->dev);
                    log_write(bp);
                }
                brelse(bp);
                return addr;
            }

            bn -= NINDIRECT;

            if(bn < NDINDIRECT){
                // Load double-indirect block, allocating if necessary.
                if((addr = ip->addrs[NDIRECT+1]) == 0)
                    ip->addrs[NDIRECT+1] = addr = balloc(ip->dev);
                bp = bread(ip->dev, addr);
                a = (uint*)bp->data;
                if((addr = a[bn/NINDIRECT]) == 0){
                    a[bn/NINDIRECT] = addr = balloc(ip->dev);
                    log_write(bp);
                }
                bpp = bread(ip->dev, addr);
                a = (uint*)bpp->data;
                if((addr = a[bn%NINDIRECT]) == 0){
                    a[bn%NINDIRECT] = addr = balloc(ip->dev);
                    log_write(bpp);
                }
                brelse(bpp);
                brelse(bp);
                return addr;
            }
            panic("bmap: out of range");
        }
        ```
        ```c
        void
        itrunc(struct inode *ip)
        {
            int i, j, k;
            struct buf *bp, *bpp;
            uint *a, *b;

            for(i = 0; i < NDIRECT; i++){
                if(ip->addrs[i]){
                    bfree(ip->dev, ip->addrs[i]);
                    ip->addrs[i] = 0;
                }
            }

            if(ip->addrs[NDIRECT]){
                bp = bread(ip->dev, ip->addrs[NDIRECT]);
                a = (uint*)bp->data;
                for(j = 0; j < NINDIRECT; j++){
                    if(a[j])
                        bfree(ip->dev, a[j]);
                }
                brelse(bp);
                bfree(ip->dev, ip->addrs[NDIRECT]);
                ip->addrs[NDIRECT] = 0;
            }

            if(ip->addrs[NDIRECT+1]){
                bp = bread(ip->dev, ip->addrs[NDIRECT+1]);
                a = (uint*)bp->data;
                for(j = 0; j < NINDIRECT; j++){
                    if(a[j]){
                        bpp = bread(ip->dev, a[j]);
                        b = (uint*)bpp->data;
                        for(k = 0; k < NINDIRECT; k++){
                            if(b[k])
                                bfree(ip->dev, b[k]);
                        }
                        brelse(bpp);
                        bfree(ip->dev, a[j]);
                        a[j] = 0;
                    }
                }
                brelse(bp);
                bfree(ip->dev, ip->addrs[NDIRECT+1]);
                ip->addrs[NDIRECT+1] = 0;
            }

            ip->size = 0;
            iupdate(ip);
        }
        ```

## Symbolic Links

在这个实验中，增加symbolic links的功能。 Symbolic links（或者叫软连接）通过文件名执行一个被链接的文件。当symbolic link被打开，内核沿着链接打开被指向的文件。Symbolic links 类似于硬链接，但是硬链接仅限于指向同一个磁盘上的文件，而符号链接可以跨磁盘设备。虽然现在的xv6不支持多设备，实现这个软连接功能能够很好地理解pathname查找功能的工作流程。

!!!题目描述
    实现系统调用`symlink(char *target, char *path)`，创建一个新的符号链接指向由`*target`确定的文件。如果需要更多的信息，查看`page symlink`的man页。为了测试，需要在`Makefile`中增加`symlinktest`命令并且运行它。最后我们要通过`symlinktest`和`usertests`。

- 创建系统调用`symlink`，增加入口到`user/usys.pl`和`user/user.h`，在`kernel/sysfile.c`中实现`sys_symlink`；（这里没有讲清楚，还有很多地方要声明，在前面的实验中有遇到过，具体可以看参考答案。）
- 在`kernel/stat.h`中增加类型`T_SYMLINK`，来代表符号链接；
- 在`kernel/fcntl.h`中增加标记`O_NOFOLLOW`，它可以和`open`这个系统调用连用。传递到`open`中的参数需要用按位或来修改，不要覆盖其他的标记。至此，我们应该能够通过编译过程。

???参考答案
    1. `user/usys.pl`中添加声明：`entry("symlink");`；
    2. `user/user.h`中添加声明： `int symlink(const char*, const char*);`；
    3. `kernel/syscall.h`中添加声明：`#define SYS_symlink 22`；
    4. `kernel/syscall.c`中添加声明：`extern uint64 sys_symlink(void);`；
    5. 修改数组`kernel/syscall.c:syscalls`：`[SYS_symlink] sys_symlink,`；
    6. `kernel/sysfile.c`中添加函数定义：

        ```c
        uint64
        sys_symlink(void)
        {
            return 0;
        }
        ```

    7. 在`kernel/stat.h`中添加定义：`#define T_SYMLINK 4   // Symbolic Link`；
    8. 在`kernel/fcntl.h`中添加定义：`#define O_NOFOLLOW 0x800`；
    9. 最后修改`Makefile`，来引入用户程序`symlinktest`：`$U/_symlinktest\`。

- 实现`symlink(target, path)`。`target`不需要存在。我们需要一个地方来保存符号路径的目标路径，例如：在inode的数据块中。`symlink`应该返回0表示成功，返回-1表示失败。
- 修改`open`系统调用来处理指向符号链接的路径。如果文件不存在就返回-1表示失败。当要打开的文件中`O_NOFOLLOW`为1时，`open`应该打开链接而不是指向链接。
- 如果被链接的文件也是一个符号链接，那么需要递归的调用到非符号链接的文件。如果形成环，那么应该报错。可以通过设定链接深度的阈值，例如：10，来实现环的检测。
- 其他系统调用（例如link和unlink）不能指向符号链接；这些系统调用接受的符号链接指向它们自己。
- 不需要处理文件夹的情况。

???参考答案
    1. 实现`sys_symlink()`函数。根据题目要求，很多事情都不用考虑。
    
    ```c
    uint64
    sys_symlink(void)
    {
        char new[MAXPATH], target[MAXPATH];
        struct inode *ip_new;

        if(argstr(0, target, MAXPATH) < 0 || argstr(1, new, MAXPATH) < 0)
            return -1;

        begin_op();

        ip_new = create(new, T_SYMLINK, 0, 0);
        if(ip_new == 0){
            end_op();
            return -1;
        }

        // Copy target name to the symbolic inode.
        if(writei(ip_new, 0, (uint64)target, ip_new->size, MAXPATH) < 0){
            panic("symbolic panic");
        }
        iunlockput(ip_new);

        end_op();
        return 0;
    }
    ```

    2. 修改`sys_open()`函数。关键是要完成对递归的符号链接的判定。我实现了一个非常简陋的版本。

    ```c
    uint64
    sys_open(void)
    {
        //...
        begin_op();

        if(omode & O_CREATE){
            //...
        } else {
            if((ip = namei(path)) == 0){
                end_op();
                return -1;
            }

            ilock(ip);

            // Symbolic inode
            if(!(omode & O_NOFOLLOW)){
                for(int i = 0; i < 10; ++i){
                    if(ip->type == T_SYMLINK){
                        memset(path, 0, MAXPATH);
                        if(readi(ip, 0, (uint64)path, ip->size-MAXPATH, MAXPATH) < 0){
                            iunlockput(ip);
                            end_op();
                            return -1;
                        }
                        iunlockput(ip);
                        if((ip = namei(path)) == 0){
                            end_op();
                            return -1;
                        }
                        ilock(ip);
                    } else {
                        break;
                    }
                }
                if(ip->type == T_SYMLINK){
                    iunlockput(ip);
                    end_op();
                    return -1;
                }
            }
            // Directory
            if(ip->type == T_DIR && omode != O_RDONLY){
                iunlockput(ip);
                end_op();
                return -1;
            }
        }
        //...
    }
    ```