Redis is an open-source in-memory database project implementing a distributed, in-memory key-value store with optional durability. 

# 1   安装

在如下地址下载redis版本：https://redis.io/download，
解压后，安装。如下：

    tar xzf redis-4.0.2.tar.gz
    $ cd redis-4.0.2
    $ make

然后运行

    $ src/redis-server  即可启动redis 服务，命令后面可以指定特定的redis.conf配置文件
    $ src/redis-cli 即可启动redis客户端，

也可以使用$ make install，执行完之后在任意目录下执行redis-server命令即可启动redis服务。
如果需要自定义redis服务的话，请先修改redis.conf的配置项。

# 2 关于数据库

redis.conf文件里关于数据库的说明和配置如下：

    # Set the number of databases. The default database is DB 0, you can select
    # a different one on a per-connection basis using SELECT <dbid> where
    # dbid is a number between 0 and 'databases'-1
    databases 16

即默认有编号为0-15的16个数据库，可以用select命令切换数据库。

# 3 关于数据模型

redis五种数据结构

（1）String            字符串 
（2）Hash             字典 
（3）List               列表 
（4）Se                集合 
（5）Sorted Set   有序集合

## String

String 数据结构是简单的 key-value 类型，value 不仅可以是 String，也可以是数字（当数字类型用 Long 可以表示的时候encoding 就是整型，其他都存储在 sdshdr 当做字符串）。使用 Strings 类型，可以完全实现目前 Memcached 的功能，并且效率更高。还可以享受 Redis 的定时持久化（可以选择 RDB 模式或者 AOF 模式），操作日志及 Replication 等功能。除了提供与 Memcached 一样的 get、set、incr、decr 等操作外。

## Hash

在 Memcached 中，我们经常将一些结构化的信息打包成 hashmap，在客户端序列化后存储为一个字符串的值（一般是 JSON 格式），比如用户的昵称、年龄、性别、积分等。这时候在需要修改其中某一项时，通常需要将字符串（JSON）取出来，然后进行反序列化，修改某一项的值，再序列化成字符串（JSON）存储回去。简单修改一个属性就干这么多事情，消耗必定是很大的，也不适用于一些可能并发操作的场合（比如两个并发的操作都需要修改积分）。而 Redis 的 Hash 结构可以使你像在数据库中 Update 一个属性一样只修改某一项属性值。
存储、读取、修改用户属性

## List

List 说白了就是链表（redis 使用双端链表实现的 List），相信学过数据结构知识的人都应该能理解其结构。使用 List 结构，我们可以轻松地实现最新消息排行等功能（比如新浪微博的 TimeLine ）。List 的另一个应用就是消息队列，可以利用 List 的 *PUSH 操作，将任务存在 List 中，然后工作线程再用 POP 操作将任务取出进行执行。Redis 还提供了操作 List 中某一段元素的 API，你可以直接查询，删除 List 中某一段的元素。
微博 TimeLine 
消息队列

## Set

Set 就是一个集合，集合的概念就是一堆不重复值的组合。利用 Redis 提供的 Set 数据结构，可以存储一些集合性的数据。比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。因为 Redis 非常人性化的为集合提供了求交集、并集、差集等操作，那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。 共同好友、二度好友 利用唯一性，可以统计访问网站的所有独立 IP 
好友推荐的时候，根据 tag 求交集，大于某个 threshold 就可以推荐

## Sorted Set

和Sets相比，Sorted Sets是将 Set 中的元素增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，比如一个存储全班同学成绩的 Sorted Sets，其集合 value 可以是同学的学号，而 score 就可以是其考试得分，这样在数据插入集合的时候，就已经进行了天然的排序。另外还可以用 Sorted Sets 来做带权重的队列，比如普通消息的 score 为1，重要消息的 score 为2，然后工作线程可以选择按 score 的倒序来获取工作任务。让重要的任务优先执行。 带有权重的元素，比如一个游戏的用户得分排行榜 
比较复杂的数据结构，一般用到的场景不算太多


# Redis键值设计

http://blog.csdn.net/javastart/article/details/41649449

# Redis命令参考手册

http://redisdoc.com/

# 一致性hash算法介绍（consistent hashing）

http://www.jianshu.com/p/e8fb89bb3a61
https://yq.aliyun.com/articles/241268

