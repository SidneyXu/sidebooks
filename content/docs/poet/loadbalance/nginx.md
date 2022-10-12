---
weight: 1
title: Nginx
---

# Nginx

## 常用方法

启动

```shell
./nginx -c nginx.conf
```

从容停止

```shell
nginx -s stop
```

验证配置语法

```shell
nginx -t -c nginx.conf
```

重新启动

```shell
nginx -s reload
```

## 配置文件

```
# 设置使用的用户（nobody, www, root 等），使用 nobody 可以提高安全性
user nobody;

#工 作进程数，通常设为 cpu 核心数或其 2 倍
worker_processes  1;

# 设置 pid 存放路径
pid /run//nginx.pid;

# 设置最大连接数
events {
    worker_connections  1024;
}

http {
    # 开启gzip压缩
    gzip  on;

    server {
        listen       80;
        server_name  localhost;

        # 默认采用combined级别
    	access_log /var/log/nginx/access.log;
    	# 默认采用error级别
        error_log /var/log/nginx/error.log;
    }
    location / {
        root   html;
        index  index.html index.htm;
    }
}

# 负载均衡配置
http{
	# 自定义的负载均衡集群，默认负载均衡算法为轮询。
	# 加权轮询为server 118.144.78.52 weight=1
	upstream myproject {
		server 182.18.22.2:80;
		server 118.144.78.52;
	}
	server {
		listen 8080;
		# 反向代理到指定的负载均衡集群
		location / {
			proxy_pass http://myproject;
		}
	}
}
```