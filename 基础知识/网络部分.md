#### 网络部分

一、 OSI七层模型  概念参考模型，概念性框架

1. 物理层
2. 数据链路层
3. 网络层  IP协议
4. 传输层  TCP/UDP
5. 会话层
6. 表示层
7. 应用层  (http协议)

二、OSI的实现 TCP/IP

  

| OSI七层模型 | TCP/IP概念模型 | 功能                                   | TCP/IP协议族                 |
| ----------- | -------------- | -------------------------------------- | ---------------------------- |
| 应用层      | 应用层         | 文件传输，电子邮件，文件服务，虚拟终端 | TFTP,HTTP,FTP,SMTPDNS,Telnet |
| 表示层      |                | 数据格式化，代码转换，数据加密         |                              |
| 会话层      |                | 删除或建立与别的接点的联系             |                              |
| 传输层      | 传输层         | 提供端对端接口                         | TCP,UDP                      |
| 网络层      | 网络层         | 为数据包选择路由                       | IP,ICMP,RIP,OSPF,BCP,IGMP    |
| 数据链路层  | 链路层         | 传输有地址的帧以及错误检测功能         | SLIP,CSLIP,PPP,ARP,RARP,MTU  |
| 物理层      |                | 以二进制数据形式在物理媒体上传输       | ISO2110,IEEE802,IEEE802.2    |

先自上而下，后自下而上处理数据头部。

HTTP报文-->TCP首部-->IP首部-->以太网首部

三、TCP的三次握手

传输控制协议TCP简介

   面向连接的、可靠地、基于字节流的传输通信协议。

   将应用层的数据流分割成报文段并发送给目标节点的TCP层。

   数据包都有序号，对方收到则发送ACK确认，未收到则重传。

   使用校验和来校验数据在传输过程中是否有误。

TCP报文头

​       Source Port                  DestinationPort

​       Sequence Number(序列号，可以让数据有序传输，如果一段数据被分多分，则可以保证有序)

​       Acknowlegment Number(期望收到下一个报文的第一个数据字节的序号) B收到A发来的报文 seq=301  size=200    301-500已经正确收到 (301+200-1)

​      则，ack = 501，表示B收到下一个发过来的报文从501开始

​       TCP flags

​       URG:紧急指针标志

​       ACK:确认序号标志  1：ACK有效  0.无效

​       PSH:push标识

​       RST:重置连接标志

​       SYN:同步序列号,用于建立连接过程

​       FIN:finish标志，用于释放连接

说说TCP的三次握手

"握手"是为了建立连接，TCP三次握手

A                      B

CLOSED      SYN=1，seq=x        CLOSED

SYN-SEND        SYN=1,ACK=1,seq=y,ack=x+1                            SYN-RCVD

ESTABLISHED          ACK=1,seq=x+1,ack=y+1                       ESTABLISHED

 数据传输

在TCP/IP协议中，TCP协议提供可靠地连接服务，采用三次握手建立一个连接。

第一次握手：建立连接时，客户端发送SYN包（syn=j）到服务器，并进入SYN_SEND状态，等待服务器确认；

第二次握手：服务器收到SYN包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包(syn=k),即SYN+ACK包，此时服务器进入SYN_RECV状态。

第三次握手:客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1),此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手.



为什么需要三次握手才能建立起连接？

为了初始化Sequence Number的初始值 (服务器端 和 客户端 互相通知自己的Sequence Number  x 和 y 。这个序号为了以后数据传输用)



首次握手的隐患----SYN超时

 Server收到Client的SYN，回复 SYN-ACK的时候未收到ACK确认。 （Client掉线）

 Server不断重试直到超时，Linux默认等待63秒才断开连接1+2+4+8+16



针对SYN FLOOD的防护措施

​     SYN队列满后，通过tcp_syncookies参数回发SYN Cookie

​     若为正常连接则Client会回发SYN Cookie，直接建立连接

建立连接，Client出现故障怎么办？

​     保活机制

​      向对方发发送保活探测报文，如果未收到响应则继续发送

​      尝试次数达到保活探测数仍未收到响应则中断连接。



谈谈TCP的四次挥手

客户端A                         服务器B

ESTABLISHD                 ESTABLISHD

FIN_WAIT_1    FIN=1 seq=u    

​                        ACK=1 seq=v, ack=u+1   CLOSE-WAIT

FIN_WAIT_2   FIN=1 ,ACK=1 ,seq=w ,ack=u+1  last_ack

TIME_WAIT  ACK=1 ,seq=u+1,ack=w+1   close

等待2MSL

close

服务器的结束时间，要比客户端稍微早一些。

总结如下：

TCP采用四次挥手来释放连接

第一次挥手：Client发送一个FIN,用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态；

第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN

占用一个序号），Server进入CLOSE_WAIT状态；

第三次挥手：Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态；

第四次挥手:Client收到FIN后，Client进入TIME_WAIT状态，接受发送一个ACK给Server,确认序号为收到序号+1

Server进入CLOSED状态，完成四次挥手。

为什么会有TIME_WAIT状态?

1.确保有足够的时间让对方收到ACK包。

2.避免新旧连接混淆。



为什么需要四次握手才能断开连接

因为全双工，发送方和接收方都需要FIN报文和ACK报文



服务器出现大量CLOSE_WAIT状态的原因？

对方关闭socket连接，我放忙于度或者写，没有及时关闭连接。

1.检查代码，特别是释放资源代码。

2.检查配置，特别是处理请求的线程配置。



netstat -n 与 AWK

too many open files



UDP简介

UDP报文结构

Source Port        Destination Port

Length                 Checksum

data octets...(optional)



UDP的特点

面向非连接

不维护连接状态，支持同时向多个客户端传输相同的消息。

数据包报头只有8个字节，额外开销较小。

吞吐量只受限于数据生成速率、传输速率以及机器性能。

尽最大努力交付，不保证可靠交付，不需要维持复杂的连接状态表。

面向报文，不对应用程序提交的报文信息进行拆分或者合并。



TCP和UDP的区别 (OSI 传输层)

结论：

   面向连接 vs 无连接（单点vs多点）

   可靠性  

   有序性 （seq到达可能无序，但是最终有序）

   速度

   量级（头部 20字节 8字节）

TCP的滑动窗口

RTT 和 RTO

​    rtt：发送一个数据包到收到对应的ACK，所花费的时间。

​    rto:重传时间间隔 (重传定时器。没有返回，则重传)



TCP使用滑动窗口做流量控制与乱序重拍

   保证TCP的可靠性

   保证TCP的流控特性



窗口数据的计算过程

AdvertisedWindow = MaxRcvBuffer-(lastByteRevd-lastByteRead)

EffectiveWindow=AdvertisedWindow-(LastByteSend-LastByteAcked)



HTTP简介

超文本传输协议HTTP主要特点

   支持客户/服务器模式

   简单快速

   灵活

   无连接

   无状态

HTTP请求报文 和响应报文

请求方法   空格   URL    空格   协议版本  回车符  换行符 请求行

头部字段名：      值      回车符   换行符



请求/响应步骤

   客户端连接到WEB服务器

   发送http请求

   服务器接收到请求并返回HTTP响应 

   释放连接TCP连接

   客户端浏览器解析HTML内容

在浏览器地址键入URL，按下回车之后经历的流程

答案

​       DNS解析  找到IP地址

​       TCP连接   （三次握手）

​       发送HTTP请求

​       服务器处理请求并返回HTTP报文

​       浏览器解析渲染页面

​       连接结束   （四次挥手）   后两部 差不多 同时发生

HTTP状态码

​      1XX:提示信息-表示请求已接收，继续处理

​      2XX:成功--表示请求已被成功接受、理解、

​      3XX:重定向--要完成请求必须尽心更近一步的操作

​      4XX:客户端错误--请求有语法错误或请求无法实现

​      5XX:服务端错误-服务器未能实现合法的请求

常见状态码：

​     200 OK

​     400 bad request：客户单请求有语法错误，不能被服务器所理解

​     401 Unauthorized：请求未经授权

​      403 Forbidden ： 服务器收到请求，但是拒绝提供服务

​     404 Not Found ： 请求资源不存在, eg,输入了错误的URL

​    500 Internal Server Error： 服务器发生不可预期的错误

  503 Server Unavailable：服务器当前不能处理客户单请求，一段时间后可能恢复请求。



Get请求和Post请求的区别

从三个层面来回答

   Http报文层面：Get将请求信息放在URL，POST放在报文体中。Get请求有URL限制（一般是浏览器做限制）

   数据库层面：Get符合幂等性和安全性，POST不符合

   其他层面: Get可以被缓存，被存储（添加书签） CDN缓存，而POST不行 



Cookie和Session的区别

   Cookie简介：

​         是由服务器发给客户端的特殊信息，以文本的形式存放在客户端

​         客户端再次请求的时候，会把Cookie回发

​         服务器接收到后，会解析Cookie生成与客户端相对应的内容

Cookie的设置以及发送的过程

​        1.Web Client     http request       Web server

​        2.Web Server   Http Response + Set-Cookie    Web Client

​        3.Web Client    http Request +Cookie      Web Server

​        4.Web Server    Http Response       Web Client

Session介绍

​      1.服务器端的机制，在服务器上保存信息。（类似 散列表   sessionID，session对象）

​      2.解析客户端请求并操作session id，按需保存状态信息 （看 客户端是否包含session id 并且能找到）

​     

Session的实现方式

​        使用Cookie实现     Set-Cookie:JSESSIONID=XXXXXXX   客户端访问服务器的时候，会带着JSESSIONID

​         使用URL回写  服务器返回给客户单的页面都带着 JSESSIONID参数

Cookie 和Session的区别

1.Cookie数据存放在客户的浏览器上，Session数据放在服务器上。

2.Session相对于Cookie更安全

3.若考虑减轻服务器负担，应当使用Cookie



HTTP和HTTPS的区别

HTTPS简介

HTTP             HTTPS

HTTP            HTTP

TCP              SSL OR  TLS

IP                 TCP

​                     IP

SSL 安全套阶层

   为网络通信提供安全及数据完整性的一种安全协议。

   是操作系统对外的API，SSL3.0后更名为TLS

   采用身份验证和数据加密保证网络通信的安全和数据的完整性。

加密方式

​       对称加密：加密和解密使用同一种秘钥

​       非对称加密：加密使用的秘钥和解密使用的秘钥是不相同的。

​        哈希算法：将任意长度的信息转化为固定长度的值，算法不可逆。

​        数字签名：证明某个消息或者文件是某人发出/认同的。

HTTPS数据传输流程

​       浏览器将支持的加密算法信息发送给服务器。

​       服务器选择一套浏览器支持的加密算法，以证书的形式回发浏览器。

​       浏览器验证证书合法性，并结合证书公钥加密信息发送给服务器。

​       服务器使用私钥解密信息，验证哈希，加密响应消息回发浏览器。

​      浏览器解密响应信息，并对消息进行验真，之后进行加密交互数据。

区别如下：

HTTPS需要到CA申请证书，HTTP不需要

HTTPS密文传输，HTTP明文传输

连接方式不同，HTTPS默认使用443端口，HTTP使用80端口

HTTPS=http+加密+认证+完整保护，较http安全。



真的安全吗？

浏览器默认填充http:// 请求需要进行转化，有被劫持的风险。

可以使用htps转化







   

   



























