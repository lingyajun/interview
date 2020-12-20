 **系统广播的源码角度分析**
  a. 自定义广播接收者 BroadcastReceiver,并且重写 onReceiver()方法。
  b. 通过 Binder 机制向 AMS(Activity Manager Service)进行注册。
  c. 广播发送者通过 Binder 机制向 AMS 发送广播。
  d. AMS 查找符合条件(IntentFilter/Permission 等)的 BroadcastReceiver，将广播发送到相应的 BroadcastReceiver(一般情况下是 Activity)的消息队列中。
	e. 消息循环执行拿到此广播，回调 BroadcastReceiver 中的 onReceiver()方法。
	

**本地广播的源码角度分析**
a. LocalBroadcast 高效的原因: 它内部是通过 Handler 实现的，它的 sendBroadcast()方法其实就是通过 Handler 发送了一个 Message 而已。
b. LocalBroadcast 安全的原因: 它是通过 Handler 实现广播发送的，别的 app 无法向我们应用发送该广播，而我们 app 内部发送的广播也不会离开我们的 app。

LocalBroadcast 内部协作主要是靠
两个 Map 集合:mReceivers 和 mActions,
一个 List 集合 mPendingBroadcasts,这个主要存储待接收的广播对象。



**Fragment 为什么会被称为第五大组件**?
Fragment 有生命周期，使用频率不输于 4 大组件，可灵活加载到 Activity 中。
Android 中的 4 大组件为:Activity，Broadcast，Service，ContentProvider.

Fragment 加载到 Activity 的方式分为静态加载和动态加载,
静态加载: 直接在 Activity 布局文件中指定 Fragment;
动态加载: 动态加载需要使用到 FragmentManager.

  
	Java 提供ReferenceQueue 来处理引用对象的回收情况。
	当SoftReference 所引用的对象被GC 后，JVM 会先将softReference 对象添加到ReferenceQueue
这个队列中。
当我们调用ReferenceQueue 的 poll() 方法，如果这个队列不是空队列，那么将返回并移除前面添加的那个 Reference 对象.


Glide的源码设计哪里很微妙?
- Glide的生命周期绑定:可以控制图片的加载状态与当前页面的生命周期同步，使整个加载过程随着页面的状态而 启动/恢复，停止，销毁.
- Glide的缓存设计:通过(三级缓存，Lru算法，Bitmap复用)对 Resource 进行缓存设计。
- Glide的完整加载过程: 采用Engine 引擎类暴露了 一系列方法供 Request 操作

