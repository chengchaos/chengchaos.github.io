---
title: Nginx 的配置文件示例
key: 2020-09-07
tags: nginx
---


Nginx 的配置文件中和安全有关的内容示例。

<!--more-->

## 一个示例

```bash

server {
    listen       443 ssl default_server;
    server_name   <server-realname.cn>;
    
    ssl_certificate     <crtfile-file-path>.crt;
    ssl_certificate_key <keyfilepath>.key;
    ssl_session_timeout 5m;
  
    ssl_ciphers ECDHE-RSA-AES256-SHA384:AES256-SHA256:HIGH:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!AESGCM;
    ssl_protocols TLSv1.2;
    ssl_prefer_server_ciphers on;

    gzip on;  
    gzip_min_length 1k;  
    gzip_buffers 16 64k;  
    gzip_http_version 1.1;  
    gzip_comp_level 6;  
    gzip_types text/plain application/json application/javascript application/x-javascript text/css application/xml;  
    gzip_vary on;

    charset utf-8;
    #access_log  /var/log/nginx/host.access.log  main;

    client_max_body_size 50m;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
#    add_header Content-Security-Policy "default-src 'self'";
    add_header Content-Security-Policy "default-src 'self' https: 'unsafe-inline' 'unsafe-eval'; connect-src https:; font-src https: data:; img-src https: data:;";
    add_header X-Xss-Protection "1; mode=block";
    add_header X-Content-Type-Options nosniff;
    add_header Cache-Control no-store;
    add_header X-Frame-Options SAMEORIGIN;
    #add_header X-Frame-Options ALLOW-FROM https://<allow-form-domain>;
    server_tokens off;
    
    location /api/some-proxy-pass/ {

        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header Content-Security-Policy "default-src https: 'unsafe-inline' 'unsafe-eval'; connect-src https:; font-src https: data:; img-src https: data:;";
        add_header Content-Security-Policy "default-src 'self'";
        add_header X-Xss-Protection "1; mode=block";
        add_header X-Content-Type-Options nosniff;
        add_header Cache-Control no-store;
        add_header X-Frame-Options SAMEORIGIN;

        #proxy_http_version 1.1;
        proxy_set_header Connection "";
        #proxy_set_header HOST $host:$server_port;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://ota-mag;

    }
    

    location / {
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header Content-Security-Policy "default-src 'self' https: 'unsafe-inline' 'unsafe-eval'; connect-src https:; script-src 'self' 'unsafe-inline' 'unsafe-eval' blob: https://*.amap.com https: ;font-src https: data:; img-src https: data:;"
        add_header X-Xss-Protection "1; mode=block";
        add_header X-Content-Type-Options nosniff;
        add_header Cache-Control no-store;
        add_header X-Frame-Options SAMEORIGIN;
        
        root   /work/nginx/html;
        index  index.html index.htm;
    }
}
```

差不多是酱紫的。

## 其他设置

### 超时设置

nginx 可以针对单个域名请求做出单个连接超时的配置。比如些动态解释和静态解释可以根据业务的需求配置。

- proxy_connect_timeout : 后端服务器连接的超时时间, 发起握手等候响应超时时间
- proxy_read_timeout: 连接成功后_等候后端服务器响应时间, 其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
- proxy_send_timeout : 后端服务器数据回传时间, 就是在规定时间之内后端服务器必须传完所有的数据

nginx 使用 proxy 模块时，默认的读取超时时间是 60s。

#### 1、请求超时

```conf
http {
    include       mime.types;
    server_names_hash_bucket_size  512;     
    default_type  application/octet-stream;
    sendfile        on;

    keepalive_timeout  65;  #保持
    tcp_nodelay on;
    client_header_timeout 15;
    client_body_timeout 15;
    send_timeout 25;
    include vhosts/*.conf;
}
```

#### 2、后端服务器处理请求的时间设置（页面等待服务器响应时间）

```conf
location / {
    proxy_read_timeout 150;  # 秒
}
```

### nginx 常用的超时配置说明

client_header_timeout

语法 client_header_timeout time
默认值 60s
上下文 http server

说明 指定等待client发送一个请求头的超时时间（例如：GET / HTTP/1.1）.仅当在一次read中，没有收到请求头，才会算成超时。如果在超时时间内，client没发送任何东西，nginx返回HTTP状态码408(“Request timed out”)

client_body_timeout

语法 client_body_timeout time
默认值 60s
上下文 http server location
说明 该指令设置请求体（request body）的读超时时间。仅当在一次readstep中，没有得到请求体，就会设为超时。超时后，nginx返回HTTP状态码408(“Request timed out”)

keepalive_timeout

语法 keepalive_timeout timeout [ header_timeout ]
默认值 75s
上下文 http server location
说明 第一个参数指定了与client的keep-alive连接超时时间。服务器将会在这个时间后关闭连接。可选的第二个参数指定了在响应头Keep-Alive: timeout=time中的time值。这个头能够让一些浏览器主动关闭连接，这样服务器就不必要去关闭连接了。没有这个参数，nginx不会发送Keep-Alive响应头（尽管并不是由这个头来决定连接是否“keep-alive”）
两个参数的值可并不相同

> 注意不同浏览器怎么处理 `keep-alive` 头

- MSIE 和 Opera 忽略掉 `Keep-Alive: timeout=<N>` header.
- MSIE 保持连接大约 60-65 秒，然后发送 TCP RST
- Opera 永久保持长连接
- Mozilla keeps the connection alive for N plus about 1-10 seconds.
- Konqueror保持长连接N秒

lingering_timeout

语法 lingering_timeout time
默认值 5s
上下文 http server location
说明 lingering_close生效后，在关闭连接前，会检测是否有用户发送的数据到达服务器，如果超过lingering_timeout时间后还没有数据可读，就直接关闭连接；否则，必须在读取完连接缓冲区上的数据并丢弃掉后才会关闭连接。

resolver_timeout

语法 resolver_timeout time
默认值 30s
上下文 http server location
说明 该指令设置DNS解析超时时间

proxy_connect_timeout

语法 proxy_connect_timeout time
默认值 60s
上下文 http server location
说明 该指令设置与upstream server的连接超时时间，有必要记住，这个超时不能超过75秒。
这个不是等待后端返回页面的时间，那是由proxy_read_timeout声明的。如果你的upstream服务器起来了，但是hanging住了（例如，没有足够的线程处理请求，所以把你的请求放到请求池里稍后处理），那么这个声明是没有用的，由于与upstream服务器的连接已经建立了。

proxy_read_timeout

语法 proxy_read_timeout time
默认值 60s
上下文 http server location
说明 该指令设置与代理服务器的读超时时间。它决定了nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading操作的时间。

proxy_send_timeout

语法 proxy_send_timeout time
默认值 60s
上下文 http server location
说明 这个指定设置了发送请求给upstream服务器的超时时间。超时设置不是为了整个发送期间，而是在两次write操作期间。如果超时后，upstream没有收到新的数据，nginx会关闭连接

proxy_upstream_fail_timeout（fail_timeout）

语法 server address [fail_timeout=30s]
默认值 10s
上下文 upstream
说明 Upstream 模块下 server 指令的参数，设置了某一个upstream后端失败了指定次数（max_fails）后，该后端不可操作的时间，默认为10秒

其他：

- [openssl 、nginx生成配置自签名证书](https://chengchaos.github.io/2020/07/16/nginx-with-ssl.html)
- [理解 Nginx 的配置文件](https://chengchaos.github.io/2020/07/01/understand-nginx-config.html)

EOF

---

Power by TeXt.
