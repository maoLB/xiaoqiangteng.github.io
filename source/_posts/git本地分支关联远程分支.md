---
title: git本地分支关联远程分支
date: 2018-09-23 13:43:15
categories:
- 编程
- git
tags:
- 编程
- git
copyright:
---

# 需求

本地新建分支，同步刅远程分支并关联远程分支。

# 新建本地分支

```git
git checkout -b devel
```

# 同步远程分支

将本地新建分支push到自己的本地远程origin上，因为只在本地创建了一个新的分支，远程    origin 上还没有该分支

```git
git push origin devel
```

# 关联远程分支

```git
git branch --set-upstream-to=origin/devel
```
