# Docker基础知识
***

### 安装Docker


### 常用命令
###### 搜索镜像
```
docker search ubuntu
```

###### 下载镜像
```
docker pull ubuntu:latest
```

镜像格式为：  
> 镜像名称:标签  
latest为下载最新版本，镜像名称中'/'之前为用户名。官方镜像名称中不会又用户名。

###### 列出本地镜像
```
docker images
```

###### 删除镜像
```
docker rmi imagename
```

###### 查看容器列表
```
docker ps -a
```

###### 启动容器
```
docker start dockername
重启
docker restart dockername
```

###### 链接容器
```
docker attach dockername
```
输入exit或Ctrl+D终止容器。若依次输入Ctrl+P或者Ctrl+Q，则退出容器而不是终止。

###### 删除容器
```
docker rm containername
```

***
```
bridge
docker0  
br-xxxxxxxx

bridge
my_net
```

网关在网桥上 

容器的网卡 挂在bridge上  
创建网络my_net是创建一个名字为my_net的网络，并在linux系统上创建一个bridge  
创建使用my_net的容器，在容器中创建一个网卡，并连接到my_net对应的bridge上
