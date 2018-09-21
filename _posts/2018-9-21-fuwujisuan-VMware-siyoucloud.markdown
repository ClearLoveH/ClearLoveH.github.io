---
layout:     post
title:      "VMware搭建私有云"
subtitle:   "服务计算博客"
date:       2018-9-21 8:42
author:     "Heng"
header-img: "img/艾欧尼亚.jpg"
catalog: true
tags:
    - 服务计算
---

>这次作业许多人都在使用virtualbox来配置私有云，而我最近更中意VMware，于此也写下一篇小博客总结一下VMware私有云的配置过程以及中间出现的小问题。

---
## 基本环境搭建
- 安装VMware

    VMware需要秘钥，支持正版，以后大家可以自行购买，现在我们做小实验就直接百度搜索vmware workstation 14 pro秘钥即可使用。

- 安装linux
    在校园网环境下我们可以使用IPV6通道去[清华大学开源镜像站](https://mirrors6.tuna.tsinghua.edu.cn/)自行下载我们喜爱的linux版本，下载速度可观。

- linux虚拟机的硬件资源分配

    ![](../img/服务计算-1.png)

    我们需保证虚拟机有足够的内存还有硬盘满足我们使用。

    ![](../img/服务计算-2.png)