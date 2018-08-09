[toc]
## 1. 引言
在fork了别人的仓库之后，可能需要保持和上游仓库的同步，查看GitHub的帮助[文档](https://help.github.com/articles/syncing-a-fork/)，官方方法分为两步，
>Before you can sync your fork with an upstream repository, you must configure a remote that points to the upstream repository in Git 

## 2. 配置remote指向上游仓库
### 2.1 进入项目根目录，显示当前的远程仓库
```
$ git remote -v
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```
### 2.2 添加上游仓库地址
```
$ git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```
## 3. 同步fork仓库
### 3.1 fetch上游提交
```
git fetch upstream
```
从上游fetch数据，并commit到本地分支，会存在本地upstream/master分支
### 3.2 checkout到master分支
```
git checkout master
```
### 3.3. 合并upstream/master分支到master
```
git merge upstream/master
```
合并之后就完成了从上游仓库的更新，不会使本地数据丢失。
### 3.4 更新GitHub的fork仓库
```
git push origin master
```
