---
title: 使用 Nexus 搭建 Maven 私服   
date: 2019-11-6 18:04:15  
tags:  
- Nexus  
- Docker
- Maven
---

### 简介  
> Nexus 是一个强大的**仓库管理器**，极大地简化了内部仓库的维护和外部仓库的访问。    

### 部署 Nexus  
#### 部署环境  
| 操作系统  | Ubuntu Server 16\.04 LTS |
|-----|---------------|
| cpu | 2核            |
| 内存  | G          |

#### 开始部署
> Docker 镜像：https://hub.docker.com/r/sonatype/nexus3  

```
# 拉取镜像  
docker pull sonatype/nexus3  

### docker-compose.yml  
version: '3.1'
services:
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    ports:
      - 8081:8081
    volumes:
      - /usr/local/docker/nexus/data:/nexus-data
###
```

### 开始使用  
> 服务启动后第一次登陆需要输入密码，初始密码在服务器文件 nexus/data/admin.password 中。  

#### 配置认证信息
```
# 在maven配置文件中添加   
<server>
  <id>nexus-releases</id>
  <username>admin</username>
  <password>adminpassowrd</password>
</server>

<server>
  <id>nexus-snapshots</id>
  <username>admin</username>
  <password>adminpassowrd</password>
</server>
```

#### 配置自动化部署

```
# 项目的pom中添加
<distributionManagement>  
  <repository>  
    <id>nexus-releases</id>  
    <name>Nexus Release Repository</name>  
    <url>http://192.168.213.128:8081/repository/maven-releases/</url>  
  </repository>  
  <snapshotRepository>  
    <id>nexus-snapshots</id>  
    <name>Nexus Snapshot Repository</name>  
    <url>http://192.168.213.128:8081/repository/maven-snapshots/</url>  
  </snapshotRepository>  
</distributionManagement> 
```


#### 上传第三方 JAR 包  

```
# 如第三方JAR包：kaptcha-2.3.0.jar
mvn deploy:deploy-file 
  -DgroupId=com.google.code.kaptcha 
  -DartifactId=kaptcha
  -Dversion=2.3.0 
  -Dpackaging=jar 
  -Dfile=D:\codes\kaptcha-2.3.0.jar 
  -Durl=http://192.168.213.128:8081/repository/maven-releases/
  -DrepositoryId=nexus-releases
```

#### 配置代理仓库  

```
<repositories>
    <repository>
        <id>nexus-releases</id>
        <name>Nexus Repository</name>
        <url>http://192.168.213.128:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>nexus-releases</id>
        <name>Nexus Plugin Repository</name>
        <url>http://192.168.213.128:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </pluginRepository>
</pluginRepositories>
```
