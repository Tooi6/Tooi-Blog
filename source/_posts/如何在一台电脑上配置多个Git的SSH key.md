---
title: 如何在一台电脑上配置多个Git的SSH key  
date: 2019-11-21 17:39:26  
tags:  
- Git
---

### 需求  
> 需要用到多个代码托管平台时，就需要在同一台电脑上配置多个SSH-Key

### 开始配置
> 下面步骤演示配置本地局域网的GitLab 和 Github  
- **生成SSH-Key**

```
# 生成GitLab的ssh key
$ ssh-keygen -t rsa -C "youemail@email.com” -f ~/.ssh/gitlab_rsa

# 生成GitHub的ssh key  
$ ssh-keygen -t rsa -C "youemail@email.com” -f ~/.ssh/github_rsa
```

- **复制 ~/.ssh 目录下的pub key到平台**  
![image](https://note.youdao.com/yws/api/personal/file/6AA860059A7B433BA22CDEFF9DB79A76?method=download&shareKey=6061be696a3fb00cd43bae82619fb497)

- **在 ~/.ssh 目录下添加 config 文件**  

```
# gitlab
Host 192.168.213.133
Port 2222
HostName 192.168.213.133
PreferredAuthentications publickey
IdentityFile C:/Users/13485/.ssh/gitlab_rsa
User Tooi
 
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile C:/Users/13485/.ssh/github_rsa
User Tooi6
```


```

# 配置文件参数
# Host : Host可以看作是一个你要识别的模式，对识别的模式，进行配置对应的的主机名和ssh文件（可以直接填写ip地址）
# HostName : 要登录主机的主机名（建议与Host一致）
# User : 登录名（如gitlab的username）
# IdentityFile : 指明上面User对应的identityFile路径
# Port: 端口号（如果不是默认22号端口则需要指定）
```


- **测试**

```
$ ssh -T git@github.com
```
