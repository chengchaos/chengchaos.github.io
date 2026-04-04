---
title: 解决 CORS 问题
key: 2023-02-03
tags: Nginx CORS 跨域
---

> Suggest search： Nginx CORS 跨域

<!--more-->

## 0x01 CORS
CORS 是一个 W3C 标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

详细的可以参考： [(阮一峰)跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)。

## 0x02 Nginx 解决方案

```conf
location / {

    add_header Access-Control-Allow-Credentials true;
    add_header Access-Control-Allow-Origin 'https://crh-int.bmw-brilliance.cn' always;
    add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,token';
    #add_header Access-Control-Allow-Headers '*';

    if ($request_method = 'OPTIONS') {
        return 204;
    }
}

```

示例：


```bash
cat nginx.conf

user  nginx;
worker_processes  auto;

events {
    worker_connections  1024;
    use epoll;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    ## 加固
    proxy_hide_header X-Powered-By; 
    proxy_hide_header Server;

    include conf.d/*.conf;

    server {
        listen       8000;
        server_name  localhost;

        ##  加固
        ssl_protocols TLSv1.2;
        server_tokens off;


        #charset koi8-r;

        #access_log  /var/log/nginx/host.access.log  main;

        location / {
            root   /srv/www/htdocs/;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /srv/www/htdocs/;
        }

## custome begin ###

        location /charging {
                proxy_pass http://charging;
        }

        location /iwb/webServices {
                proxy_pass http://iwb-web-socket;
                proxy_read_timeout 300s;
                proxy_send_timeout 300s;
                proxy_set_header  Host $http_host;
                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header  X-Forwarded-Proto $scheme;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
        }

        # iwb-app-backend
        location /iwb/app {
                proxy_pass http://iwb-app-backend;
                proxy_read_timeout 300s;
                proxy_send_timeout 300s;
                proxy_set_header  Host $http_host;
                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header  X-Forwarded-Proto $scheme;
                #proxy_http_version 1.1;
        }
        # cp-auc
        location /cp-auc/ {
                proxy_pass http://cp-auc;
                proxy_read_timeout 300s;
                proxy_send_timeout 300s;
                proxy_set_header  Host $http_host;
                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header  X-Forwarded-Proto $scheme;
        }
        # monitor
        location /monitor/ {
                add_header Access-Control-Allow-Credentials true;
                add_header Access-Control-Allow-Origin 'https://静态页面的域名' always;
                add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS';
                add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,token';
                #add_header Access-Control-Allow-Headers '*';

                if ($request_method = 'OPTIONS') {
                    return 204;
                }
                proxy_pass http://monitor;
                proxy_read_timeout 300s;
                proxy_send_timeout 300s;
                proxy_set_header  Host $http_host;
                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header  X-Forwarded-Proto $scheme;
        }

        # EarlyWarning 
        location /EarlyWarning/ {
                add_header Access-Control-Allow-Credentials true;
                add_header Access-Control-Allow-Origin 'https://静态页面的域名' always;
                add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS';
                add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,token';

                if ($request_method = 'OPTIONS') {
                    return 204;
                }
                proxy_pass http://EarlyWarning;
                proxy_read_timeout 300s;
                proxy_send_timeout 300s;
                proxy_set_header  Host $http_host;
                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header  X-Forwarded-Proto $scheme;
        }
    }
    include vhosts.d/*.conf;

}
```


实际配置的时候，需要参照 firefox 的 Console 的错误提示进行调整。

EOF


