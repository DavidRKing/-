### Redis缓存

客户端------>缓存层------>存储层

穿透查询:缓存层miss

回种:回写到缓存

### 缓存中间件 Memcache和Redis区别

Memcache:代码层类似Hash

   支持简单数据类型

   不支持数据持久化存储

   不支持主从

   不支持分片

Redis

​    数据类型丰富

​    数据磁盘持久化存储

​    支持主从

​    支持分片

### 为什么Redis能这么快

100000+QPS(每秒内查询次数)

​    完全基于内存，绝大部分请求是纯粹的内存操作，执行效率高。

​    数据结构简单，对数据操作也简单。

​    采用单线程，单线程也能处理高并发请求，想多核也可以启动多实例。

​    使用多路IO复用模型，非阻塞IO

### 多路IO复用模型

#### FD：File Descriptor，文件描述符

​          一个打开的文件通过唯一的描述符进行引用，该描述符是打开文件的元数据到文件本身的映射。

#### 传统的阻塞I/O模型

#### 多路复用IO模型

#### Select系统调用

​    selector 负责监听很多文件描述符。

#### Redis采用的I/O多路复用函数:epoll/kqueue/evport/select?

   因地制宜

   优先选择时间复杂度O(1)的I/O多路复用函数作为底层实现

   以时间复杂度为O(n)的select作为保底

   基于react设计模式监听I/O事件          

IO模式和IO多路复用:http://www.cnblogs.com/zingp/p/6863170.html

### 说说你用过的Redis的数据类型

#### 供用户使用的数据类型

 String:最基本的数据类型，二进制安全（512M)

​             set name "redis"

​             get name

​             set name "memcache"

​             get name

​             set count 1

​             get count

​             incr count

​             get count

​            统计用户访问量:  incr userId_180903

​           

```c
struct sdshdr{
    // buf中已占用空间长度
    int len;
    // buf 中剩余可用空间长度
    int free;
    // 数据空间
    char buf[];
}
```

Hash:String元素组成的字典，适用于存储对象

​          hmset lilei name "lilei" age 26 title "Senior"

​          hget lilei age

​          hget lilei title

​          hset lilei title "Pricipal"

​          hget lilei title

List:列表,按照String元素插入顺序排序

​         lpush mylist aaa

​         lpush mylist bbb

​         lpush mylist ccc

​         lrange mylist 0 10     //ccc  bbb aaa

 Set:String元素组成的无序集合，通过哈希表实现，不允许重复

​        sadd myset 111

​        sadd myset 222

​        sadd myset 333

​        sadd myset 222 (0 添加失败)

​        smembers myset       //111   222   333 返回无序

​        共同关注，共同喜好，每个用户的关注人保存到一个set中，提供交叉并补功能

Sorted Set:通过分数来为集合中的成员进行从小到大的排序

​        zadd myzset 3 abc

​        zadd myzet 1 abd

​        zadd myzset 2 abb

​        zrangebyscore myzset 0 10     abd   abb   abc

​       带权重的消息队列

用于计数的HyperLoglog，用于支持存储地理信息的Geo

#### 底层数据类型基础

​     简单动态字符串

​     连表

​     字典

​     跳跃表

​     整数集合

​     压缩列表

​     对象

