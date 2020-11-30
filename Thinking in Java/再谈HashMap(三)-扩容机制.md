### 扩容机制

​		前面说到，当threshold=capacity*loadFactor，即当size>=threshold时，就会触发HashMap的扩容。扩容指的是对主干数组进行扩容，也就是所谓的桶。其实，HashMap实行了懒加载。HashMap懒加载的意思是在新建一个HashMap时并不会对table进行赋值，而是等到第一次插入时，调用resize进行构建table。看一下resize()的源码

```java
final Node<K,V>[] resize() {
    //定义一个oldTable存放当前的table
    Node<K,V>[] oldTab = table;
    //判断当前的table是否为空（主要是看是否为第一次构建HashTable）
    //若为空，oldCap大小置为0；否则oldCap大小设置为当前table大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //将oldThr值设置为当前table的扩容阈值
    int oldThr = threshold;
    //定义newCap，newThr，即定义新的容量和长度，并将其初始值置0
    int newCap, newThr = 0;
    //oldCap>0,说明并不是第一次构建table，则需扩容
    if (oldCap > 0) {
        //判断当前table的长度是否大于能提供的最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            //如果达到最大容量，给int的最大值作为其扩容阈值，然后将当前table返回
            //意思就是你已经扩容到极限了不能再扩了，我给你设置个基本够不着的扩容阈值
            //你里面爱咋碰撞就咋碰撞，我不管了，也别再来找我扩容了
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //将新的table的扩容阈值定义为当前table扩容阈值的两倍
        //同时还要满足新的容量小于HashMap所能提供的最大容量，当前table容量大于默认容量
        //oldCap<<1,旧容量向左移一位，即newCap=oldCap*2.
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            //将新table的扩容阈值扩容阈值设置为当前table扩容阈值的二倍，double
            newThr = oldThr << 1; // double threshold
    }
    //如果当前table的容量<0(即table为null)，且扩容阈值>0
    //就将新table的容量值设置为当前table的扩容阈值
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        //如果当前table的容量<0，且扩容阈值<0,说明第一次构建
        //将新table的容量和扩容阈值都设置为默认值
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //经过了以上步骤后，若新table的扩容阈值仍为0，则对其进行赋值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //将新的阈值赋值给内置的threshold，用来给HashMap构建
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    //初始化一个容量大小为newCap的Node数组
    //到这里才真正开始初始化/扩容
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //如果当前table不为空，说明其中有数值，则对它进行遍历
    if (oldTab != null) {
        //遍历整个数组，把每个桶子元素都移动到新的桶子中，将非空元素进行复制
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
             //若当前table，即原来的table中的第j个位置的元素不为空
            if ((e = oldTab[j]) != null) {
                //将旧的数组第j个位置的桶子置空，元素已经存在了e中
                oldTab[j] = null;
                //若为空，说明这个桶子下面没有挂任何结点，即该位置没有哈希冲突，转移
                if (e.next == null)
                    //以e.hash & (newCap - 1)作为e在新table中的位置
                    newTab[e.hash & (newCap - 1)] = e;
                //判断e是否为红黑树结点，是的话说明这个桶子下面长了一棵红黑树，进行处理
                else if (e instanceof TreeNode)
                    //调用split方法，将红黑树节点也也转移到新的数组中
                    //split方法中有将红黑树还原成链表的方法
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                //否则，这个桶下面肯定挂了一个链表
                else { // preserve order
                    //存放不需要移动索引位置的链表的头、尾节点
                    Node<K,V> loHead = null, loTail = null;
                    //存放需要移动索引位置的链表的头、尾节点
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //循环旧链表
                    do {
                        //获取下一个节点
                        next = e.next;
                        //(e.hash & oldCap) == 0用来确定元素是否需要移动，0不需要，1需要
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        //对需要移动的链表进行处理
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    //将上面的旧链表迁移到新的数组中
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

### 思考

​		1.(e.hash & oldCap) == 0为何能够作为元素是否需要移动的标准？

​		2.红黑树和链表的相互转化细节是怎样的？

​		2.初始化的整个过程还需要再捋几遍。

### 小结

​		这部分代码看的我脑壳疼，整了整整一下午，前四分之三基本撸通顺了，但是后面的红黑树和链表的转移还是没给整明白，包括一些字段的含义以及各种位运算背后的逻辑还是不通的。还有，看着后面的东西再回顾之前的，发现之前自己忽略的小点深究起来也得累死个人，就跟树根似的。看来，看源码的能力还是得逐步加强啊。