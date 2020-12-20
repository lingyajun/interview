#### 具体说说你理解的MVVM
1）先说说MVVM是怎么解决了其他两个架构所在的缺陷和问题：
- 解决了各个层级之间耦合度太高的问题，也就是更好的完成了解耦。
MVP层中，Presenter还是会持有View的引用，但是在MVVM中，View和Model进行双向绑定，从而使viewModel基本只需要处理业务逻辑，无需关系界面相关的元素了。

- 解决了代码量太多，或者模式化代码太多的问题。由于双向绑定，所以UI相关的代码就少了很多，这也是代码量少的关键。而这其中起到比较关键的组件就是DataBinding，使所有的UI变动都交给了被观察的数据模型。

- 解决了可能会有的内存泄漏问题。
MVVM架构组件中有一个组件是LiveData，它具有生命周期感知能力，可以感知到Activity等的生命周期，所以就可以在其关联的生命周期遭到销毁后自行清理，就大大减少了内存泄漏问题。

- 解决了因为Activity停止而导致的View空指针问题。在MVVM中使用了LiveData，那么在需要更新View的时候，如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。
也就是他会保证在界面可见的时候才会进行响应，这样就解决了空指针问题。

- 解决了生命周期管理问题。这主要得益于Lifecycle组件，它使得一些控件可以对生命周期进行观察，就能随时随地进行生命周期事件。


2）再说说响应式编程
响应式编程，说白了就是我先构建好事物之间的关系，然后就可以不用管了。
他们之间会因为这层关系而互相驱动。
其实就是我们常说的观察者模式，或者说订阅发布模式。

MVVM的本质思想就是这种。
不管是双向绑定，还是生命周期感知，其实都是一种观察者模式，使所有事物变得可观察，那么我们只需要把这种观察关系给稳定住，那么项目也就稳健了。

3）最后再说说MVVM为什么这么强大？
因为这种响应式编程的优势比较大，再加上Google官方的大力支持，出了这么多支持的组件，来维系MVVM架构，其实也是官方想进行项目架构的统一。

ViewModel是MVVM架构的一个层级，用来联系View和model之间的关系。
注重生命周期的方式。
由于ViewModel的生命周期是作用于整个Activity的，所以就节省了一些关于状态维护的工作，
最明显的就是对于屏幕旋转这种情况，以前对数据进行保存读取，而ViewModel则不需要，他可以自动保留数据。

其次，由于ViewModel在生命周期内会保持局部单例，所以可以更方便Activity的多个Fragment之间通信，因为他们能获取到同一个ViewModel实例，也就是数据状态可以共享了。


**ViewModel 为什么被设计出来，解决了什么问题？**
**在ViewModel组件被设计出来之前，MVVM又是怎么实现ViewModel这一层级的呢？**

就是自己编写类，然后通过接口，内部依赖实现View和数据的双向绑定。
所以Google出这个ViewModel组件，就是为了规范MVVM架构的实现，并尽量让ViewModel这一层级只触及到业务代码，不去关心VIew层级的引用等。
然后配合其他的组件，包括livedata，databindingrang等让MVVM架构更加完善，规范，健硕。


**解决了什么问题呢？**
比如：
1）不会因为屏幕旋转而销毁，减少了维护状态的工作
2）由于在作用域内单一实例的特性，使得多个fragment之间可以方便通信，并且维护同一个数据状态。
3）完善了MVVM架构，使得解耦更加纯粹。


**说说ViewModel原理**
1. **首先说说是怎么保存生命周期。**

ViewModel2.0之前呢，其实原理是在Activity上add一个HolderFragment，然后设置setRetainInstance(true)方法就能让这个Fragment在Activity重建时存活下来，也就保证了ViewModel的状态不会随Activity的状态所改变。
总结来说就是用一个空的fragment来管理维护ViewModelStore，然后对应的activity销毁的时候就去把viewmodel的映射删除。就让ViewModel的生命周期保持和Activity一样了。
这也是很多三方库用到的巧妙方法，比如Glide，也是建立空的Fragment来管理。

2.0之后，其实是用到了Activity的onRetainNonConfigurationInstance()和getLastNonConfigurationInstance()这两个方法，相当于在横竖屏切的时候会保存ViewModel的实例，然后恢复，所以也就保证了ViewModel的数据。
2.0之后，有了androidx支持,是用到了Activity的一个子类ComponentActivity，然后重写了onRetainNonConfigurationInstance()方法保存ViewModelStore，并在需要的时候，也就是重建的Activity中去通过getLastNonConfigurationInstance()方法获取到ViewModelStore实例。
这样也就保证了ViewModelStore中的ViewModel不会随Activity的重建而改变。

同时由于实现了LifecycleOwner接口，所以能利用Lifecycles组件组件感知每个页面的生命周期，就可以通过它来订阅当Activity销毁时，且不是因为配置导致的destory情况下，去清除ViewModel

2.0之前呢，都是通过他们创建了一个空的fragment，然后跟随这个fragment的生命周期。

2.0之后呢，是因为不管是Activity或者Fragment，都实现了LifecycleOwner接口，所以ViewModel是可以通过Lifecycles感知到他们的生命周期，从而进行实例管理的。


2. **再说说怎么保证作用域内唯一实例**

首先，ViewModel的实例是通过反射获取的，反射的时候带上application的上下文，这样就保证了不会持有Activity或者Fragment等View的引用。
然后实例创建出来会保存到一个ViewModelStore容器里面，其实也就是一个集合类，这个ViewModelStore 类其实就是保存在界面上的那个实例，而我们的ViewModel就是里面的一个集合类的子元素。

所以我们每次获取的时候，首先看看这个集合里面有无我们的ViewModel，
如果没有就去实例化，如果有就直接拿到实例使用，这样就保证了唯一实例。


**LiveData 是什么**？
主要作用在两点：
- 数据存储器类。也就是一个用来存储数据的类。

- 可观察。这个数据存储类是可以观察的，也就是比一般的数据存储类多了这么一个功能，
对于数据的变动能进行响应。


**LiveData 为什么被设计出来，解决了什么问题**？
LiveData作为一种观察者模式设计思想，观察者模式的最大好处就是事件发射的上游 和 接收事件的下游 互不干涉，大幅降低了互相持有的依赖关系所带来的强耦合性。

其次，LiveData还能无缝衔接到MVVM架构中，主要体现在其可以感知到Activity等生命周期，这样就带来了很多好处：
- 不会发生内存泄漏 观察者会绑定到 Lifecycle对象，并在其关联的生命周期遭到销毁后进行自我清理。
- 不会因 Activity 停止而导致崩溃 如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。
- 自动判断生命周期并回调方法 如果观察者的生命周期处于 STARTED 或 RESUMED状态，则 LiveData 会认为该观察者处于活跃状态，就会调用onActive方法，否则，如果 LiveData 对象没有任何活跃观察者时，会调用 onInactive()方法。


**说说LiveData原理**
说到原理，其实是两个方法：
订阅方法，也就是observe方法。通过该方法把订阅者和被观察者关联起来，形成观察者模式。
回调方法，也就是onChanged方法。通过改变存储值，来通知到观察者也就是调用onChanged方法。
