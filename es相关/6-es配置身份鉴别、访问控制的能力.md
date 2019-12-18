# es 6.8和7.1以后的版本，xpack有免费的基础版，支持了一些安全功能。有：
加密通信————是指集群间，并不是es的访问、查询。官方说明里的kibana访问，浏览器那里页是叹号❗️
* 基于角色的访问控制
* 文件和原生身份验证
* Kibana Spaces
* Kibana 功能控制
* API 密钥管理
# 参考资料
可参考：https://www.elastic.co/cn/subscriptions


es集群配置访问控制：https://www.elastic.co/cn/blog/getting-started-with-elasticsearch-security
                 https://www.elastic.co/guide/en/elasticsearch/reference/7.1/configuring-security.html


es单节点配置案例：https://blog.csdn.net/qq_24570443/article/details/94136443
这个测试过了，可以。是http访问，要输入密码才能访问es，片段：
```
[es@localhost bin]$ ./elasticsearch-setup-passwords interactive
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]:
passwords must be at least [6] characters long
Try again.
Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [apm_system]:
Reenter password for [apm_system]:
Enter password for [kibana]:
Reenter password for [kibana]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Enter password for [beats_system]:
Reenter password for [beats_system]:
Enter password for [remote_monitoring_user]:
Reenter password for [remote_monitoring_user]:
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
[es@localhost bin]$ 
```

# 照着上面的说明操作都没问题，验证过了

