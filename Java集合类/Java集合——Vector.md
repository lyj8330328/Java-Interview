# 一、介绍

 Vector 是**矢量队列**，它是JDK1.0版本添加的类。继承于AbstractList，实现了List, RandomAccess, Cloneable这些接口。
Vector 继承了AbstractList，实现了List；所以，**它是一个队列，支持相关的添加、删除、修改、遍历等功能**。
Vector 实现了RandmoAccess接口，即**提供了随机访问功能**。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。在Vector中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。
Vector 实现了Cloneable接口，即实现clone()函数。它能被克隆。

和ArrayList不同，**Vector中的操作是线程安全的**。  

# 二、数据结构

![](http://mycsdnblog.work/201919122150-x.png)

Vector的数据结构和ArrayList差不多，它包含了3个成员变量：elementData , elementCount， capacityIncrement。

(01) elementData 是"Object[]类型的数组"，它保存了添加到Vector中的元素。elementData是个动态数组，如果初始化Vector时，没指定动态数组的>大小，则使用默认大小10。随着Vector中元素的增加，Vector的容量也会动态增长，capacityIncrement是与容量增长相关的增长系数，具体的增长方式，请参考源码分析中的ensureCapacity()函数。

(02) elementCount 是动态数组的实际大小。

(03) capacityIncrement 是动态数组的增长系数。如果在创建Vector时，指定了capacityIncrement的大小；则，每次当Vector中动态数组容量增加时，增加的大小都是capacityIncrement。

# 三、源码解析

## 3.1 添加

![](http://mycsdnblog.work/201919122152-Z.png)

因为加了synchronized关键字，所以添加操作是线程安全的。添加元素前通过调用`ensureCapacityHelper`方法来保证容量大小：

![](http://mycsdnblog.work/201919122155-W.png)

![](http://mycsdnblog.work/201919122156-l.png)

如果数组越界，说明数组放不下了，需要进行扩容，`grow`方法：

![](http://mycsdnblog.work/201919122157-x.png)

如果指定了`capacityIncrement`增长系数的话，就做相应的扩大，否则扩容一倍

## 3.2 删除

![](http://mycsdnblog.work/201919122158-W.png)

主要方法就是`removeElement`：

![](http://mycsdnblog.work/201919122158-l.png)

使用synchronized关键字，所以线程安全。找到要移除元素的下标，然后判断位置是否合理，合理的话就进行移除。

## 3.3 获取

![](http://mycsdnblog.work/201919122200-t.png)

获取的方法上也加了synchronized，验证范围是否合理，合理的话返回对应的值。

# 四、遍历方式

- 迭代器
- 随机访问
- 增强for循环
- **Enumeration**遍历

# 五、示例