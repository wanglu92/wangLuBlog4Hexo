---
title: Java多线程
date: 2021-01-13 14:28:19
tags:

---

## 进程、线程、多线程



## 线程的实现

继承Thread类

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("my thread");
    }

    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }

}
```

实现Runnable接口

```java
ublic class MyRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable, "my runnable");
        thread.start();
    }

}
```

实现Callable接口

```
public class MyCallable implements Callable<String> {

    @Override
    public String call() throws Exception {
        return Thread.currentThread().getName();
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<String> submit = executorService.submit(new MyCallable());
        String s = submit.get();
        System.out.println(s);
    }

}
```

## 线程的状态

线程有六种状态，分别如下，可以通过Thread中的State内部枚举类来查看：

* NEW：尚未启动的线程处于此状态
* RUNNABLE：在Java虚拟机中执行的线程处于此状态
* BLOCKED：被阻塞等待监视器锁定的线程处于此状态
* WAITING：正在等待另一个线程执行特定动作的线程处于此状态
* TIMED_WAITING：正在等待另一个线程执行动作达到指定等待时间的线程处于此状态
* TERMINATED：已退出的线程处于此状态

```
public enum State {  
    NEW, // 初始状态
    RUNNABLE, // 运行状态
    BLOCKED, // 阻塞状态
    WAITING, // 等待状态
    TIMED_WAITING, // 超时等待状态
    TERMINATED; // 终止状态
}
```

{% asset_img 线程的状态.jpeg 线程的状态 %}

```
public class ThreadStatus implements Runnable {

    @Override
    public void run() {
        try {
            Thread.sleep(100L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "执行完毕");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new ThreadStatus(), "线程状态测试 -- ");

        System.out.println(thread.getState());

        thread.start();
        System.out.println(thread.getState());

        Thread.sleep(500L);
        System.out.println(thread.getState());
    }

}
```

## 并发问题

多个线程操作同时操作一个对象，线程不安全。

解决方法：线程同步。

```
public class Concurrency implements Runnable {

    private int ticketCount = 10;

    @Override
    public void run() {
        while (ticketCount > 0 ) {
            try {
                Thread.sleep(100L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "获得票: " + ticketCount--);
        }
    }

    public static void main(String[] args) {
        Concurrency concurrency = new Concurrency();
        Thread thread1 = new Thread(concurrency, "1-");
        Thread thread2 = new Thread(concurrency, "2-");
        Thread thread3 = new Thread(concurrency, "3-");
        thread1.start();
        thread2.start();
        thread3.start();
    }

}

// output
2-获得票: 9
1-获得票: 10
3-获得票: 9
3-获得票: 8
1-获得票: 7
2-获得票: 6
1-获得票: 5
3-获得票: 4
2-获得票: 3
1-获得票: 2
3-获得票: 1
2-获得票: 0
1-获得票: -1
```

## Thread中的静态代理

静态代理

```java
public class StaticProxy implements Sleep {

    private Myself myself;

    public StaticProxy(Myself myself) {
        this.myself = myself;
    }

    @Override
    public void sleep() {
        System.out.println("先洗漱");
        myself.sleep();
        System.out.println("睡醒整理床铺");
    }

    public static void main(String[] args) {
        StaticProxy staticProxy = new StaticProxy(new Myself());
        staticProxy.sleep();
    }

}

interface Sleep {
    void sleep();
}

class Myself implements Sleep{

    @Override
    public void sleep() {
        System.out.println("I sleep");
    }

}
```

在Thread类内部使用了类似的方法，接收一个Runnable的实例。

```java
/**
 * Allocates a new {@code Thread} object. This constructor has the same
 * effect as {@linkplain #Thread(ThreadGroup,Runnable,String) Thread}
 * {@code (null, target, gname)}, where {@code gname} is a newly generated
 * name. Automatically generated names are of the form
 * {@code "Thread-"+}<i>n</i>, where <i>n</i> is an integer.
 *
 * @param  target
 *         the object whose {@code run} method is invoked when this thread
 *         is started. If {@code null}, this classes {@code run} method does
 *         nothing.
 */
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
```

## 线程停止

线程停止不建议使用线程本身的stop、destroy方法，建议使用标记停止的方法。

```java
public class MyStopThread implements Runnable {

    boolean run = true;

    @Override
    public void run() {
        while (true) {
            for (int i = 0; i < 1000; i++) {
                if (!run) {
                    return;
                }
                try {
                    Thread.sleep(100L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(i);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        MyStopThread myStopThread = new MyStopThread();
        Thread thread = new Thread(myStopThread, "my stop thread");
        thread.start();
        Thread.sleep(1000L);
        myStopThread.run = false;
    }

}
```

## 线程休眠、线程礼让、线程强制执行

Thread.sleep线程休眠。

Object.yield线程礼让。

Thread.join线程强制执行，会中断当前的线程，强制执行join的线程到结束。

## 线程的优先级

Java提供一个线程调度器来监视程序中启动后进入就绪状态的多有线程，线程调度器按照优先级决定应该调度哪个线程来执行。

线程优先级用数字来表示，范围1~10。

使用getPriority获取线程优先级，使用setPriority来设置优先级，设置线程优先级需要在线程启动之前设置，否则无法生效。

并不是说优先级高的线程一定会先执行。

```java
public class ThreadPriority implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ", priority: " + Thread.currentThread().getPriority());
    }

    public static void main(String[] args) {
        List<Thread> threadList = new LinkedList<>();
        for (int i = 1; i <= 10; i++) {
            Thread thread = new Thread(new ThreadPriority(), "thread-" + i);
            thread.setPriority(i);
            threadList.add(thread);
        }
        threadList.forEach(Thread::start);
    }

}
// output:
thread-1, priority: 1
thread-4, priority: 4
thread-3, priority: 3
thread-2, priority: 2
thread-5, priority: 5
thread-6, priority: 6
thread-7, priority: 7
thread-8, priority: 8
thread-9, priority: 9
thread-10, priority: 10
```

## 守护线程

线程分为用户线程和守护线程，虚拟机确保用户线程执行完毕，但不用等待守护线程执行完毕，守护线程如操作日志、监控内存、垃圾回收等

```java
public class DaemonThread implements Runnable {

    private int count = 0;

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(100L);
                System.out.println(count++);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        DaemonThread daemonThread = new DaemonThread();
        Thread thread = new Thread(daemonThread, "daemonThread");
        thread.setDaemon(true); // 设置线程为守护线程
        thread.start();
        Thread.sleep(1000L);
        System.out.println(Thread.currentThread().getName() + " stop ");
    }

}
```

## 并发

同一个对象被多个线程同时操作。

处理多线程问题，多个线程访问同一个对象，并且会修改这个对象，需要用到线程同步，线程同步本质是一种等待机制，需要同时访问此对象的线程进入对象的等待池形成队列，等待前面的线程执行完毕，下一个线程再去执行。

线程同步：队列 + 锁，引入了锁机制`synchronized`，引入互斥锁后会出现以下问题：

+ 一个线程持有锁会导致其他所有需要此锁的线程挂起
+ 在多线程竞争的条件下，加锁、释放锁会导致比较多的上下文切换和调度延迟，引起性能问题
+ 如果一个优先级高的线程等待一个优先级低的线程释放锁，会导致优先级倒置，引起性能问题。

