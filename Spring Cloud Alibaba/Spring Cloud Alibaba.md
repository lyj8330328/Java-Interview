分布式系统遇到的四个问题：

这么多服务，客户端如何访问——Zuul网关

这么多服务，客户端如何通信——Feign（http，同步阻塞）

这么多服务，如何治理——Eureka

服务挂了怎么办？——Hystix

dubbo 

zookeeper

微服务的困难

服务注册、发现

链路跟踪

负载均衡

超时

熔断

限流

健康检查

版本控制

日志收集

# 一、Spring Cloud Alibaba简介

## 1.1 主要功能

- **服务限流降级**：默认支持 Servlet、Feign、RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。

## 1.2 组件

- **Sentinel**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **Nacos**：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **RocketMQ**：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- **Alibaba Cloud ACM**：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
- **Alibaba Cloud OSS**: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **Alibaba Cloud SchedulerX**: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

# 二、Spring Cloud Alibaba

## 2.1 概述

在 Spring Cloud Netflix 阶段采用 Eureka 做作为服务注册与发现服务器，现利用 Spring Cloud Alibaba 提供的 Nacos 组件替代该方案。

## 2.2 什么是Nacos

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

## 2.3 基本架构及概念

![](http://mycsdnblog.work/201919201632-Z.png)

### 2.3.1 服务 (Service)

服务是指一个或一组软件功能（例如特定信息的检索或一组操作的执行），其目的是不同的客户端可以为不同的目的重用（例如通过跨进程的网络调用）。Nacos 支持主流的服务生态，如 Kubernetes Service、gRPC|Dubbo RPC Service 或者 Spring Cloud RESTful Service.

### 2.3.2 服务注册中心 (Service Registry)

服务注册中心，它是服务，其实例及元数据的数据库。服务实例在启动时注册到服务注册表，并在关闭时注销。服务和路由器的客户端查询服务注册表以查找服务的可用实例。服务注册中心可能会调用服务实例的健康检查 API 来验证它是否能够处理请求。

### 2.3.3 服务元数据 (Service Metadata)

服务元数据是指包括服务端点(endpoints)、服务标签、服务版本号、服务实例权重、路由规则、安全策略等描述服务的数据

### 2.3.4 服务提供方 (Service Provider)

是指提供可复用和可调用服务的应用方

### 2.3.5 服务消费方 (Service Consumer)

是指会发起对某个服务调用的应用方

### 2.3.6 配置 (Configuration)

在系统开发过程中通常会将一些需要变更的参数、变量等从代码中分离出来独立管理，以独立的配置文件的形式存在。目的是让静态的系统工件或者交付物（如 WAR，JAR 包等）更好地和实际的物理运行环境进行适配。配置管理一般包含在系统部署的过程中，由系统管理员或者运维人员完成这个步骤。配置变更是调整系统运行时的行为的有效手段之一。

### 2.3.7 配置管理 (Configuration Management)

在数据中心中，系统中所有配置的编辑、存储、分发、变更管理、历史版本管理、变更审计等所有与配置相关的活动统称为配置管理。

### 2.3.8 名字服务 (Naming Service)

提供分布式系统中所有对象(Object)、实体(Entity)的“名字”到关联的元数据之间的映射管理服务，例如 ServiceName -> Endpoints Info, Distributed Lock Name -> Lock Owner/Status Info, DNS Domain Name -> IP List, 服务发现和 DNS 就是名字服务的2大场景。

### 2.3.9 配置服务 (Configuration Service)

在服务或者应用运行过程中，提供动态配置或者元数据以及配置管理的服务提供者。

## 2.4 下载安装

### 2.4.1 准备环境

Nacos 依赖 Java 环境来运行。如果从代码开始构建并运行 Nacos，还需要为此配置 Maven 环境，请确保是在以下版本环境中安装使用:

- 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
- 64 bit JDK 1.8+
- Maven 3.2.x+

### 2.4.2 下载并安装

```bash
# 下载源码
git clone https://github.com/alibaba/nacos.git

# 安装到本地仓库
cd nacos/
mvn -Prelease-nacos clean install -U
```

-P（profile Maven 多环境配置 ）

-U：使用最新依赖

### 2.4.3 启动服务

```bash
cd distribution/target/nacos-server-1.1.3/nacos/bin

# Linux
./startup.sh -m standalone

# Windows
startup.cmd
```

### 2.4.4 访问服务

地址：http://localhost:8848/nacos

默认账号和密码为nacos

![](http://mycsdnblog.work/201919211730-m.png)

![](http://mycsdnblog.work/201919211736-G.png)

## 2.5 创建服务提供者

### 2.5.1 新建工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <groupId>com.springcloud.alibaba</groupId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-alibaba-provider</artifactId>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.example.NacosProviderApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

### 2.5.2 配置文件

```yaml
server:
  port: 8081
spring:
  application:
    name: nacos-provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
management:
  endpoints:
    web:
      exposure:
        include: "*"    #端点检测
```

### 2.5.3 启动类

```java
package com.example.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @Author: 98050
 * @Time: 2019-08-21 17:44
 * @Feature:
 */
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }
}
```

### 2.5.4 提供接口

```java
package com.example.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: 98050
 * @Time: 2019-08-21 17:52
 * @Feature:
 */
@RestController
public class StringController {

    @GetMapping("/test")
    public String test(){
        return "test";
    }
}
```

### 2.5.5 启动工程

![](http://mycsdnblog.work/201919211910-e.png)

### 2.5.6 服务端点检查

spring-cloud-starter-alibaba-nacos-discovery 在实现的时候提供了一个 EndPoint, EndPoint 的访问地址为 `http://ip:port/actuator/nacos-discovery`。 EndPoint 的信息主要提供了两类:

```text
1、subscribe: 显示了当前有哪些服务订阅者
2、NacosDiscoveryProperties: 显示了当前服务实例关于 Nacos 的基础配置
```

通过浏览器访问 `http://localhost:8081/actuator/nacos-discovery` 你会在浏览器上看到：

![](http://mycsdnblog.work/201919211912-d.png)

与nacos中的服务详情对应：

![](http://mycsdnblog.work/201919211912-V.png)

## 2.6 Nacos Starter更多配置项信息

| 配置项          | Key                                            | 默认值                     | 说明                                                         |
| --------------- | ---------------------------------------------- | -------------------------- | ------------------------------------------------------------ |
| 服务端地址      | spring.cloud.nacos.discovery.server-addr       | 无                         | Nacos Server 启动监听的ip地址和端口                          |
| 服务名          | spring.cloud.nacos.discovery.service           | ${spring.application.name} | 给当前的服务命名                                             |
| 权重            | spring.cloud.nacos.discovery.weight            | 1                          | 取值范围 1 到 100，数值越大，权重越大                        |
| 网卡名          | spring.cloud.nacos.discovery.network-interface | 无                         | 当IP未配置时，注册的IP为此网卡所对应的IP地址，如果此项也未配置，则默认取第一块网卡的地址 |
| 注册的IP地址    | spring.cloud.nacos.discovery.ip                | 无                         | 优先级最高                                                   |
| 注册的端口      | spring.cloud.nacos.discovery.port              | -1                         | 默认情况下不用配置，会自动探测                               |
| 命名空间        | spring.cloud.nacos.discovery.namespace         | 无                         | 常用场景之一是不同环境的注册的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。 |
| AccessKey       | spring.cloud.nacos.discovery.access-key        | 无                         | 当要上阿里云时，阿里云上面的一个云账号名                     |
| SecretKey       | spring.cloud.nacos.discovery.secret-key        | 无                         | 当要上阿里云时，阿里云上面的一个云账号密码                   |
| Metadata        | spring.cloud.nacos.discovery.metadata          | 无                         | 使用 Map 格式配置，用户可以根据自己的需要自定义一些和服务相关的元数据信息 |
| 日志文件名      | spring.cloud.nacos.discovery.log-name          | 无                         |                                                              |
| 接入点          | spring.cloud.nacos.discovery.enpoint           | UTF-8                      | 地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址 |
| 是否集成 Ribbon | ribbon.nacos.enabled                           | true                       | 一般都设置成 true 即可                                       |

## 2.7 创建服务消费者

服务消费者的创建与服务提供者大同小异，这里采用最原始的一种方式，即显示的使用 LoadBalanceClient 和 RestTemplate 结合的方式来访问。

### 2.7.1 新建工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <groupId>com.springcloud.alibaba</groupId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-alibaba-consumer</artifactId>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.example.consumer.NacosConsumerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2.7.2 配置文件

```yaml
server:
  port: 9091
spring:
  application:
    name: nacos-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
management:
  endpoints:
    web:
      exposure:
        include: "*"    #端点检测
```

### 2.7.3 启动类

```java
package com.example.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @Author: 98050
 * @Time: 2019-08-21 19:23
 * @Feature:
 */
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }
}
```

### 2.7.4 配置类

```java
package com.example.consumer.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @Author: 98050
 * @Time: 2019-08-21 19:30
 * @Feature:
 */
@Configuration
public class NacosConsumerConfiguration {

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

### 2.7.5 Controller

```java
package com.example.consumer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @Author: 98050
 * @Time: 2019-08-21 19:31
 * @Feature:
 */
@RestController
public class NacosConsumerController {

    @Autowired
    private LoadBalancerClient loadBalancerClient;

    @Autowired
    private RestTemplate restTemplate;

    @Value("${spring.application.name}")
    private String name;

    @GetMapping("/getTest")
    public String getTest(){
        ServiceInstance instance = loadBalancerClient.choose("nacos-provider");
        String url = String.format("http://%s:%s/test", instance.getHost(),instance.getPort());
        return this.restTemplate.getForObject(url, String.class) + name;
    }
}
```

### 2.7.6 启动工程

![](http://mycsdnblog.work/201919212001-K.png)

### 2.7.7 服务端点检查

通过浏览器访问 `http://localhost:9091/actuator/nacos-discovery` 你会在浏览器上看到：

![](http://mycsdnblog.work/201919212004-h.png)

### 2.7.8 访问接口

http://localhost:9091/getTest

![](http://mycsdnblog.work/201919212311-j.png)

## 2.8 创建服务消费者（Feign）

Feign 是一个声明式的伪 Http 客户端，它使得写 Http 客户端变得更简单。使用 Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用 Feign 注解和 JAX-RS 注解。Feign 支持可插拔的编码器和解码器。Feign 默认集成了 Ribbon，Nacos 也很好的兼容了 Feign，默认实现了负载均衡的效果

- Feign 采用的是基于接口的注解
- Feign 整合了 Ribbon

### 2.8.1 新建工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <groupId>com.springcloud.alibaba</groupId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-alibaba-consumer-feign</artifactId>


    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>
</project>
```

### 2.8.2 配置文件

```yaml
server:
  port: 9092
spring:
  application:
    name: nacos-consumer-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
management:
  endpoints:
    web:
      exposure:
        include: "*"    #端点检测
```

### 2.8.3 启动类

```java
package com.example.consumer.feign;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * @Author: 98050
 * @Time: 2019-08-21 20:24
 * @Feature:
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class NacosConsumerFeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerFeignApplication.class, args);
    }
}
```

### 2.8.4 创建Feign Client接口

```java
package com.example.consumer.feign.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * @Author: 98050
 * @Time: 2019-08-21 20:29
 * @Feature:
 */
@FeignClient(value = "nacos-provider")
public interface FeignServiceTest {

    @GetMapping("/test")
    String getTest();
}
```

### 2.8.5 Controller

```java
package com.example.consumer.feign.controller;

import com.example.consumer.feign.client.FeignServiceTest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: 98050
 * @Time: 2019-08-21 20:29
 * @Feature:
 */
@RestController
public class NacosConsumerFeignController {

    @Autowired
    private FeignServiceTest feignTest;

    @GetMapping("/getTest")
    public String getTest(){
        return this.feignTest.getTest() + "from feign";
    }
}
```

### 2.8.6 启动工程

![](http://mycsdnblog.work/201919212040-v.png)

### 2.8.7 测试

![](http://mycsdnblog.work/201919212316-D.png)

### 2.8.8 测试负载均衡

- 启动多个 `consumer-provider` 实例，效果图如下：

![](http://mycsdnblog.work/201919212056-j.png)

修改nacos-provider中的代码：

```java
package com.example.provider.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: 98050
 * @Time: 2019-08-21 17:52
 * @Feature:
 */
@RestController
public class StringController {

    @Value("${server.port}")
    private String port;

    @GetMapping("/test")
    public String test(){
        return "test,port:" + port;
    }
}
```

测试接口：http://localhost:9092/feignTest

![](http://mycsdnblog.work/201919212103-e.png)

![](http://mycsdnblog.work/201919212102-I.png)

## 2.9 使用熔断防止服务器雪崩

在微服务架构中，根据业务来拆分成一个个的服务，服务与服务之间可以通过 `RPC` 相互调用，在 Spring Cloud 中可以用 `RestTemplate + LoadBalanceClient` 和 `Feign` 来调用。为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证 100% 可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，`Servlet` 容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的 **“雪崩”** 效应。

为了解决这个问题，业界提出了熔断器模型。

阿里巴巴开源了 Sentinel 组件，实现了熔断器模式，Spring Cloud 对这一组件进行了整合。在微服务架构中，一个请求需要调用多个服务是非常常见的，如下图：

![](http://mycsdnblog.work/201919212115-A.png)

较底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用的不可用达到一个阀值熔断器将会被打开。

![](http://mycsdnblog.work/201919212117-3.png)

熔断器打开后，为了避免连锁故障，通过 `fallback` 方法可以直接返回一个固定值。

### 2.9.1 什么是Sentinel

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。 Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

### 2.9.2 Sentinel的特征

- **丰富的应用场景：** Sentinel 承接了阿里巴巴近 10 年的 **双十一大促流量** 的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、实时熔断下游不可用应用等。
- **完备的实时监控：** Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态：** Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点：** Sentinel 提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑。例如定制规则管理、适配数据源等。

### 2.9.3 Feign中使用Sentinel

如果要在您的项目中引入 Sentinel，使用 group ID 为 `org.springframework.cloud` 和 artifact ID 为 `spring-cloud-starter-alibaba-sentinel` 的 starter。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

Sentinel 适配了 Feign 组件。但默认是关闭的。需要在配置文件中配置打开它，在配置文件增加以下代码：

```yaml
feign:
  sentinel:
    enabled: true
```

### 2.9.4 在FeignTest中添加fallback指定类

```java
package com.example.consumer.feign.client;

import com.example.consumer.feign.fallback.FeignServiceTestFallback;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * @Author: 98050
 * @Time: 2019-08-21 20:29
 * @Feature:
 */
@FeignClient(value = "nacos-provider",fallback = FeignServiceTestFallback.class)
public interface FeignServiceTest {

    @GetMapping("/test")
    String getTest();
}
```

```java
package com.example.consumer.feign.fallback;

import com.example.consumer.feign.client.FeignServiceTest;
import org.springframework.stereotype.Component;

/**
 * @Author: 98050
 * @Time: 2019-08-21 21:33
 * @Feature:
 */
@Component
public class FeignServiceTestFallback implements FeignServiceTest {

    @Override
    public String getTest() {
        return "test fallback";
    }
}
```

### 2.9.5 测试熔断器

此时关闭服务提供者：

![](http://mycsdnblog.work/201919212140-m.png)

然后再次请求http://localhost:9092/getTest

![](http://mycsdnblog.work/201919212318-p.png)

## 2.10 使用熔断器仪表盘监控

### 2.10.1 Sentinel控制台

Sentinel 控制台提供一个轻量级的控制台，它提供机器发现、单机资源实时监控、集群资源汇总，以及规则管理的功能。您只需要对应用进行简单的配置，就可以使用这些功能。

**注意:** 集群资源汇总仅支持 500 台以下的应用集群，有大概 1 - 2 秒的延时。

![](https://user-images.githubusercontent.com/9434884/55449295-84866d80-55fd-11e9-94e5-d3441f4a2b63.png)

### 2.10.2 下载并打包

```bash
# 下载源码
git clone https://github.com/alibaba/Sentinel.git

# 编译打包
mvn clean package
```

### 2.10.3 启动控制台

Sentinel 控制台是一个标准的 SpringBoot 应用，以 SpringBoot 的方式运行 jar 包即可。

```bash
cd sentinel-dashboard\target
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```

如若 8080 端口冲突，可使用 `-Dserver.port=新端口` 进行设置。

### 2.10.4 访问服务

打开浏览器访问：http://localhost:8080/#/dashboard/home

默认用户和密码：sentinel

![](http://mycsdnblog.work/201919212209-1.png)

### 2.10.5 配置控制台信息

在Feign的`application.yml` 配置文件中增加如下配置：

![](http://mycsdnblog.work/201919212202-t.png)

这里的 `spring.cloud.sentinel.transport.port` 端口配置会在应用对应的机器上启动一个 Http Server，该 Server 会与 Sentinel 控制台做交互。比如 Sentinel 控制台添加了 1 个限流规则，会把规则数据 push 给这个 Http Server 接收，Http Server 再将规则注册到 Sentinel 中

### 2.10.6 测试Sentinel

使用之前的 Feign 客户端，`application.yml` 完整配置如下：

```yaml
server:
  port: 9092
spring:
  application:
    name: nacos-consumer-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080
management:
  endpoints:
    web:
      exposure:
        include: "*"    #端点检测
feign:
  sentinel:
    enabled: true
```

打开浏览器访问：http://localhost:8080/#/dashboard/home

此时会多一个名为 `nacos-consumer-feign` 的服务，执行几次熔断，然后查看

![](http://mycsdnblog.work/201919212212-D.png)

## 2.11 使用路由网关统一访问接口

### 2.11.1 什么是Spring Cloud GateWay

Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，Spring Cloud Gateway 旨在为微服务架构提供一种简单而有效的统一的 API 路由管理方式。**Spring Cloud Gateway 作为 Spring Cloud 生态系中的网关，目标是替代 Netflix ZUUL**，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。

![](http://mycsdnblog.work/201919212216-g.png)

### 2.11.2 Spring Cloud Gateway功能特征

- 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
- 动态路由
- Predicates 和 Filters 作用于特定路由
- 集成 Hystrix 断路器
- 集成 Spring Cloud DiscoveryClient
- 易于编写的 Predicates 和 Filters
- 限流
- 路径重写

### 2.11.3 Spring Cloud Gateway工程流程

![](http://mycsdnblog.work/201919212222-b.png)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（`pre`）或之后（`post`）执行业务逻辑

### 2.11.4 创建工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <groupId>com.springcloud.alibaba</groupId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-alibaba-gateway</artifactId>


    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!-- Spring Cloud End -->

        <!-- Commons Begin -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
        </dependency>
        <!-- Commons Begin -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.example.gateway.GatewayApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

主要增加了 `org.springframework.cloud:spring-cloud-starter-gateway` 依赖

**注：**

- Spring Cloud Gateway 不使用 **Web 作为服务器**，而是 **使用 WebFlux 作为服务器**，Gateway 项目已经依赖了 `starter-webflux`，所以这里 **千万不要依赖 starter-web**
- 由于过滤器等功能依然需要 Servlet 支持，故这里还需要依赖 `javax.servlet:javax.servlet-api`

### 2.11.5 启动类

```java
package com.example.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * @Author: 98050
 * @Time: 2019-08-21 22:30
 * @Feature:
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

### 2.11.6 配置文件

```yaml
spring:
  application:
    # 应用名称
    name: spring-gateway
  cloud:
    # 使用Nacos作为服务注册发现
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    # 使用Sentinel作为熔断器
    sentinel:
      transport:
        port: 8720
        dashboard: localhost:8080
    # 路由网关配置
    gateway:
      # 设置与服务注册发现组件结合，这样可以采用服务名的路由策略
      discovery:
        locator:
          enabled: true
      # 配置路由规则
      routes:
        # 采用自定义路由ID（有固定用法，不同的id有不同的功能）
        - id: NACOS-CONSUMER
          # 采用LoadBalanceClient方式请求，以lb://开头，后面的是注册在Nacos上的服务名
          uri: lb://nacos-consumer
          # Predicate的意思是谓词，必须，主要作用是匹配用户的请求，有很多种用法
          predicates:
            #Method 方法谓词，这里是匹配GET和POST请求
            - Method=GET,POST
        - id: NACOS-CONSUMER-FEIGN
          uri: lb://nacos-consumer-feign
          predicates:
            - Method=GET,POST
server:
  port: 9000

# 目前无效
feign:
  sentinel:
    enabled: true

# 目前无效
management:
  endpoints:
    web:
      exposure:
        include: "*"

# 配置日志级别，方别调试
logging:
  level:
    org.springframework.cloud.gateway: debug
```

### 2.11.7 测试访问

依次运行 Nacos 服务、`NacosProviderApplication`、`NacosConsumerApplication`、`NacosConsumerFeignApplication`、`GatewayApplication`

访问地址：http://localhost:9000/nacos-consumer-feign/getTest

![](http://mycsdnblog.work/201919212322-8.png)

访问地址：http://localhost:9000/nacos-consumer/getTest

![](http://mycsdnblog.work/201919212322-2.png)

## 2.12 使用路由网关的全局过滤功能

全局过滤器作用于所有的路由，不需要单独配置，我们可以用它来实现很多统一化处理的业务需求，比如权限认证，IP 访问限制等等。

### 2.12.1 声明周期

![](http://mycsdnblog.work/201919231711-d.png)

Spring Cloud Gateway 基于 Project Reactor 和 WebFlux，采用响应式编程风格，打开它的 Filter 的接口 GlobalFilter 你会发现它只有一个方法 filter。

GlobalFilter 全局过滤器

GatewayFilter 局部过滤器

### 2.12.2 创建全局过滤器

实现 `GlobalFilter`, `Ordered` 接口并在类上增加 `@Component` 注解就可以使用过滤功能了，非常简单方便。

```java
package com.example.gateway.filter;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.common.collect.Maps;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Map;

/**
 * @Author: 98050
 * @Time: 2019-08-23 17:16
 * @Feature:
 */
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        if (token == null || token.isEmpty()){
            ServerHttpResponse response = exchange.getResponse();
            //封装错误信息
            Map<String,Object> responseData = Maps.newHashMap();
            responseData.put("code", 401);
            responseData.put("message", "非法请求");
            responseData.put("cause", "Token is empty");
            try {
                ObjectMapper objectMapper = new ObjectMapper();
                byte[] data = objectMapper.writeValueAsBytes(responseData);
                DataBuffer buffer = response.bufferFactory().wrap(data);
                response.setStatusCode(HttpStatus.UNAUTHORIZED);
                response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
                return response.writeWith(Mono.just(buffer));
            }catch (Exception e){
                e.printStackTrace();
            }
            return response.setComplete();
        }


        return chain.filter(exchange);
    }

    /**
     * 设置过滤器的顺序
     * @return
     */
    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```

### 2.12.3 测试

访问：http://localhost:9000/nacos-consumer/getTest

![](http://mycsdnblog.work/201919231747-j.png)

### 2.12.4 Spring Cloud Gateway Benchmark

Spring 官方人员提供的网关基准测试报告 [GitHub](https://github.com/spencergibb/spring-cloud-gateway-bench)

| Proxy    | Avg Latency | Avg Req/Sec/Thread |
| -------- | ----------- | ------------------ |
| gateway  | 6.61ms      | 3.24k              |
| linkered | 7.62ms      | 2.82k              |
| zuul     | 12.56ms     | 2.09k              |
| none     | 2.09ms      | 11.77k             |

- 这里的 Zuul 为 1.x 版本，是一个基于阻塞 IO 的 API Gateway
- Zuul 已经发布了 Zuul 2.x，基于 Netty，非阻塞的，支持长连接，但 Spring Cloud 暂时还没有整合计划
- Linkerd 基于 Scala 实现的、目前市面上仅有的生产级别的 Service Mesh（其他诸如 **Istio、Conduit 暂时还不能用于生产**）。

# 三、Spring Cloud Alibaba 服务配置

## 3.1 Nacos Config 服务端初始化

### 3.1.1 分布式配置中心

在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。

### 3.1.2 Nacos Config

Nacos 提供用于存储配置和其他元数据的 key/value 存储，为分布式系统中的外部化配置提供服务器端和客户端支持。使用 Spring Cloud Alibaba Nacos Config，您可以在 Nacos Server 集中管理你 Spring Cloud 应用的外部属性配置。

Spring Cloud Alibaba Nacos Config 是 Spring Cloud Config Server 和 Client 的替代方案，客户端和服务器上的概念与 Spring Environment 和 PropertySource 有着一致的抽象，在特殊的 bootstrap 阶段，配置被加载到 Spring 环境中。当应用程序通过部署管道从开发到测试再到生产时，您可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容。

### 3.1.3 创建配置文件

需要在 Nacos Server 中创建配置文件，我们依然采用 YAML 的方式部署配置文件，操作流程如下：

![](http://mycsdnblog.work/201919231759-R.png)

![](http://mycsdnblog.work/201919231759-Y.png)

## 3.2 Nacos Config 客户端的使用

### 3.2.1 添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

### 3.2.2 创建bootstrap.yml

创建名为 `bootstrap.yml` 的配置文件并删除之前创建的 `application.yml` 配置文件，由于已经在服务端配置，此处不再赘述

```yaml
# 这里的应用名对应 Nacos Config 中的 Data ID，实际应用名称以配置中心的配置为准
spring:
  application:
    name: nacos-provider-config
  # 指定查找名为 nacos-provider-config.yaml 的配置文件
  cloud:
    nacos:
      config:
        file-extension: yaml
        server-addr: 127.0.0.1:8848
```

### 3.2.3 启动服务

![1566555914717](http://mycsdnblog.work/201919231825-B.png)

### 3.2.4 配置动态更新

Nacos Config 也支持配置的动态更新，操作流程如下：

- 修改服务端配置，增加一个 `user.name` 的属性

![1566556778593](http://mycsdnblog.work/201919231839-K.png)

添加一个读取配置文件的方法，测试动态更新效果

```java
package com.example.provider.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: 98050
 * @Time: 2019-08-21 17:52
 * @Feature:
 */
@RestController
public class StringController {

    @Value("${server.port}")
    private String port;

    @Autowired
    private ConfigurableApplicationContext configurableApplicationContext;

    @GetMapping("/test")
    public String test(){
        return "test,port:" + port;
    }


    @GetMapping("/getName")
    public String getName(){
        return "name:" + configurableApplicationContext.getEnvironment().getProperty("user.name");
    }
}
```

效果：

![](http://mycsdnblog.work/201919231855-L.png)

修改服务端配置：

![](http://mycsdnblog.work/201919231855-n.png)

再次访问：

![](http://mycsdnblog.work/201919231856-U.png)

![](http://mycsdnblog.work/201919231856-9.png)

**注意：你可以使用 spring.cloud.nacos.config.refresh.enabled=false 来关闭动态刷新**

## 3.3 Nacos Config多环境的配置

### 3.3.1 Spring Boot Profile

我们在做项目开发的时候，生产环境和测试环境的一些配置可能会不一样，有时候一些功能也可能会不一样，所以我们可能会在上线的时候手工修改这些配置信息。但是 Spring 中为我们提供了 Profile 这个功能。我们只需要在启动的时候添加一个虚拟机参数，激活自己环境所要用的 Profile 就可以了。

操作起来很简单，只需要为不同的环境编写专门的配置文件，如：`application-dev.yml`、`application-prod.yml`， 启动项目时只需要增加一个命令参数 `--spring.profiles.active=环境配置` 即可，启动命令如下：

```bash
java -jar hello-spring-cloud-alibaba-nacos-provider-1.0.0-SNAPSHOT.jar --spring.profiles.active=prod
```

### 3.3.2 Nacos Config Profile

spring-cloud-starter-alibaba-nacos-config 在加载配置的时候，不仅仅加载了以 dataid 为 `${spring.application.name}.${file-extension:properties}` 为前缀的基础配置，还加载了 dataid 为 `${spring.application.name}-${profile}.${file-extension:properties}` 的基础配置。在日常开发中如果遇到多套环境下的不同配置，可以通过 Spring 提供的 `${spring.profiles.active}` 这个配置项来配置。

**在Nacos Server中添加配置**

增加一个名为 `nacos-provider-config-prod.yaml` 的配置

![](http://mycsdnblog.work/201919231903-d.png)

**注意：此时，我将配置文件中的端口由 8081 -> 8082**

**在项目中增加配置**

增加一个名为 `bootstrap-prod.yml` 的配置文件，内容如下

```yaml
spring:
  profiles:
    active: prod
  application:
    name: nacos-provider-config
  cloud:
    nacos:
      config:
        file-extension: yaml
        server-addr: 127.0.0.1:8848
```

主要增加了 `spring.profiles.active=prod` 配置，用于指定访问 Nacos Server 中的 `nacos-provider-config-prod.yaml` 配置

### 3.3.3 启动应用程序

此时我们有两个配置文件，分别为 `bootstrap.properties` 和 `bootstrap-prod.properties` ，我们需要指定启动时加载哪一个配置文件，操作流程如下：

![1566558467634](http://mycsdnblog.work/201919231907-3.png)

观察日志，判断是否成功加载配置

![](http://mycsdnblog.work/201919231908-V.png)

![](http://mycsdnblog.work/201919231908-t.png)

