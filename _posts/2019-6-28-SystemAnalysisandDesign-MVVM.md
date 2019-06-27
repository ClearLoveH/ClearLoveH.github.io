---
layout:     post
title:      "设计模式——MVVM"
date:       2019-6-28 0:25
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - 系统分析与设计
---

### 什么是MVVM？

>MVP是对MVC的C的演化，MVVM是对MVP的P的演化。

#### MVC

![](/img/in-post/post-SystemAnalyse/final/MVC.jpg)

MVC全名是`Model-View-Controller`，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种`业务逻辑、数据、界面显示分离`的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑
- **Model**：应用程序中用于处理应用程序数据逻辑的部分，它主要负责网络请求，数据库处理，I/O等的操作。
- **View**：应用程序中处理数据显示的部分。
- **Controller**：应用程序中处理用户交互的部分，接受用户的输入并调用Model和View去完成用户的需求。

MVC架构肯定是有优点的，它通过Controller来掌控全局，同时将View展示和Model的变化分离开。

但是一旦一个页面的业务繁多复杂的话，Controller里的代码就会越来越臃肿，这是因为View不仅仅要负责业务逻辑，同时还要加载应用的布局和初始化使用界面，也就是说View杂糅了Controller和View的功能，这不能算作一个真正意义上的MVC。因此在这种情况下，MVP架构呼之欲出


#### MVP

![](/img/in-post/post-SystemAnalyse/final/MVP.jpg)

MVP全名是`Model-View-Presenter`。MVP是从经典的MVC模式演变而来，它们的基本思想有相通的地方：Controller/Presenter负责逻辑的处理，Model提供数据，View负责显示。

作为一种新的模式，MVP与MVC有着一个重大的区别：**MVP中的View并不直接使用Model**，它们之间的通信是通过`Presenter(MVC中的Controller)`来进行的，所有的交互都发生在Presenter内部，而在MVC中View会直接从Model中读取数据而不是通过Controller
- **Model**：应用程序中用于处理应用程序数据逻辑的部分，它主要负责网络请求，数据库处理，I/O等的操作。
- **View**：应用程序中处理数据显示的部分。
- **Presenter**：整个MVP体系的控制中心，作为View与Model交互的中间纽带，处理View与Model间的交互和业务逻辑。

综上所述，MVP的优点如下：
- `模型与视图完全分离`，我们可以修改视图而不影响模型；
- `项目代码结构清晰`，一看就知道什么类干什么事情；
- 我们可以将`一个Presenter用于多个视图`，而不需要改变Presenter的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化更频繁
- `协同工作`（例如在设计师没出图之前可以先写一些业务逻辑代码）

MVP也有不足之处：
- **接口过多**，一定程度影响了编码效率。
- 一定程度上导致**Presenter的代码量过大**。

为了降低Presenter中业务繁多的问题，Google又推出了MVVM，试图通过数据驱动来减少Presenter的代码量


#### MVVM

![](/img/in-post/post-SystemAnalyse/final/MVVM.png)

MVVM是一种思想。即UI随数据更改而更改。它独特的地方就在于它的DataBinding特性。但请注意，虽然Google2015专门发布了一个库叫做DataBinding，但是这里说的DataBinding并不是指的某个具体的库，而是指的一种“**行为**”，指的是一类的“**将数据Model映射到View**”的框架。


MVVM分为三个部分：分别是M（Model，模型层），V（View，视图层），VM（ViewModel，V与M`连接的桥梁`，也可以看作为控制器）
1. **Model**：模型层，主要负责业务数据相关；
2. **View**：视图层，顾名思义，负责视图相关，细分下来就是html+css层；
    - View层负责处理UI相关的工作，我们不在View层写业务逻辑和业务数据相关的代码。
3. **ViewModel**：V与M沟通的桥梁，负责监听M或者V的修改，是实现MVVM双向绑定的要点。
    - ViewModel只负责`业务逻辑`，不做任何和UI相关的事情。更新UI通过数据绑定实现，尽量在ViewModel里面做。

MVVM支持双向绑定，意思就是当M层数据进行修改时，VM层会监测到变化，并且通知V层进行相应的修改，反之修改V层则会通知M层数据进行修改，以此也实现了视图与模型层的相互解耦。

---
### MVVM特点

#### 数据驱动
在常规的开发模式中，数据变化需要更新UI的时候，需要先获取UI控件的引用，然后再更新UI。获取用户的输入和操作也需要通过UI控件的引用。在MVVM中，这些都是通过数据驱动来自动完成的，数据变化后会自动更新UI，UI的改变也能自动反馈到数据层，数据成为主导因素。这样MVVM层在业务逻辑处理中只要关心数据，不需要直接和UI打交道，在业务处理过程中简单方便很多。

#### 低耦合度
MVVM模式中，数据是独立于UI的。

数据和业务逻辑处于一个独立的ViewModel中，ViewModel只需要关注数据和业务逻辑，不需要和UI或者控件打交道。UI想怎么处理数据都由UI自己决定，ViewModel不涉及任何和UI相关的事，也不持有UI控件的引用。即便是控件改变了（比如：TextView换成EditText），ViewModel也几乎不需要更改任何代码。它非常完美的解耦了View层和ViewModel，解决了上面我们所说的MVP的痛点。

#### 更新UI
在MVVM中，数据发生变化后，我们在工作线程直接修改（在数据是线程安全的情况下）ViewModel的数据即可，不用再考虑要切到主线程更新UI了，这些事情相关框架都帮我们做了。

#### 团队协作
MVVM的分工是非常明显的，由于View和ViewModel之间是松散耦合的：一个是处理业务和数据、一个是专门的UI处理。所以，完全由两个人分工来做，一个做UI（XML和Activity）一个写ViewModel，效率更高。

#### 可复用性
一个ViewModel可以复用到多个View中。同样的一份数据，可以提供给不同的UI去做展示。对于版本迭代中频繁的UI改动，更新或新增一套View即可。如果想在UI上做A/B Testing，那MVVM是你不二选择。

#### 单元测试

ViewModel层做的事是数据处理和业务逻辑，View层中关注的是UI，两者完全没有依赖。不管是UI的单元测试还是业务逻辑的单元测试，都是低耦合的。在MVVM中数据是直接绑定到UI控件上的（部分数据是可以直接反映出UI上的内容），那么我们就可以直接通过修改绑定的数据源来间接做一些前端/Android的UI上的测试。


---
### Vue和MVVM四部分的关系

![](/img/in-post/post-SystemAnalyse/final/MVVM-Vue.png)

对应关系：
- 视图：对应真实的html和css
- 视图模型：对应Vue的模板语法
- 绑定器：对应v-bind v-model @click :prop等绑定数据语法
- 模型：Vue的实例中的那些属性 datamethods $computed 等等

---
### MVVM架构在Android上的架构实现

![](/img/in-post/post-SystemAnalyse/final/MVVM-Android.png)
