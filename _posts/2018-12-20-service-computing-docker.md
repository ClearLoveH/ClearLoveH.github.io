---
layout:     post
title:      "Docker 开发初体验"
subtitle:   "极简博客使用Docker部署"
date:       2018-12-20 11:35
author:     "Heng"
header-img: "img/比尔吉沃特.jpg"
catalog: true
tags:
    - 服务计算
---
>上周我们小组一起完成了极简博客的开发，搭建好了简单的 web 服务与客户端，这周我们便通过使用 Docker 将我们的应用来实现容器化。

---
### 关于Docker
- Docker 是一个开源的应用容器引擎，基于 Go 语言并遵从Apache2.0协议开源。
- Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
- 容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app），更重要的是容器性能开销极低。
- Docker 开发技能适合`运维工程师`及`后端开发人员`来掌握。

### Docker的应用场景
- Web 应用的自动化打包和发布。
- 自动化测试和持续集成、发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。
- 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS(Platform-as-a-Service：平台即服务)环境

### Docker的优势所在
- 更快速的交付和部署
    - Docker在整个开发周期都可以完美的辅助你实现快速交付。Docker允许开发者在装有应用和服务本地容器做开发。可以直接集成到可持续开发流程中。开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。 Docker 可以快速创建容器，快速迭代应用程序，并让整个过程全程可见，使团队中的其他成员更容易理解应用程序是如何创建和工作的。 Docker 容器很轻很快！容器的启动时间是秒级的，大量地节约开发、测试、部署的时间。
- 高效的部署和扩容
    - Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。Docker的兼容性和轻量特性可以很轻松的实现负载的动态管理。你可以快速扩容或方便的下线的你的应用和服务，这种速度趋近实时。

- 更高的资源利用率
    - Docker 对系统资源的利用率很高，一台主机上可以同时运行数千个 Docker 容器。容器除了运行其中应用外，基本不消耗额外的系统资源，使得应用的性能很高，同时系统的开销尽量小。传统虚拟机方式运行 10 个不同的应用就要起 10 个虚拟机，而Docker 只需要启动 10 个隔离的应用即可。

- 更简单的管理
    - 使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。

---
### Docker的架构
- 我们小组所做的极简博客采用了Client-Service架构，Docker同样使用的是C/S架构，Client 通过接口与Server进程通信实现容器的构建，运行和发布。client和server可以运行在同一台集群，也可以通过跨主机实现远程通信，所以我们的应用可以使用Docker来进行部署。
    - Linux上Docker的架构：

        ![](/img/in-post/post-fuwujisuan/docker/1.png)

    - Windows上Docker架构:

        ![](/img/in-post/post-fuwujisuan/docker/2.png)

### Docker vs VM
- 同样是虚拟化技术，Docker又与我们常用的Virtual Machine有有什么差别呢？

    ![](/img/in-post/post-fuwujisuan/docker/3.png)

- VM是一个运行在宿主机之上的完整的操作系统，VM运行自身操作系统会占用较多的CPU、内存、硬盘资源。Docker不同于VM，只包含应用程序以及依赖库，基于libcontainer运行在宿主机上，并处于一个隔离的环境中，这使得Docker更加轻量高效，启动容器只需几秒钟之内完成。由于Docker轻量、资源占用少，使得Docker可以轻易的应用到构建标准化的应用中。但Docker目前还不够完善，比如隔离效果不如VM，共享宿主机操作系统的一些基础库等；网络配置功能相对简单，主要以桥接方式为主；查看日志也不够方便灵活。
        
- Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得 Docker 技术比虚拟机技术更为轻便、快捷。作为一种新兴的虚拟化方式，Docker 跟传统的虚拟化方式相比具有众多的优势。Docker 容器的启动可以在秒级实现，这相比传统的虚拟机方式要快得多；Docker 对系统资源的利用率很高，一台主机上可以同时运行数千个 Docker 容器，但是却没办法在一台主机上运行这么多的虚拟机。

---
### Docker的使用
#### Docker的安装
- 建议在linux环境下安装Docker，window也可以搭建Docker，但是环境搭建比较复杂且容易出错，使用Centos7+yum来安装Docker环境很方便。

- Docker 软件包已经包括在默认的 CentOS-Extras 软件源里。因此想要安装 docker，只需要运行下面的 yum 命令：

        yum install docker
- 安装完成后，使用下面的命令来启动 docker 服务，并将其设置为开机启动：

        service docker start
        chkconfig docker on

- 如采用CentOS 7中支持的新式 systemd 语法，如下：

        systemctl start docker.service
        systemctl enable docker.service
- 测试是否安装成功：

        docker version
- 输入上述命令，返回docker的版本相关信息，证明docker安装成功。

### Docker简单使用
- Docker部署`极简博客-MinimalBlog`的服务端

        FROM golang:1.8
        MAINTAINER Hend "964683913@qq.com"
        WORKDIR $GOPATH/src/github.com/GoProjectGroupForEducation/Go-Blog
        ADD . $GOPATH/src/github.com/GoProjectGroupForEducation/Go-Blog
        RUN go get -v github.com/GoProjectGroupForEducation/Go-Blog
        EXPOSE 8081
        ENTRYPOINT ["Go-Blog"]

#### 常用命令
- 拉取docker镜像

        docker pull image_name
- 查看宿主机上的镜像，Docker镜像保存在/var/lib/docker目录下:

        docker images
- 删除镜像

        docker rmi  docker.io/tomcat:7.0.77-jre7   /  docker rmi b39c68b7af30
- 查看当前有哪些容器正在运行

        docker ps
- 查看所有容器

        docker ps -a
- 启动、停止、重启容器命令：

        docker start container_name/container_id
        docker stop container_name/container_id
        docker restart container_name/container_id
- 后台启动一个容器后，如果想进入到这个容器，可以使用attach命令：

        docker attach container_name/container_id
- 删除容器的命令：

        docker rm container_name/container_id
- 查看当前系统Docker信息

        docker info
- 从Docker hub上下载某个镜像:

        docker pull centos:latest
        docker pull centos:latest
- 执行 `docker pull centos` 会将Centos这个仓库下面的所有镜像下载到本地repository。

---
### 参考
- 非常详细的Docker教程：[Docker 教程](http://www.runoob.com/docker/docker-tutorial.html)
- [Docker 中文社区](http://www.docker.org.cn/)
- [几张图帮你理解 docker 基本原理及快速入门](http://www.cnblogs.com/SzeCheng/p/6822905.html)
- [Docker 命令大全](http://www.runoob.com/docker/docker-command-manual.html)