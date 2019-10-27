---
title: 使用Hexo+Github搭建博客
date: 2018-12-01 01:06:34
tags:
- Hexo
- Github
---

### 准备工作  
- 创建仓库，创建一个名为 <你的用户名>.github.io 的仓库  
- 安装软件  

```
# 下载、安装、配置 git
官网下载：https://www.git-scm.com/download/   
## 配置ssh key

# 下载、安装 node.js  
https://nodejs.org/en/download/ 

# 安装 hexo  
npm install -g hexo  
```

### 开始搭建  

```
# 初始化（必须在空文件夹内，使用git bash运行）  
hexo init 
# 清理生成内容
hexo clean 
# 生成
hexo g 
# 启动服务
hexo s  

# 修改主题
## 到官网下载主题 https://hexo.io/themes/  
## 下载后解压到 themes 目录下  
## 修改配置文件_config.yml  
- theme: landscape
+ theme: suka  

# 上传到github
## 配置_config.yml中deploy部分
deploy:
  type: git
  repository: git@github.com:Tooi6/Tooi6.github.io.git
  branch: master  
## 安装插件（不安装提示Deployer not found: git错误）
npm install hexo-deployer-git --save
## 上传
hexo d
```

### 常用的hexo命令  

```
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
```
