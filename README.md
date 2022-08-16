# Nginx使用手册

## 目录
- [1. Nginx基础](#1-nginx基础)
	- [1.1 安装](#11-简单性)
	- [1.2 相关文件](#12-相关文件)
	- [1.3 命令](#13-命令)
    - [1.4 静态文件服务](#13-静态文件服务)

### 1. Nginx基础


#### 安装

```
# 安装nginx包
apt-get install -y nginx 
# 开机启动
systemctl enable nginx 
# 开启nginx服务
systemctl start nginx 
```
* docker容器
```
docker run -d -p 80:80 -p 443:443 --name nginx nginx
```

#### 相关文件

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

#### 命令

```
# 测试配置文件
nginx -t
# 动态命令 stop/quit/reload/reopen
nginx -s <signal>
# 动态重载
nginx -s reload
```

#### 静态文件服务

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


