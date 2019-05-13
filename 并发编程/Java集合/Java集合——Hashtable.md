### 一、介绍

和HashMap一样，Hashtable 也是一个**散列表**，它存储的内容是**键值对(key-value)映射**。
Hashtable **继承于Dictionary**，实现了Map、Cloneable、java.io.Serializable接口。
Hashtable 的函数都是**同步的**，这意味着它是线程安全的。它的key、value都不可以为null。此外，Hashtable中的映射不是有序的。

Hashtable 的实例有两个参数影响其性能：**初始容量** 和 **加载因子**。

容量 是哈希表中桶 的数量，初始容量 就是哈希表创建时的容量。注意，哈希表的状态为 open：在发生“哈希冲突”的情况下，单个桶会存储多个条目，这些条目必须按顺序搜索。

加载因子 是对哈希表在其容量自动增加之前可以达到多满的一个尺度。初始容量和加载因子这两个参数只是对该实现的提示。关于何时以及是否调用 rehash 方法的具体细节则依赖于该实现。
通常，**默认加载因子是 0.75**, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查找某个条目的时间（在大多数 Hashtable 操作中，包括 get 和 put 操作，都反映了这一点）。

### 二、数据结构

#### 2.1 Hashtable的继承关系

![](F:\images/201818271546-M.png)

#### 2.2 Hashtable结构图

![](F:\images/201818271545-K.png)

从图中可以看出：  

<1>Hashtable继承于Dictionary类，实现了Map接口。Map是"key-value键值对"接口，Dictionary是声明了操作"键值对"函数接口的抽象类。  

<2>Hashtable是通过"拉链法"实现的哈希表。它包括几个重要的成员变量:table, count, threshold, loadFactor, modCount。 　　

- table是一个Entry[]数组类型，而Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。  　　
- count是Hashtable的大小，它是Hashtable保存的键值对的数量。  　　
- threshold是Hashtable的阈值，用于判断是否需要调整Hashtable的容量。threshold的值="容量*加载因子"。
- loadFactor就是加载因子。  　　
- modCount是用来实现fail-fast机制的 

### 三、源码解析（JDK 1.8）

#### 3.1 Hashtable的“拉链法”

> **Hashtable数据存储数组**

![](F:\images/201818271608-l.png)

> **Entry数据结构**

```java
private static class Entry<K,V> implements Map.Entry<K,V> {
    // 哈希值
    int hash;
    K key;
    V value;
    // 指向的下一个Entry，即链表的下一个节点
    Entry<K,V> next;

    // 构造函数
    protected Entry(int hash, K key, V value, Entry<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    protected Object clone() {
        return new Entry<K,V>(hash, key, value,
              (next==null ? null : (Entry<K,V>) next.clone()));
    }

    public K getKey() {
        return key;
    }

    public V getValue() {
        return value;
    }

    // 设置value。若value是null，则抛出异常。
    public V setValue(V value) {
        if (value == null)
            throw new NullPointerException();

        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    // 覆盖equals()方法，判断两个Entry是否相等。
    // 若两个Entry的key和value都相等，则认为它们相等。
    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;

        return (key==null ? e.getKey()==null : key.equals(e.getKey())) &&
           (value==null ? e.getValue()==null : value.equals(e.getValue()));
    }

    public int hashCode() {
        return hash ^ (value==null ? 0 : value.hashCode());
    }

    public String toString() {
        return key.toString()+"="+value.toString();
    }
}
```

Entry 实际上就是一个单向链表。Hashtable是通过拉链法解决哈希冲突的。 Entry 实现了Map.Entry 接口，即实现getKey(), getValue(), setValue(V value), equals(Object o), hashCode()这些函数。这些都是基本的读取/修改key、value值的函数。 

#### 3.2 Hashtable的构造函数

```java
// 默认构造函数。
public Hashtable() {
    // 默认构造函数，指定的容量大小是11；加载因子是0.75
    this(11, 0.75f);
}

// 指定“容量大小”的构造函数
public Hashtable(int initialCapacity) {
    this(initialCapacity, 0.75f);
}

// 指定“容量大小”和“加载因子”的构造函数
public Hashtable(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);

    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    table = new Entry[initialCapacity];
    threshold = (int)(initialCapacity * loadFactor);
}

// 包含“子Map”的构造函数
public Hashtable(Map<? extends K, ? extends V> t) {
    this(Math.max(2*t.size(), 11), 0.75f);
    // 将“子Map”的全部元素都添加到Hashtable中
    putAll(t);
}
```

#### 3.3 put操作

![](F:\images/201818271617-K.png)

- 使用synchronized关键字进行加锁
- value不能为空
- 计算hash值，直接通过key.hashcode() 得到，所以当key为null时，调用hashcode() 会出错。所以Hashtable的key不能为null，而HashMap的key值可以为空主要是因为计算hash值时，如果key为null，会返回0。
- Hashtable中对hash值进行寻址的方法为hash%数组长度(与HashMap不同，所以不要求数组长度必须为2的n次方)。 
- 得到下标为index的桶内第一个元素。
- 遍历table[index]所连接的链表，查找是否已经存在key与需要插入的key值相同的节点，如果存在则直接更新value，并返回旧的value。 
- 如果table[index]所连接的链表上不存在相同的key,则通过addEntry()方法将新节点加载链表的开头。 

![](F:\images/201818271638-M.png)

- 如果加入新节点后，hashtable中元素的个数超过了阈值threshold，则利用rehash()对数组进行扩容。 
- e=table[index]，并将新节点加在了e节点的1前面，最后table[index]=e。相当于把把新节点放在table[index]位置，**即整个链表的首部，头插法。**  

#### 3.4 get操作

![](F:\images/201818271615-e.png)

- 使用synchronized关键字进行加锁
- 计算数组下标
- 遍历table[index]链表，找到key值相同的节点的value返回，注意同样在该过程中使用到了key的equal方法。

#### 3.5 rehash()

![](F:\images/201818271641-9.png)

在这个rehash()方法中，容量扩大两倍+1，同时需要将原来HashTable中的元素都复制到新的HashTable中，这个过程是比较消耗时间的，需要重新计算hash值的。HashMap中扩容时不需要重新计算hash值是因为initialCapacity是2的幂次方，只需通过e.hash & oldCap就可以将链表拆分。

![](F:\images/201818271706-2.png)

### 四、遍历方式

#### 4.1 遍历键值对

```java
// 假设table是Hashtable对象
// table中的key是String类型，value是Integer类型
Integer integ = null;
Iterator iter = table.entrySet().iterator();
while(iter.hasNext()) {
    Map.Entry entry = (Map.Entry)iter.next();
    // 获取key
    key = (String)entry.getKey();
        // 获取value
    integ = (Integer)entry.getValue();
}
```

#### 4.2 遍历Hashtable的键

```java
// 假设table是Hashtable对象
// table中的key是String类型，value是Integer类型
String key = null;
Integer integ = null;
Iterator iter = table.keySet().iterator();
while (iter.hasNext()) {
        // 获取key
    key = (String)iter.next();
        // 根据key，获取value
    integ = (Integer)table.get(key);
}
```

```java
Enumeration enu = table.keys();
while(enu.hasMoreElements()) {
    System.out.println(enu.nextElement());
}   
```

#### 4.3 遍历Hashtable的值

```java
// 假设table是Hashtable对象
// table中的key是String类型，value是Integer类型
Integer value = null;
Collection c = table.values();
Iterator iter= c.iterator();
while (iter.hasNext()) {
    value = (Integer)iter.next();
}
```

```java
Enumeration enu = table.elements();
while(enu.hasMoreElements()) {
    System.out.println(enu.nextElement());
}
```

### 五、Hashtable与HashMap的比较

- **HashMap中key和value均可以为null，但是Hashtable中key和value均不能为null。**
- **HashMap采用的是数组(桶位)+链表+红黑树结构实现，而Hashtable中采用的是数组(桶位)+链表实现。**

- **HashMap中出现hash冲突时，如果链表节点数小于8时是将新元素加入到链表的末尾，而Hashtable中出现hash冲突时采用的是将新元素加入到链表的开头。**

- **HashMap中数组容量的大小要求是2的n次方，如果初始化时不符合要求会进行调整，而Hashtable中数组容量的大小可以为任意正整数。**

- **HashMap中的寻址方法采用的是位运算按位与，而HashTable中寻址方式采用的是求余数。**
- **HashMap不是线程安全的，而Hashtable是线程安全的，Hashtable中的get和put方法均采用了synchronized关键字进行了方法同步。**

- **HashMap中默认容量的大小是16，而Hashtable中默认数组容量是11。**

### 六、示例

**代码：**

```java
package com.util.hashtable;

import java.util.Hashtable;
import java.util.Iterator;
import java.util.Map;

/**
 * @Author: 98050
 * @Time: 2018-12-27 15:42
 * @Feature:
 */
public class Test {

    public static void main(String[] args) throws InterruptedException {
        final Hashtable<String,String> hashtable = new Hashtable();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    hashtable.put(Thread.currentThread().getName() + "放入" + i, i +"");
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
                    hashtable.put(Thread.currentThread().getName() + "放入" + i, i +"");
                }
            }
        }).start();

        Thread.sleep(1000);
        System.out.println("hashtable的大小：" + hashtable.size());
        Iterator iterator = hashtable.entrySet().iterator();
        while(iterator.hasNext()) {
            Map.Entry entry = (Map.Entry)iterator.next();
            // 获取key和value
            System.out.println("key：" + entry.getKey() + ", value：" + entry.getValue());
        }
    }
}
```

**结果：**

![](F:\images/201818271745-e.png)