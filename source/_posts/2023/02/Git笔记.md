---
title: Git笔记
date: 2023-02-27 20:54:53
tags: [Git]
categories: [笔记]
---
# Git概述

本地仓库：开发人员自己电脑上的Git仓库

远程仓库：远程服务器上的Git仓库

**commit**：提交，将本地文件和版本信息保存到本地仓库

**push**：推送，将本地仓库文件和版本信息上传到远程仓库

**pull**：拉取，将远程仓库文件和版本信息下载到本地仓库

![image-20230227163328361](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20230227163328361.png)

# 下载和安装

![image-20230227163403465](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20230227163403465.png)

验证：鼠标右键是否有Git菜单

![image-20230227163436830](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20230227163436830.png)

# Git代码托管服务

## 常用的代码托管服务

GitHub	码云	GitLab

## 码云代码托管服务

流程：注册，登录，创建，邀请其他用户加入

# Git常用命令

## Git全局设置

设置用户名称和email地址，每次Git提交都会使用该用户信息

**设置用户信息**
git config --global user.name "coder"
git config --global user.email "coder@ncut.cn"

**查看配置信息**
git config --list

## 获取Git仓库

**从本地初始化一个Git仓库**

1. 创建一个空目录
2. 右键打开Git bash窗口
3. 执行命令git init

如果看到.git文件夹，说明仓库创建成功

![](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20230227170022769.png)

**从远程仓库克隆**

命令形式：git clone 【远程Git仓库地址】

## 仓库概念

**版本库：**前面提到的.git隐藏文件夹就是版本库，版本库中存储了很多配置信息、日志信息和文件版本信息等

**工作区：**包含.git文件夹的目录就是工作区，也称为工作目录，主要用于放开发的代码

**暂存区：**.git文件夹中有很多文件，其中有一个index文件就是暂存区，也可以叫做stage，暂存区就是一个临时保存修改文件的地方

![image-20230227185725010](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20230227185725010.png)

## Git工作区中文件的状态

**untracked：**未跟踪，未被纳入版本控制

**tracked：**已跟踪，被纳入版本控制
	Unmodified 未修改状态
	Modified 已修改状态
	Staged 已暂存状态

## 本地仓库操作

> git status	查看文件状态
>
> git add	将文件的修改加入暂存区
>
> git reset	将暂存区的文件取消暂存或者是切换到指定版本
>
> git commit 将暂存区的文件修改提交到版本库
>
> git log	查看日志

## 远程操作

> git remote	查看远程仓库	git remote -v
>
> git remote add	添加远程仓库	git remote add <shortname> <url>
>
> git clone	从远程仓库克隆	git clone <url>
>
> git pull	从远程仓库拉取	git pull [remote-name] [branch-name]
>
> git push	推送到远程仓库	git push [remote-name] [branch-name]

![image-20230227193704858](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20230227193704858.png)

## 分支操作

分支是Git使用过程中非常重要的概念，使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线，同一个仓库可以有多个分支，各个分支相互独立，互不干扰
通过git init命令创建本地仓库时默认会创建一个master分支

相关命令具体如下：

> git branch	查看分支
>
> git branch [name]	创建分支
>
> git checkout [name]	切换分支
>
> git push [shortname] [name]	推送至远程仓库分支
>
> git merge [name]	合并分支

## 标签操作

> git tag	列出已有标签
>
> git tag [name]	创建标签
>
> git push [shortName] [name]	将标签推送至远程仓库
>
> git checkout -b [branch] [name]	检出标签