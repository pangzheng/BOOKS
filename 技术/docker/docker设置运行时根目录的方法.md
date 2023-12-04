# Docker设置运行时根目录的方法

-g, --graph="/var/lib/docker"	设置Docker运行时根目录

Docker 的配置文件可以设置大部分的后台进程参数，在各个操作系统中的存放位置不一致
    
    # ubuntu 中的位置是：
    /etc/default/docker
    # centos 中的位置是：
    /etc/sysconfig/docker

我使用的是ubuntu14.04 在/etc/default/docker 添加或修改OPTIONS参数

    DOCKER_OPTS="--graph=/data/lib/docker"

然后service docker restart 重新启动Docer 的路径就改成/data/lib/docker了
