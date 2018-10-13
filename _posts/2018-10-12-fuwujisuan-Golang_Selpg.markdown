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

### [源码仓库](https://github.com/ClearLoveH/Go/tree/master/Golang-selpg)

---
#### 实验需求 

使用 golang 开发 开发 Linux 命令行实用程序 中的 selpg

---
#### 参考文档

- 使用golang开发 [开发 Linux 命令行实用程序](https://www.ibm.com/developerworks/cn/linux/shell/clutil/index.html)中的 `selpg`

- [selpg的C语言实现版本](https://www.ibm.com/developerworks/cn/linux/shell/clutil/selpg.c)

- [Golang之使用Flag和Pflag](https://o-my-chenjian.com/2017/09/20/Using-Flag-And-Pflag-With-Golang/)

---

#### 输入指令格式
`selpg [--s start_page] [--e end_page] [--l lines_per_page | --f ] [ --d dest ] [ input_source ]`

---

#### 实验思路
-  selpg是一个 Linux命令行实用程序，这个名称代表 SELect PaGes。selpg 允许用户指定从输入文本抽取的页的范围，这些输入文本可以来自文件或另一个进程。
- 在做此次实验之前，我是先阅读过了selpg的C语言版本代码，再根据自己的理解对已有的C源代码改写成go语言的模式
- 在以往开发的cli项目中，常用的是flag包来进行命令行参数解析，不过这次实验我们使用 pflag 替代 goflag包，目的是为了满足 Unix 命令行规范，两者其实使用方式差别不大，只是导入包的方式略为改变——`flag "github.com/spf13/pflag"`。
