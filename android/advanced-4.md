##### 设计模式
单例模式
```
// Double Check 的写法
public class Singleton02 {
	private static Singleton02 instance;
	
	public static Singleton02 getInstance() {
			if (instance == null) {
				synchronized (Singleton02.class) {
				if (instance == null) {
					instance = new Singleton02();
					}
				} 
			}
		return instance; 
	}
	
}
```

冒泡排序
```
//从小到大排序
void bubbleSort_MinToMax(vector<int> &numberList)
{
	int N = numberList.size();
	for (int i = 1; i < N; i++)
	{
		for (int j = 0; j < N - i; j++)
		{
			if (numberList[j] > numberList[j + 1])
			{
				//交换
				int buffer = numberList[j];
				numberList[j] = numberList[j + 1];
				numberList[j + 1] = buffer;
			}
		}
	}
}
```

##### RecyclerView 和 ListView 的区别
RecyclerView 可以完成 ListView,GridView 的效果，还可以完成瀑布流的效果。
同时还可以设置列表的滚动方向(垂直或者水平);

RecyclerView 中 view 的复用不需要开发者自己写代码，系统已经帮封装完成了。
RecyclerView 可以进行局部刷新。
RecyclerView 提供了 API 来实现 item 的动画效果。

在性能上:
如果需要频繁的刷新数据，需要添加动画，则 RecyclerView 有较大的优势。
如果只是作为列表展示，则两者区别并不是很大。

##### OKhttp
OKhttp:Android 开发中是可以直接使用现成的 api 进行网络请求的。就是使用 HttpUrlConnection 进行操作。
okhttp 针对 Java 和 Android 程序，封装的一个高性 能的 http 请求库，支持同步，异步，而且 okhttp 又封装了线程池，封装了数据转换，封装 了参数的使用，错误处理等。
API 使用起来更加的方便。但是我们在项目中使用的时候仍然 需要自己在做一层封装，这样才能使用的更加的顺手。

##### Retrofit
Retrofit:Retrofit 是 Square 公司出品的默认基于 OkHttp 封装的一套 RESTful 网络请求框架， RESTful 是目前流行的一套 api 设计的风格， 并不是标准。
Retrofit 的封装可以说是很强大， 里面涉及到一堆的设计模式,可以通过注解直接配置请求，可以使用不同的 http 客户端，虽 然默认是用 http ，
可以使用不同 Json Converter 来序列化数据，
同时提供对 RxJava 的支持， 使用 Retrofit + OkHttp + RxJava + Dagger2 可以说是目前比较潮的一套框架，但是需要有比较 高的门槛。

