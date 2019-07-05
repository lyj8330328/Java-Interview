# 一、Mybatis延迟加载

指的是我们真正使用数据时，才去数据库查询，而不是一上来就去把数据显示出来。

```yaml
mybatis.configuration.lazy-loading-enabled=true
#false 为按需加载
mybatis.configuration.aggressive-lazy-loading=false
```

这样就实现了全局懒加载，若个别需要关闭，可用 fetchType=“eager”，例如下图

![](http://mycsdnblog.work/201919042119-H.png)

# 二、Mybatis缓存

一级缓存：Mybatis的一级缓存在session上，只要通过session查过的数据，都会放在session上，下一次再查询相同id的数据，都直接冲缓存中取出来，而不用到数据库里去取了。

> sqlSession级缓存 ，作用域为单个sqlSession，存储格式HashMap，查询先走一级缓存，不存在则走数据库，执行更新操作清理一级缓存避免读脏，一般编程结构事务控制在service层中，**事务开始的时候创建sqlSession，在中间处理crud，结束后关闭sqlSession**，这种情况一级缓存作用其实不大，较少的业务会在一个service里面重复读相同数据。

二级缓存：Mybatis二级缓存是SessionFactory，如果两次查询基于同一个SessionFactory，那么就从二级缓存中取数据，而不用到数据库里去取了。

**在需要开启二级缓存的namespace中加入**：

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

**如果需要开启二级缓存，resultMap的对应的实体类必须要序列化不然会报java.io.NotSerializableException**



mapper（namespace）级缓存，作用域为mapper，默认存储格式HashMap，可以扩展为其他分布式缓存如ehcache，相对一级缓存而言，作用范围更大，作用于多个sqlsession ,a 事务查询userA，写入二级缓存，b 事务查询userA将直接走二级缓存，c 事务更新user将清理二级缓存，这里清理是指清理整个（namespace下）二级缓存，并非细粒度的清理userA，二级缓存机制在一些实时性要求不高，切不想依赖一些外部（三级）缓存的系统可以通过设置定时清理缓存达到提高减少DB压力的目的，在我们系统具体mapper并没开启二级缓存，这块实用性个人感觉不高，正常业务需要细粒度的管理缓存，会采用redis等组件实现，更重要的一点当然是写好SQL，在关系数据库系统里面，bad sql啥缓存都没用

