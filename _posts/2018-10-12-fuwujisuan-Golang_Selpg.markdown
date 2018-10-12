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

# 用Go语言实现Selpg命令行程序

### 参考文档

使用golang开发 ![开发 Linux 命令行实用程序](https://www.ibm.com/developerworks/cn/linux/shell/clutil/index.html)中的 `selpg`

![selpg的C语言实现版本](https://www.ibm.com/developerworks/cn/linux/shell/clutil/selpg.c)


- 输入指令格式
`selpg [--s start_page] [--e end_page] [--l lines_per_page | --f ] [ --d dest ] [ input_source ]`