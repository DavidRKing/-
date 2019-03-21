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

### 从海量Key里查询出某一固定前缀的Key

#### 留意细节

​     摸清数据规模，即访问清楚边界 （养成好习惯）

​     Keys pattern:查找所有符合给定模式pattern的key

​     keys k1*(以k1开头的key，返回数据量非常大，对内存消耗比较多)

​     keys指令一次性返回所有匹配的key

​     键的数量过大会使服务卡顿

​     SCAN cursor [match pattern] [count count]

​        基于游标的迭代器，需要基于上一次的游标延续之前的迭代过程

​        以0作为游标开始一次新的迭代，直到命令返回游标0完成一次遍历

​        不保证每次执行都会返回某个给定数量的元素，支持模糊查询

​        一次返回的数量不可控，只能大概率符合count参数

​    scan 0 match k1* 10 (开始迭代返回k1开头的key，0第一次,返回不一定是10 个，redis服务不会卡顿)

   此命令会返回一个游标和结果集。

   1000000

   k1000001

   k2000002

   k3000003

  scan 1000000 match k1* 10

  .........

  直到返回0，过程可能返回重复的key，自己程序过滤，说明遍历结束。

### 如何通过redis实现分布式锁

   互斥性（只能由一个客户端持有）

   安全性（只能由持有该客户端锁的客户端删除）

   死锁 (如果持有锁的客户端挂掉，导致锁无法释放，其他客户端无法获取锁)

   容错(redis某些节点挂掉，不影响业务处理)

#### SETNX key value:如果key不存在，则创建并赋值

   时间复杂度：O(1)

   返回值：设置成功，返回1；设置失败：返回0

   eg:

​      setnx locknx test

​      1

​     setnx locknx task

####  如何解决SETNX长期有效的问题

####  EXPIRE key seconds

​     设置key的生存时间，当key过期时(生存时间为0)，会被自动删除

  expire locknx 2 (2 秒过期时间)

 

```java
RedisService redisService = SpringUtils.getBean(RedisService.class);
long status = redisService.setnx(key,"1");
if(status ==1){
    redisService.expire(key,expire);
    //执行独占资源逻辑
    doWork();
}
//如果sexnx时，系统挂掉，则key被一直占用。两个原子性的操作放在一起，就不是原子性了
```

####  SET key value [EX seconds] [PX milliseconds] [NX|XX]

​     EX second:设置键的过期时间为second秒

​     PX millisecond:设置键的过期时间millisecond毫秒

​     NX:只在键不存在时，才对键进行设置操作

​     XX:只在键已经存在时，才对键进行设置操作

​     SET操作完成时，返回OK，否则返回nil

​        eg: 

​            set locktarget 123423(请求id，线程id等) ex 10 nx      

   

```java
RedisService redisService = SpringUtils.getBean(RedisService.class);
String result = redisService.set(lockKey,requestId,SET_IF_NOT_EXIST,SET_WITH_EXPIRE_TIME,expireTime);
if(result.equals("OK")){
    //执行独占资源逻辑
    doWork();
}
```

#### 大量的key同时过期的注意事项

   集中过期，由于清除大量的key很耗时，会出现短暂的卡顿现象

​          解决方案：在设置key的过期时间的时候，给每个key加上随机值

#### 如何使用Redis做异步队列

​    使用List作为队列，RPUSH生产消息，LPOP消费消息

​    eg: 

​    rpush testlist aaa

​    rpush testlist bbb

​    rpush testlist ccc

​    lpop testlist

   缺点:没有等待队列里有值就直接消费。

   弥补:可以通过应用层引入Sleep机制去调用LPOP重试

   BLPOP key [key...] timeout:阻塞直到队列有消息或者超时

   eg:

​       blpop testlist 30 (如果超过30秒没有数据返回，则返回nil)

​       缺点:只能提供一个消费者消费

####  pub/sub主题订阅模式

​    发送者(pub)发送消息，订阅者(sub)接受消息 (一条消息，可以让多个消费者消费)

​    订阅者可以订阅任意数量的频道

   eg:

​     subscribe myTopic  (client1)

​     subscribe myTopic  (client2)

​     subscribe anotherTopic (client3)

​     publish myTopic "Hello"  (client1 client2 收到)

​     publish myTopic  "world" (client1 client2 收到)      

   缺点：

​      消息的发布是无状态的，无法保证可达。(生产者产生消息，消费者下线，重新上线，是接受不到这个消息的。请使用专业的消息队列工具)

### Redis如何做持久化

####     RDB(快照)持久化:保存某个时间点的全量数据快照。

​     vim redis.conf

​     save "" (禁用rdb)

​     save 900 1

​     save 300 10

​     save 60 10000  (60秒内，有10000次写入，则保存一次快照)

​     stop-writes-bgsave-error yes

​     rdbcompression yes (建议no)

​        save:阻塞Redis的服务进程，直到RDB文件被创建完毕

​        BGSAVE:Fork出一个子进程来创建RDB文件，不阻塞服务器进程(子进程处理完之后，发送信号给父进程即redis主进程，父进程通过轮询，接受子进程的信号)

​        lastSave:查看上一次save的时间

​       eg:

​            save

​           (服务器卡主)

​            ok

​           lastsave

​           1598989898(上次执行save的时间)

​           bgsave

​           (服务器未被卡顿)

​           lastsave

​         可以java写个定时任务，去bgsave 和 lastsave

#### 自动触发RDB持久化的方式      

​    根据redis.conf配置里的save m n定时触发(用的是BGSAVE)

​    主从辅助时，主节点自动触发

​    执行Debug reload

​    执行shutdown且没有开启AOF持久化

#### BGSAVE原理

​    BGSAVE 查看是否有其他正在执行BGSAVE的子进程，如果存在则报错。

​    触发持久化（调用 rdbSaveBackground）

​    系统调用fork():创建进程，实现了Copy-on-Write (写时复制)

​       如果有多个调用者同时要求相同的资源(如内存或磁盘上的数据存储),他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本给调用者，而其他调用者所见到的最初的资源仍然保持不变。

   缺点：

​      内存数据的全部同步，数据量大会由于I/O而严重影响性能。

​      可能会因为Redis挂掉而丢失从当前至最近一次快照期间的数据

#### AOF(append-only-file)持久化:保存写状态

​    记录下除了查询以外的所有变更数据库状态的指令

​    以append的形式追加保存到AOF文件中(增量)

​    appendonly yes

​    appendfilename "appendonly.aof"

​    appendfsync everysec(推荐)|always

#### 日志重写解决AOF文件大小不断增大的问题，原理如下:

   调用fork(),创建一个子进程，

   子进程把新的AOF写到一个临时文件里，不依赖原来的AOF文件

   主进程持续将新的变动同时写到内存和原来的AOF里

   主进程获取子进程重写AOF的完成信号，往新AOF同步增量变动

   使用新的AOF文件替换掉旧的AOF文件

   bgrewrite(手动触发)

#### RDB文件和AOF文件共存情况下的恢复流程

  redis启动判断是否存在aof文件，若存在，加载aof文件。

  若不存在，判断是否存在rdb文件，若存在，加载rdb文件。

  若不存在，正常启动。

#### RDB和AOF的优缺点

  rdb优点：全量数据快照，文件小，恢复快

  rdb缺点：无法保存最近一次快照之后的数据

  AOF优点：可读性高，适合保存增量数据，数据不易丢失。

  AOF缺点：文件体积大，恢复时间长。

#### RDB-AOF混合持久化方式

   BGSAVE做镜像全量持久化，AOF做增量持久化.(BGSAVE做全量，AOF做最近的增量，组合一起，就全了)

#### 使用Pipeline的好处

   

```bash
批量生成redis测试数据
1.Linux Bash下面执行
  for((i=1;i<=20000000;i++)); 
    do echo "set k$i v$i" >> /tmp/redisTest.txt ;
    done;
  生成2千万条redis批量设置kv的语句(key=kn,value=vn)写入到/tmp目录下的redisTest.txt文件中
2.用vim去掉行尾的^M符号，使用方式如下：：
  vim /tmp/redisTest.txt
    :set fileformat=dos #设置文件的格式，通过这句话去掉每行结尾的^M符号
    ::wq #保存退出
3.通过redis提供的管道--pipe形式，去跑redis，传入文件的指令批量灌数据，需要花10分钟左右
  cat /tmp/redisTest.txt | 路径/redis-5.0.0/src/redis-cli -h 主机ip -p 端口号 --pipe

```

   Pipeline和linux的管道类似

   Redis基于请求/响应模型，单个请求处理需要一一作答

   Pipeline批量执行指令，节省多次IO往返的时间

   有顺序依赖的指令建议分批发送

####  Redis的同步机制

   主从同步原理 或 从从同步

   一个Master，多个slave。(写Master，读salve，定期的数据备份，选择几个salve进行操作)

   第一次同步时，主节点做一次bgsave，并将后续的操作记录到内存的buffer中，操作完成后，将rdb文件，全量同步到从节点中。从节点接受完成后，将rdb加载到内存中，加载完成后，通知主节点，将这期间的增量数据，同步到从节点。

   全同步过程

   Salve发送sync命令到master

   master启动一个后台进程，将redis中的快照数据保存到文件中（bgsave）

   master将保存数据快照期间接收到的写命令缓存起来(增量数据缓存)

   Master完成写文件操作后，将该文件发送给salve

   使用新的AOF文件替换掉旧的AOF文件。

   Master将这期间收集的增量写命令发送给salve端，进行回放。

  

增量同步过程 （运行期间，同步过程?）

​     master接收到用户的操作命令，判断是否需要传播到slave   

​     将操作记录追加到AOF文件

​     将操作传播到其他salve:1、对齐主从库；2、往响应缓存写入指令。

​     将缓存中的数据发送给Slave

主从模式的弊端不具备高可用性，当mater挂掉之后，对外无法提供写入操作。

#### Redis Sentinel(本身是个独立进程，监控集群)

解决主从同步Master宕机的主从切换问题:

​     监控:检查主从服务器是否运行正常

​     提醒：通过API向管理员或者其他应用程序发送故障通知

​     自动故障迁移:主从切换 (客户端访问一个无效的master时，会返回新master的ip地址)

#### 留言协议Gossip

在杂乱无章中寻求一致

​     每个节点都随机地与对方通信，最终所有节点的状态达成一致.

​     种子节点定期随机向其他节点发送节点列表以及需要传播的消息。

​     不保证信息一定会传递给所有节点，但是最终会趋于一致

#### Redis的集群原理

如何从海量数据里快速找到所需？(类似分库， 分表。不同的key分散到不同的redis节点)

​     分片：按照某种规则去划分数据，分散存储在多个节点上 (降低单节点的压力) 

​     常规的按照哈希划分无法实现节点的动态增减. (之后，对服务器数量取模)



一致性哈希算法:对2^32取模,将哈希值空间组织成虚拟的圆环.

​         0到  2^32-1

将数据key使用相同的函数hash计算出哈希值。在圆环上顺时针寻找，第一个遇到的服务器，就是key应该存放的服务器.

如果此时某台机器宕机:则仅影响，此服务器到前一个服务器之间的数据.

如果增加服务器:则仅影响，此服务器到前一个服务器之间的数据.



hash环的数据倾斜问题

​    两台服务器。大部分数据通过hash计算之后，都存到A上。少量的数据存到了B上。造成数据分布不均匀。

引入虚拟节点解决数据倾斜问题

​     对每个服务器节点计算多个hash,每个计算结果位置都放一个虚拟节点，ip_编号. nodeA#A1 nodeA#2 nodeA#3  nodeB#1 nodeB#2 nodeB#3

   通常虚拟节点个数设置（32，或者更大）



通过redis主从同步，哨兵机制，和一致性hash算法，保证了redis的高可用性(主流应用做法)



















​     















