### 一、介绍

- LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
- LinkedList 实现 List 接口，能对它进行队列操作。
- LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
- LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
- LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
- LinkedList 是非同步的。

### 二、数据结构

![](F:\images/201919032139-Y.png)

**LinkedList的本质是双向链表。** 

(01) LinkedList继承于AbstractSequentialList，并且实现了Dequeue接口。  

(02) LinkedList包含两个重要的成员：first和 last。 　　

- **first指向链表的头**
- **last指向链表的尾**

其中Node是LinkedList的内部类，是一个双向链表。

### 三、源码解析

LinkedList实际上是通过双向链表去实现的。既然是双向链表，那么它的**顺序访问会非常高效，而随机访问效率比较低**。

既然LinkedList是通过双向链表的，但是它也实现了List接口；也就是说，它实现了get(int location)、remove(int location)等根据索引值来获取、删除节点的函数。LinkedList是如何实现List的这些接口的，如何将“**双向链表和索引值联系起来的**”？实际原理非常简单，它就是通过一个**计数索引值**来实现的。

例如，当我们调用get(int location)时，首先会比较“双向链表长度的1/2”和“location”；若前者大，则从链表头开始往后查找，直到location位置；否则，从链表末尾开始先前查找，直到location位置。

这就是双线链表和索引值联系起来的方法。

#### 3.1 添加

> **add(E e)**

实际调用linkLast(e)方法，代码如下：

![](F:\images/201919032149-k.png)

首先获取当前链表的最后一个节点last，然后创建一个以尾节点为前驱的新节点，然后将尾节点指向新节点。

如果链表为空，那么这个新节点既是头节点也是尾节点；如果不为空的话，将新节点插入到原来链表的尾部。

> **add(int index, E element)**

向指定位置插入元素

![](F:\images/201919032204-Q.png)

首先检查索引是否在[0~size]之间。

如果index和size相等，那么直接调用linkedLast方法把该节点插入到链表尾部；否则通过node方法找到index位置上的节点，然后调用linkBefore方法：

![](F:\images/201919032208-t.png)

**目的很明确，就是将新节点插入到原来index位置上元素的前一个位置。**先找到原来index位置上的元素的前一个节点pred，然后创建一个以这个节点为前驱，以succ为后继的新节点，最后把节点插入即可。

#### 3.2 删除

> **remove()**

![](F:\images/201919032215-Z.png)

实际调用的是removeFirst方法，即删除链表第一个元素：

![](F:\images/201919032217-A.png)

获取链表的头节点，然后判断其是否为空，不为空的话调用unlinkFirst来删除节点：

![](F:\images/201919032217-j.png)

先获取被删节点中保存的值以便返回，然后获取被删节点的下一个节点next，同时将被删节点的数据域置为空，再断掉被节点和next的指针；最后再让头指针指向next。如果此时next为null，说明被删的节点是链表中唯一的一个节点，删除完毕后需要把尾节点指向null；否则让next的前驱指向头节点null。

> **remove(int index)** 

删除指定位置上的元素：

![](F:\images/201919032226-d.png)

先检查删除位置是否合法，然后调用unlink方法，参数就是被删节点（通过node方法获取）：

![](F:\images/201919032227-y.png)

首先进行准备工作：保存被删节点的数据域以便返回，然后得到被删节点的前驱和后继节点；

如果前驱节点为null，则说明删除的是链表的第一个节点，那么让头节点指向next即可；否则让x的前驱节点指向x的后继节点，并且让x的前驱置为空。

如果后继节点为null，那么删除的就是链表最后一个节点，让位指针指向x的前一个节点；否则改变x的后继节点的前驱为x的前驱节点，再把x的后继指针置为空。

整体都比较简单，只要学过数据结构，这一块就很容易理解。

**LinkedList可以作为FIFO的队列：通过add()和removeFirst()方法可以实现**

**LinkedList可以作为LIFO的栈：通过add()和removeLast()方法可以实现**

### 四、遍历方式

#### 4.1 通过迭代器

```java
for(Iterator iter = list.iterator(); iter.hasNext();)
    iter.next();
```

#### 4.2 通过随机访问

```java
int size = list.size();
for (int i=0; i<size; i++) {
    list.get(i);        
}
```

#### 4.3 使用for循环

```java
for (Integer integ:list) 
    ;
```

#### 4.4 通过pollFirst()

```java
while(list.pollFirst() != null)
    ;
```

#### 4.4 通过pollLast()

```java
while(list.pollLast() != null)
    ;
```

#### 4.5 通过removeFirst()

```java
try {
    while(list.removeFirst() != null)
        ;
} catch (NoSuchElementException e) {
}
```

#### 4.6 通过removeLast()

```java
try {
    while(list.removeLast() != null)
        ;
} catch (NoSuchElementException e) {
}
```

### 五、模拟栈和队列

```java
package com.util.linkedlist;

import java.util.LinkedList;
import java.util.NoSuchElementException;

/**
 * @Author: 98050
 * @Time: 2019-01-03 21:36
 * @Feature: 模拟栈和队列
 */
public class Test {

    public static void main(String[] args) {
        MyQueue();
        MyStack();
    }

    public static void MyStack(){
        LinkedList<Integer> linkedList = new LinkedList<Integer>();
        linkedList.add(1);
        linkedList.add(2);
        linkedList.add(3);
        linkedList.add(4);
        try {
            while (true) {
                Integer result = linkedList.removeLast();
                System.out.println(result);
                if (result == null){
                    break;
                }
            }
        }catch (NoSuchElementException e){

        }
    }

    public static void MyQueue(){
        LinkedList<Integer> linkedList = new LinkedList<Integer>();
        linkedList.add(1);
        linkedList.add(2);
        linkedList.add(3);
        linkedList.add(4);
        try {
            while (true) {
                Integer result = linkedList.removeFirst();
                System.out.println(result);
                if (result == null){
                    break;
                }
            }
        }catch (NoSuchElementException e){

        }
    }
}

```

**结果：**

![](F:\images/201919032311-M.png)

### 六、线程安全问题

LinkedList是非线程安全的，例子如下：

```java
package com.util.linkedlist;

import java.util.LinkedList;

/**
 * @Author: 98050
 * @Time: 2019-01-03 23:12
 * @Feature: 非线程安全
 */
public class Test2 {

    public static void main(String[] args) throws InterruptedException {
        final LinkedList<String> linkedList = new LinkedList<String>();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 30; i < 40; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    linkedList.add(Thread.currentThread().getName()+"放入" + i);
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 40; i < 50; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    linkedList.add(Thread.currentThread().getName()+"放入" + i);
                }
            }
        }).start();
        Thread.sleep(1000);
        System.out.println("list的大小：" + linkedList.size());
        for (String s : linkedList){
            System.out.println(s);
        }
    }
}
```

![](F:\images/201919032315-l.png)

****

