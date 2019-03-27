# Git简介
Git是一个版本控制系统,相比较其他版本控制系统,Git最大的特点是直接记录文件快照，而非进行差异比较。
## Git概念
Git采用三种状态记录你的文件状态：已提交（committed)、已修改（modified）和已暂存（staged）。已提交表示数据已经安全的保存在本地数据库中。已修改表示修改了文件，但还没保存到数据库中。已暂存表示对一个已经修改文件的当前版本做了标记，使之包含在下次提交的快照中。  
Git项目包含三个工作区域的概念(与文件状态一一对应)：Git仓库、工作目录以及暂存区域。

![Aaron Swartz](./areas.png)
## Git基础
### 获取Git仓库
获取一个Git仓库有两种方法：1. 在现有项目上创建新的Git仓库，2. 从服务器克隆一个现有的Git仓库。

1.创建一个新的仓库命令
    
    git init  // 创建一个新的空仓库
    git add * // 跟踪工作目录下的文件，并暂存以备后续快照提交
    git commit -m "initial project version" // 将暂存文件提交到Git仓库

2.克隆现有仓库

    git clone [url] // git支持两种协议的url: https协议与git://协议.
    git clone https://github.com/yan-ace62/LearnNotebook.git
    git clone git@github.com:yan-ace62/LearnNotebook.git

### 分支操作
切换分支分为切换本地分支与远程分支。

1.显示分支信息

    git branch     显示本地分支信息
    git branch -r  显示远程分支信息
    git branch -a  显示所有分支信息


2.切换分支

    git checkout branchname     切换到branchname分支
    git checkout -b branchname  创建branchname分支并切换过去

3.删除分支

    git branch -d branchname      删除本地branchname分支
    git push -d origin branchname 删除远程branchname分支
    git remote prune origin       删除不存在的远程分支在本地缓存的分支信息

4.推送本地分支到远程服务端

    git push [remote-name] [branch-name] 推送branch-name到remote-name远程仓库
    git push origin develop  推送develop到远程仓库，如果远程仓库不存在develop分支，那么新建develop分支

5.将本地分支与远程分支进行关联

    git branch --set-upstream-to=origin develop 将本地分支与远程分支develop建立关联

6.分支重命名

    git branch -m oldbranchname newbranchname
