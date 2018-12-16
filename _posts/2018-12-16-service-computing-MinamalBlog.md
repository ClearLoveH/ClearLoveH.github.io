---
layout:     post
title:      "极简博客——Go后端开发"
subtitle:   "简单 web 服务与客户端开发实战"
date:       2018-12-14 12:20
author:     "Heng"
header-img: "img/比尔吉沃特.jpg"
catalog: true
tags:
    - 服务计算
---

## [源码仓库](https://github.com/GoProjectGroupForEducation/Go-Blog)

---

### 本人负责的后台模块

- 用户的登陆与注册
- 用户的关注与取关
- 更新用户头像
- 获取所有用户及根据id获取用户信息
- 后端下载前端发送的文件的接口

### 负责部分的API设计

- GET /user/allusers 获取所有用户
- GET, PUT /user/{user_id} 获取含有对应id的用户
- GET /user/{user_id}/follower 获取含有对应id的用户的follower
- GET /user/{user_id}/following 获取含有对应id的用户follow的用户
- POST /user/login 登陆请求
- POST /user/register 注册请求
- POST /user/icon/{filename} 更新用户头像
- POST /user/follow follow某用户
- POST /user/unfollow 取消follow某用户

---

### 实验体会
- 本次实验为了实现前后端开发的分离，我们通过`Easy Mock`来模拟生成所需要的接口，对于前端攻城狮来说，生成的接口供前端进行独立于后端开发来进行测试，因为Mock服务会生成随机数据，模拟各种场景，而不需要等待我们后端开发了相应的接口才开始测试，这样就实现了前后端的并行开发，效率大大提升；而对于我们后端开发的攻城狮来说，预先讨论出的API接口，等于是给了我们后端攻城狮开发提供了需求，所以不仅是前端，Mock对于后端开发也是有一个“产品经理”的作用，根据生成的API来一步步实现所有接口，而对于在完成中的接口的测试，我们使用postman来测试即可，所以前后端在开发过程基本没有交集，可以说是有条不紊地各自进行工作。

- 关于`RESTful架构`
    - 本次实验我们后端的开发使用的是`REST风格`
        - REST即表述性状态传递（Representational State Transfer，简称REST），它是一种针对网络应用的设计和开发方式，可以降低开发的复杂性，提高系统的可伸缩性。
            - REST 是Web服务的一种架构风格
            - 使用HTTP、URL 等广泛流行的标准和协议
            - 轻量级、跨平台‘跨语言的架构设计。
            - REST是一种设计风格。它不是一种标准，也不是一种软件，而是一种思想。
            - REST通常基于使用HTTP ,URL ,和XML,json 以及 HTML这些现有的广泛流行的协议和标准。
        - RESTful是什么?
            - RESTful对应的中文是 REST 式的。
            - RESTful Web Service是一种常见的REST的应用，是遵守了REST风格的Web服务。
            - REST式的Web服务是一种ROA （面向自资源的架构）
        - REST 架构的主要原则
            - 网络上的所有事物都可以被抽象为资源 （Resource）
            - 每个资源都有一个唯一的资源标识符 （Resource identifier）
            - 同一资源具有多种表现形式 （XML  JSON等）
            - 对资源的各种操作不会改变资源标识符。
            - 所有的操作都是无状态的  （Stateless）
            - 符合REST原则的架构方式即可称为RESTful
        - `RESTful的无状态性`：
            - 无状态性使得客户端和服务端不必保存对方的详细信息，`服务器只需要处理当前Request，而不必了解前面Request的历史`。从而可以更容易地释放资源。
        - RESTful 架构的对于资源的操作：
            - 资源操作 
                - GET       获得一个资源
                - POST      创建一个新的资源
                - PUT       修改一个资源的状态
                - DELETE    删除一个资源
            - 资源呈现的主要方式
                - XML    
                - JSON   
                - ...

- `关于jwt的使用`
    - 这次实验对于我来说最大的收获就是关于jwt的使用经验了。我们学习web和计网都有了解到许多用户认证的方式，常用的就是我们经常了解到的Cookie认证机制：
        - Cookie认证机制就是为一次请求认证在服务端创建一个Session对象，同时在客户端的浏览器端创建了一个Cookie对象；通过客户端带上来Cookie对象来与服务器端的session对象匹配来实现状态管理的。默认的，当我们关闭浏览器的时候，cookie会被删除。但可以通过修改cookie 的expire time使cookie在一定时间内有效。
    - 而这次实验为了方便实现用户认证，我们所选择的是JWT 产生 token的方式来实现用户认证。JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。相对于cookie，token认证机制有许多的优点，像我们这次实验出现的跨域访问问题，cookie是不支持跨域访问的，但是token是支持的；相对于cookie，token的性能也要好上不少：一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多。
    - `本次实验实现token 认证的步骤：`
        - 我们创建了一个`AuthToken`类来存取每一个新建用户的关键信息（唯一标识id与用户名）和该用户对应生成的token，而在前端每次请求login接口时，我们通过接受前端传来的登陆用户的username和password，在数据库中寻找该用户，若找到则返回该用户的token给前端，前端则会在登陆成功的情况下，把token添加在后续请求的报文头部中，而后端对于需要认证的请求，都会检查是否有token，以及该token是否在我们数据库中存在，存在才会允许前台的请求通过，否则告诉前端：need to login。
        - AuthToken类
            ```java
            type AuthToken struct {
                // TokenID      int    `json:"id"`
                Token        string `json:"token"`
                AuthorizedID int    `json:"authorized_id"`
                Username     string `json:"username"`
                ExpiredTime  string `json:"expired_time"`
            }
            ```
        - 我们发送一个注册新的用户的请求，后端在创建新用户成功后（用户名已存在则创建失败）返回给前端关于该用户的基本信息：
            ![](/img/in-post/post-fuwujisuan/MinimalBlog/1.png)
        - 重复创建则返回前端错误信息：
            ![](/img/in-post/post-fuwujisuan/MinimalBlog/2.png)
        - 而前端在知道成功创建好新用户之后，可以使用该用户账号密码进行登陆，登陆成功，返回该用户的token，前端便将返回的token添加在后续请求的报文头部中，以满足后端对需要认证的请求的验证。
            ![](/img/in-post/post-fuwujisuan/MinimalBlog/3.png)
        - 后端使用postman，手动把生成的token添加在header中，若没添加在头部，对于需要认证才能通过的请求，后端就会返回错误信息：
            ![](/img/in-post/post-fuwujisuan/MinimalBlog/4.png)
        - 添加在header之后，再次发送上次的follow请求:
            ![](/img/in-post/post-fuwujisuan/MinimalBlog/5.png)
        - 可见我们的JWT认证方式基本实现且测试成功
    - `那么这次实验，我们的token是怎么生成的呢？`
        - 在GenerateAuthToken方法中，通过使用java的md5类来生成唯一的token。而为了保证token的唯一性，我们通过token生成的当前时间加上三小时生成失效时间，尾部再加上具有唯一性的用户id，这样出来的source字符串必是唯一的，通过对接收的传输数据执行散列运算，计算出的校验和最后生成了我们的唯一的token。
            ```java       
            var checksum = []byte("heng-is-a-very-handsome-boy")

            func GenerateAuthToken(userID int, username string) models.AuthToken {
                h := md5.New()
                expiredTime := time.Now().UnixNano()/1e6 + 1000*60*60*3 // expired time is 3 hours
                source := strconv.FormatInt(expiredTime, 10) + strconv.Itoa(userID)
                io.WriteString(h, source)
                token := fmt.Sprintf("%x", h.Sum(checksum))
                authToken := models.AuthToken{
                    // 0,
                    token,
                    userID,
                    username,
                    strconv.FormatInt(expiredTime, 10),
                }
                models.CreateToken(authToken)
                // authToken.TokenID = id
                return authToken
            }
            ```
- 关于 `BoltDB`
    - 本次后端开发所使用的数据库是`boltDB`，在开始开发之前我也学习了许多关于boltDB的知识。
    - [BoltDB官方文档](https://github.com/boltdb/bolt)
    - Bolt是基于纯Go语言开发的K-V存储，灵感来自于Howard Chu的LMDB项目。该项目目标是开发一个简单、快速、可靠的无服务端的数据库。API非常小巧和简洁，仅仅关注如何获取或设置数据，这就是全部。 BoltDB是K-V存储，没有关系型数据库中类似表、行、列结构，数据以key-value对存储（类似GO语言中的map） 。相似的key-value对存储在同一bucket中，类似于关系型数据库中的Table。因此，为了获取一个value，需要知道其所在的bucket以及对应的key。
    - BoltDB设计源于LMDB，具有以下特点：
        - 直接使用API存取数据，没有查询语句；
        - 支持完全可序列化的ACID事务，这个特性比LevelDB强；
        - 数据保存在内存映射的文件里。没有wal、线程压缩和垃圾回收；
        - 通过COW技术，可实现无锁的读写并发，但是无法实现无锁的写写并发，这就注定了读性能超高，但写性能一般，适合与读多写少的场景。
    - 本次实验使用BoldDB最大的感觉就是使用起来比mysql轻便许多，对比mysql数据库的操作可帮助我们更好对比二者的差别。
        - 如果我们在使用mysql，当我们需要创建一个新表来存储我们定义好的一个类的信息时，需要定义所有列的属性名还有数据类型，如:

                mysql> CREATE TABLE user (
                -> id int,
                -> name varchar(20)
                -> );
        - 但是使用BoldDB，我们的读写操作被简化了，只需提供对应的bucket及其键值，我们就可以完成读写操作，例：

                将key/value对写入Bucket：
                    db.Update(func(tx *bolt.Tx) error {
                        b := tx.Bucket([]byte("user"))
                        err := b.Put([]byte("id"), []byte("2"))
                        return err
                    })

                从Bucket中获取value：
                    db.View(func(tx *bolt.Tx) error {
                        b := tx.Bucket([]byte("user"))
                        v := b.Get([]byte("id"))
                        fmt.Printf("The answer is: %s\n", v)
                        return nil
                    })