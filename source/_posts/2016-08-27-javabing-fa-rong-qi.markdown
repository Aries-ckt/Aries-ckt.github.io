---
layout: post
title: "Java并发容器"
date: 2016-08-27 14:47:46 +0800
comments: true
categories: Java并发
---
### 并发容器（CopyOnWrite系列，ConcurrentHashMap）
#### copyOnWriteArrayList
get()没有加锁

```
public E get(int index) {
        return get(getArray(), index);
}

```
add()，加锁

```
public boolean add(E e){
    final ReentrantLock lock = this.lock;
    lock.lock();
    try{
        Object[] elements=getArray();
        int len=elements.length;
        Object[] newElements=Arrays.copyOf(elements,len+1);
        newElements[len]=e;
        setArray(newElements);
        return true;
    } finally{
        lock.unlock();
    }
}
```
### ConcurrentHashMap
ConcurrentHashMap数据结构：（是一个Segment数组）

```
final Segment<K,V>[] segments;
```
Segment的数据结构：（是一个HashEntry的数组）

```
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    ...
    transient volatile HashEntry<K,V>[] table; //发现Segment实际上是HashEntry数组
    ...
}
```
HashEntry的数据结构：
```
static final class HashEntry<K,V> {
    final K key;
    final int hash;
    volatile V value;
    final HashEntry<K,V> next;
    ...
}
```
所涉及的方法：
不用加锁的：get，containsKey等 
段（Segment）内加锁的:put，putIfAbsent，remove，replace等 
整个数据结构加锁的：size，containsValue等

构造方法：

```
public ConcurrentHashMap(int initialCapacity,float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();  //参数合法性校验
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS; 
    //比如我输入的concurrencyLevel=12，那么sshift = 4，ssize =16，所以sshift是意思就是1左移了几次比concurrencyLevel大，ssize就是那个大于等于concurrencyLevel的最小2的幂次方的数
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;  //ssize = ssize << 1 , ssize = ssize * 2
    }
    segmentShift = 32 - sshift;
    segmentMask = ssize - 1;   //segmentMask的二进制是一个全是1的数 
    this.segments = Segment.newArray(ssize); //segment个数是ssize，也就是上图黄色方块数，默认16个
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = 1;
    while (cap < c)
        cap <<= 1;   
    for (int i = 0; i < this.segments.length; ++i)
        this.segments[i] = new Segment<K,V>(cap, loadFactor);//上图浅绿色块的个数是cap，是一个2的幂次方的数，默认是1，也就是每个segment下都构造了cap大小的table数组
}
Segment(int initialCapacity, float lf) {
    loadFactor = lf;
    setTable(HashEntry.<K,V>newArray(initialCapacity));//构造了一个initialCapacity大小的table
}
```
put方法：

```
public V put(K key, V value) {
    if (value == null)
        throw new NullPointerException(); //明确指定value不能为null
    int hash = hash(key.hashCode());
    return segmentFor(hash).put(key, hash, value, false); //segmentFor如下，定位segment下标，然后再put
}

final Segment<K,V> segmentFor(int hash) {         //寻找segment的下标
    return segments[(hash >>> segmentShift) & segmentMask];     //前面说了segmentMask是一个2进制全是1的数，那么&segmentMask就保证了下标小于等于segmentMask，与HashMap的寻下标相似。
}

//真正的put操作
V put(K key, int hash, V value, boolean onlyIfAbsent) {
    lock();          //先加锁，可以看到，put操作是在segment里面加锁的，也就是每个segment都可以加一把锁
    try {
        int c = count;
        if (c++ > threshold) // ensure capacity
            rehash();   //判断容量，如果不够了就扩容
        HashEntry<K,V>[] tab = table;      //将table赋给一个局部变量tab，这是因为table是volatile变量，读写volatile变量的开销很大，编译器也不能对volatile变量的读写做任何优化，直接多次访问非volatile实例变量没有多大影响，编译器会做相应优化。
        int index = hash & (tab.length - 1);   //寻找table的下标
        HashEntry<K,V> first = tab[index];
        HashEntry<K,V> e = first;
        while (e != null && (e.hash != hash || !key.equals(e.key)))
            e = e.next;  //遍历单链表，找到key相同的为止，如果没找到，e指向链表尾

        V oldValue;
        if (e != null) { //如果有相同的key，那么直接替换
            oldValue = e.value;
            if (!onlyIfAbsent)
                e.value = value;
        }
        else {  //否则在链表表头插入新的结点
            oldValue = null;
            ++modCount;
            tab[index] = new HashEntry<K,V>(key, hash, first, value);
            count = c; // write-volatile
        }
        return oldValue;
    } finally {
        unlock();
    }
```
该方法也是在持有段锁的情况下执行的，首先判断是否需要rehash，需要就先rehash，扩容都是针对单个段的，也就是单个段的数据数量大于设定的量的时候会触发扩容。接着是找是否存在同样一个key的结点，如果存在就直接替换这个结点的值。否则创建一个新的结点并添加到hash链的头部，这时一定要修改modCount和count的值，同样修改count的值一定要放在最后一步。put方法调用了rehash方法，reash方法实现得也很精巧，主要利用了table的大小为2^n，和HashMap的扩容基本一样，这里就不介绍了。

还有一个叫putIfAbsent(K key, V value)的函数，这个函数的实现和put几乎一模一样，作用是，如果map中不存在这个key，那么插入这个数据，如果存在这个key，那么不覆盖原来的value，也就是不插入了。

get方法

```
public V get(Object key) {
    int hash = hash(key.hashCode());   //双hash，和HashMap一样，也是为了更好的离散化
    return segmentFor(hash).get(key, hash);  //先寻找segment的下标，然后再get
}

final Segment<K,V> segmentFor(int hash) {      //寻找segment的下标
    return segments[(hash >>> segmentShift) & segmentMask];  //前面说了segmentMask是一个2进制全是1的数，那么&segmentMask就保证了下标小于等于segmentMask，与HashMap的寻下标相似。
}

V get(Object key, int hash) {// count是一个volatile变量，count非常巧妙，每次put和remove之后的最后一步都要更新count，就是为了get的时候不让编译器对代码进行重排序，来保证
    if (count != 0) { 
        HashEntry<K,V> e = getFirst(hash);   //寻找table的下标，也就是链表的表头
        while (e != null) {
            if (e.hash == hash && key.equals(e.key)) {
                V v = e.value;
                if (v != null)
                    return v;
                return readValueUnderLock(e); // recheck  加锁读，这个加锁读不用重新计算位置，而是直接拿e的值
            }
            e = e.next;
        }
    }
    return null;
}

HashEntry<K,V> getFirst(int hash) {
    HashEntry<K,V>[] tab = table;
    return tab[hash & (tab.length - 1)];
}
```
size方法：

```
public int size() {
    final Segment<K,V>[] segments = this.segments;
    long sum = 0;
    long check = 0;
    int[] mc = new int[segments.length];
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    for (int k = 0; k < RETRIES_BEFORE_LOCK; ++k) {
        check = 0;
        sum = 0;
        int mcsum = 0;
        for (int i = 0; i < segments.length; ++i) {
            sum += segments[i].count;           //循环相加每个段内数据的个数
            mcsum += mc[i] = segments[i].modCount;        //循环相加每个段内的modCount
        }
        if (mcsum != 0) {                   //如果是0，代表根本没有过数据更改，也就是size是0
            for (int i = 0; i < segments.length; ++i) {
                check += segments[i].count;    //再次循环相加每个段内数据的个数，这里为什么会再算一次呢，后面会说
                if (mc[i] != segments[i].modCount) {    
                    check = -1; // force retry  //如果modCount和之前统计的不一致了，check直接赋值-1，重新再来
                    break;
                }
            }
        }
        if (check == sum)
            break;
    }
    if (check != sum) { // Resort to locking all segments
        sum = 0;
        for (int i = 0; i < segments.length; ++i)  //循环获取所有segment的锁
            segments[i].lock();
        for (int i = 0; i < segments.length; ++i)  //在持有所有段的锁的时候进行count的相加
            sum += segments[i].count;
        for (int i = 0; i < segments.length; ++i)  //循环释放所有段的锁
            segments[i].unlock();
    }
    if (sum > Integer.MAX_VALUE)            //return
        return Integer.MAX_VALUE;
    else
        return (int)sum;
}
```
  如果我们要统计整个ConcurrentHashMap里元素的大小，就必须统计所有Segment里元素的大小后求和。Segment里的全局变量count是一个volatile变量，那么在多线程场景下，我们是不是直接把所有Segment的count相加就可以得到整个ConcurrentHashMap大小了呢？不是的，虽然相加时可以获取每个Segment的count的最新值，但是拿到之后可能累加前使用的count发生了变化，那么统计结果就不准了。所以最安全的做法，是在统计size的时候把所有Segment的put，remove和clean方法全部锁住，但是这种做法显然非常低效，因为在累加count操作过程中，之前累加过的count发生变化的几率非常小，所以ConcurrentHashMap的做法是先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小。那么ConcurrentHashMap是如何判断在统计的时候容器是否发生了变化呢？使用modCount变量，在put , remove和clear方法里操作元素前都会将变量modCount进行加1，那么在统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生变化。size()的实现还有一点需要注意，必须要先segments[i].count，才能segments[i].modCount，这是因为segment[i].count是对volatile变量的访问，接下来segments[i].modCount才能得到几乎最新的值，这里和get方法的方式是一样的，也是一个volatile写 happens-before volatile读的问题。

上面18行代码，抛出了一个问题，就是为什么会再算一遍，上面说只需要比较modCount不变不就可以了么？但是仔细分析，就会发现，在13行14行代码那里，计算完count之后，计算modCount之前有可能count的值又变了，那么18行的代码主要是解决这个问题。