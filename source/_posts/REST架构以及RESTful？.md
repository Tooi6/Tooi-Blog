---
title: REST架构以及RESTful？
date: 2019-10-28 23:16:34
tags:
- RESTful
- 架构风格
---
### 介绍  
#### 什么是REST？  
> **REST（Representational State Transfer）** 表象化状态转变（表述性状态转变），在2000年被提出，基于HTTP、URI、XML、JSON等标准和协议，支持**轻量级、跨平台、跨语言**的架构设计。是Web服务的一种新的**架构风格**（一种思想）。 

#### REST架构的主要原则  
- 对网络上所有的资源都有一个**资源标志符**。
- 对资源的操作**不会**改变标识符。
- 同一资源有多种表现形式（xml、json）
- 所有操作都是**无状态**的（Stateless） 

> **无状态性：** 使得客户端和服务器端不必保存对方的详细信息，服务器只需要处理当前的请求，不需了解请求的历史。可以更容易的释放资源，让服务器利用Pool（连接池）技术来提高稳定性和性能。  

#### 什么是RESTful风格？
> RESTful是一种常见的**REST应用**，是遵循REST风格的**web服务**，REST式的web服务是一种ROA（面向资源的架构）。 

### API设计  
#### 设计原则  
- URI 不能包含动词，只能是名词  
> 形容词也是可以使用的，但是尽量少用  
- API 的名词要以复数进行命名  
- 命名名词的时候，要使用小写、数字及下划线来区分多个单词  
> 样的设计是为了与 json 对象及属性的命名方案保持一致  
#### 请求方式

| http方法 | 资源操作   | 幂等 | 安全 |
|--------|--------|----|----|
| GET    | SELECT | 是  | 是  |
| POST   | INSERT | 否  | 否  |
| PUT    | UPDATE | 是  | 否  |
| DELETE | DELETE | 是  | 否  |

#### 状态码  

| 状态码 | 描述      |
|-----|---------|
| 200 | 请求成功    |
| 201 | 创建成功    |
| 400 | 错误的请求   |
| 401 | 未验证     |
| 403 | 被拒绝     |
| 404 | 无法找到    |
| 409 | 资源冲突    |
| 500 | 服务器内部错误 |

#### 请求参数  
- **对请求参数进行限制说明，写明参数要求**
```
【POST】     /v1/users                             // 创建用户信息
请求内容
{
    "username": "Tooi",                 // 必填, 用户名称, max 20
    "password": "123456",              // 必填, 用户密码, max 20
    "email": "tooi997@163.com",     // 选填, 电子邮箱, max 32
    "sex": 1                           // 必填, 用户性别[1-男 2-女 99-未知]
}
```

#### 响应参数
- **单条数据，直接返回json**

```
HTTP/1.1 200 OK
{
    "id" : "01",
    "name" : "Tooi6",
    "created_time": 2019-10-26,
    "updated_time": 2019-10-26,
    ...
}
```

- 多条数据，封装结构体

```
HTTP/1.1 200 OK
{
    "count":100,
    "items":[
        {
            "id" : "01",
            "name" : "Tooi6",
            "created_time": 2019-10-26,
            "updated_time": 2019-10-26,
            ...
        },
        ...
    ]
}
```


#### 异常响应  

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "code": "INVALID_ARGUMENT",
    "message": "{error message}",
    "cause": "{cause message}",
    "request_id": "01234567-89ab-cdef-0123-456789abcdef",
    "host_id": "{server identity}",
    "server_time": "2014-01-01T12:00:00Z"
}
```  



