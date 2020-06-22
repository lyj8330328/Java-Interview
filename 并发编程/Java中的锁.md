# 一、悲观锁

总是假设最坏的情况，每次取数据时都认为其他线程会修改，所以都会加锁（读锁、写锁、行锁等），当其他线程想要访问数据时，都需要阻塞挂起。可以依靠数据库实现，如行锁、读锁和写锁等，都是在操作之前加锁，在Java中，synchronized的思想也是悲观锁。

# 二、乐观锁

总是认为不会产生并发问题，每次去取数据的时候总认为不会有其他线程对数据进行修改，因此不会上锁，但是在更新时会判断其他线程在这之前有没有对数据进行修改，**一般会使用版本号机制或CAS操作实现**。

version方式：一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

核心SQL语句

update table set x=x+1, version=version+1 where id=#{id} and version=#{version};    

CAS操作方式：即compare and swap 或者 compare and set，涉及到三个操作数，数据所在的内存值，预期值，新值。当需要更新时，判断当前内存值与之前取到的值是否相等，若相等，则用新值更新，若失败则重试，一般情况下是一个自旋操作，即不断的重试。

# 三、重入锁

ReentrantLock(轻量级)等等 ) 。这些已经写好提供的锁为我们开发提供了便利。

重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。
在JAVA环境下 ReentrantLock 和synchronized 都是 可重入锁

ReentrantLock加锁和释放锁的底层原理

![](http://mycsdnblog.work/201919271538-x.png)

好了，那么现在如果有一个线程过来尝试用ReentrantLock的lock()方法进行加锁，会发生什么事情呢？

很简单，这个AQS对象内部有一个核心的变量叫做**state**，是int类型的，代表了**加锁的状态**。初始状态下，这个state的值是0。

另外，这个AQS内部还有一个**关键变量**，用来记录**当前加锁的是哪个线程**，初始化状态下，这个变量是null。

![](http://mycsdnblog.work/201919111651-v.png)

接着线程1跑过来调用ReentrantLock的lock()方法尝试进行加锁，这个加锁的过程，直接就是用CAS操作将state值从0变为1。

如果之前没人加过锁，那么state的值肯定是0，此时线程1就可以加锁成功。

一旦线程1加锁成功了之后，就可以设置当前加锁线程是自己。所以大家看下面的图，就是线程1跑过来加锁的一个过程。

![](http://mycsdnblog.work/201919111652-9.png)

其实每次线程1可重入加锁一次，会判断一下当前加锁线程就是自己，那么他自己就可以可重入多次加锁，每次加锁就是把state的值给累加1，别的没啥变化。

接着，如果线程1加锁了之后，线程2跑过来加锁会怎么样呢？

**我们来看看锁的互斥是如何实现的？**线程2跑过来一下看到，哎呀！state的值不是0啊？所以CAS操作将state从0变为1的过程会失败，因为state的值当前为1，说明已经有人加锁了！

接着线程2会看一下，是不是自己之前加的锁啊？当然不是了，**“加锁线程”**这个变量明确记录了是线程1占用了这个锁，所以线程2此时就是加锁失败。

给大家来一张图，一起来感受一下这个过程：

![](http://mycsdnblog.work/201919111653-K.png)

接着，线程2会将自己放入AQS中的一个等待队列，因为自己尝试加锁失败了，此时就要将自己放入队列中来等待，等待线程1释放锁之后，自己就可以重新尝试加锁了

所以大家可以看到，AQS是如此的核心！AQS内部还有一个等待队列，专门放那些加锁失败的线程！

同样，给大家来一张图，一起感受一下：

![](http://mycsdnblog.work/201919111654-t.png)

接着，线程1在执行完自己的业务逻辑代码之后，就会释放锁！**他释放锁的过程非常的简单**，就是将AQS内的state变量的值递减1，如果state值为0，则彻底释放锁，会将“加锁线程”变量也设置为null！

![](http://mycsdnblog.work/201919111655-Y.png)

接下来，会从**等待队列的队头唤醒线程2重新尝试加锁。**

好！线程2现在就重新尝试加锁，这时还是用CAS操作将state从0变为1，此时就会成功，成功之后代表加锁成功，就会将state设置为1。

此外，还要把**“加锁线程”**设置为线程2自己，同时线程2自己就从等待队列中出队了。

最后再来一张图，大家来看看这个过程。

![](http://mycsdnblog.work/201919111656-5.png)

# 四、读写锁

## 4.1 原理

ReentrantReadWriteLock的实现，主要包括：读写状态的设计、写锁的获取与释放、读锁的获取与释放以及锁降级（以下没有特别说明读写锁均可认为是ReentrantReadWriteLock）

### 4.1.1 读写状态的设计

如果在一个整型变量上维护多种状态，就一定需要“按位切割使用”这个变量，读写锁将变量切分成了两个部分，高16位表示读，低16位表示写，划分方式如下图所示：

![](http://mycsdnblog.work/201919051430-B.png)

**假设当前同步状态值为S，写状态等于S&0x0000FFFF（将高16位全部抹去），读状态等于S>>>16（无符号补0右移16位）。当写状态增加1时，等于S+1，当读状态增加1时，等于S+(1<<16)，也就是S+0x00010000。**

根据状态的划分能得出一个推论：S不等于0时，当写状态（S&0x0000FFFF）等于0时，则读状态（S>>>16）大于0，**即读锁已被获取**。

### 4.1.2 写锁的获取与释放

**写锁是一个支持重进入的排它锁**。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程，则当前线程进入等待状态，获取写锁的代码如下7所示。

```java
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState(); //获取当前状态值S
    int w = exclusiveCount(c); //获取写状态
    if (c != 0) {
        // S不为0，w为0，则说明有读锁或者当前线程已经不是获取写锁的线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

如果存在读锁，则写锁不能被获取，原因在于：读写锁要确保写锁的操作对读锁可见，如果允许读锁在已被获取的情况下对写锁的获取，那么正在运行的其他读线程就无法感知到当前写线程的操作。因此，只有等待其他读线程都释放了读锁，写锁才能被当前线程获取，而写锁一旦被获取，则其他读写线程的后续访问均被阻塞。

写锁的释放与ReentrantLock的释放过程基本类似，每次释放均减少写状态，当写状态为0时表示写锁已被释放，从而等待的读写线程能够继续访问读写锁，同时前次写线程的修改对后续读写线程可见。

### 4.1.3 读锁的获取与释放

**读锁是一个支持重进入的共享锁**，它能够被多个线程同时获取，在没有其他写线程访问（或者写状态为0）时，读锁总会被成功地获取，**而所做的也只是（线程安全的）增加读状态**。如果当前线程已经获取了读锁，则增加读状态。如果当前线程在获取读锁时，写锁已被其他线程获取，则进入等待状态。

读状态是所有线程获取读锁次数的总和，而每个线程各自获取读锁的次数只能选择保存在ThreadLocal中，由
线程自身维护，这使获取读锁的实现变得复杂。

```java
protected final int tryAcquireShared(int unused) {

    Thread current = Thread.currentThread();
    int c = getState();
    //如果写锁状态不为0且当前线程不是已经获取读锁的线程，那么就进入等待状态
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    //获取读状态
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        //第一个读线程
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            //还是第一个线程重复获取锁
            firstReaderHoldCount++;
        } else {
            //读状态累加
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

读锁的每次释放（线程安全的，可能有多个读线程同时释放读锁）均减少读状态，减少的值是（1<<16）。

### 4.1.4 锁降级

锁降级指的是写锁降级成为读锁。如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指把持住（当前拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁的过程。

```java
package com.example.readandwrite;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @Author: 98050
 * @Time: 2019-06-05 14:40
 * @Feature: 锁降级
 */
public class Test {

    static final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    static Lock read = lock.readLock();
    static Lock write = lock.writeLock();
    static volatile boolean update = false;
    static int count = 0;

    static class MyThread implements Runnable{

        @Override
        public void run() {
            processData();
        }

        public void processData(){
            read.lock();
            if (!update){
                //1.必须先释放读锁
                read.unlock();
                //2.锁降级从写锁获取到开始
                write.lock();
                try{
                    if (!update){
                        //数据更新
                        count ++;
                        //Thread.sleep(2000);
                        update = true;
                    }
                    read.lock();
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    write.unlock();
                }
                //锁降级完成
            }
            try {
                //Thread.sleep(2000);
                //数据使用
                System.out.println(count);
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                read.unlock();
            }
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(new MyThread()).start();
        }
    }
}
```

锁降级中读锁的获取是否必要呢？答案是必要的。主要是为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程（记作线程T）获取了写锁并修改了数据，那么当前线程无法感知线程T的数据更新。如果当前线程获取读锁，即遵循锁降级的步骤，则线程T将会被阻塞，直到当前线程使用数据并释放读锁之后，线程T才能获取写锁进行数据更新。

## 4.2 使用

​	static ReentrantReadWriteLock *rwl* = new ReentrantReadWriteLock();

​	static Lock *r* = *rwl*.readLock();

​	static Lock *w* = *rwl*.writeLock();

```java
package com.thread.readwritelock;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @Author: 98050
 * @Time: 2019-04-29 20:53
 * @Feature:
 */
public class Cache {
    static Map<String,Object> map = new HashMap<String, Object>();
    static ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    static Lock r = rwl.readLock();
    static Lock w = rwl.writeLock();

    public static final Object get(String key){
        r.lock();
        try{
            System.out.println("正在读操作，key：" + key + "开始");
            Thread.sleep(100);
            Object o = map.get(key);
            System.out.println("读操作，key：" + key + "结束");
            return o;
        }catch (Exception e){

        }finally {
            r.unlock();
        }
        return null;
    }

    public static final Object put(String key,Object value){
        w.lock();
        try {
            System.out.println("正在做写操作，key：" + key + ",value" + value + "开始");
            Thread.sleep(100);
            Object o = map.put(key, value);
            System.out.println("写操作，key：" + key + ",value" + value + "结束");
            return o;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            w.unlock();
        }
        return null;
    }

    public static void main(String[] args) {
        new Thread(new Runnable(){

            public void run() {
                for (int i = 0; i < 10; i++) {
                    Cache.put(i+"", i);
                }
            }
        }).start();

        new Thread(new Runnable(){

            public void run() {
                for (int i = 0; i < 10; i++) {
                    Cache.get(i+"");
                }
            }
        }).start();
    }
}
```

运行结果：

```java
正在做写操作，key：0,value0开始
写操作，key：0,value0结束
正在读操作，key：0开始
读操作，key：0结束
正在做写操作，key：1,value1开始
写操作，key：1,value1结束
正在读操作，key：1开始
读操作，key：1结束
正在做写操作，key：2,value2开始
写操作，key：2,value2结束
正在做写操作，key：3,value3开始
写操作，key：3,value3结束
正在做写操作，key：4,value4开始
写操作，key：4,value4结束
正在做写操作，key：5,value5开始
写操作，key：5,value5结束
正在做写操作，key：6,value6开始
写操作，key：6,value6结束
正在做写操作，key：7,value7开始
写操作，key：7,value7结束
正在做写操作，key：8,value8开始
写操作，key：8,value8结束
正在做写操作，key：9,value9开始
写操作，key：9,value9结束
正在读操作，key：2开始
读操作，key：2结束
正在读操作，key：3开始
读操作，key：3结束
正在读操作，key：4开始
读操作，key：4结束
正在读操作，key：5开始
读操作，key：5结束
正在读操作，key：6开始
读操作，key：6结束
正在读操作，key：7开始
读操作，key：7结束
正在读操作，key：8开始
读操作，key：8结束
正在读操作，key：9开始
读操作，key：9结束
```

# 六、自旋锁

## 6.1 自旋锁

所谓自旋，就是指当有另外一个线程来竞争锁时，这个线程会在原地循环等待，**而不是把该线程给阻塞**，直到那个获得锁的线程释放锁之后，这个线程就可以马上获得锁的。**锁在原地循环的时候，是会消耗cpu的，就相当于在执行一个啥也没有的for循环。**

**本来一个线程把锁释放之后，当前线程是能够获得锁的，但是假如这个时候有好几个线程都在竞争这个锁的话，那么有可能当前线程会获取不到锁，还得原地等待继续空循环消耗cup，甚至有可能一直获取不到锁。**

**基于这个问题，我们必须给线程空循环设置一个次数，当线程超过了这个次数，我们就认为，继续使用自旋锁就不适合了，此时锁会再次膨胀，升级为重量级锁。**

所以自旋等待的时间必须有一定的限度，超过了限定的次数仍然没有成功获取锁，就应当使用传统的方式挂起线程了。**自旋次数的默认值是10**，用户可以通过-XX:PreBlockSpin来更改。

## 6.2 自适应自旋

所谓自适应自旋锁就是线程空循环等待的**自旋次数并非是固定的**，而是会动态着根据实际情况来改变自旋等待的次数

自适应自旋锁的自适应反映在自旋的时间不在固定了。如果在同一个锁对象上，自旋线程之前刚刚获得过锁，且现在持有锁的线程正在运行中，那么虚拟机会认为这次自旋也很有可能会成功，进而允许该线程等待持续相对更长的时间，比如100个循环。反之，如果某个锁自旋很少获得过成功，那么之后再获取锁的时候将可能省略掉自旋过程，以避免浪费处理器资源。

## 6.3 保证自旋锁的公平性

第一种方法就是按先来后到的顺序为每个线程编个号，按编号顺序来分配锁。这就类似于银行挂号或者医院挂号一样，按照先来后到的顺序为每个问诊者排个号，医生按号依次为问诊者服务。

第二种方法就是**检测前驱结点的locked状态**：如果为`false`说明该线程是队列中的第一个线程；如果为`true`说明为前面已经有线程获取了锁，当前线程需要自旋阻塞。

第三种方法就是MSC与CLH最大的不同并不是链表是显示还是隐式，而是线程自旋的规则不同:**CLH是在前趋结点的locked域上自旋等待，而MCS是在自己的**结点的locked域上自旋等待。正因为如此，它解决了CLH在NUMA系统架构中获取locked域状态内存过远的问题。

# 七、CAS无锁机制

它包含三个参数CAS(V,E,N): V表示要更新的变量，E表示预期值，N表示新值。仅当V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后，CAS返回当前V的真实值。

如何解决ABA的问题：版本号

**JVM中的CAS操作利用了处理器提供的CMPXCHG指令实现。**

## 7.1 CAS操作

自旋CAS实现的基本思路就是循环进行CAS操作直到成功为止。

```java
package com.example.cas;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Author: 98050
 * @Time: 2019-05-23 17:32
 * @Feature: CAS计数器
 */
public class Counter {
    private AtomicInteger atomicInteger = new AtomicInteger(0);
    private int i = 0;

    /**
     * 线程安全的计数器
     */
    private void safeCount(){
        for (;;){
            int temp = atomicInteger.get();
            boolean success = atomicInteger.compareAndSet(temp, ++temp);
            if (success){
                break;
            }
        }
    }

    /**
     * 非线程安全的计数器
     */
    private void count(){
        i++;
    }

    public static void main(String[] args) throws InterruptedException {
        final Counter counter = new Counter();
        List<Thread> list = new ArrayList<>();
        CountDownLatch countDownLatch = new CountDownLatch(100);
        for (int i = 0; i < 100; i++) {
            list.add(new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    counter.count();
                    counter.safeCount();
                }
                countDownLatch.countDown();
            }));
        }
        for (Thread thread : list){
            thread.start();
        }
        countDownLatch.await();
        System.out.println("count:" + counter.i);
        System.out.println("safeCount:" + counter.atomicInteger.get());
    }
}
```

![](http://mycsdnblog.work/201919231749-w.png)

## 7.2 CAS的缺点

1、ABA的问题

2、高竞争下的开销问题

3、功能限制

## 7.3 CAS操作多个变量

```java
package com.example.cas;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

/**
 * @Author: 98050
 * @Time: 2019-05-23 19:08
 * @Feature:
 */
public class MCas {


    T t = new T(0,0,0);

    AtomicReference<T> reference = new AtomicReference<>(new T(0,0,0));

    /**
     * 线程安全的计数器
     */
    private void safeCount(){
        for (;;){
            T temp = reference.get();
            T t1 = new T(temp.i + 1,temp.j + 1,temp.k + 1);
            boolean success = reference.compareAndSet(temp, t1);
            if (success){
                break;
            }
        }
    }

    /**
     * 非线程安全的计数器
     */
    private void count(){
        T t1 = new T(t.i + 1,t.j + 1,t.k + 1);
        t = t1;
    }

    public static void main(String[] args) throws InterruptedException {
        final MCas cas = new MCas();
        List<Thread> list = new ArrayList<>();
        CountDownLatch countDownLatch = new CountDownLatch(100);
        for (int i = 0; i < 100; i++) {
            list.add(new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    cas.count();
                    cas.safeCount();
                }
                countDownLatch.countDown();
            }));
        }
        for (Thread thread : list){
            thread.start();
        }
        countDownLatch.await();
        System.out.println(cas.reference.get());
        System.out.println(cas.t);
    }
}
class T{
    public int i;
    public int j;
    public int k;


    public T(int i, int j, int k) {
        this.i = i;
        this.j = j;
        this.k = k;
    }

    @Override
    public String toString() {
        return "T{" +
                "i=" + i +
                ", j=" + j +
                ", k=" + k +
                '}';
    }
}
```

![](http://mycsdnblog.work/201919231944-r.png)

# 八、AQS

## 8.1 队列同步器的接口和示例

### 8.1.1 访问或修改同步状态

- getState() 获取当前状态
- setState(int newState) 设置当前同步状态
- compareAndSetState(int expect，int update) 使用CAS设置当前状态

### 8.1.2 同步器可重写的方法

![](http://mycsdnblog.work/201919031550-q.png)

### 8.1.3 同步器提供的模板方法

![](http://mycsdnblog.work/201919031550-1.png)

## 8.2 队列同步器的实现分析

主要包括：同步队列、独占式同步状态获取与释放、共享式同步状态获取与释放以及超时获取同步状态等同步器的核心数据结构与模板方法。

### 8.2.1 同步队列

**同步器依赖内部的同步队列（一个FIFO双向队列）来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成为一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。**

同步队列中的节点（Node）:

![](http://mycsdnblog.work/201919031601-r.png)

**当一个线程成功地获取了同步状态（或者锁），其他线程将无法获取到同步状态，转而被构造成为节点并加入到同步队列中，而这个加入队列的过程必须要保证线程安全，因此同步器提供了一个基于CAS的设置尾节点的方法：`compareAndSetTail(Node expect,Nodeupdate)`，它需要传递当前线程“认为”的尾节点和当前节点，只有设置成功后，当前节点才正式与之前的尾节点建立关联。**

**同步队列遵循FIFO，首节点是获取同步状态成功的节点，首节点的线程在释放同步状时，将会唤醒后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点。**

**设置首节点是通过获取同步状态成功的线程来完成的，由于只有一个线程能够成功获取到同步状态，因此设置头节点的方法并不需要使用CAS来保证，它只需要将首节点设置成为原首节点的后继节点并断开原首节点的next引用即可。**

### 8.2.2 独占式同步状态获取与释放

通过调用同步器的acquire(int arg)方法可以获取同步状态，该方法对中断不敏感，也就是由于线程获取同步状态失败后进入同步队列中，后续对线程进行中断操作时，线程不会从同步队列中移出。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

首先调用自定义同步器实现的`tryAcquire(int arg)`方法，该方法保证线程安全的获取同步状态，如果同步状态获取失败，则构造同步节点（独占式Node.EXCLUSIVE，同一时刻只能有一个线程成功获取同步状态）并通过addWaiter(Node node)方法将该节点加入到同步队列的尾部，最后调用acquireQueued(Node node,int arg)方法，使得该节点以“死循环”的方式获取同步状态。

节点进入同步队列之后，就进入了一个自旋的过程，每个节点（或者说每个线程）都在自省地观察，当条件满足，获取到了同步状态，就可以从这个自旋过程中退出，否则依旧留在这个自旋过程中（并会阻塞节点的线程）。

通过调用同步器的release(int arg)方法可以释放同步状态，该方法在释放了同步状态之后，会唤醒其后继节点（进而使后继节点重新尝试获取同步状态）。

![](http://mycsdnblog.work/201919031753-2.png)

总结：在获取同步状态时，同步器维护一个同步队列，获取状态失败的线程都会被加入到队列中并在队列中进行自旋；移出队列（或停止自旋）的条件是前驱节点为头节点且成功获取了同步状态。在释放同步状态时，同步器调用tryRelease(int arg)方法释放同步状态，然后唤醒头节点的后继节点。

### 8.2.3 共享式同步状态获取与释放

共享模式的逻辑：

- 线程调用`acquireShared`方法获取锁
- 如果失败则创建共享类型的节点放入FIFO队列，等待唤醒
- 有线程释放锁后唤醒队列最前端的节点，然后唤醒所有后面的共享节点

通过调用同步器的acquireShared(int arg)方法可以共享式地获取同步状态

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    //唤醒后面的节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

在acquireShared(int arg)方法中，同步器调用tryAcquireShared(int arg)方法尝试获取同步状态，tryAcquireShared(int arg)方法返回值为int类型，当返回值大于等于0时，表示能够获取到同步状态。因此，在共享式获取的自旋过程中，成功获取到同步状态并退出自旋的条件就是tryAcquireShared(int arg)方法返回值大于等于0。可以看到，在doAcquireShared(int arg)方法的自旋过程中，如果当前节点的前驱为头节点时，尝试获取同步状态，如果返回值大于等于0，表示该次获取同步状态成功并从自旋过程中退出。

**共享模式与独占模式最大的不同就是，同一时刻能否有多个线程同时获取到同步状态，而且共享模式唤醒第一个节点后会迭代唤醒后面所有的共享节点。**

### 8.2.4 独占式超时获取同步状态

通过调用同步器的doAcquireNanos(int arg,long nanosTimeout)方法可以超时获取同步状态，即在指定的时间段内获取同步状态，如果获取到同步状态则返回true，否则，返回false。

doAcquireNanos(int arg,long nanosTimeout)方法在支持响应中断的基础上，增加了超时获取的特性。针对超时获取，主要需要计算出需要睡眠的时间间隔nanosTimeout，如果nanosTimeout大于0则表示超时时间未到，需要继续睡眠nanosTimeout纳秒，反之，表示已经超时：

```java
private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

![](http://mycsdnblog.work/201919041607-A.png)

## 8.3 自定义同步组件

```java
package com.example.aqs;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

/**
 * @Author: 98050
 * @Time: 2019-06-04 16:25
 * @Feature:
 */
public class TwinsLock implements Lock {

    private final Sync sync = new Sync(2);



    private static final class Sync extends AbstractQueuedSynchronizer{

        public Sync(int count) {
            if (count <= 0){
                throw new IllegalArgumentException("count must large than zero");
            }
            setState(count);
        }

        @Override
        protected int tryAcquireShared(int arg) {
            for (;;){
                int now = getState();
                int result = now - arg;
                if (result < 0 || compareAndSetState(now, result)){
                    return result;
                }
            }
        }

        @Override
        protected boolean tryReleaseShared(int arg) {
            for (;;){
                int now = getState();
                int result = now + arg;
                if (compareAndSetState(now, result)){
                    return true;
                }
            }
        }

        Condition newCondition(){
            return new ConditionObject();
        }
    }

    @Override
    public void lock() {
        sync.acquireShared(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquireShared(1) == 0;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.releaseShared(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }

}
```

测试类：

```java
package com.example.aqs;

import java.util.concurrent.locks.Lock;

/**
 * @Author: 98050
 * @Time: 2019-06-04 16:55
 * @Feature:
 */
public class TwinsLockTest {

    static Lock lock = new TwinsLock();

    static class MyThread implements Runnable{

        @Override
        public void run() {
            lock.lock();
            try{
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName());
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            Thread t1 = new Thread(new MyThread());
            t1.setDaemon(true);
            t1.start();
        }
        for (int i = 0; i < 10; i++) {
            Thread.sleep(1000);
            System.out.println();
        }
    }
}
```

结果：

![](http://mycsdnblog.work/201919041710-e.png)

AbstractQueuedSynchronizer ： 抽象队列同步器

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。

AQS对象内部有一个核心变量叫做state，是int类型，代表了加锁的状态，初始值为0。

另一个核心变量就是用来记录当前加锁的是哪个线程，初始值为null



ReentrantLock和AQS之间的关系。

![](http://mycsdnblog.work/201919111651-j.png)

## 8.4 总结

**AQS就是一个并发包的基础组件，用来实现各种锁，各种同步组件的。它包含了state变量、加锁线程、等待队列等并发中的核心组件。**

# 九、synchronized底层实现原理

## 9.1 底层原理

**同步代码块**

synchronized的对象锁，其指针指向的是一个monitor对象（由C++实现）的起始地址。每个对象实例都会有一个 monitor。其中monitor可以与对象一起创建、销毁；亦或者当线程试图获取对象锁时自动生成。

每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

3、如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

![](http://mycsdnblog.work/201919151359-R.png)

**同步方法**

从反编译的结果来看，方法的同步并没有通过指令monitorenter和monitorexit来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了**ACC_SYNCHRONIZED**标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。

![](http://mycsdnblog.work/201919151402-A.png)

------

```java
package com.example.communication;

/**
 * @Author: 98050
 * @Time: 2019-05-30 15:22
 * @Feature:
 */
public class Synchronized {

    public static void main(String[] args) {
        synchronized (Synchronized.class){

        }
        m();
    }

    public static synchronized void m(){

    }
}
```

在Synchronized.class同级目录执行`javap -v Synchronized.class`：

![](http://mycsdnblog.work/201919301535-V.png)

![](http://mycsdnblog.work/201919301535-8.png)

## 9.2 synchronized方法

```java 
package com.thread.lock;

import java.util.ArrayList;

/**
 * @Author: 98050
 * @Time: 2019-04-29 19:56
 * @Feature:
 */
public class Test006 {

    public static void main(String[] args) {
        Common common = new Common();
        new Thread(() -> {
            common.insert(Thread.currentThread());
        }).start();

        new Thread(() -> {
            common.insert(Thread.currentThread());
        }).start();
    }

}

class Common{
    private ArrayList<Integer> list = new ArrayList<>();

    public synchronized void insert(Thread thread){
        for (int i = 0; i < 5; i++) {
            System.out.println(thread.getName() + "在插入数据：" + i);
            list.add(i);
        }
    }
}
```

运行结果：

![](http://mycsdnblog.work/201919292030-Z.png)

当一个线程正在访问一个对象的synchronized方法，那么其他线程不能访问该对象的其他synchronized方法。这个原因很简单，因为一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所以无法访问该对象的其他synchronized方法。

2）当一个线程正在访问一个对象的synchronized方法，那么其他线程能访问该对象的非synchronized方法。这个原因很简单，访问非synchronized方法不需要获得该对象的锁，假如一个方法没用synchronized关键字修饰，说明它不会使用到临界资源，那么其他线程是可以访问这个方法的，

```java
package com.thread.lock;

import java.util.ArrayList;

/**
 * @Author: 98050
 * @Time: 2019-04-29 19:56
 * @Feature:
 */
public class Test006 {

    public static void main(String[] args) {
        Common common = new Common();
        new Thread(() -> {
            common.insert(Thread.currentThread());
        }).start();

        new Thread(() -> {
            //common.insert(Thread.currentThread());
            common.insert2();
        }).start();
    }

}

class Common{
    private ArrayList<Integer> list = new ArrayList<>();

    public synchronized void insert(Thread thread){
        for (int i = 0; i < 5; i++) {
            System.out.println(thread.getName() + "在插入数据：" + i);
            list.add(i);
        }
    }

    public void insert2(){
        for (int i = 0; i < 5; i++) {
            System.out.println("插入数据：" + i);
            list.add(i);
        }
    }
}
```

运行结果：

![](http://mycsdnblog.work/201919292034-3.png)　　

3）如果一个线程A需要访问对象object1的synchronized方法fun1，另外一个线程B需要访问对象object2的synchronized方法fun1，即使object1和object2是同一类型），也不会产生线程安全问题，因为他们访问的是不同的对象，所以不存在互斥问题。

```java
package com.thread.lock;

import java.util.ArrayList;

/**
 * @Author: 98050
 * @Time: 2019-04-29 19:56
 * @Feature:
 */
public class Test006 {

    public static void main(String[] args) {
        Common common = new Common();
        Common common2 = new Common();
        new Thread(() -> {
            common.insert(Thread.currentThread());
        }).start();

        new Thread(() -> {
            common2.insert(Thread.currentThread());
        }).start();
    }

}

class Common{
    private ArrayList<Integer> list = new ArrayList<>();

    public synchronized void insert(Thread thread){
        for (int i = 0; i < 5; i++) {
            System.out.println(thread.getName() + "在插入数据：" + i);
            list.add(i);
        }
    }

    public void insert2(){
        for (int i = 0; i < 5; i++) {
            System.out.println("插入数据：" + i);
            list.add(i);
        }
    }
}
```

运行结果：

![](http://mycsdnblog.work/201919292036-s.png)

## 9.3 同步代码块

**同步代码块中使用的锁对象必须是已经实例化的对象，否则会报空指针异常。**

```java
package com.thread.lock;

import java.util.ArrayList;

/**
 * @Author: 98050
 * @Time: 2019-04-29 19:56
 * @Feature:
 */
public class Test006 {

    public static void main(String[] args) {
        Common common = new Common();
        new Thread(() -> {
            common.insert(Thread.currentThread());
        }).start();

        new Thread(() -> {
            common.insert(Thread.currentThread());
        }).start();
    }

}

class Common{
    private ArrayList<Integer> list = new ArrayList<>();

    public void insert(Thread thread){
        synchronized (this) {
            for (int i = 0; i < 5; i++) {
                System.out.println(thread.getName() + "在插入数据：" + i);
                list.add(i);
            }
        }
    }
}
```

运行结果：

![](http://mycsdnblog.work/201919292039-F.png)

## 9.4 静态同步方法

```java
package com.thread.lock;

/**
 * @Author: 98050
 * @Time: 2019-04-29 19:56
 * @Feature:
 */
public class Test006 {

    public static void main(String[] args) {
        Common common = new Common();
        new Thread(new Runnable(){

            @Override
            public void run() {
                common.insert();
            }
        }).start();

        new Thread(new Runnable(){

            @Override
            public void run() {
                Common.insert2();
            }
        }).start();
    }

}

class Common{

    public synchronized void insert(){
        System.out.println("执行insert1");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("执行insert1完毕");
    }

    public static synchronized void insert2(){
        System.out.println("执行insert2");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("执行insert2完毕");
    }
}
```

运行结果：

![](http://mycsdnblog.work/201919292044-Q.png)

如果一个线程执行一个对象的非static synchronized方法，另外一个线程需要执行这个对象所属类的static synchronized方法，此时不会发生互斥现象，**因为访问static synchronized方法占用的是类锁，而访问非static synchronized方法占用的是对象锁，所以不存在互斥现象**。

------

**所有的非静态同步方法用的都是同一把锁——实例对象本身，也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待获取锁的方法释放锁后才能获取锁，可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁，所以毋须等待该实例对象已获取锁的非静态同步方法释放锁就可以获取他们自己的锁。**

**而所有的静态同步方法用的也是同一把锁——类对象本身，这两把锁是两个不同的对象，所以静态同步方法与非静态同步方法之间是不会有竞态条件的。但是一旦一个静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁，而不管是同一个实例对象的静态同步方法之间，还是不同的实例对象的静态同步方法之间，只要它们同一个类的实例对象！**

**而对于同步块，由于其锁是可以选择的，所以只有使用同一把锁的同步块之间才有着竞态条件**，这就得具体情况具体分析了，但这里有个需要注意的地方，同步块的锁是可以选择的，但是不是可以任意选择的！！！！这里必须要注意一个物理对象和一个引用对象的实例变量之间的区别！使用一个引用对象的实例变量作为锁并不是一个好的选择，因为同步块在执行过程中可能会改变它的值，其中就包括将其设置为null，而对一个null对象加锁会产生异常，并且对不同的对象加锁也违背了同步的初衷！这看起来是很清楚的，但是一个经常发生的错误就是选用了错误的锁对象，因此必须注意：**同步是基于实际对象而不是对象引用的！多个变量可以引用同一个对象，变量也可以改变其值从而指向其他的对象，因此，当选择一个对象锁时，我们要根据实际对象而不是其引用来考虑！作为一个原则，不要选择一个可能会在锁的作用域中改变值的实例变量作为锁对象！！！！**  

# 十、偏向锁

在没有实际竞争的情况下，还能够针对部分场景继续优化。如果不仅仅没有实际竞争，自始至终，使用锁的线程都只有一个，那么，维护轻量级锁都是浪费的。**偏向锁的目标是，减少无竞争且只有一个线程使用锁的情况下，使用轻量级锁产生的性能消耗**。**轻量级锁每次申请、释放锁都至少需要一次CAS，但偏向锁只有初始化时需要一次CAS。**

“偏向”的意思是，偏向锁假定将来只有第一个申请锁的线程会使用锁（不会有任何线程再来申请锁），因此，只需要在Mark Word中CAS记录owner（本质上也是更新，但初始值为空），如果记录成功，则偏向锁获取成功，记录锁状态为偏向锁，以后当前线程等于owner就可以零成本的直接获得锁；否则，说明有其他线程竞争，膨胀为轻量级锁。

**偏向锁无法使用自旋锁优化，因为一旦有其他线程申请锁，就破坏了偏向锁的假定。**

## 10.1 偏向锁的撤销

偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。

## 10.2 关闭偏向锁

`-XX:-UseBiasedLocking=false`

# 十一、轻量级锁

## 11.1 轻量级锁加锁

JVM会先在当前线程的栈帧中创建用于存储锁记录的空间，并且将对象头中的Mark Word复制到锁记录中，称为Displaced Mark Word。然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。

## 11.2 轻量级锁解锁

轻量级锁解锁时，会使用原子操作CAS将Displaced Mark Word替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀为重量级锁。

![](http://mycsdnblog.work/201919011950-T.png)

# 十二、锁分配和膨胀过程

![](http://mycsdnblog.work/201919231701-c.png)

# 十三、锁的优缺点对比

| 锁       | 优点                                                         | 缺点                                           | 适用场景                         |
| -------- | ------------------------------------------------------------ | ---------------------------------------------- | -------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步方法相比仅存在纳秒级的差距 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗 | 适用于只有一个线程访问同步块场景 |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的响应速度                     | 如果始终得不到锁竞争的线程，使用自旋会消耗CPU  | 追求响应时间同步块执行速度非常快 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU                              | 线程阻塞，响应时间缓慢                         | 追求吞吐量同步块执行速度较长=    |

# 十四、Lock锁

## 14.1 synchronized的缺陷

synchronized是java中的一个关键字，也就是说是Java语言内置的特性。那么为什么会出现Lock呢？

如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里获取锁的线程释放锁只会有两种情况：

　　1）获取锁的线程执行完了该代码块，然后线程释放对锁的占有；

　　2）线程执行发生异常，此时JVM会让线程自动释放锁。

　　那么如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程便只能干巴巴地等待，试想一下，这多么影响程序执行效率。

　　**因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断），通过Lock就可以办到。**

　　再举个例子：当有多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作会发生冲突现象，但是读操作和读操作不会发生冲突现象。

　　但是采用synchronized关键字来实现同步的话，就会导致一个问题：

　　如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。

　　**因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过Lock就可以办到。**

　　**另外，通过Lock可以知道线程有没有成功获取到锁。这个是synchronized无法办到的。**

　　总结一下，也就是说Lock提供了比synchronized更多的功能。但是要注意以下几点：

　　**1）Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；**

　　**2）Lock和synchronized有一点非常大的不同，采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。**

Lock 接口可以尝试非阻塞地获取锁 当前线程尝试获取锁。如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁。
Lock 接口能被中断地获取锁 与 synchronized 不同，获取到锁的线程能够响应中断，当获取到的锁的线程被中断时，中断异常将会被抛出，同时锁会被释放。

Lock 接口在指定的截止时间之前获取锁，如果截止时间到了依旧无法获取锁，则返回。

1）Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；

2）synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

3）Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；

4）通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

5）Lock可以提高多个线程进行读操作的效率。

　　在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。所以说，在具体使用时要根据适当情况选择。

## 14.2 Lock

```java
public interface Lock {


    void lock();

    void lockInterruptibly() throws InterruptedException;

    boolean tryLock();

    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;


    void unlock();


    Condition newCondition();
}
```

lock()、tryLock()、tryLock(long time, TimeUnit unit)和lockInterruptibly()这四个都是用来获取锁的。

unlock()是用来释放锁的。

> lock()

如果采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一般来说，使用Lock必须在try-catch块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。通常使用Lock来进行同步的话，是以下面这种形式去使用的：

```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){
     
}finally{
    lock.unlock();   //释放锁
}
```

> tryLock()

tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。

使用方法：

```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){
         
     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

> tryLock(long time, TimeUnit unit)

tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。使用方法同上

> lockInterruptibly()

lockInterruptibly()方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

由于lockInterruptibly()的声明中抛出了异常，所以lock.lockInterruptibly()必须放在try块中或者在调用lockInterruptibly()的方法外声明抛出InterruptedException。

因此lockInterruptibly()一般的使用形式如下：

```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```

注意:

当一个线程获取了锁之后，是不会被interrupt()方法中断的。因为单独调用interrupt()方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程。因此当通过lockInterruptibly()方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。

而用synchronized修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

## 14.3 ReentrantLock

ReentrantLock，意思是“可重入锁”。ReentrantLock是唯一实现了Lock接口的类，并且ReentrantLock提供了更多的方法。下面通过一些实例看具体看一下如何使用ReentrantLock。

> lock()

```java
package com.thread.lock;

import java.util.ArrayList;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2018-12-06 18:03
 * @Feature: lock()
 */
public class Test {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();
    private static Lock lock = new ReentrantLock();
    public static void main(String[] args)  {
        final Test test = new Test();

        new Thread(){
            @Override
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();

        new Thread(){
            @Override
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }

    public void insert(Thread thread) {
        lock.lock();
        try {
            System.out.println(thread.getName()+"得到了锁");
            for(int i=0;i<5;i++) {
                arrayList.add(i);
            }
        } catch (Exception e) {
            // TODO: handle exception
        }finally {
            System.out.println(thread.getName()+"释放了锁");
            lock.unlock();
        }
    }
}
```

结果：

![img](https://img-blog.csdnimg.cn/20181206181207972.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> tryLock()

```java
package com.thread.lock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2018-12-06 18:12
 * @Feature: tryLock()的使用
 */
public class Test2 {

    private static Lock lock = new ReentrantLock();
    public static void main(String[] args)  {
        final Test2 test = new Test2();

        new Thread(){
            @Override
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();

        new Thread(){
            @Override
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }

    public void insert(Thread thread) {

        if (lock.tryLock()){
            try {
                System.out.println(thread.getName()+"得到了锁");
                Thread.sleep(3000);
            } catch (Exception e) {
                // TODO: handle exception
            }finally {
                System.out.println(thread.getName()+"释放了锁");
                lock.unlock();
            }
        }else {
            System.out.println(thread.getName() + "获取锁失败");
        }
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

结果：

![img](https://img-blog.csdnimg.cn/20181206182230411.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> tryLock(long time, TimeUnit unit)

```java
package com.thread.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2018-12-06 18:24
 * @Feature: tryLock(long time, TimeUnit unit)的使用
 */
public class Test3 {
    private static Lock lock = new ReentrantLock();
    public static void main(String[] args)  {
        final Test3 test = new Test3();

        new Thread(){
            @Override
            public void run() {
                try {
                    test.insert(Thread.currentThread());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();

        new Thread(){
            @Override
            public void run() {
                try {
                    test.insert(Thread.currentThread());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
    }

    public void insert(Thread thread) throws InterruptedException {
        if (lock.tryLock(4, TimeUnit.SECONDS)){
            try {
                System.out.println(thread.getName()+"得到了锁");
                Thread.sleep(3000);
            } catch (Exception e) {
                // TODO: handle exception
            }finally {
                System.out.println(thread.getName()+"释放了锁");
                lock.unlock();
            }
        }else {
            System.out.println(thread.getName() + "获取锁失败");
        }
    }
}
```

 等待时间为4秒，结果：

![img](https://img-blog.csdnimg.cn/20181206193701736.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> lockInterruptibly()

```java
package com.thread.lock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2018-12-06 19:38
 * @Feature: lockInterruptibly()使用方法
 */
public class Test4 {
    private Lock lock = new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        Test4 test = new Test4();
        MyThread thread1 = new MyThread(test);
        MyThread thread2 = new MyThread(test);
        thread1.start();
        thread2.start();
        Thread.sleep(2000);
        thread2.interrupt();
    }

    public void insert(Thread thread) throws InterruptedException{
        lock.lockInterruptibly();   //注意，如果需要正确中断等待锁的线程，必须将获取锁放在外面，然后将InterruptedException抛出
        try {
            System.out.println(thread.getName()+"得到了锁");
            Thread.sleep(4000);
        }
        finally {
            System.out.println(Thread.currentThread().getName()+"执行finally");
            lock.unlock();
            System.out.println(thread.getName()+"释放了锁");
        }
    }
}

class MyThread extends Thread {
    private Test4 test = null;

    public MyThread(Test4 test) {
        this.test = test;
    }

    @Override
    public void run() {
        try {
            test.insert(Thread.currentThread());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

结果，线程2被中断：

![img](https://img-blog.csdnimg.cn/20181206195458816.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 十五、死锁检测

```java
package com.example.deadlock;

/**
 * @Author: 98050
 * @Time: 2019-05-23 14:30
 * @Feature:
 */
public class DeadLockDemo {

    private static String A = "A";
    private static String B = "B";

    public static void main(String[] args) {
        new DeadLockDemo().deadLock();
    }

    private void deadLock(){
        new Thread(new Runnable() {
            public void run() {
                synchronized (A) {
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (B){
                        System.out.println(1);
                    }
                }
            }
        }).start();

        new Thread(new Runnable() {
            public void run() {
                synchronized (B) {
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (A){
                        System.out.println(2);
                    }
                }
            }
        }).start();
    }
}
```

使用Jconsole

![](http://mycsdnblog.work/201919231456-6.png)

![](http://mycsdnblog.work/201919231456-Z.png)