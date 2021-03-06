---
title: 微服务架构 & Spring Cloud   
date: 2019-11-10 10:54:06  
tags:  
- 微服务架构  
- Spring Cloud
---

### 概述  
#### 什么是微服务？
> 微服务的概念源于2014年3月Martin Fowler所写的一篇文章“Microservices”。  
**微服务架构：** 是一种架构模式，它提倡将单一应用程序划分成一组小的服务，服务之间互相协调、互相配合，为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务间采用**轻量级的通信机制**互相沟通（通常是基于HTTP的RESTful API）。  
**微服务：** 是一种架构风格，一个大型复杂软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是**松耦合**的。

#### 微服务架构的优势
- **复杂度可控：**  
> 每一个微服务专注于**单一功能**，并通过定义良好的接口清晰表述服务边界。由于体积小、复杂度低，每个微服务可由一个小规模开发团队完全掌控，易于保持高可维护性和开发效率。  
- **独立部署：**  
> 由于微服务具备**独立的运行进程**，所以每个微服务也可以独立部署。  
- **技术选型灵活：**  
> 微服务架构下，技术选型是去中心化的。每个团队可以根据自身服务的需求和行业发展的现状，自由选择最适合的技术栈。
- **容错：**  
> 在微服务架构下，故障会被隔离在单个服务中。若设计良好，其他服务可通过**重试、平稳退化**等机制实现应用层面的容错。
- **扩展**  
> 微服务架构在扩展功能方面具有灵活性，因为每个服务可以**根据实际需求独立进行扩展**

#### 什么是SpringCloud？
> SpringCloud是**微服务架构的集大成者**，将一系列优秀的组件进行了整合。SpringCloud的组件相当繁杂，拥有诸多子项目。重点关注**Netflix**


### Spring Cloud的核心组件    
![image](https://note.youdao.com/yws/api/personal/file/D47A17F9F0054D368770574184DB81FB?method=download&shareKey=56a5caadbcea630f69ae65cc14c960cd)  