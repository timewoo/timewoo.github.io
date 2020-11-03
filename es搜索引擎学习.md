## elasticsearch（es）
记录学习elasticsearch的笔记，基本资料来源于https://github.com/doocs/advanced-java ，推荐阅读

### elasticsearch架构
设计理念是分布式搜索引擎，底层基于lucene，分布式架构，es中存储的基本单位是索引，所有类型相同的数据都会写到这个索引中，一个索引相当于mysql里的一张表。
一个index下默认一个type，type指的是映射类型，指的是具有相同类型的数据存储，但是可能存在一个index下存在多个type的情况(在elasticsearch7.X中type完全移除，
移除原因是由于同一个index下多个type中名称相同的字段是由同一个lucene来支持，意味着同一个字段的类型必须相同，在此之上需要考虑一点，
如果同一个索引中存储的各个实体如果只有很少或者根本没有同样的字段，这种情况会导致稀疏数据，并且会影响到Lucene的高效压缩数据的能力),一个document就代表一条数据，
field就代表document中的字段。同时支持分布式存储，索引可以拆分成多个shard，每个shard存储部分数据，支持横向扩展和提升性能。具备高可用，shard有数据备份，
写入primary shard，replica shard同步数据。多个shard存储不同数据是分布式，每个shard有primary shard和replica shard备份是高可用。es集群拥有多个节点，
master节点处理管理事件，比如维护索引元数据，切换primary shard和replica shard的身份等。master节点宕机后会重新选举一个master节点，如果非master节点宕机，
如果该节点上有primary shard，master节点会将该节点上的primary shard对应的replica shard提升为primary shard，让集群能够正常工作，在服务器恢复之后，
该服务器上原来的primary shard会变为replica shard，同步宕机过程缺失的数据。具有高可用和分布式的特性。

### elasticsearch数据的处理原理

### es写入数据
客户端选择一个node发送请求，该node就是coordinating node（协调节点），该节点对document进行路由，将该节点转发给对应的node，该node上的primary shard对数据进行处理，
同时同步到replica shard节点，coordinating node如果发现primary shard和所有的replica shard都同步完成，就会返回响应结果给客户端。

### es读取数据
主要是根据数据的doc id来查询，客户端发送请求到任意node(coordinating node),coordinating node将doc id进行哈希路由，把请求转发到对应的node，
此时会使用round-robin随机轮询算法，会在primary shard和replica shard中随机选择一个，让读请求负载均衡，接受请求的node返回document给coordinate node，
coordinate node在将document给客户端。

### es检索数据
客户端将请求发送到coordinating node，node将搜索结果转发到所有的shard，将所有结果（doc id）返回给node，node节点将数据进行合并、排序、分页等操作，
然后node根据doc id去各个节点上拉取实际的document数据，最终返回给客户端。

### 写入数据原理
先将数据写入内存buffer(这时候是搜索不到数据的)，同时将数据写入到translog日志文件。如果这时候buffer快满了或者到了一定时间，就会将buffer的数据refresh到
一个新的segment file(磁盘文件)中，但是此时数据并不是直接写入segment file，而是写入到os cache(操作系统缓存)，这个时候的segment file的数据都是在os cache中，
并没有写入磁盘中。每隔1秒钟，es将buffer中的数据写入到新的segment file，即每个segment file存储的就是1s内写入的数据。在写入os cache时数据就可以被搜索到，
es被称为near real-time(NRT,准实时)，是指数据在写入之后1秒才能被搜索到，同时可以通过restful api和Java api手动refresh数据，
数据就可以实时搜索到。只要数据被refresh到os cache中，buffer就会被清空，因为数据已经被持久化到translog中，在这个过程中translog文件会越来越大，
当到达一定大小之后就会触发commit操作，commit将buffer中的数据refresh到os cache中，os cache中存储的都是segment file文件，同时清空buffer，然后将一个commit point写入磁盘文件，
里面存储这个commit point对应的所有os cache中的segment file的标志，后面可以根据这个commit points来获取这段时间变更的segment file，同时强行将os cache中的segment file都fsync(同步到)到磁盘文件，
最后清空translog日志文件，重启一个translog，commit操作结束。这个操作就是flush。es默认30分钟自动执行一次flush，如果translog文件过大(默认512M)也会触发flush，同时也可以通过es api手动执行flush。
如果没有translog日志文件，由于在执行commit操作之前，数据都是存放在buffer或os cache中，即使生成了segment file也是存储在os cache中，那么在服务器宕机之后，在内存中的数据就会全部丢失，
所以就需要有对应的translog日志文件来将数据恢复到buffer和os cache中，保证数据不丢失。其实translog也是先写入os cache中，默认5秒刷新到磁盘文件中，所以会有可能丢失5秒的数据，也可以将translog设置成同步fsync，
但是这样性能会差很多。所以一般为了性能，丢失5秒数据是可以接受的。

### 删除更新操作原理
删除操作在commit时会生成一个.del文件，里面会将某个doc标识为deleted状态，搜索时根据.del文件就会知道doc是否删除。如果是更新操作，就是将原来的doc标识为deleted状态，
然后写入一条新的数据。由于buffer每隔1秒会刷新到segment file，这样文件会越来越多，此时就会定期执行merge，将多个segment file合并成一个，同时会将标识为deleted的doc给物理删除，
然后将新的 segment file 写入磁盘，这里会写一个 commit point，标识所有新的 segment file，然后打开 segment file 供搜索使用，同时删除旧的 segment file。

### 底层的lucene倒排索引
lucene简单来说就是一个封装好的包含各种建立倒排索引的算法代码，倒排索引是指在对文档进行分词后提取关键词，对关键词和文档ID的映射即为倒排索引。用户可以输入关键词，
通过倒排索引快速找到对应的文档ID,从而找到文档。倒排索引中的所有词项对应一个或多个文档；倒排索引中的词项根据字典顺序升序排列。

### es查询效率
es搜索效率依赖于底层的filesystem cache，因为es在写入数据时，数据都存储在磁盘文件中，查询时，如果filesystem cache中没有，系统会把数据自动缓存到filesystem cache中，
如果filesystem cache足够多，能够容纳es的索引数据文件，那搜索时就会走内存，性能会非常高。所以为了提高es的搜索效率，最好让机器的内存能够容纳数据的一半，
这要一半的数据都会在内存中搜索。而且最好不要在es中存储所有的数据，这样会挤占内存的空间，最好只在es中存储索引，即数据的标识，然后将完整数据存储在mysql或hbase，
推荐使用hbase，通过es搜索到的doc id，在将完整的数据查出来，这样就提升了es的搜索效率。如果数据量确实足够大，filesystem cache内存不够，还可以采用数据预热的方式，
即后台运行系统，定时将搜索多的数据搜索一遍，将数据刷新到filesystem cache中，这样后面的搜索效率也会提升。这样需要自己运行一个专门的预热系统。还可以对es的索引进行冷热分离处理，
即将访问频繁的数据建立索引，访问少的建立一个索引，然后把索引分别放在不同的机器上，这样热数据的索引就会留在filesystem cache中，不会出现冷数据把热数据冲刷掉。
最好不要在es中用复杂的关联查询条件，因为es主要是用来全文检索的，不会很好支持复杂的语句，尽量将关联好的数据写入es。es本身的分页性能不够好，因为es的数据存在在不同的shard中，
会将所有shard的数据查出来，然后数据合并，处理，最后再获取最终的分页结果。举个例子吧，假如你每页是 10 条数据，你现在要查询第 100 页，实际上是会把每个 shard 上存储的前 1000 条
数据都查到一个协调节点上，如果你有个 5 个 shard，那么就有 5000 条数据，接着协调节点对这 5000 条数据进行一些合并、处理，再获取到最终第 100 页的 10 条数据。
这样就导致翻页的深度越深，每个shard返回的数据就越多，node节点处理的时间就越长，数据返回就越慢。要么就是不允许深度翻页搜索，要么就是下拉翻页，采用es的scroll api，
会返回一个所有数据的快照，然后每次滑动都是根据游标scroll_id移动，获取下一页的数据，这样比一般的分页性能要高很多，但是这个只适用于下拉加载的，不适用于跳到指定页数的场景。
scroll需要初始化参数，保存搜索数据多久时间，确保用户不会持续不断翻页翻几个小时，否则可能因为超时而失败。除了scroll还可以使用search_after,同样是维护数据的游标，
只是search_after是实时去搜索数据，通过上一个数据页的最后一条数据的标识来获取下一页的数据，也不允许随意翻页，初始化时，需要使用一个唯一值的字段作为 sort 字段。

### es集群部署的配置
