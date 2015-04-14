title: "java内存模型"
date: 2015-04-06 10:55:35
categories: [java]
tags: jvm
---

###Java虚拟机
Java虚拟机（Java Virtual Machine 简称JVM）是运行所有Java程序的抽象计算机，是Java语言的运行环境，它是Java 最具吸引力的特性之一。
Java虚拟机有自己完善的硬体架构，如处理器、堆栈、寄存器等，还具有相应的指令系统。
JVM屏蔽了与具体操作系统平台相关的信息，使得Java程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。

一个运行时的Java虚拟机实例的天职是：负责运行一个java程序。当启动一个Java程序时，一个虚拟机实例也就诞生了。当该程序关闭退出，这个虚拟机实例也就随之消亡。如果同一台计算机上同时运行三个Java程序，将得到三个Java虚拟机实例。每个Java程序都运行于它自己的Java虚拟机实例中。

---

###JVM的体系结构
如下图所示，JVM的体系结构包含几个主要的子系统和内存区：
![](/img/jvm_memory.jpg)

1. 垃圾回收器（Garbage Collection）：负责回收堆内存（Heap）中没有被使用的对象，即这些对象已经没有被引用了。
2. 类装载子系统（Classloader Sub-System）：除了要定位和导入二进制class文件外，还必须负责验证被导入类的正确性，为类变量分配并初始化内存，以及帮助解析符号引用。
3. 执行引擎（Execution Engine）：负责执行那些包含在被装载类的方法中的指令。
4. 运行时数据区（Java Memory Allocation Area）：<font color ="red">又叫虚拟机内存或者Java内存，虚拟机运行时需要从整个计算机内存划分一块内存区域存储许多东西</font>。例如：字节码、从已装载的class文件中得到的其他信息、程序创建的对象、传递给方法的参数，返回值、局部变量等等。


---
###java内存分区
运行时数据区即是java内存，而且数据区要存储的东西比较多，如果不对这块内存区域进行划分管理，会显得比较杂乱无章。程序喜欢有规律的东西，最讨厌杂乱无章的东西。 
根据存储数据的不同，：<font color ="red">java内存通常被划分为5个区域：程序计数器（Program Count Register）、本地方法栈（Native Stack）、方法区（Methon Area）、栈（Stack）、堆（Heap）</font>。

1. 程序计数器（Program Count Register）：又叫程序寄存器。JVM支持多个线程同时运行，当每一个新线程被创建时，它都将得到它自己的PC寄存器（程序计数器）。如果线程正在执行的是一个Java方法（非native），那么PC寄存器的值将总是指向下一条将被执行的指令，如果方法是 native的，程序计数器寄存器的值不会被定义。 JVM的程序计数器寄存器的宽度足够保证可以持有一个返回地址或者native的指针。
2. 栈（Stack）：又叫堆栈。JVM为每个新创建的线程都分配一个栈。<font color = "red">也就是说,对于一个Java程序来说，它的运行就是通过对栈的操作来完成的</font>.**栈以帧为单位保存线程的状态**。`JVM对栈只进行两种操作：以帧为单位的压栈和出栈操作`。我们知道,某个线程正在执行的方法称为此线程的当前方法。我们可能不知道，当前方法使用的帧称为当前帧。当线程激活一个Java方法，JVM就会在线程的 Java堆栈里新压入一个帧，这个帧自然成为了当前帧。在此方法执行期间，这个帧将用来保存参数、局部变量、中间计算过程和其他数据。从Java的这种分配机制来看,堆栈又可以这样理解：栈(Stack)是操作系统在建立某个进程时或者线程(在支持多线程的操作系统中是线程)为这个线程建立的存储区域，该区域具有先进后出的特性。其相关设置参数：
>-Xss –设置方法栈的最大值
3. 本地方法栈（Native Stack）：存储本地方方法的调用状态。
![](/img/runtime_data_memory.png)
4. 方法区（Method Area）：当虚拟机装载一个class文件时，它会从这个class文件包含的二进制数据中解析类型信息，然后把这些类型信息（包括类信息、常量、静态变量等）放到方法区中，该内存区域被所有线程共享，如下图所示。<font color = "red">本地方法区存在一块特殊的内存区域，叫常量池（Constant Pool）</font>
5. 常量池
常量池指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。除了包含代码中所定义的各种基本数据类型和对象型（String及数组）的常量值（final）还包含一些以文本形式出现的符号引用，比如：
类和接口的全限定名
字段的名称和描述符
方法和名称和描述符
虚拟机必须在每个被装载的类型维护一个常量池。常量池就是该类型所用到常量的一个有序集合，包括直接常量（string、integer等）和其他类型，字段和方法的符号引用
对于String常量，它的值是在常量池中。而JVM中的常量池在内存当中是以表的形式存在的，对于String类型，有一张固定长度的CONSTANT_String_info表用来存储文件字符串值，注意：该表只存储文字字符串值，不存储符号引用。说到这里，对常量池中的字符串值的存储位置应该有一个比较明了的理解了
在程序执行的时候，常量池会存在Method Area，而不是堆中
![](/img/method_areo.png)
6. 堆（Heap）：Java堆（Java Heap）是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域。在此区域的唯一目的就是存放对象实例，几乎所有的对象实例都是在这里分配内存，但是这个对象的引用却是在栈（Stack）中分配。因此，执行`String s = new String(“s”)`时，需要从两个地方分配内存：在堆中为String对象分配内存，在栈中为引用（这个堆对象的内存地址，即指针）分配内存，如下图所示。
![](/img/heap.png)


JAVA虚拟机有一条在堆中分配新对象的指令，却没有释放内存的指令，正如你无法用Java代码区明确释放一个对象一样。虚拟机自己负责决定如何以及何时释放不再被运行的程序引用的对象所占据的内存，通常，虚拟机把这个任务交给垃圾收集器（Garbage Collection）。其相关设置参数：
>-Xms — 设置堆内存初始大小
-Xmx — 设置堆内存最大值
-XX:MaxTenuringThreshold — 设置对象在新生代中存活的次数
-XX:PretenureSizeThreshold — 设置超过指定大小的大对象直接分配在旧生代中

---
###GC回收
Java堆是垃圾收集器管理的主要区域，因此又称为“GC 堆”（Garbage Collectioned Heap）。
现在的垃圾收集器基本都是采用的`分代收集`算法，所以Java堆还可以细分为：新生代（Young Generation）和老年代（Old Generation），如下图所示。
分代收集算法的思想：
* 第一种说法，用较高的频率对年轻的对象(young generation)进行扫描和回收，这种叫做minor collection，而对老对象(old generation)的检查回收频率要低很多，称为major collection。这样就不需要每次GC都将内存中所有对象都检查一遍，以便让出更多的系统资源供应用系统使用；
* 另一种说法，在分配对象遇到内存不足时，先对新生代进行GC（Young GC）；当新生代GC之后仍无法满足内存空间分配需求时， 才会对整个堆空间以及方法区进行GC（Full GC）。
![](/img/gc.jpg)


在这里可能会有读者表示疑问：记得还有一个什么永久代（Permanent Generation）的啊，难道它不属于Java堆？亲，你答对了！其实传说中的永久代就是上面所说的方法区，存放的都是jvm初始化时加载器加载的一些类型信息（包括类信息、常量、静态变量等），这些信息的生存周期比较长，GC不会在主程序运行期对PermGen Space进行清理，所以如果你的应用中有很多CLASS的话,就很可能出现PermGen Space错误。
>其相关设置参数：
-XX:PermSize –设置Perm区的初始大小
-XX:MaxPermSize –设置Perm区的最大值

新生代（Young Generation）又分为：Eden区和Survivor区，Survivor区有分为From Space和To Space。Eden区是对象最初分配到的地方；默认情况下，From Space和To Space的区域大小相等。
JVM进行Minor GC时，将Eden中还存活的对象拷贝到Survivor区中，还会将Survivor区中还存活的对象拷贝到Tenured区中。在这种GC模式下，JVM为了提升GC效率， 将Survivor区分为From Space和To Space，这样就可以将对象回收和对象晋升分离开来。新生代的大小设置有2个
>相关参数：
-Xmn — 设置新生代内存大小。
-XX:SurvivorRatio — 设置Eden与Survivor空间的大小比例

老年代（Old Generation）： 当 OLD 区空间不够时， JVM 会在 OLD 区进行 major collection；完全垃圾收集后，若Survivor及OLD区仍然无法存放从Eden复制过来的部分对象，导致JVM无法在Eden区为新对象创建内存区域，则出现”Out of memory错误”  。

##参考资料
[http://www.importnew.com/15671.html](http://www.importnew.com/15671.html?from=timeline&isappinstalled=1&nsukey=%2FF7sCOoegm7KjMmTGq2IlsGnrW%2F7YayNB0SWTB43sWRxMuo1A0J8mmzqI4nbPVNyZi6J%2B6xIuwOA%2FCtmoxQ0dQ%3D%3D)
##感谢
之前对这块有一定的理解，但是很模糊，看了参考的博文之后感觉收货相当多多呢，谢谢!