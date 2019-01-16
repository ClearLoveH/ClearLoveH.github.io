---
layout:     post
title:      "初识TUF"
subtitle:   "CoreDNS初尝试"
date:       2019-1-1 0:28
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - 服务计算
---

### 什么是CNCF？

![](/img/in-post/post-fuwujisuan/CoreDNS/CNCF.png)

- CNCF（Cloud Native Compute Foundation） 是 Linux 基金会旗下的一个组织，旨在推动以容器为中心的云原生系统。从 2016 年 11 月，CNCF 开始维护了一个名为 Cloud Native Landscape 的 repo，汇总目前比较流行的云原生技术，并加以分类，希望能为企业构建云原生体系提供参考。
- CNCF 致力于使云原生计算具有普遍性和可持续性。云原生计算使用开源软件技术栈将应用程序部署为微服务，将每个部分打包到自己的容器中，并动态编排这些容器以优化资源利用率。云原生技术使软件开发人员能够更快地构建出色的产品。

---

### 什么是CoreDNS？

![](/img/in-post/post-fuwujisuan/CoreDNS/CoreDNS.png)

- **CoreDNS**是一个Go语言实现的链式插件DNS服务端，是CNCF成员，是一个高性能、易扩展的DNS服务端。可以很方便的部署在k8s集群中，用来代替[kube-dns](http://www.cnblogs.com/allcloud/p/7614123.html)。
- CoreDNS其实就是一个 DNS 服务，而 DNS 作为一种常见的服务发现手段，所以很多开源项目以及工程师都会使用 CoreDNS 为集群提供服务发现的功能，[Kubernetes](http://www.dockone.io/article/932) 就在集群中使用 CoreDNS 解决服务发现的问题。

---
