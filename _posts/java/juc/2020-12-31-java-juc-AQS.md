---
layout: post
title:  "java多线程--AQS"
date:   2020-12-31
categories: java
tag: juc
---

* content
{:toc}
# AQS

- AQS 是一个用来构建锁和同步器的框架

# AQS的原理

- 如果请求的资源空闲，当前请求资源的线程就可以获取资源，并且给资源加上锁。如果当前请求的资源被暂用，就需要一套阻塞和唤醒机制，AQS使用CLH队列实现，将目前获取不到资源的锁的现场，加入到队列中
- CLH 队列
  - AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个节点 （Node） 来实现锁的分配。
  - 虚拟的双向队列，并没有队列实例，仅存在节点间的指向关系。

- AQS 使用一个volatile int state 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。 AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

#  AQS 资源共享方式

-  Exclusive (独占)：：只有一个线程能执行， 如 ReentantLock
- Share（共享）：多个线程可同时执行， 如 Semaphore/CountDownLatch

# 公平锁和非公平锁

在 AQS 独占式资源共享方式中

- 公平锁：按照线程在队列中的排队顺序，先到先拿到锁。
- 非公平锁：当线程要获取锁时，无视队列的顺序直接去抢锁（CAS），谁先抢到就是谁的

# ReentrantLock

- 以 ReentrantLock 为例， state 初始化为 0 ， 表示未锁定状态。 A 线程 lock() 时，会调用 tryAcquire() 独占该锁并将 share +1 。此后，其他现场再 tryAcquire() 时就会失败， 直到 A 线程 unlock() 到 state = 0 （即释放锁）为止， 其他线程才有机会获取该锁。当然，锁释放之前， A 线程自己是可以重复获取此锁的 （state 会累加），这就是可重入的概念。但是要注意，获取多少次就要释放多少次，这样才能保证 state 是能回到零态的

# CountDownLatch

- 再以 CountDownLatch 为例，任务分配为 N 个线程去执行， state 也初始化为 N（注意 N 要与线程个数一致）。 该 N 个子线程是并行执行的， 每个子线程执行完后 countDown() 一次， state 会 CAS 减1。等到所有子线程都执行完后 （即 state =0）, 会 unpark() 主调用线程，然后主调用线程就会从 await() 函数返回，继续后余动作。
- （倒计时器）： CountDownLatch 是一个同步工具类， 用来协调多个线程之间同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，在开始执行。

# Semaphore

- Semaphore（信号量）-允许多个线程同时访问： synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源， Semaphore （信号量）可以指定多个线程同时访问某个资源。

```java
	Semaphore semaphore = new Semaphore(3);

	ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

    for (int i=0;i<10;i++){
        final long num = i;
        cachedThreadPool.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    //请求获得许可，如果有可获得的许可则继续往下执行，许可数减1。否则进入阻塞状态
                    semaphore.acquire();
                    //执行
                    System.out.println("Accessing: " + num);
                    Thread.sleep(new Random().nextInt(5000)); // 模拟随机执行时长
                    //释放
                    semaphore.release();
                    System.out.println("Release..." + num);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    cachedThreadPool.shutdown();
```

# CyclicBarrier(循环栅栏)

- 可以实现线程间的技术等待
- CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier 默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用 await()方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。