# 1、nginx配置技巧

* 设置密码登陆：比如使用开源的kibana，设置访问控制能力，配置个密码。
* 反向代理，这个没啥好说的，常见用法
* 定制化日志格式，记录post参数。一般是不记录request_body的，但有时候为了审计需求，还是得记录的。如toutiao

# 2、配置文件
```
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      '"upstream_addr" "$request_body"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8080;
        server_name  localhost;
        auth_basic "Please input password";
        auth_basic_user_file /opt/es/nginx_install/conf/passwd;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass http://192.10.2.4:5601$request_uri;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;
```

# 日志样例
```
110.101.181.223 - audit [03/Dec/2019:17:29:22 +0800] "POST /api/index_management/indices/reload HTTP/1.1" 200 559 "http://shell.xxx.com/app/kibana" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "192.168.209.22, 158.22.2.6""upstream_addr" "{\x22indexNames\x22:[\x22audit-paas-2019.12\x22,\x22audit-paas-2019.09\x22,\x22audit-paas-2019.07\x22,\x22audit-paas-2019.08\x22,\x22audit-paas-2019.10\x22,\x22audit-qizhi-fort_log-2019.06\x22,\x22audit-qizhi-fort_log-2019.09\x22,\x22audit-qizhi-fort_log-2019.08\x22,\x22audit-qizhi-fort_log-2019.10\x22,\x22audit-qizhi-fort_log-2019.07\x22]}"
10.102.81.223 - audit [03/Dec/2019:17:29:48 +0800] "GET /api/saved_objects/_find?type=index-pattern&fields=title&per_page=10000 HTTP/1.1" 200 210 "http://shell.xxx.com/app/kibana" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "192.168.209.22, 158.22.2.6""upstream_addr" "-"
```
