---
title: 一篇文章教你轻松使用Fastjson
date: 2022/08/14 22:17:06
update: 2022/08/14
categories:
 - [Java, 技术实践]
tags:
 - 实战
 - 笔记
 - Fastjson

---

:::info

在前后端数据传输交互中,经常会遇到字符串(String)与Json、XML等格式相互转换与解析，其中json以跨语言，跨前后端的优点在开发中被频繁使用，基本上可以说是标准的数据交换格式。下面我们就来学习一下常用的Fastjson。
:::



# Fastjson介绍



## 简介

Fastjson 是阿里巴巴公司开源的一个 Java 语言编写的高性能功能完善的 JSON 库。它采用一种“**假定有序快速匹配**”的算法，把 JSON Parse 的性能提升到极致，是目前 Java 语言中最快的 JSON 库，并且它不依赖于其它任何库。



## 主要特点：

- 快速 FAST (比其它任何基于 Java 的解析器和生成器更快，包括 jackson）
- 强大（支持普通 JDK 类包括任意 Java Bean Class、Collection、Map、Date 或 enum）
- 零依赖（没有依赖其它任何类库除了 JDK）



# Fastjson使用



maven依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <!--（起码1.2.67以上）因为这个版本以下存在漏洞-->
    <version>版本根据自己需要</version>
</dependency>
```

android版本：

```xml
<dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>fastjson</artifactId>
     <version>x.x.xx.android</version>
</dependency>
```





# 基本用法



## JSON



`fastjson` 提供了 `JSON.parseObject()` 和 `JSON.toJSONString` 两个直接用于解析和生成的方法。前者实现了反序列化，后者实现了序列化。

```java
public class User {
    public String name;
    public int age;
    public String emailAddress;

    // 省略 getter/setter
}
```



**生成 JSON 格式字符串：**<small>（Object to JSON-String）</small>

```java
User user = new User("怪盗tiangeng",23);

String jsonStr = JSON.toJSONString(user);
```



**解析 JSON 格式字符串：**<small>（JSON-String to Object）</small>

> `注意`，需要 User 类有无参的构造方法。

```java
String jsonString = "{\"name\":\"怪盗tiangeng\",\"age\":23}";

User user = JSON.parseObject(jsonString, User.class);
```



## JSONObject

> 在 `Java` 中，一个` Map<String, Object> `对象可以等价描述一个` JavaBean`。`Map` 中的每个键值对，对应` JavaBean` 中的一个属性。键为属性名，值为属性值。



`JSONObject `就是` fastjson `专门提供的一个用于这样的使用场景的` Map`：

```java
public class JSONObject extends ... implements Map<String, Object>, ... {
  ...
}
```



`fastJson` 提供了 `JSON.parseObject()` 将 `JSON `格式字符串转换成一个`JSONObject` 对象（本质上，就是转换成一个`Map`）。

```java
String jsonString = "{\"name\":\"怪盗tiangeng\",\"age\":23}";

JSONObject map = JSON.parseObject(jsonString);

String val1 = map.getString("name");
int val2 = map.getIntValue("age");
```

> 对于不同类型的各种属性，JSONObject 提供了不同的 `getXXX()` 方法与之对应。



## JSONArray



反序列化数组形式的 `JSON `格式字符串的第三种方案是利用 `fastJson `提供的 `JSONArray` 。

```java
public class JSONArray extends ... implements List<Object>, ... {
  ...
}
```

从其定义看，`JSONArray `本质上也就是一个` List`，所以这种方案本质上和之前的第二种方案没太大区别。

只不过它不需要在最开始提供所要转换的类型，而是通过各种 `getXXX()` 方法在获取值数据时进行转换。

```java
JSONArray arr2 = JSON.parseArray(jsonStr);

for (int i = 0; i < arr2.size(); i++) {
    System.out.println(arr2.getObject(i, User.class));
}
```



## 数组、集合的序列化和反序列化



数组的序列化很简单，与普通对象类似;

```java
User[] arr = new User[3];
arr[0] = new User("tom", 20, "123@qq.com");
arr[1] = new User("jerry", 19, "456@qq.com");
arr[2] = new User("bean", 21, null);

String jsonStr = JSON.toJSONString(arr);
```



数组的反序列化，有几种方案，其中一种方案与普通对象的反序列化类似，，唯一需要注意的就是填写正确的数组类型：

```java
User[] arr = JSON.parseObject(jsonStr, User[].class);

for (User user : arr)
     System.out.println(user);
```



鉴于 `List` 与 数组 有极大地相似性，另一种反序列化数组形式的`JSON` 字符串的方案是，将其反序列化为对象的链表：

```java
List<User> list = JSON.parseArray(jsonStr, User.class);

for (User user : list)
    System.out.println(user);
```



## 泛型 和 RESETfull API



泛型的引入可以减少无关的代码，这在` resetful api `返回数据时反映得更为清楚，通常 `resetful api` 接口返回的` JSON` 数据为：

```javascript
{"status":0,"msg":'...',"data":{...}},
{"status":0,"msg":'...',"data":[...]},
```

> `status` 表示响应状态码，`msg` 表示状态码配套的信息，而` data `才是最核心的部分：响应的数据。一般而言，它要么表示返回一个对象，要么表示返回一个对象的集合<small>（数组或` List`）</small>。

在这里会把这个结果对象定义成泛型类：

```java
public class Result<T> {
    public int code;
    public String message;
    public T data;
}
```

否则，`data `的类型无穷无尽，你就需要为每一种情况定义一种` Result` 类。

那么对于 `data` 字段，当 T 是` User` 时则可以写为 `Result<User>`，当是个列表的时候为 `Result<List<User>>`，其它类型同理（`Result<XXX>` 和 `Result<List<XXX>>`）。



对此，`Json` 字符串转对象时，代码自然就类似如下：

```java
JSONArray list = JSON.parseArray(jsonStr);

for (int i = 0; i < list.size(); i++) {
    Result<User> result = list.getObject(i, Result.class);
    System.out.println(result);
}
```



## @JSONField 注解：name 属性

> 默认/一般情况下，`JSON`字段中的名字和类的属性名是一致的。但是也有不一致的情况，因为本身驼峰命名法（如 Java）和下划线命名法（如 C）本身就是两大命名规则“流派”。



对类的属性使用 `@JSONField` 注解（及其 `name` 属性），可以重新指定与该属性对应的` JSON `字符串中的名字。

```java
@JSONField(name="email_address")
public String emailAddress
```



你甚至可以重新命名为另一个看起来毫不相关的名字：

```java
@JSONField(name="xxx")
public String emailAddress;
```



```java
String jsonString = "{\"name\":\"怪盗taingeng\",\"age\":23, \"email_address\":\"123456@qq.com\"}";
User user = JSON.parseObject(jsonString, User.class);
```



## @JSONField 注解：serialzeFeatures 属性



在将一个对象转换成·`JSON` 格式字符串时，`fastJson `会自动忽略掉其中值为 `null` 的属性，及不包含在` JSON `格式为字符串中。

```java
User user = new User();
user.setName("tom");
System.out.println(JSON.toJSONString(user));  // {"name":"tom"}
```

有时你需要以更完善的形式来表达出这个对象中的所有属性，即便其值为 `null` 。

为此，`fastJson` 通过 `@JSONField` 注解的 `serialzeFeatures` 属性值进行设置：

```java
@JSONField(serialzeFeatures={SerializerFeature.WriteNullNumberAsZero})
private Integer age;

@JSONField(serialzeFeatures={SerializerFeature.WriteMapNullValue})
private String emailAddress;

@JSONField(serialzeFeatures={SerializerFeature.WriteNullStringAsEmpty})
private List<String> list;
```



```java
User user = new User();
user.setName("tom");
System.out.println(JSON.toJSONString(user));
// {"age":0,"emailAddress":null,"list":null,"name":"tom"}
```



## @JSONField 注解：format 属性



当所需要序列化和反序列化的对象的属性有 `Date` 类型时，这里就涉及到 `Date` 类型的字符串形式的格式问题，为此 `@JSONField`注解提供了 `format` 属性用以自定义其字符串格式：

```java
@JSONField(format="yyyy-MM-dd")
private Date birthDate;
```

使用ISO-8601日期格式:

```java
@JSONField(serialzeFeatures={SerializerFeature.UseISO8601DateFormat})
private Date birthDate;
```

反序列化能够自动识别如下日期格式：

- ISO-8601日期格式
- yyyy-MM-dd
- yyyy-MM-dd HH:mm:ss
- yyyy-MM-dd HH:mm:ss.SSS
- 毫秒数字
- 毫秒数字字符串
- .NET JSON日期格式
- new Date(198293238)



## @JSONField 注解：serialize 属性

标注了 `@JSONField(serialize=false)` 的属性在序列化成`JSON`格式字符串的过程中，会被 `fastJSON `忽略。无论其是否有值。



## 循环引用



`fastJson` 自动识别循环混用的问题，并将其替换成 `$ref` 形式。而是不是 `StackOverflowError` 异常。

> `$ref`：`..` 上一级
>
> `$ref`：`@` 当前对象，也就是自引用
>
> `$ref`：`$` 根对象
>
> `$ref`：`$.children.0` 基于路径的引用，相当于 `root.getChildren().get(0)`


如果百分百肯定所要序列化的对象不存在循环引用问题，可以在序列化时关闭该检测功能（默认是开启的）。

```java
JSON.toJSONString(object, SerializerFeature.DisableCircularReferenceDetect);
```





