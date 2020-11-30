### GET流程

​		我们看一下Get操作的源码

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; 
    Node<K,V> first, e; 
    int n; 
    K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //桶中不止一个节点
        if ((e = first.next) != null) {
            //红黑树
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //链表
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### 关于各个元素的详细含义

​		loadFactor	加载因子：loadFactor	加载因子（以下简称LD）是用来控制数组存放数据的疏密程度的。LD越趋近于1，那么数组中存放的数据（Entry）也就越多，越密，会让链表的长度增加；LD越小，即越趋近于0，数组中存放的数据（Entry）也就越少，越稀疏。同时，LD太大将导致查找效率降低，太小将导致数组利用率降低，存放点额数据过于分散。

​		threshold：临界值，即扩容阈值。threshold=capacity*loadFactor。当size>=threshold时，就要考虑对数组进行扩容了。

### 还需思考的问题

​		 1.tab[(n - 1) & hash]：&即按位与操作，只有对应的两个二进制位都为1时结果才为1，如1100&0100=0111。这行代码的意思是获取到当前数组的最大下标然后与key的hash值进行按位与操作。这里的下标为什么出不了table[]的范围呢？

​		2.为什么LD最佳会是0.75？

​		3.如何扩容？

### 小结

​		愉快的周末过去了，周一的状态不是很好。HashMap的问题还有很多，要抓紧时间搞明白。

​		

​		

​		