---
title: 终于弄明白了ThreadLocal
date: 2021-10-23 11:11:55
categories:
- [Java, 学习笔记]
tags:
- java
- 面试
 
---


:::info
🌈写SpringBoot项目的时候，经常用到的一个保存用户信息的类就是Threadlocal，我们今天就来详细介绍一下这个类。
:::


## Threadlocal有什么用：

📌简单的说就是，<font color="blue">一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的（每个线程都只能看到自己线程的值）。</font>如下图：

![image.png](/assets/2021-10/tl1.png)


## ThreadLocal使用实例

### API介绍

🦄在使用Threadlocal之前我们先看以下它的API：
![16314961931.png](/assets/2021-10/tl2.png)

ThreadLocal类的API非常的简单，在这里比较重要的就是get()、set()、remove(),set用于赋值操作，get用于获取变量的值，remove就是删除当前变量的值.需要注意的是initialValue方法会在第一次调用时被触发，用于初始化当前变量值，默认情况下initialValue返回的是null。


### ThreadLocal的使用

🍋说完了ThreadLocal类的API了，那我们就来动手实践一下了，来理解前面的那句话：**一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的（每个线程都只能看到自己线程的值）**


~~~java
public class ThreadLocalTest {

    private static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>() {
	// 重写这个方法，可以修改“线程变量”的初始值，默认是null
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };

    public static void main(String[] args) throws InterruptedException {

        //一号线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("一号线程set前：" + threadLocal.get());
                threadLocal.set(1);
                System.out.println("一号线程set后：" + threadLocal.get());
            }
        }).start();

        //二号线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("二号线程set前：" + threadLocal.get());
                threadLocal.set(2);
                System.out.println("二号线程set后：" + threadLocal.get());

            }
        }).start();

        //主线程睡1s
        Thread.sleep(1000);

        //主线程
        System.out.println("主线程的threadlocal值：" + threadLocal.get());

    }

}

~~~


**稍微解释一下上面的代码：💡**

每一个ThreadLocal实例就类似于一个变量名，不同的ThreadLocal实例就是不同的变量名，它们内部会存有一个值（暂时这么理解）<font color="violet">在后面的描述中所说的“<font color="blue">ThreadLocal变量或者是线程变量</font>”代表的就是ThreadLocal类的实例。</font>


在类中创建了一个静态的 **“ThreadLocal变量”**，在主线程中创建两个线程，在这两个线程中分别设置ThreadLocal变量为1和2。然后等待一号和二号线程执行完毕后，在主线程中查看ThreadLocal变量的值。

**程序结果及分析⌛**
![16314972101.png](/assets/2021-10/tl3.png)

程序结果重点看的是主线程输出的是0，如果是一个普通变量，在一号线程和二号线程中将普通变量设置为1和2，那么在一二号线程执行完毕后在打印这个变量，输出的值肯定是1或者2（到底输出哪一个由操作系统的线程调度逻辑有关）。但使用ThreadLocal变量通过两个线程赋值后，在主线程程中输出的却是初始值0。在这也就是为什么“一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的”，每个线程都只能看到自己线程的值，这也就是 <font color="red">ThreadLocal的核心作用：实现线程范围的局部变量。</font>



## Threadlocal 的源码分析

### 原理
> 每个Thread对象都有一个ThreadLocalMap，**当创建一个ThreadLocal的时候，就会将该ThreadLocal对象添加到该Map中，其中键就是ThreadLocal，值可以是任意类型。** 这句话刚看可能不是很懂，下面我们一起看完源码就明白了。


前面我们的理解是所有的常量值或者是引用类型的引用都是保存在ThreadLocal实例中的，但实际上不是的，这种说法只是让我们更好的理解ThreadLocal变量这个概念。<font color = "blue">**向ThreadLocal存入一个值，实际上是向当前线程对象中的ThreadLocalMap存入值**</font>，ThreadLocalMap我们可以简单的理解成一个Map，而向这个Map存值的key就是ThreadLocal实例本身。

### 源码
![image.png](/assets/2021-10/tl4.png)


👉也就是说，想要存入的ThreadLocal中的数据实际上并没有存到ThreadLocal对象中去，而是以**这个ThreadLocal实例作为key存到了当前线程中的一个Map中去了**，获取ThreadLocal的值时同样也是这个道理。这也就是为什么ThreadLocal可以实现线程之间隔离的原因了。


## 内部类ThreadLocalMap
ThreadLocalMap是ThreadLocal的内部类，实现了一套自己的Map结构✨

**ThreadLocalMap属性：**
~~~java
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        //初始容量16
        private static final int INITIAL_CAPACITY = 16;
        //散列表
        private Entry[] table;
        //entry 有效数量 
        private int size = 0;
        //负载因子
        private int threshold; 
~~~


ThreadLocalMap设置ThreadLocal 变量
~~~java
    private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            
            //与运算  & (len-1) 这就是为什么 要求数组len 要求2的n次幂 
            //因为len减一后最后一个bit是1 与运算计算出来的数值下标 能保证全覆盖 
            //否者数组有效位会减半 
            //如果是hashmap 计算完下标后 会增加链表 或红黑树的查找计算量 
            int i = key.threadLocalHashCode & (len-1);
            
            // 从下标位置开始向后循环搜索  不会死循环  有扩容因子 必定有空余槽点
            for (Entry e = tab[i];   e != null;  e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();
                //一种情况 是当前引用 返回值
                if (k == key) {
                    e.value = value;
                    return;
                }
                //槽点被GC掉 重设状态 
                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }
			//槽点为空 设置value
            tab[i] = new Entry(key, value);
            //设置ThreadLocal数量
            int sz = ++size;
			
			//没有可清理的槽点 并且数量大于负载因子 rehash
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
~~~

ThreadLocalMap属性介绍😎：

- 和普通Hashmap类似存储在一个数组内，但与hashmap使用的拉链法解决散列冲突不同的是 ThreadLocalMap使用开放地址法
- 数组 初始容量16，负载因子2/3
- node节点 的key封装了WeakReference 用于回收


### ThreadLocalMap存储位置
>储存在Thread中，有两个ThreadLocalMap变量

![16315167211.png](/assets/2021-10/tl4.5.png)

1. threadLocals 在ThreadLocal对象方法set中去创建 也由ThreadLocal来维护
~~~java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
~~~

2. inheritableThreadLocals 和ThreadLocal类似 InheritableThreadLocal重写了createMap方法
~~~java
void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
~~~

3. inheritableThreadLocals 作用是将ThreadLocalMap传递给子线程

![image.png](/assets/2021-10/tl5.png)

4. init方法中 条件满足后直接为子线程创建ThreadLocalMap

![image.png](/assets/2021-10/tl6.png)

**注意：**

1. 仅在初始化子线程的时候会传递 中途改变副线程的inheritableThreadLocals 变量 不会将影响结果传递到子线程 。
2. 使用线程池要注意 线程不回收 尽量避免使用父线程的inheritableThreadLocals 导致错误


## Key的弱引用问题
为什么要用弱引用,官方是这样回答的
> To help deal with very large and long-lived usages, the hash table entries use WeakReferences for keys.
为了处理非常大和生命周期非常长的线程，哈希表使用弱引用作为 key。

生命周期长的线程可以理解为：线程池的核心线程

ThreadLocal在没有外部对象强引用时如Thread，发生GC时弱引用Key会被回收，而Value是强引用不会回收，如果创建ThreadLocal的线程一直持续运行如线程池中的线程，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

- **key 使用强引用🌴：** 引用的ThreadLocal的对象被回收了，但是ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。
- **key 使用弱引用🌴：** 引用的ThreadLocal的对象被回收了，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一次ThreadLocalMap调用set,get，remove的时候会被清除。

Java8中已经做了一些优化如，在ThreadLocal的get()、set()、remove()方法调用的时候会清除掉线程ThreadLocalMap中所有Entry中Key为null的Value，并将整个Entry设置为null，利于下次内存回收。

### java中的四种引用
1. **强引用📍：** 如果一个对象具有强引用，它就不会被垃圾回收器回收。即使当前内存空间不足，JVM也不会回收它，而是抛出 OutOfMemoryError 错误，使程序异常终止。如果想中断强引用和某个对象之间的关联，可以显式地将引用赋值为null，这样一来的话，JVM在合适的时间就会回收该对象
2. **软引用📍：** 在使用软引用时，如果内存的空间足够，软引用就能继续被使用，而不会被垃圾回收器回收，只有在内存不足时，软引用才会被垃圾回收器回收。（软引用可用来实现内存敏感的高速缓存,比如网页缓存、图片缓存等。使用软引用能防止内存泄露，增强程序的健壮性）
3. **弱引用📍：** 具有弱引用的对象拥有的生命周期更短暂。因为当 JVM 进行垃圾回收，一旦发现弱引用对象，无论当前内存空间是否充足，都会将弱引用回收。不过由于垃圾回收器是一个优先级较低的线程，所以并不一定能迅速发现弱引用对象
4. **虚引用📍：** 虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。（注意哦，其它引用是被JVM回收后才被传入ReferenceQueue中的。由于这个机制，所以虚引用大多被用于引用销毁前的处理工作。可以使用在对象销毁前的一些操作，比如说资源释放等。）


通常ThreadLocalMap的生命周期跟Thread（注意线程池中的Thread）一样长，如果没有手动删除对应key（线程使用结束归还给线程池了，其中的KV不再被使用但又不会GC回收，可认为是内存泄漏），一定会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用ThreadLocal会被GC回收，不会内存泄漏，对应的value在下一次ThreadLocalMap调用set,get,remove的时候会被清除，Java8已经做了上面的代码优化。

## 总结：

**ThreadLocal的作用：** 实现线程范围内的局部变量，即ThreadLocal在一个线程中是共享的，在不同线程之间是隔离的。

**ThreadLocal的原理：** ThreadLocal存入值时<font color="red">使用当前ThreadLocal实例作为key（并不是以当前线程对象作为key）</font>，存入当前线程对象中的Map中去。最开始在看源码之前，我以为是以当前线程对象作为key将对象存入到ThreadLocal中的Map中去....

🚀🚀🚀
<br>