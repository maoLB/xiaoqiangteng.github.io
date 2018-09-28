---
title: git常见问题
date: 2018-09-25 15:21:27
categories:
- 编程
- git
tags:
- 编程
- git
copyright:
---

# git合并常见问题

## 合并失败后报错：

```git
bogon:AR_VIO didi$ git add .
error: unable to index file OCARKitTest/ThirdParty/CocoaAsyncSocket
fatal: updating files failed
```

解决方法：

```git
$ git rm --cached -r OCARKitTest/ThirdParty/
$ git add .
$ git commit -m ""
$ git push
```

## 摆错二

```git
bogon:AR_VIO didi$ git checkout devel
error: The following untracked working tree files would be overwritten by checkout:
	OCARKitTest/OCARKitTest.xcodeproj/project.xcworkspace/contents.xcworkspacedata
	OCARKitTest/OCARKitTest.xcodeproj/project.xcworkspace/xcshareddata/IDEWorkspaceChecks.plist
Please move or remove them before you switch branches.
Aborting
```

通过错误提示可知，是由于一些untracked working tree files引起的问题。所以只要解决了这些untracked的文件就能解决这个问题。

解决方法：

```git
git clean  -d  -fx .
git add .
git commit -m ""
git push
```
