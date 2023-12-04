# Centos7 docker 主机安装注意事项

```
文档信息
创建人 刘金烨
邮件地址 jyliu@dataman-inc.com
建立时间 2015年10月13号
更新时间 2016年1月5号
```

## 1. 基本信息

```
OS：centos7 x86_64
docker version: 1.8以上版本
```

## 2. 安装docker
参考：https://docs.docker.com/installation/centos/

```
$ sudo yum update
$ cat >/etc/yum.repos.d/docker.repo <<-EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
$ sudo yum install docker-engine
```

## 3. 安装生成docker lvm pool 分区的工具

```
rpm -ivh http://get.dataman-inc.com/repos/centos/7/0/cloud-utils-growpart-0.27-20.el7.centos.x86_64.rpm
rpm -ivh http://get.dataman-inc.com/repos/centos/7/0/docker-storage-setup-0.5-3.el7.centos.noarch.rpm
```

## 4. 修改docker daemon启动参数
参考：https://docs.docker.com/reference/commandline/daemon/

### 4.1 设置启动参数加载的配置文件


**加载的配置文件目录**

EnvironmentFile=... 

**docker daemon 启动命令**

ExecStart=... 

**修改后的配置文件**

cat /usr/lib/systemd/system/docker.service

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
ExecStart=/usr/bin/docker daemon $other_args $DOCKER_OPTS $DOCKER_STORAGE_OPTIONS $docker_dataman_opts -H fd://
MountFlags=slave
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
```

### 4.2 设置docker daemon启动其它参数

--insecure-registry #添加信任的私有registry地址（如果没有可以不添加）

--storage-opt dm.basesize=... #basesize 限制单个镜像和容器的空间大小，默认10G，建议basesize设为datadev设备分区大小的1/3,设置后一般不修改，如果修改需要重新生成存储分区。

cat /etc/sysconfig/docker

```
other_args="--storage-opt dm.basesize=50G"
docker_dataman_opts="--insecure-registry 10.3.10.33:5000"
```

### 4.3 设置docker启动存储相关参数（由docker-storage-setup生成，不修改）
cat /etc/sysconfig/docker-storage

```
DOCKER_STORAGE_OPTIONS=-s devicemapper --storage-opt dm.fs=xfs --storage-opt dm.thinpooldev=/dev/mapper/docker--vg-docker--pool
```



### 4.4 生成docker devicemapper 存储分区

**修改生成docker devicemapper 存储分区的必要参数其它非必须修改参数可查看/usr/lib/docker-storage-setup/docker-storage-setup，需要的话可以修改添加到/etc/sysconfig/docker-storage-setup中修改）**


**存储（必须是独立存储分区磁盘）**

DEVS=...

**lvm的docker逻辑卷组**

VG=...

**docker使用的逻辑卷占逻辑组的比例（默认60%，不能设置为100%，否则docker启动不了)**

DATA_SIZE=\*%FREE or \*G

**修改后的配置文件**

cat /etc/sysconfig/docker-storage-setup

```
DEVS=/dev/vdb
VG=docker-vg
DATA_SIZE=99%FREE
```

执行生成docker devicemapper 存储分区命令

```
docker-storage-setup
```

## 5. 启动docker

```
systemctl daemon-reload
systemctl enable docker
systemctl restart docker
```

查看docker信息

```
docker info
```