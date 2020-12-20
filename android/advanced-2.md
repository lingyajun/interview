##### 讲解一下 Context
Context 是一个抽象基类。在翻译为上下文，也可以理解为环境，是提供一些程序的运行环 境基础信息。
Context 有两个子类，ContextWrapper 是上下文功能的封装类，而 ContextImpl 则是上下文功能的实现类。
ContextWrapper 有三个直接的子类，ContextThemeWrapper、 Service 和 Application。
ContextThemeWrapper 是一个带主题的封装类，而它有一个直 接子类就是 Activity.只有 Activity 需要主题，Service 不需要主题.

Context 一共有三种类型，分别是 Application、Activity 和 Service。
这三个类虽然分别各种承担着不同的作用，但它们都属于 Context 的一种，而它 们具体 Context 的功能则是由 ContextImpl 类去实现的，因此在绝大多数场景下，Activity、 Service 和 Application 这三种类型的 Context 都是可以通用的。

getApplicationContext()和 getApplication()方法得到的对象都是同一个 application 对象，只是 对象的类型不一样。

出于安全原因的考虑，Android 是不允许 Activity 或 Dialog凭空出现的，
一个 Activity 的启动必须要建立在另一个 Activity 的基础之上，也就是以此形 成的返回栈。
Dialog 则必须在一个 Activity 上面弹出(除非是 System Alert 类型的 Dialog)， 因此在这种场景下，我们只能使用 Activity 类型的 Context，否则将会出错。


##### 理解 Activity，View,Window 三者关系
Activity —— 控制单元 (像一个工匠)
Window —— 承载模型 (像窗户)
View —— 显示视图 (像窗花)
LayoutInflater 像剪刀，Xml 配置像窗花图纸。

1:Activity 构造的时候会初始化一个 Window，准确的说是 PhoneWindow。
2:这个 PhoneWindow 有一个“ViewRoot”，这个“ViewRoot”是一个 ViewGroup，是最初始的根视图。
3: “ViewRoot”通过 addView 方法来一个个的添加 View。比如 TextView，Button 等
4: 这些View的事件监听，是由WindowManagerService来接受消息，并且回调Activity函数。比如 onClickListener，onKeyDown 等。


##### Android 中跨进程通讯的几种方式
Android 跨进程通信，像 intent，contentProvider,广播，service 都可以跨进程通信。
intent:这种跨进程方式并不是访问内存的形式，它需要传递一个 uri,比如说打电话。
contentProvider:这种形式，是使用数据共享的形式进行数据共享。
service:远程服务，aidl

##### AIDL 理解
每一个进程都有自己的 Dalvik VM 实例，都有自己的一块独立的内存，都在自己的内 存上存储自己的数据，执行着自己的操作，都在自己的那片狭小的空间里过完自己的一生。

aidl 就类似与两个进程之间的桥梁，使得两个进程之间可以进行数据的传输，
跨进程通信 有多种选择，比如 BroadcastReceiver , Messenger 等，
BroadcastReceiver 占用的系统 资源比较多，如果是频繁的跨进程通信的话显然是不可取的;
Messenger 进行跨进程通信时 请求队列是同步进行的，无法并发执行。

##### Binde 机制简单理解:
在 Android 系统的 Binder 机制中，是有 Client,Service,ServiceManager,Binder驱动程序 组成的，
其中 Client，service，Service Manager 运行在用户空间，
Binder 驱动程序是运行在内核空间 的。
Binder 就是把这 4 种组件粘合在一块的粘合剂，其中核心的组件就是 Binder 驱动程 序，
ServiceManager 提供辅助管理的功能，
Client 和 Service 正是在 Binder 驱动程序和 ServiceManager 提供的基础设施上实现 C/S 之间的通信。

Binder 驱动程序 提供设备文件/dev/binder 与用户控件进行交互，
Client、Service，ServiceManager 通过 open 和 ioctl 文件操作相应的方法与 Binder 驱动程序 进行通信。

Client 和 Service 之间的进程间通信是通过 Binder 驱动程序间接实现的。
BinderManager 是一个守护进程，用来管理 Service，并向 Client 提供查询 Service 接口的能力。


##### Handler 的原理
它的作用就是实现线程之间的通信。
Handler 整个流程中，主要有四个对象，Handler，Message,MessageQueue,Looper。

需要传送的消息保存到 Message 中，handler 通过调用 sendMessage 方法将 Message 发送到 MessageQueue 中，
Looper 对象就会不断的调用 loop()方法不断的从 MessageQueue 中取出 Message 交给 handler 进行处理。 从而实现线程之间的通信。

##### 热修复的原理
(Java 虚拟机和) Android 虚拟机  加载类的时候(都)需要 ClassLoader,
ClassLoader 有一个子类 BaseDexClassLoader,
BaseDexClassLoader 下有一个数组——DexPathList，是用来存放 dex 文件，
当 BaseDexClassLoader 通过调用 findClass 方法时，实际上就是遍历数组，找到相应的 dex 文件，找到，则直接将它 return。
**热修复的解决方法就是将新的 dex 添加到该集合中**，并且是在旧的 dex 的前面，就会优先被取出来并且 return 返回。

