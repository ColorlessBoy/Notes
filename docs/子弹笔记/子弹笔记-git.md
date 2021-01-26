# 子弹笔记-Git

## `git clone --recursive`部分失败

当`git clone --recursive`部分失败后该怎么办呢？
可以进入项目文件夹后，使用如下命令来检查并下载子模块：

```bash
git submodule update --init --recursive --progress
```