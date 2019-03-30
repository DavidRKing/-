#### 如何选择框架

​    对应的开发者社区是否有名、是否活跃

​    框架的模块是否不断迭代

#### 家族体系

​    全家桶

#### Spring IOC

   IOC(Inversion of Control):控制反转

   Spring core最核心的部分

   需要先了解依赖注入(Dependency Inversion)

DI举例：设计行李箱

   轮子<-----底盘<-----箱体<-----行李箱（后者依赖于前者）

上层建筑依赖下层建筑，如果轮子改了，将颠覆整个设计

   行李箱<----箱体<----底盘<----轮子(后者注入前者)

含义:把底层类作为参数传递给上层类，实现上层对下层的"控制"  

##### 依赖注入方式

​    Setter 

​    Interface

​    constructor

​    annotation

依赖导致原则、IOC、DI、IOC容器的关系

​    IOC容器的优势:避免在各处使用new来创建类，并且可以做到统一的维护

​    创建实例的时候不需要了解细节

1.读取Bean配置信息(xml,java类@Configuration,注解@Autowire)

2.根据Bean注册表实例化Bean

3.将Bean实例放到Spring容器中

4.使用Bean(应用程序)

##### Spring IOC支持的功能

  依赖注入

  依赖检查

  自动装配

  支持集合

  制定初始化方法和销毁方法

  支持回调方法

##### Spring IOC容器的核心接口

   BeanFactory

​       spring框架最核心的接口

​       包含Bean的各种定义，便于实例化Bean

​       建立Bean之间的依赖关系

​       Bean生命周期的控制

   ApplicationContext



  BeanDefinition

​      主要用来描述Bean定义(将配置文件信息转换为beanDefinition对象)

   BeanDefinitionRegistry

​      提供向IOC容器注册BeanDefinition对象的方法

Beanfactory与ApplicationContext的比较

​    BeanFactory是Spring框架的基础设施，面向Spring

​    ApplicationContext面向使用Spring框架开发者

​    ApplicationContext的功能(继承 多个接口,应用上下文，代表整个大容器的所有功能) 

​       BeanFactory:能够管理，装配Bean

​       ResourcePatternResolver:能够加载资源文件

​       MessageSource:能够实现国际化等功能

​       ApplicationEventPublisher:能够注册监听器，实现监听机制



##### getBean方法的逻辑

​     转换beanName

​     从缓存中加载实例

​     实例化bean

​     检测parentBeanFactory

​     初始化依赖bean

​    创建Bean

##### Spring bean的作用域

   singleton

   prototype:针对每个getBean请求，容器都会创建一个bean

   request:会为每个http请求创建一个bean

   session:会为每个session创建一个实例

   globalSession:

##### Spring bean的生命周期

   so easy

##### Spring AOP

 关注点分离：不同的问题交给不同的部分去处理

 面向切面编程AOP正式此种技术的体现

 通用化功能代码的实现，对应的就是所谓的切面(Aspect)

 业务功能代码和切面代码分开，架构将变得高内聚低耦合

 确保功能的完整性：切面最终需要被合并到业务中(Weave)

##### AOP的三种植入方式

  编译时注入：需要特殊的Java编译器，AspectJ

  类加载时注入：需要特殊的java编译器，AspectJ 和 AspectWerkz

  运行时置入: Spring采用的方式,通过动态代理的方式，实现简单

使用：so easy

##### AOP的实现:jdkProxy 和 Cglib

   由AopProxyfactory根据AdvisedSupport对象的配置来决定

   默认策略如果目标类是接口，则用JDKProxy类来实现，否者用后者

  JDKProxy的核心:InvocationHandler接口和Proxy类 (反射)

  Cglib：以继承的方式动态生成目标类的代理  (借助ASM实现)

  注:反射机制在生成类的过程中比较高效，ASM在生成类之后的执行过程中比较高效

##### Spring里的代理模式的实现

 真实实现类的逻辑包含在了getBean方法里

 getBean方法返回的实际上是Proxy的实力

 Proxy实例是Spring采用JDK Proxy 或CGLIB动态生成的

##### Spring事务的相关问题

   ACID

   隔离级别

   事务传播

 

