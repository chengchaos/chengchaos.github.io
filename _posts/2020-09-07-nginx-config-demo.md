---
title: Nginx 的配置文件示例
key: 2020-09-07
tags: nginx
---


Nginx 的配置文件中和安全有关的内容示例。



<!--more-->

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

其他：

- [openssl 、nginx生成配置自签名证书](https://chengchaos.github.io/2020/07/16/nginx-with-ssl.html)
- [理解 Nginx 的配置文件](https://chengchaos.github.io/2020/07/01/understand-nginx-config.html)

EOF

---

Power by TeXt.
