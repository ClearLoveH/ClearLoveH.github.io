---
layout:     post
title:      "使用Swagger Editor来设计和管理API文档"
date:       2019-6-23 22:59
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - 系统分析与设计
---

### 关于Swagger Editor
Swagger 是最受欢迎的RESTful API设计文档生成工具之一，专门用来管理API一个工具。在开发过程中，API一直是纷争的聚焦点，能有效管理API（保存好记录、及时更新、方便查看、接口测试）。会让整个项目开发效率提升很大。 

而其中Swagger Edit是用来编辑API文档的小程序，简单易用。在官网上分为`Web 版本的在线编辑和本地的 Swagger-editor线下编辑`两种编辑方式。Swagger是通过`YAML`来定义你的接口规范。可以通过API文档帮你生成不同框架的服务端和客户端，方便你mock和契约测试。最后导出JSON格式的API规范，通过Swagger UI对外发布。下面带来本地的 Swagger-editor的安装及使用的小小教程。

---
### 安装Swagger Editor
1. Swagger 是使用node写的，所以我们需要先配置好nodejs相关环境。在安装好nodejs后node和npm会一并安装，npm（node package manager）是nodejs的包管理器，但是因为npm安装插件是从国外服务器下载，下载的速度较慢，我们通常会使用国内的npm镜像来替代官方的npm，通常使用淘宝的npm镜像，安装如下:
    ```
    npm install -g cnpm --registry=https://registry.npm.taobao.org
    ```
    - 安装好之后就可以使用cnpm来替代npm进行包管理了
2. http-server的安装：`npm install -g http-server`
![](/img/in-post/post-SystemAnalyse/final/SwaggerEditor1.png)

3. 从官网下载swagger-editor.zip，或者从[GitHub上下载](https://github.com/swagger-api/swagger-editor)项目然后解压。 
4. 进入swagger-editor的根目录打开cmd窗口，运行http-server，默认端口为8080。 
![](/img/in-post/post-SystemAnalyse/final/SwaggerEditor2.png)

5. 浏览器访问 http://127.0.0.1:8080/ 就可以看到swagger页面了。
![](/img/in-post/post-SystemAnalyse/final/SwaggerEditor3.png)

- 也可以直接访问在线版的Swagger Editor，可以直接使用，无需部署：  http://editor.swagger.io/


---
### 使用Swagger Editor

#### File
这个用来导入或者导出API文档在swagger edit里面进行编辑，也可以保存下来，输出为YAML或JSON格式。

![](/img/in-post/post-SystemAnalyse/final/SwaggerEditor4.png)

#### Edit
把编辑区转换成YAML格式或者OpenAPI 3的格式

![](/img/in-post/post-SystemAnalyse/final/SwaggerEditor5.png)

#### Generate Server
把接口文档生成服务器文件，导出一个压缩包，接口文档生成java、js等服务器文件。是非常实用的工具。 

![](/img/in-post/post-SystemAnalyse/final/SwaggerEditor6.png)

#### Generate Client

生成可以查看的API文档,编写好API文档的下一步就是展示文档。在这里我们可以选择导出什么样格式的API文档。 

![](/img/in-post/post-SystemAnalyse/final/SwaggerEditor7.png)


---
### 文档的编写语法
Swagger的API文档是用YAML语法来进行编写的，而YAML的具体语法可参考：[YAML 语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html?f=tt)，文档的各字段的解释如下：
```yaml
swagger: '2.0'                      # swagger的版本
info:
  title: 文档标题
  description:  描述
  version: "v1.0"                   # 版本号
  termsOfService: ""                # 文档支持截止日期
  contact:                          # 联系人的信息
    name: ""                        # 联系人姓名
    url: ""                         # 联系人URL
    email: ""                       # 联系人邮箱
  license:                          # 授权信息
    name: ""                        # 授权名称，例如Apache 2.0
    url: ""                         # 授权URL
host: api.haofly.net                # 域名，可以包含端口，如果不提供host，那么默认为提供yaml文件的host
basePath: /                         # 前缀，比如/v1
schemes:                            # 传输协议
  - http
  - https

securityDefinitions:                # 安全设置
  api_key:
    type: apiKey
    name: Authorization             # 实际的变量名比如，Authorization
    in: header                      # 认证变量放在哪里，query或者header
  OauthSecurity:                    # oauth2的话有些参数必须写全
    type: oauth2
    flow: accessCode                # 可选值为implicit/password/application/accessCode
    authorizationUrl: 'https://oauth.simple.api/authorization'
    tokenUrl: 'https://oauth.simple.api/token'
    scopes:
      admin: Admin scope
      user: User scope
      media: Media scope
  auth:
    type: oauth2
    description: ""                 # 描述
    authorizationUrl: http://haofly.net/api/oauth/
    name: Authorization             # 实际的变量名比如，Authorization
    tokenUrl:
    flow: implicit                  # oauth2认证的几种形式，implicit/password/application/accessCode
    scopes:
      write:post: 修改文件
      read:post: 读取文章

security:                           # 全局的安全设置的一个选择吧
  auth:
    - write:pets
    - read:pets

consumes:                           # 接收的MIME types列表
  - application/json                # 接收响应的Content-Type
  - application/vnd.github.v3+json

produces:                           # 请求的MIME types列表
  - application/vnd.knight.v1+json  # 请求头的Accept值
  - text/plain; charset=utf-8
tags:                               # 相当于一个分类
  - name: post  
    description: 关于post的接口

externalDocs:
  description: find more info here
  url: https://haofly.net

paths:                              # 定义接口的url的详细信息
  /projects/{projectName}:          # 接口后缀，可以定义参数
    get:
      tags:                         # 所属分类的列表
        - post  
      summary: 接口描述              # 简介
      description:                  # 详细介绍
      externalDocs:                 # 这里也可以加这个
        description:
        url:
      operationId: ""               # 操作的唯一ID
      consumes: [string]            # 可接收的mime type列表
      produces: [string]            # 可发送的mime type列表
      schemes: [string]             # 可接收的协议列表
      deprecated: false             # 该接口是否已经弃用
      security:                     # OAuth2认证用
        - auth: 
            - write:post
            - read: read
      parameters:                   # 接口的参数
        - name: projectName         # 参数名
          in: path                  # 该参数应该在哪个地方，例如path、body、query等，但是需要注意的是如果in body，只能用schema来指向一个定义好的object，而不能直接在这里定义
          type: string              # 参数类型
          allowEmptyValue: boolean          # 是否允许为空值
          description: 项目名        # 参数描述
          required: true            # 是否必须
          default: *                # 设置默认值
          maximum: number           # number的最大值
          exclusiveMaximum: boolean # 是否排除最大的那个值
          minimum: number           # number的最小值
          exclusiveMinimum: boolean
          maxLength: integer        # int的最大值
          minLength: integer
          enum: [*]                 # 枚举值
          items:                    # type为数组的时候可以定义其项目的类型
        - $ref: "#/parameters/uuidParam"   # 这样可以直接用定义好的
      responses:                    # 设置响应
        200:                        # 通过http状态来描述响应
          description: Success      # 该响应的描述
          schema:                   # 定义返回数据的结构
            $ref: '#/definitions/ProjectDataResponse'  # 直接关联至某个model

  /another: # 另一个接口
      responses:
        200:
            description:
            schema:
              type: object
              properitis:
                data:
                    $ref: '#/definitions/User' # 关联

definitions:            # Model/Response的定义，这里的定义不强制要求返回数据必须和这个一致，但是在swagger-ui上，会展示这里面的字段。
  Product:              # 定义一个model
    type: object        # model类型
    properties:         # 字段列表
      product_id:       # 字段名
        type: integer   # 字段类型
        description:    # 字段描述
      product_name:
        type: string
        description: 
  ProjectDataResponse:
    type: object
    properties:
        data:
            $ref: '#/definitions/ProjectResponse'  # model之间的关联，表示在data字段里面包含的是一个ProjectResponse对象
parameters:             # 可以供很多接口使用的params
  limitParam:
    name: limit
    in: query
    description: max records to return
    required: true
    type: integer
    format: int32
responses:              # 可以供很多接口使用的responses
  NotFound:
    description: Entity not found.
```