# 一、MyBatis与数据库的交互方式

- 使用传统的MyBatis提供的API
- 使用Mapper接口

# 二、使用Mapper接口

MyBatis 将配置文件中的每一个<mapper> 节点抽象为一个 Mapper 接口：

这个接口中声明的方法和<mapper> 节点中的<select|update|delete|insert> 节点项对应，即<select|update|delete|insert> 节点的id值为Mapper 接口中的方法名称，parameterType 值表示Mapper 对应方法的入参类型，而resultMap 值则对应了Mapper 接口表示的返回值类型或者返回结果集的元素类型。

![1551851960403](http://mycsdnblog.work/201919061609-A.png)

根据MyBatis 的配置规范配置好后，通过SqlSession.getMapper(XXXMapper.class)方法，MyBatis 会根据相应的接口声明的方法信息，通过动态代理机制生成一个Mapper 实例，我们使用Mapper接口的某一个方法时，MyBatis会根据这个方法的方法名和参数类型，确定Statement Id，底层还是通过SqlSession.select("statementId",parameterObject);或者SqlSession.update("statementId",parameterObject); 等等来实现对数据库的操作，MyBatis引用Mapper 接口这种调用方式，纯粹是为了满足面向接口编程的需要。（其实还有一个原因是在于，面向接口的编程，使得用户在接口上可以使用注解来配置SQL语句，这样就可以脱离XML配置文件，实现“0配置”）。

# 三、数据处理层

数据处理层可以说是MyBatis的核心，从大的方面上讲，它要完成两个功能：

- 通过传入参数构建动态SQL语句；
- SQL语句的执行以及封装查询结果集成List<E>

# 四、框架支撑

## 4.1 事务管理机制

事务管理机制对于ORM框架而言是不可缺少的一部分，事务管理机制的质量也是考量一个ORM框架是否优秀的一个标准。

## 4.2 连接池管理机制

由于创建一个数据库连接所占用的资源比较大，对于数据吞吐量大和访问量非常大的应用而言，连接池的设计就显得非常重要。

## 4.3 缓存机制

为了提高数据利用率和减小服务器和数据库的压力，MyBatis 会对于一些查询提供会话级别的数据缓存，会将对某一次查询，放置到SqlSession 中，在允许的时间间隔内，对于完全相同的查询，MyBatis会直接将缓存结果返回给用户，而不用再到数据库中查找

## 4.4 Sql语句配置方式

传统的MyBatis 配置SQL语句方式就是使用XML文件进行配置的，但是这种方式不能很好地支持面向接口编程的理念，为了支持面向接口的编程，MyBatis 引入了Mapper接口的概念，面向接口的引入，对使用注解来配置SQL语句成为可能，用户只需要在接口上添加必要的注解即可，不用再去配置XML文件了，但是，目前的MyBatis 只是对注解配置SQL语句提供了有限的支持，某些高级功能还是要依赖XML配置文件配置SQL 语句。

# 五、MyBatis主要构件

从MyBatis代码实现的角度来看，MyBatis的主要的核心部件有以下几个：

SqlSession：作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能；

Executor：MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护；

StatementHandler：封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。

ParameterHandler：负责对用户传递的参数转换成JDBC Statement 所需要的参数；

ResultSetHandler：负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；

TypeHandler：负责java数据类型和jdbc数据类型之间的映射和转换；

MappedStatement：MappedStatement维护了一条<select|update|delete|insert>节点的封装；

SqlSource：负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回；

BoundSql：表示动态生成的SQL语句以及相应的参数信息；

Configuration：MyBatis所有的配置信息都维持在Configuration对象之中；

![1551852827882](http://mycsdnblog.work/201919061608-7.png)

# 六、手写MyBatis

## 6.1 工具类JDBCUtils

负责与数据库交互

```java
package com.example.mybatis.utils;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public final class JDBCUtils {

   private static String connect;
   private static String driverClassName;
   private static String URL;
   private static String username;
   private static String password;
   private static boolean autoCommit;

   /** 声明一个 Connection类型的静态属性，用来缓存一个已经存在的连接对象 */
   private static Connection conn;

   static {
      config();
   }

   /**
    * 开头配置自己的数据库信息
    */
   private static void config() {
      /*
       * 获取驱动
       */
      driverClassName = "com.mysql.jdbc.Driver";
      /*
       * 获取URL
       */
      URL = "jdbc:mysql://localhost:3306/test2?useUnicode=true&characterEncoding=utf8";
      /*
       * 获取用户名
       */
      username = "root";
      /*
       * 获取密码
       */
      password = "123456";
      /*
       * 设置是否自动提交，一般为false不用改
       */
      autoCommit = false;

   }

   /**
    * 载入数据库驱动类
    */
   private static boolean load() {
      try {
         Class.forName(driverClassName);
         return true;
      } catch (ClassNotFoundException e) {
         System.out.println("驱动类 " + driverClassName + " 加载失败");
      }

      return false;
   }

   /**
    * 专门检查缓存的连接是否不可以被使用 ，不可以被使用的话，就返回 true
    */
   private static boolean invalid() {
      if (conn != null) {
         try {
            if (conn.isClosed() || !conn.isValid(3)) {
               return true;
               /*
                * isValid方法是判断Connection是否有效,如果连接尚未关闭并且仍然有效，则返回 true
                */
            }
         } catch (SQLException e) {
            e.printStackTrace();
         }
         /*
          * conn 既不是 null 且也没有关闭 ，且 isValid 返回 true，说明是可以使用的 ( 返回 false )
          */
         return false;
      } else {
         return true;
      }
   }

   /**
    * 建立数据库连接
    */
   public static Connection connect() {
      if (invalid()) { /* invalid为true时，说明连接是失败的 */
         /* 加载驱动 */
         load();
         try {
            /* 建立连接 */
            conn = DriverManager.getConnection(URL, username, password);
         } catch (SQLException e) {
            System.out.println("建立 " + connect + " 数据库连接失败 , " + e.getMessage());
         }
      }
      return conn;
   }

   /**
    * 设置是否自动提交事务
    **/
   public static void transaction() {

      try {
         conn.setAutoCommit(autoCommit);
      } catch (SQLException e) {
         System.out.println("设置事务的提交方式为 : " + (autoCommit ? "自动提交" : "手动提交") + " 时失败: " + e.getMessage());
      }

   }

   /**
    * 创建 Statement 对象
    */
   public static Statement statement() {
      Statement st = null;
      connect();
      /* 如果连接是无效的就重新连接 */
      transaction();
      /* 设置事务的提交方式 */
      try {
         st = conn.createStatement();
      } catch (SQLException e) {
         System.out.println("创建 Statement 对象失败: " + e.getMessage());
      }

      return st;
   }

   /**
    * 根据给定的带参数占位符的SQL语句，创建 PreparedStatement 对象
    * 
    * @param SQL
    *            带参数占位符的SQL语句
    * @return 返回相应的 PreparedStatement 对象
    */
   private static PreparedStatement prepare(String SQL, boolean autoGeneratedKeys) {

      PreparedStatement ps = null;
      connect();
      /* 如果连接是无效的就重新连接 */
      transaction();
      /* 设置事务的提交方式 */
      try {
         if (autoGeneratedKeys) {
            ps = conn.prepareStatement(SQL, Statement.RETURN_GENERATED_KEYS);
         } else {
            ps = conn.prepareStatement(SQL);
         }
      } catch (SQLException e) {
         System.out.println("创建 PreparedStatement 对象失败: " + e.getMessage());
      }

      return ps;

   }

   public static ResultSet query(String SQL, List<Object> params) {

      if (SQL == null || SQL.trim().isEmpty() || !SQL.trim().toLowerCase().startsWith("select")) {
         throw new RuntimeException("你的SQL语句为空或不是查询语句");
      }
      ResultSet rs = null;
      if (params.size() > 0) {
         /* 说明 有参数 传入，就需要处理参数 */
         PreparedStatement ps = prepare(SQL, false);
         try {
            for (int i = 0; i < params.size(); i++) {
               ps.setObject(i + 1, params.get(i));
            }
            rs = ps.executeQuery();
         } catch (SQLException e) {
            System.out.println("执行SQL失败: " + e.getMessage());
         }
      } else {
         /* 说明没有传入任何参数 */
         Statement st = statement();
         try {
            rs = st.executeQuery(SQL); // 直接执行不带参数的 SQL 语句
         } catch (SQLException e) {
            System.out.println("执行SQL失败: " + e.getMessage());
         }
      }

      return rs;

   }

   private static Object typeof(Object o) {
      Object r = o;

      if (o instanceof java.sql.Timestamp) {
         return r;
      }
      // 将 java.util.Date 转成 java.sql.Date
      if (o instanceof java.util.Date) {
         java.util.Date d = (java.util.Date) o;
         r = new java.sql.Date(d.getTime());
         return r;
      }
      // 将 Character 或 char 变成 String
      if (o instanceof Character || o.getClass() == char.class) {
         r = String.valueOf(o);
         return r;
      }
      return r;
   }

   public static boolean execute(String SQL, Object... params) {
      if (SQL == null || SQL.trim().isEmpty() || SQL.trim().toLowerCase().startsWith("select")) {
         throw new RuntimeException("你的SQL语句为空或有错");
      }
      boolean r = false;
      /* 表示 执行 DDL 或 DML 操作是否成功的一个标识变量 */

      /* 获得 被执行的 SQL 语句的 前缀 */
      SQL = SQL.trim();
      SQL = SQL.toLowerCase();
      String prefix = SQL.substring(0, SQL.indexOf(" "));
      String operation = ""; // 用来保存操作类型的 变量
      // 根据前缀 确定操作
      switch (prefix) {
      case "create":
         operation = "create table";
         break;
      case "alter":
         operation = "update table";
         break;
      case "drop":
         operation = "drop table";
         break;
      case "truncate":
         operation = "truncate table";
         break;
      case "insert":
         operation = "insert :";
         break;
      case "update":
         operation = "update :";
         break;
      case "delete":
         operation = "delete :";
         break;
         default:
      }
      if (params.length > 0) { // 说明有参数
         PreparedStatement ps = prepare(SQL, false);
         Connection c = null;
         try {
            c = ps.getConnection();
         } catch (SQLException e) {
            e.printStackTrace();
         }
         try {
            for (int i = 0; i < params.length; i++) {
               Object p = params[i];
               p = typeof(p);
               ps.setObject(i + 1, p);
            }
            ps.executeUpdate();
            commit(c);
            r = true;
         } catch (SQLException e) {
            System.out.println(operation + " 失败: " + e.getMessage());
            rollback(c);
         }

      } else { // 说明没有参数

         Statement st = statement();
         Connection c = null;
         try {
            c = st.getConnection();
         } catch (SQLException e) {
            e.printStackTrace();
         }
         // 执行 DDL 或 DML 语句，并返回执行结果
         try {
            st.executeUpdate(SQL);
            commit(c); // 提交事务
            r = true;
         } catch (SQLException e) {
            System.out.println(operation + " 失败: " + e.getMessage());
            rollback(c); // 回滚事务
         }
      }
      return r;
   }

   /*
    * 
    * @param SQL 需要执行的 INSERT 语句
    * 
    * @param autoGeneratedKeys 指示是否需要返回由数据库产生的键
    * 
    * @param params 将要执行的SQL语句中包含的参数占位符的 参数值
    * 
    * @return 如果指定 autoGeneratedKeys 为 true 则返回由数据库产生的键； 如果指定 autoGeneratedKeys
    * 为 false 则返回受当前SQL影响的记录数目
    */
   public static int insert(String SQL, boolean autoGeneratedKeys, List<Object> params) {
      int var = -1;
      if (SQL == null || SQL.trim().isEmpty()) {
         throw new RuntimeException("你没有指定SQL语句，请检查是否指定了需要执行的SQL语句");
      }
      // 如果不是 insert 开头开头的语句
      if (!SQL.trim().toLowerCase().startsWith("insert")) {
         System.out.println(SQL.toLowerCase());
         throw new RuntimeException("你指定的SQL语句不是插入语句，请检查你的SQL语句");
      }
      // 获得 被执行的 SQL 语句的 前缀 ( 第一个单词 )
      SQL = SQL.trim();
      SQL = SQL.toLowerCase();
      if (params.size() > 0) { // 说明有参数
         PreparedStatement ps = prepare(SQL, autoGeneratedKeys);
         Connection c = null;
         try {
            c = ps.getConnection(); // 从 PreparedStatement 对象中获得 它对应的连接对象
         } catch (SQLException e) {
            e.printStackTrace();
         }
         try {
            for (int i = 0; i < params.size(); i++) {
               Object p = params.get(i);
               p = typeof(p);
               ps.setObject(i + 1, p);
            }
            int count = ps.executeUpdate();
            if (autoGeneratedKeys) { // 如果希望获得数据库产生的键
               ResultSet rs = ps.getGeneratedKeys(); // 获得数据库产生的键集
               if (rs.next()) { // 因为是保存的是单条记录，因此至多返回一个键
                  var = rs.getInt(1); // 获得值并赋值给 var 变量
               }
            } else {
               var = count; // 如果不需要获得，则将受SQL影像的记录数赋值给 var 变量
            }
            commit(c);
         } catch (SQLException e) {
            System.out.println("数据保存失败: " + e.getMessage());
            rollback(c);
         }
      } else { // 说明没有参数
         Statement st = statement();
         Connection c = null;
         try {
            c = st.getConnection(); // 从 Statement 对象中获得 它对应的连接对象
         } catch (SQLException e) {
            e.printStackTrace();
         }
         // 执行 DDL 或 DML 语句，并返回执行结果
         try {
            int count = st.executeUpdate(SQL);
            if (autoGeneratedKeys) { // 如果企望获得数据库产生的键
               ResultSet rs = st.getGeneratedKeys(); // 获得数据库产生的键集
               if (rs.next()) { // 因为是保存的是单条记录，因此至多返回一个键
                  var = rs.getInt(1); // 获得值并赋值给 var 变量
               }
            } else {
               var = count; // 如果不需要获得，则将受SQL影像的记录数赋值给 var 变量
            }
            commit(c); // 提交事务
         } catch (SQLException e) {
            System.out.println("数据保存失败: " + e.getMessage());
            rollback(c); // 回滚事务
         }
      }
      return var;
   }

   /** 提交事务 */
   private static void commit(Connection c) {
      if (c != null && !autoCommit) {
         try {
            c.commit();
         } catch (SQLException e) {
            e.printStackTrace();
         }
      }
   }

   /** 回滚事务 */
   private static void rollback(Connection c) {
      if (c != null && !autoCommit) {
         try {
            c.rollback();
         } catch (SQLException e) {
            e.printStackTrace();
         }
      }
   }

   /**
    * 释放资源
    **/
   public static void release(Object cloaseable) {

      if (cloaseable != null) {

         if (cloaseable instanceof ResultSet) {
            ResultSet rs = (ResultSet) cloaseable;
            try {
               rs.close();
            } catch (SQLException e) {
               e.printStackTrace();
            }
         }

         if (cloaseable instanceof Statement) {
            Statement st = (Statement) cloaseable;
            try {
               st.close();
            } catch (SQLException e) {
               e.printStackTrace();
            }
         }

         if (cloaseable instanceof Connection) {
            Connection c = (Connection) cloaseable;
            try {
               c.close();
            } catch (SQLException e) {
               e.printStackTrace();
            }
         }

      }

   }

}
```

## 6.2 定义注解

定义三个注解：@Insert，@Param，@Select

```java
package com.example.mybatis.annotation;

import java.lang.annotation.*;

/**
 * @Author: 98050
 * @Time: 2019-03-05 20:16
 * @Feature: 自定义插入注解
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Insert {
    String value() default "";
}
```

```java
package com.example.mybatis.annotation;

import java.lang.annotation.*;

/**
 * @Author: 98050
 * @Time: 2019-03-05 20:19
 * @Feature: 自定义参数注解
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
public @interface Param {
    String value();
}
```

```java
package com.example.mybatis.annotation;

import java.lang.annotation.*;

/**
 * @Author: 98050
 * @Time: 2019-03-05 22:59
 * @Feature: 自定义查询注解
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Select {
    String value();
}
```

## 6.3 Java动态代理

MyBatis在使用的时候是在Mapper文件上加一个@Mapper注解，这个和前面手写Spring中的Service注解原理是一样的，当加载上下文的时候，会扫描带@Mapper的类，然后通过Java反射技术将其实例化，放入专门负责MyBatis的IOC容器中，然后在Service层通过@Autowired进行注入。在这里为了方便，就不去重写@Mapper，我们最终目的就是获取一个Mapper接口的对象，那么就直接使用动态代理。

```java
package com.example.mybatis.aop;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @Author: 98050
 * @Time: 2019-03-06 14:49
 * @Feature:
 */
public class MybatisInvocationHandler implements InvocationHandler {
    /**
     * 代理的对象
     */
    private Object object;

    public MybatisInvocationHandler(Object object) {
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("动态代理");
        return 1;
    }
}
```

```java
package com.example.mybatis.sql;

import com.example.mybatis.aop.MybatisInvocationHandler;
import com.example.mybatis.aop.MybatisInvocationHandler2;

import java.lang.reflect.Proxy;

/**
 * @Author: 98050
 * @Time: 2019-03-05 20:57
 * @Feature:
 */
public class SqlSession {
    /**
     * 加载Mapper文件
     * @param c
     * @param <T>
     * @return
     */
    public static <T> T getMapper(Class c){
        MybatisInvocationHandler handler = new MybatisInvocationHandler(c);
        return (T)Proxy.newProxyInstance(c.getClassLoader(), new Class[]{c}, handler);
    }
}
```

创建Mapper文件：

```java
package com.example.mybatis.mapper;

/**
 * @Author: 98050
 * @Time: 2019-03-06 14:56
 * @Feature:
 */
public interface UserMapper {

    int insert();
}
```

测试：

```java
package com.example.mybatis;

import com.example.mybatis.mapper.UserMapper;
import com.example.mybatis.mapper.UserMapper2;
import com.example.mybatis.pojo.User;
import com.example.mybatis.sql.SqlSession;

import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-03-05 20:16
 * @Feature:
 */
public class Test {

    public static void main(String[] args) {
        UserMapper userMapper = SqlSession.getMapper(UserMapper.class);
        System.out.println(userMapper.insert());
    }
}
```

结果：

![](http://mycsdnblog.work/201919061509-v.png)

动态代理完成后，我们需要做的就是获取Mapper接口中方法上的注解，然后替换参数，通过JDBCUtils来执行Sql语句。

## 6.4 Mapper接口

```java
package com.example.mybatis.mapper;

import com.example.mybatis.annotation.Insert;
import com.example.mybatis.annotation.Param;
import com.example.mybatis.annotation.Select;
import com.example.mybatis.pojo.User;

import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-03-05 20:18
 * @Feature:
 */
public interface UserMapper {
    /**
     * 用户插入
     * @param name
     * @param age
     * @return
     */
    @Insert("insert into user(name,age) values(#{name},#{age})")
    int insertUser(@Param("name") String name,@Param("age") Integer age);

    /**
     * 查询
     * @param name
     * @param age
     * @return
     */
    @Select("select * from user where name = #{name} and age = #{age}")
    User selectUser(@Param("name") String name,@Param("age") Integer age);

    /**
     * 根据年龄查询
     * @param age
     * @return
     */
    @Select("select * from user where age = #{age}")
    List<User> selectUserList(@Param("age") Integer age);
}
```

三个方法，用的都是自定义注解。我们需要做的就是获取注解中的Sql语句，将参数替换成`?`，然后通过传入的参数构造参数列表，最后使用jdbc执行sql语句返回结果。所以主要操作还是在MybatisInvocationHandler中的invoke方法中进行。

## 6.5 封装sql工具类

将工具类进行封装：

```java
package com.example.mybatis.utils;

import com.example.mybatis.annotation.Param;

import java.lang.reflect.Method;
import java.lang.reflect.Parameter;
import java.util.ArrayList;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @Author: 98050
 * @Time: 2019-03-05 23:04
 * @Feature:
 */
public class SqlUtils {

}
```

在这里主要完成对sql语句的处理

> **替换sql中参数**

```java
/**
 * 替换sql语句中的参数为‘？’
 * @param sql
 * @return
 */
public static String getNewSql(String sql) {
    String newSql = sql;
    int start = 0;
    int end;
    for (int i = 0; i < sql.length(); i++) {
        if (sql.charAt(i) == '{'){
            start = i;
        }
        if (sql.charAt(i) == '}'){
            end = i;
            newSql = newSql.replace(sql.substring(start-1, end + 1),"?");
        }
    }
    return newSql;
}
```

> **sql语句参数匹配**

```java
public static ArrayList<Object> geSqlParamsValue(String sql,Method method, Object[] args) {
    Parameter[] parameters = method.getParameters();
    ConcurrentHashMap<String,Object> map = new ConcurrentHashMap<>();
    for (int i = 0; i < parameters.length; i++) {
        Param param = parameters[i].getDeclaredAnnotation(Param.class);
        if (param != null){
            map.put(param.value(), args[i]);
        }
    }
    ArrayList<Object> sqlParams = new ArrayList<>();
    ArrayList<String> param = getParam(sql);
    for (String p : param){
        sqlParams.add(map.get(p));
    }
    return sqlParams;
}
```

还有一个方法是getParam，获取sql中的参数，获取到后进行参数匹配

```java
/**
 * 返回sql语句中的参数
 * @param sql
 * @return
 */
public static ArrayList<String> getParam(String sql) {
    ArrayList<String> list = new ArrayList<>();
    int start = 0;
    int end;
    for (int i = 0; i < sql.length(); i++) {
        if (sql.charAt(i) == '{'){
            start = i;
        }
        if (sql.charAt(i) == '}'){
            end = i;
            list.add(sql.substring(start + 1, end));
        }
    }
    return list;
}
```

## 6.6 实现@Insert

思路：

- 先判断动态代理的方法上是否有注解@Insert
- 有的话，获取sql语句
- 将sql语句中的参数进行替换，示例如下：`insert into user(name,age) values(#{name},#{age})`替换为`insert into user(name,age) values(？,？)`
- 通过`insertUser`方法传入的参数进行参数匹配，构造参数列表
- 执行sql语句

```java
/**
 * @param proxy ：代理对象
 * @param method ：拦截的方法
 * @param args ： 方法上的参数
 * @return
 * @throws Throwable
 */
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    Object result = new Object();
    String sql;
    Insert insert = method.getDeclaredAnnotation(Insert.class);
    if (insert != null){
        sql = insert.value();
        result = extInsert(sql,method,args);
    }
    return result;
}
```

```java
private Object extInsert(String sql, Method method, Object[] args) {
    String newSql = SqlUtils.getNewSql(sql);
    ArrayList<Object> sqlParams = SqlUtils.geSqlParamsValue(sql,method,args);
    return JDBCUtils.insert(newSql, false, sqlParams);
}
```

## 6.7 实现@Select

思路：

- 判断被代理对象上是否有@Select注解
- 有的话获取sql
- 将sql语句中的参数进行替换
- 执行sql语句
- 结果返回的时候需要分情况：
  - 当返回的是单一对象时，那么通过被代理的方法可以获取方法的返回类型，然后创建对象
  - 当返回的是集合时，那么要得到List中泛型的具体类型，然后构造List返回

在invoke方法中添加：

```java
Select select = method.getDeclaredAnnotation(Select.class);
if (select != null){
    sql = select.value();
    result = extSelect(sql, method, args);
}
```

将具体步骤封装在extSelect中：

```java
private Object extSelect(String sql, Method method, Object[] args) throws SQLException, IllegalAccessException, InstantiationException, NoSuchFieldException, ClassNotFoundException {
    String newSql = SqlUtils.getNewSql(sql);
    ArrayList<Object> sqlParams = SqlUtils.geSqlParamsValue(sql,method,args);
    ResultSet resultSet = JDBCUtils.query(newSql, sqlParams);
    Class<?> returnType = method.getReturnType();
    Object result;
    if (returnType == java.util.List.class){
        result = getResultByList(method,resultSet);
    }else {
        result = getResult(method, resultSet);
    }
    return result;
}
```

判断方法的返回类型是否是list，如果是的话就调用`getResultByList`方法进行处理，不是就调用

`getResult`方法处理。

> **getResultByList**

```java 
private Object getResultByList(Method method, ResultSet resultSet) throws IllegalAccessException, InstantiationException, SQLException {
    Type genericReturnType = method.getGenericReturnType();
    ParameterizedType pt = (ParameterizedType) genericReturnType;
    Type[] actualTypeArguments = pt.getActualTypeArguments();
    Class<?> c = (Class<?>) actualTypeArguments[0];
    Object result = new ArrayList();
    while (resultSet.next()){
        ((ArrayList) result).add(setFieldValue(c,resultSet));
    }
    return result;
}
```

`java.lang.reflect.Method.getGenericReturnType()`方法返回一个`Type`对象，该对象表示此`Method`对象表示的方法的正式返回类型。

对返回的Type对象强转为ParameterizedType，然后获取正真的参数类型

> **getResult**

```java
private Object getResult(Method method, ResultSet resultSet) throws IllegalAccessException, InstantiationException, SQLException {
    Class<?> returnType = method.getReturnType();
    Object temp = new Object();
    while (resultSet.next()){
        temp = setFieldValue(returnType, resultSet);
    }
    return temp;
}
```

这个就比较直接了，直接可以拿到方法的返回类型

以上两个方法，将具体的成员变量赋值放到`setFieldValue`方法中了：

```java
private Object setFieldValue(Class<?> c, ResultSet resultSet) throws IllegalAccessException, InstantiationException, SQLException {
    Object temp = c.newInstance();
    Field[] declaredFields = c.getDeclaredFields();
    for (Field field : declaredFields){
        Object value = resultSet.getObject(field.getName());
        field.setAccessible(true);
        field.set(temp, value);
    }
    return temp;
}
```

根据查询的结果集，给对象中的字段进行赋值。

# 七、测试

## 7.1 创建表

```sql
CREATE TABLE `user`  (
  `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `age` int(11) NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

## 7.2 测试类

```java
package com.example.mybatis;

import com.example.mybatis.mapper.UserMapper;
import com.example.mybatis.pojo.User;
import com.example.mybatis.sql.SqlSession;

import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-03-05 20:16
 * @Feature:
 */
public class Test {

    public static void main(String[] args) {
        UserMapper userMapper = SqlSession.getMapper(UserMapper.class);
        //插入三条数据
        int i = userMapper.insertUser("张三", 12);
        System.out.println(i);

        int j = userMapper.insertUser("李四", 13);
        System.out.println(j);

        int k = userMapper.insertUser("王五", 13);
        System.out.println(k);

        //查询张三
        User user1 = userMapper.selectUser("张三", 12);
        System.out.println(user1);

        //根据年龄查询
        List<User> user2 = userMapper.selectUserList(13);
        for (User user : user2){
            System.out.println(user);
        }
    }
}
```

## 7.3 结果

![](http://mycsdnblog.work/201919061607-t.png)

数据库：

![](http://mycsdnblog.work/201919061607-m.png)

# 八、#{}和${}的区别

#{}防止sql注入