# 一、CountDownLatch

## 1.1 概念

CountDownLatch 类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

![](http://mycsdnblog.work/201919142037-K.png)

构造器中的**计数值（count）实际上就是闭锁需要等待的线程数量**。这个值只能被设置一次，而且CountDownLatch**没有提供任何机制去重新设置这个计数值**。

与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用**CountDownLatch.await()**方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他N 个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过 **CountDownLatch.countDown()**方法来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。

## 1.2 使用场景

1. **实现最大的并行性**：有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数为1的CountDownLatch，并让所有线程都在这个锁上等待，那么我们可以很轻松地完成测试。我们只需调用 一次countDown()方法就可以让所有的等待线程同时恢复执行。
2. **开始执行前等待n个线程完成各自任务**：例如应用程序启动类要确保在处理用户请求前，所有N个外部系统已经启动和运行了。
3. **死锁检测：**一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。

电商的详情页，由众多的数据拼装组成，如可以分成一下几个模块

- 交易的收发货地址，销量
- 商品的基本信息（标题，图文详情之类的）
- 推荐的商品列表
- 评价的内容
- ....

上面的几个模块信息，都是从不同的服务获取信息，且彼此没啥关联；所以为了提高响应，完全可以做成并发获取数据，如

- 线程1获取交易相关数据
- 线程2获取商品基本信息
- 线程3获取推荐的信息
- 线程4获取评价信息
- ....

**但是最终拼装数据并返回给前端，需要等到上面的所有信息都获取完毕之后，才能返回，这个场景就非常的适合 `CountDownLatch`来做了**

1. 在拼装完整数据的线程中调用 `CountDownLatch#await(long, TimeUnit)` 等待所有的模块信息返回
2. 每个模块信息的获取，由一个独立的线程执行；执行完毕之后调用 `CountDownLatch#countDown()` 进行计数-1

## 1.3 使用

```java
package com.juc.countdownlatch;

import java.util.concurrent.CountDownLatch;

/**
 * @Author: 98050
 * @Time: 2018-12-20 20:50
 * @Feature: CountDownLatch的使用
 */
public class Test {

    public static void main(String[] args) throws InterruptedException {
        final CountDownLatch countDownLatch = new CountDownLatch(2);

        new Thread(new Runnable() {
            public void run() {
                System.out.println(Thread.currentThread().getName()+",子线程开始执行");
                /**
                 * 计数器的值减一
                 */
                countDownLatch.countDown();
                System.out.println(Thread.currentThread().getName()+",子线程执行结束");
            }
        }).start();

        new Thread(new Runnable() {
            public void run() {
                System.out.println(Thread.currentThread().getName()+",子线程开始执行");
                /**
                 * 计数器的值减一
                 */
                countDownLatch.countDown();
                System.out.println(Thread.currentThread().getName()+",子线程执行结束");
            }
        }).start();

        /**
         * 计数器值为0，恢复任务继续执行
         */
        countDownLatch.await();
        System.out.println("两个子线程执行完毕，主线程继续执行");
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName()+","+i);
        }
    }
}
```

## 1.4 实现原理

同ReentrantLock一样，依然是借助AQS的双端队列，来实现原子的计数-1，线程阻塞和唤醒

### 1.4.1 计数器的初始化

CountDownLatch内部实现了AQS，并覆盖了`tryAcquireShared()`和`tryReleaseShared()`两个方法，下面说明干嘛用的

通过前面的使用，清楚了计数器的构造必须指定计数值,这个直接初始化了 AQS内部的state变量

![](http://mycsdnblog.work/201919142051-l.png)

后续的计数-1/判断是否可用都是基于sate进行的

### 1.4.2 countDown()计数-1的实现

```java
// 计数-1
public void countDown() {
    sync.releaseShared(1);
}


public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) { // 首先尝试释放锁
        doReleaseShared();
        return true;
    }
    return false;
}

protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();
        if (c == 0) //如果计数已经为0，则返回失败
            return false;
        int nextc = c-1;
        // 原子操作实现计数-1
        if (compareAndSetState(c, nextc)) 
            return nextc == 0;
    }
}

// 唤醒被阻塞的线程
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) { // 队列非空，表示有线程被阻塞
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) { 
            // 头结点如果为SIGNAL,则唤醒头结点下个节点上关联的线程，并出队
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head) // 没有线程被阻塞，直接跳出
            break;
    }
}
```

上面给出计数减1的完整调用链

1.尝试释放锁`tryReleaseShared`，实现计数-1

- 若计数已经小于0，则直接返回false
- 否则执行计数(AQS的state)减一
- 若减完之后，state==0，表示没有线程占用锁，即释放成功，然后就需要唤醒被阻塞的线程了

2.释放并唤醒阻塞线程 `doReleaseShared`

- 如果队列为空，即表示没有线程被阻塞（也就是说没有线程调用了 CountDownLatch.wait()方法），直接退出
- 头结点如果为SIGNAL, 则依次唤醒头结点下个节点上关联的线程，并出队

CountDownLatch计数为0之后，所有被阻塞的线程都会被唤醒，且彼此相对独立，不会出现独占锁阻塞的问题

### 1.4.3 await()阻塞等待计数为0

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
    

public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted()) // 若线程中端，直接抛异常
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}


// 计数为0时，表示获取锁成功
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}

// 阻塞，并入队
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.SHARED); // 入队
    boolean failed = true;
    try {
        for (;;) {
            // 获取前驱节点
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 获取锁成功，设置队列头为node节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) // 线程挂起
              && parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

阻塞的逻辑相对简单

1. 判断state计数是否为0，不是，则直接放过执行后面的代码
2. 大于0，则表示需要阻塞等待计数为0
3. 当前线程封装Node对象，进入阻塞队列
4. 然后就是循环尝试获取锁，直到成功（即state为0）后出队，继续执行线程后续代码

## 1.5 总结

### 1.5.1 注意

- 在创建实例时，必须指定初始的计数值，且应大于0
- 必须有线程中显示的调用了`countDown()`计数-1方法；必须有线程显示调用了`await()`方法（没有这个就没有必要使用CountDownLatch了）
- 由于await()方法会阻塞到计数为0，如果在代码逻辑中某个线程漏掉了计数-1，导致最终计数一直大于0，直接导致死锁了；
- 鉴于上面一点，更多的推荐 `await(long, TimeUnit)`来替代直接使用`await()`方法，至少不会造成阻塞死只能重启的情况
- 允许多个线程调用`await`方法，当计数为0后，所有被阻塞的线程都会被唤醒

# 二、CyclicBarrier

## 2.1 概念

**CyclicBarrier初始化时规定一个数目，然后计算调用了CyclicBarrier.await()进入等待的线程数。当线程数达到了这个数目时，所有进入等待状态的线程被唤醒并继续。** 

CyclicBarrier就象它名字的意思一样，可看成是个障碍， 所有的线程必须到齐后才能一起通过这个障碍。 

CyclicBarrier初始时还可带一个Runnable的参数， 此Runnable任务在CyclicBarrier的数目达到后，所有其它线程被唤醒前被执行。

## 2.2 使用场景

现在用户点击开始按钮后，需要匹配包括自己在内的五个玩家才能开始游戏，匹配玩家成功后进入到选择角色阶段。当5位玩家角色都选择完毕后，开始进入游戏。进入游戏时需要加载相关的数据，待全部玩家都加载完毕后正式开始游戏。

## 2.3 场景模拟

从需求中可以知道，想要开始游戏需要经过三个阶段，分别是

1. 匹配玩家
2. 选择角色
3. 加载数据

在这三个阶段中，都需要互相等待对方完成才能继续进入下个阶段。
这时可以采用`CyclicBarrier`来作为各个阶段的节点，等待其他玩家到达，在进入下个阶段。

### 2.3.1 StartGame

包含两个属性，player和cyclicBarrier，然后模拟游戏，在每个阶段完成后都要调用await方法，等待其余玩家

```java
package com.juc.cyclicbarrier;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

/**
 * @Author: 98050
 * @Time: 2019-03-14 21:38
 * @Feature:
 */
public class StartGame implements Runnable {

    private String player;

    private CyclicBarrier cyclicBarrier;

    public StartGame(String player, CyclicBarrier cyclicBarrier) {
        this.player = player;
        this.cyclicBarrier = cyclicBarrier;
    }

    public String getPlayer() {
        return player;
    }

    public void setPlayer(String player) {
        this.player = player;
    }

    public CyclicBarrier getCyclicBarrier() {
        return cyclicBarrier;
    }

    public void setCyclicBarrier(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }

    @Override
    public void run() {
        try{
            System.out.println(this.getPlayer() + "开始匹配玩家....");
            findOtherPlayer();
            cyclicBarrier.await();

            System.out.println(this.getPlayer() + "开始选择角色");
            chooseRole();
            System.out.println(this.getPlayer() + "角色选择完毕等待其它玩家");
            cyclicBarrier.await();

            System.out.println(this.getPlayer() + "游戏开始，游戏加载中");
            loading();
            System.out.println(this.getPlayer() + "游戏加载完毕，等待其它玩家加载完成");
            cyclicBarrier.await();

            start();

        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }

    private void start() {
        System.out.println("游戏开始");
    }

    private void loading() throws InterruptedException {
        Thread.sleep(500);
    }

    private void chooseRole() throws InterruptedException {
        Thread.sleep(500);
    }

    private void findOtherPlayer() throws InterruptedException {
        Thread.sleep(500);
    }


}
```

### 2.3.2 测试

CyclicBarrier有两个构造函数：

```java
public CyclicBarrier(int parties) {}
public CyclicBarrier(int parties, Runnable barrierAction) {}
```

#### 2.3.2.1 一个参数

```java
package com.juc.cyclicbarrier;

import java.util.concurrent.CyclicBarrier;

/**
 * @Author: 98050
 * @Time: 2019-03-14 21:47
 * @Feature:
 */
public class GameTest {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
        Thread palyer1 = new Thread(new StartGame("A", cyclicBarrier));
        Thread palyer2 = new Thread(new StartGame("B", cyclicBarrier));
        Thread palyer3 = new Thread(new StartGame("C", cyclicBarrier));
        Thread palyer4 = new Thread(new StartGame("D", cyclicBarrier));
        Thread palyer5 = new Thread(new StartGame("E", cyclicBarrier));

        palyer1.start();
        palyer2.start();
        palyer3.start();
        palyer4.start();
        palyer5.start();
    }
}
```

![](http://mycsdnblog.work/201919142156-E.png)

#### 2.3.2.2 两个参数

```java
package com.juc.cyclicbarrier;

import java.util.concurrent.CyclicBarrier;

/**
 * @Author: 98050
 * @Time: 2019-03-14 21:47
 * @Feature:
 */
public class GameTest {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
            System.out.println("阶段完成，等待2秒");
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("进入下个阶段");

        });
        Thread palyer1 = new Thread(new StartGame("A", cyclicBarrier));
        Thread palyer2 = new Thread(new StartGame("B", cyclicBarrier));
        Thread palyer3 = new Thread(new StartGame("C", cyclicBarrier));
        Thread palyer4 = new Thread(new StartGame("D", cyclicBarrier));
        Thread palyer5 = new Thread(new StartGame("E", cyclicBarrier));

        palyer1.start();
        palyer2.start();
        palyer3.start();
        palyer4.start();
        palyer5.start();
    }
}
```

![](http://mycsdnblog.work/201919142200-B.png)

可以看到在到达某个节点时，会执行实例化CyclicBarrier时传入的Runnable对象。而且每一次到达都会执行一次。

## 2.4 底层原理

CyclicBarrier是由ReentrantLock可重入锁和Condition共同实现的。

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```

```java
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;

        if (g.broken)
            throw new BrokenBarrierException();

        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        int index = --count;
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            if (g != generation)
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```

```java
/**
 * Updates state on barrier trip and wakes up everyone.
 * Called only while holding lock.
 */
private void nextGeneration() {
    // signal completion of last generation
    trip.signalAll();
    // set up next generation
    count = parties;
    generation = new Generation();
}
```

dowait(boolean, long)方法的主要逻辑处理比较简单，如果该线程不是最后一个调用await方法的线程，则它会一直处于等待状态，除非发生以下情况：

最后一个线程到达，即index == 0
某个参与线程等待超时
某个参与线程被中断

调用了CyclicBarrier的reset()方法。该方法会将屏障重置为初始状态

## 2.5 CyclicBarrier和CountDownLatch的区别

|                        CountDownLatch                        |                      CyclicBarrier                       |
| :----------------------------------------------------------: | :------------------------------------------------------: |
|                     计数为0时，无法重置                      |          计数达到0时，计数置为传入的值重新开始           |
| 调用countDown()方法计数减一，调用await()方法只进行阻塞，对计数没任何影响 | 调用await()方法计数减一，若减一后的值不等于0，则线程阻塞 |
|                         不可重复使用                         |                        可重复使用                        |

# 三、Semaphore

## 3.1 概念

Semaphore是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。它的用法如下：

availablePermits函数用来获取当前可用的资源数量

wc.acquire(); //申请资源

wc.release();// 释放资源

## 3.2 使用场景

Semaphore可以用于做流量控制，特别公用资源有限的应用场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发的读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制只有十个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。

## 3.3 使用

```java
package com.juc.semaphore;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreTest {
    private static final int THREAD_COUNT = 10;

    private static ExecutorService executorService = Executors.newFixedThreadPool(THREAD_COUNT);
    // 创建5个许可，允许5个并发执行
    private static Semaphore s = new Semaphore(5);

    public static void main(String[] args) {
        //创建10个线程执行任务
        for (int i = 0; i < THREAD_COUNT; i++) {
            executorService.execute(() -> {
                try {
                    //同时只能有5个线程并发执行保存数据的任务
                    s.acquire();
                    System.out.println("线程" + Thread.currentThread().getName() + " 保存数据");
                    Thread.sleep(2000);
                    //5个线程保存完数据，释放1个许可，其他的线程才能获取许可，继续执行保存数据的任务
                    s.release();
                    System.out.println("线程" + Thread.currentThread().getName() + " 释放许可");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        executorService.shutdown();
    }

}
```

![](http://mycsdnblog.work/201919142214-O.png)

## 3.4 实现原理

Semaphore实现主要基于java同步器AQS

### 3.4.1 构造函数

```java
//非公平的构造函数
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

//通过fair参数决定公平性
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

### 3.4.2 acquire(非公平策略)

```java
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

可以看出，如果remaining <0 即获取许可后，许可数小于0，则获取失败，在doAcquireSharedInterruptibly方法中线程会将自身阻塞，然后入列。

为什么不公平？

当一个线程A执行acquire方法时，会直接尝试获取许可，而不管同一时刻阻塞队列中是否有线程也在等待许可，如果恰好有线程C执行release释放许可，并唤醒阻塞队列中第一个等待的线程B，这个时候，线程A和线程B是共同竞争可用许可，不公平性就是这么体现出来的，线程A一点时间都没等待就和线程B同等对待。

公平策略

```java
protected int tryAcquireShared(int acquires) {
    for (;;) {
        if (hasQueuedPredecessors())
            return -1;
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

acquires值默认为1，表示尝试获取1个许可，remaining代表剩余的许可数。可以看到和非公平策略相比，就多了一个对阻塞队列的检查。

- 如果阻塞队列没有等待的线程，则参与许可的竞争。
- 否则直接插入到阻塞队列尾节点并挂起，等待被唤醒。

### 3.4.3 release

```java
public void release() {
    sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        int next = current + releases;
        if (next < current) // overflow
           throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
}
```

可以看出释放许可就是将AQS中state的值加1。然后通过doReleaseShared唤醒等待队列的第一个节点。可以看出Semaphore使用的是AQS的共享模式，等待队列中的第一个节点，如果第一个节点成功获取许可，又会唤醒下一个节点，以此类推。

# 四、Condition

## 4.1 概念

Condition的作用是对锁进行更精确的控制。Condition中的await()方法相当于Object的wait()方法，Condition中的signal()方法相当于Object的notify()方法，Condition中的signalAll()相当于Object的notifyAll()方法。

不同的是，Object中的wait(),notify(),notifyAll()方法是和"同步锁"(synchronized关键字)捆绑使用的；而Condition是需要与"互斥锁"/"共享锁"捆绑使用的。

## 4.2 使用

```java
package com.juc.condation;

/**
 * @Author: 98050
 * @Time: 2019-03-27 15:44
 * @Feature:
 */
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionTest {

    private static Lock lock = new ReentrantLock();
    private static Condition condition = lock.newCondition();

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");

        lock.lock(); // 获取锁
        try {
            System.out.println(Thread.currentThread().getName()+" start ta");
            ta.start();

            System.out.println(Thread.currentThread().getName()+" block");
            condition.await();    // 等待

            System.out.println(Thread.currentThread().getName()+" continue");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();    // 释放锁
        }
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        @Override
        public void run() {
            lock.lock();    // 获取锁
            try {
                System.out.println(Thread.currentThread().getName()+" wakup others");
                condition.signal();    // 唤醒“condition所在锁上的其它线程”
            } finally {
                lock.unlock();    // 释放锁
            }
        }
    }
}
```

Condition除了支持上面的功能之外，它更强大的地方在于：**能够更加精细的控制多线程的休眠与唤醒**。对于同一个锁，我们可以创建多个Condition，在不同的情况下使用不同的Condition。

例如，假如多线程读/写同一个缓冲区：当向缓冲区中写入数据之后，唤醒"读线程"；当从缓冲区读出数据之后，唤醒"写线程"；并且当缓冲区满的时候，"写线程"需要等待；当缓冲区为空时，"读线程"需要等待。         如果采用Object类中的wait(), notify(), notifyAll()实现该缓冲区，当向缓冲区写入数据之后需要唤醒"读线程"时，不可能通过notify()或notifyAll()明确的指定唤醒"读线程"，而只能通过notifyAll唤醒所有线程(但是notifyAll无法区分唤醒的线程是读线程，还是写线程)。  但是，通过Condition，就能明确的指定唤醒读线程。

### 4.2.1 实现阻塞队列

```java
package com.juc.condation;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2019-03-27 16:00
 * @Feature:
 */
public class MyBlockingQueue {
    final Lock lock = new ReentrantLock();
    final Condition notFull = lock.newCondition();
    final Condition notEmpty = lock.newCondition();

    final Object[] items = new Object[5];
    int putptr = 0,takeptr = 0,count = 0;

    public void put(Object x){
        //1.获取锁
        lock.lock();
        try{
            //2.如果缓存已满，则等待
            while (count == items.length){
                notFull.await();
            }
            //3.缓存不满，将x添加到缓冲区
            items[putptr] = x;
            //4.putptr加1，如果缓存已满，则设putptr为0
            if (++putptr == items.length){
                putptr = 0;
            }
            //5.缓存数量+1
            count++;
            //6.唤醒消费者线程，因为消费者线程通过notEmpty.await()等待
            notEmpty.signal();
            //7.打印写入的数据
            System.out.println(Thread.currentThread().getName() + " put：" + x);
        }catch (Exception e){
            e.getMessage();
        }finally {
            //8.释放锁
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        //1.获取锁
        lock.lock();
        try{
            //2.如何缓存区为空，则等待
            while (count == 0){
                notEmpty.await();
            }
            //3.从缓存区中取出一个元素（从队头）
            Object x = items[takeptr];
            //4.将takeptr加1，如果缓存区为空，则takeptr置为0
            if (++takeptr == items.length){
                takeptr = 0;
            }
            //5.缓存数量减1
            count--;
            //6.唤醒生产者线程，因为生产者线程是通过notFull.await()阻塞
            notFull.signal();
            //7.打印取出的数据
            System.out.println(Thread.currentThread().getName() + " get：" + x);
            return x;
        }finally {
            //8.释放锁
            lock.unlock();
        }
    }
}
```

### 4.2.2 测试

```java
package com.juc.condation;

/**
 * @Author: 98050
 * @Time: 2019-03-27 16:21
 * @Feature:
 */
public class MyBlockingQueueTest {

    private static MyBlockingQueue myBlockingQueue = new MyBlockingQueue();

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new PutThread("put" + i, i).start();
            new GetThread("get" + i).start();
        }
    }

    static class PutThread extends Thread{
        int num;
        PutThread(String name, int num){
            super(name);
            this.num = num;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(100);
                myBlockingQueue.put(num);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class GetThread extends Thread{
        GetThread(String name){
            super(name);
        }
        @Override
        public void run() {
            try {
                Thread.sleep(100);
                Integer num = (Integer) myBlockingQueue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

结果：

![](http://mycsdnblog.work/201919271638-7.png)

结果

(01) MyBlockingQueue是容量为5的缓冲，缓冲中存储的是Object对象，支持多线程的读/写缓冲。多个线程操作“一个MyBlockingQueue对象”时，它们通过互斥锁lock对缓冲区items进行互斥访问；而且同一个MyBlockingQueue对象下的全部线程共用“notFull”和“notEmpty”这两个Condition。

(02)notFull用于控制写缓冲，notEmpty用于控制读缓冲。当缓冲已满的时候，调用put的线程会执行notFull.await()进行等待；当缓冲区不是满的状态时，就将对象添加到缓冲区并将缓冲区的容量count+1，最后，调用notEmpty.signal()缓冲notEmpty上的等待线程(调用notEmpty.await的线程)。 简言之，notFull控制“缓冲区的写入”，当往缓冲区写入数据之后会唤醒notEmpty上的等待线程。同理，notEmpty控制“缓冲区的读取”，当读取了缓冲区数据之后会唤醒notFull上的等待线程。

(03) 在测试类中，启动10个“写线程”，向MyBlockingQueue中不断的写数据(写入0-9)；同时，也启动10个“读线程”，从MyBlockingQueue中不断的读数据。

## 4.3 底层实现

**await()源码**

当我们往队列里插入一个元素时，如果队列不可用，阻塞生产者主要通过LockSupport.park(this);来实现

```java
public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            Node node = addConditionWaiter();
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)

reportInterruptAfterWait(interruptMode);
        }
```

继续进入源码，发现调用setBlocker先保存下将要阻塞的线程，然后调用unsafe.park阻塞当前线程。

```java
public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        unsafe.park(false, 0L);
        setBlocker(t, null);
    }
```

unsafe.park是个native方法，代码如下：

```java
public native void park(boolean isAbsolute, long time);
```

park这个方法会阻塞当前线程，只有以下四种情况中的一种发生时，该方法才会返回。

- 与park对应的unpark执行或已经执行时。注意：已经执行是指unpark先执行，然后再执行的park。
- 线程被中断时。
- 如果参数中的time不是零，等待了指定的毫秒数时。
- 发生异常现象时。这些异常事先无法确定。

我们继续看一下JVM是如何实现park方法的，park在不同的操作系统使用不同的方式实现，在linux下是使用的是系统方法pthread_cond_wait实现。实现代码在JVM源码路径src/os/linux/vm/os_linux.cpp里的 os::PlatformEvent::park方法，代码如下：

```c
void os::PlatformEvent::park() {
     	     int v ;
	     for (;;) {
		v = _Event ;
	     if (Atomic::cmpxchg (v-1, &_Event, v) == v) break ;
	     }
	     guarantee (v >= 0, "invariant") ;
	     if (v == 0) {
	     // Do this the hard way by blocking ...
	     int status = pthread_mutex_lock(_mutex);
	     assert_status(status == 0, status, "mutex_lock");
	     guarantee (_nParked == 0, "invariant") ;
	     ++ _nParked ;
	     while (_Event < 0) {
	     status = pthread_cond_wait(_cond, _mutex);
	     // for some reason, under 2.7 lwp_cond_wait() may return ETIME ...
	     // Treat this the same as if the wait was interrupted
	     if (status == ETIME) { status = EINTR; }
	     assert_status(status == 0 || status == EINTR, status, "cond_wait");
	     }
	     -- _nParked ;

	     // In theory we could move the ST of 0 into _Event past the unlock(),
	     // but then we'd need a MEMBAR after the ST.
	     _Event = 0 ;
	     status = pthread_mutex_unlock(_mutex);
	     assert_status(status == 0, status, "mutex_unlock");
	     }
	     guarantee (_Event >= 0, "invariant") ;
	     }

     }
```

pthread_cond_wait是一个多线程的条件变量函数，cond是condition的缩写，字面意思可以理解为线程在等待一个条件发生，这个条件是一个全局变量。这个方法接收两个参数，一个共享变量_cond，一个互斥量_mutex。而unpark方法在linux下是使用pthread_cond_signal实现的。park 在windows下则是使用WaitForSingleObject实现的。