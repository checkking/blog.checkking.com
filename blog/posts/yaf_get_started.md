<!--
author: checkking
date: 2016-12-27
title: Yaf Hello World
tags: php,yaf
category:php
status: publish
summary: 开始运行yaf
-->
## 开始使用Yaf
### 生成yaf框架代码
```
php yaf_cg Sample
```
将output下的Sample拷贝到自己的目录下，然后配置好nginx.conf, 并重启。
配置如下
```
    server {
        listen       8091;
        server_name  localhost;
        access_log  logs/yaf.access.log;
        root /root/github/yaflearn/Sample;
        index index.html index.htm index.php;

        location ~ \.(jpg|png|gif|js|css|swf|flv|ico)$ {
            expires 12h;
        }

        location / {
            if (!-e $request_filename) {
                rewrite ^(.*)$ /index.php?$1 last ;
                break;
            }
        }

        location ~ .*\.(php|php5)?$ {
            fastcgi_connect_timeout 300;
            fastcgi_send_timeout 300;
            fastcgi_read_timeout 300;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }

```
为了便于发现php错误，开启错误日志
```
error_reporting=E_ALL
display_errors=Off
log_errors=On
log_errors_max_len=1024
error_log=/usr/local/php/log/php.log.error

```
