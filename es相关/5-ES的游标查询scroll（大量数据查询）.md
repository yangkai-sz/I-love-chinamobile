# 1、es是什么
es是一个分布式的搜索引擎，也有人说是数据库。它能存数据，也可以分析，例如：avg、sum、count、max、min等（kibana，挺方便）。

# 2、数据查询

## 2.1、一般的query查询
按照一般的查询流程来说，如果我想查询前10条数据：

* 客户端请求发给某个节点
* 节点转发给个个分片，查询每个分片上的前10条
* 结果返回给节点，整合数据，提取前10条
* 返回给请求客户端

这个一次只能查询1w条（from+size的个数是有限制的，二者之和不能超过1w），再多查不了，而且翻页的话效率极低，因为他们都会加载到内存里。<br>
但是如果请求的是第1000页的数据，即第10001至第10010条数据。那么es会从5个分片中取出各个分片顶端的10010条数据，然后将这总共50050条数据返回给请求节点，请求节点再次对这50050条数据进行一次全局排序，取出第50041至50050条数据，抛弃50040条数据，这就造成了很大的浪费。
```
GET qizhi-2019/_search
{
  “from”:0，"size": 5,
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "log_operator": "110abc"
          }
        },
        {
          "range": {
            "log_time.keyword": {
              "gte": "2019-12-04",
              "lte": "2019-12-27",
              "format": "yyyy-MM-dd",
              "time_zone": "+08:00"
            }
          }
        }
      ]
    }
  }
}
```
## 2.2、scroll游标查询、分页
相对于from和size的分页来说，使用scroll可以模拟一个传统数据的游标，记录当前读取的文档信息位置。这个分页的用法，不是为了实时查询数据，而是为了一次性查询大量的数据（甚至是全部的数据）。

因为这个scroll相当于维护了一份当前索引段的快照信息，这个快照信息是你执行这个scroll查询时的快照。在这个查询后的任何新索引进来的数据，都不会在这个快照中查询到。但是它相对于from和size，不是查询所有数据然后剔除不要的部分，而是记录一个读取的位置，保证下一次快速继续读取。
* 官方参考文档：https://www.elastic.co/guide/cn/elasticsearch/guide/current/scroll.html
### 2.2.1 查询语句
```
curl -XGET "http://192.168.14.3:9399/ad_log/_search?pretty&scroll=5m" -H 'Content-Type: application/json' -d'
{
  "size": 10000,
  "sort" : ["_doc"],
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "app_Name": "browsermng"
          }
        },
        {
          "range": {
            "time.keyword": {
              "gte": "2019-10-12",
              "lte": "2019-12-12",
              "format": "yyyy-MM-dd",
              "time_zone": "+08:00"
            }
          }
        }
      ]
    }
  }
}'

```


### 2.2.1 分页查询：
```
curl -XGET "http://192.168.14.3:9399/_search?pretty&scroll=5m" -H 'Content-Type: application/json' -d'
{
    "scroll" : "5m", 
    "scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAOSkFkFfbm5jNGdEUWp1Y1FhVm02NkRwaXcAAAAAAADkpRZBX25uYzRnRFFqdWNRYVZtNjZEcGl3AAAAAAAAq4IWcHdlaE9IMk9STU9RT0RwVWk0dVo4ZwAAAAAAAOSmFkFfbm5jNGdEUWp1Y1FhVm02NkRwaXcAAAAAAACrgxZwd2VoT0gyT1JNT1FPRHBVaTR1Wjhn" 
}'
```
* 自作聪明：在这个scroll查询里，我又加上“ad_log“，就导致一直查询失败、报错。网上写的都没有，我却认为要加上。。。
* 网上说每次查询之后，scroll_id会变化，就要使用新的scroll_id，但我设置scroll=5m（分钟），经过测试的时候，发现没变化（5分钟以内）。可写自动化脚本来实现。

## 3、es的时间字段问题
* 技巧：有的时候有2b把日志的时间字段设置成text类型，这个时候再用range的时间段查询时，就要加上.keyword 
* 我还是认为，一般最好设置成日期date字段最好，否则kibana岂不是完蛋了？？？

