# k8s 调度
## 调度流程任务
- 客户端提交创建请求
		
	通过 API Server 的 Restful API，也可以使用 `kubectl` 命令行工具。支持的数据类型包括 JSON 和 YAML。
- API Server 处理用户请求

	存储 Pod 数据到 etcd
- 调度器通过 API Server 查看未绑定的 Pod

	尝试为 Pod 分配主机
- 过滤主机

	调度器用一组规则过滤掉不符合要求的主机。比如 Pod 指定了所需要的资源量，那么可用资源比 Pod 需要的资源量少的主机会被过滤掉。规则可以在 `kubernetes/plugin/pkg/scheduler/algorithm/predicates`找到,可以通过配置修改 kubernetes 默认过滤规则
	
	- NoDiskConflict

		检查在此主机上是否存在卷冲突。如果这个主机已经挂载了卷，其它同样使用这个卷的 Pod 就不能调度到这个主机上。因GCE,Amazon EBS, and Ceph RBD使用的规则如下：
		- GCE 允许同时挂载多个卷，只要这些卷都是只读的
		- Amazon EBS 不允许不同的Pod挂载同一个卷
		- Ceph RBD 不允许任何两个 Pods 分享相同的 monitor，match pool 和 image
	- NoVolumeZoneConflict

		检查给定的 zone 限制前提下，检查如果在此主机上部署 Pod 是否存在卷冲突。假定一些 volumes 可能有 zone 调度约束， VolumeZonePredicate 根据 volumes 自身需求来评估 Pod 是否满足条件。必要条件就是任何 volumes 的 zone-labels 必须与节点上的 zone-labels 完全匹配。节点上可以有多个 zone-labels 的约束（比如一个复制卷可能会允许进行区域范围内的访问）。目前，这个只对 PersistentVolumeClaims 支持，而且只在 PersistentVolume 的范围内查找标签。处理在Pod 的属性中定义的 volumes（即不使用 PersistentVolume ）有可能会变得更加困难，因为要在调度的过程中确定 volume 的zone，这可能会需要调用云提供商。
	- PodFitsResources

		检查主机的资源是否满足 Pod 的需求。注意:这里是根据实际已经分配的资源量做调度，而不是使用已实际使用的资源量做调度。
	- PodFitsHostPorts

		检查 Pod 内每一个容器所需的 HostPort 是否已被其它容器占用。如果有所需的 HostPort 不满足需求，那么 Pod 不能调度到这个主机上
	- HostName

		检查主机名称是不是 Pod 指定的 HostName
	- MatchNodeSelector

		检查主机的标签是否满足 Pod 的 `*nodeSelector*` 属性需求
	- MaxEBSVolumeCount

		确保已挂载的 EBS 存储卷不超过设置的最大值。默认值是39。它会检查直接使用的存储卷，和间接使用这种类型存储的 PVC。计算不同卷的总目，如果新的 Pod 部署上去后卷的数目会超过设置的最大值，那么 Pod 不能调度到这个主机上。
	- MaxGCEPDVolumeCount

		确保已挂载的 GCE 存储卷不超过设置的最大值。默认值是16。规则同上。
- 主机打分

	对筛选出的符合要求的主机进行打分，在主机打分阶段，调度器会考虑一些整体优化策略，最终选择一个分值最高的主机部署 Pod。kubernetes 用一组优先级函数处理每一个待选的主机 `kubernetes/plugin/pkg/scheduler/algorithm/priorities`。每一个优先级函数会返回一个 0-10 的分数，分数越高表示主机越“好”， 同时每一个函数也会对应一个表示权重的值。最终主机的得分用以下公式计算得出

		finalScoreNode = (weight1 * priorityFunc1) + (weight2 * priorityFunc2) + … + (weightn * priorityFuncn)
优先级函数包括

	- LeastRequestedPriority

		如果新的 Pod 要分配给一个节点，这个节点的优先级就由节点空闲的那部分与总容量的比值来决定。
			
			（总容量-节点上Pod的容量总和-新Pod的容量）/总容量）
		CPU和memory权重相当，比值最大的节点的得分最高。需要注意的是，这个优先级函数起到了按照资源消耗来跨节点分配 Pods 的作用。计算公式如下：

			cpu((capacity – sum(requested)) * 10 / capacity) + memory((capacity – sum(requested)) * 10 / capacity) / 2
	- BalancedResourceAllocation

		尽量选择在部署 Pod 后各项资源更均衡的机器。`*BalancedResourceAllocation*` 不能单独使用，而且必须和`*LeastRequestedPriority*` 同时使用，它分别计算主机上的 cpu 和 memory 的比重，主机的分值由 cpu 比重和 memory 比重的“距离”决定。计算公式如下：
			
			score = 10 – abs(cpuFraction-memoryFraction)*10
	- SelectorSpreadPriority

		对于属于同一个 service、replication controller 的 Pod，尽量分散在不同的主机上。如果指定了区域，则会尽量把 Pod 分散在不同区域的不同主机上。调度一个 Pod 的时候，先查找 Pod 对于的 service 或者 replication controller，然后查找 service 或 replication controller 中已存在的 Pod，主机上运行的已存在的 Pod 越少，主机的打分越高。
	- CalculateAntiAffinityPriority

		对于属于同一个service的Pod，尽量分散在不同的具有指定标签的主机上。
	- ImageLocalityPriority

		根据主机上是否已具备 Pod 运行的环境来打分。ImageLocalityPriority 会判断主机上是否已存在 Pod 运行所需的镜像，根据已有镜像的大小返回一个0-10的打分。如果主机上不存在Pod所需的镜像，返回0；如果主机上存在部分所需镜像，则根据这些镜像的大小来决定分值，镜像越大，打分就越高。
	- NodeAffinityPriority（Kubernetes1.2实验中的新特性）

		Kubernetes 调度中的亲和性机制。Node Selectors（调度时将 Pod 限定在指定节点上），支持多种操作符（In, NotIn, Exists, DoesNotExist, Gt, Lt），而不限于对节点labels的精确匹配。另外，Kubernetes 支持两种类型的选择器：
		- “hard（requiredDuringSchedulingIgnoredDuringExecution）”选择器

			它保证所选的主机必须满足所有Pod对主机的规则要求。这种选择器更像是之前的 nodeselector，在 nodeselector 的基础上增加了更合适的表现语法。
		- “soft（preferresDuringSchedulingIgnoredDuringExecution）”选择器

			它作为对调度器的提示，调度器会尽量但不保证满足NodeSelector的所有要求
- 选择主机

	选择打分最高的主机，进行 binding 操作，结果存储到 etcd 中。
- 所选主机对于的 kubelet 根据调度结果执行 Pod 创建操作

## 多层次资源隔离
Kubernetes 对资源隔离的设计，有 3 个层次的资源限制方式，分别在 Container、Pod、Namespace。
	
- Container 层次主要利用容器本身的支持，比如 Docker 对CPU、内存等的支持；
- Pod 方面可以限制系统内创建 Pod 的资源范围，比如最大或者最小的 CPU、memory 需求；
- Namespace 层次就是对用户级别的资源限额了，包括 CPU、内存，还可以限定 Pod、rc、service 的数量。
	
![](./pic/resources_limit.jpg)

- Resources Requests Limit(容器/pod)

	Resources Requests Limit 针对容器层，可以通过 `spec.container[].resources.requests` 和 `spec.container[].resources.limits` 两个参数管理容器的 cpu 和内存资源。
	- requests
		- requests 用于 schedule 阶段，scheduler 在调度 pod 时会保证所有 pod 的 requests 总和小于 node 能提供的计算能力
			- requests.cpu 会被转换成 `docker run` 的 `–cpu-shares` 参数。该参数与 cgroup `cpu.shares` 功能相同，具体为： 
				- 用来设置容器的 cpu 使用的相对权重。
				- 该参数只有在 CPU 资源不足时生效，系统会根据容器 requests.cpu 的比例来分配 cpu 资源给该容器。例如两个container 设置了相同的 requests.cpu，当两者抢占 CPU 资源时，系统会给两个 container 分配相同比例的CPU资源。
				- CPU资源充足时，requests.cpu 不会限制单个 container 占用的最大值，即单个 container 可以独占CPU资源。
			- requests.memory 没有对应的 `docker run` 参数，只作为 kubernetes 调度依据。
		- 可以使用 requests 来设置各容器需要的最小资源，使得 kubernetes 可以将更多的 pod 分配到一个 node 来实现超卖。
		- *spec.container[].resources.requests.cpu*
		- *spec.container[].resources.requests.memory*
		- requests未设置时，默认与limits相同。
	- limits
		- limits 用于限制运行时容器占用的资源。
			- limits.cpu 会被转换成 `docker run` 的 `–cpu-quota` 参数。该参数与 cgroup `cpu.cfs_quota_us` 功能相同，具体为
				- 用来限制容器的最大 CPU 使用率。
				- `cpu.cfs_quota_us` 参数与 `cpu.cfs_period_us` 通常结合使用，后者用来设置时间周期，前者设置在时间周期内可以使用的 CPU 时间。两者的比例即为 CPU 的最大使用率。
					- 默认系统将 `docker run` 的 `–cpu-period` 参数设置为 100000，即100毫秒。该参数就对应着 cgroup 的 `cpu.cfs_period_us` 参数。
					- limits.cpu 的单位使用 m，意义为 millicore，即千分之一核。250m 就表示25%的最大 cpu 使用率，会将其转换为25毫秒的 `-cpu-quota` 参数。
				- 容器的 CPU 使用率不允许长时间超过 limits ，当超过 limits 时不会被终止。 
			- limits.memory 会被转换成 `docker run`的 `–memory` 参数。用来限制容器使用的最大内存。
				- 容器的申请内存超过 limits 时会被终止，并根据重启策略进行重启。		- *spec.container[].resources.limits.cpu*
		- *spec.container[].resources.limits.memory*
		- limits未设置时，默认值与集群配置相关。
- LimitRange(pod/node)

	LimitRange 针对 pod 层，使用该功能需要在 kube-apiserver 上启动参数需要设置 `--admission_control=LimitRanger`。
	
	参数用途如
	
	- 集群中每个节点都有 2G 内存，所以下发的 pod 大小不能大于 2G 内存，超过的一律会被拒绝。
	- 组织中有2个部门分享一个集群，运行开发和生产任务。生产任务需要 8G 内存，而开发任务只需要 512M 内存。集群管理员会给每个工作负载创建一个单独的命名空间，并对命名空间进行限制。用户可以创建低于限制的 pod 。如果空间设置太小，会导致无法使用，而空间设置太大则会浪费。集群管理员可能需要在每个节点设定大于 pod 消耗20%的 cpu 和内存资源，提供更统一的调度和限制浪费。	
	所以 LimitRange 是在命名空间级别设置的 pod 容量限制。在对应命名空间中的 Pod，都会受到 LimitRange 的资源限制。创建步骤
	- 创建命名空间

			$ kubectl create -f docs/user-guide/limitrange/namespace.yaml
			namespaces/limit-example
			$ kubectl get namespaces
			NAME            LABELS             STATUS
			default         <none>             Active
			limit-example   <none>             Active 
	- 设置命名空间配额

			$ kubectl create -f docs/user-guide/limitrange/limits.yaml --namespace=limit-example
			limitranges/mylimits
			$ kubectl describe limits mylimits --namespace=limit-example
			Name:   mylimits
			Type      Resource  Min  Max Default
			----      --------  ---  --- ---
			Pod       memory    6Mi  1Gi -   <- pod 总消耗内存必须在6m-1g之间
			Pod       cpu       250m   2 -   <- pod 总消耗cpu必须在250m-2核之间
			Container memory    6Mi  1Gi 100Mi <- 容器可消耗内存资源同pod，如果没设置，初始资源将获得100m
			Container cpu       250m   2 250m <- 容器可消耗cpu资源同pod，如果没设置，初始资源将获得250m
	- 创建 pod 限制	
	
		命名空间限制其实就是虚拟的限制，并不引入底层，只在下发 pod 的时候起资源超出申请过滤作用，更改限制则不会影响先前创建的 pod(因为已经被下发)。如果下发的 pod 资源超过限制，用户在创建时会收到错误。	
		- 创建 pod 和查看状态
			
				$ kubectl run nginx --image=nginx --replicas=1 --namespace=limit-example
				CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS
				nginx        nginx          nginx      run=nginx   1
				$ kubectl get pods --namespace=limit-example
				POD           IP           CONTAINER(S)   IMAGE(S)   HOST      LABELS      STATUS    CREATED          MESSAGE
				nginx-ykj4j   10.246.1.3                             10.245.1.3/   			run=nginx   Running   About a minute
                       						nginx       nginx                           			Running   54 seconds
		- 查看 pod 详情
		
			nginx容器已经占用了命名空间默认的 cpu 和内存限制	
			
				$ kubectl get pods nginx-ykj4j --namespace=limit-example -o yaml | grep resources -C 5
				containers:
  					- capabilities: {}
    					image: nginx
    					imagePullPolicy: IfNotPresent
	    				name: nginx
    					resources:
     				 	limits:
      				 	 cpu: 250m
       			 	 memory: 100Mi
    					terminationMessagePath: /dev/termination-log
    					volumeMounts
    		创建一个超过命名空间限制资源的 pod
    		
	    		$ kubectl create -f docs/user-guide/limitrange/invalid-pod.yaml --namespace=limit-example
				Error from server: Pod "invalid-pod" is forbidden: Maximum CPU usage per pod is 2, but requested 3
	- 清理

		如果需要删除这个命名空间限制，只需要删除这个命名空间即可。
			
				$ kubectl delete namespace limit-example
				namespaces/limit-example
				$ kubectl get namespaces
				NAME      LABELS    STATUS
				default   <none>    Active					
	Pod 的每一类资源限制是 Pod 中对应的资源类型限制的总和，在调度的过程中调度器使用这个总和值和主机上可用容量做比较，决定主机是否满足资源需求。这种调度属于静态的资源调度，即使主机上的真实负载很低，只要容量限制不满足需求，也不能在主机上部署Pod。这个类似 mesos 资源管理的能力是静态的。
- ResourceQuota

	同 	LimitRange 需要在 kube-apiserver 上启动参数需要设置 `--admission_control=ResourceQuota`
	
	- 参数用途	
		- 一个集群有多个团队使用，如果没有限制可能导致这个团队占用了大部分的系统资源，而其他团队无法使用。
		- 使用 ResourceQuota 来限制一个命名空间的整体消耗，定义如下
			-  每个团队都有自己不同的命名空间，可以通过 acl 进行。
			-  管理员给每个空间创建一个或者多个资源陪额对象
			-  用户可以在命名空间中创建自己的资源,如:pod,service等，并被对使用配额使用情况进行跟踪，以确保不会超过配额定义的硬件资源限制。
			-  如果创建或者更新和现有配额有冲突，请求则会返回 http 状态码 403，并显示错误信息。
			-  如果在命名空间中的计算资源(cpu\mem)启用了配额，客户必须为这些对象指定 requests 和 limits。否则系统可能会拒绝创建 pod 。注意:限制使用 LimitRange 强制限制设定默认值资源的 pod。
	- 例
		- 集群16c/32g内存，让a组使用 10c/20g，让b组使用 4c/10g，保留 2c/2g 备份 
		- 将命名空间 "test" 设定为 1c/1g， 将命名空间 "prod" 设定为任何数量
		- 如果集群总容量小于命名空间容量，资源分配将采用先到先得的抢占模式
		- 抢占模式并不会影响已经下发的服务
	- 计算资源配额
		- 可以限定命名空间中可请求的计算资源总额，资源类型
			- cpu & requests.cpu
			
				所有 pod 的非终结 (non-terminal) 状态,CPU requests(请求)总额不能超过这个值。
			- limits.cpu

				所有 pod 的非终结 (non-terminal) 状态,CPU limits(限制)总额不能超过这个值。 	 		- memort & requests.memory
				
				所有 pod 的非终结 (non-terminal) 状态,内存 requests(请求)总额不能超过这个值。
			- limits.memory
				
				所有 pod 的非终结 (non-terminal) 状态,内存 limits(限制)总额不能超过这个值。
	- 存储资源配额
		- 可以限定命名空间中存储使用的资源总额
			- requests.storage

				命名空间中所有持久卷声明中，存储请求的总和不能超过这个值。
			- persistentvolumeclaims

				命名空间中可存在持久卷声明总数
			- <storage-class-name>.storageclass.storage.k8s.io/requests.storage
			- <storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims
		 

	
