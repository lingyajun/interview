OkHttpClient client = new OkHttpClient();
Request request = new Request.Builder()
        .url(url)
        .build();
Call call = client.newCall(request)；
//同步请求
Response response = call.execute()；
//异步请求
call.enqueue(new Callback() {});

一次请求:
1 创建OkHttpClient实例
2 创建Request实例
3 用client实例的newCall方法创建一个Call实例
4 调用Call的同步或异步请求方法，得到Response实例
其中真正发起网络请求的，是第4步，即调用Call的execute或enqueue方法。



Call & RealCall 主要方法：
同步请求 ：client.newCall(request).execute();
异步请求： client.newCall(request).enqueue();

synchronized (this) 确保每个call只能被执行一次不能重复执行;


Dispatcher 是 Okhttp 的调度器，用来管理控制请求的队列。内部通过线程池来确保队列的有序运行。
Dispatcher内部存在两个队列，一个是正在运行的队列 runningAsyncCalls，另一个是 readyAsyncCalls 队列。
如果当前运行数小于最大运行数，并且当前请求的host小于最大请求个数，那么就会直接加入运行队列(runningAsyncCalls)，并运行。如果超了，就会加入准备队列(readyAsyncCalls)。
Dispatcher实际上还有一个同步队列(runningSyncCalls)，没有给同步队列做限制，只要一加入就开始执行请求。
当请求队列完成请求后需要进行移除.



AsyncCall 是 RealCall 里面的内部类，获取响应数据最终是是通过 getResponseWithInterceptorChain() 来获取的。然后通过回调将 Response 返回给用户。
 finally 执行了client.dispatcher().finished(this) 通过调度器移除队列。

getResponseWithInterceptorChain(): 是 Okhttp 最核心的部分，采用责任链的模式来使每个功能分开，每个 Interceptor 自行完成自己的任务，并且将不属于自己的任务交给下一个，简化了各自的责任和逻辑。


RealInterceptorChain 责任链:
内部持有了当前责任链的所有拦截器list，index (当前正在处理拦截器索引)等。
RealInterceptorChain的 proceed 方法里的逻辑，归来起来就是如下：

1. 首先是通过 index 和 calls 来做了一些安全判断，避免重复处理，。

2. 将索引号 index +1，新创建一个 chain。

3. 根据目前的 index 获取拦截器，然后将新的 chain 传入到获取拦截器中。

4. 拦截器做完自己的操作后，会调用新创建的 chain 的 proceed 方法，交由下一个拦截器来处理。

5. 当数据返回后，从后往前，拦截器会依次对数据做一些处理，最终用户获得请求的数据。

通过上述往复循环，最终所有的拦截器都会走两遍，一次是对请求体做操作，一次是对返回体做操作，最终用户获得处理后的数据。



拦截器

RetryAndFollowUpInterceptor：重试和失败重定向拦截器 
BridgeInterceptor：桥接拦截器，处理一些必须的请求头信息的拦截器 
CacheInterceptor：缓存拦截器，用于处理缓存 
ConnectInterceptor：连接拦截器，建立可用的连接，是CallServerInterceptor的基本 
CallServerInterceptor：请求服务器拦截器将 http 请求写进 IO 流当中，并且从 IO 流中读取响应 Response




https://www.cnblogs.com/huansky/p/11772402.html
