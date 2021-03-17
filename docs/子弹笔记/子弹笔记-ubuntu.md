# 子弹笔记-Ubuntu

## 使用官方网站的脚本来安装Nvidia、Cuda和Cudnn（针对Ubuntu 18.04安装CUDA11.2）

1. 禁用`nouveau`驱动：在文件`/etc/modprobe.d/blacklist.conf`中添加
   
    ```txt
    blacklist nouveau
    options nouveau modeset=0
    ```

    然后执行命令

    ```bash
    sudo update-initramfs -u
    ```

    重启后执行如下操作，如果没有输出，则说明成功。

    ```bash
    lsmod | grep nouveau
    ```

2. 卸载旧的驱动和`CUDA`，具体路径各个机器不同。

    ```bash
    sudo apt-get remove --purge nvidia*
    sudo /usr/local/cuda-10.0/bin/uninstall_cuda_10.0.pl
    sudo rm -rf /usr/local/cuda-10.0/
    sudo apt-get autoremove
    ```

3. 安装驱动和CUDA。[官网](https://developer.nvidia.com/cuda-downloads)下载`cuda`本地安装包。（网络要好，电脑内存要足够大，但是不需要提前安装驱动。）

    ```bash
    wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda_11.2.0_460.27.04_linux.run
    sudo sh cuda_11.2.0_460.27.04_linux.run
    ```

    安装的时候，内容全选。安装完成后需要添加CUDA地址到路径中，具体修改`.bashrc`文件：
    另外，在VSCODE中的Terminal中安装会因为BUG而无法安装，不要在VSCODE中安装。

    ```txt
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11.2/lib64
    export PATH=$PATH:/usr/local/cuda-11.2/bin
    ```

4. 测试`CUDA`是否安装成功。正常情况下输出`Result = PASS`。

    ```bash
    cd /usr/local/cuda-11.2/samples/1_Utilities/deviceQuery
    sudo make
    ./deviceQuery
    ```

5. 安装`CUDNN`。[官网](https://developer.nvidia.com/rdp/cudnn-download)下载三个`deb`包。(需要登陆)

    ```bash
    cuDNN Runtime Library for Ubuntu20.04 x86_64 (Deb)
    cuDNN Developer Library for Ubuntu20.04 x86_64 (Deb)
    cuDNN Code Samples and User Guide for Ubuntu20.04 x86_64 (Deb)
    ```

    使用`sudo dpkg -i libcudnn8-**`来安装这三个包。

6. 测试`CUDNN`。结果应该输出`Test passed!`。

   
    ```bash
    cd /usr/src/cudnn_samples_v8/mnistCUDNN
    sudo make clean && sudo make
    ./mnistCUDNN
    ```

    编译有可能报错，缺少某些文件，可以执行如下命令安装

    ```bash
    sudo apt-get install libfreeimage3 libfreeimage-dev
    ```

## 使用Ubuntu的软件源来安装Nvidia驱动

这种方法安装的驱动自带`CUDA`和`CUDNN`，使用`Pytorch`自带的安装教程直接支持了`CUDA`和`CUDNN`。我不清楚具体的差别。

1. 添加源`sudo add-apt-repository ppa:graphics-drivers/ppa`;
2. 更新软件库`sudo apt update`；
3. 查找可选的驱动`sudo ubuntu-drivers devices`;
4. 安装驱动`sudo apt install nvidia-xxx`。

## 安装并开启SSH

1. 安装`sudo apt-get install openssh-server`；
2. 开启自动启动`sudo systemctl enable ssh`；
3. 启动`sudo systemctl start ssh`。

## youtube-dl

1. 安装：`pip install youtube-dl`。
2. 使用范例：

    ```bash
    youtube-dl -R "infinite" --write-auto-sub --sub-lang en --embed-subs -o <FILENAME> -f bestvideo+bestaudio --merge-output-format mp4 <URL>
    ```

其中`-R`表示网络尝试次数；`-*sub*`表示字幕相关设定；`-o`表示输出文件名字；`-f`下载的文件音视频分辨率。

以YouTube为例，不是所有的视频流都有音频，例如1080p以上的视频就没有音频，所以`bestvideo+bestaudio`是最方便的选择。音视频文件格式不相同，所以使用`--merge-output`来合并两个文件。

## ffmpeg

我最近下载了冰血暴的第四季，发现只有英文字幕。所以我又在人人视频上下载了一份对应的字幕。
后来我希望将字幕文件添加到视频文件的中。这里我使用了一个批量添加的命令，学到了不少东西，所以记录一下。

```sh
ls | grep mkv | sed "s/.mkv//g" \
   | xargs -i ffmpeg -i {}.mkv -i {}.ChsEngA.ass \
    -map 0:0 -map 0:1 -map 0:s -map 1 \
    -c:a copy -c:v copy -c:s:0 copy -c:s:1 copy \
    -metadata:s:s:1 title="简体中文"  \
    ./output/{}.mkv
```

前面三个命令`ls`、`grep`和`sed`用于获取文件名字，然后传给命令`ffmpeg`。
`ffmpeg`命令接受两个文件（`-i`），然后确定四个流（`-map`），接着四个流都使用复制的操作（`-c`），然后给新添加的字幕流添加抬头名字（`-metadata`），最后指定输出位置。

## 字幕转码

下载下来的字幕编码为`ANSI`，在Linux中打开乱码，需要转成`UTF-8`的编码。可以使用如下命令。

```bash
iconv -f gbk -t utf8 <FILENAME> -o <OUTPUT-FILENAME> 
```

因为`ANSI`指的是和系统字体保持一致。而我下载的是中文字幕，所以实际上的编码应该是`gbk`。

## CMake版本太低

在编译`Katago`的时候遇到了CMake版本太低导致编译不成功的问题，这里附一个安装符合版本的CMake的指令。

```bash
wget https://github.com/Kitware/CMake/releases/download/v3.16.2/cmake-3.16.2-Linux-x86_64.tar.gz
tar zxvf cmake-3.16.2-Linux-x86_64.tar.gz
sudo mv cmake-3.16.2-Linux-x86_64 /opt/cmake-3.16.2
sudo ln -sf /opt/cmake-3.16.2/bin/* /usr/bin/
sudo reboot
cmake --version
```

## 创建用户并加入sudo组

```bash
adduser <USERNAME>
adduser <USERNAME> <GROUPNAME,such as: sudo>
```

## 批量结束进程

```bash
ps -aux|grep python|grep -v grep|cut -c 9-15|xargs kill -15
```