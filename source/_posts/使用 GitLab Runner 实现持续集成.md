---
title: 使用 GitLab Runner 实现持续集成  
date: 2019-11-16 21:18:19  
tags:  
- 持续集成
- GitLab Runner
---

### 概述  
#### 什么是持续集成、交付、部署？  
- **持续集成：**  
> 持续集成强调开发人员提交了新代码之后，**立刻进行构建、（单元）测试**。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。
- **持续交付：**
> 持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的 **类生产环境（production-like environments）** 中。如果代码没有问题，可以继续手动部署到生产环境中。
- **持续部署：**
> 持续部署则是在持续交付的基础上，把**部署到生产环境的过程自动化**    
![image](https://note.youdao.com/yws/api/personal/file/31BFA0F6FF4D4BB3A5CC30B1F301E8FB?method=download&shareKey=3662d215358c3e875a367f453f5ab3ef)

#### 持续集成的流程
- **提交**
> 开发者想代码仓库提交代码
- **测试（第一轮）：**
> 代码仓库对 commit 操作配置了钩子（hook），只要提交代码或者合并进主干，就会跑自动化测试
- **构建**
> 通过第一轮测试，代码就可以合并进主干，就算可以交付了。交付后，就先进行构建（build），再进入第二轮测试。所谓构建，指的是**将源码转换为可以运行的实际代码**，比如安装依赖，配置各种资源（样式表、JS脚本、图片）等等
- **测试（第二轮）**  
> 第二轮是**全面测试**，单元测试和集成测试都会跑，有条件的话，也要做端对端测试。所有测试以自动化为主，少数无法自动化的测试用例，就要人工跑。  
- **部署**
> 通过了第二轮测试，当前代码就是一个可以直接部署的版本（artifact）。将这个版本的所有文件打包（ tar filename.tar * ）存档，发到生产服务器。
- **回滚**
> 一旦当前版本发生问题，就要回滚到上一个版本的构建结果。最简单的做法就是修改一下符号链接，指向上一个版本的目录。

### 使用GitLab持续集成  

#### Docker 部署GitLab Runner

- **环境准备**

```
# 创建目录
mkdir /usr/local/docker/runner
mkdir /usr/local/docker/runner/environment

# 下载jdk到 environment 目录下
# 在 environment 下新建 daemon.json 配置加速器和仓库地址  
{
  "registry-mirrors": [
    "https://kzzvxj0z.mirror.aliyuncs.com"
  ],
  "insecure-registries": [
    "192.168.213.132:5000"
  ]
}
```

- **Dockerfile**

```
# 在 environment 目录下新建 Dockfile

FROM gitlab/gitlab-runner:v11.0.2
MAINTAINER Lusifer <topsale@vip.qq.com>

# 修改软件源
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse' > /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse' >> /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse' >> /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse' >> /etc/apt/sources.list && \
    apt-get update -y && \
    apt-get clean

# 安装 Docker
RUN apt-get -y install apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" && \
    apt-get update -y && \
    apt-get install -y docker-ce
COPY daemon.json /etc/docker/daemon.json

# 安装 Docker Compose
WORKDIR /usr/local/bin
RUN wget https://raw.githubusercontent.com/topsale/resources/master/docker/docker-compose
RUN chmod +x docker-compose

# 安装 Java
RUN mkdir -p /usr/local/java
WORKDIR /usr/local/java
COPY jdk-8u152-linux-x64.tar.gz /usr/local/java
RUN tar -zxvf jdk-8u152-linux-x64.tar.gz && \
    rm -fr jdk-8u152-linux-x64.tar.gz

# 安装 Maven
RUN mkdir -p /usr/local/maven
WORKDIR /usr/local/maven
RUN wget https://raw.githubusercontent.com/topsale/resources/master/maven/apache-maven-3.5.3-bin.tar.gz
# COPY apache-maven-3.5.3-bin.tar.gz /usr/local/maven
RUN tar -zxvf apache-maven-3.5.3-bin.tar.gz && \
    rm -fr apache-maven-3.5.3-bin.tar.gz
# COPY settings.xml /usr/local/maven/apache-maven-3.5.3/conf/settings.xml

# 配置环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_152
ENV MAVEN_HOME /usr/local/maven/apache-maven-3.5.3
ENV PATH $PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin

WORKDIR /
```

- **开始构建**  

```
# 在 runner 目录下新建docker-compose.yml  
version: '3.1'
services:
  gitlab-runner:
    build: environment
    restart: always
    container_name: gitlab-runner
    privileged: true
    volumes:
      - /usr/local/docker/runner/config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock

# 开始构建
docker-compose up -d
```

- **注册Runner**
> 先在 项目>设置>CI/CD>Runner 下查看：  
![image](https://note.youdao.com/yws/api/personal/file/050A604056834955BFB6CBBDEC3DD364?method=download&shareKey=9b94fca8ff0a6b19bf4b873c38ed0bcb)

```
# 注册Runner
docker exec -it gitlab-runner gitlab-runner register

# 输入 GitLab 地址
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.213.133:8080/

# 输入 GitLab Token
Please enter the gitlab-ci token for this runner:
ui6B3hj28Biv99ssyTJz

# 输入 Runner 的说明
Please enter the gitlab-ci description for this runner:
可以为空

# 设置 Tag，可以用于指定在构建规定的 tag 时触发 ci
Please enter the gitlab-ci tags for this runner (comma separated):
deploy


# 选择 runner 执行器，这里我们选择的是 shell
Please enter the executor: virtualbox, docker+machine, parallels, shell, ssh, docker-ssh+machine, kubernetes, docker, docker-ssh:
shell
```

#### 配置项目   
> 以 iCoin-eureka 为例子：

- **新建 .gitlab-ci.yml 文件**

```
stages:
  - build
  - push
  - run
  - clean

build:
  stage: build
  script:
    - /usr/local/maven/apache-maven-3.5.3/bin/mvn clean package   # mvn 打包
    - cp target/icoin-eureka-1.0.0-SNAPSHOT.jar docker            # 取出jar包
    - cd docker                                                   
    - docker build -t 192.168.213.132:5000/icoin-eureka .         # 构建镜像

push:
  stage: push
  script:
    - docker push 192.168.213.132:5000/icoin-eureka               # push 到register

run:
  stage: run
  script:
    - cd docker
    - docker-compose down                                        # 先关闭之前的
    - docker-compose up -d                                       # 启动
clean:
  stage: clean
  script:
    - docker rmi $(docker images -q -f dangling=true)            # 清理

```

- **新建Dockerfile**  
> 使用 dockerize 等待 icoin-config 启动再运行
```
FROM openjdk:8-jre
MAINTAINER Tooi <1348562497@qq.com>

ENV APP_VERSION 1.0.0-SNAPSHOT

ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz
#COPY dockerize-alpine-linux-amd64-v0.6.1.tar.gz /
#
#RUN tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-v0.6.1.tar.gz \
#	&& rm dockerize-alpine-linux-amd64-v0.6.1.tar.gz

RUN mkdir /app

COPY icoin-eureka-$APP_VERSION.jar /app/app.jar

ENTRYPOINT ["dockerize", "-timeout", "5m", "-wait", "http://192.168.213.129:8888/icoin-eureka/prod", "java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app/app.jar", "--spring.profiles.active=prod"]

EXPOSE 8888
```

- **新建 docker-conpose.yml**

```
version: '3.1'
services:
  icoin-eureka:
    restart: always
    image: 192.168.213.132:5000/icoin-eureka
    container_name: icoin-eureka
    ports:
      - 8761:8761
    networks:
      - eureka_network

networks:
  eureka_network:
```

