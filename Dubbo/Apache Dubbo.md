# 1、Apache Dubbo 概述

## 1.1 简介

Apache Dubbo是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：**面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现**。

## 1.2 架构

![image-20200303155252388](http://mycsdnblog.work/202020221616-f.png)

Dubbo 的RPC 调用流程，这里主要涉及到4个模块：

- **Registry**：服务注册，我们一般会采取Zookeeper 作为我们的注册中心
- **Provider**：服务提供者（生产者），提供具体的服务实现
- **Consumer**：消费者，从注册中心中订阅服务
- **Monitor**：监控中心，RPC调用次数和调用时间监控

# 2、服务注册中心

​		ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。
　　ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。
　　ZooKeeper包含一个简单的原语集，提供Java和C的接口。
　　ZooKeeper代码版本中，提供了分布式独享锁、选举、队列的接口，代码在zookeeper-3.4.8\src\recipes。其中分布锁和队列有Java和C两个版本，选举只有Java版本。

# 3、Dubbo快速入门

## 3.1 项目构建

因为dubbo是面向接口的远程方法调用，所以抽取一个独立的模块用来存放接口，然后再创建一个服务提供方和服务消费方，总体目录结构如下：

![image-20200304180245322](http://mycsdnblog.work/202020221617-d.png)

父工程的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.5.RELEASE</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>dubbo-test</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>dubbo-provider</module>
        <module>dubbo-consumer</module>
        <module>dubbo-api</module>
    </modules>

    <dependencyManagement>
        <dependencies>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>2.2.5.RELEASE</version>
            </dependency>

            <!-- Dubbo dependencies -->
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-spring-boot-starter</artifactId>
                <version>2.7.3</version>
            </dependency>

            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-framework</artifactId>
                <version>2.8.0</version>
            </dependency>
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-recipes</artifactId>
                <version>2.8.0</version>
            </dependency>


            <!-- Zookeeper dependencies -->
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-dependencies-zookeeper</artifactId>
                <version>${dubbo.version}</version>
                <type>pom</type>
                <exclusions>
                    <exclusion>
                        <groupId>org.slf4j</groupId>
                        <artifactId>slf4j-log4j12</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

## 3.2 公共接口

```java
package com.dubbo.api;

public interface UserService {

    String add();

    String get();

    String delete();

    String update();
}
```

```java
package com.dubbo.api;

public interface ConsumerService {

    String add();

    String get();

    String delete();

    String update();
}
```

## 3.3 服务提供方

### 3.3.1 依赖

```xml
<dependencies>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>


        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
        </dependency>


        <!-- Zookeeper dependencies -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>
```

### 3.3.2 配置文件

```yaml
server:
  port: 8088
  
spring:
  application:
    name: dubbo-provider

dubbo:
  application:
    name: ${spring.application.name}
  registry:
    address: zookeeper://127.0.0.1:2181
  protocol:
    name: dubbo
    port: 20880
  scan:
    base-packages: com.dubbo.api
```

### 3.3.3 启动类

注意使用`@EnableDubbo`注解读取Dubbo配置

```java
package com.example.provider;

import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
@EnableDubbo
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class,args);
    }
}
```

### 3.3.4 Service

这里面使用的是Dubbo下的`@Service`注解，将此接口实现暴露出去，即注册到zookeeper中。

```java
package com.example.provider.service;

import com.dubbo.api.UserService;
import org.apache.dubbo.config.annotation.Service;

@Service(version = "1.0.0",interfaceClass = UserService.class)
public class UserServiceImpl implements UserService {
    @Override
    public String add() {
        return "用户新增成功！";
    }

    @Override
    public String get() {
        return "用户获取成功！";
    }

    @Override
    public String delete() {
        return "用户删除成功！";
    }

    @Override
    public String update() {
        return "用户修改成功！";
    }
}
```

### 3.3.5 Controller

```java
package com.example.provider.controller;

import com.dubbo.api.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/provider")
public class ProviderController {

    private final UserService userService;

    @Autowired
    public ProviderController(UserService userService) {
        this.userService = userService;
    }


    @GetMapping("/")
    public String get(){
        return userService.get();
    }

    @PostMapping("/")
    public String add(){
        return userService.add();
    }

    @PutMapping("/")
    public String update(){
        return userService.update();
    }

    @DeleteMapping("/")
    public String delete(){
        return userService.delete();
    }
}
```

## 3.4 服务消费方

### 3.4.1 依赖

```xml
    <dependencies>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>


        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
        </dependency>


        <!-- Zookeeper dependencies -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>
```

### 3.4.2 配置文件

注意这里面的dubbo.protocol.port要与服务提供方的区别开，避免端口占用错误。

```yaml
server:
  port: 8082

spring:
  application:
    name: dubbo-consumer

dubbo:
  application:
    name: ${spring.application.name}
  registry:
    address: zookeeper://127.0.0.1:2181
  protocol:
    name: dubbo
    port: 20881
  scan:
    base-packages: com.dubbo.api

```

### 3.4.3 启动类

```java
package com.example.consumer;



import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableDubbo
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class,args);
    }
}
```

### 3.4.4 Service

这里使用 `@Reference`注解来创建远程服务代理，用来调用具体的远程服务

```java
package com.example.consumer.service;

import com.dubbo.api.UserService;
import com.dubbo.api.ConsumerService;
import org.apache.dubbo.config.annotation.Reference;
import org.apache.dubbo.config.annotation.Service;

@Service
public class ConsumerServiceImpl implements ConsumerService {


    @Reference(version = "1.0.0",check = false)
    private UserService userService;

    @Override
    public String add() {
        return this.userService.add();
    }

    @Override
    public String get() {
        return this.userService.get();
    }

    @Override
    public String delete() {
        return "3123213";
    }

    @Override
    public String update() {
        return "123213";
    }
}
```

### 3.4.5 Controller

```java
package com.example.consumer.controller;

import com.dubbo.api.ConsumerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/consumer")
public class ConsumerController {

    private final ConsumerService consumerService;

    @Autowired
    public ConsumerController(ConsumerService consumerService) {
        this.consumerService = consumerService;
    }

    @GetMapping("/")
    public String get(){
        return this.consumerService.get();
    }

    @PostMapping("/")
    public String add(){
        return consumerService.add();
    }

    @PutMapping("/")
    public String update(){
        return consumerService.update();
    }

    @DeleteMapping("/")
    public String delete(){
        return consumerService.delete();
    }
}
```

## 3.5 测试

根据官方教程安装dubbo admin，首先启动zookpeer，然后启动dubbo admin，服务提供方，最后提供服务消费方。

![image-20200304183355210](http://mycsdnblog.work/202020221618-f.png)

点击dubbo-provider的详情：

![image-20200304183611606](http://mycsdnblog.work/202020221619-C.png)

消费已经绑定。使用Postman进行测试：

1、服务提供方暴露的用户查询接口

![image-20200304183713759](http://mycsdnblog.work/202020221619-q.png)

2、服务消费方调用

![image-20200304183802832](http://mycsdnblog.work/202020221620-a.png)

使用dubbo admin查看接口的元数据信息（2.7.3版本的dubbo）：

![image-20200304183847172](http://mycsdnblog.work/202020221620-c.png)

## 3.6 group的使用

当同一个接口有多个实现，那么可以通过指定group，在调用的时候进行区分。

再创建一个服务提供方：

![image-20200304184151328](http://mycsdnblog.work/202020221621-C.png)

实现UserService接口，然后在暴露接口的时候指定group：

![image-20200304184500032](http://mycsdnblog.work/202020221621-4.png)

再修改原来的服务提供方：

![image-20200304184305279](http://mycsdnblog.work/202020221621-A.png)

在服务消费方进行具体指定：

![image-20200304184524858](http://mycsdnblog.work/202020221621-x.png)

重新启动

![image-20200304184809483](http://mycsdnblog.work/202020221622-h.png)

通过dubbo admin进行查看：

![image-20200304184827925](http://mycsdnblog.work/202020221622-E.png)

点击provider2的详情：

![image-20200304184906935](http://mycsdnblog.work/202020221623-j.png)

通过Postman测试：

![image-20200304184951879](http://mycsdnblog.work/202020221624-n.png)

# 4、Dubbo相关配置说明

## 4.1 包扫描

`dubbo.scan.base-packages`配置要扫描的包，一般在服务提供方进行配置

## 4.2 协议

`dubbo.protocol.name` 配置协议

Dubbo支持的协议有：dubbo、rmi、hessian、http、webservice、rest、redis

dubbo协议采用单一长连接和NIO异步通讯，适合小数据量大并发的服务调用，以及服务消费者机器远大于服务提供者机器数的情况。

## 4.3 启动时检查

```java
@Reference(version = "1.0.0",check = false)
```

需要在服务消费方进行配置，如果不配置默认check值为true。Dubbo缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止Spring初始化完成，以便上线时能及早发现问题。

开发阶段check设置为false，生产环境下改为true。

## 4.4 负载均衡

负载均衡(Load Balance)：其实是将请求分摊到多个操作单元上进行执行，从而完成共同的工作任务。

在集群负载均衡时，Dubbo提供了多种策略（随机，轮询、最少活跃调用数、一致性Hash），缺省为random随机调用。

配置负载均衡策略，即 可以在服务提供者一方配置，也可以在服务消费者一方配置

将dubbo-provider、dubbo-provider-2和dubbo-consumer中指定group删除，然后重新启动：

![image-20200304215330758](http://mycsdnblog.work/202020221624-C.png)

点击详情：

![image-20200304215353141](http://mycsdnblog.work/202020221625-v.png)

因为此时选择的负载均衡策略是random，可以通过Postman进行测试。

# 5、Dubbo无法发布带事物注解的Service问题

**在使用dubbo 2.5.3之前的版本@Service来发布服务时，当该服务中有@Transactional,是无法正常发布的**

原因是事物控制的底层原理是基于动态代理来实现的，默认情况下Spring是基于JDK动态代理的方式来创建代理对象，而代理对象最终类名为com.sun.proxy.$Proxy11(最后两位数字不固定)，导致dubbo在发布服务进行包匹配时无法完成匹配，导致服务无法成功发布。

解决方法就是更换Spring默认的代理实现方式为CGLIB，并且在@Service中通过interfaceClass指定具体的实现接口。

**在之后的版本中不存在这个问题！**

# 6、源码导读

## 6.1 Dubbo SPI

### 6.1.1 简介

PI 全称为 Service Provider Interface，是一种服务发现机制。**SPI 的本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类**。这样可以在运行时，动态为接口替换实现类。正因此特性，我们可以很容易的通过 SPI 机制为我们的程序提供拓展功能。SPI 机制在第三方框架中也有所应用，比如 Dubbo 就是通过 SPI 机制加载所有的组件。不过，Dubbo 并未使用 Java 原生的 SPI 机制，而是对其进行了增强，使其能够更好的满足需求。在 Dubbo 中，SPI 是一个非常重要的模块。基于 SPI，我们可以很容易的对 Dubbo 进行拓展。

### 6.1.2 SPI示例

#### 6.1.2.1 Java SPI示例

定义一个接口：

```java
package com.dubbo.api;

public interface Robot {

    void sayHello();
}
```

添加两个实现类：

```java
package com.dubbo.api.impl;

import com.dubbo.api.Robot;

public class Bumblebee implements Robot {

    @Override
    public void sayHello() {
        System.out.println("Hello, I am Bumblebee.");
    }
}
```

```java
package com.dubbo.api.impl;

import com.dubbo.api.Robot;

public class OptimusPrime implements Robot {
    
    @Override
    public void sayHello() {
        System.out.println("Hello, I am Optimus Prime.");
    }
}
```

接下来在resources目录下新建META-INF/services 文件夹，然后创建一个文件，名称为 Robot 的全限定名 com.dubbo.api.Robot。文件内容为实现类的全限定的类名，如下：

```java
com.dubbo.api.impl.Bumblebee
com.dubbo.api.impl.OptimusPrime
```

![image-20200305181802199](http://mycsdnblog.work/202020221626-S.png)

编写代码进行测试：

```java
package test;

import com.dubbo.api.Robot;

import java.util.ServiceLoader;

public class Main {

    public static void main(String[] args) {
        ServiceLoader<Robot> serviceLoader = ServiceLoader.load(Robot.class);
        System.out.println("Java SPI");
        serviceLoader.forEach( Robot::sayHello);
    }
}
```

运行结果：

![image-20200305181843825](F:\Apache Dubbo.assets\image-20200305181843825.png)

#### 6.1.2.2 Dubbo SPI示例

Dubbo 并未使用 Java SPI，而是重新实现了一套功能更强的 SPI 机制。Dubbo SPI 的相关逻辑被封装在了 ExtensionLoader 类中，通过 ExtensionLoader，我们可以加载指定的实现类。Dubbo SPI 所需的配置文件需放置在 META-INF/dubbo 路径下，配置内容如下。

```java
bumblebee = com.dubbo.api.impl.Bumblebee
optimusPrime = com.dubbo.api.impl.OptimusPrime
```

![image-20200305183148768](http://mycsdnblog.work/202020221627-B.png)

与 Java SPI 实现类配置不同，Dubbo SPI 是通过键值对的方式进行配置，这样我们可以按需加载指定的实现类。另外，在测试 Dubbo SPI 时，需要在 Robot 接口上标注 @SPI 注解。下面来演示 Dubbo SPI 的用法：

```java
package test;

import com.dubbo.api.Robot;
import org.apache.dubbo.common.extension.ExtensionLoader;

import java.util.ServiceLoader;

public class Main {

    public static void main(String[] args) {
        ServiceLoader<Robot> serviceLoader = ServiceLoader.load(Robot.class);
        System.out.println("Java SPI");
        serviceLoader.forEach(Robot::sayHello);

        System.out.println("Dubbo SPI");
        ExtensionLoader<Robot> extensionLoader = ExtensionLoader.getExtensionLoader(Robot.class);
        Robot optimusPrime = extensionLoader.getExtension("optimusPrime");
        optimusPrime.sayHello();
        Robot bumblebee = extensionLoader.getExtension("bumblebee");
        bumblebee.sayHello();
    }
}
```

测试结果：

![image-20200305183102285](http://mycsdnblog.work/202020221628-x.png)

Dubbo SPI 除了支持按需加载接口实现类，还增加了 IOC 和 AOP 等特性

### 6.1.3 Dubbo SPI源码分析

前面简单演示了 Dubbo SPI 的使用方法。首先通过 ExtensionLoader 的 getExtensionLoader 方法获取一个 ExtensionLoader 实例，然后再通过 ExtensionLoader 的 getExtension 方法获取拓展类对象。

这其中，getExtensionLoader 方法用于从缓存中获取与拓展类对应的 ExtensionLoader，若缓存未命中，则创建一个新的实例。

```java
public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
    if (type == null) {
        throw new IllegalArgumentException("Extension type == null");
    } else if (!type.isInterface()) {
        throw new IllegalArgumentException("Extension type (" + type + ") is not an interface!");
    } else if (!withExtensionAnnotation(type)) {
        throw new IllegalArgumentException("Extension type (" + type + ") is not an extension, because it is NOT annotated with @" + SPI.class.getSimpleName() + "!");
    } else {
        /**
        * 从缓存中获取指定类型的ExtensionLoader，如果不存在就新建一个放入缓存
        * EXTENSION_LOADERS是一个ConcurrentHashMap
        */
        ExtensionLoader<T> loader = (ExtensionLoader)EXTENSION_LOADERS.get(type);
        if (loader == null) {
            EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader(type));
            loader = (ExtensionLoader)EXTENSION_LOADERS.get(type);
        }

        return loader;
    }
}
```

接下来从 ExtensionLoader 的 getExtension 方法作为入口，对拓展类对象的获取过程进行详细的分析。

```java
public T getExtension(String name) {
    if (StringUtils.isEmpty(name)) {
        throw new IllegalArgumentException("Extension name == null");
    } else if ("true".equals(name)) {
        //返回默认的ExtensionLoader实现类
        return this.getDefaultExtension();
    } else {
        // Holder，顾名思义，用于持有目标对象，从缓存中获取Holder
        Holder<Object> holder = this.getOrCreateHolder(name);
        Object instance = holder.get();
        //双重校验
        if (instance == null) {
            synchronized(holder) {
                instance = holder.get();
                if (instance == null) {
                    //如果无法获取ExtensionLoader实例对象，那么就需要创建一个
                    instance = this.createExtension(name);
                    //将新创建的实例对象放入Holder中
                    holder.set(instance);
                }
            }
        }
        return instance;
    }
}
```

上面代码的逻辑比较简单，首先检查缓存，缓存未命中则创建拓展对象。对于获取Holder的方法，如下所示：

```java
private Holder<Object> getOrCreateHolder(String name) {
    Holder<Object> holder = (Holder)this.cachedInstances.get(name);
    if (holder == null) {
        this.cachedInstances.putIfAbsent(name, new Holder());
        holder = (Holder)this.cachedInstances.get(name);
    }

    return holder;
}
```

从缓存中读取，如果不存在则新建一个Holder对象放入缓存中。接下来分析一下创建拓展对象的过程是怎样的。

```java
private T createExtension(String name) {
    // 从配置文件中加载所有的拓展类，可得到“配置项名称”到“配置类”的映射关系表
    Class<?> clazz = (Class)this.getExtensionClasses().get(name);
    if (clazz == null) {
        throw this.findException(name);
    } else {
        try {
            T instance = EXTENSION_INSTANCES.get(clazz);
            if (instance == null) {
                // 通过反射创建实例
                EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance());
                instance = EXTENSION_INSTANCES.get(clazz);
            }
			// 向实例中注入依赖
            this.injectExtension(instance);
            Set<Class<?>> wrapperClasses = this.cachedWrapperClasses;
            Class wrapperClass;
            if (CollectionUtils.isNotEmpty(wrapperClasses)) {
                // 循环创建 Wrapper 实例
                // 将当前 instance 作为参数传给 Wrapper 的构造方法，并通过反射创建 Wrapper 实例。
                // 然后向 Wrapper 实例中注入依赖，最后将 Wrapper 实例再次赋值给 instance 变量
                for(Iterator var5 = wrapperClasses.iterator(); var5.hasNext(); instance = this.injectExtension(wrapperClass.getConstructor(this.type).newInstance(instance))) {
                    wrapperClass = (Class)var5.next();
                }
            }

            return instance;
        } catch (Throwable var7) {
            throw new IllegalStateException("Extension instance (name: " + name + ", class: " + this.type + ") couldn't be instantiated: " + var7.getMessage(), var7);
        }
    }
}
```

createExtension 方法的逻辑稍复杂一下，包含了如下的步骤：

1. 通过 getExtensionClasses 获取所有的拓展类
2. 通过反射创建拓展对象
3. 向拓展对象中注入依赖
4. 将拓展对象包裹在相应的 Wrapper 对象中

以上步骤中，第一个步骤是加载拓展类的关键，第三和第四个步骤是 Dubbo IOC 与 AOP 的具体实现。接下来会重点分析 getExtensionClasses 方法的逻辑，以及简单介绍 Dubbo IOC 的具体实现。

#### 6.1.3.1 获取所有的拓展类

在通过名称获取拓展类之前，首先需要根据配置文件解析出拓展项名称到拓展类的映射关系表（Map<名称, 拓展类>），之后再根据拓展项名称从映射关系表中取出相应的拓展类即可。相关过程的代码分析如下：

```java
private Map<String, Class<?>> getExtensionClasses() {
    // 从缓存中获取已加载的拓展类
    Map<String, Class<?>> classes = (Map)this.cachedClasses.get();
    // 双重检查
    if (classes == null) {
        Holder var2 = this.cachedClasses;
        synchronized(this.cachedClasses) {
            classes = (Map)this.cachedClasses.get();
            if (classes == null) {
                // 加载拓展类
                classes = this.loadExtensionClasses();
                this.cachedClasses.set(classes);
            }
        }
    }

    return classes;
}
```

这里也是先检查缓存，若缓存未命中，则通过 synchronized 加锁。加锁后再次检查缓存，并判空。此时如果 classes 仍为 null，则通过 loadExtensionClasses 加载拓展类。下面分析 loadExtensionClasses 方法的逻辑。

```java
private Map<String, Class<?>> loadExtensionClasses() {
    //对@SPI注解进行解析
    this.cacheDefaultExtensionName();
    //加载配置文件
    Map<String, Class<?>> extensionClasses = new HashMap();
    this.loadDirectory(extensionClasses, "META-INF/dubbo/internal/", this.type.getName());
    this.loadDirectory(extensionClasses, "META-INF/dubbo/internal/", this.type.getName().replace("org.apache", "com.alibaba"));
    this.loadDirectory(extensionClasses, "META-INF/dubbo/", this.type.getName());
    this.loadDirectory(extensionClasses, "META-INF/dubbo/", this.type.getName().replace("org.apache", "com.alibaba"));
    this.loadDirectory(extensionClasses, "META-INF/services/", this.type.getName());
    this.loadDirectory(extensionClasses, "META-INF/services/", this.type.getName().replace("org.apache", "com.alibaba"));
    return extensionClasses;
}
```

loadExtensionClasses 方法总共做了两件事情，一是对 SPI 注解进行解析，二是调用 loadDirectory 方法加载指定文件夹配置文件。对SPI注解的解析是在`cacheDefaultExtensionName`方法中完成的：

```java
private void cacheDefaultExtensionName() {
    SPI defaultAnnotation = (SPI)this.type.getAnnotation(SPI.class);
    if (defaultAnnotation != null) {
        String value = defaultAnnotation.value();
        if ((value = value.trim()).length() > 0) {
            String[] names = NAME_SEPARATOR.split(value);
            if (names.length > 1) {
                throw new IllegalStateException("More than 1 default extension name on extension " + this.type.getName() + ": " + Arrays.toString(names));
            }

            if (names.length == 1) {
                this.cachedDefaultName = names[0];
            }
        }
    }

}
```

接下来分析loadDirectory 做了哪些事情。

```java
private void loadDirectory(Map<String, Class<?>> extensionClasses, String dir, String type) {
    // fileName = 文件夹路径 + type 全限定名 
    String fileName = dir + type;

    try {
        ClassLoader classLoader = findClassLoader();
        Enumeration urls;
        // 根据文件名加载所有的同名文件
        if (classLoader != null) {
            urls = classLoader.getResources(fileName);
        } else {
            urls = ClassLoader.getSystemResources(fileName);
        }

        if (urls != null) {
            while(urls.hasMoreElements()) {
                java.net.URL resourceURL = (java.net.URL)urls.nextElement();
                // 加载资源
                this.loadResource(extensionClasses, classLoader, resourceURL);
            }
        }
    } catch (Throwable var8) {
        logger.error("Exception occurred when loading extension class (interface: " + type + ", description file: " + fileName + ").", var8);
    }

}
```

loadDirectory 方法先通过 classLoader 获取所有资源链接，然后再通过 loadResource 方法加载资源。我们继续跟下去，看一下 loadResource 方法的实现。

```java
private void loadResource(Map<String, Class<?>> extensionClasses, ClassLoader classLoader, java.net.URL resourceURL) {
    try {
        BufferedReader reader = new BufferedReader(new InputStreamReader(resourceURL.openStream(), StandardCharsets.UTF_8));
        Throwable var5 = null;

        try {
            String line;
            try {
                // 按行读取配置内容
                while((line = reader.readLine()) != null) {
                    // 定位 # 字符
                    int ci = line.indexOf(35);
                    if (ci >= 0) {
                        // 截取 # 之前的字符串，# 之后的内容为注释，需要忽略
                        line = line.substring(0, ci);
                    }

                    line = line.trim();
                    if (line.length() > 0) {
                        try {
                            String name = null;
                            int i = line.indexOf(61);
                            if (i > 0) {
                                // 以等于号 = 为界，截取键与值
                                name = line.substring(0, i).trim();
                                line = line.substring(i + 1).trim();
                            }

                            if (line.length() > 0) {
                                // 加载类，并通过 loadClass 方法对类进行缓存
                                this.loadClass(extensionClasses, resourceURL, Class.forName(line, true, classLoader), name);
                            }
                        } catch (Throwable var19) {
                            IllegalStateException e = new IllegalStateException("Failed to load extension class (interface: " + this.type + ", class line: " + line + ") in " + resourceURL + ", cause: " + var19.getMessage(), var19);
                            this.exceptions.put(line, e);
                        }
                    }
                }
            } catch (Throwable var20) {
                var5 = var20;
                throw var20;
            }
        } finally {
            if (reader != null) {
                if (var5 != null) {
                    try {
                        reader.close();
                    } catch (Throwable var18) {
                        var5.addSuppressed(var18);
                    }
                } else {
                    reader.close();
                }
            }

        }
    } catch (Throwable var22) {
        logger.error("Exception occurred when loading extension class (interface: " + this.type + ", class file: " + resourceURL + ") in " + resourceURL, var22);
    }

}
```

loadResource 方法用于读取和解析配置文件，并通过反射加载类，最后调用 loadClass 方法进行其他操作。loadClass 方法用于主要用于操作缓存，该方法的逻辑如下：

```java
private void loadClass(Map<String, Class<?>> extensionClasses, java.net.URL resourceURL, Class<?> clazz, String name) throws NoSuchMethodException {
    if (!this.type.isAssignableFrom(clazz)) {
        throw new IllegalStateException("Error occurred when loading extension class (interface: " + this.type + ", class line: " + clazz.getName() + "), class " + clazz.getName() + " is not subtype of interface.");
    } else {
        // 检测目标类上是否有 Adaptive 注解
        if (clazz.isAnnotationPresent(Adaptive.class)) {
            // 设置 cachedAdaptiveClass缓存
            this.cacheAdaptiveClass(clazz);
        } else if (this.isWrapperClass(clazz)) {
            // 检测 clazz 是否是 Wrapper 类型,存储 clazz 到 cachedWrapperClasses 缓存中
            this.cacheWrapperClass(clazz);
        } else {
            // 检测 clazz 是否有默认的构造方法，如果没有，则抛出异常
            clazz.getConstructor();
            if (StringUtils.isEmpty(name)) {
                name = this.findAnnotationName(clazz);
                if (name.length() == 0) {
                    throw new IllegalStateException("No such extension name for the class " + clazz.getName() + " in the config " + resourceURL);
                }
            }
 			// 切分 name
            String[] names = NAME_SEPARATOR.split(name);
            if (ArrayUtils.isNotEmpty(names)) {
                // 如果类上有 Activate 注解，则使用 names 数组的第一个元素作为键，
                // 存储 name 到 Activate 注解对象的映射关系
                this.cacheActivateClass(clazz, names[0]);
                String[] var6 = names;
                int var7 = names.length;

                for(int var8 = 0; var8 < var7; ++var8) {
                    String n = var6[var8];
                     // 存储 Class 到名称的映射关系
                    this.cacheName(clazz, n);
                    // 存储名称到 Class 的映射关系
                    this.saveInExtensionClass(extensionClasses, clazz, n);
                }
            }
        }

    }
}
```

如上，loadClass 方法操作了不同的缓存，比如 cachedAdaptiveClass、cachedWrapperClasses 和 cachedNames 等等。除此之外，该方法没有其他什么逻辑了。

#### 6.1.3.2 Dubbo IOC

Dubbo IOC 是通过 setter 方法注入依赖。Dubbo 首先会通过反射获取到实例的所有方法，然后再遍历方法列表，检测方法名是否具有 setter 方法特征。若有，则通过 ObjectFactory 获取依赖对象，最后通过反射调用 setter 方法将依赖设置到目标对象中。整个过程对应的代码如下：

```java
private T injectExtension(T instance) {
    try {
        if (this.objectFactory != null) {
            Method[] var2 = instance.getClass().getMethods();
            int var3 = var2.length;
			// 遍历目标类的所有方法
            for(int var4 = 0; var4 < var3; ++var4) {
                Method method = var2[var4];
                // 通过isSetter方法检测方法是否以 set 开头，且方法仅有一个参数，且方法访问级别为 public
                if (this.isSetter(method) && method.getAnnotation(DisableInject.class) == null) {
                    // 获取 setter 方法参数类型
                    Class<?> pt = method.getParameterTypes()[0];
                    if (!ReflectUtils.isPrimitives(pt)) {
                        try {
                            // 获取属性名，比如 setName 方法对应属性名 name
                            String property = this.getSetterProperty(method);
                            // 从 ObjectFactory 中获取依赖对象
                            Object object = this.objectFactory.getExtension(pt, property);
                            if (object != null) {
                                // 通过反射调用 setter 方法设置依赖
                                method.invoke(instance, object);
                            }
                        } catch (Exception var9) {
                            logger.error("Failed to inject via method " + method.getName() + " of interface " + this.type.getName() + ": " + var9.getMessage(), var9);
                        }
                    }
                }
            }
        }
    } catch (Exception var10) {
        logger.error(var10.getMessage(), var10);
    }

    return instance;
}
```

上面代码中，objectFactory 变量的类型为 AdaptiveExtensionFactory，AdaptiveExtensionFactory 内部维护了一个 ExtensionFactory 列表，用于存储其他类型的 ExtensionFactory。

Dubbo 目前提供了两种 ExtensionFactory，分别是 SpiExtensionFactory 和 SpringExtensionFactory。前者用于创建自适应的拓展，后者是用于从 Spring 的 IOC 容器中获取所需的拓展。这两个类的类的代码不是很复杂，这里就不一一分析了。

Dubbo IOC 目前仅支持 setter 方式注入，总的来说，逻辑比较简单易懂。

## 6.2 自适应拓展机制

在 Dubbo 中，很多拓展都是通过 SPI 机制进行加载的，比如 Protocol、Cluster、LoadBalance 等。有时，有些拓展并不想在框架启动阶段被加载，而是希望在拓展方法被调用时，根据运行时参数进行加载。这听起来有些矛盾。拓展未被加载，那么拓展方法就无法被调用（静态方法除外）。拓展方法未被调用，拓展就无法被加载。对于这个矛盾的问题，Dubbo 通过自适应拓展机制很好的解决了。自适应拓展机制的实现逻辑比较复杂，首先 Dubbo 会为拓展接口生成具有代理功能的代码。然后通过 javassist 或 jdk 编译这段代码，得到 Class 类。最后再通过反射创建代理类

### 6.2.1 Adaptive 注解

在对自适应拓展生成过程进行深入分析之前，先来看一下与自适应拓展息息相关的一个注解，即 Adaptive 注解。该注解的定义如下：

```Java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Adaptive {
    String[] value() default {};
}
```

从上面的代码中可知，Adaptive 可注解在类或方法上。**当 Adaptive 注解在类上时，Dubbo 不会为该类生成代理类。注解在方法（接口方法）上时，Dubbo 则会为该方法生成代理逻辑。**Adaptive 注解在类上的情况很少，在 Dubbo 中，仅有两个类被 Adaptive 注解了，分别是 AdaptiveCompiler 和 AdaptiveExtensionFactory。此种情况，表示拓展的加载逻辑由人工编码完成。以AdaptiveExtensionFactory为例：

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.apache.dubbo.common.extension.factory;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Iterator;
import java.util.List;
import org.apache.dubbo.common.extension.Adaptive;
import org.apache.dubbo.common.extension.ExtensionFactory;
import org.apache.dubbo.common.extension.ExtensionLoader;

@Adaptive
public class AdaptiveExtensionFactory implements ExtensionFactory {
    private final List<ExtensionFactory> factories;

    public AdaptiveExtensionFactory() {
        ExtensionLoader<ExtensionFactory> loader = ExtensionLoader.getExtensionLoader(ExtensionFactory.class);
        List<ExtensionFactory> list = new ArrayList();
        Iterator var3 = loader.getSupportedExtensions().iterator();

        while(var3.hasNext()) {
            String name = (String)var3.next();
            list.add(loader.getExtension(name));
        }

        this.factories = Collections.unmodifiableList(list);
    }

    public <T> T getExtension(Class<T> type, String name) {
        Iterator var3 = this.factories.iterator();

        Object extension;
        do {
            if (!var3.hasNext()) {
                return null;
            }

            ExtensionFactory factory = (ExtensionFactory)var3.next();
            extension = factory.getExtension(type, name);
        } while(extension == null);

        return extension;
    }
}
```

可见其getExtension方法自己进行了实现，属性factories中放的就是两个类： SPIExtensionFactory跟SpringExtensionFactory，分别是Dubbo自身的SPI扩展工厂以及Spring的相关扩展工厂。

**在拓展接口的方法被调用时，通过 SPI 加载具体的拓展实现类，并调用拓展对象的同名方法。**

**更多时候，Adaptive 是注解在接口方法上的，表示拓展的加载逻辑需由框架自动生成。**

注解加在方法上的情况，以Protocol接口为例：

```java
package org.apache.dubbo.rpc;

import org.apache.dubbo.common.URL;
import org.apache.dubbo.common.extension.Adaptive;
import org.apache.dubbo.common.extension.SPI;

@SPI("dubbo")
public interface Protocol {
    int getDefaultPort();

    @Adaptive
    <T> Exporter<T> export(Invoker<T> var1) throws RpcException;

    @Adaptive
    <T> Invoker<T> refer(Class<T> var1, URL var2) throws RpcException;

    void destroy();
}
```

在ServiceConfig类中的doExportUrlsFor1Protocol方法中，有一段这样的代码：

```java
1 Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
2 DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
3 Exporter<?> exporter = protocol.export(wrapperInvoker);
```

此处就是往远程导出服务的触发点。先生成了invoker，然后生成invoker的包装类可以看到在第三行调用了protocol接口的export方法。protocol属性为：

```java
private static final Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
```

### 6.2.2 获取自适应拓展

getAdaptiveExtension 方法是获取自适应拓展的入口方法，因此下面我们从这个方法进行分析。相关代码如下：

```java
public T getAdaptiveExtension() {
    //从缓存中获取自适应扩展
    Object instance = this.cachedAdaptiveInstance.get();
    if (instance == null) {
        //缓存未命中
        if (this.createAdaptiveInstanceError != null) {
            throw new IllegalStateException("Failed to create adaptive instance: " + this.createAdaptiveInstanceError.toString(), this.createAdaptiveInstanceError);
        }

        Holder var2 = this.cachedAdaptiveInstance;
        synchronized(this.cachedAdaptiveInstance) {
            instance = this.cachedAdaptiveInstance.get();
            if (instance == null) {
                try {
                    //创建自适应拓展
                    instance = this.createAdaptiveExtension();
                    //设置到缓存中
                    this.cachedAdaptiveInstance.set(instance);
                } catch (Throwable var5) {
                    this.createAdaptiveInstanceError = var5;
                    throw new IllegalStateException("Failed to create adaptive instance: " + var5.toString(), var5);
                }
            }
        }
    }

    return instance;
}
```

getAdaptiveExtension 方法首先会检查缓存，缓存未命中，则调用 createAdaptiveExtension 方法创建自适应拓展。下面，看一下 createAdaptiveExtension 方法的代码。

```java
private T createAdaptiveExtension() {
    try {
        // 获取自适应拓展类，并通过反射实例化
        return this.injectExtension(this.getAdaptiveExtensionClass().newInstance());
    } catch (Exception var2) {
        throw new IllegalStateException("Can't create adaptive extension " + this.type + ", cause: " + var2.getMessage(), var2);
    }
}
```

createAdaptiveExtension 方法的代码比较少，但却包含了三个逻辑，分别如下：

1. 调用 getAdaptiveExtensionClass 方法获取自适应拓展 Class 对象
2. 通过反射进行实例化
3. 调用 injectExtension 方法向拓展实例中注入依赖

前两个逻辑比较好理解，第三个逻辑用于向自适应拓展对象中注入依赖。这个逻辑看似多余，但有存在的必要，这里简单说明一下。前面说过，Dubbo 中有两种类型的自适应拓展，一种是手工编码的，一种是自动生成的。手工编码的自适应拓展中可能存在着一些依赖，而自动生成的 Adaptive 拓展则不会依赖其他类。这里调用 injectExtension 方法的目的是为手工编码的自适应拓展注入依赖，这一点需要大家注意一下。关于 injectExtension 方法，前文已经分析过了，这里不再赘述。接下来，分析 getAdaptiveExtensionClass 方法的逻辑。

```java
private Class<?> getAdaptiveExtensionClass() {
    this.getExtensionClasses();
    return this.cachedAdaptiveClass != null ? this.cachedAdaptiveClass : (this.cachedAdaptiveClass = this.createAdaptiveExtensionClass());
}
```

```java
private Class<?> createAdaptiveExtensionClass() {
    // 构建自适应拓展代码
    String code = (new AdaptiveClassCodeGenerator(this.type, this.cachedDefaultName)).generate();
    ClassLoader classLoader = findClassLoader();
    // 获取编译器实现类
    Compiler compiler = (Compiler)getExtensionLoader(Compiler.class).getAdaptiveExtension();
    // 编译代码，生成 Class
    return compiler.compile(code, classLoader);
}
```

createAdaptiveExtensionClass 方法用于生成自适应拓展类，该方法首先会生成自适应拓展类的源码，然后通过 Compiler 实例（Dubbo 默认使用 javassist 作为编译器）编译源码，得到代理类 Class 实例。接下来，把重点放在代理类代码生成的逻辑上，其他逻辑大家自行分析。

### 6.2.3 自适应拓展类代码生成

```java 
public String generate() {
    if (!this.hasAdaptiveMethod()) {
        throw new IllegalStateException("No adaptive method exist on extension " + this.type.getName() + ", refuse to create the adaptive class!");
    } else {
        StringBuilder code = new StringBuilder();
        code.append(this.generatePackageInfo());
        code.append(this.generateImports());
        code.append(this.generateClassDeclaration());
        Method[] methods = this.type.getMethods();
        Method[] var3 = methods;
        int var4 = methods.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            Method method = var3[var5];
            code.append(this.generateMethod(method));
        }

        code.append("}");
        if (logger.isDebugEnabled()) {
            logger.debug(code.toString());
        }

        return code.toString();
    }
}
```

#### 6.2.3.1 Adaptive 注解检测

```java
private boolean hasAdaptiveMethod() {
    return Arrays.stream(this.type.getMethods()).anyMatch((m) -> {
        return m.isAnnotationPresent(Adaptive.class);
    });
}
```

#### 6.2.3.2 生成类

通过 Adaptive 注解检测后，即可开始生成代码。代码生成的顺序与 Java 文件内容顺序一致，首先会生成 package 语句，然后生成 import 语句，紧接着生成类名等代码。在generatePackageInfo、generateImports和

generateClassDeclaration这三个方法中实现。

```Java
 private String generatePackageInfo() {
        return String.format("package %s;\n", this.type.getPackage().getName());
    }

    private String generateImports() {
        return String.format("import %s;\n", ExtensionLoader.class.getName());
    }

    private String generateClassDeclaration() {
        return String.format("public class %s$Adaptive implements %s {\n", this.type.getSimpleName(), this.type.getCanonicalName());
    }
```

以 Dubbo 的 Protocol 接口为例，生成的代码如下：

```Java
package com.alibaba.dubbo.rpc;
import com.alibaba.dubbo.common.extension.ExtensionLoader;
public class Protocol$Adaptive implements com.alibaba.dubbo.rpc.Protocol {
    // 省略方法代码
}
```

#### 6.2.3.3 生成方法

获取目标方法列表，然后生成，主要在generateMethod方法中完成

```java
private String generateMethod(Method method) {
    //获取方法返回类型
    String methodReturnType = method.getReturnType().getCanonicalName();
    //获取方法名称
    String methodName = method.getName();
    //获取方法内容
    String methodContent = this.generateMethodContent(method);
    //获取方法参数
    String methodArgs = this.generateMethodArguments(method);
    //获取方法异常
    String methodThrows = this.generateMethodThrows(method);
    return String.format("public %s %s(%s) %s {\n%s}\n", methodReturnType, methodName, methodArgs, methodThrows, methodContent);
}
```

一个方法可以被 Adaptive 注解修饰，也可以不被修饰。这里将未被 Adaptive 注解修饰的方法称为“无 Adaptive 注解方法”，下面我们先来看看此种方法的代码生成逻辑是怎样的。

> **无 Adaptive 注解方法代码生成逻辑**

```java
private String generateMethodContent(Method method) {
    Adaptive adaptiveAnnotation = (Adaptive)method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        // 如果方法上无 Adaptive 注解，则生成 throw new UnsupportedOperationException(...) 代码
        return this.generateUnsupported(method);
    } else {
        //无关代码
    }
}
```

```java
private String generateUnsupported(Method method) {
    return String.format("throw new UnsupportedOperationException(\"The method %s of interface %s is not adaptive method!\");\n", method, this.type.getName());
}
```

> **获取 URL 数据**

前面说过方法代理逻辑会从 URL 中提取目标拓展的名称，因此代码生成逻辑的一个重要的任务是从方法的参数列表或者其他参数中获取 URL 数据。举例说明一下，我们要为 Protocol 接口的 refer 和 export 方法生成代理逻辑。在运行时，通过反射得到的方法定义大致如下：

```java
Invoker refer(Class<T> arg0, URL arg1) throws RpcException;
Exporter export(Invoker<T> arg0) throws RpcException;
```

对于 refer 方法，通过遍历 refer 的参数列表即可获取 URL 数据，这个还比较简单。对于 export 方法，获取 URL 数据则要麻烦一些。export 参数列表中没有 URL 参数，因此需要从 Invoker 参数中获取 URL 数据。获取方式是调用 Invoker 中可返回 URL 的 getter 方法，比如 getUrl。如果 Invoker 中无相关 getter 方法，此时则会抛出异常。整个逻辑如下：

```java
private String generateMethodContent(Method method) {
    Adaptive adaptiveAnnotation = (Adaptive)method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        //无关代码
    } else {
        int urlTypeIndex = this.getUrlTypeIndex(method);
        if (urlTypeIndex != -1) {
            code.append(this.generateUrlNullCheck(urlTypeIndex));
        } else {
            code.append(this.generateUrlAssignmentIndirectly(method));
        }

        //无关代码
    }
}
```

首先获取获取url的参数索引：

```java
private int getUrlTypeIndex(Method method) {
    int urlTypeIndex = -1;
    Class<?>[] pts = method.getParameterTypes();

    for(int i = 0; i < pts.length; ++i) {
        if (pts[i].equals(URL.class)) {
            urlTypeIndex = i;
            break;
        }
    }

    return urlTypeIndex;
}
```

如果参数列表中存在url参数，那么调用generateUrlNullCheck方法，为URL类型参数生成判空代码格式如下：

```java
	if (arg + urlTypeIndex == null) 
        throw new IllegalArgumentException("url == null");
```

```java
private String generateUrlNullCheck(int index) {
    return String.format("if (arg%d == null) throw new IllegalArgumentException(\"url == null\");\n%s url = arg%d;\n", index, URL.class.getName(), index);
}
```

如果参数列表中不存在url参数，那么调用generateUrlAssignmentIndirectly方法，这里面遍历方法的参数类型列表，获取某一类型参数的全部方法，寻找可返回 URL 的 getter 方法。

```java
private String generateUrlAssignmentIndirectly(Method method) {
    Class<?>[] pts = method.getParameterTypes();

    for(int i = 0; i < pts.length; ++i) {
        Method[] var4 = pts[i].getMethods();
        int var5 = var4.length;

        for(int var6 = 0; var6 < var5; ++var6) {
            Method m = var4[var6];
            String name = m.getName();
            if ((name.startsWith("get") || name.length() > 3) && Modifier.isPublic(m.getModifiers()) && !Modifier.isStatic(m.getModifiers()) && m.getParameterTypes().length == 0 && m.getReturnType() == URL.class) {
                return this.generateGetUrlNullCheck(i, pts[i], name);
            }
        }
    }

    throw new IllegalStateException("Failed to create adaptive class for interface " + this.type.getName() + ": not found url parameter or url attribute in parameters of method " + method.getName());
}
```

找到后调用generateGetUrlNullCheck方法：

```java
private String generateGetUrlNullCheck(int index, Class<?> type, String method) {
    StringBuilder code = new StringBuilder();
    code.append(String.format("if (arg%d == null) throw new IllegalArgumentException(\"%s argument == null\");\n", index, type.getName()));
    code.append(String.format("if (arg%d.%s() == null) throw new IllegalArgumentException(\"%s argument %s() == null\");\n", index, method, type.getName(), method));
    code.append(String.format("%s url = arg%d.%s();\n", URL.class.getName(), index, method));
    return code.toString();
}
```

上述代码主要目的是为了获取 URL 数据，并为之生成判空和赋值代码。以 Protocol 的 refer 和 export 方法为例，上面的代码为它们生成如下内容

```java
refer:
if (arg1 == null) 
    throw new IllegalArgumentException("url == null");
com.alibaba.dubbo.common.URL url = arg1;

export:
if (arg0 == null) 
    throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument == null");
if (arg0.getUrl() == null) 
    throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument getUrl() == null");
com.alibaba.dubbo.common.URL url = arg0.getUrl();
```

> **获取 Adaptive 注解值**

```java
private String generateMethodContent(Method method) {
    Adaptive adaptiveAnnotation = (Adaptive)method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        //无关代码
    } else {
        //无关代码
        String[] value = this.getMethodAdaptiveValue(adaptiveAnnotation);
        //无关代码
    }
}
```

Adaptive 注解值 value 类型为 String[]，可填写多个值，默认情况下为空数组。若 value 为非空数组，直接获取数组内容即可。若 value 为空数组，则需进行额外处理。处理过程是将类名转换为字符数组，然后遍历字符数组，并将字符放入 StringBuilder 中。若字符为大写字母，则向 StringBuilder 中添加点号，随后将字符变为小写存入 StringBuilder 中。比如 LoadBalance 经过处理后，得到 load.balance。

```java
private String[] getMethodAdaptiveValue(Adaptive adaptiveAnnotation) {
    String[] value = adaptiveAnnotation.value();
    if (value.length == 0) {
        String splitName = StringUtils.camelToSplitName(this.type.getSimpleName(), ".");
        value = new String[]{splitName};
    }

    return value;
}
```

> **检测 Invocation 参数**

```java
private String generateMethodContent(Method method) {
    Adaptive adaptiveAnnotation = (Adaptive)method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        //无关代码
    } else {
        //无关代码
        boolean hasInvocation = this.hasInvocationArgument(method);
        code.append(this.generateInvocationArgumentNullCheck(method));
       //无关代码
    }
}
```

此段逻辑是检测方法列表中是否存在 Invocation 类型的参数，若存在，则为其生成判空代码和其他一些代码。相应的逻辑如下：

```java
private boolean hasInvocationArgument(Method method) {
    Class<?>[] pts = method.getParameterTypes();
    return Arrays.stream(pts).anyMatch((p) -> {
        return "org.apache.dubbo.rpc.Invocation".equals(p.getName());
    });
}
```

```java
private String generateInvocationArgumentNullCheck(Method method) {
    Class<?>[] pts = method.getParameterTypes();
    return (String)IntStream.range(0, pts.length).filter((i) -> {
        return "org.apache.dubbo.rpc.Invocation".equals(pts[i].getName());
    }).mapToObj((i) -> {
        return String.format("if (arg%d == null) throw new IllegalArgumentException(\"invocation == null\"); String methodName = arg%d.getMethodName();\n", i, i);
    }).findFirst().orElse("");
}
```

>  **生成拓展名获取逻辑**

```java
private String generateMethodContent(Method method) {
    Adaptive adaptiveAnnotation = (Adaptive)method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        //无关代码
    } else {
		//无关代码
        code.append(this.generateExtNameAssignment(value, hasInvocation));
        code.append(this.generateExtNameNullCheck(value));
		//无关代码
    }
}
```

本段逻辑用于根据 SPI 和 Adaptive 注解值生成“获取拓展名逻辑”，同时生成逻辑也受 Invocation 类型参数影响，综合因素导致本段逻辑相对复杂。本段逻辑可能会生成但不限于下面的代码：

```java
String extName = (url.getProtocol() == null ? "dubbo" : url.getProtocol());
String extName = url.getMethodParameter(methodName, "loadbalance", "random");
String extName = url.getParameter("client", url.getParameter("transporter", "netty"));
```

具体逻辑如下：

```java
private String generateExtNameAssignment(String[] value, boolean hasInvocation) {
    //这里的value就是Adaptive 注解值
    String getNameCode = null;
	//这个循环的目的是生成从 URL 中获取拓展名的代码，生成的代码会赋值给 getNameCode 变量。注意这
    //个循环的遍历顺序是由后向前遍历的。
    for(int i = value.length - 1; i >= 0; --i) {
        if (i == value.length - 1) {
            // cachedDefaultName 源于 SPI 注解值，默认情况下，
            //SPI 注解值为空串，此时cachedDefaultName = null
            if (null != this.defaultExtName) {
                // protocol 是 url 的一部分，可通过 getProtocol 方法获取，其他的则是从
                // URL 参数中获取。因为获取方式不同，所以这里要判断 value[i] 是否为 protocol
                if (!"protocol".equals(value[i])) {
                    if (hasInvocation) {
                        // 生成的代码功能等价于下面的代码：
                        //  url.getMethodParameter(methodName, value[i], defaultExtName)
                        // 以 LoadBalance 接口的 select 方法为例，最终生成的代码如下：
                        //  url.getMethodParameter(methodName, "loadbalance", "random")
                        getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], this.defaultExtName);
                    } else {
                        // 生成的代码功能等价于下面的代码：
	                   //   url.getParameter(value[i], defaultExtName)
                        getNameCode = String.format("url.getParameter(\"%s\", \"%s\")", value[i], this.defaultExtName);
                    }
                } else {
                  // 生成的代码功能等价于下面的代码：
                  //   ( url.getProtocol() == null ? defaultExtName : url.getProtocol() )
                    getNameCode = String.format("( url.getProtocol() == null ? \"%s\" : url.getProtocol() )", this.defaultExtName);
                }
                // 默认拓展名为空
            } else if (!"protocol".equals(value[i])) {
              
                if (hasInvocation) {
                    getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], this.defaultExtName);
                } else {
                    // 生成的代码功能等价于下面的代码：
	                //   url.getParameter(value[i])
                    getNameCode = String.format("url.getParameter(\"%s\")", value[i]);
                }
            } else {
                // 生成从 url 中获取协议的代码，比如 "dubbo"
                getNameCode = "url.getProtocol()";
            }
        } else if (!"protocol".equals(value[i])) {
            if (hasInvocation) {
                getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], this.defaultExtName);
            } else {
                // 生成的代码功能等价于下面的代码：
                //   url.getParameter(value[i], getNameCode)
                // 以 Transporter 接口的 connect 方法为例，最终生成的代码如下：
                //   url.getParameter("client", url.getParameter("transporter", "netty"))
                getNameCode = String.format("url.getParameter(\"%s\", %s)", value[i], getNameCode);
            }
        } else {
            // 生成的代码功能等价于下面的代码：
            //   url.getProtocol() == null ? getNameCode : url.getProtocol()
            // 以 Protocol 接口的 connect 方法为例，最终生成的代码如下：
            //   url.getProtocol() == null ? "dubbo" : url.getProtocol()
            getNameCode = String.format("url.getProtocol() == null ? (%s) : url.getProtocol()", getNameCode);
        }
    }

    return String.format("String extName = %s;\n", getNameCode);
}
```

上面代码比较复杂，不是很好理解。对于这段代码，建议对 Protocol、LoadBalance 以及 Transporter 等接口的自适应拓展类代码生成过程进行调试。这里我以 Transporter 接口的自适应拓展类代码生成过程举例说明。首先看一下 Transporter 接口的定义，如下：

```java
@SPI("netty")
public interface Transporter {
	// @Adaptive({server, transporter})
    @Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY}) 
    Server bind(URL url, ChannelHandler handler) throws RemotingException;

    // @Adaptive({client, transporter})
    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;
}
```

下面对 connect 方法代理逻辑生成的过程进行分析，此时生成代理逻辑所用到的变量如下：

```java
String defaultExtName = "netty";
boolean hasInvocation = false;
String getNameCode = null;
String[] value = ["client", "transporter"];
```

下面对 value 数组进行遍历，此时 i = 1, value[i] = "transporter"，生成的代码如下：

```java
getNameCode = url.getParameter("transporter", "netty");
```

接下来，for 循环继续执行，此时 i = 0, value[i] = "client"，生成的代码如下：

```java
getNameCode = url.getParameter("client", url.getParameter("transporter", "netty"));
```

for 循环结束运行，现在为 extName 变量生成赋值和判空代码，如下：

```java
private String generateExtNameNullCheck(String[] value) {
    return String.format("if(extName == null) throw new IllegalStateException(\"Failed to get extension (%s) name from url (\" + url.toString() + \") use keys(%s)\");\n", this.type.getName(), Arrays.toString(value));
}
```

```java
String extName = url.getParameter("client", url.getParameter("transporter", "netty"));
if (extName == null) {
    throw new IllegalStateException(
        "Fail to get extension(com.alibaba.dubbo.remoting.Transporter) name from url(" + url.toString()
        + ") use keys([client, transporter])");
}
```

> **生成拓展加载与目标方法调用逻辑**

本段代码逻辑用于根据拓展名加载拓展实例，并调用拓展实例的目标方法。相关逻辑如下：

```java
private String generateMethodContent(Method method) {
    Adaptive adaptiveAnnotation = (Adaptive)method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
       //无关代码
    } else {
        //无关代码
        code.append(this.generateExtensionAssignment());
        code.append(this.generateReturnAndInvocation(method));
        //无关代码
    }
}
```

```java
private String generateExtensionAssignment() {
    // 生成拓展获取代码，格式如下：
    // type全限定名 extension = (type全限定名)ExtensionLoader全限定名
    //     .getExtensionLoader(type全限定名.class).getExtension(extName);
    // Tips: 格式化字符串中的 %<s 表示使用前一个转换符所描述的参数，即 type 全限定名
    return String.format("%s extension = (%<s)%s.getExtensionLoader(%s.class).getExtension(extName);\n", this.type.getName(), ExtensionLoader.class.getSimpleName(), this.type.getName());
}
```

```java
private String generateReturnAndInvocation(Method method) {
    // 如果方法返回值类型非 void，则生成 return 语句。
    String returnStatement = method.getReturnType().equals(Void.TYPE) ? "" : "return ";
    // 生成目标方法调用逻辑，格式为：
    //     extension.方法名(arg0, arg2, ..., argN);
    String args = (String)Arrays.stream(method.getParameters()).map(Parameter::getName).collect(Collectors.joining(", "));
    return returnStatement + String.format("extension.%s(%s);\n", method.getName(), args);
}
```

以 Protocol 接口举例说明，上面代码生成的内容如下：

```java
com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol) ExtensionLoader
    .getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);
return extension.refer(arg0, arg1);
```

> **生成完整的方法**

```java
private String generateMethod(Method method) {
    String methodReturnType = method.getReturnType().getCanonicalName();
    String methodName = method.getName();
    String methodContent = this.generateMethodContent(method);
    //上面的工作已经将代码生成完毕，开始最后的收尾
    String methodArgs = this.generateMethodArguments(method);
    String methodThrows = this.generateMethodThrows(method);
    return String.format("public %s %s(%s) %s {\n%s}\n", methodReturnType, methodName, methodArgs, methodThrows, methodContent);
}
```

本节进行代码生成的收尾工作，主要用于生成方法定义的代码。相关逻辑如下：

```java
private String generateMethodArguments(Method method) {
    //添加参数列表
    Class<?>[] pts = method.getParameterTypes();
    return (String)IntStream.range(0, pts.length).mapToObj((i) -> {
        return String.format("%s arg%d", pts[i].getCanonicalName(), i);
    }).collect(Collectors.joining(", "));
}
```

```java
private String generateMethodThrows(Method method) {
    // 添加异常抛出代码
    Class<?>[] ets = method.getExceptionTypes();
    if (ets.length > 0) {
        String list = (String)Arrays.stream(ets).map(Class::getCanonicalName).collect(Collectors.joining(", "));
        return String.format("throws %s", list);
    } else {
        return "";
    }
}
```

最后，将所有内容进行组合：

```java
return String.format("public %s %s(%s) %s {\n%s}\n", methodReturnType, methodName, methodArgs, methodThrows, methodContent);
```

以 Protocol 的 refer 方法为例，最终生成的代码如下所示：

```java
public com.alibaba.dubbo.rpc.Invoker refer(java.lang.Class arg0,
			com.alibaba.dubbo.common.URL arg1)
			throws com.alibaba.dubbo.rpc.RpcException {
		if (arg1 == null)
			throw new IllegalArgumentException("url == null");
		com.alibaba.dubbo.common.URL url = arg1;
		String extName = (url.getProtocol() == null ? "dubbo" : url
				.getProtocol());
		if (extName == null)
			throw new IllegalStateException(
					"Fail to get extension(com.alibaba.dubbo.rpc.Protocol) name from url(" + url.toString() + ") use keys([protocol])");
		com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol) ExtensionLoader
				.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class)
				.getExtension(extName);
		return extension.refer(arg0, arg1);
	}
```

#### 6.2.3.4 编译

```java
private Class<?> createAdaptiveExtensionClass() {
    //生成代码
    String code = (new AdaptiveClassCodeGenerator(this.type, this.cachedDefaultName)).generate();
    ClassLoader classLoader = findClassLoader();
    //调用编译器对代码进行编译
    Compiler compiler = (Compiler)getExtensionLoader(Compiler.class).getAdaptiveExtension();
    return compiler.compile(code, classLoader);
}
```

### 6.2.4 Demo

有一个车轮制造厂接口 WheelMaker:

```java 
public interface WheelMaker {
    Wheel makeWheel(URL url);
}
```

WheelMaker 接口的自适应实现类如下：

```java
public class AdaptiveWheelMaker implements WheelMaker {
    public Wheel makeWheel(URL url) {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        
    	// 1.从 URL 中获取 WheelMaker 名称
        String wheelMakerName = url.getParameter("Wheel.maker");
        if (wheelMakerName == null) {
            throw new IllegalArgumentException("wheelMakerName == null");
        }
        
        // 2.通过 SPI 加载具体的 WheelMaker
        WheelMaker wheelMaker = ExtensionLoader
            .getExtensionLoader(WheelMaker.class).getExtension(wheelMakerName);
        
        // 3.调用目标方法
        return wheelMaker.makeWheel(url);
    }
}
```

AdaptiveWheelMaker 是一个代理类，与传统的代理逻辑不同，AdaptiveWheelMaker 所代理的对象是在 makeWheel 方法中通过 SPI 加载得到的。makeWheel 方法主要做了三件事情：

1. 从 URL 中获取 WheelMaker 名称
2. 通过 SPI 加载具体的 WheelMaker 实现类
3. 调用目标方法

接下来，我们来看看汽车制造厂 CarMaker 接口与其实现类。

```java
public interface CarMaker {
    Car makeCar(URL url);
}

public class RaceCarMaker implements CarMaker {
    WheelMaker wheelMaker;
 
    // 通过 setter 注入 AdaptiveWheelMaker
    public setWheelMaker(WheelMaker wheelMaker) {
        this.wheelMaker = wheelMaker;
    }
 
    public Car makeCar(URL url) {
        Wheel wheel = wheelMaker.makeWheel(url);
        return new RaceCar(wheel, ...);
    }
}
```

RaceCarMaker 持有一个 WheelMaker 类型的成员变量，在程序启动时，我们可以将 AdaptiveWheelMaker 通过 setter 方法注入到 RaceCarMaker 中。在运行时，假设有这样一个 url 参数传入：

```
dubbo://192.168.0.101:20880/XxxService?wheel.maker=MichelinWheelMaker
```

RaceCarMaker 的 makeCar 方法将上面的 url 作为参数传给 AdaptiveWheelMaker 的 makeWheel 方法，makeWheel 方法从 url 中提取 wheel.maker 参数，得到 MichelinWheelMaker。之后再通过 SPI 加载配置名为 MichelinWheelMaker 的实现类，得到具体的 WheelMaker 实例。

上面的示例展示了自适应拓展类的核心实现 ---- 在拓展接口的方法被调用时，通过 SPI 加载具体的拓展实现类，并调用拓展对象的同名方法。而自适应代码的生成其实就是在dubbo容器内部生成类似**AdaptiveWheelMaker** 类中的代码。

## 6.3 服务导出

dubbo 服务导出过程始于 Spring 容器发布刷新事件，Dubbo 在接收到事件后，会立即执行服务导出逻辑。整个逻辑大致可分为三个部分，第一部分是前置工作，主要用于检查参数，组装 URL。第二部分是导出服务，包含导出服务到本地 (JVM)，和导出服务到远程两个过程。第三部分是向注册中心注册服务，用于服务发现。