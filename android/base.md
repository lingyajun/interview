##### Service
1. bindService 绑定的:onCreate->onBind->onUnBind->onDestory
(不管调用 bindService 几次，onCreate 只会调用一次，onStart 不会被调用，建立连接后，service 会一直运行，直到调用 unBindService 或是之前调用的 bindService 的 Context 不存在了，系统会自动停止 Service,对 应的 onDestory 会被调用)

2. startService 启动的:onCreate->onStartCommand->onDestory(start 多次，onCreate 只会被 调用一次，onStart 会调用多次，该 service 会在后台运行，直至被调用 stopService 或是 stopSelf)

3. 既被启动又被绑定的服务，不管如何调用 onCreate()只被调用一次，startService 调用多少 次，onStart 就会被调用多少次，而 unbindService 不会停止服务，必须调用 stopService 或是 stopSelf 来停止服务。
必须 unbindService 和 stopService(stopSelf)同时都调用了才会停止服 务。

##### BroadcastReceiver
a) 动态注册:存活周期是在 Context.registerReceiver 和 Context.unregisterReceiver 之间，
BroadcastReceiver 每次收到广播都是使用注册传入的对象处理的。
最后需要解绑，否会会内存泄露。

b) 静态注册:进程在的情况下，receiver 会正常收到广播，调用 onReceive 方法;
生命周期 只存活在 onReceive 函数中，此方法结束，BroadcastReceiver 就销毁了。
onReceive()只有十 几秒存活时间，在 onReceive()内操作超过 10S，就会报 ANR。
进程不存在的情况，广播相应的进程会被拉活，Application.onCreate 会被调用，再调用 onReceive。
这种注册的方式的广播是常驻型广播，会占用 CPU 的资源。
这种广播一般用于想开机自启动等等。

##### 请描述一下广播 BroadcastReceiver 的理解
BroadcastReceiver 是一种全局监听器，用来实现系统中不同组件之间的通信。
有时候也会用 来作为传输少量而且发送频率低的数据，但是如果数据的发送频率比较高或者数量比较大就 不建议用广播接收者来接收了，因为这样的效率很不好，因为 BroadcastReceiver 接收数据的 开销还是比较大的。

##### 本地广播和全局广播有什么差别?
1)LocalBroadcastReceiver 仅在自己的应用内发送接收广播，也就是只有自己的应用能收到， 数据更加安全。广播只在这个程序里，而且效率更高。
只能动态注册，在发送和注册的时候 采用 LocalBroadcastManager 的 sendBroadcast 方法和 registerReceiver 方法。
3)全局广播: 发送的广播事件可被其他应用程序获取，也能响应其他应用程序发送的广播 事件(可以通过 exported–是否监听其他应用程序发送的广播 在清单文件中控制) 全局 广播既可以动态注册，也可以静态注册。


##### ContentProvider
和应用的生命周期一样，它属于系统应用，	应用启动时，它会跟 着初始化，应用关闭或被杀，它会跟着结束。

##### Activity 之间的通信方式
1)通过 Intent 方式传递参数跳转
2)通过广播方式
3)通过接口回调方式
4)借助类的静态变量或全局变量
5)借助 SharedPreference 或是外部存储，如数据库或本地文件


AlertDialog 并不会影响 Activity 的生命周期，
按 Home 键后才会使 Activity 走 onPause->onStop， AlertDialog 只是一个组件，并不会使 Activity 进入后台。

##### 两个 Activity 之间跳转时必然会执行的是哪几个方法? 
前一个 Activity 的 onPause，后一个 Activity 的 onResume

##### service 和 activity 怎么进行数据交互?
1)通过 bindService 启动服务，可以在 ServiceConnection 的 onServiceConnected 中获取到 Service 的实例，这样就可以调用 service 的方法，
2)如果 service 想调用 activity 的方法，使用接口。可以 在 service 中定义接口类及相应的 set 方法，在 activity 中实现相应的接口，这样 service 就可 以回调接口言法;
3)通过广播方式。

##### 说说 ContentProvider、ContentResolver、ContentObserver 之间的关系 
ContentProvider 实现各个应用程序间数据共享，用来提供内容给别的应用操作。
如联系人应 用中就使用了 ContentProvider，可以在自己应用中读取和修改联系人信息，不过需要获取相 应的权限。
它也只是一个中间件，真正的数据源是文件或 SQLite 等。
ContentResolver 内容解析者，用于获取内容提供者提供的数据，通 ContentResolver.notifyChange(uri)发出消息；
ContentObserver 内容监听者，可以监听数据的改变状态，观察特定 Uri 引起的数据库变化， 继而做一些相应的处理，类似于数据库中的触发器，当 ContentObserver 所观察的 Uri 发生 变化时，便会触发它。

##### AlertDialog,popupWindow 区别
(1)Popupwindow 在显示之前一定要设置宽高，Dialog 无此限制。
(2)Popupwindow 默认不会响应物理键盘的 back，除非显示设置了 popup.setFocusable(true); 而在点击 back 的时候，Dialog 会消失。
(3)Popupwindow 不会给页面其他的部分添加蒙层，而 Dialog 会。
(4) Popupwindow 没有标题， Dialog 默认有标题，可以通过
dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);取消标题
(5)二者显示的时候都要设置 Gravity。如果不设置，Dialog 默认是 Gravity.CENTER。
(6)二者都有默认的背景，都可以通过 setBackgroundDrawable(new ColorDrawable(android.R.color.transparent));去掉。
(7)Popupwindow 弹出后，取得了用户操作的响应处理权限，使得其他 UI 控件不被触发。 而 AlertDialog 弹出后，点击背景，AlertDialog 会消失。

##### Application 和 Activity 的 Context 对象的区别
1)Application Context 是伴随应用生命周期;
	不可以 showDialog, startActivity, LayoutInflation
	可以 startService\BindService\sendBroadcast\registerBroadcast\load Resource values
2)Activity Context 指生命周期只与当前 Activity 有关，而 Activity Context 这些操作都可以，
	即凡是跟 UI 相关的，都得用 Activity 做为 Context 来处理。

一个应用 Context 的数量=Activity 数量+Service 数量+1(Application 数量)

