---
title: Redis 常见数据结构以及使用场景分析
date: 2021/10/11 21:45:48
update: 2021/10/11 21:45:48
categories:
- [Java, 学习笔记]
tags:
- Redis
- 面试

sticky: true
audio:
- https://music.163.com/#/song?id=1901371647

---

:::primary
Redis是面试中的最经常问到的数据库之一，在日常项目中我们为了快速访问到一些数据，首先想到的肯定就是Redis了。下面就来介绍一下Redis的功能，方便我们以后使用👩
:::

## Redis 常见数据结构以及使用场景分析
**redis支持丰富的数据类型，不同的场景使用合适的数据类型可以有效的优化内存数据的存放空间：🌴🌴**
> 1. **string：** 最基本的数据类型，二进制安全的字符串，最大512M。
> 2. **list：** 按照添加顺序保持顺序的字符串列表。
> 3. **hash：** key-value对的一种集合。
> 4. **set：** 无序的字符串集合，不存在重复的元素。
> 5. **sorted set：** 已排序的字符串集合。
> 6. **bitmap：** 更细化的一种操作，以bit为单位。
> 7. **hyperloglog：** 基于概率的数据结构。 # 2.8.9新增
> 8. **Geo:**  地理位置信息储存起来， 并对这些信息进行操作   # 3.2新增
> 9. **流（Stream）** # 5.0新增



<br>

#### String



1. **介绍 ：** string 数据结构是简单的 key-value 类型。虽然 Redis 是用 C 语言写的，但是 Redis 并没有使用 C 的字符串表示，而是自己构建了一种 **[简单动态字符串]{.blue}** (simple dynamic string，**SDS**)。相比于 C 的原生字符串，Redis 的 SDS 不光可以保存文本数据还可以保存二进制数据，并且获取字符串长度复杂度为 O(1)（C 字符串为 O(N)）,除此之外，Redis 的 SDS API 是安全的，不会造成缓冲区溢出。
2. **常用命令：** set,get,strlen,exists,decr,incr,setex 等等。
3. **应用场景：** 一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。



下面我们简单看看它的使用！😎
**普通字符串的基本操作：**

~~~shell
127.0.0.1:6379> set key value #设置 key-value 类型的值
OK
127.0.0.1:6379> get key # 根据 key 获得对应的 value
"value"
127.0.0.1:6379> exists key  # 判断某个 key 是否存在
(integer) 1
127.0.0.1:6379> strlen key # 返回 key 所储存的字符串值的长度。
(integer) 5
127.0.0.1:6379> del key # 删除某个 key 对应的值
(integer) 1
127.0.0.1:6379> get key
(nil)
~~~
**批量设置 :**
~~~shell
127.0.0.1:6379> mset key1 value1 key2 value2 # 批量设置 key-value 类型的值
OK
127.0.0.1:6379> mget key1 key2 # 批量获取多个 key 对应的 value
1) "value1"
2) "value2"
~~~
**计数器（字符串的内容为整数的时候可以使用）：**
~~~shell
127.0.0.1:6379> set number 1
OK
127.0.0.1:6379> incr number # 将 key 中储存的数字值增一
(integer) 2
127.0.0.1:6379> get number
"2"
127.0.0.1:6379> decr number # 将 key 中储存的数字值减一
(integer) 1
127.0.0.1:6379> get number
"1"
~~~
**过期（默认为永不过期）：**
~~~shell
127.0.0.1:6379> expire key  60 # 数据在 60s 后过期
(integer) 1
127.0.0.1:6379> setex key 60 value # 数据在 60s 后过期 (setex:[set] + [ex]pire)
OK
127.0.0.1:6379> ttl key # 查看数据还有多久过期
(integer) 56
~~~

<br>



#### List



1. **介绍 ：** list 即是 **链表**。链表是一种非常常见的数据结构，特点是易于数据元素的插入和删除并且可以灵活调整链表长度，但是链表的随机访问困难。许多高级编程语言都内置了链表的实现比如 Java 中的 LinkedList，但是 C 语言并没有实现链表，所以 Redis 实现了自己的链表数据结构。Redis 的 list 的实现为一个 **[双向链表]{.orange}**，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。
2. **常用命令:** rpush,lpop,lpush,rpop,lrange,llen 等。
3. **应用场景:** 发布与订阅或者说消息队列、慢查询。



下面我们简单看看它的使用！🍬

**通过 rpush/lpop 实现队列：**

~~~shell
127.0.0.1:6379> rpush myList value1 # 向 list 的头部（右边）添加元素
(integer) 1
127.0.0.1:6379> rpush myList value2 value3 # 向list的头部（最右边）添加多个元素
(integer) 3
127.0.0.1:6379> lpop myList # 将 list的尾部(最左边)元素取出
"value1"
127.0.0.1:6379> lrange myList 0 1 # 查看对应下标的list列表， 0 为 start,1为 end
1) "value2"
2) "value3"
127.0.0.1:6379> lrange myList 0 -1 # 查看列表中的所有元素，-1表示倒数第一
1) "value2"
2) "value3"
~~~

**通过 rpush/rpop 实现栈：**
~~~shell
127.0.0.1:6379> rpush myList2 value1 value2 value3
(integer) 3
127.0.0.1:6379> rpop myList2 # 将 list的头部(最右边)元素取出
"value3"
~~~
下边图方便大家理解：
![image.png](/assets/2021-10/r1.png)

**通过 lrange 查看对应下标范围的列表元素：**
~~~shell
127.0.0.1:6379> rpush myList value1 value2 value3
(integer) 3
127.0.0.1:6379> lrange myList 0 1 # 查看对应下标的list列表， 0 为 start,1为 end
1) "value1"
2) "value2"
127.0.0.1:6379> lrange myList 0 -1 # 查看列表中的所有元素，-1表示倒数第一
1) "value1"
2) "value2"
3) "value3"
~~~

通过 lrange 命令，你可以基于 list 实现分页查询，性能非常高！
**通过 llen 查看链表长度：**
~~~shell
127.0.0.1:6379> llen myList
(integer) 3
~~~



<br>

#### Hash



1. **介绍 ：** hash 类似于 JDK1.8 前的 HashMap，内部实现也差不多(数组 + 链表)。不过，Redis 的 hash 做了更多优化。另外，hash 是一个 string 类型的 field 和 value 的映射表，**[特别适合用于存储对象]{.pink}**，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。
2. **常用命令：** hset,hmset,hexists,hget,hgetall,hkeys,hvals 等。
3. **应用场景：**  系统中对象数据的存储。



下面我们简单看看它的使用！🎈

~~~shell
127.0.0.1:6379> hmset userInfoKey name "guide" description "dev" age "24"
OK
127.0.0.1:6379> hexists userInfoKey name # 查看 key 对应的 value中指定的字段是否存在。
(integer) 1
127.0.0.1:6379> hget userInfoKey name # 获取存储在哈希表中指定字段的值。
"guide"
127.0.0.1:6379> hget userInfoKey age
"24"
127.0.0.1:6379> hgetall userInfoKey # 获取在哈希表中指定 key 的所有字段和值
1) "name"
2) "guide"
3) "description"
4) "dev"
5) "age"
6) "24"
127.0.0.1:6379> hkeys userInfoKey # 获取 key 列表
1) "name"
2) "description"
3) "age"
127.0.0.1:6379> hvals userInfoKey # 获取 value 列表
1) "guide"
2) "dev"
3) "24"
127.0.0.1:6379> hset userInfoKey name "GuideGeGe" # 修改某个字段对应的值
127.0.0.1:6379> hget userInfoKey name
"GuideGeGe"
~~~



<br>

#### Set



1. **介绍 ：** set 类似于 Java 中的 HashSet 。Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序。当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作。比如：你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。
2. **常用命令：** sadd,spop,smembers,sismember,scard,sinterstore,sunion 等。
3. **应用场景:** 需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景



下面我们简单看看它的使用！🙇

~~~shell
127.0.0.1:6379> sadd mySet value1 value2 # 添加元素进去
(integer) 2
127.0.0.1:6379> sadd mySet value1 # 不允许有重复元素
(integer) 0
127.0.0.1:6379> smembers mySet # 查看 set 中所有的元素
1) "value1"
2) "value2"
127.0.0.1:6379> scard mySet # 查看 set 的长度
(integer) 2
127.0.0.1:6379> sismember mySet value1 # 检查某个元素是否存在set 中，只能接收单个元素
(integer) 1
127.0.0.1:6379> sadd mySet2 value2 value3
(integer) 2
127.0.0.1:6379> sinterstore mySet3 mySet mySet2 # 获取 mySet 和 mySet2 的交集并存放在 mySet3 中
(integer) 1
127.0.0.1:6379> smembers mySet3
1) "value2"
~~~

<br>



#### Sorted set



1. **介绍：** 和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，还可以通过 score 的范围来获取元素的列表。有点像是 Java 中 HashMap 和 TreeSet 的结合体。
2. **常用命令：** zadd,zcard,zscore,zrange,zrevrange,zrem 等。
3. **应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

常用命令展示：🍋
~~~shell
127.0.0.1:6379> zadd myZset 3.0 value1 # 添加元素到 sorted set 中 3.0 为权重
(integer) 1
127.0.0.1:6379> zadd myZset 2.0 value2 1.0 value3 # 一次添加多个元素
(integer) 2
127.0.0.1:6379> zcard myZset # 查看 sorted set 中的元素数量
(integer) 3
127.0.0.1:6379> zscore myZset value1 # 查看某个 value 的权重
"3"
127.0.0.1:6379> zrange  myZset 0 -1 # 顺序输出某个范围区间的元素，0 -1 表示输出所有元素
1) "value3"
2) "value2"
3) "value1"
127.0.0.1:6379> zrange  myZset 0 1 # 顺序输出某个范围区间的元素，0 为 start  1 为 stop
1) "value3"
2) "value2"
127.0.0.1:6379> zrevrange  myZset 0 1 # 逆序输出某个范围区间的元素，0 为 start  1 为 stop
1) "value1"
2) "value2"
~~~

<br>

#### Bitmap 位图



1. **介绍：**  bitmap 存储的是连续的二进制数字（0 和 1），通过 bitmap, 只需要一个 bit 位来表示某个元素对应的值或者状态，key 就是对应元素本身 。我们知道 8 个 bit 可以组成一个 byte，所以 bitmap 本身会极大的节省储存空间。
2. **常用命令：**  setbit 、getbit 、bitcount、bitop
3. **应用场景：**  适合需要保存状态信息（比如是否签到、是否登录...）并需要进一步对这些信息进行分析的场景。比如用户签到情况、活跃用户情况、用户行为统计（比如是否点赞过某个视频）。


~~~shell
# SETBIT 会返回之前位的值（默认是 0）这里会生成 7 个位
127.0.0.1:6379> setbit mykey 7 1
(integer) 0
127.0.0.1:6379> setbit mykey 7 0
(integer) 1
127.0.0.1:6379> getbit mykey 7
(integer) 0
127.0.0.1:6379> setbit mykey 6 1
(integer) 0
127.0.0.1:6379> setbit mykey 8 1
(integer) 0
# 通过 bitcount 统计被被设置为 1 的位的数量。
127.0.0.1:6379> bitcount mykey
(integer) 2
~~~

针对上面提到的一些场景，这里进行进一步说明。



**使用场景一📍：用户行为分析** 很多网站为了分析你的喜好，需要研究你点赞过的内容。
~~~shell
# 记录你喜欢过 001 号小姐姐
127.0.0.1:6379> setbit beauty_girl_001 uid 1
~~~


**使用场景二📍：统计活跃用户**

使用时间作为 key，然后用户 ID 为 offset，如果当日活跃过就设置为 1



那么我该如何计算某几天/月/年的活跃用户呢(暂且约定，统计时间内只要有一天在线就称为活跃)，有请下一个 redis 的命令
~~~shell
# 对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。
# BITOP 命令支持 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种参数
BITOP operation destkey key [key ...]
~~~
初始化数据：
~~~shell
127.0.0.1:6379> setbit 20210308 1 1
(integer) 0
127.0.0.1:6379> setbit 20210308 2 1
(integer) 0
127.0.0.1:6379> setbit 20210309 1 1
(integer) 0
~~~
统计 20210308~20210309 总活跃用户数: 1
~~~shell
127.0.0.1:6379> bitop and desk1 20210308 20210309
(integer) 1
127.0.0.1:6379> bitcount desk1
(integer) 1	
~~~
统计 20210308~20210309 在线活跃用户数: 2
~~~shell
127.0.0.1:6379> bitop or desk2 20210308 20210309
(integer) 1
127.0.0.1:6379> bitcount desk2
(integer) 2
~~~
**使用场景三📍：用户在线状态**

对于获取或者统计用户在线状态，使用 bitmap 是一个节约空间且效率又高的一种方法。

只需要一个 key，然后用户 ID 为 offset，如果在线就设置为 1，不在线就设置为 0。



<br>

#### HyperLogLog 基数统计

**Redis 的基数统计**，这个结构可以非常省内存的去统计各种计数，比如注册 IP 数、每日访问 IP 数、页面实时UV）、在线用户数等。但是它也有局限性，就是只能统计数量，而没办法去知道具体的内容是什么。

>当然用集合也可以解决这个问题。但是一个大型的网站，每天 IP 比如有 100 万，粗算一个 IP 消耗 15 字节，那么 100 万个 IP 就是 15M。而 HyperLogLog 在 Redis 中每个键占用的内容都是 12K，理论存储近似接近 2^64 个值，不管存储的内容是什么，它一个基于基数估算的算法，只能比较准确的估算出基数，可以使用少量固定的内存去存储并识别集合中的唯一元素。而且这个估算的基数并不一定准确，是一个带有 0.81% 标准错误的近似值。



**应用场景:✨**
HyperLogLog 主要的应用场景就是进行基数统计。这个问题的应用场景其实是十分广泛的。例如：对于 Google 主页面而言，同一个账户可能会访问 Google 主页面多次。于是，在诸多的访问流水中，如何计算出 Google 主页面每天被多少个不同的账户访问过就是一个重要的问题。那么对于 Google 这种访问量巨大的网页而言，其实统计出有十亿 的访问量或者十亿零十万的访问量其实是没有太多的区别的，因此，在这种业务场景下，为了节省成本，其实可以只计算出一个大概的值，而没有必要计算出精准的值。

这个数据结构的命令有三个：PFADD、PFCOUNT、PFMERGE
~~~shell
redis> PFADD databases "Redis" "MongoDB" "MySQL"
(integer) 1
 
redis> PFADD databases "Redis"  # Redis 已经存在，不必对估计数量进行更新
(integer) 0
 
redis> PFCOUNT databases
(integer) 3
~~~

<br>



#### Geo 地理位置

Redis 的 GEO 特性在 Redis 3.2 版本中推出， 这个功能可以将用户给定的地理位置信息储存起来， 并对这些信息进行操作。GEO的数据结构总共有六个命令：geoadd、geopos、geodist、georadius、georadiusbymember、gethash,**GEO使用的是国际通用坐标系WGS-84。**



常用指令：
>1. GEOADD: 添加地理位置
>2. GEOPOS：查询地理位置（经纬度），返回数组
>3. GEODIST：计算两位位置间的距离
>4. GEORADIUS：以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。
>5. GEORADIUSBYMEMBER：以给定的地理位置为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

~~~shell
127.0.0.1:6379> geoadd kcityGeo 116.405285 39.904989 "beijing"
(integer) 1
127.0.0.1:6379> geoadd kcityGeo 121.472644 31.231706 "shanghai"
(integer) 1 
127.0.0.1:6379> geodist kcityGeo beijing shanghai km
"1067.5980"
127.0.0.1:6379> geopos kcityGeo beijing
1) 1) "116.40528291463851929"
   2) "39.9049884229125027"
127.0.0.1:6379> geohash kcityGeo beijing
1) "wx4g0b7xrt0"
127.0.0.1:6379> georadiusbymember kcityGeo beijing 1200 km withdist withcoord asc count 5
1) 1) "beijing"
   2) "0.0000"
   3) 1) "116.40528291463851929"
      2) "39.9049884229125027"
2) 1) "shanghai"
   2) "1067.5980"
   3) 1) "121.47264629602432251"
      2) "31.23170490709807012"
~~~



<br>

#### Streams 流

**支持多播的可持久化的消息队列，用于实现发布订阅功能**，借鉴了 kafka 的设计。Redis Stream的结构有一个消息链表，将所有加入的消息都串起来，每个消息都有一个唯一的ID和对应的内容。消息是持久化的，Redis重启后，内容还在。🚀🚀🚀

![image.png](/assets/2021-10/r2.png)

每个Stream都有唯一的名称，它就是Redis的key，在我们首次使用xadd指令追加消息时自动创建。

>每个**Stream**都可以挂多个消费组，每个消费组会有个游标last_delivered_id在Stream数组之上往前移动，表示当前消费组已经消费到哪条消息了。每个消费组都有一个Stream内唯一的名称，消费组不会自动创建，它需要单独的指令xgroup create进行创建，需要指定从Stream的某个消息ID开始消费，这个ID用来初始化last_delivered_id变量。
>
>每个**消费组**(Consumer Group)的状态都是独立的，相互不受影响。也就是说同一份Stream内部的消息会被每个消费组都消费到。
>
>同一个消费组(Consumer Group)可以挂接多个消费者(Consumer)，这些消费者之间是竞争关系，任意一个消费者读取了消息都会使游标last_delivered_id往前移动。每个消费者者有一个组内唯一名称。
>
>**消费者**(Consumer)内部会有个状态变量pending_ids，它记录了当前已经被客户端读取的消息，但是还没有ack。如果客户端没有ack，这个变量里面的消息ID会越来越多，一旦某个消息被ack，它就开始减少。这个pending_ids变量在Redis官方被称之为PEL，也就是Pending Entries List，这是一个很核心的数据结构，它用来确保客户端至少消费了消息一次，而不会在网络传输的中途丢失了没处理。
>
>**消息ID：** 消息ID的形式是timestampInMillis-sequence，例如1527846880572-5，它表示当前的消息在毫米时间戳1527846880572时产生，并且是该毫秒内产生的第5条消息。消息ID可以由服务器自动生成，也可以由客户端自己指定，但是形式必须是整数-整数，而且必须是后面加入的消息的ID要大于前面的消息ID。
>
>**消息内容：** 消息内容就是键值对，形如hash结构的键值对，这没什么特别之处。

~~~shell
127.0.0.1:6379> XADD mystream * field1 value1 field2 value2 field3 value3
"1588491680862-0"
127.0.0.1:6379> XADD mystream * username lisi age 18
"1588491854070-0"
127.0.0.1:6379> xlen mystream
(integer) 2
127.0.0.1:6379> XADD mystream * username lisi age 18
"1588491861215-0"
127.0.0.1:6379> xrange mystream - + 
1) 1) "1588491680862-0"
   2) 1) "field1"
      2) "value1"
      3) "field2"
      4) "value2"
      5) "field3"
      6) "value3"
2) 1) "1588491854070-0"
   2) 1) "username"
      2) "lisi"
      3) "age"
      4) "18"
3) 1) "1588491861215-0"
   2) 1) "username"
      2) "lisi"
      3) "age"
      4) "18"
127.0.0.1:6379> xdel mystream 1588491854070-0 
(integer) 1
127.0.0.1:6379> xrange mystream - + 
1) 1) "1588491680862-0"
   2) 1) "field1"
      2) "value1"
      3) "field2"
      4) "value2"
      5) "field3"
      6) "value3"
2) 1) "1588491861215-0"
   2) 1) "username"
      2) "lisi"
      3) "age"
      4) "18"
127.0.0.1:6379> xlen mystream
(integer) 2
~~~



<br>
