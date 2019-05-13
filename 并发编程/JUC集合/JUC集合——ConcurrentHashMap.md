## 一、介绍

ConcurrentHashMap是线程安全的哈希表。HashMap、HashTable、ConcurrentHashMap之间的关联如下：

- **HashMap**是非线程安全的哈希表，常用于单线程程序中。

- **HashTable**是线程安全的哈希表，它是通过synchronized来保证线程安全的，即多个线程通过同一个“对象的同步锁”来实现并发控制。HashTable在线程竞争激烈时，效率比较低(此时建议使用ConcurrentHashMap！因为当一个线程访问Hashtable的同步方法时，其它线程就访问Hashtable的同步方法时，可能会进入阻塞状态。

- **ConcurrentHashMap**是线程安全的哈希表，利用CAS + synchronized 来保证线程安全。

  （jdk 1.7 是通过“锁分段”来保证线程安全的。ConcurrentHashMap将哈希表分成许多片段(Segment)，每一个片段除了保存哈希表之外，本质上也是一个“可重入的互斥锁”(ReentrantLock)。多线程对同一个片段的访问，是互斥的；但是，对于不同片段的访问，却是可以同步进行的。）

> **JDK 1.7简单分析**

ConcurrentHashMap采用 分段锁的机制，实现并发的更新操作，底层采用数组+链表的存储结构。
 其包含两个核心静态内部类 Segment和HashEntry。

- Segment继承ReentrantLock用来充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶。
- HashEntry 用来封装映射表的键 / 值对；
- 每个桶是由若干个 HashEntry 对象链接起来的链表。

一个 ConcurrentHashMap 实例中包含由若干个 Segment 对象组成的数组，下面通过一个图来演示一下 ConcurrentHashMap 的结构：

 ![](F:\images/201818272132-a.png)

## 二、原理和数据结构（JDK 1.8）

JDK1.8的实现已经抛弃了Segment分段锁机制，利用**CAS+Synchronized**来保证并发更新的安全，底层采用数组+链表+红黑树的存储结构。 

![](F:\images/201818272046-M.png)

ConcurrentHashMap在1.8中的实现，相比于1.7的版本基本上全部都变掉了。首先，**取消了Segment分段**锁的数据结构，取而代之的是**数组+链表+红黑树**的结构。而对于锁的粒度，调整为**对每个数组元素加锁**（Node）。然后是定位节点的hash算法被简化了，这样带来的弊端是Hash冲突会加剧。因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。这样一来，查询的时间复杂度就会由原先的O(n)变为O(logN)。

## 三、方法列表

```java
// 创建一个带有默认初始容量 (16)、加载因子 (0.75) 和 concurrencyLevel (16) 的新的空映射。
ConcurrentHashMap()
// 创建一个带有指定初始容量、默认加载因子 (0.75) 和 concurrencyLevel (16) 的新的空映射。
ConcurrentHashMap(int initialCapacity)
// 创建一个带有指定初始容量、加载因子和默认 concurrencyLevel (16) 的新的空映射。
ConcurrentHashMap(int initialCapacity, float loadFactor)
// 创建一个带有指定初始容量、加载因子和并发级别的新的空映射。
ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel)
// 构造一个与给定映射具有相同映射关系的新映射。
ConcurrentHashMap(Map<? extends K,? extends V> m)

// 从该映射中移除所有映射关系
void clear()
// 一种遗留方法，测试此表中是否有一些与指定值存在映射关系的键。
boolean contains(Object value)
// 测试指定对象是否为此表中的键。
boolean containsKey(Object key)
// 如果此映射将一个或多个键映射到指定值，则返回 true。
boolean containsValue(Object value)
// 返回此表中值的枚举。
Enumeration<V> elements()
// 返回此映射所包含的映射关系的 Set 视图。
Set<Map.Entry<K,V>> entrySet()
// 返回指定键所映射到的值，如果此映射不包含该键的映射关系，则返回 null。
V get(Object key)
// 如果此映射不包含键-值映射关系，则返回 true。
boolean isEmpty()
// 返回此表中键的枚举。
Enumeration<K> keys()
// 返回此映射中包含的键的 Set 视图。
Set<K> keySet()
// 将指定键映射到此表中的指定值。
V put(K key, V value)
// 将指定映射中所有映射关系复制到此映射中。
void putAll(Map<? extends K,? extends V> m)
// 如果指定键已经不再与某个值相关联，则将它与给定值关联。
V putIfAbsent(K key, V value)
// 从此映射中移除键（及其相应的值）。
V remove(Object key)
// 只有目前将键的条目映射到给定值时，才移除该键的条目。
boolean remove(Object key, Object value)
// 只有目前将键的条目映射到某一值时，才替换该键的条目。
V replace(K key, V value)
// 只有目前将键的条目映射到给定值时，才替换该键的条目。
boolean replace(K key, V oldValue, V newValue)
// 返回此映射中的键-值映射关系数。
int size()
// 返回此映射中包含的值的 Collection 视图。
Collection<V> values()
```

## 四、源码分析

#### 4.1 类的继承关系

![](F:\images/201818272058-n.png)

说明：ConcurrentHashMap继承了AbstractMap抽象类，该抽象类定义了一些基本操作，同时，也实现了ConcurrentMap接口，ConcurrentMap接口也定义了一系列操作，实现了Serializable接口表示ConcurrentHashMap可以被序列化。 

#### 4.2 内部类

![](F:\images/201818272102-a.png)

太多了，主要内部类框架图如下所示：

![](F:\images/201818272103-N.png)

说明：ConcurrentHashMap的内部类非常的庞大，下面对其中主要的内部类进行分析和讲解。

　　**1. Node类**

　　Node类主要用于存储具体键值对，其子类有ForwardingNode（一个特殊的Node节点，hash值为-1，其中存储nextTable的引用）、ReservationNode、TreeNode和TreeBin四个子类。四个子类具体的代码在之后的具体例子中进行分析讲解。

　　**2. Traverser类**

　　Traverser类主要用于遍历操作，其子类有BaseIterator、KeySpliterator、ValueSpliterator、EntrySpliterator四个类，BaseIterator用于遍历操作。KeySplitertor、ValueSpliterator、EntrySpliterator则用于键、值、键值对的划分。

　　**3. CollectionView类**

　　CollectionView抽象类主要定义了视图操作，其子类KeySetView、ValueSetView、EntrySetView分别表示键视图、值视图、键值对视图。对视图均可以进行操作。

　　**4. Segment类**

　　Segment类在JDK1.8中与之前的版本的JDK作用存在很大的差别，JDK1.8下，其在普通的ConcurrentHashMap操作中已经没有失效，其在序列化与反序列化的时候会发挥作用。

　　**5. CounterCell**

　　CounterCell类主要用于对baseCount的计数。

#### 4.3 主要属性

```java 
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    private static final long serialVersionUID = 7249069246763182397L;
    // 表的最大容量
    private static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认表的大小
    private static final int DEFAULT_CAPACITY = 16;
    // 最大数组大小
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    // 默认并发数
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    // 装载因子
    private static final float LOAD_FACTOR = 0.75f;
    // 转化为红黑树的阈值
    static final int TREEIFY_THRESHOLD = 8;
    // 由红黑树转化为链表的阈值
    static final int UNTREEIFY_THRESHOLD = 6;
    // 转化为红黑树的表的最小容量
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 每次进行转移的最小值
    private static final int MIN_TRANSFER_STRIDE = 16;
    // 生成sizeCtl所使用的bit位数
    private static int RESIZE_STAMP_BITS = 16;
    // 进行扩容所允许的最大线程数
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
    // 记录sizeCtl中的大小所需要进行的偏移位数
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;    
    // 一系列的标识
    static final int MOVED     = -1; // hash for forwarding nodes
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
    // 

    // 获取可用的CPU个数
    static final int NCPU = Runtime.getRuntime().availableProcessors();
    // 

    // 进行序列化的属性
    private static final ObjectStreamField[] serialPersistentFields = {
        new ObjectStreamField("segments", Segment[].class),
        new ObjectStreamField("segmentMask", Integer.TYPE),
        new ObjectStreamField("segmentShift", Integer.TYPE)
    };
    
    // 表
    transient volatile Node<K,V>[] table;
    // 扩容时新生成的数组
    private transient volatile Node<K,V>[] nextTable;

    // 基本计数
    private transient volatile long baseCount;
    //
    
    // 对表初始化和扩容控制
    // 1 代表table正在初始化
    // N 表示有N-1个线程正在进行扩容操作
    // 其余情况：
    //   1、如果table未初始化，表示table需要初始化的大小。
    //   2、如果table初始化完成，表示table的容量，默认是table大小的0.75倍，计算公式：0.75（n - (n >>> 2)）。
    private transient volatile int sizeCtl;
    
    // 扩容下另一个表的索引
    private transient volatile int transferIndex;

    // 旋转锁
    private transient volatile int cellsBusy;

    // counterCell表
    private transient volatile CounterCell[] counterCells;

    // views
    // 视图
    private transient KeySetView<K,V> keySet;
    private transient ValuesView<K,V> values;
    private transient EntrySetView<K,V> entrySet;
    
    // Unsafe mechanics
    private static final sun.misc.Unsafe U;
    private static final long SIZECTL;
    private static final long TRANSFERINDEX;
    private static final long BASECOUNT;
    private static final long CELLSBUSY;
    private static final long CELLVALUE;
    private static final long ABASE;
    private static final int ASHIFT;

    static {
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentHashMap.class;
            SIZECTL = U.objectFieldOffset
                (k.getDeclaredField("sizeCtl"));
            TRANSFERINDEX = U.objectFieldOffset
                (k.getDeclaredField("transferIndex"));
            BASECOUNT = U.objectFieldOffset
                (k.getDeclaredField("baseCount"));
            CELLSBUSY = U.objectFieldOffset
                (k.getDeclaredField("cellsBusy"));
            Class<?> ck = CounterCell.class;
            CELLVALUE = U.objectFieldOffset
                (ck.getDeclaredField("value"));
            Class<?> ak = Node[].class;
            ABASE = U.arrayBaseOffset(ak);
            int scale = U.arrayIndexScale(ak);
            if ((scale & (scale - 1)) != 0)
                throw new Error("data type scale not a power of two");
            ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

#### 4.4 构造函数

> **ConcurrentHashMap()**

![](F:\images/201818272154-g.png)

该构造函数用于创建一个带有默认初始容量DEFAULT_CAPACITY (16)、加载因子LOAD_FACTOR (0.75) 和 DEFAULT_CONCURRENCY_LEVEL (16) 的映射。 

> **ConcurrentHashMap(int initialCapacity)**

![](F:\images/201818272202-9.png)

该构造函数用于创建一个带有指定初始容量、加载因子LOAD_FACTOR (0.75) 和 DEFAULT_CONCURRENCY_LEVEL (16) 的映射。 其中tableSizeFor方法和HashMap中的一样。

![1545919660660](F:\images/201818272207-F.png)

> **ConcurrentHashMap(Map<? extends K, ? extends V> m)**

![](F:\images/201818272208-9.png)

该构造函数用于构造一个与给定映射具有相同映射关系的新映射。 

> **ConcurrentHashMap(int initialCapacity, float loadFactor)**

![](F:\images/201818272209-f.png)

该构造函数用于创建一个带有指定初始容量、加载因子和默认 DEFAULT_CONCURRENCY_LEVEL (1)的新的空映射。 

> **ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel)**

![](F:\images/201818272210-k.png)

该构造函数用于创建一个带有指定初始容量、加载因子和并发级别的新的空映射 。

对于构造函数而言，会根据输入的initialCapacity的大小来确定一个最小的且大于等于initialCapacity大小的2的n次幂，如initialCapacity为15，则sizeCtl为16，若initialCapacity为16，则sizeCtl为16。若initialCapacity大小超过了允许的最大值，则sizeCtl为最大值。 

值得注意的是，构造函数中的concurrencyLevel参数已经在JDK1.8中的意义发生了很大的变化，其并不代表所允许的并发数，其只是用来确定sizeCtl大小，在JDK1.8中的并发控制都是针对具体的桶而言，即有多少个桶就可以允许多少个并发数。 

#### 4.4 核心操作

##### 4.4.1 添加

假设table已经初始化完成，put操作采用==CAS+synchronized==实现并发插入或更新操作： 
- 当前bucket为空时，使用CAS操作，将Node放入对应的bucket中。 
- 出现hash冲突，则采用synchronized关键字。倘若当前hash对应的节点是链表的头节点，遍历链表，若找到对应的node节点，则修改node节点的val，否则在链表末尾添加node节点；倘若当前节点是红黑树的根节点，在树结构上遍历元素，更新或增加节点。 
- 倘若当前map正在扩容f.hash == MOVED， 则跟其他线程一起进行扩容



![](F:\images/201818272217-N.png)

put方法底层调用了putVal进行数据的插入 ，putVal方法：

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    //ConcurrentHashMap 不允许插入null键，HashMap允许插入一个null键
    if (key == null || value == null) throw new NullPointerException();
    //计算key的hash值
    int hash = spread(key.hashCode());
    int binCount = 0;
    //for循环的作用：因为更新元素是使用CAS机制更新，需要不断的失败重试，直到成功为止。
    for (Node<K,V>[] tab = table;;) {
        // f：链表或红黑二叉树头结点，向链表中添加元素时，需要synchronized获取f的锁。
        Node<K,V> f; int n, i, fh;
        //判断Node[]数组是否初始化，没有则进行初始化操作
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //通过hash定位Node[]数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加（链表的头结点），添加失败则进入下次循环。
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //检查到内部正在移动元素（Node[] 数组扩容）
        else if ((fh = f.hash) == MOVED)
            //帮助它扩容
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //锁住链表或红黑二叉树的头结点
            synchronized (f) {
                //判断f是否是链表的头结点
                if (tabAt(tab, i) == f) {
                    //如果fh>=0 是链表节点
                    if (fh >= 0) {
                        binCount = 1;
                        //遍历链表所有节点
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果节点存在，则更新value
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            //不存在则在链表尾部添加新节点。
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //TreeBin是红黑二叉树节点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        //添加树节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                      value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            
            if (binCount != 0) {
                //如果链表长度已经达到临界值8 就需要把链表转换为树结构
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //将当前ConcurrentHashMap的size数量+1
    addCount(1L, binCount);
    return null;
}
```

大体流程如下：

　　① 判断存储的key、value是否为空，若为空，则抛出异常，否则，进入步骤②

　　② 计算key的hash值，随后进入无限循环，该无限循环（自旋锁）可以确保成功插入数据，若table表为空或者长度为0，则初始化table表，否则，进入步骤③

　　③ 根据key的hash值取出table表中的结点元素，若取出的结点为空（该桶为空），则使用CAS将key、value、hash值生成的结点放入桶中。否则，进入步骤④

　　④ 若该结点的的hash值为MOVED，则对该桶中的结点进行转移，否则，进入步骤⑤

　　⑤ 对桶中的第一个结点（即table表中的结点）进行加锁，对该桶进行遍历，桶中的结点的hash值与key值与给定的hash值和key值相等，则根据标识选择是否进行更新操作（用给定的value值替换该结点的value值），若遍历完桶仍没有找到hash值与key值和指定的hash值与key值相等的结点，则直接新生一个结点并赋值为之前最后一个结点的下一个结点。进入步骤⑥

　　⑥ 若binCount值达到红黑树转化的阈值，则将桶中的结构转化为红黑树存储，最后，增加binCount的值。

------

在putVal方法中有几个地方需要说明一下：

> **Hash算法** 

```java
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
static final int spread(int h) {return (h ^ (h >>> 16)) & HASH_BITS;}
```

> **定位索引**

```Java
int index = (n - 1) & hash  // n为bucket的个数
```

> **initTable**

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) { // 无限循环
            if ((sc = sizeCtl) < 0) // sizeCtl小于0，则进行线程让步等待
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { // 比较sizeCtl的值与sc是否相等，相等则用-1替换
                try {
                    if ((tab = table) == null || tab.length == 0) { // table表为空或者大小为0
                        // sc的值是否大于0，若是，则n为sc，否则，n为默认初始容量
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        // 新生结点数组
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        // 赋值给table
                        table = tab = nt;
                        // sc为n * 3/4
                        sc = n - (n >>> 2);
                    }
                } finally {
                    // 设置sizeCtl的值
                    sizeCtl = sc;
                }
                break;
            }
        }
        // 返回table表
        return tab;
    }
```

对于table的大小，会根据sizeCtl的值进行设置，如果没有设置szieCtl的值，那么默认生成的table大小为16，否则，会根据sizeCtl的大小设置table大小。 

> **tabAt**

![](F:\images/201818272304-r.png)

采用Unsafe.getObjectVolatie()来获取，而不是直接用table[index]的原因跟ConcurrentHashMap的弱一致性有关。在java内存模型中，我们已经知道每个线程都有一个工作内存，里面存储着table的副本，虽然table是volatile修饰的，但不能保证线程每次都拿到table中的最新元素，**Unsafe.getObjectVolatile可以直接获取指定内存的数据，保证了每次拿到数据都是最新的。**

> **casTabAt**

![](F:\images/201818272302-U.png)

用于比较table数组下标为i的结点是否为c，若为c，则用v交换操作。否则，不进行交换操作。 

> **helpTransfer**

在扩容时将table表中的结点转移到nextTable中。 

> **putTreeVal**

用于将指定的hash、key、value值添加到红黑树中，若已经添加了，则返回null，否则返回该结点。 

> **treeifyBin**

 用于将桶中的数据结构转化为红黑树，其中，值得注意的是，当table的长度未达到阈值时，会进行一次扩容操作，该操作会使得触发treeifyBin操作的某个桶中的所有元素进行一次重新分配，这样可以避免某个桶中的结点数量太大。 

> **addCount**

主要完成binCount的值加1的操作。 

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    //如果更新失败才会进入的 if 的主体代码中
    //s = b + x  其中 x 等于 1
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        //高并发下 CAS 失败会执行 fullAddCount 方法
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```

第一个if的功能是完成的是对 baseCount 的 CAS 更新。第二个if是判断是否需要扩容。

> **扩容**

扩容的时机：

- 如果新增节点之后，所在的链表的元素个数大于等于8，则会调用treeifyBin把链表转换为红黑树。在转换结构时，若tab的长度小于MIN_TREEIFY_CAPACITY，默认值为64，则会将数组长度扩大到原来的两倍，并触发transfer，重新调整节点位置。（只有当tab.length >= 64, ConcurrentHashMap才会使用红黑树。） 

  ![1545976238915](F:\images/201818281350-L.png)

  tryPresize中进行扩容：

  ```java
   private final void tryPresize(int size) {
          int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
              tableSizeFor(size + (size >>> 1) + 1);
          int sc;
          while ((sc = sizeCtl) >= 0) {
              Node<K,V>[] tab = table; int n;
              if (tab == null || (n = tab.length) == 0) {
                  n = (sc > c) ? sc : c;
                  if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                      try {
                          if (table == tab) {
                              @SuppressWarnings("unchecked")
                              Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                              table = nt;
                              sc = n - (n >>> 2);
                          }
                      } finally {
                          sizeCtl = sc;
                      }
                  }
              }
              else if (c <= sc || n >= MAXIMUM_CAPACITY)
                  break;
              else if (tab == table) {
                  int rs = resizeStamp(n);
                  if (sc < 0) {
                      Node<K,V>[] nt;
                      if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                          sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                          transferIndex <= 0)
                          break;
                      if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                          transfer(tab, nt);
                  }
                  else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                               (rs << RESIZE_STAMP_SHIFT) + 2))
                      transfer(tab, null);
              }
          }
      }
  ```

- 新增节点后，addCount统计tab中的节点个数大于阈值（sizeCtl），会触发transfer，重新调整节点位置。

  ![1545976127224](F:\images/201818281348-O.png)
> **transfer**

当table的元素数量达到容量阈值sizeCtl，需要对table进行扩容：

构建一个nextTable，大小为table两倍  

把table的数据复制到nextTable中。  

在扩容过程中，依然支持并发更新操作；也支持并发插入。  

![](F:\images/201818281355-l.png)

##### 4.4.2 获取

```java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        // 计算key的hash值
        int h = spread(key.hashCode()); 
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) { // 表不为空并且表的长度大于0并且key所在的桶不为空
            if ((eh = e.hash) == h) { // 表中的元素的hash值与key的hash值相等
                if ((ek = e.key) == key || (ek != null && key.equals(ek))) // 键相等
                    // 返回值
                    return e.val;
            }
            else if (eh < 0) // 结点hash值小于0
                // 在桶（链表/红黑树）中查找
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) { // 对于结点hash值大于0的情况
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

##### 4.4.3 删除

![](F:\images/201818272320-Z.png)

remove函数底层是调用的replaceNode函数实现结点的删除。 

```java
final V replaceNode(Object key, V value, Object cv) {
        // 计算key的hash值
        int hash = spread(key.hashCode());
        for (Node<K,V>[] tab = table;;) { // 无限循环
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0 ||
                (f = tabAt(tab, i = (n - 1) & hash)) == null) // table表为空或者表长度为0或者key所对应的桶为空
                // 跳出循环
                break;
            else if ((fh = f.hash) == MOVED) // 桶中第一个结点的hash值为MOVED
                // 转移
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                boolean validated = false;
                synchronized (f) { // 加锁同步
                    if (tabAt(tab, i) == f) { // 桶中的第一个结点没有发生变化
                        if (fh >= 0) { // 结点hash值大于0
                            validated = true;
                            for (Node<K,V> e = f, pred = null;;) { // 无限循环
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) { // 结点的hash值与指定的hash值相等，并且key也相等
                                    V ev = e.val;
                                    if (cv == null || cv == ev ||
                                        (ev != null && cv.equals(ev))) { // cv为空或者与结点value相等或者不为空并且相等
                                        // 保存该结点的val值
                                        oldVal = ev;
                                        if (value != null) // value为null
                                            // 设置结点value值
                                            e.val = value;
                                        else if (pred != null) // 前驱不为空
                                            // 前驱的后继为e的后继，即删除了e结点
                                            pred.next = e.next;
                                        else
                                            // 设置table表中下标为index的值为e.next
                                            setTabAt(tab, i, e.next);
                                    }
                                    break;
                                }
                                pred = e;
                                if ((e = e.next) == null)
                                    break;
                            }
                        }
                        else if (f instanceof TreeBin) { // 为红黑树结点类型
                            validated = true;
                            // 类型转化
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> r, p;
                            if ((r = t.root) != null &&
                                (p = r.findTreeNode(hash, key, null)) != null) { // 根节点不为空并且存在与指定hash和key相等的结点
                                // 保存p结点的value
                                V pv = p.val;
                                if (cv == null || cv == pv ||
                                    (pv != null && cv.equals(pv))) { // cv为空或者与结点value相等或者不为空并且相等
                                    oldVal = pv;
                                    if (value != null) 
                                        p.val = value;
                                    else if (t.removeTreeNode(p)) // 移除p结点
                                        setTabAt(tab, i, untreeify(t.first));
                                }
                            }
                        }
                    }
                }
                if (validated) {
                    if (oldVal != null) {
                        if (value == null)
                            // baseCount值减一
                            addCount(-1L, -1);
                        return oldVal;
                    }
                    break;
                }
            }
        }
        return null;
    }
```

##### 4.4.4 计数

**ConcurrentHashMap的元素个数等于baseCounter和数组里每个CounterCell的值之和，这样做的原因是，当多个线程同时执行CAS修改baseCount值，失败的线程会将值放到CounterCell中。所以统计元素个数时，要把baseCount和CounterCells数组都考虑。**

![](F:\images/201818281449-W.png)

![](F:\images/201818281449-G.png)

可能你会有所疑问，ConcurrentHashMap 中的 baseCount 属性不就是记录的所有键值对的总数吗？直接返回它不就行了吗？

之所以没有这么做，是因为我们的 addCount 方法用于 CAS 更新 baseCount，但很有可能在高并发的情况下，更新失败，那么这些节点虽然已经被添加到哈希表中了，但是数量却没有被统计。

**还好，addCount 方法在更新 baseCount 失败的时候，会调用 fullAddCount 将这些失败的结点包装成一个 CounterCell 对象，保存在 CounterCell 数组中。那么整张表实际的 size 其实是 baseCount 加上 CounterCell 数组中元素的个数。**

## 五、使用

**代码：**

```java
package com.juc.concurrenthashmap;

import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @Author: 98050
 * @Time: 2018-12-27 20:05
 * @Feature:
 */
public class Test {
    public static void main(String[] args) throws InterruptedException {
        final ConcurrentHashMap<String,String> map = new ConcurrentHashMap(20);
        new Thread(new Runnable() {
            @Override
            public void run() {
                //线程1放入0——9
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    map.put(Thread.currentThread().getName() + "放入" + i, i + "");
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                //线程2放入10——19
                for (int i = 10; i < 20; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    map.put(Thread.currentThread().getName() + "放入" + i, i + "");
                }
            }
        }).start();


        Thread.sleep(1000);
        System.out.println("map的大小：" + map.size());
        Iterator iterator = map.entrySet().iterator();
        while(iterator.hasNext()) {
            Map.Entry entry = (Map.Entry)iterator.next();
            // 获取key和value
            System.out.println("key：" + entry.getKey() + ", value：" + entry.getValue());
        }
    }
}
```

**结果：**

![](F:\images/201818272333-9.png)

## 六、ConcurrentHashMap1.7和1.8的比较

|              |                           JDK 1.7                            |                           JDK 1.8                            |
| :----------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    结构图    |              ![](F:\images/201818281400-u.png)               |              ![](F:\images/201818281400-d.png)               |
| 数据结构说明 |             采用Segment + HashEntry的方式实现。              |     采用Node + CAS + Synchronized来保证并发安全进行实现      |
|    初始化    | 初始化时，计算出Segment数组的大小ssize和每个Segment中HashEntry数组的大小cap，并初始化Segment数组的第一个元素；其中ssiz大小为2的幂次方，默认为16，ca大小也是2的幂次方，最小值为2，最终结果根据根据初始化容量initialCapacit进行计算 | 只有在执行第一次put方法时才会调用initTable()初始化Node数组。 |
|   put实现    | 多个线程同时竞争获取同一个segment锁，获取成功的线程更新map；失败的线程尝试多次获取锁仍未成功，则挂起线程，等待释放锁 | **访问相应的bucket时，使用sychronizeded关键字，防止多个线程同时操作同一个bucket，如果该节点的hash不小于0，则遍历链表更新节点或插入新节点；如果该节点是TreeBin类型的节点，说明是红黑树结构，则通过putTreeVal方法往红黑树中插入节点；更新了节点数量，还要考虑扩容和链表转红黑树** |
|   size实现   | 统计每个Segment对象中的元素个数，然后进行累加，但是这种方式计算出来的结果并不一样的准确的。先采用不加锁的方式，连续计算元素的个数，最多计算3次：如果前后两次计算结果相同，则说明计算出来的元素个数是准确的；如果前后两次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数 | 通过累加baseCount和CounterCell数组中的数量，即可得到元素的总个数； |

## 七、ConcurrentHashMap能完全替代Hashtable吗？

Hashtable虽然性能上不如ConcurrentHashMap，但并不能完全被取代，两者的迭代器的一致性不同的，Hashtable的迭代器是强一致性的，而ConcurrentHashMap是弱一致的。 ConcurrentHashMap的get，clear，iterator 都是弱一致性的。 
下面是大白话的解释： 

- **Hashtable的任何操作都会把整个表锁住，是阻塞的。好处是总能获取最实时的更新，比如说线程A调用putAll写入大量数据，期间线程B调用get，线程B就会被阻塞，直到线程A完成putAll，因此线程B肯定能获取到线程A写入的完整数据。坏处是所有调用都要排队，效率较低。** 
- **ConcurrentHashMap 是设计为非阻塞的。在更新时会局部锁住某部分数据，但不会把整个表都锁住。同步读取操作则是完全非阻塞的。好处是在保证合理的同步前提下，效率很高。坏处 是严格来说读取操作不能保证反映最近的更新。例如线程A调用putAll写入大量数据，期间线程B调用get，则只能get到目前为止已经顺利插入的部分 数据。**

选择哪一个，是在性能与数据一致性之间权衡。ConcurrentHashMap适用于追求性能的场景，大多数线程都只做insert/delete操作，对读取数据的一致性要求较低。