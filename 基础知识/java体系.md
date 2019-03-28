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









