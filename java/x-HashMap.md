[HashMap和Hashtable的区别](https://www.jianshu.com/p/d3664a1e7c2b)
HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。

主要的区别有：线程安全性，同步(synchronization)，以及速度。
1. HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
2. HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。
3. 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
4. 由于Hashtable是线程安全的也是synchronized，所以在单线程环境下它比HashMap要慢。如果你不需要同步，只需要单一线程，那么使用HashMap性能要好过Hashtable。
5. HashMap不能保证随着时间的推移Map中的元素次序是不变的。


**一些术语**：
1. sychronized 意味着在一次仅有一个线程能够更改Hashtable。就是说任何线程要更新Hashtable时要首先获得同步锁，其它线程要等到同步锁被释放之后才能再次获得同步锁更新Hashtable。
2. Fail-safe和iterator迭代器相关。如果某个集合对象创建了Iterator或者ListIterator，然后其它的线程试图“结构上”更改集合对象，将会抛出ConcurrentModificationException异常。但其它线程可以通过set()方法更改集合对象是允许的，因为这并没有从“结构上”更改集合。但是假如已经从结构上进行了更改，再调用set()方法，将会抛出IllegalArgumentException异常。
3. 结构上的更改指的是删除或者插入一个元素，这样会影响到map的结构。

**能否让HashMap同步**？
HashMap可以通过下面的语句进行同步：
```
Map m = Collections.synchronizeMap(hashMap);
```

---
[HashMap底层实现原理以及工作原理 ](https://www.jianshu.com/p/0a6f8c500c31)

**HashMap的put实现原理**：

第一步：将k和v封装到Node对象当中(节点)

第二步：底层调用k的hashCode()方法得出hash值

第三步：通过哈希表函数/哈希算法，将hash值转换成数组的下标。

**HashMap的get实现原理**：

第一步：调用k的hashCode()方法得出哈希值，通过哈希算法转换成数组下标。

第二步：通过数组下标快速定位到某个位置上。注意：如果这个位置上什么都没用，则返回null。

**随机增删、查询效率都很高的原因是**  
增删是在链表上完成的，而查询只需扫描部分，则效率高.

**HashMap底层实现原理**
数组+链表

**Hash冲突**：
HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。

