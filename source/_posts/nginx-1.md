---
title: nginx 速记1
date: 2016-9-10
desc: nginx
---

示例:
``` conf
http {
	
	log_format upstream_time '$remote_addr - $remote_user [$time_local] '
                             '"$request" $status $body_bytes_sent '
                             '"$http_referer" "$http_user_agent"'
                             'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time" urt="$upstream_response_time"';
    server {

    	access_log /spool/logs/nginx-access.log upstream_time;
    	error_log logs/error.log warn;
		listen 80 backlog 4096;
		server_name example.org www.example.org;

		location /images/ {
        	root /data;
        	index index.html;
        	sendfile   on;
    		tcp_nopush on;
    	}

    	location ~ \.php {
    		proxy_set_header Host $host;
    		proxy_set_header X-Real-IP $remote_addr;
    		proxy_set_header Accept-Encoding "";
    		proxy_buffers 16 4k;
    		proxy_buffer_size 2k;
		    proxy_pass http://localhost:8000;
		}
    }
}

```

## web服务器


http 服务
server 虚拟机
location 路径
listen 监听端口 backlog 堆大小

## 静态资源

location 路径(正则表达式):
root 目录
index  默认页面

上传文件设置:
sendfile           on;
sendfile_max_chunk 1m;

## 代理

针对路径
proxy_pass 转发地址
proxy_set_header 设置头信息 
Ｈost 域名
X-Real-IP 客户端地址
proxy_buffers off/on
proxy_buffers 缓存
proxy_buffer_size 缓存大小

## 日志

针对主机
error_log
access_log

查看监听:
netstat -Lan