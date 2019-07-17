# Kubernetes 的本地持久卷进入 Beta 阶段
Kubernetes 1.10 中的 [Local Persistent Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#local) beta 功能使您可以利用 `StatefulSet` 中的本地磁盘。可以将直接连接的本地磁盘指定为 `PersistentVolumes`，并在 `StatefulSets` 中使用它们，这些对象具有之前仅支持远程卷类型的相同 `PersistentVolumeClaim` 对象。

持久存储对于运行有状态应用程序非常重要，Kubernetes 使用 `StatefulSets`，`PersistentVolumeClaims` 和 `PersistentVolumes` 支持这些工作负载。这些原语很好地支持远程卷类型，其中卷可以从群集中的任何节点访问，但不支持本地卷，其中只能从特定节点访问卷。随着在 Kubernetes 中运行更多工作负载的需求，在复制的有状态工作负载中使用本地快速SSD的需求也在增加。

## 解决hostPath挑战
通过 hostPath 卷访问本地存储的先前机制存在许多挑战。hostPath 卷很难在大规模生产中使用：

- 运营商在使用 hostPath 卷时需要关注本地磁盘管理，拓扑和各个 pod 的调度，并且无法使用许多 Kubernetes 功能（如StatefulSets）。
- 使用远程卷的 Helm 无法轻松移植到使用 hostPath 卷。Local Persistent Volumes 功能旨在解决 hostPath 卷的可移植性，磁盘记帐和调度问题。

## 放弃
在详细了解如何使用本地持久卷之前，请注意本地卷不适合大多数应用程序。
	
- 使用本地存储将您的应用程序绑定到该特定节点，使您的应用程序更难以安排。如果该节点或本地卷遇到故障并且无法访问，则该 pod 也将变得不可访问。
- 此外，许多云提供商不为本地存储提供广泛的数据持久性保证，因此您可能会在某些情况下丢失所有数据。

出于这些原因，大多数应用程序应继续使用高可用，可远程访问的持久存储。
## 合适的工作量
一些适合本地存储的用例包括：

- 缓存可以利用数据重力进行快速处理的数据集
- 分布式存储系统，用于跨多个节点分片或复制数据。示例包括像 Cassandra 这样的分布式数据存储，或者像 Gluster 或 Ceph 这样的分布式文件系统。

合适的工作负载可以容忍节点故障，数据不可用和数据丢失。它们为群集的其余部分提供关键的，对延迟敏感的基础结构服务，并且与其他工作负载相比应该以高优先级运行。

## 启用更智能的计划和卷绑定
管理员必须为本地持久卷启用更智能的调度。在创建本地 `PersistentVolumes` 的任何 `PersistentVolumeClaims` 之前，必须创建 `StorageClass` 并将 `volumeBindingMode` 设置为 `WaitForFirstConsumer`：

	kind: StorageClass
	apiVersion: storage.k8s.io/v1
	metadata:
	  name: local-storage
	provisioner: kubernetes.io/no-provisioner
	volumeBindingMode: WaitForFirstConsumer
此设置告诉 `PersistentVolume` 控制器不立即绑定 `PersistentVolumeClaim` 。相反，系统会等到安排需要使用卷的 Pod。然后，调度程序选择适当的本地 `PersistentVolume` 进行绑定，同时考虑 Pod 的其他调度约束和策略。这可确保初始卷绑定与任何Pod资源要求，选择器，关联和反关联策略等兼容。

请注意，测试版不支持动态配置。必须静态创建所有本地 `PersistentVolumes`。
## 创建本地持久卷
对于此初始测试版，必须首先由管理员对本地磁盘进行预分区，格式化和安装在本地节点上。还支持共享文件系统上的目录，但也必须在使用前创建。

设置本地卷后，可以为其创建 PersistentVolume 。在此示例中，本地卷安装在节点 “my-node” 上的 `/mnt/disks/vol1` 中：

	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: example-local-pv
	spec:
	  capacity:
	    storage: 500Gi
	  accessModes:
	  - ReadWriteOnce
	  persistentVolumeReclaimPolicy: Retain
	  storageClassName: local-storage
	  local:
	    path: /mnt/disks/vol1
	  nodeAffinity:
	    required:
	      nodeSelectorTerms:
	      - matchExpressions:
	        - key: kubernetes.io/hostname
	          operator: In
	          values:
	          - my-node
请注意，`PersistentVolume` 对象中有一个新的 `nodeAffinity` 字段：这是 Kubernetes 调度程序了解此 `PersistentVolume` 与特定节点关联的方式。`nodeAffinity` 是本地 `PersistentVolumes` 的必填字段。

当像这样手动创建本地卷时，唯一支持的 `persistentVolumeReclaimPolicy` 是“保留”。从 `PersistentVolumeClaim` 释放 `PersistentVolume` 时，管理员必须再次手动清理并设置本地卷以供重用。
## 自动创建和删除本地卷
手动创建和清理本地卷是一个很大的管理负担，因此我们编写了一个简单的本地卷管理器来自动化其中的一些部分。它可以在[外部存储仓库](https://github.com/kubernetes-incubator/external-storage/tree/master/local-volume)中作为可以在群集中部署的可选程序使用，包括有关如何运行它的说明和示例部署规范。

要使用此功能，必须首先由管理员设置本地卷并将其安装在本地节点上。管理员需要将本地卷装入本地卷管理器可识别的可配置“发现目录”中。支持共享文件系统上的目录，但必须将它们绑定到发现目录中。

此本地卷管理器监视发现目录，查找任何新的挂载点。管理器创建一个 `PersistentVolume` 对象，其中包含适当的 `storageClassName`，`path`，`nodeAffinity` 和它检测到的任何新安装点的容量。这些`PersistentVolume` 对象最终可以由 `PersistentVolumeClaims` 声明，然后安装在 Pod 中。

使用卷完成 Pod 并删除 `PersistentVolumeClaim` 后，本地卷管理器通过删除本地安装中的所有文件来清除本地安装，然后删除 `PersistentVolume` 对象。这会触发发现周期：为卷创建新的`PersistentVolume`，并且可以由新的 `PersistentVolumeClaim` 重用。

管理员最初设置本地卷装入后，此本地卷管理器将接管剩余的 `PersistentVolume` 生命周期，而无需任何进一步的管理员干预。 
## 在pod中使用本地卷
在所有管理员工作之后，用户如何将本地卷实际安装到他们的 Pod 中？幸运的是，从用户的角度来看，可以以与任何其他 `PersistentVolume` 类型完全相同的方式请求本地卷：通过 `PersistentVolumeClaim` 。只需在 `PersistentVolumeClaim` 对象中为本地卷指定适当的 `StorageClassName`，系统即可完成其余工作！

	kind: PersistentVolumeClaim
	apiVersion: v1
	metadata:
	  name: example-local-claim
	spec:
	  accessModes:
	  - ReadWriteOnce
	  storageClassName: local-storage
	  resources:
	    requests:
	      storage: 500Gi
或者在 `StatefulSet` 中作为 `volumeClaimTemplate`：

	kind: StatefulSet
	...
	 volumeClaimTemplates:
	  - metadata:
	      name: example-local-claim
	    spec:
	      accessModes:
	      - ReadWriteOnce
	      storageClassName: local-storage
	      resources:
	        requests:
	          storage: 500Gi
## 文档
Kubernetes 网站提供了[本地持久卷](https://kubernetes.io/docs/concepts/storage/volumes/#local)的完整文档。
## 未来的改进
到目前为止，本地持久性卷 beta 功能尚未完成。正在开发的一些值得注意的增强：

- 从1.10开始，本地原始块卷可用作 alpha 功能。这对于需要直接访问块设备和管理自己的数据格式的工作负载非常有用。
- 使用 LVM 动态配置本地卷正在设计中，未来版本将遵循 alpha 实现。只要工作负载的性能要求可以容忍共享磁盘，这将消除管理员当前对预分区，格式化和装载本地卷的需求。

## 补充功能
Pod 优先级和抢占是另一个 Kubernetes 功能，它是对本地持久卷的补充。当您的应用程序使用本地存储时，必须将其安排到本地卷所在的特定节点。您可以为本地存储工作负载提供高优先级，因此如果该节点没有足够的空间来运行您的工作负载，Kubernetes 可以抢占较低优先级的工作负载以为其腾出空间。

Pod 中断预算对于必须维持法定人数的工作负载也非常重要。为您的工作负载设置中断预算可确保它不会因自愿中断事件（例如升级期间的节点耗尽）而降至法定人数以下。

Pod 亲和性和反关联性可确保您的工作负载保持在同一位置或跨故障域分布。如果在单个节点上有多个本地持久卷可用，则最好指定 pod 反关联策略以跨节点分布工作负载。请注意，如果您希望多个 pod 共享同一本地持久卷，则无需指定 Pod 关联策略。调度程序了解本地持久卷的位置约束，并将 pod 调度到正确的节点。

## 转翻
[Local Persistent Volumes for Kubernetes Goes Beta](https://kubernetes.io/blog/2018/04/13/local-persistent-volumes-beta/)

	