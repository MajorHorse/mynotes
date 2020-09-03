### 一、问题定义

生产者消费者问题（Producer-Consumer Problem），又称为有限缓冲问题（Bounded-Buffer Problem），是一个多线程中解决线程同步问题的经典案例。问题描述为生产者只负责源源不断的生产数据，然后将生产的数据堆积到大小固定的缓冲区内，不断地重复这个过程；而消费者则负责不停地消耗缓冲区内的数据。问题的难点在于生产者和消费者之间必须保持同步，要保证生产者不能在缓冲区满的时候往缓冲区内丢数据，消费者不能在缓冲区已经空了的时候去尝试消耗缓冲区内的数据。如果处理不好将会导致死锁问题。

### 二、解决思路

要保证每个线程在操作资源时行为的原子性，即在同一时刻，资源只能被一个线程访问。

### 三、案例实现

可以采用唤醒机制来解决此问题 ，即wait()/notify()机制，即往缓冲区的任何操作都需加上锁，当生产者生产后缓冲区满了时，生产者睡眠自身；消费者发现缓冲区已空时，放弃锁。生产和消费的动作发生后都必须执行唤醒所有睡眠的线程这一操作，为了避免所有线程都陷入睡眠。

```java
package offer;

import java.util.LinkedList;

/**
 * @author Major Tom
 * @date 2020/8/21 12:20
 * @description 生产者-消费者问题
 */
public class PCProblem {
    public static void main(String[] args) {
        //创建缓冲区
        Storage storage=new Storage();
        //创建三个生产者线程
        Thread p1=new Thread(new Producer(storage));
        Thread p2=new Thread(new Producer(storage));
        Thread p3=new Thread(new Producer(storage));
        //创建三个消费者线程
        Thread c1=new Thread(new Consumer(storage));
        Thread c2=new Thread(new Consumer(storage));
        Thread c3=new Thread(new Consumer(storage));
        //开启线程
        p1.start();
        p2.start();
        p3.start();
        c1.start();
        c2.start();
        c3.start();
    }
}
/**生产者-只负责生产数据，往缓冲区里面丢*/
class Producer implements Runnable{
    /**
     * 持有缓冲区
     */
    private Storage storage;

    public Producer(Storage storage){
        this.storage=storage;
    }

    @Override
    public void run() {
        while (true){
            try {
                //打印的慢一点，没别的意思
                Thread.sleep(1000);
                //一直生产
                storage.produce();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
/**消费者-只负责把缓冲区内的数据拿来消耗*/
class Consumer implements Runnable{
    /**
     * 持有缓冲区
     */
    private Storage storage;

    public Consumer(Storage storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(1000);
                //一直消费
                storage.consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

/**
 * 缓冲区
 */
class Storage{
    /**
     * 定义好缓冲区内的最大容量
     */
    private final int MAX_SIZE=10;
    /**
     * 缓冲区内存储的载体
     */
    private LinkedList<Object> list=new LinkedList<>();

    /**
     * 定义生产方法
     */
    public void produce(){
        //加锁，所有操作都需保持状态一致性
        synchronized (list){
            //若缓冲区已满，则睡眠当前线程
            while (list.size()+1>MAX_SIZE){
                System.out.println("生产者"+Thread.currentThread().getName()+"无法生产，因为缓冲区已满");
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //若缓冲区未满，则继续生产
            list.add(new Object());
            System.out.println("生产者"+Thread.currentThread().getName()+"生产了一个商品，当前缓冲区内商品数为："+list.size());
            //生产完毕 ，放弃持有锁并唤醒所有线程
            list.notifyAll();
        }
    }

    /**
     * 定义消费方法
     */
    public void consume(){
        //加锁，消费方法全程也需保持状态一致性
        synchronized (list){
            //若缓冲区空了，则睡眠当前线程，停止消费
            while (list.size()==0){
                System.out.println("消费者"+Thread.currentThread().getName()+"无法消费，因为缓冲区已空");
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            list.remove();
            //单次消费完毕，放弃持有锁并唤醒所有线程
            System.out.println("消费者"+Thread.currentThread().getName()+"消费了一个商品，当前缓冲区内商品数为："+list.size());
            list.notifyAll();
        }
    }
}
/**
 * 运行结果：
 * 消费者Thread-5消费了一个商品，当前缓冲区内商品数为：0
 * 消费者Thread-5无法消费，因为缓冲区已空
 * 消费者Thread-3无法消费，因为缓冲区已空
 * 生产者Thread-2生产了一个商品，当前缓冲区内商品数为：1
 * 生产者Thread-1生产了一个商品，当前缓冲区内商品数为：2
 * 消费者Thread-4消费了一个商品，当前缓冲区内商品数为：1
 * 生产者Thread-0生产了一个商品，当前缓冲区内商品数为：2
 * 消费者Thread-3消费了一个商品，当前缓冲区内商品数为：1
 * 消费者Thread-5消费了一个商品，当前缓冲区内商品数为：0
 * 消费者Thread-4无法消费，因为缓冲区已空
 * 消费者Thread-3无法消费，因为缓冲区已空
 * 生产者Thread-0生产了一个商品，当前缓冲区内商品数为：1
 * 生产者Thread-1生产了一个商品，当前缓冲区内商品数为：2
 * 生产者Thread-2生产了一个商品，当前缓冲区内商品数为：3
 * 消费者Thread-5消费了一个商品，当前缓冲区内商品数为：2
 * 消费者Thread-3消费了一个商品，当前缓冲区内商品数为：1
 * 消费者Thread-4消费了一个商品，当前缓冲区内商品数为：0
 */
```

