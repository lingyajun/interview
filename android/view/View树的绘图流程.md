> 整个 View 树的绘图流程在 ViewRoot.java 类的 performTraversals()函数展开，该函数 所做 的工作可简单概况为是否需要重新计算视图大小(measure)、是否需要重新 安置视图的位置(layout)、以及是否需要重绘(draw)

measure -> layout -> draw

树的遍历是有序的，由父视图到子视图，每一个 ViewGroup 负责测绘它所有的子视图，而最底层的 View 会负责测绘自身。

##### measure
measure 过程由 measure(int, int)方法发起，从上到下有序的测量 View，
在measure 过程的最后，每个视图存储了自己的尺寸大小和测量规格。

onMeasure(int widthMeasureSpec, int heightMeasureSpec)
该方法就是我们自定义视图中实现测量逻辑的方法，该方法的参数是父视图对子 视图的 width 和 height 的测量要求。

setMeasuredDimension()
测量阶段终极方法，在 onMeasure(int widthMeasureSpec, int heightMeasureSpec) 方法中调用，将计算得到的尺寸，传递给该方法，测量阶段即结束。
该方法也是必须要调用的方法，否则会报异常。

##### layout
要明确的是，子视图的具体位置都是相对于父视图而言的。

在 layout 过程中，子视图会调用 getMeasuredWidth()和 getMeasuredHeight()
方法获取到 measure 过程得到的 mMeasuredWidth 和 mMeasuredHeight，作为自己的 width 和 height。
然后调用每一个子视图的 layout(l, t, r, b)函数， 来确定每个子视图在父视图中的位置。

##### draw

View 的 onDraw(Canvas)默认是空实现，自定义绘制过程需要复写的方法，绘制自身的内容。

dispatchDraw() 发起对子视图的绘制。
ViewGroup 复写 了 dispatchDraw()来对其子视图进行绘制。

invalidate()请求重绘 View 树，即 draw 过程，假如视图发生大小没有变化就不会调用 layout()过程，并且只绘制那些调用了 invalidate()方法的 View。

requestLayout()当布局变化的时候，比如方向变化，尺寸的变化，会调用该方 法，在自定义的视图中，如果某些情况下希望重新测量尺寸大小，应该手动去调 用该方法，它会触发 measure()和 layout()过程，但不会进行 draw。

----

当 Activity 接收到焦点的时候，它会被请求绘制布局,该请求由 Android framework 处理.
绘制是从根节点开始，对布局树进行 measure 和 draw。
```
View 随着 Activity 的创建而加载，startActivity 启动一个 Activity 时，在 ActivityThread 的 handleLaunchActivity 方法中会执行 Activity 的 onCreate 方法，
这个时候会调用 setContentView 加载布局创建出 DecorView 并将我们的 layout 加载到 DecorView 中，
当执行到 handleResumeActivity 时，Activity 的 onResume 方法被调用，然后 WindowManager 会将 DecorView 设置给 ViewRootImpl,这样， DecorView 就被加载到 Window 中了，此时界面还没有显示出来，
还需要经过 View 的 measure，layout 和 draw 方法，才能完成 View 的工作流程。

我们需要知道 View 的绘制是由 ViewRoot 来负责的，每一个 DecorView 都有一个与之关联的 ViewRoot, 
这种关联关系是由 WindowManager 维护的，
将 DecorView 和 ViewRoot 关联之后， ViewRootImpl 的 requestLayout 会被调用以完成初步布局，通过 scheduleTraversals 方法向主线程发送消息请求遍历，
最终调用 ViewRootImpl 的 performTraversals 方法，这个方法会执行 View 的 measure layout 和 draw 流程
```
