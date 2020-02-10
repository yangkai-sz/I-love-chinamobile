# 1、背景
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
* 结构特点：aggs、自定义名字、关键字（cardinaliyt之类的）、field：字段名，共计4部分，算上"size":0，共计5部分！！！
* 如果涉及排序的aggs terms，就要增加排序和另外的size。
* 看size的用法，里面还可以穿插其他中的,例如query，from size，_source(只打印部分字段，如果size为0则无意义了）。 
* sum、min、max、average、cardinality都是这么用的,terms略微不同。

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
count、cardinality上执行的字段，不需要是数字，普通text字段就行了。但是avg、mix、max，就需要在数字、日期上执行了，跟sql语句是一样的道理，居然都忘记了。。。

例如：
```
{
  "aggs": {
    "2": {
      "terms": {
        "field": "beat.hostname.keyword",
        "size": 99,
        "order": {
          "1": "desc"
        }
      },
      "aggs": {
        "1": {
          "max": {
            "field": "@timestamp"
          }
        }
      }
    }
  }
```
效果是：
以beat.hostname为分组，统计每个beat.hostname最新（最后）上报日志的时间。这个是vssh日志采集的索引。max用在了时间字段上，group by的另外一种用法。
第二个aggs是跟上面的terms是一个层级的，它相当于是嵌套到里面的。效果实例：
```
一共两列，第一列为hostname，第二列为最大的日期，倒序：

beat.hostname.keyword:Descending 			Max @timestamp 
spid-plan1					2020-01-20, 19:47:12.597
spid-plan2					2020-01-20, 19:47:12.597
依次排列
 	 
```

## 1.4 一般aggs要根query配合着用
一般aggs要跟query配合着用才符合日常需要，至少在时间段上选择吧，不可能查所有的数据。格式就是：
```
{
  aggs：。。。。，
  query：。。。。
}
```
可以再穿插 size、_source、from之类的字段。

## 1.5 aggs查下中的field，用使用xxx.keyword的才行


## 这篇写的真不错，至少我看懂了，这个才最重要!
https://www.cnblogs.com/xing901022/p/4944043.html  </br>
注意结构：
```
{
    "size":0,
    "aggs" : {
        "max_price" : { "max" : { "field" : "price" } }
    }
}
```
# 2、感觉会用这几个就够了吧。。。。之前我觉得会用query就够了。。。

## 2.1 补充一个复杂一点的query，游标scollL：
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

## 2.2 同时用到了size，agg，query的查询
 另外一个，日志查询的，谛听的操作的用户统计group by：

```
{
	"size": 0,
	"aggs": {
		"yk": {
			"terms": {
				"field": "userId.keyword",
				"size": 10,
				"order": {
					"_count": "desc"
				}
			}
		}
	},
	"query": {
		"bool": {
			"must": [
				{
					"match": {
						"appName.keyword": "diting-console"
					}
				},
				{
					"range": {
						"time.keyword": {
							"gte": "2020-01-01",
							"lte": "2020-01-20",
							"format": "yyyy-MM-dd"
						}
					}
				}
			]
		}
	}
}
```
这里面的time是text类型，不是date，所以用了time.keyword，text类型也可以比较，开始以为不可以的。

## 2.3 kibana上的标准group by的案例，复制下来一个：
效果基本与1.3完全相同，少了一点其他的东西而已。
```
{
  "aggs": {
    "2": {
      "terms": {
        "field": "beat.hostname.keyword",
        "size": 99,
        "order": {
          "_count": "desc"
        }
      }
    }
  },
  "size": 0,
  "_source": {
    "excludes": []
  },
  "stored_fields": [
    "*"
  ],
  "script_fields": {},
  "docvalue_fields": [
    "@timestamp"
  ],
  "query": {
    "bool": {
      "must": [
        {
          "match_all": {}
        },
        {
          "range": {
            "@timestamp": {
              "gte": 1579675376814,
              "lte": 1579676276814,
              "format": "epoch_millis"
            }
          }
        }
      ],
      "filter": [],
      "should": [],
      "must_not": []
    }
  }
}
```
注意一下最后query中的must、filter、should、must_not的用法。
