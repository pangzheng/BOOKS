# 集群管理员-管理 pod
主题描述了如何管理 pod ，包括限制其运行一次多长时间，以及可以使用多少带宽。
## 限制运行一次 pod 的持久时间
ocp 依赖于运行一次的 pod 来执行任务，如部署 pod 或执行构建。运行一次的 pod 具有`Never` 或 `OnFailure` 的  `RestartPolicy` 的 pod。

集群管理员可以使用 `RunOnceDuration` 准入控制插件来强行限制这些运行一次性 pod 可以处于活动状态的时间。一旦时间到期，集群将尝试主动终止这些 pod。有这样一个限制时间的主要原因是防止诸如构建任务运行过多时间占用资源。

## 配置 `RunOnceDuration` 插件
插件配置应该包含运行一次性 pod 的默认活动截止日期。这个截止日期是全局的，但可以在每个项目的删除上取代。

```
kubernetesMasterConfig:
  admissionConfig:
    pluginConfig:
      RunOnceDuration:
        configuration:
          apiVersion: v1
          kind: RunOnceDurationConfig
          activeDeadlineSecondsOverride: 3600 #在几秒中内指定运行一次性 pod 的全局默认值
```
## 指定每个项目的自定义持续时间
除了为运行一次性 pod 执行全局最大持续时间外，管理员还可以向特定的项目添加注解(`openshift.io/active-deadline-seconds-override`)以覆盖全局默认值。

```
apiVersion: v1
kind: Project
metadata:
  annotations:
    openshift.io/active-deadline-seconds-override: "1000" #运行一次性 pod 的默认活动时间修改成 1000秒。注意覆盖值必须以字符串形形式指定
```
## 部署一个出口路由器 pod
```
apiVersion: v1
kind: Pod
metadata:
  name: egress-1
  labels:
    name: egress-1
  annotations:
    pod.network.openshift.io/assign-macvlan: "true"
spec:
  containers:
  - name: egress-router
    image: openshift/origin-egress-router
    securityContext:
      privileged: true
    env:
    - name: EGRESS_SOURCE # 集群管理员保留的节点子网上的 IP，供此 pod 使用。
      value: 192.168.12.99
    - name: EGRESS_GATEWAY # 节点本身使用的默认网关 
      value: 192.168.12.1
    - name: EGRESS_DESTINATION # 到 pod 重定向到 203.0.113.25，源 ip 地址为 192.168.12.99
      value: 203.0.113.25
  nodeSelector:
    site: springfield-1 # 改 pod 只能部署到标签为 springfield-1 的节点上
```

`pod.network.openshift.io/assign-macvlan ` 注释在主网络接口傻姑娘创建一个 Macvlan 网络接口，然后在启动出口路由器容器之间将其启动到 pod 的 网络命名空间中。

	需要保留 `true` 的引号。省略将导致错误。
该 pod 包含一个容器，使用 `openshift/origin-egress-router` 镜像，并且该容器有 `privileged` 属性，一百年可以配置 macvlan 接口并设置 iptables 规则。

环境变量告诉出口路由器镜像要使用的地址；它将配置 macvlan 接口使用 `EGRESS_SOURCE` 作为其 ip 地址，并以 `EGRESS_GATEWAY` 作为其网关。

建立 NAT 规则，以便连接到集群 ip 地址上任何 tcp 或者 udp 端口的连接将重定向到 `EGRESS_DESTINATION` 上的相同端口。

如果只有集群中某些节点能够声明指定的源 ip 地址并使用指定的网关，则可以指定 `nodeName` 或者 `nodeSelector` 指出那些节点可以接受。
## 部署一个出口路由的服务
虽然不是必须的，但通常需要创建指向出口路由的服务

```
apiVersion: v1
kind: Service
metadata:
  name: egress-1
spec:
  ports:
  - name: http
    port: 80
  - name: https
    port: 443
  type: ClusterIP
  selector:
    name: egress-1
```	

pod 现在可以连接到这个服务。连接被重定向到外部的服务器的相应端口，使用保留的出口 IP 地址。
## 使用出口防火墙限制 pod 访问
作为 ocp 集群管理员，可以使用出口策略来限制部分或全部 pod 可以从集群内访问到外部地址，以便:

- 一个 pod 只能与内部主机通讯，不能启动与公网的通讯
- 一个 pod 与公网通讯但不能与内部主机通讯。
- 一个 pod 无法到达它应改没有理由联系的指定内部子网或主机

例如,可以使用不同的出口策略配置项目，允许 `<project A>` 访问指定的 IP 范围，但拒绝 `<project B>` 相同访问。

必须启动 [ovs-multitenant 插件](https://docs.openshift.org/3.6/install_config/configuring_sdn.html#install-config-configuring-sdn)才能通过出口策略限制 pod 访问。

项目管理员既不能创建 `EgressNetworkPolicy` 对象，也不能编辑项目中的对象。 `EgressNetworkPolicy` 可以创建的地方有几个其他限制：

- `default` 项目和通过 ` oadm pod-network make-projects-global` 创建的其他项目，不能有出口策略。
- 如果将两个项目合并在一起(通过 ` oadm pod-network join-projects`)，则不能在任何连接的项目中使用出口策略。
- 没有项目可以有多个出口策略对象

违反任何这些限制导致该项目的出口策略中断，并可能导致所有外部流量被丢弃。

## 配置 pod 访问限制
要配置 pod 访问限制，必须使用 oc 命令或者 rest api.可以使用 `oc [create|replace|delete]` 来操作 `EgressNetworkPolicy` 对象。 `api/swagger-spec/oapi-v1.json` 文件具有 API 对象实际工作的详细信息。步骤

- 导航到需要设置的项目
- 为 pod 限制策略创建一个文件

		oc create -f <policy>.json
- 使用策略详细信息配置 json 文件

```
{
    "kind": "EgressNetworkPolicy",
    "apiVersion": "v1",
    "metadata": {
        "name": "default"
    },
    "spec": {
        "egress": [
            {
                "type": "Allow",
                "to": {
                    "cidrSelector": "1.2.3.0/24" #允许该网段访问
                }
            },
            {
                "type": "Allow",
                "to": {
                    "dnsName": "www.foo.com" #允许该域名访问
                }
            },
            {
                "type": "Deny",
                "to": {
                    "cidrSelector": "0.0.0.0/0" #拒绝所有 ip,其他 pod 的流量不受影响，因为策略仅适用于外部流量。
                }
            }
        ]
    }
}
```	
`EgressNetworkPolicy` 中的规则按照顺序检查，匹配的第一个生效。如果上述例子中的三条规则点到过来，那么流量将不会被允许到 `1.2.3.0/24` 和 `www.foo.com`，因为 `0.0.0.0/0` 规则将首先生效，并将它匹配的所有流量拒绝。

根据本地非权威服务器的域的 TTL 轮询域名更新，或30分钟，如果 TTL 无法提取。必要时，还应用相同的本地非权威 dns 服务器解析域，否则出口网络策略控制器和 pod 所感知的域的IP地址将会不同，而出口网络策略可能无法按期执行。上面的例子中，假设 `www.foo.com` 解析为 `10.11.12.13`，并具有一分钟的 DNS TTL，但后来更改为 `20.21.22.23`。则 ocp 将话费1分钟来适应这些 dns 更新。	
## 限制可用于 pod 的带宽
可以将 `quality-of-service` 流量应用用于 pod，并有效限制其可用带宽。

- 出口流量(pod 出)

	由策略处理，它只是丢弃超过配置速率的数据包。
- 入口流量(到 pod) 

	通过定型排队来处理数据包，以有效处理数据。

防止在 pod 上的限制不会影响到其他 pod。要限制 pod 上的带宽，清执行以下操作：

- 编写一个对象定义的 json 文件，并使用 ` kubernetes.io/ingress-bandwidth` 和 `kubernetes.io/egress-bandwidth` 注释来指定数据流速度，例如，要将 pod 出口和入口带宽限制为 10m/s

```
{
    "kind": "Pod",
    "spec": {
        "containers": [
            {
                "image": "nginx",
                "name": "nginx"
            }
        ]
    },
    "apiVersion": "v1",
    "metadata": {
        "name": "iperf-slow",
        "annotations": {
            "kubernetes.io/ingress-bandwidth": "10M",
            "kubernetes.io/egress-bandwidth": "10M"
        }
    }
}
```

- 使用对象定义创建 pod

		oc create -f <file_or_dir_path>

## 设置 pod 中断预算
pod 中断预算是 k8s api 的一部分，可以像其他对象类型一样使用 oc 命令来管理。它们允许在操作期间对 pod 的安全限制进行规定，如耗尽节点将进行维护。

`PodDisruptionBudget` 是指定一个最小数量的 api 对象或一定时间内复制的百分比。在项目中设置这些项目可能会有助于节点维护(例如缩小集群或集群升级)而且只有在自愿迁移(不是节点失效)防霾呢猜得到遵守。

`PodDisruptionBudget ` 对象的配置由以下关键部分组成

- 一个标签选择器，它是一组标签查询。
- 可用性级别，指定必须同时可用的最小 pod 数量

一个例子：

```
apiVersion: policy/v1beta1 # PodDisruptionBudget 是 policy/v1beta1 组的一部分
kind: PodDisruptionBudget
metadata:
  name: my-pdb
spec:
  selector:  # 一组资源的标签查询。 matchLabels 和 matchExpressions 的结果是逻辑关联的
    matchLabels:
      foo: bar
  minAvailable: 2 #必须同时指定可用最小的 pod 数量。这可以是一个整数或指定百分比的字符串(如 20%)
```

创建描述

	oc create -f </path/to/file> -n <project_name>
可以检查所有项目的中断预算

	oc get poddisruptionbudget --all-namespaces
当系统中至少运行 `minAvailable` 的 pod 时， `PodDisruptionBudget` 被认为是健康的。超过这个限制的每个 pod 都会被[驱逐](https://docs.openshift.org/3.6/admin_guide/out_of_resource_handling.html#out-of-resource-eviction-policy)。	