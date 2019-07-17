# DOCKER 任务
OKD 用 Docker 启动任意数量的容器组成的 pod 中运行应用程序。

作为集群管理员，有时 Docker 需要一些额外的配置才能有效地运行 pod.
## 增加 Docker 存储空间
增加可用存储量可确保持续部署而不会出现任何中断。为此，必须提供包含适当可用容量磁盘空间。
### 撤离节点
步骤

1. 作为集群管理员，允许从节点撤出任何 pod 并禁用该节点调度其他 pod

		$ NODE=ose-app-node01.example.com
		$ oc adm manage-node ${NODE} --schedulable=false
		NAME                          STATUS                     AGE       VERSION
		ose-app-node01.example.com   Ready,SchedulingDisabled   20m       v1.6.1+5115d708d7
		
		$ oc adm drain ${NODE} --ignore-daemonsets
		node "ose-app-node01.example.com" already cordoned
		pod "perl-1-build" evicted
		pod "perl-1-3lnsh" evicted
		pod "perl-1-9jzd8" evicted
		node "ose-app-node01.example.com" drained
		
	如果有运行本地卷的容器无法迁移，请运行以下命令 `oc adm drain ${NODE} --ignore-daemonsets --delete-local-data`
-  列出节点上的 pod 以验证它们是否已被删除

		$ oc adm manage-node ${NODE} --list-pods
		
		Listing matched pods on node: ose-app-node01.example.com
		
		NAME      READY     STATUS    RESTARTS   AGE
- 对每个节点重复前两个步骤。

有关疏散和排空pod或节点的更多信息，请参阅[节点维护](https://docs.okd.io/3.10/day_two_guide/host_level_tasks.html#day-two-guide-node-maintenance)。