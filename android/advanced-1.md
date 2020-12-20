##### Activity 的启动过程  (不要回答生命周期)
app 启动的过程有两种情况，第一种是从桌面 launcher 上点击相应的应用图标，第二种是在
activity 中通过调用 startActivity 来启动一个新的 activity。

我们从桌面点击应用 图标的时候，系统就会调用 startActivitySately()，这个方法内 部则是调用 startActivty(),
startActivity()方法最终还是会调用 startActivityForResult()这个方 法。
因为 startActivityForResult()方法是有返回结果的， 所以系统就直接给一个-1，就表示不需要结果返回了。
startActivityForResult()这个方法实 际是通过 Instrumentation 类中的 execStartActivity()方法来启动 activity，Instrumentation 这个类主要作用就是监控程序和系统之间的交互。
在这个 execStartActivity()方法中会获取 ActivityManagerService 的代理对象，通过这个代理对象进行启动 activity。
启动会就会调用一 个 checkStartActivityResult()方法，如果说没有在配置清单中配置有这个组件，就会在这个方 法中抛出异常了。
最后是调用的是 Application.scheduleLaunchActivity()进行启动 activity，
这个方法中通过获取得到一个 ActivityClientRecord 对象，这个 ActivityClientRecord 通过 handler 来进行消息的发送。
系统内部会将每一个 activity 组件使用 ActivityClientRecord 对象 来进行描述。
 ActivityClientRecord 对象中保存有一个 LoaderApk 对象，通过这个对象调用 handleLaunchActivity 来启动 activity 组件，页面的生命周期方法也就是在这个方法中进行 调用。


##### HttpClient 与 HttpUrlConnection 的区别
首先 HttpClient 和 HttpUrlConnection 这两种方式都支持 Https 协议，
都是以流的形式进行上传或者下载数据，也可以说是以流的形式进行数据的传输，
还有 ipv6,以及连接池等功能。

HttpClient 这个拥有非常多的 API，所以如果想要进行扩展的话，并且不破坏它的兼容性的话，很难进行扩展，也就是这个原因，Google 在 Android6.0 的时候，直接就弃用了这个 HttpClient.

HttpUrlConnection 相对来说就是比较轻量级了，API 比较少，容易扩展，并且能够满足 Android 大部分的数据传输。
比较经典的一个框架 volley，在 2.3 版本以前都是使用 HttpClient, 在 2.3 以后就使用了 HttpUrlConnection。

##### java 虚拟机和 Dalvik 虚拟机的区别
Java 虚拟机:
1、java 虚拟机基于栈。 基于栈的机器必须使用指令来载入和操作栈上数据，所需指令更多 。
2、java 虚拟机运行的是 java 字节码。(java 类会被编译成一个或多个字节码.class 文件)

Dalvik 虚拟机:
1、dalvik 虚拟机是基于寄存器的
2、Dalvik 运行的是自定义的.dex 字节码格式。
(java 类被编译成.class 文件后，会通过一个 dx 工具将所有的.class 文件转换成一个.dex 文件，然后 dalvik 虚拟机会从其中读取指令和数据)
3、常量池已被修改为只使用 32 位的索引，以 简化解释器。
4、一个应用，一个虚拟机实例，一个进程
(所有 android 应用的线程都是对应一个 linux 线程，都运行在自己的沙盒中，不同的应用在不同的进程中运行。每个 android dalvik 应用程 序都被赋予了一个独立的 linux PID(app_*))


##### 进程保活(不死进程)
当前业界的 Android 进程保活手段主要分为 **黑、白、灰** 三种，其大致的实现思路如下: 
黑色保活:不同的 app 进程，用广播相互唤醒(包括利用系统提供的广播进行唤醒) .
举个 3 个比较常见的场 景:
场景 1:开机，网络切换、拍照、拍视频时候，利用系统产生的广播唤醒 app
场景 2:接入第三方 SDK 也会唤醒相应的 app 进程，如微信 sdk 会唤醒微信，支付宝 sdk 会
唤醒支付宝。由此发散开去，就会直接触发了下面的 场景 3
场景 3:假如你手机里装了支付宝、淘宝、天猫、UC 等阿里系的 app，那么你打开任意一个 阿里系的 app 后，有可能就顺便把其他阿里系的 app 给唤醒了。(只是拿阿里打个比方，其 实 BAT 系都差不多)

白色保活:启动前台 Service.
白色保活手段非常简单，就是调用系统 api 启动一个前台的 Service 进程，这样会在系统的 通知栏生成一个 Notification，用来让用户知道有这样一个 app 在运行着.

灰色保活:利用系统的漏洞启动前台 Service.
这种保活手段是应用范围最广泛。它是利用系统的漏洞来启动一个前台的 Service 进程，与普通的启动方式区别在于，它不会在系统通知栏处出现一个 Notification，看起来就 如同运行着一个后台 Service 进程一样。
大致的实现思路和代码 :
思路一:API < 18，启动前台 Service 时直接传入 new Notification();
思路二:API >= 18，同时启动两个 id 相同的前台 Service，然后再将后启动的 Service 做 stop
处理.

进程的重要性，划分 5 级:
前台进程 (Foreground process)
可见进程 (Visible process) 
服务进程 (Service process)
后台进程 (Background process)
空进程 (Empty process)

什么是 oom_adj?
它是 linux 内核分配给 每个系统进程的一个值，代表进程的优先级，进程回收机制就是根据这个优先级来决定是否 进行回收。
进程的 oom_adj 越大，表示此进程优先级越低，越容易被杀回收;越小，表示进程优先级 越高，越不容易被杀回收

普通 app 进程的 oom_adj>=0,系统进程的 oom_adj 才可能<0

有些手机厂商把这些知名的 app 放入了自己的白名单中，保证了进程不死来提高用户体验 (如微信、QQ、陌陌都在小米的白名单中)。
如果从白名单中移除，他们终究还是和普通 app 一样躲避不了被杀的命运，为了尽量避免被杀，还是老老实实去做好优化工作吧。
