https://www.cnblogs.com/xishuai/p/elk-logstash-filebeat.html
这篇文章写得很好，包括了：
1. 环境准备
2. 安装 Logstash
3. 配置 Logstash
4. Logstash 采集的日志数据，在 Kibana 中显示
5. 安装配置 Filebeat
6. Filebeat 采集的日志数据，在 Kibana 中显示
7. Filebeat 采集日志数据，Logstash 过滤
8. Filebeat 采集的日志数据，Logstash 过滤后，在 Kibana 中显示
====================
我在linux安装软件，从来不喜欢使用yum，除了依赖包，我从来都是下载编译或者下载解压即用的安装包。
目标：使用filebeat采集文件发送到logstash

去官网下载相同版本的filebeat。
解压，编辑filebeat.yml文件：
---------------
filebeat.prospectors:
- type: log
   paths:
      - /var/log/messages  # 指定需要收集的日志文件的路径

output.logstash:
  # The Logstash hosts
  hosts: ["node1:10515"]
-----------------

修改logstash配置文件：
input {
 beats {
   port => 10515
  }
}
-----------------
启动logstash，启动filebeat：./filebeat -e -c ./filebeat.yml
测试：文件可以正常写入es、正常在kibana上展示。
都是beat比logstash轻量级，那我就看看：
[root@localhost log]# ps aux |grep logstash
es         5946 11.2（cpu占比） 37.8 （内存占比）

[es@bogon es]$ ps aux |grep filebeat
es        13480  0.0（cpu占比）  1.0 （内存占比）

对比结果很明显
