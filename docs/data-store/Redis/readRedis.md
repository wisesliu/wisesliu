![5b557a0f2856b.jpg](http://ww1.sinaimg.cn/large/9b9f09a9ly1g9ypjztws7j20pl0cdmyu.jpg)

#### Redis简介

Redis:**REmote DIctionary Server**(远程字典服务器)。

Redis 是一个全开源免费（BSD许可）的，内存中的数据结构存储系统，它可以用作**数据库、缓存和消息中间件**。一般作为一个高性能的(key/value)分布式内存数据库，基于**内存**运行并支持持久化的NoSQL数据库，是当前最热门的NoSql数据库之一,也被人们称为**数据结构服务器**

<br>

#### Redis 介绍

​        redis是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库。

​        redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括**string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）****。**这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步，所以Redis也可以被看成是一个数据结构服务器。

​        **Redis 是一个高性能的key-value数据库**。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。

​        Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

​        redis的官网地址，非常好记，是redis.io。（域名后缀io属于国家域名，是british Indian Ocean territory，即英属印度洋领地）目前，Vmware在资助着redis项目的开发和维护

​        Redis的所有数据都是保存在内存中，然后不定期的通过异步方式保存到磁盘上(这称为“半持久化模式”)；也可以把每一次数据变化都写入到一个append only file(aof)里面(这称为“全持久化模式”)。这就是redis提供的两种持久化的方式，RDB（Redis DataBase）和AOF（Append Only File）。

<br>

#### Redis 特点

Redis是一个开源，先进的key-value存储，并用于构建高性能，可扩展的Web应用程序的完美解决方案。

Redis从它的许多竞争继承来的三个主要特点：

- Redis数据库完全在内存中，使用磁盘仅用于持久性。

- 相比许多键值数据存储，Redis拥有一套较为丰富的数据类型。

- Redis可以将数据复制到任意数量的从服务器。

  <br>

#### Redis 优势

- 异常快速：Redis的速度非常快，每秒能执行约11万集合，每秒约81000+条记录。SET操作每秒钟 110000 次，GET操作每秒钟 81000 次，网站一般使用Redis作为**缓存服务器**。
- 支持**丰富的数据类型**：Redis支持大多数开发人员已经知道像列表，集合，有序集合，散列数据类型。这使得它非常容易解决各种各样的问题，因为我们知道哪些问题是可以处理通过它的数据类型更好。
- 操作都是**原子性**：所有Redis操作是原子的，这保证了如果两个客户端同时访问的Redis服务器将获得更新后的值。
- MultiUtility工具：Redis是一个多功能实用工具，可以在很多如：缓存，消息传递队列中使用（Redis原生支持发布/订阅），在应用程序中，如：Web应用程序会话，网站页面点击数等任何短暂的数据；

<br>

#### redis使用场景

- 取最新N个数据的操作
- 排行榜应用,取TOP N 操作
- 需要精确设定过期时间的应用
- 定时器、计数器应用
- Uniq操作,获取某段时间所有数据排重值
- 实时系统,反垃圾系统
- Pub/Sub构建实时消息系统
- 构建队列系统
- 缓存



**具体以某一论坛为例：**

- 记录帖子的点赞数、评论数和点击数 (hash)。
-  记录用户的帖子 ID 列表 (排序)，便于快速显示用户的帖子列表 (zset)。 
- 记录帖子的标题、摘要、作者和封面信息，用于列表页展示 (hash)。 
- 记录帖子的点赞用户 ID 列表，评论 ID 列表，用于显示和去重计数 (zset)。 
- 缓存近期热帖内容 (帖子内容空间占用比较大)，减少数据库压力 (hash)。 
- 记录帖子的相关文章 ID，根据内容推荐相关帖子 (list)。 
- 如果帖子 ID 是整数自增的，可以使用 Redis 来分配帖子 ID(计数器)。 
- 收藏集和帖子之间的关系 (zset)。 
- 记录热榜帖子 ID 列表，总热榜和分类热榜 (zset)。 
- 缓存用户行为历史，进行恶意行为过滤 (zset,hash)。

<br>

**安装**

```
$ wget http://download.redis.io/releases/redis-5.0.6.tar.gz
$ tar xzf redis-5.0.6.tar.gz
$ cd redis-5.0.6
$ make
```

新版本的编译文件在src中（之前在bin目录），启动server

```
$ src/redis-server
```

启动客户端

```
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```



