# 一、Stack介绍

Stack是栈。它的特性是：**先进后出**(FILO, First In Last Out)。

java工具包中的Stack是继承于Vector(矢量队列)的，由于Vector是通过数组实现的，这就意味着，**Stack也是通过数组实现的**，**而非链表**。当然，我们也可以将LinkedList当作栈来使用！

![](http://mycsdnblog.work/201919271441-x.png)

重要API：

![](http://mycsdnblog.work/201919271441-9.png)

# 二、源码分析

## 2.1 push

```java
public E push(E item) {
    addElement(item);

    return item;
}
```

调用父类中的addElement方法

```java
public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = obj;
}
```

## 2.2 peek

```java
public synchronized E peek() {
    int     len = size();

    if (len == 0)
        throw new EmptyStackException();
    return elementAt(len - 1);
}
```
```java
public synchronized E elementAt(int index) {
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
    }

    return elementData(index);
}
```

```java
E elementData(int index) {
    return (E) elementData[index];
}
```

## 2.3 pop

```java
public synchronized E pop() {
    E       obj;
    int     len = size();

    obj = peek();
    removeElementAt(len - 1);

    return obj;
}
```

调用父类中的removeElementAt方法

```java
public synchronized void removeElementAt(int index) {
    modCount++;
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                 elementCount);
    }
    else if (index < 0) {
        throw new ArrayIndexOutOfBoundsException(index);
    }
    int j = elementCount - index - 1;
    if (j > 0) {
        System.arraycopy(elementData, index + 1, elementData, index, j);
    }
    elementCount--;
    elementData[elementCount] = null; /* to let gc do its work */
}
```

## 2.4 empty

```java
public boolean empty() {
    return size() == 0;
}
```

