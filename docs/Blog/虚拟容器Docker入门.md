# 虚拟化容器Docker入门

虚拟化容器Docker是一个非常时尚的技术了，我现在想了解一下。最近在学习6.S081的课程，要安装
一系列的编译工具，一方面我不希望直接安装到电脑的操作系统里，另一方面我希望能够快速部署到
其他机器上，方便其他人。所以我想了解一下Docker，看它能不能帮我完成这件事情。同时在平时
科研的时候，代码环境的部署，以及代码本身也需要保存一些重要的数据，不知道Docker能不能完成
这件事情。

## Docker的安装

我是在Ubuntu上安装Docker，所以这里的安装方法直接引用
[官网链接](https://docs.docker.com/engine/install/ubuntu/)吧，写得非常好。

## Docker的入门教程——Getting Started

Docker官网的入门教程的使用也非常有意思，它也是一个Docker镜像，从官方自动下载下来一个
教程镜像并运行，其实就是启动一个静态网页。命令行输入

```bash
$ docker run -dp 80:80 docker/getting-started
```

然后在浏览器打开`http://localhost`。

- `-d`:容器运行在`detached`模式，后台运行；
- `-p Host:Container`:主机的80端口映射到容器的80端口；

## 命令摘录

- `docker run`:启动容器，上面已经提到；
- `docker ps`:查看正在运行的容器；
- `docker stop <container-ID>`:关闭容器；
- `docker rm <container-ID>`:删除容器，容器不能正在运行；
- `docker rm -f <container-ID>`:强制删除容器，容器可以正在运行；
- `docker image ls`:查看安装的镜像；
- `docker tag <container-NAME> <tag>`:给容器添加标签；
- `docker exec <container-ID> <command>`:在容器中运行程序；

## 上传镜像

- shell登陆账号: `docker login -u YOUR-USER-NAME`;
- 登陆网站`hub.docker.com`;
- 创建仓库；
- 修改本地容器名称：`docker tag <container-NAME> YOUR-USER-NAME/<container-name>`;
- 本地推送：`docker push YOUR-USER-NAME/<container-NAME>`;

## 使用Play-with-docker

- 登陆网站`labs.play-with-docker.com`;

我无法登录，也就没办法尝试了。

## 保留容器数据

Docker使用`Volumes`来保存数据，是连接主机文件系统和容器文件系统的工具。容器分为两类，
这里先介绍第一类`named volumes`。

- 创建`volumes`：`docker volume create <volume-NAME>`；
- 挂载`volumes`运行容器：`docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started`；(这里使用了官方例子)
- Docker把`volumes`放在机器的哪里了呢？使用命令：`docker volume inspect <volumes-NAME>`
来查看详情；

上面的`volumes`不需要关心数据真的存在哪里，docker会帮我们管理。但是我们有时候会想要确定
使用哪里的数据，所以我们需要另一类`bind volumes`。

这里摘录一下官方教程，如何以开发模式进入容器。
这里会有三个步骤：挂载源代码，安装依赖，开启`nodemon`来观察文件系统是否修改。

1. 确定没有运行`getting-started`；（源码包含在docker/getting-started容器里，启动这个
容器到网页`Our Application`就能找到链接。）
2. 运行如下指令：
  
    ```bash
    docker run -dp 3000:3000 \
        -w /app -v "$(pwd):/app" \
        node:12-alpine \
        sh -c "yarn install && yarn run dev"
    ```

    - `-dp 3000:3000` 后台运行并且映射端口号`3000`;
    - `-w /app` 设置容器中运行的目录；
    - `-v` 将主机当前目录挂载到容器的`/app`目录下；
    - `node:12-alpine` 所使用的容器；
    - `sh xxx` 启动镜像后运行的命令。

3. 接着可以使用`docker logs -f <container-ID>`来查看镜像运行情况。有时候网络不好，启动
会比较慢，当启动成功，应该能看到一行`Listening on port 3000`。
4. 修改源代码，镜像会立刻作出反应。
5. 开发完成，可以创建一个新的镜像`docker build -t getting-started ..`。

## 多容器合作

Docker的容器使用网络来通讯。

1. 创建网络
   
    ```bash
    docker network create todo-app  
    ```

2. 我们想使用MySQL来作为容器，那么需要运行：
   
    ```bash
    docker run -d \
        --network todo-app --network-alias mysql \
        -v todo-mysql-data:/var/lib/mysql \
        -e MYSQL_ROOT_PASSWORD=secret \
        -e MYSQL_DATABASE=todos \
        mysql:5.7
    ```

    这里有一大堆不认识的东西，不过不用慌，我们不必都搞清楚。首先一些关于MySQL的环境变量
    我们不需要知道。`--network-alias` 后面也会讲。名字为`todo-mysql-data`的`volumn`
    不同我们创建，docker会自动帮我们创建。
3. 确定MySQL已经运行，我们使用`docker exec -it <mysql-container-id> mysql -p`来查看。
   我们设置了数据库的密码为`secret`，当需要输入密码时，输入这个就行了。按下面的命令查看
   创建的数据库表格。
   
    ```bash
    mysql> SHOW DATABASES; 
    ```

然后，我们需要连接这个容器了。但是每个容器有各自的IP地址，我们该如何连接呢？这里可以呼叫
`nicolaka/netshoot`容器来帮忙。

1. 运行一个`nicolaka/netshoot`容器：

    ```bash
    docker run -it --network todo-app nicolaka/netshoot
    ```

2. 在容器中，使用`dig`命令，一个非常有用的DNS工具：`dig mysql`。会看到返回信息里有一个
   `mysql.  600 IN A <ip address>`，这个就是相关ip地址了。
   
最后我们如何在应用的容器里使用MySQL呢。
首先了解MySQL的几个属性参数：

- `MYSQL_HOST`: MySQL服务器的地址；
- `MYSQL_USER`: 发起连接请求的用户名；
- `MYSQL_PASSWARD`: 密码；
- `MYSQL_DB`: 使用的数据库名字。

1. 使用下面指令来启动应用容器：

    ```bash
    docker run -dp 3000:3000 \
        -w /app -v "$(pwd):/app" \
        --network todo-app \
        -e MYSQL_HOST=mysql \
        -e MYSQL_USER=root \
        -e MYSQL_PASSWARD=secret \
        -e MYSQL_DB=todos \
        node:12-alpine \
        sh -c "yarn install && yarn run dev"
    ```

2. 查看容器运行情况:`docker logs <container-ID>`。
3. 打开app写入一些要完成的事项。
4. 验证mysql容器被写入了相应内容：
   
    ```bash
    docker exec -it <mysql-container-id> mysql -p todos
    ```

    然后再mysql的shell中使用命令

    ```bash
    mysql> select * from todo_items;
    ```

    应该能看到刚刚再应用容器中添加的内容。

## 使用Docker Compose

Docker使用一个文件来描述项目需要用到的技术栈，这样很容易再其他机器上配置相同的内容。

首先要安装打包器:`docker-compose`：

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

接下来配置文件怎么写就比较繁琐了，我不摘录了，可以查看[官方文档](https://docs.docker.com/compose/compose-file/)。

1. 在项目根目录下面创建一个`docker-compose.yml`文件。
2. 使用命令`docker-compose up -d`运行。
3. 使用`docker-compose logs -f`查看运行情况。
4. 使用`docker-compose down`关闭容器组。