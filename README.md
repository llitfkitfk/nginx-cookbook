# Nginx 使用手册

## 目录

- [1. Nginx 基础](#1-nginx-基础)
  - [1.1 安装](#11-安装)
  - [1.2 相关文件](#12-相关文件)
  - [1.3 命令](#13-命令)
  - [1.4 静态文件服务](#14-静态文件服务)
- [2. 负载均衡](#2-负载均衡)
  - [2.1 HTTP 负载均衡](#21-http-负载均衡)
  - [2.2 TCP 负载均衡](#22-tcp-负载均衡)
  
### 1. Nginx 基础

#### 1.1 安装

```
# 安装nginx包
apt-get install -y nginx
# 开机启动
systemctl enable nginx
# 开启nginx服务
systemctl start nginx
```

- docker 容器

```
docker run -d -p 80:80 -p 443:443 --name nginx nginx
```

#### 1.2 相关文件

```
# 默认配置路径
/etc/nginx
# 默认配置文件
/etc/nginx/nginx.conf
# 默认http服务配置文件
/etc/nginx/conf.d/
# 默认日志文件
/var/log/nginx/
```

#### 1.3 命令

```
# 测试配置文件
nginx -t
# 动态命令 stop/quit/reload/reopen
nginx -s <signal>
# 动态重载
nginx -s reload
```

#### 1.4 静态文件服务

```
server {
    listen 80 default_server;
    # www.gokit.info 可以改为其他domain
    server_name www.gokit.info;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
}
```

### 2. 负载均衡

#### 2.1 HTTP 负载均衡

```
# 文件: /etc/nginx/conf.d/http_load_balancing_test.conf
upstream backend {
    server 192.168.100.101:80   weight=1;
    server web.gokit.info:80    weight=2;
    server web2.gokit.info:80   backup;
}
server {
    location / {
        proxy_pass http://backend;
    }
}
```

#### 2.2 TCP 负载均衡

```
# 文件: /etc/nginx/stream.conf.d/tcp_load_balancing_test.conf

upstream redis_read {
    server redis1.gokit.info:6789   weight=5;
    server redis2.gokit.info:6789;
    server 192.168.100.101:6789     backup;
}
server {
    listen 6789;
    proxy_pass redis_read;
}

```

- 注意如果使用 tcp 负载均衡
  需要在/etc/nginx/nginx.conf 文件内添加stream模块

```
user nginx;
...
events {
    ...
}

# 添加以下内容
stream {
    include /etc/nginx/stream.conf.d/*.conf;
}
# 添加以上内容

http {
    ...
}
```
