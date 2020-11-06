# 子弹笔记-Docker

## 创建镜像

```bash
docker build -t xv6-labs-2020 .
```

## 解决tzdata卡死的问题

在`Dockfile`中设置先运行：

```bash
RUN export DEBIAN_FRONTEND=noninteractive \
    && apt-get update \
    && apt-get install -y tzdata \
    && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && dpkg-reconfigure --frontend noninteractive tzdata
```

## 运行镜像

```bash
docker run  -it --name "xv6-labs-2020"\
            -w /xv6-labs-2020 -v "$(shell pwd):/xv6-labs-2020" \
            penglingwei/xv6-labs-2020:latest \
            /bin/bash -c "make qemu-gdb" 
```

## 在已经运行的镜像中创建新的终端执行新的命令

```bash
docker exec -it xv6-labs-2020 /bin/bash -c "gdb-multiarch --command .gdbinit"
```