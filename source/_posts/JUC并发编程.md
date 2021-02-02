---
title: JUC并发编程
date: 2021-01-30 18:07:15
tags:
---

## 什么是JUC

`java.util.concurrent`包下提供的一些方法

### 线程和进程

进程：一个程序

一个进程往往可以包含多个线程，至少包含一个线程

java默认有几个线程？2个 main、GC

线程：先进程中运行

java使用native方法开启线程

### 并发和并行

并发编程：并发、并行

并发（多个线程操作同一个资源）

+ CPU一核，模拟出多条线程，快速交替

并行（多个核同时执行）

+ CPU多核，多个线程可以同时执行；线程池？

```JAVA
// 获取CPU核数
System.out.println(Runtime.getRuntime().availableProcessors());
```

并发编程的本质：充分利用CPU的资源

### 线程的几个状态

```java
public enum State {
  /**
         * Thread state for a thread which has not yet started.
         */
  NEW,

  /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
  RUNNABLE,

  /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
  BLOCKED,

  /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
  WAITING,

  /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
  TIMED_WAITING,

  /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
  TERMINATED;
    }
```

### wait/sleep

+ 来自不同的类 Object、Thread
+ 关于锁的释放 wait会释放 sleep不会释放锁
+ 使用的范围是不同的 wait只能在同步代码块中使用 sleep可以在任何地方使用
+ 是否需要捕获异常

## Lock锁（重点）

> 传统Synchronized

线程就是一个单独的资源类，没有任何附属操作

1. 属性 方法 
2. 并发：多线程操 作同一个资源类，把资源放入到线程中 

> Lock接口
>
> + ReentrantLock
> + ReentrantReadWriteLock.ReadLock
> + ReentrantReadWriteLock.WriteLock

公平锁：先来后到

非公平锁：可以插队（默认 ）

> Synchronized和Lock区别
>
> 1. Synchronized内置的java关键字，Lock是一个java类
> 2. Synchronized无法判断获取锁的状态，Lock可以判断是否获取到了锁
> 3. Synchronized会自动释放锁，Lock必须要手动释放锁，如果不释放锁，会出现死锁问题
> 4. Synchronized线程一（获取锁、阻塞）、线程二（等待）Lock锁就不一定会等待下去。
> 5. Synchronized可重入，不可中断，非公平锁；Lock可以设置，可以判断，可设置的锁
> 6. Synchronized适合少量的代码同步问题，Lock适合锁大量的同步代码块

> 锁是什么？如何判断锁的是谁？

## 生产者消费者问题

线程之间的通信问题

防止**虚假唤醒**问题不能使用if判断，需要使用while判断

使用Lock实现生产者和消费者问题

+ 通过Lock可以获取Condition对象
+ await方法
+ signal方法

Condition可以精准的通知和唤醒线程

## 八锁现象：关于锁的八个问题

如何判断锁是谁

永远知道什么锁，所得到底是谁

对象、Class

## 集合类不安全(面试重点)

> List不安全
>
> CopyOnWriteArrayList

> Set不安全
>
> CopyOnWriteSet

> Map不安全
>
> ConcurrentHashMap
>
> `loadFactor` 加载因子 `static final float DEFAULT_LOAD_FACTOR = 0.75f;`
>
> `initialCapacity` 初始化容量 16
>
> ConcurrentHashMap原理？？？

## Callable

1. 可以有返回值
2. 可以抛出异常
3. 调用方法不同

```java
// 一种启动方式
new Thread(new FutureTask<String>(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "123";
            }
        })).start();
```

+ 结果有缓存
+ 结果可能需要等待，会阻塞线程。

## JUC中常用辅助类

+ CountDownLatch

可以理解为加法计数器

允许一个或多个线程等待直到其他线程中执行的一组操作完成同步。用来计数

```java
CountDownLatch countDownLatch = new CountDownLatch(10);
        countDownLatch.countDown();
        countDownLatch.await();
        countDownLatch.await(1000, TimeUnit.SECONDS);
```



+ CyclicCarrier

可以理解为减法计数器

凑齐等待后执行

```
public class Main60 {

    public static void main(String[] args) throws Exception {

        CyclicBarrier cyclicCarrier = new CyclicBarrier(7, () -> {
            System.out.println("123");
        });
        
        for (int i = 0; i < 7; i++) {
            final int temp = i;
            new Thread(() -> {
                System.out.println(temp);
                try {
                    cyclicCarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }

    }

}

// output
0
4
3
5
2
1
6
123
```



+ Semaphore

`Semaphore` 信号量 

```java
Semaphore semaphore = new Semaphore(6);
semaphore.acquire(); // 获取
semaphore.release(); // 释放
```

## ReadWriteLock

读写锁：读的时候可以被多个线程同时读，写的时候只有一个线程可以操作。

读写锁注意读锁和写锁是互斥的，读取的时候不可以写入。防止幻读。

独占锁？排它锁？

共享锁？

## 阻塞队列

+ 阻塞`block`

+ 队列`queue`
  + 写入：如果队列满，阻塞等待
  + 取：如果队列是空的，必须等待生产

什么情况下会使用阻塞队列？

多线程、线程池

添加、移除

**四组API**

1. 抛出异常
2. 不抛出异常
3. 阻塞等待
4. 超时等待

| 方式     | 抛出异常 | 有返回值 | 阻塞等待 | 超时等待  |
| -------- | -------- | -------- | -------- | --------- |
| 添加     | add      | offer    | put      | off time  |
| 移除     | remove   | poll     | take     | poll time |
| 检测队首 | element  | peek     | -        | -         |

## 同步队列SynchronousSueue

没有容量：进入一个元素，必须等待取出后，才能再放入元素。 

## 线程池

