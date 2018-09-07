---
title: git教程
date: 2018-04-06 21:25:51
categories:
- 编程
- git
tags:
- 编程
- git
copyright:
---

# Github常用命令总结

## 远程仓库的使用

本小节将远程仓库变为自己可推送数据的仓库。

```bash
$ git clone git://github.com/schacon/ticgit.git
$ cd grit
$ git remote -v
$ git remote add pb git://github.com/paulboone/ticgit.git
$ git fetch pb
$ git push origin master
$ git remote show origin
```

# Git命令总结

- git init  命令把这个目录变成Git可以管理的仓库
- git add  文件添加到仓库
- git commit  文件提交到仓库
- git log  历史记录
- git log --pretty=oneline  历史记录
- HEAD  当前版本
- HEAD^  上一个版本
- HEAD^^  上上一个版本
- HEAD~100  往上100个版本
- git reset --hard HEAD^  回退到上一个版本
- git reflog  记录你的每一次命令
- git diff HEAD -- readme.txt  查看工作区和版本库里面最新版本的区别
- git status  版本的状态
- git checkout -- file  丢弃工作区的修改
- git reset HEAD file  暂存区的修改撤销掉（unstage），重新放回工作区
- git reset  既可以回退版本，也可以把暂存区的修改回退到工作区
- git rm  从版本库中删除该文件
- git remote add origin git@github.com:michaelliao/learngit.git  关联远程库
- git push -u origin master  本地库的所有内容推送到远程库上
- git push  本地库的内容推送到远程
- git push origin master  推送最新修改
- git clone git@github.com:michaelliao/gitskills.git  克隆一个本地库
- git checkout -b dev  创建dev分支，然后切换到dev分支
- git branch  查看当前分支
- git checkout master  切换回master分支
- git merge dev  把dev分支的工作成果合并到master分支上
- git merge  合并指定分支到当前分支
- git branch -d dev  删除dev分支
- git branch <name>  创建分支
- git checkout <name>  切换分支
- git log --grap  分支合并图
- git merge --no-ff -m "merge with no-ff" dev  强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息
- git stash  把当前工作现场“储藏”起来，等以后恢复现场后继续工作
- git stash list  工作现场
- git stash apply  恢复stash内容并不删除
- git stash drop  删除工作现场
- git stash pop  恢复的同时把stash内容也删了
- git branch -D <name>  丢弃一个没有被合并过的分支
- git remote  查看远程库的信息
- git remote -v  显示更详细的信息
- git pull  最新的提交从origin/dev抓下来
- git branch --set-upstream dev origin/dev  git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接，建立本地分支和远程分支的关联
- git checkout -b branch-name origin/branch-name  在本地创建和远程分支对应的分支，本地和远程分支的名称最好一致
- git tag <name>  打一个新标签
- git tag  查看所有标签
- git log --pretty=oneline --abbrev-commit  找到历史提交的commit id
- git show <tagname>  查看标签信息
- git tag -a <tagname> -m "blablabla..."  指定标签信息
- git tag -s <tagname> -m "blablabla..."  用PGP签名标签
- git push origin <tagname>  推送某个标签到远程
- git push origin --tags  一次性推送全部尚未推送到远程的本地标签
- git tag -d <tagname>  删除一个本地标签
- git push origin :refs/tags/<tagname>  删除一个远程标签
- git config --global alias.st status  告诉Git，以后st就表示status 


# 安装Git

- Linux上安装Git

    $ sudo apt-get install git
    
- Mac OS X上安装Git

    $ brew install git
    
- Windows上安装Git

    在Windows上使用Git，可以从Git官网直接下载安装程序，（网速慢的同学请移步国内镜像），然后按默认选项安装即可。

    安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！
    
- 安装完成后，还需要最后一步设置，在命令行输入：

    $ git config --global user.name "Your Name"
    
    $ git config --global user.email "email@example.com"
    
    因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。你也许会担心，如果有人故意冒充别人怎么办？这个不必担心，首先我们相信大家都是善良无知的群众，其次，真的有冒充的也是有办法可查的。

    注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。
    
# 创建版本库

版本库又名仓库，英文名repository。首先，选择一个合适的地方，创建一个空目录：

    $ mkdir learngit
    $ cd learngit
    $ pwd
    /Users/michael/learngit
    
第二步，通过git init命令把这个目录变成Git可以管理的仓库：

    $ git init
    Initialized empty Git repository in /Users/michael/learngit/.git/
    
瞬间Git就把仓库建好了，而且告诉你是一个空的仓库（empty Git repository），细心的读者可以发现当前目录下多了一个.git的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。

如果你没有看到.git目录，那是因为这个目录默认是隐藏的，用ls -ah命令就可以看见。

把文件添加到版本库,现在我们编写一个readme.txt文件，内容如下：

    Git is a version control system.
    Git is free software.
    
用命令git add告诉Git，把文件添加到仓库：

    $ git add readme.txt
    
执行上面的命令，没有任何显示，这就对了，Unix的哲学是“没有消息就是好消息”，说明添加成功。

用命令git commit告诉Git，把文件提交到仓库：

    $ git commit -m "wrote a readme file"
    [master (root-commit) cb926e7] wrote a readme file
    1 file changed, 2 insertions(+)
    create mode 100644 readme.txt
    
简单解释一下git commit命令，-m后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。

git commit命令执行成功后会告诉你，1个文件被改动（我们新添加的readme.txt文件），插入了两行内容（readme.txt有两行内容）。

为什么Git添加文件需要add，commit一共两步呢？因为commit可以一次提交很多文件，所以你可以多次add不同的文件，比如：

    $ git add file1.txt
    $ git add file2.txt file3.txt
    $ git commit -m "add 3 files."

# 版本回退

在实际工作中，我们脑子里怎么可能记得一个几千行的文件每次都改了什么内容，不然要版本控制系统干什么。版本控制系统肯定有某个命令可以告诉我们历史记录，在Git中，我们用git log命令查看

    $ git log
    commit 3628164fb26d48395383f8f31179f24e0882e1e0
    Author: Michael Liao <askxuefeng@gmail.com>
    Date:   Tue Aug 20 15:11:49 2013 +0800

    append GPL

    commit ea34578d5496d7dd233c827ed32a8cd576c5ee85
    Author: Michael Liao <askxuefeng@gmail.com>
    Date:   Tue Aug 20 14:53:12 2013 +0800

    add distributed

    commit cb926e7ea50ad11b8f9e909c05226233bf755030
    Author: Michael Liao <askxuefeng@gmail.com>
    Date:   Mon Aug 19 17:51:55 2013 +0800

    wrote a readme file
    
git log命令显示从最近到最远的提交日志，我们可以看到3次提交，最近的一次是append GPL，上一次是add distributed，最早的一次是wrote a readme file。
如果嫌输出信息太多，看得眼花缭乱的，可以试试加上--pretty=oneline参数：

    $ git log --pretty=oneline
    3628164fb26d48395383f8f31179f24e0882e1e0 append GPL
    ea34578d5496d7dd233c827ed32a8cd576c5ee85 add distributed
    cb926e7ea50ad11b8f9e909c05226233bf755030 wrote a readme file
    
需要友情提示的是，你看到的一大串类似3628164...882e1e0的是commit id（版本号），和SVN不一样，Git的commit id不是1，2，3……递增的数字，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示，而且你看到的commit id和我的肯定不一样，以你自己的为准。为什么commit id需要用这么一大串数字表示呢？因为Git是分布式的版本控制系统，后面我们还要研究多人在同一个版本库里工作，如果大家都用1，2，3……作为版本号，那肯定就冲突了。

好了，现在我们启动时光穿梭机，准备把readme.txt回退到上一个版本，也就是“add distributed”的那个版本，怎么做呢？

首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，也就是最新的提交3628164...882e1e0（注意我的提交ID和你的肯定不一样），上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。

现在，我们要把当前版本“append GPL”回退到上一个版本“add distributed”，就可以使用git reset命令：

    $ git reset --hard HEAD^
    HEAD is now at ea34578 add distributed
    
看看readme.txt的内容是不是版本add distributed：

    $ cat readme.txt
    Git is a distributed version control system.
    Git is free software.
    
现在，你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的commit id怎么办？

在Git中，总是有后悔药可以吃的。当你用$ git reset --hard HEAD^回退到add distributed版本时，再想恢复到append GPL，就必须找到append GPL的commit id。Git提供了一个命令git reflog用来记录你的每一次命令：

    $ git reflog
    ea34578 HEAD@{0}: reset: moving to HEAD^
    3628164 HEAD@{1}: commit: append GPL
    ea34578 HEAD@{2}: commit: add distributed
    cb926e7 HEAD@{3}: commit (initial): wrote a readme file
    
# 撤销修改

- 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

- 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。

- 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

# 删除文件

一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用rm命令删了

这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，git status命令会立刻告诉你哪些文件被删除了

现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令git rm删掉，并且git commit：

    $ git rm test.txt
    rm 'test.txt'
    $ git commit -m "remove test.txt"
    [master d17efd8] remove test.txt
     1 file changed, 1 deletion(-)
     delete mode 100644 test.txt
     
 现在，文件就从版本库中被删除了。

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：

    $ git checkout -- test.txt
    
git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

# 添加远程库

在Repository name填入learngit，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库

目前，在GitHub上的这个learngit仓库还是空的，GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。

现在，我们根据GitHub的提示，在本地的learngit仓库下运行命令：

    $ git remote add origin git@github.com:michaelliao/learngit.git
    
添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。

下一步，就可以把本地库的所有内容推送到远程库上：

    $ git push -u origin master
    Counting objects: 19, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (19/19), done.
    Writing objects: 100% (19/19), 13.73 KiB, done.
   Total 23 (delta 6), reused 0 (delta 0)
   To git@github.com:michaelliao/learngit.git
     * [new branch]      master -> master
     Branch master set up to track remote branch master from origin.
把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。

由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

从现在起，只要本地作了提交，就可以通过命令：

    $ git push origin master
    
把本地master分支的最新修改推送至GitHub

## 创建与合并分支

1. 原理：

在版本回退里，你已经知道，每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。

一开始的时候，master分支是一条线，Git用master指向最新的提交，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点，每次提交，master分支都会向前移动一步，这样，随着你不断提交，master分支的线也越来越长

当我们创建新的分支，例如dev时，Git新建了一个指针叫dev，指向master相同的提交，再把HEAD指向dev，就表示当前分支在dev，你看，Git创建一个分支很快，因为除了增加一个dev指针，改改HEAD的指向，工作区的文件都没有任何变化！不过，从现在开始，对工作区的修改和提交就是针对dev分支了，比如新提交一次后，dev指针往前移动一步，而master指针不变

假如我们在dev上的工作完成了，就可以把dev合并到master上。Git怎么合并呢？最简单的方法，就是直接把master指向dev的当前提交，就完成了合并，所以Git合并分支也很快！就改改指针，工作区内容也不变！合并完分支后，甚至可以删除dev分支。删除dev分支就是把dev指针给删掉，删掉后，我们就剩下了一条master分支：

2. 实战

首先，我们创建dev分支，然后切换到dev分支：

    $ git checkout -b dev
    Switched to a new branch 'dev'
    
git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：

    $ git branch dev
    $ git checkout dev
    Switched to branch 'dev'

然后，用git branch命令查看当前分支

    $ git branch
    * dev
      master
      
git branch命令会列出所有分支，当前分支前面会标一个*号。

然后，我们就可以在dev分支上正常提交

    $ git add readme.txt 
    $ git commit -m "branch test"
    [dev fec145a] branch test
     1 file changed, 1 insertion(+)
     
现在，dev分支的工作完成，我们就可以切换回master分支

    $ git checkout master
    Switched to branch 'master'
    
切换回master分支后，再查看一个readme.txt文件，刚才添加的内容不见了！因为那个提交是在dev分支上，而master分支此刻的提交点并没有变

现在，我们把dev分支的工作成果合并到master分支上：

    $ git merge dev
    Updating d17efd8..fec145a
    Fast-forward
     readme.txt |    1 +
     1 file changed, 1 insertion(+)
     
git merge命令用于合并指定分支到当前分支。合并后，再查看readme.txt的内容，就可以看到，和dev分支的最新提交是完全一样的。

注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。

当然，也不是每次合并都能Fast-forward，我们后面会讲其他方式的合并。

合并完成后，就可以放心地删除dev分支了：

    $ git branch -d dev
    Deleted branch dev (was fec145a).
    
删除后，查看branch，就只剩下master分支了：

    $ git branch
    * master
因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。

3. 小结

Git鼓励大量使用分支：

查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>

创建+切换分支：git checkout -b <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>

## 解决冲突

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

用git log --graph命令可以看到分支合并图。

## 分支管理策略

通常，合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

下面我们实战一下--no-ff方式的git merge：

首先，仍然创建并切换dev分支：

    $ git checkout -b dev
    Switched to a new branch 'dev'
修改readme.txt文件，并提交一个新的commit：

    $ git add readme.txt 
    $ git commit -m "add merge"
    [dev 6224937] add merge
     1 file changed, 1 insertion(+)
现在，我们切换回master：

   $ git checkout master
   Switched to branch 'master'
准备合并dev分支，请注意--no-ff参数，表示禁用Fast forward：

    $ git merge --no-ff -m "merge with no-ff" dev
    Merge made by the 'recursive' strategy.
     readme.txt |    1 +
     1 file changed, 1 insertion(+)
因为本次合并要创建一个新的commit，所以加上-m参数，把commit描述写进去。

合并后，我们用git log看看分支历史：

     $ git log --graph --pretty=oneline --abbrev-commit
    *   7825a50 merge with no-ff
    |\
    | * 6224937 add merge
    |/
    *   59bc1cb conflict fixed
    ...
可以看到，不使用Fast forward模式，merge后就像这样：

分支策略
在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

## Bug分支

软件开发中，bug就像家常便饭一样。有了bug就需要修复，在Git中，由于分支是如此的强大，所以，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。当你接到一个修复一个代号101的bug的任务时，很自然地，你想创建一个分支issue-101来修复它，但是，等等，当前正在dev上进行的工作还没有提交，并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？

幸好，Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：

    $ git stash
    Saved working directory and index state WIP on dev: 6224937 add merge
    HEAD is now at 6224937 add merge
    
现在，用git status查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。

首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支：

    $ git checkout master
    Switched to branch 'master'
    Your branch is ahead of 'origin/master' by 6 commits.
    $ git checkout -b issue-101
    Switched to a new branch 'issue-101'
    
现在修复bug，需要把“Git is free software ...”改为“Git is a free software ...”，然后提交：

    $ git add readme.txt 
    $ git commit -m "fix bug 101"
    [issue-101 cc17032] fix bug 101
     1 file changed, 1 insertion(+), 1 deletion(-)
     
修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：

    $ git checkout master
    Switched to branch 'master'
    Your branch is ahead of 'origin/master' by 2 commits.
    $ git merge --no-ff -m "merged bug fix 101" issue-101
    Merge made by the 'recursive' strategy.
     readme.txt |    2 +-
     1 file changed, 1 insertion(+), 1 deletion(-)
    $ git branch -d issue-101
    Deleted branch issue-101 (was cc17032).
    
太棒了，原计划两个小时的bug修复只花了5分钟！现在，是时候接着回到dev分支干活了！

    $ git checkout dev
    Switched to branch 'dev'
    $ git status
    # On branch dev
    nothing to commit (working directory clean)
    
工作区是干净的，刚才的工作现场存到哪去了？用git stash list命令看看：

    $ git stash list
    stash@{0}: WIP on dev: 6224937 add merge
    
工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；

另一种方式是用git stash pop，恢复的同时把stash内容也删了：

    $ git stash pop
    # On branch dev
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #       new file:   hello.py
    #
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #       modified:   readme.txt
    #
    Dropped refs/stash@{0} (f624f8e5f082f2df2bed8a4e09c12fd2943bdd40)
    
再用git stash list查看，就看不到任何stash内容了：

    $ git stash list
    
你可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令：

    $ git stash apply stash@{0}
    
## Feature分支

软件开发中，总有无穷无尽的新的功能要不断添加进来。

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

现在，你终于接到了一个新任务：开发代号为Vulcan的新功能，该功能计划用于下一代星际飞船。

于是准备开发：

    $ git checkout -b feature-vulcan
    Switched to a new branch 'feature-vulcan'
    
5分钟后，开发完毕：

    $ git add vulcan.py
    $ git status
    # On branch feature-vulcan
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #       new file:   vulcan.py
    #
    $ git commit -m "add feature vulcan"
    [feature-vulcan 756d4af] add feature vulcan
     1 file changed, 2 insertions(+)
     create mode 100644 vulcan.py
     
切回dev，准备合并：

    $ git checkout dev
    
一切顺利的话，feature分支和bug分支是类似的，合并，然后删除。

但是，就在此时，接到上级命令，因经费不足，新功能必须取消！虽然白干了，但是这个分支还是必须就地销毁：

    $ git branch -d feature-vulcan
    error: The branch 'feature-vulcan' is not fully merged.
    If you are sure you want to delete it, run 'git branch -D feature-vulcan'.
    
销毁失败。Git友情提醒，feature-vulcan分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用命令git branch -D feature-vulcan。

现在我们强行删除：

    $ git branch -D feature-vulcan
    Deleted branch feature-vulcan (was 756d4af).
    
## 多人协作

当你从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且，远程仓库的默认名称是origin。

要查看远程库的信息，用git remote：

    $ git remote
    origin
    
或者，用git remote -v显示更详细的信息

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

    $ git push origin master
    
如果要推送其他分支，比如dev，就改成：

    $ git push origin dev
    
当你的小伙伴从远程库clone时，默认情况下，你的小伙伴只能看到本地的master分支。不信可以用git branch命令看看：

    $ git branch
    * master
    * 
现在，你的小伙伴要在dev分支上开发，就必须创建远程origin的dev分支到本地，于是他用这个命令创建本地dev分支：

    $ git checkout -b dev origin/dev
    
现在，他就可以在dev上继续修改，然后，时不时地把dev分支push到远程

你的小伙伴已经向origin/dev分支推送了他的提交，而碰巧你也对同样的文件作了修改，并试图推送：

    $ git add hello.py 
    $ git commit -m "add coding: utf-8"
    [dev bd6ae48] add coding: utf-8
     1 file changed, 1 insertion(+)
     $ git push origin dev
    To git@github.com:michaelliao/learngit.git
     ! [rejected]        dev -> dev (non-fast-forward)
    error: failed to push some refs to 'git@github.com:michaelliao/learngit.git'
    hint: Updates were rejected because the tip of your current branch is behind
    hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
    hint: before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.
    
推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，再推送：

    $ git pull
    remote: Counting objects: 5, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 3 (delta 0), reused 3 (delta 0)
    Unpacking objects: 100% (3/3), done.
    From github.com:michaelliao/learngit
       fc38031..291bea8  dev        -> origin/dev
    There is no tracking information for the current branch.
    Please specify which branch you want to merge with.
    See git-pull(1) for details

        git pull <remote> <branch>

    If you wish to set tracking information for this branch you can do so with:

        git branch --set-upstream dev origin/<branch>
        
git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接：

    $ git branch --set-upstream dev origin/dev
    Branch dev set up to track remote branch dev from origin.
    
再pull：

    $ git pull
    Auto-merging hello.py
    CONFLICT (content): Merge conflict in hello.py
    Automatic merge failed; fix conflicts and then commit the result.
    
这回git pull成功，但是合并有冲突，需要手动解决，解决的方法和分支管理中的解决冲突完全一样。解决后，提交，再push：

    $ git commit -m "merge & fix hello.py"
    [dev adca45d] merge & fix hello.py
    $ git push origin dev
    Counting objects: 10, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (6/6), 747 bytes, done.
    Total 6 (delta 0), reused 0 (delta 0)
    To git@github.com:michaelliao/learngit.git
       291bea8..adca45d  dev -> dev
       
因此，多人协作的工作模式通常是这样：

首先，可以试图用git push origin branch-name推送自己的修改；

1. 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

2. 如果合并有冲突，则解决冲突，并在本地提交；

3. 没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！

4. 如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

小结

- 查看远程库信息，使用git remote -v；

- 本地新建的分支如果不推送到远程，对其他人就是不可见的；

- 从本地推送分支，使用git push origin branch-name，如果推送失败，先用git pull抓取远程的新提交；

- 在本地创建和远程分支对应的分支，使用git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；

- 建立本地分支和远程分支的关联，使用git branch --set-upstream branch-name origin/branch-name；

- 从远程抓取分支，使用git pull，如果有冲突，要先处理冲突。

# 参考

- https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000
- https://git-scm.com