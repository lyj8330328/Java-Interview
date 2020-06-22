# 一、什么是线程

一个普通的Java程序包含哪些线程？

```java
package com.example.thread;

import java.lang.management.ManagementFactory;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;

/**
 * @Author: 98050
 * @Time: 2019-05-29 15:41
 * @Feature:
 */
public class MultiThread {

    public static void main(String[] args) throws InterruptedException {
        //1.获取Java线程管理MXBean
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        //2.不需要获取同步的monitor和synchronizer信息，仅获取线程和线程堆栈信息
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
        //3.输出
        for (ThreadInfo threadInfo : threadInfos){
            System.out.println(threadInfo.getThreadId() + ":" + threadInfo.getThreadName());
        }
    }
}
```

![](http://mycsdnblog.work/201919291614-0.png)

# 二、为什么使用多线程

- 多个处理器核心
- 更快的响应时间
- 更好的编程模型

# 三、线程创建的方式

- 继承Thread类，重写run方法
- 实现Runable接口，重写run方法
- 使用匿名内部类

# 四、线程的分类

Java中有两种线程，一种是用户线程，另一种是守护线程。

用户线程是指用户自定义创建的线程，主线程停止，用户线程不会停止

守护线程当进程不存在或主线程停止，守护线程也会被停止。

使用setDaemon(true)方法设置为守护线程

# 五、线程的优先级

## 5.1 基本概念

通过setPriority(int)方法来修改优先级，默认优先级为5，优先级高的线程分配时间片的数量要多于优先级低的线程。

高IO的线程优先级设置较高，高CPU的优先级低

**注意设置了优先级， 不代表每次都一定会被执行。 只是CPU调度会有限分配**

## 5.2 yield方法

Thread.yield()方法的作用：暂停当前正在执行的线程，并执行其他线程。（可能没有效果）

yield()让当前正在运行的线程回到可运行状态，以允许具有相同优先级的其他线程获得运行的机会。因此，使用yield()的目的是让具有相同优先级的线程之间能够适当的轮换执行。但是，实际中无法保证yield()达到让步的目的，因为，让步的线程可能被线程调度程序再次选中。

结论：大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。

## 5.3 join方法

当在主线程当中执行到t1.join()方法时，就认为主线程应该把执行权让给t1

# 六、线程的状态

## 6.1 基本概念

![](http://mycsdnblog.work/201919052036-q.png)

| 状态名称     |                             说明                             |
| :----------- | :----------------------------------------------------------: |
| NEW          |         初试状态，线程被创建，但还没调用start()方法          |
| RUNNABLE     | 运行状态，**Java线程将操作系统中的就绪和运行统一为“运行中”** |
| BLOCKED      |                  阻塞状态，表示线程阻塞于锁                  |
| WAITING      |                等待状态，表示线程进入等待状态                |
| TIME_WAITING |            超时等待状态，可以在指定的时间自行返回            |
| TERMINATED   |                           终止状态                           |

```java 
package com.example.threadstate;

import java.util.concurrent.ThreadFactory;

/**
 * @Author: 98050
 * @Time: 2019-05-29 16:33
 * @Feature:
 */
public class ThreadState {

    /**
     * 该线程不断进行睡眠
     */
    static class TimeWaiting implements Runnable{

        @Override
        public void run() {
            while (true){
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 该线程一直在等待锁
     */
    static class Waiting implements Runnable{

        @Override
        public void run() {
            while (true){
                synchronized (Waiting.class){
                    try {
                        Waiting.class.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    /**
     * 该线程加锁，但是不释放锁
     */
    static class Blocked implements Runnable{

        @Override
        public void run() {
            synchronized (Blocked.class){
                while (true){
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        new Thread(new TimeWaiting(),"TimeWaitingThread").start();
        new Thread(new Waiting(),"WaitingThread").start();
        //两个Blocked线程，一个获取锁，另外一个阻塞
        new Thread(new Blocked(),"BlockedThread-1").start();
        new Thread(new Blocked(),"BlockedThread-2").start();
    }
}
```

![1559120626687](http://mycsdnblog.work/201919291704-n.png)

## 6.2 操作操作系统中的线程和Java线程状态的关系

从实际意义上来讲，操作系统中的线程除去new和terminated状态，一个线程真实存在的状态，只有：

- ready：表示线程已经被创建，正在等待系统调度分配CPU使用权。
- running：表示线程获得了CPU使用权，正在进行运算
- waiting：表示线程等待（或者说挂起），让出CPU资源给其他线程使用

**为什么除去new和terminated状态？是因为这两种状态实际上并不存在于线程运行中，所以也没什么实际讨论的意义。**

对于Java中的线程状态：

**无论是Timed Waiting ，Waiting还是Blocked，对应的都是操作系统线程的waiting（等待）状态。**

**而Runnable状态，则对应了操作系统中的ready和running状态。**

而对不同的操作系统，由于本身设计思路不一样，对于线程的设计也存在种种差异，所以JVM在设计上，就已经声明：

**虚拟机中的线程状态，不反应任何操作系统线程状态**

# 七、启动和终止线程

## 7.1 启动

线程初始化：在运行线程之前首先要构造一个线程对象，线程对象在构造的时候需要提供线程所需要的属性，如线程所属的线程组、线程优先级、是否是守护线程等信息。

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    this.name = name;

    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    if (g == null) {
        /* Determine if it's an applet or not */

        /* If there is a security manager, ask the security manager
           what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }

        /* If the security doesn't have a strong opinion of the matter
           use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* checkAccess regardless of whether or not threadgroup is
       explicitly passed in. */
    g.checkAccess();

    /*
     * Do we have the required permissions?
     */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();

    this.group = g;
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext =
            acc != null ? acc : AccessController.getContext();
    this.target = target;
    setPriority(priority);
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID();
}
```

启动：调用start()方法启动线程，当前线程同步告知Java虚拟机，只要线程规划器空闲，应立即启动调用start()方法的线程。

```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
```



> **start和run的区别**

**直接调用run方法是无法开启一个新线程的。start方法其实是在一个新的操作系统线程上面去调用run方法。换句话说，直接调用run方法而不是调用start方法的话，它并不会开启新的线程，而是在调用run的当前的线程当中执行你的操作。**

## 7.2 安全的终止线程

中断或者boolean变量来控制

中断：线程的一个标识位

```java
package com.example.threadstop;

/**
 * @Author: 98050
 * @Time: 2019-05-29 17:47
 * @Feature:
 */
public class Shutdown {

    private static class Runner implements Runnable{

        private long i;

        private volatile boolean on = true;

        @Override
        public void run() {
            while (on && !Thread.currentThread().isInterrupted()){
                i++;
            }
            System.out.println("i:" + i);
        }

        public void cancel(){
            on = false;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Runner one = new Runner();
        Thread thread1 = new Thread(one,"CountThread");
        thread1.start();
        Thread.sleep(1000);
        thread1.interrupt();

        Runner two = new Runner();
        Thread thread2 = new Thread(two,"CountThread");
        thread2.start();
        Thread.sleep(1000);
        two.cancel();
    }
}
```

![](http://mycsdnblog.work/201919291838-J.png)

# 八、线程之间的通讯

## 8.1 多线程的三大特性

### 8.1.1 基本概念

**原子性**：是指一个操作是不可中断的。即使是多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。比如，对于一个静态全局变量int i，两个线程同时对它赋值，线程A给他赋值为1，线程B给他赋值为-1。那么不管这两个线程以何种方式。何种步调工作，i的值要么是1，要么是-1.线程A和线程B之间是没有干扰的。这就是原子性的一个特点，不可被中断。

**可见性**：是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道这个修改。显然，对于串行来说，可见性问题是不存在的。

**有序性**：在并发时，程序的执行可能会出现乱序。给人的直观感觉就是：写在前面的代码，会在后面执行。有序性问题的原因是因为程序在执行时，可能会进行指令重排，重排后的指令与原指令的顺序未必一致。

------

**原子性与可见性的关系**

### 8.1.2 Java内存模型

共享内存模型指的就是Java内存模型(简称JMM)，**JMM决定一个线程对共享变量的写入时,能对另一个线程可见**。从抽象的角度来看，**JMM定义了线程和主内存之间的抽象关系**：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。

![](http://mycsdnblog.work/201919082154-j.png)

Java内存模型简称JMM，定义了一个线程对另一个线程可见。共享变量存放在主内存中，每个线程都有自己的本地内存，当多个线程同时访问一个数据的时候，可能本地内存没有及时刷新到主内存，所以就会发生线程安全问题。

## 8.2 synchronized和volatile关键字

### 8.2.1 synchronized

同步代码块

synchronized 修饰方法使用锁是当前this锁。

synchronized 修饰静态方法使用锁是当前类的字节码文件

#### 8.2.1.1 synchronized的实现原理

JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，代码块同步是使用monitorenter和monitorexit指令实现的。monitorenter指令时在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处，JVM要保证每个monitorenter必须有对应的monitorexit与之配对。

### 8.2.2 volatile

#### 8.2.2.1 volatile的实现原理

如果对声明了volatile的变量进行写操作，**JVM就会向处理器发送一条Lock前缀的指令**，将这个变量所在缓存行的数据写回到系统内存中，这个回写操作会使在其它CPU里缓存了该内存地址的数据无效（MESI缓存一致性协议）。

#### 8.2.2.2 volatile的使用优化

> **伪共享概念**

**缓存系统中是以缓存行（cache line）为单位存储的，当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。**

> **计算机的基本结构**

下图是计算的基本结构。L1、L2、L3分别表示一级缓存、二级缓存、三级缓存，越靠近CPU的缓存，速度越快，容量也越小。所以L1缓存很小但很快，并且紧靠着在使用它的CPU内核；L2大一些，也慢一些，并且仍然只能被一个单独的CPU核使用；L3更大、更慢，并且被单个插槽上的所有CPU核共享；最后是主存，由全部插槽上的所有CPU核共享。

![](http://mycsdnblog.work/201919192142-P.png)

当CPU执行运算的时候，它先去L1查找所需的数据、再去L2、然后是L3，如果最后这些缓存中都没有，所需的数据就要去主内存拿。走得越远，运算耗费的时间就越长。所以如果你在做一些很频繁的事，你要尽量确保数据在L1缓存中。另外，线程之间共享一份数据的时候，需要一个线程把数据写回主存，而另一个线程访问主存中相应的数据。

> **缓存行**

Cache是由很多个cache line组成的。每个cache line通常是**64**字节，并且它有效地引用主内存中的一块儿地址。一个Java的long类型变量是8字节，因此在一个缓存行中可以存8个long类型的变量。

**CPU每次从主存中拉取数据时，会把相邻的数据也存入同一个cache line。**

在访问一个long数组的时候，如果数组中的一个值被加载到缓存中，它会自动加载另外7个。因此能非常快的遍历这个数组。事实上，可以非常快速的遍历在连续内存块中分配的任意数据结构。

```java
package com.thread.falsesharing;
 
/**
 * @Author: 98050
 * @Time: 2018-12-19 23:25
 * @Feature: cache line特性
 */
public class CacheLineEffect {
 
    private static long[][] result;
 
    public static void main(String[] args) {
        int row =1024 * 1024;
        int col = 8;
        result = new long[row][];
        for (int i = 0; i < row; i++) {
            result[i] = new long[col];
            for (int j = 0; j < col; j++) {
                result[i][j] = i+j;
            }
        }
 
        long start = System.currentTimeMillis();
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                result[i][j] = 0;
            }
        }
        System.out.println("使用cache line特性，循环时间：" + (System.currentTimeMillis() - start));
 
        long start2 = System.currentTimeMillis();
        for (int i = 0; i < col; i++) {
            for (int j = 0; j < row; j++) {
                result[j][i] = 1;
            }
        }
        System.out.println("没有使用cache line特性，循环时间：" + (System.currentTimeMillis() - start2));
 
    }
}
```

结果：

![](http://mycsdnblog.work/201919192144-T.png)

> **伪共享**

![](http://mycsdnblog.work/201919192146-V.png)

如上图变量x,y同时被放到了CPU的一级和二级缓存，当线程1使用CPU1对变量x进行更新时候，首先会修改cpu1的一级缓存变量x所在缓存行，这时候缓存一致性协议会导致cpu2中变量x对应的缓存行失效，那么线程2写入变量y的时候就只能去二级缓存去查找，这就破坏了一级缓存，而一级缓存比二级缓存更快。更坏的情况下如果cpu只有一级缓存，那么会导致频繁的直接访问主内存。

```java
package com.thread.falsesharing;
 
/**
 * @Author: 98050
 * @Time: 2018-12-20 12:06
 * @Feature: 伪共享
 */
public class FalseSharing implements Runnable {
 
    /**
     * 线程数
     */
    public static int NUM_THREADS = 4;
    /**
     * 迭代次数
     */
    public final static long ITERATIONS = 500L * 1000L * 1000L;
    private final int arrayIndex;
    private static VolatileLong[] longs;
    public static long SUM_TIME = 0L;
 
    public FalseSharing(final int arrayIndex) {
        this.arrayIndex = arrayIndex;
    }
    public void run() {
        long i = ITERATIONS + 1;
        while (0 != --i){
            longs[arrayIndex].value = i;
        }
    }
 
    public final static class VolatileLong {
        public volatile long value = 0L;
        public long p1, p2, p3, p4, p5, p6; //缓存行填充
    }
 
    private static void runTest() throws InterruptedException {
        Thread[] thread = new Thread[NUM_THREADS];
        for (int i = 0; i < thread.length; i++) {
            thread[i] = new Thread(new FalseSharing(i));
        }
 
        for (Thread t : thread){
            t.start();
        }
        for (Thread t : thread){
            t.join();
        }
    }
 
    public static void main(String[] args) throws InterruptedException {
        Thread.sleep(10000);
        for (int i = 0; i < 10; i++) {
            System.out.println(i);
            if (args.length == 1){
                NUM_THREADS = Integer.parseInt(args[0]);
            }
            longs = new VolatileLong[NUM_THREADS];
            for (int j = 0; j < longs.length; j++) {
                longs[j] = new VolatileLong();
            }
            final long start = System.nanoTime();
            runTest();
            final long end = System.nanoTime();
            SUM_TIME += end - start;
        }
        System.out.println("平均耗时：" + SUM_TIME / 10);
    }
}
```

四个线程修改一数组不同元素的内容。元素的类型是 VolatileLong，只有一个长整型成员 value 和 6 个没用到的长整型成员。value 设为 volatile 是为了让 value 的修改对所有线程都可见。

程序分两种情况执行，第一种情况为不屏蔽缓存行填充，第二种情况为屏蔽缓存行填充。

为了"保证"数据的相对可靠性，程序取 10 次执行的平均时间。执行情况如下：

**屏蔽缓存行填充**

![](http://mycsdnblog.work/201919192154-D.png)

**不屏蔽缓存行填充**

![](http://mycsdnblog.work/201919192155-f.png)

两个逻辑一模一样的程序，前者的耗时大概是后者的 2倍。那么这个时候，我们再用伪共享（False Sharing）的理论来分析一下，前者 longs 数组的 4 个元素，由于 VolatileLong 只有 1 个长整型成员，所以一个数组单元就是16个字节（long数据类型8个字节+类对象的字节码的对象头8个字节），进而整个数组都将被加载至同一缓存行（16*4字节），但有4个线程同时操作这条缓存行，于是伪共享就悄悄地发生了。

> **避免伪共享**

**一条缓存行有 64 字节，而 Java 程序的对象头固定占 8 字节(32位系统)或 12 字节( 64 位系统默认开启压缩, 不开压缩为 16 字节)，所以只需要填 6 个无用的长整型补上6*8=48字节，让不同的 VolatileLong 对象处于不同的缓存行，就避免了伪共享( 64 位系统超过缓存行的 64 字节也无所谓，只要保证不同线程不操作同一缓存行就可以)。**

**Java8中已经提供了官方的解决方案，Java8中新增了一个注解：@sun.misc.Contended。加上这个注解的类会自动补齐缓存行，需要注意的是此注解默认是无效的，需要在jvm启动时设置-XX:-RestrictContended才会生效。**

![](http://mycsdnblog.work/201919192203-b.png)

#### 8.2.2.3 volatile 总结

volatile 保证了线程间共享变量的及时可见性，但不能保证原子性。

1、保证此变量对所有的线程的可见性，这里的“可见性”，当一个线程修改了这个变量的值，volatile 保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。但普通变量做不到这点，普通变量的值在线程间传递均需要通过主内存（详见：Java内存模型）来完成。

2、禁止指令重排序优化。有volatile修饰的变量，赋值后多执行了一个“load addl $0x0, (%esp)”操作，这个操作相当于一个内存屏障（指令重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；（什么是指令重排序：是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理）。

3、volatile 性能

volatile 的读性能消耗与普通变量几乎相同，但是写操作稍慢，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。

### 8.2.3 区别

**synchronized保证可见性和操作的原子性：**

在Java内存模型中，synchronized规定，线程在加锁时，**先清空工作内存**→在主内存中拷贝最新变量的副本到工作内存→执行完代码→将更改后的共享变量的值刷新到主内存中→释放互斥锁。

**volatile保证可见性但不保证原子性：**

volatile实现内存可见性`是通过store和load指令完成的`；也就是`对volatile变量执行写操作时`，会在`写操作`后加入一条`store指令`，即`强迫线程将最新的值刷新到主内存中`；而在`读操作`时，会加入一条`load指令`，即`强迫从主内存中读入变量的值`。

## 8.3 等待/通知机制

### 8.3.1 基本概念

1、因为涉及到对象锁,他们必须都放在synchronized中来使用. wait、notify一定要在synchronized里面进行使用。

2、notify()通知一个在对象上等待的线程，使其从wait()方法返回，而返回的前提是该线程获取到了对象的锁

3、notifyAll()通知所有等待在该对象上的线程

4、wait()调用该方法的线程进入WAITING状态，只有等待另外线程的通知或被中断才会返回，需要注意，调用wait()方法后，会释放对象的锁

**注意:一定要在线程同步中使用,并且是同一个锁的资源**

### 8.3.2 wait与sleep区别

- 对于sleep()方法，我们首先要知道该方法是属于Thread类中的。而wait()方法，则是属于Object类中的。
- sleep()方法导致了程序暂停执行指定的时间，让出cpu该其他线程，但是他的监控状态依然保持者，当指定的时间到了又会自动恢复运行状态。
- 在调用sleep()方法的过程中，线程不会释放对象锁。
- 而当调用wait()方法的时候，线程会放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象调用notify()方法后本线程才进入对象锁定池准备获取对象锁进入运行状态。

### 8.3.3 wait与join的区别

join方法在调用wait方法后, 并没有执行notify方法, 这个是在jvm中实现的, 一个线程执行结束后会执行该线程自身对象的notifyAll方法。

join源码：

```java
public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;    
	if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
 
    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```
注意这是一个用synchronized修饰的方法，所以它是一个同步方法。

逻辑：

- 判断参数时间参数，如果参数小于0，抛出IllegalArgumentException("timeout value is negative")异常
- 参数等于0，判断调用join的线程(假设是A)是否存活，不存活就不执行操作，如果存活，就调用wait(0)，阻塞join方法，等待A线程执行完在结束join方法。
- 参数大于0，判断调用join的A线程是否存活，不存活就不执行操作，如果存活，就调用wait(long millis)，阻塞join方法，等待时间结束再继续执行join方法。

------

**举个例子，现有两个线程A与B，A中调用了B.join()，那么A线程会停止，等B先运行完后A才开始运行。那么有人会想当然的理解成，B.join()等同于B.wait()，不就是B在wait，也就是B等待吗，为什么反而会让A停运，让B先运行呢？要理解这里，先假设一个对象D，需要知道其实wait()方法是D.wait()，也就是说获取了对象D的对象锁的线程释放该对象锁，重新进入对象的锁获取队列中，与其他线程一起竞争该锁。**

**主线程A看作是调用B对象的线程，在线程对象B被A线程调用了join后，A线程会释放该对象的对象锁，进入该对象的锁获取队列中，等待唤醒（例如该对象执行结束），这样A线程才能继续执行下去，这也是为什么join能控制线程执行顺序的原因所在。**

------

- join方法内部是通过wait进行阻塞的，所以join和wait都会释放锁。而sleep不释放锁，sleep的锁是当前线程对象。

- 释放锁和不释放锁的区别：释放锁后，该对象同步方法可被其他对象异步调用，而不释放锁则该对象其他同步方法被调用时会进入等待获得锁。

- wait和join唤醒后，需要重新获得锁。

## 8.4 等待/通知经典范式（生产者-消费者）

### 8.4.1 基本知识

```java
package com.example.communication;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author: 98050
 * @Time: 2019-05-30 16:05
 * @Feature:
 */
public class WaitNotify {

    static boolean flag = true;
    static Object lock = new Object();

    static class Wait implements Runnable{
        @Override
        public void run() {
            synchronized (lock){
                while (flag){
                    System.out.println(Thread.currentThread() + "flag is true wait" + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread() + "flag is false running" + new SimpleDateFormat("HH:mm:ss").format(new Date()));
            }
        }
    }


    static class Notify implements Runnable{

        @Override
        public void run() {
            synchronized (lock){
                System.out.println(Thread.currentThread() + "hold lock notify" + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            synchronized (lock){
                System.out.println(Thread.currentThread() + "hold lock again sleep" + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread waitThread = new Thread(new Wait(),"WaitThread");
        waitThread.start();

        Thread.sleep(1000);

        Thread notifyThread = new Thread(new Notify(),"NotifyThread");
        notifyThread.start();
    }
}
```

![](http://mycsdnblog.work/201919301628-c.png)

**消费者**

1. 获取对象的锁
2. 如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件
3. 条件满足则执行对应的逻辑

```java
synchronized (对象){
    while (条件不满足){
        对象.wait();
    }
    对应的处理逻辑
}
```

**生产者**

1. 获得对象锁
2. 改变条件
3. 通知所有等待在对象上的线程

```java
synchronized (对象){
    改变条件;
    对象.notifyAll();
}
```

### 8.4.2 wait/notify存在的问题

> **notify早期通知**

线程A还没有wait的时候，线程B就已经notify了，这样线程B就没有任何响应了，当线程B退出synchronized后，线程A再开始wait，便会一直阻塞等待，直到被别的线程打断。

```java
package com.thread.problem;

/**
 * @Author: 98050
 * @Time: 2019-08-02 17:18
 * @Feature: notify早期通知
 */
public class Test1 {

    /**
     * notify 通知的遗漏很容易理解，即 threadA 还没开始 wait 的时候，threadB 已经 notify 了，
     * 这样，threadB 通知是没有任何响应的，
     * 当 threadB 退出 synchronized 代码块后，threadA 再开始 wait，便会一直阻塞等待，直到被别的线程打断。
     */

    static class WaitThread extends Thread{

        private Object lock;

        public WaitThread(Object lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            synchronized (lock){
                try {
                    System.out.println(Thread.currentThread().getName() + "开始wait");
                    lock.wait();
                    System.out.println(Thread.currentThread().getName() + "结束wait");
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
    }

    static class NotifyThread extends Thread{

        private Object lock;

        public NotifyThread(Object lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            synchronized (lock){
                System.out.println(Thread.currentThread().getName() + "开始notify");
                lock.notify();
                System.out.println(Thread.currentThread().getName() + "结束notify");
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();
        WaitThread waitThread = new WaitThread(lock);
        NotifyThread notifyThread = new NotifyThread(lock);
        notifyThread.start();
        Thread.sleep(3000);
        waitThread.start();
    }
}
```

解决方法，设置标记位

```java
package com.thread.problem;

/**
 * @Author: 98050
 * @Time: 2019-08-02 17:18
 * @Feature: notify早期通知
 */
public class Test1 {

    /**
     * notify 通知的遗漏很容易理解，即 threadA 还没开始 wait 的时候，threadB 已经 notify 了，
     * 这样，threadB 通知是没有任何响应的，
     * 当 threadB 退出 synchronized 代码块后，threadA 再开始 wait，便会一直阻塞等待，直到被别的线程打断。
     */

    static volatile boolean tag = false;

    static class WaitThread extends Thread{

        private Object lock;


        public WaitThread(Object lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            synchronized (lock){
                try {
                    while (!tag) {
                        System.out.println(Thread.currentThread().getName() + "开始wait");
                        lock.wait();
                        System.out.println(Thread.currentThread().getName() + "结束wait");
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
    }

    static class NotifyThread extends Thread{

        private Object lock;

        public NotifyThread(Object lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            synchronized (lock){
                System.out.println(Thread.currentThread().getName() + "开始notify");
                lock.notify();
                tag = true;
                System.out.println(Thread.currentThread().getName() + "结束notify");
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();
        WaitThread waitThread = new WaitThread(lock);
        NotifyThread notifyThread = new NotifyThread(lock);
        notifyThread.start();
        Thread.sleep(3000);
        waitThread.start();
    }
}
```

> **线程中条件的判断不能使用if，要使用while**

> **假死状态**

现象：如果是多消费者和多生产者情况，如果使用notify方法可能会出现“假死”的情况，即唤醒的是同类线程。

原因分析：假设当前多个生产者线程会调用wait方法阻塞等待，当其中的生产者线程获取到对象锁之后使用notify通知处于WAITTING状态的线程，如果唤醒的仍然是生产者线程，就会造成所有的消费者线程都处于等待状态。

解决办法：将notify方法替换成notifyAll方法，如果使用的是lock的话，就将signal方法替换成signalAll方法。

### 8.4.3 实例

> **使用wait/notify实现**

```java
package com.thread.produceconsumer;

import java.util.LinkedList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @Author: 98050
 * @Time: 2019-08-02 16:28
 * @Feature:
 */
public class ProduceConsumer {


    static class Produce implements Runnable{

        private List<Integer> list;
        private int maxLength;

        public Produce(List<Integer> list, int maxLength) {
            this.list = list;
            this.maxLength = maxLength;
        }

        @Override
        public void run() {
            while (true){
                synchronized (list){
                    try {
                        while (list.size() == maxLength) {
                            System.out.println("生产者" + Thread.currentThread().getName() + "list达到最大容量，进行wait");
                            list.wait();
                            System.out.println("生产者" + Thread.currentThread().getName() + "退出wait");
                        }
                        Thread.sleep(2000);
                        Random random = new Random();
                        int i = random.nextInt();
                        System.out.println("生产者" + Thread.currentThread().getName() + "生产数据：" + i);
                        list.add(i);
                        list.notifyAll();
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    static class Consumer implements Runnable{

        private List<Integer> list;

        public Consumer(List<Integer> list) {
            this.list = list;
        }

        @Override
        public void run() {
            while (true){
                synchronized (list){
                    try {
                        while (list.isEmpty()){
                            System.out.println("消费者" + Thread.currentThread().getName() + "list为空，进行wait");
                            list.wait();
                            System.out.println("消费者" + Thread.currentThread().getName() + "退出wait");
                        }
                        Thread.sleep(2000);
                        Integer element = list.remove(0);
                        System.out.println("消费者" + Thread.currentThread().getName() + "消费了数据" + element);
                        list.notifyAll();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        LinkedList<Integer> list = new LinkedList<>();
        ExecutorService service = Executors.newFixedThreadPool(4);
        //生产者
        for (int i = 0; i < 2; i++) {
            service.execute(new Produce(list, 5));
        }
        //消费者
        for (int i = 0; i < 2; i++) {
            service.execute(new Consumer(list));
        }
    }
}
```

> **使用lock+condition实现**

```java
package com.thread.produceconsumer;

import java.util.LinkedList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2019-08-02 19:49
 * @Feature:
 */
public class ProduceConsumer2 {

    private static ReentrantLock lock = new ReentrantLock();
    private static Condition full = lock.newCondition();
    private static Condition empty = lock.newCondition();

    static class Product implements Runnable {

        private List<Integer> list;
        private int maxLength;
        private Lock lock;

        public Product(List<Integer> list, int maxLength, Lock lock) {
            this.list = list;
            this.maxLength = maxLength;
            this.lock = lock;
        }

        @Override
        public void run() {
            while (true){
                lock.lock();
                try {
                    while (list.size() == maxLength){
                        System.out.println("生产者" + Thread.currentThread().getName() + "，list以达到最大容量，进行wait");
                        full.await();
                        System.out.println("生产者" + Thread.currentThread().getName() + "，退出wait");
                    }
                    Random random = new Random();
                    int i = random.nextInt(10);
                    System.out.println("生产者" + Thread.currentThread().getName() + "生产数据：" + i);
                    list.add(i);
                    empty.signalAll();
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        }
    }

    static class Consumer implements Runnable{

        private List<Integer> list;
        private Lock lock;

        public Consumer(List<Integer> list, Lock lock) {
            this.list = list;
            this.lock = lock;
        }

        @Override
        public void run() {
            while (true){
                lock.lock();
                try {
                    while (list.size() == 0){
                        System.out.println("消费者" + Thread.currentThread().getName() + "，list为空，进入等待");
                        empty.await();
                        System.out.println("消费者" + Thread.currentThread().getName() + "，退出wait");
                    }
                    Integer element = list.remove(0);
                    System.out.println("消费者" + Thread.currentThread().getName() + "，消费数据：" + element);
                    full.signalAll();
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        }
    }

    public static void main(String[] args) {
        LinkedList<Integer> list = new LinkedList<>();
        ExecutorService service = Executors.newFixedThreadPool(4);
        for (int i = 0; i < 2; i++) {
            service.execute(new Product(list, 5, lock));
        }
        for (int i = 0; i < 2; i++) {
            service.execute(new Consumer(list, lock));
        }
    }
}
```

> **使用阻塞队列实现**

```java
package com.thread.produceconsumer;

import java.util.Random;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedBlockingQueue;

/**
 * @Author: 98050
 * @Time: 2019-08-02 20:23
 * @Feature: 使用阻塞队列实现生产者-消费者
 */
public class ProduceConsumer3 {

    private static LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<>(5);

    static class Product implements Runnable{

        private BlockingQueue queue;

        public Product(BlockingQueue queue) {
            this.queue = queue;
        }

        @Override
        public void run() {
            try {
                while (true){
                    Random random = new Random();
                    int i = random.nextInt(10);
                    System.out.println("生产者" + Thread.currentThread().getName() + "生产数据：" + i);
                    queue.put(i);
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable{

        private BlockingQueue queue;

        public Consumer(BlockingQueue queue) {
            this.queue = queue;
        }

        @Override
        public void run() {
            try {
                while (true){
                    int o = (int) queue.take();
                    System.out.println("消费者" + Thread.currentThread().getName() + "正在消费：" + o);
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(4);
        for (int i = 0; i < 2; i++) {
            service.execute(new Product(queue));
        }
        for (int i = 0; i < 2; i++) {
            service.execute(new Consumer(queue));
        }
    }

}
```

## 8.5 等待超时模式

由于经典的等待/通知范式无法做到超时等待，也就是说，当消费者在获得锁后，如果条件不满足，等待生产者改变条件之前会一直处于等待状态，在一些实际应用中，会浪费资源，降低运行效率。

事实上，只要对经典范式做出非常小的改动，就可以加入超时等待。

假设超时时间段是T，那么可以推断出，在当前时间now+T之后就会超时。

定义如下变量：

等待持续时间remaining = T；

超时时间future = now + T。

```java
public synchronized Object get(long mills) throws InterruptedException {
    long future = System.currentTimeMillis() + mills;
    long remaining = mills;
    while ((result == null) && remaining > 0){
        wait(remaining);
        remaining = future - System.currentTimeMillis();
    }
    return result;
}
```

例子：

```java
package com.example.communication;

import java.util.LinkedList;

/**
 * @Author: 98050
 * @Time: 2019-06-02 17:34
 * @Feature:
 */
public class Test3 {

    static Operator operator = new Operator(1);
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(new MyThread(10)).start();
        }
    }

    static class MyThread implements Runnable{

        private int count;

        public MyThread(int count) {
            this.count = count;
        }

        @Override
        public void run() {
            while (count > 0){
                try {
                    Object o = operator.get(1000);
                    if (o != null) {
                        /**
                         * 实际操作，睡眠2秒
                         */
                        Thread.sleep(2000);
                        operator.find();
                        System.out.println(o);
                    }else {
                        System.out.println("超时");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    count--;
                }
            }
        }
    }


    static class Operator{

        private final LinkedList<Object> result = new LinkedList<>();

        public Operator(int size) {
            for (int i = 0; i < size; i++) {
                result.addLast(i);
            }
        }

        public Object get(long mills) throws InterruptedException {
            synchronized (result) {
                long future = System.currentTimeMillis() + mills;
                long remaining = mills;
                while ((result .size() == 0) && remaining > 0) {
                    result.wait(remaining);
                    remaining = future - System.currentTimeMillis();
                }
                Object o = null;
                if (result.size() != 0){
                    o = result.removeFirst();
                }
                return o;
            }
        }

        private void find(){
            synchronized (result) {
                result.add(2);
                result.notifyAll();
            }
        }
    }
}
```

## 8.6 ThreadLocal

### 8.6.1 基本概念

ThreadLocal是如何为每个线程创建变量的副本的：

　　首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。

　　**初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。**

　　然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

ThreadLocal提高一个线程的局部变量，访问某个线程拥有自己局部变量。当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

ThreadLocal中解决冲突的策略是使用的线性探测法

ThreadLocalMap解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。处理hash冲突的效率低，**所以，每个线程只存一个变量，这样的话所有的线程存放到map中的Key都是相同的ThreadLocal，如果一个线程要保存多个变量，就需要创建多个ThreadLocal，多个ThreadLocal放入Map中时会极大的增加Hash冲突的可能。**

每个线程都有一个ThreadLoalMap

![](http://mycsdnblog.work/201919082203-t.png)

### 8.6.2 弱引用问题

![](http://mycsdnblog.work/201919082205-t.png)

由于**ThreadLocalMap**的key是弱引用，而Value是强引用。这就导致了一个问题，ThreadLocal在没有外部对象强引用时，发生GC时弱引用Key会被回收，而Value不会回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

**1、如何避免泄漏**

 既然Key是弱引用，那么我们要做的事，**就是在调用ThreadLocal的get()、set()方法时完成后再调用remove方法，将Entry节点和Map的引用关系移除**，这样整个Entry对象在GC Roots分析后就变成不可达了，下次GC的时候就可以被回收。

如果使用ThreadLocal的set方法之后，没有显示的调用remove方法，就有可能发生内存泄露，所以养成良好的编程习惯十分重要，**使用完ThreadLocal之后，记得调用remove方法**。

**2、为什么要这样设计**

**设计成弱引用的目的是为了更好地对ThreadLocal进行回收，当我们在代码中将ThreadLocal的强引用置为null后，这时候Entry中的ThreadLocal理应被回收了，但是如果Entry的key被设置成强引用则该ThreadLocal就不能被回收，这就是将其设置成弱引用的目的。**

## 8.7 面试题

### 8.7.1 示例1

**1、有三个线程分别打印A、B、C,请用多线程编程实现，在屏幕上循环打印10次ABCABC…** 

Lock+Condition实现细粒度的控制

```java
package com.example.join;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2019-05-30 17:02
 * @Feature:
 */
public class Test {

    final static Lock lock = new ReentrantLock();
    final static Condition conditionA = lock.newCondition();
    final static Condition conditionB = lock.newCondition();
    final static Condition conditionC = lock.newCondition();
    volatile static String now = "A";


    static class MyThreadPrintA implements Runnable{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    lock.lock();
                    while (!now.equals("A")) {
                        conditionA.await();
                    }
                    System.out.println(now);
                    now = "B";
                    conditionB.signal();
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        }
    }

    static class MyThreadPrintB implements Runnable{

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    lock.lock();
                    while (!now.equals("B")) {
                        conditionB.await();
                    }
                    System.out.println(now);
                    now = "C";
                    conditionC.signal();
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        }
    }

    static class MyThreadPrintC implements Runnable{

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    lock.lock();
                    while (!now.equals("C")) {
                        conditionC.await();
                    }
                    System.out.println(now);
                    now = "A";
                    conditionA.signal();
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        }
    }


    public static void main(String[] args) throws InterruptedException {
        MyThreadPrintA myThreadPrintA = new MyThreadPrintA();
        MyThreadPrintB myThreadPrintB = new MyThreadPrintB();
        MyThreadPrintC myThreadPrintC = new MyThreadPrintC();

        Thread t1 = new Thread(myThreadPrintA);
        Thread t2 = new Thread(myThreadPrintB);
        Thread t3 = new Thread(myThreadPrintC);

        t1.start();
        t2.start();
        t3.start();
    }

}
```

synchronized + wait、notifyAll

```java
package com.example.communication;

/**
 * @Author: 98050
 * @Time: 2019-05-30 20:21
 * @Feature:
 */
public class Test2 {

    static Object lock = new Object();

    static volatile String name = "A";

    static class MyThreadA implements Runnable{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                synchronized (lock){
                    while (!name.equals("A")){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }
                    System.out.println("A");
                    name = "B";
                    lock.notifyAll();
                }
            }
        }
    }

    static class MyThreadB implements Runnable{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                synchronized (lock){
                    while (!name.equals("B")){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }
                    System.out.println("B");
                    name = "C";
                    lock.notifyAll();
                }
            }
        }
    }

    static class MyThreadC implements Runnable{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                synchronized (lock){
                    while (!name.equals("C")){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }
                    System.out.println("C");
                    name = "A";
                    lock.notifyAll();
                }
            }
        }
    }


    public static void main(String[] args)  {
        new Thread(new MyThreadA()).start();
        new Thread(new MyThreadB()).start();
        new Thread(new MyThreadC()).start();
    }
}
```

在多线程操作中，我们常常会遇到需要先判断信号量状态是否就绪，然后执行后续操作的场景。这里对状态的判断使用的是while而不是单线程下常用的if。 

原因：

在线程中notify或者notifyAll会唤醒一个或多个线程，当线程被唤醒后，被唤醒的线程继续执行阻塞后的操作。

这里分析一下get操纵： 当某个线程得到锁时storage为空，此时它应该wait，下次被唤醒时（任意线程调用notify），storage可能还是空的。因为有可能其他线程清空了storage。如果此时用的是if它将不再判断storage是否为空，直接继续，这样就引起了错误。但如果用while则每次被唤醒时都会先检查storage是否为空再继续，这样才是正确的操作；生产也是同一个道理。

### 8.7.2 示例2

**2、两个线程轮流打印1~100**

思路一：synchronized+wait/notify

```java
package com.thread.test;

/**
 * @Author: 98050
 * @Time: 2019-06-11 21:03
 * @Feature:
 */
public class Test2 {

    private static volatile int i = 1;

    private static Object object = new Object();

    private static boolean flag = false;


    static class MyThread implements Runnable{

        public void run() {
            synchronized (object){
                while (i <= 100){
                    if (flag) {
                        System.out.println(Thread.currentThread().getName() + "打印：" + i++);
                        flag = false;
                    }else {
                        object.notifyAll();
                        try {
                            object.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
                if (i > 100){
                    System.exit(0);
                }
            }
        }
    }

    static class MyThread2 implements Runnable{

        public void run() {
            synchronized (object){
                while (i <= 100) {
                    if (!flag) {
                        System.out.println(Thread.currentThread().getName() + "打印：" + i++);
                        flag = true;
                    } else{
                        object.notifyAll();
                        try {
                            object.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
                if (i > 100){
                    System.exit(0);
                }
            }
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(new MyThread());
        Thread t2 = new Thread(new MyThread2());

        t1.start();
        t2.start();
    }
}
```

------

```java
package com.thread.test;

/**
 * @Author: 98050
 * @Time: 2019-06-11 21:47
 * @Feature:
 */
public class Test5 {

    static volatile int count = 1;
    static volatile Object object = new Object();

    static class MyThread implements Runnable{

        @Override
        public void run() {
            synchronized (object) {
                while (count <= 100) {
                    System.out.println(Thread.currentThread().getName() + "打印：" + count++);
                    try {
                        object.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    static class MyThread2 implements Runnable{

        @Override
        public void run() {
            synchronized (object) {
                while (count <= 100) {
                    System.out.println(Thread.currentThread().getName() + "打印：" + count++);
                    object.notify();
                }
            }
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(new Test2.MyThread());
        Thread t2 = new Thread(new Test2.MyThread2());

        t1.start();
        t2.start();
    }
}
```

思路二：while+boolean变量

```java
package com.thread.test;

/**
 * @Author: 98050
 * @Time: 2019-06-11 21:27
 * @Feature:
 */
public class Test3 {

    static volatile boolean tag = false;

    static volatile int i = 1;

    public static void main(String[] args) {
        new Thread(() -> {
            while (i <= 100){
                if (tag){
                    System.out.println(Thread.currentThread().getName() + "打印：" + i++);
                    tag = false;
                }
            }
        }).start();

        new Thread(() -> {
            while (i <= 100){
                if (!tag){
                    System.out.println(Thread.currentThread().getName() + "打印：" + i++);
                    tag = true;
                }
            }
        }).start();
    }
}
```

思路三：lock + condition

```java
package com.thread.test;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: 98050
 * @Time: 2019-06-11 21:31
 * @Feature:
 */
public class Test4 {

    static Lock lock = new ReentrantLock();
    static Condition condition1 = lock.newCondition();
    static Condition condition2 = lock.newCondition();
    static volatile int count = 1;

    static class MyThread implements Runnable{

        @Override
        public void run() {
            lock.lock();
            try {
                for (int i = 0; i < 50; i++) {
                    while (count % 2 != 0) {
                        condition1.await();
                    }
                    System.out.println(Thread.currentThread().getName() + "打印：" + count++);
                    condition2.signal();
                }
            }catch (Exception e ){

            }finally {
                lock.unlock();
            }
        }
    }

    static class MyThread2 implements Runnable{

        @Override
        public void run() {
            lock.lock();
            try {
                for (int i = 0; i < 50; i++) {
                    while (count % 2 == 0){
                        condition2.await();
                    }
                    System.out.println(Thread.currentThread().getName() + "打印：" + count++);
                    condition1.signal();
                }
            }catch (Exception e ){

            }finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(new MyThread());
        Thread t2 = new Thread(new MyThread2());

        t1.start();
        t2.start();
    }
}
```

### 8.7.3 示例3

**3、四个线程，每个线程计算25个数的合，然后在主线程中计算最后的结果**

方法一：

```java
package com.example.countdownlatch;

import java.util.concurrent.CountDownLatch;

/**
 * @Author: 98050
 * @Time: 2019-06-19 10:30
 * @Feature: 四个线程计算1~100的和
 */
public class Test2 {

    static volatile int result = 0;
    static CountDownLatch countDownLatch = new CountDownLatch(4);

    static class MyThread implements Runnable{

        int start;
        int end;

        public MyThread(int start, int end) {
            this.start = start;
            this.end = end;
        }

        @Override
        public void run() {
            countDownLatch.countDown();
            for (int i = start; i <= end; i++) {
                result += i;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 4; i++) {
            new Thread(new MyThread((i - 1)*25 + 1, i * 25)).start();
        }
        countDownLatch.await();
        System.out.println(result);
    }

}
```

方法二：

```java
package com.example.countdownlatch;

import java.util.concurrent.*;

/**
 * @Author: 98050
 * @Time: 2019-06-19 10:45
 * @Feature: 计算1~100的和
 */
public class Test3 {
    static CountDownLatch countDownLatch = new CountDownLatch(4);
    static ExecutorService executorService = Executors.newFixedThreadPool(4);

    static class MyThread implements Callable{

        int start;
        int end;

        public MyThread(int start, int end) {
            this.start = start;
            this.end = end;
        }

        @Override
        public Object call() throws Exception {
            countDownLatch.countDown();
            int temp = 0;
            for (int i = start; i <= end; i++) {
                temp += i;
            }
            return temp;
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        int result = 0;
        for (int i = 1; i <= 4; i++) {
            result += (int) executorService.submit(new MyThread((i - 1)*25 + 1, i * 25)).get();
        }
        countDownLatch.await();
        System.out.println(result);
        executorService.shutdown();
    }

}
```

# 九、Java中的原子操作

## 9.1 实现方式

### 9.1.1 使用循环CAS

**JVM中的CAS操作利用处理器提供的CMPXCHG指令实现。自旋CAS实现的基本思路就是循环进行CAS操作直到成功为止。**

**CAS实现原子操作的三大问题**

> **ABA问题**

如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有变化，但实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加版本号，每次变量更新的时候把版本号加1，那么A->B->C就变成了1A->2B->3A。

从Java 1.5开始，JDK的Atomic包里提供了一个类`AtomicStampedReference`来解决ABA问题。这个类的compareAndSet方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。 

> **循环时间长开销大**

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

> **只能保证一个共享变量的原子操作**

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。

还有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如，有两个共享变量i＝2，j=a，合并一下ij=2a，然后用CAS来操作ij。从Java 1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。 

### 9.1.2 使用锁

锁机制保证了只有获得锁的线程才能操作锁定的内存区域。

**JVM实现锁的方式都用了循环CAS，即当一个线程想进入同步块的时候使用循环CAS的方式来获取锁，当它退出同步块的时候使用循环CAS释放锁。** 

## 9.2 Java中的原子类

![](http://mycsdnblog.work/201919201504-m.png)

### 9.2.1 原子更新基本类型

- AtomicBoolean：原子更新布尔类型。 
- AtomicInteger：原子更新整型。 
- AtomicLong：原子更新长整型。 

以AtomicInteger为例进行说明：

- int addAndGet（int delta）：以原子方式将输入的数值与实例中的值（AtomicInteger里的value）相加，并返回结果。 
- boolean compareAndSet（int expect，int update）：如果输入的数值等于预期值，则以原子方式将该值设置为输入的值。 
- int getAndIncrement()：以原子方式将当前值加1，注意，这里返回的是自增前的值。 
- void lazySet（int newValue）：最终会设置成newValue，使用lazySet设置值后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值。 
- int getAndSet（int newValue）：以原子方式设置为newValue的值，并返回旧值。 

```java
package com.example.base;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Author: 98050
 * @Time: 2019-07-20 15:09
 * @Feature:
 */
public class Test {
    static AtomicInteger atomicInteger = new AtomicInteger(1);

    public static void main(String[] args) {
        System.out.println(atomicInteger.getAndIncrement());

        System.out.println(atomicInteger.get());
    }

}
```

getAndIncrement的源码：

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

### 9.2.2 原子更新数组

- AtomicIntegerArray：原子更新整型数组里的元素。 
- AtomicLongArray：原子更新长整型数组里的元素。 
- AtomicReferenceArray：原子更新引用类型数组里的元素。 

以AtomicIntegerArray为例进行说明，其主要提供以下方法：

- int addAndGet（int i，int delta）：以原子方式将输入值与数组中索引i的元素相加。 
- boolean compareAndSet（int i，int expect，int update）：如果当前值等于预期值，则以原子方式将数组位置i的元素设置成update值。 

```java
package com.example.base;

import java.util.concurrent.atomic.AtomicIntegerArray;

/**
 * @Author: 98050
 * @Time: 2019-07-20 15:23
 * @Feature:
 */
public class Test2 {

    static int[] value = new int[]{1,2};

    static AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(value);

    public static void main(String[] args) {
        atomicIntegerArray.getAndAdd(0, 3);
        System.out.println(atomicIntegerArray.get(0));
        System.out.println(value[0]);
    }
}
```

输出：

```
4
1
```

需要注意的是，数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部的数组元素进行修改时，不会影响传入的数组。

```java
public final int getAndAdd(int i, int delta) {
    return unsafe.getAndAddInt(array, checkedByteOffset(i), delta);
}
```

### 9.2.3 原子更新引用类型

原子更新基本类型的AtomicInteger，只能更新一个变量，如果要原子更新多个变量，就需要使用这个原子更新引用类型提供的类。

AtomicReference：原子更新引用类型。 

AtomicReferenceFieldUpdater：原子更新引用类型里的字段。 

AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法是AtomicMarkableReference（V initialRef，boolean initialMark）。 

以AtomicReference为例进行说明：

```java
package com.example.base;

import java.util.concurrent.atomic.AtomicReference;

/**
 * @Author: 98050
 * @Time: 2019-07-20 15:30
 * @Feature:
 */
public class Test3 {

    static class User{
        private String name;
        private int age;

        public User(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }

    public static AtomicReference<User> atomicReference = new AtomicReference<User>();

    public static void main(String[] args) {
        User user = new User("张三", 11);
        atomicReference.set(user);

        User update = new User("李四", 14);
        atomicReference.compareAndSet(user, update);
        System.out.println(atomicReference.get());
    }
}
```

结果：

```
User{name='李四', age=14}
```

### 9.2.4 原子更新字段类

如果需原子地更新某个类里的某个字段时，就需要使用原子更新字段类。

AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。 

AtomicLongFieldUpdater：原子更新长整型字段的更新器。 

AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题。 

要想原子地更新字段类需要两步。第一步，因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。第二步，更新类的字段（属性）必须使用public volatile修饰符。

仅以AstomicIntegerFieldUpdater为例进行说明：

```java
package com.example.base;

import java.util.concurrent.atomic.*;

import static com.example.base.Test3.atomicReference;

/**
 * @Author: 98050
 * @Time: 2019-07-20 15:44
 * @Feature:
 */
public class Test4 {

    static class User{
        private String name;
        public volatile int age;

        public User(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }

    public static AtomicIntegerFieldUpdater<User> atomicReferenceFieldUpdater = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");

    public static void main(String[] args) {
        User user = new User("张三", 11);

        System.out.println(atomicReferenceFieldUpdater.getAndIncrement(user));
        System.out.println(user);

    }
}
```

结果：

```
11
User{name='张三', age=12}
```

# 十、Java内存模型

Java的并发采用的是共享内存模型，Java线程之间的通信总是隐式进行的。

## 10.1 Java内存模型的抽象结构

Java线程之间的通信由Java内存模型（JMM）控制，**JMM决定一个线程对共享变量的写入何时对另一个线程可见**。

从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的 一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优 化。Java内存模型的抽象示意如下图所示。

![](http://mycsdnblog.work/201919151410-N.png)

## 10.2 重排序

重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段。

as-if-serial语义：不管怎么重排序，程序的执行结果不能被改变。

## 10.3 happens-before 

happens-before来指定两个操作之间的执行顺序

一个线程中的每个操作，happens-before于该线程中的任意后续操作

对一个锁的解锁，happens-before于随后对这个锁的加锁

两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行

happens-before仅仅要求前一个操作的执行结果对后一个操作可见，且前一个操作按顺序排在第二个操作之前。

## 10.4 线程安全的延迟初始化

### 10.4.1 基于volatile的解决方案

```java
public class Singleton{
    private static volatile Singleton instance;
    
    private Singleton(){
        
    }
    
    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### 10.4.2 基于类初始化的解决方案

JVM在类初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。

```java
public class Singleton{
    private static class SingletonHolder{
        public static Singleton instance = new Singleton();
    }
    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }
}
```

### 10.4.3 比较

基于volatile的双重校验锁除了对静态字段实现延迟初始化外，还可以对实例字段实现延迟初始化

字段延迟初始化降低了初始化类或创建实例的开销，但增加了访问被延迟初始化的字段的开销。

**普通字段延迟初始化使用基于volatile的解决方案**

**静态字段使用基于类初始化的解决方案**

## 10.5 锁的内存语义

### 10.5.1 锁的释放和获取内存语义

当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主存中。

当线程获取锁时，JMM会把该线程对应的本地内存置为无效，从而使得被监视器保护的临界区代码必须从主存中读取共享变量。

### 10.5.2 锁内存语义的实现

- 利用volatile变量的写-读所具有的内存语义

- 利用CAS所附带的volatile读和volatile写的内存语义

# 十一、从JVM的角度看Java线程的创建与运行

## 11.1 线程的创建

```java
        Thread thread = new Thread(() -> System.out.println(Thread.currentThread().getName()));
        thread.start();
```

### 11.1.1 构造方法

```java
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
```

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    this.name = name;

    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    if (g == null) {
        /* Determine if it's an applet or not */

        /* If there is a security manager, ask the security manager
           what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }

        /* If the security doesn't have a strong opinion of the matter
           use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* checkAccess regardless of whether or not threadgroup is
       explicitly passed in. */
    g.checkAccess();

    /*
     * Do we have the required permissions?
     */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();

    this.group = g;
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext =
            acc != null ? acc : AccessController.getContext();
    this.target = target;
    setPriority(priority);
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID();
}
```

主要工作为：

1. 设置线程的名字，一般情况下默认名字格式为“Thread-X”，X是一个数字，代表用这个线程类初始化的第X个实例
2. 设置线程的ThreadGroup
3. 设置此线程是否是守护线程
4. 设置线程的优先级为父线程的优先级（子线程的优先级是不能大于父线程的）
5. 设置此线程的类加载器
6. 设置这个Thread的target，这个target就是携带了我们写好的run()方法的Runnable对象
7. 设置stackSize
8. 设置线程ID

### 11.1.2 Thread的run方法

```java
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

- 如果我们是通过传递Runnable对象来初始化Thread实例的话，那么就是将实例的run方法“替换”为Runnable对象的run()方法（逻辑上的替换）
- 如果我们的Thread继承类重写了run()方法，也就是说原本的“替换”逻辑被覆盖了，这时我们再通过传递Runnable对象来初始化Thread实例的话，这个实例的run不会再被替换，也就是这个实例仅会执行我们重写的run方法。

## 11.2 线程的开始

开始一个线程就是调用这个线程的start方法：

```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
```

第一步，非常重要，就是先检查线程的状态，如果线程的状态不是NEW状态的话，那么就会抛出一个异常（关于Java线程的状态。

第二步，将这个Thread添加到之前引用的ThreadGroup中

第三步，调用native start0()

> 需要说明一下start和run的区别：
>
> start会真正的开启一个线程，直接调用run方法是不会开始线程的，只是普通的方法调用

```java
package test;

/**
 * @Author: 98050
 * @Time: 2019-07-16 16:54
 * @Feature:
 */
public class Test {

    public static void main(String[] args) {
        Mythread mythread = new Mythread();
        mythread.run();

        A a = new A();
        a.test();

        System.out.println("------------调用start----------------");
        mythread.start();
    }

    public static class Mythread extends Thread{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
            System.out.println("线程开始");
        }
    }

    public static class A{

        public void test() {
            System.out.println(Thread.currentThread().getName());
            System.out.println("线程开始");
        }
    }
}
```

结果：

![](http://mycsdnblog.work/201919161711-6.png)

## 11.3 Thread类中的native方法

```java
private native void start0();
```

在Thread.java里面，有一个registerNatives()的native方法，并且在Threa.java的第一个static块中就调用了这个方法，保证这个方法在类加载中是第一个被调用的方法。这个native方法的作用是为其他native方法注册到JVM中（可见：[JVM查找java native方法的规则](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fxyang81%2Farticle%2Fdetails%2F41854185)）。
 通过这个方法，我们可以找到Thrad.java中native方法在JVM中对应方法：
 [Thread.c$Java_java_lang_Thread_registerNatives](https://link.jianshu.com?t=http%3A%2F%2Fhg.openjdk.java.net%2Fjdk6%2Fjdk6%2Fjdk%2Ffile%2F5f4bfda58ef8%2Fsrc%2Fshare%2Fnative%2Fjava%2Flang%2FThread.c%23l65)

```c++
static JNINativeMethod methods[] = {
    {"start0",           "()V",        (void *)&JVM_StartThread},
    {"stop0",            "(" OBJ ")V", (void *)&JVM_StopThread},
    {"isAlive",          "()Z",        (void *)&JVM_IsThreadAlive},
    {"suspend0",         "()V",        (void *)&JVM_SuspendThread},
    {"resume0",          "()V",        (void *)&JVM_ResumeThread},
    {"setPriority0",     "(I)V",       (void *)&JVM_SetThreadPriority},
    {"yield",            "()V",        (void *)&JVM_Yield},
    {"sleep",            "(J)V",       (void *)&JVM_Sleep},
    {"currentThread",    "()" THD,     (void *)&JVM_CurrentThread},
    {"countStackFrames", "()I",        (void *)&JVM_CountStackFrames},
    {"interrupt0",       "()V",        (void *)&JVM_Interrupt},
    {"isInterrupted",    "(Z)Z",       (void *)&JVM_IsInterrupted},
    {"holdsLock",        "(" OBJ ")Z", (void *)&JVM_HoldsLock},
    {"getThreads",        "()[" THD,   (void *)&JVM_GetAllThreads},
    {"dumpThreads",      "([" THD ")[[" STE, (void *)&JVM_DumpThreads},
};
JNIEXPORT void JNICALL
Java_java_lang_Thread_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls, methods, ARRAY_LENGTH(methods));
}
```

基本概念：

**java.lang.Thread:** 这个是Java语言里的线程类，由这个Java类创建的instance都会 1:1 映射到一个操作系统的osthread。

**JavaThread:** JVM中C++定义的类，一个JavaThread的instance代表了在JVM中的java.lang.Thread的instance, 它维护了线程的状态，并且维护一个指针指向java.lang.Thread创建的对象(oop)。它同时还维护了一个指针指向对应的OSThread，来获取底层操作系统创建的osthread的状态。

**OSThread:** JVM中C++定义的类，代表了JVM中对底层操作系统的osthread的抽象，它维护着实际操作系统创建的线程句柄handle，可以获取底层osthread的状态

 **VMThread:** JVM中C++定义的类，这个类和用户创建的线程无关，是JVM本身用来进行虚拟机操作的线程，比如GC

补充：需要注意的是osthread和OSThread的区别，简单的说，osthread是操作系统级别的线程，也就是真正意义上的线程，它的具体实现由操作系统完成；而OSThread则是关于osthread的抽象，通过操作系统暴露出来的接口（句柄）对osthread进行操作

在实际的运行中，OSThread会根据所运行的操作系统（平台）来创建，也就是说，如果字节码运行的平台是Linux的话，那么创建出来的OSThread就是Linux的OSThread；如果是Windows的话，那么创建出来的就是Windows的OSThread

## 11.4 整个创建流程概述

整个过程大概是：

1. 在Java中，使用java.lang.Thread的构造方法来构建一个java.lang.Thread对象，此时只是对这个对象的部分字段(例如线程名，优先级等)进行初始化；
2. 调用java.lang.Thread对象的start()方法，开始此线程。此时，在start()方法内部，调用start0() 本地方法来开始此线程；
3. start0()在VM中对应的是JVM_StartThread，也就是，在VM中，实际运行的是JVM_StartThread方法(宏),在这个方法中，创建了一个JavaThread对象；
4. 在JavaThread对象的创建过程中，会根据运行平台创建一个对应的OSThread对象，且JavaThread保持这个OSThread对象的引用；
5. 在OSThread对象的创建过程中，创建一个平台相关的底层级线程，如果这个底层级线程失败，那么就抛出异常；
6. 在正常情况下，这个底层级的线程开始运行，并执行java.lang.Thread对象的run方法；
7. 当java.lang.Thread生成的Object的run()方法执行完毕返回后，或者抛出异常终止后，终止native thread;
8. 最后就是释放相关的资源(包括内存、锁等)

在上述过程，穿插着各种的判断检测，其中很大一部分都是关于各种层次下的线程的状态的检测，在JVM中，无论哪种层次的线程，都只允许执行一次。

