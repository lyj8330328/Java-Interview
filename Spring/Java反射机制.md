# 一、什么是Java反射

就是类正在运行，动态获取这个类的所有信息。

# 二、反射机制的作用

反编译：.class-->.java

通过反射机制访问java对象的属性，方法，构造方法等；

# 三、应用

Jdbc 加载驱动-----

Spring IOC

框架 

# 四、反射机制获取类的三种方法

```java
public class Test {

    public static void main(String[] args) throws ClassNotFoundException {
        Class class1 = Class.forName("Emplyee");
        Class class2 = Employee.class;
        Class class3 = new Employee().getClass();
    }
}
```

# 五、API

| 方法名称              | 作用               |
| --------------------- | ------------------ |
| getDeclaredMethods [] | 获取该类的所有方法 |
| getReturnType()       | 获取该类的返回值   |
| getParameterTypes()   | 获取传入参数       |
| getDeclaredFields()   | 获取该类的所有字段 |
| setAccessible         | 允许访问私有成员   |

# 六、禁止使用反射机制初始化

将构造函数为私有化

# 七、Class.forName()和ClassLoader.loadClass的区别

## 7.1 Java类的装载过程

![](http://mycsdnblog.work/201919160937-r.png)

1.装载：通过类的全限定名获取二进制字节流，将二进制字节流转换成方法区中的运行时数据结构，在内存中生成Java.lang.class对象； 

2.链接：执行下面的校验、准备和解析步骤，其中解析步骤是可以选择的； 

- 　　校验：检查导入类或接口的二进制数据的正确性；（文件格式验证，元数据验证，字节码验证，符号引用验证） 

- 　　准备：给类的静态变量分配并初始化存储空间； 

- 　　解析：将常量池中的符号引用转成直接引用； 


3.初始化：激活类的静态变量的初始化Java代码和静态Java代码块，并初始化程序员设置的变量值。

## 7.2 **Class.forName()和ClassLoader.loadClass**

Class.forName(className)方法，内部实际调用的方法是  Class.forName(className,true,classloader);

第2个boolean参数表示类是否需要初始化，  Class.forName(className)默认是需要初始化。

一旦初始化，就会触发目标对象的 static块代码执行，static参数也也会被再次初始化。

​    

ClassLoader.getSystemClassLoader().loadClass(className)方法，内部实际调用的方法是  ClassLoader.getSystemClassLoader().loadClass(className,false);

第2个 boolean参数，表示目标对象是否进行链接，false表示不进行链接，由上面介绍可以，

不进行链接意味着不进行包括初始化等一些列步骤，那么静态块和静态对象就不会得到执行

## 7.3 为什么数据库连接要使用Class.forName(className)

JDBC  Driver源码如下,因此使用Class.forName(classname)才能在反射回去类的时候执行static块。

```java
static {
    try {
        java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
    }
}
```

