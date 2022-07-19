# ipfscluster
## 文档更新到1.0.1 版（参见变更日志）。
欢迎使用 IPFS 集群文档。文档的不同部分将解释如何设置、启动和操作集群。如果您不熟悉 [IPFS](https://ipfs.io/) 和  peer -2- peer  网络（特别是 [libp2p](https://libp2p.io/)）的概念，那么操作生产 IPFS 集群可能是一项艰巨的任务。我们的目标是提供全面的文档和指南，但我们始终愿意改进：文档问题可以提交到 [ipfs-cluster-website 存储库](https://github.com/ipfs-cluster/ipfs-cluster-website)。

您是 IPFS 集群用户吗？通过参与 [IPFS 集群用户注册表](https://docs.google.com/forms/d/e/1FAIpQLSdWF5aXNXrAK_sCyu1eVv2obTaKVO3Ac5dfgl2r5_IWcizGRg/viewform) 让我们了解您的设置。

## 测试集群快速入门
这将帮助您使用 [Docker](https://docs.docker.com/install/) 和 [Docker Compose](https://docs.docker.com/compose/install/) 设置 IPFS 集群的测试本地实例。目的是让您快速预览什么是运行 IPFS 集群以及如何与之交互。要成功遵循这些说明，您需要熟悉 Docker 以及从命令行运行命令（包括签出 git-repository）。

我们将使用 docker compose 模板启动一个 3  peer  集群（以及 IPFS 守护进程）。一旦集群启动并运行，我们将使用 `ipfs-cluster-ctl`.

0. 安装 Docker 和 Docker Compose
	- Docker
	- Docker Compose
1. 下载 `ipfs-cluster-ctl`

	从 [dist.ipfs.io](https://dist.ipfs.io/#ipfs-cluster-ctl)  下载适用于您平台的 `ipfs-cluster-ctl` 最新版本并将其解压缩到您选择的文件夹中。

	`ipfs-cluster-ctl` 是 IPFS 集群守护进程的命令行客户端，我们将使用它来检查集群、添加和固定内容。
2. 下载 `docker-compose.yml` 文件

	下载 [docker-compose.yml](https://raw.githubusercontent.com/ipfs-cluster/ipfs-cluster/master/docker-compose.yml) 并将其放在与`ipfs-cluster-ctl`
3. 启动集群

	从下载这两个文件的文件夹中，运行：

		$ docker-compose up
	等到所有容器都在运行。您可能会看到一些错误，因为集群 peer 在 IPF 准备好之前启动得太快，但它们是无害的。

	- SELinux 用户
	
		如果服务因“权限被拒绝”错误而无法启动，您可能需要执行以下操作：

		- 查找 AVC 拒绝并授予 ipfs 进程所需的 SELinux 权限。
		- `sudo chmod -R 1000:100 compose. root` 创建撰写目录时，服务似乎使用用户。
		- 添加 `:z` 到 `docker-compose.yml` 文件中卷的末尾。有关更多信息，请参阅在 [Docker 中配置 SELinux 卷](https://docs.docker.com/storage/bind-mounts/#configure-the-selinux-label)。
4. 玩集群

	您现在应该有一个 3  peer  IPFS 集群正在运行！在不同的终端（相同的文件夹）上使用 `ipfs-cluster-ctl` 来与之交互：

		./ipfs-cluster-ctl  peer s ls                      # 查看集群中的 peer 信息
		./ipfs-cluster-ctl add somefile                  # 向集群添加一个文件
		./ipfs-cluster-ctl pin add /ipns/ipfscluster.io  # 固定集群网站
		./ipfs-cluster-ctl status <cid>                  # 使用上面显示的CID查看每个 peer 的状态
		./ipfs-cluster-ctl pin ls <cid>                  # 检查pin信息
	您可以在 [pinning guide](https://ipfscluster.io/documentation/guides/pinning) 中了解有关管理 pinset 的更多信息。

	完成后，您可以运行 `docker-compose kill`.
	
## 生产部署
本部分适用于希望在生产环境中设置 IPFS 和 IPFS 集群的管理员。管理员应熟悉 IPFS 和生产应用程序的部署（包括读取日志、能够验证端口是否打开、 peer 之间是否存在连接或进程是否正在运行）。

此外，在生产环境中运行 IPFS 集群需要：

- 基本了解 Cluster 应用架构以及它如何与 IPFS 交互
- 根据环境和集群的要求调整  `ipfs` 和 `ipfs-cluster-service` 配置，以及确保事物具有所需的连接性（防火墙端口等）。
- 启动 `ipfs-cluster-service` 守护程序并验证它们是否可以相互连接和同步。
- 可选择自动化集群 peer 的部署和生命周期。

这些主题在以下部分中进行了解释：

### 架构概述
在开始安装和运行 IPFS 集群之前，澄清一些基本概念很重要。

![](./pic/ipfscluster.png)
IPFS 集群软件由三个二进制文件组成：

- [ipfs-cluster-service](https://ipfscluster.io/documentation/reference/service)

	`ipfs daemon` 使用配置文件并通过在磁盘上存储一些信息来运行集群 peer （类似于）。
- [ipfs-cluster-ctl](https://ipfscluster.io/documentation/reference/ctl)

	用于与集群 peer 通信并执行操作，例如将 IPFS CID 固定到集群。
- [ipfs-cluster-follow](https://ipfscluster.io/documentation/reference/follow)

	运行一个追随者集群 peer 。它可以用作 `ipfs-cluster-service` 此特定用例的替代品并混合了一些 `service` 功能 `ctl`。

Cluster  peer  使用 HTTP API ( ) 与 IPFS 守护进程通信 `localhost:5001`。因此，IPFS 守护进程必须单独启动和运行。

通常，在运行 `ipfs-cluster-ctl` 的同一台机器或服务器上 `ipfs-cluster-service` 使用。

例如 `ipfs-cluster-ctl pin add <hash>` 将指示本地 Cluster  peer  向 Cluster 提交 pin。然后，集群中的不同 peer 将继续要求其本地 IPFS 守护程序固定该内容。集群中的 pin 数将取决于为每个 pin 设置的复制因子（默认设置在 `ipfs-cluster-servic`e 配置文件中）。

#### 集群群
IPFS 集群是一个完全分布式的应用程序。`ipfs-cluster-service` 运行一个集群 peer ，所有 peer 都是平等的。集群 peer 形成一个单独的、隔离的 `libp2p [私有] ` 网络，该网络使用`cluster_secret`（每个 peer 的配置中存在的 32 位十六进制编码密码）。

该网络不与主 IPFS 网络交互，也不与其他私有 IPFS 网络交互，仅用于集群 peer 可以通信和操作。该网络使用 IPFS 也使用的许多块（DHT、PubSub、Bitswap ......），但与 IPFS 不同，它不享受公共引导程序。

这意味着集群 peer 通常需要自己的引导程序（它可以是集群中的任何 peer ），尽管有时它们可​​以依赖 mDNS 发现。

这也意味着集群 peer 在 NAT 打孔、端口等方面与 IPFS 分开运行。
#### 共享状态：共识
集群中的所有 peer 都维护一个全局 pinset。无论并发的固定操作和分布式应用程序布局如何，让每个 peer 都保持相同的 pinset 视图需要由所谓的 `consensus` 组件提供的协调。集群支持两种实现：

- 基于无冲突复制数据类型的基于 CRDT 的方法
- 一种基于 Raft 的方法，基于流行的基于日志的共识算法

[共识组件](https://ipfscluster.io/documentation/guides/consensus)部分概述了它们之间的相关细节和权衡。该选择（必须在初始化期间执行并且不能轻易更改）严重影响集群 peer 节点中添加、删除和处理故障的过程。我们建议使用 CRDT 共识组件。

共享状态可以被检查，并且是每个 peer 本地存在的 `ipfs-cluster-ctl pin ls` 唯一信息。必须在运行时从它们各自的 peer 获取 pinset 状态 ( status) 信息或 peer 信息 (  peer s ls)，而不是运行命令的 peer 并组装在一起。

### 下载和安装
为了运行 IPFS 集群 peer 并在集群上执行操作，您需要获取 `ipfs-cluster-service` 和 `ipfs-cluster-ctl` 二进制文件。前者运行集群 peer 体。后者允许与之交互：

- 访问[下载页面](https://ipfscluster.io/download)以获取有关获取 `ipfs-cluster-service` 和 `ipfs-cluster-ctl`.
- 将二进制文件放在系统用户可以在无人看管的情况下运行它们的地方 `ipfs`（通常 `/usr/local/bin`）。应该安装并运行 IPFS 集群 `ipfs`（go-ipfs）。
- 考虑将您的系统配置为自动启动 `ipfs`（ `ipfs-cluster-service` 注意检查您是否需要确保您的集群完全可操作并且 peer 事先发现彼此）。此处提供了一些示例 Systemd 服务文件：[ipfs-cluster-service](https://raw.githubusercontent.com/hsanjuan/ansible-ipfs-cluster/master/roles/ipfs-cluster/templates/etc/systemd/system/ipfs-cluster.service)、[ipfs](https://raw.githubusercontent.com/hsanjuan/ansible-ipfs-cluster/master/roles/ipfs/templates/etc/systemd/system/ipfs.service)。

#### 初始化
要创建和生成默认配置文件、 peer 的唯一身份和空的 peer 存储文件，请运行：

	$ ipfs-cluster-service init --consensus <crdt/raft>
这假定该 `ipfs-cluster-service` 命令安装在您的 `$PATH`.

如果一切顺利，运行此命令后将有三个不同的文件 `$HOME/.ipfs-cluster`

- `service.json`

	包含默认 peer 配置。集群中的所有 peer 都应该具有完全相同的配置。
- `identity.json`

	包含 peer 私钥和 ID。这些对于每个集群 peer 都是唯一的。
- ` peer store`

	是一个空文件，用于存储其他 peer 的 peer 地址，以便该 peer 知道在哪里联系它们。

` --consensus` 标志选择是使用 `raft` 还是使用 `crdt` 段初始化配置。所有 peer 体都应该以相同的方式初始化。`raf
t` 和 `crdt` 之间的选择取决于多种因素并影响集群启动和修改  peer set 的方式。

 `ipfs-cluster-service init`  生成的新 `service.json` 文件在选择中会有一个随机生成 `secret` 的值。`cluster` 要使集群正常工作，该值在所有集群 peer 中应该相同。这通常是陷阱的来源，因为在各处初始化默认配置会导致不同的随机秘密。

如果存在，`CLUSTER_SECRET` 则在运行时使用环境值 `ipfs-cluster-service init` 来设置集群的 `secret` 值。

##### 远程配置
`ipfs-cluster-service` 可以初始化为使用可在 HTTP(s) 位置访问的远程配置文件，每次启动 peer 体时读取该文件以获取运行配置。这对于初始化具有相同配置的所有 peer 并为其提供无缝升级很有用。

一个很好的技巧是使用 IPFS 来存储实际配置，例如，`init` 使用网关 url 调用如下：

	$ ipfs-cluster-service init http://localhost:8080/ipns/config.mydomain.com
（需要为上面的示例配置 [DNSLink TXT 记录](https://dnslink.io/)才能正常工作。也可以使用常规 URL）。

除非可以公开集群密码，否则不要公开托管配置。这仅在基于 `crdt` 且配置 `trusted_ peer s` 为非 `*`
### 可信赖的  peer  
该文件的 `crdt` 部分包含数组 `service.json` 的单个 `*` 值。`trusted_ peer s` 默认情况下，在 `crdt-mode` 上运行的 peer 信任所有其他 peer 。在 `raft` 模式下所有 peer 都信任所有其他 peer 并且此选项不存在。

[在安全和端口指南](https://ipfscluster.io/documentation/guides/security)中阅读有关受信任 peer 的更多信息。
####  peer 存储文件
` peer store` 文件将由正在运行的集群 peer 维护并将用于存储已知 peer 地址。但也可以预填充此文件（每个多地址一行）以帮助此 peer 在第一次启动时连接到其他 peer 。这是一个例子：

	/dns4/cluster1.domain/tcp/9096/ipfs/QmcQ5XvrSQ4DouNkQyQtEoLczbMr6D9bSenGy6WQUCQUBt
	/dns4/cluster2.domain/tcp/9096/ipfs/QmdFBMf9HMDH3eCWrc1U11YCPenC3Uvy9mZQ2BedTyKTDf
	/ip4/192.168.1.10/tcp/9096/ipfs/QmSGCzHkz8gC9fNndMtaCZdf9RFtwtbTEEsGo4zkVfcykD
您也可以使用 ` peer _addresses` 配置值来为其他 peer 提供地址。
#### 端口
默认情况下，集群使用：

- `9096/tcp` 

	作为集群 swarm 端点，它应该由其他集群 peer 打开和调用
- `9094/tcp`

	作为 HTTP API 端点，启用时
- `9095/tcp`

	作为代理 API 端点，启用时
- `9097/tcp`

	作为 IPFS Pinning API 端点，启用时
- `8888/tcp`

	启用后，作为 Prometheus 指标端点。
- `6831/tcp`

	作为跟踪的 Jaeger 代理端点（启用时）
	
[安全指南](https://ipfscluster.io/documentation/guides/security) 中提供了端口和端点的完整描述。

#### 生产设置
默认的 IPFS 和集群设置是保守的，适用于开箱即用的简单设置。但有许多选项可以针对以下方面进行优化：

- 大型 pinsets
- 大数量  peer s
- 具有非常高或低延迟的网络

除了此处提到的设置之外，[配置参考](https://ipfscluster.io/documentation/reference/configuration) 还包含每个配置部分的详细信息，以及每个值含义的扩展描述。

##### IPFS 配置
IPFS 守护进程可以针对生产进行优化。这些选项记录在官方存储库中

- [config 文件](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md)
- [实验功能文档](https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md)

配置说明
	
- 云部署的服务器配置文件
	
	使用 ipfs 配置文件初始化  `server`
	
	- `ipfs init --profile=server`
	- 或者如果配置已经存在使用 `ipfs config profile apply server`
	
	注意 `AddrFilters` 和 `NoAnnounce` 选项。server应该使用配置文件将它们预填充为合理的值，但根据您运行的网络类型，您可能需要修改它们。
- 数据存储设置
	
	与之前的建议不同，我们发现 `flatfs` 数据存储在现代硬件上的性能优于 `badger` 或非常大的存储库并且减少了麻烦（即不需要几分钟即可准备好）。
	
	`sync` 应该在配置中设置为  `false` （否则会对性能产生很大影响），并且支持文件系统可能应该是 `XFS` 或 `ZFS`（在处理包含大量文件的文件夹时更快）。IPFS 通过执行随机读取给磁盘施加了巨大压力，特别是在提供流行内容时。
	
	改进 Flatfs 可以通过将 [Sharding](https://github.com/ipfs/go-ipfs/blob/master/docs/datastores.md?) 功能设置为 `/repo/flatfs/shard/v1/next-to-last/3` ( `next-to-last/2` 是默认值) 。这只应针对多 TB 存储库进行。
	
	更新分片功能可以通过从配置模板初始化或通过设置它 `config` 和 `datastore_spec` 删除 `blocks/` 文件夹来完成。尽管可以使用 `ipfs-ds-convert`，但它应该在 IPFS 节点的第一次设置期间完成.
	
	`Datastore.BloomFilterSize` 根据预期的 IPFS 存储库大小，在大多数情况下应考虑增加： `1048576(1MB)` 是一个很好的开始值（更多信息在[这里](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#datastore)）。
	
	不要忘记 `Datastore.StorageMax` 根据您要专用于 ipfs 存储库的磁盘设置一个值。这将影响集群如何计算每个 peer 中有多少可用空间。
- 连接管理器设置
	
	增加 `Swarm.ConnMgr.HighWater`（最大连接数）并减少 `GracePeriod` 到 `20s`. 它可以与您的机器一样高（`10000`对于大型机器来说是一个很好的价值）。调整`Swarm.ConnMgr.LowWater` 到 HighWater 值的 25% 左右。
- 实验性 DHT 提供
	
	大型数据存储将有很多提供，因此启用 `AcceleratedDHTClient` 是一件好事。
- 文件描述符限制
	
	`go-ipfs` 环境变量控制为自己设置的 `IPFS_FD_MAXFDulimit` 值。根据您的 `Highwater` 价值，您可能希望将其增加到 `8192` 或更多。
- 垃圾收集GC
	
	我们建议在使用 IPFS 集群添加内容时禁用 IPFS 上的自动垃圾收集，因为 GC 进程可能会在添加内容时删除内容。或者可以直接将内容添加到 IPFS（这将同时锁定 GC 进程），并且仅在添加后使用集群对其进行固定。
- Bitswap
	
	当节点有大量流量时，调整 [Bitswap 内部配置](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#internalbitswap)也很重要。将默认值乘以 100 在大型机器中并非没有意义。但是，您必须找到正确的平衡点，因为这会使 IPFS 在忙于位交换时消耗更多的内存。具有 128GB RAM 的机器示例：
	
		  "Internal": {
		    "Bitswap": {
		      "EngineBlockstoreWorkerCount": 2500,
		      "EngineTaskWorkerCount": 500,
		      "MaxOutstandingBytesPer peer ": 1048576,
		      "TaskWorkerCount": 500
		    }
		  }
	如果您的机器可以处理，请将这些值乘以 2。
-  peer ings  peer 互连
	
	[ peer ings](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md# peer ing) 可用于确保集群中的 IPFS  peer 始终保持连接。
##### IPFS 集群配置
首先，确保 `ipfs-cluster-service` 守护程序可以在足够的“文件描述符计数”限制下运行是很重要的。这可以通过添加 `LimitNOFILE=infinity` 到 systemd 服务单元文件（在该`[Service]`部分中）或通过确保正确设置 ulimit 来完成。守护进程使用的文件描述符的数量与

- 打开的 badger 表和预写日志文件的数量（badger 配置）
- 并行固定的项目数（pintracker 配置）
- 正在进行的添加请求的数量。

此外，`service.json` 配置文件包含一些选项应根据您的环境、容量和要求进行调整。

- cluster 部分

	在处理大量 pins 时，您可以进一步增加 `cluster.pin_recover_interval`. 此操作将对 `pinset` 中的每个 pin 进行检查并会触发 `ipfs pin ls --type=recursive` 调用，当 pinned 的项目数量很大时调用可能会很慢。例如，对于数百万个 `pinset`，这应该至少设置为 2+ 小时。

	考虑增加 `cluster.monitor_ping_interval` 和 `monitor.*.check_interval`。`cluster.disable_repinning` 这决定了集群需要多长时间才能意识到 peer 没有响应（如果设置为 `false`，则可能会触发重新 pin ）。在您的集群中重新 pin 可能非常昂贵。因此，您可能希望将其设置为足够长。您可以对两者使用相同的值。一般来说，我们建议禁用重新 pin，尽管启用时，`replication_factor_max` 和留出 `replication_factor_min` 一些余地：即<sup>2</sup>/<sub>3</sub> 将允许一个 peer 关闭而无需在其他地方重新分配 pin 内容。
- `raft` 部分

	这些选项仅在运行基于 raft 共识的集群时适用。

	如果您计划同时重新启动所有 Raft  peer （例如，在升级之后），请考虑设置 `raft.wait_for_leader_timeout` ,为让所有 peer 有足够的时间重新启动并立即上线的设置。通常设置 `30s` 或 `1m`。

	如果您的网络非常不稳定，您可以尝试增加 `raft.commit_retries`、`raft.commit_retry_delay`。注意：更多的重试和更高的延迟意味着更慢的失败。

	对于高延迟集群（比如在世界各地拥有 peer 节点），您可以尝试增加 `heartbeat_timeout`、`election_timeout`，`commit_timeout` 和`leader_lease_timeout` 尽管默认值已经相当大了。对于低延迟集群，这些都可以减少（至少减少一半）。

	对于非常大的 pin ，增加 `raft.snapshot_interval`. 如果您的集群 pin 或取消 pin 非常频繁，请增加 `raft.snapshot_threshold`.
- `crdt` 部分

	这些选项仅在运行基于 `crdt` 的集群时适用。

	`crdt.rebroadcast_interval` 将（默认 1m) 减少到几秒钟应该会使新的 peer 开始更快地下载状态，并且连接不良的 peer 应该有更多的选择来接收信息位，但会增加网络中的 `pubsub` 聊天。对于 10 个 peer 以下的集群，它可以减少到 10 秒，但通常我们建议将其保留在 1 分钟。

	最重要的是，`batching` 配置允许通过在单个增量中批量更新多个 pin（以延迟为代价）将 CRDT 吞吐量提高几个数量级。例如，您可以让集群每分钟只提交新的 pin 或者当一个批次有 500 个时，如下所示：

	      "batching": {
	        "max_batch_size": 500,
	        "max_batch_age": "1m",
	        "max_queue_size": 50000
	      },
	您可以编辑 `crdt.cluster_name`, 只要它对所有 peer 都相同。
- `pin_tracker` 部分

	pin 跟踪器 `stateless` 处理两个 pinning 队列：一个优先级和一个“普通”队列。在处理队列时，优先队列中排队的 pin 始终具有优先权。所有新的 pin 都会进入优先级队列并留在那里，直到它们变旧或重试次数过多：

		 "pin_tracker": {
		    "stateless": {
		      "max_pin_queue_size": 200000000,
		      "concurrent_pins": 10,
		      "priority_pin_max_age" : "24h",
		      "priority_pin_max_retries" : 3
		    }
		  },
	`concurrent_pins` 正确值取决于 pin 的大小和 IPFS 获取和写入磁盘的性能。我们观察到，10-20 之间的值往往比较大的值更有效（这可能会导致过多的争用）。
- `ipfs_connector` 部分

	集群执行的 pin 请求基于 IPFS 获取的最后一个阻塞（而不是 pin 请求的总长度）超时。因此 `pin_timeout` 设置可以设置得很低：如果 20 秒内没有块可以被提取，20 秒将要求集群放弃一个 pin。较低的 pin 超时让集群通过 pinning 队列更快地搅动。稍后将重试 Pin 错误。

	默认情况下，该值 `ipfs_request_timeout` 设置为保守 5m 值。但在具有数百万个项目的 `pinset` 上，这可能太低并且有必要增加它。
- `api` 部分

	`api.restapi/pinsvcapi/ipfsproxy` 根据您的 API 使用情况调整网络超时。这可以防止滥用 API 或 DDoS 攻击。请注意，如果您控制客户端，通常也可以修改客户端超时。

	只需从 api 部分中删除其配置即可禁用每个 API。
- `informer` 部分

	在具有许多 `pin/ peer s/storage` 大小的大型集群上，您可以将 `metric_ttl` 所有通知者的时间增加到 5 或 10 分钟，因为不需要非常新的空间指标等。这意味着 peer 将产生一组稳定的分配只要指标没有刷新，新的 `pin` 就可以了，这允许其他 peer “rest”，从而为处理其他任务（如 pin 恢复）提供一些空间。

	如果您想对您的 pin 进行地理分布，请使用 `region` 标签和每个 peer 的正确值设置 `tags` 分配器。

	在我们开始 pin 其他地方之前，应该将分配器设置为高的 `weight_bucket_size`，以调整 pin 队列增长的可接受程度 `pinqueue`。 默认 `weight_bucket_size` 值为 100k，这是一个非常高的数字。这意味着队列中 100k 项以下的所有 peer 在此指标方面都被视为相等，从而允许该 `freespace` 指标成为 pin 分配的决定因素（当在 `balanced` 分配器部分中进行了如此配置时）。
- `allocator` 部分

	`balanced` 分配器可以帮助有效地分配 pin 和负载。我们建议设置 `allocate_by` 为：

		[ "tag:region", "pinqueue", "freespace" ]
	给定复制因子为 3，并且正确配置了在 3 个区域中具有 peer 的集群，这会将每个 pin 副本放置在其中 1 个区域中，从队列权重最低的区域中选择该区域中具有最多可用空间的 peer （`pin queue size / weight_bucket_size`）。您还可以添加其他 `tag` 指标（即 `["tag:region", "tag:availability_zone", ...]`）。有关 pin 过程如何发生的更多信息，请参阅[pin 指南](https://ipfscluster.io/documentation/guides/pinning)。

- datastore 部分

	`badger` 默认值非常保守，旨在最大限度地减少 `Badger` 的内存消耗。在具有 128GB RAM 的大型机器上，这些已被证明要好得多：

		   "badger": {
		      "gc_discard_ratio": 0.2,
		      "gc_interval": "15m0s",
		      "gc_sleep": "10s",
		      "badger_options": {
		        "dir": "",
		        "value_dir": "",
		        "sync_writes": false,
		        "table_loading_mode": 2,
		        "value_log_loading_mode": 0,
		        "num_versions_to_keep": 1,
		        "max_table_size": 268435456,
		        "level_size_multiplier": 10,
		        "max_levels": 7,
		        "value_threshold": 512,
		        "num_memtables": 10,
		        "num_level_zero_tables": 10,
		        "num_level_zero_tables_stall": 20,
		        "level_one_size": 268435456,
		        "value_log_file_size": 1073741823,
		        "value_log_max_entries": 1000000,
		        "num_compactors": 2,
		        "compact_l_0_on_close": true,
		        "read_only": false,
		        "truncate": false
		      }
		    }
	在任何情况下，花时间了解 [Badger 选项](https://pkg.go.dev/github.com/dgraph-io/badger?utm_source=godoc#Options)都非常重要。诸如表和值日志加载模式、`max_table_size`、`value_threshold` 等对 Badger 使用多少内存以及它的执行方式有很大影响。这与 10M+ pin 组特别相关。

### 引导集群
本节说明如何根据初始化期间做出的共识选择（`crdt` 或 `raft`）首次启动集群。

集群的第一次启动是生命周期中最关键的一步。我们必须确保 peer 能够相互联系（连通性）并丢弃常见的配置错误（例如对 secret 使用不同的值）。

启动集群 peer 就像运行一样简单：

	ipfs-cluster-service daemon
但与默认情况下连接到公共 IPFS 网络并可以通过首先连接到已知的可用引导程序列表来发现其中的其他 peer 的 IPFS 守护进程不同，集群 peer 在专用网络上运行并且没有任何公共 peer 引导到。

	在启动集群之前确保 `ipfs` 守护进程正在运行。虽然这不是一个严格的要求，但它避免了一些错误消息。
因此，在第一次启动 IPFS 集群 peer 时，提供信息非常重要，以便它们可以发现其他 peer 并加入集群。一旦 peer 成功启动一次，就可以随后使用上面的命令重新启动它。在关闭期间，每个 peer 的 ` peer store` 文件将被更新以记住其他 peer 的已知地址。

正如我们将在下面看到的，第一次启动的要求略有不同，具体取决于您将运行基于 [crdt](https://ipfscluster.io/documentation/guides/consensus#crdt) 的集群还是基于 [raft](https://ipfscluster.io/documentation/guides/consensus#raft) 的集群。您可以在 [CRDT 与 Raft](https://ipfscluster.io/documentation/guides/consensus#crdt-vs-raft-comparison)表中阅读有关两者之间差异的更多信息。

	集群中的所有 peer 必须以相同的模式运行，无论是 CRDT 还是 Raft。

- 以 CRDT 模式引导集群

	这是启动集群的最简单选项，因为基于 crdt 的 peer 必须成为集群的一部分的唯一要求是至少联系一个其他 peer 。这可以通过多种方式实现：
	
	- 将其他 peer 的地址添加到 ` peer _addresses` 配置中的阵列。
	- 用其他 peer 的地址预先填充 ` peer store` 文件。
	- `--bootstrap < peer -multiaddress1, peer -multiaddress2>` 带着标志启动。

		请注意，使用此标志将自动信任给定的 peer 。有关信任的更多信息，请阅读 [CRDT 部分](https://ipfscluster.io/documentation/guides/consensus#crdt)。
	- 在支持 mDNS 发现的本地网络中， peer 将自动发现彼此，无需采取额外措施。

	示例 
	
	1. 在基于 CRDT 的集群中启动第一个 peer ：

			ipfs-cluster-service daemon
	2. 通过自定义 ` peer store` 在基于 CRDT 的集群中启动更多 peer 。给定的多地址对应于第一个 peer ：

			echo "/dns4/cluster1.domain/tcp/9096/ipfs/QmcQ5XvrSQ4DouNkQyQtEoLczbMr6D9bSenGy6WQUCQUBt" >> ~/.ipfs-cluster/ peer store

			ipfs-cluster-service daemon
	3. `--bootstrap` 使用标志在基于 CRDT 的集群中启动更多 peer 。给定的多地址对应于第一个 peer ：

			ipfs-cluster-service daemon --bootstrap /dns4/cluster1.domain/tcp/9096/ipfs/QmcQ5XvrSQ4DouNkQyQtEoLczbMr6D9bSenGy6WQUCQUBt
- 在 Raft 模式下引导集群

	在 Raft Clusters 中， peer  的第一次启动不仅要联系不同的  peer ，还要完成成为 Raft Cluster 成员的任务。因此， peer 的第一次启动必须始终使用 `--bootstrap` 标志：

	示例

	1. 在基于 Raft 的集群中启动第一个 peer ：

			ipfs-cluster-service daemon
	2. 在基于 Raft 的集群中启动更多 peer 。给定的多地址对应于第一个 peer ：

			ipfs-cluster-service daemon --bootstrap /dns4/cluster1.domain/tcp/9096/ipfs/QmcQ5XvrSQ4DouNkQyQtEoLczbMr6D9bSenGy6WQUCQUBt
	3. 在  peer  之前已经成功加入过 Raft 集群的情况下开始：

			ipfs-cluster-service daemon
- 验证成功的引导

	启动集群 peer 后（尤其是第一次这样做），您应该检查一切是否正常：

	检查日志中的错误。成功的 peer 启动将打印“READY”消息：

		INFO    cluster: ** IPFS Cluster is READY **
	运行 `ipfs-cluster-ctl id` 以验证本地集群 peer 的详细信息。您应该能够看到集群 peer 和它所连接的 IPFS 守护进程的信息：

		$ ipfs-cluster-ctl id
		QmYY1ggjoew5eFrvkenTR3F4uWqtkBkmgfJk8g9Qqcwy51 |  peer name | Sees 3 other  peer s
		  > Addresses:
		    - /ip4/127.0.0.1/tcp/9096/ipfs/QmYY1ggjoew5eFrvkenTR3F4uWqtkBkmgfJk8g9Qqcwy51
		    - /ip4/192.168.1.10/tcp/9096/ipfs/QmYY1ggjoew5eFrvkenTR3F4uWqtkBkmgfJk8g9Qqcwy51
		  > IPFS: QmPFJcZfhFCmz1rAoew214h9d7Nv4aseqtCg5sm4fMdeYq
		    - /ip4/127.0.0.1/tcp/4001/ipfs/QmPFJcZfhFCmz1rAoew214h9d7Nv4aseqtCg5sm4fMdeYq
		    - /ip4/127.0.0.1/tcp/4002/ws/ipfs/QmPFJcZfhFCmz1rAoew214h9d7Nv4aseqtCg5sm4fMdeYq
		    - /ip6/::1/tcp/4001/ipfs/QmPFJcZfhFCmz1rAoew214h9d7Nv4aseqtCg5sm4fMdeYq
		    - /ip6/::1/tcp/4002/ws/ipfs/QmPFJcZfhFCmz1rAoew214h9d7Nv4aseqtCg5sm4fMdeYq
	- CRDT 集群 peer 可能需要几分钟才能发现集群中的其他 peer （取决于它们是如何引导的），即使在 READY 消息之后也是如此。
	- 如果 `Raft Cluster  peer ` 没有成功加入或重新加入 Raft Cluster，几秒钟后将无法启动。READY 消息表明 peer 很好，即使在联系其他已关闭的 peer 时它可能会显示错误。
- 常见问题

	以下是一些在第一次启动时通常会出错的事情。有关如何调试集群 peer 的更多说明，请参阅故障[排除指南](https://ipfscluster.io/documentation/guides/troubleshooting)。

	- 集群 peer 之间的不同秘密

		使用不同的集群机密会导致集群 peer 无法通信，同时导致 `libp2p` 在日志上抛出不明显的错误：

			dial attempt failed: incoming message was too large
	-  peer 之间没有连接

		以下错误消息表明存在连接问题。确保[打开了必要的端口](https://ipfscluster.io/documentation/guides/security)并且可以在 peer 之间建立连接：

			dial attempt failed: context deadline exceeded
			dial backoff
			dial attempt failed: connection refused

### 部署自动化
我们有许多自动化来帮助配置和部署 IPFS 集群：

- [Ansible 角色](https://ipfscluster.io/documentation/deployment/automations/#ansible-roles)

	可在 [https://github.com/hsanjuan/ansible-ipfs-cluster](https://github.com/hsanjuan/ansible-ipfs-cluster) 获得用于配置和部署的 Ansible 角色 `ipfs-cluster-service`，`ipfs-cluster-ctl` 以及 go-ipfs（包括模板化配置文件）。
- [Docker](https://ipfscluster.io/documentation/deployment/automations/#docker)

	IPFS 集群在 [https://hub.docker.com/r/ipfs/ipfs-cluster/](https://hub.docker.com/r/ipfs/ipfs-cluster/) 提供官方 dockerized 版本以及 `docker-compose`.

	如果你想运行 `/ipfs/ipfs-clusterDocker` 容器之一，重要的是要知道：

	- 容器不运行 `go-ipfs`，您应该单独运行 IPFS 守护程序，例如，使用 `ipfs/go-ipfsDocker` 容器。需要相应地 `ipfs_connector/ipfshttp/node_multiaddress` 调整配置值才能访问 IPFS API。此路径支持 DNS 地址 ( `/dns4/ipfs1/tcp/5001`)，并 `CLUSTER_IPFSHTTP_NODEMULTIADDRESS` 在启动容器时从环境变量中设置，并且不存在先前的配置。
	- 默认情况下，我们使用 `./data/ipfs-cluster` 作为 IPFS 集群配置路径。我们建议安装此文件夹以便为您的  peer  提供自定义配置和/或数据持久性。这通常是通过传递 `-v <your_local_path>:/data/ipfs-cluster` 给来实现的 `docker run`。

	容器（此处的 [Dockerfile](https://github.com/ipfs-cluster/ipfs-cluster/blob/master/Dockerfile) 运行一个 `entrypoint.sh` 脚本，该脚本在不存在配置时初始化 IPFS 集群。配置值可以通过设置环境变量来控制，如 [配置参考](https://ipfscluster.io/documentation/reference/configuration)中所述。

	默认情况下 `crdt`，共识用于初始化配置。这可以通过设置覆盖 `IPFS_CLUSTER_CONSENSUS=raft`。同样，`badger` 用作默认后端，并且可以覆盖设置 `IPFS_CLUSTER_DATASTORE=leveldb`。

		除非您使用 docker 运行--net=host，否则您将需要设置 $CLUSTER_IPFSHTTP_NODEMULTIADDRESS 或确保配置具有正确的 node_multiaddress.
	
		除非您使用 docker 运行--net=host，否则 REST API 端点将仅在容器内本地侦听。如果您希望从容器外部访问此端点，您可能需要设置$CLUSTER_RESTAPI_HTTPLISTENMULTIADDRESS为/ip4/0.0.0.0/tcp/9096，以及必要的端口转发 ( -p 9094:9094）.

	- docker 撰写

		我们还提供了一个示例，该示例 `docker-compose.yml` 能够启动一个 IPFS 集群，其中有两个集群 peer 和两个 IPFS 守护进程正在运行。

		在第一次启动期间配置会自动生成并将与 `./compose` 文件夹一起保存在文件夹中以供下次启动 `ipfs` 使用。删除此文件夹以重置 Docker Compose 设置。

		只有 IPFS swarm 端口 (tcp `4001/4101`) 和 IPFS Cluster API 端口 ( `tcp 9094/9194`) 暴露在容器之外。

		此 compose 文件作为示例提供，说明如何使用 Docker 容器设置多 peer 集群。您可能需要根据自己的需要对其进行调整。
- [Kubernetes 与 Kustomize](https://ipfscluster.io/documentation/deployment/automations/#kubernetes-with-kustomize)

	会单独介绍

## 协作集群
本节介绍 IPFS Cluster 中的协作集群功能。协作集群允许单独的、不受信任的 peer 作为“追随者”加入 IPFS 集群（但无权修改或编辑 pinset）。
### 托管协作集群(协作集群设置)
为了创建其他人可以订阅的您自己的协作集群，您需要：

- 在 CRDT 模式下设置 [IPFS 集群的常规生产部署](https://ipfscluster.io/documentation/collaborative/setup/#trusted- peer s-setup)（这些将是您的“受信任的 peer ”）。
- [分发配置模板](https://ipfscluster.io/documentation/collaborative/setup/#distributing-a-configuration-template)，以便追随者可以轻松加入集群。
- 让用户利用 [ipfs-cluster-follow](https://ipfscluster.io/documentation/collaborative/setup/#ipfs-cluster-follow)。

#### 可信 peer 体设置
	协作集群必须使用 CRDT 模式。
设置协作集群的第一步是部署一个具有一个或多个 peer 的常规 CRDT 集群。

请遵循[生产部署](https://ipfscluster.io/documentation/deployment/)部分中的说明，尤其是与 [CRDT 模式引导](https://ipfscluster.io/documentation/deployment/bootstrap/)相关的说明。最简单的是运行单个 peer 集群，尽管您可以选择运行两个或更多集群以获得一些冗余。

具有默认配置的单个 peer 的说明摘要版本如下：

	# Start your ipfs daemon
	$ ipfs-cluster-service init --consensus crdt
	$ ipfs-cluster-service daemon
	# Write down:
	# - 生成的集群秘密(需要在其他 peer 节点中重用)
	# -  peer 体ID(这将是一个“可信的 peer 体”)
	# - 其他 peer 体可以访问的多地址(通常为/ip4/public_ip/tcp/9096/p2p/ peer _id)
配置并运行基本集群后，您需要确保配置部分中的 `trusted_ peer s` 阵列 `crdt` 设置为基本集群中的 peer  ID。否则（如果设置为默认值 `*`），任何人都可以修改 `pinset`，这可能是您不想要的。

查看集群 peer 中生成的配置

- 您设置中的所有受信任的 peer 可能应该具有相同的配置（除非它们在具有不同要求的机器上运行）
- `trusted_ peer s` 应设置为原始集群中受您控制（或某人的受信任控制）的 peer  ID 列表。
- 您应该已经生成了一个集群 `secret`。稍后分发这个秘密就可以了。
- 根据您的集群设置、您计划加入集群的人员以及对这些追随者 peer 的信任级别，您可以设置 `replication_factor_min/max`. 对于一般用例，我们建议 `-1`（所有东西都固定在任何地方）。协作集群的主要用例是确保内容的广泛分发和复制。
- 您可以根据自己的喜好修改 `crdt/cluster_name` 值，但请记住告知您的关注者其值。

原则上，追随者可以使用与您信任的同伴完全相同的配置，但我们建议定制特定的追随者配置，如下一节所述。

- 分发配置模板

	任何追随者 peer 都可以启动一个 IPFS 集群 peer ，该 peer 将使用以下信息加入您的协作集群：

	- `trusted_ peer s` 集群中的列表。
	- 至少其中一个 peer 的完整、可访问的多地址。
	- 集群 `secret`。
	- 改变 `crdt/cluster_name` 值

	但是，最好分发一个配置模板，该模板为您的集群设置了正确的所有选项。最终，您将希望追随者  peer  使用 `ipfs-cluster-follow`，而不是 `ipfs-cluster-service`.

	- 为关注者创建配置模板

		从技术上讲，追随者节点可以使用与受信任节点相同的配置，但我们建议考虑进行一些修改。以下内容适用于 `service.json` 您将分发给关注者的文件副本：

		- 将 Badger 设置为“文件加载模式 file loading mode”

			如果您希望用户从低内存设备（1GB RAM 或更少）加入您的集群，请务必在 `badger 部分` 为这些 peer 分配的配置部分中设置 `table_loading_mode` 和 `value_log_loading_mode` 为零 。
		- 设置 ` peer _addresses` 为您信任的  peer  的地址。

			每当任何追随者 peer 启动时，这些都必须是可访问的，因此请确保与您的集群有连接。
		- 考虑删除 `api( restapi, ipfsproxy)` 部分中的任何配置

			不应该告诉追随者  peer  他们的  peer   API 应该是什么样子。错误配置 API 可能会打开不需要的安全漏洞。通过创建一个安全的、仅限本地的 API 端点来覆盖任何  `ipfs-cluster-follow`  配置。
		- 如果您为受信任的 peer 配置修改了它们，请重置 `connection_manager/high_water` 并设置为合理的 `low_water` 默认值。
		- 设置 `follower_mode` 为 `true`

			虽然不受信任的 peer 不能对集群 `pinset` 做任何事情，但它们仍然可以修改自己的视图，这可能会非常令人困惑。此设置（ `ipfs-cluster-follow` 自动激活）确保在尝试执行写入操作时返回有用的错误消息。
		- 如果您正在运行多个协作集群或者希望您的用户这样做，请考虑 `listen_multiaddress` 通过将默认端口更改为其他（希望未使用）来修改定义的地址。

			您也可以使用零以便 peer 在启动期间选择一个随机的空闲端口，但这将导致 peer 在每次重新启动时更改端口（这取决于您的设置有多重要）。

		在所有这些更改之后，您将拥有一个 `service.json` 可以分发给追随者的文件。先测试一下：

			ipfs-cluster-service -c someFolder init
		
		- 将生成的默认值替换为 `service.json` 您创建的跟随配置。
		- 运行 `ipfs-cluster-service -c someFolder daemon` 并确保 peer 启动并加入主集群（`ipfs-cluster-ctl  peer s ls` 在受信任的 peer 之一上应该显示它）。
	- 分发模板
		- 一旦你有一个配置模板要提供给你的追随者，你可以将它托管在网络服务器上并通过 HTTP(s) 访问它或者更有趣的是，将它添加到 IPFS（以及你创建的集群）并制作它可通过每个 IPFS 守护进程的网关访问：

				ipfs-cluster-ctl add follower_service.json --name follower-config
		- 一旦配置在 IPFS 上，任何追随者 peer 都可以通过本地 IPFS 网关读取它来配置。
			- 例如，当使用 `ipfs-cluster-service`：

					ipfs-cluster-service init http://127.0.0.1:8080/ipfs/Qm....
			- 但是，我们建议使用ipfs-cluster-follow, 代替：

					ipfs-cluster-follow myCluster init http://127.0.0.1:8080/ipfs/Qm...
		如果您有可以控制的域名，我们建议使用它来设置指向配置哈希的 [DNSLINK TXT](https://dnslink.io/) 记录 `/ipfs/Qm...`。这样您的用户可以改用这种方式 `http://127.0.0.1:8080/ipns/my.domain.com`，并且您可以通过更新 `dnslink` 值来更新配置。配置总是在 peer 体启动期间读取（而不是在初始化期间）。

- ipfs-cluster-follow

	`ipfs-cluster-follow` 命令经过特殊设计，可以让加入协作集群变得超级容易。我们建议始终使用此命令而不是 `ipfs-cluster-service`.

	`ipfs-cluster-follow` 在加入协作集群时，做了很多事情来改善用户体验：

	- 它允许通过基于给定集群名称为每个节点分离配置文件夹来初始化和运行多个节点。
	- 它被简化为使用 IPFS 托管的配置模板，在初始化期间进行 `my.domain.com` 转换 `http://127.0.0.1:8080/ipns/my.domain.com`。
	- 它可以在单个命令中初始化和运行 peer 体。
	- 默认情况下，它在本地套接字上运行本地 HTTP API 端点，防止在同一端口上侦听的多个 API 端点发生冲突，并确保无法从外部访问追随者 peer  API。模板中提供的任何 API 配置都将被丢弃。
	- 它 `follower_mode` 会自动设置以及对配置的许多其他小更改，以获得更好的用户体验。

	您可以在 `ipfs-cluster-follow` [此处](https://ipfscluster.io/documentation/reference/follow/)和 [加入协作集群](https://ipfscluster.io/documentation/collaborative/joining)找到更多信息。

- 加入协作集群

	协作集群允许人们联合起来在 IPFS 网络上备份和分发有趣的内容。

	这些集群由一组受信任的 peer 组成，它们添加和修改集群中的项目列表，以及一组追随者，它们订阅集群并相应地固定事物。

	- 使用 `ipfs-cluster-follow` 加入集群

		您应该能够使用一行命令加入：

			ipfs-cluster-follow <clusterName> run --init <template-configuration-url>
		始终在与集群中受信任 peer 使用的版本相同或更新的 IPFS 集群版本上运行。

		如果您没有或不知道您的配置的 URL，您可以向为该集群运行受信任 peer 的任何人询问它。
		
		- 如果您知道正确的信息（见下文），您可以构建自己的配置，将其添加到 IPFS，并为其使用网关 URL `http://127.0.0.1:8080/ipfs/Qmhash...`。
		- 您也可以 `ipfs-cluster-follow <clusterName> init `使用随机 url 运行，然后将生成的替换为 `service.json`您自己的。

	您可以使用环境变量自定义和覆盖在 [remote] 模板中设置的任何配置选项，如[配置参考](https://ipfscluster.io/documentation/reference/configuration/#using-environment-variables-to-overwrite-configuration-values)中所述。
	
	- 寻找要加入的协作集群

		访问 [collab.ipfscluster.io](https://collab.ipfscluster.io/) 以获取您可以加入的协作集群的列表。
	- 手动配置跟随节点

		加入协作集群的主要要求是了解有关它的正确信息：

		- `trust_ peer s` 列表
		- 联系至少一个受信任 peer 的多地址
		- 集群的 `secret` 值 
		- （`crdt/cluster_name` 如果不是默认值）

		将该信息放在由 `service.json` 生成的配置文件中 `ipfs-cluster-service init` 应该足以创建一个使用 `ipfs-cluster-service` 或 `ipfs-cluster-follow` 加入现有集群的 peer  。

		有关详细信息 `ipfs-cluster-follow`，请查看[参考资料](https://ipfscluster.io/documentation/reference/follow)。

