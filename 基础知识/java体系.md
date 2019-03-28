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