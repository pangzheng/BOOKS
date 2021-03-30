# ceph 
## 基础概念
### 什么是 Ceph
- 简介

	利用一个分布式集群来提供对象，块和文件存储的统一存储平台。Ceph 是一种基于软件的分布式存储解决方案，并在“商品硬件”上运行。该系统被设计成自动修复和智能管理，最初是 SageWeil 一项关于存储系统的 PhD 研究项目(2004年发表)。
- 目标

	没有单点故障的完全分布式存储系统，使数据能容错和无缝的复制，可扩展EB水平(EB,PB,TB,GB)，并且降低成本。
- 特点
	- 高性能
		-  ceph 摒弃了从元数据中寻址的方案，采用 CRUSH 算法，数据分布均衡，并行度高。
		-  考虑了容灾域的隔离，可以实现各类的副本分布规则，如跨机房、机架感知等。
		-  能够支持上千个存储节点的规模，支持TB到EB级的数据。
	- 高可用性
		- 副本数可以灵活控制
		- 支持故障域分隔，数据强一致性
		- 多种故障场景自动进行修复自愈
		- 没有单点故障，自动管理
	- 高可扩展性
		- 去中心化
		- 扩展灵活
		- 随着节点增加而线性增长
	- 特性丰富
		- 支持三种存储接口：块存储、文件存储、对象存储
		- 支持自定义接口，支持多种语言驱动。


### Ceph 架构设计
Ceph 通过 RADOS 组件，在核心层实现了分布式存储的能力，然后在客户端提供三种存储方式来使用它。
生态系统可以大致划分为四部分：

![](./pic/ceph-top.png)
### 存储方式的基本概念

1. Filesystem
	- 典型设备(磁盘阵列，硬盘)

		主要是将裸磁盘空间映射给主机使用的。
	- 优点
		- 通过Raid与LVM等手段，对数据提供了保护
		- 多块廉价的硬盘组合起来，提高容量
		- 多块磁盘组合出来的逻辑盘，提升读写效率
	- 缺点
		- 采用SAN架构组网时，光纤交换机，造价成本高
		- 主机之间无法共享数据。
	- 使用场景
		- 操作系统存储 
		- docker容器、虚拟机磁盘存储分配
		- 日志存储
		- 数据库存储
		- 文件存储
			…
2. Block Device
	- 典型设备(FTP、NFS服务器)

		为了克服块存储文件无法共享的问题，所以有了文件存储。
在服务器上架设FTP与NFS服务，就是文件存储。
	- 优点
		- 造价低，随便一台机器就可以了
		- 方便文件共享
	- 缺点
		- 读写速率低
		- 传输速率慢
	- 使用场景
		- 日志存储
		- 有目录结构的文件存储
			…
3. Object Storage
	- 典型设备(内置大容量硬盘的分布式服务器swift, s3)

		多台服务器内置大容量硬盘，安装上对象存储管理软件，对外提供读写访问功能
	- 优点
		- 具备块存储的读写高速
		- 具备文件存储的共享等特性
	- 使用场景(适合更新变动较少的数据)
		- 图片存储
		- 视频存储
			…

### ceph 核心概念
1. client 

	客户端使用元数据服务器，执行元数据操作（来确定数据位置）
2. metadata server

	MDS 元数据服务器, Ceph FS 依赖的元数据服务，管理数据位置，以及在何处存储新数据。
3. Object Storage Device

	OSD 全称对象存储设备，也就是负责响应客户端请求返回具体数据的进程。一个 Ceph 集群一般都有很多个 OSD，最少2个
4. monitor

	一个 Ceph 集群需要多个 Monitor 组成的小集群，它们通过 Paxos 同步数据，用来保存 OSD 的元数据。
5. Object

	Ceph 最底层的存储单元，每个 Object 均包含元数据和原始数据
6. Placement Grouops

	PG 安置组是一个逻辑的概念，一个 PG 包含多个 OSD(副本)。引入 PG 这一层为了更好的分配数据和定位数据
7. Reliable Autonomic Distributed Object Store

	RADOS 可靠的自主分布式对象存储协议 ，Ceph 的核心，实现数据分配、Failover 等集群操作。
8. Libradio

	Librados 是 Rados 提供库，因为 RADOS 是协议很难直接访问，因此上层的 RBD、RGW 和 CephFS 都是通过 librados 访问的，目前提供 PHP、Ruby、Java、Python、C 和 C++ 支持
9. CRUSH

	CRUSH 是 Ceph 使用的数据分布算法，类似一致性哈希，让数据下发的调度算法
10. RADOS block device(块存储)

	RBD全称RADOS 块设备，是 Ceph 对外提供的块设备服务。块设备将信息存储在固定大小的块中，每个块都有自己的地址。数据块的大小通常在 512 - 32768 字节之间。块设备的基本特征是每个块都能独立于其它块而读写。磁盘是最常见的块设备。
11. RADOS gateway(对象存储)

	RGW 全称 RADOS 网关，是 Ceph 对外提供的对象存储服务，接口与 S3 和 Swift 兼容。对象存储系统（Object-Based StorageSystem）是综合了 NAS(Network Storage Technologies) 和 SAN(Storage AreaNetwork) 的优点，同时提供高速直接访问(san)和数据共享(nas)等优势，提供了高可靠性、跨平台性以及安全的数据共享的存储体系结构。
12. Ceph File System（文件存储）

	CephFS 是 Ceph 对外提供的文件系统服务，一个 POSIX 兼容的文件系统,使用 RADOS 集群存储其数据,使用存储设备与块设备存储和对象存储相同,至少需要配置一个 MDS
13. Cluster Map

	RADOS 的核心数据结构，指定了 OSDs 和数据分布信息，在 monitor 上存有最新副本，依靠 epoch 增加来维护及时更新增量信息。数据包括

	1. Monitor Map

		包括集群的 FSID、位置、名字地址和每个监控器的端口
	2. OSD Map

		包括集群的 FSID、pool 列表、副本信息、pg号码、osd 列表和它们的状态。
	3. PG Map

		包括 pg 版本、时间戳、最后一个 osd map 的 epoch，全部比率、每个pg的相信信息(pgid、Up Set、 Acting Set、pg 状态和每个 pool 的数据统计信息)
	4. CRUSH Map

		包括设备列表、故障域层次结构(硬盘、主机、机架、机架所在行、所在机房房间信息等)和存储数据时遍历层次结构的规划
	5. MDS Map

		MDS map epoch、用于存储元数据的pool、元数据服务器列表、元数据服务器启动状态。
14. pools

	Pool是 ceph 存储数据时的逻辑分区，它起到 namespace 的作用。其他分布式存储系统，比如 Mogilefs、Couchbase、Swift 都有 pool 的概念，只是叫法不同。每个 pool 包含一定数量的 PG，PG 里的对象被映射到不同的 OSD 上，因此 pool 是分布到整个集群的。
	
	pool 有两种方法增强数据的可用性，可以设置其中之一，默认是副本。
	
	- 一种是副本(replicas)
	- 一种是EC(erasure coding 纠删码）

### PG
PG 就是目录

- PG 中写入一个数据
	- 在生成 pool 的时候，会对应生成一个 ID 号
	- 然后系统将会用这个 Pool ID 号生成规则在所有的 OSD 生成对应的所有 PG
	- 每一个 OSD ，对应的 `current ` 目录中都保存了部分 PG
		- 一个 Pool 可以对应很多以 Pool ID 起名的 PG
		- 这些 PG 分布在不同的 OSD 下面
		- 比如 `0.xxx_head` 目录等于一个 pool 下面对应的一个 PG 目录
	- 写入一个文件，通过算法命中一个 PG，这个PG按照副本设置和规则会分配到不同的主机 OSD，默认3副本就分配到3个相同 PG 上 ,比如在 osd4.6.7 的0.44_head PG目录写入数据 
- PG 异常

	Ceph 系统默认会设计副本数和最小副本数，在 OSD 异常引起 PG 异常而导致副本异常
	
	- 在不超过最小副本数时，ceph 会标注异常 OSD 所有的 PG 为 `degraded ` ，但该数据还可以在集群正常读写。 
	- 在 PG 副本数小于设置最小副本数时，异常的 PG 就会被标注  `peered`, 比如 3副本故障了2个 `min_size=2`，那么集群即使存在1个副本，也不会对外部响应了。用来提醒集群管理员，数据已经异常了。需要集群管理员将 `min_size` 设置为1才可以响应。
		- `peered ` 表示该 PG 数据已经极度危险，需要恢复 
- PG 异常恢复

	- OSD 异常
	
		有一个参数 `mon_osd_down_out_interval = 300` 该参数表示如果 osd 在5分钟内没有恢复，那么该 OSD 就不属于集群，该 OSD 上的所有 PG 将 remap 到其他 OSD 上，根据算法被选中的 OSD 会创建对应数量的 PG 的目录，目录创建完后，才会根据算法从数据存在的 OSD 上取回 PG 对应的数据，这会这个 PG 会标注为 `backfilling` , 当一个 PG 处于 `remapped+backfilling` 时，该 PG 是数据恢复中。
	- OSD 正常 PG 异常

		OSD 正常，但是 PG 因为某种原因异常，通过主动扫描命令 ` scrub` PG 可以发现该 PG 状态多了一个  `inconsistent ` ，可以通过命令修复 PG ，如 `ceph pg repair 0.44`
	- OSD 异常导致 PG 数据不同步

		这块也是会在 OSD 上线后，自动同步更新的数据			
 
### CRUSH 算法
#### 为什么要有 CRUSH
需要有一个统一的数据寻址算法 Locator,其中 `Locator ID` 是数据的唯一标识符，输出 Device 列表是一系列存储设备,（多设备冗余以达到多份数据保护或切分提高并发等效果),早期的直观方案是维护一张全局的 Key-Value 表，随着数据量的增多和集群规模的扩大，要在整个系统中维护这么一张不断扩大的表变得越来越困难,CRUSH 就是为了解决这个问题。 

	 Locator(ID) -> [Device_1, Device_2, Device_3, ...]
#### 算法说明
- CRUSH 算法的全称为：Controlled Scalable Decentralized Placement of Replicated Data，可控的、可扩展的、分布式的副本数据放置算法
- PG 到 OSD 的映射的过程算法叫做 CRUSH 算法。(一个 Object 需要保存三个副本，也就是需要保存在三个 osd 上)
- CRUSH 算法是一个伪随机的过程，它可以从所有的 OSD 中，随机性选择一个 OSD 集合，但是同一个 PG 每次随机选择的结果是不变的，也就是映射的 OSD 集合是固定的

#### CRUSH 需要解决的问题
- 数据分布和负载均衡
	1. 数据分布均衡，使数据能均匀的分布到各个节点上
	2. 负载均衡，使数据访问读写操作的负载在各个节点和磁盘的负载均衡(?读写)
- 灵活应对集群伸缩
	1. 系统可以方便的增加或者删除节点设备，并且对节点失效进行处理
	2. 增加或者删除节点设备后，能自动实现数据的均衡，并且尽可能少的迁移数据
- 支持大规模集群
	1. 要求数据分布算法维护的元数据相对较小，并且计算量不能太大。随着集群规模的增加，数据分布算法开销相对比较小

#### Ceph CRUSH 算法原理
- CRUSH 算法因子
	- 层次化的 Cluster Map

		反映了存储系统层级的物理拓扑结构。定义了 OSD 集群具有层级关系的静态拓扑结构。OSD 层级使得 CRUSH 算法在选择 OSD 时实现了机架感知能力，也就是通过规则定义，使得副本可以分布在不同的机架、不同的机房中、提供数据的安全性。
		
		![](./pic/cluster-map1.png)
	
		CRUSH Map 是一个树形结构，OSDMap 更多记录的是 OSDMap 的属性(epoch/fsid/pool信息以及osd 的 ip 等等)。
		
			# types
			type 0 osd
			type 1 host
			type 2 chassis
			type 3 rack
			type 4 row
			type 5 pdu
			type 6 pod
			type 7 room
			type 8 datacenter
			type 9 region
			type 10 root
	
		- root 节点(最上层)

			树形结构只有一个最终的根节点称之为 root 节点
		- bucket 节点(中间层)

			bucket 是虚构的节点，根据物理结构进行抽象是分级，创建桶分级结构的目的是按故障域隔离叶节点，像主机、机箱、机柜、电力分配单元、机群、行、房间、和数据中心。除了表示叶节点的 OSD ，其它分级结构都是任意的，要把归置组映射到跨故障域的 OSD
			
			推荐 CRUSH 图内的命名符合公司的硬件命名规则，并且采用反映物理硬件的例程名。良好的命名可简化集群管理和故障排除，当 OSD 和/或其它硬件出问题时，管理员可轻易找到对应物理硬件。
			
			但是在 RADOS 网关接口的术语中，它又是不同的概念可以是
			
			- 数据区(region)
			- 数据中心(datacenter)
			- 房间(room)
			- 集群(pod)
			- 电力分配单元(pdu)
			- 行(raw)
			- 机架(rack)
			- 机箱(chassis）
			- 主机(host)

			Bucket 随机算法类型
			
			- 映射速度说明和添加或删除项目时不同桶类型的数据重组
	
				![](./pic/ceph-bucket1.png)
			
				- uniform
				
					这种桶用完全相同的权重汇聚设备,适合所有子节点权重相同，而且很少添加删除 item，时间复杂度最低
				- list	
					
					列表桶 ,适用于集群扩展类型。增加 item，产生最优的数据移动，查找 item，这种桶适合永不或极少缩减的场景，然而，如果从链表的中间或末尾删除了一些条目，将会导致大量没必要的挪动。时间复杂度 O(n)。
				- tree 
				
					二进制搜索树桶 ,在桶包含大量条目时比 list 桶更高效，添加删除叶子节点时，其他节点 node_id 不变,查找时间复杂度 O(log n)，这使得它们更适合管理更大规模的设备或嵌套桶。
				- straw 
			
					list 和 tree 桶用分而治之策略，给特定条目一定优先级（如位于链表开头的条目）、或避开对整个子树上所有条目的考虑。这样提升了副本归置进程的性能，但是也导致了重新组织时的次优结果，如增加、拆除、或重设某条目的权重。 straw 桶类型允许所有条目模拟拉稻草的过程公平地相互“竞争”副本归置
		- 叶子节点
	
			最终的 OSD
	- 数据分布策略 Placement Rules

		决定了一个 PG 的对象副本如何选择的规则，用户可以自定义设置副本在集群中的分布。Placement Rules 主要有特点
		
		- 从 CRUSH Map 中的哪个节点开始查找
		- 使用那个节点作为故障隔离域
		- 定位副本的搜索模式
			- 广度优先
			- 深度优先
			
		配置规则
		
			rule replicated_ruleset  	  # 规则集的命名，创建 pool 时可以指定 rule 集
			{
			    ruleset 0                # rules 集的编号，顺序编即可   
			    type replicated          # 定义 pool 类型为 replicated or erasure 模式  
			    min_size 1               # pool 中最小指定的副本数量不能小 1
			    max_size 10              # pool 中最大指定的副本数量不能大于 10       
			    step take default        # 查找 bucket 入口点，也就是 bucket 规则的算法起点，默认是 root 类型的 bucket    
			    step chooseleaf  firstn  0  type  host # 选择数据副本故障域进行切割，并递归选择叶子节点 osd。(默认是 OSD，这个测试可以但是生产明显不行,)     
			    step emit        		   # 结束
			}
	

## 参考
- [Ceph介绍及原理架构分享](https://www.jianshu.com/p/cc3ece850433)
- [《 大话 Ceph 》](https://cloud.tencent.com/developer/article/1006292)
- [ceph 官方](https://docs.ceph.com/)
	- 计算 PG 的 ID
		1. Client 输入 pool ID 和对象 ID（如pool=‘liverpool’，object-id=‘john’）
		2. CRUSH 获得对象 ID 并对其 hash
		3. CRUSH 计算 OSD 个数 hash 取模获得 PG 的 ID（如 0x58）
		4. CRUSH 获得已命名 pool 的 ID（如 liverpool=4）
		5. CRUSH 预先考虑到 pool ID 相同的 PG ID（如4.0x58）