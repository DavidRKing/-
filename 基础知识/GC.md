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





   