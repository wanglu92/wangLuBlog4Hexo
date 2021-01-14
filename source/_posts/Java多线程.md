---
title: Java多线程
date: 2021-01-13 14:28:19
tags:

---

## 进程、线程、多线程

* 进程`process`，译作进程，是计算机中已运行程序的实体。进程本身不会运行，是线程的容器。程序本身只是指令的集合，进程才是程序(那些指令)的真正运行。若干进程有可能与同一个程序相关系，且每个进程皆可以同步(循序)或不同步(平行)的方式独立运行。进程为现今分时系统的基本运作单位。

* 线程`thread`，操作系统技术中的术语，是操作系统能够进行运算调度的最小单位。它被包涵在进程之中，一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。在Unix System V及SunOS中也被称为轻量进程(lightweight processes)，但轻量进程更多指内核线程(kernel thread)，而把用户线程(user thread)称为线程。
* 多线程，最开始，线程只是用于分配单个处理器的处理时间的一种工具。但假如操作系统本身支持多个处理器，那么每个线程都可分配给一个不同的处理器，真正进入“并行运算”状态。从程序设计语言的角度看，多线程操作最有价值的特性之一就是程序员不必关心到底使用了多少个处理器。程序在逻辑意义上被分割为数个线程;假如机器本身安装了多个处理器，那么程序会运行得更快，毋需作出任何特殊的调校。多线程是为了同步完成多项任务，不是为了提高运行效率，而是为了提高资源使用效率来提高系统的效率。线程是在同一时间需要完成多项任务的时候实现的。

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

## 线程不安全案例

+ 抢购火车票

```
public class BuyTicket implements Runnable {

    private int ticketCount = 10;

    void buy () throws InterruptedException {
        while (ticketCount > 0) {
            Thread.sleep(100L);
            System.out.println(Thread.currentThread().getName() + " -> 买到票" + ticketCount +  "，剩余:" + --ticketCount);
        }
    }

    @Override
    public void run() {
        try {
            buy();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        BuyTicket buyTicket = new BuyTicket();
        Thread thread1 = new Thread(buyTicket, "小明");
        Thread thread2 = new Thread(buyTicket, "小王");
        thread1.start();
        thread2.start();
    }

}

// output
小王 -> 买到票10，剩余:9
小明 -> 买到票10，剩余:8
小王 -> 买到票8，剩余:7
小明 -> 买到票7，剩余:6
小王 -> 买到票6，剩余:5
小明 -> 买到票6，剩余:5
小王 -> 买到票5，剩余:4
小明 -> 买到票5，剩余:3
小明 -> 买到票3，剩余:2
小王 -> 买到票2，剩余:1
小明 -> 买到票1，剩余:0
小王 -> 买到票1，剩余:-1
```

+ 银行统一账户多人同时取款

```
public class UnsafeBank {

    public static void main(String[] args) {
        Account account = new Account(100, "bank");
        DrawingMoney one = new DrawingMoney(account, 50, "小明");
        DrawingMoney two = new DrawingMoney(account, 100, "小王");
        one.start();
        two.start();
    }

}

class Account extends Thread {

    private int money;
    private String userNumber;

    public Account(int money, String userNumber) {
        this.money = money;
        this.userNumber = userNumber;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public String getUserNumber() {
        return userNumber;
    }

    public void setUserNumber(String userNumber) {
        this.userNumber = userNumber;
    }
}

class DrawingMoney extends Thread {

    private Account account;
    private int drawMoney;

    public DrawingMoney(Account account, int drawMoney, String name) {
        super(name);
        this.account = account;
        this.drawMoney = drawMoney;
    }

    void drawingMoney(Account account, int drawMoney) {
        System.out.println(Thread.currentThread().getName() + "取" + drawMoney);
        if (account.getMoney() < drawMoney) {
            System.out.println(Thread.currentThread().getName() + " 余额不足");
            return;
        }
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        account.setMoney(account.getMoney() - drawMoney);
        System.out.println(Thread.currentThread().getName() + "取" + drawMoney + ",剩余" + account.getMoney());
    }

    @Override
    public void run() {
        drawingMoney(account, drawMoney);
    }
}

// output
小王取100
小明取50
小明取50,剩余-50
小王取100,剩余-50
```

+ 不安全集合

```
public class UnsafeList {

    public static void main(String[] args) throws InterruptedException {
        List<String> unsafeList = new LinkedList<>();
        for (int i = 0; i < 10000; i++) {
            Thread thread = new Thread(() -> unsafeList.add(Thread.currentThread().getName()), "thread name: " + i);
            thread.start();
        }
        Thread.sleep(10000L);
        System.out.println(unsafeList.size());
    }

}
```

## 同步方法和同步代码块

+ 同步方法：在方法中加入`synchronized`关键字
+ 同步代码块：

```
synchronized (obj) {
	// 同步的内容
}
```

## 线程不安全问题解决案例

+ 购票问题

```
public class BuyTicket implements Runnable {

    private int ticketCount = 10;

    synchronized void buy () throws InterruptedException {
        while (ticketCount > 0) {
            Thread.sleep(100L);
            System.out.println(Thread.currentThread().getName() + " -> 买到票" + ticketCount +  "，剩余:" + --ticketCount);
        }
    }

    @Override
    public void run() {
        try {
            buy();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        BuyTicket buyTicket = new BuyTicket();
        Thread thread1 = new Thread(buyTicket, "小明");
        Thread thread2 = new Thread(buyTicket, "小王");
        thread1.start();
        thread2.start();
    }

}
```

+ 银行账户余额问题

```
public class UnsafeBank {

    public static void main(String[] args) {
        Account account = new Account(100, "bank");
        DrawingMoney one = new DrawingMoney(account, 50, "小明");
        DrawingMoney two = new DrawingMoney(account, 100, "小王");
        one.start();
        two.start();
    }

}

class Account extends Thread {

    private int money;
    private String userNumber;

    public Account(int money, String userNumber) {
        this.money = money;
        this.userNumber = userNumber;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public String getUserNumber() {
        return userNumber;
    }

    public void setUserNumber(String userNumber) {
        this.userNumber = userNumber;
    }
}

class DrawingMoney extends Thread {

    private Account account;
    private int drawMoney;

    public DrawingMoney(Account account, int drawMoney, String name) {
        super(name);
        this.account = account;
        this.drawMoney = drawMoney;
    }

    void drawingMoney(Account account, int drawMoney) {
        synchronized (account) {
            System.out.println(Thread.currentThread().getName() + "取" + drawMoney);
            if (account.getMoney() < drawMoney) {
                System.out.println(Thread.currentThread().getName() + " 余额不足");
                return;
            }
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            account.setMoney(account.getMoney() - drawMoney);
            System.out.println(Thread.currentThread().getName() + "取" + drawMoney + ",剩余" + account.getMoney());
        }
    }

    @Override
    public void run() {
        drawingMoney(account, drawMoney);
    }
}
```

+ 不安全集合

```
public class UnsafeList {

    public static void main(String[] args) throws InterruptedException {
        List<String> unsafeList = new LinkedList<>();
        for (int i = 0; i < 10000; i++) {
            Thread thread = new Thread(() -> {
                synchronized (unsafeList) {
                    unsafeList.add(Thread.currentThread().getName());
                }
            }, "thread name: " + i);
            thread.start();
        }
        Thread.sleep(10000L);
        System.out.println(unsafeList.size());
    }

}
```

## 死锁问题

多个线程各自占有一些共享资源，并且相互其他资源才能继续运行，两个或者多个线程都在等待对方释放资源，双方都停止执行的情况，称为死锁。

当一个同步块同时拥有**两个以上的对象锁**时，就有可能出现死锁问题。

```
public class DeadLock {

    public static void main(String[] args) {

        KeyOne keyOne = new KeyOne();
        KeyTwo keyTwo = new KeyTwo();

        Thread thread1 = new Thread(() -> {
            synchronized (keyOne) {
                System.out.println(Thread.currentThread().getName() + " I have key one, I need key two");
                try {
                    Thread.sleep(100L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (keyTwo) {
                    System.out.println(Thread.currentThread().getName() + " finish");
                }
            }
        }, "thread-1");

        Thread thread2 = new Thread(() -> {
            synchronized (keyTwo) {
                System.out.println(Thread.currentThread().getName() + " I have key two, I need key one");
                try {
                    Thread.sleep(100L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (keyOne) {
                    System.out.println(Thread.currentThread().getName() + " finish");
                }
            }
        }, "thread-1");

        thread1.start();
        thread2.start();

    }

}

class KeyOne {

}

class KeyTwo {

}

// output
thread-1 I have key one, I need key two
thread-1 I have key two, I need key one
```

## Lock

从JDK1.5开始，可以显式定义同步锁。

`ReentrantLock`为可重入锁，在`java.util.concurrent.locks`下。

```
public class LockDemo implements Runnable {

    private int ticketCount = 10;
    private Lock lock = new ReentrantLock();

    void buy() throws InterruptedException {
        try {
            lock.lock();
            while (ticketCount > 0) {
                Thread.sleep(100L);
                System.out.println(Thread.currentThread().getName() + " -> 买到票" + ticketCount + "，剩余:" + --ticketCount);
            }
        } catch (Exception e) {
            System.out.println(e);
        } finally {
            lock.unlock();
        }
    }

    @Override
    public void run() {
        try {
            buy();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        BuyTicket buyTicket = new BuyTicket();
        Thread thread1 = new Thread(buyTicket, "小明");
        Thread thread2 = new Thread(buyTicket, "小王");
        thread1.start();
        thread2.start();
    }

}
```

## 生产者消费者问题

> 问题描述：
>
> + 假设仓库中只能存放一件产品，生产者将生产出的商品放入仓库中，消费者将仓库中的产品取走消费掉。
> + 如果仓库中没有产品，则生产者将产品放入仓库，否则停止生产并等待，直到仓库中的产品被消费者取走为止。
> + 如果仓库中放有产品，则消费者可以将产品取走消费，否则停止消费并等待，直到仓库中再次放入产品为止。

Java提供的几个解决线程之间通信问题的方法，线程通信是通信在等待队列中的线程，需要用锁对象来调用线程间通信方法。调用这些方法必须在`synchronized`中否则会抛出`IllegalMonitorStateException`异常

+ wait()
+ ait(long timeout)
+ notify()
+ notifyAll()

利用缓冲区解决问题，也可以利用标记来解决问题。

```java
public class ProducerAndConsumer {

    public static void main(String[] args) {
        ProductContainer productContainer = new ProductContainer();
        for (int i = 0; i < 100; i++) {
            Producer producer = new Producer(productContainer, "producer -> " + i);
            Consumer consumer = new Consumer(productContainer, "consumer -> " + i);
            producer.start();
            consumer.start();
        }
    }

}

class Producer extends Thread {

    private ProductContainer container;

    public Producer(ProductContainer container, String name) {
        super(name);
        this.container = container;
    }

    public void product(Production production) throws InterruptedException {
        container.push(production);
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                product(new Production(Thread.currentThread().getName() + " & " + i));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Consumer extends Thread {

    private ProductContainer container;

    public Consumer(ProductContainer container, String name) {
        super(name);
        this.container = container;
    }

    public void consume() throws InterruptedException {
        container.pop();
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 100; i++) {
                consume();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class ProductContainer {

    /**
     * 安全容器用来存贮商品
     */
    private LinkedList<Production> productions = new LinkedList<>();

    public void push(Production production) throws InterruptedException {
        synchronized (this) {
            // 多生产者多消费者是需要使用while循环判断
            while (productions.size() >= 10) {
                this.wait();
            }
            productions.push(production);
            System.out.println(Thread.currentThread().getName() + " ++ " + production.getProductId());
            this.notifyAll();
        }
    }

    public void pop() throws InterruptedException {
        synchronized (this) {
            while (productions.size() <= 0) {
                this.wait();
            }
            Production production = productions.pop();
            System.out.println(Thread.currentThread().getName() + " -- " + production.getProductId());
            this.notifyAll();
        }
    }

}

class Production {

    private String productionId;

    public Production(String productId) {
        this.productionId = productId;
    }

    public String getProductId() {
        return productionId;
    }

}
```

## 线程池

经常创建和销毁线程会使用大量资源，对性能的影响很大。

可以提前创建好一些线程，在使用的时候直接获取，使用完后放回池子中循环利用。

好处：

+ 提高响应速度
+ 降低资源消耗
+ 便于线程管理

核心参数：

+ corePoolSize：核心线程数量
+ maximumPoolSize：最大线程数
+ keepAliveTime：等待线程最大存活时间

> 使用到的类`ExecutorService`和`Executors`
>
> `ExecutorService`：真正的线程池接口，常见子类ThreadPoolExecutor。
>
> `Executors`：工具类，线程池的工厂类，用于创建并返回不同类型的线程池。





