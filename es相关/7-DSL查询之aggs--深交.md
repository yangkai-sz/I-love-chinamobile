# 1、背景
某2b公司，让统计es中已接入设备的数量，写命令（语句）。
想到distinct用法，但是一直想不起来es里咋写了，又好像es没有这用法，就想到了es里的类似group by的用法，通过曲线救国的方式，也可以实现查询。
如果依赖kibana，操作起来就很方便，但是这个2b让你默写。。。
上网查也没看明白，继续在kibana上摸索，发现了这个用法其实是：cardinality 靠你奶了个腿啊，废话不说了，上内容：
```
curl -XGET "http:///vssh--2020.01/_search" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "yk": {
      "cardinality": {
        "field": "log_daddr.keyword"
      }
    }
  }
}'
```
结构特点：aggs、自定义名字、关键字（cardinaliyt之类的）、field：字段名。

看size的用法，里面还可以穿插其他中的。 sum、min、max、average、terms都是这么用的。
## 1.1 其中terms相当于count 。。。 group by。。。的用法
```
curl -XGET "http://ip:9993/vssh--2020.01/_search" -H 'Content-Type: application/json' -d'
{
  "size": 0, 
  "aggs": {
    "kaige":{
      "terms": {
        "field": "beat.hostname.keyword",
        "size": 10，
        "order":{"_count":"desc"}
      }
    }
  }
}'
```
以beat.hostname.keyword进行分组，然后count，倒序排列。
## 1.2 count的话，直接这样就可以了:
```
curl -XGET "http:///vssh--2020.01/_search" -H 'Content-Type: application/json' -d'
{
  "size":0,
  "aggs":{}
}'
```

## 1.3 特别注意
count、cardinality上执行的字段，不需要是数字，普通text字段就行了。但是avg、mix、max，就需要在数字上执行了，跟sql语句是一样的道理，居然都忘记了。。。
# 这篇写的真不错，至少我看懂了，这个才最重要!
https://www.cnblogs.com/xing901022/p/4944043.html

# 2、感觉会用这几个就够了吧。。。。之前我觉得会用query就够了。。。

补充一个复杂一点的query，游标scollL：
```
curl -XPOST "http://ip:9993/audit-qizhi-fort_log-2019.12/_search?pretty&scroll=5m" -H 'Content-Type: application/json' -d'
{
  "size": 5,
  "sort" : ["_doc"],
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "log_operator": "1100000000155"
          }
        },
        {
          "range": {
            "@timestamp": {
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
}'
```

