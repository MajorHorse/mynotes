# 前言

​		try、catch、finally是Java的异常处理语句。按正常情况来讲，若未捕获异常，执行顺序为try-->finally；若捕获到了异常，执行顺序为try-->catch-->finally。但是，当各个语句块中加入了return语句后，执行结果可能会出现多种不同。下面来做测试，解释说明下情况

# 测试1：try中有return，不捕获异常

```java
/**
 * 测试1：try中有return,不捕获异常
 * @return
 * 结果：try:2
 *      finally:3
 *      return:2
 */
private static int test1(){
    int count=1;
    try {
        count++;
        System.out.println("try:"+count);
        return count;
    }catch (Exception e){
        count++;
        System.out.println("catch:"+count);
    }finally {
        count++;
        System.out.println("finally:"+count);
    }
    return count;
}
```

​		分析：当try中带有return时，会先执行return前的代码，然后暂时保存需要return的信息（可以理解为用一个temp遍历保存）再执行finally中的代码，最后再通过return返回之前保存（temp）的信息。

# 测试2：try、catch中有return，捕获异常

```java
/**
 * 测试2：try、catch中有return,捕获异常
 * @return
 * 结果：try:2
 *      catch:3
 *      finally:4
 *      return:3
 */
private static int test2(){
    int count=1;
    try {
        count++;
        System.out.println("try:"+count);
        int x=count/0;
        return count;
    }catch (Exception e){
        count++;
        System.out.println("catch:"+count);
        return count;
    }finally {
        count++;
        System.out.println("finally:"+count);
    }
}
```

​		分析：原理和try一致，捕获了异常，便会返回异常处理时的值。

# 测试3：try、catch、finally中都有return，捕获异常

```java
/**
 * 测试3：try、catch、finally中都有return,捕获异常
 * @return
 * 结果：try:2
 *      catch:3
 *      finally:4
 *      return:4
 */
private static int test3(){
    int count=1;
    try {
        count++;
        System.out.println("try:"+count);
        int x=count/0;
        return count;
    }catch (Exception e){
        count++;
        System.out.println("catch:"+count);
        return count;
    }finally {
        count++;
        System.out.println("finally:"+count);
        return count;
    }
}
```

​		分析：当finally中有return时，会返回最后在return语句块中运算所得的值（之前在try、catch中算的值也要保存）。可以简单的理解为，finally语句块中一旦有return，便会屏蔽try、catch块中的return。

# 小结

​		finally语句块中的代码无论如何总会被执行。当try、catch、finally中都有return时，会直接执行finally中的return，导致try、catch中的return失效。