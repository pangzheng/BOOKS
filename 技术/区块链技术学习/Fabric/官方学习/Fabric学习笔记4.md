# Fabric 学习笔记 4 (Fabric设计)
## Fabric CA 为 Hyperledger Fabric 行使证书机构的功能
Fabric CA 提供以下功能：

- 身份注册，或者将连接到 LDAP 作为用户注册；
- 颁发登录证书(ECerts)；
- 颁发交易证书(TCerts)，保证链上交易的匿名性与不可连接性；
- 证书续期与撤销

Fabric CA 包含一个服务端组件和一个客户端组件
## 概述
下图说明了 Fabric CA 服务端如何在 Hyperledger Fabric 架构中发挥作用

![](./pic/fabric-ca.png)

有两种方式与 Fabric CA 服务端交互：通过 Fabric CA 客户端，或者 Fabric SDK，所有与 Fabric CA 的交互都是通过 REST APIs 来实现的。REST APIs 的 swagger 说明文档见 `fabric-ca/swagger/swagger-fabric-ca.json`

Fabric CA 客户端或者 SDK 可能会连接到 Fabric CA 集群中某一个 Fabric CA 服务端，这一部分可以通过上图右上部分获得更好的理解。客户端连接的是一个 HA 代理节点，这个 HA 代理节点为 Fabric CA 集群作负载均衡。所有的 Fabric CA 服务端共享同一个数据库。数据库用来保存用户和证书信息。如果配置了 LDAP，那么用户信息将会保存在 LDAP 中，而不是数据库中。

[CA部分忽略](https://hyperledgercn.github.io/hyperledgerDocs/ca-setup_zh/)

## Farbic CA 客户端
这一部分讲解如何使用 fabric-ca-client 的命令。

Fabric CA 客户端的根目录定义规则如下

- 如果存在环境变量 FABRIC_CA_CLIENT_HOME ，则使用这个值
- 否则，如果存在环境变量 FABRIC_CA_HOME ，则使用这个值
- 否则，如果存在环境变量 CA_CFG_PATH ，则使用这个值
- 否则，使用 $HOME/.fabric-ca-client

下面的指引假设客户端的配置文件存在于客户端根目录。

### 登陆启动用户
首先，如果需要在配置文件自定义CSR（证书签名请求），注意 `csr.cn` 必须设置为引导身份的 ID。默认 CSR 如下

	csr:
	    cn: <<enrollment ID>>
	    key:
	        algo: ecdsa
	        size: 256
	    names:
	        - C: US
	        ST: North Carolina
	        L:
	        O: Hyperledger Fabric
	        OU: Fabric CA
	    hosts:
	    - <<hostname of the fabric-ca-client>>
	    ca:
	        pathlen:
	        pathlenzero:
	        expiry:

查看字段的描述，CSR fields

然后运行 `fabric-ca-client enroll` 命令来登录一个身份。比如，下面的命令向一个本地运行在 7054 端口的 Fabric CA 服务端，登录了一个ID为 admin，password 为 adminpw 的身份。

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
	# fabric-ca-client enroll -u http://admin:adminpw@localhost:7054
登录命令会存储一个登录证书（ECert），相对应的私钥，还有 CA 证书链 PEM 文件。这些存储在 Fabric CA 客户端的 msp 目录的子目录下，你会看到信息提示 PEM 存储在哪里。

### 注册一个新的身份
只有已经登录了的身份才能发起注册的请求，而且必须有相应的权限来注册对应的身份类型。

特别地，注册时 Fabric CA 服务端做两项权限检查

1. 注册发起者的 `hf.Registrar.Roles` 属性中必须有请求注册的类型。举个例子，如果发起者的 `hf.Registrar.Roles` 属性的值为 `peer,app,user`，那么他能注册的类型为 peer，app 和 user，不能注册 orderer。
2. 发起者的 `affiliation` 必须与他请求注册的身份的 `affiliation` 相同，或者是所请求注 `affiliation` 的前缀。举个例子，一个 `affiliation`为 `a.b` 的发起者，可以注册一个 `affiliation` 为 `a.b.c` 的身份，但是不能注册一个`affiliation` 为 ``a.c`` 的身份。

下面的命令使用 admin 身份的凭证来注册一个新的身份，登录ID是 `admin2`，类型为``user`，affiliation` 为 ``org1.department1`` ，还有 ``hf.Revoker`` 属性为 ``true``。

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
	# fabric-ca-client register --id.name admin2 --id.type user --id.affiliation org1.department1 --id.attr hf.Revoker=true

密码会被打印出来，登录这个新注册的身份的时候，需要用到这个密码。这允许一个管理员注册身份，然后把这个身份的ID和密码给别人来登陆。

你可以通过编辑配置文件来设置注册时一些字段的默认值。举个例子，假设配置文件包含下面的内容：

	id:
	    name:
	    type: user
	    affiliation: org1.department1
	    attributes:
	        - name: hf.Revoker
	        value: true
	        - name: anotherAttrName
	        value: anotherAttrValue

下面的命令会注册一个新的身份，id为admin3，其他的内容会从配置文件中读取出来。包括：类型`user`，`affiliation`, `org1.department1`，还有两个属性，`hf.Revoker`和`anotherAttrName`。

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
	# fabric-ca-client register --id.name admin3

注册有多个属性的身份需要在配置文件中指明所有属性名和属性值，如上所示。

接下来，让我们注册一个节点身份，会在下面内容登陆节点的时候用到。下面的命令注册了一个 peer1 身份，在这里我们选择指明自己的密码，而不是由服务器生成。

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
	# fabric-ca-client register --id.name peer1 --id.type peer --id.affiliation org1.department1 --id.secret peer1pw
	
### 登录一个节点
现在你成功地注册了一个节点身份，你可以用ID和密码登陆。这部分与登陆一个引导身份类似。我们还会介绍如何使用“-M”选项来更换MSP的目录。

下面的命令登陆 peer1。记得在“-M”选项下更改你自己的MSP目录，MSP目录是由节点的 `core.yaml` 里的 `mspConfigPath` 指定的。你也可以设置`FABRIC_CA_CLIENT_HOME` 环境变量为 peer 的根目录。

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
	# fabric-ca-client enroll -u http://peer1:peer1pw@localhost:7054 -M $FABRIC_CA_CLIENT_HOME/msp	

登陆一个 orderer 也是一样的，除了 MSP 目录是设置在你的 orderer 的 `orderer.yaml` 文件里的 `LocalMSPDir`。

### 从另一个 Fabric CA 服务器获得 CA 证书链
通常，MSP目录的ca证书目录必须包含证书链，代表这个节点所有信任的信任中心。

`fabric-ca-client getcacerts` 命令用于从其他 Fabric CA 服务器实例获取这些证书链。

举个例子，下面的命令会在本地启动第二个Fabric CA 服务器，监听 7055 端口，命名为`CA2`。这代表两个由不同成员管理的分开的信任中心。

	# export FABRIC_CA_SERVER_HOME=$HOME/ca2
	# fabric-ca-server start -b admin:ca2pw -p 7055 -n CA2
下面的命令会把 CA2 的证书链安装进 peer1 的 MSP 目录。

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
	# fabric-ca-client getcacert -u http://localhost:7055 -M $FABRIC_CA_CLIENT_HOME/msp	

### 重新登陆一个身份
假设你的登陆证书快过期了，你可以重新登陆来替换你的登陆证书（ECert）。

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
	# fabric-ca-client reenroll

### 撤销一个证书或身份
身份和证书都能被撤销。撤销一个身份会撤销该身份拥有的所有证书，该身份也不能再获得新的证书。撤销一个证书会使该证书失效。

为了撤销一个证书或身份，发起者必须有 `hf.Revoker` 属性。发起者只能撤销与自己的`affiliation` 相同的证书或身份，或者发起者的 `affiliation` 是被撤销者的`affiliation` 的前缀。

举个例子，一个 `orgs.org1` 的发起者只能撤销 `orgs.org1` 或者`orgs.org1.department1` 的身份，而不能撤销 `orgs.org2` 的身份。

下面的命令撤销一个身份。将来所有发自该身份的请求都会被 Fabric CA 服务器拒收。

	fabric-ca-client revoke -e <enrollment_id> -r <reason>

下面是-r选项支持的理由：

- unspecified
- keycompromise
- cacompromise
- affiliationchange
- superseded
- cessationofoperation
- certificatehold
- removefromcrl
- privilegewithdrawn
- aacompromise

举个例子，有着根affiliation的admin可以回收peer1身份：

	# export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
	# fabric-ca-client revoke -e peer1
一个身份可以撤销自己的登陆证书（ECert），需要指定ECert的AKI和序列号：

	fabric-ca-client revoke -a xxx -s yyy -r <reason>
举个例子，你可以通过openssl命令来获取一个证书的AKI和序列号：

	serial=$(openssl x509 -in userecert.pem -serial -noout | cut -d "=" -f 2)
	aki=$(openssl x509 -in userecert.pem -text | awk '/keyid/ {gsub(/ *keyid:|:/,"",$1);print tolower($0)}')
	fabric-ca-client revoke -s $serial -a $aki -r affiliationchange

[CA部分忽略](https://hyperledgercn.github.io/hyperledgerDocs/ca-setup_zh/)

## 须知（Caveat emptor）
该文档假设读者已经基本了解如何去搭建 Kafka 集群和 ZooKeeper 集群。本文档的目的是确定您使用 Kafka 集群搭建一套 Hyperledger Fabric 排序服务节点集(OSNs)以及为你的区块链网络提供排序服务所需要采取的步骤。
### 概览（Big picture）
每一个通道(channel)在 Kafka 中被映射到一个单独的单分区(partition)类别(topic)。(译者注：通常每个Topic包含一个或多个Partition，此处每个Topic只包含一个Partition)

当排序节点通过 RPC 广播(Broadcast)接收到交易时，它会检查广播交易的客户端是否有权限去修改通道(channel)数据，然后反馈（即产生）这些交易到 Kafka 的适当分区(partition)中。

该分区也被排序节点所消费(consume)，排序节点将接收到的交易分组写入到本地区块，将其保留在本地账本中，并通过Deliver RPC提供给需要接收的客户端。
### 步骤（Steps）
设定变量 K 和 Z 分别是 Kafka 集群和 ZooKeeper 集群的节点数量

- `K` 的最小值需要是4。(我们将在步骤4中解释，这是实现故障容错(crash fault tolerance) 所需要的最小数值，也就是说， 4个节点可以容许1个节点宕机，所有的通道能够继续读写且可以创建通道。)(译者：Kafka节点被称为broker)
- `Z` 可以是3、5或者7。它必须是一个奇数来避免分裂(split-brain)情景，大于1以避免单点故障。 超过7个ZooKeeper服务器则被认为是多余的。

请按照以下步骤进行

- Orderers 操作
	1.  Kafka 相关信息被写在网络的初始区块中. 如果你使用 configtxgen 工具, 编辑 `configtx.yaml` 文件或者挑一个现成的系统通道的初始区块配置文件 – 其中:
		-  a. ``Orderer.OrdererType`` 字段被设置为 ``kafka``.
		-  b. ``Orderer.Kafka.Brokers`` 字段包含 *至少两个* Kafka集群中的节点``IP:port`` 样式的地址。这个列表没有必要详尽无遗(这些是你的 seed brokers.)
	2. 设置区块最大容量. 每一个区块最多只能有 `Orderer.AbsoluteMaxBytes` bytes 的容量(不含区块头信息), 这是一个你可以修改的值，存放在 `configtx.yaml` 配置文件中. 假设此处你设置的数值为 A,将此数字记下来 – 这会影响你在步骤4中对于 Kafka brokers 的配置.
	3. 使用 configtxgen 工具创建初始区块. 在步骤1和2中的设置是全局的设置, 也就是说这些设置的生效范围是网络中所有的排序节点. 记录下初始区块的位置.

- Kafka 集群: 
	- 适当配置你的Kafka集群. 确保每一个Kafka节点都配置了以下的值

			a. ``unclean.leader.election.enable = false`` -- Data consistency is key in
			a blockchain environment. We cannot have a channel leader chosen outside of
			the in-sync replica set, or we run the risk of overwriting the offsets that
			the previous leader produced, and --as a result-- rewrite the blockchain
			that the orderers produce.
			
			a. ``unclean.leader.election.enable = false`` -- 数据一致性是区块链环境的关键. 我们不能选择不在同步副本集中的channel leader, 也不能冒风险去覆盖前一leader所产生的偏移量, 那样的结果就是重写orderers所产生的区块链数据.
			
			b.  ``min.insync.replicas = M`` -- Where you pick a value ``M`` such that
			1 < M < N (see ``default.replication.factor`` below). Data is considered
			committed when it is written to at least ``M`` replicas (which are then
			considered in-sync and belong to the in-sync replica set, or ISR). In any
			other case, the write operation returns an error. Then:
			
			b.  ``min.insync.replicas = M`` --  ``M`` 的值需要满足
			1 < M < N (N的值参考后面的 ``default.replication.factor``). 数据被认为是完成提交当它被写入到至少 ``M`` 个副本中(也就是说它被认为是同步的,然后被写入到同步副本集中,也成为ISR). 其他情况, 写入操作返回错误信息. 然后:
			
			    i. If up to N-M replicas -- out of the N that the channel data is
			    written to -- become unavailable, operations proceed normally.
			    i. 如果有 N-M 个副本不可访问, 操作将正常进行.
			    ii. If more replicas become unavailable, Kafka cannot maintain an ISR
			    set of M, so it stops accepting writes. Reads work without issues.
			    The channel becomes writeable again when M replicas get in-sync.
			    ii. 如果更多副本不可访问, Kafka 不能位置数量 M 的同步副本集(ISR), 所以它会停止接受写入操作. 读操作可以正常运行.
			    当M个副本重新同步后,通道就可以再次变为可写入状态.
			
			
			c. ``default.replication.factor = N`` -- Where you pick a value ``N`` such
			that N < K. A replication factor of ``N`` means that each channel will have
			its data replicated to ``N`` brokers. These are the candidates for the ISR
			set of a channel. As we noted in the ``min.insync.replicas section`` above,
			not all of these brokers have to be available all the time. ``N`` should be
			set *strictly smaller* to ``K`` because channel creations cannot go forward
			if less than ``N`` brokers are up. So if you set N = K, a single broker
			going down means that no new channels can be created on the blockchain
			network -- the crash fault tolerance of the ordering service is
			non-existent.
			
			c. ``default.replication.factor = N`` -- 选择一个 ``N`` 的数值满足 N < K (Kafak集群数量). 参数 ``N`` 表示每个channel 的数据会复制到 ``N`` 个 broker 中. 这些是 channel 同步副本集的候选. 正如前面 ``min.insync.replicas`` 部分所说的, 不是所有broker都需要是随时可用的. ``N`` 值需要设置为绝对小于 ``K`` , 因为channel的创建需要不少于 ``N`` 个broker是启动的. 所以如果设置 N = K , 一个 broker 宕机就意味着区块链网络不能再创建channel. 那么故障容错的排序服务也就不存在了.
			
			
			d. ``message.max.bytes`` and ``replica.fetch.max.bytes`` should be set to a
			value larger than ``A``, the value you picked in
			``Orderer.AbsoluteMaxBytes`` in Step 2 above. Add some buffer to account for
			headers -- 1 MiB is more than enough. The following condition applies:
			
			d. ``message.max.bytes`` 和 ``replica.fetch.max.bytes`` 的值需要大于 ``A``, 就是在步骤2中选取的 ``Orderer.AbsoluteMaxBytes`` 的值. 再为区块头增加一些余量 -- 1 MiB 就足够了. 需要满足以下条件:
			
			::
			
			    Orderer.AbsoluteMaxBytes < replica.fetch.max.bytes <= message.max.bytes
			
			(For completeness, we note that ``message.max.bytes`` should be strictly
			smaller to ``socket.request.max.bytes`` which is set by default to 100 MiB.
			If you wish to have blocks larger than 100 MiB you will need to edit the
			hard-coded value in ``brokerConfig.Producer.MaxMessageBytes`` in
			``fabric/orderer/kafka/config.go`` and rebuild the binary from source.
			This is not advisable.)
			
			(补充, 我们注意到 ``message.max.bytes`` 需要严格小于 ``socket.request.max.bytes`` , 这个值默认是100Mib. 如果你希望区块大于100MiB, 你需要去修改硬代码中的变量 ``brokerConfig.Producer.MaxMessageBytes`` , 代码位置是 ``fabric/orderer/kafka/config.go`` , 再重新编译代码, 不建议这么做.)
			
			e. ``log.retention.ms = -1``. Until the ordering service adds
			support for pruning of the Kafka logs, you should disable time-based
			retention and prevent segments from expiring. (Size-based retention -- see
			``log.retention.bytes`` -- is disabled by default in Kafka at the time of
			this writing, so there's no need to set it explicitly.)
			
			e. ``log.retention.ms = -1``. 直到排序服务增加了对于 Kafka 日志分割(pruning)的支持之前, 应该禁用基于时间分割的方式以避免单个日志文件到期分段. (基于文件大小的分割方式 -- 看参数 ``log.retention.bytes`` -- 在本文书写时, 在 Kafka 中是默认被禁用的, 所以这个值没有必要指定地很明确. )
			
			Based on what we've described above, the minimum allowed values for ``M``
			and ``N`` are 2 and 3 respectively. This configuration allows for the
			creation of new channels to go forward, and for all channels to continue to
			be writeable.
			
			基于上文所描述的, ``M`` 和 ``N`` 的最小值分别为 2 和 3 . 这个配置可以创建 channel 并让所有 channel 都是随时可以写入的.

- Orderers

	将所有排序节点指向初始区块. 编辑 `orderer.yaml` 文件中的参数 `General.GenesisFile` 使其指向步骤3中所创建的初始区块. (同时, 确保YAML文件中所有其他参数都是正确的.)
	
	- 调整轮询间隔和超时时间. (可选步骤.)	
		1. orderer.yaml 文件中的 Kafka.Retry 区域让你能够调整 metadata/producer/consumer 请求的频率以及socket的超时时间. (这些应该就是所有在 kafka 的生产者和消费者 中你需要的设置)
		2. 另外, 当一个 channel 被创建, 或当一个现有的 channel 被重新读取(刚启动 orderer 的情况), orderer 通过以下方式和 Kafka 集群进行交互.

				a. 为 channel 对应的 Kafka 分区 创建一个 Kafka 生产者.
				b. 通过生产者向这个分区发一个空的连接信息.
				c. 为这个分区创建一个 Kafka 消费者.
				
				如果任意步骤出错, 你可以调整其重复的频率. 
				这些步骤会在每一个 Kafka.Retry.ShortInterval 指定的时间间隔后进行重试 Kafka.Retry.ShortTotal 次, 
				再以 Kafka.Retry.LongInterval 规定的时间间隔重试 Kafka.Retry.LongTotal 次直到成功. 
				需要注意的是 orderer 不能读写该 channel 的数据直到所有上述步骤都成功执行.	
	- 将排序节点和 Kafka 集群间设置为通过 SSL 通讯. (可选步骤,强烈推荐) 参考 the Confluent guide
<http://docs.confluent.io/2.0.0/kafka/ssl.html>_ 文档中关于 Kafka 集群的设置, 来设置每个排序节点 orderer.yaml 文件中 Kafka.TLS 部分的内容.

启动节点请按照以下顺序: ZooKeeper 集群, Kafka 集群, 排序节点
### 其他注意事项（Additional considerations）
首选的消息大小. 在上面的步骤2中, 你也能通过参数 
`Orderer.Batchsize.PreferredMaxBytes` 设置首选的区块大小. Kafka 处理相对较小的信息有更高的吞吐量; 针对小于 1 MiB 大小的值.

使用环境变量重写设置. 你能够通过设置环境变量来重写 Kafka 节点和 Zookeeper 服务器的设置. 替换配置参数中的点为下划线

- 例如 `KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false` 环境变量重写配置参数 `unclean.leader.election.enable.` 环境变量重写同样适用于排序节点的本地配置, 即 `orderer.yaml` 中所能设置的. 例如 `ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s` 环境变量可以重写本地配置文件中的 `Orderer.Kafka.Retry.ShortInterval.`

[其他省略](https://hyperledgercn.github.io/hyperledgerDocs/kafka_zh/)
## Channels
在超级账本 Fabric 中，一个通道是指一个在两个或多个特定网络成员间的专门为私人的和机密的交易为目的而建立的私有”子网”。

一个通道的定义中包含：

- 成员(组织)
- 每个成员的锚节点
- 共享帐本
- 链上代码应用程序
- 排序服务节点

网络上的每个交易都在一个指定的通道中执行，在通道中交易必须通过通道中的每部分的认证和授权。要加入一个通道的每个节点都必须有自己的通过成员服务提供商（MSP）获得的身份标识，用于鉴定每个节点在通道中的是什么节点和服务。

要创建一个通道，客户端 SDK 调用配置系统链上代码和参考属性，比如锚节点和成员（组织）。这个请求会为通道的账本创建一个创世区块，用于存储关于通道的策略，成员和锚节点的配置信息。当需要添加一个新成员到现有通道时，这个创世区块，或者最新的新配置区块（如果可用），将会共享给这个新成员。

从通道的所有节点中选举出的领导节点决定哪个节点用于代表其他成员节点与排序服务通讯。如果还没有领导节点，那么一个算法可以用于标识出领导节点。共识服务对交易进行排序，并打包成区块，发送区块给每个领导节点，然后领导节点把区块分发给其成员节点，然后使用 gossip 协议穿过通道。

虽然任意一个锚节点都可以属于多个通道，而且维护了多个账本，但是不会有任何账本数据会从一个通道传到另一个通道。这就是根据通道对账本的分离，这种分离是在配置链上代码，成员标识服务和 gossip 传播协议中定义和实现。数据的传播，包括交易的信息，账本状态和通道成员等都在通道内受限制的验证成员身份的节点之间。这种根据通道对节点和账本数据进行隔离，允许网络成员可以在同一个区块链网络中请求私有的和保密的交易给业务上的竞争对手和其他受限的成员。


## 参考
[排序服务](https://hyperledgercn.github.io/hyperledgerDocs/kafka_zh/)
[Channels](https://hyperledgercn.github.io/hyperledgerDocs/channels_zh/)			