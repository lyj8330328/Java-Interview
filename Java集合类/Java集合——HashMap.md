#### 一、介绍

HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。

HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。

HashMap 的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为null。此外，HashMap中的映射不是有序的。

HashMap 的实例有两个参数影响其性能：“**初始容量**” 和 “**加载因子**”。

- 容量 是哈希表中桶的数量，初始容量 只是哈希表在创建时的容量。加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有**大约两倍**的桶数。
- 通常，**默认加载因子是 0.75**, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。

> **HashMap的构造函数**

```java
// 默认构造函数。
HashMap()

// 指定“容量大小”的构造函数
HashMap(int capacity)

// 指定“容量大小”和“加载因子”的构造函数
HashMap(int capacity, float loadFactor)

// 包含“子Map”的构造函数
HashMap(Map<? extends K, ? extends V> map)
```

> **HashMap的API**
>

```java
void                 clear()
Object               clone()
boolean              containsKey(Object key)
boolean              containsValue(Object value)
Set<Entry<K, V>>     entrySet()
V                    get(Object key)
boolean              isEmpty()
Set<K>               keySet()
V                    put(K key, V value)
void                 putAll(Map<? extends K, ? extends V> map)
V                    remove(Object key)
int                  size()
Collection<V>        values()
```

```java
/*
 * Implementation notes.
 *
 * This map usually acts as a binned (bucketed) hash table, but
 * when bins get too large, they are transformed into bins of
 * TreeNodes, each structured similarly to those in
 * java.util.TreeMap. Most methods try to use normal bins, but
 * relay to TreeNode methods when applicable (simply by checking
 * instanceof a node).  Bins of TreeNodes may be traversed and
 * used like any others, but additionally support faster lookup
 * when overpopulated. However, since the vast majority of bins in
 * normal use are not overpopulated, checking for existence of
 * tree bins may be delayed in the course of table methods.
 *
 * Tree bins (i.e., bins whose elements are all TreeNodes) are
 * ordered primarily by hashCode, but in the case of ties, if two
 * elements are of the same "class C implements Comparable<C>",
 * type then their compareTo method is used for ordering. (We
 * conservatively check generic types via reflection to validate
 * this -- see method comparableClassFor).  The added complexity
 * of tree bins is worthwhile in providing worst-case O(log n)
 * operations when keys either have distinct hashes or are
 * orderable, Thus, performance degrades gracefully under
 * accidental or malicious usages in which hashCode() methods
 * return values that are poorly distributed, as well as those in
 * which many keys share a hashCode, so long as they are also
 * Comparable. (If neither of these apply, we may waste about a
 * factor of two in time and space compared to taking no
 * precautions. But the only known cases stem from poor user
 * programming practices that are already so slow that this makes
 * little difference.)
 *
 * Because TreeNodes are about twice the size of regular nodes, we
 * use them only when bins contain enough nodes to warrant use
 * (see TREEIFY_THRESHOLD). And when they become too small (due to
 * removal or resizing) they are converted back to plain bins.  In
 * usages with well-distributed user hashCodes, tree bins are
 * rarely used.  Ideally, under random hashCodes, the frequency of
 * nodes in bins follows a Poisson distribution
 * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
 * parameter of about 0.5 on average for the default resizing
 * threshold of 0.75, although with a large variance because of
 * resizing granularity. Ignoring variance, the expected
 * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
 * factorial(k)). The first values are:
 *
 * 0:    0.60653066
 * 1:    0.30326533
 * 2:    0.07581633
 * 3:    0.01263606
 * 4:    0.00157952
 * 5:    0.00015795
 * 6:    0.00001316
 * 7:    0.00000094
 * 8:    0.00000006
 * more: less than 1 in ten million
 *
 * The root of a tree bin is normally its first node.  However,
 * sometimes (currently only upon Iterator.remove), the root might
 * be elsewhere, but can be recovered following parent links
 * (method TreeNode.root()).
 *
 * All applicable internal methods accept a hash code as an
 * argument (as normally supplied from a public method), allowing
 * them to call each other without recomputing user hashCodes.
 * Most internal methods also accept a "tab" argument, that is
 * normally the current table, but may be a new or old one when
 * resizing or converting.
 *
 * When bin lists are treeified, split, or untreeified, we keep
 * them in the same relative access/traversal order (i.e., field
 * Node.next) to better preserve locality, and to slightly
 * simplify handling of splits and traversals that invoke
 * iterator.remove. When using comparators on insertion, to keep a
 * total ordering (or as close as is required here) across
 * rebalancings, we compare classes and identityHashCodes as
 * tie-breakers.
 *
 * The use and transitions among plain vs tree modes is
 * complicated by the existence of subclass LinkedHashMap. See
 * below for hook methods defined to be invoked upon insertion,
 * removal and access that allow LinkedHashMap internals to
 * otherwise remain independent of these mechanics. (This also
 * requires that a map instance be passed to some utility methods
 * that may create new nodes.)
 *
 * The concurrent-programming-like SSA-based coding style helps
 * avoid aliasing errors amid all of the twisty pointer operations.
 */
```

#### 二、数据结构

**在JDK1.8之前，HashMap采用数组+链表实现，即使用链表处理冲突，同一hash值的节点都存储在一个链表里。但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，HashMap采用数组+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。**

下图是jdk1.8之前的hashmap结构，左边部分即代表哈希表，也称为哈希数组，数组的每个元素都是一个单链表的头节点，链表是用来解决冲突的，如果不同的key映射到了数组的同一位置处，就将其放入单链表中。

![JDK1.8之前HashMap结构图](http://mycsdnblog.work/201818241335-v.png)

jdk1.8之前的hashmap都采用上图的结构，都是基于一个数组和多个单链表，hash值冲突的时候，就将对应节点以链表的形式存储。如果在一个链表中查找其中一个节点时，将会花费O（n）的查找时间，会有很大的性能损失。到了jdk1.8，当同一个hash值的节点数大于8时，则不再采用单链表形式存储，而是采用红黑树，如下图所示。 

![JDK1.8 HashMap结构图](http://mycsdnblog.work/201818241341-L.png)

![](http://mycsdnblog.work/201818241342-d.png)

上图很形象的展示了HashMap的数据结构（数组+链表+红黑树），桶中的结构可能是链表，也可能是红黑树，红黑树的引入是为了提高效率。 

![](http://mycsdnblog.work/201818241409-c.png)

##### 2.1 链表的实现

![](http://mycsdnblog.work/201818241416-1.png)

Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。上图中的每个黑色圆点就是一个Node对象。来看具体代码： 

```java
//Node是单向链表，它实现了Map.Entry接口
static class Node<k,v> implements Map.Entry<k,v> {
    final int hash;
    final K key;
    V value;
    Node<k,v> next;
    //构造函数Hash值 键 值 下一个节点
    Node(int hash, K key, V value, Node<k,v> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
 
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + = + value; }
 
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
 
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //判断两个node是否相等,若key和value都相等，返回true。可以与自身比较为true
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<!--?,?--> e = (Map.Entry<!--?,?-->)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

##### 2.2 红黑树

具体请参考：https://blog.csdn.net/lyj2018gyq/article/details/85265403

![](http://mycsdnblog.work/201818261719-v.png)

TreeNode是继承LinkedHashMap的！！！！！！！！！！

##### 2.3 位桶

![](http://mycsdnblog.work/201818261721-O.png)

HashMap类中有一个非常重要的字段，就是 Node[] table，即哈希桶数组，明显它是一个Node的数组。 

首先有一个每个元素都是链表（可能表述不准确）的数组，当添加一个元素（key-value）时，就首先计算元素key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这时就添加到同一hash值的元素的后面，他们在数组的同一位置，但是形成了链表，所以说数组存放的是链表。而当链表长度太长时，链表就转换为红黑树，这样大大提高了查找的效率。 

#### 三、源码解析（JDK 1.8）

##### 3.1 类的继承关系

![](http://mycsdnblog.work/201818262009-j.png)

HashMap继承自父类（AbstractMap），实现了Map、Cloneable、Serializable接口。其中，Map接口定义了一组通用的操作；Cloneable接口则表示可以进行拷贝，在HashMap中，实现的是浅层次拷贝，即对拷贝对象的改变会影响被拷贝的对象；Serializable接口表示HashMap实现了序列化，即可以将HashMap对象保存至本地，之后可以恢复状态。 

##### 3.2 类的属性

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;    
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table; 
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 填充因子
    final float loadFactor;
}
```

##### 3.3 构造函数

<1>**HashMap(int, float)型构造函数** 

![](http://mycsdnblog.work/201818262014-Y.png)

初始容量不能小于0；如果大于最大值，那么就初始化为最大容量；装载因子不能小于等于0，必须为数字。

初始化装载因子：this.loadFactor = loadFactor

初始化临界值： this.threshold = tableSizeFor(initialCapacity)

tableSizeFor(initialCapacity)返回大于initialCapacity的最小的二次幂数值。 

![](http://mycsdnblog.work/201818262030-R.png)

操作符表示无符号右移，高位取0。 

**为什么要这样做？**

**如果不是2的幂次方就会造成分布不均匀了，增加了碰撞的几率，减慢了查询的效率，造成空间的浪费**。 

<2>**HashMap(int initialCapacity)**

![](http://mycsdnblog.work/201818262038-O.png)

<3>**HashMap()**

![](http://mycsdnblog.work/201818262039-T.png)

<4>**HashMap(Map<? extends K, ? extends V> m)**

![](http://mycsdnblog.work/201818262040-s.png)

初始化装载因子，然后将m中的所有元素添加到HashMap中。

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { // pre-size
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

##### 3.4 Hash算法

![](http://mycsdnblog.work/201818262046-3.png)

首先获取对象的hashCode()值，然后将hashCode值右移16位，然后将右移后的值与原来的hashCode做**异或**运算，返回结果。（其中h>>>16，在JDK1.8中，优化了高位运算的算法，使用了零扩展，无论正数还是负数，都在高位插入0） 

hashCode()：返回该对象的内部地址转换成整数。

------

**在putVal源码中，通过(n-1)&hash获取该对象的键在HashMap中的位置。（其中hash的值就是通过hash(Object key)获得的值）其中n表示的是hash桶数组的长度，并且该长度为2的n次方，这样(n-1)&hash就等价于hash%n。因为&运算的效率高于%运算。** (**当 lenth = 2^n 时，X % length = =X & (length - 1)**)

![1545828744891](http://mycsdnblog.work/201818262052-C.png)

tab即是table，n是map集合的容量大小，hash是上面方法的返回值。**因为通常声明map集合时不会指定大小，或者初始化的时候就创建一个容量很大的map对象，所以这个通过容量大小与key值进行hash的算法在开始的时候只会对低位进行计算，虽然容量的2进制高位一开始都是0，但是key的2进制高位通常是有值的，因此先在hash方法中将key的hashCode右移16位在与自身异或，使得高位也可以参与hash，更大程度上减少了碰撞率。** 

下面举例说明下，n为table的长度。 

![](http://mycsdnblog.work/201818262103-c.png)

##### 3.5 重要方法分析

<1> **putVal方法**

HashMap并没有直接提供putVal接口给用户调用，而是提供的put方法，而put方法就是通过putVal来插入元素的。 

![](http://mycsdnblog.work/201818262127-P.png)

putVal方法执行过程可以通过下图来理解： 

![](http://mycsdnblog.work/201818262216-D.png)

①.判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；

②.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；

③.判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；

④.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；

⑤.遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树（只有table的长度大于64的时候才会转换，否则进行扩容），在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

⑥.插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。

源码：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 步骤①：tab为空则创建 
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤②：计算index，并对null做处理  
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 步骤③：节点key存在，直接覆盖value 
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // 步骤④：判断该链为红黑树 
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 步骤⑤：该链为链表 
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 步骤⑥：超过最大容量 就扩容 
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

**HashMap的数据存储实现原理** ：

1. 根据key计算得到key.hash = (h = k.hashCode()) ^ (h >>> 16)；

2. 根据key.hash计算得到桶数组的索引index = key.hash & (table.length - 1)，这样就找到该key的存放位置了：

   ① 如果该位置没有数据，用该数据新生成一个节点保存新数据，返回null；

   ② 如果该位置有数据是一个红黑树，那么执行相应的插入 / 更新操作；

   ③ 如果该位置有数据是一个链表，分两种情况一是该链表没有这个节点，另一个是该链表上有这个节点，注意这里判断的依据是key.hash是否一样：如果该链表没有这个节点，那么采用尾插法新增节点保存新数据，返回null；如果该链表已经有这个节点了，那么找到该节点并更新新数据，返回老数据。

注意：HashMap的put会返回key的上一次保存的数据 （插入相同key的数据）

```java
public static void main(String[] args) {
    Map<String,String> map = new HashMap<String, String>();
    System.out.println(map.put("1", "2"));
    System.out.println(map.put("1", "4"));
    System.out.println(map.put("1", "6"));
}
```

结果：

![](http://mycsdnblog.work/201818262221-6.png)

<2>**getNode方法** 

HashMap同样并没有直接提供getNode接口给用户调用，而是提供的get方法，而get方法就是通过getNode来取得元素的。 

![](http://mycsdnblog.work/201818262223-s.png)

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // table已经初始化，长度大于0，根据hash寻找table中的项也不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 桶中第一项(数组元素)相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个结点
        if ((e = first.next) != null) {
            // 为红黑树结点
            if (first instanceof TreeNode)
                // 在红黑树中查找
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 否则，在链表中查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

<3>**removeNode方法**

![](http://mycsdnblog.work/201818272317-Z.png)

HashMap同样并没有直接提供removeNode接口给用户调用，而是提供的remove方法，而remove方法就是通过removeNode来删除元素的。

```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        // 1. 定位桶位置
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        // 如果键的值与链表第一个节点相等，则将 node 指向该节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {  
            // 如果是 TreeNode 类型，调用红黑树的查找逻辑定位待删除节点
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 2. 遍历链表，找到待删除节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        
        // 3. 删除节点，并修复链表或红黑树
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

<4>**size()方法**

![](http://mycsdnblog.work/201818281458-J.png)

<5>**resize()方法**

①.在jdk1.8中，resize方法是在HashMap中的键值对大于阈值时或者初始化时，就调用resize方法进行扩容；

②.**每次扩展的时候，都是扩展2倍；**

③.**扩展后Node对象的位置要么在原位置，要么移动到原位置加原长度的位置**。

**源码分析：**

第一部分，生成新hash桶数组

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;//oldTab指向hash桶数组
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {//如果oldCap不为空的话，就是hash桶数组不为空
        if (oldCap >= MAXIMUM_CAPACITY) {//如果大于最大容量了，就赋值为整数最大的阀值
            threshold = Integer.MAX_VALUE;
            return oldTab;//返回
        }//如果当前hash桶数组的长度在扩容后仍然小于最大容量 并且oldCap大于默认值16
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold 双倍扩容阀值threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//新建hash桶数组
    table = newTab;//将新数组的值复制给旧的hash桶数组
```

第二部分，链表拆分，重点在 `e.hash & oldCap` 

```java
if (oldTab != null) {//进行扩容操作，复制Node对象值到新的hash桶数组
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {//如果旧的hash桶数组在j结点处不为空，复制给e
                oldTab[j] = null;//将旧的hash桶数组在j结点处设置为空，方便gc
                if (e.next == null)//如果e后面没有Node结点
                    newTab[e.hash & (newCap - 1)] = e;//直接对e的hash值对新的数组长度求模获得存储位置
                else if (e instanceof TreeNode)//如果e是红黑树的类型，那么添加到红黑树中
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {  // 如果 table[j] 后是一个链表 ，将原链表拆分为两条链，分别放到newTab中 
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;//将Node结点的next赋值给next
                        if ((e.hash & oldCap) == 0) {//如果结点e的hash值与原hash桶数组的长度作与运算为0
                            if (loTail == null)//如果loTail为null
                                loHead = e;//将e结点赋值给loHead
                            else
                                loTail.next = e;//否则将e赋值给loTail.next
                            loTail = e;//然后将e复制给loTail
                        }
                        else {//如果结点e的hash值与原hash桶数组的长度作与运算不为0
                            if (hiTail == null)//如果hiTail为null
                                hiHead = e;//将e赋值给hiHead
                            else
                                hiTail.next = e;//如果hiTail不为空，将e复制给hiTail.next
                            hiTail = e;//将e复制个hiTail
                        }
                    } while ((e = next) != null);//直到e为空
                    if (loTail != null) {//如果loTail不为空
                        loTail.next = null;//将loTail.next设置为空
                        newTab[j] = loHead;//将loHead赋值给新的hash桶数组[j]处
                    }
                    if (hiTail != null) {//如果hiTail不为空
                        hiTail.next = null;//将hiTail.next赋值为空
                        newTab[j + oldCap] = hiHead;//将hiHead赋值给新的hash桶数组[j+旧hash桶数组长度]
                    }
                }
            }
        }
    }
    return newTab;
}
```

![](http://mycsdnblog.work/201818262251-F.png)

- 是未扩容时， `key1` 和 `key2` 得出的 hash & (n - 1) 均为 5。
- 是扩容之后，`key1` 计算出的 newTab 角标依旧为 5，但是 `key2` 由于 扩容， 得出的角标 加了 16，即21， 16是oldTab的length，再来看`e.hash & oldCap`，oldCap(oldTab.length即n) 本身为 `0000 0000 0000 0000 0000 0000 0001 0000` ，这个与运算可以得出扩容后哪些key 在 扩容新增位是1，哪些是0，一个位运算替换了`rehash`过程，大概扩容的过程如下：

 ![](http://mycsdnblog.work/201818262302-4.png)

#### 四、遍历方式

##### 4.1 遍历HashMap的键值对

```java
// 假设map是HashMap对象
// map中的key是String类型，value是Integer类型
Integer value = null;
Iterator iter = map.entrySet().iterator();
while(iter.hasNext()) {
    Map.Entry entry = (Map.Entry)iter.next();
    // 获取key
    key = (String)entry.getKey();
    // 获取value
    value = (Integer)entry.getValue();
}
```

##### 4.2 遍历HashMap的键

```java
// 假设map是HashMap对象
// map中的key是String类型，value是Integer类型
String key = null;
Integer integ = null;
Iterator iter = map.keySet().iterator();
while (iter.hasNext()) {
        // 获取key
    key = (String)iter.next();
        // 根据key，获取value
    integ = (Integer)map.get(key);
}
```

##### 4.3 遍历HashMap的值

```java
// 假设map是HashMap对象
// map中的key是String类型，value是Integer类型
Integer value = null;
Collection c = map.values();
Iterator iter= c.iterator();
while (iter.hasNext()) {
    value = (Integer)iter.next();
}
```

#### 五、线程安全问题

JDK1.7 HashMap在多线程的扩容时确实会出现循环引用，导致下次get时死循环的问题，具体可以参考[HashMap死循环问题](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Fxuefeng0707%2Farticle%2Fdetails%2F40797085)。很多文章在说到死循环时都以JDK1.7来举例，其实JDK1.8的优化已经避免了死循环这个问题，但是会造成数据丢失。

数据丢失问题，主要发生在resize内部。创建 thread1 和 thread2 去添加数据，此时都在resize，两个线程分别创建了两个newTable，并且thread1在`table = newTab;`处调度到thread2(没有给table赋值)，等待thread2扩容之后再调度回thread1，注意，扩容时`oldTab[j] = null;` 也就将 oldTable中都清掉了，当回到thread1时，将table指向thread1的newTable，但访问oldTable中的元素全部为null，所以造成了数据丢失。

**代码：**

```java
package com.util.hashmap;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

/**
 * @Author: 98050
 * @Time: 2018-12-23 17:14
 * @Feature: HashMap线程安全问题
 */
public class Test {

    public static void main(String[] args) throws InterruptedException {
        final Map<String,String> map = new HashMap<String, String>();


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
        System.out.println("set的大小：" + map.size());
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

![](http://mycsdnblog.work/201818262332-g.png)

##### 5.1 线程安全1（放入）

插入数据的时候，两个线程同时获取到空的桶位，那么就会发生覆盖现象

##### 5.2 线程安全2（扩容）

数据丢失问题，主要发生在resize内部。创建 thread1 和 thread2 去添加数据，此时都在resize，两个线程分别创建了两个newTable，并且thread1在`table = newTab;`处调度到thread2(没有给table赋值)，等待thread2扩容之后再调度回thread1，注意，扩容时`oldTab[j] = null;` 也就将 oldTable中都清掉了，当回到thread1时，将table指向thread1的newTable，但访问oldTable中的元素全部为null，所以造成了数据丢失。

##### 5.3 线程安全3（删除）

覆盖修改

#### 六、对HashMap查找时间复杂度O(1)的理解

因为hashMap内部维护了一个Entry数组，hashcode即数组下标，根据key.hashcode()即可在数组中get到Entry对象，即O(1)。当然，这是理想情况。
倘若数据量大，则可能发生hash碰撞，即一个hashcode可能对应多个key，这时候这个Entry数组中的元素就不是Entry了，而是一个Entry链表。调用map.get(key)的时候，遇到了链表，则会遍历链表，调用equals方法比较key。当然，jdk8做了优化，链表长度超过8的时候，会转变为红黑树结构。当然，除了数据量之外，发生hash碰撞的概率还跟负载因子loadFactor有关。

#### 七、HashMap如何存放null key和null value

1.先在table[0]的链表中寻找null key，如果有null key就直接覆盖原来的value，返回原来的value； 

2.如果在table[0]中没有找到，就进行头插，但是要先判断是否要扩容，需要就扩容，然后进行头插，此时table[0]就是新插入的null key Entry了。

**table的类型是Node，Node实现了Map.Entry**

```java
package com.test;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.util.HashMap;
import java.util.Map;

/**
 * @Author: 98050
 * @Time: 2019-05-22 16:53
 * @Feature:
 */
public class Test {

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, NoSuchMethodException, InvocationTargetException, InstantiationException {
        HashMap<Integer,Integer> map = new HashMap<Integer, Integer>();
        map.put(null, null);
        System.out.println(map.get(null));

        map.put(null, 2);
        System.out.println(map.get(null));

        map.put(1, null);
        System.out.println(map.get(1));

        map.put(2, 3);
        map.put(4, 5);
        map.put(6, 7);
        map.put(8, 9);

        System.out.println("------------------------");
        Class<? extends HashMap> mapClass = map.getClass();
        Field field = mapClass.getDeclaredField("table");
        field.setAccessible(true);
        Map.Entry[] entries = (Map.Entry[]) field.get(map);
        for (int i = 0; i < entries.length; i++) {
            System.out.println("第" + i + "个桶：" + entries[i]);
        }
    }
}
```

![](http://mycsdnblog.work/201919221728-C.png)

#### 八、HashMap1.7和1.8的区别

（1）**JDK1.7用的是头插法，而JDK1.8及之后使用的都是尾插法，那么他们为什么要这样做呢？因为JDK1.7是用单链表进行的纵向延伸，当采用头插法时会容易出现逆序且环形链表死循环问题。但是在JDK1.8之后是因为加入了红黑树使用尾插法，能够避免出现逆序且链表死循环的问题。**

（2）扩容后数据存储位置的计算方式也不一样：

**1. 在JDK1.7的时候是直接用hash值和需要扩容的二进制数进行&（这里就是为什么扩容的时候为啥一定必须是2的多少次幂的原因所在，因为如果只有2的n次幂的情况时最后一位二进制数才一定是1，这样能最大程度减少hash碰撞）（hash值 & length-1）**

**2、而在JDK1.8的时候直接用了JDK1.7的时候计算的规律，也就是扩容前的原始位置+扩容的大小值=JDK1.8的计算方式，而不再是JDK1.7的那种异或的方法。但是这种方式就相当于只需要判断Hash值的新增参与运算的位是0还是1就直接迅速计算出了扩容后的储存方式。**

![](https://img-blog.csdn.net/20180905105129591?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NTIwMjM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1.7死循环问题https://blog.csdn.net/maohoo/article/details/81531925

#### 九、为什么链表长度超过8会转换成红黑树

HashMap在jdk1.8之后引入了红黑树的概念，表示若桶中链表元素超过8时，会自动转化成红黑树；若桶中元素小于等于6时，树结构还原成链表形式。

原因：

　　红黑树的平均查找长度是log(n)，长度为8，查找长度为log(8)=3，链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，这才有转换成树的必要；链表长度如果是小于等于6，6/2=3，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短。

还有选择6和8的原因是：

　　中间有个差值7可以防止链表和树之间频繁的转换。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低。