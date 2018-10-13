---
layout:     post
title:      "用Go语言实现Selpg命令行程序"
subtitle:   "服务计算博客"
date:       2018-10-12 9:33
author:     "Heng"
header-img: "img/艾欧尼亚2.jpg"
catalog: true
tags:
    - 服务计算
---

[源码仓库](https://github.com/ClearLoveH/Go/tree/master/Golang-selpg)

---
#### 参考文档

- 使用golang开发 [开发 Linux 命令行实用程序](https://www.ibm.com/developerworks/cn/linux/shell/clutil/index.html)中的 `selpg`

- [selpg的C语言实现版本](https://www.ibm.com/developerworks/cn/linux/shell/clutil/selpg.c)

---

#### 输入指令格式
`selpg [--s start_page] [--e end_page] [--l lines_per_page | --f ] [ --d dest ] [ input_source ]`

---

#### 实验思路

在做此次实验之前，我是先阅读过了，selpg的C语言版本，再根据自己的理解对已有的C源代码改写成go语言的模式

