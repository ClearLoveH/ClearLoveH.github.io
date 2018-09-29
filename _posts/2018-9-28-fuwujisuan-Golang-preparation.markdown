---
layout:     post
title:      "Ubuntu下安装go语言开发环境"
subtitle:   "服务计算博客"
date:       2018-9-28 16:00
author:     "Heng"
header-img: "img/艾欧尼亚.jpg"
catalog: true
tags:
    - 服务计算
---

>此博客分享乌班图虚拟机配置go语言开发环境

>因为我是vim用户，所以就没有进行VScode的安装，这部分关于VScode配置过程就直接忽略。但是我自己平时在win下git的日常使用以及打开各种文件还是很喜欢使用VScode的，强推！

---
## 基本环境搭建
---

### 小插曲：Ubuntu中yum的安装

> 因为我安装的是乌班图虚拟机，Ubuntu默认的包管理器为apt，而yum是Fedora，RedHat以及CentOS的包管理器。这里也顺带讨论下乌班图下yum的安装步骤及遇到的问题。

###### 关于yum

- yum是一个软件包管理器，很多时候安装一个软件安装一个软件需要安装1个或者多个其他软件yum仓库用来存放所有的现有的.rpm包，当使用yum安装一个rpm包时，需要依赖关系时会自动在仓库中查找依赖软件并安装。
- yum将我们对软件及其依赖包的管理和安装变得极为简单。
- Ubuntu默认的包管理器为apt，而yum是fedora，redhat的包管理器。所以我们使用yum需要注意依赖的问题。

###### 安装步骤

1. apt-get install build-essential，通过apt检测是否安装了build-essential程序包

    ![](/img/in-post/post-fuwujisuan/GolangInstall/1.png)

2. apt-get install yum，通过apt安装yum

    ![](/img/in-post/post-fuwujisuan/GolangInstall/2.png)

3. 若出现权限不足的问题，只需获取root权限即可 `sudo -i` 或 `su root`

    ![](/img/in-post/post-fuwujisuan/GolangInstall/3.png)

    临时获取root权限之后：

    ![](/img/in-post/post-fuwujisuan/GolangInstall/4.png)

>装好yum包之后即可使用老师博客的指令来继续后面的golang环境配置

---
### 配置golang于我的乌班图

#### 安装

`$ sudo yum install golang` （root权限下sudo可略）

-  查看golang的安装目录

    `$ rpm -ql golang |more` 

-  测试golang是否安装成功

    ![](/img/in-post/post-fuwujisuan/GolangInstall/5.png)

#### 设置环境变量

- 创建工作空间

    `$ mkdir $HOME/gowork`

- 配置环境变量

    `$ vim ~/.profile`

    添加

    `export GOPATH=$HOME/gowork`

    `export PATH=$PATH:$GOPATH/bin`
    
    ![](/img/in-post/post-fuwujisuan/GolangInstall/6.png)

- 检查配置是否完成

    ![](/img/in-post/post-fuwujisuan/GolangInstall/7.png)


#### 创建hello world

- 创建源代码目录：
    
    `$ mkdir $GOPATH/src/github.com/github-user/hello -p`

- 创建 hello.go :

    `touch hello.go`

- vim修改hello.go文件：

    ![](/img/in-post/post-fuwujisuan/GolangInstall/8.png)

- 保存，在终端运行

    ![](/img/in-post/post-fuwujisuan/GolangInstall/9.png)

- 运行成功，本次实验结束

---

>这里便是我在服务计算本次golang的实验过程中的一些总结，希望能够帮助到同样是使用`Ubuntu`来做此次试验的其他同学们。