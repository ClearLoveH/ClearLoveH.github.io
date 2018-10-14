---
layout:     post
title:      "用Go语言实现Selpg命令行程序"
subtitle:   "服务计算博客"
date:       2018-10-12 9:33
author:     "Heng"
header-img: "img/比尔吉沃特.jpg"
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

- [golang 中bufio包的用法](https://blog.csdn.net/weiyuefei/article/details/78026026)

---

#### Usage

    selpg [-s start_page] [-e end_page] [-l lines_per_page | -f ] [ -d print_dest ] filename

    ``
    Options:
    -s, --start_page int 	       Input Your startPage
    -e, --end_page int	           Input Your endPage
    -d, --dest string		       (Optional) Enter printing destination
    -f, --pageBreak		           (Optional) Choosing pageBreaks mode
    -l, --lines_per_page int       (Optional) Choosing pageLen mode, enter pageLen
    

---

#### 实验思路
-  selpg是一个 Linux命令行实用程序，这个名称代表 SELect PaGes。selpg 允许用户指定从输入文本抽取的页的范围，这些输入文本可以来自文件或另一个进程。
- 在做此次实验之前，我是先阅读过了selpg的C语言版本代码，再根据自己的理解对已有的C源代码改写成go语言的模式
- 在以往开发的cli项目中，常用的是flag包来进行命令行参数解析，不过这次实验我们使用 pflag 替代 goflag包，目的是为了满足 Unix 命令行规范，两者其实使用方式差别不大，只是导入包的方式略为改变——`flag "github.com/spf13/pflag"`。
- 在看完了相关的实验文档之后，我分析了这次selpg的需要实现的内容，总结来说就是简单的五个部分：
    - 读取输入的命令
    - 解析命令，并判断其中参数是否合法
    - 命令正确，执行命令
    - 命令错误，则给出报错信息
    - 将结果输出
- 本次实验还使用`bufio`来实现了带缓存的I/O操作，[参考博客](http://www.cnblogs.com/golove/p/3282667.html)，又因为bufio包封装了io.Reader和io.Writer，使用其来进行输入输出的重定向变得很方便。

---

#### 代码结构：
代码主要三部分函数组成：

- main：主函数
- processArgs：处理参数，进行错误处理
- processInput：经过processArgs函数后，执行命令

---

#### 实验结果：

- `selpg -s1 -e1 test`——默认每20行为一页

    ![](/img/in-post/post-fuwujisuan/selpg/1.png)

- ` selpg -s1 -e1 -l15 test`

    ![](/img/in-post/post-fuwujisuan/selpg/5.png)

- `selpg -s1 -e1 test >testout.txt`

    把第一页输出至`testout.txt`文件中
    
    ![](/img/in-post/post-fuwujisuan/selpg/2.png)

- `selpg -s1 -e1 -l10 test >testout.txt`

    把第一页的前10行输出至该文件中

    ![](/img/in-post/post-fuwujisuan/selpg/3.png)

- `错误参数`

    ![](/img/in-post/post-fuwujisuan/selpg/4.png)

- `测试带换页符的文件`
    
    - 创建带换页符文件

        ![](/img/in-post/post-fuwujisuan/selpg/6.png)

    - `selpg -s1 -e4 -f out >testout.txt`

        ![](/img/in-post/post-fuwujisuan/selpg/7.png)

- ` selpg -s1 -e1 test | grep -n "1"`

    测试查找文件里符合条件的字符串

    ![](/img/in-post/post-fuwujisuan/selpg/8.png)