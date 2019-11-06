---
title: 使用 Docker Registry 搭建 Docker 私服   
date: 2019-11-6 21:37:52  
tags:  
- Docker Registry  
- Docker  
---

### 概述  
> Docker官方提供了Docker Hub来维护管理所有的镜像,只是对于免费用户而言,只能创建一个私有仓库,付费用户才拥有更多私有仓库的权限,对此**官方开源**了Docker Registry的源代码,我们可以通过它在局域网内部搭建私有的镜像注册中心.

### 部署和使用 Docker Registry  
#### 部署环境  
| 操作系统  | Ubuntu Server 16\.04 LTS |
|-----|---------------|
| cpu | 2核            |
| 内存  | 2G          |

#### 部署  
> Docker 镜像：https://hub.docker.com/_/registry  

```
### docker-compose.yml  
version: '3.1'
services:
  registry:
    image: registry
    restart: always
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry/data:/var/lib/registry
###

# 运行  
docker-compose up

# 测试 访问: http://ip:5000/v2/
```

#### 客户端使用 Registry 私服  

```
### 修改 /etc/docker/daemon.json 文件（没有则添加），加速器可以换成阿里云
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    "ip:5000"
  ]
}
###

# 重启
systemctl daemon-reload
systemctl restart docker

# 查看配置是否成功
docker info

# 上传本地镜像到 Registry
## 标记本地镜像库（ip:port/image_name:tag，该格式为标记版本号）
docker tag tomcat 192.168.213.128:5000/tomcat:8.5.34

## 提交镜像到仓库
docker push 192.168.213.128:5000/tomcat:8.5.34

# 查看全部镜像
curl -XGET http://192.168.213.128:5000/v2/_catalog

# 查看指定镜像
curl -XGET http://192.168.213.128:5000/v2/nginx/tags/list

# 从 Registry 拉去镜像  
docker pull 192.168.213.128:5000/nginx
```

### 部署 Docker Registry WebUI 
> 给 Registry 安装一个UI界面  
镜像：https://hub.docker.com/r/konradkleine/docker-registry-frontend


```
### docker-compose.yml 
version: '3.1'
services:
  registry:
    image: registry
    restart: always
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry/data:/var/lib/registry
  frontend:
    image: konradkleine/docker-registry-frontend:v2
    ports:
      - 8080:80
    volumes:
      - ./certs/frontend.crt:/etc/apache2/server.crt:ro
      - ./certs/frontend.key:/etc/apache2/server.key:ro
    environment:
      - ENV_DOCKER_REGISTRY_HOST=192.168.213.128
      - ENV_DOCKER_REGISTRY_PORT=5000
###

# 启动
docker-compose up

# 访问：http://192.168.213.128:8080/
```
