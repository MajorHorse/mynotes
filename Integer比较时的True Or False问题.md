​		我们先看下面一段代码的运行结果

```java
public static void main(String[] args) {
    Integer a=100;
    Integer a1=100;
    Integer b=200;
    Integer b1=200;
    Integer c=new Integer(300);
    Integer c1=new Integer(300);
    System.out.println(a==a1);//true
    System.out.println(b==b1);//false
    System.out.println(c==c1);//false
    System.out.println(c.equals(c1));//true
}
```
​		首先需要明确一点，Java中的==比较如果比较的是基本数据类型，会把数据从常量池里面取出进行比较；比较引用数据类型时，则会比较其引用的地址是否相同。

​		再说回第一行代码，Integer a=100实质上是Integer a=Integer.valueof(100),而查看valueof方法的源码

```java
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

​		再查看IntegerCatch的定义源码

```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer[] cache;
        static Integer[] archivedCache;
        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    h = Math.max(parseInt(integerCacheHighPropValue), 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(h, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;
            // Load IntegerCache.archivedCache from archive, if possible
            VM.initializeFromArchive(IntegerCache.class);
            int size = (high - low) + 1;
            // Use the archived cache if it exists and is large enough
            if (archivedCache == null || size > archivedCache.length) {
                Integer[] c = new Integer[size];
                int j = low;
                for(int i = 0; i < c.length; i++) {
                    c[i] = new Integer(j++);
                }
                archivedCache = c;
            }
            cache = archivedCache;
            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }
        private IntegerCache() {}
    }
```

​		可以看出，Java内部维护了一个缓存数组catch，而这个数组的范围是-128到127。所以说，当使用Integer a=xx的形式进行赋值时，若赋的值在-128到127之间，并不创建新对象，即引用的地址始终相同，所以比较的结果为true；若赋的值不在在-128到127之间，则此语句等效于Integer a=new Integer(xx)，此时地址自然不同，结果自然为false。

​		而对于最后一行的equals比较，查看equals的源码

```java
public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```

​		可以看出，若比较的双方为同一数据类型时，其比较的仍是具体的数值，所以比较的结果自然为true。