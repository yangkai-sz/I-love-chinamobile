# 1、背景
自研发的堡垒机，也要收集审计日志，那么约定日志格式，为json，关键字段还是哪些：某人、在某时间、在某主机上、执行了某命令等。
日志格式约定：
## 1、日志要求
路径建议为：/data/selfBastionHost/selfShell.log
文件编码为utf8；
日志字段要求：
```
{
  "log_operator": "操作人工号",
  "log_operator_name": "操作人姓名",
  "log_operator_dep": "操作人所属部门",
  "log_time": "操作时间",
  "warning_event": "default",
  "warning_rank": "default",
  "audit_obj_type": "default",
  "log_cmd": "操作指令（如果是登录、注销，则打印oppo.login或oppo.logout",
  "log_output": "命令输出内容(要控制内容长度，超过1000个字符，自己保存，不要输出到该文件)",
  "log_daddr": "操作目的地址",
  "log_saddr": "操作源地址",
  "dst_hostname": "操作目的主机的主机名",
  "log_source":"selfBastionHost" 
  }
```
日志样例：
```
{
  "log_operator": "11111111110",
  "log_operator_name": "张三",
  "log_operator_dep": "互联平台部",
  "log_time": "2020-03-10 09:39:28.45",
  "warning_event": "default",
  "warning_rank": "default",
  "audit_obj_type": "default",
  "log_cmd": "ls",
  "log_output": "filea fileb dira",
  "log_daddr": "110.102.181.111",
  "log_saddr": "110.102.181.112",
  "dst_hostname": "sping2.o.lan", 
  "log_source":"selfBastionHost"      
  }
```
## 2、采集方式
在生成日志文件的服务器上，部署filebeat，监听selfShell.log，待准备工作完成好，请联系我来测试、部署。
为了避免日志文件过大，可定期rotate。但不要随意改名、移除文件、关闭filebeat进程。
## 3、日志安全问题
由于日志中存在敏感信息(如密码)，请注意保护日志的访问权限、避免日志文件外泄。基于“最小化原则”，对可访问日志的模块的访问权限应严格控制，避免无关人员访问。具体的细节，可以基于你们的实现方式我们再沟通。
# 2、filebeat配置
格式还是上一篇一样，还是原来的那种方式，当文本来采集。type为log
# 3、logstash配置：
```
$ cat selfAudit.conf-bak
input {
    beats {
        port => 10516
        codec => "json"
    }
}

filter {
    if "log_time" in [message]{
        if [log_time] {
            date{
                match => ["log_time", "yyyy-MM-dd HH:mm:ss.SSSSSS"]
                target => "@timestamp"
            }
        }
        fingerprint{
            source => ["log_time","log_cmd","log_operator","log_daddr"]
            target => "[@metadata][fingerprint]"
            method => "SHA1"
            key => "Log analytics"
            base64encode => true
            concatenate_sources => true
        }
    }
    mutate { remove_field => ["host"] }
}

output {
    elasticsearch {
        hosts => ["110.192.162.114:9123"]
        index => "selfaudit--%{+YYYY.MM}"
        document_id => "%{[@metadata][fingerprint]}"
    }
}
```
# 4、遇到的问题
### （1）开发写的json的大括号外层有双引号，则filebeat无法读取，这个没搞明白为什么，删除就可以了。
### （2）logstash索引的名字为selftAudit--，忘记了index名字里不能有大写字母里，日志里其实有写
```
[2020-03-20T15:09:59,755][ERROR][logstash.outputs.elasticsearch] Could not index event to Elasticsearch. {:status=>400, :action=>["index", {:_id=>nil, :_index=>"selfAudit--2020.03", :_type=>"doc", :_routing=>nil}, #<LogStash::Event:0x34587b19>], :response=>{"index"=>{"_index"=>"selfAudit--2020.03", "_type"=>"doc", "_id"=>nil, "status"=>400, "error"=>{"type"=>"invalid_index_name_exception", "reason"=>"Invalid index name [selfAudit--2020.03], must be lowercase", "index_uuid"=>"_na_", "index"=>"selfAudit--2020.03"}}}}
```

### （3）开发输出的json格式不对，用的是单引号，应该是双引号
```
JSON parse error, original data now in message field {:error=>#<LogStash::Json::ParserError: incompatible json object type=java.lang.String , only hash map or arrays are supported>, :data=>"\"{'log_operator': '119999916', 'log_operator_name': '磊', 'log_operator_dep': '互联网平台部', 'log_time': '2020-03-20 15:44:24', 'warning_event': 'default', 'warning_rank': 'default', 'audit_obj_type': 'default', 'log_cmd': 'logout', 'log_output': '', 'log_daddr': '110.130.99.37', 'log_saddr': '110.209.81.25', 'dst_hostname': 'vsflow-po.lan', 'log_source': 'selfBastionHost'}\""}
```
### （4）logstash的input里面，要有json解析的配置，否则收到的只有一个整体的message:
```
codec => "json"
```

# 5、最终结果样例
## 成功解析json的情况
```
{
  "_index": "selfaudit--2020.03",
  "_type": "doc",
  "_id": "%{[@metadata][fingerprint]}",
  "_version": 1,
  "_score": null,
  "_source": {
    "log_saddr": "110.209.18.215",
    "warning_event": "default",
    "log_cmd": "oppo.logout",
    "tags": [
      "beats_input_codec_json_applied"
    ],
    "log_operator_dep": "互联网平台部",
    "log_source": "selfBastionHost",
    "dst_hostname": "vsflow-po.lan",
    "log_daddr": "110.110.99.37",
    "beat": {
      "hostname": "vjumpserv.lan",
      "name": "vjumpserver-.lan",
      "version": "6.3.2"
    },
    "audit_obj_type": "default",
    "log_operator_name": "磊",
    "warning_rank": "default",
    "log_operator": "11111111",
    "log_output": "",
    "@timestamp": "2020-03-20T07:28:33.874Z",
    "input": {
      "type": "log"
    },
    "@version": "1",
    "source": "/data/selfBastionHost/selfShell1.log",
    "offset": 0,
    "prospector": {
      "type": "log"
    },
    "log_time": "2020-03-19 15:31:15"
  },
  "fields": {
    "@timestamp": [
      "2020-03-20T07:28:33.874Z"
    ]
  },
  "sort": [
    1584689313874
  ]
}
```
## json解析失败的情况
```
{
  "_index": "selfaudit--2020.03",
  "_type": "doc",
  "_id": "2bUehFUgH+6ifa3fss/1PQY5R3s=",
  "_version": 5,
  "_score": null,
  "_source": {
    "beat": {
      "hostname": "vjumpserve.lan",
      "name": "vjumpserver-o.lan",
      "version": "6.3.2"
    },
    "message": "\"{'log_operator': '11111111', 'log_operator_name': '磊', 'log_operator_dep': '互联网平台部', 'log_time': '2020-03-20 16:02:14', 'warning_event': 'default', 'warning_rank': 'default', 'audit_obj_type': 'default', 'log_cmd': 'oppo.login', 'log_output': '', 'log_daddr': '110.130.102.115', 'log_saddr': '110.209.118.125', 'dst_hostname': 'vsflow-p.lan', 'log_source': 'selfBastionHost'}\"",
    "@version": "1",
    "tags": [
      "_jsonparsefailure",
      "beats_input_codec_json_applied"
    ],
    "source": "/data/selfBastionHost/selfShell.log",
    "offset": 1735,
    "prospector": {
      "type": "log"
    },
    "@timestamp": "2020-03-20T08:02:15.520Z",
    "input": {
      "type": "log"
    }
  },
  "fields": {
    "@timestamp": [
      "2020-03-20T08:02:15.520Z"
    ]
  },
  "sort": [
    1584691335520
  ]
}
```
