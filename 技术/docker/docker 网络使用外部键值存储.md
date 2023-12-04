# docker 网络使用外部键值存储
## 准备项目

- docker 支持 consul、etcd、zookeeper等分布式键值存储。
- 集群中的每个主机必须正确配置 docker daemon。
- 集群主机名必须唯一，因为密钥值存储使用主机名来识别集群成员。
- docker swan 集群与 docker 外部键值存储相互冲突

## 设置一个键值存储
网络需要键值存储来保存有关的网络信息，包括服务发现、网络、endpoints、ip地址等。修改 docker daemon 守护进程配置时

- ` --cluster-store `配置键值存储位置。
- ` --cluster-advertise`配置广告ip
- 通过 `docker info` 查看 

## 创建网络
- 检查 