---
title: Docker 容器技术 & Docker Compose   
date: 2019-11-3 21:03:2  
tags:  
- Docker  
- Docker Compose
---

### 简介
#### 什么是Docker？  
> Docker 是一个开源的**应用容器引擎**，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现**虚拟化**。容器是完全使用**沙箱机制**，相互之间不会有任何接口。  

#### 虚拟机和容器的区别？  
> **虚拟机：** 传统的虚拟机需要模拟整台机器包括硬件，每台虚拟机都需要有**自己的操作系统**，虚拟机一旦被开启，**预分配给它的资源将全部被占用**  

> **容器：** 容器技术是和我们的宿主机**共享硬件资源及操作系统**，可以实现**资源的动态分配**。容器包含应用和其所有的依赖包，但是与其他容器共享内核。容器在宿主机操作系统中，在用户空间以**分离的进程运行。**    

#### 容器技术的优势  
| 特性    | 容器        | 虚拟机    |
|-------|-----------|--------|
| 启动    | 秒级        | 分钟级    |
| 硬盘使用  | 一般为MB     | 一般为GB  |
| 性能    | 接近原生      | 弱于     |
| 系统支持量 | 单机支持上千个容器 | 一般是几十个 |

### Docker 的基本概念  
- **镜像（Image）：** 
> Docker 镜像是一个**特殊的文件系统**，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。**镜像不包含任何动态数据，其内容在构建之后也不会被改变**
- **容器（Container）：** 
> 容器的实质是**进程**，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个**隔离的环境**里，使用起来，就好像是在一个独立于宿主的系统下操作一样  

**镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。**
- **仓库（Repository）：**  

> Docker 仓库是**集中存放镜像文件的场所**。镜像构建完成后，可以很容易的在当前宿主上运行，但是， 如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry (仓库注册服务器)就是这样的服务  
 官方仓库：https://hub.docker.com/search?q=&type=image   
 
- **Docker引擎**  
> Docker 引擎是一个包含以下主要组件的客户端服务器应用程序（CS架构）。  
**服务器(server)**，它是一种称为**守护进程**并且长时间(后台)运行的程序。  
**REST API**，用于指定程序可以用来与守护进程通信的接口，并指示它做什么。  
**客户端（Client）**，有命令行界面 (CLI) 工具  

![image](https://note.youdao.com/yws/api/personal/file/851252360E394E6A8A63E181ED3E3203?method=download&shareKey=c1c71de21d8043aaa3d033697078fb02)

### 如何使用？  
#### 安装Docker 

```
# 卸载旧版本  
sudo apt-get autoremove docker docker-engine docker.io

# 使用APT安装  
sudo apt-get update 
## 安装必要的工具
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common  
## 安装 GPG 证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -  
## 写入软件源信息  
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"  
## 更新并安装 
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 使用脚本自动安装（推荐）  
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun

# 启动 Docker CE
sudo systemctl enable docker
sudo systemctl start docker

# 添加docker用户组  
groupadd docker    # 一般脚本已经建好了
usermod -aG docker $USER

# 运行hello-world
docker run hello-world
```

#### 配置镜像加速  
> 下面是docker官方提供的，使用阿里云的更快：https://cr.console.aliyun.com/#/accelerator

```
vim /etc/docker/daemon.json  # 没有则新建
## 写入内容
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}

# 重启docker
systemctl daemon-reload
systemctl restart docker
```

#### 基本操作  
```
# 下载镜像  
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签] 
#下载tomcat镜像，默认下载官方的latest标签
docker pull tomcat   

# 查看本地镜像  
docker images

# 运行镜像  
docker run -d -p 8080:8080 tomcat  # -d:后台运行 -p:绑定端口 
# 查看容器  
docker ps
# 停止容器  
docker stop [NAME]
# 其他命令
https://www.runoob.com/docker/docker-command-manual.html
```

### 使用 Dockerfile 定制镜像
#### demo：

```
# 创建Dockerfile
FORM tomcat
WORKDIR /usr/local/tomcat/webapps/ROOT/
RUN "hello Dockerfile" > index.html

# 构建镜像  
docker build -t mytomcat .

# 运行  
docker run -p 80:8080 mytomcat bash  
```

#### 镜像构建上下文（Context）  
> https://blog.csdn.net/small_to_large/article/details/77435541  

#### Dockerfile 指令  
> 官方文档：https://docs.docker.com/engine/reference/builder/

### 使用数据卷构建MySQL  
#### 什么是数据卷？  
> 数据卷 是一个可供一个或多个容器使用的特殊目录，它绕过 **UFS**，可以提供很多有用的特性  
- 数据卷 可以在**容器之间共享和重用**  
- 对 数据卷 的**修改会立马生效**  
- 对 数据卷 的**更新，不会影响镜像**  
- 数据卷 默认会**一直存在**，即使容器被删除 **（MySQL可以用它来实现持久化）**

#### 构建MySQL
> 将本地文件加挂载到MySQL容器上，实现MySQL的持久化

```
# 拉取镜像  
docker pull mysql:5.7.28

# 使用数据卷启动MySQL  
docker run -p 3306:3306 --name mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7.28

## 复制docker容器内容到宿主机  
docker cp mysql:/etc/mysql .  # 可以将MySQL配置负责到本地
```

### Docker Compose  
#### 什么是 Docker Compose
> Compose 项目是 Docker 官方的开源项目，负责实现**对 Docker 容器集群的快速编排。**    
Docker Compose 它允许用户通过一个单独的 **docker-compose.yml** 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

#### 下载、安装  

```
# GitHub地址：https://github.com/docker/compose

# 下载，并安装
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# 验证
docker-compose version

# 卸载，直接删除二进制文件即可  
rm /usr/local/bin/docker-compose
```

#### DEMO
> 

```
# 新建文件docker-compose.yml

### 配置tomcat、MySQL
version: '3'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
    volumes:
      - /usr/local/docker/tomcat/ROOT:/usr/local/tomcat/webapps/ROOT
  mysql:
    restart: always
    image: mysql:5.7.22
    container_name: mysql
    ports:
      - 3306:3306
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 123456
    command:
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
      --max_allowed_packet=128M
      --sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO"
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
###

# 启动
docker-compose up -d
```

### Docker 常用命令  

```
# 查看 Docker 版本
docker version

# 从 Dockerfile 文件构建镜像
docker build -t image-name .

# 运行 Docker 镜像
docker run -d image-name

# 查看Docker镜像
docker images

# 查看运行的容器
docker ps
docker ps -al # 查看所有容器  

# 停止容器  
docker stop conainer_id

# 删除一个镜像  
docker rmi iamge-name

# 删除所有镜像
docker rmi $(docker images -q)

# 强制删除所有镜像
docker rmi -r $(docker images -a)

# 删除所有虚悬镜像
docker rmi $(docker images -q -f dangling=true)

# 删除所有容器
docker rm $(docker ps -a -q)

# 进入 Docker 容器
docker exec -it container-id /bin/bash

# 查看数据卷  
docker volume ls

# 删除指定数据卷
docker volume rm [volume_name]

# 删除所有未关联的数据卷
docker volume rm $(docker volume ls -qf dangling=true)

# 从主机复制文件到容器
sudo docker cp host_path containerID:container_path

# 从容器复制文件到主机
sudo docker cp containerID:container_path host_path
```

