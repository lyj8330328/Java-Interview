# 一、Spring MVC的运行流程

![1550848309783](http://mycsdnblog.work/201919021749-w.png)

⑴ 用户发送请求至前端控制器DispatcherServlet

⑵ DispatcherServlet收到请求调用HandlerMapping处理器映射器。

⑶ 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

⑷ DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

⑸ 执行处理器(Controller，也叫后端控制器)。

⑹ Controller执行完成返回ModelAndView

⑺ HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

⑻ DispatcherServlet将ModelAndView传给ViewReslover视图解析器

 ⑼ ViewReslover解析后返回具体View

 ⑽ DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

 ⑾ DispatcherServlet响应用户。

## 1.1 Spring MVC初始化

```java
protected void initStrategies(ApplicationContext context) {  
        initMultipartResolver(context);//文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析  
        initLocaleResolver(context);//本地化解析  
        initThemeResolver(context);//主题解析  
        initHandlerMappings(context);//通过HandlerMapping，将请求映射到处理器  
        initHandlerAdapters(context);//通过HandlerAdapter支持多种类型的处理器  
        initHandlerExceptionResolvers(context);//如果执行过程中遇到异常，将交给HandlerExceptionResolver来解析  
        initRequestToViewNameTranslator(context);//直接解析请求到视图名  
        initViewResolvers(context);//通过viewResolver解析逻辑视图到具体视图实现  
        initFlashMapManager(context);//flash映射管理器  
    }

```

# 二、Servlet

## 2.1 什么是Servlet

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

使用 Servlet，可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。

Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）实现的程序可以达到异曲同工的效果。但是相比于 CGI，Servlet 有以下几点优势：

- 性能明显更好。
- Servlet 在 Web 服务器的地址空间内执行。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
- Servlet 是独立于平台的，因为它们是用 Java 编写的。
- 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
- Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互。

## 2.2 Servlet的生命周期

 加载—>实例化—>服务—>销毁。

**init（）：**

在Servlet的生命周期中，仅执行一次init()方法。它是在服务器装入Servlet时执行的，负责初始化Servlet对象。可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init（）。因为Servlet是单例模式，所以要注意线程安全问题。

**service（）：**

它是Servlet的核心，负责响应客户的请求。每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。

**destroy（）：**

仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

## 2.3 Servlet实例

```java
package com.example.myspring;

import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @Author: 98050
 * @Time: 2019-02-22 23:18
 * @Feature:
 */
public class Test3 extends HttpServlet {

    @Override
    public void init() throws ServletException {
        super.init();
    }
    
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.service(req, resp);
    }
    
    @Override
    public void destroy() {
        super.destroy();
    }
}
```

在servlet中默认情况下，无论你是get还是post 提交过来都会经过service（）方法来处理，然后转向到doGet或是doPost方法。

# 三、手写SpringMVC

## 3.1 思路分析

**1.web.xml加载**

为了读取web.xml中的配置，编写ServletConfig这个类，它代表当前Servlet在web.xml中的配置信息。通过web.xml中加载自己写的MyDispatcherServlet和读取配置文件。

**2、初始化阶段**

DispatcherServlet的initStrategies方法会初始化9大组件，但是这里将实现一些SpringMVC的最基本的组件而不是全部，按顺序包括：

- 加载配置文件
- 扫描用户配置包下面所有的类
- 拿到扫描到的类，通过反射机制，实例化。并且放到ioc容器中(Map的键值对 beanName-bean) beanName默认是首字母小写
- 初始化HandlerMapping，这里其实就是把url和method对应起来放在一个k-v的Map中,在运行阶段取出

**3、运行阶段**

每一次请求将会调用doGet或doPost方法，所以统一运行阶段都放在doDispatch方法里处理，它会根据url请求去HandlerMapping中匹配到对应的Method，然后利用反射机制调用Controller中的url对应的方法，并得到结果返回。按顺序包括以下功能：

- 异常的拦截
- 获取请求传入的参数并处理参数
- 通过初始化好的handlerMapping中拿出url对应的方法名，反射调用

## 3.2 实现

### 3.2.1 添加依赖

只需一个servlet

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>3.1.0</version>
  <scope>compile</scope>
</dependency>
```

### 3.2.2 定义注解

```java
package com.example.myspringmvc.annotation;

import java.lang.annotation.*;

/**
 * @Author: 98050
 * @Time: 2019-02-23 14:41
 * @Feature:
 */
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface MyController {
    String value() default "";
}
```

```java
package com.example.myspringmvc.annotation;

import java.lang.annotation.*;

/**
 * @Author: 98050
 * @Time: 2019-02-23 14:41
 * @Feature:
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface MyRequestMapping {
    String value() default "";
}
```

### 3.2.3 自定义DispatcherServlet

自定义DispatcherServlet，继承HttpServlet，重写init、doGet、doPost三个方法。

框架：

```java
package com.example.myspringmvc.dispatchservlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @Author: 98050
 * @Feature:
 */
public class MyDispatcherServlet extends HttpServlet {

    @Override
    public void init() throws ServletException {
        super.init();
    }
    
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
    
}
```

### 3.2.4 配置web.xml

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <!-- Spring MVC 核心控制器 DispatcherServlet 配置 -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>com.example.myspringmvc.dispatchservlet.MyDispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <!-- 拦截所有/* 的请求,交给DispatcherServlet处理,性能最好 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

将原来的`org.springframework.web.servlet.DispatcherServlet`替换为自己的DispatcherServlet：

`com.example.myspringmvc.dispatchservlet.MyDispatcherServlet`

### 3.2.5 具体实现

> **init()方法**

配置完成后，前端所有请求就会被MyDispatcherServlet所拦截，所以在重写init方法的时候我们要明确几个任务：

- 获取扫包范围
- 获取包下所有类
- 对类进行筛选，只要类上有`@MyController`注解的类都放入Spring MVC容器当中
- 遍历Spring MVC容器中的类，对每一个类下所属的方法进行筛选，只要方法上有`@MyRequestMapping`注解，那就进行url和方法的映射

注意：扫包范围应该通过解析SpringMVC配置文件获取，这里为了方便直接写死；使用工具类获取包下所有类。

定义三个map，用来充当SpringMVC容器、url与类映射、url与方法映射。

```java
/**
 * 用来存放spring mvc的bean
 */
private ConcurrentHashMap<String,Object> springMVCBeans = new ConcurrentHashMap<>();
/**
 * url与类映射
 */
private ConcurrentHashMap<String,Object> urlBeans = new ConcurrentHashMap<>();

/**
 * url与方法映射
 */
private ConcurrentHashMap<String,String> urlMethod = new ConcurrentHashMap<>();
```

**因为通过Java反射机制执行目标方法时，要获取到目标类的对象，目标方法名，**为了方便查询，定义上述三个map。

重写init方法：

```java
@Override
public void init() throws ServletException {
    System.out.println("初始化");
    try {
        //1.获取扫包范围
        String packages = "com.example.myspringmvc.controller";
        //2.获取包下的所有的类
        List<Class<?>> classList = ClassUtil.getClasses(packages);
        //3.对类进行筛选，将包含MyController注解的类进行初始化，并且放入spring mvc bean容器中
        findAndInitMyControllerClass(classList);
        //4.url与方法映射
        handlerMapping(springMVCBeans);
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    } catch (InstantiationException e) {
        e.printStackTrace();
    }
}
```

```java
private void findAndInitMyControllerClass(List<Class<?>> classList) throws IllegalAccessException, InstantiationException {
    for (Class c : classList){
        MyController annotation = (MyController) c.getAnnotation(MyController.class);
        if (annotation != null){
            Object o = c.newInstance();
            String name = toLowerCaseFirstOne(c.getSimpleName());
            springMVCBeans.put(name, o);
        }
    }
}
```
```java
 private String toLowerCaseFirstOne(String simpleName) {
        String first = (simpleName.charAt(0)+"").toLowerCase();
        String rest = simpleName.substring(1);
        return new StringBuffer(first).append(rest).toString();
    }
```

```java
private void handlerMapping(ConcurrentHashMap<String, Object> springMVCBeans) {
    for (String key : springMVCBeans.keySet()){
        Object o = springMVCBeans.get(key);
        MyController myController = o.getClass().getAnnotation(MyController.class);
        Method[] methods = o.getClass().getMethods();
        String url = "";
        for (Method method : methods){
            MyRequestMapping myRequestMapping = method.getAnnotation(MyRequestMapping.class);
            if (myRequestMapping != null){
                //1.拼接url
                url = myController.value() + myRequestMapping.value();
                //2.url与方法映射
                urlMethod.put(url, method.getName());
            }
        }
        //3.url与类映射
        urlBeans.put(url, o);
    }
}
```

以上都是在init方法中要完成的任务，具体的请求响应放在doPost和doGet中执行。

> **doGet和doPost**

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    doPost(req, resp);
}

@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    try {
        doDispatch(req,resp);
    } catch (NoSuchMethodException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        e.printStackTrace();
    }
}
```

具体执行过程封装在doDispatch中，具体任务如下：

- 获取请求的url
- 根据url获取对应的控制器类
- 根据url获取对应的处理方法
- 使用Java反射机制执行目标方法
- 获取目标方法的返回值，然后进行视图解析

```java
private void doDispatch(HttpServletRequest req, HttpServletResponse resp) throws IOException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, ServletException {
    //1.获取请求url
    String url = req.getRequestURI();
    //2.获取对应的控制器
    Object o = urlBeans.get(url);
    if (o == null){
        resp.getWriter().println("404 not found");
    }
    //3.获取url对应的方法
    String methodName = urlMethod.get(url);
    //4.使用反射机制执行方法
    String result = (String) methodInvoke(o.getClass(),o,methodName);
    //5.视图展示
    viewResolver(result,req,resp);
}
```

```java
private Object methodInvoke(Class<?> aClass, Object o, String methodName) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    Method method = aClass.getMethod(methodName);
    Class<?>[] parameterTypes = method.getParameterTypes();
    Object result = method.invoke(o,parameterTypes);
    return result;
}
```

```java
private void viewResolver(String pageName, HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //1.获取后缀信息
    String suffix = ".jsp";
    //2.页面目录地址
    String prefix = "/";
    req.getRequestDispatcher(prefix + pageName + suffix).forward(req, resp);
}
```

### 3.2.6 测试

**index.jsp**

![1550917918683](http://mycsdnblog.work/201919021748-o.png)

**TestController.java**

```java
package com.example.myspringmvc.controller;

import com.example.myspringmvc.annotation.MyController;
import com.example.myspringmvc.annotation.MyRequestMapping;

/**
 * @Author: 98050
 * @Time: 2019-02-23 15:00
 * @Feature:
 */
@MyController("/test")
public class TestController {

    @MyRequestMapping("/test")
    public String test(){
        return "index";
    }
}
```

**结果：**

![1550917982183](http://mycsdnblog.work/201919021748-M.png)

## 3.3 总结

以上只是完成了一个简易版的Spring MVC，目的是学习Spring MVC的执行流程，许多细节性的问题就不过多纠结了（url地址参数读取、xml配置文件解析等）