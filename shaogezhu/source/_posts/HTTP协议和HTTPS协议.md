
title: HTTP协议 和 HTTPS协议 详解
date: 2021-11-5 22:57:49
update: 2021-11-6
categories: 
- 学习笔记
tags:
- 面试
- 八股文
- 基础

---

:::info
我们经常说的通信协议，以及三次握手四次挥手，都是什么？ http和https有什么关系，建立链接的流程一样么？为什么https就安全了？ 读完这篇文章你就会有非常深刻的认识🙇
:::


## 协议
> 📌网络协议是计算机之间为了实现网络通信而达成的一种“约定”或者”规则“，有了这种”约定“，不同厂商的生产设备，以及不同操作系统组成的计算机之间，就可以实现通信。

## HTTP 协议

**✨HTTP**（HyperText Transfer Protocol：超文本传输协议）是一种用于分布式、协作式和超媒体信息系统的应用层协议。 简单来说就是一种发布和接收 HTML 页面的方法，被用于在 Web 浏览器和网站服务器之间传递信息。


#### http建立链接的过程

- 客户端向服务器发送请求报文
- 服务器根据请求报文收集对应的组合成响应报文
- 客户端收到响应报文后进行解析渲染




#### HTTP特点

1. http协议支持客户端/服务端模式，也是一种请求/响应模式的协议。
2. 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。
3. 灵活：HTTP允许传输任意类型的数据对象。传输的类型由Content-Type加以标记。
4. 无连接：限制每次连接只处理一个请求。服务器处理完请求，并收到客户的应答后，即断开连接，但是却不利于客户端与服务器保持会话连接，为了弥补这种不足，产生了两项记录http状态的技术，一个叫做Cookie,一个叫做Session。
5. 无状态：无状态是指协议对于事务处理没有记忆，后续处理需要前面的信息，则必须重传。


## HTTPS 协议
**✨HTTPS**（Hypertext Transfer Protocol Secure：超文本传输安全协议）是一种透过计算机网络进行安全通信的传输协议。HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包。HTTPS 开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。

#### https建立链接的过程

![image.png](/assets/2021-11/http1.png)

**具体步骤如下：**
1. client向server发送请求https://www.shaogezhu.cn ，然后连接到server的443端口，发送的信息包含客户端支持的加密算法。
2. server接收到信息之后给予client响应握手信息，包含匹配好的协商加密算法，这个加密算法一定是client发送给server加密算法的子集。
3. 随即server给client发送第二个响应报文是数字证书。服务端必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面，这套证书其实就是一对公钥和私钥。传送证书，这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间、服务端的公钥，第三方证书认证机构(CA)的签名，服务端的域名信息等内容。
4. 客户端解析验证证书，这部分工作是由客户端的SSL/TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值（预主秘钥）。
5. 客户端认证证书通过之后，接下来是通过生成会话密钥。然后通过证书的公钥加密会话密钥。
6. 服务端使用私钥解密得到会话密钥，然后服务端会通过会话秘钥加密一条消息回传给客户端，如果客户端能够正常接受的话表明SSL层连接建立完成了。

#### https的特点
1. **内容加密：** 采用混合加密技术，中间者无法直接查看明文内容
2. **验证身份：** 通过证书认证客户端访问的是自己的服务器
3. **保护数据完整性：** 防止传输的内容被中间人冒充或者篡改


## HTTP 与 HTTPS 区别
1. HTTP 明文传输，数据都是未加密的，安全性较差，HTTPS（SSL+HTTP） 数据传输过程是加密的，安全性较好。
2. 使用 HTTPS 协议需要到 **CA**（Certificate Authority，数字证书认证机构） 申请证书，一般免费证书较少，因而需要一定费用。
3. HTTP 页面响应速度比 HTTPS 快，主要是因为 HTTP 使用 TCP 三次握手建立连接，客户端和服务器需要交换 3 个包，而 HTTPS除了 TCP 的三个包，还要加上 ssl 握手需要的 9 个包，所以一共是 12 个包。
4. http 和 https 使用的是完全不同的连接方式，用的端口也不一样，前者是 80，后者是 443。
5. HTTPS 其实就是建构在 SSL/TLS 之上的 HTTP 协议，所以，要比较 HTTPS 比 HTTP 要更耗费服务器资源。


> **CA认证**：第三方数字签名认证机构，为了确认https中服务器发送的公开秘钥没有被篡改，因此会先发送给可信的第三方进行签名，客户端使用机构的公开秘钥进行签名认证，用来保证服务器公开秘钥的可靠性。
> 
> **数字摘要：** 通过单向hash函数对原文进行哈希，将需加密的明文“摘要”成一串固定长度(如128bit)的密文，不同的明文摘要成的密文其结果总是不相同，同样的明文其摘要必定一致，并且即使知道了摘要也不能反推出明文。

#### 对称秘钥加密和非对称秘钥加密

**非对称加密🌴：** 公开秘钥是任何人都可以轻易获得，并且可以随意发送。但是公开密钥加密后的内容只有提供公开秘钥的服务方可以使用自己的私有秘钥进行解密
**对称加密🌴：** 加密和解密使用的是同一秘钥，所以只要获得加密秘钥就可以进行解密。


#### TCP三次握手过程
![image.png](/assets/2021-11/http2.png)

**具体过程如下👀：** 
（1）第一次握手：客户端尝试连接服务器，向服务器发送 syn 包（同步序列编号Synchronize Sequence Numbers），seq=a，客户端进入 SYN_SEND 状态等待服务器确认

（2）第二次握手：服务器接收客户端syn包并确认（Ack=a+1），seq=b，同时向客户端发送一个 SYN包（syn=1），此时服务器进入 SYN_RECV 状态

（3）第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=b+1），此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手

（4）Server端在收到Client的ACK帧之后，会从SYN_RCVD状态会进入ESTABLISHED状态，至此，Server方向的通道连接建立成功，Server可以发送数据给Client，TCP的全双工连接建立完成。


#### 四次挥手过程
![image.png](/assets/2021-11/http3.png)

**具体过程如下👀：**
（1）第一次挥手：主动断开方（可以是客户端，也可以是服务器端），向对方发送一个FIN结束请求报文，此报文的FIN位被设置为1，并且正确设置Sequence Number（序列号）和AcknowledgmentNumber（确认号）。发送完成后，主动断开方进入FIN_WAIT_1状态，这表示主动断开方没有业务数据要发送给对方，准备关闭SOCKET连接了。

（2）第二次挥手：正常情况下，在收到了主动断开方发送的FIN断开请求报文后，被动断开方会发送一个ACK响应报文，报文的Acknowledgment Number（确认号）值为断开请求报文的Sequence Number（序列号）加1，该ACK确认报文的含义是：“我同意你的连接断开请求”。之后，被动断开方就进入了CLOSE-WAIT（关闭等待）状态，TCP协议服务会通知高层的应用进程，对方向本地方向的连接已经关闭，对方已经没有数据要发送了，若本地还要发送数据给对方，对方依然会接受。被动断开方的CLOSE-WAIT（关闭等待）还要持续一段时间，也就是整个CLOSE-WAIT状态持续的时间。

👉🏻主动断开方在收到了ACK报文后，由FIN_WAIT_1转换成FIN_WAIT_2状态。

（3）第三次挥手：在发送完成ACK报文后，被动断开方还可以继续完成业务数据的发送，待剩余数据发送完成后，或者CLOSE-WAIT（关闭等待）截止后，被动断开方会向主动断开方发送一个FIN+ACK结束响应报文，表示被动断开方的数据都发送完了，然后，被动断开方进入LAST_ACK状态。

（4）第四次挥手：主动断开方收在到FIN+ACK断开响应报文后，还需要进行最后的确认，向被动断开方发送一个ACK确认报文，然后，自己就进入TIME_WAIT状态，等待超时后最终关闭连接。处于TIME_WAIT状态的主动断开方，在等待完成2MSL的时间后，如果期间没有收到其他报文，则证明对方已正常关闭，主动断开方的连接最终关闭。


## 拓展知识

#### 为什么连接的时候是三次握手，关闭的时候却是四次握手？

- 因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。


#### 为什么连接建立的时候是三次握手，可以改成两次握手吗？
- 3次握手完成两个重要的功能，既要双方做好发送数据的准备工作(双方都知道彼此已准备好)，也要允许双方就初始序列号进行协商，这个序列号在握手过程中被发送和确认。现在把三次握手改成仅需要两次握手，死锁是可能发生的。作为例子，考虑计算机S和C之间的通信，假定C给S发送一个连接请求分组，S收到了这个分组，并发 送了确认应答分组。按照两次握手的协定，S认为连接已经成功地建立了，可以开始发送数据分组。可是，C在S的应答分组在传输中被丢失的情况下，将不知道S 是否已准备好，不知道S建立什么样的序列号，C甚至怀疑S是否收到自己的连接请求分组。在这种情况下，C认为连接还未建立成功，将忽略S发来的任何数据分 组，只等待连接确认应答分组。而S在发出的分组超时后，重复发送同样的分组。这样就形成了死锁。

#### 为什么主动断开方在TIME-WAIT状态必须等待2MSL的时间？

- 虽然按道理，四个报文都发送完毕，我们可以直接进入CLOSE状态了，但是我们必须假象网络是不可靠的，有可以最后一个ACK丢失。所以TIME_WAIT状态就是用来重发可能丢失的ACK报文。在Client发送出最后的ACK回复，但该ACK可能丢失。Server如果没有收到ACK，将不断重复发送FIN片段。所以Client不能立即关闭，它必须确认Server接收到了该ACK。Client会在发送出ACK之后进入到TIME_WAIT状态。Client会设置一个计时器，等待2MSL的时间。如果在该时间内再次收到FIN，那么Client会重发ACK并再次等待2MSL。所谓的2MSL是两倍的MSL(Maximum Segment Lifetime)。MSL指一个片段在网络中最大的存活时间，2MSL就是一个发送和一个回复所需的最大时间。如果直到2MSL，Client都没有再次收到FIN，那么Client推断ACK已经被成功接收，则结束TCP连接。

#### 如果已经建立了连接，但是客户端突然出现故障了怎么办？
- TCP还设有一个保活计时器，显然，客户端如果出现故障，服务器不能一直等下去，白白浪费资源。服务器每收到一次客户端的请求后都会重新复位这个计时器，时间通常是设置为2小时，若两小时还没有收到客户端的任何数据，服务器就会发送一个探测报文段，以后每隔75秒钟发送一次。若一连发送10个探测报文仍然没反应，服务器就认为客户端出了故障，接着就关闭连接。


#### TCP怎么保证可靠性

1. **序列号和确认号机制：**
	TCP 发送端发送数据包的时候会选择序列号，接收端收到数据包后会检测数据包的完整性，如果检测通过会响应确认号表示收到了数据包。
2. **超时重发机制：**
	发送端发送了数据包后会启动一个定时器，如果一定时间没有收到接收端的确认，将会重新发送该数据包。
3. **对乱序数据包重新排序：**
	从 IP 网络层传输到 TCP 层的数据包可能会乱序，TCP 层会对数据包重新排序再发给应用层。
4. **丢弃重复数据：**
 	从 IP 网络层传输到 TCP 层的数据包可能会重复，TCP 层会丢弃重复的数据包。
5. **流量控制：**
	TCP 发送端和接收端都有一个固定大小的缓冲空间，为了防止发送端发送数据的速度太快导致接收端缓冲区溢出，发送端只能发送接收端可以接纳的数据，为了达到这种控制效果，TCP 用了流量控制协议（可变大小的滑动窗口协议）来实现。
	【滑动窗口详解】
	滑动窗口通俗来讲就是一种流量控制技术。
	它本质上是描述接收方的TCP数据报缓冲区大小的数据，发送方根据这个数据来计算自己最多能发送多长的数据，如果发送方收到接收方的窗口大小为0的话，那么发送方将停止发送数据，等到接收方发送窗口大小不为0的数据报的到来。首次发送数据时的窗口是链路带宽决定。
6. **拥塞控制：**
	 在数据传输过程中，可能由于网络状态的问题，造成网络拥堵，此时引入拥塞控制机制，在保证TCP可靠性的同时，提高性能，具体为慢启动、拥塞避免、快重传与快恢复……


**【拥塞控制算法】**
![image.png](/assets/2021-11/http4.png)
- **慢启动：** TCP刚建立连接之后，会一点一点提高发送数据报的数量，即每当发送方收到一个ACK，就会将拥塞窗口的大小变为原来的2倍；当增长到慢启动门限ssthresh时，启动拥塞避免算法
- **拥塞避免：** 为了避免拥塞，超过慢启动门限后，拥塞窗口的大小每收到一个ACK，加1，为线性增长
- **快重传：** 当出现拥塞的时候，发生了丢包的现象，快速重传算***将拥塞窗口变为原来的一半，并且让慢启动门限变为拥塞窗口大小
如果不进行快速重传，进行超时重传，则是将慢启动门限设置为拥塞窗口大小的一半，并且将拥塞窗口大小置为1
- **快恢复：** 让拥塞窗口大小 +3 （即确认收到了三个数据报），并执行拥塞避免的线性增长


#### 状态码分类
- 1XX- 信息型，服务器收到请求，需要请求者继续操作。
- 2XX- 成功型，请求成功收到，理解并处理。
- 3XX - 重定向，需要进一步的操作以完成请求。
- 4XX - 客户端错误，请求包含语法错误或无法完成请求。
- 5XX - 服务器错误，服务器在处理请求的过程中发生了错误。


#### 常见的状态码
- 200 OK - 客户端请求成功
- 301 - 资源（网页等）被永久转移到其它URL
- 302 - 临时跳转
- 400 Bad Request - 客户端请求有语法错误，不能被服务器所理解
- 401 Unauthorized - 请求未经授权，这个状态代码必须和WWW- Authenticate报头域一起使用（简单理解：表示用户没有权限 令牌,用户名,密码错误）
- 403 Forbidden 代表客户端错误，指的是服务器端有能力处理该请求，但是拒绝授权访问。
- 404 - 请求资源不存在，可能是输入了错误的URL
- 500 - 服务器内部发生了不可预期的错误
- 503 Server Unavailable - 服务器当前不能处理客户端的请求，一段时间后可能恢复正常。

🚀🚀🚀
<br>