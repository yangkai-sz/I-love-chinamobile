# 3个平台背景
hadoop应该算是最早的大数据平台，核心能力就是海量数据的批处理，随着互联网的发展，出现了流处理框架如storm，但是它的exactly-once能力和性能渐渐的被flink超越了。
## 三个平台数据处理的核心概念：
* hadoop，批处理，数据处理的过程核心是map、reduce
* storm，流处理，数据处理过程的核心是topology，拓扑又有spout、bolt来组成。再往下就是task任务、worker工作进程、executor执行者
* flink，这个就牛逼了，支持批处理和流处理；数据处理过程的核心是datasource数据源、translecation计算单元（算子）、datasink（数据输出）。
* spark、spark streaming本质还是批处理，只是加入了内存计算、切分更小的片段。

## 再说一下批处理和流处理的区别，核心区别在数据传输上，具两个最极端的场景：
* 批处理：当一条数据处理完成之后，序列化到缓存里，待缓存写满时，持久化到硬盘上，这一批处理处理完毕之后，再发送到下一个节点；
* 流处理：当一条数据处理完成之后，序列化到缓存里，立马发送到下一个节点（如bolt）；
这两个极端的场景，一个吞吐量最大、一个实时性最高，不是说那个一定最好，还是要看业务场景的，这个时候，flink出现了更好的选择。flink可以手动配置缓存的大小，如果是0，那么就是上面的极端流处理场景，如果是无穷大，就是上面的极端批处理场景。手动配置的好处就是可以平衡实时性和吞吐量。

flink确实屌。flink依赖hadoop、scala，现在官方一般有一体化的包，直接下载就行了。flink有多种部署模式，一般都是flink on yarn这种。

# 其他，疑问
我看storm那本书里的案例，数据基本都是flume采集、落到hdfs上，然后再去处理的，是因为那个年代还没kafka、kafka还不流行吗？
通过pycharm默认安装的库，不一定靠谱，比如安装Python-nmap的时候，import nmap时自动导入。

portscanner()普通扫描类
异步扫描类
迭代器，推荐使用

50，35000+1500
https://www.cnblogs.com/binyue/p/10308754.html 
Kafka为啥快？Kafka会把收到的消息都写入到硬盘中，它绝对不会丢失数据。为了优化写入速度Kafka采用了两个技术， 顺序写入 和 MMFile 。
因为硬盘是机械结构，每次读写都会寻址->写入，其中寻址是一个“机械动作”，它是最耗时的。所以硬盘最讨厌随机I/O，最喜欢顺序I/O。为了提高读写硬盘的速度，Kafka就是使用顺序I/O
即便是顺序写入硬盘，硬盘的访问速度还是不可能追上内存。所以Kafka的数据并 不是实时的写入硬盘 ，它充分利用了现代操作系统 分页存储 来利用内存提高I/O效率。
Memory Mapped Files(后面简称mmap)也被翻译成 内存映射文件 ，在64位操作系统中一般可以表示20G的数据文件，它的工作原理是直接利用操作系统的Page来实现文件到物理内存的直接映射。完成映射之后你对物理内存的操作会被同步到硬盘上（操作系统在适当的时候）。
通过mmap，进程像读写硬盘一样读写内存（当然是虚拟机内存），也不必关心内存的大小有虚拟内存为我们兜底。
使用这种方式可以获取很大的I/O提升， 省去了用户空间到内核空间 复制的开销（调用文件的read会把数据先放到内核空间的内存中，然后再复制到用户空间的内存中。）也有一个很明显的缺陷——不可靠， 写到mmap中的数据并没有被真正的写到硬盘，操作系统会在程序主动调用flush的时候才把数据真正的写到硬盘。 Kafka提供了一个参数——producer.type来控制是不是主动flush，如果Kafka写入到mmap之后就立即flush然后再返回Producer叫 同步 (sync)；写入mmap之后立即返回Producer不调用flush叫 异步 (async)。

# es的bulk

curl -X POST "localhost:9200/_bulk?pretty" -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }

# es的乱七八糟的东西
es
Es的优点是搜索、方便、快，hbase是nosql  key-value模式，存储很强。es存储结构可扩展性更强(对任意文档，无限加字段）。hbase数据更新更方便。
推荐的es用来搜索、hbase存储原始的。
Elk自成体系，hadoop的太多复杂了。   hbase一开始设计就是为了解决海量数据的，可能会更强。
————————————
分片只有在创建索引时确定，一旦确定就不能修改了，每个分片的不能太大，否则磁盘压力大，搜索也不快。设计index的时候，最佳建议是根据时间，比如天、周、月，当然数据数据量严重不均衡，
Elk在访问控制层面，好像没有。。。——怎么可能，只是收费而已，x-pack

一个是搜索引擎，一个是数据库
=============================
集群、节点、分片、索引、type、文档。全文检索、倒排索引、lucense，mysql对比。github、wiki、百度很多产品也使用这个。
=============================
查询es的数据，根据log_cmd去判断，设置告警事件和级别，然后根据查出来的文档ID去更新文档。
查询---字符串—-json.load()字典，然后根据字典的查询方式获取对应的值，中间的是list，就按照list的方式来查询。data3 = data2["hits"]["hits"][0]


=============================
使用elastic search使用filtered过滤数据时报no [query] registered for [filtered]错误，原因在于过滤器filter 已被弃用，并在ES5.0中删除所以应用bool/must/filter查询来替换

===========dsl==================
restful api，dsl，基于url。
Match 	term		terms 	multp_match	query_string
Query bool must must_not 	should filter 	_source:[“field1”,”field2”]
store_field	constant_score	prefix	size 		from		
至于data需要转为json，headers如果转了就报错！！！
Requests.post(url, data, headers=heard)
Header = content_type: content_json
res.text
Data = Json.dump(‘str’)
Url = ‘http://localhost:9200/_search?source=log_cmd,log_time'
Json.loads()	Json.dumps()		re.search(’str’,xx)	line.strip()	line.split(‘xx’,xx)
constant_score
sort [field:【order：desc]
==========_souce=====kibana的自动语法不对==============

GET mysql*2019.08*/_search
{"query": {
  "match_all": {}
},"_source": ["log_time"]
}
============日志字符串转化。range、gte、lte=================
1. 日期字符串
GET /ops-coffee-2019.05.14/_search
{
  "query": {
    "range":{
      "@timestamp":{
        "gte": "2019-08-14 00:00:00",
        "lte": "2019-08-15”,
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd",
        "time_zone": "+08:00"
      }
    }
  }
}
=================好的写法======================
Must must_not should filter,	后面都先做中括号[]，然后在里面分别用大括号扩起来些表达式；
"filter": [{
        "term": {
          "obj_type": "host"
        }
      }]
注意看关键字的等级，range和term、match是同一级别，同一级别的放的位置、层级都是一样的。要么在query下单独使用，要么在bool，must、must_not、should（或filter）下，在[]内跟别的同时使用。很多时候语法不对，可能是因为你的并行查询没用[{},	{}]。	query、size、from、_source这四个是同一个级别的。
GET mysql-log-fort_log-2019.08/_search
{
  "query": {
    "bool": {
      "filter": [{
        "term": {
          "log_cmd": "sz"
        }
      },{
        "term": {
          "warning_rank": "3"
        }
      }]
    }
  }
}


通常更推荐用这种日期字符串的方式，看起来比较清晰，日期格式可以按照自己的习惯输入，只需要format字段指定匹配的格式，如果格式有多个就用||分开，像例子中那样，不过我更推荐用同样的日期格式
如果日期中缺少年月日这些内容，那么缺少的部分会用unix的开始时间（即1970年1月1日）填充，当你将"format":"dd"指定为格式时，那么"gte":10将被转换成1970-01-10T00:00:00.000Z
elasticsearch中默认使用的是UTC时间，所以我们在使用时要通过time_zone来设置好时区，以免出错
===================keyword==========================
* 其实 Elasticsearch 5.X 之后给 text 类型的分词字段，又默认新增了一个子字段 keyword，这个字段的类型就是 keyword，是不分词的，默认保留 256 个字符。假设 product_name 是分词字段，那有一个 product_name.keyword 是不分词的字段，也可以用这个子字段来做完全匹配查询。
term 用法（与 match 进行对比）（term 一般用在不分词字段上的，因为它是完全匹配查询，如果要查询的字段是分词字段就会被拆分成各种分词结果，和完全查询的内容就对应不上了。）
query 和 filter 一起使用的话，filter 会先执行，看本文下面的：多搜索条件组合查询
http://www.youmeek.com/elasticsearch-dsl/ 
======================官网文档==================================
1、green 正常，2 red有主分片不正常，3、yellow，主分片正常、有副本不正常；
2、创建索引时分片确定后不能改、副本还可以改；相同的主分片跟副本分片不会在同一个机器上
3、相同的cluster.name,可以加入一个集群。好像还得把ip配置上去，discover_host[]；
4、blogs 索引现在拥有9个分片：3个主分片和6个副本分片。 这意味着我们可以将集群扩容到9个节点，每个节点上一个分片。相比原来3个节点时，集群搜索性能可以提升 3 倍。
5、field名字不能包含英文句号。文档5元组：_id、_index_name、_type、_score、_souce字段。
6、索引名，这个名字必须小写，不能以下划线开头，不能包含逗号。——潘磊测试的时候，犯了这个错误，应该及时查看日志！！！logs/xxx.log
7、PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
8、分片太少影响扩展性，太多影响性能；凡是讲究一个平衡
========================================================
悲观并发控制
这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。
乐观并发控制
Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。

搜索 1 个有着 50 个分片的索引与搜索 50 个每个都有 1 个分片的索引完全等价：搜索请求均命中 50 个分片。
当你需要在不停服务的情况下增加容量时，下面有一些有用的建议。相较于将数据迁移到更大的索引中，你可以仅仅做下面这些操作：
* 创建一个新的索引来存储新的数据。
* 同时搜索两个索引来获取新数据和旧数据。
实际上，通过一点预先计划，添加一个新索引可以通过一种完全透明的方式完成，你的应用程序根本不会察觉到任何的改变。
在 索引别名和零停机，我们提到过使用索引别名来指向当前版本的索引。 举例来说，给你的索引命名为 tweets_v1 而不是 tweets 。你的应用程序会与 tweets 进行交互，但事实上它是一个指向 tweets_v1 的别名。 这允许你将别名切换至一个更新版本的索引而保持服务运转。
我们可以使用一个类似的技术通过增加一个新索引来扩展容量。这需要一点点规划，因为你需要两个别名：一个用于搜索另一个用于索引数据：
PUT /tweets_1/_alias/tweets_search 
PUT /tweets_1/_alias/tweets_index 
	tweets_search 与 tweets_index 这两个别名都指向索引 tweets_1 。
新文档应当索引至 tweets_index ，同时，搜索请求应当对别名 tweets_search 发出。目前，这两个别名指向同一个索引。

基于时间序列的是比较好的管理方式，按时间进行索引，也访问清理历史数据，如果为了性能，可能关闭比较老的索引，post fork-log.2018*/_close，下次用的时候再执行一下_open即可。非常旧的索引 可以通过snapshot-restore API归档至长期存储例如共享磁盘或者 Amazon S3。
另外一种是基于用户的索引，es支持多租户的模式，可以为每个用户建立一个index。

当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点，因为所有节点都能处理需求、都知道每个文档的位置。

在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。
写文档的时候，都是先写入主分片，然后再同步到副本分片。查询的时候，主、副都行。

1） “我应该有多少个分片？”
答： 每个节点的分片数量保持在低于每1GB堆内存对应集群的分片在20-25之间。
2） “我的分片应该有多大”？
答：分片大小为50GB通常被界定为适用于各种用例的限制。


=======================20190819=================================
1、range一般配filter使用，不用must什么的；
2、如何选择查询与过滤
编辑
通常的规则是，使用 查询（query）语句来进行 全文 搜索或者其它任何需要影响 相关性得分 的搜索。除此以外的情况都使用过滤（filters)。
3、当一个搜索请求被发送到某个节点时，这个节点就变成了协调节点。 这个节点的任务是广播查询请求到所有相关分片并将它们的响应整合成全局排序后的结果集合，这个结果集合会返回给客户端。
4、当进行精确值查找时， 我们会使用过滤器（filters）。过滤器很重要，因为它们执行速度非常快，不会计算相关度（直接跳过了整个评分阶段）而且很容易被缓存。我们会在本章后面的 过滤器缓存 中讨论过滤器的性能优势，不过现在只要记住：请尽可能多的使用过滤式查询。
5、所以当我们用 term 查询查找产品id等于精确值 XHDK-A-1293-#fJ3 的时候，找不到任何文档，因为它并不在我们的倒排索引中，正如前面呈现出的分析结果，索引里有四个 token 。在es6以后每个text类型，自动增加了一个叫.keyword的字段！！！——不是的，是logstash会自动给text类型增加一个keyword字段，是es5以后去掉string类型，增加了text和keyword这两种类型，keyword是不分析的、可以用来完全匹配的。
6、term的完全匹配，不是等于：
GET mysql-log-fort_log-2019.06*/_search
{
  "query": {
    "bool": {"filter": {
    "term": {
      "log_cmd": {
        "value": "sz"
      }
    }
  }}},"_source": ["log_cmd"],"size": 20
}
查出来sz开头的所有的指令。符合预期。也可以只用term，但最佳实践是能用filter就用。

一定要了解 term 和 terms 是 包含（contains） 操作，而非 等值（equals） （判断）。 如何理解这句话呢？
如果我们有一个 term（词项）过滤器 { "term" : { "tags" : "search" } } ，它会与以下两个文档 同时 匹配：
{ "tags" : ["search"] }
{ "tags" : ["search", "open_source"] } 
	尽管第二个文档包含除 search 以外的其他词，它还是被匹配并作为结果返回。
用log_cmd.keyword才会完全匹配sz ！！！！not_analysis!!!
========================================================
1、敏锐的读者会注意，目前为止本书介绍的所有查询都是针对整个词的操作。为了能匹配，只能查找倒排索引中存在的词，最小的单元为单个词。还可以用prefix前缀匹配、wildcard正则表达式，但是效率很差，不建议使用。
2、输入即搜索（search-as-you-type） ——在用户键入搜索词过程的同时就呈现最可能的结果。

========================分词问题================================
1、针对不通的语言，处理起来也不一样，要用不同分词器，可以在创建索引的时候制定，一般默认有默认的standard和english的；这对德育、法语等，太多了，没必要都掌握，用不到。针对中文，用icu插件来支持，针对html，用html_strip过滤器来处理。
2、分词的特点，不改变语义、提取关键词，比如jump、jumper、leap会认为是同1个词（同义词过滤器）；不同的场景，用不同的方法；
3、sz命令vssh不支持，但是可以继续做。——欧俊，找张伟装logstash
========================================================
应用层关联，先查出来符合条件的文档的id，然后基于id进行更新。
Post ip/index/doc/id/_update
{“doc”:{dict:dict}}
如何加了id，那么后面不能用search、直接查询就行，doc必须要有，是doc API。kibana的ide也不错，有语法提醒。不过有时候没提醒，也未必是错的。
======================防止脑裂==================================
1、合理设置master数量
2、master不做data.node,防止负载过高，导致被认为挂了———一个直观的解决方案便是将master节点与data节点分离，我们添加了三台服务器进入ES集群，不过它们的角色只是master节点，不担任存储和搜索的角色，故它们是相对轻量级的进程。可以通过以下配置来限制其角色：
* node.master: true
* node.data: false
https://blog.csdn.net/cnweike/article/details/39083089 详细介绍。
3、有效的保障网络的稳定性，不跨机房，最好在同网断，或者同一汇聚交换机下；
========================================================
========================================================


？病毒、上网行为控制，如果我们做搭一个人进去都未必能够。

Proxy的告警，发日志给我、还是开账号给我？沟通一下。邮件监控。拦截的、不拦截。————拦截的警告一下责任人，不拦截的  违规的就要处罚。

Proxy的功能？上网行为管理。
=======老板====
没有时间考虑管理运作，知道该怎么做，但没时间。
安全是重运营的工作。缺人、数据安全也缺人。
Hids天眼云镜和云镜。  横向团队认可，没挑战、攻击性言论。

汇报ppt、指标放到km，大家看下，如有需要调整就调整。
2019年的规划和目标。月底老板给讲一下。

专利！！！
大学生一年左右就升职，大家一起想办法。——老板
应付写一篇其实对自己也不好，浪费时间。必须得认真写。必须有一篇是自己为主要发明人，不能一直搭别得车。晋升c2的，找找人想想办法。不要去买，买的写的太专业了，专利工程师直接给打回。
业务安全在7月份累计过了8篇，基础安全是0。
专利需要静下心来去写。我们组专利不达标，对曹老板会处罚1w多。

我们这边重要是看结果，isc大会报了10个人，深圳的会，大家想去给曹老板说一下。不能开个大会，办公室剩我一个了！各个小组的人轮着去。不提倡加班，最近看大家走的特别晚，我们是不是有特殊的事情？人的精力是有限的，长期这样不行。公司有加班的风气，大家把握一个度，但公司不要求。v消息，半个小时看一次，我们协作的工作太多。半天各种被打扰。大家有什么想聊想说的。

自己做的事情是不是想做的，目标清楚。晋升、职业发展的诉求。


头条的告警数据字段分别是什么意思？需要给标记一下。——16进制、base64、unicode
