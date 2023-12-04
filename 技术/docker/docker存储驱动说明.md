# Docker的存储方式—Storage Driver
## 1 什么是 Docker Storage Driver
Docker在启动容器的时候，需要创建文件系统，为rootfs提供挂载点。最初Docker仅能在支持Aufs文件系统的ubuntu发行版上运行，但是由于Aufs未能加入Linux内核，为了寻求兼容性、扩展性，Docker在内部通过graphdriver机制这种可扩展的方式来实现对不同文件系统的支持。

Docker 模型的核心部分是有效利用分层镜像机制，镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。其中主要的机制就是分层模型和将不同目录挂载到同一个虚拟文件系统下。
docker采用了几种不同的存储drivers

- aufs
- devicemapper
- btrfs
- overlay
- vfs


## 2 AUFS
### 2.1 说明
AUFS（AnotherUnionFS）是一种联合文件系统。所谓 UnionFS 就是把不同物理位置的目录合并 `mount` 到同一个目录中。UnionFS 的一个最主要的应用是，把一张 CD/DVD 和一个硬盘目录给联合 `mount` 在一起，然后就可以对这个只读的 CD/DVD 上的文件进行修改（当然，修改的文件存于硬盘上的目录里）。
AUFS 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 AUFS 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。
### 2.2 基础试验
#### 2.2.1 环境准备
ubuntu14.04－3.10.xx内核默认安装了aufs模块(linux主线不支持，如果升级了主线内核会缺乏aufs模块)
                
        # lsmod | grep aufs
        aufs                  202783  215
#### 2.2.2 例1
1. `/data/dataman`目录结构
  
        /data/dataman# tree
        .
        ├── aufs
        ├── dir1
        │   └── file1
        └── dir2
            └── file2
- `file1`和`file2`的文件内容

        # cat dir1/file1
        dataman1
        # cat dir2/file2
        dataman2
- 将 `dir1` & `dir2` `mount` 到 `aufs` 目录
    
        # mount -t aufs -o br=/data/dataman/dir1=ro:/data/dataman/dir2=rw none /data/dataman/aufs     
    `mount` 参数说明
        
        -o 指定mount传递给文件系统的参数
        br 指定需要挂载的文件夹，这里包括dir1和dir2
        ro/rw 指定文件的权限只读和可读写
        none 这里没有设备，用none表示
- 再次查看目录结构

        # tree
        .
        ├── aufs
        │   ├── file1
        │   └── file2
        ├── dir1
        │   └── file1
        └── dir2
            └── file2
- 进入 `aufs` 目录测试

        # cd aufs
        # echo "test1" > file1
        -su: file1: Read-only file system
        # echo "test2" > file2
- 查看结果
        
        # cat file1
        dataman1
        # cat file2
        test2
- 如果映射的两个目录文件相同如何处理,在`dir1`创建`file2`，重新`mount`
        
        # umount /data/dataman/aufs
        # cat dir1/file2
        dataman3
        # mount -t aufs -o br=/data/dataman/dir1=ro:/data/dataman/dir2=rw none /data/dataman/aufs
        #echo 
        # cat  aufs/file2
        dataman3
结论若出现有同名文件的情况，则以先挂载的为主，其他的不再挂载。说明Docker镜像为什么采用增量的方式：利用Aufs的特性达到节约空间的目的。 
                    
### 2.3 缺点
- AUFS 到现在还没有加入内核主线( centos 无法直接使用)
- BUG 还比较多
- 代码方法比较粗糙(后期改了很多)

### 2.4 优点
- Ubuntu 10.04，Debian6.0, Gentoo Live CD 默认已经支持
- AUFS唯一一个 `storage driver` 可以实现容器间共享可执行及可共享的运行库, 跑成千上百个拥有相同程序代码或者运行库时时候，AUFS是个相当不错的选择。
- Docker 第一版支持

### 2.5 Docker 例子
运行一个实例应用是删除一个文件`/etc/shadow`，看aufs的结果

    # docker run centos rm /etc/shadow
    # ls -la /var/lib/docker/aufs/diff/$(docker ps --no-trunc -lq)/etc
    
    total 8
    drwxr-xr-x 2 root root 4096 Sep  2 18:35 .
    drwxr-xr-x 5 root root 4096 Sep  2 18:35 ..
    -r--r--r-- 2 root root    0 Sep  2 18:35 .wh.shadow
### 2.6 目录结构
- 容器挂载点(只有容器运行时才被加载)

        /var/lib/docker/aufs/mnt/$CONTAINER_ID/
- 分支(和镜像不同的文件，只读活着读写)

        /var/lib/docker/aufs/diff/$CONTAINER_OR_IMAGE_ID/
- 镜像索引表(每个镜像引用镜像名)

        /var/lib/docker/aufs/layers/
        
### 2.7 其他
#### 2.7.1 AUFS 文件系统可使用的磁盘空间大小
    # df -h /var/lib/docker/
    
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        20G  4.0G   15G  22% /
#### 2.7.2 系统挂载方式
启动的 Docker

    docker ps
    
    CONTAINER ID        IMAGE                        COMMAND                CREATED             STATUS              PORTS                      NAMES
    3f2e9de1d9d5        mesos/bamboo:v0.1c           "/usr/bin/bamboo-hap   5 days ago          Up 5 days                                      mesos-20150825-162813-1248613158-5050-1-S0.88c909bc-6301-423a-8283-5456435f12d3
    dc9a7b000300        mesos/nginx:base             "/bin/sh -c nginx"     7 days ago          Up 7 days           0.0.0.0:31967->80/tcp      mesos-20150825-162813-1248613158-5050-1-S0.42667cb2-1134-4b1a-b11d-3c565d4de418
    1b466b5ad049        mesos/marathon:omega.v0.1    "/usr/bin/dataman_ma   7 days ago          Up 16 hours                                    dataman-marathon
    0a01eb99c9e7        mesos/nginx:base             "/bin/sh -c nginx"     7 days ago          Up 7 days           0.0.0.0:31587->80/tcp      mesos-20150825-162813-1248613158-5050-1-S0.4f525828-1217-4b3d-a169-bc0eb901eef1
    c2fb2e8bd482        mesos/dns:v0.1c              "/usr/bin/dataman_me   7 days ago          Up 7 days                                      mesos-20150825-162813-1248613158-5050-1-S0.82d500eb-c3f0-4a00-9f7b-767260d1ee9a
    df102527214d        mesos/zookeeper:omega.v0.1   "/data/run/dataman_z   8 days ago          Up 8 days                                      dataman-zookeeper
    b076a43693c1        mesos/slave:omega.v0.1       "/usr/sbin/mesos-sla   8 days ago          Up 8 days                                      dataman-slave
    e32e9fc9a788        mesos/master:omega.v0.1      "/usr/sbin/mesos-mas   8 days ago          Up 8 days                                      dataman-master
    c8454c90664e        shadowsocks_server           "/usr/local/bin/ssse   9 days ago          Up 9 days           0.0.0.0:57980->57980/tcp   shadowsocks
    6dcd5bd46348        registry:v0.1                "docker-registry"      9 days ago          Up 9 days           0.0.0.0:5000->5000/tcp     dataman-registry
对照系统挂载点

    grep aufs /proc/mounts
    
    /dev/mapper/ubuntu--vg-root /var/lib/docker/aufs ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
    none /var/lib/docker/aufs/mnt/6dcd5bd463482edf33dc1b0324cf2ba4511c038350e745b195065522edbffb48 aufs rw,relatime,si=d9c018051ec07f56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/c8454c90664e9a2a2abbccbe31a588a1f4a5835b5741a8913df68a9e27783170 aufs rw,relatime,si=d9c018051ba00f56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/e32e9fc9a788e73fc7efc0111d7e02e538830234377d09b54ffc67363b408fca aufs rw,relatime,si=d9c018051b336f56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/b076a43693c1d5899cda7ef8244f3d7bc1d102179bc6f5cd295f2d70307e2c24 aufs rw,relatime,si=d9c018051bfecf56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/df102527214d5886505889b74c07fda5d10b10a4b46c6dab3669dcbf095b4154 aufs rw,relatime,si=d9c01807933e1f56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/c2fb2e8bd4822234633d6fd813bf9b24f9658d8d97319b1180cb119ca5ba654c aufs rw,relatime,si=d9c01806c735ff56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/0a01eb99c9e702ebf82f30ad351d5a5a283326388cd41978cab3f5c5b7528d94 aufs rw,relatime,si=d9c018051bfebf56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/1b466b5ad049d6a1747d837482264e66a87871658c1738dfd8cac80b7ddcf146 aufs rw,relatime,si=d9c018052b2b1f56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/dc9a7b000300a36c170e4e6ce77b5aac1069b2c38f424142045a5ae418164241 aufs rw,relatime,si=d9c01806d9ddff56,dio,dirperm1 0 0
    none /var/lib/docker/aufs/mnt/3f2e9de1d9d51919e1b6505fd7d3f11452c5f00f17816b61e6f6e97c6648b1ab aufs rw,relatime,si=d9c01806c708ff56,dio,dirperm1 0 0
#### 2.7.3 性能分析
优点

- AUFS `mount()` 方法很快，所以创建容器也很快。
- 读写访问都具有本机效率(一旦找到后)
- 顺序读写和随机读写的性能大于kvm

缺点在初始化 `open()\stat()` 方法会有两种情况代价很高

- 当写入大文件的时候(比如日志或者数据库..)
- 动态mount多目录路径的问题,导致branch越多，查找文件的性能也就越慢。

解决办法

重要数据，入数据库等直接使用 `-v` 参数挂载到系统盘

特性

同时启动1000个一样的容器，数据只从磁盘加载一次，缓存也只从内存加载一次。然后重复 
 
## 3 Device Mapper
### 3.1 Logical Volume 概念说明
Logical Volume 主要包括两大特性:Snapshot，Thinly-Provisioned
#### Snapshot

Snapshot是Lvm提供的一种特性，它可以在不中断服务运行的情况下为the origin（original device）创建一个虚拟快照(Snapshot)，具有以下几个特点：

1. 当the origin内容发生变化时，snapshot对变化的部分做一个拷贝以用来对the origin进行重构。

- 因为只对变化的部分做拷贝，所以Lvm的Snapshot在读操作频繁而写操作不频繁的情况下占用很少的一部分空间便能完成特定任务。
- 当Snapshot大小耗尽或者远大于实际需求时，我们可以对其大小进行调节。
- 当对Snapshot的数据进行写操作的时候，Snapshot实施相应操作，并丢弃从the origin的拷贝，以后的操作以写操作之后Snapshot中的数据为准。
- 在某些发行版的Linux系统下，可以使用lvconvert的--merge选项将Snapshot合并回the origin。

#### Thinly-Provisioned
Thin-Provisioning是一项利用虚拟化方法减少物理存储部署的技术，可最大限度提升存储空间利用率。例如某位用户向服务器管理员请求分配10TB的资源的情形。实际情况中这个数值往往是峰值，根据使用情况，分配2TB就已足够。因此，系统管理员准备2TB的物理存储，并给服务器分配10TB的虚拟卷。服务器即可基于仅占虚拟卷容量1/5的现有物理磁盘池开始运行。这样的“始于小”方案能够实现更高效地利用存储容量。
#### Thin-provisioning Snapshot
Thin-provisioning Snapshot结合Thin-Provisioning和Snapshot两种技术，允许多个虚拟设备同时挂载到一个数据卷以达到数据共享的目的。

Thin-Provisioning Snapshot的特点如下：

1. 可以将不同的snaptshot挂载到同一个the origin上，节省了磁盘空间。
- 当多个Snapshot挂载到了同一个the origin上，并在the origin上发生写操作时，将会触发COW操作。这样不会降低效率。
- Thin-Provisioning Snapshot支持递归操作，即一个Snapshot可以作为另一个Snapshot的the origin，且没有深度限制。
- 在Snapshot上可以创建一个逻辑卷，这个逻辑卷在实际写操作（COW，Snapshot写操作）发生之前是不占用磁盘空间的。

### 3.2 Device mapper 说明
Thin-Provisioning Snapshot是作为 Device mapper 的一个 target 在内核中实现。Device mapper 是 Linux 2.6 内核中提供的一种从逻辑设备到物理设备的映射框架机制，用户可以很方便的根据自己的需要制定实现存储资源的管理策略.[详细说明](http://www.ibm.com/developerworks/cn/linux/l-devmapper/index.html).
### 3.3 基础实验
#### 3.3.1 环境准备
2.6以上内核全部支持。
#### 3.3.3.2 例子
1. Thin-Provisioning Snapshot需要一个data设备和一个metadata设备分别用来存放实际数据和元数据，seek表示为略过of选项指定的输出文件的前2G空间再写入内容。这2G在硬盘上是没有占有空间的，占有空间只有1k的内容。当向其写入内容时，才会在硬盘上为其分配空间。

创建metadata(元数据)
        
        ＃dd if=/dev/zero of=/tmp/metadata bs=1K count=1 seek=2G
        1+0 records in
        1+0 records out
        1024 bytes (1.0 kB) copied, 0.000153611 s, 6.7 MB/s
创建data(存储池)
        
        # dd if=/dev/zero of=/tmp/data bs=1K count=1 seek=100G
        1+0 records in
        1+0 records out
        1024 bytes (1.0 kB) copied, 8.7567e-05 s, 11.7 MB/s        
2. 将其挂载到loopback设备上，以供使用：

        # losetup /dev/loop0 /tmp/metadata
        # losetup /dev/loop1 /tmp/data
        # losetup -a
        /dev/loop0: [0035]:183033 (/tmp/metadata)
        /dev/loop1: [0035]:185651 (/tmp/data)
3. 创建Snapshot。首先创建一个thin-pool，指定要用哪些设备来创建Snapshot,将逻辑设备的0～20971520之间的sector映射到/dev/loop0上，元数据信息存储到/dev/loop1

        # dmsetup create pool --table "0 20971520 thin-pool /dev/loop0 /dev/loop1 128 32768 1 skip_block_zeroing"
参数说明

        pool指定thin-pool的名字
        --table指定dmsetup的一个target，其中：
            0表示起始sector
            20971520表示sector的数量
            thin-pool表示target的名字
            /dev/loop0 /dev/loop1为设备名
            128表示一次可分配的最小sector数
            32768表示空闲空间的阈值
            1特征参数的个数
            skip_block_zeroing特征参数，表示略过用0填充的块

4. 创建一个Thinly-Provisioned volume并格式化

        # dmsetup message /dev/mapper/pool 0 "create_thin 0"
        # dmsetup create thin --table "0 2097152 thin /dev/mapper/pool 0"
        #mkfs.ext4 -E discard,lazy_itable_init=0 /dev/mapper/thin
5. 根据需求创建一个internal(或者 external) snapshot

        # dmsetup suspend /dev/mapper/thin
        # dmsetup message /dev/mapper/pool 0 "create_snap 1 0"
        # dmsetup resume /dev/mapper/thin
        # dmsetup create snap --table "0 2097152 thin /dev/mapper/pool 1"      
        # dmsetup status
        thin: 0 2097152 thin 99840 2097151
        fedora-swap: 0 8126464 linear 
        fedora-root: 0 104857600 linear 
        snap: 0 2097152 thin 99840 2097151
        pool: 0 20971520 thin-pool 0 280/4161600 780/163840 - rw discard_passdown queue_if_no_space 
        fedora-home: 0 97509376 linear

6. 挂载到任何一个文件夹下，并对其进行创建文件等操作：
         
        mount /dev/mapper/snap dv 
7. 在dv文件夹下创建一个内容为this is snap的文件file，然后，在snap的基础上再创建一个snap1
        
        # dmsetup message /dev/mapper/pool 0 "create_snap 2 1"
        # dmsetup create snap1 --table "0 2097152 thin /dev/mapper/pool 2"
        # mkdir dv1
        # mount /dev/mapper/snap1 dv1
        # cd dv1
        # cat file
        this is snap
        
### 3.4 Docker Device Mapper            
该系统非常复杂，Docker用的是基于 Device Mapper 的“精简目标”的特性。它实际上是目标块设备的快照，之所以被称为“精简”是因为它允许精简配置。精简配置意味着你有一个（希望很大）可用存储块的池，接着你可以从那个池中创建任意大小的块设备（虚拟磁盘，如有需要）；在你实际读写后，这些存储块将会被标记为已使用（或者从池中拿走）。意味着是可以超额使用这个池，比如在一个 100GB 的池里面创建几千个 10GB 的卷，甚至可能是一个 100TB 的卷在一个 1GB 的池里面。只要实际读写的块的容量不大于池的大小都 OK 。
可以做:

- 裸块设备
- RAID
- 加密设备
- 快照
- 等

### 3.5 查看存储信息
#### Docker info
 poll name ＃通过`dmsetup ls`获取,想要知道更多信息使用`dmsetup info` & `dmsetup status` 
  
    # docker info
    Containers: 0
    Images: 4
    Storage Driver: devicemapper
     Pool Name: docker-253:16-3145732-pool  
     Pool Blocksize: 65.54 kB
     Backing Filesystem: extfs            #系统的文件系统
     Data file: /dev/loop0                #实际data文件所在
     Metadata file: /dev/loop1            #实际meta文件所在
     Data Space Used: 540.9 MB            #data 已使用
     Data Space Total: 107.4 GB           #data 总共
     Data Space Available: 50.29 GB       #data 可用
     Metadata Space Used: 942.1 kB        #metadata 已使用
     Metadata Space Total: 2.147 GB       #metadata 总共  
     Metadata Space Available: 2.147 GB   #metadata 可用
     Udev Sync Supported: true
     Deferred Removal Enabled: false
     #实际data文件所在,单独分块设备，块大小docker用64kB
     Data loop file: /data/lib/docker/devicemapper/devicemapper/data
     #实际meta文件所在
     Metadata loop file: /data/lib/docker/devicemapper/devicemapper/    metadata
     Library Version: 1.02.93-RHEL7 (2015-01-28)
    Execution Driver: native-0.2
    Logging Driver: json-file
    Kernel Version: 3.19.3-1.el7.elrepo.x86_64
    Operating System: CentOS Linux 7 (Core)
    CPUs: 2
    Total Memory: 3.859 GiB
    Name: localhost.localdomain
    ID: PVOV:7QTG:YN5B:3POB:UDSF:3ZK3:QXEP:NUFA:PHML:Y6IL:AXZL:74Y2
### 3.5 目录结构
- 容器挂载点(只有容器运行时才被加载,数据存储在2个文件data & metadata)

        /var/lib/docker/devicemapper/mnt/$CONTAINER_ID/
- 已 json 文件来跟踪镜像id和其他镜像情况

        /var/lib/docker/devicemapper/metadata
- 默认挂盘使用地点(如果写满，将停止尝试，直到增加)

        /var/lib/docker/devicemapper/devicemapper
### 3.6 性能分析
#### 3.6.1 优点 

- 默认配置`data` & `metadata` 将输入到一个循环装置，无需配置
- 兼容性比较好
- 因为存储为1个文件，减少了inond消耗
- 
#### 3.6.2 缺点

- 每次1个容器写数据都是一个新块，块必须从池中分配，真正写的时候是稀松文件,虽然它的利用率很高，但性能不好，因为额外增加了vfs开销.
- 每个容器都有自己的块设备时，它们是真正的磁盘存储，所以当启动1000个容器时，它都会从磁盘加载1000次
- 默认存储池只有100GB
- 是所有空间是静态值     
        
### 3.7 解决方案
#### 3.7.1 测试环境需要更大的存储池
- 停止 Docker 守护进程
- 擦去 /var/lib/docker。（警告：正如前面提到的，这个操作会把你所有的容器和镜像都删除掉。）
- 创建存储目录： mkdir -p /var/lib/docker/devicemapper/devicemapper
- 创建你的池： dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/data bs=1G count=0 seek=250 ，创建一个 250G 的稀疏文件。如果你指定 bs=1G count=250（不使用 seek 选项），那么它会创建一个普通文件（而不是一个稀疏文件）。
- 重启 Docker 守护进程。提示：在默认情况下，如果你有 AUFS 的支持， Docker 会使用它；所以如果你要强制使用 Device Mapper 的插件，需要在启动 Docker 的命令中增加 -s devicemapper 的选项。
- 使用 docker info 来检查 Data Space Total 的值是否正确。
#### 3.7.2 需要一个更快的存储池(挂独立的存储)

- 停止 Docker 守护进程
- 擦去 /var/lib/docker。（警告：正如前面提到的，这个操作会把你所有的容器和镜像都删除掉。）
- 创建一个存储目录： mkdir -p /var/lib/docker/devicemapper/devicemapper
在目录下创建一个数据软链接，指向设备：
ln -s /dev/sdb /var/lib/docker/devicemapper/devicemapper/data
- 重启 Docker
- 使用 docker info 来检查 Data Space Total 的值是否正确
#### 3.7.3 生产优化参考
##### Centos
CentOS/Redhat/Fedora/AMI系统，使用`devicemapper`作为存储后端，但注意使用`devicemapper`必须分配两个独立的磁盘分区给`docker`使用，默认`loopback`模式会出现故障，严禁生产使用！

方法编辑`/etc/sysconfig/docker`配置文件，注意替换里面的`basesize`,`datadev`和`metadatadev`

    DOCKER_OPTS="--storage-driver=devicemapper --storage-opt dm.basesize=总磁盘大小/3(单位 G) --storage-opt dm.datadev=/dev/sde1 --storage-opt dm.metadev=/dev/sdf1"
注解：

- basesize

        表示镜像和容器的空间大小，不同容器如果使用相同镜像，不会增加大小。容器内的修改应用COW技术，修改后的内容不计算此空间中，默认10G，建议basesize设为datadev设备分区大小的1/3
- datadev

        存放镜像和容器，要求必须是ext4或xfs文件系统，默认ext4，如果设为xfs，使用dm.fs指定
- metadatadev

        用于存放元数据，对空间要求不大，可建立一个10G的分区。建议和datadev使用不同设备，如果条件许可，使用SSD，使用时需要将分区前4k清0。 
        dd if=/dev/zero of=$metadata_dev bs=4096 count=1
##### Ubuntu
待


## 4 Overlay
## 修改 Centos overlay
1. 升级到3.10.0-327.13.1.el7.x86_64
    - yum -y install kernel-*
        
            tar包在跳板机／data／kernelbak.tar 注意/etc/yum.conf文件中exclude值
    - yum update        
3. reboot
4. modinfo overlay && modprobe overlay && lsmod overlay 
5. vi /usr/lib/systemd/system/docker.service
6. ExecStart=/usr/bin/docker daemon 后面的 -s  devicemap 改成 overlay
        
            ExecStart=/usr/bin/docker daemon  -H fd:// -s overlay
7. systemctl daemon-reload
8. systemctl restart docker
9. systemctl enable docker
## Overlay 系统兼容问题
### 问题
在 Centos 的系统中增加了 Docker 支持 Overlay 文件系统后，运行 Ubuntu 为基础的镜像报错:

Error response from daemon: mkdir /var/lib/docker/overlay/21bff7d8b748d19927d3c102283921adc561b0300848fea7209639a54eccb619-init/merged/dev/shm: invalid argument

[官方查询问题](https://bugs.centos.org/view.php?id=8493)

查询环境

内核版本

    # uname -r
    3.19.3-1.el7.elrepo.x86_64
系统版本

    # cat /etc/centos-release
    CentOS Linux release 7.0.1406 (Core)
docker版本

    # docker info
    Containers: 3
    Images: 9
    Storage Driver: overlay
     Backing Filesystem: xfs    <-------问题
    Execution Driver: native-0.2
    Logging Driver: json-file
    Kernel Version: 3.19.3-1.el7.elrepo.x86_64
    Operating System: CentOS Linux 7 (Core)
    CPUs: 2
    Total Memory: 3.859 GiB
    Name: localhost.localdomain
    ID: EBSN:R4M6:QU22:YT5G:FANT:SPGW:7QK3:KDVA:T5HX:TXPA:LYTM:MS5F
### 找问题
经过多次测试发现在相同内核、系统版本、docker版本有些机器有问题有些机器没有问题，最终发现是 Centos 提供的新文件系统 XFS 和 Overlay 兼容问题导致。

### 解决办法
1 将文件系统更换成 EXT4 -正常
2 使用 Device Mapper -正常
3 使用 Centos 为基础制作镜像-正常

# 参考
[五种存储驱动原理以及应用场景和性能对比测试](https://gold.xitu.io/entry/578060b0d342d30057c27b39)

[Docker存储驱动浅析](http://t.51gocloud.com/?p=1148)

[剖析Docker文件系统：Aufs与Devicemapper](http://www.infoq.com/cn/articles/analysis-of-docker-file-system-aufs-and-devicemapper/)

[Deep dive into Docker storage drivers](http://jpetazzo.github.io/assets/2015-03-03-not-so-deep-dive-into-docker-storage-drivers.html#1)

[Linux 内核中的 Device Mapper 机制](https://www.ibm.com/developerworks/cn/linux/l-devmapper/)

[Docker 环境 Storage Pool 用完解决方案：resize-device-mapper](http://segmentfault.com/a/1190000002931564)

[IBM Research Report](http://domino.research.ibm.com/library/cyberdig.nsf/papers/0929052195DD819C85257D2300681E7B/$File/rc25482.pdf)