### 一、进程&线程

之前看别人写的博客或者文章时，很多人都把进程和线程这两个概念混为一谈。按照严格的定义来讲，进程是资源分配的最小单位，线程是CPU调度的最下单位。但是，定义讲的太刻板，不容易理解。举个例子来讲：当我们打开电脑启动Windows系统后，然后登录上自己的QQ，这时我们再打开系统的任务管理器，便可以看到一个名为“QQ.exe”的进程出现在了任务列表中，后面跟着它此刻占用了多少内存，占用了多少CPU使用率。此时我们可以说，QQ就是一个进程。QQ是一个聊天工具，我们往往会同时跟多个好友进行交流，而这每个好友的聊天框，就分别是一个个不同的进程：它们的状态互不影响，你关了一个，剩下的还能继续聊。再比如我们打开电脑上的火绒进行杀毒，这里的火绒就是一个主进程，而杀毒就是一个线程，在杀毒的同时我们可以进行垃圾清理，垃圾清理就又是一个线程。可以看出，进程和线程明显是一种一对多的关系，一个进程包含多个线程，而每个线程则属于一个唯一的进程。

### 二、Java中的多线程

#### 1.多线程的开启--实现Runnable接口

```java
public class MultiThreadDemo {
    public static void main(String[] args) {
        //实例化三个对象
        Mythread mythread1=new Mythread("mythread1");
        Mythread mythread2=new Mythread("mythread2");
        Mythread mythread3=new Mythread("mythread3");
        //创建线程
        Thread th1=new Thread(mythread1);
        Thread th2=new Thread(mythread2);
        Thread th3=new Thread(mythread3);
        //开启线程
        th1.start();
        th2.start();
        th3.start();
        //PS:不可以直接使用mythread1.run()来开启线程必须得创建线程，通过start()方法开启一个线程
        /**
         * 输出：线程mythread3打印了.....664984
         *      线程mythread3打印了.....664985
         *      线程mythread3打印了.....664986
         *      线程mythread3打印了.....664987
         *      线程mythread3打印了.....664988
         *      线程mythread1打印了.....585490
         *      可以看到线程的切换
         */
    }
}
//1.实现Runnable接口，覆盖run方法
class Mythread implements Runnable{
    //表示线程名称
    private String name;

    public Mythread(String name) {
        this.name = name;
    }
    //重点，覆盖run方法
    @Override
    public void run() {
        for (long i = 0; i < 100000000000000L; i++) {
            System.out.println("线程"+name+"打印了"+"....."+i);
        }
    }
}
```

#### 2.继承Thread类，重写run方法

```java
public class MultiThreadDemo {
    public static void main(String[] args) {
        //由于继承了Thread类，此时可以直接创建线程
        MyThread2 th1=new MyThread2("thread1");
        MyThread2 th2=new MyThread2("thread2");
        //开启线程
        th1.start();
        th2.start();
        /**
         * 输出：线程thread1打印了.....344025
         *      线程thread1打印了.....344026
         *      线程thread2打印了.....310678
         *      线程thread2打印了.....310679
         *      线程thread2打印了.....310680
         */
    }
}

//2.继承Thread类，重写run方法
class MyThread2 extends Thread{
    //表示线程的名称
    private String name;
    public MyThread2(String name) {
        this.name = name;
    }
    public void run(){
        for (long i = 0; i < 100000000000000L; i++) {
            System.out.println("线程"+name+"打印了"+"....."+i);
        }
    }
}
```