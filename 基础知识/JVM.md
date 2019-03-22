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









































