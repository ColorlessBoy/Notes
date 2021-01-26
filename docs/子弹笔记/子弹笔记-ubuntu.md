# 子弹笔记-Ubuntu

## 使用官方网站的脚本来安装Nvidia、Cuda和Cudnn（针对Ubuntu 12.04安装CUDA11.2）

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

## 使用Ubuntu的软件源来安装Nvidia驱动

这种方法安装的驱动自带`CUDA`和`CUDNN`，使用`Pytorch`自带的安装教程直接支持了`CUDA`和`CUDNN`。我不清楚具体的差别。

1. 添加源`sudo add-apt-repository ppa:graphics-drivers/ppa`;
2. 更新软件库`sudo apt update`；
3. 查找可选的驱动`sudo ubuntu-drivers devices`;
4. 安装驱动`sudo apt install nvidia-xxx`。