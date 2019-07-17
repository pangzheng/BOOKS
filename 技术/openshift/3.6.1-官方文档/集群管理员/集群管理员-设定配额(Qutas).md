# 集群管理员-设定配额(Qutas)
资源配额由 `ResourceQuota` 对象定义，提供了限制每个项目总资源消耗的限制。它可以按类型限制项目中可以创建的对象数量，以及该项目资源可能消耗的计算资源和存储总量。[计算资源更多信息](https://docs.openshift.org/3.6/dev_guide/compute_resources.html#dev-guide-compute-resources)
## 按配额进行资源管理
以下描述了可由配额管理的一组计算资源和对象类型。如果(失败或成功)中的 `status.phase` 为真，则 pod 处于终结状态。
### 计算资源
- `cpu`/`requests.cpu`

	在 `non-terminal` 状态下的所有 pod 上的 cpu 请求总数不能超过此值。此值和 `requests.cpu` 等价，可以互换。
- `memory`/`requests.memory`

	在 `non-terminal` 状态下的所有 pod 上的内存请求总数不能超过此值。此值和 `requests.memory` 等价，可以互换。
- `limits.cpu`

	在 `non-terminal` 状态下的所有 pod 上的 cpu 限制总数不能超过此值。
- `limits.memory`

	在 `non-terminal` 状态下的所有 pod 上的内存限制总数不能超过此值。

### 存储资源
- `requests.storage`

	任何状态下，所有持久化声明的存储总额不能超过此值。
- `persistentvolumeclaims`

	项目中可能存在的持计划声明总数。
- `<storage-class-name>.storageclass.storage.k8s.io/requests.storage`

	在任何状态下具有一个匹配存储类的所有持久化请求中的存储总和不能超过此值。
- `<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims`	

	与项目中可以存在的一个匹配的存储类的持久化声明的总数不能超过此值。

### 对象数量管理	
- `pods`

	项目中可能存在的 `non-terminal` 状态的 pod 总数。
- `replicationcontrollers `

	项目中可以存在的复制控制器的总数。
- `resourcequotas`

	项目中可以存在资源配额的总数。
- `services`

	项目中可以存在服务的总数
- `secrets`

	项目中可以存在 `secrets ` 的总数
- `configmaps`

	项目中可以存在 `configmaps ` 的总数
- `persistentvolumeclaims`

	项目中可以存在持久化声明的总数
- `openshift.io/imagestreams`

	项目中可以存在镜像流的总数

## 配额范围
每个配额可以有一组相关联的作用域。如果配额与枚举范围的交集匹配，则配额将权衡资源的使用情况。

将范围添加到配额会限制配额可应用的资源集。在允许集之外指定资源会导致验证错误。

- `Terminating `

	匹配的 pod 将 `spec.activeDeadlineSeconds >= 0`
- `NotTerminating`

	匹配的 pod 将 `spec.activeDeadlineSeconds` 是 `nil`
- `BestEffort`

	匹配具有 `BE` 性质 的 pod 的 cpu 和内存。[更多信息参考](https://docs.openshift.org/3.6/admin_guide/overcommit.html#qos-classes)
- `NotBestEffort `

	匹配没有 `BE` 性质的 pod 设定的 cpu 和内存

### 一个 `BE` 作用域限制配合的资源
- `pods`

### 一个 `Terminating, NotTerminating, NotBestEffort` 作用域限制配合的资源
- `pods`
- `memory`
- `requests.memory`
- `limits.memory`
- `cpu`
- `requests.cpu`
- `limits.cpu`

## 配额执行
在首次创建项目的资源配额后，该项目将限制创建可能违反配额限制的任何新资源的能力，直到其更新了使用情况统计信息。

创建配额并更新使用统计信息后，该项目接受创建新的内容。当创建或修改资源时，在请求创建或修改资源时，配额使用率会立即递增。

当删除资源时，配额使用会在下次完成重新计算项目的配额统计信息时递减。配置的时间两决定了需要多长时间将配额使用统计数据减少到他们当前观察到的系统价值。

如果项目修改超出配额使用限制，则服务器拒绝该操作，并向用户返回相应的错误消息，解释违反配额约束的情况以及系统当前观察到的使用情况统计信息。
## 请求与限制
在分配[计算资源](https://docs.openshift.org/3.6/dev_guide/compute_resources.html#dev-compute-resources)时，每个容器可以分为 cpu 和内存分别指定一个请求和限制值。配额对它们进行限制。

如果配额具有 ` requests.cpu` 或 `requests.memory` 指定的值，则要求每个传入的容器都必须明确请求这些资源。如果配额的值为 `limits.cpu` 或 `limits.memory` 指定的值，那么它要求每个传入的容器为这些资源执行一个明确的限制资源。
### 一个简单的资源配额定义
- 核心对象数量限制

```
vi core-object-counts.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: core-object-counts
spec:
  hard:
    configmaps: "10" # 项目限制总数为10的 configmap 的数量
    persistentvolumeclaims: "4"  # 项目限制总数为4的 PVC 的数量
    replicationcontrollers: "20" # 项目限制总数为20的 RC 的数量
    secrets: "10" # 项目限制总数为10的 secrets 的数量
    services: "10" # 项目限制总数为10的 services 的数量
```

- openshift 对象数量限制

```
vi openshift-object-counts.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: openshift-object-counts
spec:
  hard:
    openshift.io/imagestreams: "10" # 项目限制总数为10的 imagestreams 的数量 
```

- 计算资源限制

```
vi compute-resources.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    pods: "4" # 项目中可能存在 non-terminal 状态 pod 的总和
    requests.cpu: "1"  # 所有 non-terminal 状态 pod 中,cpu 请求的总和不能超过 1.
    requests.memory: 1Gi # 所有 non-terminal 状态 pod 中,内存请求的总和不能超过 1 Gi.  
    limits.cpu: "2" # 所有 non-terminal 状态 pod 中,cpu 限制的总和不能超过 2.
    limits.memory: 2Gi # 所有 non-terminal 状态 pod 中,内存限制的总和不能超过 2 Gi.
```

- BF 范围限制

```
vi besteffort.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: besteffort
spec:
  hard:
    pods: "1" # 具有 BF 服务质量可存在的 non-terminal 状态总和
  scopes:
  - BestEffort # 将配额限制为只有内存或 cpu 具有 BF 服务质量的容器。
```

- 长时服务

```
vi compute-resources-long-running.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources-long-running
spec:
  hard:
    pods: "4" # 状态为 non-terminal 的所有 pods 总数 
    limits.cpu: "4"  # 状态为 non-terminal 的所有 pods cpu 限制总量不能超过这个值。
    limits.memory: "2Gi" 状态为 non-terminal 的所有 pods 内存限制总量不能超过这个值。
  scopes:
  - NotTerminating #将配额限制为只有 spec.activeDeadlineSeconds 为 nil 的容器。例如这个配额不会为构建或部署的容器计费。
```

- 计算资源的时间限制

```
vi compute-resources-time-bound.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources-time-bound
spec:
  hard:
    pods: "2" 状态为 non-terminal 的所有 pods 总数
    limits.cpu: "1" 状态为 non-terminal 的所有 pods cpu 限制总量不能超过这个值。
    limits.memory: "1Gi" 状态为 non-terminal 的所有 pods 内存限制总量不能超过这个值。
  scopes:
  - Terminating 配额限制为只有匹配  spec.activeDeadlineSeconds >=0 的 pod。例如这个配额将计算部署和部署的 pod 数量，但不是像类似 web 服务或数据库那样的长时运行的 pod。
```

- 存储资源限制 

```
vi storage-consumption.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-consumption
spec:
  hard:
    persistentvolumeclaims: "10" #项目中持久化声明的总数
    requests.storage: "50Gi" #项目中所有持久化声明中，请求存储总和不能超过这个值。
    gold.storageclass.storage.k8s.io/requests.storage: "10Gi" #项目中所有持久化声明中，黄金类型的请求存储总和不能超过这个值。
    silver.storageclass.storage.k8s.io/requests.storage: "20Gi" #项目中所有持久化声明中，白银类型的请求存储总和不能超过这个值。
    silver.storageclass.storage.k8s.io/persistentvolumeclaims: "5" #项目中所有持久化声明中，白银类型的持久化声明的总数不能超过这个值。
    bronze.storageclass.storage.k8s.io/requests.storage: "0" #项目中所有持久化声明中，青铜类型的请求存储总和不能超过这个值。这个值是0说明不能创建这个请求存储。
    bronze.storageclass.storage.k8s.io/persistentvolumeclaims: "0" #项目中所有持久化声明中，青铜类型的持久化声明的总数不能超过这个值。设置零说明不能创建。
```

## 创建一个配额
创建一个配额，首先需要在文件中定义配额的规格，如 ` Sample Resource Quota` 中，然后将这个文件使用到对应的项目中。

	oc create -f <resource_quota_definition> [-n <project_name>]
例

	oc create -f resource-quota.json -n demoproject
## 查看配额
可以通过在 web 控制台中导航到项目的 `配额` 的页面，查看与项目配额中定义的任何硬限制相关的使用情况统计信息。也可以使用命令行查看。

- 首先获取项目定义的配额清单。例如对一个名为 `demoproject` 的项目

```
$ oc get quota -n demoproject
NAME                AGE
besteffort          11m
compute-resources   2m
core-object-counts  29m
```

- 然后，描述配额，例如 `core-object-counts` 配额

```
oc describe quota core-object-counts -n demoproject
Name:			core-object-counts
Namespace:		demoproject
Resource		Used	Hard
--------		----	----
configmaps		3	10
persistentvolumeclaims	0	4
replicationcontrollers	3	20
secrets			9	10
services		2	10
```

## 配置配额同步周期
当删除一组资源时，资源的同步时间范围由 `/etc/origin/master/master-config.yaml` 文件中的 `resource-quota-sync-period` 设定决定。

配额使用恢复之前，用户尝试重新使用资源时，可能会遇到问题。可以更改 `resource-quota-sync-period` 资源配额同步期限设定，以便在所需的时间(单位秒)重新生成一组资源，并使资源再次可用

```
kubernetesMasterConfig:
  apiLevels:
  - v1beta3
  - v1
  apiServerArguments: null
  controllerArguments:
    resource-quota-sync-period:
      - "10s"
```

进行任何更改后，重新启动 master 来激活设定。调整同步时间有助于创建资源并确定使用自动化时的资源使用情况。资源配额同步期间设置宗旨是平衡系统性能。减少同步周期可能导致的主机负载过重。
## 部署配置中对配额进行计费
如果已经为项目定义了配额，请参考[部署资源](https://docs.openshift.org/3.6/dev_guide/deployments/basic_deployment_operations.html#deployment-resources)了解
## 需要显示配额才能使用资源(技术预览)
如果配额不受管理，则用户对可消耗资源的数量没有限制。如:如果没有与黄金存储类相关的存储配额，项目可以创建的黄金存储量是无限的。

对于高成本的计算存储，管理员可能希望要求显示配额以消耗资源。例如，如果某个项目没有明确给出配额，则该项目的用户将无法创建该类型的任何存储。

为了要求明确的配额使用特定的资源，需要将下面内容增加到 `master-config.yaml`

```
admissionConfig:
  pluginConfig:
    ResourceQuota:
      configuration:
        apiVersion: resourcequota.admission.k8s.io/v1alpha1
        kind: Configuration
        limitedResources:
        - resource: persistentvolumeclaims #默认限制消费的组/资源
          matchContains:
        - gold.storageclass.storage.k8s.io/requests.storage #组/资源关联的配额所跟踪的资源名称，默认清咖滚下不会限制。
```
例子中，配额系统会拦截每个创建或更新的 PVC 操作。会检查配额所理解的资源会被消耗，乳沟项目中这些资源没有覆盖配额，该请求将被拒绝。如果用户创建使用存储类关联的存储 PVC 并没有在项目中使用配额，则请求拒绝。
		