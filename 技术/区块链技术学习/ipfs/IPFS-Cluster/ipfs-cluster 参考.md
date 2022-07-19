# ipfs-cluster 参考
## 配置参考
默认情况下，可以在 `~/.ipfs-cluster` 文件夹中找到所有 IPFS 集群配置和持久数据。有关此文件夹中持久数据的更多信息，请参阅[数据、备份和恢复部分](https://ipfscluster.io/documentation/guides/backups)。

	ipfs-cluster-service -c <path> 设置配置文件夹的位置。这也由 IPFS_CLUSTER_PATH 环境变量控制。
 `ipfs-cluster-service` 程序使用两个主要的配置文件：

- `service.json`

	包含集群对等配置，通常在所有集群对等中相同。
- `identity.json`

	包含每个对等方使用的唯一标识。`identity.json` 文件是在 `ipfs-cluster-service init` . 它包括一个 base64 编码的私钥和与之关联的公共对等 ID。此 peer  ID 标识集群中的 peer 。你可以在[这里](https://ipfscluster.io/0.14.2_identity.json)看到一个例子。

	重新运行时不会覆盖此文件 `ipfs-cluster-service -f init` 。如果您想生成一个新的，您需要先将其删除。

	可以使用 `CLUSTER_ID` 和 `CLUSTER_PRIVATEKEY` 环境值覆盖身份字段。

### 手动身份生成
在为多个 peer 自动部署或创建配置时，事先手动生成 peer  ID 和私钥可能会很方便。

您可以使用以下配置以配置预期的格式获取有效的对等 ID 及其关联的私钥 `ipfs-key` ：

	ipfs-key -type ed25519 | base64 -w 0
- `service.json`

	`service.json` 文件包含集群 peer 及其不同组件的所有可配置选项。配置文件分为多个部分。每个部分代表一个组件。该部分中的每个项目都代表该组件的一个实现，并包含特定的选项。 `ipfs-cluster-service init`运行时会创建一个具有合理值的默认 `service.json`  文件。

	重要提示：`cluster` 配置部分存储一个 32 字节的十六进制编码 `secret`，用于保护所有集群对等方之间的通信。`secret` 必须由所有集群对等方共享。使用空密钥具有安全隐患（请参阅[安全](https://ipfscluster.io/documentation/guides/security)部分）。

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

`leave_on_shutdown` 选项允许 peer 在完全关闭时将自己从 peer 集中删除。使用 `raft` 时最相关。这意味着，对于任何后续启动，都需要[引导](https://ipfscluster.io/documentation/deployment/bootstrap/#bootstrapping-the-cluster-in-raft-mode)对等方才能重新加入集群。

参数|默认|描述
---|---|---
`peername`|`<hostname>`|此 peer 的人名。
`secret`|`<randomly generated>`|集群密钥（在所有 peer 中必须相同）。
`leave_on_shutdown`|`false`| peer 将在关闭时将自己从集群 peer 集中删除。
`listen_multiaddress`|`["/ip4/0.0.0.0/tcp/9096", "/ip4/0.0.0.0/udp/9096/quic"]`|peers Cluster-RPC 监听端点。
`connection_manager {`|方法上限|		连接管理器配置对象。
`high_water`|`400`|	此对等方将拥有的最大连接数。如果是，则连接将关闭，直到low_water达到该值。
`low_water`|`100`|libp2p 主机将尝试与其他对等方保持至少这么多的连接。
`grace_period`|`"2m0s"`|	至少在此期间不会丢弃新连接。
`}`|方法下限|
`dial_peer_timeout`|`"3s"`|	在放弃之前拨打集群对等方时要等待多长时间。
`state_sync_interval`|`"10m0s"`|	的自动触发之间的间隔StateSync。
`pin_recover_interval`|`"1h0m0s"`|	的自动触发之间的间隔RecoverAllLocal。这将自动重试失败的 pin 和 unpin 操作。
`replication_factor_min`|`-1`|	指定应固定项目的默认最小 peer 数。-1 == 全部。
`replication_factor_max`|`-1`|	指定应该固定项目的默认最大 peer 数。-1 == 全部。
`monitor_ping_interva`l	|`"15s"`|	发送时间间隔ping（用于检测停机时间）。
`peer_watch_interval`|`"5s"`|	检查当前集群 peerset 并检测此 peer 是否已从集群中删除（并关闭）的时间间隔。
`mdns_interval`|`"10s"`|	将其设置为"0"禁用 mDNS。设置为较大的值会启用 mDNS，但不再控制任何内容。
`enable_relay_hop`|`true`|	让集群 peer 充当无法直接访问的其他 peer 的中继。
`pin_only_on_trusted_peers`|`false`|集群 peer 只会将引脚分配给受信任的 peer （如配置）
`disable_repinning`|`true`|不要自动重新固定分配给变得不健康（关闭）的对等方的所有项目。
`follower_mode`|`false`|追随者模式下的对等方在尝试执行固定等操作时会提供有用的错误消息。
`peer_addresses`|`[]`| peer 在启动时连接的完整 peer 多地址（类似于手动添加到peerstore文件的条目。
`pin_only_on_trusted_peers`|`false`|	将给予引脚的可能分配限制为trusted_peers列表中的那些

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
`cluster_name`|`"ipfs-cluster"`|任意名称。它成为集群中所有对等节点订阅的发布订阅主题，因此它必须对所有节点都相同。
`trusted_peers`|`[]`|默认的受信任对等点集。有关详细信息，请参阅CRDT 模式下的信任。可以设置[ "*" ]为信任所有对等方。
`batching {`||将 pin 提交到 CRDT 层时的批处理设置。两者max_batch_size和都max_batch_age需要大于 0 才能启用批处理。
    `max_batch_size`|`0`|在提交之前要包含在批处理中的最大固定/取消固定操作数。
    `max_batch_age`|`"0s"`|未提交的批处理在提交之前等待的最长时间。
    `max_queue_size`|`1000`|等待包含在批处理中的最大 pin/unpin 操作数。
}||		
`peerset_metric`|`"ping"`|用于确定当前 pinset 的监视器指标的名称。
`rebroadcast_interval`|`"1m0s"`|在没有看到其他 pubsub 消息时重新发布当前头的频率。减少这将允许新的对等节点更快地了解当前状态。
`repair_interval`|`"1h0m0s"`|检查 crdt-datastore 是否被标记为脏的频率，并在这种情况下触发 DAG 的重新处理。

