title: spring事务 常见的失效场景
date: 2021-10-29 22:49:32
update: 2021-10-29
category:

- [Java, 学习笔记]
tags:
- spring
- 面试

---

:::info
使用spring的项目，在处理**转账、付款**等操作的时候，为了数据能被正确的修改，通常会使用spring的注解@Transactional对事务进行控制。spring对事物也有失效的场景我们来看一下，防止以后程序出现bug👀
:::

## spring事务失效场景

### 1、数据库引擎不支持事务

以 MySQL 为例🌰，其 MyISAM 引擎是不支持事务操作的，InnoDB 才是支持事务的引擎，一般要支持事务都会使用 InnoDB。


### 2、没有被 Spring 管理

如下面代码所示：
~~~java
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        // 数据库增删改操作
    }
    
}
~~~

如果把 @Service 注解注释掉，这个类就不会被加载成一个 Bean，那这个类就不会被 Spring 管理了，事务自然就失效了。



### 3、方法不是 public 的

以下来自 Spring 官方文档✨：
>When using proxies, you should apply the @Transactional annotation only to methods with public visibility. If you do annotate protected, private or package-visible methods with the @Transactional annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. Consider the use of AspectJ (see below) if you need to annotate non-public methods.
>
>大概意思就是 **@Transactional** 只能用于 **public** 的方法上，否则事务不会失效，如果要用在非 public 方法上，可以开启 AspectJ 代理模式。

### 4、异常被捕获了

如下代码所示：
~~~java
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // 数据库增删改操作
        } catch {
            
        }
    }
    
}
~~~

将异常捕获了，没有抛出来，spring并不知道发生了异常，所以不会回滚。

### 5、异常类型错误

还是上面的列子：
~~~java
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // 数据库增删改操作
        } catch {
            throw new Exception("发生异常");
        }
    }
    
}
~~~

这样事务也是不生效的，因为默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，如：

~~~java
@Transactional(rollbackFor = Exception.class)
~~~
这个配置仅限于 Throwable 异常类及其子类。

### 6、不支持事务

来看下面这个例子：
~~~java
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void update(Order order) {
        updateOrder(order);  //调用下边的方法
    }
    
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void updateOrder(Order order) {
        // 数据库增删改操作
    }
    
}
~~~

🎈Propagation.NOT_SUPPORTED： 表示不以事务运行，当前若存在事务则挂起，详细的事务传播机制在下面。


## spring事务的7种传播机制


- **REQUIRED**

	> 如果当前方法有事务则加入事务，没有则创建一个事务。


- **NOT_SUPPORTED**

	>不支持事务，如果当前有事务则挂起事务运行。


- **REQUIREDS_NEW**

	>新建一个事务并在这个事务中运行，如果当前存在事务就把当前事务挂起。新建的事务的提交与回滚一挂起事务没有联系，不会影响挂起事务的操作。


- **MANDATORY**

	>强制当前方法使用事务运行，如果当前没有事务则抛出异常。


- **NEVER**

	>当前方法不能存在事务，即非事务状态运行，如果存在事务则抛出异常。


- **SUPPORTS**

	>支持当前事务，如果当前没事务也支持非事务状态运行。


- **NESTED**

	>如果当前存在事务，则在嵌套事务内执行。嵌套事务的提交与回滚与父事务没有任务关系，反之，当父事务提交嵌套事务也一起提交，父事务回滚会也回滚嵌套事务的。
	>
	>如果当前没有事务，则新建一个事务运行，这时候则与PROPAGATION_REQUIRED场景一致。




🚀🚀🚀
<br>