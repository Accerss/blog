---
title: Git子模块
date: 2022-09-24 19:25:51
tags: [子模块]
categories: [Git]
sticky: 999
---
# [Git 子模块](https://www.cnblogs.com/renjt1991/p/15925259.html)

Git 子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录，它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立。

示例代码使用的值如下：

- 主仓库路径：github.com/base.git
- 子模块仓库：github.com/common.git
- 子模块在主仓库路径：src/common

### 子模块操作

#### 添加子模块

在主仓库执行 `git submoudle add <url> <repo_name>`，执行后会在主仓库下创建一个 `repo_name` 目录存放子项目；并会创建一个 `.git` 隐藏文件

```bash
git submoudle add github.com/common.git src/common
```

添加子模块后，主仓库会增加一个 `.gitmodules` 文件，记录子模块信息

```properties
# .gitmodules 文件
[submodule "src/common"]
	path = src/common
	url = github.com/common.git
```

#### 查看子模块

```bash
git submodule
# 输出：f5dfd8b05f594ca3c914393e7c641e3ff5285373 src/common (remotes/origin/HEAD)
```

#### 初始化子模块配置

```bash
git submodule init
```

#### 更新子模块

```bash
# 更新全部子模块
git submodule update
# 更新指定子模块
git submodule update src/common

# 递归更新子模块
git submodule update --init --recursive

# 更新子模块到服务器最新版本
git submodule update --remote
```

#### 提交子模块代码

子模块的默认分支不是 **master** ，进入目录后需要先切换分支，再修改提交代码

```bash
cd src/common
git checkout master
git add .
git commit -m "update"
git push
```

#### 删除子模块

执行命令后，会删除 `src/common` 文件夹和修改 `.gitmodules` 文件

```bash
git rm src/common
git commit -m "remove submodule"
git push
```

本地子模块相关文件（非必须删除）：

- 删除 `.git/config` 文件中相关配置
- 删除 `.git/modules/src/common` 文件夹

### 克隆包含子模块的主项目

在克隆主项目时，会包含子模块目录，但目录中没有任何文件，此时需要初始化子模块配置，然后再更新子模块，才会获取到对应的文件

```bash
git clone github.com/base.git
cd base
git submodule init
git submodule update
```