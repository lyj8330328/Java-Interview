# 一、Eureka：注册中心

## 1.1 Eureka基本架构

Eureka的作用：负责管理、记录服务提供者的信息。服务调用者无需自己寻找服务，而是把自己的需求告诉Eureka，然后Eureka会把符合你需求的服务告诉你。

同时，服务提供方与Eureka之间通过`“心跳”`机制进行监控，当某个服务提供方出现问题，Eureka自然会把它从服务列表中剔除。

这就实现了服务的自动注册、发现、状态监控。

![](http://mycsdnblog.work/201919191705-9.png)

- Eureka：就是服务注册中心（可以是一个集群），对外暴露自己的地址
- 提供者：启动后向Eureka注册自己信息（地址，提供什么服务）
- 消费者：向Eureka订阅服务，Eureka会将对应服务的所有提供者地址列表发送给消费者，并且定期更新
- 心跳(续约)：提供者定期通过**http方式向**Eureka刷新自己的状态

## 1.2 Eureka-Server接收注册

正常情况下会进入 `PeerAwareInstanceRegistryImpl#register(...)` 方法：

### 1.2.1 注册表的真正结构

```java
private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry = new ConcurrentHashMap();
```

它就是一个ConcurrentHashMap, 其Key为应用的AppID，Value为一个Map，其中的键值对为该应用的各个实例。（InstanceId为键， `Lease<InstanceInfo>>` 为值）

### 1.2.2 核心注册过程

- **Eureka Server：**注册中心，里面有一个注册表，保存了各个服务所在的机器和端口号

@EnableEurekaServer

- **Eureka** **Client：**负责将这个服务的信息注册到Eureka Server中

@EnableDiscoveryClient

# 二、Zuul：服务网关

# 三、Ribbon：负载均衡

# 四、Feign：服务调用

# 五、Hystix：熔断器