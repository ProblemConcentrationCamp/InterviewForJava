# HashMap (基于 Jdk8 的实现)

学习了一下 HashMap 的内部原理，忽略了红黑树相关的代码。

## 1. HashMap 的内部的数据结构

![Alt 'HashMap-DataStructure'](https://s2.ax1x.com/2019/12/07/QtK2Lt.png)

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

通过 Node 和 TreeNode 我们可以知道，HashMap 的数据是存在 table 数组里面， 这个数组的类型是 Node,
而 Node 的实现可以为 Node 本身, 也就是链表 和 TreeNode 红黑树。

## 2. HashMap 中几个比较重要的属性介绍
```java

public class HashMap<K,V> {

    /**
     * 数据存储的地方, 后文说到的 table，就是指这一个了数组
     */
    transient Node<K,V>[] table;

    /**
     * 16, 初始时，没有指定容量，我们 table 的大小， 
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
     * table 的项从链表变为红黑树的临界点, 项里面的节点
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * table 的项从红黑树重新转为链表的临界点, 项里面的数据节点长度等于 6
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * table 的项从链表变为红黑树的另一个条件, 存储数据的 table 的长度大于 64 了
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
}
```

## 3. HashMap 中几个设定

> 1.HashMap 的 table 的数组的长度必须是 2 的 n 次方。

> 2.我们往 HashMap 中存入一个 <key, value> 时，是存在 table 数组的，但是不是默认往最后一位添加，而是通过存进去的 key 
的 hash 值经过计算, 得到他在 table 的位置。


>> 2.2通过 key 计算出来的 hash, 取到在 table 的位置
```java

// 取到当前 table 数组的长度
int n = table.length;

// 1. pos就是在 table 的位置, 将 table 的长度 - 1 后, 再和 key 的 hash 值进行与运算

// 2. 一般情况下, 我们知道了 key 值的 hash 值，和 table 的长度，那么 key 在 table 位
// 置就可以直接通过 hash % table 就能知道他的位置了。

// 3. 但是模运算是一个耗时的操作，直接可以通过位运算替代模运算，同时达到相关的效果就好了，也就是  a % b == a 
//位运算 b,后面观察发现  a & (2^n - 1) 的效果和 a % (2^n - 1)[a,n 都是正整数], 
// 所以对 table的长度做了限制，限制为 2 的 n 次方, 这样就能通过位运算直接获取 key 在 table 的位置。
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
* ** tableSizeFor 方法**
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

* ** putMapEntries 方法 **
``` java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {

    // 获取 需要存放的 map 中已有的数据量
    int s = m.size();

    // 大于 0, 说明有数据需要存储到当前的 HashMap 对象
    if (s > 0) {
        // 当前的 HashMap 存储数据的 table 为空,
        if (table == null) {
            // 计算出来需要的容量 =   长度 / 赋值因子 + 1
            float ft = ((float)s / loadFactor) + 1.0F;

             // 限制最大的容量 为  2^ 30
            int t = ((ft < (float)MAXIMUM_CAPACITY) ? (int)ft : MAXIMUM_CAPACITY);

            // 计算出来的需要的容量 > 实际的阈值，进行新容量的计算
            if (t > threshold)
                threshold = tableSizeFor(t);

        } else if (s > threshold)
            // 数据量大于阈值, 进行 扩容
            resize();
        // 遍历 整个 HashMap, 依次放到 当前的 HashMap 对象    
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {

            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }    

        // putMapEntries 的方法先到这里，里面设计到的 resize() 扩容和 putVale 存放 value 留在下面 HashMap 的 put方法进行讲解
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

* ** hash 方法**
```java
static final int hash(Object key) {

    int h;
    // 1. 如果 key 为 null 时， 直接返回 0, 也就是 HashMap 允许存放 key 为 null 的情况
    
    // 2. 如果 key 不为 null 的话， 先取到 key 的 hashCode, 然后把 hashCode 无符号右移16位后，
    // 在和原来的hashCode异或

    // 3. 第 2 步叫做 扰乱，作用是把 key 的 hashCode 的高位也进入运算，
    // 减少 hash 值相同的情况，也就是减少后续通过 hash 值计算这个 key 在 table 的位置，减少了 hash 冲突
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

* ** 这个方法在 HashMap 中是不存在的, 个人定义的，只是为了方便理解 HashMap 是如何通过 hash 知道对象存在数组的哪个位置  **
```java

int pos(int hash) {
    // 取到当前 table 数组的长度
    int n = table.length;

    // 1. pos就是在 table 的位置, 将 table 的长度 - 1 后, 再和 key 的 hash 值进行与运算

    // 2. 一般情况下, 我们知道了 key 值的 hash 值，和 table 的长度，那么 key 在 table 位
    // 置就可以直接通过 hash % table 就能知道他的位置了。

    // 3. 但是模运算是一个耗时的操作，直接可以通过位运算替代模运算，同时达到相关的效果就好了，也就是  a % b == a 
    //位运算 b,后面观察发现  a & (2^n - 1) 的效果和 a % (2^n - 1)[a,n 都是正整数], 
    // 所以对 table的长度做了限制，限制为 2 的 n 次方, 这样就能通过位运算直接获取 key 在 table 的位置。
    int pos = (n - 1) & hash;
}
```

* ** putVal 方法**
```java
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
}

```


* ** resize 方法 **
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
            // 实际的数组长度为 2^30, 但是阈值为长度 乘以 负载因子的长度, 现在直接把阈值变为了 (2^31 - 1), 
            // 为了能够利用数组剩余的空间吧 
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
        // 3. 扩容时, 新的容量 < 32  这一种情况，需要结合上面的扩容的 oldCap >= DEFAULT_INITIAL_CAPACITY 
        //进行分析, 只有 oldCap 的 大于等于 16 了， newThr 才会有值 
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
                    // 如果e.next为空, 则代表旧数组的该位置只有1个节点, 那么把这个节点直接放到新数组里面，
                    // 就行了通过 (节点 的 hash 值 & 新容量 -1 ) 取到新的位置、
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
                        // 通过对象的 hash 值 & 旧的容量，可以确定 这个对象在 高纬度链表，还是低纬度链表， 
                        // 具体原因看下面的 备注2
                        if ((e.hash & oldCap) == 0) {
                            // 如果loTail为空, 代表该节点为第一个节点
                            if (loTail == null)
                                loHead = e;
                            else
                                // 否则将节点添加在链表的尾部
                                loTail.next = e; 
                            // 重新设置尾结点      
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
                        // 设置数组的 j位置 为 lo链表
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        // 设置数组 的 j + oldCap 为 hi链表
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

[备注1]  
在 resize 方法中, 旧数组已有数据了, 但是在进行扩容时, 容量默认是2倍扩展，但是阈值只有在 旧的容量 >= 16 时才会进行2倍扩展, 否则是以 新的容量 * 负载因子, 原因：为了在容量少的情况下，尽可能的利用数组的空间，不造成浪费。

假设我们在初始时，指定了容量为2, 那么在初始后 容量值为 2, 阈值为 * 0.75 = 1 

第一次扩容：

|     | 旧的容量 | 新的容量 | 旧的阈值 | 新的阈值|
| :-: | :-: | :-:| :-:| :-:|
| 旧的阈值 << 1  | 2 | 4 | 1| 2 |
| 新的容量 * 0.75  | 2| 4| 1| 3|

我们可以看到 初始的容量为 2时, 新的容量默认扩大2倍，变为 4, 旧的阈值都为 2* 0.75 = 1, 如果阈值按照扩大 2 倍来算的话，新的阈值为 2, 按照 * 阈值来算的话，新的阈值为 3。 那么 4 个空间里面, 按照乘以阈值的形式来算的话，可以有 3 个的有效位置, 比直接扩大 2 倍，大了 1 个空间。


第二次扩容：

|     | 旧的容量 | 新的容量 | 旧的阈值 | 新的阈值|
| :-: | :-: | :-:| :-:| :-:|
| 旧的阈值 << 1  | 4 | 8 | 2| 4 |
| 新的容量 * 0.75  | 4| 8| 3| 6|

第三次扩容：

|     | 旧的容量 | 新的容量 | 旧的阈值 | 新的阈值|
| :-: | :-: | :-:| :-:| :-:|
| 旧的阈值 << 1  | 8 | 16 | 4 | 8 |
| 新的容量 * 0.75  | 8| 16 | 6| 12|

第四次扩容：

|     | 旧的容量 | 新的容量 | 旧的阈值 | 新的阈值|
| :-: | :-: | :-:| :-:| :-:|
| 旧的阈值 << 1  | 16 | 32 |  8  | 16 |
| 新的容量 * 0.75  | 16| 32 | 12 | 24|


通过上面可以发现：在旧容量 <16 时, 通过 新容量 × 负载因子, 阈值会大一些, 可以更充分的理用数组的空间。 

在第四次扩容时, 旧的阈值为12, 新容量为32时,

12 << 1 和 32 * 0.75 的值一样。
所以存在 `x(旧的阈值) * 2 == y(新的容量) * 0.75`, 后续的的阈值 都是在 上一个阈值的基础上 * 2, 而 新的容量的扩容都是上一次容量 * 2, 2 者相约，值都是一样的。但是通过位运算比较快，所以在旧的容量 >= 16, 也就是 新容量 >= 32 时，阈值的计算方式变为 << 1。

综上: 在 旧容量 < 16 时，阈值通过 新的容量 * 0.75 时, 比直接阈值扩大 2 倍大，能更充分的利用 table 的空间，在旧容量 >= 16 后，旧的阈值扩大 2 倍 =  新的容量 * 0.75 一样，但是位运算更快，所以后续通过 阈值 << 1, 也就是 扩大2倍，直接获取新的阈值

[备注2]  
在 resize 方法中, 一个节点的 (e.hash & oldCap) 是否等于 0 就可以判断节点在新数组的位置和旧数组的是否一样。能达到这样的效果的必须知道：
>1. oldCap 是 2 ^ n, newCap 是 在 oldCap的基础 * 2, 也就是 newCap 是 2 ^ (n + 1)
>2. 2 ^ n 在二进制的表示为 1 + n 个 0 , 2 ^ (n + 1) 二进制表示 1 + (n+1) 个 0, 也就是  2 + n 个 0。
>3. 2 ^ n - 1 在二进制的表示为 n 个 1, 那么 2 ^ (n + 1) - 1 就是 n + 1 个 1

>4. (2 ^ n - 1) & hash 我们只需要取 hash 的 1 到 n 位就行了, 因为 2 ^ n - 1 前面都是 0

开始  
1) 假设 现在 oldCap是16,既 2^4, 16 - 1 = 15, 二进制表示为 00000000 00000000 00000000 00001111
2)  (16-1) & hash 的值, 自然只需要取 hash 值的低4位, 我们假设它为 abcd。
3)  oldCap扩大了一倍, 新的index的位置就变成了 (32-1) & hash, 就是取 hash值的低5位, 那么对于同一个Node, 低5位的值无外乎下面两种情况: 0abcd 或者 1abcd
4)  0abcd = index, 而 1abcd = 0abcd + 10000 = 0abcd + oldCap = index + oldCap, 从这里可以知道 容量扩大了一倍, 那么 新的index是有规律的, 要么不变, 要么就是 index + oldCap
5)  新旧index是否一致就体现在 hash 值的第5位, 那么第5位怎么知道呢？ 这时我们可以知道 2^n 的二进制形式 1 + 5个0, 那么 hash & oldCap 就能知道 hash 的第5位 是 0 或者 1
6)  hash & oldCap = 0 => 当前节点在数组的位置不用变, hash & oldCap != 0 => 当前节点在新数组的 index + oldCap 的位置


## 6. HashMap.get 方法
```java
/**
 * 通过 key 获取value, 如果 节点为null, 返回nulL
 */
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

* **getNode 方法**
```java

final Node<K,V> getNode(int hash, Object key) {

    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;

    // 数组不为空，长度大于 0 ， 定位到的位置的链表的第一个节点 不为 null
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {

        // 第一个节点 符合条件了, 直接返回        
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
            return first;   

        if ((e = first.next) != null) {
            if (first instanceof TreeNode) 
                // 树节点, 调用 树的操作
               return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            do {
                // 遍历 链表的其他节点, 找到符合条件的，直接返回
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);   
        }
    }
    return null;
}

```


## 7. HashMap.remove 方法
```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
}
```

* **removeNode 方法**
```java
/**
 * 删除节点
 * matchValue : 为 true，找到了节点，还会比较他们的值，值相同才会删除
 * movable : 为false, 当节点删除了, 不改变其他节点的位置
 */
final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {

    Node<K,V>[] tab; Node<K,V> p; int n, index;

    // 数组不为空，长度大于 0 ， 定位到的位置的链表的第一个节点 不为 null
    if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {

        Node<K,V> node = null, e; K k; V v;
        // 链表的头部就是需要删除的节点, 把节点赋给 node
        if (p.hash == hash &&  ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            // 树节点，通过数找到需要删除的节点
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 遍历链表 找到需要删除的节点
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }  
        }

        // 需要删除的节点不为空
        if (node != null && (!matchValue || (v = node.value) == value ||  (value != null && value.equals(v)))) {
            // 调用树节点的移除方法
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                // 把 链表的头部直接指向了下一个节点
                tab[index] = node.next;    
            else
                // p的下一个节点指向需要删除节点的下一个节点
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

## 8. HashMap 相关的一些 面试问题

1. HashMap 的存储过程?
这里可以在细分为 (1)key 的 hash 计算 (2) 通过 key 的 hash 定位到 table 数组的位置 (3) 在table 数组的位置上如何确定这个对象已存在 (4) 存在怎么操作, 不存在是怎么处理 (5) 什么时候进行红黑树化 (6)如何进行扩容(可以分 2 部分进行分析，1: 新的容量和新阈值的计算， 2. 将旧的 HashMap 上的数据移到新的上)  
2. HashMap 可以存放 null 值吗?  
3. HashMap的数据结构是什么样子的？  
4. HashMap的长度为什么是2的倍数？  
5. HashMap 和 Hashtable 的区别？
这个可以从 3 个方面入手: 安全性，效率，能否允许 key, value 为null
6. hashMap是线程安全的吗？为什么？
可以通过多个线程同时进行新增进行思考

## 9. 参考 

[Java集合：HashMap详解(JDK 1.8)）](https://blog.csdn.net/v123411739/article/details/78996181)