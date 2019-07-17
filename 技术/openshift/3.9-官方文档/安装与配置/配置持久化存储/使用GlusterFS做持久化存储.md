# 使用 GlusterFS 做持久化存储
## 概览
GlusterFS(GlusterFS) 可以配置为 openshift 提供[持久化存储](https://docs.openshift.org/3.9/architecture/additional_concepts/storage.html#architecture-additional-concepts-storage)和动态配置。它既可以使用 openshift 内建的容器GlusterFS集群，也可以使用集群外部的非容器GlusterFS集群。
## 容器 GlusterFS
使用容器 GlusterFS，GlusterFS 直接在 openshift 节点上运行容器启动。这允许计算和存储可以使用同一组硬件来进行调度和运行。

![](./pic/docker-GlusterFS.png)

## 外部 GlusterFS
借助外部 GlusterFS ，GlusterFS 可以运行在自己的物理节点上，并由 GlusterFS 卷管理 REST 服务 [heketi](https://github.com/heketi/heketi) 进行实例管理。这个 heketi 服务可以独立运行或容器运行。容器化允许一种简单的机制为服务提供高可用性。本文将重点介绍 heketi 容器的配置。

### 独立 GlusterFS
如果客户现有环境中有独立的 GlusterFS 集群，则可以使用 openshift 的 GlusterFS 卷插件使用该集群上的卷。此解决方案是传统部署，其中应用程序在专用计算节点上运行，openshift 集群以及从其节点专用节点提供存储。

![](./pic/GlusterFS.png)

有关 [GlusterFS](https://docs.gluster.org/en/latest/Install-Guide/Overview/)更多信息，需要参考 [GlusterFS 安装指南](https://docs.gluster.org/en/latest/Install-Guide/Overview/) 和 [GlusterFS 管理指南](https://docs.gluster.org/en/latest/Administrator%20Guide/overview/)

	基础设施中的高可用性存储由底层存储提供商提供

### GlusterFS 卷
GlusterFS 卷提供了符合 POSIX 标准的文件系统，并且在其集群中的一个或多个节点上包含一个或多个“砖块”。砖块只是给定存储节点上的一个目录，通常是块存储设备的装入点。GlusterFS处理在每个卷的配置下，给定卷的文件的分发和复制。

建议使用 heketi 进行最常见的卷管理操作，例如创建，删除和调整大小。OpenShift 希望在使用GlusterFS 预配器时，heketi会出现。默认情况下，heketi 将创建三副本的卷，即每个文件在三个不同节点上具有三个副本的卷。因此，建议由Heketi使用的任何GlusterFS群集至少有三个节点可用。

GlusterFS 卷有许多可用功能，但它们超出了本文档的范围。

### gluster块体积
gluster-block 卷是可以通过 iSCSI 进行挂载的卷。这是通过在现有的 GlusterFS 卷上创建文件，然后通过 iSCSI 目标将该文件显示为块设备来完成的。这种 GlusterFS 卷被称为块托管卷。

gluster-block卷呈现出一种折衷。作为iSCSI目标消费者，gluster-block卷一次只能由一个节点/客户端挂载，这与 GlusterFS 卷相比可以由多个节点/客户端挂载。然而，将后端文件放在 GlusterFS 卷（例如元数据查找）上的操作通常会被转换为 GlusterFS 卷上通常要快得多的操作（例如读取和写入）。这对于某些工作负载可能会有显着的性能改进。

	目前，建议仅将OpenShift Logging和OpenShift Metric存储的gluster-block卷使用。
### Gluster S3存储
Gluster S3服务允许用户应用程序通过S3接口访问GlusterFS存储。该服务绑定到两个GlusterFS卷，一个用于对象数据，另一个用于对象元数据，并将传入的S3 REST请求转换为卷上的文件系统操作。建议在OpenShift内将该服务作为一个容器运行。

	目前，Gluster S3服务的使用和安装在技术预览中。
## 注意事项
本节涵盖了在 OpenShift 中使用 GlusterFS 时需要考虑的几个主题。

### 软件先决条件
要访问GlusterFS卷，该 `mount.glusterfs` 命令必须在所有可调度节点上可用。对于基于 RPM 的系统，必须安装 `glusterfs-fuse` 包装：

	# yum install glusterfs-fuse
如果节点上已安装 `glusterfs-fuse`，请确保已安装最新版本：

	# yum update glusterfs-fuse
### 硬件要求
在容器 GlusterFS 或外部 GlusterFS 集群中使用的任何节点均被视为存储节点。尽管单个节点不能位于多个组中，但存储节点可以分组到不同的群集组中。对于每组存储节点：

- 每组至少需要三个存储节点。
- 每个存储节点必须至少具有8 GB的内存。这是为了允许运行GlusterFS pod以及其他应用程序和底层操作系统。
	- 每个GlusterFS卷还会在其存储群集中的每个存储节点上占用大约30 MB的内存。内存的总量应根据需要或预期的并发卷数来确定。
- 每个存储节点必须至少有一个原始块设备，而没有当前的数据或元数据。这些块设备将全部用于GlusterFS存储。确保以下内容不存在：
	- 分区表（GPT或MSDOS）
	- 文件系统或残留文件系统签名
	- 前卷组和逻辑卷的LVM2签名
	- LVM2物理卷的LVM2元数据

如果有疑问，`wipefs -a <device>` 应该清除上述任何一项。

	建议计划两个群集：一个专用于存储基础设施应用程序（例如OpenShift容器注册表），另一个专用于存储常规应用程序。这将需要总共六个存储节点。本建议旨在避免对I / O性能和批量创建的潜在影响。
### 存储大小
每个GlusterFS集群都必须根据将使用其存储的预期应用程序的需求进行调整。例如，[OpenShift Logging](https://docs.openshift.org/3.9/install_config/aggregate_logging_sizing.html#install-config-aggregate-logging-sizing) 和 [OpenShift Metrics](https://docs.openshift.org/3.9/install_config/cluster_metrics.html#capacity-planning-for-openshift-metrics) 都有可用的大小指南 。

一些额外的事情要考虑的是：

- 对于每个容器 GlusterFS 或外部 GlusterFS 集群，缺省行为是使用三个请求向复制创建 GlusterFS卷。因此，计划的总存储量应该是所需的容量乘以三。
	- 例如，每个heketi实例都会创建一个 `heketidbstorage` 大小为2 GB 的卷，在存储群集中的三个节点上总共需要6 GB的原始存储空间。此容量始终是必需的，因此需要考虑计算大小。
	- 像集成的OpenShift 容器镜像仓库这样的应用程序，多个实例之间共享一个GlusterFS卷。
- gluster-block卷需要GlusterFS块托管卷的存在，其容量足以容纳任何给定块卷的容量的全部大小。
	- 默认情况下，如果不存在这样的块托管卷，则会以设定的大小自动创建一个。此大小的默认值为100 GB。如果群集中没有足够的空间来创建新的数据块托管卷，则创建块卷将失败。自动创建行为和自动创建的卷大小都是可配置的。
	- 具有多个使用gluster-block卷的实例的应用程序（例如OpenShift Logging和OpenShift Metrics）将为每个实例使用一个卷。
- Gluster S3服务绑定到两个GlusterFS卷。在通过[高级安装程序](https://docs.openshift.org/3.9/install_config/install/advanced_install.html#install-config-install-advanced-install)的默认安装中 ，这些卷每个都是1 GB，共占用6 GB的原始存储空间。

### 卷操作行为
批量操作（例如创建和删除）可能会受到各种环境条件的影响，并可能影响应用程序。

- 如果应用程序pod请求动态配置的GlusterFS持久性卷声明（PVC），则可能需要考虑额外的时间以创建卷并绑定到相应的PVC。这会影响应用程序pod的启动时间。

		GlusterFS卷的创建时间根据卷的数量线性缩放。例如，如果使用推荐的硬件规格对群集中的100个卷进行分区，则每个卷需要约6秒才能创建，分配并绑定到一个群集。
- 当PVC被删除时，该操作将触发删除底层的GlusterFS卷。虽然PVC会立即从 `oc get pvc` 输出中消失 ，但这并不意味着卷被完全删除。GlusterFS体积只能被认为删除时，不会在用于命令行输出显示 `heketi-cli volume list` 和 `gluster volume list`。

		删除GlusterFS卷并回收其存储所需的时间取决于GlusterFS卷的数量并线性扩展。虽然挂起的卷删除不会影响正在运行的应用程序，但存储管理员应该知道并能够估计它们将花费多长时间，特别是在调整资源消耗规模时。

### 卷安全
本节介绍GlusterFS卷的安全性，包括可移植操作系统接口[用于Unix]（POSIX）权限和SELinux注意事项。了解[卷安全性](https://docs.openshift.org/3.9/install_config/persistent_storage/pod_security_context.html#install-config-persistent-storage-pod-security-context)，POSIX 权限和 SELinux 的基础知识是必须知道的。

#### POSIX权限
GlusterFS卷提供符合POSIX标准的文件系统。因此，可以使用标准命令行工具（如`chmod` 和`chown`）来管理访问权限。

对于容器式GlusterFS和外部GlusterFS，还可以指定在创建卷时拥有卷的根的组ID。对于静态配置，这是作为 `heketi-cli` 卷创建命令的一部分指定的：

	$ heketi-cli volume create --size=100 --gid=10001000

	与此卷关联的PersistentVolume必须使用组标识进行注释，以便使用PersistentVolume的Pod可以访问文件系统。此注释采用以下形式：

	pv.beta.kubernetes.io/gid：“<GID>”---

对于动态配置，配置器会自动生成并应用组ID。可以使用 `gidMin`和`gidMax` StorageClass参数控制从中选择该组ID的范围（请参阅 [Dynamic Provisioning](https://docs.openshift.org/3.9/install_config/persistent_storage/persistent_storage_glusterfs.html#provisioning-dynamic)）。配置程序还负责使用组ID标识生成的PersistentVolume。

### SELINUX的
默认情况下，SELinux不允许从pod到远程GlusterFS服务器写入数据。要启用在启用SELinux的情况下写入GlusterFS卷，请在运行GlusterFS的每个节点上运行以下命令：

	$ sudo setsebool -P virt_sandbox_use_fusefs on 
		# -P选项使 bool 在重新启动之间保持不变。
		# virt_sandbox_use_fusefs 布尔值由 docker-selinux 包定义。如果出现错误，说明未定义，请确保已安装此软件包。
## 安装
对于独立的GlusterFS，没有组件安装需要与OpenShift一起使用。OpenShift带有一个内置的GlusterFS卷驱动程序，允许它使用现有集群上的现有卷。有关如何使用现有卷的更多信息，请参阅[配置](https://docs.openshift.org/3.9/install_config/persistent_storage/persistent_storage_glusterfs.html#provisioning)。

对于容器GlusterFS和外部GlusterFS，建议使用[高级安装程序](https://docs.openshift.org/3.9/install_config/install/advanced_install.html#install-config-install-advanced-install)来安装所需的组件。
### 外部GlusterFS：安装GlusterFS节点
对于外部GlusterFS，每个GlusterFS节点必须具有适当的系统配置（例如防火墙端口，内核模块），并且GlusterFS服务必须正在运行。这些服务不应该进一步配置，并且不应该形成可信存储池。

GlusterFS节点的安装超出了本文档的范围。有关更多信息，请参阅[GlusterFS安装指南](https://docs.gluster.org/en/latest/Install-Guide/Overview/)。



		