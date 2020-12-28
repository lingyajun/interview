回答：
一、简单说一下okhttp。

二、Okhttp的核心类有哪些？简单讲一下

三、OkHttp方面的其他面试题
	1、如何使用OkHttp进行异步网络请求，并根据请求结果刷新UI
	2、OkHttp对于网络请求都有哪些优化，如何实现的
	3、OkHttp框架中都用到了哪些设计模式

简略回答：
一、简单说一下okhttp。
1、支持SPDY、HTTP2.0
2、无缝支持GZIP来减少数据流量
3、支持同步、异步（异步使用较多）
4、缓存响应数据来减少重复的网络请求
5、可以从很多常用的连接问题中自动恢复

二、Okhttp的核心类有哪些？简单讲一下
Dispatcher类：
Interceptor类：

三、OkHttp方面的其他面试题

---
一、简单说一下okhttp。
1、支持SPDY、HTTP2.0

在早期的互联网中，由于协议都是一些比较简单的协议，内容基本上都是一些静态的页面、图片等，所以无连接、无状态的HTTP可以发挥自己简单快速、灵活的优势。 但随着业务逻辑越来越复杂以及我们对安全性的重视，无连接、无状态反而成为了HTTP的劣势，所以也就又来后来更加高级的互联网协议的诞生。

第一个要讲的就是SPDY，这是谷歌开发的一种互联网协议。它是一种HTTP的兼容协议、支持多路复用请求、对请求划分优先级（优先返回文字，图片音频等随后返回）、压缩HTTP头，以减少请求数据量。 

而HTTP2.0是在SPDY的基础上开发而来的，那么既然有了SPDY，为什么还要开发HTTP2.0。这是因为SPDY是完全由谷歌公司开发，这么重要的网络协议被把持在一家公司手里显然是不合适的。所以IETF（国际互联网工程任务组 The Internet Engineering Task Force，简称 IETF）就重新开发了HTTP2.0。HTTP2.0在SPDY的基础上又添加了更安全的SSL协议。SPDY和HTTP2.0作为一种更高级的网络协议，Okhttp必须得支持才行啊。而像其他的框架volley暂时是不支持HTTP/2的。所以这可以作为它的一个优点！

2、无缝支持GZIP来减少数据流量

为什么叫无缝支持的，意思就是说，你发送的数据和接受的收据在传递过程中都是经过gzip压缩的，并且这基本上你不需要你手动处理的，框架自动会帮你处理好。

首先，我们给request的请求头添加了("Accept-Encoding", "gzip")的键值对，说明发送的请求数据是经过gzip压缩的；

其次，我们在处理从服务器返回的数据时也会判断：

"gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding")
这两个步骤都是在BridgeIntercept的intercept方法中设置的。

3、支持同步、异步（异步使用较多）

4、缓存响应数据来减少重复的网络请求
CacheInterceptor.java  这个简单，如题

5、可以从很多常用的连接问题中自动恢复

RetryAndFollowUpInterceptor：重试重定向拦截器

a、请求失败后重新尝试连接：从Retry这个单词理解，但是在OKHttp中并不是所有的请求失败后（即返回码不是200）都会去重新连接，而是在发生 RouteException 或者 IOException 后再根据一些策略进行一些判断，如果可以恢复，就重新进请求
b、继续请求：FollowUp本意是跟进的意思，主要有以下几种类型可以继续发起请求：部分以3或4开头的返回码的情况下可以继续发送请求。
注意：其中FollowUp的次数受到限制，OKHTTP内部限制次数为20次以内

二、Okhttp的核心类有哪些？简单讲一下
Dispatcher类：

Dispatcher 通过维护一个线程池，来维护、管理、执行OKHttp的请求。
Dispatcher 内部维护着三个队列：同步请求队列 runningSyncCalls、异步请求队列 runningAsyncCalls、异步缓存队列 readyAsyncCalls，和一个线程池 executorService。
Dispatcher类整体可以参照生产者消费者模式来理解：
Dispatcher是生产者，executorService是消费者池，runningSyncCalls、runningAsyncCalls和readyAsyncCalls是消费者，用来消费请求Call。

Interceptor类：

官网：拦截器是Okhttp中提供的一种强大机制，它可以实现网络监听、请求、以及响应重写、请求失败重试等功能。
RetryAndFollowUpInterceptor、BridgeIntercept、CacheIntercept、ConnectIntercept、CallServerIntercept

RetryAndFollowUpInterceptor：重试和失败重定向拦截器 
BridgeInterceptor：桥接拦截器，处理一些必须的请求头信息的拦截器 
CacheInterceptor：缓存拦截器，用于处理缓存 
ConnectInterceptor：连接拦截器，建立可用的连接，是CallServerInterceptor的基本 
CallServerInterceptor：请求服务器拦截器将 http 请求写进 IO 流当中，并且从 IO 流中读取响应 Response


三、OkHttp方面的其他面试题
1、如何使用OkHttp进行异步网络请求，并根据请求结果刷新UI
第一步，创建一个OkHttpClient对象 OkHttpClient mClient = new OkHttpClient.Builder().build();
第二步，创建携带请求信息的Request对象 Request request = new Request.Builder().url("http://www.baidu.com").get().build();
第三步，创建Call对象  Call call = mClient.newCall(request);
第四步，call.enqueue()
需要注意的是，不能直接在Callback中更新UI，否则会报出异常

2、OkHttp对于网络请求都有哪些优化，如何实现的
a、通过连接池来减少请求延时:

我们知道，在okhttp中，我们每次的request请求都可以理解为一个connection，而每次发送请求的时候我们都要经过tcp的三次握手，然后传输数据，最后在释放连接。在高并发或者多个客户端请求的情况下，多次创建就会导致性能低下。如果能够connection复用的话，就能够很好地解决这个问题了。能够复用的关键就是客户端和服务端能够保持长连接，并让一个或者
多个连接复用。怎么保持长连接呢？

在BridgeInterceptor的intercept()方法中requestBuilder.header("Connection", "Keep-Alive")，我们在request的请求头添加了("Connection", "Keep-Alive")的键值对，这样就能够保持长连接。而连接池ConnectionPoll就是专门负责管理这些长连接的类。需要注意的是，我们在初始化 ConnectionPoll的时候，会设置 闲置的connections 的最大数量为5个，每个最长的存活时间为5分钟。

b、无缝支持GZIP来减少数据流量

c、缓存响应数据来减少重复的网络请求

d、可以从很多常用的连接问题中自动恢复

3、OkHttp框架中都用到了哪些设计模式
构造者模式
工厂模式
单例模式
责任链模式等等

---
[Okhttp面试简答_songzi1228的博客-CSDN博客_okhttp面试](https://blog.csdn.net/songzi1228/article/details/101050603)
