# 一、ES的分布式架构原理

ElasticSearch 设计的理念就是分布式搜索引擎，底层其实还是基于 lucene 的。核心思想就是在多台机器上启动多个 es 进程实例，组成了一个 es 集群。

ES中存储数据的基本单位是**索引**，比如说你现在要在 es 中存储一些订单数据，你就应该在 es 中创建一个索引 `order_idx`，所有的订单数据就都写到这个索引里面去，一个索引差不多就是相当于是 mysql 里的一张表。

文档（Document）----------------Row记录

字段（Field）-------------------Columns 列



搞一个索引，这个索引可以拆分成多个 `shard`，每个 shard 存储部分数据。拆分多个 shard 是有好处的，一是**支持横向扩展**，比如你数据量是 3T，3 个 shard，每个 shard 就 1T 的数据，若现在数据量增加到 4T，怎么扩展，很简单，重新建一个有 4 个 shard 的索引，将数据导进去；二是**提高性能**，数据分布在多个 shard，即多台服务器上，所有的操作，都会在多台机器上并行分布式执行，提高了吞吐量和性能。

接着就是这个 shard 的数据实际是有多个备份，就是说每个 shard 都有一个 `primary shard`，负责写入数据，但是还有几个 `replica shard`。`primary shard` 写入数据之后，会将数据同步到其他几个 `replica shard` 上去。

![](http://mycsdnblog.work/201919131621-W.png)

通过这个 replica 的方案，每个 shard 的数据都有多个备份，如果某个机器宕机了，没关系啊，还有别的数据副本在别的机器上呢。高可用了吧。

es 集群多个节点，**会自动选举一个节点为 master 节点**，这个 master 节点其实就是干一些管理的工作的，比如维护索引元数据、负责切换 primary shard 和 replica shard 身份等。要是 master 节点宕机了，那么会重新选举一个节点为 master 节点。

如果是非 master节点宕机了，那么会由 master 节点，让那个宕机节点上的 primary shard 的身份转移到其他机器上的 replica shard。接着你要是修复了那个宕机机器，重启了之后，master 节点会控制将缺失的 replica shard 分配过去，同步后续修改的数据之类的，让集群恢复正常。

说得更简单一点，就是说如果某个非 master 节点宕机了。那么此节点上的 primary shard 不就没了。那好，master 会让 primary shard 对应的 replica shard（在其他机器上）切换为 primary shard。如果宕机的机器修复了，修复后的节点也不再是 primary shard，而是 replica shard。


# 二、ES搜索原理

## 2.1 搜索过程

es 最强大的是做全文检索，就是比如你有三条数据：

```
java真好玩儿啊
java好难学啊
j2ee特别牛
```

你根据 `java` 关键词来搜索，将包含 `java`的 `document` 给搜索出来。es 就会给你返回：java真好玩儿啊，java好难学啊。

- 客户端发送请求到一个 `coordinate node`。
- 协调节点将搜索请求转发到**所有**的 shard 对应的 `primary shard` 或 `replica shard`，都可以。
- query phase：每个 shard 将自己的搜索结果（其实就是一些 `doc id`）返回给协调节点，由协调节点进行数据的合并、排序、分页等操作，产出最终结果。
- fetch phase：接着由协调节点根据 `doc id` 去各个节点上**拉取实际**的 `document` 数据，最终返回给客户端。

**写请求是写入 primary shard，然后同步给所有的 replica shard；读请求可以从 primary shard 或 replica shard 读取，采用的是随机轮询算法。**

ES在搜索过程中存在以下几个问题：

1、数量问题。 

比如， 用户需要搜索"衣服"， 要求返回符合条件的前 10 条。 但在 5个分片中， 可能都存储着衣服相关的数据。 所以 ES 会向这 5 个分片都发出查询请求， 并且要求每个分片都返回符合条件的 10 条记录。当ES得到返回的结果后，进行整体排序，然后取最符合条件的前10条返给用户。 这种情况， ES 中 5 个 shard 最多会收到 10*5=50条记录， 这样返回给用户的结果数量会多于用户请求的数量。
2、排名问题。 

上面说的搜索， **每个分片计算符合条件的前 10 条数据都是基于自己分片的数据进行打分计算的**。计算分值使用的词频和文档频率等信息都是基于自己分片的数据进行的， **而 ES 进行整体排名是基于每个分片计算后的分值进行排序的**(相当于打分依据就不一样， 最终对这些数据统一排名的时候就不准确了)， 这就可能会导致排名不准确的问题。**如果我们想更精确的控制排序**， 应该先将计算排序和排名相关的信息（ 词频和文档频率等打分依据） 从 5 个分片收集上来， 进行统一计算， 然后使用整体的词频和文档频率为每个分片中的数据进行打分， 这样打分依据就一样了。

## 2.2 ES的搜索类型（SearchType类型）

### 2.2.1 query and fetch

向索引的所有分片 （ shard）都发出查询请求， 各分片返回的时候把元素文档 （ document）和计算后的排名信息一起返回。
这种搜索方式是最快的。 因为相比下面的几种搜索方式， 这种查询方法只需要去 shard查询一次。 但是各个 shard 返回的结果的数量之和可能是用户要求的 size 的 n 倍。

优点：这种搜索方式是最快的。因为相比后面的几种es的搜索方式，这种查询方法只需要去shard查询一次。

缺点：返回的数据量不准确， 可能返回(N*分片数量)的数据并且数据排名也不准确，同时各个shard返回的结果的数量之和可能是用户要求的size的n倍。

### 2.2.2 query then fetch

**ES默认搜索方式**

如果你搜索时， 没有指定搜索方式， 就是使用的这种搜索方式。 这种搜索方式， 大概分两个步骤：

第一步， 先向所有的 shard 发出请求， 各分片只返回文档 id(注意， 不包括文档 document)和排名相关的信息(也就是文档对应的分值)， 然后按照各分片返回的文档的分数进行重新排序和排名， 取前 size 个文档。

第二步， 根据文档 id 去相关的 shard 取 document。 这种方式返回的 document 数量与用户要求的大小是相等的。

优点：
　　返回的数据量是准确的。

缺点：
　　性能一般，并且数据排名不准确。

### 2.2.3 DFS query and fetch

这种方式比第一种方式多了一个 DFS 步骤，有这一步，可以更精确控制搜索打分和排名。也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作。
　　优点：
　　　　数据排名准确
　　缺点：
　　　　性能一般
　　　　返回的数据量不准确， 可能返回(N*分片数量)的数据

### 2.2.4 DFS query then fetch

比第 2 种方式多了一个 DFS 步骤。
　　也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作、

　　优点：
　　　　返回的数据量是准确的
　　　　数据排名准确
　　缺点：
　　　　性能最差【 这个最差只是表示在这四种查询方式中性能最慢， 也不至于不能忍受，如果对查询性能要求不是非常高， 而对查询准确度要求比较高的时候可以考虑这个】

> **DFS是什么样的过程？**

从 es 的官方网站我们可以发现， DFS 其实就是在进行真正的查询之前， 先把各个分片的词频率和文档频率收集一下， 然后进行词搜索的时候， 各分片依据全局的词频率和文档频率进行搜索和排名。 显然如果使用 DFS_QUERY_THEN_FETCH 这种查询方式， 效率是最低的，因为一个搜索， 可能要请求 3 次分片。 但是使用 DFS 方法， 搜索精度是最高的。

总结一下， 从性能考虑 QUERY_AND_FETCH 是最快的， DFS_QUERY_THEN_FETCH 是最慢的。从搜索的准确度来说， DFS 要比非 DFS 的准确度更高。

## 2.3 _score

使用ES时，对于查询出的文档无疑会有文档相似度之别。而理想的排序是和查询条件相关性越高排序越靠前，而这个排序的依据就是`_score`。

一个文档对于搜索的评分一定是有据可依的，而接下来就要介绍根据哪些参数查找匹配的文档以及评分的标准。

Lucene（或 Elasticsearch）使用 [*布尔模型（Boolean model）*](http://en.wikipedia.org/wiki/Standard_Boolean_model) 查找匹配文档， 并用一个名为 [*实用评分函数（practical scoring function）*](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/practical-scoring-function.html) 的公式来计算相关度。这个公式借鉴了 [*词频/逆向文档频率（term frequency/inverse document frequency）*](http://en.wikipedia.org/wiki/Tfidf) 和 [*向量空间模型（vector space model）*](http://en.wikipedia.org/wiki/Vector_space_model)，同时也加入了一些现代的新特性，如协调因子（coordination factor），字段长度归一化（field length normalization），以及词或查询语句权重提升。

### 2.3.1 布尔模型

*布尔模型（Boolean Model）* 只是在查询中使用 `AND` 、 `OR` 和 `NOT` （与、或和非）这样的条件来查找匹配的文档，以下查询：

```
full AND text AND search AND (elasticsearch OR lucene)
```

会将所有包括词 `full` 、 `text` 和 `search` ，以及 `elasticsearch` 或 `lucene` 的文档作为结果集。

这个过程简单且快速，它将所有可能不匹配的文档排除在外。

### 4.3.2 词频/逆向文档频率（TF/IDF）

当匹配到一组文档后，需要根据相关度排序这些文档，不是所有的文档都包含所有词，有些词比其他的词更重要。一个文档的相关度评分部分取决于每个查询词在文档中的 *权重* 。

词的权重由三个因素决定词频、逆向文档频率、字段长度归一值

**词频**

词在文档中出现的频度是多少？ 频度越高，权重 *越高* 。 5 次提到同一词的字段比只提到 1 次的更相关。词频的计算方式如下：

```
tf(t in d) = √frequency 
```

词 `t` 在文档 `d` 的词频（ `tf` ）是该词在文档中出现次数的平方根。

**逆向文档频率**

词在集合所有文档里出现的频率是多少？频次越高，权重 *越低* 。 常用词如 `and` 或 `the` 对相关度贡献很少，因为它们在多数文档中都会出现，一些不常见词如 `elastic` 或 `hippopotamus` 可以帮助我们快速缩小范围找到感兴趣的文档。逆向文档频率的计算公式如下：

```
idf(t) = 1 + log ( numDocs / (docFreq + 1))
```

词 `t` 的逆向文档频率（ `idf` ）是：索引中文档数量除以所有包含该词的文档数，然后求其对数。

**字段长度归一值**

字段的长度是多少？ 字段越短，字段的权重 *越高* 。如果词出现在类似标题 `title` 这样的字段，要比它出现在内容 `body` 这样的字段中的相关度更高。字段长度的归一值公式如下：

```
norm(d) = 1 / √numTerms
```

字段长度归一值（ `norm` ）是字段中词数平方根的倒数。

以下三个因素——词频（term frequency）、逆向文档频率（inverse document frequency）和字段长度归一值（field-length norm）——是在索引时计算并存储的。 最后将它们结合在一起计算单个词在特定文档中的 *权重* 。

### 2.3.3 向量空间模型

*向量空间模型（vector space model）* 提供一种比较多词查询的方式，单个评分代表文档与查询的匹配程度，为了做到这点，这个模型将文档和查询都以 *向量（vectors）* 的形式表示：

向量实际上就是包含多个数的一维数组，例如：

```
[1,2,5,22,3,8]
```

在向量空间模型里， 向量空间模型里的每个数字都代表一个词的 *权重* ，与 [词频/逆向文档频率（term frequency/inverse document frequency）](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/scoring-theory.html#tfidf) 计算方式类似。

### 2.3.4 评分函数

对于多词查询， Lucene 使用 [布尔模型（Boolean model）](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/scoring-theory.html#boolean-model) 、 [TF/IDF](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/scoring-theory.html#tfidf) 以及 [向量空间模型（vector space model）](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/scoring-theory.html#vector-space-model) ，然后将它们组合到单个高效的包里以收集匹配文档并进行评分计算。

只要一个文档与查询匹配，Lucene 就会为查询计算评分，然后合并每个匹配词的评分结果。这里使用的评分计算公式叫做 *实用评分函数（practical scoring function）* 。看似很高大上，但是别被吓到——多数的组件都已经介绍过，下一步会讨论它引入的一些新元素。

```
score(q,d)  =  
            queryNorm(q)  
          · coord(q,d)    
          · ∑ (           
                tf(t in d)   
              · idf(t)²      
              · t.getBoost() 
              · norm(t,d)    
            ) (t in q)   
```

![](http://mycsdnblog.work/201919031850-7.png)

### 2.3.5 查询归一因子

*查询归一因子* （ `queryNorm` ）试图将查询 *归一化* ， 这样就能将两个不同的查询结果相比较。

尽管查询归一值的目的是为了使查询结果之间能够相互比较，但是它并不十分有效，因为相关度评分 `_score` 的目的是为了将当前查询的结果进行排序，比较不同查询结果的相关度评分没有太大意义。

这个因子是在查询过程的最前面计算的，具体的计算依赖于具体查询，一个典型的实现如下：

```
queryNorm = 1 / √sumOfSquaredWeights
```

`sumOfSquaredWeights` 是查询里每个词的 IDF 的平方和。

### 2.3.6 查询协调

*协调因子* （ `coord` ） 可以为那些查询词包含度高的文档提供奖励，文档里出现的查询词越多，它越有机会成为好的匹配结果。

设想查询 `quick brown fox` ，每个词的权重都是 1.5 。如果没有协调因子，最终评分会是文档里所有词权重的总和。例如：

- 文档里有 `fox` → 评分： 1.5
- 文档里有 `quick fox` → 评分： 3.0
- 文档里有 `quick brown fox` → 评分： 4.5

协调因子将评分与文档里匹配词的数量相乘，然后除以查询里所有词的数量，如果使用协调因子，评分会变成：

- 文档里有 `fox` → 评分： `1.5 * 1 / 3` = 0.5
- 文档里有 `quick fox` → 评分： `3.0 * 2 / 3` = 2.0
- 文档里有 `quick brown fox` → 评分： `4.5 * 3 / 3` = 4.5

协调因子能使包含所有三个词的文档比只包含两个词的文档评分要高出很多。

# 三、写数据底层原理

- 客户端选择一个 node 发送请求过去，这个 node 就是 `coordinating node`（协调节点）。
- `coordinating node` 对 document 进行**路由**，将请求转发给对应的 node（有 primary shard）。
- 实际的 node 上的 `primary shard` 处理请求，然后将数据同步到 `replica node`。
- `coordinating node` 如果发现 `primary node` 和所有 `replica node` 都搞定之后，就返回响应结果给客户端。

![](http://mycsdnblog.work/201919131624-9.png)

![](http://mycsdnblog.work/201919101500-E.png)

先写入内存 buffer，在 buffer 里的时候数据是搜索不到的；同时将数据写入 translog 日志文件。

如果 buffer 快满了，或者到一定时间，就会将内存 buffer 数据 `refresh` 到一个新的 `segment file` 中，但是此时数据不是直接进入 `segment file` 磁盘文件，而是先进入 `os cache` 。这个过程就是 `refresh`。

每隔 1 秒钟，es 将 buffer 中的数据写入一个**新的** `segment file`，每秒钟会产生一个**新的磁盘文件** `segment file`，这个 `segment file` 中就存储最近 1 秒内 buffer 中写入的数据。

但是如果 buffer 里面此时没有数据，那当然不会执行 refresh 操作，如果 buffer 里面有数据，默认 1 秒钟执行一次 refresh 操作，刷入一个新的 segment file 中。

操作系统里面，磁盘文件其实都有一个东西，叫做 `os cache`，即操作系统缓存，就是说数据写入磁盘文件之前，会先进入 `os cache`，先进入操作系统级别的一个内存缓存中去。**只要 `buffer` 中的数据被 refresh 操作刷入 `os cache`中，这个数据就可以被搜索到了。**

为什么叫 es 是**准实时**的？ `NRT`，全称 `near real-time`。默认是每隔 1 秒 refresh 一次的，所以 es 是准实时的，因为写入的数据 1 秒之后才能被看到。可以通过 es 的 `restful api` 或者 `java api`，**手动**执行一次 refresh 操作，就是手动将 buffer 中的数据刷入 `os cache`中，让数据立马就可以被搜索到。只要数据被输入 `os cache` 中，buffer 就会被清空了，因为不需要保留 buffer 了，数据在 translog 里面已经持久化到磁盘去一份了。

重复上面的步骤，新的数据不断进入 buffer 和 translog，不断将 `buffer` 数据写入一个又一个新的 `segment file` 中去，每次 `refresh` 完 buffer 清空，translog 保留。随着这个过程推进，translog 会变得越来越大。当 translog 达到一定长度的时候，就会触发 `commit` 操作。

commit 操作发生第一步，就是将 buffer 中现有数据 `refresh` 到 `os cache` 中去，清空 buffer。然后，将一个 `commit point` 写入磁盘文件，里面标识着这个 `commit point` 对应的所有 `segment file`，同时强行将 `os cache` 中目前所有的数据都 `fsync` 到磁盘文件中去。最后**清空** 现有 translog 日志文件，重启一个 translog，此时 commit 操作完成。

这个 commit 操作叫做 `flush`。默认 30 分钟自动执行一次 `flush`，但如果 translog 过大，也会触发 `flush`。flush 操作就对应着 commit 的全过程，我们可以通过 es api，手动执行 flush 操作，手动将 os cache 中的数据 fsync 强刷到磁盘上去。

translog 日志文件的作用是什么？你执行 commit 操作之前，数据要么是停留在 buffer 中，要么是停留在 os cache 中，无论是 buffer 还是 os cache 都是内存，一旦这台机器死了，内存中的数据就全丢了。所以需要将**数据对应的操作**写入一个专门的日志文件 `translog` 中，一旦此时机器宕机，再次重启的时候，es 会自动读取 translog 日志文件中的数据，恢复到内存 buffer 和 os cache 中去。

**translog 其实也是先写入 os cache 的，默认每隔 5 秒刷一次到磁盘中去**，所以默认情况下，可能有 5 秒的数据会仅仅停留在 buffer 或者 translog 文件的 os cache 中，如果此时机器挂了，会**丢失** 5 秒钟的数据。但是这样性能比较好，最多丢 5 秒的数据。也可以将 translog 设置成每次写操作必须是直接 `fsync` 到磁盘，但是性能会差很多。

实际上你在这里，如果面试官没有问你 es 丢数据的问题，你可以在这里给面试官炫一把，你说，其实 es 第一是准实时的，数据写入 1 秒后可以搜索到；可能会丢失数据的。有 5 秒的数据，停留在 buffer、translog os cache、segment file os cache 中，而不在磁盘上，此时如果宕机，会导致 5 秒的**数据丢失**。

**总结一下**：**数据先写入内存 buffer，然后每隔 1s，将数据 refresh 到 os cache，到了 os cache 数据就能被搜索到（所以我们才说 es 从写入到能被搜索到，中间有 1s 的延迟）。每隔 5s，将数据写入 translog 文件（这样如果机器宕机，内存数据全没，最多会有 5s 的数据丢失），translog 大到一定程度，或者默认每隔 30mins，会触发 commit 操作，将缓冲区的数据都 flush 到 segment file 磁盘文件中。**

**数据写入 segment file 之后，同时就建立好了倒排索引。**

# 四、删除/更新数据底层原理

如果是删除操作，commit 的时候会生成一个 `.del` 文件，里面将某个 doc 标识为 `deleted` 状态，那么搜索的时候根据 `.del` 文件就知道这个 doc 是否被删除了。

**如果是更新操作，就是将原来的 doc 标识为 `deleted` 状态，然后新写入一条数据。**

buffer 每 refresh 一次，就会产生一个 `segment file`，所以默认情况下是 1 秒钟一个 `segment file`，这样下来 `segment file` 会越来越多，此时会定期执行 merge。每次 merge 的时候，会将多个 `segment file` 合并成一个，同时这里会将标识为 `deleted` 的 doc 给**物理删除掉**，然后将新的 `segment file` 写入磁盘，这里会写一个 `commit point`，标识所有新的 `segment file`，然后打开 `segment file` 供搜索使用，同时删除旧的 `segment file`。

# 五、倒排索引

## 5.1 基本概念

Elasticsearch最关键的就是提供强大的索引能力了，为了提高搜索的性能，难免会牺牲某些其他方面，比如插入/更新，否则其他数据库不用混了。往Elasticsearch里插入一条记录，其实就是直接PUT一个json的对象，这个对象有多个fields，比如name, sex, age, about, interests，那么在插入这些数据到Elasticsearch的同时，Elasticsearch还默默的为这些字段建立索引--倒排索引，因为Elasticsearch最核心功能是搜索。

Elasticsearch使用的倒排索引比关系型数据库的B+Tree索引（为了提高查询的效率，减少磁盘寻道次数，将多个值作为一个数组通过连续区间存放，一次寻道读取多个数据，同时也降低树的高度。）快。

![](http://mycsdnblog.work/201919071943-b.png)

例子：

![](http://mycsdnblog.work/201919071943-7.png)

Elasticsearch建立的索引如下:

![](http://mycsdnblog.work/201919071944-t.png)

> **Posting List**

Elasticsearch分别为每个field都建立了一个倒排索引，Kate, John, 24, Female这些叫term，而[1,2]就是**Posting List**。Posting list就是一个int的数组，存储了所有符合某个term的文档id。

看到这里，不要认为就结束了，精彩的部分才刚开始...

通过posting list这种索引方式似乎可以很快进行查找，比如要找age=24的同学，爱回答问题的小明马上就举手回答：我知道，id是1，2的同学。但是，如果这里有上千万的记录呢？如果是想通过name来查找呢？

> **Term Dictionary**

Elasticsearch为了能快速找到某个term，将所有的term排个序，二分法查找term，logN的查找效率，就像通过字典查找一样，这就是**Term Dictionary**。现在再看起来，似乎和传统数据库通过B-Tree的方式类似啊，为什么说比B+Tree的查询快呢？

> ##### **Term Index**
>

**B+Tree通过减少磁盘寻道次数来提高查询性能，Elasticsearch也是采用同样的思路**，直接通过内存查找term，不读磁盘，但是如果term太多，term dictionary也会很大，放内存不现实，于是有了**Term Index**，就像字典里的索引页一样，A开头的有哪些term，分别在哪页，可以理解term index是一颗树：

![](http://mycsdnblog.work/201919071946-N.png)

这棵树不会包含所有的term，它包含的是term的一些前缀。通过term index可以快速地定位到term dictionary的某个offset，然后从这个位置再往后顺序查找。

![](http://mycsdnblog.work/201919071948-n.png)

所以term index不需要存下所有的term，而仅仅是他们的一些前缀与Term Dictionary的block之间的映射关系，再结合FST(Finite State Transducers)的压缩技术，可以使term index缓存到内存中。从term index查到对应的term dictionary的block位置之后，再去磁盘上找term，大大减少了磁盘随机读的次数。

## 5.2 前缀树

Trie树，即字典树，又称单词查找树或键树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计和排序大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：最大限度地减少无谓的字符串比较，查询效率比哈希表高。

Trie的核心思想是空间换时间。利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。 
它有3个基本性质：

1. 根节点不包含字符，除根节点外每一个节点都只包含一个字符。
2. 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串。
3. 每个节点的所有子节点包含的字符都不相同。

trie树的实现比较简单。它使在字符串集合中查找某个字符串的操作的复杂度降到最大只需O(n),其中n为字符串的长度。trie是典型的将时间置换为空间的算法，好在ACM中一般对空间的要求很宽松。trie的原理是利用字符串集合中字符串的公共前缀来降低时间开销以达到提高效率的目的。 它具有以下性质:

- 根结点不包含任何字符信息;
- 如果字符的种数为n,则每个结点的出度为n(这样必然会导致浪费很多空间,这也是trie的缺点,还没有想到好点的办法避免);
- 查找，插入复杂度为O(n)，n为字符串长度。

### 5.2.1 前缀树的构建

举个在网上流传颇广的例子，如下：

题目：给你100000个长度不超过10的单词。对于每一个单词，我们要判断他出没出现过，如果出现了，求第一次出现在第几个位置。 

分析：这题当然可以用hash来解决，但是本文重点介绍的是trie树，因为在某些方面它的用途更大。比如说对于某一个单词，我们要询问它的前缀是否出现过。这样hash就不好搞了，而用trie还是很简单。 

现在回到例子中，如果我们用最傻的方法，对于每一个单词，我们都要去查找它前面的单词中是否有它。那么这个算法的复杂度就是O(n^2)。显然对于100000的范围难以接受。现在我们换个思路想。假设我要查询的单词是abcd，那么在他前面的单词中，以b，c，d，f之类开头的我显然不必考虑。而只要找以a开头的中是否存在abcd就可以了。同样的，在以a开头中的单词中，我们只要考虑以b作为第二个字母的，一次次缩小范围和提高针对性，这样一个树的模型就渐渐清晰了。 
    好比假设有b，abc，abd，bcd，abcd，efg，hii 这6个单词，我们构建的树就是如下图这样的：

![](http://mycsdnblog.work/201919032015-5.png)

### 5.2.2 前缀树的查询

Trie树是简单但实用的数据结构，通常用于实现字典查询。我们做即时响应用户输入的AJAX搜索框时，就是Trie开始。本质上，Trie是一颗存储多个字符串的树。相邻节点间的边代表一个字符，这样树的每条分支代表一则子串，而树的叶节点则代表完整的字符串。和普通树不同的地方是，相同的字符串前缀共享同一条分支。下面，再举一个例子。给出一组单词，inn, int, at, age, adv, ant, 我们可以得到下面的Trie：

![](http://mycsdnblog.work/201919032026-0.png)

1. 可以看出：
   - 每条边对应一个字母。
   - 每个节点对应一项前缀。叶节点对应最长前缀，即单词本身。
   - 单词inn与单词int有共同的前缀“in”, 因此他们共享左边的一条分支，root->i->in。同理，ate, age, adv, 和ant共享前缀"a"，所以他们共享从根节点到节点"a"的边。

  查询操纵非常简单。比如要查找int，顺着路径i -> in -> int就找到了。 

1. 考察前缀"a"，发现边a已经存在。于是顺着边a走到节点a。
2. 考察剩下的字符串"dd"的前缀"d"，发现从节点a出发，已经有边d存在。于是顺着边d走到节点ad
3. 考察最后一个字符"d"，这下从节点ad出发没有边d了，于是创建节点ad的子节点add，并把边ad->add标记为d。

## 5.3 前缀树的应用

 除了本文引言处所述的问题能应用Trie树解决之外，Trie树还能解决下述问题（节选自此文：[海量数据处理面试题集锦与Bit-map详解](http://blog.csdn.net/v_july_v/article/details/6685962)）：

- 3、有一个1G大小的一个文件，里面每一行是一个词，词的大小不超过16字节，内存限制大小是1M。返回频数最高的100个词。
- 9、1000万字符串，其中有些是重复的，需要把重复的全部去掉，保留没有重复的字符串。请怎么设计和实现？
- 10、 一个文本文件，大约有一万行，每行一个词，要求统计出其中最频繁出现的前10个词，请给出思想，给出时间复杂度分析。
- 13、寻找热门查询：搜索引擎会通过日志文件把用户每次检索使用的所有检索串都记录下来，每个查询串的长度为1-255字节。假设目前有一千万个记录，这些查询串的重复读比较高，虽然总数是1千万，但是如果去除重复和，不超过3百万个。一个查询串的重复度越高，说明查询它的用户越多，也就越热门。请你统计最热门的10个查询串，要求使用的内存不能超过1G。 
  (1) 请描述你解决这个问题的思路； 
  (2) 请给出主要的处理流程，算法，以及算法的复杂度。



# 七、ES配置

```yaml
cluster.name: elasticsearch
配置es的集群名称，默认是elasticsearch，es会自动发现在同一网段下的es，如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群。
node.name: "Franz Kafka"
节点名，默认随机指定一个name列表中名字，该列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字。
node.master: true
指定该节点是否有资格被选举成为node，默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。
node.data: true
指定该节点是否存储索引数据，默认为true。
index.number_of_shards: 5
设置默认索引分片个数，默认为5片。
index.number_of_replicas: 1
设置默认索引副本个数，默认为1个副本。
path.conf: /path/to/conf
设置配置文件的存储路径，默认是es根目录下的config文件夹。
path.data: /path/to/data
设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开，例：
path.data: /path/to/data1,/path/to/data2
path.work: /path/to/work
设置临时文件的存储路径，默认是es根目录下的work文件夹。
path.logs: /path/to/logs
设置日志文件的存储路径，默认是es根目录下的logs文件夹
path.plugins: /path/to/plugins
设置插件的存放路径，默认是es根目录下的plugins文件夹
bootstrap.mlockall: true
设置为true来锁住内存。因为当jvm开始swapping时es的效率会降低，所以要保证它不swap，可以把ES_MIN_MEM和 ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。同时也要允许elasticsearch的进程可以锁住内存，linux下可以通过`ulimit -l unlimited`命令。
network.bind_host: 192.168.0.1
设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0。 
network.publish_host: 192.168.0.1
设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。
network.host: 192.168.0.1
这个参数是用来同时设置bind_host和publish_host上面两个参数。
transport.tcp.port: 9300
设置节点间交互的tcp端口，默认是9300。
transport.tcp.compress: true
设置是否压缩tcp传输时的数据，默认为false，不压缩。
http.port: 9200
设置对外服务的http端口，默认为9200。
http.max_content_length: 100mb
设置内容的最大容量，默认100mb
http.enabled: false
是否使用http协议对外提供服务，默认为true，开启。
gateway.type: local
gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，Hadoop的HDFS，和amazon的s3服务器。
gateway.recover_after_nodes: 1
设置集群中N个节点启动时进行数据恢复，默认为1。
gateway.recover_after_time: 5m
设置初始化数据恢复进程的超时时间，默认是5分钟。
gateway.expected_nodes: 2
设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。
cluster.routing.allocation.node_initial_primaries_recoveries: 4
初始化数据恢复时，并发恢复线程的个数，默认为4。
cluster.routing.allocation.node_concurrent_recoveries: 2
添加删除节点或负载均衡时并发恢复线程的个数，默认为4。
indices.recovery.max_size_per_sec: 0
设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。
indices.recovery.concurrent_streams: 5
设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。
discovery.zen.minimum_master_nodes: 1
设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）
discovery.zen.ping.timeout: 3s
设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。
discovery.zen.ping.multicast.enabled: false
设置是否打开多播发现节点，默认是true。
discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]
设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。
下面是一些查询时的慢日志参数设置
index.search.slowlog.level: TRACE
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 5s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms
index.search.slowlog.threshold.fetch.warn: 1s
index.search.slowlog.threshold.fetch.info: 800ms
index.search.slowlog.threshold.fetch.debug:500ms
index.search.slowlog.threshold.fetch.trace: 200ms
```

