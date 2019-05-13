### 一、介绍

ConcurrentLinkedQueue是线程安全的队列，它适用于“高并发”的场景。
它是一个基于链接节点的无界线程安全队列，按照 FIFO（先进先出）原则对元素进行排序。队列元素中不可以放置null元素（内部实现的特殊节点除外）。**该队列是一个无界且线程安全的非阻塞队列**。

### 二、原理和数据结构

![](http://mycsdnblog.work/201919022209-G.png)

**说明**： 

1. ConcurrentLinkedQueue继承于AbstractQueue。 

2. ConcurrentLinkedQueue内部是通过链表来实现的。它同时包含链表的头节点head和尾节点tail。ConcurrentLinkedQueue按照 FIFO（先进先出）原则对元素进行排序。元素都是从尾部插入到链表，从头部开始返回。 

3. ConcurrentLinkedQueue的链表Node中的next的类型是volatile，而且链表数据item的类型也是volatile。

   ![](http://mycsdnblog.work/201919022214-P.png)

   关于volatile，它的语义包含：“即对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入”。ConcurrentLinkedQueue就是通过volatile来实现多线程对竞争资源的互斥访问的。 

**CoucurrentLinkedQueue规定了如下几个不变性：**

1. 在入队的最后一个元素的next为null
2. 队列中所有未删除的节点的item都不能为null且都能从head节点遍历到
3. 对于要删除的节点，不是直接将其设置为null，而是先将其item域设置为null（迭代器会跳过item为null的节点）
4. 允许head和tail更新滞后。这是什么意思呢？意思就说是head、tail不总是指向第一个元素和最后一个元素。

**head的不变性和可变性：**

> 不变性 

- 所有未删除的节点都可以通过head节点遍历到
- head不能为null
- head节点的next不能指向自身

> 可变性 

- head的item可能为null，也可能不为null 
- 允许tail滞后head，也就是说调用succc()方法，从head不可达tail

**tail的不变性和可变性：**

> 不变性 

- tail不能为null

> 可变性 

- tail的item可能为null，也可能不为null
- tail节点的next域可以指向自身 
- 允许tail滞后head，也就是说调用succc()方法，从head不可达tail

### 三、方法列表

```java
// 创建一个最初为空的 ConcurrentLinkedQueue。
ConcurrentLinkedQueue()
// 创建一个最初包含给定 collection 元素的 ConcurrentLinkedQueue，按照此 collection 迭代器的遍历顺序来添加元素。
ConcurrentLinkedQueue(Collection<? extends E> c)

// 将指定元素插入此队列的尾部。
boolean add(E e)
// 如果此队列包含指定元素，则返回 true。
boolean contains(Object o)
// 如果此队列不包含任何元素，则返回 true。
boolean isEmpty()
// 返回在此队列元素上以恰当顺序进行迭代的迭代器。
Iterator<E> iterator()
// 将指定元素插入此队列的尾部。
boolean offer(E e)
// 获取但不移除此队列的头；如果此队列为空，则返回 null。
E peek()
// 获取并移除此队列的头，如果此队列为空，则返回 null。
E poll()
// 从队列中移除指定元素的单个实例（如果存在）。
boolean remove(Object o)
// 返回此队列中的元素数量。
int size()
// 返回以恰当顺序包含此队列所有元素的数组。
Object[] toArray()
// 返回以恰当顺序包含此队列所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。<T> T[] toArray(T[] a)
```

### 四、源码分析

从创建、添加、删除三个方面来分析。

#### 4.1 创建

![](http://mycsdnblog.work/201919022220-H.png)

在构造函数中，新建了一个“内容为null的节点”，并设置表头head和表尾tail的值为新节点。 

head和tail都是volatile类型：

![](http://mycsdnblog.work/201919022221-4.png)

Node的声明如下： 

```java
private static class Node<E> {
    volatile E item;
    volatile Node<E> next;

    Node(E item) {
        UNSAFE.putObject(this, itemOffset, item);
    }

    boolean casItem(E cmp, E val) {
        return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
    }

    void lazySetNext(Node<E> val) {
        UNSAFE.putOrderedObject(this, nextOffset, val);
    }

    boolean casNext(Node<E> cmp, Node<E> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }

    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
    private static final long itemOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class k = Node.class;
            itemOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("item"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

Node是个单向链表节点，next用于指向下一个Node，item用于存储数据。**Node中操作节点数据的API，都是通过Unsafe机制的CAS函数实现的；例如casNext()是通过CAS函数“比较并设置节点的下一个节点”。** 

#### 4.2 添加

![](http://mycsdnblog.work/201919022225-D.png)

add()实际上是调用的offer()来完成添加操作的。由于ConcurrentLinkedQueue是无界的，所以offer永远返回true，不能通过返回值来判断是否入队成功。  offer的源码：

![](http://mycsdnblog.work/201919022226-c.png)

首先判断e是否为null，是的话抛出NullPointerException异常；然后创建一个新节点newNode，遍历链表，找到链表的末尾；最后将newNode插入。关键就在遍历过程（for循环），其内部分为三种情况来处理：

> **情况1（q == null）**

这意味着p节点就是链表的尾节点，将newNode插入到其后即可。插入的过程通过CAS保证插入的安全性。CAS的具体操作：

- 如果“p的下一个节点为null”(即p为尾节点)，则设置p的下一个节点为newNode。            
- 如果该CAS操作成功的话，则比较“p和t”(若p不等于t，则设置newNode为新的尾节点)，然后返回true。            
- 如果该CAS操作失败，这意味着“其它线程对尾节点进行了修改”，则重新循环。

> **情况2（p==q）**

p == q 代表着该节点已经被删除了

由于多线程的原因，offer()的时候也会poll()，如果offer()的时候正好该节点已经poll()了

那么在poll()方法中的updateHead()方法会将head指向当前的q，而把p.next指向自己，即：p.next == p

这样就会导致tail节点滞后head（tail位于head的前面），则需要重新设置p

> **情况3**

tail没有指向尾节点

`p = (p != t && t != (t = tail)) ? t : q`将其转换为如下代码：

```java
if (p==t) {
    p = q;
} else {
    Node<E> tmp=t;
    t = tail;
    if (tmp==t) {
        p=q;
    } else {
        p=t;
    }
}
```

判断tail节点有没有被更新，如果没被更新，p=q：p指向p.next继续寻找尾节点；如果被更新了，p=t，p赋值为新的tail节点。
什么情况下p!=t只有本分支和else if (p == q)分支含有更新变量p和t的语句，所以在p!=t出现之前已经循环过这两个分支至少一次。

**添加过程**：

<1>ConcurrentLinkedQueue初始化

![](http://mycsdnblog.work/201919031055-g.png)

<2>添加元素1

第一次插入元素1，head = tail ，所以q = p.next ，q= null，进入情况1的if中：p.casNext(null, newNode)，由于 p == t成立，所以不会执行步骤：casTail(t, newNode)，直接return。插入1节点后如下：

![](http://mycsdnblog.work/201919031056-q.png)

<3>添加元素2

q = p.next 指向1节点 ，p = tail 指向null，所以直接跳到情况3：`p = (p != t && t != (t = tail)) ? t : q`，让p向后移动一个节点；然后进行第二次循环 q = p.next = null，此时就又回到了情况1中，将该节点插入;又因为现在的p指向1节点，而t = tail指向的还是头节点，所以p != t 成立，执行casTail(t, newNode)，让tail指向新插入的节点，最后return。如下： 

![](http://mycsdnblog.work/201919031110-A.png)

<4>添加元素3

和添加1的时候一致。

![](http://mycsdnblog.work/201919031112-t.png)

**注意**：当多个线程同时进行入队时，如果有一个线程正在入队，那么它必须先获取尾节点，然后设置尾节点的下一个节点为入队节点，但这时可能有另外一个线程插队了，那么队列的尾节点就会发生变化，这时当前线程要暂停入队操作，然后重新获取尾节点。 

#### 4.3 删除

```java
public E poll() {
    // 设置“标记”
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;

            // 情况1
            // 表头的数据不为null，并且“设置表头的数据为null”这个操作成功的话;
            // 则比较“p和h”(若p!=h，即表头发生了变化，则更新表头，即设置表头为p)，然后返回原表头的item值。
            if (item != null && p.casItem(item, null)) {
                if (p != h) // hop two nodes at a time
                    updateHead(h, ((q = p.next) != null) ? q : p);
                return item;
            }
            // 情况2
            // 表头的下一个节点为null，即链表只有一个“内容为null的表头节点”。则更新表头为p，并返回null。
            else if ((q = p.next) == null) {
                updateHead(h, p);
                return null;
            }
            // 情况3
            // 当一个线程在poll的时候，另一个线程已经把当前的p从队列中删除——将p.next = p，p已经被移除不能继续，需要重新开始。
            else if (p == q)
                continue restartFromHead;
            // 情况4
            // 设置p为q
            else
                p = q;
        }
    }
}
```

poll()的作用就是删除链表的表头节点，并返回被删节点对应的值。

![](http://mycsdnblog.work/201919031112-t.png)

**poll上边的链表**

<1>poll 1

head = null，p = head， item = p.item = null，情况1不成立，情况2：(q = p.next) == null不成立，p.next = 1，跳到情况4进入下一个循环；此时p = 1，所以情况1 item != null成立，进行p.casItem(item, null)成功，此时p  != h（p指向1节点），所以执行updateHead(h, ((q = p.next) != null) ? q : p)，q = p.next指向2节点不为空，所以将head CAS更新成2，如下： 

![](http://mycsdnblog.work/201919031200-3.png)

<2>poll 2

head = 2 ， p = head = 2，item = p.item = 2，情况1成立，但此时p==h，所以直接return，如下： 

![](http://mycsdnblog.work/201919031203-N.png)

<3>poll 3

head = null ，p = head = null，tiem = p.item = null，情况1不成立，跳到情况2：(q = p.next) == null，不成立，然后跳到情况4，让p向后移动一位，进入下一次循环；此时，p = q = 3，item = 3，情况1成立，所以将3设置为null，且p != h成立，执行updateHead(h, ((q = p.next) != null) ? q : p)，如下：

![](http://mycsdnblog.work/201919031208-B.png)

> **分析offer中的情况2，p == q**

ConcurrentLinkedQueue中规定，p == q表明，该节点已经被删除了，也就说tail滞后于head，head无法通过遍历找到tail，怎么做？ p = (t != (t = tail))? t : head

```java
Node<E> tmp = t;
t = tail;
if(temp != t){
    p = t;
}else{
    p = head;
}
```

这段代码主要是来判读tail节点是否已经发生了改变，如果发生了改变，则说明tail已经重新定位了，只需要重新找到tail即可，否则就只能指向head了。

<1>在上边的链表中继续插入4

则p = tail，q = p.next = null(p)，即满足情况2，p==q，然后重新给p赋值为head，因为此时tail没有改变；进入下一次循环，此时的p=head，q=p.next=null，进入情况1中执行， q == null且 p != t ，所以执行casTail(t, newNode)，tail重定位。如下： 

![](http://mycsdnblog.work/201919031241-i.png)

<2>继续插入5

q=p.next=null，进入情况1中执行，但是此时p==t，所以tail不进行重定位，如下：

![](http://mycsdnblog.work/201919031242-K.png)

### 五、使用

```java
package com.juc.concurrentlinkedqueue;

import java.util.Iterator;
import java.util.concurrent.ConcurrentLinkedQueue;

/**
 * @Author: 98050
 * @Time: 2019-01-02 22:08
 * @Feature:
 */
public class Test {

    public static void main(String[] args) throws InterruptedException {
        final ConcurrentLinkedQueue<String> concurrentLinkedQueue = new ConcurrentLinkedQueue<String>();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    concurrentLinkedQueue.add(Thread.currentThread().getName()+"放入" + i);
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 10; i < 20; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    concurrentLinkedQueue.add(Thread.currentThread().getName()+"放入" + i);
                }
            }
        }).start();

        Thread.sleep(1000);
        Iterator iterator = concurrentLinkedQueue.iterator();
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }
    }
}
```

**结果：**

![](http://mycsdnblog.work/201919022332-D.png)