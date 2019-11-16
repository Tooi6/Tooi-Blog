---
title: Python web 之 Django 入门   
date: 2019-11-14 23:02:06  
tags:  
- Python  
- Django
---

### Python 虚拟环境  

#### 为什么使用虚拟环境？  
> 在使用Python语言的时候我们使用pip来安装第三方包，但是由于pip的特性，系统中只能安装每个包的一个版本。但是在实际项目开发中，不同项目可能需要第三方包的不同版本，Python的解决方案就是虚拟环境。顾名思义，虚拟环境就是虚拟出来的一个隔离的Python环境，每个项目都可以有自己的虚拟环境，用pip安装各自的第三方包，不同项目之间也不会存在冲突。

#### 使用

```
# 安装 virtualenvwrapper-win  
pip3 install virtualenvwrapper-win  

# 新建虚拟环境 py3scrapy 
mkvirtualenv py3scrapy

# 进入虚拟环境  
workon py3entest

# 退出
deactivate

# 虚拟环境默认放在 C:\%User%\13485\Envs 目录下 
```

### 概述
#### 什么是 Django？   
> Django是一个开放源代码的Web应用框架，由Python写成。采用了**MTV**的框架模式，即**模型M，视图V和模版T**。它最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站的，即是CMS（内容管理系统）软件。

#### MTV 与 MVC 的区别？
- **MVC**
    - Model：数据存取层，负责业务对象和数据库的对象(ORM)的映射
    - View：视图层，负责与用户的交互(书写逻辑)
    - Controller：完成用户对模型层和视图层调用,来完成用户的请求
- **MTV**
    - Model：数据存取层，负责业务对象与数据库的对象(ORM)的映射
    - Template：负责如何把页面展示给用户(html)
    - View：负责业务逻辑，并在适当的时候调用Model和Template
- **区别：**
> MVC中的View的目的是**呈现哪一个数据**，而MTV的View的目的是**数据如何呈现** 。  
也就说MTV把View分成了**视图（展现哪些数据）** 和 **模板（如何展现）** 2个部分，而Contorller这个要素由框架自己来实现了


### Demo  
> 参考：https://blog.csdn.net/niedongri/article/details/81978284

### 详细教程
> W3School：https://www.w3cschool.cn/django/django-tutorial.html
