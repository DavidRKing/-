### Linux知识考点

####  Linux的体系结构

   应用程序---shell、公共函数库--系统调用--内核



   体系结构主要分为用户态(用户上层活动)和内核态

   内核：本质是一段管理计算机硬件设备的程序(访问系统资源等，都需要内核提供api)

   系统调用:内核的访问接口，是一种能再简化的操作（如果你想完成内存分配等操作，需要使用多个系统调用）

   tips:windows下使用cigwin模拟linux指令

   公用函数库:系统调用的组合拳

  Shell:命令解释器，可编程  (充当用户界面角色) 

```shell
ls -lrt
which ls
/bin/ls
#使用/bin/ls文件，同时将参数-lrt传进去
```

​    ls   less more cat vi  vim

#### 如何查找特定文件

​    语法 find path [options] params

​    作用：在制定目录下查找文件 

​    

```shell
#在当前目录下递归查找文件
find -name "target3.java"
#全局搜索从根目录递归查找文件
find / -name "target3.java"
#在home目录中查找target开头的文件
find ~ -name "target*"
#忽略大小写
find ~ -iname "target*"
```

#### 常间问题:

   find ~ -name "target3.java":精确查找文件

   find ~ -name "target*":模糊查询

   find ~ -iname "target*"不区分大小写

   man find:更多关于find指令的使用说明

#### 检索文件内容

   grep

   语法： grep [option] patten file

   全称: Global Regular Expression Print

   作用: 查找文件里符合条件的字符串

```shell
#查找以target开头，并且包含moo的文件
grep "moo" target*
#从标准输入设备中检索内容包含haha的。此时后面没有写文件名称
grep "haha"
```

  管道命令操作符 |

  可将指令连接起来，前一个指令的输出作为后一个指令的输入。(错误输出不会连接)

  command1->stdout  |stdin command2->stdout | stdin command 3

```shell
find ~ -name "target*"
#两者效果一样.
#find ~当前用户目录下的所有文件。
#grep "target*":从标准输入中获取数据。
find ~ | grep "target*"
```

  注意事项:只处理前一个命令正确输出，不处理错误输出

​                  右边命令必须能够接受标准输入流，否则传递过程中数据会被抛弃

 常用接受数据管道命令:

sed,awk,grep,cut,head,top,less,more,wc,join,sort,split等。

```shell
#筛选日志中包含某些字段的行
grep 'partial\[true\]' xxxxxx.log
#我还想进一步筛选,匹配相关正则表达式的
grep 'partial\[true\]' xxxxxx.log|grep -o 'engin\[[0-9 a-z]*\]'
#engin[werwedsjfksljl123413241234]
#想查询Tomcat进程相关的信息
ps -ef | grep tomcat
#想过滤到自己查询tomcat的这条命令,过滤掉该命令本身,过滤掉包含相关字符串的内容
ps -ef | grep tomcat | grep -v "grep"

```



#### 对文件内容做统计

awk

​      语法:awk [options]  'cmd' file

​      一次读取一行文本，按输入分隔符进行切片，切成多个组成部分(默认用空格，将列分开)。

​     将切片直接保存在内建的变量中，$1,$2...($0表示行的全部，即整行)。

​     支持对单个切片的判断，支持循环判断，默认分隔符为空格。

```shell
cat netstat.txt
# proto recv-q xx xx               foreign address        state
# tcp  0  48 115.28.159.6:ssh  113.25.178.324:23432   establish
# tcp  0  48 115.28.159.6:ssh  113.25.178.324:23432   establish
# tcp  0  48 115.28.159.6:ssh  113.25.178.324:23432   establish

#打印第一列和第四列内容
awk '{print $1,$4}' netstat.txt
#筛选proto为tcp recv-q为1 的数据
awk '$1=="tcp" && $2=1 {print $0}' netstat.txt
#筛选proto为tcp recv-q为1 的数据并且加上表头，NR==1表示第一行
awk '($1=="tcp" && $2=1)||NR==1 {print $0}' netstat.txt
#按照“，”分割。-F表示以什么分隔符来分割
awk -F "," '{print $2}' test.txt
#根据上一个例子，统计次数。某个引擎 partial为true的次数
#数组下标存的是引擎名称，数组内容存储的是数量
grep 'partial\[true\]' xxxxxx.log|grep -o 'engin\[[0-9 a-z]*\]'
|awk '{enginearr[$1]++} END {for(i in enginearr) print i "\t" enginearr[i]}'
```

#### 批量替换文件内容

   sed

​     语法：sed [option] 'sed command' filename

​     全名stream editor,流编辑器

​     适合用于对文本内容进行处理



```shell
#将str替换为 String,此时仅仅输入到终端 -i在目标文本中修改
sed 's/^Str/String/' replace.java
#将文本文件内容也替换了
sed -i 's/^Str/String/' replace.java
#将结尾的.替换为；
sed -i 's/\.$/\;/' replace.java
#将 jack的名字替换掉 每行首次替换  /g全部替换
sed -i 's/Jack/me/g' replace.java
#将 空行删除
sed -i '/^ *&/d' replace.java
#将含有Integer的行删除
sed -i '/Integer/d' replace.java
```

面试要偷偷摸摸的

面试时间不要一味将就对方

提离职要谨慎(拿到offer在提)

好聚好散

跳槽时间衔接:一般15号之后离职，下个月15号前入职社保不会断

