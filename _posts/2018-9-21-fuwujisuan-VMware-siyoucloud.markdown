---
layout:     post
title:      "VMware搭建私有云"
subtitle:   "服务计算博客"
date:       2018-9-21 8:42
author:     "Heng"
header-img: "img/德玛西亚.jpg"
catalog: true
tags:
    - 服务计算
---

>这次作业许多人都在使用virtualbox来配置私有云，而我最近更中意VMware，于此也写下一篇小博客总结一下VMware私有云的配置过程以及中间出现的小问题。

---
## 基本环境搭建
#### 安装VMware

VMware需要秘钥，支持正版，以后大家可以自行购买，现在我们做小实验就直接百度搜索vmware workstation 14 pro秘钥即可使用。

#### 安装linux
    
在校园网环境下我们可以使用IPV6通道去[清华大学开源镜像站](https://mirrors6.tuna.tsinghua.edu.cn/)自行下载我们喜爱的linux版本，下载速度可观。

#### linux虚拟机的硬件资源分配

![](/img/in-post/post-fuwujisuan/fuwujisuan-1.png)

我们需保证虚拟机有足够的内存还有硬盘满足我们使用。

![](/img/in-post/post-fuwujisuan/fuwujisuan-2.png)

----

## Xshell的使用

#### 关于Xshell
Xshell可以在Windows界面下用来访问远端不同系统下的服务器，从而比较好的达到远程控制终端的目的，通俗来讲就是远程操控。

Xshell是一款功能强大且安全的终端模拟器，支持SSH、SFTP、TELNET、RLOGIN和SERIAL。

这次实验的目的就是需要我们做到可以使用远程桌面来远程访问我们的虚拟机，而Xshell的功能就是可以实现这样的目的，并且我们的操作都提供了一个可视化的过程。

#### Xshell配置过程

![](/img/in-post/post-fuwujisuan/3.png)

重新创建新的远程链接：

![](/img/in-post/post-fuwujisuan/4.png)

![](/img/in-post/post-fuwujisuan/5.png)

可以看见，新建远程链接，Xshell客户端会提供课选择的协议，包括ssh，我们可以输入虚拟机的IP和对应端口来进行远程链接。

远程链接成功后的结果：

![](/img/in-post/post-fuwujisuan/6.png)

#### Xshell使用总结：
我们可以发现，相对于通过命令行窗口来实现虚拟机的远程链接，Xshell的可视化界面将我们的操作变得更易于理解，每一步都清晰简单了许多，是我们值得入手的一个软件。

---

## 关于实验中出现的一个很大的“坑”

在进行试验的过程中，我有将虚拟机从笔记本中备份至宿舍主机里（使用VMware自带的克隆功能）:

![](/img/in-post/post-fuwujisuan/8.png)

但是克隆至宿舍主机后，我们再使用SSH发现笔记本无法通过SSH远程连接主机上的虚拟机。通过排查了许久之后，我终于发现了此问题的“罪魁祸首”————子网IP。

在克隆至宿舍主机之后，我一直认为克隆是会将虚拟机的所有设置完整不变的移植过去，但是对比了两台电脑上虚拟机的子网IP之后，（如下）才发现发生了改变，将子网IP改回原虚拟机的IP问题便成功解决了，我先行帮以后克隆虚拟机的同学们来排了此坑。

![](/img/in-post/post-fuwujisuan/7.png)


>这里便是我在服务计算本次实验过程中的一些总结，希望能够帮助到遇到同样问题的其他同学们。