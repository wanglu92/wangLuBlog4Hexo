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

池化技术

优化资源的技术 -> 池化技术

***线程池的好处***

1. 降低资源的消耗
2. 提高响应的速度
3. 方便管理

线程复用、控制最大并发数、管理线程

**线程池：三大方法、七大参数、四种拒绝策略**

> Executors三大方法
>
> + ```java
>   ExecutorService executorService = Executors.newCachedThreadPool();
>   ExecutorService executorService1 = Executors.newFixedThreadPool(10);
>   ExecutorService executorService2 = Executors.newSingleThreadExecutor();
>   ```

> 七大参数
>
> + ```
>   public static ExecutorService newCachedThreadPool() {
>           return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
>                                         60L, TimeUnit.SECONDS,
>                                         new SynchronousQueue<Runnable>());
>       }
>   ```
>
> + ```
>   public static ExecutorService newFixedThreadPool(int nThreads) {
>           return new ThreadPoolExecutor(nThreads, nThreads,
>                                         0L, TimeUnit.MILLISECONDS,
>                                         new LinkedBlockingQueue<Runnable>());
>       }
>   ```
>
> + ```
>   public static ExecutorService newSingleThreadExecutor() {
>           return new FinalizableDelegatedExecutorService
>               (new ThreadPoolExecutor(1, 1,
>                                       0L, TimeUnit.MILLISECONDS,
>                                       new LinkedBlockingQueue<Runnable>()));
>       }
>   ```
>
> ```java
> /**
>      * Creates a new {@code ThreadPoolExecutor} with the given initial
>      * parameters.
>      *
>      * @param corePoolSize the number of threads to keep in the pool, even
>      *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
>      * @param maximumPoolSize the maximum number of threads to allow in the
>      *        pool
>      * @param keepAliveTime when the number of threads is greater than
>      *        the core, this is the maximum time that excess idle threads
>      *        will wait for new tasks before terminating.
>      * @param unit the time unit for the {@code keepAliveTime} argument
>      * @param workQueue the queue to use for holding tasks before they are
>      *        executed.  This queue will hold only the {@code Runnable}
>      *        tasks submitted by the {@code execute} method.
>      * @param threadFactory the factory to use when the executor
>      *        creates a new thread
>      * @param handler the handler to use when execution is blocked
>      *        because the thread bounds and queue capacities are reached
>      * @throws IllegalArgumentException if one of the following holds:<br>
>      *         {@code corePoolSize < 0}<br>
>      *         {@code keepAliveTime < 0}<br>
>      *         {@code maximumPoolSize <= 0}<br>
>      *         {@code maximumPoolSize < corePoolSize}
>      * @throws NullPointerException if {@code workQueue}
>      *         or {@code threadFactory} or {@code handler} is null
>      */
>     public ThreadPoolExecutor(int corePoolSize,
>                               int maximumPoolSize,
>                               long keepAliveTime,
>                               TimeUnit unit,
>                               BlockingQueue<Runnable> workQueue,
>                               ThreadFactory threadFactory,
>                               RejectedExecutionHandler handler) {
>         if (corePoolSize < 0 ||
>             maximumPoolSize <= 0 ||
>             maximumPoolSize < corePoolSize ||
>             keepAliveTime < 0)
>             throw new IllegalArgumentException();
>         if (workQueue == null || threadFactory == null || handler == null)
>             throw new NullPointerException();
>         this.acc = System.getSecurityManager() == null ?
>                 null :
>                 AccessController.getContext();
>         this.corePoolSize = corePoolSize;
>         this.maximumPoolSize = maximumPoolSize;
>         this.workQueue = workQueue;
>         this.keepAliveTime = unit.toNanos(keepAliveTime);
>         this.threadFactory = threadFactory;
>         this.handler = handler;
>     }
> ```
>
> 

> 四种拒绝策略
>
> ```java
> /**
>      * A handler for rejected tasks that throws a
>      * {@code RejectedExecutionException}.
>      */
> public static class AbortPolicy implements RejectedExecutionHandler {
>   /**
>          * Creates an {@code AbortPolicy}.
>          */
>   public AbortPolicy() { }
> 
>   /**
>          * Always throws RejectedExecutionException.
>          *
>          * @param r the runnable task requested to be executed
>          * @param e the executor attempting to execute this task
>          * @throws RejectedExecutionException always
>          */
>   public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
>     throw new RejectedExecutionException("Task " + r.toString() +
>                                          " rejected from " +
>                                          e.toString());
>   }
> }
> ```
>
> ```java
> /**
>      * A handler for rejected tasks that runs the rejected task
>      * directly in the calling thread of the {@code execute} method,
>      * unless the executor has been shut down, in which case the task
>      * is discarded.
>      */
> public static class CallerRunsPolicy implements RejectedExecutionHandler {
>   /**
>          * Creates a {@code CallerRunsPolicy}.
>          */
>   public CallerRunsPolicy() { }
> 
>   /**
>          * Executes task r in the caller's thread, unless the executor
>          * has been shut down, in which case the task is discarded.
>          *
>          * @param r the runnable task requested to be executed
>          * @param e the executor attempting to execute this task
>          */
>   public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
>     if (!e.isShutdown()) {
>       r.run();
>     }
>   }
> }
> ```
>
> ```java
> /**
>      * A handler for rejected tasks that discards the oldest unhandled
>      * request and then retries {@code execute}, unless the executor
>      * is shut down, in which case the task is discarded.
>      */
> public static class DiscardOldestPolicy implements RejectedExecutionHandler {
>   /**
>          * Creates a {@code DiscardOldestPolicy} for the given executor.
>          */
>   public DiscardOldestPolicy() { }
> 
>   /**
>          * Obtains and ignores the next task that the executor
>          * would otherwise execute, if one is immediately available,
>          * and then retries execution of task r, unless the executor
>          * is shut down, in which case task r is instead discarded.
>          *
>          * @param r the runnable task requested to be executed
>          * @param e the executor attempting to execute this task
>          */
>   public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
>     if (!e.isShutdown()) {
>       e.getQueue().poll();
>       e.execute(r);
>     }
>   }
> }
> ```
>
> ```java
> /**
>      * A handler for rejected tasks that silently discards the
>      * rejected task.
>      */
> public static class DiscardPolicy implements RejectedExecutionHandler {
>   /**
>          * Creates a {@code DiscardPolicy}.
>          */
>   public DiscardPolicy() { }
> 
>   /**
>          * Does nothing, which has the effect of discarding task r.
>          *
>          * @param r the runnable task requested to be executed
>          * @param e the executor attempting to execute this task
>          */
>   public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
>   }
> }
> ```

## CPU密集型和IO密集型

+ CPU密集型计算比较多的任务，合适使用核心数
+ IO密集型，适合使用最大线程数的两倍

涉及到调优：最大的线程如何设置

## 四大函数式接口

lambda表达式、函数式接口、Stream流计算

> 函数式接口：四大原生的函数式接口
>
> + Consumer： 消费型接口，一个输入，没有输出
>
> + Function：一个输入一个输出。
>
> + Predicate：断定接口，一个输入，返回boolean值。
>
> + Supplier：供给型接口，没有输入，一个输出。

## Stream流

## ForkJoin

ForkJoin 1.7 出现，并行执行任务，提高运行效率，大数据量。

MapReduce思想，把大任务拆分为小任务

工作窃取：线程维护的是双端队列

## 异步回调

## JMM

JMM：java内存模型

关于JMM的一些同步的约定

1. 线程解锁前，必须把共享变量**立刻**刷新到主存中。
2. 线程加锁前，必须读取主寸中的最新的值到工作内存中。
3. 加锁和解锁必须是同一把锁

线程：工作内存、主存

> 8种操作、4组
>
> 1. read、load：从主寸中加载到工作内存中。
> 2. use、assign：从工作内存加载到执行引擎中。
> 3. write、store：将工作内存中的数据刷回主寸中。
> 4. lock、unlock：给数据加锁，作用于主内存，标记变量为一个线程独占。

## Volatile

volatile是java提供的轻量级的同步机制

1. 保证可见性
   + 保证别的线程对数据的可见性
2. 不保证原子性
   + 使用原子类解决单个资源的原子性问题
   + Atomic包下提供了一些基本数据类型的原子类（直接在内存类中修改值）
3. 禁止指令重排
   + 源代码 -》 编译器优化 -》指令并行会重排-》内存系统重排-》执行。
   + 编译和优化中会将指令重排。
   + 编译器在进行指令重排的时候，会考虑数据的依赖性。
   + 内存屏障？CPU指令作用：保证特定的操作的执行顺序。可以保证某些变量的内存的可见性（李忠这些特性volatile实现了可见性） 

## CAS

`compare and swap` 乐观锁的一种实现

java中unsafe类中提供native方法实现

## 单例模式

饿汉式

DCL懒汉式

使用反射破坏单例模式-在构造器中加入判断防止反射破坏

使用枚举类实现单例模式

## 深入理解CAS

`compare and set` 更新并交换，乐观锁思想的一种实现。 

CAS是CPU的并发原语

UnSafe类，java可以通过这个类操作内存。

> 缺点？：
>
> 1. 循环耗时？？？
> 2. 一次性只能保证一个共享变量的原子性
> 3. 会存在ABA问题

## CAS的ABA问题

使用带版本号的原子引用来解决ABA问题

## 各种锁的理解

### 公平锁、非公平锁

+ 公平锁：按照先来后到
+ 非公平锁：可以插队（默认使用非公平锁）

### 可重入锁

可重入锁、递归锁

 如果线程已经拿到锁，线程中执行的方法也可以拿到锁。

### 自旋锁

spinlock

自己使用原子类实现自旋锁

### 死锁

持有资源，并尝试获取一个不能获得的资源。

排查解决问题

jdk bin 

1. 使用jps定位进程 `jps -l` 看java的进程

2. 使用jstack看进程 `jstack 进程号 

面试工作中，排查问题

1. 日志
2. 堆栈信息  