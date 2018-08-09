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
git config --global core.quotepath false     #让git支持中文
```
### 1.3 初始化
```
git init
```
## 2. 创建
```
git add                         #添加到缓存区
git commit -m "message..."      #提交到分支
```
## 3. 版本回退
### 3.1 查看修改情况
```
git status                      #显示当前状态
git diff                        #显示修改内容
```
### 3.2 查看修改历史
```
git log                         #显示提交历史
git log --pretty=oneline        #显示简略提交历史
git reflog                      #显示所有的历史提交
```
### 3.2 版本回退
在Git中，用HEAD表示当前版本，也就是最新的提交的上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100
```
git reset --hard HEAD^          #回退到上一个版本
git reset --hard commitId       #回退到指定版本
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
git checkout --file       #丢弃工作区的修改 
```
### 5.2 在暂存区撤销
```
git reset HEAD <file>     #把暂存区的修改撤销掉（unstage），重新放回工作区
git checkout --file       #丢弃工作区的修改
```
### 5.3 在分支上撤销
```
git reset --hard commitId #返回指定版本
```
### 5.4 正常删除
```
rm file                   #正常删除文件
git rm file               #从版本库删除文件
git branch -d futureName  #删除分支名
git commit -m "del..."    #向分支提交删除
```
###5.5 意外删除
```
rm file                   #意外删除文件
git chickout -- file      #删除错误的删除操作
```
## 6. 远程操作
### 6.1 remote添加远程仓库
```
git remote add origin gitAddress   #添加远端地址
```
远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。Git会自动将原版本库路径存储为他的origin，通过-v命令显示可被获取或推送提交的路径
```
git remote -v
```
### 6.3 fetch获取数据
fetch命令是单向操作，可用于从另一个版本库中获取所有分支本地版本库不存在的提交。
```
git fetch origin             #获取远端提交
```
### 6.4 pull
获取操作通常会带来冲突，因为会有新的提交被添加到本地或别处版本库，在大多数情况下他们需要进行合并,pull命令会从远程版本库导入这些提交，然后在必要的情况下将他们合并到当前分支，pull = fetch + merge
```
git pull
```

### 6.5 push推送
格式是指定远程版本库名（例如origin），以及本地分支名（例如branch-a），branch-a分支下的新本地提交传送给origin所指向的远程版本库，并更新分支指针。
```
git push -u origin branch-a      #首次推送远端空仓库加-u，git会把本地的指定分支与远程新分支关联起来
git push origin branch-a         #将branch-a分支推送到远程origin仓库
```
需要注意的是push只针对快进合并，也就是远程版本库没有比本地更多更新的提交，同时，无参数调用push将只发送那些与其他版本库中同名的本地分支，而pull和fetch无参会选取全部分支。
### 6.6 克隆
```
git clone address     
git clone /Users/add/add...   #从本地克隆一个仓库      
```
Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快

## 7. 合并
假如有两个分支master和develop分支，现在将develop分支合并到master
```
# on the branch "master"
git merge feature

#多个分支合并
git octopus branches
```
### 7.1 合并的过程
git会查找两个分支的共同祖先，然后比较后合并，但有时如果两个分支都修改了某一处则会发生合并冲突，常见的有两种

- 编辑冲突：两个开发者对同一行代码做了不同的修改
- 内容冲突：两个开发者对某份代码的几个部分做出各自修改的时候

### 7.2 编辑冲突
当有内容冲突时会发生conflict提示，没有发生冲突的部分文件会被add，有冲突的部分文件会被插入冲突标志，开发者需要手动排除，默认显示两个分支的冲突标志，如果需要显示共同祖先输入下面命令
```
git config merge.conflictstyle diff3
```
解决冲突后即可add和commit
### 7.3 合并撤销
```
git reset --merge
```
### 7.4 快进合并
当分支只有一次提交，合并到主分支时git会直切前移为了保持主分支的线性，但是缺点是看不到版本库的发展，解决方法是
```
git merge --no-ff a-branch
```

参考文章：https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000