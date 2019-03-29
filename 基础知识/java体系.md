#### JAVA异常

  what：异常类型回答了什么被抛出

  where:异常堆栈跟踪回答了在哪抛出

  why:异常信息回答了为什么被抛出

##### Java的异常体系

  error：程序无法处理的系统错误，编译器不做检查(JVM层。内存溢出，栈溢出......程序无法预防)

  Exception:程序可以处理的异常，捕获后可能恢复

  总结:前者是程序无法处理的错误,后者是可以处理的异常

  Exception:

​        RuntimeException:不可预知的，程序应当自行避免（空指针了）不用 throws

​        非RuntimeException:可预知，从编译器校验的异常(文件不存在，IOexcetion) throws Exception

 从责任角度看:

​      error属于JVM需要负担的责任

​      RuntimeException是程序应该负担的责任;

​      Checked Exception可检查异常时Java编译器应该负担的责任。

##### 常见的Error以及Exception

RuntimeException

   NullPointerException

   ClassCastException 类型强制转换异常

   IllegalArgumentException  传递非法参数异常

   IndexOutofboundsException  

   NumberFormatException  数字格式异常

非RuntimeException

​    ClassNotFoundException  找不到指定的Class类异常

​    IOException  - IO 操作异常

Error

  NoClassDefFoundError-找不到class定义异常  (类依赖的class或者jar不存在，类文件存在，但是存在不同的域中，大小写问题，javac编译的时候是无视大小写的，很可能编译出来的class文件与想要的不一样)

  StackOverflowError-深递归导致栈被耗尽而抛出的异常

  OutOfMemoryError-内存溢出异常

#### 异常处理机制

   抛出异常：创建异常对象，交由运行时系统处理

   捕获异常: 寻找合适的异常处理器处理异常，否则终止运行

   具体明确：

   提早抛出

   延迟捕获

#### 高效主流的异常处理框架

  在用户看来，应用系统发生的所有异常都是应用系统内部的异常

  设计一个通用的继承自RuntimeException的异常来统一处理

  其余异常都同意转移为上述异常AppException

  在catch之后，抛出上述异常的子类，并提供足以定位的信息

  有前端接受AppExcetion做统一处理

#### Java异常处理消耗性能的地方

  try-catch块影响JVM的优化

  异常对象实例需要保存栈快照等信息，开销较大

####  Java集合框架

​    数据结构

​       数组和链表的区别

​       链表的操作，如反转，链表循环检测，双向链表，循环链表相关操作；

​       队列，栈的应用

​       二叉树的遍历方式及其递归和非递归的实现

​       红黑树的旋转

​      内部排序:如递归排序，交换排序(冒泡，快排)，选择排序，插入排序

​      外部排序:应掌握如何利用有限的内存配合海量的外部存储来处理超大的数据集，要有具体的相关思路

​      哪些排序时不稳定的，稳定意味着什么

​      不同数据集，各种排序最好或最差的情况

​      如何优化算法

Collections看源码  so easy

##### Map源码   so easy

HashMap  HashTable  ConcurrentHashMap

HashMap(jdk8以前) ：数组+链表

​                 (jdk8以后):数组+链表+红黑树   TREEIFY_THRESHOLD    UNTREEIFY_THRESHOLD

HashMap：put方法的逻辑

​      1.如果hashmap未被初始化，则初始化 (换算成 2的 n次方)

​      2.对Key求hash值,然后再计算下标

​      3.如果没有碰撞，直接放入桶中

​      4.如果碰撞了，以链表的方式链接到后面

​      5.如果链表长度超过阈值，就把链表转换成红黑树

​      6.如果链表长度低于6，就把红黑树转回链表

​      7.如果节点已经存在就替换旧值。

​      8.如果桶满了(容量16*加载因子0.75),就需要resize(扩容2倍后重排)

HashMap:如何有效减少碰撞

​      扰动函数:促使元素位置分布均匀，减少碰撞几率

​      使用final对象，并采用合适的equals()和hashCode()方法 (String ,Integer比较适合做key)

​      

```java
h=hashCode();
hash = h^(h>>>16);     //hash右移16位 与 原 hash值异或,混合 高位和低位。
index = (n-1) & hash;  //数组长度 2的n次方
```

HashMap:扩容的问题

​    多线程环境下，调整大小会存在条件竞争，容易造成死锁

​    rehashing是一个比较耗时的过程

总结:

​    成员变量:数据结构，树化阈值

​    构造函数:延迟创建

​    put和get的流程

​    哈希算法，扩容，性能

##### HashTable

​    早期Java类库提供的哈希表的实现

​    线程安全：涉及到修改Hashtable的方法，使用synchronized修饰

​    串行化的方式运行，效率差

##### 如何优化HashTable?

​    通过锁细粒度化，将整锁拆解成多个锁进行优化

​    早期的ConcurrentHashMap：通过分段锁Segment来实现   (数组+链表)

​    当前的ConcurrentHashMap:CAS+synchronized使锁更细化(数组+链表+红黑树)

​    ConcurrentHashMap:put方法的逻辑

​      1.判断Node[]数组是否初始化，没有则进行初始化操作

​      2.通过hash定位数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加(链表头节点),添加失败则进入下次循环。

​      3.检查到内部正在扩容，就帮助它一块扩容

​      4.如果f!=null,则使用synchronized锁住f元素(链表/红黑二叉树的头元素)

​         4.1如果Node(链表结构)则执行链表的添加操作。

​         4.2如果是TreeNode(树形结构)则执行树添加操作。

​      5.判断链表长度已经达到临界值8，当然这个8是默认值，大家也可以去做调整，当节点数超过这个值就需要把链表转换为树结构。

​    ConcurrentHashMap总结：比起Segment,锁拆得更细。(hash不冲突，就不会锁)

​        首先使用无锁操作CAS插入头节点，失败则循环尝试

​        若头节点已存在，则尝试获取头节点的同步锁，再进行操作

#####  ConcurrentHashMap:其他注意点

​       size()方法和mappingCount()方法的异同，两者计算是否准确?

​       多线程下如何进行扩容？

##### 三者区别:

​      hashMap线程不安全，数组+链表+红黑树

​      Hashtable线程安全，锁住整个对象，数组+链表

​      ConcurrentHashMap线程安全,CAS+同步锁,数组+链表+红黑树

​      HashMap的key,value均可以为null,而其他的两个类不支持

#### JUC包

   CAS：atomic包的基础

   AQS：locks包以及一些常用类比如Semophore，ReentrantLock等类的基础

   线程执行器 executor

   锁  locks

   原子变量类 atomic

   并发工具类 tools

​        CountDownLatch:让主线程等待一组事件发生后继续执行

​              事件指的是：CountDownLatch里的countDown()方法

​        CyclicBarrier:阻塞当前线程，等待其他线程，

​           等待其他线程，且会阻塞自己当前线程，所有线程必须同时到达栅栏位置后，才能继续执行；

​           所有线程到达栅栏处，可以出发执行另外一个预先设置的线程

​        Semaphore:控制某个资源可被同时访问的线程个数

​        Exchanger:两个线程到达同步点后，相互交换数据

​                             

   并发集合collections

​         BlockingQueue:提供可阻塞的入队和出队操作

​         主要用于生产者-消费者模式，在多线程场景时生产者线程在队列尾部添加元素，而消费者线程则在队列头部消费元素，通过这种方式能够达到将任务生产和消费进行隔离的目的

ArrayBlockingQueue:一个由数组结构组成的有界阻塞队列 (看源码)

LinkkedBlockingQueue:一个由链表结构组成的有界/无界队列 (看源码)

PriorityBlockingQueue:一个支持优先级排序的无界阻塞队列；(看源码)

DealyQueue:一个使用优先级队实现的无界阻塞队列

SynchronousQueue:一个不存储元素的阻塞队列

LinkedTransferQueue:一个由链表结构组成的无界阻塞队列

LinkedBlockingDeque:一个由链表结构组成的双向阻塞队列

#### JAVA的IO机制

   BIO、NIO、AIO

 BLOCK-IO:InputStream和Outputstream, Reader和Writer区别

   1.系统调用                                                  这期间  程序等待处理数据。线程被挂起

   2.操作系统（无数据）

   3。等待返回数据

   4.数据准备ok

   5.数据从内核复制到用户空间

   6.数据拷贝完成 

   7.程序可以使用数据

NonBlock-IO:构建多路复用的、同步非阻塞的IO操作

   调用系统级别的select\poll\epoll

​    1.系统调用

​    2.数据还没准备好

​    3.系统调用

​    4.数据还未准备好

​    。。。。这期间  应用程序反复查看数据是否准备好。不阻塞

​    5.数据准备好，

   6，数据从内核复制到用户空间（数据拷贝完成）

   7.程序可以使用数据

   8.拷贝数据阻塞等待

NIO核心

   Channels

​       FileChannel

​       DatagramChannel

​       SocketChannel

​       ServerSocketChannel

   Buffer

​      

   Selectors

​     selector 处理多个channel

支持一个进程所能打开的最大连接数



| select | 单个进程所能打开的最大连接数由FD_SETSIZE宏定义，其大小是32个整数的大小(在32位机器上，大小是32*32,64位机器上是32*  ),我们可以对其进行修改，然后重新编译内核，但实性能无法保证，需要做进一步测试 |
| ------ | ------------------------------------------------------------ |
| poll   | 本质上与select没有区别，但是它没有最大连接数的限制，原因是它是基于链表来存储的 |
| epoll  | 虽然连接数有上线，但是很大，1G内存的机器上可以打开10万左右的连接 |

FD剧增后带来的IO效率问题

| select | 因为每次调用时都会对连接进行线性遍历，所以随着FD的增加会造成遍历速度的"线性下降"的性能问题 |
| ------ | ------------------------------------------------------------ |
| poll   | 同上                                                         |
| epoll  | 由于epoll是根据每个fd上的callback函数来实现的，只有活跃的socket才会主动调用callback，所以在活跃socket较少的情况下，使用epoll不会有"线性下降"的性能问题，但是所有socket都很活跃的情况下，可能会有性能问题 |

消息传递方式

| select | 内核需要将消息传递到用户空间，需要内核的拷贝动作 |
| ------ | ------------------------------------------------ |
| poll   | 同上                                             |
| epoll  | 通过内核和用户空间共享一块内存来实现，性能较高   |

Asynchronous IO:基于时间和回调机制

1.系统调用aio_read                期间 程序可以处理其他别的时间

2.注册完立即返回

3.等待数据

4.数据准备ok

5.数据从内核复制到用户空间

6.数据拷贝ok

7.通知程序已经可以使用

AIO如何进一步加工处理结果

基于回调:实现CompletionHandler接口,调用时触发回调函数

返回Future:通过isDone()查看是否准备好，通过get()等待数据返回

| 属性\模型             | 阻塞BIO    | 非阻塞NIO    | 异步AIO      |
| --------------------- | ---------- | ------------ | ------------ |
| blocking              | 阻塞并同步 | 非阻塞但同步 | 非阻塞并异步 |
| 线程数(server:client) | 1:1        | 1:N          | 0:N          |
| 复杂度                | 简单       | 较复杂       | 复杂         |
| 吞吐量                | 底         | 高           | 高           |

