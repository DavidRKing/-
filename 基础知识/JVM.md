### 对java的理解

​      平台无关性

​      GC

​      语言特性

​      面向对象

​      类库

​      异常处理

#### Compile Once,Run Anywhere如何实现

​    编译时

​    运行时

​    javap -c xxxx.class

#### JVM如何加载.class文件

   Java虚拟机

  class file->Class Loader

  class loader:依据特定格式，加载class文件到内存     

  Runtime Data Area:JVM内存空间结构模型 

  Native Interface:融合不同开发语言的原生为java所用  

  Execution Engine:对命令进行解析   

#### 谈谈反射

​        JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它任意方法和属性；这种动态获取信息以及动态调用对象方法的工能称为java语言的反射机制。

#### 写一个反射的例子

​       

```java
public class Robot{
    prvate String name;
    public void sayHi(String message){
        System.out.println(message+" "+name);
    }
    private String throwHello(String message){
        return "Hello"+msg;
    }

    public static void main(String args[]) throws Exception{
        Class rc = Class.forName("com.xxx.xxx.xx.Robot");
        Robot r = (Robot) rc.newInstance();
        System.out.println("class name is "+rc.getName);
        //获取throwHello方法
        Method getHello = r.getDeclaredMethod("throwHello",String.class);
        //因为是私有方法，所以要设置为true
        getHello.setAccessible(true);
        //调用 r的 throwHello方法
        Object str = getHello.invoke(r,"Bob");
        System.out.println("getHello result is "+str);
        //只能获取public方法
        Method sayHi = rc.getMethod("sayHi",String.class);
        sayHi.invoke(r,"Welcome");
        //获取name属性
        Field name = rc.getDeclaredField("name");
        name.setAccessible(true);
        name.set(r,"Alice");
        sayHi.invoke(r,"Welcome");
    }
}




```

#### 类从编译到执行的过程

   编译器将Robot.java源文件编译为Robot.class字节码文件

   ClassLoader将字节码转换为JVM中的Class<Robot>对象(byte数组传进来)

   JVM利用Class<Robot>对象实例化Robot对象

#### 谈谈ClassLoader

​        ClassLoader在Java中有着非常重要的作用，它主要工作在Class装载的加载阶段，其主要作用是从系统外部获得Class二进制流。它是Java的核心组件，所有的Class都是由ClassLoader进行加载的，ClassLoader负责通过将Class文件里的二进制数据流装载进系统，然后交给Java虚拟机进行连接、初始化等操作。

ClassLoader.java中 主要看loadClass(String name)方法

#### ClassLoader的种类

   BootStrapClassLoader:C++编写,加载核心库java.*

   ExtClassLoader:Java编写，加载扩展库javax.*

   AppClassLoader:Java编写，加载程序所在目录(Classpath下的类)

   自定义ClassLoader:Java编写,定制化加载

#### 自定义ClassLoader的实现

   关键函数    findClass();

​                      defineClass();

```java
//自定义ClassLoader去读取自己写的Class文件流,并解析，返回class对象
public class MyClassLoader extends ClassLoader{
    private String path;
    private String classLoaderName;
    
    public MyClassLoader(String path,String classLoaderName){
        this.path = path;
        this.classLoaderName=classLoaderName;
    }
    //用于寻找类文件,
    @Override
    public Class findClass(String name){
        //可以从任何地方获取 class文件的二进制流。
        byte[] b =  loadClassData(name);
        return defineClass(name,b,0,b.length);
    }
    //用于加载类文件
    private byte[] loadClassData(String name){
        name = path + name +".class";
        InputStream in = null;
        ByteArrayOutputStream out = null;
        try{
            in = new FileInputStream(new File(name))
            out = new ByteArrayOutputStream();
            int i=0;
            while((i = in.read))!=-1){
                out.write(i);
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            try{
              out.close();
              in.close();    
            }catch(Exception e){
                
            }
        }
        return out.toByteArray();
    }
    
    public static void main(String[] args) throws Exception{
        MyClassLoader my = new MyClassLoader("path","myClassLoader");
        Class c  =my.loadClass("MyClass.class");
        System.out.println(c.getClassLoader());
        c.newInstance();
        System.out.println(c.getClassLoader());
        System.out.println(c.getClassLoader().getParent());
        System.out.println(c.getClassLoader().getParent().getParent());
        //myClassLoader,appClassLoader,ExtClassLoader,null
    }
}
```

#### 谈谈类加载器的双亲委派机制(ClassLoader中 loadClass方法)

​      1.自底向上检查类是否已经加载

​      2.自顶向下尝试加载类         

#### 为什么要使用双亲委派机制去加载类

​      避免多份同样字节码的加载(class对象)

#### 类的加载方式

​      隐式记载:new

​      显示加载:loadClass,forName等          (class. newInstance());

#### loadClass和forName的区别

​     Class.forName得到的class是已经初始化完成的。

​     Classloader.loadClass得到的class是还没有链接的。

```java
//Robot里有一个静态代码块,没写
public class LoadDifference{
    public static void main(String[] args){
        //不会打印静态代码块内容,class还没有链接
        ClassLoader c1 = Robot.class.getClassLoader();
        //会打印静态代码块内容
        Class r = Class.forName("package.Robot");
        //加载mysql  Driver中有静态代码块，完成初始化
        Class.forName("com.mysql.jdbc.Driver");
        //Spring ioc等，需要快速启动，延时加载，所以用classLoad加载,把加载工作留到需要使用的时候再去做
  
        
    }
}
```



#### 类的装载过程

​     加载(通过ClassLoader加载class文件字节码,生成Class对象)

​     链接

​           校验:检查加载的class的正确性和安全性

​           准备:为类变量(static变量)分配存储空间并设置类变量初始值(类变量类型的默认值，而非实际值)

​           解析:JVM将常量池内的符号引用转换为直接引用(可选)

   初始化

​           执行类变量赋值和静态代码块

#### JAVA内存模型知多少?

​     内存简介

​     32位处理器:2^32的可寻址范围(4GB)

​     64位处理器:2^64的可寻址范围

​     地址空间划分:

​        内核空间:

​        用户空间:java进程实际运行时的空间

​     JVM内存模型-JDK8

   线程私有:虚拟机栈(java方法SOF&OOM)，

​                       Java方法执行的内存模型

​                       包含多个栈帧（每个栈帧包括:局部变量表，操作数栈，动态链接，返回地址等）

​                           局部变量表:包含方法执行过程中的所有变量(this引用)

​                           操作数栈:入栈，出栈，复制，交换，产生消费变量

```java
public class ByteCodeSample{
    public static int add(int a,int b){
        int c =0;
        c = a+b;
        return c;
    }
   //执行（1,2）
      
}
```

​                                                 

​                   本地方法栈(native方法SOF&OOM)，

​                       与虚拟机栈相似，主要用于标注了native方法

​                   程序计数器（字节码指令 no OOM）

​                       当前线程所执行的字节码行号指示器(逻辑)

​                       改变计数器的值来选取下一条需要执行的字节码指令

​                      和线程是一对一的关系，即线程私有

​                      对Java方法计数，如果是Native方法则计数器值为Undefined

​                     不会发生内存泄漏

   线程共享:MetaSpace（类加载信息），堆(常量池:字面量和符号引用OOM)



#### 递归为什么会引发java.long.stack.overflowerror异常

递归过深，栈帧树超出虚拟机栈深度

虚拟机栈过多会引发java.lang.outofmemoryError异常



#### 元空间(MetaSpace)与永久代(PermGen)的区别

  类的元数据 ，class的信息。（方法区等）

  元空间使用本地内存，而永久代使用的是JVM的内存

  java.lang.outOfMemoryError:PermGen space不复存在，因为使用了本地内存.

#### MetaSpace相比PermGen的优势

   字符串常量池存在永久代中，容易出现性能问题和内存溢出。

   类和方法的信息大小难以确定，给永久代指定大小带来困难。

   永久代会为GC带来不必要的复杂性

   方便HotSpace与其他JVM如Jrockit的集成

#### Java堆(Heap)

  对象实例的分配区域

  GC管理的主要区域

#### JVM三大性能调优参数-Xms -Xmx -Xss的含义

  java -Xms128m -Xmx128m -Xss256k -jar xxxx.jar

  -Xss:规定了每个线程虚拟机栈(堆栈)的大小

 -Xms:堆得初始值

 -Xmx:堆能达到的最大值

#### Java内存模型中堆和栈的区别-内存分配策略

  静态存储:编译时确定每个数据目标在运行时的存储空间需求。

  栈式存储:数据区需求在编译时未知，运行时模块入口前确定。

  堆式存储:编译时或运行时模块入口都无法确定，动态分配。

联系:引用对象、数组时、栈里定义变量保存堆中目标的首地址。

​    Person p = new Person()；（堆和栈中怎么分配的）

管理方式:栈自动释放，堆需要GC

空间大小:栈比堆小

碎片相关:栈产生的碎片远小于堆

分配方式:栈支持静态分配和动态分配，而堆仅支持动态分配

效率:栈的效率比堆高

#### 元空间、堆、栈独占部分的联系-内存角度

元空间-CLASS:HelloWorld-Method:sayHello\setName\main-Field:name

​             CLASS:System

Java堆:Object:String("test");

​            Object:HelloWorld

线程独占:Parameter reference:"test" to String object

​               :Variable reference:"hw" to HelloWorld object

​               :Local Variables: a with 1,lineNo

#### 不同JDK版本之间intern()方法的区别-JDK6 VS JDK6+

```java
String s = new String("a");
//jk6: 当调用intern方法时，如果字符串常量池先前已创建出该字符串对象，则返回池中的该字符串的引用。
//否则，将此字符串对象添加到字符串常量池中，并且返回该字符串对象的引用。

//Jdk6+:当调用intern方法时，如果字符串常量池先前已创建出该字符串对象，则返回池中的该字符串的引用。
//否则，如果该字符串对象已经存在于java堆中，则将堆中的此对象的引用添加到字符串常量池，并且返回该引用，
//如果堆中不存在，则在池中创建该字符串并返回引用。

//jdk8将字符串常量池移动到java堆中。避免以前在方法区会引发内存溢出
s.intern();

//jdk 1.6 PermGen space 内存溢出
public static void main(){
    for(int i=0;i<1000;i++){
        getRandomString(1000000).intern();
    }
    System.out.println("complete!!!")
}
//jdk 7 没问题


String s = new String ("a");
s.intern();
String s2 ="a";
System.out.println(s==s2);

String s3 = new String("a")+new String ("a");
s3.intern();
String a4 = "aa";
System.out.println(s3==s4);
//1.6+ false true
//1.6 false false

//"a","aa"直接在常量池中。 new String("a"),new String("aa") 直接在堆中创建出来
//在jdk1.6中   s引用 java堆中地址。   s2引用java常量池中地址。 两个地址肯定不一样

//对于jdk1.6+以上:
//第一个是一样的。第二个。s3在堆中创建 aa。s3.intern()。将堆中引用复制到常量池中。此时，s4与s3地址一样
```







































