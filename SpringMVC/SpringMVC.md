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

# 四、Web环境中的Spring MVC

Spring MVC是建立在IoC容器基础上的。

```xml
<!-- 配置Spring的IOC容器-->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<!--配置Spring MVC的核心控制器-->
<servlet>
  <servlet-name>mvc-dispatcher</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- spring mvc的配置文件 -->
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springMVC.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>mvc-dispatcher</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

重点关注两个类：`ContextLoaderListener`和`DispatcherServlet`

ContextLoaderListener被定义为一个监听器，这个监听器是与Web服务器的生命周期相关联的，由ContextLoaderListener监听器负责完成IoC容器在Web环境中的启动工作

在建立起一个**IoC**体系后，把DispatcherServlet作为Spring MVC处理Web请求的转发器建立起来，从而完成响应HTTP请求的准备。

# 五、上下文在Web容器中的启动

## 5.1 IoC容器启动的基本过程

**ContextLoaderListener启动的上下文为根上下文**。在根上下文的基础上，还有一个与Web MVC相关的上下文用来保存控制器（DispatcherServlet）需要的MVC对象，作为根上下文的子上下文，构成一个层次化的上下文体系。

![](http://mycsdnblog.work/201919201447-R.png)

ContextLoaderListener实现了ServletContextListener接口，这个接口提供了与Servlet生命周期结合的回调，比如contextInitialized方法和contextDestoryed方法。

在Web容器中，建立WebApplicationContext的过程，是在contextInitialized的接口实现中完成的。

具体的载入IoC容器的过程是由ContextLoaderListener交给ContextLoader来完成的。

![](http://mycsdnblog.work/201919201452-1.png)

ContextLoader中完成了两个IoC容器建立的基本过程：

1. 在Web容器中建立起双亲IoC容器
2. 生成相应的WebApplicationContext并将其初始化

## 5.2 Web容器中的上下文设计

Spring为Web应用提供了上下文的扩展接口WebApplicationContext来满足启动过程的需要。

```java
package org.springframework.web.context;

import javax.servlet.ServletContext;

import org.springframework.context.ApplicationContext;
import org.springframework.lang.Nullable;


public interface WebApplicationContext extends ApplicationContext {
   //这里定义的常量用于在ServletContext中存取根上下文
   String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";

   //获取Web容器的ServletContext
   @Nullable
   ServletContext getServletContext();

}
```

以XmlWebApplicationContext的实现为例。

```java
package org.springframework.web.context.support;

import java.io.IOException;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.xml.ResourceEntityResolver;
import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;

public class XmlWebApplicationContext extends AbstractRefreshableWebApplicationContext {

   /** Default config location for the root context. */
   public static final String DEFAULT_CONFIG_LOCATION = "/WEB-INF/applicationContext.xml";

   /** Default prefix for building a config location for a namespace. */
   public static final String DEFAULT_CONFIG_LOCATION_PREFIX = "/WEB-INF/";

   /** Default suffix for building a config location for a namespace. */
   public static final String DEFAULT_CONFIG_LOCATION_SUFFIX = ".xml";


   /**
    * Loads the bean definitions via an XmlBeanDefinitionReader.
    * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
    * @see #initBeanDefinitionReader
    * @see #loadBeanDefinitions
    */
   @Override
   protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
      // Create a new XmlBeanDefinitionReader for the given BeanFactory.
      XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

      // Configure the bean definition reader with this context's
      // resource loading environment.
      beanDefinitionReader.setEnvironment(getEnvironment());
      beanDefinitionReader.setResourceLoader(this);
      beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

      // Allow a subclass to provide custom initialization of the reader,
      // then proceed with actually loading the bean definitions.
      initBeanDefinitionReader(beanDefinitionReader);
      loadBeanDefinitions(beanDefinitionReader);
   }

   /**
    * Initialize the bean definition reader used for loading the bean
    * definitions of this context. Default implementation is empty.
    * <p>Can be overridden in subclasses, e.g. for turning off XML validation
    * or using a different XmlBeanDefinitionParser implementation.
    * @param beanDefinitionReader the bean definition reader used by this context
    * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader#setValidationMode
    * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader#setDocumentReaderClass
    */
   protected void initBeanDefinitionReader(XmlBeanDefinitionReader beanDefinitionReader) {
   }

   protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
      String[] configLocations = getConfigLocations();
      if (configLocations != null) {
         for (String configLocation : configLocations) {
            reader.loadBeanDefinitions(configLocation);
         }
      }
   }

   /**
    * The default location for the root context is "/WEB-INF/applicationContext.xml",
    * and "/WEB-INF/test-servlet.xml" for a context with the namespace "test-servlet"
    * (like for a DispatcherServlet instance with the servlet-name "test").
    */
   @Override
   protected String[] getDefaultConfigLocations() {
      if (getNamespace() != null) {
         return new String[] {DEFAULT_CONFIG_LOCATION_PREFIX + getNamespace() + DEFAULT_CONFIG_LOCATION_SUFFIX};
      }
      else {
         return new String[] {DEFAULT_CONFIG_LOCATION};
      }
   }

}
```

获取Bean定义的信息是从默认的配置文件`applicationContext.xml`中获取的

## 5.3 ContextLoader的设计与实现

ContextLoaderListener通过使用ContextLoader来完成实际的WebApplicationContext，也就是IoC容器的初始化工作。

### 5.3.1 ContextLoaderListener的context初始化

**ContextLoaderListener实现的是ServletContextListener接口。ServletContextListener监听的是ServletContext，那么ServletContext发生变化就会触发相应的事件。**

服务启动时contextInitialized方法被调用，服务关闭时contextDestroyed方法被调用。

```java
 */
@Override
public void contextInitialized(ServletContextEvent event) {
   initWebApplicationContext(event.getServletContext());
}
```

具体初始化工作在父类 ContextLoader中完成

```java
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
   if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
      throw new IllegalStateException(
            "Cannot initialize context because there is already a root application context present - " +
            "check whether you have multiple ContextLoader* definitions in your web.xml!");
   }

   servletContext.log("Initializing Spring root WebApplicationContext");
   Log logger = LogFactory.getLog(ContextLoader.class);
   if (logger.isInfoEnabled()) {
      logger.info("Root WebApplicationContext: initialization started");
   }
   long startTime = System.currentTimeMillis();

   try {
      // Store context in local instance variable, to guarantee that
      // it is available on ServletContext shutdown.
      if (this.context == null) {
          //创建根上下文
         this.context = createWebApplicationContext(servletContext);
      }
      if (this.context instanceof ConfigurableWebApplicationContext) {
         ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
         if (!cwac.isActive()) {
            // The context has not yet been refreshed -> provide services such as
            // setting the parent context, setting the application context id, etc
            if (cwac.getParent() == null) {
               // The context instance was injected without an explicit parent ->
               // determine parent for root web application context, if any.
               // 载入双亲上下文
               ApplicationContext parent = loadParentContext(servletContext);
               cwac.setParent(parent);
            }
             //配置上下文，然后启动容器的初始化
            configureAndRefreshWebApplicationContext(cwac, servletContext);
         }
      }
       // 将根上下文存储到ServletContext中
  servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

      ClassLoader ccl = Thread.currentThread().getContextClassLoader();
      if (ccl == ContextLoader.class.getClassLoader()) {
         currentContext = this.context;
      }
      else if (ccl != null) {
         currentContextPerThread.put(ccl, this.context);
      }

      if (logger.isInfoEnabled()) {
         long elapsedTime = System.currentTimeMillis() - startTime;
         logger.info("Root WebApplicationContext initialized in " + elapsedTime + " ms");
      }

      return this.context;
   }
   catch (RuntimeException | Error ex) {
      logger.error("Context initialization failed", ex);
      servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
      throw ex;
   }
}
```

### 5.3.2 创建根上下文

**createWebApplicationContext**：

```java
protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
   // 判断使用什么样的类在Web容器中作为IoC容器
   Class<?> contextClass = determineContextClass(sc);
   if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
      throw new ApplicationContextException("Custom context class [" + contextClass.getName() +
            "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
   }
   return (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

> **判断使用什么样的类作为Web容器中的IoC容器**

**determineContextClass**

```java
protected Class<?> determineContextClass(ServletContext servletContext) {
   String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
   if (contextClassName != null) {
      try {
         return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
      }
      catch (ClassNotFoundException ex) {
         throw new ApplicationContextException(
               "Failed to load custom context class [" + contextClassName + "]", ex);
      }
   }
   else {
      contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
      try {
         return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
      }
      catch (ClassNotFoundException ex) {
         throw new ApplicationContextException(
               "Failed to load default context class [" + contextClassName + "]", ex);
      }
   }
}
```

- 读取ServletContext中对CONTEXT_CLASS_PARAM参数的配置

- 如果在ServletContext中配置了需要使用的CONTEXT_CLASS，那么就使用这个class，前提是这个class可用

- 如果没有配置的话就使用默认的ContextClass

**defaultStrategies**

```java
private static final String DEFAULT_STRATEGIES_PATH = "ContextLoader.properties";


private static final Properties defaultStrategies;

static {
    // Load default strategy implementations from properties file.
    // This is currently strictly internal and not meant to be customized
    // by application developers.
    try {
        ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, ContextLoader.class);
        defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
    }
    catch (IOException ex) {
        throw new IllegalStateException("Could not load 'ContextLoader.properties': " + ex.getMessage());
    }
}
```

加载默认的ContextLoader实现：

![](http://mycsdnblog.work/201919191510-P.png)

**所以默认的IoC容器是XmlWebApplicationContext**。

> **直接实例化需要产生的IoC容器**

这就是IoC容器在Web容器中的启动过程，在初始化这个上下文后，该上下文会被存储到ServletContext中，这样就建立了一个全局的关于整个应用的上下文。在Spring MVC启动的时候，这个上下文会被设置为DispatcherServlet自带的上下文的双亲上下文。

# 六、Spring MVC的设计与实现

DispatcherServlet会建立自己的上下文来持有Spring MVC的Bean对象，在建立这个自己持有的IoC容器时，会从

ServletContext中得到根上下文作为DispatcherServlet持有上下文的双亲上下文。

DispatcherServlet通过继承FrameworkServlet和HttpServletBean而继承了HttpServlet ，通过使用Servlet API来对HTTP请求进行响应。

![](http://mycsdnblog.work/201919201552-L.png)

接下来重点关注initStrategies和doDispatch。

## 6.1 DispatcherServlet对IoC容器的初始化

DispatcherServlet的启动与Servlet的启动过程是相联系的。在Servlet的初始化过程中，Servlet的init方法会被调用，以进行初始化，具体过程在DispatcherServlet的基类HttpServletBean中实现：

```java
@Override
public final void init() throws ServletException {

   // 获取Servlet的初始化参数，对Bean属性进行配置
   PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
   if (!pvs.isEmpty()) {
      try {
         BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
         ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
         bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
         initBeanWrapper(bw);
         bw.setPropertyValues(pvs, true);
      }
      catch (BeansException ex) {
         if (logger.isErrorEnabled()) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
         }
         throw ex;
      }
   }

   // 调用子类的initServletBean进行具体的初始化 
   initServletBean();
}
```

**initServletBean的初始化过程在FrameworkServlet中完成**

```java
@Override
protected final void initServletBean() throws ServletException {
   getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
   if (logger.isInfoEnabled()) {
      logger.info("Initializing Servlet '" + getServletName() + "'");
   }
   long startTime = System.currentTimeMillis();

   try {
       //初始化上下文
      this.webApplicationContext = initWebApplicationContext();
      initFrameworkServlet();
   }
   catch (ServletException | RuntimeException ex) {
      logger.error("Context initialization failed", ex);
      throw ex;
   }

   if (logger.isDebugEnabled()) {
      String value = this.enableLoggingRequestDetails ?
            "shown which may lead to unsafe logging of potentially sensitive data" :
            "masked to prevent unsafe logging of potentially sensitive data";
      logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
            "': request parameters and headers will be " + value);
   }

   if (logger.isInfoEnabled()) {
      logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
   }
}
```

**初始化上下文**

```java
protected WebApplicationContext initWebApplicationContext() {
   //获取根上下文
   WebApplicationContext rootContext =
         WebApplicationContextUtils.getWebApplicationContext(getServletContext());
   WebApplicationContext wac = null;

   if (this.webApplicationContext != null) {
      // A context instance was injected at construction time -> use it
      wac = this.webApplicationContext;
      if (wac instanceof ConfigurableWebApplicationContext) {
         ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
         if (!cwac.isActive()) {
            // The context has not yet been refreshed -> provide services such as
            // setting the parent context, setting the application context id, etc
            if (cwac.getParent() == null) {
               // The context instance was injected without an explicit parent -> set
               // the root application context (if any; may be null) as the parent
               // 使用这个根上下文作为当前MVC上下文的双亲上下文
               cwac.setParent(rootContext);
            }
            configureAndRefreshWebApplicationContext(cwac);
         }
      }
   }
   if (wac == null) {
      // No context instance was injected at construction time -> see if one
      // has been registered in the servlet context. If one exists, it is assumed
      // that the parent context (if any) has already been set and that the
      // user has performed any initialization such as setting the context id
      wac = findWebApplicationContext();
   }
   if (wac == null) {
      // 创建DispatcherServlet的上下文
      wac = createWebApplicationContext(rootContext);
   }

   if (!this.refreshEventReceived) {
      // Either the context is not a ConfigurableApplicationContext with refresh
      // support or the context injected at construction time had already been
      // refreshed -> trigger initial onRefresh manually here.
      synchronized (this.onRefreshMonitor) {
         onRefresh(wac);
      }
   }
	// 把当前建立的上下文存储到ServletContext中，注意使用的属性名是和当前Servlet名称相关的
   if (this.publishContext) {
      // Publish the context as a servlet context attribute.
      String attrName = getServletContextAttributeName();
      getServletContext().setAttribute(attrName, wac);
   }

   return wac;
}
```

> 1、首先获取根上下文

```java
@Nullable
public static WebApplicationContext getWebApplicationContext(ServletContext sc) {
   return getWebApplicationContext(sc, WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
}
```

> 2、创建DispatcherServlet的上下文

```java
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
   Class<?> contextClass = getContextClass();
   if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
      throw new ApplicationContextException(
            "Fatal initialization error in servlet with name '" + getServletName() +
            "': custom WebApplicationContext class [" + contextClass.getName() +
            "] is not of type ConfigurableWebApplicationContext");
   }
   ConfigurableWebApplicationContext wac =
         (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

   wac.setEnvironment(getEnvironment());
    //配置双亲上下文
   wac.setParent(parent);
   String configLocation = getContextConfigLocation();
   if (configLocation != null) {
      wac.setConfigLocation(configLocation);
   }
   configureAndRefreshWebApplicationContext(wac);

   return wac;
}
```

**2.1  getContextClass()**

``` java
/**
* Default context class for FrameworkServlet.
* @see org.springframework.web.context.support.XmlWebApplicationContext
*/
public static final Class<?> DEFAULT_CONTEXT_CLASS = XmlWebApplicationContext.class;
/** WebApplicationContext implementation class to create. */
private Class<?> contextClass = DEFAULT_CONTEXT_CLASS;
/**
 * Return the custom context class.
 */
public Class<?> getContextClass() {
   return this.contextClass;
}
```

**所以DispatcherServlet 中使用的IoC容器是**`XmlWebApplicationContext`

**2.2 配置双亲上下文**

```java
wac.setParent(parent);
```

**2.3 设置ServletContext的引用和其他相关的配置信息**

```Java
configureAndRefreshWebApplicationContext(wac);
```

``` Java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
   if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
      // The application context id is still set to its original default value
      // -> assign a more useful id based on available information
      if (this.contextId != null) {
         wac.setId(this.contextId);
      }
      else {
         // Generate default id...
         wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
               ObjectUtils.getDisplayString(getServletContext().getContextPath()) + '/' + getServletName());
      }
   }

   wac.setServletContext(getServletContext());
   wac.setServletConfig(getServletConfig());
   wac.setNamespace(getNamespace());
   wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));

   // The wac environment's #initPropertySources will be called in any case when the context
   // is refreshed; do it eagerly here to ensure servlet property sources are in place for
   // use in any post-processing or initialization that occurs below prior to #refresh
   ConfigurableEnvironment env = wac.getEnvironment();
   if (env instanceof ConfigurableWebEnvironment) {
      ((ConfigurableWebEnvironment) env).initPropertySources(getServletContext(), getServletConfig());
   }

   postProcessWebApplicationContext(wac);
   applyInitializers(wac);
    //进行容器初始化
   wac.refresh();
}
```

这样DispatcherServlet中的IoC容器已经建立起来了，这个IoC容器是根上下文的子容器。这样设置，使得对具体的一个Bean定义查找过程来说，如果要查找一个由DispatcherServlet所在的IoC容器来管理的Bean，系统会首先到根上下文去查找，找不到才会去DispatcherServlet所管理的IoC容器去查找，这个机制是在getBean中实现的。

![](http://mycsdnblog.work/201919201718-o.png)

**2.4 把当前建立的上下文存储到ServletContext中，注意使用的属性名是和当前Servlet名称相关的**

```java
if (this.publishContext) {
   // Publish the context as a servlet context attribute.
   String attrName = getServletContextAttributeName();
   getServletContext().setAttribute(attrName, wac);
}
```

其实就是放在一个HashMap当中。

## 6.2 DispatcherServlet对MVC的初始化

在FrameworkServlet中对IoC容器初始化完成后，会在onRefresh方法中调用DispatcherServlet的initStrategies

方法，在这个方法中启动整个Spring MVC框架的初始化。

```java
@Override
protected void onRefresh(ApplicationContext context) {
   initStrategies(context);
}

/**
 * Initialize the strategy objects that this servlet uses.
 * <p>May be overridden in subclasses in order to initialize further strategy objects.
 */
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

具体以HandlerMapping的初始化过程来说明：

```java
	private void initHandlerMappings(ApplicationContext context) {
		this.handlerMappings = null;
		/**
		 * 这里导入所有的HandlerMapping Bean，这些Bean可以在当前的DispatcherServlet的IoC容器中，也可能在其双亲容器中
		 * detectAllHandlerMappings的默认值为true，即从所有IoC容器获取
		 */
		if (this.detectAllHandlerMappings) {
			// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
			Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.handlerMappings = new ArrayList<>(matchingBeans.values());
				// We keep HandlerMappings in sorted order.
				AnnotationAwareOrderComparator.sort(this.handlerMappings);
			}
		}
		else {
			/**
			 * 可以根据名称从IoC容器中通过getBean方法获取HandlerMapping
			 */
			try {
				HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
				this.handlerMappings = Collections.singletonList(hm);
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Ignore, we'll add a default HandlerMapping later.
			}
		}
		/**
		 * 如果没有找到HandlerMapping，那么就需要为Servlet设置默认的HandlerMapping，这些默认值在DispatcherServlet.properties中
		 */
		// Ensure we have at least one HandlerMapping, by registering
		// a default HandlerMapping if no other mappings are found.
		if (this.handlerMappings == null) {
			this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
			if (logger.isTraceEnabled()) {
				logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
						"': using default strategies from DispatcherServlet.properties");
			}
		}
	}
```

经过上述的过程，handlerMappings变量就已经获取了在BeanDefinition中配置好的映射关系。

## 6.3 DispatcherServlet对HTTP请求的分发处理

### 6.3.1 HandlerMapping的配置和设计原理

 每一个HandlerMapping可以持有一系列从URL请求到Controller的映射，Spring MVC提供了一系列的HandlerMapping实现：

![](http://mycsdnblog.work/201919201958-1.png)

```java
package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;

import org.springframework.lang.Nullable;

public interface HandlerMapping {

   String BEST_MATCHING_HANDLER_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingHandler";

   String LOOKUP_PATH = HandlerMapping.class.getName() + ".lookupPath";

   String PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE = HandlerMapping.class.getName() + ".pathWithinHandlerMapping";

   String BEST_MATCHING_PATTERN_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingPattern";

   String INTROSPECT_TYPE_LEVEL_MAPPING = HandlerMapping.class.getName() + ".introspectTypeLevelMapping";

   String URI_TEMPLATE_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".uriTemplateVariables";

   String MATRIX_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".matrixVariables";

   String PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE = HandlerMapping.class.getName() + ".producibleMediaTypes";
    //调用getHandler实际返回的是一个HandlerExecutionChain，这个是典型的Command的模式的使用
   // 这个HandlerExecutionChain不但持有handler本身，还包括了处理这个HTTP请求相关的拦截器
   @Nullable
   HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;

}
```

以SimpleUrlHandlerMapping的实现来进行具体分析。

SimpleUrlHandlerMapping中定义了一个map来持有一系列的映射关系，通过这些映射关系，使Spring MVC应用可以根据HTTP请求确定一个对应的Controller。这些映射关系是通过HandlerMapping来进行封装的，通过getHandler方法可以获得与HTTP对应的HandlerExecutionChain，在HandlerExecutionChain中封装了具体的Controller对象。

> **HandlerExecutionChain**

```java
/*
 * Copyright 2002-2019 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.web.servlet;

import java.util.ArrayList;
import java.util.List;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.lang.Nullable;
import org.springframework.util.CollectionUtils;
import org.springframework.util.ObjectUtils;

/**
 * Handler execution chain, consisting of handler object and any handler interceptors.
 * Returned by HandlerMapping's {@link HandlerMapping#getHandler} method.
 *
 * @author Juergen Hoeller
 * @since 20.06.2003
 * @see HandlerInterceptor
 */
public class HandlerExecutionChain {

   private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);

   private final Object handler;

   @Nullable
   private HandlerInterceptor[] interceptors;

   @Nullable
   private List<HandlerInterceptor> interceptorList;

   private int interceptorIndex = -1;


   /**
    * Create a new HandlerExecutionChain.
    * @param handler the handler object to execute
    */
   public HandlerExecutionChain(Object handler) {
      this(handler, (HandlerInterceptor[]) null);
   }

   /**
    * Create a new HandlerExecutionChain.
    * @param handler the handler object to execute
    * @param interceptors the array of interceptors to apply
    * (in the given order) before the handler itself executes
    */
   public HandlerExecutionChain(Object handler, @Nullable HandlerInterceptor... interceptors) {
      if (handler instanceof HandlerExecutionChain) {
         HandlerExecutionChain originalChain = (HandlerExecutionChain) handler;
         this.handler = originalChain.getHandler();
         this.interceptorList = new ArrayList<>();
         CollectionUtils.mergeArrayIntoCollection(originalChain.getInterceptors(), this.interceptorList);
         CollectionUtils.mergeArrayIntoCollection(interceptors, this.interceptorList);
      }
      else {
         this.handler = handler;
         this.interceptors = interceptors;
      }
   }


   /**
    * Return the handler object to execute.
    */
   public Object getHandler() {
      return this.handler;
   }

   public void addInterceptor(HandlerInterceptor interceptor) {
      initInterceptorList().add(interceptor);
   }

   public void addInterceptor(int index, HandlerInterceptor interceptor) {
      initInterceptorList().add(index, interceptor);
   }

   public void addInterceptors(HandlerInterceptor... interceptors) {
      if (!ObjectUtils.isEmpty(interceptors)) {
         CollectionUtils.mergeArrayIntoCollection(interceptors, initInterceptorList());
      }
   }

   private List<HandlerInterceptor> initInterceptorList() {
      if (this.interceptorList == null) {
         this.interceptorList = new ArrayList<>();
         if (this.interceptors != null) {
            // An interceptor array specified through the constructor
            CollectionUtils.mergeArrayIntoCollection(this.interceptors, this.interceptorList);
         }
      }
      this.interceptors = null;
      return this.interceptorList;
   }

   /**
    * Return the array of interceptors to apply (in the given order).
    * @return the array of HandlerInterceptors instances (may be {@code null})
    */
   @Nullable
   public HandlerInterceptor[] getInterceptors() {
      if (this.interceptors == null && this.interceptorList != null) {
         this.interceptors = this.interceptorList.toArray(new HandlerInterceptor[0]);
      }
      return this.interceptors;
   }


   /**
    * Apply preHandle methods of registered interceptors.
    * @return {@code true} if the execution chain should proceed with the
    * next interceptor or the handler itself. Else, DispatcherServlet assumes
    * that this interceptor has already dealt with the response itself.
    */
   boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
      HandlerInterceptor[] interceptors = getInterceptors();
      if (!ObjectUtils.isEmpty(interceptors)) {
         for (int i = 0; i < interceptors.length; i++) {
            HandlerInterceptor interceptor = interceptors[i];
            if (!interceptor.preHandle(request, response, this.handler)) {
               triggerAfterCompletion(request, response, null);
               return false;
            }
            this.interceptorIndex = i;
         }
      }
      return true;
   }

   /**
    * Apply postHandle methods of registered interceptors.
    */
   void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
         throws Exception {

      HandlerInterceptor[] interceptors = getInterceptors();
      if (!ObjectUtils.isEmpty(interceptors)) {
         for (int i = interceptors.length - 1; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            interceptor.postHandle(request, response, this.handler, mv);
         }
      }
   }

   /**
    * Trigger afterCompletion callbacks on the mapped HandlerInterceptors.
    * Will just invoke afterCompletion for all interceptors whose preHandle invocation
    * has successfully completed and returned true.
    */
   void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex)
         throws Exception {

      HandlerInterceptor[] interceptors = getInterceptors();
      if (!ObjectUtils.isEmpty(interceptors)) {
         for (int i = this.interceptorIndex; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            try {
               interceptor.afterCompletion(request, response, this.handler, ex);
            }
            catch (Throwable ex2) {
               logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
            }
         }
      }
   }

   /**
    * Apply afterConcurrentHandlerStarted callback on mapped AsyncHandlerInterceptors.
    */
   void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
      HandlerInterceptor[] interceptors = getInterceptors();
      if (!ObjectUtils.isEmpty(interceptors)) {
         for (int i = interceptors.length - 1; i >= 0; i--) {
            if (interceptors[i] instanceof AsyncHandlerInterceptor) {
               try {
                  AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptors[i];
                  asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
               }
               catch (Throwable ex) {
                  logger.error("Interceptor [" + interceptors[i] + "] failed in afterConcurrentHandlingStarted", ex);
               }
            }
         }
      }
   }


   /**
    * Delegates to the handler and interceptors' {@code toString()}.
    */
   @Override
   public String toString() {
      Object handler = getHandler();
      StringBuilder sb = new StringBuilder();
      sb.append("HandlerExecutionChain with [").append(handler).append("] and ");
      if (this.interceptorList != null) {
         sb.append(this.interceptorList.size());
      }
      else if (this.interceptors != null) {
         sb.append(this.interceptors.length);
      }
      else {
         sb.append(0);
      }
      return sb.append(" interceptors").toString();
   }

}
```

HandlerExecutionChain中定义的**Handler和Interceptor**需要在定义HandlerMapping时配置好。具体什么时候配置好的，这个需要一个注册过程，这个注册过程在容器对Bean进行依赖注入时发生，它实际上是通过一个Bean的postProcessor来完成的。

> **SimpleUrlHandlerMapping的注册过程**

因为用到了容器的回调，只有SimpleUrlHandlerMapping是ApplicationContextAware的子类才能启动这个注册过程，这个注册过程完成URL和Controller之间的映射关系。

```java
@Override
public void initApplicationContext() throws BeansException {
   super.initApplicationContext();
   registerHandlers(this.urlMap);
}
```

```java
protected void registerHandlers(Map<String, Object> urlMap) throws BeansException {
   if (urlMap.isEmpty()) {
      logger.trace("No patterns in " + formatMappingName());
   }
   else {
      urlMap.forEach((url, handler) -> {
         // Prepend with slash if not already present.
         if (!url.startsWith("/")) {
            url = "/" + url;
         }
         // Remove whitespace from handler bean name.
         if (handler instanceof String) {
            handler = ((String) handler).trim();
         }
         registerHandler(url, handler);
      });
      if (logger.isDebugEnabled()) {
         List<String> patterns = new ArrayList<>();
         if (getRootHandler() != null) {
            patterns.add("/");
         }
         if (getDefaultHandler() != null) {
            patterns.add("/**");
         }
         patterns.addAll(getHandlerMap().keySet());
         logger.debug("Patterns " + patterns + " in " + formatMappingName());
      }
   }
}
```

这个注册过程的完成需要它的基类来配合，这个基类就是AbstractUrlHandlerMapping

```java
protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
   Assert.notNull(urlPath, "URL path must not be null");
   Assert.notNull(handler, "Handler object must not be null");
   Object resolvedHandler = handler;

   // Eagerly resolve handler if referencing singleton via name.
   if (!this.lazyInitHandlers && handler instanceof String) {
      String handlerName = (String) handler;
      ApplicationContext applicationContext = obtainApplicationContext();
      if (applicationContext.isSingleton(handlerName)) {
         resolvedHandler = applicationContext.getBean(handlerName);
      }
   }

   Object mappedHandler = this.handlerMap.get(urlPath);
   if (mappedHandler != null) {
      if (mappedHandler != resolvedHandler) {
         throw new IllegalStateException(
               "Cannot map " + getHandlerDescription(handler) + " to URL path [" + urlPath +
               "]: There is already " + getHandlerDescription(mappedHandler) + " mapped.");
      }
   }
   else {
      if (urlPath.equals("/")) {
         if (logger.isTraceEnabled()) {
            logger.trace("Root mapping to " + getHandlerDescription(handler));
         }
         setRootHandler(resolvedHandler);
      }
      else if (urlPath.equals("/*")) {
         if (logger.isTraceEnabled()) {
            logger.trace("Default mapping to " + getHandlerDescription(handler));
         }
         setDefaultHandler(resolvedHandler);
      }
      else {
         this.handlerMap.put(urlPath, resolvedHandler);
         if (logger.isTraceEnabled()) {
            logger.trace("Mapped [" + urlPath + "] onto " + getHandlerDescription(handler));
         }
      }
   }
}
```

完成这个解析处理过程以后，会把URL和handler作为键值对放到一个handlerMap中去。

```java
private final Map<String, Object> handlerMap = new LinkedHashMap<>();
```

### 6.3.2 使用HandlerMapping完成请求的映射处理

AbstractHandlerMapping中getHandler的实现：

```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
   Object handler = getHandlerInternal(request);
   if (handler == null) {
      handler = getDefaultHandler();
   }
   if (handler == null) {
      return null;
   }
   // Bean name or resolved handler?
   if (handler instanceof String) {
      String handlerName = (String) handler;
      handler = obtainApplicationContext().getBean(handlerName);
   }
	// 把handler封装到HandlerExecutionChain中并加上拦截器
   HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);

   if (logger.isTraceEnabled()) {
      logger.trace("Mapped to " + handler);
   }
   else if (logger.isDebugEnabled() && !request.getDispatcherType().equals(DispatcherType.ASYNC)) {
      logger.debug("Mapped to " + executionChain.getHandler());
   }

   if (hasCorsConfigurationSource(handler)) {
      CorsConfiguration config = (this.corsConfigurationSource != null ? this.corsConfigurationSource.getCorsConfiguration(request) : null);
      CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
      config = (config != null ? config.combine(handlerConfig) : handlerConfig);
      executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
   }

   return executionChain;
}
```

获得handler的具体过程在getHandlerInternal方法中实现，它的实现在AbstractHandlerMapping的子类AbstractUrlHandlerMapping中实现

```java
@Override
@Nullable
protected Object getHandlerInternal(HttpServletRequest request) throws Exception {
   //获取url路径
   String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
   request.setAttribute(LOOKUP_PATH, lookupPath);
   //url和handler进行匹配
   Object handler = lookupHandler(lookupPath, request);
   if (handler == null) {
      // We need to care for the default handler directly, since we need to
      // expose the PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE for it as well.
      Object rawHandler = null;
      if ("/".equals(lookupPath)) {
         rawHandler = getRootHandler();
      }
      if (rawHandler == null) {
         rawHandler = getDefaultHandler();
      }
      if (rawHandler != null) {
         // Bean name or resolved handler?
         if (rawHandler instanceof String) {
            String handlerName = (String) rawHandler;
            rawHandler = obtainApplicationContext().getBean(handlerName);
         }
         validateHandler(rawHandler, request);
         handler = buildPathExposingHandler(rawHandler, lookupPath, lookupPath, null);
      }
   }
   return handler;
}
```

```java
@Nullable
protected Object lookupHandler(String urlPath, HttpServletRequest request) throws Exception {
   // Direct match?
   Object handler = this.handlerMap.get(urlPath);
   if (handler != null) {
      // Bean name or resolved handler?
      if (handler instanceof String) {
         String handlerName = (String) handler;
         handler = obtainApplicationContext().getBean(handlerName);
      }
      validateHandler(handler, request);
      return buildPathExposingHandler(handler, urlPath, urlPath, null);
   }

   // Pattern match?
   List<String> matchingPatterns = new ArrayList<>();
   for (String registeredPattern : this.handlerMap.keySet()) {
      if (getPathMatcher().match(registeredPattern, urlPath)) {
         matchingPatterns.add(registeredPattern);
      }
      else if (useTrailingSlashMatch()) {
         if (!registeredPattern.endsWith("/") && getPathMatcher().match(registeredPattern + "/", urlPath)) {
            matchingPatterns.add(registeredPattern + "/");
         }
      }
   }

   String bestMatch = null;
   Comparator<String> patternComparator = getPathMatcher().getPatternComparator(urlPath);
   if (!matchingPatterns.isEmpty()) {
      matchingPatterns.sort(patternComparator);
      if (logger.isTraceEnabled() && matchingPatterns.size() > 1) {
         logger.trace("Matching patterns " + matchingPatterns);
      }
      bestMatch = matchingPatterns.get(0);
   }
   if (bestMatch != null) {
      handler = this.handlerMap.get(bestMatch);
      if (handler == null) {
         if (bestMatch.endsWith("/")) {
            handler = this.handlerMap.get(bestMatch.substring(0, bestMatch.length() - 1));
         }
         if (handler == null) {
            throw new IllegalStateException(
                  "Could not find handler for best pattern match [" + bestMatch + "]");
         }
      }
      // Bean name or resolved handler?
      if (handler instanceof String) {
         String handlerName = (String) handler;
         handler = obtainApplicationContext().getBean(handlerName);
      }
      validateHandler(handler, request);
      String pathWithinMapping = getPathMatcher().extractPathWithinPattern(bestMatch, urlPath);

      // There might be multiple 'best patterns', let's make sure we have the correct URI template variables
      // for all of them
      Map<String, String> uriTemplateVariables = new LinkedHashMap<>();
      for (String matchingPattern : matchingPatterns) {
         if (patternComparator.compare(bestMatch, matchingPattern) == 0) {
            Map<String, String> vars = getPathMatcher().extractUriTemplateVariables(matchingPattern, urlPath);
            Map<String, String> decodedVars = getUrlPathHelper().decodePathVariables(request, vars);
            uriTemplateVariables.putAll(decodedVars);
         }
      }
      if (logger.isTraceEnabled() && uriTemplateVariables.size() > 0) {
         logger.trace("URI variables " + uriTemplateVariables);
      }
      return buildPathExposingHandler(handler, bestMatch, pathWithinMapping, uriTemplateVariables);
   }

   // No handler found...
   return null;
}
```

### 6.3.3 Spring MVC对HTTP请求的分发处理

DispatcherServlet的doService方法：

```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
   logRequest(request);

   // Keep a snapshot of the request attributes in case of an include,
   // to be able to restore the original attributes after the include.
   Map<String, Object> attributesSnapshot = null;
   if (WebUtils.isIncludeRequest(request)) {
      attributesSnapshot = new HashMap<>();
      Enumeration<?> attrNames = request.getAttributeNames();
      while (attrNames.hasMoreElements()) {
         String attrName = (String) attrNames.nextElement();
         if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
            attributesSnapshot.put(attrName, request.getAttribute(attrName));
         }
      }
   }

   // Make framework objects available to handlers and view objects.
   request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
   request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
   request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
   request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

   if (this.flashMapManager != null) {
      FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
      if (inputFlashMap != null) {
         request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
      }
      request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
      request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
   }

   try {
       // 这个doDispatch是分发请求的入口
      doDispatch(request, response);
   }
   finally {
      if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
         // Restore the original attribute snapshot, in case of an include.
         if (attributesSnapshot != null) {
            restoreAttributesAfterInclude(request, attributesSnapshot);
         }
      }
   }
}
```

这个doDispatch中准备ModelAndView，调用getHandler来响应HTTP请求，然后通过执行Handler的处理来得到返回的ModelAndView结果，最后把这个ModelAndView对象交给相应的视图对象去呈现。

![](http://mycsdnblog.work/201919202113-m.png)

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   HttpServletRequest processedRequest = request;
   HandlerExecutionChain mappedHandler = null;
   boolean multipartRequestParsed = false;

   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

   try {
       //准备一个ModelAndView，用来持有handler处理请求的结果
      ModelAndView mv = null;
      Exception dispatchException = null;

      try {
         processedRequest = checkMultipart(request);
         multipartRequestParsed = (processedRequest != request);

         // 根据请求得到对应的handler，handler的注册以及getHandler的实现.
         mappedHandler = getHandler(processedRequest);
         if (mappedHandler == null) {
            noHandlerFound(processedRequest, response);
            return;
         }

         // 这里是实际调用handler的地方,在handler之前.用HandlerAdapter先检查一下handler的合法性
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

         // Process last-modified header, if supported by the handler.
         String method = request.getMethod();
         boolean isGet = "GET".equals(method);
         if (isGet || "HEAD".equals(method)) {
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
               return;
            }
         }

         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
         }

         // 通过调用HandlerAdapter的handle方法，实际上触发对Contrller的handlerRequest方法的调用.
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

         if (asyncManager.isConcurrentHandlingStarted()) {
            return;
         }
		
         applyDefaultViewName(processedRequest, mv);
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
         dispatchException = ex;
      }
      catch (Throwable err) {
         // As of 4.3, we're processing Errors thrown from handler methods as well,
         // making them available for @ExceptionHandler methods and other scenarios.
         dispatchException = new NestedServletException("Handler dispatch failed", err);
      }
       //这里使用视图对ModelAndView数据的展现
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }
   catch (Exception ex) {
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
   }
   catch (Throwable err) {
      triggerAfterCompletion(processedRequest, response, mappedHandler,
            new NestedServletException("Handler processing failed", err));
   }
   finally {
      if (asyncManager.isConcurrentHandlingStarted()) {
         // Instead of postHandle and afterCompletion
         if (mappedHandler != null) {
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
         }
      }
      else {
         // Clean up any resources used by a multipart request.
         if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
         }
      }
   }
}
```

> **获得handler的过程getHandler**

```java
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
   if (this.handlerMappings != null) {
      for (HandlerMapping mapping : this.handlerMappings) {
         HandlerExecutionChain handler = mapping.getHandler(request);
         if (handler != null) {
            return handler;
         }
      }
   }
   return null;
}
```

> **getHandlerAdapter检查handler的正确性**

通过判断，可以知道这个handler是不是Controller接口的实现

```Java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
   if (this.handlerAdapters != null) {
      for (HandlerAdapter adapter : this.handlerAdapters) {
         if (adapter.supports(handler)) {
            return adapter;
         }
      }
   }
   throw new ServletException("No adapter for handler [" + handler +
         "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

主要通过support方法来实现：

```java
/*
 * Copyright 2002-2012 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.web.servlet.mvc;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.lang.Nullable;
import org.springframework.web.servlet.HandlerAdapter;
import org.springframework.web.servlet.ModelAndView;

/**
 * Adapter to use the plain {@link Controller} workflow interface with
 * the generic {@link org.springframework.web.servlet.DispatcherServlet}.
 * Supports handlers that implement the {@link LastModified} interface.
 *
 * <p>This is an SPI class, not used directly by application code.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @see org.springframework.web.servlet.DispatcherServlet
 * @see Controller
 * @see LastModified
 * @see HttpRequestHandlerAdapter
 */
public class SimpleControllerHandlerAdapter implements HandlerAdapter {

   @Override
   public boolean supports(Object handler) {
      return (handler instanceof Controller);
   }

   @Override
   @Nullable
   public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
         throws Exception {

      return ((Controller) handler).handleRequest(request, response);
   }

   @Override
   public long getLastModified(HttpServletRequest request, Object handler) {
      if (handler instanceof LastModified) {
         return ((LastModified) handler).getLastModified(request);
      }
      return -1L;
   }

}
```

## 6.4 Spring MVC视图的呈现

M：ModelAndView的生成

C：DispatcherServlet、handler实现

V：视图呈现的处理是在render方法调用中完成

DispatcherServlet中的doDispatch方法中的processDispatchResult方法用来完成视图的呈现

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
      @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
      @Nullable Exception exception) throws Exception {

   boolean errorView = false;

   if (exception != null) {
      if (exception instanceof ModelAndViewDefiningException) {
         logger.debug("ModelAndViewDefiningException encountered", exception);
         mv = ((ModelAndViewDefiningException) exception).getModelAndView();
      }
      else {
         Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
         mv = processHandlerException(request, response, handler, exception);
         errorView = (mv != null);
      }
   }

   // Did the handler return a view to render?
   if (mv != null && !mv.wasCleared()) {
      render(mv, request, response);
      if (errorView) {
         WebUtils.clearErrorRequestAttributes(request);
      }
   }
   else {
      if (logger.isTraceEnabled()) {
         logger.trace("No view rendering, null ModelAndView returned.");
      }
   }

   if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
      // Concurrent handling started during a forward
      return;
   }

   if (mappedHandler != null) {
      mappedHandler.triggerAfterCompletion(request, response, null);
   }
}
```

``` Java
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
   // Determine locale for request and apply it to the response.
   Locale locale =
         (this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
   response.setLocale(locale);

   View view;
   String viewName = mv.getViewName();
   if (viewName != null) {
      // 对视图名进行解析.
      view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
      if (view == null) {
         throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
               "' in servlet with name '" + getServletName() + "'");
      }
   }
   else {
      // 从ModelAndView中获取到实际的视图对象.
      view = mv.getView();
      if (view == null) {
         throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
               "View object in servlet with name '" + getServletName() + "'");
      }
   }

   // Delegate to the View object for rendering.
   if (logger.isTraceEnabled()) {
      logger.trace("Rendering view [" + view + "] ");
   }
   try {
      if (mv.getStatus() != null) {
         response.setStatus(mv.getStatus().value());
      }
       //提交视图对象进行展示
      view.render(mv.getModelInternal(), request, response);
   }
   catch (Exception ex) {
      if (logger.isDebugEnabled()) {
         logger.debug("Error rendering view [" + view + "]", ex);
      }
      throw ex;
   }
}
```

View对象

![](http://mycsdnblog.work/201919202200-W.png)

JstlView对象

# 七、