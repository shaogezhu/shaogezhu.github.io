---
title: MySQL常见的锁分析
date: 2021-11-21 16:43:00
update: 2021-11-21
categories:
- [Java, 学习笔记]
tags:
- MySQL
- 面试

---

:::info
🌴🌴数据库锁定机制简单来说，就是数据库**为了保证数据的一致性**，而使各种共享资源在被并发访问变得有序所设计的一种规则。对于任何一种数据库来说都需要有相应的锁定机制，所以MySQL自然也不能例外。下面就给大家详细介绍一下
:::


## MySQL的锁

### 概述

🌻🌻MySQL数据库由于其自身架构的特点，存在多种数据存储引擎，每种存储引擎所针对的应用场景特点都不太一样，为了满足各自特定应用场景的需求，每种存储引擎的锁定机制都是为各自所面对的特定场景而优化设计，所以各存储引擎的锁定机制也有较大区别。本文没有说明的情况下默认使用的是Innodb引擎。



#### Innodb原理(简单说一下)： 
- innodb一定存在聚簇索引，默认以主键作为聚簇索引🚀
- 有几个索引，就有几棵B+树(不考虑hash索引的情形)🚀
- 聚簇索引的叶子节点为磁盘上的真实数据。非聚簇索引的叶子节点还是索引，指向聚簇索引B+树。🚀

## 锁的分类
- **共享锁(S锁)🔒:** 假设事务T1对数据A加上共享锁，那么事务T2可以读数据A，不能修改数据A。
- **排他锁(X锁)🔒:** 假设事务T1对数据A加上排他锁，那么事务T2不能读数据A，不能修改数据A。
   我们通过```update```、```delete```等语句加上的锁都是行级别的锁。只有```LOCK TABLE … READ```和```LOCK TABLE … WRITE```才能申请表级别的锁。
- **意向共享锁(IS锁)🔒:** 一个事务在获取（任何一行/或者全表）S锁之前，一定会先在所在的表上加IS锁。
- **意向排他锁(IX锁)🔒:** 一个事务在获取（任何一行/或者全表）X锁之前，一定会先在所在的表上加IX锁。

### 意向锁存在的目的

> 这里说一下意向锁存在的目的。假设事务T1，用X锁来锁住了表上的几条记录，那么此时表上存在IX锁，即意向排他锁。那么此时事务T2要进行```LOCK TABLE … WRITE```的表级别锁的请求，可以直接根据意向锁是否存在而判断是否有锁冲突。🎈

### 加锁算法
- **Record Locks：** 简单翻译为**行锁**。注意了，该锁是对索引记录进行加锁！锁是加在索引上而不是行上的。注意了，innodb一定存在聚簇索引，因此行锁最终都会落到聚簇索引上！
- **Gap Locks：** 简单翻译为**间隙锁**，是对索引的间隙加锁，其目的只有一个，防止其他事物插入数据。在```Read Committed```隔离级别下，不会使用间隙锁。这里我对官网补充一下，隔离级别比```Read Committed```低的情况下，也不会使用间隙锁，如隔离级别为```Read Uncommited```时，也不存在间隙锁。当隔离级别为```Repeatable Read```和```Serializable```时，就会存在间隙锁。
- **Next-Key Locks：** 这个理解为Record Lock+索引前面的Gap Lock。记住了，锁住的是索引前面的间隙！比如一个索引包含值，10，11，13和20。那么，间隙锁的范围如下
```java
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
```
	
还有不懂的地方可以看一下MySQL的官方文档：https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html


### 快照读和当前读
>在mysql中select分为快照读和当前读，执行下面的语句
>```select * from table where id = ?; ```
>执行的是快照读，读的是数据库记录的快照版本，是不加锁的。（这种说法在隔离级别为Serializable中不成立，后面会再补充。）


>那么，执行
```select * from table where id = ? lock in share mode; ```
会对读取记录加S锁 (共享锁)，执行	
```select * from table where id = ? for update```
会对读取记录加X锁 (排他锁)，那么加的是表锁还是行锁呢？

### 表锁or行锁
🍦针对这点，我们先回忆一下事务的四个隔离级别，他们由弱到强如下所示:
- **Read Uncommited(RU)：** 读未提交，一个事务可以读到另一个事务未提交的数据！
- **Read Committed (RC)：** 读已提交，一个事务可以读到另一个事务已提交的数据!
- **Repeatable Read (RR):** 可重复读，加入间隙锁，一定程度上避免了幻读的产生！注意了，只是一定程度上，并没有完全避免!我会在下一篇文章说明!另外就是记住从该级别才开始加入间隙锁(这句话记下来，后面有用到)!
- **Serializable：** 串行化，该级别下读写串行化，且所有的```select```语句后都自动加上```lock in share mode```，即使用了共享锁。因此在该隔离级别下，使用的是当前读，而不是快照读。 

那么关于是表锁还是行锁，大家可以看到网上最流传的一个说法是这样的
> InnoDB行锁是通过给索引上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。 InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！

🍋这句话大家可以搜一下，都是你抄我的，我抄你的。那么，这句话本身有两处错误！
**错误一:** 并不是用表锁来实现锁表的操作，而是利用了```Next-Key Locks```，也可以理解为是用了行锁+间隙锁来实现锁表的操作!

为了便于说明，我来个例子，假设有表数据如下，pId为主键索引

|pId(int)|name(varchar)|num(int)|
|-------|-------|-------|
|1|aaa|100|
|2|bbb|200|
|7|ccc|200|

执行语句(name列无索引)
```sql
select * from table where name = `aaa` for update
```
那么此时在pId=1,2,7这三条记录上存在行锁(把行锁住了)。另外，在(-∞,1)(1,2)(2,7)(7,+∞)上存在间隙锁(把间隙锁住了)。因此，给人一种整个表锁住的错觉！

**ps:** 对该结论有疑问的，可自行执行```show engine innodb status```;语句进行分析。

**错误二:** 所有文章都不提隔离级别！
注意上面说的，之所以能够锁表，是通过**行锁+间隙锁**来实现的。那么，RU和RC都不存在间隙锁，这种说法在RU和RC中还能成立么？
因此，该说法只在RR和Serializable中是成立的。如果隔离级别为RU和RC，无论条件列上是否有索引，都不会锁表，只锁行！

### 分析
🍹下面来对开始的问题作出解答，假设有表如下，pId为主键索引

|pId(int)|name(varchar)|num(int)|
|-------|-------|-------|
|1|aaa|100|
|2|bbb|200|
|3|bbb|300|
|7|ccc|200|

**🍍🍍RC/RU+条件列非索引**
1. ```select * from table where num = 200```
不加任何锁，是快照读。
2. ```select * from table where num > 200```
不加任何锁，是快照读。
3. ```select * from table where num = 200 lock in share mode```
当num = 200，有两条记录。这两条记录对应的pId=2，7，因此在pId=2，7的聚簇索引上加行级S锁，采用当前读。
4. ```select * from table where num > 200 lock in share mode```
当num > 200，有一条记录。这条记录对应的pId=3，因此在pId=3的聚簇索引上加上行级S锁，采用当前读。
5. ```select * from table where num = 200 for update```
当num = 200，有两条记录。这两条记录对应的pId=2，7，因此在pId=2，7的聚簇索引上加行级X锁，采用当前读。
6. ```select * from table where num > 200 for update```
当num > 200，有一条记录。这条记录对应的pId=3，因此在pId=3的聚簇索引上加上行级X锁，采用当前读。

**🍉🍉RC/RU+条件列是聚簇索引**
恩，大家应该知道pId是主键列，因此pId用的就是聚簇索引。此情况其实和RC/RU+条件列非索引情况是类似的。
1. ```select * from table where pId = 2```
不加任何锁，是快照读。
2. ```select * from table where pId > 2```
不加任何锁，是快照读。
3. ```select * from table where pId = 2 lock in share mode```
在pId=2的聚簇索引上，加S锁，为当前读。
4. ```select * from table where pId > 2 lock in share mode```
在pId=3，7的聚簇索引上，加S锁，为当前读。
5. ```select * from table where pId = 2 for update```
在pId=2的聚簇索引上，加X锁，为当前读。
6. ```select * from table where pId > 2 for update```
在pId=3，7的聚簇索引上，加X锁，为当前读。

### 为什么条件列加不加索引，加锁情况是一样的？
> 其实是不一样的。在RC/RU隔离级别中，MySQL Server做了优化。在条件列没有索引的情况下，尽管通过聚簇索引来扫描全表，进行全表加锁。但是，MySQL Server层会进行过滤并把不符合条件的锁当即释放掉，因此你看起来最终结果是一样的。但是RC/RU+条件列非索引比本例多了一个释放不符合条件的锁的过程！

**🍍🍍RC/RU+条件列是非聚簇索引**
👉🏻我们在num列上建上非唯一索引。此时有一棵聚簇索引(主键索引，pId)形成的B+索引树，其叶子节点为硬盘上的真实数据。以及另一棵非聚簇索引(非唯一索引，num)形成的B+索引树，其叶子节点依然为索引节点，保存了num列的字段值，和对应的聚簇索引。

接下来分析开始👀
1. ```select * from table where num = 200```
不加任何锁，是快照读。
2. ```select * from table where num > 200```
不加任何锁，是快照读。
3. ```select * from table where num = 200 lock in share mode```
当num = 200，由于num列上有索引，因此先在 num = 200的两条索引记录上加行级S锁。接着，去聚簇索引树上查询，这两条记录对应的pId=2，7，因此在pId=2，7的聚簇索引上加行级S锁，采用当前读。
4. ```select * from table where num > 200 lock in share mode```
当num > 200，由于num列上有索引，因此先在符合条件的 num = 300的一条索引记录上加行级S锁。接着，去聚簇索引树上查询，这条记录对应的pId=3，因此在pId=3的聚簇索引上加行级S锁，采用当前读。
5. ```select * from table where num = 200 for update```
当num = 200，由于num列上有索引，因此先在 num = 200的两条索引记录上加行级X锁。接着，去聚簇索引树上查询，这两条记录对应的pId=2，7，因此在pId=2，7的聚簇索引上加行级X锁，采用当前读。
6. ```select * from table where num > 200 for update```
当num > 200，由于num列上有索引，因此先在符合条件的 num = 300的一条索引记录上加行级X锁。接着，去聚簇索引树上查询，这条记录对应的pId=3，因此在pId=3的聚簇索引上加行级X锁，采用当前读。


**RR/Serializable+条件列非索引📍**
RR级别需要多考虑的就是```gap lock```，他的加锁特征在于，无论你怎么查都是锁全表。如下所示
接下来分析开始
1. ```select * from table where num = 200```
在RR级别下，不加任何锁，是快照读。
在Serializable级别下，在pId = 1,2,3,7（全表所有记录）的聚簇索引上加S锁。并且在
聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock
2. ```select * from table where num > 200```
在RR级别下，不加任何锁，是快照读。
在Serializable级别下，在pId = 1,2,3,7（全表所有记录）的聚簇索引上加S锁。并且在
聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock
3. ```select * from table where num = 200 lock in share mode```
在pId = 1,2,3,7（全表所有记录）的聚簇索引上加S锁。并且在
聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock
4. ```select * from table where num > 200 lock in share mode```
在pId = 1,2,3,7（全表所有记录）的聚簇索引上加S锁。并且在
聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock
5. ```select * from table where num = 200 for update```
在pId = 1,2,3,7（全表所有记录）的聚簇索引上加X锁。并且在
聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock
6. ```select * from table where num > 200 for update```
在pId = 1,2,3,7（全表所有记录）的聚簇索引上加X锁。并且在
聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock

**RR/Serializable+条件列是聚簇索引📍**
恩，大家应该知道pId是主键列，因此pId用的就是聚簇索引。该情况的加锁特征在于，如果where后的条件为精确查询(=的情况)，那么只存在record lock。如果where后的条件为范围查询(>或<的情况)，那么存在的是record lock+gap lock。
1. ```select * from table where pId = 2```
在RR级别下，不加任何锁，是快照读。
在Serializable级别下，是当前读，在pId=2的聚簇索引上加S锁，不存在gap lock。
2. ```select * from table where pId > 2```
在RR级别下，不加任何锁，是快照读。
在Serializable级别下，是当前读，在pId=3,7的聚簇索引上加S锁。在(2,3)(3,7)(7,+∞)加上gap lock
3. ```select * from table where pId = 2 lock in share mode```
是当前读，在pId=2的聚簇索引上加S锁，不存在gap lock。
4. ```select * from table where pId > 2 lock in share mode```
是当前读，在pId=3,7的聚簇索引上加S锁。在(2,3)(3,7)(7,+∞)加上gap lock
5. ```select * from table where pId = 2 for update```
是当前读，在pId=2的聚簇索引上加X锁。
6. ```select * from table where pId > 2 for update```
在pId=3,7的聚簇索引上加X锁。在(2,3)(3,7)(7,+∞)加上gap lock
7. ```select * from table where pId = 6 [lock in share mode|for update]```
注意了，pId=6是不存在的列，这种情况会在(3,7)上加gap lock。
8. ```select * from table where pId > 18 [lock in share mode|for update]```
注意了，pId>18，查询结果是空的。在这种情况下，是在(7,+∞)上加gap lock。


**RR/Serializable+条件列是非聚簇索引📍**

>这里非聚簇索引，需要区分是否为唯一索引。因为如果是非唯一索引，间隙锁的加锁方式是有区别的。

>先说一下，唯一索引的情况。如果是唯一索引，情况和RR/Serializable+条件列是聚簇索引类似，唯一有区别的是: 这个时候有两棵索引树，加锁是加在对应的非聚簇索引树和聚簇索引树上！大家可以自行推敲!

下面说一下，非聚簇索引是非唯一索引的情况，他和唯一索引的区别就是通过索引进行精确查询以后，不仅存在record lock，还存在gap lock。而通过唯一索引进行精确查询后，只存在record lock，不存在gap lock。老规矩在num列建立非唯一索引

1. ```select * from table where num = 200```
在RR级别下，不加任何锁，是快照读。
在Serializable级别下，是当前读，在pId=2，7的聚簇索引上加S锁，在num=200的非聚集索引上加S锁，在(100,200)(200,300)加上gap lock。
2. ```select * from table where num > 200```
在RR级别下，不加任何锁，是快照读。
在Serializable级别下，是当前读，在pId=3的聚簇索引上加S锁，在num=300的非聚集索引上加S锁。在(200,300)(300,+∞)加上gap lock
3. ```select * from table where num = 200 lock in share mode```
是当前读，在pId=2，7的聚簇索引上加S锁，在num=200的非聚集索引上加S锁，在(100,200)(200,300)加上gap lock。
4. ```select * from table where num > 200 lock in share mode```
是当前读，在pId=3的聚簇索引上加S锁，在num=300的非聚集索引上加S锁。在(200,300)(300,+∞)加上gap lock。
5. ```select * from table where num = 200 for update```
是当前读，在pId=2，7的聚簇索引上加S锁，在num=200的非聚集索引上加X锁，在(100,200)(200,300)加上gap lock。
6. ```select * from table where num > 200 for update```
是当前读，在pId=3的聚簇索引上加S锁，在num=300的非聚集索引上加X锁。在(200,300)(300,+∞)加上gap lock
7. ```select * from table where num = 250 [lock in share mode|for update]```
注意了，num=250是不存在的列，这种情况会在(200,300)上加gap lock。
8. ```select * from table where num > 400 [lock in share mode|for update]```
注意了，pId>400，查询结果是空的。在这种情况下，是在(400,+∞)上加gap lock。


## MySQL中的死锁

### 死锁的产生
**死锁🔒：** 是指两个或两个以上的进程在执行过程中,因争夺资源而造成的一种互相等待的现象,若无外力作用，它们都将无法推进下去.此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。表级锁不会产生死锁.所以解决死锁主要还是针对于最常用的InnoDB。

> 死锁的关键在于：两个(或以上)的Select加锁的顺序不一致。

那么对应的解决死锁问题的关键就是：让不同的Select加锁有次序


### 死锁的检测
✨在并发系统中不同线程出现循环资源依赖，涉及的线程都在等待别的线程释放资源时，就会导致这几个线程都进入无限等待的状态，称为死锁。

|事务A|事务B|
|-------|-------|
|```begin; ```||
|```update t set k=k+1 where id=1; ```|```begin; ```|
||```update t set k=k+1 where id=2; ```|
|```update t set k=k+1 where id=2; ```||
||```update t set k=k+1 where id=1; ```|

事务A在等待事务B释放id=2的行锁，而事务B在等待事务A释放id=1的行锁。事务A和事务B在互相等待对方的资源释放，就是进入了死锁状态。当出现死锁以后，有两种策略🥂：

- 一种策略是，直接进入等待，直到超时。这个超时时间可以通过参数```innodb_lock_wait_timeout```来设置
- 另一种策略是，发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数```innodb_deadlock_detect```设置为on，表示开启这个逻辑

在InnoDB中，```innodb_lock_wait_timeout```的默认值是50s，意味着如果采用第一个策略，当出现死锁以后，第一个被锁住的线程要过50s才会超时退出，然后其他线程才有可能继续执行。对于在线服务来说，这个等待时间往往是无法接受的

✨正常情况下还是要采用主动死锁检查策略，而且```innodb_deadlock_detect```的默认值本身就是on。主动死锁监测在发生死锁的时候，是能够快速发现并进行处理的，但是它有额外负担的。每当一个事务被锁的时候，就要看看它所依赖的线程有没有被别人锁住，如此循环，最后判断是否出现了循环等待，也就是死锁

🍬如果所有事务都要更新同一行的场景，每个新来的被堵住的线程都要判断会不会由于自己的加入导致死锁，这是一个时间复杂度是 **O(n)** 的操作

**👩怎么解决由这种热点行更新导致的性能问题？**

1. 如果确保这个业务一定不会出现死锁，可以临时把死锁检测关掉
2. 控制并发度
3. 将一行改成逻辑上的多行来减少锁冲突。以影院账户为例，可以考虑放在多条记录上，比如10个记录，影院的账户总额等于这10个记录的值的总和。这样每次要给影院账户加金额的时候，随机选其中一条记录来加。这样每次冲突概率变成员原来的1/10，可以减少锁等待个数，也就减少了死锁检测的CPU消耗


### 解除死锁的两种方法
1. ⭐终止（或撤销）进程。终止（或撤销）系统中的一个或多个死锁进程，直至打破循环环路，使系统从死锁状态中解除出来。
2. ⭐抢占资源。从一个或多个进程中抢占足够数量的资源，分配给死锁进程，以打破死锁状态。

**🔑第一种方法手动实现：**
在mysql中，输入
```
show processlist;
```
kill掉等待中或者睡眠的进程,重复操作即可
```
kill 10430(进程Id);
```

🚀🚀🚀
<br>