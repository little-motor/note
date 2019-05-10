[toc]
# 1. 引言
常用的docker命令总结
# 2. 常用命令
## 2.1 停止和删除容器
```
// 查看所有正在运行容器
docker ps
// containerId 是容器的ID  
docker stop containerId  
// 查看所有容器
docker ps -a  
// 查看所有容器ID
docker ps -a -q  
// stop停止所有容器
docker stop $(docker ps -a -q) 
// remove删除所有容器 
docker rm $(docker ps -a -q) 
```
## 2.2 运行的容器中执行命令
```
//执行容器中的终端
docker exec -it container name  /bin/sh
```