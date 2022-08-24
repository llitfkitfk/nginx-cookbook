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
  - [2.3 UDP 负载均衡](#23-udp-负载均衡)
  - [2.4 负载均衡方法](#24-负载均衡方法)
- [3. 流量管理](#3-流量管理)
  - [3.1 A/B 测试](#31-ab-测试)
  - [3.2 限制连接数](#32-限制连接数)
  - [3.3 限制速率](#33-限制速率)
  - [3.4 限制带宽](#34-限制带宽)

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
  需要在/etc/nginx/nginx.conf 文件内添加 stream 模块

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

#### 2.3 UDP 负载均衡

```
# 文件: /etc/nginx/stream.conf.d/udp_load_balancing_test.conf

upstream ntp {
    server ntp1.gokit.info:456   weight=2;
    server ntp1.gokit.info:456;
}
server {
    listen 456 udp;
    proxy_pass ntp;
}

```

#### 2.4 负载均衡方法

- Round robin - 轮询 (默认)

```
upstream backend {
    server backend1.gokit.info;
    server backend2.gokit.info;
}
```

- Least connections - 最少连接

```
upstream backend {
    least_conn;
    server backend1.gokit.info;
    server backend2.gokit.info;
}
```

- IP hash - IP 散列

```
upstream backend {
    ip_hash;
    server backend1.gokit.info;
    server backend2.gokit.info;
}
```

### 3. 流量管理

#### 3.1 A/B 测试

```
http {
    split_clients "${remote_addr}" $site_root_folder {
        20% "/var/www/site1"
        *   "/var/www/site2"
    }
    server {
        listen 80;
        server_name static-test.gokit.info;
        root $site_root_folder;
        location / {
            index index.html;
        }
    }

    split_clients "${remote_addr}AAA" $variant {
        20% "backendv2";
        *   "backendv1";
    }
    server {
        listen 80;
        server_name proxy-test.gokit.info;
        location / {
            proxy_pass http://$variant;
        }
    }
}
```

#### 3.2 限制连接数

```
http {
    limit_conn_zone $binary_remote_addr zone=limitbyaddr:16m;
    limit_conn_status 429; # 429 Too Many Requests
    server {
        limit_conn limitbyaddr 40;
    }
}
```

#### 3.3 限制速率

```
http {
    limit_req_zone $binary_remote_addr zone=limitbyaddr:16m rate=4r/s;
    limit_req_status 429;
    server {
        limit_req zone=limitbyaddr burst=16 delay=8;
    }
}
```

#### 3.4 限制带宽

```
location /download/ {
    limit_rate_after 16m;
    limit_rate 1m; # 每秒1m
}
```
