---
title: GitLab 的使用与搭建   
date: 2019-11-6 18:04:15  
tags:  
- GitLab  
- Docker
---

### 简介  
> GitLab 是一个用于仓库管理系统的开源项目，使用Git作为代码管理工具，并在此基础上搭建起来的web服务。简单的说就是一个私人的GitHub。  

官方文档：https://docs.gitlab.com/ee/README.html

### 开始部署  
#### 部署环境  
| 操作系统  | Ubuntu Server 16\.04 LTS |
|-----|---------------|
| cpu | 2核            |
| 内存  | 2G          |

#### 基于Docker部署  
> Docker镜像：https://hub.docker.com/r/twang2218/gitlab-ce-zh

```
# 拉取镜像  
docker pull twang2218/gitlab-ce-zh

### docker-compose.yml
version: '3'
services:
    web:
      image: 'twang2218/gitlab-ce-zh'
      restart: always
      hostname: '192.168.213.128'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://192.168.213.128:8080'
          gitlab_rails['gitlab_shell_ssh_port'] = 2222
          unicorn['port'] = 8888
          nginx['listen_port'] = 8080
      ports:
        - '8080:8080'
        - '8443:443'
        - '2222:22'
      volumes:
        - /usr/local/docker/gitlab/config:/etc/gitlab
        - /usr/local/docker/gitlab/data:/var/opt/gitlab
        - /usr/local/docker/gitlab/logs:/var/log/gitlab
###

# 启动  
docker-compose up
```
