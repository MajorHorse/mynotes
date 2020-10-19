### 一、String

想要弄清一些类及其特性的本质，必须得学着从源码入手并分析。看一下String类的源码

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
}
```

透过源码，可以看出String类本质上内部是一个字符数组。String是被final修饰的，这也就意味着它不能被继承，并且它的成员方法默认都为final方法。一个变量一旦被final修饰就意味着：若此变量为值类型，则它的值不能被改变；若此变量为引用类型，则它指向的地址不能被改变。简单来讲，String是一份不可变的字符串，它的底层是一个用final修饰的字符数组。

```java
String str="abc";//字面量“abc”存储在运行时常量池内，然后它的引用，即符号引用s指向它
str="abcde";//并不是直接修改abc，而是在运行时常量池内新建了字面量“abcde”，让s指向它
```

上面实例代码中，第二次赋值时并不会改变“abc”对象的值，而是新建了一个对象。

同理，这也就能解释下面一组代码的结果为什么是true

```java
String str="abc";//运行时常量池内新建字面量“abc”，然后符号引用str指向它
String s2="abc";//发现运行时常量池内已经有“abc”，将s2直接指向它
String s4="a"+"bc";//编译期会被优化，将加号去掉，相当于直接定义了一个字面量“abc”
System.out.println(str==s2);//outPut:true
System.out.println(str==s2);//outPut:true
```

而直接 相加时，则会出现一些不同

```java
String str="abc";
String s3="a";
s3+="bc";
System.out.println(str==s3);//false
```

之所以结果为false，是因为s3+="bc"操作会被jvm优化为

```java
StringBuilder s_3=new StringBuilder(s3);
s_3.append("bc");
s3=str.toString();
```

总结来看，两个或者两个以上的字符串常量直接相加，在预编译的时候“+”会被优化，相当于把两个或者两个以上字符串常量自动合成一个字符串常量，其他方式的字符串相加都会使用到 StringBuilder。

PS：关于字面量和符号引用（转自知乎）

​		常量池中主要存放有两大类常量：字面量和符号引用。

​		字面量比较接近于Java语言层面的常量概念，如文本字符串、声明为final的常量值等。

​		String类型的字符串常量也就是属于字面量；

​		符号引用顾名思义，就是引用；

​		符号引用属于编译原理方面的概念，包括了下面三类常量：

​		类和接口的全限定名

​		字段的名称和描述符

​		方法的名称和描述符

​		所以，以上符号引用的具体内容也就是常量池内的字面常量。

### 二、StringBuiler&StringBuffer

StringBuilder和StringBuffer都继承自AbstractStringBuilder，看一下StringBuilder的源码

```
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence{}
```

看一下其父类AbstractStringBuilder的源码

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;

    /**
     * This no-arg constructor is necessary for serialization of subclasses.
     */
    AbstractStringBuilder() {}
```

可以看到，其底层定义的数组并没有final修饰符，所以在做字符串拼接的时候就直接可以在原来的内存上进行拼接，并不会浪费内存空间，即其为可变的。

所以，当一个字符串需要被经常修改时，应当使用StringBuffer实现。如果使用String来存储一个经常被修改的字符串，则该字符串每次被修改时都会创建新的无用对象，造成资源的浪费，影响程序的性能。

### 三、StringBuilder和StringBuffer的区别

看一下StringBuffer的源码

```java
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{

    /**
     * A cache of the last value returned by toString. Cleared
     * whenever the StringBuffer is modified.
     */
    private transient char[] toStringCache;

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    static final long serialVersionUID = 3388685877147921107L;

    /**
     * Constructs a string buffer with no characters in it and an
     * initial capacity of 16 characters.
     */
    public StringBuffer() {
        super(16);
    }

    @Override
    public synchronized int length() {
        return count;
    }

    @Override
    public synchronized int capacity() {
        return value.length;
    }


    @Override
    public synchronized void ensureCapacity(int minimumCapacity) {
        super.ensureCapacity(minimumCapacity);
    }

    /**
     * @since      1.5
     */
    @Override
    public synchronized void trimToSize() {
        super.trimToSize();
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @see        #length()
     */
    @Override
    public synchronized void setLength(int newLength) {
        toStringCache = null;
        super.setLength(newLength);
    }
}
```

可以看到，其内部方法全部都加了同步锁，这也就说明其是线程安全的。

#### 小结

StringBuffer与StringBuilder的功能与用法基本相同，只是StringBuilder是线程不安全的，StringBuffer是线程安全的。所以，如果只是在单线程中使用字符串缓冲区，则StringBuilder的效率会高些，但是当多线程访问时，最好使用StringBuffer。

综上，在执行效率方面，StringBuilder最高，StringBuffer次之，String最低，对于这种情况，一般而言，如果要操作的数量比较小，应优先使用String类；如果是在单线程下操作大量数据，应优先使用StringBuilder类；如果是在多线程下操作大量数据，应优先使用StringBuilder类。