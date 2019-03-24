#### JAVA 垃圾回收机制

​    对象被判定为垃圾的标准？

​    没有被其他对象引用

#### 判定对象是否为垃圾算法

   引用计数算法

   可达性分析算法

#### 判断对象引用数量

   通过判断对象的引用数量来决定对象是否可以被回收

   每个对象实例都有一个引用计数器，被引用则+1，完成引用则-1.

   任何引用计数为0的对象实例可以被当作垃圾收集

   优点：执行效率高，程序执行受影响较小

   缺点:无法检测出循环引用的情况，导致内存泄漏

```java
MyObject obj1 = new MyObject();
MyObject obj2 = new Myobject();
obj1.child = obj2;
obj2.child = obj1;
obj1=null;
obj2=null;
//obj2.child = obj1; obj1.child=obj2 ................
//obj1和obj2的引用都是2，如果我要回收obj1，则obj2必须被回收。但是obj2被obj.child引用。发生了循环引用
```

#### 可达性分析

  通过判断对象的引用链是否可达来决定对象是否可以被回收(离散数学，图论)

  GC Root Set (一些列GC Root)

 如果GC root到对象可达，则不能回收

#### 可以作为GC ROOT的对象

  虚拟机栈中引用的对象(栈帧中的本地变量表   java方法里new一个对象，赋给一个局部变量。这个对象就是GC ROOT)

  方法区中的常量引用的对象(类里定义了一个常量,该常量保存的是某个对象的地址,那么被保存的对象就成了GC ROOT。当其他对象引用了他的时候，就会形成链)

  方法区中的类静态属性引用的对象

  本地方法栈中JNI（Native方法）的引用对象

  活跃线程引用的对象

####   垃圾回收算法

​    标记-清除算法(Mark and Sweep)

​    标记:从根集合进行扫描，对存活的对象进行标记

​    清除:对堆内存从头到尾进行线性遍历，回收不可达对象内存

​      缺点:碎片化(有可能提前触发垃圾收集)

​      

​    复制算法(Coping  新生代)

​       分为对象面和空闲面

​       对象在对象面上创建

​       存活的对象被从对象面复制到空闲面

​       将对象面所有对象内存清除

​          解决了碎片化的问题

​          顺序分配内存，简单高效

​          适用于对象存活率底的场景



  标记-整理算法(Compacting  老年代)

​      标记:从根集合进行扫描，堆存活的对象进行标记

​      清除:移动所有存活对象，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收。     

   避免内存的不连续性。

   不用设置两块内存互换

   适用于存活率高的场景



分代收集算法(Generational Collector)

   垃圾回收算法的组合拳

   按照对象声明周期的不同划分区域以采用不同的垃圾回收算法

   目的:提高JVM回收效率

JDK6,7

young generation

old generation

permanent generation

JDK8

young generation

old generation (Tenured space)



#### GC的分类

  Minor GC (发生在年轻代的垃圾收集动作,复制算法)

  Full GC (老年代)

#### 年轻代：尽可能快速地收集掉那些生命周期短的对象

   Eden区

   两个Surivor区:每次分配对象用eden 和 一个 surivor区。当发生gc时，将eden和surivior区中存活的对象复制到另一个surivor区中，同时清理掉eden和surivior。当surivior区不同用的时候，需要老年代进行分配担保

​        Eden(8/10)   from(1/10) to(1/10)   

​        Young (1/3堆空间)

​       

​       Old(2/3堆空间)

#### 对象如何晋升到老年代

   经历一定Minor次数依然存活的对象

   Survivor区中放不下的对象

   新生的大对象(-XX+PretenuerSizeThreshold)

#### 常用的调优参数

   -XX:SurvivorRatio:Eden和Survivor的比值,默认(8:1)

   -XX:NewRatio:老年代和年轻代内存大小比例 (2: old:yong = 2)

   -XX:MaxTenuringThreshold:对象从年轻代晋升到老年代经历GC次数的最大阈值

#### 老年代:存放生命周期较长的对象

   标记-清理算法

   标记-整理算法

   Full GC 和Major GC (老年代的GC)

   Full GC比Minor GC慢,但执行频率低   

#### 触发Full GC的条件

​    老年代空间不足(一个大对象，如果Eden空间不足，会存放到老年代，如果老年代空间不足，则触发Full GC)

​    永久代空间不足(JDK 7 以前的jdk)

​    CMS GC时出现promotion failed，concurrent mode failure

​    Minor GC晋升到老年代平均大小大于老年代的剩余空间

​    调用System.gc()

​    使用RMI来进行RPC或管理JDK应用，每小时执行1次FULL GC

#### STOP-the-world

   JVM由于要执行GC而停止了应用程序的执行

   任何一种GC算法中都会发生

   多数GC优化通过减少stop-the-world发生的时间来提高程序性能。（ 高吞吐，低停顿）

#### Safepoint

  分析过程中对象引用关系不会发生变化的点。

  产生Safepoint的地方:方法调用;循环跳转;异常跳转等.

  安全点数量得适中

#### 常见的垃圾收集器

  JVM的运行模式

​     Server:启动较慢,但稳定后，速度要比client块

​     Client:启动较快

#### 垃圾收集器之间的联系

  见深入理解java虚拟机图

 Serial收集器(-XX:+UseSerialGC,复制算法  Client默认收集器)

​     单线程收集，进行垃圾收集器，必须暂停所有工作线程。

​     简单高效，Client模式下默认的年轻代收集器

 ParNew收集器(-XX:+UseParNewGC,复制算法)

​     多线程收集，其余行为、特点和Serial收集器一样。

​     单核执行效率不如Serial,在多核下执行才有优势

 Parallel Scavenge收集器(-XX:+UseParallelGC,复制算法 Server默认收集器)

​     吞吐量=运行用户代码的时间/(运行用户代码时间+垃圾收集时间)

​     比起关注用户线程停顿时间，更关注系统的吞吐量

​     在多核下执行才有优势，Server模式下默认的年轻代收集器

​     -XX:+UseAdaptiveSizePolicy:把内存管理的调优任务交给JVM

####  老年代常见的收集器

   Serial Old收集器(-XX:+UseSerialOldGC,标记-整理算法)

​      单线程收集，进行垃圾收集时，必须暂停所有工作线程

​      简单高效，Client模式下默认的老年代收集器

   Parallel Old收集器(-XX:+UseParallelOldGC,标记-整理算法) (JDK 6 以后)

​      多线程，吞吐量优先

   CMS收集器(-XX:+UserConcMarkSweepGC,标记-清除算法)

​      1.初始标记:STOP-THE-WORLD(虚拟机停顿正在执行的任务,从GC ROOTS 开始 扫描只扫描到与他直接关联的对象并做标记,很快)  stop-world

​      2.并发标记:并发追溯标记，程序不会停顿(从1结果，继续向下标记) 

​      3.并发预清理:查找执行并发标记阶段从年轻代晋升到老年代的对象

​      4.重新标记:暂停虚拟机，扫描CMS堆中的剩余对象(从GC ROOTS开始进行对象关联，慢) stop-world

​      5.并发清理:清理垃圾对象，程序不会停顿

​      6.并发重置:重置CMS收集器的数据结构     

​    G1收集器(-XX:+UserG1GC,复制+标记-整理算法)

#### GarBage First收集器的特点

​     并行和并发

​     分代收集

​     空间整合

​     可预测的停顿

  将整个Java堆内存划分为多个大小相等的Region.

  年轻代和老年代不再物理隔离

   JDK11  Epsilon GC 和 ZGC

#### GC相关的问题

  Object的finalize()方法的作用是否与C++的析构函数作用相同?

​    与C++的析构函数不同，析构函数调用确定，而它是不确定的

​    将未被引用的对象放置与F-QUEUE队列中

​    方法执行随时可能会被中止

​    给予对象最后一次重生的机会

#### Java中的强引用，软引用，弱引用，虚引用有什么作用

​     强引用

​         最普遍的引用:Object obj = new Object();

​         抛出OutOfMemoryError终止程序也不会回收具有强引用的对象。

​         通过将对象设置为null来弱化引用,使其被回收

​    软引用

​         对象处在有用但非必须的状态

​         只有当内存空间不足时，GC会回收该引用的对象内存。

​         可以用来实现内存敏感的高速缓存

​        

```java
String str = new String ("abc");
SoftReference<String> softRef = new SoftReference<String>(str);//软引用
```

   

​      弱引用

​         非必须的对象，比软引用更弱一些

​         GC时会被回收

​         被回收的概率也不大，因为GC线程优先级比较低

​         适用于引用偶尔被使用且不影响垃圾收集的对象

```java
String str = new String ("abc");
WeakReference<String> weakRef = new WeakReference<String>(str);//弱引用
```

​      虚引用

​          不会决定对象的生命周期

​          任何时候都可能被垃圾回收期回收(相当于没有引用)

​          跟踪对象被垃圾回收器回收的活动，起哨兵的作用

​          必须和引用队列ReferenceQueue联合使用

```java
String str = new String ("abc");
ReferenceQueue queue = new ReferenceQueue();
PhantomReference ref = new PhantomReference(str,queue);
```



强>软>弱>虚

引用队列

​     无实际存储结构，存储逻辑依赖于内部节点之间的关系来表达  (进保存head节点,类似连表)

​     存储关联的且被GC的软引用，弱引用以及虚引用



​     