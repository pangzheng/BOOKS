# CentOS7 或 RHEL7 的 docker 存储优化

```
文档信息
创建人 周伟涛
邮件地址 wtzhou@dataman-inc.com
```

## 1. 基本信息

```
OS：centos7 或 RHEL7 x86_64
docker version: 1.8以上版本
```

## 2. 存储优化方案选择

### 2.1 docker 与操作系统共享磁盘的单磁盘存储优化

由于某些原因，我们想让 docker 直接使用系统盘上的存储，或者说，docker 需要与系统共享磁盘。这种情况下，我们需要在操作系统安装时，在 VG 层为 docker 预留需要的存储空间，具体操作步骤如下。

安装 RHEL7 操作系统，在 "安装信息摘要" 步，

- 选择-系统-安装位置，进入磁盘分区界面
- 选择-其它存储选项-分区-我要配置分区，点左上角的“完成”，进入 “手动分区” 页面
- 我们需要在这一步为 docker 预留存储空间，对擅长磁盘分区的用户来说，您可以按事先规划的分区进行配置并保留一定的存储空间； 对不是很熟悉磁盘分区的用户来，

  - 点击 “自动创建分区”， 这时系统会默认将磁盘的所有空间进行分区
  - 我们需要将根目录 “/” 的空间释放一部分给 docker，首先将 “／” 的容量改小（譬如将当前值减去100G留给docker使用）， 其次点击 “／” 右侧的 “VG”->“编辑” 按钮，进入到“VG”设置页面，将 “VG” 大小设为 “as large as possible”

### 2.2 docker 使用独立的数据磁盘存储

该方法是我们更推崇的存储方案，这种方案在操作系统装机时无需做特殊配置。

## 3. 安装 docker
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

## 4. 安装生成docker lvm pool 分区的工具

```
rpm -ivh http://get.dataman-inc.com/repos/centos/7/0/cloud-utils-growpart-0.27-20.el7.centos.x86_64.rpm
rpm -ivh http://get.dataman-inc.com/repos/centos/7/0/docker-storage-setup-0.5-3.el7.centos.noarch.rpm
```

## 5. 格式化 docker devicemapper 存储分区

### 5.1 创建或编辑文件 */etc/sysconfig/docker-storage-setup*

- 对于单磁盘存储方案来说，配置如下

   ```
   VG=rhel 这是 VG 的 name，若在装机时没有做过更改，默认是 rhel，若已更改，请使用更改后的 VG name。
   DATA_SIZE=95%FREE
   ```

- 对于 docker 独立磁盘存储方案来说，配置如下

   ```
   DEVS=sdb # 这是为 docker 准备的独立磁盘的盘符，可能是 sdc 等。
   DATA_SIZE=95%FREE
   ```

### 5.2 执行生成 docker devicemapper 存储分区命令

```
docker-storage-setup
```

## 6. 修改docker daemon启动参数

本部分主要参考了：https://docs.docker.com/reference/commandline/daemon/

### 6.1 设置启动参数加载的配置文件

编辑文件 */usr/lib/systemd/system/docker.service*,

 **修改后的配置文件** 如下，

```bash
cat /usr/lib/systemd/system/docker.service

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/docker-storage
ExecStart=/usr/bin/docker daemon --storage-opt dm.basesize=50G $DOCKER_OPTS $DOCKER_STORAGE_OPTIONS -H fd://
MountFlags=slave
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
```

其中，

- EnvironmentFile:  **加载的配置文件目录**
- ExecStart:  **docker daemon 启动命令**
- --storage-opt dm.basesize=... #basesize 限制单个镜像和容器的空间大小，默认10G，建议basesize设为datadev设备分区大小的1/3,设置后一般不修改，如果修改需要重新生成存储分区。


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
