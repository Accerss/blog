---
title: Git使用
date: 2022-09-14 21:55:14
tags: [tag]
categories: [cate]
sticky: 9
---
# Git使用
**Git Bash**：Unix与Linux风格的命令行，使用最多，推荐最多
**Git CMD**：Windows风格的命令行
**Git GUI**：图形界面的Git，不建议初学者使用，尽量先熟悉常用命令
所有的配置文件，其实都保存在本地！
查看配置 git config -l

```
#查看系统config
git config --system --list
　　
#查看当前用户（global）配置
git config --global  --lis
```

## 三个区域
Git本地有三个工作区域：工作目录（Working Directory）、暂存区(Stage/Index)、资源库(Repository或Git Directory)。如果在加上远程的git仓库(Remote Directory)就可以分为四个工作区域。文件在这四个区域之间的转换关系如下：
![图片](https://gwzone.oss-cn-beijing.aliyuncs.com/markdown/git/001.png)
**Workspace**：工作区，就是你平时存放项目代码的地方

**Index / Stage**：暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息

**Repository**：仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本

**Remote**：远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换