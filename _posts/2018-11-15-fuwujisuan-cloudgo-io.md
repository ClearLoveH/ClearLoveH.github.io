---
layout:     post
title:      "Go——处理 web 程序的输入与输出"
subtitle:   "服务计算博客"
date:       2018-11-15 9:33
author:     "Heng"
header-img: "img/比尔吉沃特.jpg"
catalog: true
tags:
    - 服务计算
---

### [源码仓库](https://github.com/ClearLoveH/Go/tree/master/CloudGo)

---
### 概述

设计一个 web 小应用，展示静态文件服务、js 请求支持、模板输出、表单处理、Filter 中间件设计等方面的能力。（不需要数据库支持）

### 任务要求

编程 web 应用程序 `cloudgo-io`

---

#### 基本要求实现

- 运行go run main.go，访问`127.0.0.1:8080`主页
![](/img/in-post/post-fuwujisuan/cloudGo/1.png)

    - 这里我们使用`template.ParseFiles()`来实现模板的渲染和输出
        ```go
            t := template.Must(template.ParseFiles("templates/index.html"))
        ```

- 支持静态文件服务
    ![](/img/in-post/post-fuwujisuan/cloudGo/5.png)
    
    - 打开css文件
    ![](/img/in-post/post-fuwujisuan/cloudGo/6.png)
    ![](/img/in-post/post-fuwujisuan/cloudGo/7.png)
    - 打开images目录下的图片
    ![](/img/in-post/post-fuwujisuan/cloudGo/9.png)
    ![](/img/in-post/post-fuwujisuan/cloudGo/8.png)

    - 打开js目录下timeNow.js
    ![](/img/in-post/post-fuwujisuan/cloudGo/10.png)
    ![](/img/in-post/post-fuwujisuan/cloudGo/11.png)

- 支持简单 js 访问，输入`http://127.0.0.1:8080/timeNow`访问js，显示当前时间
![](/img/in-post/post-fuwujisuan/cloudGo/2.png)

    ```go
    mx.HandleFunc("/timeNow", showTime(formatter)).Methods("GET")
    ```
    `对url为"/timeNow"的请求进行处理，具体实现是下面定义的showTime函数`，主要实现是获得并显示出当前请求的具体时间。
    ```java
    func showTime(formatter *render.Render) http.HandlerFunc {
        return func(w http.ResponseWriter, req *http.Request) {
            formatter.Text(w, http.StatusOK,"The date now is: " +  time.Now().Format("2006/01/02 15:04:05"))
        }
    }
    ```

- 提交表单，并输出一个表格
![](/img/in-post/post-fuwujisuan/cloudGo/3.png)
![](/img/in-post/post-fuwujisuan/cloudGo/4.png)

- 对 /unknown 给出开发中的提示，返回码 501
![](/img/in-post/post-fuwujisuan/cloudGo/12.png)

- 客户端返回数据
![](/img/in-post/post-fuwujisuan/cloudGo/13.png)


---



