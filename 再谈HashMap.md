## 前言

​		昨天面试时又问到了关于HashMap相关的问题，而我虽然大致回答上来了，但是回答的并不是很好。所以借此机会，把关于HashMap的知识点再重新梳理下，增进理解，加深记忆。

## 何为HashMap

​		在讲何为HashMap之前，我们必须得先搞定另一个概念：Map。

### Map的定义

​		从宏观意义上来讲，Map代表的是一种映射关系。比如有两个集合，集合A和集合B，A集合当中的每一个元素都能在B集合中找到唯一的一个值与之对应，我们就把A集合和B集合共同组成的这种映射关系称之为Map。而A集合中的元素，我们称之为Key，即键；B集合中的元素，我们称之为值，即Value。所以，Map又被称之为双列集合。其实，Map集合在我们生活中的应用无处不在，比如身份证和姓名，IP地址和MAC地址等等。同时，我们也可以感觉到，Key-Value这种数据结构很符合人类的存储习惯。

​		而HashMap相比于Map而言，前面多加了个名字叫做Hash的前缀。我们来看一下什么是Hash。

### Hash算法

​		先看一下Hash算法的定义：将任意长度的数据映射到有限长度的域上面。翻译成人话，意思就是对一串数据m进行杂糅，输出为另一段固定长度的数据h，作为这段数据的特征（数据指纹）。也就是说，无论输入的数据m有多大，长度或短或长，经过Hash算法的运算过后，其输出值h的长度永远都是固定的。至于Hash算法的具体实现细节，我们暂且不讲。

​		所以综上所述，HashMap就是一个基于Hash算法的Map实现。

## 为什么需要HashMap

​		搞清楚什么是HashMap之后，我们接下来要思考另一个问题：为什么需要HashMap呢？

​		我们都知道，集合框架是用来存储数据的。而对于数据的存储而言，插入、删除和遍历的速度变得至关重要。对于其他的Java集合类，比如ArrayList、LinkedList和Vector，它们统统都是在某一方面存在着缺陷，要么插入删除慢，要么遍历速度慢。而为了实现快速查找，HashMap底层使用了数组，从而达到了时间复杂度为O(1)的查找效率；同时有了Hash算法的加入，解决Hash冲突所采用的链地址法和后期红黑树的加入，HashMap的插入和删除性能也变得十分优异。

## HashMap底层详解

​		首先，我们来看一下HashMap定义部分的源码

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    private static final long serialVersionUID = 362498820763181265L;
    //默认初识化大小。此处1<<4表示将1向左移动四位
    //0000 0001 ----> 0001 0000
    //默认会创建16个结点。节点的个数要适当，如果太多遍历哈希表会比较慢，太少的话则会触发扩容
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    //最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //默认加载因子.初始情况下，当键值对的数量>16*0.75=12时，就会触发扩容
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //默认转为红黑树的阈值。即当某个结点下面挂载的链表长度大于8时，链表将转为红黑树
    static final int TREEIFY_THRESHOLD = 8;
    //在哈希表扩容时，如果发现链表的长度小于6，则会由树退化成链表
    static final int UNTREEIFY_THRESHOLD = 6;
    //在由链表转换为红黑树之前，还需再进行一次判断，只有当整个哈希表的键值对总数量大于64时才会发生转换
    //这是为了避免哈希表在建立初期，多个键值对恰巧都被放入了同一个链表中而导致不必要的转换
    static final int MIN_TREEIFY_CAPACITY = 64;
    //HashMap主体的每个结点定义，即数组的结点定义
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;//哈希值，用来定位数组的索引位置
        final K key;
        V value;
        Node<K,V> next;//链表的下一个Node

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
		//自定义的hashCode()方法
        public final int hashCode() {
            //此处的^为异或运算。相同二进制位进行^运算，结果为0；不同，结果为1
            //1001^1100=0101
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
		
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
    //计算哈希值
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    } 
	//用transient修饰符修饰的变量不参与序列化过程
    transient Node<K,V>[] table;//哈希表的主干数组
    transient Set<Map.Entry<K,V>> entrySet;//遍历用
    transient int size;//哈希表中元素的数量
    transient int modCount;//标记字段，记录对该哈希表进行结构修改的次数，主要用于迭代器访问时检测哈希表
    					   //是否因为删除等其他操作引起内部机构发生变化
    int threshold;//扩容阈值
    final float loadFactor;//加载因子
```

​		看一下它的构造方法

```java
//构造方法1，指定初始容量和负载因子
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
//构造方法2，指定初始容量
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
//无参构造方法
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
//指定集合，转换为哈希表
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

​		再看一下最关键的put和get操作

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    //onlyIfAbsent:如果当前位置已存在一个值，是否替换。false是替换，true是不替换
    //evict:表是否在创建模式，如果为false，则表是在创建模式
    Node<K,V>[] tab; 
    Node<K,V> p; 
    int n, i;
    //判断table是否初始化，否则对其进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //计算存储的索引位置，如果该位置上没有元素，即不发生哈希碰撞，就直接赋值，存储
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        //如果发生哈希碰撞，做以下处理
        Node<K,V> e; 
        K k;
        //节点已存在，执行赋值操作
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断链表是否为红黑树
        else if (p instanceof TreeNode)
            //为红黑树，对红黑树操作
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //为链表，对链表操作
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //链表长度为8，将链表转化为红黑树进行操作
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //key存在，直接覆盖
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;//即p=p.next,向后遍历
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;//记录修改的次数
    //判断是否需要扩容
    if (++size > threshold)
        resize();
    //空操作
    afterNodeInsertion(evict);
    return null;
}
```

​		简单来讲，哈希表的存放过程分为以下六步：

​		1.判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；

​		2.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向6，如果table[i]不为空，转向3；

​		3.判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向4，这里的相同指的是hashCode以及equals；

​		4.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向5；

​		5.遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

​		6.插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。

## 总结

​		仅仅是把HashMap定义、构造方法、put过程回顾梳理一下就花了整整一下午，这里面值得深究的点实在是太多了。还有扩容机制，与其它集合相比较还没做呢，任重而道远啊。