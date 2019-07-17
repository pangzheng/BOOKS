# 集群管理员-超售QoS
容器可以指定计算资源请求和限制。

- 请求用于调度容器并提供最低的服务保证。
- 限制则限制了节点上最多消耗的计算资源

调度器程序尝试优化集群所有节点的计算资源使用。它将 pod 调度到特定的节点上，考虑到 pod 的计算资源请求和节点的可用容量。

请求和限制使管理员能够允许和管理节点上资源的超售，在开发环境中是可取的，在这种环境中，容量保证性能是可以接受的。

## 请求和限制
对于每个计算资源，容器可以指定资源请求和限制。根据请求进行调度策略，以确保节点具有足够的容量来满足请求值。如果一个容器规定了限制，但省略了请求，则这些请求默认为限制。容器不能超过节点上的指定限制。

限制的实施取决于计算资源的类型。如果一个容器没有请求或限制，那么该容器被调度到没有资源保证的节点。在实践中，容器能够消耗尽可能多的指定资源，并且有最低的本地优先级。在低资源的情况下，不指定资源请求的容器被赋予最低的服务质量(QOS)。
## 调整缓冲区块限制
如果 `Fluentd` 记录器无法很上大量日志，则需要增加计算资源值。

内存限制用于计算 `Fluentd` `buffer_queue_limit`，默认为 1MB 。如下:

```
buffer_queue_limit = resource memory limit / (number of output * buffer_chunk_size)
```

以下步骤可以调整可用资源

- 编辑 `Fluentd` 的守护进程

```
$ oc edit daemonset logging-fluentd

resources:
  limits:
    cpu: 100m
    memory: 512Mi
```

- 根据可用资源增加值

```
resources:
  limits:
    cpu: 150m
    memory: 1Gi
```

如果 `mux` 多路服务器位于传入日志的后面，相同的配置是可用的。内存限制用于计算多路复用的 (`mux`) `buffer_queue_limit`

	buffer_queue_limit = resource memory limit / (number of output * buffer_chunk_size)
	
- 配置 `mux` 部署配置

```
$ oc edit deploymentconfig logging-mux

resources:
  limits:
    cpu: 500m
    memory: 2Gi
```

- 根据可用资源增加值	

```
resources:
  limits:
    cpu: 600m
    memory: 2.5Gi
```

## 计算资源
计算资源是节点执行行为的特定资源类型。

- CPU(可压缩)

	一个容器保证了它所请求的 cpu 数量，另外还能够消耗节点上可用的多余 cpu，直到容器指定的任何限制。如果多个容器试图使用多余的 cpu，cpu 时间将根据每个容器请求的 cpu 数量进行分配。

	例如 一个容器请求 500m 的 cpu 时间，而另一个容器请求 250m 的 cpu 时间，则节点上可用的任何额外的 cpu 时间以 2:1 的比率在容器之间分配。如果一个容器指定了一个限制，它将被限制不适用比指定限制更多的 cpu。

	cpu 请求使用 linux 内核中的 CFS 共享支持来执行。默认情况下， cpu 限制在 100ms 测量间隔内使用 linux 内核中的 CFS 配额支持强制执行，但可以禁用。

- 内存(不可压缩)

	一个容器保证它所有请求的内存量。一个容器可能会使用比请求更多的内存，但一旦超过了请求的数量，它可能会在节点内存不足的情况下被终止。
	
	如果容器使用的内存少于所请求的内存，那么除非系统任务或守护进程需要比节点的资源预留中记录的内存更多的内存时，否则不会被终止。如果一个容器指定了一个内存限制，如果它超过了限制数量，它会立即被杀死。

## 服务质量 QoS
如果某个节点没有发出请求，或该节点上所有节点的限制总和超过了可用的机器容量，则该节点将被超售。

在超售的环境中，节点上的 pod 可能尝试使用比任何给定时间点可用的计算资源更多的计算资源。发生这种情况时，节点必须有限考虑将一个 pod 迁移。用于做出这一判断的设施被称为服务质量(QoS)类别。对于每个计算资源，一个容器按照优先级顺序分为三个 QoS 类别之一：

- 1 完全可靠(类型名称:`Guaranteed`)

	如果所有资源的限制和可选请求都被设置(不等于0)，并且它们相等，则容器被分类为保证(`Guaranteed `)
- 2 弹性比较可靠(类型名称:` `)

	如果所有资源的请求和选择的限制都被设定(不等于0)，并且它们不相等，则容器被分类为(`Burstable `)
- 3 尽力而为，不太可靠(类型名称:`BestEffort`)

	如果没有为任何资源设定请求和限制，则该容器被归类为尽力而为(`BestEffort`)
	
内存是不可压缩的资源，所以在内存不足的情况下，具有最低优先级的容器将会被杀死:

- `Guaranteed ` 的容器被认定是最高优先级，并保证只有当它们超出限制时才会被杀死，或者如果系统处于内存压力下，并没有可以被迁移的优先级容器，被杀。
- `Burstable` 在系统压力下一旦超过了它们的要求，就更有可能会被杀死，没有其他的 `BestEffort ` 容器存在，更是如此。
- `BestEffort` 容器被视为最低优先级。如果系统内存不足，将首先处理这些容器的进程。

## 配置主机以用于超售
调度是基于请求的资源，而配额和硬限制是指资源限制，可以设置为高于请求的资源。请求与限制之间的差异决定了超售的水平；例如，如果容器被赋予1 Gi 的内存请求并且2 Gi 的限制，则基于在节点上可用的 1 Gi 请求来调度容器，但是可以使用达到 2 Gi;所以这是 200% 的超售。

如果 ocp 管理员希望控制节点上的超售使用和管理容器的密度级别，则可以将 master 配置覆盖在项目用户容器上设置的请求和限制之间的比率。结合每个项目 `LimitRange` 指定限制的默认值，这将调整容器限制和请求，以达到期望的超售说平。

这要求在 `master-config.yaml` 中配置 `ClusterResourceOverride` 准入控制器(更改配置后，重启)：

```
kubernetesMasterConfig:
  admissionConfig:
    pluginConfig:
      ClusterResourceOverride: #这是插件名称，但一个插件名称要完全匹配被忽略  
        configuration:
          apiVersion: v1
          kind: ClusterResourceOverrideConfig
          memoryRequestToLimitPercent: 25  #(可选 1-100) 如果指定或默认了容器内存限制，则内存请求将被覆盖到此限制的百分比
          cpuRequestToLimitPercent: 25 #(可选 1-100) 如果指定或默认了容器 cpu 限制，则 cpu 请求将被覆盖到此限制的百分比    
          limitCPUToMemoryPercent: 200 #(可选，正整数) 如果已经指定或默认了容器内存限制，则将 cpu 限制覆盖到内存限制的百分比，100%的 1Gi 内存等于 1个 cpu 内核。这个是覆盖 cpu 请求之前处理的 
``` 
请注意，如果对容器没有设置限制，则这些覆盖不起作用。创建具有默认限制的 `LimitRange` 对象(每个项目或者项目模版)，以确保覆盖适用。

还要注意，覆盖后，容器限制和请求仍然必须由项目中的任何 `LimitRange` 对象进行验证。例如项目用户可以指定一个接近最小限制的限制，然后请求覆盖到最小限制以下，从而禁止 pod。这种用户体验营修改会在未来的版本中解决，但是现在请谨慎的配置此功能和 `LimitRange`。

配置后，可以用过编辑项目并添加一下注释来禁用每个项目覆盖(如，允许独立于覆盖的配置基础架构组件)

	quota.openshift.io/cluster-resource-override-enabled: "false"
## 配置节点超售
在超售环境中，正确的配置节点以提供最佳系统行为是非常重要的。
## 保留服务层级的内存(技术预览)
可以使用 `experimental-qos-reserved` 参数来指定某个特定的 QoS 级别中保留的内存百分比。此功能会尝试保留请求的资源，以便使用更高的 QoS 类中的 pod 请求的资源，从较低的 QoS 类中迁移 pod。

通过为更高的 QoS 级别预留资源，防止没有资源限制的 pod 在更高的 QoS 级别上侵占 Pod 所有资源。 

要配置 `experimental-qos-reserved`，请编辑该节点的 `/etc/origin/node/node-config.yaml` 文件

```
kubeletArguments:
  cgroups-per-qos:
  - true
  cgroup-driver:
  - 'systemd'
  cgroup-root:
  - '/'
  experimental-qos-reserved: #指定如何在 QoS 级别保留 pod 资源请求 
  - 'memory=50%'
```

ocp 使用 `experimental-qos-reserved` 参数，如下所示

- `experimental-qos-reserved=memory=100%` 的值将阻止 `Burstable` 和 `BestEffort` QoS类消耗更高 QoS 类所请求的内存。这增加了在  `Burstable` 和 `BestEffort` 工作负载引发 OOM 的风险，有利于增加 `Guaranteed` 突发工作负载内存保证。
- 实验性 `experimental-qos-reserved=memory=50%` 值将允许 `Burstable` 和 `BestEffort` QoS 类消耗更高 QoS 类所请求的一半内存

- 如果可用值为 `experimental-qos-reserved=memory=0%`，将允许`Burstable` 和 `BestEffort` QoS 类使用最多全节点可分配量，但是增加了 `Guaranteed `  工作负载高时无法访问请求的内存风险。这种情况有效的禁用了这个功能。

## 强制执行 CPU 限制
默认情况下，节点使用 linux 内核中的 `CFS` 配额支持强制执行执行 cpu 限制。如果不想在节点傻姑娘强制执行 cpu 限制，则可以通过修改节点配置文件 ` node-config.yaml` 来禁用其强制，包含一下内容

```
kubeletArguments:
  cpu-cfs-quota:
    - "false"
``` 
如果禁用 cpu 强制限制，则了解对节点影响很重要：

- 如果容器请求 cpu，它将继续由 linux 内核中的 cfs 共享执行。
- 如果一个容器没有明确的请求 cpu，但指定了一个限制，那么这个请求将被默认为指定限制，并被 linux 内核中的 cfs 共享执行。
- 如果一个容器为 cpu 执行了一个请求和限制，那么这个请求将有 linux 内核中的 cfs 共享执行，并且这个限制对节点没有影响。

## 为系统进程保留资源
调度程序基于 pod 请求确保节点上的所有 pod 都有足够的资源。它验证节点上容器的请求总和不能大于节点容量。它包括由节点启动的所有容器，但不包括在集群知识之外启动的容器或进程。

建议保留节点容量的一部分，以允许在节点上运行所需要的守护进程，如 sshd 等。特别是建议为内存等不可压缩资源预留资源。

如果要为非 pod 进程显示保留资源，有两种方法:

- 首先的方法是通过指定可用于调度的资源来分配节点的资源。请参考[分配节点资源](https://docs.openshift.org/3.6/admin_guide/allocating_node_resources.html#admin-guide-allocating-node-resources)
- 可以创建一个资源预留池，该预留空间除了保留集群在节点上调度的容量外，什么都不做。例如

```
apiVersion: v1
kind: Pod
metadata:
  name: resource-reserver
spec:
  containers:
  - name: sleep-forever
    image: gcr.io/google_containers/pause:0.8.0
    resources:
      limits:
        cpu: 100m #要在集群上位置的主机级守护程序节点上保留的 cpu 数量
        memory: 150Mi #要在集群上位置的主机级守护程序节点上保留的内存数量
```
可以将定义保存到文件中，如 `resource-reserver.yaml` ，然后放置到节点配置目录中，例如 `/etc/origin/node/` 或 `--config=<dir>` 位置。

此外，节点服务需要配置为通过命名节点配置文件，通常是 ` node-config.yaml` 中的 `kubeletArguments.config` 字段中的目录来从节点配置目录读取定义:

```
kubeletArguments:
  config:
    - "/etc/origin/node" #配置文件
```

使用 `resource-reserver.yaml` 文件，启动节点服务器也会启动永久休眠容器。调度程序考虑节点的剩余容量，相应的调整 pod 调度。

要删除资源库管理器，可以从节点配置目录中删除或移除 `resource-reserver.yaml` 文件。

## 内核可调标志
当节点启动时，它为内存管理设置正确确保内核可调标志。内核应该永远不会分配内存失败，除非它耗尽物理内存。为了确保这种行为，节点指示内核始终超售使用内存:

		sysctl -w vm.overcommit_memory=1
当内存不足时，节点还会指示内存不要慌张。相反，内核 OOM 杀手应该会根据优先级来终止进程。

		sysctl -w vm.panic_on_oom=0
以上设置应该已经设置在节点上，不需要进一步设置
## 关闭交换内存
可以在节点上默认禁用交换内存，以保证服务质量。否则，影响保证级 k8s 调度的 pod 从节点上超额订阅资源。如果两个保证级的容器已经达到其内存限制，则每个容器都可以使用交换内存。最后如果交换空间不够，系统被超额订购时，可能会终止进程。禁止交换空间:

	swapoff -a
无法禁用交换空间结果的节点没有意识到它们正在体验内存压力 `MemoryPressure`，导致 pod 没有收到它们正在调度请求中所需要的内存。因此，节点上会放置额外的容器，以进一步增加内存压力，最终增加遇到系统内存不足事件的风险。(oom)

如果启用了交换，任何超出资源处理驱动阈值的可用内存都无法按预期工作。充分利用资源处理的优势，允许在内存压力下将 pod 迁移，并重新安排在没有压力的代替节点上。				