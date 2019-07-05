# 一、synchronized

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

![1559201658664](http://mycsdnblog.work/201919301535-V.png)



![1559201686917](http://mycsdnblog.work/201919301535-8.png)

