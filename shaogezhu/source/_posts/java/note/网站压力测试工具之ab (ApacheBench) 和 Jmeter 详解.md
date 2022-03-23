---
title: 网站压力测试工具之ab (ApacheBench) 和 Jmeter 详解
date: 2021/09/03 20:46:25
update: 2021/09/03
categories:
- [Java, 学习笔记]
tags:
- java
- 教程
- 压测
- 工具

---


:::info
我们写项目的时候经常需要测试，不仅需要功能测试还需要压力测试。接下来就介绍一下大家经常使用的测试工具。在学习DDoS知识的过程中，了解到ab这个测压工具，之前在其他博客上也了解到jmeter的测压工具。这两款软件都是Apache的开源项目，也是目前常用的网站测压工具。
:::

## ab和jmeter的区别

**1. jmeter是一次完整的请求和返回；AB只是发出去请求
2. ab是shell模式下轻量级的测试工具，jmeter则作为有GUI界面的更高级测试工具
3. jmeter可以提供更加详细的统计结果数据，比如接口错误信息、单线程的请求时间等，而AB则不支持
4. ab更加轻量级，软件消耗的资源少，不会占用太多内存和cpu，而jmeter消耗资源比较多
5. Jmeter可以动态查看结果，而ab不可以，只有执行完才出结果，同时Jmeter的结果更加准确**

## ab如何使用

ab是httpd（也叫apache）携带的轻量级测压工具，我们要从apache的官网上下载httpd

### 下载httpd（apache）🏫
> linux版本下载比较容易，以windows版本,apache 2.4为例。

官网：https://httpd.apache.org/

![image.png](/assets/2021-9/ab1.jpg)
点击download

![image.png](/assets/2021-9/ab2.jpg)

此处随便选一个提供商。

![image.png](/assets/2021-9/ab3.jpg)

选择版本下载

![image.png](/assets/2021-9/ab4.jpg)

下载完之后解压

### 使用ab和abs
解压后，在bin目录下有两个小工具ab.exe和abs.exe ，分别用于http和https。

![image.png](/assets/2021-9/ab5.jpg)

之后可以打开cmd或者shell，直接使用ab和abs工具

使用ab -help 列出了我们可以使用的参数

![image.png](/assets/2021-9/ab6.jpg)

我们最常用的指令是ab -n 100 -c 10 http://test.com/ 其中－n表示请求总数，－c表示并发数

对于https协议 使用abs -n 100 -c 10 https://test.com/

以对我的网站进行测压为例（别乱用啊）：

正在测压：

![image.png](/assets/2021-9/ab7.jpg)
返回的结果：

![image.png](/assets/2021-9/ab8.jpg)
测试结果：

> **Server Software:** nginx/1.21.1 #被测试的Web服务器软件名称。
> **Server Hostname:** shaogezhu.cn/ #请求的URL主机名。
> **Server Port:** 443 #被测试的Web服务器软件的监听端口。
> **SSL/TLS Protocol:** TLSv1.2,ECDHE-RSA-AES256-GCM-SHA384,2048,256 Server Temp Key: X25519 253 bits TLS 
>
> **Server Name:** shaogezhu.cn
> **Document Path:** / #请求的URL中的根绝对路径，通过该文件的后缀名，我们一般可以了解该请求的类型。
>
> **Document Length:** 1270 bytes #表示HTTP响应数据的正文长度。
> **Concurrency Level:** 10 #并发用户数，这是我们设置的参数之一。
> **Time taken for tests:** 27.378 seconds #所有这些请求被处理完成所花费的总时间。
>
> **Complete requests:** 100 #总请求数量，这是我们设置的参数之一。 Failed requests: 1 #失败的请求数量， (Connect: 1, Receive: 0, Length: 0, Exceptions: 0) Total transferred: 150400 bytes # 所有请求的响应数据长度总和，包括每个HTTP响应数据的头信息和正文数据的长度。注意这里不包括HTTP请求数据的长度，仅仅为web服务器流向用户PC的应用层数据总长度 HTML transferred: 127000 bytes #表示所有请求的响应数据中正文数据的总和，也就是减去了Total transferred中HTTP响应数据中的头信息的长度。
>
> **Requests per second:** 3.65 [#/sec] (mean) #每秒多少请求 
>
> **Time per request:** 2737.802 [ms] (mean) #用户平均请求等待时间 
>
> **Time per request:** 273.780 [ms] (mean, across all concurrent requests) #服务器平均处理时间，也就是服务器吞吐量的倒数 Transfer rate: 5.36 [Kbytes/sec] received #表示这些请求在单位时间内从服务器获取的数据长度
>
> **Percentage of the requests served within a certain time (ms)** #这部分数据用于描述每个请求处理时间的分布情况 50% 584 66% 600 75% 624 80% 669 90% 862 95% 21606 98% 21623 99% 21626 100% 21626 (longest request)

**最重要的三个参数：** ⭐

>**Requests per second：吞吐率 ，指的是某个并发用户数下单位时间内处理的请求数**
>
>**Time per request：上面的是用户平均请求等待时间**
>
>**Time per request：下面的是服务器平均请求处理时间 是吞吐率的倒数**


## 使用jmeter

JMeter是Apache组织开发的基于Java的压力测试工具。用于对软件做压力测试，它最初被设计用于Web应用测试，但后来扩展到其他测试领域。 它可以用于测试静态和动态资源，例如静态文件、Java 小服务程序、CGI 脚本、Java 对象、数据库、FTP 服务器， 等等。JMeter 还支持通过创建带有断言的脚本来验证你的程序返回了你期望的结果。

推荐一个jmeter的使用教程：[http://www.jmeter.com.cn/category/jmeter-book](http://www.jmeter.com.cn/category/jmeter-book)

### 下载
因为它是基于java的，下载前请确保你有jdk（8以上）的环境。

**下载地址：[http://jmeter.apache.org/download_jmeter.cgi](http://jmeter.apache.org/download_jmeter.cgi)**

选择右侧需要下载的压缩包版本（我这里下载的是apache-jmeter-5.4.1.zip），下载完成后解压即可。

![image.png](/assets/2021-9/ab9.jpg)

### 启动
🎈我们在解压后的bin目录下找到jmeter.bat的批处理脚本，只要有jdk8的系统环境，点击即可启动，出现一个cmd窗口和Jmeter的GUI界面（基于Swing），注意不要关闭cmd窗口，不然GUI窗口也会关闭。

![image.png](/assets/2021-9/ab10.jpg)
也可以把Jmeter放入系统的环境变量中，命令行中输入Jmeter即可启动。我们一般使用不多，就不做讲解，如果想要了解详情，可以参考上面提供的连接中的使用教程。

### 使用
1.右击左边的Test Plan 添加一个线程组。

![image.png](/assets/2021-9/ab11.jpg)
在这个线程组下这个线程组的线程数，也就是并发用户数，循环次数每个并发用户的请求数。

参数 ramp-up period 用于告知JMeter 要在多长时间内建立全部的线程。默认值是1。假如说ramp-up period 为零， JMeter 将立即建立所有线程，并发访问，假设ramp-up period 设置成T 秒， 全部线程数设置成N个， JMeter 将每隔T/N秒建立一个线程

👉🏻 Loop Count 是循环次数，**请求总数量 = 线程数*循环次数**。

![image.png](/assets/2021-9/ab12.jpg)

2.在这个线程组下右键在sampler（取样器）里面选择http请求，创建http请求。

![image.png](/assets/2021-9/ab13.jpg)
可以设置http请求的协议，参数，路径等。

![image.png](/assets/2021-9/ab14.jpg)
3.在HTTP请求右键里面选择监听器下的聚合报告，创建一个聚合报告，可以查看这次测试的结果。

也可以添加结果树。

![image.png](/assets/2021-9/ab15.jpg)
4.点击启动的图标，之后再聚合报告和视图结果树中查看本次测试的结果。

聚合报告可以查看统计数据：

![image.png](/assets/2021-9/ab16.jpg)


数据含义：
>**Label：** 每个 JMeter 的 element（例如 HTTP Request）都有一个 Name 属性，这里显示的就是 Name 属性的值
>
>**Samples：** 表示你这次测试中一共发出了多少个请求，如果模拟10个用户，每个用户迭代10次，那么这里显示100
>
>**Average：** 平均响应时间——默认情况下是单个 Request 的平均响应时间
>
>**Median：**  中位数，也就是 50 ％用户的响应时间
>
>**90% Line：**   90 ％用户的响应时间
>
>**Min：** 最小响应时间
>
>**Max：**  最大响应时间
>
>**Error%：** 本次测试中出现错误的请求的数量/请求的总数
>
>**Throughput：** 吞吐量——默认情况下表示每秒完成的请求数
>
>**KB/Sec：** 每秒从服务器端接收到的数据量

结果树可以查看每一个请求和请求返回的结果：

![image.png](/assets/2021-9/ab17.jpg)

我们看到，虽然设置为了https 和443端口，但是Jmeter还是走了http，而要让Jmeter使用Https协议，需要我们导入Https证书，常用的方法有录制Https和手动配置证书两种办法。

录制https：

这种方式Jmeter会代理浏览器的请求和响应。

手动配置证书：

这里我们选择这种办法。

1）下载被测网站证书导入Jmeter

![image.png](/assets/2021-9/ab18.jpg)
![image.png](/assets/2021-9/ab19.jpg)
使用jdk自带的工具keytool将 cer格式的证书转换为store格式

命令如下keytool -import -alias "test.store" -file "my.cer" -keystore test.store 中间会要求设置口令

![image.png](/assets/2021-9/ab20.jpg)
再Jmeter中选择SSL Manager 导入证书

![image.png](/assets/2021-9/ab21.jpg)
2)之后即可使用https协议


## 最后
补充ab工具的所有参数解释🚀🚀🚀
>-n 测试会话中所执行的请求个数,默认仅执行一个请求
-c 一次产生的请求个数,即同一时间发出多少个请求,默认为一次一个
-t 测试所进行的最大秒数,默认为无时间限制....其内部隐含值是[-n 50000],它可以使对服务器的测试限制在一个固定的总时间以内
-p 包含了需要POST的数据的文件
-T POST数据所使用的Content-type头信息
-v 设置显示信息的详细程度
-w 以HTML表格的形式输出结果,默认是白色背景的两列宽度的一张表
-i 以HTML表格的形式输出结果,默认是白色背景的两列宽度的一张表
-C 对请求附加一个Cookie行，其典型形式是name=value的参数对,此参数可以重复
-H 对请求附加额外的头信息,此参数的典型形式是一个有效的头信息行,其中包含了以冒号分隔的字段和值的对(如"Accept-Encoding: zip/zop;8bit")
-A HTTP验证,用冒号:分隔传递用户名及密码
-P 无论服务器是否需要(即是否发送了401认证需求代码),此字符串都会被发送
-X 对请求使用代理服务器
-V 显示版本号并退出
-k 启用HTTP KeepAlive功能,即在一个HTTP会话中执行多个请求,默认为不启用KeepAlive功能
-d 不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)
-S 不显示中值和标准背离值,且均值和中值为标准背离值的1到2倍时,也不显示警告或出错信息,默认会显示最小值/均值/最大值等(为以前的版本提供支持)
-g 把所有测试结果写入一个'gnuplot'或者TSV(以Tab分隔的)文件
-e 产生一个以逗号分隔的(CSV)文件,其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间 -h 显示使用方法
-k 发送keep-alive指令到服务器端

