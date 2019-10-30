---
title: GitFlow 工作流   
date: 2019-10-30 22:40:51  
tags:  
- GitFlow 工作流  
- Tortoise Git  
---
### Tortoise Git  
> 官网下载：https://tortoisegit.org/download/  

### Git几种主流的工作流
#### 集中式工作流  
> 所有的**功能开发与修改都在 master 分支上进行**的。开发者开始先克隆中央仓库。在自己的项目拷贝中像SVN一样的编辑文件和提交修改；但修改是存在本地的，和中央仓库是完全隔离的。开发者可以把和上游的同步延后到一个方便时间点。  
![image](https://note.youdao.com/yws/api/personal/file/64CEFF1425774282A82F1E00D0FA8649?method=download&shareKey=5833170a020b4bb4651c41f983e47b37)  

#### 功能分支工作流  
> 功能分支工作流以集中式工作流为基础，不同的是**为各个新功能分配一个专门的分支来开发**。功能分支工作流背后的核心思路是所有的功能开发应该在一个专门的分支，而不是在 master 分支上。  
![image](https://note.youdao.com/yws/api/personal/file/B1701B5FAB8140FEA44B3AD7372F114E?method=download&shareKey=c879c215afd63fdabfb4884b11cb9124)  

#### GitFlow 工作流  
> GitFlow 工作流通过为**功能开发、发布准备**和**维护分配**独立的分支，让发布迭代过程更流畅。严格的分支模型也为大型项目提供了一些非常必要的结构。
![image](https://note.youdao.com/yws/api/personal/file/FB06F01E87C044949FABB9CDDE711F87?method=download&shareKey=a8fe1902c3daa4a33ea992469f5fa276)

#### Forking 工作流  
> Forking 工作流是分布式工作流，充分利用了 Git 在分支和克隆上的优势。可以安全可靠地管理大团队的开发者（developer），并能接受不信任贡献者（contributor）的提交。  
![image](https://note.youdao.com/yws/api/personal/file/C89D9EEB7E08493380A8964FE4424739?method=download&shareKey=432315df4ddd4ba6ae44d3ee9af5724a)

#### 流程解析  

- **master分支：** 存放所有**正式发布的版本**，可以作为项目**历史版本记录分支**，不直接提交代码。仅用于保持一个对应线上运行代码的 code base。
- **develop分支：** 为**主开发分支**，一般不直接提交代码  
- **feature分支：** 为**新功能分支**，feature分支都是基于develop创建的，开发完成后会合并到develop分支上。
- **release分支：** 为**发布分支**，基于最新develop分支创建。
- **hotfix分支：** **基于master分支创建**，对线上版本的**bug进行修复**，完成后直接合并到master分支和develop分支，如果当前还有新功能release分支，也同步到release分支上。