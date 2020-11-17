---
title: Nginx 跨域配置
key: 2020-11-17
tags: nginx
---

使用SpringBoot开发前后端分离的应用，可以使用Nginx作为网关来统一解决跨域问题。

<!--more-->

```conf
server {
    listen 80;
    server_name localhost 127.0.0.1;
	location / {
		# 允许跨域请求的“域”
		add_header 'Access-Control-Allow-Origin' $http_origin;
		# 允许客户端提交Cookie
		add_header 'Access-Control-Allow-Credentials' 'true';
		# 允许客户端的请求方法
		add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, DELETE, PUT';
		# 允许客户端提交的的请求头
		add_header 'Access-Control-Allow-Headers' 'Origin, x-requested-with, Content-Type, Accept, Authorization';
		# 允许客户端访问的响应头
		add_header 'Access-Control-Expose-Headers' 'Cache-Control, Content-Language, Content-Type, Expires, Last-Modified, Pragma';
		# 处理预检请求
		if ($request_method = 'OPTIONS') {
			# 预检请求缓存时间
			add_header 'Access-Control-Max-Age' 1728000;
			add_header 'Content-Type' 'text/plain; charset=utf-8';
			add_header 'Content-Length' 0;
			return 204;
		}

		proxy_connect_timeout 600;
		proxy_read_timeout 600;

        #proxy_http_version 1.1;
        proxy_set_header Connection "";
        #proxy_set_header HOST $host:$server_port;
        proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto "https";

        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
#        add_header Content-Security-Policy "default-src 'self'";
        add_header Content-Security-Policy "default-src https: 'unsafe-inline' 'unsafe-eval'; connect-src https:; font-src https: data:; img-src
 https: data:;";
        add_header X-Xss-Protection "1; mode=block";
        add_header X-Content-Type-Options nosniff;
        add_header Cache-Control no-store;
        add_header X-Frame-Options SAMEORIGIN;
		# SpringBoot 应用访问路径
		proxy_pass http://127.0.0.1:8080;
	}
}
```

参考：

- [Nginx的跨域配置](https://www.cnblogs.com/kevinblandy/p/13294320.html)

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
