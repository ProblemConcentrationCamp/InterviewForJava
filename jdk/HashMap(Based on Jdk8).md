# HashMap (基于 Jdk8 的实现)

## 1. HashMap 的内部的数据结构

![Alt 'HashMap-DataStructure'](https://s2.ax1x.com/2019/12/03/QMLTMQ.png)

如图: HashMap 内部是通过一个数组存储数据的，只是数组的项是一个链表，或者是一棵红黑树。 

其中链表的结构
```java
static class Node<K,V> implements Map.Entry<K,V> {

    /**
     * 存储的节点的 key 计算出来的 hash 值
     */
    final int hash;

    /**
     * key 值
     */
    final K key;

    /**
     * value 值
     */
    V value;

    /*
     * 下一个节点
     */
    Node<K,V> next;
}
```

红黑树的结构

```java

/**
 * 继承了 LinkedHashMap.Entry, Entry 继承了 HashMap.Node  所以 TreeNode 具有 链表的特点
 */
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {

    /**
     * 父节点
     */
    TreeNode<K,V> parent;

    /*
     * 左子节点
     */
    TreeNode<K,V> left;

    /**
     * 右子节点
     */
    TreeNode<K,V> right;

    /*
     * 前置节点
     */
    TreeNode<K,V> prev;
    
    /*
     * 红黑标识
     */
    boolean red;
}
```

通过 Node 和 TreeNode 我们可以知道，HashMap 的数据是存在 table 数组里面， 这个数组的类型是 Node, 而 Node 的实现可以为 Node 本身, 也就是链表 和 TreeNode 红黑树。

## 2. HashMap 中几个比较重要的属性介绍
```java

public class HashMap<K,V> {

    /**
     * 数据存储的地方, 后文说到的 table，就是指这一个了数组
     */
    transient Node<K,V>[] table;

    /**
     * 16, 初始时，没有指定容量时，我们 table 的大小， 
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

    /**
     * 2^30 table 的最大上限
     * 之所以不是 Integer.MAX_VALUE， 是因为 HashMap 需要维护 table 的长度是 2 的 n 次方
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认的负载因子， 
     * 在 HashMap 的 table，是不会在存满的时候，才进行扩容的
     * 默认情况下会在 table 的存储的长度 = 总容量 * 负载因子的时候，就进行扩容
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 桶的项从链表变为红黑树的临界点, 项里面的节点
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 桶的项从红黑树重新转为链表的临界点, 项里面的数据节点长度等于 6
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 桶的项从链表变为红黑树的另一个条件, 存储数据的 table 的长度大于 64 了
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
}
```

## 3. HashMap 中几个设定

> 1.HashMap 的 table 的数组的长度必须是 2 的 n 次方。

> 2.我们往 HashMap 中存入一个 <key, value> 时，是存在 table 数组的，但是不是默认往最后一位添加，而是通过存进去的 key 的 hash 值模于 table 的长度, 得到他在 table 的位置。

>> 2.1通过 key 获取到 对应的 hash 值
```java
static final int hash(Object key) {

    int h;
    // 1. 如果 key 为 null 时， 直接返回 0, 也就是 HashMap 允许存放 key 为 null 的情况
    // 2. 如果 key 不为 null 的话， 先取到 key 的 hashCode, 然后把 hashCode 无符号右移16位后，在和原来的hashCode异或
    // 3. 第 2 步叫做 扰乱，作用是把 key 的 hashCode 的高位也进入运算，减少 hash 值相同的情况，也就是减少后续通过 hash 值计算这个 key 在 table 的位置，减少了 hash 冲突
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

>> 2.2通过 key 计算出来的 hash, 取到在 table 的位置
```java

// 取到当前 table 数组的长度
int n = table.length;

// 1. pos就是在 table 的位置, 将 table 的长度 - 1 后, 再和 key 的 hash 值进行与运算
// 2. 一般情况下, 我们知道了 key 值的 hash 值，和 table 的长度，那么 key 在 table 位置就可以直接通过 hash % table 就能知道他的位置了。
// 3. 但是模运算是一个耗时的操作，直接可以通过位运算替代模运算，同时达到相关的效果就好了，也就是  a % b == a 位运算 b, 后面观察发现  a & (2^n - 1) 的效果和 a % (2^n - 1)[a,n 都是正整数], 所以对 table的长度做了限制，限制为 2 的 n 次方, 这样就能通过位运算直接获取 key 在 table 的位置。
int pos = (n - 1) & hash;
```


## 4. HashMap 的创建

```java
public class HashMap {

    /** 当前 HashMap 的负载因子 */
    final float loadFactor;

    /**
     * 无参的构造函数
     */
    public HashMap() {
        // 设置了负载因子为默认值： 0.75, 其他的属性默认
        this.loadFactor = DEFAULT_LOAD_FACTOR;
    }

    /**
     * 指定容量的初始
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /** 
     * 指定初始容量 和 负载因子 
     */
    public HashMap(int initialCapacity, float loadFactor) {

    	// 格式不正确
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);

        // 设置最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;

        // 格式不正确  isNaN(arg) => arg 不是数字，返回true
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);

        this.loadFactor = loadFactor;
        // 通过 tableSizeFor 获取到 第一个大于 initialCapacity 的 2的 n 次方的数
        this.threshold = tableSizeFor(initialCapacity);
    }


    /**
     * 指定 Map的的创建, 把 一个旧的 map 的数据移到当前 HashMap 中
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        // 这个方法，可以先不看，后续在回来理也可以
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
}
```


## 5. HashMap.put 方法
```java

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * 存值
     * @ hash 存的 key 的 hash 值
     * @ key  对象的key
     * @ value 存的值
     * @ onlyIfAbsent 存入的节点已在 HashMap 中存在， 是否用新的 value 替代 旧的 value, false进行修改, true不修改
     * @ evict 这个值用于 LinkedHashMap 在HashMap 中没有作用
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {

        Node<K,V>[] tab; Node<K,V> p; int n, i;

        // table 为空 或者 table 的长度为 0, 进行扩容, 可以看成第一次存数据的时候
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;

        // i = (n - 1 ) & hash 就是求出 这个对象存在 table 的位置, 然后赋值给 i
        // tab 的 i 位置为 null 的话, 把 数据封装为一个 Node 节点, 放到 tab 的 i 位置
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 需要放入的位置的第一个节点 的 hash 和 key 一样，取到这个节点
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                // 调用红黑树 进行处理
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);    
            else {

                for (int binCount = 0; ; ++binCount) {
                    // 从链表的第一个节点开始遍历到尾部

                    // 下一个节点为空, 直接将当前的节点放到尾部
					if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 当前链表的节点个数大于了7, 原本有7了, 加上新的节点，达到了8个，进行红黑树树化
                        if (binCount >= TREEIFY_THRESHOLD - 1)
                            treeifyBin(tab, hash);
                        break;    
                    }
                    // 链表中有一个节点的 hash 和 key 和要插入的一样，取到这个节点
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) 
						break;
                    p = e;    

                }

                if (e != null) {
                    V oldValue = e.value;      
                    // 新值 替换旧值
                    if (!onlyIfAbsent || oldValue == null)
                        e.value = value; 
                    // HashMap 中 这个方法为 空方法
                    afterNodeAccess(e);
                    return oldValue;                 
                }
            }    
        }
        // 用于支持 fast-fail 机制
        ++modCount;
        // 达到了阈值，进行扩容
        if (++size > threshold)
            resize();
        // HashMap 的这个方法也是空方法 
        afterNodeInsertion(evict);
        return null;

    }


```

## 5. HashMap 的方法块

* `tableSizeFor`
```java
/**
 * 返回 第一个大于或者等于 cap 的 2 的 n 次方的数
 */
static final int tableSizeFor(int cap) {
    // 此次减1的原因时，让后面基本处理处理的结果变为 n是第一个大于等于cap的 2的n次方的数
    // 如果这里不减 1 的话， 我们刚好传进来的数是 2的n次方, 经过下面几步的处理会变成 2的n+1次方
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // 做一个容错 如果 cap 是一个 小于等于 0 的数, 经过上面的处理，n 也会是一个小数
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

* `resize`
```java
/**
    * 重新扩容
    */
final Node<K,V>[] resize() {

    Node<K,V>[] oldTab = table;

    // 当前的容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;

    // 当前的阈值
    int oldThr = threshold;
    // 新的容量，新的阈值
    int newCap, newThr = 0;

    // 当前的容量不为0, 也就是有数据存在了
    if (oldCap > 0) {

        // 当前的容量已经达到最大了，直接不进行操作
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 直接将阈值设为int 的最大值
            threshold = Integer.MAX_VALUE;
            // 实际的数组长度为 2^30, 但是阈值为长度 乘以 负载因子的长度, 现在直接把阈值变为了 (2^31 - 1), 为了能够利用数组剩余的空间吧 
            return oldTab;
          // 新的容量 = 旧的容量 * 2
          // 新的容量 < 最大容量 && 旧的容量 >= 初始默认的容量(16) 直接设置新的负载 = 旧的 * 2  
          // oldCap >= DEFAULT_INITIAL_CAPACITY 的 作用 看下面的 备注1
        } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;

    } else if (oldThr > 0)
        // 当旧的阈值大于 0, 设置 新的容量 = 阈值
        // 能够走到这一步的情况有：
        // 1. 我们初始时，只指定了容量， 然后第一次往里面加数据
        // 2. 初始时, 只传递了一个Map, 然后第一次把Map里面的数据放到当前HashMap时
        newCap = oldThr;
    } else {
        // 当我们创建时, 没有指定容量时, 在第一次放数据，会走这一步进行容量的设置 默认时 16, 阈值为 16 * 0.75
        // 能够走到这一步的情况有：
        // 我们初始时, 没有指定任何参数
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }

    if (newThr == 0) {
        // 计算 阈值 = 新容量 * 0.75, 如果 阈值超过了最大容量, 则直接设置为 int的最大值

        // 能够走到这一步的情况有：
        // 1. 我们初始时，只指定了容量， 然后第一次往里面加数据
        // 2. 初始时, 只传递了一个Map, 然后第一次把Map里面的数据放到当前HashMap时
        // 3. 扩容时, 新的容量 < 32  这一种情况，需要结合上面的扩容的 oldCap >= DEFAULT_INITIAL_CAPACITY 进行分析
        // 只有 oldCap 的 大于等于 16 了， newThr 才会有值 
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }

    // 设置新的阈值
    threshold = newThr;

    // 到此， 扩容的第一步已经完成了， 计算新的容量和阈值
    // 下面 进行 扩容的第二步：将创建新的 table, 将旧的 table 里面的数据移到新的 table

    // 创建 新的数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];

    // 数组 重新赋值
    if (oldTab != null) {
        // 循环遍历数组的每一项
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 获取到当前数组的第 j个元素
            if ((e = oldTab[j]) != null) {
                // 旧的值设置为空, 方便垃圾回收
                oldTab[j] = null;

                if (e.next == null)
                    // 如果e.next为空, 则代表旧数组的该位置只有1个节点, 那么把这个节点直接放到新数组里面，就行了
                    // 通过 (节点 的 hash 值 & 新容量 -1 ) 取到新的位置、
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 原本的这个节点为 树节点, 调用TreeNode的split进行处理
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  
                else { 

                     // 原本在 A 位置的 链表，在重新排序的时候，会有规律的分布在新数组的 2 个地方,
                     // 靠前面的为低纬度， 靠后面的为高纬度   

                    // 低链表 链表 链头，和链尾
                    Node<K,V> loHead = null, loTail = null;
                    // 高纬度 链表
                    Node<K,V> hiHead = null, hiTail = null;
                    // 下一个节点
                    Node<K,V> next;

                    do {
                        next = e.next;
                        // 通过对象的 hash 值 & 旧的容量，可以确定 这个对象在 高纬度链表，还是低纬度链表， 具体原因看下面的 备注2
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;   
                            loTail = e;     
                        } else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);

                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }

                }      
            }
        }
    }
    return newTab;
}
```



## 6. 参考 

[Java集合：HashMap详解(JDK 1.8)）](https://blog.csdn.net/v123411739/article/details/78996181)