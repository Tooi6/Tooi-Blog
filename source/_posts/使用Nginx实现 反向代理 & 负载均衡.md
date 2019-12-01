---
title: 使用Nginx实现 反向代理 & 负载均衡  
date: 2019-11-24 09:09:15  
tags:  
- Nginx
- 负载均衡
- 反向代理
---

### 什么是 Nginx？  
> Nginx 是一个很强大的高性能Web和反向代理服务,在连接高并发的情况下，Nginx是Apache服务不错的替代品。能够支持高达 50,000 个并发连接数的响应。

### Nginx 的应用场景

#### Http 代理（正向代理、反向代理）
- **什么是代理服务器？**
> 代理服务器，客户机在发送请求时，不会直接发送给目的主机，而是先发送给代理服务器，代理服务接受客户机请求之后，再向主机发出，并接收目的主机返回的数据，存放在代理服务器的硬盘中，再发送给客户机。  
- **正向代理和反向代理？**  
> **正向代理**，架设在客户机与目标主机之间，只用于代理内部网络对 Internet 的连接请求，客户机必须指定代理服务器,并将本来要直接发送到 Web 服务器上的 Http 请求发送到代理服务器中。  

> **反向代理**服务器架设在服务器端，通过缓冲经常被请求的页面来缓解服务器的工作量，将客户机请求转发给内部网络上的目标服务器；并将从服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器与目标主机一起对外表现为一个服务器。    

![image](https://note.youdao.com/yws/api/personal/file/13C7AE322B474D6C9F2C0EDBCF791A60?method=download&shareKey=ae96160af45f2e89cf8925ce3e4a3d5f)  

#### 负载均衡  
> 负载均衡，英文名称为 Load Balance，其意思就是**分摊到多个操作单元上进行执行**，例如 Web 服务器、FTP 服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务  
Nginx提供的负载均衡策略有2种：**内置策略和扩展策略**。内置策略为轮询，加权轮询，Ip hash。

### DEMO  

#### 安装、运行 Nginx 

```
# 新建 docker-compose.yml 文件 
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    ports:
      - 80:8080
      - 81:8081
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./wwwroot:/usr/local/docker/nginx/wwwroot
### end file

# 新建 ./wwwroot 目录 和 ./conf/nginx.conf 文件
```


#### 代理 tomcat 服务器
> 启动两个 tomcat ，用 Nginx 反向代理 tomcat 

```
# 使用docker-compose.yml启动 tomcat
version: '3'
services:
  tomcat1:
    image: tomcat
    container_name: tomcat1
    ports:
      - 8080:8080

  tomcat2:
    image: tomcat
    container_name: tomcat2
    ports:
      - 8081:8080
# 启动
docker-compose up -d

# 修改 ./conf/nginx.conf 配置文件
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream tomcatServer1 {
        server 192.168.213.128:8080;
    }
    upstream tomcatServer2 {
        server 192.168.213.128:8081;
    }
    # 配置一个虚拟主机
    server {
        listen 8080;
        server_name tomcat1;
        location / {
            proxy_pass http://tomcatServer1;
            index index.jsp index.html index.htm;
        }
    }
    server {
        listen 8081;
        server_name tomcat2;
        location / {
            proxy_pass http://tomcatServer2;
            index index.jsp index.html index.htm;
        }
    }
}

# 启动 Nginx
docker-compose up

# 测试，访问 http://NginxIp:80 
```

#### 如何配置负载均衡？  

```
# ./conf/nginx.conf 文件upstream下配置

# 定义负载均衡设备的 Ip及设备状态 
upstream myServer {
    server 127.0.0.1:9090 down;
    server 127.0.0.1:8080 weight=2;
    server 127.0.0.1:6060;
    server 127.0.0.1:7070 backup;
}

# upstream：每个设备的状态:
# down：表示当前的 server 暂时不参与负载
# weight：默认为 1 weight 越大，负载的权重就越大。
# max_fails：允许请求失败的次数默认为 1 当超过最大次数时，返回 proxy_next_upstream 模块定义的错误
# fail_timeout:max_fails 次失败后，暂停的时间。
# backup：其它所有的非 backup 机器 down 或者忙的时候，请求 backup 机器。所以这台机器压力会最轻

# 在需要使用负载的 Server 节点下添加
proxy_pass http://myServer;
```