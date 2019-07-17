# Mount propagation(挂载命名空间的传播)
## 特性说明
挂载传播允许将容器挂载的卷共享给相同 Pod 中的其他容器或者可以共享到同一节点上的其他 Pod。
一个卷的挂载传播由参数 `Container.volumeMounts` 中的`mountPropagation` 字段控制。

可选值有：

- None(默认模式)

	此卷挂载不会接收到任何后续挂载到该卷或是挂载到该卷的子目录下的挂载。以类似的方式，在主机上不会显示 Container 创建的装载。
此模式等同于 [Linux 内核](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)文档中所述的 private 传播。

- HostToContainer

	此卷挂载将会接收到任何后续挂载到该卷或是挂载到该卷的子目录下的挂载。
换句话说，如果主机在卷挂载中挂载任何内容，则 Container 将看到它挂载在那里。
类似地，如果任何具有 Bidirectional 挂载传播设置的 Pod 挂载到同一个卷中，那么具有 HostToContainer 挂载传播的 Container 将会看到它。
此模式等同于Linux内核文档中描述的 rslave 挂载传播。

- Bidirectional 

	此卷挂载的行为与 HostToContainer 挂载相同。此外，Container 创建的所有卷挂载都将传播回主机和所有使用相同卷的 Pod 的所有容器。
此模式的典型用例是具有 Flexvolume 或 CSI 驱动程序的Pod 需要使用 hostPath 卷模式在主机上挂载内容。
此模式等同于Linux内核文档中描述的 rshared 安装传播。

	注意：Bidirectional 挂载传播可能很危险。它可能会损坏主机操作系统，因此只允许在特权容器中使用它。强烈建议熟悉Linux内核行为。此外，容器在容器中创建的任何卷装入必须在终止时由容器销毁（卸载）。

Bidirectional 一些使用场景：

- 在不同的 pod 之间共享设备，其中挂载发生在 pod 中，但是在 pod 之间共享。
- 从容器内部附加设备。例如，从容器内部附加 ISCSI 设备。这时候因为如果容器死掉，主机将不能获得所需的信息（除非使用双向安装传播）来正确刷新写入和分离设备。

## 例

	deployment:
	  containers:
	  - image: gcr.io/google_containers/busybox:1.24
	    name: reader
	    volume:
	    - mount: /usr/test-pod
	      store: local-vol
	      propagation: bidirectional <-- 这里
	  name: local-test-reader
	  version: extensions/v1beta1
	  volumes:
	    local-vol: pvc:example-local-claim

## 配置
在挂载传播可以在某些部署（CoreOS，RedHat/Centos，Ubuntu）上正常工作之前，必须在 Docker 中正确配置挂载共享，如下所示。

- 编辑 Docker 的 systemdservice 文件。设置`MountFlags` 如下：

		MountFlags=shared
- 或者，如果存在，则删除 `MountFlags=slave`。然后重启 Docker 守护进程：

		$ sudo systemctl daemon-reload
		$ sudo systemctl restart docker

## 场景
日志收集通过 sidecar 方式，将容器日志 mount 到主机目录。由于日志量大，所以经常写满磁盘。解决方案是通过实现 `log-tracer`，专门根据 pod 日志挂载信息清理过期日志。该组件通过 daemonset 方式部署。如果 mount 设置 `private` 时，将缺乏广播通知。新增 pod 后，部分日志挂载无法感知，无法彻底清理。
## 原理
该特性并不是k8s或是容器实现的。本质上是 `mount namespace` 的特性。

`mount namespace` 通过隔离文件系统挂载点对隔离文件系统提供支持，它是历史上第一个 `Linux namespace`，所以它的标识位比较特殊，就是 `CLONE_NEWNS`。隔离后，不同`mount namespace` 中的文件结构发生变化也互不影响。你可以通过 `/proc/[pid]/mounts` 查看到所有挂载在当前`namespace` 中的文件系统，还可以通过 `/proc/[pid]/mountstats` 看到 `mount namespace` 中文件设备的统计信息，包括挂载文件的名字、文件系统类型、挂载位置等等。

进程在创建 `mount namespace` 时，会把当前的文件结构复制给新的 `namespace` 。新 `namespace` 中的所有mount 操作都只影响自身的文件系统，而对外界不会产生任何影响。这样做非常严格地实现了隔离，但是某些情况可能并不适用。比如父节点 `namespace` 中的进程挂载了一张 `CD-ROM` ，这时子节点 `namespace` 拷贝的目录结构就无法自动挂载上这张 `CD-ROM`，因为这种操作会影响到父节点的文件系统。

2006 年引入的挂载传播 `mount propagation` 解决了这个问题，挂载传播定义了挂载对象 `mount object` 之间的关系，系统用这些关系决定任何挂载对象中的挂载事件如何传播到其他挂载对象[参考](http://www.ibm.com/developerworks/library/l-mount-namespaces/)：。所谓传播事件，是指由一个挂载对象的状态变化导致的其它挂载对象的挂载与解除挂载动作的事件。

- 共享关系（share relationship）。

	如果两个挂载对象具有共享关系，那么一个挂载对象中的挂载事件会传播到另一个挂载对象，反之亦然。
- 从属关系（slave relationship）。

	如果两个挂载对象形成从属关系，那么一个挂载对象中的挂载事件会传播到另一个挂载对象，但是反过来不行；在这种关系中，从属对象是事件的接收者。
	
一个挂载状态可能为如下的其中一种：

- 共享挂载（shared）

	在默认情况下，所有挂载都是私有的。可以用以下命令将挂载对象显式地标为共享挂载：

		mount --make-shared <mount-object>
例如，如果 / 上的挂载必须是共享的，那么执行以下命令：

		mount --make-shared /
从共享挂载克隆的挂载对象也是共享的挂载；它们相互传播挂载事件。
- 从属挂载（slave）

	通过执行以下命令，可以显式地将一个共享挂载转换为从属挂载：

		mount --make-slave <shared-mount-object>
	
	从从属挂载克隆的挂载对象也是从属的挂载，它也从属于原来的从属挂载的主挂载对象。
- 共享/从属挂载（shared and slave）
- 私有挂载（private）

	通过执行以下命令，可以将挂载对象标为私有的：

		mount --make-private <mount-object>
- 不可绑定挂载（unbindable）

	通过执行以下命令，可以将挂载对象标为不可绑定的：

		mount --make-unbindable <mount-object>
		
最后，这些设置都可以递归地应用，这意味着它们将应用于目标挂载之下的所有挂载。

	mount --make-rshared /
将 / 之下的所有挂载转换为共享挂载。

## 参考
[谈谈k8s1.12新特性--Mount propagation(挂载命名空间的传播)](https://segmentfault.com/a/1190000016567617)
[官方文档](https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation)
[Docker背后的内核知识——Namespace资源隔离](http://www.infoq.com/cn/articles/docker-kernel-knowledge-namespace-resource-isolation)