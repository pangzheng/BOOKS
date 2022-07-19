# IPFS 集群指南
以下部分包含有关与 IPFS 集群使用相关的多个主题的深入指南：
## 添加和固定(pin)
集群使用主要包括添加和删除 pin。这通常使用 `ipfs-cluster-ctl` 实用程序或与集群 API 之一进行通信来执行。

您可以使用 `ipfs-cluster-ctl --help` 和 `ipfs-cluster-ctl <command> --help` 获取所有 `ipfs-cluster-ctl `命令的帮助和使用信息

使用大量固定时，重要的是要密切注意固定组的状态，是否每个固定都正确固定并分配。本节深入解释了固定的工作原理以及 cluster peer 可以执行的不同操作来简化和维护 cluster pinset。

为清楚起见，我们使用 `ipfs-cluster-ctl` 命令，但每个命令都使用来自集群对等方的 `HTTP REST API` 端点，因此所有命令都可以直接针对 API 执行。

### 添加文件
	ipfs-cluster-ctl add myfile.txt
该命令与该 `ipfs-cluster-ctl add` 命令非常相似，`ipfs add` 并且共享大多数相同的选项（例如那些定义分块、DAG 类型或要使用的 CID 版本的选项）。

但是，如果该 `ipfs add` 命令仅添加到本地 IPFS 守护程序，该 `ipfs-cluster-ctl add` 命令将同时添加到多个集群 peer 。它添加多少取决于您设置为命令标志固定的复制因子或配置文件中的默认值。

这意味着当添加过程完成时，您的文件将被完全添加到几个 IPFS 守护进程（不一定是本地守护进程）。例如：

	$ ipfs-cluster-ctl add pinning.md
	added QmarNBnreCx4YtT4ETXxQ4dn2xQpcTGd2PaVM4b2UuyGku pinning.md
	
	$ ipfs-cluster-ctl pin ls QmarNBnreCx4YtT4ETXxQ4dn2xQpcTGd2PaVM4b2UuyGku # check pin data
	QmarNBnreCx4YtT4ETXxQ4dn2xQpcTGd2PaVM4b2UuyGku |  | PIN | Repl. Factor: -1 | Allocations: [everywhere] | Recursive
	
	$ ipfs-cluster-ctl status QmarNBnreCx4YtT4ETXxQ4dn2xQpcTGd2PaVM4b2UuyGku # request status from every peer
	QmarNBnreCx4YtT4ETXxQ4dn2xQpcTGd2PaVM4b2UuyGku :
	    > cluster0        : PINNED | 2019-07-26T12:25:18.23191214+02:00
	    > cluster1        : PINNED | 2019-07-26T10:25:18.234842017Z
	    > cluster2        : PINNED | 2019-07-26T10:25:18.212836746Z
	    > cluster3        : PINNED | 2019-07-26T10:25:18.238415569Z
	    > cluserr4        : PINNED | 2019-07-26T10:25:24.508614677Z
以这种方式添加的过程比添加到本地 IPFS 守护进程要慢，因为所有块都并行发送到它们的远程位置。作为替代方案，`--local` 可以将标志提供给 `add` 命令。在这种情况下，内容将被添加到接收请求的对等方的本地 IPFS 守护进程中，然后正常固定。这将使 `add` 调用花费更少的时间，但是当它们返回时内容还不会被完全复制。

添加 `--local` 和不添加其他选项将始终在内容分配中包括本地 peer （无论可用空间等）。如果需要可以使用该 `--allocations` 标志手动提供不同的分配来解决此问题。

该命令中 IPFS 不可用的另一个功能 `add` 是可以导入 CAR 文件（类似于 ipfs dag import）。为了导入 CAR 文件，您可以执行 `ipfs-cluster-ctl add --format car myfile.car`. CAR 文件应该有一个根，即导入后固定的 CID。IPFS 集群不执行检查以验证 CAR 文件是否完整并包含所有块等。

### 固定 CID
	ipfs-cluster-ctl pin add <cid/ipfs-path>
在许多情况下，您知道要添加到集群中的 IPFS 网络中的哪些内容。`ipfs-cluster-ctl pin add` 操作类似于 `ipfs pin add` ，但允许设置特定于集群的标志，例如复制因子或与 pin 关联的名称。例如：

	$ ipfs-cluster-ctl pin add --name cluster-website --replication 3 /ipns/ipfscluster.io
	QmXvQLhK2heNz65fWRabTfbzXwYfaBgEBuTdUJNzp69Xjx :
	    > cluster2             : PINNED | 2021-12-07T15:24:58Z | Attempts: 0 | Priority: false
	    > cluster3             : PINNING | 2021-12-07T15:24:58Z | Attempts: 0 | Priority: true
	    > cluster0             : PINNING | 2021-12-07T15:24:58Z | Attempts: 0 | Priority: true
	    > Qmabc123             : REMOTE | 2021-12-07T16:25:07.48933158+01:00 | Attempts: 0 | Priority: false
	    > Qmabc456             : REMOTE | 2021-12-07T16:25:07.4893358+01:00 | Attempts: 0 | Priority: false

	$ ipfs-cluster-ctl pin ls QmXvQLhK2heNz65fWRabTfbzXwYfaBgEBuTdUJNzp69Xjx
	QmXvQLhK2heNz65fWRabTfbzXwYfaBgEBuTdUJNzp69Xjx | cluster-website | PIN | Repl. Factor: 3--3 | Allocations: [12D3KooWGbmjg3MDUYFosLNPbE1jKkv5fzKHD7wyGDa1P95iKMjF QmYY1ggjoew5eFrvkenTR3F4uWqtkBkmgfJk8g9Qqcwy51] | Recursive | Metadata: no | Exp: ∞ | Added: 2021-12-07 16:24:58

如我们所见，固定开始固定在两个位置（复制 = 3）。当我们检查固定对象时，我们会看到它被分配到的对等 ID，并且它被 `cluster-website` 调用。我们还查看它是否附加了任何元数据、到期日期和上次添加的日期。

可以随时使用 `ipfs-cluster-ctl pin rm` 删除固定。

### 复制因子
提交给 IPFS 集群的每个 pin 都带有两个复制选项：

	replication_factor_min（ `--replication-min` 标志在 `ipfs-cluster-ctl`）
	replication_factor_max（`--replication-max` 标志在 `ipfs-cluster-ctl`）
集群配置设置未设置该选项时应用的默认值。

- `replication_factor_min`

	值指定 pin 在集群中应具有的最小副本数。如果启用了自动重新固定并且集群检测到应该固定项目的 peer 不可用，并且该项目副本不足（固定它的 peer 数量低于`replication_factor_min`），它将重新分配该项目到新的 peer 。当没有足够的 peer 来固定某个东西时，固定将直接失败 `replication_factor_min`。
- `replication_factor_max`

	值指示应将多少 peer 分配给该固定。在 pin 提交时，集群将尝试分配那么多 peer ，但如果找不到这么多点，只要找到超过 `replication_factor_min`. 项目的重新固定将尝试增加对 `replication_factor_max` 的分配，但是项目的自动重新固定在启用时不会影响位于两个阈值之间的引脚。

建议在设置为 `disable_repinning` 时使用具有一定余量（通常为 2-3 或 3-5）的阈值false。在这种情况下，如果没有余地，集群 peer 宕机几秒钟可能会触发重新固定并导致集群不平衡，即使 peer 稍后恢复正常并且仍然​​保存内容（此时它将被取消固定，因为它是不再分配给它）。

### 固定过程
集群固定和取消固定是集群操作的核心，涉及多个内部组件，但主要分为两个阶段：

- Cluster-pinning 阶段

	pin 被集群正确地持久化并广播给所有其他 peer 。
	
	当集群固定阶段完成时，我们认为 `pin add` 操作已经成功。这意味着该 pin 已被 Cluster 摄取，并且正在告诉 IPFS 固定内容。如果 IPFS 无法固定内容，Cluster 会知道、报告并尝试处理这种情况。cluster-pinning 阶段相对较快但 ipfs-pinning 阶段可能需要更长的时间，具体取决于被固定的事物的数量和所涉及的大小。因此，一旦集群固定阶段完成，第二阶段就会异步发生。

	该过程可概括如下：

	- 一个 pin 请求到达，包括某些选项。
	- 分配过程（见下文）根据复制因素和指标选择应该固定内容的对等方。
	- 这些和其他事情导致了一个 pin 对象，该对象被提交并广播给每个人（如何取决于共识组件）。

	`分配过程` 负责为 pin（集群对等 id 列表）生成最终分配，这些分配附加到 pin 对象。它努力选择最好的地方来固定东西。该过程需要几个输入：

	- pin 最小和最大复制因子和用户集分配。
	- 集群中所有对等方广播的指标。
	- 分配器的配置 (`allocate_by`)。
	- 如果 pin 已存在，则为现有分配。

	通常，该过程包括构建一个优先级列表，其中包含哪些对等方应固定 CID 并从  `replication_factor_max` 中选择。此列表中的顺序取决于“分配器”组件及其配置使用的指标。为了将集群中的对等节点视为分配，所有必要的指标都必须是最近的（未过期）。

	例如，如果 `allocate_by` 设置为[ `"freespace"` ]，则 peer 按此度量的值排序，并且 `freespace` 选择最多的点作为分配。如果 `allocate_by` 设置为`[ "tag:region", "freespace" ]`，那么我们将按它们的“区域”对点进行分组，然后按它们的“自由空间”对它们进行排序，然后按照它们的总自由空间对这些区域进行排序。第一个 peer 将是该区域中具有最多聚合可用空间的 peer 。然而，第二个 peer 将来自具有最多可用空间的第二个区域，依此类推。这导致 pin 被分配给不同区域的 peer 。一般来说，该过程可以看作是将 peer 放入不同大小的桶中，其中有额外的桶，选择过程通过从尽可能多的不同桶中选择 peer 来工作。

	如何确定分配有一些例外：
	
	- 如果 pin 已经存在并且有分配，则它们不会更改，除非那些分配的 peer 变得不可用导致 `min_replication_factor` 超出限制。
	- 同样，如果一个 pin 已经分配给一些 peer 并且我们正在更改复制因子，则之前分配的 peer 会受到尊重。
	- 如果用户在固定时指定了手动分配，则无论其度量值如何，只要它们可用（否则它们将被忽略），这些 peer 都会被优先考虑。
- IPFS 固定阶段

	分配给该 pin 的每个 peer 都必须确保 IPFS 守护进程将其固定。它们 `pin add` 产生的阶段和结果实际上可以通过以下不同选项进行检查 ：

	- `ipfs-cluster-ctl pin add ...` 

		默认情况下等待 1 秒，并报告 status 正在进行的 IPFS 固定过程产生的结果。
	- `ipfs-cluster-ctl pin add --no-status ...` 

		不等待并报告 `pin ls` 集群固定过程产生的信息。
	- `ipfs-cluster-ctl pin add --wait ...`

		等待至少 1 个对等方中的 IPFS 固定过程完成。

	一旦集群固定阶段完成，每个 peer 都会收到 pinset 中的新项目的通知。如果 peer 在该项目的分配中，它将继续要求 ipfs 固定它：

	- Peer 开始“跟踪”CID
	- 如果分配给它，它会将其排队等待固定，当它进入队列时，会触发 `ipfs pin add` 操作
	- 跟踪过程等待 `ipfs pin add` 操作成功。锁定操作可能会根据 ipfs 接收到的最后一个块的时间而超时，而不是基于锁定所花费的总时间。
	- 固定完成后，该项目被视为已固定。
	- 如果在固定时发生错误，该项目将进入错误状态并最终由集群重试，从而增加其尝试次数。

	固定过程有两个不同的队列，它们占用可用的固定槽。
	
	- 第一个是新项目和多次固定失败的项目的“优先级”队列。
	- 另一个用于其余项目。
	
	这样一来，无论出于何种原因无法完成的固定都不会妨碍您，而是将固定​​插槽用于新固定。
	
	它们产生的阶段和结果实际上可以通过  `pin add` 以下不同选项进行检查：

	- `ipfs-cluster-ctl pin add ...` 默认情况下等待 1 秒，并报告 `status` 正在进行的 IPFS 固定过程产生的结果。
	- `ipfs-cluster-ctl pin add --no-status ...` 不等待并报告 `pin ls` 集群固定过程产生的信息。
	- `ipfs-cluster-ctl pin add --wait ...` 等待至少 1 个对等方中的 IPFS 固定过程完成。
	
- CRDT-batch

	当启用 CRDT-batch 时，pin 将在接收 pin 请求的本地 peer 中进行批处理并在一个批次中一起提交给网络。这只发生在批次达到其最大时间或满时（这两件事都在配置中控制）。

	如果在广播批次之前重新启动 peer ，这些固定将丢失。因此，我们建议在重新启动 peer 之前停止请求并等待批处理 max_age。
- `pin ls` 对比 `status`

	区分 `ipfs-cluster-ctl pin ls` 和非常重要 `ipfs-cluster-ctl status`。两个端点都提供了固定在集群中的 CID 列表，但它们以非常不同的方式进行：

	- `pin ls`

		显示来自集群共享状态或全局 pinset 的信息，这些信息在每个 peer 中都完全可用。它显示了哪些是分配以及如何配置固定。例如：

			QmXvQLhK2heNz65fWRabTfbzXwYfaBgEBuTdUJNzp69Xjx | cluster-website | PIN | Repl. Factor: 2--3 | Allocations: [12D3KooWGbmjg3MDUYFosLNPbE1jKkv5fzKHD7wyGDa1P95iKMjF QmYY1ggjoew5eFrvkenTR3F4uWqtkBkmgfJk8g9Qqcwy51] | Recursive | Metadata: no | Exp: ∞ | Added: 2021-12-07 16:24:58
	- `status` 

		但是，请求有关分配给它的每个集群 peer 上每个 pin 的状态的信息，包括该 CID 是否在 IPFS 上被 PINNED 或者仍然 PINNING，或者由于某种原因出错：

			bafybeiary2ibmljf3l466qzk5hud3rnlk7zped37oik64zlfh22sa5nrg4 | cluster-website:
			    > cluster2             : PINNED | 2021-12-07T15:24:58Z | Attempts: 0 | Priority: false
			    > cluster3             : PINNED | 2021-12-07T15:24:58Z | Attempts: 0 | Priority: false
			    > cluster0             : PINNED | 2021-12-07T15:24:58Z | Attempts: 0 | Priority: false
			    > QmYAajUVaFMw7EyUsZqwDhbNmCsP8L7VDRLuXNkEw6DCC1 : REMOTE | 2021-12-07T16:40:08.122100484+01:00 | Attempts: 0 | Priority: false
			    > QmaHvxFk6DoNsRHqe2a7UJH66AjGDKPG2HCBxr25YYop32 : REMOTE | 2021-12-07T16:40:08.122100484+01:00 | Attempts: 0 | Priority: false
		为了显示此信息，`status` 请求必须调用分配给 pin 的每个 peer （只有固定某些东西的 peer 才能知道该操作的状态）。因此，在 `status` 具有许多 peer 的集群上，请求可能非常昂贵，但提供了有关 pin 状态的非常有用的信息。
		
		这两个命令都采用可选 `cid` 的方式将结果限制为单个项目。结果 `status` 包括检查发生了多少次尝试固定某事以及最后一次尝试是否通过优先固定队列发生的信息。最后，该 `status` 请求支持一个 `--local` 标志来仅报告来自本地 peer 的状态。

### 过滤结果
这些 `status` 命令支持过滤以仅显示在给定情况下（至少在一个对等方中）的固定。支持以下过滤器：

- `cluster_error`

	我们无法获取状态信息的固定（即集群对等体已关闭）
- `pin_error`

	无法固定的pin （由于 ipfs 问题或超时）
- `unpin_error`

	无法取消固定的 pin（由于 ipfs 问题或超时）
- `error`

	插针 `pin_error,unpin_error `或 `cluster_error`
- `unexpected_unpinned`

	没有被 ipfs 固定的 pin，但它们应该被固定并且现在不是 pin_queued 或 pinning。
- `pinned`

	pin 被正确固定
- `pinning`

	 当前被 ipfs 固定的引脚
- `unpinning`

	当前被 ipfs 取消固定的 pin
- `remote`

	分配给其他集群 peer 的固定（远程表示：不由该 peer 处理）。
- `pin_queued`

	等待开始固定的 pin（通常是因为 ipfs 已经固定了一堆其他东西）
- `unpin_queued`

	等待开始取消固定的 pin（通常是因为正在取消固定其他内容）
- `queued`

	引脚 `pin_queued` 或 `unpin_queued` 状态。
	
有时将此选项与 `--local` 一个选项结合起来很有用。例如：
	
- 例1
	
		ipfs-cluster-ctl status --local --filter pinning,queued //只会在本地 peer 中显示仍在固定或排队（等待开始固定）的 CID。
- 例2
	
		ipfs-cluster-ctl status --filter error // 将显示由于某种原因处于错误状态的 CID 的状态信息（错误消息包含更多信息）。
`ipfs-cluster-ctl status --help` 提供有关使用和选项的更多信息

### 恢复
有时某个项目被固定在集群中，但由于不同的原因，它实际上无法固定在分配的 IPFS 守护程序上：
	
- IPFS 守护程序已关闭或没有响应
- 固定操作超时或错误
- 它是从 IPFS 中手动删除的

在这些情况下，项目将显示 `PIN_ERROR`（等效地，`UNPIN_ERROR` 删除时）或的状态 `UNEXPECTEDLY_UNPINNED`。这不是集群问题，它通常表明 IPFS 存在问题（内容不可用等）。

在这种情况下 `ipfs-cluster-ctl recover`，一旦问题得到解决就可以根据需要对分配的 ipfs 守护进程重新触发 pin 或 unpin 操作。如下所述，`recover` 无论如何，每个集群 peer 都会定期自动触发操作。请注意，也可以使用重新添加引脚 `pin add`，获得类似的效果。主要区别在于 `recover` 同步发生（等待直到完成），而 `pin add` 是异步的(立即返回结果)。

`ipfs-cluster-ctl recover --help` 提供有关用法和选项的更多信息。

### 自动同步和恢复
集群 peer 对过期项目运行取消固定操作并自动恢复操作，时间间隔在配置中定义：

- `state_sync_interval` 检查过期项目并可能触发取消固定请求的频率。
- `pin_recover_interval` 控制间隔以触发所有处于错误状态的引脚的恢复操作。

## 共识组件
IPFS 集群 peer 可以使用不同的选择来启动某些组件的实现。最重要的一项是“共识”。“共识组件”负责：

- 通过接收来自其他对等方的修改并发布它们来管理全局集群 pinset。
- 管理磁盘上 pinset 相关数据的持久存储。
- 在所有 peer 之间实现强大的最终一致性：所有 peer 应收敛到相同的 pinset。
- 管理集群 peer 集：执行必要的操作以在集群中添加或删除 peer 。
- 设置 peer 信任：定义受信任的 peer 执行操作和访问本地 RPC 端点。

IPFS 集群提供了两个“共识组件”选项，用户在初始化集群对等体时必须通过提供 `--consensus crdt` 或 `init` 命令 `--consensus raft` 来做出选择。

对于离线集群 pinset 管理，请查看[数据、备份和恢复部分](https://ipfscluster.io/documentation/guides/backups)。

### CRDT
`crdt` 是基于 ipfs 驱动的分布式键值存储的集群 “共识组件” 的默认实现。它：

- 通过 `libp2p-pubsub (GossipSub)` 向 pinset 发布更新，通过 [ipfs-lite](https://github.com/hsanjuan/ipfs-lite) (dht+bitswap) 定位和交换数据。
- 将所有持久数据存储在 `.ipfs-cluster/badger` 文件夹中的本地 BadgerDB 数据存储中。
- 使用 Merkle-CRDTs 通过 [go-ds-crdt](https://github.com/ipfs/go-ds-crdt) 获得最终一致性。这些是仅附加的、不可变的 Merkle-DAG。它们不能在正常情况下被压缩，新的 peer 必须从根发现并遍历它们，如果 DAG 非常深，这可能是一个缓慢的操作。
- 不需要执行任何 peerset 管理。我们通过 pubsub 收到 “ping” 的每个 peer 都被视为集群的成员，直到它们的最后一个指标到期。
- 信任 `trusted_peers` 配置选项中定义的 peer ：只有那些 peer 才能修改本地 peer 中的 pinset 并可以访问“受信任的” RPC 端点。
- 可以选择在单个更新上批处理许多 pin/unpin 操作，从而允许扩展 pin 摄取功能。

### raft
raft 是基于 Raft 共识的 Cluster 的“共识组件”的实现。它：

- 通过将更新直接连接并发送到每个集群对等方来发布更新。
- 将所有持久数据存储在本地 BoltDB 存储和 `.ipfs-cluster/raft` 文件夹中的常规快照上。
- 使用 Raft 共识实现（`hashicorp/raft` 使用 libp2p 网络传输）来获得网络分区的最终一致性和保护。Peerset 视图在 Raft 中可能已经过时，但它们永远不会以需要协调的方式分歧。Raft-clusters 选举一个负责将每个条目提交到日志的领导者。为了使其有效，集群中超过一半的对等方必须确认每个操作。仅附加日志可以合并并压缩为快照，该快照可以发送给新的对等方。
- 通过在 Raft 日志中对 peerset 进行提交操作(添加和删除 peerset)来执行 peerset 管理，因此受制于它们的限制:
	- 一个民选的 leader 
	- 和超过一半的 peer 在线。
- 信任所有 peer 。任何 peer 都可以请求加入基于 Raft 的集群，并且任何 peer 都可以访问其他人的 RPC 端点（只要他们知道集群 secret）。

### 选择共识组件
- 在以下情况下选择 CRDT：
	- 您希望您的集群能够很好地与轻松进出的同行一起工作
	- 您计划让追随者 peer 无权修改 pinset
	- 您没有用于引导的固定对等方，或者您需要利用 mDNS 自动发现
	- 集群需要适应固定/取消固定操作的定期和大量爆发（批处理支持有帮助）。
- 在以下情况下选择 Raft：
	- 您的集群 peerset 是稳定的（总是相同的 peer，总是在运行）并且不经常更新
	- 您需要保持最安全的一面（Raft 共识较旧，经过更多测试的实现方式）
	- 您不能容忍导致不同状态的临时分区
	- 您不需要 CRDT 模式提供的任何东西

### CRDT 与 Raft 比较
CRDT| Raft
---|---
GossipSub 广播|与每个明确联系的 peer 联系
可信赖的 peer 支持|所有对等方都信任
“Follower peers”和“Publisher peers”支持|所有对等方都是发布者
peer 可以自由出入|> 50% 必须始终在线，否则无效。有人不在线时记录的错误。
国家规模总是在增长|删除后状态大小减小
集群状态压缩只能通过使整个集群脱机来实现|自动压缩状态
首次同步可能很慢|通过发送完整快照加快首次同步
与大量 peer 一起工作|与少数 peer 合作
基于 IPFS 技术（bitswap、dht、pubsub)|基于hashicorp-tech (raft)
强大的最终一致性：Pinsets 在合并之前可能会发散|共识：Pinsets 可能会过时，但绝不会出现分歧
具有批处理支持的快速固定提取|缓慢的 pin 提取
Pin 已提交 不等于 Pin 已到达大多数 peer |已提交固定等于固定已到达大多数 peer 
最大测试尺寸：60M pin|100k pin

## 安全和端口
本节探讨运行 IPFS 集群时的一些安全注意事项。

在保护对系统的访问时，需要考虑 IPFS 集群中的四种类型的端点。暴露未受保护的端点可能会让任何人控制集群。集群配置使用合理的默认值。

- [集群秘密](https://ipfscluster.io/documentation/guides/security/#the-cluster-secret)

	 `service.json` 文件中的 `secret` 有 32 字节十六进制编码充当 libp2p 网络保护器。这为使用预共享密钥的 peer  (libp2p) 之间的所有通信提供了额外的加密。

	这使得无法与对等方的 swarm 端点进行通信（见下文），因此无法在事先不知道秘密的情况下向该对等方发送 RPC 命令。

	该密钥是基于 raft 的集群的安全要求，它不强制执行任何 RPC 授权策略。只要 `trusted_peers` 设置正确，基于 CRDT 的集群就可以使用空密钥运行：只有其中的对等 `trusted_peers` 方可以修改 pinset 并执行操作。

	但是，我们建议 secret 在所有情况下都设置它，因为它提供了网络隔离：没有秘密运行的集群可能会发现并连接到主 IPFS 网络，这对于集群 peer （和 IPFS 网络）几乎没有用处。
- [在 trusted_peers CRDT 模式下](https://ipfscluster.io/documentation/guides/security/#the-trusted-peers-in-crdt-mode)

	该文件部分中的 `trusted_peers` 选项提供对对等 RPC 端点的访问控制，并允许修改由该数组中的 peer 发出的 pinset（由它们的 peer  ID 标识），但 `仅适用于 crdtservice.json 在 crdt-mode 中运行的集群`。

	受信任的 peer 可以：

	- 修改 pinset：间接触发 `pin/unpin` 操作
	- 触发状态同步操作（导致 `ipfs pin ls`）
	- 向 peer 添加内容（导致 `ipfs block put`）

	不受信任的 peer 只能访问 ID 和版本端点（返回 IPFS 和集群 peer 信息）。

	`trusted_peers` 可以设置为 `[ "*" ]` 信任所有其他对等方。
- [端口和端点概述](https://ipfscluster.io/documentation/guides/security/#ports-overview)
	- 端口概述
		- Cluster swarm `tcp:9096` 

			由 Cluster swarm 使用并受共享密钥保护。暴露这个端口是可以的（集群 `secret` 作为密码与之交互）。
		- HTTP API `tcp:9094`

			可以在启用 SSL 和设置基本身份验证时公开
		- IPFS Pinning API 端点 `tcp:9097` 

			类似于上面的 HTTP 端点。
		- libp2p-HTTP API

			当使用替代 [libp2p](https://ipfscluster.io/documentation/reference/configuration/#restapi) 主机时，对于 `api，libp2p_listen_multiaddress` 可以在启用基本身份验证时公开。
		- IPFS API `tcp:5001` 

			是 IPFS 守护进程的 API，不应暴露给 `localhost`.
		- IPFS 代理端点 `tcp:9095`

			如果没有顶部的身份验证机制（等...），则不应公开。nginx 默认情况下，它不提供身份验证或加密（类似于 IPFS 的 `tcp:5001`）
		- Prometheus `tcp:8888`

			启用指标
		- Jaeger `tcp:6831`

			启用跟踪时，默认使用公开 Jaeger 服务。

		阅读以下部分以获得更详细的说明。
	- 集群群端点

		端点由 `cluster.listen_multiaddress` 配置密钥控制，默认为 `/ip4/0.0.0.0/tcp/9096` 并表示与其他 peer 建立通信的侦听地址（通过远程 RPC 调用和共识协议）。

		如上所述，共享密钥通过锁定此端点来控制授权，以便只有持有该密钥的集群对等方可以通信。我们建议永远不要使用空密钥运行。
	- HTTP API 和 IPFS 固定服务 API 端点

		IPFS 集群 peer 默认提供一个 `HTTP API 端点` 和 `一个 IPFS 固定服务 API 端点`，可以使用 SSL 进行配置。它还为每个提供了一个 `libp2p API ` 端点，该端点重用集群 libp2p 主机或专门配置的 libp2p 主机。

		这些端点由 `*.http_listen_multiaddress`（默认 `/ip4/127.0.0.1/tcp/9094`）和 `*.libp2p_listen_multiaddress`（如果是特定的 `private_key` 并且 `id` 在 `restapi/pinsvcapi` 部分中配置）控制。

		请注意，
		
		- 当没有配置额外的 libp2p 主机并在 Raft 模式下运行时，集群的对等 libp2p 主机（侦听 `0.0.0.0`）被重新用于提供 libp2p API 端点。如前所述，此端点仅受集群机密保护。
		- 在 CRDT 模式下运行时，libp2p 端点被禁用，因为集群机密可能会被共享，否则会向世界公开一个开放端点。为了在使用 CRDT 模式时运行 lib2p API 端点，请在配置中配置一个额外的、单独的 libp2p restapi 主机。

		两个端点都支持基本身份验证，但默认情况下未进行身份验证。

		对这些端点的访问允许完全控制 IPFS 集群 peer ，因此当它们向非 `localhost`. 可配置的 SSL 或 libp2p 端点提供的安全通道以及基本身份验证允许安全地使用这些端点进行远程管理。

		配置 `restapi/pinsvc api` 提供了许多变量来配置 CORS 标头。默认情况下，`Allow-Origin` 设置为 `*` 和 `Allow-MethodsGET`。您应该验证此配置是否适合您的需求、应用程序和环境。

		可以通过从 `restapi/pinsvcapi`  配置中删除部分来完全禁用这些 API 。如果 REST API 被禁用，`ipfs-cluster-ctl` 将无法与对等方交谈。
	- IPFS 和 IPFS 代理端点

		IPFS 集群 peer 使用 IPFS HTTP API（默认情况下在 `/ip4/127.0.0.1/tcp/9095`.

		IPFS 集群 peer 还提供未经身份验证的 HTTP IPFS 代理端点，由 `ipfshttp.proxy_listen_multiaddress` 默认为 `/ip4/127.0.0.1/tcp/9095`.

		访问这两个端点中的任何一个都意味着在一定程度上控制 IPFS 守护进程和 IPFS 集群。因此，它们默认运行  `localhost`。

		IPFS 代理将尝试从 IPFS 守护程序模仿 CORS 配置。如果您的应用程序安全性依赖于 CORS，您应该首先配置 IPFS 守护程序，然后验证来自代理中被劫持端点的响应是否符合预期。`OPTIONS` 请求总是代理到 IPFS。

		通过从配置中删除该 `ipfsproxy` 部分，可以完全禁用 IPFS 代理 API。

### 数据、备份和恢复
`ipfs-cluster-service` 默认情况下，正在运行的 IPFS 集群 peer （带有 ）保存的配置和数据位于 `$HOME/.ipfs-cluster/`文件夹中。集群对等体在磁盘上保留了几种类型的信息：

- 供将来使用的已知对等地址列表。`peerstore` 关机时存储在文件中。
- 集群 pinset（固定在集群中的对象列表以及与其关联的所有选项（如名称、分配或复制因子）根据选择的共识组件进行存储：
	- `crdt badger` 将所有内容存储在文件夹中的键值 BadgerDB 数据存储中。
	- `raft` 存储构成 pinset 的仅附加日志，以及 BoltDB 存储中经常快照的集群 peer 列表。全部保存在 `raft` 文件夹中。
- `service.json` 并且 `identity.json` 也是持久性数据，但通常它们不会被修改。

#### 离线状态：导出导入
由于 pinset 信息是持久存储在磁盘上的，因此可以通过以下方式从离线 peer 导出它：

	ipfs-cluster-service state export
这将生成代表当前 pinset 的 json 对象列表（非常类似于 `ipfs-cluster-ctl --enc=json pin ls` 在线 peer ）。可以使用以下命令重新导入生成的文件：

	ipfs-cluster-service state import

`ipfs-cluster-service` 始终使用您导出时使用的相同版本重新导入。

将状态导入多个集群 peer 时，请考虑 `https://github.com/ipfs-cluster/ipfs-cluster/issues/1547`。

请注意，`状态转储仅包含 pinset`。它不包括任何

- 记账信息
- Raft peerset 成员资格
- Raft 当前期限
- CRDT Merkle-DAG 节点等。

因此，在重新导入 pinset 时，请务必记住：

- 在 `raft` 中，给定的 pinset 将用于创建一个新的快照，比任何现有的快照都更新，但包括现有 peerset 等信息。
- 在 `crdt` 中，导入将[彻底清除](https://ipfscluster.io/documentation/guides/backups/#resetting-a-peer-state-cleanup)状态并创建单个批次的 `Merkle-DAG` 节点。这通过替换 Merkle-DAG 有效地压缩了状态，但是为了防止这个 peer 重新下载旧的 DAG，集群中的所有其他 peer 也应该替换或删除它。

有关详细信息，请参阅下面的[灾难恢复](https://ipfscluster.io/documentation/guides/backups/#disaster-recovery)。

`raft` 状态转储可以作为 `crdt` pinset 导入，反之亦然。

#### 重置 peer ：状态清理
清理状态会导致一个空白的集群 peer 。这样的 peer 将需要重新引导 ( raft) 或重新连接 ( crdt) 到集群，以便重新下载状态。如上所述，状态也可以通过导入来提供。可以通过以下方式执行清理：

	ipfs-cluster-service state cleanup
请注意，这不会删除或重写配置、身份或 peerstore 文件。删除 `raft` 或 `crdt` 数据文件夹在所有效果上都等同于状态清理。

使用 Raft 时，`raft` 文件夹将重命名为 `raft.old.X`. 根据 `backups_rotate` 配置值，将保留多个副本。使用 CRDT 时，`crdt` 相关数据将从 badger 数据存储中删除。
#### 灾难恢复
IPFS 集群存储的唯一内容是集群对等方独有的内容是 pinset。IPFS 内容由 IPFS 存储。通常，如果您正在运行一个集群，将会有几个 peer 复制内容和集群 pinset，因此当一个或多个 peer 崩溃、被破坏、消失或只是失败时，它们可以重置为干净的形式重新同步其他现有的 peer。

	一个健康的集群是具有至少 50% 的健康在线 peer  ( raft) 或至少一个受信任的、健康的 peer  ( crdt)。
因此，任何 peer 都可以完全[重置](https://ipfscluster.io/documentation/guides/backups/#resetting-a-peer-state-cleanup)并重新加入一个健康的集群，过程与添加新 peer 相同。在 `ipfs-cluster-ctl peer rm` 中，如果 raft 离开的 peer 永远不会再次加入 ，则应该手动删除它们。

- 不健康的集群

	不健康的集群情况会发生变化：

	- 在 `crdt`中，缺少受信任的 peer 将阻止恢复的 peer 重新同步到集群状态（尽管作为一种解决方法，它可以暂时信任任何其他 peer ）。
	- 在 `raft` 中，当超过 50% 的 peer 宕机时，缺乏仲裁会阻止添加新的 peer 、删除损坏的 peer 或运行集群。

	在这种情况下，简单地挽救状态并按照下一个过程重新创建集群可能会更容易：

	1. 找到仍然存储状态（`raft` 或 `badger` 文件夹）的 peer 
	2. 导出 pinset `ipfs-cluster-service state export`
	3. 重置您的 peer 或从头开始设置新的 peer 
	4. 运行 `ipfs-cluster-service state import` 以从步骤 2 导入状态副本
	5. 将 peer 作为单个 peer 集群启动
	6. 完全清理、升级和引导其余的 peer 到正在运行的 peer 

	状态升级

	从 0.10.0 版本开始，集群 peer 将不需要手动状态升级（ `state upgrade` 命令已消失）。

## peer 管理
根据集群使用的“共识”组件（共识组件负责管理 peerset），在集群中添加和删除 peer 可能是更简单或更复杂的操作。
### 列出 peer 
	ipfs-cluster-ctl peers ls

该 `peers ls` 命令将生成集群中的 peer 列表，并将其所有信息。这相当于调用 `ipfs-cluster-ctl id` 每个集群 peer 并使用结果构建一个列表，但要使其工作，它需要联系集群的所有当前 peer ，这意味着它可能是一个缓慢的操作。
	
相反，如果您只想要集群中 peer  ID 的列表，您可以使用以下 `id` 命令查看它（ `text` 输出仅显示 peer 的数量）：

	ipfs-cluster-ctl --enc=json id
### 添加新的 peer 
[向集群添加新 peer](https://ipfscluster.io/documentation/deployment/bootstrap) 的工作方式与引导集群部分中的描述完全相同。通用方法是使用 

	ipfs-cluster-service daemon --bootstrap
### 删除 peer
- CRDT模式

	在 CRDT 模式下，可以简单地停止 peer 。其他 peer 可能会认为它们是 peer 集的一部分，直到它们的最后一个指标到期。因此，减少 [度量ttls](https://ipfscluster.io/documentation/guides/metrics) 将加快这一速度。
- Raft 模式

	在 Raft 模式下，可以停止 peer ，但是它们将无法参与集群操作，并且仍将被视为 peer 集的一部分。如果将来会重新启动 peer 并且大多数集群 peer 仍将在线，则这非常好。否则，需要手动删除离开的对等体：

		ipfs-cluster-ctl peers rm <pid> \\ 注意只有当 Raft 集群至少有 50% 的成员在线时，才能删除 Raft peer。
	
	这可以从对等方关闭（自删除）或任何其他对等方调用。在任何情况下，它都会导致对等方在意识到它已被删除时自行关闭。

	或者，`leave_on_shutdown` 可以将配置选项设置为 `true`。使用此选项，干净关闭的 peer 将尝试在此过程中将自己从 Raft  peer 集中删除。已从 Raft peerset 中删除的 Peers 会自动清理其状态，并且需要再次引导到它以重新加入它。

## 集群发布订阅指标
本节是关于集群 peer 之间广播的指标。有关监控的 Prometheus 指标，请参 [阅监控指南](https://ipfscluster.io/documentation/guides/monitoring)。

集群 peer 定期在彼此之间广播（使用 gossipsub）指标。这些指标有几个目的：

- 它们允许检测 peer 何时离开集群（每个指标都有一个到期日期，预计会在到期之前更新）。这可用于触发诸如重新固定之类的操作。
- 在 `crdt` 模式下，它们用于识别当前集群 `peerset`（具有未过期指标的 peer 列表）。
- 它们用于传达诸如 peer 名称和可用空间之类的信息，例如，这些信息可用于做出固定分配决策。

指标由 [“informer”](https://ipfscluster.io/documentation/configuration/#the-informer-section) 组件生成。目前有几种类型的指标：

- `ping`

	缺少来自给定集群 peer 的 ping 表示 peer 已关闭，并在启用时用于触发重新固定。ping 指标包括有关每个 peer 的信息，例如其 peer 名称、IPFS 守护进程 ID 和地址等，然后在请求时重新用于填充 pin 状态对象中的字段。
- `freespace`

	此指标告知 IPFS 在其存储库中有多少可用空间，并用于决定是否将新固定分配给此 peer 或其他点。
- `tag:*`

	“标签” 指标提供来自标签通知者的值。例如，peer 可以广播一个 `tag:group` 带有 value 的指标 `server`。平衡分配器使用这些值在单个标签的不同值之间分配 pin。
- `pinqueue`

	该指标包含排队等待固定的项目数，也可用于避免固定在具有长固定队列的 peer 上。

管理员可以使用以下命令检查对等方收到的最新指标：

	ipfs-cluster-ctl health metrics # lists available metrics
	ipfs-cluster-ctl health metrics ping
	ipfs-cluster-ctl health metrics freespace
	...
注意：

- `freespace` 与其他 `informer-metrics` 相关联的生存时间 `metric_ttl` 由不同 `informer` 的选项控制（`disk` 默认情况下使用 `informer`）。增加它会减少 peer 向网络发送指标的次数。
- 与指标关联的生存时间 `ping` 由该 `cluster.monitor_ping_interval` 选项控制。
- `pubsubmon.check_interval` 选项控制 peer 检查其他 peer 的过期指标的频率。

您可以[阅读配置](https://ipfscluster.io/documentation/reference/configuration)参考中的所有详细信息。

## 升级
IPFS 集群项目定期发布新版本。本节介绍以最少或没有停机时间升级集群的过程。

[在升级之前检查更新日志](https://github.com/ipfs-cluster/ipfs-cluster/blob/master/CHANGELOG.md)非常重要，以便熟悉自上一个版本以来的更改。有关潜在不兼容性和重大更改的所有信息都包含在此处。

另一个主要考虑因素是：

	从 v0.12.1 开始，所有集群 peer 都需要在相同的 RPC 协议版本上运行。
RPC 协议版本可以在 `ipfs-cluster-ctl --enc=json id( rpc_protocol_versionfield) ` 的响应中看到，也可以在[源码](https://github.com/ipfs-cluster/ipfs-cluster/blob/master/version/version.go)中看到。它应该在多个 IPFS 集群版本中保持稳定，并且仅在发生非向后兼容的 RPC 更改时才会更改。

	注意 如果对等方的 RPC 协议版本不匹配，它们将无法通信。
当 RPC 协议版本相同时，即使集群由运行不同版本的对等体组成，对等体的核心功能也将起作用，除非[更改日志](https://github.com/ipfs-cluster/ipfs-cluster/blob/master/CHANGELOG.md)另有说明。

### 运行升级
一般的做法是：

1. 升级 `ipfs-cluster-ctl`，`ipfs-cluster-service`或 `ipfs-cluster-follow`到新版本。
2. 重新启动所有 peer （按顺序或一次）。

在 `raft` 的情况下，重新启动仅在以下情况下有效：

- `leave_on_shutdown` 设置为 `false`。否则，这些 peer 将需要在下一次启动时被引导。
- `wait_for_leader_timeout` 足够高以解释集群中大多数对等方的重启（在大多数情况下默认应该没问题）

## 故障排除
本节包含一些提示，用于在运行 IPFS 集群时识别和纠正问题。
### IPFS 集群构建失败
请阅读[下载页面](https://ipfscluster.io/download)。它有关于如何构建软件的说明（请遵循它们）。
### 调试日志
发现问题时，尝试找出问题并在寻求帮助时可能提供一些相关日志总是有用的。

- `ipfs-cluster-service`

	默认情况下，仅 `ipfs-cluster-service` 打印和消息。您可以通过多种方式提高日志记录级别：`INFO` `WARNING` `ERROR`

	- `--debug`

		将设置 `DEBUG` 所有日志子系统中的日志级别。这会产生大量信息，甚至可能会显着减慢您的 peer 。默认不使用！
	- 允许对 `--loglevel` 应该记录哪些组件以及它们应该使用哪些级别进行细粒度控制。
		- `--loglevel debug`

			使所有与集群相关的组件打印 DEBUG 消息。这也可以受组件限制：`--loglevel error,restapi:debug,pintracker:debug` . 将默认日志级别设置为，`ERROr` 同时,设置 `restapi `和 `pintracker` 组件 DEBUG 级别。
			
	解释调试信息可能很棘手。举个例子：
	
		2020-05-17T00:51:52.953+0200    ERROR   ipfshttp        ipfshttp/ipfshttp.go:722        Post "http://127.0.0.1:5001/api/v0/repo/stat?size-only=true": dial tcp 127.0.0.1:5001: connect: connection refused
	- 上一行显示了 ERROR 来自 ipfshttp 设施的严重性消息。它已登录 `ipfshttp/ipfshttp.go:722`（文件名和行号）。该工具对应于 `ipfshttp` 实现 IPFS 连接器组件的模块。此信息有助于缩小错误来自的上下文。错误消息表明组件未能对 IPFS API 执行 GET 请求。
	- 鉴于所有这些上下文，我们可以确定 ipfs 守护进程很可能没有运行，或者无法访问。
	- 调试时，您可以找出哪个组件产生了错误，然后使用它 `--loglevel <component>:debug` 来获取有关该组件正在做什么的更多信息。
- `ipfs-cluster-ctl`

	`ipfs-cluster-ctl` 提供一个 `--debug` 标志，该标志将打印有关该工具使用的 API 端点的信息。`--enc json` 允许 `json` 从 API 打印原始响应。
- `ipfs-cluster-follow`

	`ipfs-cluster-follow` 不包括增加详细程度的方法。但是，您可以在所示的 `ipfs-cluster-service -c <folder> daemon `地方运行。然后，您可以增加详细程度，如上所示。`<folder>Config folderipfs-cluster-follow <clusterName> info` 仅在必要时进行调试！

### peer 未启动
当您的 peer 未启动时：

- 检查日志并查找错误
- 所有的监听地址都是 free 的还是被不同的进程使用？
- 集群的其他 peer 是否可达？
- `cluster.secret` 对所有同龄人都一样吗？
- 仔细检查 `peerstore` 文件是否具有正确的内容，并且您已遵循[引导集群](https://ipfscluster.io/documentation/deployment/bootstrap)部分中的方法之一。
- 仔细检查集群的其余部分是否处于健康状态。
- 在某些情况下，它可能有助于运行 `ipfs-cluster-service state clean`（特别是如果未启动的原因是 raft 状态和集群 peer 之间的不匹配）。假设集群是健康的，这将允许非启动对等方在引导时从集群领导者那里拉一个干净的状态。

### peer 意外停止
当 peer 意外停止时：

- 确保您没有从集群中删除 peer 或触发关闭
- 检查日志以获取进程因内部故障而死亡的任何线索
- 检查您的系统日志以查找是否有任何外部因素杀死了该进程
- 报告任何应用程序恐慌，因为它们不应该发生，连同日志

### libp2p 错误
由于集群是建立在 libp2p 之上的，因此新用户面临的许多错误都来自 libp2p，并且具有令人困惑的消息，乍一看并不明显。此列表编译了其中一些：

- `dial attempt failed: misdial to <peer.ID XXXXXX> through ....`

	这意味着您正在联系的多地址中的 peer 与预期的不同。
- `dial attempt failed: connection refused`

	 peer 未运行或未侦听预期的地址/协议/端口。
- `dial attempt failed: context deadline exceeded`

	这意味着该地址不可访问或使用了错误的秘密。
- `dial backoff`

	同上。
- `dial attempt failed: incoming message was too large`

	这可能意味着您的集群  peer  没有共享相同的秘密。
- `version not supported`

	这意味着您的节点运行在不兼容的版本上。

## 监控和追踪
IPFS 集群可以公开 Prometheus 端点以进行度量抓取，还可以向 Jaeger 提交代码跟踪信息。

这些是在配置 `observations` 部分中[配置](https://ipfscluster.io/documentation/reference/configuration/#the-observations-section)的，可以从那里启用或者通过启动集群 peer 来启用：

	ipfs-cluster-service daemon --stats --tracing
除了所有特定于 go 的指标外，集群还导出了一些指标来跟踪集群 peer 的当前状态，这些指标可以通过 `curl 'http://127.0.0.1:8888/metrics | grep cluster` 快速检查 ，包括 

- pin 总数
- 排队项目数等

### 跟踪和指标的开发设置
以下部分显示如何：

- 使用 Docker 在本地配置和运行 Jaeger 和 Prometheus 服务
- 配置 IPFS 集群以将跟踪信息发送到 Jaeger 并将指标发送到 Prometheus

本节介绍如何在本地开发环境中部署和配置跟踪和指标。Jaeger 或 Prometheus 的生产部署超出了此处所涵盖的范围。

### Jaeger
首先，拉下 Jaeger 一体化镜像：

	$ docker pull jaegertracing/all-in-one:1.9
下载映像后，使用以下配置运行映像：

	$ docker run -d --name jaeger \
	  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
	  -p 5775:5775/udp \
	  -p 6831:6831/udp \
	  -p 6832:6832/udp \
	  -p 5778:5778 \
	  -p 16686:16686 \
	  -p 14268:14268 \
	  -p 9411:9411 \
	  jaegertracing/all-in-one:1.9
特别值得注意的是 Jaeger 容器上的以下端口：

- ` -6831` 是 IPFS 集群使用的默认代理端点 
- `-16686` 公开 Jaeger 服务的 Web UI，您可以在其中查询和搜索收集的跟踪

### Prometheus
为了配置 Prometheus，我们创建一个 `prometheus.yml` 文件，如下所示：

	global:
	  scrape_interval:     15s
	  evaluation_interval: 15s
	
	scrape_configs:
	  - job_name: ipfs-cluster-daemon
	    scrape_interval:     2s
	    static_configs:
	      - targets: ['localhost:8888']
指定的目标地址与 IPFS 集群中指标配置中的默认地址匹配，但感觉将其更改为更适合您的环境的地址，只需确保更新您 `~/.ipfs-cluster/service.json` 的匹配即可。

为了运行 prometheus，拉取以下 Docker 镜像：

	$ docker pull prom/prometheus
然后运行 ​​Prometheus 容器，确保挂载我们刚刚创建的配置文件：

	$ docker run --network host -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml --name promy prom/prometheus
请注意，要让 Prometheus 到达 IPFS 集群公开的指标端点，它要求容器在主机的网络上运行，这是通过 `--network host` 上面运行命令中的标志完成的。

### IPFS 集群配置
配置文件中的 `observations` 部分 `service.json` 如下：

	{
	  "metrics": {
	    "enable_stats": true,
	    "prometheus_endpoint": "/ip4/0.0.0.0/tcp/8888",
	    "reporting_interval": "2s"
	  },
	  "tracing": {
	    "enable_tracing": true,
	    "jaeger_agent_endpoint": "/ip4/0.0.0.0/udp/6831",
	    "sampling_prob": 0.3,
	    "service_name": "cluster-daemon"
	  }
	}
对于本地开发跟踪，建议将 `observations.tracing.sampling_prob` 更改为 `1`，以便记录系统中的每个动作并发送给 Jaeger。

使用上述配置运行集群对等体应该为 Prometheus 提供一个端点来收集指标并将跟踪推送到 Jaeger。

集群 peer 启动后，转到 `http://localhost:9090/targets` 以确认 Prometheus 已经能够开始从 IPFS 集群中抓取指标。

为了确认跟踪功能正常，我们将使用 IPFS Cluster 命令一步添加一个文件和 pin 到 IPFS Cluster `add`，然后在 Jaeger 中搜索它的跟踪。

	$ echo 'test tracing file' > test.file
	$ ipfs-cluster-ctl add test.file
转到 `https://localhost:16686`，您应该会看到一个跟踪，它可能 `<trace-without-root-span>` 由于 Jaeger 如何创建/确定根跨度的问题而被标记，但所有信息仍在内部。如果那里什么都没有，请给它一些时间将痕迹刷新到 Jaeger Collector，因为它不是即时的。

在运行了一些命令以获取一些跟踪之后，现在是查看[Prometheus](http://localhost:9090/graph?g0.range_input=1h&g0.expr=histogram_quantile(0.95%2C%20sum(rate(cluster_gorpc_libp2p_io_server_server_latency_bucket%5B5m%5D))%20by%20(le%2Cgorpc_server_method))&g0.tab=0) 的图形页面的 gorpc 好时机，该页面预先填充了 IPFS  集群组件之间调用的请求延迟的直方图。为收集配置了许多其他指标，可以在Execute 按钮旁边的下拉菜单中找到它们。

希望此工具能够让您更好地了解 IPFS 集群的运行和执行方式

## 在 Kubernetes 上部署
### Monaparty 模板
Monaparty 在 [https://github.com/monaparty/helm-ipfs-cluster](https://github.com/monaparty/helm-ipfs-cluster) 维护用于集群部署的 Helm 模板
### 自定义
请注意，我们有一段时间没有更新以下说明。他们展示了如何使用 [Kustomize](https://kustomize.io/) 在 Kubernetes 上运行一个简单的集群。

	本指南假设您有一个正在运行的 Kubernetes 集群来部署并正确配置了 `kubectl`。
### 准备配置值
- 配置秘密资源

	在 Kubernetes 中，`Secret` 对象用于保存值，例如令牌或私钥。

		# secret.yaml
		apiVersion: v1
		kind: Secret
		metadata:
		  name: secret-config
		type: Opaque
		data:
		  cluster-secret: <INSERT_SECRET>
- 生成集群秘密

	要在 `secret.yaml` 中生成 `cluster_secret`值，请运行以下命令并将输出插入 `secret.yaml` 文件中的适当位置：

		$ od  -vN 32 -An -tx1 /dev/urandom | tr -d ' \n' | base64 -w 0 -
- 引导对等 ID 和私钥

	要生成 `bootstrap_peer_id` 和 `bootstrap_peer_priv_key` 的值，请安装 `ipfs-key` 并运行以下命令：

		$ ipfs-key | base64 -w 0
	将 id 复制到 `env-configmap.yaml` 文件中。

		# env-configmap.yaml
		apiVersion: v1
		kind: ConfigMap
		metadata:
		  name: env-config
		data:
		  bootstrap-peer-id: <INSERT_PEER_ID>
	然后复制私钥值并使用它运行以下命令：

		$ echo "<INSERT_PRIV_KEY_VALUE_HERE>" | base64 -w 0 -
	将输出复制到 `secret.yaml` 文件中。

		# secret.yaml
		apiVersion: v1
		kind: Secret
		metadata:
		  name: secret-config
		type: Opaque
		data:
		  cluster-secret: <INSERT_SECRET>
		  bootstrap-peer-priv-key: <INSERT_KEY>
- 定义有状态集

	来自 [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset)的 Kubernetes 文档：

	管理一组 Pod 的部署和扩展，并保证这些 Pod 的顺序和唯一性。

	与 Deployment 一样，StatefulSet 管理基于相同容器规范的 Pod。与 Deployment 不同，StatefulSet 为其每个 Pod 维护一个粘性标识。这些 pod 是根据相同的规范创建的，但不可互换：每个 pod 都有一个持久标识符，它在任何重新调度时都会维护该标识符。

	这对我们来说意味着，任何 Kubernetes 生成的配置，例如主机名，即 `ipfs-cluster-0`，都将与相同的 Pod 和 VolumeClaim 相关联，这意味着例如，主机名将始终与存储在 `~/.ipfs-cluster/service.json` 文件中的相同对等 id 相关联。所有这一切都很重要，因为它允许我们引导我们的 ipfs 集群，而无需启动特定的引导对等点。

	将 StatefulSet 定义分解成块，首先是 StatefulSet 规范的序言和开始：

		apiVersion: apps/v1
		kind: StatefulSet
		metadata:
		  name: ipfs-cluster
		spec:
		  serviceName: ipfs-cluster
		  replicas: 3
		  selector:
		    matchLabels:
		      app: ipfs-cluster
	接下来是 `go-ipfs` 容器的定义：

		  template:
		    metadata:
		      labels:
		        app: ipfs-cluster
		    spec:
		      initContainers:
		        - name: configure-ipfs
		          image: "ipfs/go-ipfs:v0.4.18"
		          command: ["sh", "/custom/configure-ipfs.sh"]
		          volumeMounts:
		            - name: ipfs-storage
		              mountPath: /data/ipfs
		            - name: configure-script
		              mountPath: /custom
		      containers:
		        - name: ipfs
		          image: "ipfs/go-ipfs:v0.4.18"
		          imagePullPolicy: IfNotPresent
		          env:
		            - name: IPFS_FD_MAX
		              value: "4096"
		          ports:
		            - name: swarm
		              protocol: TCP
		              containerPort: 4001
		            - name: swarm-udp
		              protocol: UDP
		              containerPort: 4002
		            - name: api
		              protocol: TCP
		              containerPort: 5001
		            - name: ws
		              protocol: TCP
		              containerPort: 8081
		            - name: http
		              protocol: TCP
		              containerPort: 8080
		          livenessProbe:
		            tcpSocket:
		              port: swarm
		            initialDelaySeconds: 30
		            timeoutSeconds: 5
		            periodSeconds: 15
		          volumeMounts:
		            - name: ipfs-storage
		              mountPath: /data/ipfs
		            - name: configure-script
		              mountPath: /custom
		          resources:
		            {}
	请注意 `initContainers` 用于使用生产适当值配置 ipfs 节点的部分，请参阅[定义配置脚本](https://ipfscluster.io/documentation/guides/k8s/#defining-configuration-scripts)。

	接下来我们定义 `ipfs-cluste` r容器：

        - name: ipfs-cluster
          image: "ipfs-cluster/ipfs-cluster:latest"
          imagePullPolicy: IfNotPresent
          command: ["sh", "/custom/entrypoint.sh"]
          envFrom:
            - configMapRef:
                name: env-config
          env:
            - name: BOOTSTRAP_PEER_ID
              valueFrom:
                configMapRef:
                  name: env-config
                  key: bootstrap-peer-id
            - name: BOOTSTRAP_PEER_PRIV_KEY
              valueFrom:
                secretKeyRef:
                  name: secret-config
                  key: bootstrap-peer-priv-key
            - name: CLUSTER_SECRET
              valueFrom:
                secretKeyRef:
                  name: secret-config
                  key: cluster-secret
            - name: CLUSTER_MONITOR_PING_INTERVAL
              value: "3m"
            - name: SVC_NAME
              value: $(CLUSTER_SVC_NAME)
          ports:
            - name: api-http
              containerPort: 9094
              protocol: TCP
            - name: proxy-http
              containerPort: 9095
              protocol: TCP
            - name: cluster-swarm
              containerPort: 9096
              protocol: TCP
          livenessProbe:
            tcpSocket:
              port: cluster-swarm
            initialDelaySeconds: 5
            timeoutSeconds: 5
            periodSeconds: 10
          volumeMounts:
            - name: cluster-storage
              mountPath: /data/ipfs-cluster
            - name: configure-script
              mountPath: /custom
          resources:
            {}
	请注意，如果 `BOOTSTRAP_PEER_ID` 是 `BOOTSTRAP_PEER_PRIV_KEY` 我们之前定义的值，它们将仅用作第一个 `ipfs-cluster` 容器 `ipfs-cluster-0`，然后 `BOOTSTRAP_PEER_ID` 将用于将引导多地址传递给其他 ipfs-cluster 容器。

	最后，我们定义配置脚本的卷以及 ipfs 和 ipfs-cluster 容器的数据卷：

	      volumes:
	      - name: configure-script
	        configMap:
	          name: ipfs-cluster-set-bootstrap-conf


		  volumeClaimTemplates:
		    - metadata:
		        name: cluster-storage
		      spec:
		        storageClassName: standard
		        accessModes: ["ReadWriteOnce"]
		        persistentVolumeReclaimPolicy: Retain
		        resources:
		          requests:
		            storage: 5Gi
		    - metadata:
		        name: ipfs-storage
		      spec:
		        storageClassName: standard
		        accessModes: ["ReadWriteOnce"]
		        persistentVolumeReclaimPolicy: Retain
		        resources:
		          requests:
		            storage: 200Gi
	根据您的云提供商，您必须将`storageClassName `的值更改为适当的值。

	可以在 [github.com/lanzafame/ipfs-cluster-k8s](https://github.com/lanzafame/ipfs-cluster-k8s/blob/deploy-k8s-guide/ipfs-cluster-base/cluster-statefulset.yaml) 找到完整的 StatefulSet 定义。

- 定义配置脚本

	ConfigMap 包含两个脚本，`entrypoint.sh` 它们可以实现 ipfs-cluster 集群的免提引导，并使用生产值 `configure-ipfs.sh` 配置守护进程。ipfs 有关为生产配置 ipfs 的更多信息，请参阅 go-ipfs [配置调整](https://ipfscluster.io/documentation/deployment/#go-ipfs-configuration-tweaks)。

		apiVersion: v1
		kind: ConfigMap
		metadata:
		  name: ipfs-cluster-set-bootstrap-conf
		data:
		  entrypoint.sh: |
		    #!/bin/sh
		    user=ipfs
		
		    # 这是 k8s 的自定义入口点，设计用于连接到集群中运行的引导节点。它已经使用 configmap 进行了设置，以允许动态更改。
		
		    if [ ! -f /data/ipfs-cluster/service.json ]; then
		      ipfs-cluster-service init
		    fi
		
		    PEER_HOSTNAME=`cat /proc/sys/kernel/hostname`
		
		    grep -q ".*ipfs-cluster-0.*" /proc/sys/kernel/hostname
		    if [ $? -eq 0 ]; then
		      CLUSTER_ID=${BOOTSTRAP_PEER_ID} \
		      CLUSTER_PRIVATEKEY=${BOOTSTRAP_PEER_PRIV_KEY} \
		      exec ipfs-cluster-service daemon --upgrade
		    else
		      BOOTSTRAP_ADDR=/dns4/${SVC_NAME}-0/tcp/9096/ipfs/${BOOTSTRAP_PEER_ID}
		
		      if [ -z $BOOTSTRAP_ADDR ]; then
		        exit 1
		      fi
		      # Only ipfs user can get here
		      exec ipfs-cluster-service daemon --upgrade --bootstrap $BOOTSTRAP_ADDR --leave
		    fi
		
		  configure-ipfs.sh: |
		    #!/bin/sh
		    set -e
		    set -x
		    user=ipfs
		   
		   # 这是k8s的自定义入口点，设计用于在生产场景的适当设置中运行ipfs节点。
		
		    mkdir -p /data/ipfs && chown -R ipfs /data/ipfs
		
		    if [ -f /data/ipfs/config ]; then
		      if [ -f /data/ipfs/repo.lock ]; then
		        rm /data/ipfs/repo.lock
		      fi
		      exit 0
		    fi
		
		    ipfs init --profile=badgerds,server
		    ipfs config Addresses.API /ip4/0.0.0.0/tcp/5001
		    ipfs config Addresses.Gateway /ip4/0.0.0.0/tcp/8080
		    ipfs config --json Swarm.ConnMgr.HighWater 2000
		    ipfs config --json Datastore.BloomFilterSize 1048576
		    ipfs config Datastore.StorageMax 100GB
- 公开 IPFS 集群端点

	我们的最后一步是定义 `Service` 将 IPFS 集群端点暴露给 `Pod`.

		apiVersion: v1
		kind: Service
		metadata:
		  name: ipfs-cluster
		  annotations:
		      external-dns.alpha.kubernetes.io/hostname: change.me.com
		  labels:
		    app: ipfs-cluster
		spec:
		  type: LoadBalancer
		  ports:
		    - name: swarm
		      targetPort: swarm
		      port: 4001
		    - name: swarm-udp
		      targetPort: swarm-udp
		      port: 4002
		    - name: ws
		      targetPort: ws
		      port: 8081
		    - name: http
		      targetPort: http
		      port: 8080
		    - name: api-http
		      targetPort: api-http
		      port: 9094
		    - name: proxy-http
		      targetPort: proxy-http
		      port: 9095
		    - name: cluster-swarm
		      targetPort: cluster-swarm
		      port: 9096
		  selector:
		    app: ipfs-cluster

	根据您设置 Kubernetes 集群的位置和方式，您可以使用 [ExternalDNS](https://github.com/kubernetes-incubator/external-dns) 注释 `external-dns.alpha.kubernetes.io/hostname`，它会自动获取提供的值，`change.me.com` 并在配置的 DNS 提供商（即 AWS Route53 或 Google）中创建 DNS 记录云DNS。

	另请注意，这些 `targetPort` 字段正在使用 `StatefulSe` 资源中 `PodSpec` 定义的命名端口。


