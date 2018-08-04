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
git config --global core.quotepath false     //让git支持中文
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
## 5. 撤销和删除
### 5.1 在工作区撤销
```
git checkout --file       //丢弃工作区的修改 
```
### 5.2 在暂存区撤销
```
git reset HEAD <file>    //把暂存区的修改撤销掉（unstage），重新放回工作区
git checkout --file      //丢弃工作区的修改
```
### 5.3 在分支上撤销
```
git reset --hard commitId //返回指定版本
```
### 5.4 正常删除
```
rm file                   //正常删除文件
git rm file               //从版本库删除文件
git commit -m "del..."    //向分支提交删除
```
###5.5 意外删除
```
rm file                   //意外删除文件
git chickout -- file      //删除错误的删除操作
```
## 6. 远程操作
### 6.1 添加远程仓库
```
git remote add origin address   //添加远端地址
```
远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
### 6.2 推送
```
git push -u origin master      //首次推送远端空仓库加-u
```
用git push命令，实际上是把当前分支master推送到远程。如果远程库是空的，第一次推送master分支时，加上-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。
```
git push origin master      //将master分支推送到远程仓库的master分支
```
### 6.3 克隆
```
git clone address           
```
Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快