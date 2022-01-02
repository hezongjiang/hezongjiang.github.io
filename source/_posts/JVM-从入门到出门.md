---
title: JVM 从入门到出门
catalog: true
date: 2019-10-16 22:49:19
subtitle:
header-img: jvm.jpeg
tags: 服务端开发
---

## 一、JVM 是什么

Java 是一种跨平台的语言，这是大家都知道常识。但是 Java 源文件是不能直接运行的，而是需要将 Java 源文件编译成一种“中间码”——字节码，但是字节码也是不能直接运行，字节码是需要在 Java 虚拟机（Java Virtual Machine，简称 JVM）上运行。同时由于每个系统平台都有自己的 JVM，所以 Java 语言编译后能通过 JVM 在不同平台上运行，从而实现了跨平台。JVM 的功能就解释执行字节码。

![Java 跨平台](https://upload-images.jianshu.io/upload_images/2708793-02bedd48acd972b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

总结起来就是，Java 的跨平台性并不是 Java 文件能跨平台运行，而是编译后的字节码文件能由不同平台上的 JVM 转化成平台上的机器指令。所以 Java 的跨平台性实质上是字节码的跨平台性。

## 二、JVM 如何加载 Class 文件
上面说了 Java 文件编译成字节码文件，然后将字节码文件交给 JVM 加载，那 JVM 如何加载呢？

![类的生命周期](https://upload-images.jianshu.io/upload_images/2708793-241c4a5390da6cf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

加载的过程包括了加载、验证、准备、解析、初始化五个阶段。在这五个阶段中，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持 Java 语言的运行时绑定。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

上述只是概念性的过程，对应到 JVM 的具体的执行情况如下图：

![JVM 加载 Class 文件](https://upload-images.jianshu.io/upload_images/2708793-7309a7648a4e83dc.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中：
* Class Loader：依据特定格式，加载 Class 文件到内存。
* Runtime Data Area：JVM 内存空间结构模型。
* Execution Engine：对命令进行解析。
* Native Interface：融合不同开发语言的原生库为 Java 所用。

小结一下类从编译到执行的过程：
1. 编译器将 Java 源文件编译为 Class 字节码文件；
2. ClassLoader 将字节码转化为 JVM 中的对象；
3. JVM 根据字节码初始化对象。

接下来将结合上图，具体谈谈 JVM 加载 Class 过程中很重要的两部分：ClassLoader 和 Runtime Data Area。

## 三、ClassLoader 与 双亲委派模型
ClassLoader 在 Java 中有着非常重要的作用，它主要工作在 Class 的加载阶段，其主要作用是从系统外部获取 Class 二进制数据流。它是 Java 的核心组件，**所有的 Class 都是由 ClassLoader 进行加载的**，ClassLoader 负责通过将 Class 文件里的数据流装载进系统，然后交给 Java 虚拟机进行链接、初始化等操作。

那么 ClassLoader 的加载流程是怎样的呢？这就涉及双亲委派模型了。

双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。

![双亲委派模型](https://upload-images.jianshu.io/upload_images/2708793-b7a59403bb7af8d3.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如上图，双亲委派模型的工作过程是：

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成。

每一个层次的类加载器都是如此。因此，所有的加载请求最终都应该传送到顶层的启动类加载器中。

只有当父加载器反馈自己无法完成这个加载请求时（搜索范围中没有找到所需的类），子加载器才会尝试自己去加载。


![ClassLoader 加载过程](https://upload-images.jianshu.io/upload_images/2708793-814b4448f8c8b57c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**为什么需要双亲委托机制？**

采用双亲委派模式的是好处是 Java 类随着它的类加载器一起具备了带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父类已经加载了该类时，就没必要子 ClassLoader 再加载一次。

其次是考虑到安全因素，Java 核心 API 中定义类型不会被随意替换，假设我自定义一个名为 java.lang.Integer 的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心 Java API 发现这个名字的类，发现该类已被加载，并不会重新加载自定义的 java.lang.Integer，而直接返回系统已加载过的 Integer.class，这样便可以防止核心 API 库被随意篡改。

## 四、Runtime Data Area

![Runtime Data Area](https://upload-images.jianshu.io/upload_images/2708793-ba17a5d1be5d1145.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

线程共享：Heap、Metaspace。
线程私有：本地方法栈、程序计数器、虚拟机栈。

**程序计数器**

程序计数器（Program Counter Register）是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。

由于 Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器内核都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，称这类内存区域为“线程私有”的内存。

**Java 虚拟机栈**

与程序计数器一样，Java 虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。

虚拟机栈描述的是 Java 方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧（Stack Frame，是方法运行时的基础数据结构）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

**本地方法栈**

本地方法栈（Native Method Stack）与虚拟机栈所发挥的作用是非常相似的，它们之间的区别是虚拟机栈为虚拟机执行 Java 方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。

**Java堆**

对于大多数应用来说，Java 堆（Java Heap）是 Java 虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。同时这里也是垃圾回收的核心区域。

**方法区**

方法区（Method Area）与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

**JVM 三大性能调优参数调优参数：**
* -Xss：规定了每个线程堆栈的大小。一般情况下256K是足够了。影响此进程中并发线程数大小；
* -Xms：设置堆的初始分配大小，默认为物理内存的 1/64；
* -Xmx：堆的最大分配内存，默认为物理内存的 1/4；

**堆与栈的区别：**
* 管理方式：栈由系统自动释放，堆需要 GC 管理；
* 空间大小：栈比堆小；
* 碎片相关：栈产生的碎片远少于堆；
* 分配方式：栈支持静态和动态分配，而堆仅支持动态分配；
* 效率：栈的效率比堆高。

## 五、总结
本篇文章先从 Java 的跨平台性说起，提到了 Java 源文件编译成字节码文件，字节码文件通过 JVM 运行在各个平台上，然后通过了解类的生命周期，解释了 JVM 是如何加载字节码文件，接着介绍了在加载的过程，需要深入了解的 ClassLoader 和 Runtime Data Area。

这些也都是 JVM 的基础。