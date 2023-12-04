#docker CPU MEMORY资源限制

    文档信息
    创建人 刘金烨
    邮件地址 jyliu@dataman-inc.com
    时间 2015-08-25

## 1. 测试环境

```
操作系统: ubuntu14.04 
配置：4核 16G内存
docker version: 1.8.1
ubuntu包含stress工具的docker镜像 # stress是进行内存及CPU资源使用测试的工具
```

## 2. 内存限制测试


### 2.1 启动指定100M内存，不使用swap的容器，执行stress占用99M内存
    
    
```
docker run -it --rm --name test1 --memory=100M --memory-swappiness=0 demoregistry.dataman-inc.com/library/ubuntu-stress stress --vm 1 --vm-bytes 99M --vm-hang 0
```

在另一终端连接主机执行
	
```
docker stats test1
```

结果如下：

CONTAINER | CPU % | MEM USAGE/LIMIT| MEM % | NET I/O
----------|-------|----------------|-------|--------
test1 | 0.00% | 104 MB/104.9 MB | 99.15% | 648 B/738 B
	

### 2.2 启动指定100M内存,不使用swap的容器，执行stress占用100M内存
    
```
docker run -it --rm --name test1 --memory=100M --memory-swappiness=0 demoregistry.dataman-inc.com/library/ubuntu-stress stress --vm 1 --vm-bytes 100M --vm-hang 0
```
启动后，docker进程被kill，报下面错误

```
WARNING: Your kernel does not support swap limit capabilities, memory limited without swap.
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: FAIL: [1] (416) <-- worker 8 got signal 9
stress: WARN: [1] (418) now reaping child worker processes
stress: FAIL: [1] (422) kill error: No such process
stress: FAIL: [1] (452) failed run completed in 0s
```
	
### 结论：不使用swap内存，docker --memory参数限制内存即为docker内程序最大使用内存


## 3. CPU限制测试
### 3.1 docker使用--cpu-period --cpu-quota 限制cpu使用比例的容器

启动限制CPU使用20%的docker容器

```
 docker run -it --rm  --name test1  --cpu-period=10000 --cpu-quota=2000 demoregistry.dataman-inc.com/library/ubuntu-stress stress  --cpu 4
```
CONTAINER|CPU %|MEM USAGE/LIMIT|MEM %|NET I/O
----------|-------|----------------|-------|--------
test1|20.04%|274.4 kB/16.83 GB| 0.00%|648 B/738 B

启动限制CPU使用 200% 的docker容器

```
 docker run -it --rm  --name test1  --cpu-period=10000 --cpu-quota=20000 demoregistry.dataman-inc.com/library/ubuntu/stress stress  --cpu 4
```
CONTAINER|CPU %|MEM USAGE/LIMIT|MEM %|NET I/O
----------|-------|----------------|-------|--------
test1|184.85%|274.4 kB/16.83 GB| 0.00%|648 B/738 B


启动限制CPU使用 600% 的docker容器(超售)

```
docker run -itd   --name test1  --cpu-period=10000 --cpu-quota=60000 demoregistry.dataman-inc.com/library/ubuntu-stress stress  --cpu 6
```

CONTAINER|CPU %|MEM USAGE/LIMIT|MEM %|NET I/O
----------|-------|----------------|-------|--------
test1|399.59%|483.3 kB/16.83 GB|0.00%|78 B/688 B

### 3.2 docker 使用 -c or --cpu-shares 划分CPU比例限制

cpu-shares 默认1024，启动一个cpu-shares 512的后台容器

```
docker run -itd    --name test1  -c 512 demoregistry.dataman-inc.com/library/ubuntu-stress stress  --cpu 4
```
结果如下：

CONTAINER|CPU %|MEM USAGE/LIMIT|MEM %|NET I/O
----------|-------|----------------|-------|--------
test1|400.54%|454.7 kB/16.83 GB|0.00%|648 B/738 B

再启动一个 cpu-shares 256 的后台容器

```
docker run -itd    --name test2  -c 256 demoregistry.dataman-inc.com/library/ubuntu-stress stress  --cpu 4
```
结果如下：

CONTAINER|CPU %|MEM USAGE/LIMIT|MEM %|NET I/O
----------|-------|----------------|-------|--------
test1|266.19%|241.7 kB/16.83 GB|0.00%|1.038 kB/738 B
test2|132.07%|241.7 kB/16.83 GB|0.00%|648 B/738 B

再启动一个 cpu-shares 128的后台容器

```
docker run -itd    --name test3  -c 128 demoregistry.dataman-inc.com/library/ubuntu-stress stress  --cpu 4
```
结果如下：

CONTAINER|CPU %|MEM USAGE/LIMIT|MEM %|NET I/O
----------|-------|----------------|-------|--------
test1|227.61%|241.7 kB/16.83 GB|0.00%|1.358 kB/738 B
test2|113.45%|241.7 kB/16.83 GB|0.00%|968 B/738 B
test3|56.98%|413.7 kB/16.83 GB|0.00%|578 B/668 B

### 结论：
### docker 使用--cpu-period --cpu-quota 限制CPU使用时，cpu-period为预设的单个CPU总资源量，cpu-quota为容器CPU使用占比，可以超售
### docker 使用-c or --cpu-shares 划分CPU使用比例，如果CPU在空闲状态，再有的容器会把CPU资源占满，如果有新的容器进程，按指定的cpu-shares数值比例划分CPU资源

参考资料：
	
	http://dl528888.blog.51cto.com/2382721/1659972
	https://docs.docker.com/reference/run/
