---
layout:     post
title:      "JVM——Java Virtual Machine"
date:       2019-6-3 17:03
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Java
---
>JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。
>Java语言的一个非常重要的特点就是与平台的无关性。而使用Java虚拟机是实现这一特点的关键。一般的高级语言如果要在不同的平台上运行，至少需要编译成不同的目标代码。而引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。这就是Java的能够“一次编译，到处运行”的原因。

![](/img/in-post/post-Android/Java/JVM_definition.png)

---
### JVM

![](/img/in-post/post-Android/Java/JVM_inDetail.png)

---
### Java 类加载器

注：static变量是静态变量，`当加载类时即已加载，非static变量实例对象时加载`。

#### 1.java.lang.ClassLoader类介绍
java.lang.ClassLoader类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个Java 类，即 java.lang.Class类的一个实例。

ClassLoader提供了一系列的方法，比较重要的方法如：

![](/img/in-post/post-Android/Java/ClassLoader.png)
 

 
#### 2.JVM中类加载器的树状层次结构
Java 中的类加载器大致可以分成两类，一类是`系统提供的`，另外一类则是由 Java 应用`开发人员编写`的。 

**引导类加载器（bootstrap class loader）：**

它用来加载 Java 的核心库(jre/lib/rt.jar)，是用原生C++代码来实现的，并不继承自java.lang.ClassLoader。

加载扩展类和应用程序类加载器，并指定他们的父类加载器，在java中获取不到。 

**扩展类加载器（extensions class loader）：**

它用来加载 Java 的扩展库(jre/ext/*.jar)。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。 

**系统类加载器（system class loader）：**

它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。

**自定义类加载器（custom class loader）：**

除了系统提供的类加载器以外，开发人员可以通过继承 java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求。

#### 3.双亲委派机制

某个特定的类加载器在接到加载类的请求时，首先将加载任务委托交给父类加载器，父类加载器又将加载任务向上委托，直到最父类加载器，如果最父类加载器可以完成类加载任务，就成功返回，如果不行就向下传递委托任务，由其子类加载器进行加载。

双亲委派机制的好处：

- `保证java核心库的安全性`（例如：如果用户自己写了一个java.lang.String类就会因为双亲委派机制不能被加载，不会破坏原生的String类的加载）

`代理模式`：与双亲委派机制相反，代理模式是先自己尝试加载，如果无法加载则向上传递。tomcat就是代理模式。


#### 4.自定义类加载器
```java
public class MyClassLoader extends ClassLoader{

    private String rootPath;
    
    public MyClassLoader(String rootPath){
        this.rootPath = rootPath;
    }
    
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        //check if the class have been loaded
        Class<?> c = findLoadedClass(name);        
        if(c!=null){
            return c;
        }
        //load the class
        byte[] classData = getClassData(name);
        if(classData==null){
            throw new ClassNotFoundException();
        }
        else{
            c = defineClass(name,classData, 0, classData.length);
            return c;
        }    
    }
    
    private byte[] getClassData(String className){
        String path = rootPath+"/"+className.replace('.', '/')+".class";
        
        InputStream is = null;
        ByteArrayOutputStream bos = null;
        try {
            is = new FileInputStream(path);
            bos = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int temp = 0;
            while((temp = is.read(buffer))!=-1){
                bos.write(buffer,0,temp);
            }
            return bos.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            try {
                is.close();
                bos.close();
            } catch (Exception e) {
                e.printStackTrace();
            }            
        }
        
        return null;        
    }    
}
```
#### 5.类加载过程详解
JVM将类加载过程分为三个步骤：`装载（Load）`，`链接（Link）`和`初始化(Initialize)`

![](/img/in-post/post-Android/Java/ClassLoader过程.png)

1. `装载：`
    - 查找并加载类的二进制数据；
2. `链接`：
    - 验证：确保被加载类信息符合JVM规范、没有安全方面的问题。
    - 准备：为类的静态变量分配内存，并将其初始化为默认值。
    - 解析：把虚拟机常量池中的符号引用转换为直接引用。
3. `初始化`：
    - 为类的静态变量赋予正确的初始值。

ps:解析部分需要说明一下，Java 中，虚拟机会为每个`加载的类维护一个常量池`【不同于字符串常量池，这个常量池只是该类的字面值（例如类名、方法名）和符号引用的有序集合。 而`字符串常量池，是整个JVM共享的`】这些符号（如int a = 5;中的a）就是符号引用，而解析过程就是把它转换成指向堆中的对象地址的相对地址。


**类的初始化步骤：**
1. 如果这个类还没有被加载和链接，那先进行加载和链接
2. 假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一次），那就初始化直接的父类（不适用于接口）
3. 如果类中存在static标识的块，那就依次执行这些初始化语句。
 


---
### Java 内存模型

在Java虚拟机规范中试图定义一种`Java内存模型（Java Memory Model，JMM）`来屏蔽各个硬件平台和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。那么Java内存模型规定了哪些东西呢，它定义了程序中变量的访问规则，往大一点说是定义了程序执行的次序。注意，为了获得较好的执行性能，Java内存模型并没有限制执行引擎使用处理器的寄存器或者高速缓存来提升指令执行速度，也没有限制编译器对指令进行重排序。也就是说，在java内存模型中，也会存在`缓存一致性问题和指令重排序`的问题。

Java内存模型规定所有的变量都是存在主存当中`（类似于前面说的物理内存），每个线程都有自己的工作内存（类似于前面的高速缓存）`。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。

举个简单的例子：在java中，执行下面这个语句：

```java
i  = 10;
```
执行线程必须先在自己的工作线程中对变量i所在的缓存行进行赋值操作，然后再写入主存当中。而不是直接将数值10写入主存当中。

那么Java语言 本身对 `原子性、可见性以及有序性`提供了哪些保证呢？

#### 原子性

在Java中，对基本数据类型的变量的`读取和赋值操作是原子性操作，`即这些操作是不可被中断的，要么执行，要么不执行。

上面一句话虽然看起来简单，但是理解起来并不是那么容易。看下面一个例子：

请分析以下哪些操作是原子性操作：
```java
x = 10;         //语句1
y = x;         //语句2
x++;           //语句3
x = x + 1;     //语句4
```

咋一看，有些朋友可能会说上面的4个语句中的操作都是原子性操作。其实只有语句1是原子性操作，其他三个语句都不是原子性操作。

语句1是直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中。

语句2实际上包含2个操作，它先要去读取x的值，再将x的值写入工作内存，虽然读取x的值以及 将x的值写入工作内存 这2个操作都是原子性操作，但是`合起来就不是原子性操作`了。

同样的，x++和 x = x+1包括3个操作：读取x的值，进行加1操作，写入新的值。

所以上面4个语句只有语句1的操作具备原子性。

也就是说，只有简单的`读取、赋值`（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。

不过这里有一点需要注意：在32位平台下，对64位数据的读取和赋值是需要通过两个操作来完成的，不能保证其原子性。但是好像在最新的JDK中，JVM已经保证对64位数据的读取和赋值也是原子性操作了。

从上面可以看出，Java内存模型只保证了基本读取和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过`synchronized和Lock`来实现。由于synchronized和Lock能够保证`任一时刻只有一个线程执行该代码块`，那么自然就不存在原子性问题了，从而保证了原子性。


#### 可见性

对于可见性，Java提供了`volatile关键字`来保证可见性。

当一个共享变量被volatile修饰时，它会保证`修改的值会立即被更新到主存`，当有其他线程需要读取时，它会去内存中读取新值。

而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，`什么时候被写入主存是不确定的`，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前`会将对变量的修改刷新到主存当中`。因此可以保证可见性。


#### 有序性

在Java内存模型中，允许编译器和处理器对指令进行重排序，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

在Java里面，可以通过volatile关键字来保证一定的“有序性”。另外可以通过`synchronized和Lock来保证有序性`，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。

另外，**Java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。**

下面就来具体介绍下happens-before原则（先行发生原则）：
- `程序次序规则`：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
- `锁定规则`：一个unLock操作先行发生于后面对同一个锁的lock操作
- `volatile变量规则`：对一个变量的`写操作先行发生于后面对这个变量的读操作`
- `传递规则`：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
- 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
- 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

对于程序次序规则来说，我的理解就是一段程序代码的执行在单个线程中看起来是有序的。注意，虽然这条规则中提到“书写在前面的操作先行发生于书写在后面的操作”，这个应该是程序看起来执行的顺序是按照代码顺序执行的，因为虚拟机可能会对程序代码进行指令重排序。虽然进行重排序，但是最终执行的结果是与程序顺序执行的结果一致的，`它只会对不存在数据依赖性的指令进行重排序`。因此，在单个线程中，程序执行看起来是有序执行的，这一点要注意理解。事实上，这个规则是用来保证`程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的正确性`。

第二条规则也比较容易理解，也就是说无论在单线程中还是多线程中，同一个锁如果出于被锁定的状态，那么必须先对`锁进行了释放操作，后面才能继续进行lock操作`。

第三条规则是一条比较重要的规则，也是后文将要重点讲述的内容。直观地解释就是，`如果一个线程先去写一个变量，然后一个线程去进行读取，那么写入操作肯定会先行发生于读操作`。

第四条规则实际上就是体现happens-before原则具备`传递性`。

---

### JVM内存模型 与 GC

#### Jvm内存浅析
虽然jvm帮我们做了内存管理的工作，但是我们仍需要了解jvm到底做了什么，下面我们就一起去看一看

jvm启动时进行一系列的工作，其中一项就是开辟一块运行时内存。而这一块内存中又分为了五大区域，分别用于不同的功能。

![](/img/in-post/post-Android/Java/JMM.png)

- `程序计数器`

    这是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的行号指示器，指的是上次代码被执行的地方，`线程私有`。

- `Java 虚拟机栈`

    它是 Java方法执行的内存模型，每一个方法被调用到执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程，`线程私有`。

- `本地方法栈`

    跟虚拟机栈类似，不过本地方法栈用于执行本地方法，`线程私有`。

- `Java 堆`

    该区域存在的唯一目的就是存放对象，几乎应用中所有的对象实例都在这里分配内存，`所有线程共享`。

- `方法区`

    它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据，`所有线程共享`。

线程私有的区域随着线程的结束就没有了，没有垃圾回收；`gc操作的地方是在所有线程共享的区域`。

#### 内存分代

堆内存是虚拟机管理的内存中最大的一块，也是`垃圾回收最频繁的一块区域`，我们程序`所有的对象实例`都存放在堆内存中。给堆内存分代是为了`提高对象内存分配和垃圾回收的效率`。试想一下，如果堆内存没有区域划分，所有的新创建的对象和生命周期很长的对象放在一起，随着程序的执行，堆内存需要频繁进行垃圾收集，而每次回收都要遍历所有的对象，遍历这些对象所花费的时间代价是巨大的，会严重影响我们的GC效率，这简直太可怕了。

有了内存分代，情况就不同了，新创建的对象会在新生代中分配内存，`经过多次回收仍然存活下来的对象存放在老年代中`，静态属性、类信息等存放在永久代中，新生代中的对象存活时间短，只需要在新生代区域中频繁进行GC，老年代中对象生命周期长，内存回收的频率相对较低，不需要频繁进行回收，永久代中回收效果太差，一般不进行垃圾回收，`还可以根据不同年代的特点采用合适的垃圾收集算法。分代收集大大提升了收集效率`，这些都是内存分代带来的好处。

#### 内存分代划分
Java虚拟机将堆内存划分为新生代、老年代和永久代，永久代是HotSpot虚拟机特有的概念，它采用永久代的方式来实现方法区，其他的虚拟机实现没有这一概念，而且HotSpot也有取消永久代的趋势，在JDK 1.7中HotSpot已经开始了“去永久化”，把原本放在永久代的字符串常量池移出。永久代主要存放常量、类信息、静态变量等数据，与垃圾回收关系不大，新生代和老年代是垃圾回收的主要区域。内存分代示意图如下：

![](/img/in-post/post-Android/Java/JMM&GC.png)

**新生代（Young）**

- 新生成的对象优先存放在新生代中，新生代对象朝生夕死，`存活率很低`，在新生代中，常规应用进行一次垃圾收集一般可以回`收70% ~ 95% 的空间，回收效率很高`。

- HotSpot将新生代划分为三块，一块较大的Eden空间和两块较小的Survivor空间，默认比例为8：1：1。划分的目的是因为HotSpot采用`复制算法`来回收新生代，设置这个比例是为了充分利用内存空间，减少浪费。新生成的对象在Eden区分配（大对象除外，大对象直接进入老年代），当Eden区没有足够的空间进行分配时，虚拟机将发起一次Minor GC。

- Minor GC开始时，对象只会存在于`Eden区和From Survivor区`，To Survivor区是空的（作为保留区域）。GC进行时，Eden区中所有存活的对象都会被复制到To Survivor区，而在From Survivor区中，仍存活的对象会根据它们的年龄值决定去向，年龄值达到年龄阀值（`默认为15，新生代中的对象每熬过一轮垃圾回收，年龄值就加1，GC分代年龄存储在对象的header中`）的对象会被移到老年代中，没有达到阀值的对象会被复制到To Survivor区。接着清空Eden区和From Survivor区，`新生代中存活的对象都在To Survivor`区。接着， From Survivor区和To Survivor区会交换它们的角色，也就是新的To Survivor区就是上次GC清空的From Survivor区，新的From Survivor区就是上次GC的To Survivor区，总之，不管怎样都会保证`To Survivor`区在一轮GC后是空的。GC时当To Survivor区**没有足够的空间存放上一次新生代收集下来的存活对象**时，需要依赖老年代进行分配担保，将这些对象存放在老年代中。

**老年代（Old）**

在新生代中经历了多次（具体看虚拟机配置的阀值）`GC后仍然存活下来的对象会进入老年代中`。老年代中的对象生命周期较长，存活率比较高，在老年代中进行GC的频率相对而言较低，而且回收的速度也比较慢。

**永久代（Permanent）**

永久代存储`类信息、常量、静态变量、即时编译器编译后的代码`等数据，对这一区域而言，Java虚拟机规范指出可以不进行垃圾收集，一般而言不会进行垃圾回收。

#### Minor GC 和 Full GC的区别

`新生代GC（Minor GC）`：Minor GC指发生在新生代的GC，因为新生代的Java对象大多都是朝生夕死，所以Minor GC非常频繁，一般回收速度也比较快。当Eden空间不足以 为对象分配内存时，会触发Minor GC。

`老年代GC（Full GC/Major GC）`：Full GC指发生在老年代的GC，出现了Full GC一般会伴随着至少一次的Minor GC（老年代的对象大部分是Minor GC过程中从新生代进入老年代），比如：分配担保失败。Full GC的速度一般会比Minor GC慢10倍以上。当`老年代内存不足或者显式调用System.gc()方法`时，会触发Full GC。

#### GC机制
>Java garbage collection is an automatic process to manage the runtime memory used by programs. By doing it automatic JVM relieves the programmer of the overhead of assigning and freeing up memory resources in a program.
java 与 C语言相比的一个优势是，可以通过自己的JVM自动分配和回收内存空间。

#### 检测算法：

1.`引用计数法`

给一个对象添加`引用计数器`，每当有个地方引用它，计数器就加1；引用失效就减1。

好了，问题来了，如果我有两个对象A和B，互相引用，除此之外，没有其他任何对象引用它们，实际上这两个对象已经无法访问，即是我们说的`垃圾对象`。但是`互相引用`，计数不为0，导致无法回收，所以还有另一种方法：

2.`可达性分析算法`

以根集对象(GC Root)为起始点进行搜索，如果有对象不可达的话，即是垃圾对象。这里的根集一般包括java栈中引用的对象、方法区常量池中引用的对象、本地方法中引用的对象等。

总之，JVM在做垃圾回收的时候，会检查堆中的所有对象是否会被这些根集对象引用，不能够被引用的对象就会被垃圾收集器回收。

#### 回收算法

#### 1.按照基本回收策略分

**标记-清除（Mark-Sweep）**

算法和名字一样，分为两个阶段：标记和清除。标记所有需要回收的对象，然后统一回收。这是最基础的算法，后续的收集算法都是基于这个算法扩展的。

`不足`：效率低；标记清除之后会产生大量碎片。

![](/img/in-post/post-Android/Java/Mark-Sweep.png)

**复制（Copying）**

此算法把内存空间划为两个相等的区域，每次只使用其中一个区域。垃圾回收时，遍历当前使用区域，把正在使用中的对象复制到另外一个区域中。此算法每次只处理正在使用中的对象，因此复制成本比较小，同时复制过去以后还能进行相应的内存整理，不会出现“碎片”问题。当然，`此算法的缺点也是很明显的，就是需要两倍内存空间`。

![](/img/in-post/post-Android/Java/Copying.png)

**标记-整理（Mark-Compact）**

此算法结合了`“标记-清除”`和`“复制”`两个算法的优点。也是分两阶段，`第一阶段从根节点开始标记所有被引用对象`，`第二阶段遍历整个堆，把清除未标记对象并且把存活对象“压缩”到堆的其中一块`，按顺序排放。此算法避免了“标记-清除”的碎片问题，同时也避免了“复制”算法的空间问题。效果图如下：

![](/img/in-post/post-Android/Java/Mark-Compact.png)



#### 2.按分区对待的方式分

`增量收集（Incremental Collecting）`:实时垃圾回收算法，即：在应用进行的同时进行垃圾回收。不知道什么原因JDK5.0中的收集器没有使用这种算法的。

`分代收集（Generational Collecting）`:基于对对象生命周期分析后得出的垃圾回收算法。把对象分为年青代、年老代、持久代，对不同生命周期的对象使用不同的算法（上述方式中的一个）进行回收。现在的垃圾回收器（从J2SE1.2开始）都是使用此算法的。

#### 3.按系统线程分（回收器类型）
1. `串行收集`:串行收集使用单线程处理所有垃圾回收工作，因为无需多线程交互，实现容易，而且效率比较高。但是，其局限性也比较明显，即无法使用多处理器的优势，所以此收集适合单处理器机器。当然，此收集器也可以用在小数据量（100M左右）情况下的多处理器机器上。默认使用串行收集器。

2. `并行收集`:并行收集使用多线程处理垃圾回收工作，因而速度快，效率高。而且理论上CPU数目越多，越能体现出并行收集器的优势。适合对吞吐量优先（科学技术，后台应用），无过多交互的应用。吞吐量=业务处理时间/（业务处理时间+垃圾回收时间）。

3. 并发收集:相对于串行收集和并行收集而言，`前面两个在进行垃圾回收工作时，需要暂停整个运行环境`，而只有垃圾回收程序在运行，因此，系统在垃圾回收时会有明显的暂停，而且暂停时间会因为堆越大而越长`。并发收集器不会暂停应用，适合响应时间优先的应用。保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等`。

---
### GC Root

GC管理的主要区域是Java堆，一般情况下只针对堆进行垃圾回收。方法区、JVM栈和Native栈不被GC所管理，因而选择这些`非堆区的对象作为GC roots`，被GC roots引用的对象不被GC回收。

一个对象可以属于多个root，GC root有几下种：
- **Class** - 由系统类加载器(system class loader)加载的对象，这些类不可以被回收，他们可以以静态字段的方式持有其它对象。我们需要注意的一点就是，通过用户自定义的类加载器加载的类，除非相应的java.lang.Class实例以其它的某种（或多种）方式成为roots，否则它们并不是roots，.
- **Thread** - 活着的线程
- **Stack Local** - Java方法的local变量或参数（存在于所有Java线程当前活跃的栈帧里，它们会指向堆里的对象）
- 【Java类的运行时常量池里的引用类型常量（String或Class类型）】（先不考虑）
- 【String常量池（StringTable）里的引用】（先不考虑）
- **JNI Local** - JNI方法的local变量或参数
- **JNI Global** - 全局JNI引用
- **Monitor Used** - Monitor被持有，用于同步互斥的对象
- **Held by JVM** - 用于JVM特殊目的由GC保留的对象，但实际上这个与JVM的实现是有关的。可能已知的一些类型是：**系统类加载器**、一些JVM知道的**重要的异常类**、一些用于**处理异常的预分配对象**以及一些**自定义的类加载器**等。JVM的一些静态数据成员会指向堆里的对象

GC收集那些`不是GC roots且没有被GC roots引用的对象`。
