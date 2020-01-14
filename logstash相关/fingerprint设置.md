# 背景
为了避免写入es的文档重复，可以zailogstash中配置fingerprint选项，然后文档的id使用fingerprint生成。
前面的那个jumper的logstash配置里，少了一行，在logstash 7版本里不报错，但是在6版本里就报错，排查了好久好久，最后一对比，才发现填写配置的时候少了一行。
# 报错信息
```
[2020-01-14T09:54:13,898][ERROR][logstash.pipeline        ] Pipeline aborted due to error {:pipeline_id=>"main", :exception=>#<LogStash::ConfigurationError: Cannot register filter fingerprint plugin. The error reported is:
  Key value is empty. Please fill in an encryption key>,
```
# 原始的配置

```
        fingerprint{
            source => ["log_time","log_operator"]
            target => "[@metadata][fingerprint]"
            method => "SHA1"
            base64encode => true
            concatenate_sources => true
```

# 正确的配置

```
        fingerprint{
            source => ["log_time","log_operator"]
            target => "[@metadata][fingerprint]"
            method => "SHA1"
            key => "Log analytics"
            base64encode => true
            concatenate_sources => true
```

少了一行：key => "Log analytics"
