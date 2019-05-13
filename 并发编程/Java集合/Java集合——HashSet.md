#### 一、介绍

HashSet 是一个**没有重复元素的集合**。 

它是由HashMap实现的，**不保证元素的顺序**，而且**HashSet允许使用 null 元素**。 

HashSet是**非同步的**。如果多个线程同时访问一个HashSet，而其中至少一个线程修改了该 set，那么它必须 保持外部同步。这通常是通过对自然封装该 set 的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSet 方法来“包装” set。最好在创建时完成这一操作，以防止对该 set 进行意外的不同步访问： 

```java
Set s = Collections.synchronizedSet(new HashSet(...));
```

> **HashSet的构造函数**

```java
// 默认构造函数
public HashSet() 

// 带集合的构造函数
public HashSet(Collection<? extends E> c) 

// 指定HashSet初始容量和加载因子的构造函数
public HashSet(int initialCapacity, float loadFactor) 

// 指定HashSet初始容量的构造函数
public HashSet(int initialCapacity) 

// 指定HashSet初始容量和加载因子的构造函数,dummy没有任何作用
HashSet(int initialCapacity, float loadFactor, boolean dummy)
```

> **HashSet的主要API** 

```java
boolean         add(E object)
void            clear()
Object          clone()
boolean         contains(Object object)
boolean         isEmpty()
Iterator<E>     iterator()
boolean         remove(Object object)
int             size()
```

#### 二、数据结构

![1545549518834](F:\images/201818271437-U.png)

从图中可以看出： 

- HashSet继承于AbstractSet，并且实现了Set接口。 
- HashSet的本质是一个"没有重复元素"的集合，**它是通过HashMap实现的**。HashSet中含有一个"HashMap类型的成员变量"map，HashSet的操作函数，实际上都是通过map实现的。 

#### 三、源码解析

```java
package java.util;

public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    // HashSet是通过map(HashMap对象)保存内容的
    private transient HashMap<E,Object> map;

    // PRESENT是向map中插入key-value对应的value
    // 因为HashSet中只需要用到key，而HashMap是key-value键值对；
    // 所以，向map中添加键值对时，键值对的值固定是PRESENT
    private static final Object PRESENT = new Object();

    // 默认构造函数
    public HashSet() {
        // 调用HashMap的默认构造函数，创建map
        map = new HashMap<E,Object>();
    }

    // 带集合的构造函数
    public HashSet(Collection<? extends E> c) {
        // 创建map。
        // 为什么要调用Math.max((int) (c.size()/.75f) + 1, 16)，从 (c.size()/.75f) + 1 和 16 中选择一个比较大的数呢？ 
        //将c.size()当作临界值
 	    //根据 threshold = capacity * loadFactor, 可以计算出 capacity
 	    //Math.max((int) (c.size()/.75f) + 1, 16) 这个意思就是capacity如果没超过16, 那么就直接使用默认的16
        map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) + 1, 16));
        // 将集合(c)中的全部元素添加到HashSet中
        addAll(c);
    }

    // 指定HashSet初始容量和加载因子的构造函数
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<E,Object>(initialCapacity, loadFactor);
    }

    // 指定HashSet初始容量的构造函数
    public HashSet(int initialCapacity) {
        map = new HashMap<E,Object>(initialCapacity);
    }

    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<E,Object>(initialCapacity, loadFactor);
    }

    // 返回HashSet的迭代器
    public Iterator<E> iterator() {
        // 实际上返回的是HashMap的“key集合的迭代器”
        return map.keySet().iterator();
    }

    public int size() {
        return map.size();
    }

    public boolean isEmpty() {
        return map.isEmpty();
    }

    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    // 将元素(e)添加到HashSet中
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    // 删除HashSet中的元素(o)
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }

    public void clear() {
        map.clear();
    }

    // 克隆一个HashSet，并返回Object对象
    public Object clone() {
        try {
            HashSet<E> newSet = (HashSet<E>) super.clone();
            newSet.map = (HashMap<E, Object>) map.clone();
            return newSet;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }

    // java.io.Serializable的写入函数
    // 将HashSet的“总的容量，加载因子，实际容量，所有的元素”都写入到输出流中
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        s.defaultWriteObject();

        // Write out HashMap capacity and load factor
        s.writeInt(map.capacity());
        s.writeFloat(map.loadFactor());

        // Write out size
        s.writeInt(map.size());

        // Write out all elements in the proper order.
        for (Iterator i=map.keySet().iterator(); i.hasNext(); )
            s.writeObject(i.next());
    }


    // java.io.Serializable的读取函数
    // 将HashSet的“总的容量，加载因子，实际容量，所有的元素”依次读出
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        s.defaultReadObject();

        // Read in HashMap capacity and load factor and create backing HashMap
        int capacity = s.readInt();
        float loadFactor = s.readFloat();
        map = (((HashSet)this) instanceof LinkedHashSet ?
               new LinkedHashMap<E,Object>(capacity, loadFactor) :
               new HashMap<E,Object>(capacity, loadFactor));

        // Read in size
        int size = s.readInt();

        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            E e = (E) s.readObject();
            map.put(e, PRESENT);
        }
    }
}
```

#### 四、遍历方式

##### 4.1 通过iterator遍历HashSet

```java
// 假设set是HashSet对象
for(Iterator iterator = set.iterator();
       iterator.hasNext(); ) { 
    iterator.next();
} 
```

##### 4.2 通过For-Each来遍历

- 根据toArray()获取HashSet的元素集合对应的数组。 
- 遍历数组，获取各个元素。 

```java
// 假设set是HashSet对象，并且set中元素是String类型
String[] arr = (String[])set.toArray(new String[0]);
for (String str:arr)
    System.out.printf("for each : %s\n", str);
```

##### 4.3 遍历方式比较

```java
package com.util.hashset;

import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;

/**
 * @Author: 98050
 * @Time: 2018-12-23 15:08
 * @Feature: HashSet遍历
 */
public class Test {

    public static void main(String[] args) {
        Set<Integer> set = new HashSet<Integer>();
        for (int i = 0; i < 1000000; i++) {
            set.add(i);
        }
        ThroughIterator(set);
        ThroughForEach(set);

    }

    public static void ThroughIterator(Set set){
        Iterator iterator = set.iterator();
        long start = System.currentTimeMillis();
        while (iterator.hasNext()){
            iterator.next();
        }
        System.out.println("使用迭代器耗时：" + (System.currentTimeMillis() - start));
    }

    public static void ThroughForEach(Set set){
        long start = System.currentTimeMillis();
        for (Object o : set.toArray()){
            ;
        }
        System.out.println("使用For-Each耗时：" + (System.currentTimeMillis() - start));
    }
}
```

![1545551281396](F:\images/201818271438-y.png)

推荐使用迭代器进行遍历。

#### 五、为什么HashSet不是线程安全的

因为HashMap就是非线程安全的，归根结底还是HashMap的原因。

证明：

```java
package com.util.hashset;

import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;

/**
 * @Author: 98050
 * @Time: 2018-12-23 15:56
 * @Feature:
 */
public class Test2 {

    public static void main(String[] args) throws InterruptedException, NoSuchFieldException, IllegalAccessException {
        final Set<String> set = new HashSet<String>();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    set.add(Thread.currentThread().getName() + "添加：" + i);
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 10; i < 20; i++) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    set.add(Thread.currentThread().getName() + "添加：" + i);
                }
            }
        }).start();

        Thread.sleep(1000);
        System.out.println("set的大小：" + set.size());
        Iterator iterator = set.iterator();
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }
    }
}
```

结果：数据丢失

![1545553423888](F:\images/201818271438-C.png)