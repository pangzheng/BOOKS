#docker in docker 命令

    文档信息
    创建人 韩路
    邮件地址 lhan@dataman-inc.com
    时间 2015-09-01

## 1. 测试环境

```
操作系统: ubuntu14.04
docker version: 1.7.1
```

```
操作系统: centos 7.1
docker version: 1.8.1
```

在两台主机测试docker in docker。其中一台ubuntu 14.04主机的docker版本为1.7.1，另一台centos 7.1主机
的docker版本为1.8.1。

本次测试在docker镜像内部启动docker进程，但是docker镜像内部不安装docker，通过目录挂载，把主机上的
docker文件挂载到镜像中。测试分为四种情况，分别为

* ubuntu docker in ubuntu docker
* centos docker in ubuntu docker
* ubuntu docker in centos docker
* centos docker in centos docker

## 2. 启动docker in docker 命令

首先在两台主机上拉取ubuntu:latest和centos:latest镜像。

`docker pull ubuntu && docker pull centos`


### 在ubuntu主机上执行如下命令
*  ubuntu docker in ubuntu docker

```
docker run -it -v /usr/bin/docker:/usr/bin/docker 
               -v /var/run/docker.sock:/var/run/docker.sock
               -v /usr/lib/x86_64-linux-gnu/libapparmor.so.1:/usr/lib/x86_64-linux-gnu/libapparmor.so.1
               ubuntu docker run hello-world
```

*  centos docker in ubuntu docker

```
docker run -it -v /usr/bin/docker:/usr/bin/docker
               -v /var/run/docker.sock:/var/run/docker.sock
               -v /usr/lib/x86_64-linux-gnu/libapparmor.so.1:/lib64/libapparmor.so.1 
               -v /lib/x86_64-linux-gnu/libdevmapper.so.1.02.1:/lib64/libdevmapper.so.1.02.1
               centos docker run hello-world
```

### 在centos主机上执行如下命令
*  ubuntu docker in centos docker

```
docker run -it -v /usr/bin/docker:/usr/bin/docker
               -v /var/run/docker.sock:/var/run/docker.sock
               -v /lib64/libdevmapper.so.1.02:/lib/x86_64-linux-gnu/libdevmapper.so.1.02
               ubuntu docker run hello-world
```

*  centos docker in centos docker

```
docker run -it -v /usr/bin/docker:/usr/bin/docker
               -v /var/run/docker.sock:/var/run/docker.sock
               -v /lib64/libdevmapper.so.1.02:/lib64/libdevmapper.so.1.02
               centos docker run hello-world
```
