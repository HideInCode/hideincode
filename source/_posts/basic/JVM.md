---
title: JVM
date: 2017-05-01
categories:
- 基础知识
tags:
- JVM
---



# JVM

## 组成部分

1. 类加载器

   > 把代码转化成字节码

2. 运行时数据区

   > 字节码加载到内存

3. 执行引擎

   > 把字节码翻译成底层指令，交给CPU。

4. 本地库接口

   > 调用其他语言本地库接口

	## 运行时数据区组成

1. 线程共享区

   1. 方法区

      > 存储虚拟机加载的类信息、常量、静态变量、编译后的代码。
      >
      > 别名：非堆。
      >
      > 不需要连续内存。
      >
      > 可以不实现垃圾收集。
      >
      > 会有OOM
      >
      > 分为运行时常量池、直接内存

   2. 堆

      > 存放对象实例，为对象实例分配内存。
      >
      > 垃圾收集器管理主要区域，垃圾堆。
      >
      > 内存空间上逻辑连续即可，会有OOM.

2. 线程私有区

   1. 栈

      > 方法执行时的内存模型：方法执行时会创建栈帧，用于变量、操作数、动态链接、方法出口等。方法的调用到完成意味着栈帧在栈中入栈和出栈的过程。
      >
      > 局部变量表的单位时槽（slot）,在编译器分配完成。
      >
      > 会有OOM，SOE两种异常。

   2. 本地方法栈

      > 区别于栈：为虚拟机用到的本地方法执行服务。

   3. 程序计数器

      > 字节码的行号指示器，分支、循环跳转、异常处理、线程恢复都依赖于此。唯一一处没有OOM的区域。

## 对象大小计算

1. 空Object是8byte
2. Object obj = new Object();会占据4 + 8个字节，栈中保存引用也要4byte空间。
3. 对对象内存分配时都是以 8 的整数倍来分
4. boolean 1byte

## 对象的定位方式

1. 句柄

   > 引用中存储句柄地址，对象被移动时只会改变句柄中的实例数据指针，引用不用改变。

2. 指针

   > 速度快，存的是对象的地址。

## 垃圾回收思路

1. 引用计数

   > 每个对象实例都有个引用计数器；
   >
   > 给对象分配变量时，就将计数器+1；
   >
   > 同理，引用超过生命周期或者变量被赋新值时，计数器-1；
   >
   > 计数器为0时实例会被垃圾收集；
   >
   > 优点：快
   >
   > 缺点：无法检测循环引用

2. 可达性分析

   > 1. 当成有向图处理
   > 2. 从GC Roots对象除法，包括
   >    1. 虚拟机栈中引⽤的对象（栈帧中的本地变量表）； 
   >    2. ⽅法区中类静态属性引⽤的对象；  
   >    3. ⽅法区中常量引⽤的对象；  
   >    4. 本地⽅法栈中 JNI（Native⽅法）引⽤的对象

## 几个内存泄漏的场景

1. 静态集合类引起的内存泄漏；
2.  当集合⾥⾯的对象属性被修改后，再调⽤ remove() ⽅法时不起作⽤； 
3. 监听器：释放对象的时候没有删除监听器； 
4. 各种连接：⽐如数据库连接（dataSourse.getConnection()），⽹络连接(socket) 和 IO 连接，除⾮其显式的 调⽤了其 close() ⽅法将其连接关闭，否则是不会⾃动被 GC 回收的；
5. 内部类：内部类的引⽤是⽐较容易遗忘的⼀种，⽽且⼀旦没释放可能导致⼀系列的后继类对象没有释放； 
6.  单例模式：单例对象在初始化后将在 JVM 的整个⽣命周期中存在（以静态变量的⽅式），如果单例对象持有 外部的引⽤，那么这个对象将不能被 JVM 正常回收，导致内存泄漏

## 尽量避免内存泄漏

1. 尽量不要使用static成员变量，减少生命周期。
2. 即使关闭资源
3. 不用的对象考虑手动设置为null

## 常见垃圾回收算法

1. 标记清除 Mark-Sweep
   + 从根集合GC Roots扫描，对存活对象进行标记，在扫描所有标记对象进行回收。
   + 不需要对象的移动
   + 会造成内存碎片；原因是直接回收了不存活得对象。
2. 复制算法 Copying
   + 把堆分成一个对象面和多个空闲面
   + 在对象面上分配空间，对象面满了后就复制到空闲面，形成对调。
   + 不会有内存碎片。
3. 标记-整理算法
   + 在标记清理的同时将存活对象向左边空闲空间移动
   + 解决内存碎片问题
4. 分代收集算法
   + 堆区域
     + 老年代 少量需要回收
     + 新生代 大量需要回收
   + 永久代

## 分代收集算法

1. 年轻代
   1. 复制为主
   2. 所有新生对象都在新生代
   3. 按照8：1：1分成一个eden区和两个survivor区，大部分对象在eden中生成。
   4. 回收时，先将eden区的复制到survivor0区，然后情况eden区，如果survivor0区满了后，将eden区和0区的复制到survivor1区，然后情空eden和0区，交换0区和1区，如此往复。
   5. 当1区也放不下时，直接放到老年代
   6. 当老年代也放不下的时候，触发一次Full GC，也叫Major GC.
2. 老年代
   1. 以标记整理为主
   2. 在年轻代中经历多次垃圾回收仍然存活的对象，放入老年代。
   3. 内存比新生代大很多，大概1：2。
   4. 触发Full GC的概率比较低。

## CMS垃圾收集器

1. 初始标记：记录下直接与 root 相连的对象，暂停所有的其他线程，速度很快； 
2. 并发标记：同时开启 GC 和⽤户线程，⽤⼀个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并 不能保证包含当前所有的可达对象。因为⽤户线程可能会不断的更新引⽤域，所以 GC 线程⽆法保证可达性分 析的实时性。所以这个算法⾥会跟踪记录这些发⽣引⽤更新的地⽅。
3. 重新标记：重新标记阶段就是为了修正并发标记期间因为⽤户程序继续运⾏⽽导致标记产⽣变动的那⼀部分对 象的标记记录。【这个阶段的停顿时间⼀般会⽐初始标记阶段的时间稍⻓，远远⽐并发标记阶段时间短】；
4. 并发清除：开启⽤户线程，同时 GC 线程开始对为标记的区域做清扫。 
5. 优点：并发收集、低停顿
6. 对CPU资源敏感、无法处理浮动垃圾、使用标记清除会有大量碎片。

## G1垃圾收集器

1. 力求最优解的收集器
2. 把堆分成多个region的做法，利用分代的思想，优先处理活跃对象小的。

## 使用监控工具调优

### 1、堆信息查看

1. 可查看堆空间⼤⼩分配（年轻代、年⽼代、持久代分配） 
2.  提供即时的垃圾回收功能 
3. 垃圾监控（⻓时间监控回收情况）
4. 查看堆内类、对象信息查看：数量、类型等 
5. 对象引⽤情况查看 

有了堆信息查看⽅⾯的功能，我们⼀般可以顺利解决以下问题： 

1. 年⽼代年轻代⼤⼩划分是否合理 

2. 内存泄漏

3. 圾回收算法设置是否合理 

### 2、线程监控

线程信息监控：系统线程数量 

线程状态监控：各个线程都处在什么样的状态下 

Dump 线程详细信息：查看线程内部运⾏情况

死锁检查 

### 3、 热点分析

1. CPU 热点：检查系统哪些⽅法占⽤的⼤量 CPU 时间； 
2. 2. 内存热点：检查哪些对象在系统中数量最⼤（⼀定时间内存活对象和销毁对象⼀起统计）这两个东⻄对于系统 优化很有帮助。我们可以根据找到的热点，有针对性的进⾏系统的瓶颈查找和进⾏系统优化，⽽不是漫⽆⽬的 的进⾏所有代码的优化。 

### 4、快照

快照是系统运⾏到某⼀时刻的⼀个定格。在我们进⾏调优的时候，不可能⽤眼睛去跟踪所有系统变化，依赖快照功 能，我们就可以进⾏系统两个不同运⾏时刻，对象（或类、线程等）的不同，以便快速找到问题。 举例说，我要检查系统进⾏垃圾回收以后，是否还有该收回的对象被遗漏下来的了。那么，我可以在进⾏垃圾回收 前后，分别进⾏⼀次堆情况的快照，然后对⽐两次快照的对象情况。

### 5、内存泄露检查

 内存泄漏是⽐较常⻅的问题，⽽且解决⽅法也⽐较通⽤，这⾥可以᯿点说⼀下，⽽线程、热点⽅⾯的问题则是具体 问题具体分析了。 内存泄漏⼀般可以理解为系统资源（各⽅⾯的资源，堆、栈、线程等）在错误使⽤的情况下，导致使⽤完毕的资源 ⽆法回收（或没有回收），从⽽导致新的资源分配请求⽆法完成，引起系统错误。内存泄漏对系统危害⽐较⼤，因 为它可以直接导致系统的崩溃

## jvm常见参数设置

1. 堆设置 

   -Xms：初始堆⼤⼩ 

   -Xmx：最⼤堆⼤⼩ 

   -XX:NewSize=n：设置年轻代⼤⼩ -XX:NewRatio=n：设置年轻代和年⽼代的⽐值。如:为3，表示年轻代与年⽼代⽐值为 1：3，年轻代占整个年轻代 年⽼代和的 1/4 

   -XX:SurvivorRatio=n：年轻代中 Eden 区与两个 Survivor 区的⽐值。注意 Survivor 区有两个。如：3，表示 Eden：Survivor=3：2，⼀个Survivor区占整个年轻代的 1/5 

   -XX:MaxPermSize=n：设置持久代⼤⼩ 

2. 收集器设置 

   -XX:+UseSerialGC：设置串⾏收集器 

   -XX:+UseParallelGC：设置并⾏收集器 

   -XX:+UseParalledlOldGC：设置并⾏年⽼代收集器 

   -XX:+UseConcMarkSweepGC：设置并发收集器 

3.  垃圾回收统计信息 

   -XX:+PrintGC：开启打印 gc 信息 

   -XX:+PrintGCDetails：打印 gc 详细信息 

   -XX:+PrintGCTimeStamps -Xloggc:filename 

4.  并⾏收集器设置 

   -XX:ParallelGCThreads=n：设置并⾏收集器收集时使⽤的 CPU 数 

   -XX:MaxGCPauseMillis=n：设置并⾏收集最⼤暂停时间 

   -XX:GCTimeRatio=n：设置垃圾回收时间占程序运⾏时间的百分⽐ 

5.  并发收集器设置 

   -XX:+CMSIncrementalMode：设置为增量模式。适⽤于单 CPU 情况 

   -XX:ParallelGCThreads=n：设置并发收集器年轻代收集⽅式为并⾏收集时，使⽤的 CPU 数。并⾏收集线程数

## 双亲委派机制

> 这个名字起的不好，叫父加载委托更合适，难道翻译也是被女权打怕了？

1. 类加载器层级
   1. 根加载器
   2. 拓展加载器
   3. 系统类加载器
   4. 自定义加载器
2. 一个类收到加载请求后，会先去委派父类加载器完成，逐步往上，如果父类完不成的时候就才会用当前加载器加载。
3. 好处：最终都由顶端的类加载器进行加载，保证类在多个加载器中都是同一个类。如果每个类使用自己的加载器，确保了类的全局唯一性。
4. 主要实现看`java.lang.ClassLoader `的` loadClass() `⽅法。
5. 自定义加载，需要继承`loadClass() `,重写`findClass() `方法;
6. 打破双亲委派机制
   1. 自己写个类加载器
   2. 重写`loadClass()`
   3. 重写`findClass()`
7. 合适打破双亲委派机制？
   1. Java 中所有涉及 SPI 的加载动作
   2. 加载核心类库需要用户代码，如JDBC使用`DriverManager.getConnection`获取连接使用双亲委派就会有问题。

## Java内存模型

> 又称JMM，内存和CPU处理速度不一样，需要高速缓存，多核处理器又会有多个缓存，需要和主存保持一致。而JMM就是为了保证内存访问一致性出现的。

内存行为规范： 关于主内存与⼯作内存之间的具体的交互协议，即：⼀个变量如何从主内存拷⻉到⼯作内存、如何从⼯作内存同步 主内存之类的实现细节，Java内存模型中定义⼀下8种操作来完成：

1. lock(锁定)：作⽤于主内存的变量。它把⼀个变量标志为⼀个线程独占的状态； 
2. unlock(解锁)：作⽤于主内存的变量，它把处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁 定； 
3. read(读取)：作⽤于主内存的变量，它把⼀个变量的值从主内存传输到线程的⼯作内存中，以便随后的load动 作使⽤； 
4.  load(载⼊)：作⽤于⼯作内存的变量，它把read操作从主内存中得到变量值放⼊⼯作内存的变量的副本中； 
5.  use(使⽤)：作⽤于⼯作内存的变量， 它把⼯作内存中⼀个变量的值传递给执⾏引擎，每当虚拟机遇到⼀个需 要使⽤到变量的值的字节码指令时将会执⾏这个操作； 
6. assign(赋值)：作⽤于⼯作内存的变量。它把⼀个从执⾏引擎接收到的值赋值给⼯作内存的变量，每当虚拟机 遇到需要给⼀个变量赋值的字节码时执⾏这个操作； 
7. store(存储)：作⽤于⼯作内存的变量。它把⼀个⼯作内存中⼀个变量的值传递到主内存中，以便随后的write 操作使⽤； 
8. write(写⼊)：作⽤于主内存的变量。它把store操作从⼯作内存中得到的变量的值放⼊主内存的变量中。 如果要把⼀个变量从⼯作内存复制到⼯作内存，那就要按顺序执⾏ read 和 load 操作，如果要把变量从⼯作内存同 步回主内存，就要按顺序执⾏ store 和 write 操作。 

**上诉 8 种基本操作必须满⾜的规则**： 

1. 不允许 read 和 load、store 和 write 操作之⼀单独出现； 
2.  不允许⼀个线程丢弃它的最近的 assign 操作，即变量在⼯作内存中改变之后必须把该变化同步回主内存； 
3.  不允许⼀个线程⽆原因地（没有发⽣过任何 assign 操作）把数据从线程的⼯作内存同步回主内存中； 
4.  ⼀个新的变量只能在主内存中“诞⽣”，不允许在⼯作内存中直接使⽤⼀个未被初始化（load 或 assign）的变 量，换句话说就是对⼀个变量实施 use 和 store 操作之前，必须执⾏过了 assign 和 load 操作； 
5. ⼀个变量在同⼀时刻只允许⼀条线程对其进⾏ lock 操作，但 lock 操作可以被同⼀线程᯿复执⾏多次，多次执 ⾏ lock 后，只有执⾏相同次数的 unlock，变量才会被解锁； 
6. 如果对⼀个变量执⾏ lock 操作，将会清空⼯作内存中此变量的值，在执⾏引擎使⽤这个变量前，需要᯿新执 ⾏ load 或 assign 操作初始化变量的值；
7. 如果⼀个变量事先没有被 lock 操作锁定，则不允许对它执⾏ unlock 操作，也不允许去 unlock ⼀个被其他线 程锁定主的变量； 
8.  对⼀个变量执⾏ unlock 操作之前，必须先把此变量同步回主内存中（执⾏ store 和 write 操作）
