# go-ipfs 的配置文件
go-ipfs 的配置文件是一个 json 文件。它在节点实例化时只读取一次，主要用于脱机命令或启动守护程序。在正在运行的守护程序上执行的命令将不会在运行时读取配置文件。

## 简介
配置文件允许快速调整配置。配置文件可以使用 `--profile` flag 在 `ipfs init` 或 `ipfs config profile apply` 命令中来应用。应用配置文件时，将在 $IPFS_PATH 中创建配置文件的备份

可用：

- server

	建议用于具有公共 IPv4 地址（服务器，VPS 等）的节点，禁用本地网络中的主机和内容发现。
- local-discovery

	将默认值设置为受 server 配置文件影响的字段，在本地网络中启用发现机制。
- test

	减少外部干扰，对于在测试环境中运行 ipfs 非常有用。请注意，使用这些设置，如果没有手动引导，节点将无法与网络的其余部分通信。
- default-networking

	恢复默认网络设置。恢复经过 test 配置的配置文件。
- badgerds

	实验性 badger 数据存储方法替换默认数据存储配置。如果您之后应用此配置文件 `ipfs init`，则需要将数据存储转换为新配置。可以使用 [ipfs-ds-convert](https://github.com/ipfs/ipfs-ds-convert) 执行此操作

	警告：badger 数据存储区是实验性的。请确保经常备份已防止数据丢失。
- default-datastore

	恢复默认数据存储配置，恢复经过 badger	配置的配置文件
- lowpower

	减少系统上的守护进程的开销。可能会影响节点功能，内容发现和数据提取的性能可能会相应下降。

## 参数说明
### Addresses
包含有关此节点要使用的各种侦听器地址的信息。

- API Multiaddr 

	描述为本地 HTTP API 提供服务的地址。默认：`/ip4/127.0.0.1/tcp/5001`
- Gateway Multiaddr

	描述为本地网关提供服务的地址。默认：`/ip4/127.0.0.1/tcp/8080`
- Swarm

	多个数据库的数组，描述要监听 p2p 群连接的地址。

	默认：

		[
		  "/ip4/0.0.0.0/tcp/4001",
		  "/ip6/::/tcp/4001"
		]
- Announce

	如果非空，则此阵列指定要向网络通告的群地址。如果为空，守护程序将通知推断的群集地址。默认： []
- NoAnnounce 

	群集地址不向网络通告。默认： []

### API
包含 API 网关使用的信息。

- HTTPHeaders 

	要在 API HTTP 服务器的响应中设置的 HTTP 标头的映射。默认： null
	
	例：
	
			{
				"Foo": ["bar"]
			}
	
### Bootstrap
Bootstrap 是一系列可信节点的多个连接器，用于连接以启动与网络的连接。 默认值：ipfs.io 引导程序节点
### Datastore
包含与磁盘存储系统的构造和操作相关的信息。

- StorageMax 

	ipfs 存储库数据存储区大小的软上限。用来提供 `StorageGCWatermark` 用于计算是否触发 gc 运行（仅当 `--enable-gc` 设置了标志时）。
	
	- 默认： 10GB
- StorageGCWatermarkStorageMax

	守护程序在启用自动 gc 的情况下运行，自动触发垃圾回收的阈值，数字为百分比（该选项当前默认为 false ）。

	- 默认： 90
- GCPeriod

	持续时间，指定运行垃圾回收的频率。仅在启用自动 gc 时使用有效。
	
	- 默认： 1h
- HashOnRead 

	布尔值，如果设置为 true，则将对磁盘中的所有块读取进行散列和验证。这将导致 CPU 利用率增加。
- BloomFilterSize 

	一个数字，表示 blockstore 的 bloom 过滤器的大小（以字节为单位）。值为零表示要禁用的功能。

	默认： 0
- Spec

	Spec 定义了 ipfs 数据存储区的结构。它是一个可组合的结构，其中每个数据存储区由 json 对象表示。数据存储可以包装其他数据存储以提供额外的功能（例如，度量，日志记录或缓存）。
	
这可以手动更改，但是，如果进行任何需要不同磁盘结构的更改，则需要运行 [ipfs-ds-convert](https://github.com/ipfs/ipfs-ds-convert) 工具将数据迁移到新结构中。有关此配置选项的可能值的更多信息，请参阅 `docs/datastores.md`

默认：

	{
	  "mounts": [
		{
		  "child": {
			"path": "blocks",
			"shardFunc": "/repo/flatfs/shard/v1/next-to-last/2",
			"sync": true,
			"type": "flatfs"
		  },
		  "mountpoint": "/blocks",
		  "prefix": "flatfs.datastore",
		  "type": "measure"
		},
		{
		  "child": {
			"compression": "none",
			"path": "datastore",
			"type": "levelds"
		  },
		  "mountpoint": "/",
		  "prefix": "leveldb.datastore",
		  "type": "measure"
		}
	  ],
	  "type": "mount"
	}

### Discovery
包含用于配置 ipfs 节点发现机制的选项。

- MDNS

	多播 dns 对等体发现的选项。
	
	- Enabled

		mdns 是否应该处于活动状态的布尔值。默认： true
- Interval

	在发现检查之间等待的秒数。
- Routing

	内容路由模式。可以用守护进程 `--routing` 标志覆盖。有效模式是：

	- dht （默认）
	- dhtclient
	- none

### Gateway
HTTP 网关的选项。

- HTTPHeaders 

	用于设置网关响应的标头。

	默认：

		{
			"Access-Control-Allow-Headers": [
				"X-Requested-With"
			],
			"Access-Control-Allow-Methods": [
				"GET"
			],
			"Access-Control-Allow-Origin": [
				"*"
			]
		}
- RootRedirect

	用于将请求重定向 `/` 到的URL 。默认： ""
- Writeable 一个布尔值

	用于配置网关是否可写。默认： false
- PathPrefixes TODO

	默认： []

### Identity
- PeerID 这个 config peer 的唯一 PKI 身份标签。设置在 init 并且永远不会读取，它仅仅是为了方便起见。Ipfs 将始终在运行时从其密钥对生成 peerID。
- PrivKey base64 编码的 protobuf 描述（和包含）节点私钥。

### Ipns
- RepublishPeriod 持续时间

	指定重新发布 ipns 记录的频率，以确保它们在网络上保持新鲜。如果未设置，默认为 4 小时。
- RecordLifetime 一个持续时间

	指定要在 ipns 记录上设置的有效生命周期的值。如果未设置，默认为24小时。
- ResolveCacheSize

	要在已解析的 ipns 条目的 LRU 高速缓存中存储的条目数。新数据将保持缓存状态，直到其生命周期结束。默认： 128	

### Mounts
FUSE 安装点配置选项。

- IPFS Mountpoint for /ipfs/。
- IPNS Mountpoint for /ipns/。
- FuseAllowOther 在挂载点上设置 FUSE allow other 选项。

### Reprovider
- Interval

	设置将本地内容重新提交到路由系统的间隔时间。如果未设置，则默认为12小时。如果设置为该值"0"，则将禁用内容重新提取。

	注意：禁用内容重新生成将导致网络上的其他节点无法发现您拥有自己拥有的对象。如果您希望禁用此功能并让网络了解您拥有的内容，则必须定期手动宣布您的内容。
- Strategy

	告诉 reprovider 应该宣布什么。有效的策略是：
	
	- `all`（默认） - 宣布所有存储的数据
	- `pinned` - 仅发布固定数据
	- `root` - 只宣告递归 pinned keys 和 root keys

### Swarm
配置群的选项。

- AddrFilters

	一组地址过滤器（multiaddr netmasks），用于过滤拨号。有关更多信息，请参阅[此问题](https://github.com/ipfs/go-ipfs/issues/1226#issuecomment-120494604)。
- DisableBandwidthMetrics 

	一个布尔值,当设置为 true，将导致 ipfs 无法跟踪带宽指标。禁用带宽指标可以带来轻微的性能提升，并减少内存使用量。
- DisableNatPortMap

	禁用 NAT 发现。
- DisableRelay

	禁用 p2p-circuit 中继传输。
- EnableRelayHop 

	为节点启用 HOP 中继。如果启用此选项，则该节点将充当连接对等体的中继环路中的中间（Hop Relay）节点。

### ConnMgr
连接管理器配置。

- Type 

	设置要使用的连接管理器的类型，选项为："none" 和 "basic"。
- LowWater 

	LowWater 是要维护的最小连接数。
- HighWater 

	HighWater 是连接数，当超过时，将触发连接 GC 操作。
- GracePeriod 

	GracePeriod 是新连接不受连接管理器优雅关闭时持续的时间。	