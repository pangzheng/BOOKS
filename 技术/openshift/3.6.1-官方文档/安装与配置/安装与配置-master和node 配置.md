# 主节点和节点配置
## 概述
`openshift start` 命令用于启动 openshift 服务器。该命令及其子命令(主要启动master 服务器和节点服务器)都采用一组有限的参数，这些参数足以在开发或实验环境中启动服务器。

但是，这些论据不足以描述和控制生产环境中所需的全套配置和安全选项。要提供这些选项，有必要使用专用主节点和节点配置文件。

master 配置文件和节点配置文件完全指定，没有默认值。因此，任何空值表示要该参数启动一个空值。这可以很容易的推断出配置是什么，但是这也是使得难以记住所有要指定的选项。为了方便起见，可以使用 `--write-config` 选项创建配置文件，然后使用 `--config` 选项。
## master 配置文件
master 配置文件的 `master-config.yaml` 无论何时修改都必须重启主服务器才能生效。
### 准入控制配置(Admission Control Configuration)
参数名|描述
------|-----
AdmissionConfig|包含准入控制插件配置
APIServerArguments|将直接传递给与 api 服务器的命令行参数相匹配的 kube api 服务器 kv 对。这些都没有迁移，但如果引用一个不存在的值，服务器将无法启动。这些值可能会覆盖 ` KubernetesMasterConfig` 中其他的设置，可能导致无效的配置。
ControllerArguments|将直接传递给与控制器管理器的命令行参数相匹配的 kube 控制器管理器的 kv 对。这些不会被迁移，但如果引用一个不存在的值，服务器将无法启动。这些值可能会覆盖 ` KubernetesMasterConfig` 中其他的设置，可能导致无效的配置。
DefaultAdmissionConfig|用于启用或禁用各种许可插件。当这种类型作为 `pluginConfig` 下的 `configuration` 对象出现时，如果允许插件支持，将导致一个 `off by default` 允许插入插件。
PluginConfig|允许为每个准入控制插件指定一个配置文件
PluginOrderOverride|准入控制插件名称将被安装在 master 上。订单是种哟啊的。如果是空的，使用插件的默认列表。
SchedulerArguments|直接传递给与调度程序的命令行参数相匹配的 kube 调度程序的kv 对。这些都没迁移，这些值可能会覆盖 ` KubernetesMasterConfig` 中其他的设置，可能导致无效的配置。
### 资产配置(Asset Configuration)
参数名|描述
------|-----
AssetConfig|为服务资产保留必要的配置选项
DisabledFeatures|不应该启动的功能列表。可能希望将其设置为 `null`，任何人都不希望手动禁用功能。
Extensions|在子上下文中从资产服务器文件系统提供的文件
ExtensionDevelopment|当设置为 `true` 时，告诉资产服务器为每个请求重新加载扩展脚本和样式表，而不是在启动时。它可以让开发扩展，而无需每次更改都重启服务器。
ExtensionProperties|将全局变量 `OPENSHIFT_EXTENSION_PROPERTIES` 注入到控制台中的 kv(字符串) 对中
ExtensionScripts|在 web 控制台加载时，资产服务器文件上的文件路径作为脚本加载。
ExtensionStylesheets|在 web 控制台加载时，资产服务器文件上的文件路径作为样式表加载。
LoggingPublicURL|登录公共端点(可选)
LogoutURL| 可选，在 web 控制台注销后，重定向 web 游览器的绝对 url。如果没有指定，显示内置的注销页面。
MasterPublicURL|web 控制台如何访问 openshift 服务器
MetricsPublicURL|监控公共端点(可选)
PublicURL|资产服务器的URL
### 身份验证和授权配置
参数名|描述
------|-----
authConfig|保持身份验证和授权配置选项
AuthenticationCacheSize|指示应该缓存多少结果的缓存。如果为0，则使用默认缓存大小。
AuthorizationCacheTTL|指示应该缓存授权结果的时间。需要有效的持续时间字符串，例如 5m。如果为空，则获得默认超时。如果为零，如 0m 则缓存被禁用。
### 控制器配置
参数名|描述
------|-----
Controllers|应该启用的控制器列表。如果设置为 none，则不会自动启动控制器。默认值是 `*` 将启动所有控制器。当使用 `*` 时，可以通过在他们的名字前面加上 `-` 来排除控制器。目前没有其他值被识别。
ControllerLeaseTTL|启用控制器选举，指示 master 尝试在控制器启动前获取租约和在由该值定义的许多秒内更新它。将此值设置为非负值会强制 `pauseControllers=true`。该值默认为 `0`或省略,并且可以用 `-1` 禁用控制器选举。
PauseControllers|指示 mster 控制器不自动启动控制器,而是在启动之前等待收到的通知被承认
### etcd 配置
参数名|描述
------|-----
Address|etcd 的客户端使用的 host:端口。
etcdClientInfo|包含如何链接到 etcd 的信息
etcdConfig|保留用于连接到一个 etcd 数据库必要的配置选项
etcdStorageConfig|包含有关 api 资源如何存储在 etcd 的信息。这些仅在 etcd 是集群的存储时才有效。
KubernetesStoragePrefix| Kubernetes 资源将存于 etcd 的路径。改变这个值降意味着 etcd 中的现有对象将不在被定位。默认值为:kubernetes.io
KubernetesStorageVersion|Kubernetes 资源在 etcd 中的 api 版本应该被序列化。在从 etcd 读取的集群中有允许他们读取新版本的代码。
OpenShiftStoragePrefix|openshift 资源将存储在 etcd 中的路径。该值如果发生改变，将意味着 etcd 中的现在的对象将不在被定位。默认值为 openshift.io
OpenShiftStorageVersion| etcd 中 os 资源应该被串行化的 api 版本。在从 etcd 读取的集群中有允许他们读取新版本的代码。
PeerAddress|advertised 的主机地址，用于与 etcd 进行对等链接的端口
PeerServingInfo|介绍如何开始为 etcd 对等服务。
ServingInfo|介绍如何开始为 etcd 主服务
StorageDir| etcd 存储目录
### 授予配置
参数名|描述
------|-----
GrantConfig |描述如何处理授予
GrantHandlerAuto|自动批准客户端授予授权请求
GrantHandlerDeny|自动拒绝客户端授予授权请求
GrantHandlerPrompt|提示用户批准新的客户端授予授权请求
Method|确定 oauth 客户杜 u 安请求授权时使用的默认策略。只有特定的 oauth 客户端不提供自己的策略时才会使用此方法。有效的授权处理方法: - 自动:始终批准授权请求，对受信任的客户端有用。 - 提示:提示最终用户批准对第三方客户端有用的授权请求。 - 否认:总是拒绝授权请求，对黑名单客户端有效。
### 镜像配置
参数名|描述
------|-----
DisableScheduledImport|允许定期后台导入镜像被禁用。
Format|系统组件的名称格式
ImageConfig|包含描述如何为系统组件创建图像名称的选项
ImagePolicyConfig|控制导入镜像的限制和行为
Latest|确定是否从注册表中提取最新的标签
MaxImagesBulkImportedPerRepository|确定是否从注册表中提取最新的标签。控制用户进行 docker 存储库批量导入时导入镜像的数量，默认为 5.设置为 -1 为无限制。
MaxScheduledImageImportsPerMinute|每分钟将在后台导入的预定镜像流的最大数量。默认值为 60
ScheduledImageImportMinimumIntervalSeconds|在计划进行后台导入的镜像流与上游存储库之间检查时间间隔的最小秒数。默认是 15分钟。
AllowedRegistriesForImport|限制普通用户可以导入镜像的docker 镜像仓库。将此列表设置为信任的包含有效 docker 镜像仓库，并希望应用程序能够从中导入。有权通过 api 创建镜像或者 `ImageStreamMappings` 的用户不受此策略影响。通常只有管理员或者系统集成将具有这些权限。
InternalRegistryHostname|设置默认内部镜像仓库的主机名。该值必须是`hostname[:port]` 格式。为了向后兼容，用户仍然可以使用 `OPENSHIFT_DEFAULT_REGISTRY` 环境变量参数，但此设置会覆盖这个参数。设置此项时，内部镜像仓库也必须设置其主机名。
ExternalRegistryHostname| `ExternalRegistryHostname` 设置默认外部镜像仓库的主机名。只有当镜像仓库向外暴露时才应该设置这个值。该值用于 `ImageStreams` 中的 `publicDockerImageRepository` 字段。该值格式必须为 `hostname[:port]`
### Kubernetes master 配置
参数名|描述
------|-----
APILevels|应该在启动时启用的 api 级别列表。
DisabledAPIGroupVersions|api 组的映射应该禁用的版本
KubeletClientInfo|包含有关如何连接 kubelets 的信息
KubernetesMasterConfig|保留 Kubernetes master 设备必要的配置选项
MasterCount|应该运行的预期 master 数量。默认值为 1，可以设置为正整数，如果设置为 -1 则表示这个集群的一部分。
MasterIP| Kubernetes 资源的公共 ip 地址。如果为空，则将使用 `net.InterfaceAddrs` 第一结果。
MasterKubeConfig|描述如何将此节点连接到主节点的 `.kubeconfig` 文件的文件名
ServicesNodePortRange|用于在主机分配服务公共端口范围。默认是 `30000-32767`
ServicesSubnet|用于分配服务ip 的子网
StaticNodeNames|静态已知的节点列表
### 网络配置
仔细选择一下参数中的 CIDR，因为 IPV4 地址空间由节点的所有用户共享。 openshift 保留来自 IPV4 地址空间的 CIDR 供自己使用，并从这个空间为外部用户和集群之间共享的地址保留 CIDR。

参数名|描述
------|-----
ClusterNetworkCIDR|CIDR字符串用于指定全局覆盖网络的三层空间。这是为集群网络的内部使用而保留的。
ExternalIPNetworkCIDRs|控制服务外部 IP 字段可接受的值。如果为空，则不能设置外部 IP。可能包括检查访问的 CIDR 列表。如果前缀为 !, 该CIDR 中的 IP将被拒绝。拒绝首先应用，然后根据允许的 CIDR 检查 IP。出于安全原因，必须确保此范围不会与节点，pod或服务的 CIDR 重叠。
IngressIPNetworkCIDR|控制在裸机上为 `LoadBalancer` 类型的服务分配入口 ip 的范围。可能包含一个将从中分配的CIDR。默认为 `172.46.0.0/16`。出于安全原因，应该确保此范围与外部 ip、节点、pod、服务保留的 CIDR 不重叠。
HostSubnetLength|要分配给每个主机子网的位数。例如 8表示主机上的一个 /24 网络
NetworkConfig |为节点提供网络选项
NetworkPluginName|要使用的网络插件的名称
ServiceNetwork|用于指定服务网络的 CIDR 字符串
### OAuth 认证配置
参数名|描述
------|-----
AlwaysShowProviderSelection|即使只有一个提供者时，也会强制提供者选择页面进行渲染。
AssetPublicURL|用于外部访问构建有效客户端重定向 URL
Error|包含用于在认证或授权流程中呈现错误页面的go模版的文件路径或授权流程如果未指定，使用默认的错误页面。
IdentityProviders|为用户标识自己的有序列表。
Login|包含用于呈现登录页面的go 模版文件路径。如果未指定将使用默认登录页面。
MasterCA|CA 验证 TLS 连接返回到 `MasterURL`
MasterPublicURL|用于外部访问构建有效客户端重定向 URL
MasterURL|用于进行 `server-to-server` 的调用以交换访问令牌的授权码
OAuthConfig|为 oauth 身份验证保留必要的配置选项
OAuthTemplates|允许定制页面，如登录页面
ProviderSelection|包含用于呈现提供者选择页面的 go 模版文件路径，如果未指定，使用默认提供者选择页面。
SessionConfig|保存有关配置会话的信息
Templates|允许像登录页面一样自定义页面
TokenConfig|包含授权和访问令牌选项
### 项目配置
参数名|描述
------|-----
DefaultNodeSelector|保存默认项目节点标签选择器
ProjectConfig|保存有关项目创建和默认值的信息
ProjectRequestMessage|如果用户无法通过项目请求 aip 端点请求项目，则该字符串会呈现给用户。
ProjectRequestTemplate|用于响应 `projectrequest` 创建项目的模版。格式是 `namespace/template`,它是可选的。如果未指定，使用默认模版。
### 调度配置
参数名|描述
------|-----
SchedulerConfigFile|指向描述如何设置调度程序的文件。如果为空，将获得默认的调度规则。
### 安全分配器配置
参数名|描述
------|-----
MCSAllocatorRange|定义将分配给命名空间的 MCS 类别的范围。格式为 ` <prefix>/<numberOfLabels>[,<maxCategory>]` 默认值是 `s0/2`，将从 c0 到 c1023 分配，这意味着总共有 `535k` 标签可用。(1024个选择 `2 ~ 535k`)。如果此值更改，新命名空间可能会收到已分配给其他命名空间的标签。前缀可以是任何有效的 `SELinux` 术语集合，包含用户、角色、类型等。尽管他们作为默认值将允许服务器自动设置他们。
SecurityAllocator|控制将 uid 和 MCS 标签自动分配给命名空间。如果为0，分配被禁用。
UIDAllocatorRange|定义将自动分配给命名空间的 Unix 用户 id(uid)的总集合，以及每个命名空间获取的块大小。例如，`1000-1999/10` 将为每个命名空间分批10个 uid，并且在空间不足之前将狗分配多达100个块。默认值是在 10k 块中分配1百万到2百万块。这是在用户命名空间启动后，容器镜像将使用的范围预期大小。
### 服务账户配置
参数名|描述
------|-----
LimitSecretReferences|控制是否允许服务账户在未明确引用它的情况下引用命名空间中的任何加密凭证。
ManagedNames|将在每个命名空间中自动创建的服务账户名称列表，如果没有指定名称，`ServiceAccountsController` 将不会启动
MasterCA|用于验证 TLS 连接返回给 master 服务器的 CA。服务账户控制器会自动将此文件的内容注入到容器中，以便它们可以验证与 master 之间的连接。
PrivateKeyFile|包含 PME 编码的专用 RSA 密钥文件，用于签署服务账户令牌。如果没有指定密钥，服务账户 `TokensController` 将不会启动。
PublicKeyFiles|文件列表，每个文件都包含一个 PEM 编码的共有 RSA 密钥。如果任何文件包含密钥，密钥的公共部分被使用。公钥列表用于验证提供的服务账户令牌。每个密钥都将按顺序尝试，知道列表用尽或验证成功。如果没有指定，没有服务账户认证可用。
ServiceAccountConfig|为服务账户保留必要的配置选项。
### 服务信息配置
参数名|描述
------|-----
AllowRecursiveQueries|允许 master 服务器上的 dns 服务器递归查询。请注意，开放解析器将可用于 dns 攻击，不应该使公网可访问 master dns
BindAddress|用于绑定服务的 `ip:port`
BindNetwork|绑定的网络
CertFile|包含 PEM 编码证书的文件
CertInfo|用于提供安全流量的 TLS 证书信息
ClientCA|可以识别传入客户端证书的所有签名者的证书包
dnsConfig|为 DNS 保留必要的配置选项
DNSDomain|拥有的域名后缀
DNSIP|拥有的 IP 地址
KeyFile|包含由 CertFile 指定的证书的 PEM 编码私钥文件
MasterClientConnectionOverrides|提供对用于连接到 master 服务器的客户端连接的重写。
MaxRequestsInFlight|允许服务器发出的最大并发请求数。如果为零，没有限制。
NamedCertificates|用于保护对特定主机名的请求的证书列表
RequestTimeoutSecond|请求超时前的秒数，默认是 60分钟。如果-1，将没有限制。
ServingInfo|资产的 http 服务信息
### 卷配置
参数名|描述
------|-----
DynamicProvisioningEnabled|一个布尔值，用于启用或禁用动态配置。默认是 true
FSGroup|可以指定为每个唯一 `FSGroup ID` 在本地存储使用傻姑娘启用配额。目前这只适用于 `emptyDir` 卷，并且如果底层 `volumeDirectory` 位于 XFS 文件系统上。
LocalQuota|包含用于控制节点上的本地卷配额的选项
MasterVolumeConfig|包含用于在 master 节点中配置卷插件的选项
NodeVolumeConfig|包含用于在 node 节点上配置卷选项。
VolumeConfig|包含用于在 node 节点上配置卷选项。
VolumeDirectory|卷存储在目录下
### 审计配置
审计提供与安全相关的按时间顺序排列的记录集，记录单个用户，管理员，系统或其他组件影响系统的活动顺序。审计在 api 服务器工作，记录所有到达服务器的请求。每个审计日志都包含两个条目:

- 请求行包含
	- 允许匹配响应行的唯一 ID
	- 请求的源IP
	- 正在调用 HTTP 方法
	- 原始用户调用该操作
	- 该操作的模拟用户，self 就是自己
	- 操作的模拟组，查找含义用户组
	- 请求的命名空间或<none>
	- 请求的 URI
- 响应行包含
	- 允许匹配响应行的唯一 ID
	- 响应代码

用户 amind 询问 pod 列表的示例输出: 

	AUDIT: id="5c3b8227-4af9-4322-8a71-542231c3887b" ip="127.0.0.1" method="GET" user="admin" as="<self>" asgroups="<lookup>" namespace="default" uri="/api/v1/namespaces/default/pods"
	AUDIT: id="5c3b8227-4af9-4322-8a71-542231c3887b" response="200"

`openshift_master_audit_config` 变量启用 API 服务审计。包含以下选项:

参数名|描述
------|-----
enabled|一个布尔值，是否打开审计日志，默认是 false
auditFilePath|请求应该记录到文件路径。如果为设置，日志将打印到 master log。
maximumFileRetentionDays|根据在其文件名中编码的时间戳，指定保留旧审计日志的文件最大天数。
maximumRetainedFiles|指定要保留的旧审计日志文件最大数量
maximumFileSizeMegabytes|指定文件轮转前最大大小以 MB 为单位，默认为 100MB

例子

	auditConfig:
		auditFilePath: "/var/log/audit-ocp.log"
		enabled: true
		maximumFileRetentionDays: 10
		maximumFileSizeMegabytes: 10
		maximumRetainedFiles: 10
高级安装可以直接设置

	openshift_master_audit_config={"enabled": true}
添加更多参数

	openshift_master_audit_config={"enabled": true, "auditFilePath": "/var/log/openpaas-oscp-audit/openpaas-oscp-audit.log", "maximumFileRetentionDays": 14, "maximumFileSizeMegabytes": 500, "maximumRetainedFiles": 5}

### 节点配置文件
以下`node-config.yaml` 文件是一个示例节点配置文件，它是在写入时使用默认值生成的。可以[创建一个新的节点配置文件](https://docs.openshift.org/3.6/install_config/master_node_configuration.html#creating-new-configuration-files)来查看已经安装版本 openshift 的有效选项。

```
allowDisabledDocker: false
apiVersion: v1
authConfig:
  authenticationCacheSize: 1000
  authenticationCacheTTL: 5m
  authorizationCacheSize: 1000
  authorizationCacheTTL: 5m
dnsDomain: cluster.local
dnsIP: 10.0.2.15 # 通过此处添加地址，将 ip 配置为预先添加到 pod 的 /etc/resolv.conf
dockerConfig:
  execHandlerName: native
imageConfig:
  format: openshift/origin-${component}:${version}
  latest: false
iptablesSyncPeriod: 5s
kind: NodeConfig
masterKubeConfig: node.kubeconfig
networkConfig:
  mtu: 1450
  networkPluginName: ""
nodeIP: ""
nodeName: node1.example.com
podManifestConfig: #允许将 pod 直接放置在特定的一组节点上，或在所有节点上，而无需通过调度程序。然后，可以使用 pod 执行相同的管理任务，并在每个节点上支持相同的服务。
  path: "/path/to/pod-manifest-file" # 指定 pod 清单文件或者目录的路径。如果它是一个目录，那么应该包含一个或多个清单文件。这用于 kubelet 用于在节点上创建 pod。
  fileCheckIntervalSeconds: 30 #这个是动态加载这个文件检查这个文件变化的间隔时间，单位是秒。间隔必须为正值。
proxyArguments:
  proxy-mode:
  - iptables #要使用的服务代理实现。
volumeConfig:
  localQuota:
   perFSGroup: null #初始支持本地 empty dir 卷配额，将此值设置为表示每个 FSGroup 所需的每个节点的配额的资源数量。即 1gi，512mi等。当前要求 volumeDirectory 位于安装有 gquota 选项的 XFS 文件系统上，匹配安全上下文限制的 fsGroup 类型设置为 `MustRunAs`
servingInfo:
  bindAddress: 0.0.0.0:10250
  bindNetwork: tcp4
  certFile: server.crt
  clientCA: node-client-ca.crt
  keyFile: server.key
  namedCertificates: null
volumeDirectory: /root/openshift.local.volumes
```	

### pod 和节点配置
参数名|描述
------|-----
NodeConfig|完全指定的配置启动一个 openshift 节点
NodeIP|节点可能有多个 ip，因此它指定了用于 pod 流量路由的 ip。如果未指定，则执行 `nodeName` 上的 `parse/lookup` 并使用第一个 non-loopback 地址。
NodeName|用于标识集群中特定节点的值。如果可能，这应该是完全限定为主机名。如果正在向 master 节点描述一组静态节点，此值必须与列表中的某个值匹配。
PodEvictionTimeout|控制在失败节点上删除 pod 的宽限期。它需要有效的持续时间字符串。如果为空，将获得默认 pod 驱逐的超时时间
ProxyClientInfo|指定代理到 pod 时使用的客户端证书/密钥
### docker 配置
参数名|描述
------|-----
AllowDisabledDocker|如果为 true，那么 kubelet 将忽略来自 docker 的错误。这意味着节点可以在没有启动  docker 的机器上启动。
DockerConfig|保留 docker 相关的配置选项
ExecHandlerName|用于在 docker 容器中执行命令的处理程序
### 并行在 docker 1.9 中拉取镜像
如果使用的是 docker 1.9,则可能需要考虑启用并行镜像拉取，因为默认情况下一次只拉取一个镜像。

在 docker 1.9之前一个潜在的问题是并行拉取镜像容器造成数据损坏。但是在 1.9 后的版本，数据损坏的问题得到解决，并且切换到并行拉取是安全的。

```
kubeletArguments:
  serialize-image-pulls:
  - "false"  #改成 true 来禁用并行拉取， false 为串行拉取。
```
### 密码和其他敏感数据
对于某些[认证配置](https://docs.openshift.org/3.6/install_config/configuring_authentication.html#install-config-configuring-authentication)，需要 `LDAP bindPassword` 或者 `OAuth clientSecret`值。这些值可以作为环境变量、外部文件或加密文件提供，而不是直接在主配置文件中指定这些值。

- 变量

```
  ...
  bindPassword:
    env: BIND_PASSWORD_ENV_VAR_NAME
```

- 文件

```
...
  bindPassword:
    file: bindPassword.txt
```

- 外部文件

```
  ...
  bindPassword:
    file: bindPassword.encrypted
    keyFile: bindPassword.key
```

创建加密文件或密钥文件的方法:

```
$ oc adm ca encrypt --genkey=bindPassword.key --out=bindPassword.encrypted
> Data to encrypt: B1ndPass0rd!
```

仅从 ansible 主机清单文件中列出的第一个主文件运行 oc adm 命令，默认情况下在 `/etc/ansible/hosts`.

加密数据只与解密密钥一样安全。应该小心限制文件系统权限和访问密钥文件。

### 创建新的配置文件
从头开始定义 openshift 配置时，首先需要创建新的配置文件。

对于 master 主机配置文件，使用带有 `--write-config` 选项的 openshift start 命令来卸乳配置文件。对于节点主机，使用 `oc adm create-node-config` 命令来写入配置文件。

以下命令将相关启动配置文件、证书文件和任何其他必须需要的文件指定的 `--write-config ` 或 `--node-dir` 目录

生成的证书文件有效期为两年，而证书颁发机构(CA)证书有效期为5年。这可以通过 `--expire-days` 和 `--signer-expire-days` 选项来修改，但出于安全原因，建议不要使用他们大于这些值。

要为指定目录中的 `all-in-one` 服务器(同一个主机上有master和节点)创建配置文件，请执行:

	$ openshift start --write-config=/openshift.local.config
要在指定目录中创建[主配置文件](https://docs.openshift.org/3.6/install_config/master_node_configuration.html#master-configuration-files)和其他必须文件，请执行:

	$ openshift start master --write-config=/openshift.local.config/master
要在指定的目录中创建[节点配置文件](https://docs.openshift.org/3.6/install_config/master_node_configuration.html#node-configuration-files)和其他必须文件，请执行

```
$ oc adm create-node-config \
    --node-dir=/openshift.local.config/node-<node_hostname> \
    --node=<node_hostname> \
    --hostnames=<node_hostname>,<ip_address> \
    --certificate-authority="/path/to/ca.crt" \
    --signer-cert="/path/to/ca.crt" \
    --signer-key="/path/to/ca.key"
    --signer-serial="/path/to/ca.serial.txt"
    --node-client-certificate-authority="/path/to/ca.crt"
```
当创建节点配置文件时， `--hostnames` 选项接受希望服务器证书有效的以逗号分隔的每个主机名或 ip地址列表。
### 使用配置文件启动服务器
一旦将主配置文件和节点配置文件修改为想要的，可以通过参数来指定它们在启动服务器时使用。记住，如果指定的了一个配置文件，则其他命令行选项将被忽略。

使用主配置和节点配置文件启动一台一体机:

	$ openshift start --master-config=/openshift.local.config/master/master-config.yaml --node-config=/openshift.local.config/node-<node_hostname>/node-config.yaml
使用主配置文件启动 master 主机	

	$ openshift start master --config=/openshift.local.config/master/master-config.yaml
使用节点配置文件启动节点主机

	$ openshift start node --config=/openshift.local.config/node-<node_hostname>/node-config.yaml
### 配置日志记录级别
openshift 使用 `systemd-journald.service` 来收集日志消息以进行调试，使用五种日志消息严重性。日志级别基于 kubernetes 日志约定，如下:

参数名|描述
------|-----
0|错误和警告信息
2|普通信息
4|debug 信息
6|api等级 debug 信息，请求和响应
8|api body 等级 debug 信息

可以在 ` /etc/sysconfig/atomic-openshift-node`、`/etc/sysconfig/atomic-openshift-master-api`和 ` /etc/sysconfig/atomic-openshift-master-controllers` 文件中通过设置 loglevel 选项来控制记录那些 INFO 信息。需要注意的是配置日志以及收集所有消息可能导致难以解释并可能专用过大的磁盘空间。收集所有消息只能用于调试情况。

无论日志配置具有 ` FATAL, ERROR, WARNING`和一些 `INFO` 严重级别的消息都会在日志中出现。可以使用以下命令查看 master 和节点系统日志。

	# journalctl -r -u <journal_name>
使用 `-r` 选项首先显示最新的条目

	# journalctl -r -u atomic-openshift-master-controllers
	# journalctl -r -u atomic-openshift-master-api
	# journalctl -r -u atomic-openshift-node.service
要更改日志记录级别:

- 编辑配置文件 `/etc/sysconfig/atomic-openshift-master` 和 `/etc/sysconfig/atomic-openshift-node`
- 在 `OPTIONS=--loglevel=`字段中，从上面的`日志级别选项中`输入一个值，如

		OPTIONS=--loglevel=4
- 重启服务加载配置

		# systemctl restart atomic-openshift-master-api atomic-openshift-master-controllers
		# systemctl restart atomic-openshift-node
重新启动后，所有新的日志消息都将符合新设定，旧信息将不会更改。默认日志级别可以使用高级安装进行设置。更多信息参考[集群变量](https://docs.openshift.org/3.6/install_config/install/advanced_install.html#cluster-variables-table)			