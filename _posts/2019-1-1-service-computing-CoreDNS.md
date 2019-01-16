---
layout:     post
title:      "CoreDNS初尝试"
subtitle:   "CNCF简单实践"
date:       2019-1-1 0:28
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - 服务计算
---

>新年快乐~

---

### 什么是CNCF？

![](/img/in-post/post-fuwujisuan/CoreDNS/CNCF.png)

- CNCF（Cloud Native Compute Foundation） 是 Linux 基金会旗下的一个组织，旨在推动以容器为中心的云原生系统。从 2016 年 11 月，CNCF 开始维护了一个名为 Cloud Native Landscape 的 repo，汇总目前比较流行的云原生技术，并加以分类，希望能为企业构建云原生体系提供参考。
- CNCF 致力于使云原生计算具有普遍性和可持续性。云原生计算使用开源软件技术栈将应用程序部署为微服务，将每个部分打包到自己的容器中，并动态编排这些容器以优化资源利用率。云原生技术使软件开发人员能够更快地构建出色的产品。

---

### 什么是CoreDNS？

![](/img/in-post/post-fuwujisuan/CoreDNS/CoreDNS.png)

- **CoreDNS**是一个Go语言实现的链式插件DNS服务端，是CNCF成员，是一个高性能、易扩展的DNS服务端。可以很方便的部署在[k8s集群](https://www.cnblogs.com/chris-cp/p/5766153.html)中，用来代替[kube-dns](http://www.cnblogs.com/allcloud/p/7614123.html)。
- CoreDNS其实就是一个 DNS 服务，而 DNS 作为一种常见的服务发现手段，所以很多开源项目以及工程师都会使用 CoreDNS 为集群提供服务发现的功能，[Kubernetes](http://www.dockone.io/article/932) 就在集群中使用 CoreDNS 解决服务发现的问题。
- Kubernetes包括用于服务发现的DNS服务器[Kube-DNS](http://blog.51cto.com/newfly/2059972)。 该DNS服务器利用SkyDNS的库来为Kubernetes pod和服务提供DNS请求。SkyDNS2的作者，Miek Gieben，创建了一个新的DNS服务器，CoreDNS，它采用*更模块化，可扩展*的框架构建。 Infoblox已经与Miek合作，将此DNS服务器作为Kube-DNS的替代品。
- CoreDNS利用作为Web服务器[Caddy](https://blog.csdn.net/yori_chen/article/details/79725845)的一部分而开发的服务器框架。该框架具有非常灵活，可扩展的模型，用于通过各种中间件组件传递请求。这些中间件组件根据请求提供不同的操作，例如记录，重定向，修改或维护。虽然它一开始作为Web服务器，但是Caddy并不是专门针对HTTP协议的，而是构建了一个基于CoreDNS的理想框架。
- 在这种灵活的模型中添加对Kubernetes的支持，相当于创建了一个Kubernetes中间件。该中间件使用Kubernetes API来满足针对特定Kubernetes pod或服务的DNS请求。而且由于Kube-DNS作为Kubernetes的另一项服务，kubelet和Kube-DNS之间没有紧密的绑定。您只需要将DNS服务的IP地址和域名传递给kubelet，而Kubernetes并不关心谁在实际处理该IP请求。

--- 
### CoreDNS的优势
- 对比下CoreDNS相对于[bind](https://blog.csdn.net/weixin_42125267/article/details/82117558)和[skydns](https://blog.csdn.net/zouyee/article/details/50755582)的优势：
    - *bind* 可以将解析存储到mysql或者文件中，*coredns* 也可以将解析存储到etcd或者文件中，也支持将 kubernetes 作为其后端，直接调用kubernetes的api获取解析数据，然后缓存到本地内存。*coredns*支持插件扩展，目前在第三方插件中还同时支持将powerdns及amazondns作为其后端，后续还会支持越来越来的后端。*bind*在kubernetes的应用场景下，基本无用武之地。
    - *coredns*本身就是*skydns*的继任者，支持*skydns*的所有特性，而且性能更好，更易于扩展。其插件式特性无论是*bind*还是*skydns*都无法比拟。

---
### CoreDNS官方文档

- [CoreDNS官方网站](https://coredns.io/)
- [CoreDNS安装](https://my.oschina.net/u/2306127/blog/1618543)
- [CoreDNS使用手册](https://coredns.io/manual/toc/)
- [CoreDNS源码](https://github.com/coredns)

---
### 在Kubernetes中部署CoreDNS

#### 1.下载CoreDNS部署包
https://github.com/coredns/deployment/tree/master/kubernetes

#### 2.安装CoreDNS到Kubernetes中
    
```java
[root@dev-master CoreDNS]# ./deploy.sh 10.3.0.0/24 cluster.local | kubectl apply -f -
configmap "coredns" created
deployment "coredns" created
service "kube-dns" created
```

- 检查Pod状态

    ```java
    [root@dev-master CoreDNS]# kubectl get pods --namespace=kube-system
    NAME                                        READY     STATUS    RESTARTS   AGE
    coredns-512496995-c1x9g                     1/1       Running   0          5m
    default-http-backend-905355492-nrt1z        1/1       Running   0          23h
    heapster-2450140206-dw408                   1/1       Running   2          23h
    kube-apiserver-172.16.71.200                1/1       Running   3          7d
    kube-controller-manager-172.16.71.200       1/1       Running   10         8d
    kube-proxy-172.16.71.200                    1/1       Running   49         37d
    kube-scheduler-172.16.71.200                1/1       Running   300        14d
    kubernetes-dashboard-654048359-p73r9        1/1       Running   0          23h
    monitoring-grafana-438219031-32btw          1/1       Running   1          5d
    monitoring-influxdb-3584808869-s6sh1        1/1       Running   2          23h
    nginx-ingress-controller-1644785683-9fxsp   1/1       Running   4          26d
    nginx-ingress-controller-1644785683-mw7nx   1/1       Running   2          23h
    tiller-deploy-411327518-q9zn3               1/1       Running   2          23h
    ```

    ```java
    [root@dev-master CoreDNS]# kubectl logs -f  coredns-512496995-c1x9g --namespace=kube-system
    .:53
    2017/09/13 02:36:31 [INFO] CoreDNS-011
    2017/09/13 02:36:31 [INFO] linux/amd64, go1.9, 1b60688d
    CoreDNS-011
    linux/amd64, go1.9, 1b60688d
    ```

#### 3.修改master节点和所有node节点的/etc/systemd/system/kube-kubelet.service，修改内容与上面的Corefile中的值对应。
```java
[Unit]
Description=Kubernetes Kubelet Master
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
Environment=KUBELET_IMAGE_TAG=v1.6.2
ExecStartPre=/usr/bin/mkdir -p /var/log/containers
ExecStart=/opt/bin/kubelet \
  --api-servers=http://127.0.0.1:8080 \
  --register-schedulable=false \
  --cni-conf-dir=/etc/kubernetes/cni/net.d \
  --container-runtime=docker \
  --allow-privileged=true \
  --pod-manifest-path=/etc/kubernetes/manifests \
  --hostname-override=172.16.71.200 \
  --pod-infra-container-image=172.16.80.94/mir/pause-amd64:3.0 \
  --v=3 \
  --cluster-dns=10.3.0.10 \
  --cluster-domain=cluster.local. \
  --resolv-conf=/etc/resolv.conf \
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### 4.测试CoreDNS
- 创建一个[nginx](https://www.cnblogs.com/fengff/p/8892590.html)的pod和service，测试一下coredns是否起作用

    ```java
    apiVersion: v1
    kind: Pod
    metadata:
    name: nginx
    labels:
        app: nginx
    spec:
    containers:
    - name: nginx
        image: 172.16.71.199/common/nginx:1.8.1
        ports:
        - containerPort: 80
    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: nginx
    spec:
    ports:
    - port: 80
        targetPort: 80
        protocol: TCP
    selector:
        app: nginx
    ```
    ```java
    [root@dev-master CoreDNS]# kubectl create -f nginx.yaml 
    pod "nginx" created
    service "nginx" created
    ```
- 检查Pod状态

    ```java
    [root@dev-master CoreDNS]# kubectl get pod
    NAME                              READY     STATUS    RESTARTS   AGE
    load-generator-1962471460-6v7lb   1/1       Running   3          22d
    nginx                             1/1       Running   0          1m
    php-apache-1106203038-w51jw       1/1       Running   3          22d
    ```

- 用curl测试，首先进入这个集群内的另一个pod，在pod内部访问刚才创建的nginx。

    ```java
    [root@mir2-handler-deployment-3595565332-bqk3t /]# 
    <5332-bqk3t /]# curl nginx.default.svc.cluster.local
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

- 发现可以成功访问刚才创建的nginx，说明CoreDNS起作用了，部署成功。

---

### 参考文档
- [k8s集群基本概念](https://www.cnblogs.com/chris-cp/p/5766153.html)
- [kubernetes 简介：kube-dns 和服务发现](http://www.cnblogs.com/allcloud/p/7614123.html)
- [十分钟带你理解Kubernetes核心概念](http://www.dockone.io/article/932)
- [kubernetes之kubedns部署](http://blog.51cto.com/newfly/2059972)
- [浅谈：WEB 服务器 -- Caddy](https://blog.csdn.net/yori_chen/article/details/79725845)
- [nginx概述](https://www.cnblogs.com/fengff/p/8892590.html)
- [BIND](https://blog.csdn.net/weixin_42125267/article/details/82117558)
- [DCOS之skydns](https://blog.csdn.net/zouyee/article/details/50755582)