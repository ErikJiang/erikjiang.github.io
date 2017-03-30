title: "Redis学习笔记"
date: 2016-01-05 14:12:24
categories: "技术" 
tags: [redis] 
---

Redis是一个开源、支持网络、基于内存、键值对存储数据库，使用ANSI C编写。Redis是时下最流行的键值对存储数据库。

历史：
> 2015年6月至今：Redis 的开发由Redis Labs赞助。

> 2013年5月至2015年6月期间：其开发由Pivotal赞助。

> 2013年5月之前：其开发由VMware赞助。

<!--more-->

---

###  一、如何安装 Redis ：

在 Linux、Unix、Mac 中源码编译的安装方式：

	$ wget http://download.redis.io/releases/redis-3.0.6.tar.gz
	$ tar xzf redis-3.0.6.tar.gz
	$ cd redis-3.0.6
	$ make
	
在 Mac 中可使用 Homebrew 管理工具进行安装：
	
	$ brew install redis
	

###  二、如何运行 Redis ：

首先需要运行 redis 服务端：

	$ redis-server
	
然后切换到另一终端命令行，运行客户端可执行程序：

	$ redis-cli
	
注意，redis默认使用 6379 端口，如果想指定其它端口：

	$ redis-cli --port 1234		//指定端口为：1234
	
然后可输入 `info` 命令查看所有信息，确认 redis 是否已被正确安装；

同时，还可以使用 redis 提供的 `ping` 命令来测试客户端与 redis 服务是否连接正常：

	redis 127.0.0.1:6379> PING
	PONG 
	
	
###  三、关于 Redis 的数据类型：

redis 存在五种数据类型，包括 string（字符串类型），hash（哈希类型），list（链表类型），set（无序集合类型）及 zset（有序集合类型）；

1. 字符串类型 [string] 是 Redis 中最基本的数据类型，也是其他 4 种数据类型的基础，它能存储任何形式的字符串，包括二进制数据。
2. 哈希类型 [hash] 是一个 string 类型的字段和字段值的映射表。哈希类型适合存储对象。相较于将对象的每个字段存成单个 string 类型，将一个对象存储在 hash 类型中会占用更少的内存，并且可以更方便的存取整个对象。
3. 链表类型 [list] 可以存储一个有序的字符串列表，内部是使用双向链表实现的。所以我们通常用链表类型的 LPUSH 和 LPOP 或者 RPUSH 和 RPOP 实现栈的功能，用 LPUSH 和 RPOP 或者 RPUSH 和 LPOP 实现队列的功能。
4. 集合类型 [set] 和数学中的集合概念相似，比如集合中的元素是唯一的、无序的，集合与集合之间可以进行交并差等操作。
5. 有序集合类型 [zset] 与集合类型一样，只不过多了个 “有序” ，有序集合中每一个元素都关联了一个分数，虽然集合中每个元素都是不同的，但是它们的分数却可以相同。
  
  
###  四、关于 Redis 的基本使用：
  
####  string［字符串类型］：

字符串类型的操作中，SET、GET 以及 INCR 是三个常用的命令。

用法：

* SET key value : 设置 key 的值为 value，成功时返回 OK。
* GET key : 获取 key 的值，成功时返回 key 的值。
* INCR key : 给 key 的值加 1，成功时返回更新后的 key 值。
* KEYS pattern : 查找当前所有符合给定模式 pattern 的 key（该命令适用于 redis 五种数据类型）


####  hash［哈希类型］：

哈希类型适合于存储对象。

用法：

* HSET key field value : 设置 key 的 field 字段值为 value，成功时返回 1，如果 key 中 field 字段已经存在且旧值已被新值覆盖，则返回 0 。
* HGET key field : 获取 key 的 field 字段的值，成功时返回 field 字段的值。
* HGETALL key : 获取 key 中所有的字段和其值，若 key 不存在，返回空。


####  list［链表类型］：

链表类型适合存储如社交网站的新鲜事，假如有一个存储了几千万个新鲜事的链表，获取头部或尾部的 10 条记录也是极快的。以下是链表类型常用的几个命令：

* LPUSH key value [value ...] : 往 key 链表左边添加元素，返回链表的长度。
* RPUSH key value [value ...] : 往 key 链表右边添加元素，返回链表的长度。
* LPOP key : 移除 key 链表左边第一个元素，并返回被移除元素的值。
* RPOP key : 移除 key 链表右边第一个元素，并返回被移除元素的值。
* LRANGE key start stop : 获取链表中某一片段，但不会修改原链表的值。另外，与很多语言中截取数组片段的 slice 方法不同的是 LRANGE 返回的值包含最右边的元素。LRANGE 命令也支持负索引，即 -1 表示最后一个元素，所以 LRANGE key 0 -1 可以获取链表 key 中所有的元素。如果 start 的位置比 stop 的位置靠后，则返回空，如果 stop 大于实际链表范围，则会返回到链表最后的元素。


####  set［集合类型］：

集合类型非常适合存储如文章的标签，这样我们在获取标签的时候就可以避免获取重复的标签了。以下是集合类型常用的几个命令：

* SADD key member [member ...] : 向集合中添加一个或多个元素，若要添加的元素集合中已经存在，则忽略这个元素。返回成功加入的元素数量（不包含忽略的）。
* SREM key member [member ...] : 从集合中删除一个或多个元素，返回成功移除的元素的数量。
* SMEMBERS key : 返回集合中所有元素。

集合类型还支持交集、差集和并集的运算，命令如下：

* SINTER key [key ...] : 多个集合执行交集运算，并返回运算结果。
* SDIFF key [key ...] : 多个集合执行差集运算，并返回运算结果。
* SUNION key [key ...] : 多个集合执行并集运算，并返回运算结果。


####  zset［有序集合类型］：

有序集合类型中每一个元素都关联了一个分数。这使得我们不仅可以进行大部分集合类型的操作，还可以通过分数获取指定分数范围内的元素等与分数有关的操作。有序集合类型适用于比如通过文章访问量排序的功能。常用命令如下：

* ZADD key score member [score member ...] ： 向有序集合中加入一个或多个元素和该元素的值，如果该元素已存在则用新的分数替换原来的分数。返回新加入的元素个数（不包含已存在的元素）。
* ZREM key member [member ...] ： 删除集合中一个或多个元素，返回成功删除的元素的个数（不包含本来就不存在的元素）。
* ZRANGE key start stop [WITHSCORES] ： 按元素分数从小到大顺序返回从 start 到 stop 之间所有的元素（同样包含两端的元素）。如果需要同时获得对应元素的分数的话，在尾部加上 WITHSCORES 参数。如果两个元素分数相同，则按照元素的字典序排列。
* ZREVRANGE key start stop [WITHSCORES] ： 同上，只不过 ZREVRANGE 是按照元素分数从大到小顺序给出结果的。

---
参考：

* [ZhiCun Github](https://github.com/island205 "zhicun")


