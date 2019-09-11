---
layout: post
title: Java 并发Concurrent
categories: Blog
description:  Java 并发工具包 Concurrent详解
keywords:     并发，Java

---

# Java 并发Concurrent

参见 [ Java 并发工具箱](http://blog.csdn.net/wbwjx/article/details/57856045),

## 概述

java.util.concurrent 包是专为 Java`并发编程`而设计的包。包下的所有类可以分为如下几大类：

- locks部分：显式锁(互斥锁和速写锁)相关；
- atomic部分：原子变量类相关，是构建非阻塞算法的基础；
- executor部分：线程池相关；
- collections部分：并发容器相关；
- tools部分：同步工具相关，如信号量、闭锁、栅栏等功能；

## 类图结构：

![J.U.C](http://img.blog.csdn.net/20170301212847989?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2J3ang=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

脑图地址： <http://www.xmind.net/m/tJy5>，感谢[深入浅出 Java Concurrency ](http://www.blogjava.net/xylz/archive/2010/06/30/324915.html)，此脑图在这篇基础上修改而来。

## BlockingQueue

此接口是一个`线程安全`的 `存取实例`的队列。

### 使用场景

BlockingQueue通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。

![BlockingQueue](http://img.blog.csdn.net/20170227071415737?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2J3ang=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 注意事项：

- 此队列是`有限的`,如果队列到达临界点，`Thread1`就会阻塞，直到`Thread2`从队列中拿走一个对象。
- 若果队列是空的，`Thread2`会阻塞，直到`Thread1`把一个对象丢进队列。

### 相关方法

BlockingQueue中包含了如下操作方法：

|         | Throws Exception | Special Value | Blocks | Times Out                   |
| ------- | ---------------- | ------------- | ------ | --------------------------- |
| Insert  | add(o)           | offer(o)      | put(o) | offer(o, timeout, timeunit) |
| Remove  | remove(o)        | poll()        | take() | poll(timeout, timeunit)     |
| Examine | element()        | peek()        |        |                             |

#### 名词解释：

- Throws Exception: 如果试图的操作无法立即执行，抛一个异常。
- Special Value: 如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
- Blocks: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
- Times Out: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

#### 注意事项：

- 无法插入 null，否则会抛出一个 NullPointerException。
- 队列这种数据结构，导致除了获取开始和结尾位置的其他对象的效率都不高，虽然可通过`remove(o)`来移除任一对象。

### 实现类

因为是一个接口，所以我们必须使用一个实现类来使用它，有如下实现类：

- ArrayBlockingQueue： 数组阻塞队列
- DelayQueue： 延迟队列
- LinkedBlockingQueue： 链阻塞队列
- PriorityBlockingQueue： 具有优先级的阻塞队列
- SynchronousQueue： 同步队列

### 使用示例：

见： [BlockingQueue](http://tutorials.jenkov.com/java-util-concurrent/blockingqueue.html)

## ArrayBlockingQueue

ArrayBlockingQueue 是一个有界的阻塞队列

- 内部实现是将对象放到一个数组里。数组有个特性：一旦初始化，大小就无法修改。因此无法修改`ArrayBlockingQueue`初始化时的上限。
- `ArrayBlockingQueue` 内部以 `FIFO(先进先出)`的顺序对元素进行存储。队列中的头元素在所有元素之中是放入时间最久的那个，而尾元素则是最短的那个。

## DelayQueue

DelayQueue 对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现 java.util.concurrent.Delayed 接口:

```
public interface Delayed extends Comparable<Delayed< {  
 public long getDelay(TimeUnit timeUnit);  // 返回将要延迟的时间段
}123
```

- 在每个元素的 getDelay() 方法返回的值的时间段之后才释放掉该元素。如果返回的是 0 或者负值，延迟将被认为过期，该元素将会在 DelayQueue 的下一次 take 被调用的时候被释放掉。
- `Delayed` 接口也继承了 `java.lang.Comparable` 接口，`Delayed`对象之间可以进行对比。这对`DelayQueue` 队列中的元素进行排序时有用，因此它们可以根据过期时间进行有序释放。

## LinkedBlockingQueue

内部以一个链式结构(链接节点)对其元素进行存储 。

- 可以选择一个上限。如果没有定义上限，将使用 Integer.MAX_VALUE 作为上限。
- 内部以 FIFO(先进先出)的顺序对元素进行存储。

## PriorityBlockingQueue

一个无界的并发队列，它使用了和类 java.util.PriorityQueue 一样的排序规则。

- 无法向这个队列中插入 null 值。
- 插入到 其中的元素必须实现 java.lang.Comparable 接口。
- 对于具有相等优先级(compare() == 0)的元素并不强制任何特定行为。
- 从一个 PriorityBlockingQueue 获得一个 Iterator 的话，该 Iterator 并不能保证它对元素的遍历是以优先级为序的。

## SynchronousQueue

一个特殊的队列，它的内部同时只能够容纳单个元素。

- 如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。
- 如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。

## BlockingDeque

此接口表示一个`线程安全`放入和提取实例的`双端队列`。

### 使用场景

通常用在一个线程既是生产者又是消费者的时候。 
![BlockingDeque](http://img.blog.csdn.net/20170227071507432?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2J3ang=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 注意事项

- 如果双端队列已满，插入线程将被阻塞，直到一个移除线程从该队列中移出了一个元素。
- 如果双端队列为空，移除线程将被阻塞，直到一个插入线程向该队列插入了一个新元素。

### 相关方法

|         | Throws Exception | Special Value | Blocks       | Times Out                        |
| ------- | ---------------- | ------------- | ------------ | -------------------------------- |
| Insert  | addFirst(o)      | offerFirst(o) | putFirst(o)  | offerFirst(o, timeout, timeunit) |
| Remove  | removeFirst(o)   | pollFirst(o)  | takeFirst(o) | pollFirst(timeout, timeunit)     |
| Examine | getFirst(o)      | peekFirst(o)  |              |                                  |

|         | Throws Exception | Special Value | Blocks      | Times Out                       |
| ------- | ---------------- | ------------- | ----------- | ------------------------------- |
| Insert  | addLast(o)       | offerLast(o)  | putLast(o)  | offerLast(o, timeout, timeunit) |
| Remove  | removeLast(o)    | pollLast(o)   | takeLast(o) | pollLast(timeout, timeunit)     |
| Examine | getLast(o)       | peekLast(o)   |             |                                 |

#### 注意事项

- 关于方法的处理方式和上节一样。
- BlockingDeque 接口继承自 BlockingQueue 接口，可以用其中定义的方法。

### 实现类

- LinkedBlockingDeque ： 链阻塞双端队列

## LinkedBlockingDeque

LinkedBlockingDeque 是一个双端队列，可以从任意一端插入或者抽取元素的队列。

- 在它为空的时候，一个试图从中抽取数据的线程将会阻塞，无论该线程是试图从哪一端抽取数据。

## ConcurrentMap

一个能够对别人的访问(插入和提取)进行并发处理的 java.util.Map接口。 
ConcurrentMap 除了从其父接口 java.util.Map 继承来的方法之外还有一些额外的原子性方法。

### 实现类

因为是接口，必须用实现类来使用它，其实现类为

- ConcurrentHashMap

### ConcurrentHashMap与HashTable比较

- 更好的并发性能，在你从中读取对象的时候 ConcurrentHashMap 并不会把整个 Map 锁住，只是把 Map 中正在被写入的部分进行锁定。
- 在被遍历的时候，即使是 ConcurrentHashMap 被改动，它也不会抛 ConcurrentModificationException。

## ConcurrentNavigableMap

一个支持并发访问的 java.util.NavigableMap，它还能让它的子 map 具备并发访问的能力。

### headMap

headMap(T toKey) 方法返回一个包含了小于给定 toKey 的 key 的子 map。

### tailMap

tailMap(T fromKey) 方法返回一个包含了不小于给定 fromKey 的 key 的子 map。

### subMap

subMap() 方法返回原始 map 中，键介于 from(包含) 和 to (不包含) 之间的子 map。

### 更多方法

- descendingKeySet()
- descendingMap()
- navigableKeySet()

## CountDownLatch

CountDownLatch 是一个并发构造，它允许一个或多个线程等待一系列指定操作的完成。

- CountDownLatch 以一个给定的数量初始化。countDown() 每被调用一次，这一数量就减一。
- 通过调用 await() 方法之一，线程可以阻塞等待这一数量到达零。

## CyclicBarrier

CyclicBarrier 类是一种同步机制，它能够对处理一些算法的线程实现同步。

更多实例参考： [CyclicBarrier](http://tutorials.jenkov.com/java-util-concurrent/cyclicbarrier.html)

## Exchanger

Exchanger 类表示一种两个线程可以进行互相交换对象的会和点。

更多实例参考： [Exchanger](http://tutorials.jenkov.com/java-util-concurrent/exchanger.html)

## Semaphore

Semaphore 类是一个计数信号量。具备两个主要方法：

- acquire()
- release()

> - 每调用一次 acquire()，一个许可会被调用线程取走。
> - 每调用一次 release()，一个许可会被返还给信号量。

### Semaphore 用法

- 保护一个重要(代码)部分防止一次超过 N 个线程进入。
- 在两个线程之间发送信号。

### 保护重要部分

如果你将信号量用于保护一个重要部分，试图进入这一部分的代码通常会首先尝试获得一个许可，然后才能进入重要部分(代码块)，执行完之后，再把许可释放掉。

```
Semaphore semaphore = new Semaphore(1);  
//critical section  
semaphore.acquire();  
...  
semaphore.release();12345
```

### 在线程之间发送信号

如果你将一个信号量用于在两个线程之间传送信号，通常你应该用一个线程调用 acquire() 方法，而另一个线程调用 release() 方法。

- 如果没有可用的许可，acquire() 调用将会阻塞，直到一个许可被另一个线程释放出来。
- 如果无法往信号量释放更多许可时，一个 release() 调用也会阻塞。

### 公平性

无法担保掉第一个调用 acquire() 的线程会是第一个获得一个许可的线程。

可以通过如下来强制公平：

```
Semaphore semaphore = new Semaphore(1, true); 1
```

- 需要注意，强制公平会影响到并发性能，建议不使用。

## ExecutorService

这里之前有过简单的总结： [Java 中几种常用的线程池](http://blog.csdn.net/wbwjx/article/details/51156125)

存在于 java.util.concurrent 包里的 ExecutorService 实现就是一个线程池实现。

### 实现类

此接口实现类包括：

- ScheduledThreadPoolExecutor ： 通过 `Executors.newScheduledThreadPool(10)`创建的
- ThreadPoolExecutor: 除了第一种的`其他三种方式`创建的

### 相关方法

- execute(Runnable)： 
  无法得知被执行的 Runnable 的执行结果
- submit(Runnable)： 
  返回一个 Future 对象，可以知道Runnable 是否执行完毕。
- submit(Callable)： 
  Callable 实例除了它的 call() 方法能够返回一个结果，通过Future可以获取。
- invokeAny(…)： 
  传入一系列的 Callable 或者其子接口的实例对象，无法保证返回的是哪个 Callable 的结果 ，只能表明其中一个已执行结束。 
  如果其中一个任务执行结束(或者抛了一个异常)，其他 Callable 将被取消。
- invokeAll(…)： 
  返回一系列的 Future 对象，通过它们你可以获取每个 Callable 的执行结果。

### 关闭ExecutorService

- shutdown() ： 不会立即关闭，但它将不再接受新的任务
- shutdownNow()： 立即关闭

## ThreadPoolExecutor

- ThreadPoolExecutor 使用其内部池中的线程执行给定任务(Callable 或者 Runnable)。

## ScheduledExecutorService（接口，其实现类为ScheduledThreadPoolExecutor）

- ScheduledExecutorService能够将任务延后执行，或者间隔固定时间多次执行。
- ScheduledExecutorService中的 任务由一个工作者线程异步执行，而不是由提交任务给 ScheduledExecutorService 的那个线程执行。

### 相关方法

- schedule (Callable task, long delay, TimeUnit timeunit)： 
  Callable 在给定的延迟之后执行，并返回结果。
- schedule (Runnable task, long delay, TimeUnit timeunit) 
  除了 Runnable 无法返回一个结果之外，和第一个方法类似。
- scheduleAtFixedRate (Runnable, long initialDelay, long period, TimeUnit timeunit) 
  这一方法规划一个任务将被定期执行。该任务将会在首个 initialDelay 之后得到执行，然后每个 period 时间之后重复执行。 
  period 被解释为前一个执行的开始和下一个执行的开始之间的间隔时间。
- scheduleWithFixedDelay (Runnable, long initialDelay, long period, TimeUnit timeunit) 
  和上一个方法类似，只是period 则被解释为前一个执行的结束和下一个执行的结束之间的间隔。

## ForkJoinPool

ForkJoinPool 在 Java 7 中被引入。它和 ExecutorService 很相似，除了一点不同。ForkJoinPool 让我们可以很方便地把任务分裂成几个更小的任务，这些分裂出来的任务也将会提交给 ForkJoinPool。

用法参考：[Java Fork and Join using ForkJoinPool ](http://tutorials.jenkov.com/java-util-concurrent/java-fork-and-join-forkjoinpool.html)

## Lock

Lock 是一个类似于 synchronized 块的线程同步机制。但是 Lock 比 synchronized 块更加灵活、精细。

### 实现类

Lock是一个接口，其实现类包括：

- ReentrantLock

### 示例

```
Lock lock = new ReentrantLock();  
lock.lock();  
//critical section  
lock.unlock();1234
```

- 调用`lock() 方法`之后，这个 lock 实例就被锁住啦。
- 当lock示例被锁后，任何其他再过来调用 lock() 方法的线程将会被阻塞住，直到调用了unlock() 方法。
- unlock() 被调用了，lock 对象解锁了，其他线程可以对它进行锁定了。

### Lock 和 synchronized区别

- synchronized 代码块不能够保证进入访问等待的线程的先后顺序。
- 你不能够传递任何参数给一个 synchronized 代码块的入口。因此，对于 synchronized 代码块的访问等待设置超时时间是不可能的事情。
- synchronized 块必须被完整地包含在单个方法里。而一个 Lock 对象可以把它的 lock() 和 unlock() 方法的调用放在不同的方法里。

## ReadWriteLock

读写锁一种先进的线程锁机制。

- 允许多个线程在同一时间对某特定资源进行读取，
- 但同一时间内只能有一个线程对其进行写入。

### 实现类

- ReentrantReadWriteLock

### 规则

- 如果没有任何写操作锁定，那么可以有多个读操作锁定该锁
- 如果没有任何读操作或者写操作，只能有一个写线程对该锁进行锁定。

示例：

```
ReadWriteLock readWriteLock = new ReentrantReadWriteLock();  
readWriteLock.readLock().lock();  
    // multiple readers can enter this section  
    // if not locked for writing, and not writers waiting  
    // to lock for writing.  
readWriteLock.readLock().unlock();  
readWriteLock.writeLock().lock();  
    // only one writer can enter this section,  
    // and only if no threads are currently reading.  
readWriteLock.writeLock().unlock();12345678910
```

## 更多原子性包装类

位于 `atomic`包下，包含一系列原子性变量。

- AtomicBoolean
- AtomicInteger
- AtomicLong
- AtomicReference 
  …

参考资料： [Java 并发工具包 java.util.concurrent 用户指南](http://blog.csdn.net/defonds/article/details/44021605#t19) 
[java.util.concurrency - Java Concurrency Utilities](http://tutorials.jenkov.com/java-util-concurrent/java-fork-and-join-forkjoinpool.html)

扩展阅读： 
[深入浅出 Java Concurrency ](http://www.blogjava.net/xylz/archive/2010/06/30/324915.html): 讲解的很详细

​