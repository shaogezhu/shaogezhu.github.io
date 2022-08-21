---
title: 推荐几个提升Java开发效率的“轮子”
date: 2022/08/21 17:31:06
update: 2022/08/21
categories:
 - [Java, 技术实践]
tags:
 - 轮子
 - 笔记
 - 工具

---

:::info

在`java`的庞大体系中，其实有很多不错的小工具，也就是我们平常说的：`轮子`。如果在我们的日常工作当中，能够将这些轮子用户，再配合一下`idea`的快捷键，可以极大得提升我们的开发效率🚀

今天把我一些压箱底的小工具，分享给大家，希望对你有所帮助📌。

:::

<br/>

## Collections

> 首先出场的是`java.util`包下的`Collections`类，该类主要用于操作集合或者返回集合，我个人非常喜欢用它。



### 排序

在工作中经常有对集合排序的需求。

看一下使用`Collections`工具 是如何实现**升序**和**降序**的：

```java
List<Integer> list = new ArrayList<>();
list.add(2);
list.add(1);
list.add(3);
Collections.sort(list);//升序
System.out.println(list);
Collections.reverse(list);//降序
System.out.println(list);
```

执行结果：

```java
[1, 2, 3]
[3, 2, 1]
```



### 获取最大或最小值

有时候需要找出集合中的`最大值`或者`最小值`，这时可以使用`Collections`的`max`和`min`方法。例如：

```java
List<Integer> list = new ArrayList<>();
list.add(2);
list.add(1);
list.add(3);
Integer max = Collections.max(list);//获取最大值
Integer min = Collections.min(list);//获取最小值
System.out.println(max);
System.out.println(min);
```

执行结果：

```java
3
1
```



### 转换线程安全集合

> 我们知道，`java`中的很多集合，比如：`ArrayList`、`LinkedList`、`HashMap`、`HashSet`等，都是线程不安全的。
>
> 换句话说，这些集合在多线程的环境中，添加数据会出现异常。

可以用`Collections`的`synchronizedxxx`方法，将这些线程不安全的集合，直接转换成线程安全集合。例如：

```java
List<Integer> list = new ArrayList<>();
list.add(2);
list.add(1);
list.add(3);

List<Integer> integers = Collections.synchronizedList(list);//将ArrayList转换成线程安全集合
System.out.println(integers);
```

它的底层会创建`SynchronizedRandomAccessList`或者`SynchronizedList`类，这两个类的很多方法都会用`synchronized`加锁。



### 返回空集合

有时，我们在判空之后，需要返回空集合，就可以使用`emptyList`方法，例如：

```java
private List<Integer> fun(List<Integer> list) {
    if (list == null || list.size() == 0) {
        return Collections.emptyList();
    }
    //业务处理
    return list;
}
```



### 二分查找

`binarySearch`方法提供了一个非常好用的`二分查找`功能，只用传入指定集合和需要找到的`key`即可。例如：

```java
List<Integer> list = new ArrayList<>();
list.add(2);
list.add(1);
list.add(3);

int i = Collections.binarySearch(list, 3);//二分查找
System.out.println(i);
```

执行结果：

```java
2
```





### 转换成不可修改集合

为了防止后续的程序把某个集合的结果修改了，有时候我们需要把某个集合定义成不可修改的，使用Collections的`unmodifiablexxx`方法就能轻松实现：

```java
List<Integer> list = new ArrayList<>();
list.add(2);
list.add(1);
list.add(3);

List<Integer> integers = Collections.unmodifiableList(list);
integers.add(4);
System.out.println(integers);
```

执行结果：

```java
Exception in thread "main" java.lang.UnsupportedOperationException
 at java.util.Collections$UnmodifiableCollection.add(Collections.java:1055)
 at com.sue.jump.service.test1.UtilTest.main(UtilTest.java:8)
```

当然Collections工具类中还有很多常用的方法，在这里就不一一介绍了，需要你自己去探索。

![image.png](/assets/2022-8/lz1.png)

![image.png](/assets/2022-8/lz2.png)

<br/>

## CollectionUtils

对集合操作，除了前面说的`Collections`工具类之后，`CollectionUtils`工具类也非常常用。

目前比较主流的是`spring`的`org.springframework.util`包下的`CollectionUtils`工具类。

![image.png](/assets/2022-8/lz3.png)

和`apache`的`org.apache.commons.collections`包下的`CollectionUtils`工具类。

![image.png](/assets/2022-8/lz4.png)

![image.png](/assets/2022-8/lz5.png)

> 我个人更推荐使用`apache`的包下的`CollectionUtils`工具类，因为它的工具更多更全面。

举个简单的例子，`spring`的`CollectionUtils`工具类没有判断集合不为空的方法。而`apache`的`CollectionUtils`工具类却有。

下面我们以`apache`的`CollectionUtils`工具类为例，介绍一下常用方法。



###  集合判空

通过`CollectionUtils`工具类的`isEmpty`方法可以轻松判断集合是否为空，`isNotEmpty`方法判断集合不为空。

```java
List<Integer> list = new ArrayList<>();
list.add(2);
list.add(1);
list.add(3);

if (CollectionUtils.isEmpty(list)) {
    System.out.println("集合为空");
}

if (CollectionUtils.isNotEmpty(list)) {
    System.out.println("集合不为空");
}
```



### 对两个集合进行操作

有时候我们需要对已有的两个集合进行操作，比如取**交集**或者**并集**等。

```java
List<Integer> list = new ArrayList<>();
list.add(2);
list.add(1);
list.add(3);

List<Integer> list2 = new ArrayList<>();
list2.add(2);
list2.add(4);

//获取并集
Collection<Integer> unionList = CollectionUtils.union(list, list2);
System.out.println(unionList);

//获取交集
Collection<Integer> intersectionList = CollectionUtils.intersection(list, list2);
System.out.println(intersectionList);

//获取交集的补集
Collection<Integer> disjunctionList = CollectionUtils.disjunction(list, list2);
System.out.println(disjunctionList);

//获取差集
Collection<Integer> subtractList = CollectionUtils.subtract(list, list2);
System.out.println(subtractList);
```

执行结果：

```java
[1, 2, 3, 4]
[2]
[1, 3, 4]
[1, 3]
```

说句实话，对两个集合的操作，在实际工作中用得挺多的，特别是很多批量的场景中。以前我们需要写一堆代码，但没想到有现成的"轮子"。

<br/>

## Lists

如果你引入`com.google.guava`的pom文件，会获得很多好用的小工具。这里推荐一款`com.google.common.collect`包下的集合工具：`Lists`。

> 它实在太好用了，让我爱不释手。

### 创建空集合

有时候，我们想创建一个空集合。这时可以用`Lists`的`newArrayList`方法，例如：

```java
List<Integer> list = Lists.newArrayList();
```



### 快速初始化集合

有时候，我们想给一个集合中初始化一些元素。这时可以用`Lists`的`newArrayList`方法，例如：

```java
List<Integer> list = Lists.newArrayList(1, 2, 3);
```

执行结果：

```java
[1, 2, 3]
```



### 笛卡尔积

如果你想将两个集合做`笛卡尔积`，`Lists`的`cartesianProduct`方法可以帮你实现：

```java
List<Integer> list1 = Lists.newArrayList(1, 2, 3);
List<Integer> list2 = Lists.newArrayList(4,5);
List<List<Integer>> productList = Lists.cartesianProduct(list1,list2);
System.out.println(productList);
```

执行结果：

```java
[[1, 4], [1, 5], [2, 4], [2, 5], [3, 4], [3, 5]]
```



### 分页|拆分

如果你想将一个`大集合`分成若干个`小集合`，可以使用`Lists`的`partition`方法：

```java
List<Integer> list = Lists.newArrayList(1, 2, 3, 4, 5);
List<List<Integer>> partitionList = Lists.partition(list, 2);
System.out.println(partitionList);
```

执行结果：

```java
[[1, 2], [3, 4], [5]]
```

这个例子中，`list`有5条数据，我将list集合按大小为2，分成了3页，即变成3个小集合。

这个是我最喜欢的方法之一，经常在项目中使用。

> 比如有个需求：现在有5000个id，需要调用批量用户查询接口，查出用户数据。但如果你直接查5000个用户，单次接口响应时间可能会非常慢。如果改成分页处理，每次只查500个用户，异步调用10次接口，就不会有单次接口响应慢的问题。



### 流处理

如果我们想把某个集合转换成另外一个接口，可以使用Lists的`transform`方法。例如：

```java
List<String> list = Lists.newArrayList("a","b","c");
List<String> transformList = Lists.transform(list, x -> x.toUpperCase());
System.out.println(transformList);
```

将小写字母转换成了大写字母。



### 颠倒顺序

Lists的有颠倒顺序的方法`reverse`。例如：

```java
List<Integer> list = Lists.newArrayList(3, 1, 2);
List<Integer> reverseList = Lists.reverse(list);
System.out.println(reverseList);
```

执行结果：

```java
[2, 1, 3]
```

list的原始顺序是3 1 2，使用`reverse`方法颠倒顺序之后，变成了2 1 3。

Lists还有其他的好用的工具，我在这里只是抛砖引玉，有兴趣的朋友，可以深入研究一下。

![image.png](/assets/2022-8/lz6.png)

<br/>

## Objects

在`jdk7`之后，提供了`Objects`工具类，我们可以通过它操作对象。

### 对象判空

在java中万事万物皆对象，对象的判空可以说无处不在。`Objects`的`isNull`方法判断对象是否为空，而`nonNull`方法判断对象是否不为空。例如：

```java
Integer integer = new Integer(1);

if (Objects.isNull(integer)) {
    System.out.println("对象为空");
}

if (Objects.nonNull(integer)) {
    System.out.println("对象不为空");
}
```



### 对象为空抛异常

如果我们想在对象为空时，抛出空指针异常，可以使用`Objects`的`requireNonNull`方法。例如：

```java
Integer integer1 = new Integer(128);

Objects.requireNonNull(integer1);
Objects.requireNonNull(integer1, "参数不能为空");
Objects.requireNonNull(integer1, () -> "参数不能为空");
```



### 判断两个对象是否相等

我们经常需要判断两个对象是否相等，`Objects`给我们提供了`equals`方法，能非常方便的实现：

```java
Integer integer1 = new Integer(1);
Integer integer2 = new Integer(1);

System.out.println(Objects.equals(integer1, integer2));
```

执行结果：

```java
true
```

但使用这个方法有坑，比如例子改成：

```java
Integer integer1 = new Integer(1);
Long integer2 = new Long(1);

System.out.println(Objects.equals(integer1, integer2));
```

执行结果：

```java
false
```

> 出现这种现象的主要原因是 两种数据类型不一致，一种是`Integer`、一种是`Long`所以为false。
>
> 更具体的就不过多解释，有兴趣的小伙们可以看一下该方法的源码，里面有非常详细的介绍。



### 获取对象的hashCode

如果我们想获取某个对象的`hashCode`，可以使用Objects的`hashCode`方法。例如：

```java
String str = new String("abc");
System.out.println(Objects.hashCode(str));
```

执行结果：

```java
96354
```

Objects的内容先介绍到这里，有兴趣的小伙，可以看看下面更多的方法：

![image.png](/assets/2022-8/lz7.png)

<br/>

## BooleanUtils

> 在`java`中布尔值，随处可见。

如果我们使用了布尔的包装类：`Boolean`，总感觉有点麻烦，因为它有三种值：`null`、`true`、`false`。我们在处理Boolean对象时，需要经常判空。

但如果使用`BooleanUtils`类处理布尔值，就方便许多了。



### 判断true或false

如果你想判断某个参数的值是true或false，可以直接使用`isTrue`或`isFalse`方法。例如：

```java
Boolean aBoolean = new Boolean(true);
System.out.println(BooleanUtils.isTrue(aBoolean));
System.out.println(BooleanUtils.isFalse(aBoolean));
```



### 判断不为true或不为false

有时候，需要判断某个参数不为`true`，即是`null`或者`false`。或者判断不为`false`，即是`null`或者`true`。

可以使用`isNotTrue`或`isNotFalse`方法。例如：

```java
Boolean aBoolean = new Boolean(true);
Boolean aBoolean1 = null;
System.out.println(BooleanUtils.isNotTrue(aBoolean));	//false
System.out.println(BooleanUtils.isNotTrue(aBoolean1));	//true
System.out.println(BooleanUtils.isNotFalse(aBoolean));	//true
System.out.println(BooleanUtils.isNotFalse(aBoolean1));	//true
```



### 转换成数字

如果你想将`true`转换成数字1，`false`转换成数字0，可以使用`toInteger`方法：

```java
Boolean aBoolean = new Boolean(true);
Boolean aBoolean1 = new Boolean(false);
System.out.println(BooleanUtils.toInteger(aBoolean));
System.out.println(BooleanUtils.toInteger(aBoolean1));
```

执行结果：

```java
1
0
```



### Boolean转换成布尔值

我们有时候需要将包装类`Boolean`对象，转换成原始的`boolean`对象，可以使用`toBoolean`方法。例如：

```java
Boolean aBoolean = new Boolean(true);
Boolean aBoolean1 = null;
System.out.println(BooleanUtils.toBoolean(aBoolean));
System.out.println(BooleanUtils.toBoolean(aBoolean1));
System.out.println(BooleanUtils.toBooleanDefaultIfNull(aBoolean1, false));
```

我们无需额外的判空了，而且还可以设置`Boolean`对象为空时返回的默认值。

`BooleanUtils`类的方法还有很多，有兴趣的小伙伴可以看看下面的内容：

![image.png](/assets/2022-8/lz8.png)

<br/>

## StringUtils

字符串（`String`）在我们的日常工作中，用得非常非常非常多。

在我们的代码中经常需要对**字符串判空，截取字符串、转换大小写、分隔字符串、比较字符串、去掉多余空格、拼接字符串、使用正则表达式**等等。

如果只用`String`类提供的那些方法，我们需要手写大量的额外代码，不然容易出现各种异常。

现在有个好消息是：`org.apache.commons.lang3`包下的`StringUtils`工具类，给我们提供了非常丰富的选择。



### 字符串判空

其实空字符串，不只是null一种，还有""，" "，"null"等等，多种情况。

`StringUtils`给我们提供了多个判空的静态方法，例如：

```java
 String str1 = null;
String str2 = "";
String str3 = " ";
String str4 = "abc";
System.out.println(StringUtils.isEmpty(str1));//true
System.out.println(StringUtils.isEmpty(str2));//true
System.out.println(StringUtils.isEmpty(str3));//false
System.out.println(StringUtils.isEmpty(str4));//false
System.out.println("=====");
System.out.println(StringUtils.isNotEmpty(str1));//false
System.out.println(StringUtils.isNotEmpty(str2));//false
System.out.println(StringUtils.isNotEmpty(str3));//true
System.out.println(StringUtils.isNotEmpty(str4));//true
System.out.println("=====");
System.out.println(StringUtils.isBlank(str1));//true
System.out.println(StringUtils.isBlank(str2));//true
System.out.println(StringUtils.isBlank(str3));//true
System.out.println(StringUtils.isBlank(str4));//false
System.out.println("=====");
System.out.println(StringUtils.isNotBlank(str1));//false
System.out.println(StringUtils.isNotBlank(str2));//false
System.out.println(StringUtils.isNotBlank(str3));//false
System.out.println(StringUtils.isNotBlank(str4));//true
```

示例中的：`isEmpty`、`isNotEmpty`、`isBlank`和`isNotBlank`，这4个判空方法你们可以根据实际情况使用。

> 优先推荐使用`isBlank`和`isNotBlank`方法，因为它会把`" "`也考虑进去。



### 分隔字符串

分隔字符串是常见需求，如果直接使用`String`类的`split`方法，就可能会出现空指针异常。

```java
String str1 = null;
System.out.println(StringUtils.split(str1,","));
System.out.println(str1.split(","));
```

执行结果：

```java
null
Exception in thread "main" java.lang.NullPointerException
 at com.sue.jump.service.test1.UtilTest.main(UtilTest.java:3)
```

使用`StringUtils`的`split`方法会返回`null`，而使用`String`的`split`方法会报指针异常。



### 判断是否纯数字

给定一个字符串，判断它是否为纯数字，可以使用`isNumeric`方法。例如：

```java
String str1 = "123";
String str2 = "123q";
String str3 = "0.33";
System.out.println(StringUtils.isNumeric(str1));//true
System.out.println(StringUtils.isNumeric(str2));//false
System.out.println(StringUtils.isNumeric(str3));//false
```



### 将集合拼接成字符串

有时候，我们需要将某个集合的内容，拼接成一个字符串，然后输出，这时可以使用`join`方法。例如：

```java
List<String> list = Lists.newArrayList("a", "b", "c");
List<Integer> list2 = Lists.newArrayList(1, 2, 3);
System.out.println(StringUtils.join(list, ","));
System.out.println(StringUtils.join(list2, " "));
```

执行结果：

```java
a,b,c
1 2 3
```

当然还有很多实用的方法，我在这里就不一一介绍了。

![image.png](/assets/2022-8/lz9.png)

![image.png](/assets/2022-8/lz10.png)

<br/>

## Assert

很多时候，我们需要在代码中做判断：如果不满足条件，则抛异常。

有没有统一的封装呢?

其实`spring`给我们提供了`Assert`类，它表示`断言`。



### 断言参数是否为空

断言`参数`是否空，如果不满足条件，则直接抛异常。

```java
String str = null;
Assert.isNull(str, "str必须为空");
Assert.isNull(str, () -> "str必须为空");
Assert.notNull(str, "str不能为空");
```

如果不满足条件就会抛出`IllegalArgumentException`异常。



### 断言集合是否为空

断言`集合`是否空，如果不满足条件，则直接抛异常。

```java
List<String> list = null;
Map<String, String> map = null;
Assert.notEmpty(list, "list不能为空");
Assert.notEmpty(list, () -> "list不能为空");
Assert.notEmpty(map, "map不能为空");
```

如果不满足条件就会抛出`IllegalArgumentException`异常。



### 断言条件是否为空

断言是否满足某个`条件`，如果不满足条件，则直接抛异常。

```java
List<String> list = null;
Assert.isTrue(CollectionUtils.isNotEmpty(list), "list不能为空");
Assert.isTrue(CollectionUtils.isNotEmpty(list), () -> "list不能为空");
```

当然Assert类还有一些其他的功能，这里就不多介绍了。

![image.png](/assets/2022-8/lz11.png)

<br/>

## IOUtils

`IO`流在我们日常工作中也用得比较多，尽管java已经给我们提供了丰富的`API`。

但我们不得不每次读取文件，或者写入文件之后，写一些重复的的代码。手动在`finally`代码块中关闭流，不然可能会造成`内存溢出`。

有个好消息是：如果你使用`org.apache.commons.io`包下的`IOUtils`类，会节省大量的时间。



### 读取文件

如果你想将某个txt文件中的数据，读取到字符串当中，可以使用`IOUtils`类的`toString`方法。例如：

```java
String str = IOUtils.toString(new FileInputStream("/temp/a.txt"), StandardCharsets.UTF_8);
System.out.println(str);
```





### 写入文件

如果你想将某个字符串的内容，写入到指定文件当中，可以使用`IOUtils`类的`write`方法。例如：

```java
String str = "abcde";
IOUtils.write(str, new FileOutputStream("/temp/b.tx"), StandardCharsets.UTF_8);
```





### 文件拷贝

如果你想将某个文件中的所有内容，都拷贝到另一个文件当中，可以使用`IOUtils`类的`copy`方法。例如：

```java
IOUtils.copy(new FileInputStream("/temp/a.txt"), new FileOutputStream("/temp/b.txt"));
```



### 读取文件内容到字节数组

如果你想将某个文件中的内容，读取字节数组中，可以使用`IOUtils`类的`toByteArray`方法。例如：

```java
byte[] bytes = IOUtils.toByteArray(new FileInputStream("/temp/a.txt"));
```

IOUtils类非常实用，感兴趣的小伙们，可以看看下面内容。

![image.png](/assets/2022-8/lz12.png)

<br/>

## MDC

`MDC`是`org.slf4j`包下的一个类，它的全称是`Mapped Diagnostic Context`，我们可以认为它是一个线程安全的**存放诊断日志的容器**。

MDC的底层是用了`ThreadLocal`来保存数据的。

我们可以用它传递参数。

> 例如现在有这样一种场景：我们使用`RestTemplate`调用远程接口时，有时需要在`header`中传递信息，比如：traceId，source等，便于在查询日志时能够串联一次完整的请求链路，快速定位问题。

这种业务场景就能通过`ClientHttpRequestInterceptor`接口实现，具体做法如下：

**第一步**，定义一个`LogFilter`拦截所有接口请求，在MDC中设置`traceId`：

```java
public class LogFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        MdcUtil.add(UUID.randomUUID().toString());
        System.out.println("记录请求日志");
        chain.doFilter(request, response);
        System.out.println("记录响应日志");
    }

    @Override
    public void destroy() {
    }
}
```



**第二步**，实现`ClientHttpRequestInterceptor`接口，MDC中获取当前请求的traceId，然后设置到`header`中：

```java
public class RestTemplateInterceptor implements ClientHttpRequestInterceptor {

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        request.getHeaders().set("traceId", MdcUtil.get());
        return execution.execute(request, body);
    }
}
```



**第三步**，定义配置类，配置上面定义的`RestTemplateInterceptor`类：

```java
@Configuration
public class RestTemplateConfiguration {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setInterceptors(Collections.singletonList(restTemplateInterceptor()));
        return restTemplate;
    }

    @Bean
    public RestTemplateInterceptor restTemplateInterceptor() {
        return new RestTemplateInterceptor();
    }
}
```



其中`MdcUtil`其实是利用`MDC`工具在`ThreadLocal`中存储和获取`traceId`

```java
public class MdcUtil {

    private static final String TRACE_ID = "TRACE_ID";

    public static String get() {
        return MDC.get(TRACE_ID);
    }

    public static void add(String value) {
        MDC.put(TRACE_ID, value);
    }
}
```

当然，这个例子中没有演示`MdcUtil`类的`add`方法具体调的地方，我们可以在`filter`中执行接口方法之前，生成`traceId`，调用`MdcUtil`类的`add`方法添加到`MDC`中，然后在同一个请求的其他地方就能通过`MdcUtil`类的`get`方法获取到该`traceId`。

能使用`MDC`保存`traceId`等参数的根本原因是，用户请求到应用服务器，`Tomcat`会从线程池中分配一个线程去处理该请求。

那么该请求的整个过程中，保存到`MDC`的`ThreadLocal`中的参数，也是该线程独享的，所以不会有线程安全问题。

<br/>

## ClassUtils



spring的`org.springframework.util`包下的`ClassUtils`类，它里面有很多让我们惊喜的功能。

它里面包含了类和对象相关的很多非常实用的方法。



### 获取对象的所有接口

如果你想获取某个对象的所有接口，可以使用`ClassUtils`的`getAllInterfaces`方法。例如：

```java
Class<?>[] allInterfaces = ClassUtils.getAllInterfaces(new User());
```



### 获取某个类的包名

如果你想获取某个类的包名，可以使用`ClassUtils`的`getPackageName`方法。例如：

```java
String packageName = ClassUtils.getPackageName(User.class);
System.out.println(packageName);
```



### 判断某个类是否内部类

如果你想判断某个类是否内部类，可以使用`ClassUtils`的`isInnerClass`方法。例如：

```java
System.out.println(ClassUtils.isInnerClass(User.class));
```



### 判断对象是否代理对象

如果你想判断对象是否代理对象，可以使用`ClassUtils`的`isCglibProxy`方法。例如：

```java
System.out.println(ClassUtils.isCglibProxy(new User()));
```

ClassUtils还有很多有用的方法，感兴趣的朋友，可以看看下面内容：

![image.png](/assets/2022-8/lz13.png)

<br/>

## BeanUtils

spring给我们提供了一个`JavaBean`的工具类，它在`org.springframework.beans`包下面，它的名字叫做：`BeanUtils`。

> 让我们一起看看这个工具可以带给我们哪些惊喜。

### 拷贝对象的属性

曾几何时，你有没有这样的需求：把某个对象中的所有属性，都拷贝到另外一个对象中。这时就能使用`BeanUtils`的`copyProperties`方法。例如：

```java
User user1 = new User();
user1.setId(1L);
user1.setName("田埂上的梦");
user1.setAddress("北京");

User user2 = new User();
BeanUtils.copyProperties(user1, user2);
System.out.println(user2);
```



### 实例化某个类

如果你想通过反射实例化一个类的对象，可以使用`BeanUtils`的`instantiateClass`方法。例如：

```java
User user = BeanUtils.instantiateClass(User.class);
System.out.println(user);
```



### 获取指定类的指定方法

如果你想获取某个类的指定方法，可以使用`BeanUtils`的`findDeclaredMethod`方法。例如：

```java
Method declaredMethod = BeanUtils.findDeclaredMethod(User.class, "getId");
System.out.println(declaredMethod.getName());
```



### 获取指定方法的参数

如果你想获取某个方法的参数，可以使用`BeanUtils`的`findPropertyForMethod`方法。例如：

```java
Method declaredMethod = BeanUtils.findDeclaredMethod(User.class, "getId");
PropertyDescriptor propertyForMethod = BeanUtils.findPropertyForMethod(declaredMethod);
System.out.println(propertyForMethod.getName());
```

如果你对`BeanUtils`比较感兴趣，可以看看下面内容：

![image.png](/assets/2022-8/lz14.png)

<br/>

## ReflectionUtils

有时候，我们需要在项目中使用`反射`功能，如果使用最原始的方法来开发，代码量会非常多，而且很麻烦，它需要处理一大堆异常以及访问权限等问题。

好消息是spring给我们提供了一个`ReflectionUtils`工具，它在`org.springframework.util`包下面。



### 获取方法

如果你想获取某个类的某个方法，可以使用`ReflectionUtils`类的`findMethod`方法。例如：

```java
Method method = ReflectionUtils.findMethod(User.class, "getId");
```



### 获取字段

如果你想获取某个类的某个字段，可以使用`ReflectionUtils`类的`findField`方法。例如：

```java
Field field = ReflectionUtils.findField(User.class, "id");
```



### 执行方法

如果你想通过反射调用某个方法，传递参数，可以使用`ReflectionUtils`类的`invokeMethod`方法。例如：

```java
 ReflectionUtils.invokeMethod(method, springContextsUtil.getBean(beanName), param);
```



### 判断字段是否常量

如果你想判断某个字段是否常量，可以使用`ReflectionUtils`类的`isPublicStaticFinal`方法。例如：

```java
Field field = ReflectionUtils.findField(User.class, "id");
System.out.println(ReflectionUtils.isPublicStaticFinal(field));
```



### 判断是否equals方法

如果你想判断某个方法是否`equals`方法，可以使用`ReflectionUtils`类的`isEqualsMethod`方法。例如：

```java
Method method = ReflectionUtils.findMethod(User.class, "getId");
System.out.println(ReflectionUtils.isEqualsMethod(method));
```

当然这个类还有不少有趣的方法，感兴趣的朋友，可以看看下面内容：

![image.png](/assets/2022-8/lz15.png)

<br/>

## Base64Utils



有时候，为了安全考虑，需要将参数只用`base64`编码。

这时就能直接使用`org.springframework.util`包下的`Base64Utils`工具类。

它里面包含：`encode`和`decode`方法，用于对数据进行加密和解密。例如：

```java
String str = "abc";
String encode = new String(Base64Utils.encode(str.getBytes()));
System.out.println("加密后：" + encode);
try {
    String decode = new String(Base64Utils.decode(encode.getBytes()), "utf8");
    System.out.println("解密后：" + decode);
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}
```

执行结果：

```java
加密后：YWJj
解密后：abc
```

<br/>

## StandardCharsets

我们在做字符转换的时候，经常需要指定字符编码，比如：`UTF-8`、`ISO-8859-1`等等。

这时就可以直接使用`java.nio.charset`包下的`StandardCharsets`类中静态变量。

例如：

```java
String str = "abc";
String encode = new String(Base64Utils.encode(str.getBytes()));
System.out.println("加密后：" + encode);
String decode = new String(Base64Utils.decode(encode.getBytes()),StandardCharsets.UTF_8);
System.out.println("解密后：" + decode);
```

<br/>

## DigestUtils

有时候，我们需要对数据进行加密处理，比如：`md5`或`sha256`。

可以使用apache的`org.apache.commons.codec.digest`包下的`DigestUtils`类。



### md5加密

如果你想对数据进行md5加密，可以使用`DigestUtils`的`md5Hex`方法。例如：

```java
String md5Hex = DigestUtils.md5Hex("苏三说技术");
System.out.println(md5Hex);
```



### sha256加密

如果你想对数据进行sha256加密，可以使用`DigestUtils`的`sha256Hex`方法。例如：

```java
String md5Hex = DigestUtils.sha256Hex("苏三说技术");
System.out.println(md5Hex);
```

当然这个工具还有很多其他的加密方法：

![image.png](/assets/2022-8/lz16.png)

<br/>

## SerializationUtils

有时候，我们需要把数据进行`序列化`和`反序列化`处理。

传统的做法是某个类实现`Serializable`接口，然后重新它的`writeObject`和`readObject`方法。

但如果使用`org.springframework.util`包下的`SerializationUtils`工具类，能更轻松实现序列化和反序列化功能。例如：

```java
Map<String, String> map = Maps.newHashMap();
map.put("a", "1");
map.put("b", "2");
map.put("c", "3");
byte[] serialize = SerializationUtils.serialize(map);
Object deserialize = SerializationUtils.deserialize(serialize);
System.out.println(deserialize);
```

<br/>

## HttpStatus

很多时候，我们会在代码中定义`http`的返回码，比如：接口正常返回200，异常返回500，接口找不到返回404，接口不可用返回502等。

```java
private int SUCCESS_CODE = 200;
private int ERROR_CODE = 500;
private int NOT_FOUND_CODE = 404;
```

其实`org.springframework.http`包下的`HttpStatus`枚举，或者`org.apache.http`包下的`HttpStatus`接口，已经把常用的`http`返回码给我们定义好了，直接拿来用就可以了，真的不用再重复定义了。

![image.png](/assets/2022-8/lz17.png)



好了，这次就分享到这里。

工作当中还有很多好用的小工具，我会继续更新的。

<br/>
