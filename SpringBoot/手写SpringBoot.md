1.纯手写SpringBoot框架

2.疑问：Spring MVC程序入口

没有任何配置文件，那么Spring容器是如何加载的

# 一、什么是SpringBoot

Spring Boot——简单快捷地构建Spring应用！

Spring Boot让我们的Spring应用变的更轻量化。比如：你可以仅仅依靠一个Java类来运行一个Spring引用。你也可以打包你的应用为jar并通过使用java -jar来运行你的Spring Web应用。

Spring Boot的主要优点：

- 为所有Spring开发者更快的入门
- 开箱即用，提供各种默认配置来简化项目配置
- 内嵌式容器简化Web项目
- 没有冗余代码生成和XML配置的要求

本章主要目标完成Spring Boot基础项目的构建，并且实现一个简单的Http请求处理，通过这个例子对Spring Boot有一个初步的了解，并体验其结构简单、开发快速的特性。

SpringBoot 是一个快速开发的框架,能够快速的整合第三方框架，简化XML配置，全部采用注解形式，内置Tomcat容器,帮助开发者能够实现快速开发，SpringBoot的Web组件 默认集成的是SpringMVC框架。

SpringMVC是控制层。

# 二、SpringBoot核心原理

基于SpringMVC无配置文件（纯Java）完全注解化+内置tomcat-embed-core实现SpringBoot框架，Main函数启动。

1、SpringBoot核心快速整合第三方框架原理：Maven继承依赖关系，相当于把需要整合的环境jar包封装好

2、完全无配置文件，注解化开发

SpringBoot采用SpringMVC注解版本实现无配置效果

传统web项目通过web.xml加载整个项目流程

使用Java代码编写Spring MVC配置初始化

如何初始化？没有web.xml那么tomcat如何启动呢?



传统web项目，通过web.xml文件加载整个项目流程

3、SpringBoot内嵌入tomcat-embed-core

Java语言创建tomcat容器，加载class文件

# 三、内置Tomcat容器

Java提供内置Tomcat容器框架，使用Java语言操作Tomcat容器。

案例： 使用Java语言创建一个Tomcat容器

## 3.1 maven依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myspringboot</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <!--Java语言操作tomcat -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>8.5.16</version>
        </dependency>
    </dependencies>
</project>
```

## 3.2 创建Servlet

```java
package com.example.embedtomcat;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @Author: 98050
 * @Time: 2019-05-02 16:58
 * @Feature:
 */
public class IndexServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().println("spring boot 2.0");
    }


}
```

## 3.3 创建Tomcat

```java
package com.example.embedtomcat;

import org.apache.catalina.LifecycleException;
import org.apache.catalina.core.StandardContext;
import org.apache.catalina.startup.Tomcat;

/**
 * @Author: 98050
 * @Time: 2019-05-02 17:04
 * @Feature:
 */
public class Test {

    //端口号
    private static int PORT = 8080;

    //项目名称
    private static String CONTEXT_PATH = "/TEST";

    //Servlet名称
    private static String SERVLET_Name = "IndexServlet";

    public static void main(String[] args) throws LifecycleException {
        //1.创建Tomcat容器
        Tomcat tomcatServer = new Tomcat();
        //2.设置Tomcat端口号
        tomcatServer.setPort(PORT);
        //3.是否设置自动部署
        tomcatServer.getHost().setAutoDeploy(false);
        //4.创建context上下文
        StandardContext standardContext = new StandardContext();
        //5.设置路径
        standardContext.setPath(CONTEXT_PATH);
        //6.设置监听上下文
        standardContext.addLifecycleListener(new Tomcat.FixContextListener());
        //7.tomcat容器添加standardContext
        tomcatServer.getHost().addChild(standardContext);
        //8.创建servlet
        tomcatServer.addServlet(CONTEXT_PATH, SERVLET_Name, new IndexServlet());
        //9.添加servlet url映射
        standardContext.addServletMappingDecoded("/index", SERVLET_Name);
        //10.启动tomcat
        tomcatServer.start();
        //11.打印日志
        System.out.println("tomcat启动");
        //12.异步接收访问请求
        tomcatServer.getServer().await();
    }
}
```

启动：

![](http://mycsdnblog.work/201919021735-R.png)

## 3.4 测试

访问地址：http://localhost:8080/TEST/index

![](http://mycsdnblog.work/201919021736-7.png)

## 3.5 思考

Spring Boot的原理就是将`IndexServlet`替换成`DispatcherServlet`即可

![1556790038353](http://mycsdnblog.work/201919021741-H.png)

# 四、Spring MVC无配置启动

## 4.1 SpringMVC原理

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

## 4.2 SpringMVC注解启动方式

DispatcherServlet是Spring MVC的核心，每当应用接受一个HTTP请求，由DispatcherServlet负责将请求分发给应用的其他组件。

在旧版本中，DispatcherServlet之类的servlet一般在web.xml文件中配置，该文件一般会打包进最后的war包中；但是Spring 3引入了注解，基于注解配置Spring MVC。

## 4.3 Maven依赖

新增依赖`spring-web`、`spring-webmvc`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myspringboot</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <!--Java语言操作tomcat -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>8.5.16</version>
        </dependency>
        <!-- tomcat对jsp支持 -->
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-jasper</artifactId>
            <version>8.5.16</version>
        </dependency>

        <!-- spring-web -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.0.4.RELEASE</version>
            <scope>compile</scope>
        </dependency>
        <!-- spring-mvc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.0.4.RELEASE</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

## 4.4 加载DispatcherServlet配置文件

***AbstractAnnotationConfigDispatcherServletInitializer*这个类负责配置*DispatcherServlet*、初始化Spring MVC容器和Spring容器。**

- getRootConfigClasses()方法用于获取Spring应用容器的配置文件，这里我们给定预先定义的RootConfig.class；
- getServletConfigClasses负责获取Spring MVC应用容器，这里传入预先定义好的WebConfig.class
- getServletMappings()方法负责指定需要由DispatcherServlet映射的路径，这里给定的是"/"，意思是由DispatcherServlet处理所有向该应用发起的请求。

## 4.5 加载SpringMVC容器

正如可以通过多种方式配置DispatcherServlet一样，也可以通过多种方式启动Spring MVC特性。

原来我们一般在xml文件中使用`<mvc:annotation-driven>`元素启动注解驱动的Spring MVC特性。

```java
package com.example.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * @Author: 98050
 * @Time: 2019-05-02 18:15
 * @Feature: 加载SpringMVC容器
 * @EnableWebMvc: 开启Spring MVC功能：扫包、注解、视图解析器
 * @Configuration: 表示一个配置文件
 */
@Configuration
@EnableWebMvc
@ComponentScan("com.example.controller")
public class WebConfig implements WebMvcConfigurer {
}
```

## 4.6 加载Spring容器

```java
package com.example.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: 98050
 * @Time: 2019-05-02 18:15
 * @Feature: 加载Spring容器
 */
@Configuration
@ComponentScan("com.example")
public class RootConfig {
}
```

## 4.7 编写启动器

```java
package com.example.start;

import com.example.embedtomcat.IndexServlet;
import org.apache.catalina.LifecycleException;
import org.apache.catalina.WebResourceRoot;
import org.apache.catalina.core.StandardContext;
import org.apache.catalina.startup.Tomcat;
import org.apache.catalina.webresources.DirResourceSet;
import org.apache.catalina.webresources.StandardRoot;

import javax.servlet.ServletException;
import java.io.File;

/**
 * @Author: 98050
 * @Time: 2019-05-02 18:45
 * @Feature:
 */
public class AppTomcat {

    //端口号
    private static int PORT = 9090;

    public static void start() throws LifecycleException, ServletException {
        //1.创建Tomcat容器
        Tomcat tomcatServer = new Tomcat();
        //2.设置Tomcat端口号
        tomcatServer.setPort(PORT);
        //3.读取项目路径,加载静态资源
        StandardContext ctx = (StandardContext) tomcatServer.addWebapp("/", new File("src/main").getAbsolutePath());
        //4.禁止重新载入
        ctx.setReloadable(false);
        //5.class文件读取地址
        File additionWebInfClasses = new File("target/classes");
        //6.创建WebRoot
        WebResourceRoot resourceRoot = new StandardRoot(ctx);
        //7.tomcat内部读取class执行
        resourceRoot.addPreResources(new DirResourceSet(resourceRoot,"/WEB-INF/classes",additionWebInfClasses.getAbsolutePath(),"/"));
        //8.启动
        tomcatServer.start();
        //9.异步接收访问请求
        tomcatServer.getServer().await();
    }
}
```

## 4.8 编写Controller

```java
package com.example.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: 98050
 * @Time: 2019-05-02 18:42
 * @Feature:
 */
@RestController
public class IndexController {

    @GetMapping("/index")
    public String index(){
        return "spring boot";
    }
}
```

## 4.9 调用启动器

```java
package com.example;

import com.example.start.AppTomcat;
import org.apache.catalina.LifecycleException;

import javax.servlet.ServletException;

/**
 * @Author: 98050
 * @Time: 2019-05-02 19:03
 * @Feature:
 */
public class MyTestApplication {

    public static void main(String[] args) throws ServletException, LifecycleException {
        AppTomcat.start();
    }
}
```

## 4.10 测试

访问链接：http://localhost:9090/index

![](http://mycsdnblog.work/201919021905-B.png)

## 4.11 增加视图解析

### 4.11.1 配置视图解析器

```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

/**
 * @Author: 98050
 * @Time: 2019-05-02 18:15
 * @Feature: 加载SpringMVC容器
 * @EnableWebMvc: 开启Spring MVC功能：扫包、注解、视图解析器
 * @Configuration: 表示一个配置文件
 */
@Configuration
@EnableWebMvc
@ComponentScan("com.example.controller")
public class WebConfig implements WebMvcConfigurer {

    /**
     * 配置JSP视图解析器
     * @return
     */
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/view/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }
}
```

### 4.11.2 Controller

```java
package com.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @Author: 98050
 * @Time: 2019-05-02 19:18
 * @Feature:
 */
@Controller
public class ViewController {

    @RequestMapping("/page")
    public String test(){
        return "index";
    }
}
```

### 4.11.3 创建目录和jsp

![1556796082989](http://mycsdnblog.work/201919021921-G.png)

```jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>


<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>

    <title>TEST</title>

    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    <meta http-equiv="description" content="This is my page">
    <!--
    <link rel="stylesheet" type="text/css" href="styles.css">
    -->

</head>
<h1>
    Spring Boot Test!
</h1>
<body>
</body>
</html>
```

### 4.11.4 测试

访问链接：http://localhost:9090/page

![](http://mycsdnblog.work/201919021922-R.png)

