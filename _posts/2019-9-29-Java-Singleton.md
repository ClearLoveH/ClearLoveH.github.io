---
layout:     post
title:      "Java 单例与Double-Checked Locking"
date:       2019-9-29 14:27
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Java
---

>参考博客：[设计模式（二）单例模式的七种写法](https://blog.csdn.net/itachi85/article/details/50510124)

### 单例模式的几种实现

#### 1.饿汉模式
```java
public class Singleton {  
     private static Singleton instance = new Singleton();  
     private Singleton (){
     }
     public static Singleton getInstance() {  
        return instance;  
     }  
 }  
```
- 这种方式在**类加载时就完成了初始化**，所以叫“饿汉”模式，这样也导致类加载较慢，但获取对象的速度快。 
- 这种方式基于类加载机制避免了多线程的同步问题，但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化instance显然没有达到懒加载的效果。

#### 2.懒汉模式（线程不安全）
```java
public class Singleton {  
      private static Singleton instance;  
      private Singleton (){
      }   
      public static Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
      }  
 } 
```
- 懒汉模式申明了一个静态对象，在用户第一次调用时初始化，虽然节约了资源，但第一次加载时需要实例化，反映稍慢一些，而且**在多线程不能正常工作**。

#### 3.懒汉模式（线程安全）
```java
public class Singleton {  
      private static Singleton instance;  
      private Singleton (){
      }
      public static synchronized Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
      }  
 }  
```
- 由于该方法是synchronized 的，需要为该方法的每一次调用付出同步的代价，即使只有第一次调用需要同步。
- 这种写法能够在多线程中很好的工作，但是**每次调用getInstance方法时都需要进行同步**，**造成不必要的同步开销**，所以不建议用这种模式。

#### 4.双重检查模式 （Double-checked locking —— DCL）
```java
public class Singleton {  
    private volatile static Singleton instance;  
    private Singleton (){
    }   
    public static Singleton getInstance() {  
        if (instance== null) {  
            synchronized (Singleton.class) {  
                if (instance== null) {  
                    instance= new Singleton();  
                }  
            }  
        }  
        return singleton;  
    }  
 }  
```
- 这种写法在getSingleton方法中对singleton进行了**两次判空，第一次是为了不必要的同步，第二次是在singleton等于null的情况下才创建实例**。在这里用到了`volatile`关键字，不了解volatile关键字的可以查看[Java多线程（三）volatile域](http://blog.csdn.net/itachi85/article/details/50274169)这篇文章，在这篇文章作者也提到了双重检查模式是正确使用volatile关键字的场景之一。
- 在这里使用`volatile会或多或少的影响性能，但考虑到程序的正确性`，牺牲这点性能还是值得的。 DCL优点是资源利用率高，第一次执行getInstance时单例对象才被实例化，效率高。缺点是第一次加载时反应稍慢一些，在高并发环境下也有一定的缺陷，虽然发生的概率很小。DCL虽然在一定程度解决了资源的消耗和多余的同步，线程安全等问题，但是**他还是在某些情况会出现失效的问题**，也就是**DCL失效**，后面会单独提及DCL失效问题，在《java并发编程实践》一书建议用**静态内部类单例模式**来替代DCL。

#### 5.静态内部类单例模式（推荐！）
```java
public class Singleton { 
    private Singleton(){
    }
      public static Singleton getInstance(){  
        return SingletonHolder.sInstance;  
    }  
    private static class SingletonHolder {  
        private static final Singleton sInstance = new Singleton();  
    }  
} 
```
- 第一次加载Singleton类时并不会初始化sInstance，只有第一次调用getInstance方法时**虚拟机加载SingletonHolder 并初始化sInstance** ，这样不仅能成功的实现 `Lazy initialization`，也可以**确保线程安全也能保证Singleton类的唯一性**，所以推荐使用这种**静态内部类单例模式**。

#### 6.枚举单例*
```java
public enum Singleton {  
     INSTANCE;  
     public void doSomeThing() {  
     }  
 }  
```
- **默认枚举实例的创建是线程安全的，并且在任何情况下都是单例**，上述讲的几种单例模式实现中，有一种情况下他们会重新创建对象，**那就是反序列化**，将一个单例实例对象写到磁盘再读回来，从而获得了一个实例。反序列化操作提供了readResolve方法，这个方法可以让开发人员控制对象的反序列化。在上述的几个方法示例中如果要杜绝单例对象被反序列化是重新生成对象，就必须加入如下方法：
```java
private Object readResolve() throws ObjectStreamException{
    return singleton;
}
```
- 枚举单例的优点就是简单，但是大部分应用开发很少用枚举，**可读性并不是很高**，不建议用。
- 关于枚举类型，可以看这篇博客：[Java枚举enum以及应用：枚举实现单例模式](https://www.cnblogs.com/cielosun/p/6596475.html)

#### 7.使用容器实现单例模式
```java
public class SingletonManager { 
　　private static Map<String, Object> objMap = new HashMap<String,Object>();
　　private Singleton() { 
　　}
　　public static void registerService(String key, Objectinstance) {
　　　　if (!objMap.containsKey(key) ) {
　　　　　　objMap.put(key, instance) ;
　　　　}
　　}
　　public static ObjectgetService(String key) {
　　　　return objMap.get(key) ;
　　}
}
```
- 用SingletonManager 将多种的单例类统一管理，**在使用时根据key获取对象对应类型的对象**。这种方式使得我们可以**管理多种类型的单例，并且在使用时可以通过统一的接口进行获取操作**，降低了用户的使用成本，也对用户隐藏了具体实现，降低了耦合度。

---
### 双重检查锁（Double-checked locking）

单例创建模式是一个通用的编程习语。和多线程一起使用时，必需使用某种类型的同步。在努力创建更有效的代码时，Java 程序员们创建了`双重检查锁定习语，将其和单例创建模式一起使用，从而限制同步代码量`。然而，**由于一些不太常见的 Java 内存模型细节的原因，并不能保证这个双重检查锁定习语有效**。

它**偶尔会失败，而不是总失败**。此外，它失败的原因并不明显，还包含 Java 内存模型的一些隐秘细节。这些事实将导致代码失败，原因是双重检查锁定难于跟踪。我们将详细介绍双重检查锁定，从而理解它在何处失效。

我们在前面介绍通过DCL来实现单例，也提到 DCL 在Java中即使配合使用 `volatile` 关键字也会导致DCL失效的情况出现，那么为什么会这样呢？现在就来分析一下：

#### 1. 单例创建
```java
class Singleton {
  private static Singleton instance;

  private Singleton() {
  }

  public static Singleton getInstance() {
    if (instance == null)          //1
      instance = new Singleton();  //2
    return instance;               //3
  }
}
```
- 此类的设计确保只创建一个 Singleton 对象。构造函数被声明为 private，getInstance() 方法只创建一个对象。这个实现**适合于单线程程序**。然而，当引入多线程时，就必须通过同步来保护 getInstance() 方法。**如果不保护 getInstance() 方法，则可能返回Singleton 对象的两个不同的实例**。假设两个线程并发调用 getInstance() 方法并且按以下顺序执行调用：
    - 线程 1 调用 getInstance() 方法并决定 instance 在 //1 处为 null。 
    - 线程 1 进入 if 代码块，但在执行 //2 处的代码行时**被线程 2 预占**。 
    - 线程 2 调用 getInstance() 方法并在 //1 处决定 instance 为 null。 
    - 线程 2 进入 if 代码块并创建一个新的 Singleton 对象并在 //2 处将变量 instance 分配给这个新对象。 
    - 线程 2 在 //3 处返回 Singleton 对象引用。
    - 线程 2 **被线程 1 预占**。 
    - 线程 1 在它停止的地方启动，并执行 //2 代码行，这导致创建另一个 Singleton 对象。 
    - 线程 1 在 //3 处返回这个对象。
- 结果是 getInstance() 方法创建了两个 Singleton 对象，而它本该只创建一个对象。通过同步 getInstance() 方法从而在同一时间只允许一个线程执行代码，这个问题得以改正，如part 2 所示：

#### 2. 线程安全的 getInstance() 方法
```java
public static synchronized Singleton getInstance() {
  if (instance == null)          //1
    instance = new Singleton();  //2
  return instance;               //3
}
```
- 这部分就和前面提到过的一样，通过同步getInstance()方法的确可以解决问题，但是每次调用都会导致同步带来的开销。
- 由于该方法是 synchronized 的，需要**为该方法的每一次调用付出同步的代价，即使只有第一次调用需要同步**。


#### 3.synchronized 移入 getInstance() 内部
```java
public static Singleton getInstance() {
  if (instance == null) {
    synchronized(Singleton.class) {
      instance = new Singleton();
    }
  }
  return instance;
}
```
- 没有使用DCL，这部分代码暴露出的是和第一part类似的问题，只不过这里会导致多个 instance 的创建，**当 instance 为 null 时，两个线程可以并发地进入 if 语句内部**。然后，**一个线程进入 synchronized 块来初始化 instance，而另一个线程则被阻断**。当第一个线程退出 synchronized 块，等待着的线程**进入并创建**另一个 Singleton 对象。
- 注意：**当第二个线程进入 synchronized 块时，它并没有检查 instance 是否非 null。**

#### 4.DCL
```java
public static Singleton getInstance() {
  if (instance == null) {
    synchronized(Singleton.class) {  //1
      if (instance == null)          //2
        instance = new Singleton();  //3
    }
  }
  return instance;
}
```
- 双重检查锁定背后的理论是：**在 //2 处的第二次检查使（如清单 3 中那样）创建两个不同的 Singleton 对象成为不可能**。假设有下列事件序列：
    1. 线程 1 进入 getInstance() 方法。 
    2. 由于 instance 为 null，线程 1 在 //1 处进入 synchronized 块。 
    3. 线程 1 **被线程 2 预占。**
    4. 线程 2 进入 getInstance() 方法。
    5. 由于 instance 仍旧为 null，**线程 2 试图获取 //1 处的锁。然而，由于线程 1 持有该锁，线程 2 在 //1 处阻塞。**
    6. 线程 2 被线程 1 预占。
    7. 线程 1 执行，由于在 //2 处实例仍旧为 null，线程 1 还创建一个 Singleton 对象并将其引用赋值给 instance。
    8. 线程 1 退出 synchronized 块并从 getInstance() 方法返回实例。 
    9. 线程 1 被线程 2 预占。
    10. 线程 2 获取 //1 处的锁并检查 instance 是否为 null。 
    11. 由于 instance 是非 null 的，并没有创建第二个 Singleton 对象，由线程 1 创建的对象被返回。

- 双重检查锁定背后的理论是完美的。**不幸地是，现实完全不同**。双重检查锁定的问题是：**并不能保证它会在单处理器或多处理器计算机上顺利运行**。
- 双重检查锁定失败的问题并不归咎于 JVM 中的实现 bug，而是归咎于 **Java 平台内存模型。内存模型允许所谓的“无序写入”，这也是这些习语失败的一个主要原因**。

#### 5.无序写入（指令重排）
为解释该问题，需要重新考察上述part 4 中代码的 //3 行。此行代码创建了一个 Singleton 对象并初始化变量 instance 来引用此对象。这行代码的问题是：**在 Singleton 构造函数体执行之前，变量 instance 可能成为非 null 的**。

什么？这一说法可能让您始料未及，**但事实确实如此**。在解释这个现象如何发生前，请先暂时接受这一事实，我们先来考察一下双重检查锁定是如何被破坏的。假设part 4 中代码执行以下事件序列：
1. 线程 1 进入 getInstance() 方法。
2. 由于 instance 为 null，线程 1 在 //1 处进入 synchronized 块。 
3. 线程 1 前进到 //3 处，但在构造函数执行之前，使实例成为非 null。 
4. 线程 1 被线程 2 预占。
5. 线程 2 检查实例是否为 null。因为实例不为 null，线程 2 将 instance 引用返回给一个**构造完整但部分初始化**了的 Singleton对象。 
6. 线程 2 被线程 1 预占。
7. 线程 1 通过运行 Singleton 对象的构造函数并将引用返回给它，来完成对该对象的初始化。

此事件序列发生在线**程 2 返回一个尚未执行构造函数的对象**的时候。

为展示此事件的发生情况，假设为代码行 `instance =new Singleton();` 执行了下列伪代码：
```java
mem = allocate();             //Allocate memory for Singleton object.
instance = mem;               //Note that instance is now non-null, but has not been initialized.
ctorSingleton(instance);      //Invoke constructor for Singleton passing instance.
```
这段伪代码不仅是可能的，而且是在一些 JIT 编译器上真实发生的。**执行的顺序是颠倒的，但鉴于当前的内存模型，这也是允许发生的**。JIT 编译器的这一行为*使双重检查锁定的问题只不过是一次学术实践而已*。

#### 6.指令重排导致DCL失效：
- 双重检测的问题是因为`instruction reorder`的关系导致在 //3 时: instance=new Singleton (); 这句假设分为三步 
    1. 先申请内存 
    2. 构造Singleton 
    3. 将instance指向新的内存区域 
- 如果不进行指令重排，这个是没问题的。如果指令重排后执行顺序是，1 3 2。这就导致执行3后，instance已经非null，此时若恰好有别的线程重新访问get_instance函数，将得到instance非null的结果,并此时返回一个**还没执行完构造函数的instance实例**，既DCL失效关键在于**getInstance 获取到的实例可能是没有初始化的**。


#### 7.使用了 volatile 的顺序一致性来解决DCL失效
```java
class test {
  private volatile boolean stop = false;
  private volatile int num = 0;

  public void foo() {
    num = 100;    //This can happen second
    stop = true;  //This can happen first
    //...
  }

  public void bar() {
    if (stop)
      num += num;  //num can == 0!
  }
  //...
}
```
- 根据 JLS，`由于 stop 和 num 被声明为 volatile，它们应该顺序一致。这意味着如果 stop 曾经是 true，num 一定曾被设置成 100`。尽管如此，**因为许多 JVM 没有实现 volatile 的顺序一致性功能，您就不能依赖此行为**。因此，如果线程 1 调用 foo 并且线程 2 并发地调用 bar，则线程 1 可能在 num 被设置成为 100 之前将 stop 设置成 true。这将导致线程见到 stop 是 true，而 num 仍被设置成 0。使用 volatile 和 64 位变量的原子数还有另外一些问题，但这已超出了本文的讨论范围。有关此主题的更多信息，请参阅相关资料。
- **在JDK1.5之后，volatile 关键字便可以正确的用来禁止指令重排序**。

---
### 总结
1. 懒汉式单例没有同步机制，**在多线程环境下实例可能被重复创建**；而双重检测锁的问题则是**getInstance 获取到的实例可能未被初始化**。
2. java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统介入，**需要在用户态与核心态之间切换，这种切换会消耗大量的系统资源**，因为用户态与内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值、变量等，以便内核态调用结束后切换回用户态继续工作
3. DCL相对于饿汉式的好处：Synchronized在方法上是一个重量级锁操作(建议看一下锁升级步骤)，是需要切换上下文的，很耗时，如果已经创建了一个对象，但是每次调用都需要重新加锁解锁切换上下文才能拿到_instance不合理，但是双重加锁就不一样儿了，一旦创建对象以后就不会进行锁操作了，效率提升很多!
