[toc]
## 1. 初始化
### 1.1 设置账户
```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```
### 1.2 查看配置信息
```
git config -l
```
### 1.3 初始化
```
git init
```
## 2. 创建
```
git add                         //添加到缓存区
git commit -m "message..."      //提交到分支
```
## 3. 版本回退
### 3.1 查看修改情况
```
git status                      //显示当前状态
git diff                        //显示修改内容
```
### 3.2 查看修改历史
```
git log                         //显示提交历史
git log --pretty=oneline        //显示简略提交历史
git reflog                      //显示所有的历史提交
```
### 3.2 版本回退
在Git中，用HEAD表示当前版本，也就是最新的提交的上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100
```
git reset --hard HEAD^          //回退到上一个版本
git reset --hard commitId       //回退到指定版本
```
Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从指向前一个版本
![当前版本](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907584977fc9d4b96c99f4b5f8e448fbd8589d0b2000/0)
<center>当前版本</center>

![之前版本](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907594057a873c79f14184b45a1a66b1509f90b7a000/0)
<center>之前版本</center>

## 4. 工作区和暂存区
### 4.1 工作区
工作区就是电脑中能看到的目录
### 4.2 版本库
指.git目录
### 4.3 暂存区
Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD。
git add命令实际上就是把要提交的所有修改放到暂存区（Stage），执行git commit就可以一次性把暂存区的所有修改提交到分支。
![缓存区](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907720458e56751df1c474485b697575073c40ae9000/0)
