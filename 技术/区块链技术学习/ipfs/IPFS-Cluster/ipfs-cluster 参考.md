# ipfs-cluster 参考
## 配置参考
默认情况下，可以在 `~/.ipfs-cluster` 文件夹中找到所有 IPFS 集群配置和持久数据。有关此文件夹中持久数据的更多信息，请参阅[数据、备份和恢复部分](https://ipfscluster.io/documentation/guides/backups)。

	ipfs-cluster-service -c <path> 设置配置文件夹的位置。这也由 IPFS_CLUSTER_PATH 环境变量控制。
 `ipfs-cluster-service` 程序使用两个主要的配置文件：

- `service.json`

	包含集群 peer 配置，通常在所有集群 peer 中相同。
- `identity.json`

	包含每个 peer 使用的唯一标识。`identity.json` 文件是在 `ipfs-cluster-service init` . 它包括一个 base64 编码的私钥和与之关联的公共 peer  ID。此 peer  ID 标识集群中的 peer 。你可以在[这里](https://ipfscluster.io/0.14.2_identity.json)看到一个例子。

	重新运行时不会覆盖此文件 `ipfs-cluster-service -f init` 。如果您想生成一个新的，您需要先将其删除。

	可以使用 `CLUSTER_ID` 和 `CLUSTER_PRIVATEKEY` 环境值覆盖身份字段。

### 手动身份生成
在为多个 peer 自动部署或创建配置时，事先手动生成 peer  ID 和私钥可能会很方便。

您可以使用以下配置以配置预期的格式获取有效的 peer  ID 及其关联的私钥 `ipfs-key` ：

	ipfs-key -type ed25519 | base64 -w 0
- `service.json`

	`service.json` 文件包含集群 peer 及其不同组件的所有可配置选项。配置文件分为多个部分。每个部分代表一个组件。该部分中的每个项目都代表该组件的一个实现，并包含特定的选项。 `ipfs-cluster-service init`运行时会创建一个具有合理值的默认 `service.json`  文件。

	重要提示：`cluster` 配置部分存储一个 32 字节的十六进制编码 `secret`，用于保护所有集群 peer 之间的通信。`secret` 必须由所有集群 peer 共享。使用空密钥具有安全隐患（请参阅[安全](https://ipfscluster.io/documentation/guides/security)部分）。

	如果存在，则在运行 `ipfs-cluster-service init` 以设置集群 `secret` 值时使用 `CLUSTER_SECRET` 环境值。

例如，这是一个默认 [service.json](https://ipfscluster.io/0.14.2_service.json) 配置文件。

该文件如下所示：

	{
	  "source": "url" // 远程配置可能会出现单个源字段
	  "cluster": {...},
	  "consensus": {
	    "crdt": {...}, //CRDT或者raft
	    "raft": {...},
	  },
	  "api": {
	    "ipfsproxy": {...},
	    "restapi": {...}
	  },
	  "ipfs_connector": {
	    "ipfshttp": {...}
	  },
	  "pin_tracker": {
	    "stateless": {...}
	  },
	  "monitor": {
	    "pubsubmon": {...}
	  },
	  "allocator": {
	    "balanced": {...}
	  },
	  "informer": {
	    "disk": {...},
		"tags": {...},
		"pinqueue": {...},
	  },
	  "observations": {
	    "metrics": {...},
	    "tracing": {...}
	  },
	  "datastore": {
	    "badger": {...}, // badger 或者 leveldb
	    "leveldb": {...},
	  }
	}
下面详细记录了不同的部分和小节。

#### 使用环境变量覆盖配置值
配置文件中的所有选项都可以通过设置环境变量来覆盖。即将

- `CLUSTER_SECRET` 覆盖 `secret `值；
- `CLUSTER_LEAVEONSHUTDOWN` 将覆盖 `leave_on_shutdown`值；
- `CLUSTER_RESTAPI_CORSALLOWEDORIGINS` 将覆盖 `restapi.cors_allowed_origins`值。

通常，环境变量采用 `CLUSTER_<COMPONENTNAME>_KEYWITHOUTUNDERSCORES=value`. 使用 `ipfs-cluster-service init`.

#### 远程配置
从 0.11.0 版开始，可以使用包含指向标准文件的 URL `service.json` 的单个字段来初始化。每次启动 peer 时都会读取 `sourceservice.json` 此配置。

远程 `service.json` 可用于将所有 peer 指向存储在相同位置的相同配置文件。也可以使用指向通过 IPFS 提供的文件的 URL。
#### 主要 cluster 部分
配置文件的主要 `cluster` 部分配置核心组件。

当这些选项（可以基于每个固定设置）未设置时，`replication_factor_min` 和控制固定默认值。`replication_factor_max` 集群总是尝试为 `replication_factor_max` 每个项目分配最多 peer 。但是，如果无法达到该数量，则只要 `replication_factor_min` 可以完成 pin 操作就会成功。一旦设置了分配，Cluster 不会自动更改它们（即增加它们）。但是，相同 CID 的新 Pin 操作将在尊重现有`replication_factor_max`  分配的同时再次尝试完成。

`leave_on_shutdown` 选项允许 peer 在完全关闭时将自己从 peer 集中删除。使用 `raft` 时最相关。这意味着，对于任何后续启动，都需要[引导](https://ipfscluster.io/documentation/deployment/bootstrap/#bootstrapping-the-cluster-in-raft-mode) peer 才能重新加入集群。

参数|默认|描述
---|---|---
`peername`|`<hostname>`|此 peer 的人名。
`secret`|`<randomly generated>`|集群密钥（在所有 peer 中必须相同）。
`leave_on_shutdown`|`false`| peer 将在关闭时将自己从集群 peer 集中删除。
`listen_multiaddress`|`["/ip4/0.0.0.0/tcp/9096", "/ip4/0.0.0.0/udp/9096/quic"]`|peers Cluster-RPC 监听端点。
`connection_manager {`|方法上限|		连接管理器配置对象。
`high_water`|`400`|	此 peer 将拥有的最大连接数。如果是，则连接将关闭，直到low_water达到该值。
`low_water`|`100`|libp2p 主机将尝试与其他 peer 保持至少这么多的连接。
`grace_period`|`"2m0s"`|	至少在此期间不会丢弃新连接。
`}`|方法下限|
`dial_peer_timeout`|`"3s"`|	在放弃之前拨打集群 peer 时要等待多长时间。
`state_sync_interval`|`"10m0s"`|	的自动触发之间的间隔StateSync。
`pin_recover_interval`|`"1h0m0s"`|	的自动触发之间的间隔RecoverAllLocal。这将自动重试失败的 pin 和 unpin 操作。
`replication_factor_min`|`-1`|	指定应固定项目的默认最小 peer 数。-1 == 全部。
`replication_factor_max`|`-1`|	指定应该固定项目的默认最大 peer 数。-1 == 全部。
`monitor_ping_interva`l	|`"15s"`|	发送时间间隔ping（用于检测停机时间）。
`peer_watch_interval`|`"5s"`|	检查当前集群 peerset 并检测此 peer 是否已从集群中删除（并关闭）的时间间隔。
`mdns_interval`|`"10s"`|	将其设置为"0"禁用 mDNS。设置为较大的值会启用 mDNS，但不再控制任何内容。
`enable_relay_hop`|`true`|	让集群 peer 充当无法直接访问的其他 peer 的中继。
`pin_only_on_trusted_peers`|`false`|集群 peer 只会将固定分配给受信任的 peer （如配置）
`disable_repinning`|`true`|不要自动重新固定分配给变得不健康（关闭）的 peer 的所有项目。
`follower_mode`|`false`|追随者模式下的 peer 在尝试执行固定等操作时会提供有用的错误消息。
`peer_addresses`|`[]`| peer 在启动时连接的完整 peer 多地址（类似于手动添加到peerstore文件的条目。
`pin_only_on_trusted_peers`|`false`|	将给予固定的可能分配限制为trusted_peers列表中的那些

### 手动生成秘密
您可以通过以下方式获得 32 位十六进制编码的随机字符串：

	export CLUSTER_SECRET=$(od  -vN 32 -An -tx1 /dev/urandom | tr -d ' \n')
####  consensus 部分
consensus 包含用于共识组件的选定实现的单个配置对象（crdt 或者 raft，但不是两者）。

- crdt

	包含 CRDT 部分使集群能够为集群状态（pinset）使用基于 [crdt 的分布式键值存储](https://ipfscluster.io/documentation/guides/consensus)。

	`batching.max_batch_size` 在本节中，通过将和设置 `batching.max_batch_age` 为大于 0（默认值）的值来启用批量提交。这两个设置控制何时提交批处理，或者通过达到最大数量的 pin/unpin 操作，或者通过达到最大时长。

	一个附加 `batching.max_queue_size` 选项提供了对 Pin/Unpin 请求执行能力。当多个 `max_queue_size` pin/unpins 等待包含在批处理中时，操作将失败。如果发生这种情况，这意味着集群无法像 pin 到达一样快地提交批次。因此，`max_queue_size` 应该增加（以适应突发）或 `max_batch_size` 增加（以执行更少的提交并希望更快地处理请求）。

	请注意，当批处理增量达到 1MB 大小时，底层 CRDT 库将自动提交。

	参数|默认|描述
	---|---|---
	`cluster_name`|`"ipfs-cluster"`|任意名称。它成为集群中所有 peer 订阅的发布订阅主题，因此它必须对所有节点都相同。
	`trusted_peers`|`[]`|默认的受信任 peer 集。有关详细信息，请参阅 [CRDT 模式](https://ipfscluster.io/documentation/guides/consensus#the-trusted-peers-in-crdt-mode)下的信任。可以设置 `[ "*" ]` 为信任所有 peer 。
	`batching {`|函数上限|将 pin 提交到 CRDT 层时的批处理设置。两者 `max_batch_size` 和 `max_batch_age` 都需要大于 0 才能启用批处理。
	    `max_batch_size`|`0`|在提交之前要包含在批处理中的最大固定/取消固定操作数。
	    `max_batch_age`|`"0s"`|未提交的批处理在提交之前等待的最长时间。
	    `max_queue_size`|`1000`|等待包含在批处理中的最大 pin/unpin 操作数。
	`}`|函数下限|		
	`peerset_metric`|`"ping"`|用于确定当前 pinset 的监视器指标的名称。
	`rebroadcast_interval`|`"1m0s"`|在没有看到其他 pubsub 消息时重新发布当前头的频率。减少这将允许新的 peer 更快地了解当前状态。
	`repair_interval`|`"1h0m0s"`|检查 crdt-datastore 是否被标记为脏的频率，并在这种情况下触发 DAG 的重新处理。
- raft

	参数|默认|描述
	---|---|---
	`init_peerset`|`[]`|当不存在 raft 状态时，指定初始 peerset 的 peer ID 数组。
	`wait_for_leader_timeout`|`"15s"`|	在timeout错误之前，要等待 raft 领导者多长时间。
	`network_timeout`|`"10s"`|Raft 协议网络操作超时前多久。
	`commit_retries`|`1`|多少次重试提交条目到 Raft 登录失败。
	`commit_retry_delay`|`"200ms"`|在提交重试之前等待多长时间。
	`backups_rotate`|`6`|清理状态时要保留多少个状态备份副本。
	`heartbeat_timeout`|`"1s"`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。
	`election_timeout`|`"1s"	`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。
	`commit_timeout`|`"50ms"`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。
	`max_append_entries`|`64`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。
	`trailing_logs`|`10240`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。
	`snapshot_interval`|`"2m0s"`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。
	`snapshot_threshold`|`8192`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。
	`leader_lease_timeout`|`"500ms"`|请参阅 [https://godoc.org/github.com/hashicorp/raft#Config](https://godoc.org/github.com/hashicorp/raft#Config)。

	`init_peersetRaft` 在内部存储和维护 peerset，但集群配置提供了使用配置部分中的 key 为 peer 的第一次启动手动提供 peerset 的选项 raft。例如：

		"init_peerset": [
		  "QmPQD6NmQkpWPR1ioXdB3oDy8xJVYNGN9JcRVScLAqxkLk",
		  "QmcDV6Tfrc4WTTGQEdatXkpyFLresZZSMD8DgrEhvZTtYY",
		  "QmWXkDxTf17MBUs41caHVXWJaz1SSAD79FVLbBYMTQSesw",
		  "QmWAeBjoGph92ktdDb5iciveKuAX3kQbFpr5wLWnyjtGjb"
		]
	这将允许您使用已经固定的 peerset 从头开始​​启动集群。

#### api 部分
 `api` 部分包含 API 组件实现的配置，旨在为与集群的交互提供端点。删除任何这些部分将禁用该组件。例如，`ipfsproxy` 从配置中删除该部分将禁用正在运行的 peer 上的代理端点。
#### ipfsproxy
该组件提供 IPFS 代理端点。这是一个模仿 IPFS 守护进程的 API。一些请求（pin、unpin、add）被 Cluster 劫持和处理。其他的则简单地转发到由`node_multiaddress`. 该组件默认配置为模仿 IPFS 守护程序中存在的 CORS 标头配置。为此，它会触发对它们的附件请求（如 CORS 预检）。

参数|默认|描述
---|---|---
`node_multiaddress`|`"/ip4/127.0.0.1/tcp/5001"`|IPFS 守护进程 API 的监听地址。
`listen_multiaddress`|`"/ip4/127.0.0.1/tcp/9095"	`|代理端点监听地址。
`log_file`|`""`|用于写入请求日志文件的文件（Apache 组合格式）。否则，它们将被写入 ipfsproxylog 设施下的集群日志。
`node_https`|`false`|使用 HTTPS 与 IPFS API 端点对话（实验性）。
`read_timeout`|`"0s"`| [https://godoc.org/net/http#Server](https://godoc.org/net/http#Server) 的参数。请注意，如果超时时间短于将某些内容添加到集群所需的时间，则设置此值可能会中断添加到集群。
`read_header_timeout`|`"30s"`| [https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。
`write_timeout`|`"0s"`| [https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。请注意，如果超时时间短于将某些内容添加到集群所需的时间，则设置此值可能会中断添加到集群。
`idle_timeout`|`"30s"`| [https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。
`extract_headers_extra`|`[]`|如果需要从 IPFS 守护进程中提取额外的标头并在被劫持的请求响应中使用，可以在此处添加它们。
`extract_headers_path`|`"/api/v0/version"`|提取标头时，会向 IPFS API 中的此路径发出请求。
`extract_headers_ttl`|`"5m"`|从中提取的标头extract_headers_path具有 TTL。它们将被记住并且仅在 TTL 之后刷新。

#### restapi	
这是提供 REST API 实现以与集群交互的组件

参数|默认|描述
---|---|---
`http_listen_multiaddress`|`"/ip4/127.0.0.1/tcp/9094"`| API HTTP 侦听端点。设置为空以禁用 HTTP 端点。
`ssl_cert_file`|`""`| x509 证书文件的路径。在 HTTP 端点上启用 SSL。除非是绝对路径，相对于配置文件夹。
`ssl_key_file`|`""`| SSL 私钥文件的路径。在 HTTP 端点上启用 SSL。除非是绝对路径，相对于配置文件夹。
`read_timeout`|`"0s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。请注意，如果超时时间短于将某些内容添加到集群所需的时间，则设置此值可能会中断添加到集群。
`read_header_timeout`|`"30s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。
`write_timeout`|`"0s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。请注意，如果超时时间短于将某些内容添加到集群所需的时间，则设置此值可能会中断添加到集群。
`idle_timeout`|`"30s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。
`libp2p_listen_multiaddress`|`""	`|备用 libp2p 主机的侦听多地址。见下文。
`id`|`""`|备用 libp2p 主机的 peer  ID（必须匹配 `private_key`）。见下文。
`private_key`|`""`|备用 libp2p 主机的私钥（必须匹配`id`）。见下文。
`basic_auth_credentials`|`null`|映射 `"username"` 到的对象 `"password"`。它为 API 启用基本身份验证。应该与启用 SSL 或 libp2p-endpoints 一起使用。
`headers`|`null`|API 端点应随对, ,请求 `key: [values]` 的每个响应返回的标头映射。即。不要在此处放置 CORS 标头，因为它们完全由以下选项处理。`GETPOSTDELETE"headers": {"header_name": [ "v1", "v2" ] }`
`http_log_file`|`""`|用于写入 API 日志文件的文件（Apache 组合格式）。否则，它们将被写入restapilog设施下的集群日志。
`cors_allowed_origins`|`["*"]`|CORS 配置： Access-Control-Allow-Origin的值。
`cors_allowed_methods`|`["GET"]`|CORS 配置： Access-Control-Allow-Methods的值。
`cors_allowed_headers`|`[]`|CORS 配置： Access-Control-Allow-Headers的值。
`cors_exposed_headers`|`["Content-Type", "X-Stream-Output", "X-Chunked-Output", "X-Content-Length"]`|	CORS 配置： Access-Control-Expose-Headers的值。
`cors_allow_credentials`|`true`|CORS 配置： Access-Control-Allow-Credentials的值。
`cors_max_age`|`"0s"`|CORS 配置： Access-Control-Max-Age 的值。

REST API 组件自动，可以将 HTTP API 作为 libp2p 服务公开在主 libp2p 集群主机上（监听端口9096）（这在 Raft 集群上默认发生）。将 HTTP API 公开为 libp2p 服务允许用户从 libp2p 提供的通道加密中受益。`id` 或者 API 支持通过提供和 `private_key` 来指定完全独立的 libp2p 主机`libp2p_listen_multiaddress`。使用单独的主机时，API 使用者无需知道集群机密。API客户端和 `ipfs-cluster-ctl`.

#### pinsvcapi
这是提供固定服务 API 实现以与集群交互的组件。它与 `restapi` 共享大部分代码，因此具有相同的选项	
参数|默认|描述
---|---|---
`http_listen_multiaddress`|`"/ip4/127.0.0.1/tcp/9094"`|固定 SVC API HTTP 侦听端点。设置为空以禁用 HTTP 端点。
`ssl_cert_file`|`""`|x509 证书文件的路径。在 HTTP 端点上启用 SSL。除非是绝对路径，相对于配置文件夹。
`ssl_key_file`|`""`|SSL 私钥文件的路径。在 HTTP 端点上启用 SSL。除非是绝对路径，相对于配置文件夹。
`read_timeout`|`"0s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。请注意，如果超时时间短于将某些内容添加到集群所需的时间，则设置此值可能会中断添加到集群。
`read_header_timeout`|`"30s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。
`write_timeout`|`"0s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。请注意，如果超时时间短于将某些内容添加到集群所需的时间，则设置此值可能会中断添加到集群。
`idle_timeout`|`"30s"`|[https://godoc.org/net/http#Server](https://godoc.org/net/http#Server)的参数。
`libp2p_listen_multiaddress`|`""`|备用 libp2p 主机的侦听多地址。请参阅 `restapi` 中的注释 。
`id`|`""`|备用 libp2p 主机的 peer  ID（必须匹配 `private_key`）。请参阅 `restapi` 中的注释 。
`private_key`|`""`|备用 libp2p 主机的私钥（必须匹配`id`）。请参阅 `restapi` 中的注释 。
`basic_auth_credentials`|`null`|映射 `"username"` 到的对象 `"password"`。它为 API 启用基本身份验证。应该与启用 SSL 或 libp2p-endpoints 一起使用。
`headers`|`null`|API 端点应随对, ,请求 `key: [values]` 的每个响应返回的标头映射。即。不要在此处放置 CORS 标头，因为它们完全由以下选项处理。`GETPOSTDELETE"headers": {"header_name": [ "v1", "v2" ] }`
`http_log_file`|`""`|用于写入 API 日志文件的文件（Apache 组合格式）。否则，它们将被写入 `restapilog` 设施下的集群日志。
`cors_allowed_origins`|`["*"]`|CORS 配置： Access-Control-Allow-Origin 的值。
`cors_allowed_methods`|`["GET"]`|CORS 配置： Access-Control-Allow-Methods 的值。
`cors_allowed_headers`|`[]	`|CORS 配置： Access-Control-Allow-Headers 的值。
`cors_exposed_headers`|`["Content-Type", "X-Stream-Output", "X-Chunked-Output", "X-Content-Length"]`|	CORS 配置： Access-Control-Expose-Headers 的值。
`cors_allow_credentials`|`true`|	CORS 配置： Access-Control-Allow-Credentials 的值。
`cors_max_age`|`"0s"`|CORS 配置： Access-Control-Max-Age 的值。

#### ipfs_connector 部分
`ipfs_connector` 部分包含 IPFS 连接器组件实现的配置，旨在为集群 peer 提供一种与 IPFS 守护进程交互的方式。

#### ipfshttp
这是默认且唯一的 IPFS 连接器实现。它为 IPFS 守护程序 API 和 IPFS HTTP 代理提供网关。

参数|默认|描述
---|---|---
`listen_multiaddress`|`"/ip4/127.0.0.1/tcp/9095"`|	IPFS 代理侦听多地址。
`node_multiaddress`|`"/ip4/127.0.0.1/tcp/5001"`|IPFS 守护程序 HTTP API 端点。这是 peer 用于固定内容的守护进程。
`connect_swarms_delay`|`"30s"`|启动时，集群 peer 将运行 `ipfs swarm connect`到其他 peer 的 IPFS 守护程序。这将设置启动后的延迟。
`ipfs_request_timeout`|`"5m0s"`|	为没有特定超时选项的请求指定对 IPFS 守护程序的一般请求的超时。
`repogc_timeout`|`"24h"`|指定 `/repo/gc` 操作超时。
`pin_timeout`|`"2m0s"`|指定 `pin/add` 从为被固定的项目收到的最后一个块开始的超时。因此，被缓慢固定的项目即使花费超过 24 小时也不会被取消。
`unpin_timeout`|`"3h0m0s"`|	指定对 `pin/rm` IPFS 守护程序的请求的超时时间。
`unpin_disable`|`false`|防止连接器取消固定任何东西（即使集群这样做）。
`informer_trigger_interval`|`0`|在此值指示的 pin 请求数之后强制广播所有 peer 指标。

#### pin_tracker 部分
`pin_tracker` 部分包含 Pin Tracker 组件实现的配置，旨在确保 IPFS 中的内容与 IPFS 集群决定的分配相匹配。

#### stateless
stateless 跟踪器实现了一个依赖于 ipfs 和共享状态的 pintracker，仅在内存中跟踪正在进行的操作。

参数|默认|描述
---|---|---
`max_pin_queue_size`|`1000000`|在我们直接出错之前，有多少个 pin 或 unpin 请求可以排队等待固定。将在下一个“状态同步”上尝试重新排队，如定义`state_sync_interval`
`concurrent_pins`|`10`|我们向 IPFS 发出多少并行 pin 或 unpin 请求。
`priority_pin_max_age`|`"24h0m0s"`|如果一个 pin 变得这么旧并且无法 pin，则在面对更新的 pin 请求时，重试将被优先考虑。
`priority_pin_max_retries`|`5`|如果 pin 已超过此 pinning 尝试次数，则在面对较新的 pin 请求时，重试将被取消优先级。

#### monitor 部分
`monitor` 部分包含 Peer Monitor 组件实现的配置，旨在向其他 peer 分发和收集监控信息（informer 指标、ping），并触发警报。

#### pubsubmon
`pubsubmon` 实现使用 libp2p 的 pubsub 收集和广播指标。这将为度量分布提供更有效和可扩展的方法。

参数|默认|描述
---|---|---
`check_interval`|`"15s"`|检查之间的间隔，以确保 peer 集中的任何 peer 都没有指标过期。如果检测到过期的指标，则会触发警报。这可能会触发重新固定项目。

#### informer 部分
`informer` 部分包含 Informers 的配置。Informers 获取用于将内容分配给不同 peer 的指标。
#### disk
Informer 会定期收集与 `disk` 磁盘相关的指标。

参数|默认|描述
---|---|---
`metric_ttl`|`"30s"`|此设置提供的指标的生存时间。这将以 TTL/2 间隔触发新的指标读数。
`metric_type`|`"freespace"`|`freespace` 或 `reposize`。监控器将报告 ipfs 守护程序存储库 ( `StorageMax-RepoSize`) 或 `RepoSize`.

#### tags
告密者根据 `tags` 用户提供的标签发布指标。这些“指标”仅用于通知其他 peer 与每个 peer 关联的标签。这些标签对于下面的平衡分配器很有用，因为它们可以是 `allocate_by` 选项的一部分。为每个定义的标签发布一个指标。

参数|默认|描述
---|---|---
`metric_ttl`|`"30s"`|监控提供的指标的生存时间。这将以 TTL/2 间隔触发新的指标读数。
`tags`|`{"group": "default"}`|	一个简单的` “tag_name:tag_value” `对象，用于指定与此 peer 关联的标签。

#### pinqueue
通知者收集 pintracker 的 pinning 队列  `pinqueue`  中的 pin 数。它可以是 `allocate_by` 平衡分配器选项的一部分，以降低对具有大队列的 peer 的固定优先级。`weight_bucket_size` 选项指定应将排队项目的实际数量除以多少。即如果两个 peer 有 53 和 58 个项目排队并且 `weight_bucket_size` 是 1，那么有 58 个项目排队的 peer 将被分配器优先于有 53 个项目的 peer 。但是，如果 `weight_bucket_size` 是 10，则两个 peer 将具有相同的权重 (5)，因此优先级将取决于其他指标（即空闲空间）。

参数|默认|描述
---|---|---
`metric_ttl`|`"30s"`|此线人提供的指标的生存时间。这将以 TTL/2 间隔触发新的指标读数。
`weight_bucket_size`|`100000`|pinqueue 在比较指标时，分配器将使用实际 pin 队列大小除以该值

#### numpin
通知者使用固定的 `numpin` 总数作为度量，每隔一段时间收集一次

参数|默认|描述
---|---|---
`metric_ttl`|`"30s"`| 通知者提供的指标的生存时间。这将以 TTL/2 间隔触发新的指标读数。

####  allocator 部分
`allocator` 用于配置分配器。分配器控制如何将固定分配给集群中的 peer 。
#### balanced
分配器通过使用从 peer 接收到的不同度量来选择将 `balanced` 固定分配给哪些 peer （当复制因子大于 0 时）进行分组，并在这些组中创建每个固定的平衡分布。

例如： `Allocate by ["tag:group", "freespace"]`

- 表示分配器将根据它们首先拥有的标记度量“组”的值来划分所有 peer 。
- 然后它将按每个组中的“自由空间”度量值对点进行排序。在决定应将 pin 分配给哪些 peer 时，它将从具有最多总可用空间的组中选择具有最多可用空间的 peer 。
- 然后它将从第二组（如果存在）中强制选择具有最多可用空间的 peer ，因为它试图平衡现有组之间的分配。

这可以扩展到子组：假设一个集群由 6 个 peer 组成，每个区域 2 个（每个“区域”标签），每个可用区一个（每个“az”标签），配置分配器 `["tag:region", "tag:az", "freespace"]` 将确保一个 pin 复制因子 = 3 位于 3 个不同区域中，位于具有最多可用聚合空间的可用区域中，并且位于具有最多可用空间的区域中的 peer 中。

参数|默认|描述
---|---|---
`allocate_by`|`["tag:group", "freespace"]`|指定每个 pin 应分配的 peer 指标。

#### observations部分
`observations` 部分包含应用程序分布式跟踪和指标收集的配置。
#### metrics
`metrics` 组件配置 OpenCensus 指标端点，以便 Prometheus 抓取指标。

参数|默认|描述
---|---|---
`enable_stats`|`false`|是否应启用指标。
`prometheus_endpoint`|`/ip4/127.0.0.1/tcp/8888`|将收集的指标发布到端点以供 Prometheus 抓取。
`reporting_interval`|`"2s"`|多久报告一次收集的指标。

#### tracing
tracing 组件配置 Jaeger 跟踪客户端以供 OpenCensus 使用。

参数|默认|描述
---|---|---
`enable_tracing`|`false`|是否应启用跟踪。
`jaeger_agent_endpoint`|`/ip4/0.0.0.0/udp/6831`|	将跟踪发送到的多地址。
`sampling_prob`|`0.3`|多久进行一次采样跟踪。
`service_name`|`cluster-daemon`|将与集群跟踪关联的服务名称

#### datastore部分
`datastore` 部分包含存储后端的配置。它可以包含一个 `badger` 或一个 `leveldb` 部分。

#### badger
`badger` 组件配置了 BadgerDB 后端，该后端用于在启用 CRDT 共识时存储东西。

参数|默认|描述
---|---|---
`gc_discard_ratio	`|`0.2`|请参阅 [RunValueLogGC](https://github.com/dgraph-io/badger/blob/725913b83470967abd97e850331d7ebe4926fa79/db.go#L1290-L1316)文档。
`gc_interval`|`"15m0s"`|运行 Badger GC 周期的频率。一个循环由几轮组成，重复直到没有空间可以释放。将此设置为`"0s"`禁用 GC 循环。
`gc_sleep`|`"10s"	`|同一个 GC 周期中的 GC 轮次之间等待多长时间。将此设置为`"0s"`会导致运行单轮。
`badger_options`|`{...}`|一些 [BadgerDB 特定选项](https://godoc.org/github.com/dgraph-io/badger#Options)初始化为优化默认值（根据 IPFS 建议，见下文）。设置 `table_loading_mode` 和 `value_log_loading_mode` to`0`应该有助于内存受限的平台（Raspberry Pis 等，具有 <1GB RAM）

默认情况下在默认 badger 选项之上执行的调整可以[在 badger 配置初始化代码](https://github.com/ipfs-cluster/ipfs-cluster/blob/master/datastore/badger/config.go#L38-L49)中看到。

#### leveldb
`leveldb` 组件配置了 LevelDB 后端，该后端用于在启用 CRDT 共识时存储东西。

参数|默认|描述
---|---|---
`leveldb_options`|`{...}`|一些[LevelDB 特定的选项](https://pkg.go.dev/github.com/syndtr/goleveldb@v1.0.0/leveldb/opt#Options)初始化为它们的默认值。

## REST API 参考
IPFS 集群 peer 包括一个 API 组件，该组件提供对 peer 功能的基于 HTTP 的访问。API 试图在形式和行为上都符合 REST。它默认启用，但可以通过从`service.json` 配置文件中删除其部分来禁用它。

我们不维护临时 API 文档，因为它很容易过时，或者最坏的情况是不准确或有缺陷。`ipfs-cluster-ctl` 相反，我们提供了一种简单的方法来使用该命令来查找如何执行您需要执行的操作。

运行 `ipfs-cluster-ctl --enc=json --debug <command>` 将打印有关该命令的端点、查询选项、请求正文和原始响应的信息。在测试集群上使用它！

`ipfs-cluster-ctl` 是 REST API 端点的 HTTP API 客户端，具有完整的功能奇偶性，始终与同一版本的集群 peer 提供的 HTTP API 一起使用。`ipfs-cluster-ctl` REST API 支持任何可以做的事情。命令标志通常控制不同的请求选项。

作为附加资源：

- [Go API Client](https://pkg.go.dev/github.com/ipfs-cluster/ipfs-cluster/api/rest/client?tab=doc#Client) 支持和记录所有可用的 API 端点及其参数和对象格式。
- API 源代码在 [这里](https://github.com/ipfs-cluster/ipfs-cluster/blob/master/api/rest/restapi.go)（ `routes` 方法是一个很好的起点）。
- 有两个 Javascript 客户端库：
	- [js-cluster-client](https://github.com/ipfs-cluster/js-cluster-client)（旧）
	- 和 [NFT.storage](https://github.com/nftstorage/ipfs-cluster) 的集群客户端（新）。
- 端点的请求体 `/add` 有点特殊，但它的工作方式与 IPFS 一样。请参阅下面的部分。

以上内容应该足以了解现有端点、它们的方法和当前支持的选项。

- `/add/` 端点_
	- `/add` 端点可用于通过集群 API 将内容上传到 IPFS。
	- 集群 peer 负责构建或提取 IPLD DAG，将其逐块发送到应该固定它的集群 peer ，然后 `block/put` 对它们连接的 IPFS 守护程序执行调用。
	- 在该过程结束时，会发生 Cluster-Pin，并且固定操作随之到达 IPFS 守护程序，该守护程序应该已经具有所有需要的块。

这里有一些考虑因素

- 通过 IPFS 集群添加内容比较慢，因为它会在添加的同时复制到所有固定位置。
- 查询参数将 `local=true` 指示接收请求的集群 peer 在本地摄取所有块。这使得添加方式更快，并以更慢的集群固定为代价：当指示固定节点时，固定节点将不得不使用 IPFS 来接收块。
- 添加时应禁用 IPFS 垃圾收集。因为块是单独的块/放置，如果在添加操作期间发生 GC 操作，并且在块被固定之前，它们将被删除。

目前 IPFS 集群支持添加两种 DAG 格式（ `?format=` 查询参数）：

- 默认情况下，它使用 `unixfs` 格式。

	在这种模式下，请求正文应该是多部分的，就像 `/api/v0/add` [文档](https://docs.ipfs.io/reference/http/api/#api-v0-add)中描述的那样。端点支持与 IPFS 相同的 `/add` 可选参数，并在添加文件时生成与 go-ipfs 完全相同的 DAG。在 UnixFS 中，请求中上传的文件被分块，并构建了一个 DAG，以复制所需的文件夹布局。这是由集群 peer 完成的。
- 或者 `/add` 端点也接受具有 `?format=car` 格式的 CAR 文件。

	在这种情况下，CAR 文件已经包含了需要添加到 IPFS 的块，Cluster 不会做任何进一步的处理（类似于 `ipfs dag import`）。目前，`/add` 端点将只处理一个 CAR 文件，并且该文件必须只有一个根（将被固定的那个）。CAR 文件允许通过集群 API 添加任意 IPLD-DAG。

使用 `/add` 前面带有 Nginx 的端点作为反向代理可能会导致问题。确保添加 `?stream-channels=false` 到每个添加请求以避免它们。

这些问题表现为日志中的“读取上游时 peer 重置连接”错误。它们是由 HTTP 请求/响应周期上的读取后写入引起的：Nginx 拒绝任何已开始发送响应正文的应用程序从请求正文中进一步读取（[请参阅错误报告](https://trac.nginx.org/nginx/ticket/1293)）。IPFS 和 IPFS 集群在添加文件时发送对象更新，因此会触发这种情况，否则根据 HTTP 规范这是合法的。该问题取决于 Nginx 内部缓冲，并且可能非常零星地出现或根本不出现，但它确实存在。

### 端点列表
作为最后的提示，此表提供了可用方法的快速摘要。

方法|端点|描述
---|---|---
`GET`|`/id`|集群 peer 信息
`GET`|`/version`|集群版本
`GET`|`/peers	`|集群 peer 。流式传输端点。
`DELETE`|`/peers/{peerID}`|	删除 peer 
`POST`|`/add`|向集群添加内容。流式传输端点。见上面的注释
`GET`|`/allocations`|固定列表及其分配（固定组）。流式传输端点。
`GET`|`/allocations/{cid}`|显示单个固定及其分配（来自固定组）
`GET`|`/pins`|所有被跟踪 CID 的本地状态。流式传输端点。
`GET`|`/pins/{cid}`|	单个 CID 的本地状态
`POST`|`/pins/{cid}`|固定 CID
`POST`|`/pins/{ipfs\|ipns\|ipld}/<path>	`|使用 IPFS 路径固定
`DELETE`|`/pins/{cid}`|取消固定 CID
`DELETE`|`/pins/{ipfs\|ipns\|ipld}/<path>`|使用 IPFS 路径取消固定
`POST`|`/pins/{cid}/recover`|恢复 CID
`POST`|`/pins/recover	`|恢复接收集群 peer 中的所有固定
`GET`|`/monitor/metrics`|获取 peer 已知的指标类型列表
`GET`|`/monitor/metrics/{metric}`|获取此 peer 看到的当前指标列表
`GET`|`/health/alerts`|显示警报列表（指标过期事件）
`GET`|`/health/graph`|获取连接图
`GET`|`/health/alerts`|获取连接图
`POST`|`/ipfs/gc`|在 IPFS 节点中执行 GC

## IPFS 代理
IPFS 代理是一个端点，它以下列方式呈现 IPFS HTTP API：

- 部分请求被拦截并触发集群操作
- 所有未拦截的请求都转发到附加到集群 peer 的 IPFS 守护进程

默认情况下启用此端点，默认情况下  `/ip4/127.0.0.1/tcp/9095`  侦听并由连接器组件提供。`ipfshttp` 可以通过从 `service.json` 配置文件中删除其部分来禁用它。

被拦截的请求如下：

- `/add` 代理将内容添加到本地 ipfs 守护进程，并将生成的 `hash[es]` 固定在集群中。
- `/pin/add` 代理将给定的 CID 固定在集群中。
- `/pin/update` 代理将给定的 pin 更新为集群中的新 pin。
- `/pin/rm` 代理从集群中取消固定给定的 CID。
- `/pin/ls` 代理列出集群中的固定项目。
- `/repo/stat` 代理以 `/repo/stat` 所有连接的 IPFS 守护程序的聚合响应。
- `/repo/gc` 代理对所有 IPFS 守护进程执行垃圾收集，并以收集的 CID 进行响应。

来自代理的响应模仿 IPFS 守护进程响应，因此允许将该端点放入之前使用 IPFS API 的地方。例如，您可以 `go-ipfs` 按如下方式使用 CLI：

- `ipfs --api /ip4/127.0.0.1/tcp/9095 pin add <cid>`
- `ipfs --api /ip4/127.0.0.1/tcp/9095 add myfile.txt`
- `ipfs --api /ip4/127.0.0.1/tcp/9095 pin rm <cid>`
- `ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls`

IPFS 代理端点可以与IPFS 伴侣扩展一起使用。

响应将来自集群，而不是来自 `go-ipfs`. 截获的端点旨在模仿 IPFS 的格式、标头和响应代码。如果您在 IPFS 中配置了自定义标头，则需要将它们的名称添加到`ipfsproxy.extract_headers_extra` 配置选项中。

## ipfs-cluster-ctl
命令行应用程序是 IPFS 集群的 `ipfs-cluster-ctl` 用户友好的 REST API 客户端。它允许执行集群 peer 支持的所有操作：

- [固定](https://ipfscluster.io/documentation/guides/pinning#pinning-cids)
- [取消固定](https://ipfscluster.io/documentation/guides/pinning#pinning-cids)
- [添加](https://ipfscluster.io/documentation/guides/pinning#adding-files)
- [列出 pinset 中的项目](https://ipfscluster.io/documentation/guides/pinning#pin-ls-vs-status)
- [检查固定状态](https://ipfscluster.io/documentation/guides/pinning#pin-ls-vs-status)
- [列出集群 peer ](https://ipfscluster.io/documentation/guides/peerset#listing-peers)
- [删除 peer ](https://ipfscluster.io/documentation/guides/peerset#removing-peers)

### 用法
可以通过运行获取使用信息：

	$ ipfs-cluster-ctl --help
您还可以使用 `ipfs-cluster-ctl help [cmd]. ( --host)` 可用于与任何远程集群 peer 通信（`localhost` 默认使用）。总之，它的工作原理如下：

	$ ipfs-cluster-ctl id                                                       # 显示集群 peer 和ipfs守护进程信息
	$ ipfs-cluster-ctl peers ls                                                 # 集群 peer 列表 
	$ ipfs-cluster-ctl peers rm <peerid>                                        # 删除一个集群中的 peer
	$ ipfs-cluster-ctl add myfile.txt http://domain.com/file.txt                # 向集群添加内容
	$ ipfs-cluster-ctl pin add Qma4Lid2T1F68E3Xa3CpE6vVJDLwxXLD8RfiB9g1Tmqp58   # 将CID添加固定到集群中
	$ ipfs-cluster-ctl pin rm Qma4Lid2T1F68E3Xa3CpE6vVJDLwxXLD8RfiB9g1Tmqp58    #将CID取消固定到集群中
	$ ipfs-cluster-ctl pin ls [CID]                                             #列出跟踪的cid(共享状态)
	$ ipfs-cluster-ctl status [CID]                                             # 列出跟踪的cid的当前状态(本地状态)
	$ ipfs-cluster-ctl sync Qma4Lid2T1F68E3Xa3CpE6vVJDLwxXLD8RfiB9g1Tmqp58      #根据 IPFS 守护进程报告的状态重新同步看到的状态 	$ ipfs-cluster-ctl recover Qma4Lid2T1F68E3Xa3CpE6vVJDLwxXLD8RfiB9g1Tmqp58   # 尝试重新固定/取消固定处于错误状态的 cid
### 验证
- IPFS 集群 API 可以配置基本身份验证支持。

	`ipfs-cluster-ctl --basic-auth <username:password> `将使用给定的凭据来执行请求。

	请注意，除非 `--force-http` 通过，否则 `basic-auth` 仅在 HTTPS 请求或使用 libp2p API 端点（使用加密通道）时支持使用。
- 使用 libp2p API 端点

	因为0.3.5，IPFS 集群为 HTTP API 提供了一个 libp2p 端点，该端点提供通道安全性而无需配置 SSL 证书，方法是重新使用 peer 的 libp2p 主机或通过在 API 配置中使用给定参数设置一个新主机。

	为了 `ipfs-cluster-ctl` 使用 libp2p 端点，请提供 `--host` 如下标志：
	
		ipfs-cluster-ctl --host /ip4/<ip>/ipfs/<peerID> ...
	或者

		ipfs-cluster-ctl --host /dnsaddr/mydomain.com ..._dnsaddr TXT dnsaddr=peer_multiaddress（在您的 dns 中设置一个字段）。

	如果您正在联系的 libp2p  peer 使用集群密钥（私有网络密钥），您还需要提供 `--secret <32 byte-hex-encoded key>` 给命令。

	我们建议您 `ipfs-cluster-ctl` 将 shell 中的命令别名为更短的名称并使用正确的全局选项。
- 退出代码

	`ipfs-cluster-ctl` 将退出：
	
	- 0：请求/操作成功。输出包含响应数据。
	- 1：参数错误、网络错误或任何其他阻止应用程序执行请求并从 IPFS 集群 API 获取响应的错误。在这种情况下，输出包含错误的内容和 HTTP 代码0。
	- 2: IPFS 集群 peer 错误。请求已正确执行，但响应为错误（HTTP 状态 4xx 或 5xx）。在这种情况下，输出包含错误的内容和与之关联的 HTTP 代码。
- 调试

	`ipfs-cluster-ctl` 接受一个 `--debug` 允许检查请求路径和原始响应主体的标志。
- 下载

	要下载 `ipfs-cluster-ctl` 检查[下载页面](https://ipfscluster.io/download)。

##  ipfs-cluster-follow
`ipfs-cluster-follow` 命令行应用程序是一种用户友好的方式，可以运行追随者节点以加入协作 IPFS 集群。您可以在相应部分获得有关协作集群的[更多信息](https://ipfscluster.io/documentation/collaborative)。

`ipfs-cluster-follow` 运行优化的集群 peer 以与协作集群一起使用。它专注于运行追随者 peer 的用户的简单性和安全性，消除了运行 peer 的大部分配置麻烦。
### 配置
`ipfs-cluster-follow`  通常使用通过本地 IPFS 网关分发的配置作为模板。

在这种情况下，`service.json` 每个已配置集群的文件都包含一个 `source` 指向 URL 的键，该键在启动 peer 时被读取。

此文件可以用自定义 `service.json` 文件替换。或可以使用环境变量覆盖每个配置值，如[配置参考](https://ipfscluster.io/documentation/reference/configuration#using-environment-variables-to-overwrite-configuration-values)中所述。如果不是默认值 ( ) ，`IPFS_GATEWAY` 环境变量 127.0.0.1:8080 可用于设置网关位置。

如果您需要在 TCP 端口而不是默认的 unix 套接字上公开 HTTP API，请进行 `CLUSTER_RESTAPI_HTTPLISTENMULTIADDRESS` 相应设置。

与 `ipfs-cluster-ctl_ipfs-cluster-follow`

 `ipfs-cluster-follow`   默认情况下，使用 unix 套接字公开 API 端点，而不是侦听本地 TCP 端口。

如果您想使用 `ipfs-cluster-ctl` 与 peer 交谈，可以运行：

	ipfs-cluster-ctl --host /unix//<home_path>/.ipfs-cluster-follow/<clusterName>/api-socket ...
### 用法
可以通过运行获取使用信息：

	$ ipfs-cluster-follow --help
### 寻找要加入的协作集群
访问 [collab.ipfscluster.io](https://collab.ipfscluster.io/) 以获取您可以加入的协作集群的列表以及更多说明。

	ipfs-cluster-ctl --host /ip4/<ip>/ipfs/<peerID> ..`.
或者

	ipfs-cluster-ctl --host /dnsaddr/mydomain.com ..._dnsaddr TXT dnsaddr=peer_multiaddress（在您的 dns 中设置一个字段）。
如果您正在链接的 libp2p  peer 使用集群密钥（私有网络密钥），您还需要提供 `--secret <32 byte-hex-encoded key>` 给命令。

我们建议您 `ipfs-cluster-ctl` 将 shell 中的命令别名为更短的名称并使用正确的全局选项。

### 下载
要下载 `ipfs-cluster-follow` 检查[下载页面](https://ipfscluster.io/download)。

## ipfs-cluster-service
`ipfs-cluster-service` 是一个运行完整集群 peer 的命令行应用程序：

- `ipfs-cluster-service init` [初始化配置和身份](https://ipfscluster.io/documentation/deployment/setup)。
- `ipfs-cluster-service daemon` [启动一个集群 peer ](https://ipfscluster.io/documentation/deployment/bootstrap)。
- `ipfs-cluster-service state` [允许导出、导入和清理持久状态](https://ipfscluster.io/documentation/guides/backups)。

通过运行或 `ipfs-cluster-service` 提供自己的帮助。

	ipfs-cluster-service --help
	
	ipfs-cluster-service <command> --help

### 调试
`ipfs-cluster-service` 提供两个调试选项：

- `--debug`

	从 `ipfs-cluster,go-libp2p-raft` 和 `go-libp2p-rpc` 层启用调试日志记录。这将是一个非常冗长的日志输出，但同时它也是最多信息的。
- `--loglevel`

	将日志级别 ( `[error, warning, info, debug]`) 设置为 `ipfs-cluster`only，从而可以大致了解集群正在做什么。默认日志级别为 info.

默认情况下，日志是彩色的。要禁用日志颜色，请将 `IPFS_LOGGING_FMT` 环境变量设置为 `nocolor`.

### 下载
要下载 `ipfs-cluster-service` 检查[下载页面](https://ipfscluster.io/download)。

## 替换
 peer 节
 peer 
 peer 
 peer 
固定

