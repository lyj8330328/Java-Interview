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

