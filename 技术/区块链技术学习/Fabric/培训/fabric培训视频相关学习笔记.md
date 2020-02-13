# fabric 命令整理(视频培训)
## 操作步骤
1. 生成证书(cryptogen)
- 创建通道

## cryptogen 命令(测试生成证书用)
cryptogen 用来生成 hyperledger fabric 的密钥，主要用于测试中的网络预配置 (通常不会在生产网络中使用，而使用另外一种证书管理系统，fabric ca）

cryptogen 需要⼀个 `crypto-config.yaml` 配置⽂件作为参数，它包含需要生成证书的所有服务，如⽹络、组织的组件，并⽣成⼀组证书和密钥。每个组织都有⼀个唯⼀的根证书（ca-cert），它将特定组件（peer和order service）绑定到该组织。通过为每个组织分配唯⼀的CA证书，模仿⼀个典型的⽹络，参与的成员将使⽤其⾃⼰的证书颁发机构。Fabric 中的交易和通信均由私钥 （keystore）进行签名，然后可通过公钥进⾏验证。

- 编译

		$ cd $GOPATH/src/github.com/hyperledger/fabric 
		$ make cryptogen 
		$ cp build/bin/cryptogen /usr/bin

### cryptogen help
显示 cryptogen 的帮助信息。

- 使用方法

		usage: cryptogen [<flags>] <command> [<args> ...]
		
		Utility for generating Hyperledger Fabric key material
		
		Flags:
		  --help  Show context-sensitive help (also try --help-long and --help-man).
		
		Commands:
		  help [<command>...]
		    Show help.
		
		  generate [<flags>]
		    Generate key material
		
		  showtemplate
		    Show the default configuration template
		
		  version
		    Show version information
		
		  extend [<flags>]
		    Extend existing network

### cryptogen version
显示cryptogen软件版本信息。

- 使用方法

		~$ cryptogen version
- 命令标志

		--help  Show context-sensitive help (also try --help-long and --help-man).

### cryptogen showtemplate
显示代码默认配置,也可以从这里获取 [crypto-config.yaml](https://github.com/hyperledger/fabric-samples/blob/master/first-network/crypto-config.yaml) 

- 使用方法

		~$ cryptogen showtemplate
- 命令标志

		--help  Show context-sensitive help (also try --help-long and --help-man).
- 执行完毕现实结果				

		# ---------------------------------------------------------------------------
		# "OrdererOrgs" - Definition of organizations managing orderer nodes
		# ---------------------------------------------------------------------------
		OrdererOrgs:
		  # ---------------------------------------------------------------------------
		  # Orderer
		  # ---------------------------------------------------------------------------
		  - Name: Orderer
		    Domain: example.com
		    EnableNodeOUs: false
		    # ---------------------------------------------------------------------------
		    # "Specs" - See PeerOrgs below for complete description
		    # ---------------------------------------------------------------------------
		    Specs:
		      - Hostname: orderer
		# ---------------------------------------------------------------------------
		# "PeerOrgs" - Definition of organizations managing peer nodes
		# ---------------------------------------------------------------------------
		PeerOrgs:
		  # ---------------------------------------------------------------------------
		  # Org1
		  # ---------------------------------------------------------------------------
		  - Name: Org1
		    Domain: org1.example.com
		    EnableNodeOUs: false
		    # ---------------------------------------------------------------------------
		    # "CA"
		    # ---------------------------------------------------------------------------
		    # Uncomment this section to enable the explicit definition of the CA for this
		    # organization.  This entry is a Spec.  See "Specs" section below for details.
		    # ---------------------------------------------------------------------------
		    # CA:
		    #    Hostname: ca # implicitly ca.org1.example.com
		    #    Country: US
		    #    Province: California
		    #    Locality: San Francisco
		    #    OrganizationalUnit: Hyperledger Fabric
		    #    StreetAddress: address for org # default nil
		    #    PostalCode: postalCode for org # default nil
		    # ---------------------------------------------------------------------------
		    # "Specs"
		    # ---------------------------------------------------------------------------
		    # Uncomment this section to enable the explicit definition of hosts in your
		    # configuration.  Most users will want to use Template, below
		    #
		    # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
		    #   - Hostname:   (Required) The desired hostname, sans the domain.
		    #   - CommonName: (Optional) Specifies the template or explicit override for
		    #                 the CN.  By default, this is the template:
		    #
		    #                              "{{.Hostname}}.{{.Domain}}"
		    #
		    #                 which obtains its values from the Spec.Hostname and
		    #                 Org.Domain, respectively.
		    #   - SANS:       (Optional) Specifies one or more Subject Alternative Names
		    #                 to be set in the resulting x509. Accepts template
		    #                 variables {{.Hostname}}, {{.Domain}}, {{.CommonName}}. IP
		    #                 addresses provided here will be properly recognized. Other
		    #                 values will be taken as DNS names.
		    #                 NOTE: Two implicit entries are created for you:
		    #                     - {{ .CommonName }}
		    #                     - {{ .Hostname }}
		    # ---------------------------------------------------------------------------
		    # Specs:
		    #   - Hostname: foo # implicitly "foo.org1.example.com"
		    #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
		    #     SANS:
		    #       - "bar.{{.Domain}}"
		    #       - "altfoo.{{.Domain}}"
		    #       - "{{.Hostname}}.org6.net"
		    #       - 172.16.10.31
		    #   - Hostname: bar
		    #   - Hostname: baz
		    # ---------------------------------------------------------------------------
		    # "Template"
		    # ---------------------------------------------------------------------------
		    # Allows for the definition of 1 or more hosts that are created sequentially
		    # from a template. By default, this looks like "peer%d" from 0 to Count-1.
		    # You may override the number of nodes (Count), the starting index (Start)
		    # or the template used to construct the name (Hostname).
		    #
		    # Note: Template and Specs are not mutually exclusive.  You may define both
		    # sections and the aggregate nodes will be created for you.  Take care with
		    # name collisions
		    # ---------------------------------------------------------------------------
		    Template:
		      Count: 1
		      # Start: 5
		      # Hostname: {{.Prefix}}{{.Index}} # default
		      # SANS:
		      #   - "{{.Hostname}}.alt.{{.Domain}}"
		    # ---------------------------------------------------------------------------
		    # "Users"
		    # ---------------------------------------------------------------------------
		    # Count: The number of user accounts _in addition_ to Admin
		    # ---------------------------------------------------------------------------
		    Users:
		      Count: 1
		  # ---------------------------------------------------------------------------
		  # Org2: See "Org1" for full specification
		  # ---------------------------------------------------------------------------
		  - Name: Org2
		    Domain: org2.example.com
		    EnableNodeOUs: false
		    Template:
		      Count: 1
		    Users:
		      Count: 1
	- 过滤后

			OrdererOrgs:  # 排序组织
			  - Name: Orderer
			    Domain: example.com
			    EnableNodeOUs: false
			    Specs:
			      - Hostname: orderer
			PeerOrgs: # peer 组织
			  - Name: Org1 #peer 组织1
			    Domain: org1.example.com
			    EnableNodeOUs: false
			    Template: # peer 数量
			      Count: 1  #除了 Count 还可以增加其他参数，参考模版
			    Users: # 真正使用系统的用户数量，注意这里不包含 admin
			      Count: 1 
			  - Name: Org2  #peer 组织2
			    Domain: org2.example.com
			    EnableNodeOUs: false
			    Template:
			      Count: 1
			    Users:
			      Count: 1  
			          		
- 命令语法

	cryptogen 命令包含5个子命令，列举如下：

	- help
	- generate
	- showtemplate
	- extend
	- version

### cryptogen generate
生成密钥文件

- 使用方法

		~$ cryptogen generate [<flags>]
		命令标志：
		
		  --help                    Show context-sensitive help (also try --help-long
		                            and --help-man).
		  --output="crypto-config"  The output directory in which to place artifacts
		  --config=CONFIG           The configuration template to use
- 示例代码
	
	下面的命令将生成一整套 fabric 测试网络使用证书
	
		cryptogen generate --config=./crypto-config.yaml
	- 生成完毕后的证书目录，crypto-config ⽬录下⽣成⽂件

			tree -L 2 crypto-config crypto-config 
			├── ordererOrganizations 
				 └── example.com 
			└── peerOrganizations
				├── org1.example.com 
				└── org2.example.com
	- 每⼀个 org ⽣成⼀个⽬录， org ⽬录下⾯的⽂件组织结构一致，如org1.example.com

			tree -L 2 crypto-config/peerOrganizations/org1.example.com/ crypto-config/peerOrganizations/org1.example.com/ 
			├── ca # 包含org的根证书和Key 
				├── ca.org1.example.com-cert.pem 
			 	└── priv_sk 
			├── msp # 包含org的根msp信息
				├── admincerts 
				├── cacerts
				└── tlscacerts 
			├── peers # 包含org下⾯的所有peer的证书 
				├── peer0.org1.example.com 
				└── peer1.org1.example.com 
			├── tlsca # 包含org的根tlsca证书和key 
				├── priv_sk 
				└── tlsca.org1.example.com-cert.pem 
			└── users # 包含org下⾯所有⽤户的证书,配置⽂件指定1个⽤户
				├── Admin@org1.example.com #管理员 
				└── User1@org1.example.com
	- users 下证书

			tree cryptoconfig/peerOrganizations/org2.example.com/users/User1\@org2.example .com/ cryptoconfig/peerOrganizations/org2.example.com/users/User1@org2.example. com/ 
			├── msp 
			│ ├── admincerts 
			│ │ └── User1@org2.example.com-cert.pem # 同org的admin证书 
			│ ├── cacerts 
			│ │ └── ca.org2.example.com-cert.pem # 同org的ca根证书 
			│ ├── keystore │ 
			│ └── priv_sk 
			│ ├── signcerts │ 
			│ └── User1@org2.example.com-cert.pem # User2的MSP证书 
			│ └── tlscacerts 
			│ └── tlsca.org2.example.com-cert.pem # 同org的tlsca根证书 
			└── tls
				├── ca.crt # 同org的tlsca根证书 
				├── client.crt # User2的 tls证书 
				└── client.key # User2的 tls key
### cryptogen extend
扩展已有的网络，扩展 fabric 系统或用户时，新增证书如何生成  

- 使用方法

		~$ cryptogen extend [<flags>]
- 命令标志

		  --help                   Show context-sensitive help (also try --help-long and
		                           --help-man).
		  --input="crypto-config"  The input directory in which existing network place
		  --config=CONFIG          The configuration template to use
- 示例代码

	下面的示例使用 `cryptogen extend` 命令扩展 crypto-config 中声明的网络：

		~$ cryptogen extend --input="crypto-config" --config=config.yaml

	上面的命令修改原有 `config.yaml`  添加了一个新的机构 `org3.example.com`
	
		OrdererOrgs:
			- Name: Orderer 
				Domain: example.com 
				Specs:
					- Hostname: orderer 
		PeerOrgs:
			- Name: Org1 
				Domain: org1.example.com 
				Template:
					Count: 1 
				Users:
					Count: 1
			- Name: Org2 
				Domain: org2.example.com 
				Template:
					Count: 1 
				Users:
					Count: 1
			- Name: Org3 Domain: org3.example.com
				Template:
					Count: 2 
				Users:
					Count: 2
	查看目录结构
	
		$tree -L 2 crypto-config crypto-config 
		├── ordererOrganizations 
		│ └── example.com 
		└── peerOrganizations
			├── org1.example.com 
			├── org2.example.com 
			└── org3.example.com	
	注意这里只新增了组织，新增其他如用户逻辑一样

### configtxgen(配置文件)
configtxgen 命令用来`创建`或`查看`通道配置等相关操作。生成的构件内容取决于 `configtx.yaml` 文件，主要有3个作用

1. 排序服务节点使⽤的创世区块
- 创建通道使⽤的通道配置
- 创建锚节点配置

编译

	$ cd $GOPATH/src/github.com/hyperledger/fabric
	$ make configtxgen 
	$ cp build/bin/cryptogen /usr/bin

- 命令语法

	configtxgen 没有子命令，使用标志来完成不同的任务

		~$ configtxgen [flag]
- 可用标志

		  -asOrg string
		        Performs the config generation as a particular organization (by name), only including values in the write set that org (likely) has privilege to set  
		        作为特定的组织(按名称string)执⾏配置⽣成，只包括org(可能)有权设置的写集 中的值。如⽤来指明⽣成的锚节点所在的组织
		  -channelID string
		        The channel ID to use in the configtx
		        在configtx中使⽤的通道ID，即通道名称，默认是"testchainid"
		  -configPath string
		        The path containing the configuration to use (if set)
		        包含要使⽤的配置的路径(如果设置的话)
		  -inspectBlock string
		        Prints the configuration contained in the block at the specified path
		        按指定路径打印块中包含的配置，⽤于检查和输出通道中创世区块的内容，锚节点 在configtx.yaml中的AnchorPeers中指定
		  -inspectChannelCreateTx string
		        Prints the configuration contained in the transaction at the specified path      
		        按指定路径打印交易中包含的配置，⽤来检查通道的配置交易信息
		  -outputAnchorPeersUpdate string
		        Creates an config update to update an anchor peer (works only with the default channel creation, and only for the first update)   
		        创建⼀个配置更新来更新锚节点(仅在默认通道创建时⼯作，并且仅在第⼀次更新时 ⼯作)
		  -outputBlock string
		        The path to write the genesis block to (if set)     
		        将genesis块写⼊(如果设置)的路径。configtx.yaml⽂件中的Profiles要指定 Consortiums，否则启动排序服务节点会失败
		  -outputCreateChannelTx string
		        The path to write a channel creation configtx to (if set)        
		        将通道配置交易⽂件写⼊(如果设置)的路径。configtx.yaml⽂件中的Profiles 必须包含Application，否则创建通道会失败
		  -printOrg string
		        Prints the definition of an organization as JSON. (useful for adding an org to a channel manually)
		        将组织的定义打印为JSON。(对于⼿动向通道添加组织⾮常有⽤)
		  -profile string
		        The profile from configtx.yaml to use for generation. (default "SampleInsecureSolo")
		        指定使⽤的是configtx.yaml中某个⽤于⽣成的Profiles配置项。(默认 为“SampleInsecureSolo”)
		  -version
		        Show version information
- `configtx.yaml` 配置文件

	Fabric 区块链⽹络运维⼯具 configtxgen ⽤于⽣成通道创世块或通道交易的配置⽂件，`configtx.yaml` 的内容直接决定了所⽣成的创世区块的内容,[具体内容](https://github.com/hyperledger/fabric/blob/master/sampleconfig/configtx.yaml) 。主
	
	要分几大部分
	
	- Capabilities/通道能力配置(定义服务版本)

		Capabilities 段用来定义 fabric 网络的能力。这是版本 v1.0.0 引入的一个新的配置段，当与版本 v1.0.x 的对等节点与排序节点混合组网时不可使用。
		
		Capabilities 段定义了 fabric 程序要加入网络所必须支持的特性。
		
		例如，如果添加了一个新的 MSP 类型，那么更新的程序可能会根据该类型识别并验证签名，但是老版本的程序就没有办法验证这些交易。这可能导致不同版本的 fabric 程序中维护的世界状态不一致。
		
		通过定义通道的能力，就明确了不满足该能力要求的 fabric 程序，将无法处理交易，除非升级到新的版本。
		
		对于v1.0.x的程序而言，如果在 Capabilities 段定义了任何能力，即使声明不需要支持这些能力，都会导致其有意崩溃。
		
			Capabilities:
			    # Global配置同时应用于排序节点和对等节点，并且必须被两种节点同时支持。
			    # 将该配置项设置为ture表明要求节点具备该能力
			    Global: &ChannelCapabilities
			        V1_3: true
			
			    # Orderer配置仅应用于排序节点，不需考虑对等节点的升级。将该配置项
			    # 设置为true表明要求排序节点具备该能力
			    Orderer: &OrdererCapabilities
			        V1_1: true
			
			    # Application配置仅应用于对等网络，不需考虑排序节点的升级。将该配置项
			    # 设置为true表明要求对等节点具备该能力
			    Application: &ApplicationCapabilities
			        V1_3: true
	- Organizations/组织机构配置

		Organizations 配置段用来定义组织机构实体，以便在后续配置中引用。例如，下面的配置文件中，定义了三个机构，可以分别使用 `ExampleCom、Org1ExampleCom `和 `Org2ExampleCom` 引用其配置：
	
			Organizations:
			    - &ExampleCom
			    	  # 组织名称
			        Name: ExampleCom
			        
			        # MSP 的 ID
			        ID: example.com
			        
			        # 组织负责人
			        AdminPrincipal: Role.ADMIN
			        
			        # MSPDir 是包含 MSP 配置的⽂件系统路径，目录中数据由 cryptogen 生成
			        MSPDir: ./ordererOrganizations/example.com/msp
			        
			        # Policies定义了在这个配置树级别的策略集，所在目录 /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
			        Policies:
			            Readers:
			                Type: Signature
			                Rule: OR('example.com.member')
			            Writers:
			                Type: Signature
			                Rule: OR('example.com.member')
			            Admins:
			                Type: Signature
			                Rule: OR('example.com.admin')
			            Endorsement:
			                Type: Signature
			                Rule: OR('example.com.member')
			
			    - &Org1ExampleCom
			        Name: Org1ExampleCom
			        ID: org1.example.com
			        MSPDir: ./peerOrganizations/org1.example.com/msp
			        AdminPrincipal: Role.ADMIN
			        
			        # 锚节点设置
			        AnchorPeers:
			        	#指明 org1 中使⽤peer0作为锚节点
			            - Host: peer0.org1.example.com
			              Port: 7051
			              
			        Policies:
			            Readers:
			                Type: Signature
			                Rule: OR('org1.example.com.member')
			            Writers:
			                Type: Signature
			                Rule: OR('org1.example.com.member')
			            Admins:
			                Type: Signature
			                Rule: OR('org1.example.com.admin')
			            Endorsement:
			                Type: Signature
			                Rule: OR('org1.example.com.member')
			
			    - &Org2ExampleCom
			        Name: Org2ExampleCom
			        ID: org2.example.com
			        MSPDir: ./peerOrganizations/org2.example.com/msp
			        AdminPrincipal: Role.ADMIN
			        AnchorPeers:
			            - Host: peer0.org2.example.com
			              Port: 7051
			        Policies:
			            Readers:
			                Type: Signature
			                Rule: OR('org2.example.com.member')
			            Writers:
			                Type: Signature
			                Rule: OR('org2.example.com.member')
			            Admins:
			                Type: Signature
			                Rule: OR('org2.example.com.admin')
			            Endorsement:
			                Type: Signature
			                Rule: OR('org2.example.com.member')
	- Orderer/排序节点配置

		Orderer 配置段用来定义要编码写入创世区块或通道交易的排序节点参数
		
		    Orderer: &OrdererDefaults
		
		    # 排序节点类型用来指定要启用的排序节点实现，不同的实现对应不同的共识算法。
		    # 目前可用的类型为：solo和kafka
		    OrdererType: solo
		    Addresses:
		        - orderer0.example.com:7050
		        
		    # 创建批处理之前要等待的时间
		    BatchTimeout: 2s
		    
		    # 控制成块的消息数量
		    BatchSize:
		    	  # 批处理中允许的最⼤消息数
		        MaxMessageCount: 10
		        # 批处理中允许序列化消息的绝对最⼤字节数
		        AbsoluteMaxBytes: 98 MB
		        # 批处理中允许序列化消息的⾸选最⼤字节数。 # ⼤于⾸选最⼤字节的消息将导致批处理⼤于改值
		        PreferredMaxBytes: 512 KB
		
		    MaxChannels: 0
		    
		    Kafka: # orderer连接到的 Kafka 代理的列表
		        Brokers:
		            - kafka0:9092
		            - kafka1:9092
		            - kafka2:9092
		            - kafka3:9092
		    # 组织被定义为⽹络的orderer⽅的参与者
		    Organizations:
		
		    # 定义本层级的排序节点策略，其权威路径为 /Channel/Orderer/<PolicyName>
		    Policies:
		        Readers:
		            Type: ImplicitMeta
		            Rule: ANY Readers
		        Writers:
		            Type: ImplicitMeta
		            Rule: ANY Writers
		        Admins:
		            Type: ImplicitMeta
		            Rule: MAJORITY Admins
		            
		        # BlockValidation配置项指定了哪些签名必须包含在区块中，以便对等节点进行验证
		        BlockValidation:
		            Type: ImplicitMeta
		            Rule: ANY Writers
		
		    # Capabilities 配置描述排序节点层级的能力需求，这里直接引用
		    # 前面 Capabilities 配置段中的 OrdererCapabilities 配置项
		    Capabilities:
		        <<: *OrdererCapabilities
	- Channel / 通道配置

		Channel 配置段用来定义要写入创世区块或配置交易的通道参数。

			Channel: &ChannelDefaults
			    # 定义本层级的通道访问策略，其权威路径为 /Channel/<PolicyName>
			    Policies:
			        Readers:
			            Type: ImplicitMeta
			            Rule: ANY Readers
			        # Writes 策略定义了调用 Broadcast API 提交交易的许可规则
			        Writers:
			            Type: ImplicitMeta
			            Rule: ANY Writers
			        # Admin 策略定义了修改本层级配置的许可规则
			        Admins:
			            Type: ImplicitMeta
			            Rule: MAJORITY Admins
			    # Capabilities 配置描通道层级的能力需求，这里直接引用
			    # 前面 Capabilities 配置段中的 ChannelCapabilities 配置项
			    Capabilities:
			        <<: *ChannelCapabilities
	- Application/应用配置

		Application 配置段用来定义要写入创世区块或配置交易的应用参数。
		
			Application: &ApplicationDefaults
			    ACLs: &ACLsDefault
			        # ACLs 配置段为系统中各种资源提供默认的策略。
			        # 这里所说的“资源”，可以是系统链码的函数，例如 qscc 系统链码的GetBlockByNumber 方法
			        # 也可以是其他资源，例如谁可以接收区块事件。
			        # 这个配置段不是用来定义资源或API，而仅仅是定义资源的访问控制策略
			        # 
			        # 用户可以在通道定义中重写这些默认策略
			
			        #---New Lifecycle System Chaincode (_lifecycle) function to policy mapping for access control--#
			
			        # _lifecycle系统链码 CommitChaincodeDefinition 函数的 ACL 定义
			        _lifecycle/CommitChaincodeDefinition: /Channel/Application/Writers
			
			        _lifecycle/QueryChaincodeDefinition: /Channel/Application/Readers
			
			        _lifecycle/QueryNamespaceDefinitions: /Channel/Application/Readers
			
			        #---Lifecycle System Chaincode (lscc) function to policy mapping for access control---#
			
			        # lscc 系统链码的 getid 函数的 ACL 定义
			        lscc/ChaincodeExists: /Channel/Application/Readers
			
			        # lscc 系统链码的 getdepspec 函数的 ACL 定义
			        lscc/GetDeploymentSpec: /Channel/Application/Readers
			
			        # lscc 系统链码的 getccdata 函数的 ACL 定义
			        lscc/GetChaincodeData: /Channel/Application/Readers
			
			        # lscc 系统链码的 getchaincodes 函数的ACL定义
			        lscc/GetInstantiatedChaincodes: /Channel/Application/Readers
			
			        #---Query System Chaincode (qscc) function to policy mapping for access control---#
			
			        # qscc系统链码的GetChainInfo函数的ACL定义
			        qscc/GetChainInfo: /Channel/Application/Readers
			
			        # qscc系统链码的GetBlockByNumber函数的ACL定义
			        qscc/GetBlockByNumber: /Channel/Application/Readers
			
			        # qscc系统 链码的GetBlockByHash函数的ACL定义
			        qscc/GetBlockByHash: /Channel/Application/Readers
			
			        # qscc系统链码的GetTransactionByID函数的ACL定义
			        qscc/GetTransactionByID: /Channel/Application/Readers
			
			        # qscc系统链码GetBlockByTxID函数的ACL定义
			        qscc/GetBlockByTxID: /Channel/Application/Readers
			
			        #---Configuration System Chaincode (cscc) function to policy mapping for access control---#
			
			        # cscc系统链码的GetConfigBlock函数的ACl定义
			        cscc/GetConfigBlock: /Channel/Application/Readers
			
			        # cscc系统链码的GetConfigTree函数的ACL定义
			        cscc/GetConfigTree: /Channel/Application/Readers
			
			        # cscc系统链码的SimulateConfigTreeUpdate函数的ACL定义
			        cscc/SimulateConfigTreeUpdate: /Channel/Application/Readers
			
			        #---Miscellanesous peer function to policy mapping for access control---#
			
			        # 访问对等节点上的链码的ACL策略定义
			        peer/Propose: /Channel/Application/Writers
			
			        # 从链码中访问其他链码的ACL策略定义
			        peer/ChaincodeToChaincode: /Channel/Application/Readers
			
			        #---Events resource to policy mapping for access control###---#
			
			        # 发送区块事件的ACL策略定义
			        event/Block: /Channel/Application/Readers
			
			        # 发送过滤的区块事件的ACL策略定义
			        event/FilteredBlock: /Channel/Application/Readers

			    # Organizations 配置列出参与到网络中的机构清单
			    Organizations:
			
			    # 定义本层级的应用控制策略，其权威路径为 /Channel/Application/<PolicyName>
			    Policies: &ApplicationDefaultPolicies
			        Readers:
			            Type: ImplicitMeta
			            Rule: "ANY Readers"
			        Writers:
			            Type: ImplicitMeta
			            Rule: "ANY Writers"
			        Admins:
			            Type: ImplicitMeta
			            Rule: "MAJORITY Admins"
			        LifecycleEndorsement:
			            Type: ImplicitMeta
			            Rule: "ANY Endorsement"
			        Endorsement:
			            Type: ImplicitMeta
			            Rule: "ANY Endorsement"
			
			    # Capabilities配置描述应用层级的能力需求，这里直接引用
			    # 前面Capabilities配置段中的ApplicationCapabilities配置项
			    Capabilities:
			        <<: *ApplicationCapabilities
	- Profiles / 配置入口

		Profiles 配置段用来定义用于 configtxgen 工具的配置入口。包含委员会（consortium）的配置入口可以用来生成排序节点的创世区块。如果在排序节点的创世区块中正确定义了consortium 的成员，那么可以仅使用机构成员名称和委员会的名称来生成通道创建请求。
		
			Profiles:
			    # 定义了一个使用 Solo 排序节点的简单配置
			    SampleInsecureSolo:
			        <<: *ChannelDefaults
			        Orderer:
			            <<: *OrdererDefaults
			            Organizations:
			                - *ExampleCom
			            Capabilities:
			                <<: *OrdererCapabilities
			        Application:
			            <<: *ApplicationDefaults
			            Organizations:
			                - *ExampleCom
			            Capabilities:
			                <<: *ApplicationCapabilities
			            Policies:
			                Readers:
			                  Type: ImplicitMeta
			                  Rule: ANY Readers
			                Writers:
			                  Type: ImplicitMeta
			                  Rule: ANY Writers
			                Admins:
			                  Type: ImplicitMeta
			                  Rule: MAJORITY Admins
			                LifecycleEndorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			                Endorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			        Consortiums:
			            SampleConsortium:
			                Organizations:
			                    - *Org1ExampleCom
			                    - *Org2ExampleCom
			
			    # 定义了一个使用 Kfaka 排序节点的配置
			    SampleInsecureKafka:
			        <<: *ChannelDefaults
			        Orderer:
			            <<: *OrdererDefaults
			            OrdererType: kafka
			            Addresses:
			                - orderer0.example.com:7050
			                - orderer1.example.com:7050
			                - orderer2.example.com:7050
			            Organizations:
			                - *ExampleCom
			            Capabilities:
			                <<: *OrdererCapabilities
			        Application:
			            <<: *ApplicationDefaults
			            Organizations:
			                - *ExampleCom
			            Capabilities:
			                <<: *ApplicationCapabilities
			            Policies:
			                Readers:
			                  Type: ImplicitMeta
			                  Rule: ANY Readers
			                Writers:
			                  Type: ImplicitMeta
			                  Rule: ANY Writers
			                Admins:
			                  Type: ImplicitMeta
			                  Rule: MAJORITY Admins
			                LifecycleEndorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			                Endorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			        Consortiums:
			            SampleConsortium:
			                Organizations:
			                    - *ExampleCom
			                    - *Org1ExampleCom
			                    - *Org2ExampleCom
			
			    # 定义了一个使用 Solo 排序节点、包含单一 MSP 的配置
			    SampleSingleMSPSolo:
			        Orderer:
			            <<: *OrdererDefaults
			            Organizations:
			                - *ExampleCom
			            Capabilities:
			                <<: *OrdererCapabilities
			        Application:
			            <<: *ApplicationDefaults
			            Organizations:
			                - *ExampleCom
			            Capabilities:
			                <<: *ApplicationCapabilities
			            Policies:
			                Readers:
			                  Type: ImplicitMeta
			                  Rule: ANY Readers
			                Writers:
			                  Type: ImplicitMeta
			                  Rule: ANY Writers
			                Admins:
			                  Type: ImplicitMeta
			                  Rule: MAJORITY Admins
			                LifecycleEndorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			                Endorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			        Consortiums:
			            SampleConsortium:
			                Organizations:
			                    - *ExampleCom
			                    - *Org1ExampleCom
			                    - *Org2ExampleCom
			
			    # 定义了一个不包含成员与访问控制策略的通道
			    SampleEmptyInsecureChannel:
			        Capabilities:
			            <<: *ChannelCapabilities
			        Consortium: SampleConsortium
			        Application:
			            Organizations:
			                - *ExampleCom
			            Capabilities:
			                <<: *ApplicationCapabilities
			            Policies:
			                Readers:
			                  Type: ImplicitMeta
			                  Rule: ANY Readers
			                Writers:
			                  Type: ImplicitMeta
			                  Rule: ANY Writers
			                Admins:
			                  Type: ImplicitMeta
			                  Rule: MAJORITY Admins
			                LifecycleEndorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			                Endorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			
			    # 定义了一个用于测试的通道
			    SysTestChannel:
			        <<: *ChannelDefaults
			        Capabilities:
			            <<: *ChannelCapabilities
			        Consortium: SampleConsortium
			        Application:
			            <<: *ApplicationDefaults
			            Organizations:
			                - *Org1ExampleCom
			                - *Org2ExampleCom
			            Capabilities:
			                <<: *ApplicationCapabilities
			            Policies:
			                Readers:
			                  Type: ImplicitMeta
			                  Rule: ANY Readers
			                Writers:
			                  Type: ImplicitMeta
			                  Rule: ANY Writers
			                Admins:
			                  Type: ImplicitMeta
			                  Rule: MAJORITY Admins
			                LifecycleEndorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			                Endorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			
			    # 定义了一个仅包含单一成员机构的通道。
			    # 该配置通常与SampleSingleMSPSolo或SampleSingleMSPKafka同时使用
			    SampleSingleMSPChannel:
			        <<: *ChannelDefaults
			        Capabilities:
			            <<: *ChannelCapabilities
			        Consortium: SampleConsortium
			        Application:
			            <<: *ApplicationDefaults
			            Organizations:
			                - *Org1ExampleCom
			                - *Org2ExampleCom
			            Capabilities:
			                <<: *ApplicationCapabilities
			            Policies:
			                Readers:
			                  Type: ImplicitMeta
			                  Rule: ANY Readers
			                Writers:
			                  Type: ImplicitMeta
			                  Rule: ANY Writers
			                Admins:
			                  Type: ImplicitMeta
			                  Rule: MAJORITY Admins
			                LifecycleEndorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement
			                Endorsement:
			                  Type: ImplicitMeta
			                  Rule: ANY Endorsement	         
- 示例代码
	- 下面的示例依据配置 `SampleSingleMSPSoloV1_1` 将通道 `orderer-system-channel` 的创世块写入文件 `genesis_block.pb`

			~$ configtxgen -outputBlock genesis_block.pb -profile SampleSingleMSPSoloV1_1 -channelID orderer-system-channel
	- 生成通道配置交易

		下面的示例依据配置 Sam 跑了 Sin 过了 `MSPChannelV1_1` 将通道创建交易写入文件`create_chan_tx.pb`

			~$ configtxgen -outputCreateChannelTx create_chan_tx.pb -profile SampleSingleMSPChannelV1_1 -channelID application-channel-1
	- 查看创始区块

		下面的示例在屏幕以 JSON 格式显示名为 `genesis_block.pb` 的创世块的内容

			~$ configtxgen -inspectBlock genesis_block.pb
		或者	
			
			configtxgen -profile TwoOrgsOrdererGenesis -inspectBlock ./channelartifacts/genesis.block
	- 查看通道交易

		下面的示例在屏幕以 JSON 格式显示通道创建交易文件 `create_chan_tx.pb` 的内容

			~$ configtxgen -inspectChannelCreateTx create_chan_tx.pb
		或
			
			configtxgen -profile TwoOrgsChannel -inspectChannelCreateTx ./channel-artifacts/channel.tx
	- 下面的示例基于 `configtx.yaml` 中的参数（例如MSPDir）构造一个组织定义，并在屏幕以 JSON格式显示：

			~$ configtxgen -printOrg Org1
		上述命令的输出对于通道重新配置任务（例如添加新的成员）非常有帮助。
	- 创建组织锚点配置

		下面的示例将配置更新交易写入文件 `anchor_peer_tx.pb`，该交易为配置`SampleSingleMSPChannelV1_1` 中的组织 Org1 设置锚节点

			~$ configtxgen -outputAnchorPeersUpdate anchor_peer_tx.pb -profile SampleSingleMSPChannelV1_1 -asOrg Org1
		或
			
			configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME asOrg Org1MSP
		-  `./channel-artifacts/Org1MSPanchors.tx` 

			为⽣成锚节点的更新交易⽂件及保存路径 
		- `asOrg Org1MSP`  

			指明该锚节点所在的组织			
- 创建一个通道的操作顺序(`byfn.sh` 说明）

	`first-network/byfn.sh` ⽂件中使⽤该⼯具来⽣成创始块、通道配置交易和锚节点更新交易 
	
	1. 第一步创建创世区块

			if [ "$CONSENSUS_TYPE" == "solo" ]; then
				configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-syschannel -outputBlock ./channel-artifacts/genesis.block
			elif [ "$CONSENSUS_TYPE" == "kafka" ]; then
				configtxgen -profile SampleDevModeKafka -channelID byfn-syschannel -outputBlock ./channel-artifacts/genesis.block 
			else
			....
		- `TwoOrgsOrdererGenesis`  

			指定使⽤的是 `configtx.yaml` 中的 `TwoOrgsOrdererGenesis` 如
		
				Profiles:
		
					TwoOrgsOrdererGenesis:
						<<: *ChannelDefaults 
						Orderer:
							<<: *OrdererDefaults
							Organizations:
								- *OrdererOrg 
							Capabilities:
								<<: *OrdererCapabilities 
						Consortiums:
							SampleConsortium:
								Organizations:
									- *Org1
									- *Org2
		- `SampleDevModeKafka` 

			指定使⽤的是 `configtx.yaml` 中的 `SampleDevModeKafka` 配置
		
				SampleDevModeKafka:
					<<: *ChannelDefaults 
					Capabilities:
						<<: *ChannelCapabilities 
					Orderer:
						<<: *OrdererDefaults 
						OrdererType: kafka 
						Kafka:
							Brokers:
							- kafka.example.com:9092
						Organizations:
							- *OrdererOrg
						Capabilities:
							<<: *OrdererCapabilities 
						Application:
							<<: *ApplicationDefaults 
							Organizations:
								- <<: *OrdererOrg 
							Consortiums:
								SampleConsortium:
									Organizations:
										- *Org1
										- *Org2
		- `-channelID byfn-sys-channel` 

			将通道名称命名为 `byfn-sys-channel` 
		- `-outputBlock ./channel-artifacts/genesis.block` 

			为⽣成的创世区块⽂件名及保存路径
	2. 创建通道配置

			configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME	
		- TwoOrgsChannel 
		
			指定使⽤的是 `configtx.yaml` 中的 TwoOrgsChannel 配置
			
				TwoOrgsChannel:
					Consortium: SampleConsortium Application:
					<<: *ApplicationDefaults
					Organizations:
						- *Org1
						- *Org2 
					Capabilities:
						<<: *ApplicationCapabilities
		- `./channel-artifacts/channel.tx` 

			指明⽣成的通道配置存储的路径
		-  `$CHANNEL_NAME`

			通道名为⾃⼰设置的通道名
	3. 创建锚节点配置
	
			configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME asOrg Org1MSP
	4. 查看创世区块信息

			configtxgen -profile TwoOrgsOrdererGenesis -inspectBlock ./channelartifacts/genesis.block
	5. 查看通道配置交易信息

			configtxgen -profile TwoOrgsChannel -inspectChannelCreateTx ./channel-artifacts/channel.tx  		

## configtxlator
configtxlator 命令用来将 fabric 的数据结构在 protobuf 和 JSON 之间进行转换，也可以用来创建配置更新。该命令可以启动一个 REST 服务来通过 HTTP 暴露服务接口，也可以直接在命令行使用。

- 命令语法

	configtxlator工具有5个子命令，列举如下：

	- start
	- proto_encode
	- proto_decode
	- compute_update
	- version
- 编译

		$ cd $GOPATH/src/github.com/hyperledger/fabric 
		$ make configtxlator 
		$ cp build/bin/configtxlator /usr/bin
- 参数

		usage: configtxlator [<flags>] <command> [<args> ...]
		Flags:
			--help 显示上下⽂敏感的帮助信息 (also try --help-long and --helpman).
		Commands:
		
			help [<command>...] 显示帮助信息.
			start [<flags>] 开启configtxlator REST服务端，默认在本地7059端⼝启动监听 
			proto_encode --type=TYPE [<flags>] 转换JSON⽂件成指定的protobuf格式 
			proto_decode --type=TYPE [<flags>] 转换proto信息为JSON格式 
			compute_update --channel_id=CHANNEL_ID [<flags>] 带两个编码的common.Config消息，并计算在两者之间转换的配置更新。 
			version 显示版本信息

### configtxlator start
启动 configtxlator 的服务

- 使用方法

		~$ configtxlator start [<flags>]
- 命令标志

		  --help                Show context-sensitive help (also try --help-long and
		                        --help-man).
		  --hostname="0.0.0.0"  The hostname or IP on which the REST server will listen
		  --port=7059           The port on which the REST server will listen
- 示例代码

	下面的示例在本地8080端口启动rest服务：

		~$ configtxlator start --hostname="127.0.0.1" --port=8080
- restful api 接口
	- 解码

		（接⼝地址为）`/protolator/decode/{msgName}`，⽀持POST操作，将⼆进制格式数据解码为 Json 格式，其中 `{msgName}` 需要指定 Fabric 中定义的对应 Protobuf 消息结构的名称。 
	- 编码

		（接⼝地址为）`/protolator/encode/{msgName}`，⽀持 POST 操作，将 Json 格式的数据编码为⼆进制格式，其中 `{msgName}` 需要指定 Protobuf 消息类 型名称 
	- 计算配置更新量

		`/configtxlator/compute/update-from-configs`，⽀持 POST 操作，计算两个配置（common.Config消息结构）之间的⼆进制格式更新量。

### configtxlator proto_encode
将一个 JSON 文档转换为 protobuf 格式数据

- 使用方法
	
		~$ configtxlator proto_encode --type=TYPE [<flags>]
- 命令标志

		  --help                Show context-sensitive help (also try --help-long and
		                        --help-man).
		  --type=TYPE           The type of protobuf structure to encode to. For
		                        example, 'common.Config'.
		  --input=/dev/stdin    A file containing the JSON document.
		  --output=/dev/stdout  A file to write the output to.
- 示例代码

	下面的示例将控制台输入的 json 格式的 policy，转换为 protobuf 格式并存入文件 policy.pb

			~$ configtxlator proto_encode --type common.Policy --output policy.pb
	在启动 rest 服务后，下面的示例使用 curl 命令通过 rest api 执行同样的操作

			~$ curl -X POST --data-binary /dev/stdin "${CONFIGTXLATOR_URL}/protolator/encode/common.Policy" > policy.pb			
### configtxlator proto_decode
将一个 protobuf 消息转换为 JSON 格式。

- 使用方法

		~$ configtxlator proto_decode --type=TYPE [<flags>]
- 命令标志

		  --help                Show context-sensitive help (also try --help-long and
		                        --help-man).
		  --type=TYPE           The type of protobuf structure to decode from. For
		                        example, 'common.Config'.
		  --input=/dev/stdin    A file containing the proto message.
		  --output=/dev/stdout  A file to write the JSON document to.
- 示例代码

	下面的示例将名为 fabric_block.pb 的区块转换为 json 格式并在 stdout 输出：

		~$ configtxlator proto_decode --input fabric_block.pb --type common.Block
	在启动 rest 服务后，下面的示例使用 curl 命令通过 rest api 执行同样的操作：

		~$ curl -X POST --data-binary @fabric_block.pb "${CONFIGTXLATOR_URL}/protolator/decode/common.Block"			
### configtxlator compute_update
计算生成两个 common.Config 消息之间的配置更新。

- 使用方法

		~$ configtxlator compute_update --channel_id=CHANNEL_ID [<flags>]
- 命令标志

		  --help                   Show context-sensitive help (also try --help-long and
		                           --help-man).
		  --original=ORIGINAL      The original config message.
		  --updated=UPDATED        The updated config message.
		  --channel_id=CHANNEL_ID  The name of the channel for this update.
		  --output=/dev/stdout     A file to write the JSON document to.
- 示例代码

	下面的示例计算从 original_config.pb 到 modified_config.pb 的配置更新，并在 stdout 输出 JSON 解码后的数据

		~$ configtxlator compute_update --channel_id testchan --original original_config.pb --updated modified_config.pb | configtxlator proto_decode --type common.ConfigUpdate
在启动 rest 服务后，下面的示例使用 curl 命令通过 rest api 执行同样的操作

		~$ curl -X POST -F channel=testchan -F "original=@original_config.pb" -F "updated=@modified_config.pb" "${CONFIGTXLATOR_URL}/configtxlator/compute/update-from-configs" | curl -X POST --data-binary /dev/stdin "${CONFIGTXLATOR_URL}/protolator/encode/common.ConfigUpdate"
				
## peer 命令
peer 命令帮助管理员对 Fabric 区块链网络中的节点进行管理，根据管理任务的不同，peer 命令细分为5 个子命令。如可以使用 `peer channel` 子命令来将一个 peer 节点添加到通道（channel）中，或者使用 `peer chaincode` 子命令来将一个智能合约链码部署到指定的 peer 节点。
### 语法
peer 命令包含如下5个子命令：

	peer chaincode [option] [flags]
	peer channel   [option] [flags]
	peer logging   [option] [flags]
	peer node      [option] [flags]
	peer version   [option] [flags]
每个子命令都有不同的选项（option），出于简化考虑，有时我们将命令（例如peer）、子命令（例如channel） 或子命令选项（例如fetch）都称为命令。

如果运行某个子命令时没有指定选项，那么它将返回相关的帮助信息。

每个 peer 子命令都有一套自己的 flag，其中许多标志被有意设计为全局标志，以便可以在所有的子命令中使用该标志。我们将在相关的 peer 子命令中介绍这些标志。

peer命令的顶层标志如下

- `--help` 

	使用 `--help` 标志获取任何 peer 命令的帮助信息，该标志非常有价值，你可以用它获得命令、子命令乃至选项的帮助

如：

	peer --help
	peer channel --help
	peer channel list --help
### 示例代码
下面的命令使用--help标志查看peer channel join命令的帮助信息：

	~$ peer channel join --help
可以在终端看到如下的帮助文本

	Joins the peer to a channel.

	Usage:
	  peer channel join [flags]
	
	Flags:
	  -b, --blockpath string   Path to file containing genesis block
	  -h, --help               help for join
	
	Global Flags:
	      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
	      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
	      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
	      --connTimeout duration                Timeout for client to connect (default 3s)
	      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
	  -o, --orderer string                      Ordering service endpoint
	      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
	      --tls                                 Use TLS when communicating with the orderer endpoint
## 子命令
### peer node
管理员可以使用 `peer node` 命令来启动 peer 节点，或者检查一个 peer 节点旳状态。
#### 命令语法
`peer node` 命令包含如下子命令：

- `peer node start`

	启动一个与网络交互的节点
- 使用方法

		~$ peer node start [flags]
			- 命令标志
				-h, --help                help for start
				-o, --orderer string      Ordering service endpoint (default "orderer:7050")
				--peer-chaincodedev   Whether peer in chaincode development mode
- 示例代码
	
	下面的命令在链码开发环境中启动一个peer节点：
			
		~$ peer node start --peer-chaincodedev
		
	通常链码容器是由 peer 节点启动和维护的。但是在链码开发模式下，由用户负责构造和启动链码。开发模式对于链码开发阶段的迭代开发非常有用
- `peer node status`

	返回运行中的 peer 节点旳状态
- 使用方法

		~$ peer node status [flags]
- 命令标志
		
		-h, --help   help for status

### peer channel
管理员可以使用 `peer channel` 命令在 peer 节点上执行通道相关的操作，例如加入通道或者列出节点当前加入的通道清单。
#### 命令语法
peer命令包含以下子命令

	create
	fetch
	getinfo
	join
	list
	signconfigtx
	update
使用方法

	~$ peer channel [command]
可用的子命令包括:

	  create       Create a channel
	  fetch        Fetch a block
	  getinfo      get blockchain information of a specified channel.
	  join         Joins the peer to a channel.
	  list         List of channels peer has joined.
	  signconfigtx Signs a configtx update.
	  update       Send a configtx update.
命令标志包括

	      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
	      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
	      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
	      --connTimeout duration                Timeout for client to connect (default 3s)
	  -h, --help                                help for channel
	      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
	  -o, --orderer string                      Ordering service endpoint
	      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
	      --tls                                 Use TLS when communicating with the orderer endpoint
	使用 peer channel [command] --help 获取关于某个子命令的更多信息。

- peer channel create

	创建一个通道，并将创世块写入文件。

	- 使用方法

			~$ peer channel create [flags]	
	- 命令标志
		  
			  -c, --channelID string     In case of a newChain command, the channel ID to create. It must be all lower case, less than 250 characters long and match the regular expression: [a-z][a-z0-9.-]*
			  -f, --file string          Configuration transaction file generated by a tool such as configtxgen for submitting to orderer
			  -h, --help                 help for create
			      --outputBlock string   The path to write the genesis block for the channel. (default ./<channelID>.block)
			  -t, --timeout duration     Channel creation timeout (default 5s)
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
	- 示例代码

		本部分示例在 `peer channel create` 命令中使用全局标志 `--orderer` 指定排序器。
		
		- 下面的示例基于在文件 `./createchannel.txn` 中的配置交易创建一个通道 `mychannel`，使用 的排序器端结点为 `orderer.example.com:7050`：

				~$ peer channel create -c mychannel -f ./createchannel.txn --orderer orderer.example.com:7050
				
				2018-02-25 08:23:57.548 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
				2018-02-25 08:23:57.626 UTC [channelCmd] InitCmdFactory -> INFO 019 Endorser and orderer connections initialized
				2018-02-25 08:23:57.834 UTC [channelCmd] readBlock -> INFO 020 Received block: 0 <-返回第0块表示通道已经成功创建
				2018-02-25 08:23:57.835 UTC [main] main -> INFO 021 Exiting.....
		- 下面的示例展示 `peer channel create` 命令选项的用法。在该示例中使用的排序器为`orderer.example.com:7050` ， 创建新的通道 `mychannel` 所需要的配置更新交易定义在文件 `./createchannel.txn` 中，等待30秒以便通道创建成功：

				~$ peer channel create -c mychannel --orderer orderer.example.com:7050 -f ./createchannel.txn -t 30s
			使用 `ls` 命令查看创建好的通道创世块
	
				~$ ls -l
				-rw-r--r-- 1 root root 11982 Feb 25 12:24 mychannel.block
			上面可以看到通道 mychannel 已经成功创建，第0块已经加入到该通道的区块链中并返回给节点，该区块在本地文件中存为 mychannel.block。
	
			第0块通常被称为创世块，因为它提供了通道的起始配置。
- `peer channel fetch`

	提取指定区块并导出写入文件
	
	- 使用方法

			~$ peer channel fetch <newest|oldest|config|(number)> [outputfile] [flags]
	- 命令标志
	
			-c, --channelID string   In case of a newChain command, the channel ID to create. It must be all lower case, less than 250 characters long and match the regular expression: [a-z][a-z0-9.-]*
			-h, --help               help for fetch
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
	- 示例代码
		- 下面的示例使用 `newest` 选项获取最新的通道区块，并存入文件 `mychannel.block` ：

				peer channel fetch newest mychannel.block -c mychannel --orderer orderer.example.com:7050
	
				2018-02-25 13:10:16.137 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
				2018-02-25 13:10:16.144 UTC [channelCmd] readBlock -> INFO 00a Received block: 32
				2018-02-25 13:10:16.145 UTC [main] main -> INFO 00b Exiting.....
			查看获取到的文件
	
				~$ ls -l
				
				-rw-r--r-- 1 root root 11982 Feb 25 13:10 mychannel.block
			可以看到获取到的区块序号为32，区块信息写入文件 `mychannel.block`
		- 下面的示例使用区块序号选项提取指定的区块 —— 在本例中为16 —— 并存入默认区块文件：

				~$ peer channel fetch 16  -c mychannel --orderer orderer.example.com:7050
	
				2018-02-25 13:46:50.296 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
				2018-02-25 13:46:50.302 UTC [channelCmd] readBlock -> INFO 00a Received block: 16
				2018-02-25 13:46:50.302 UTC [main] main -> INFO 00b Exiting.....
			查看获取到的文件
	
				~$ ls -l
				
				-rw-r--r-- 1 root root 11982 Feb 25 13:10 mychannel.block
				-rw-r--r-- 1 root root  4783 Feb 25 13:46 mychannel_16.block
			可以看到获取到的区块编号为16，该区块信息被写入默认文件mychannel_16.block。
	
			对于配置区块，可以使用 `configtxlatore` 命令解码区块文件。用户交易区块也可以解码，但需要编写一个额外的程序来完成这一工作。	
- `peer channel getinfo`

	获取指定通道上的区块链信息。需要-c标志。

	- 使用方法

			~$ peer channel getinfo [flags]
	- 命令标志
  
			  -c, --channelID string   In case of a newChain command, the channel ID to create. It must be all lower case, less than 250 characters long and match the regular expression: [a-z][a-z0-9.-]*
			  -h, --help               help for getinfo
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
	- 示例代码

		下面的示例获取通道mychannel的信息：

			~$ peer channel getinfo -c mychannel

			2018-02-25 15:15:44.135 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
			Blockchain info: {"height":5,"currentBlockHash":"JgK9lcaPUNmFb5Mp1qe1SVMsx3o/22Ct4+n5tejcXCw=","previousBlockHash":"f8lZXoAn3gF86zrFq7L1DzW2aKuabH9Ow6SIE5Y04a4="}
			2018-02-25 15:15:44.139 UTC [main] main -> INFO 006 Exiting.....
		可以看到通道 mychannel 的最新区块序号为5，以及最新区块的密码学哈希值等信息
- `peer channel join`

	将指定节点加入到一个通道中。
	
	- 使用方法

			~$ peer channel join [flags]
	- 命令标志
  
			  -b, --blockpath string   Path to file containing genesis block
			  -h, --help               help for join
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		       -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
	- 示例代码

		下面的示例将 peer 节点加入到文件 `./mychannel.genesis.block` 中包含的创世块中定义的通， 该通道区块文件是之前使用 `peer channel fetch` 命令获取到的

			~$ peer channel join -b ./mychannel.genesis.block
			
			2018-02-25 12:25:26.511 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
			2018-02-25 12:25:26.571 UTC [channelCmd] executeJoin -> INFO 006 Successfully submitted proposal to join channel
			2018-02-25 12:25:26.571 UTC [main] main -> INFO 007 Exiting.....
		可以看到 peer 节点已经成功地提交了加入通道的请求。
- `peer channel list`

	列出peer节点已经加入的通道。

	- 使用方法

			~$ peer channel list [flags]
	- 命令标志
	
			-h, --help   help for list
	- 全局标志
		
		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
	- 示例代码

		下面的示例列出一个 peer 节点已经加入的通道清单：

			~$ peer channel list
			
			2018-02-25 14:21:20.361 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
			Channels peers has joined:
			mychannel
			2018-02-25 14:21:20.372 UTC [main] main -> INFO 006 Exiting.....
		可以看到节点已经加入了通道 mychannel
- `peer channel signconfigtx`

	为指定的 `configtx` 更新文件原地签名。需要 `-f` 标志。

	- 使用方法

			~$ peer channel signconfigtx [flags]
	- 命令标志
 
			 -f, --file string   Configuration transaction file generated by a tool such as configtxgen for submitting to orderer
			  -h, --help          help for signconfigtx
	- 全局标志

			      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
			      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
			      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
			      --connTimeout duration                Timeout for client to connect (default 3s)
			      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
			  -o, --orderer string                      Ordering service endpoint
			      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
			      --tls                                 Use TLS when communicating with the orderer endpoint
	- 示例代码

		下面的示例签名文件 `./updatechannel.tx` 中定义的通道更新交易。示例列出了在签名前后的配置交易文件

			~$ ls -l
			-rw-r--r--  1 anthonyodowd  staff   284 25 Feb 18:16 updatechannel.tx
			~$ peer channel signconfigtx -f updatechannel.tx
			
			2018-02-25 18:16:44.456 GMT [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
			2018-02-25 18:16:44.459 GMT [main] main -> INFO 002 Exiting.....
			
			~$ ls -l
			
			-rw-r--r--  1 anthonyodowd  staff  2180 25 Feb 18:16 updatechannel.tx

		可以看到节点已经成功地完成了配置交易的签名，文件 `updatechannel.tx` 的大小在签名前后有了明显变化， 从284字节变成了2180字节
- `peer channel update`

	签名并发送指定的通道 configtx 更新文件。需要 `-f、-o` 和 `-c`标志。

	- 使用方法

			~$ peer channel update [flags]
	- 命令标志
			
			-c, --channelID string   In case of a newChain command, the channel ID to create. It must be all lower case, less than 250 characters long and match the regular expression: [a-z][a-z0-9.-]*
			  -f, --file string        Configuration transaction file generated by a tool such as configtxgen for submitting to orderer
			  -h, --help               help for update
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		       -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
	- 示例代码

		下面的示例使用文件 `./updatechannel.txn` 中的配置交易更新通道 mychannel， 使用位于`orderer.example.com:7050` 的排序器向所有 peer 节点发送配置交易以便更新其通道配置：

			~$ peer channel update -c mychannel -f ./updatechannel.txn -o orderer.example.com:7050

			2018-02-23 06:32:11.569 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
			2018-02-23 06:32:11.626 UTC [main] main -> INFO 010 Exiting.....
执行到这一步时，通道 mychannel 已经成功更新了。	

### peer chaincode
管理员可以使用 `peer chaincode` 命令在 peer 节点上执行链码相关的操作， 例如安装、实例化、调用、打包、查询以及升级链码。

- 语法

	`peer chaincode` 命令包含以下子命令

		install
		instantiate
		invoke
		list
		package
		query
		signpackage
		upgrade

	不同的子命令选项（install、instantiate...）对应于 peer 节点上链码的不同操作。 例如，使用`peer chaincode install` 子命令选项在 peer 节点上安装一个链码，或者使用 `peer chaincode query` 子命令选项查询节点上账本中的状态当前值。
- 标志

	每个 `peer chaincode` 子命令都有自己的标志（flag），同时也有一组全局性标志。 并非所有的子命令都会使用这些标志，例如，query 子命令不需要 `--orderer` 标志。
- 全局标志包括

	- --cafile ：用于排序端结点的PEM编码可信证书文件路径
	- --certfile ：用于与排序器（orderer）的TLS通信的PEM编码的X509公钥文件路径
	- --keyfile ：用于与排序器的TLS通信的PEM编码的私钥文件路径
	- -o 或--orderer ：排序服务端结点，格式为<主机名或ip地址>:<端口号>
	- --ordererTLSHostnameOverride ：验证与排序器的TLS连接时使用的主机名
	- --tls：与排序器的通信是否使用TLS加密
	- --transient ：JSON编码的临时参数映射表	

- `peer chaincode install`

	将指定的链码打包为 CDS（Chaincode Deployment Spec）格式，并存入节点上指定的路径。

	- 使用方法

			~$ peer chaincode install [flags]
	- 标志
			      
			 --connectionProfile string       Connection profile that provides the necessary connection information for the network. Note: currently only supported for providing peer connection information
			  -c, --ctor string                    Constructor message for the chaincode in JSON format (default "{}")
			  -h, --help                           help for install
			  -l, --lang string                    Language of chaincode, either "golang" (default), "node", or "java"
			  -n, --name string                    Name of the chaincode
			  -p, --path string                    Path to chaincode, for "golang" use relative path from $GOPATH/src, for "node" or "java" use absolute path
			      --peerAddresses stringArray      The addresses of the peers to connect to
			      --tlsRootCertFiles stringArray   If TLS is enabled, the paths to the TLS root cert files of the peers to connect to. The order and number of certs specified should match the --peerAddresses flag
			  -v, --version string                 Version of the chaincode specified in install/instantiate/upgrade commands
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
		      --transient string                    Transient map of arguments in JSON encoding
- `peer chaincode instantiate`

	将指定的链码部署到网络上。

	- 使用方法

			~$ peer chaincode instantiate [flags]
	- 标志
  
			  -C, --channelID string               The channel on which this command should be executed
			      --collections-config string      The fully qualified path to the collection JSON file including the file name
			      --connectionProfile string       Connection profile that provides the necessary connection information for the network. Note: currently only supported for providing peer connection information
			  -c, --ctor string                    Constructor message for the chaincode in JSON format (default "{}")
			  -E, --escc string                    The name of the endorsement system chaincode to be used for this chaincode
			  -h, --help                           help for instantiate
			  -l, --lang string                    Language the chaincode is written in (default "golang")
			  -n, --name string                    Name of the chaincode
			      --peerAddresses stringArray      The addresses of the peers to connect to
			  -P, --policy string                  The endorsement policy associated to this chaincode
			      --tlsRootCertFiles stringArray   If TLS is enabled, the paths to the TLS root cert files of the peers to connect to. The order and number of certs specified should match the --peerAddresses flag
			  -v, --version string                 Version of the chaincode specified in install/instantiate/upgrade commands
			  -V, --vscc string                    The name of the verification system chaincode to be used for this chaincode
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
		      --transient string                    Transient map of arguments in JSON encoding
	- 示例代码
		- 在下面的示例中，使用 `peer chaincode instantiate` 命令将名为 mycss、 版本为1.0的链码在 mychannel 通道上实例化，在启用TLS加密通信的网络中， 还需要使用全局标志 `--tls` 和 `--cafile` 来实例化链码。

				~$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
				~$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
				
				2018-02-22 16:33:53.324 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
				2018-02-22 16:33:53.324 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
				2018-02-22 16:34:08.698 UTC [main] main -> INFO 003 Exiting.....
		- 在下面的示例中，没有使用全局标志来实例化链码，因为在禁用 TLS 的网络中不需要 `--tls` 和 `--cafile` 全局标志：

				~$ peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
				2018-02-22 16:34:09.324 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc 2018-02-22 16:34:09.324 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc 2018-02-22 16:34:24.698 UTC [main] main -> INFO 003 Exiting.....		      		
- `peer chaincode invoke`

	调用指定给的链码。该命令将尝试向网络提交背书过的交易

	- 使用方法

			~$ peer chaincode invoke [flags]
	- 命令标志

			  -C, --channelID string               The channel on which this command should be executed
			      --connectionProfile string       Connection profile that provides the necessary connection information for the network. Note: currently only supported for providing peer connection information
			  -c, --ctor string                    Constructor message for the chaincode in JSON format (default "{}")
			  -h, --help                           help for invoke
			  -n, --name string                    Name of the chaincode
			      --peerAddresses stringArray      The addresses of the peers to connect to
			      --tlsRootCertFiles stringArray   If TLS is enabled, the paths to the TLS root cert files of the peers to connect to. The order and number of certs specified should match the --peerAddresses flag
			      --waitForEvent                   Whether to wait for the event from each peer's deliver filtered service signifying that the 'invoke' transaction has been committed successfully
			      --waitForEventTimeout duration   Time to wait for the event from each peer's deliver filtered service signifying that the 'invoke' transaction has been committed successfully (default 30s)
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
		      --transient string                    Transient map of arguments in JSON encoding
	- 示例代码

		下面的示例展示课如何使用 `peer chaincode invoke` 命令通过节点 `peer0.org1.example.com:7051` 和节点 `peer0.org2.example.com:7051 `调用通道 mychannel 上的链码 mycc（版本1.0），请求链码从变量 a向变量b转义10个单位。

			~$ peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --peerAddresses peer0.org2.example.com:7051 -c '{"Args":["invoke","a","b","10"]}'
			
			2018-02-22 16:34:27.069 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
			2018-02-22 16:34:27.069 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
			.
			.
			.
			2018-02-22 16:34:27.106 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 00a ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \237mM\376? [\214\002 \332\204\035\275q\227\2132A\n\204&\2106\037W|\346#\3413\274\022Y\nE\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\003\022-\n\004mycc\022%\n\007\n\001a\022\002\010\003\n\007\n\001b\022\002\010\003\032\007\n\001a\032\00290\032\010\n\001b\032\003210\032\003\010\310\001\"\013\022\004mycc\032\0031.0" endorsement:<endorser:"\n\007Org1MSP\022\262\006-----BEGIN CERTIFICATE-----\nMIICLjCCAdWgAwIBAgIRAJYomxY2cqHA/fbRnH5a/bwwCgYIKoZIzj0EAwIwczEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh\nLm9yZzEuZXhhbXBsZS5jb20wHhcNMTgwMjIyMTYyODE0WhcNMjgwMjIwMTYyODE0\nWjBwMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN\nU2FuIEZyYW5jaXNjbzETMBEGA1UECxMKRmFicmljUGVlcjEfMB0GA1UEAxMWcGVl\ncjAub3JnMS5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABDEa\nWNNniN3qOCQL89BGWfY39f5V3o1pi//7JFDHATJXtLgJhkK5KosDdHuKLYbCqvge\n46u3AC16MZyJRvKBiw6jTTBLMA4GA1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAA\nMCsGA1UdIwQkMCKAIN7dJR9dimkFtkus0R5pAOlRz5SA3FB5t8Eaxl9A7lkgMAoG\nCCqGSM49BAMCA0cAMEQCIC2DAsO9QZzQmKi8OOKwcCh9Gd01YmWIN3oVmaCRr8C7\nAiAlQffq2JFlbh6OWURGOko6RckizG8oVOldZG/Xj3C8lA==\n-----END CERTIFICATE-----\n" signature:"0D\002 \022_\342\350\344\231G&\237\n\244\375\302J\220l\302\345\210\335D\250y\253P\0214:\221e\332@\002 \000\254\361\224\247\210\214L\277\370\222\213\217\301\r\341v\227\265\277\336\256^\217\336\005y*\321\023\025\367" >
			2018-02-22 16:34:27.107 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00b Chaincode invoke successful. result: status:200
			2018-02-22 16:34:27.107 UTC [main] main -> INFO 00c Exiting.....
		根据日志消息可以看到调用已经成功提交：

			2018-02-22 16:34:27.107 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00b Chaincode invoke successful. result: status:200
		上面的响应表示交易已经成功提交给排序器，该交易将随后被加入一个区块，并最终被通道中的每个 peer 节点验证

- `peer chaincode list`

查询指定节点上已经安装的链码，或者查询指定通道中实例化的链码。

- 使用方法

		~$ peer chaincode list [flags]
- 命令标志
	  
		  -C, --channelID string               The channel on which this command should be executed
		      --connectionProfile string       Connection profile that provides the necessary connection information for the network. Note: currently only supported for providing peer connection information
		  -h, --help                           help for list
		      --installed                      Get the installed chaincodes on a peer
		      --instantiated                   Get the instantiated chaincodes on a channel
		      --peerAddresses stringArray      The addresses of the peers to connect to
		      --tlsRootCertFiles stringArray   If TLS is enabled, the paths to the TLS root cert files of the peers to connect to. The order and number of certs specified should match the --peerAddresses flag
- 全局标志

	      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
	      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
	      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
	      --connTimeout duration                Timeout for client to connect (default 3s)
	      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
	      -o, --orderer string                      Ordering service endpoint
	      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
	      --tls                                 Use TLS when communicating with the orderer endpoint
	      --transient string                    Transient map of arguments in JSON encoding
- 示例代码
	- 下面的示例代码使用 `--installed` 标志列出在当前 peer 节点上安装的链码

			~$ peer chaincode list --installed
			Get installed chaincodes on peer:
			Name: mycc, Version: 1.0, Path: github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02, Id: 8cc2730fdafd0b28ef734eac12b29df5fc98ad98bdb1b7e0ef96265c3d893d61
			2018-02-22 17:07:13.476 UTC [main] main -> INFO 001 Exiting.....
		可以看到，节点已经安装了名为 mycc 的链码，其版本为1.0。
	- 下面的示例使用 `--instantiated` 与 `-C` (通道ID)的组合来列出指定通道上实例化的链码

			~$ peer chaincode list --instantiated -C mychannel
			Get instantiated chaincodes on channel mychannel:
			Name: mycc, Version: 1.0, Path: github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02, Escc: escc, Vscc: vscc
			2018-02-22 17:07:42.969 UTC [main] main -> INFO 001 Exiting.....
		可以看到，版本为1.0的链码mycc已经在通道mychannel上实例化了。

- `peer chaincode package`

	将指定的链码打包为CDS（Chaincode Deployment Spec：链码部署规范）格式。

	- 使用方法

			~$ peer chaincode package [flags]
	- 命令标志
  
			  -s, --cc-package                  create CC deployment spec for owner endorsements instead of raw CC deployment spec
			  -c, --ctor string                 Constructor message for the chaincode in JSON format (default "{}")
			  -h, --help                        help for package
			  -i, --instantiate-policy string   instantiation policy for the chaincode
			  -l, --lang string                 Language of chaincode, either "golang" (default), "node", or "java"
			  -n, --name string                 Name of the chaincode
			  -p, --path string                 Path to chaincode, for "golang" use relative path from $GOPATH/src, for "node" or "java" use absolute path
			  -S, --sign                        if creating CC deployment spec package for owner endorsements, also sign it with local MSP
			  -v, --version string              Version of the chaincode specified in install/instantiate/upgrade commands
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		       -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
		      --transient string                    Transient map of arguments in JSON encoding
	- 示例代码

		下面的示例使用 `peer chaincode package` 命令将版本1.1的 mycc 链码进行打包，然后使用本地 MSP 签名，最后输出位 `ccpack.out`：

			~$ peer chaincode package ccpack.out -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 1.1 -s -S
			.
			.
			.
			2018-02-22 17:27:01.404 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
			2018-02-22 17:27:01.405 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
			.
			.
			.
			2018-02-22 17:27:01.879 UTC [chaincodeCmd] chaincodePackage -> DEBU 011 Packaged chaincode into deployment spec of size <3426>, with args = [ccpack.out]
			2018-02-22 17:27:01.879 UTC [main] main -> INFO 012 Exiting.....

- `peer chaincode query`

	获取并显示链码函数调用的背书结果。该命令不会生成交易。
- 使用方法

		~$ peer chaincode query [flags]
- 命令标志

		 -C, --channelID string               The channel on which this command should be executed
		      --connectionProfile string       Connection profile that provides the necessary connection information for the network. Note: currently only supported for providing peer connection information
		  -c, --ctor string                    Constructor message for the chaincode in JSON format (default "{}")
		  -h, --help                           help for query
		  -x, --hex                            If true, output the query value byte array in hexadecimal. Incompatible with --raw
		  -n, --name string                    Name of the chaincode
		      --peerAddresses stringArray      The addresses of the peers to connect to
		  -r, --raw                            If true, output the query value as raw bytes, otherwise format as a printable string
		      --tlsRootCertFiles stringArray   If TLS is enabled, the paths to the TLS root cert files of the peers to connect to. The order and number of certs specified should match the --peerAddresses flag
- 全局标志
	
		--cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
	      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
	      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
	      --connTimeout duration                Timeout for client to connect (default 3s)
	      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
	       -o, --orderer string                      Ordering service endpoint
	      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
	      --tls                                 Use TLS when communicating with the orderer endpoint
	      --transient string                    Transient map of arguments in JSON encoding
- 示例代码

	下面的示例使用 `peer chaincode query` 命令，通过版本为1.0的链码mycc查询节点账本的变量a的值

		~$ peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
		
		2018-02-22 16:34:30.816 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
		2018-02-22 16:34:30.816 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
		Query Result: 90
	可以看到，在查询时变量a的值为90。			

- `peer chaincode signpackage`

	签名指定的链码包。
- 使用方法

		~$ peer chaincode signpackage [flags]
- 命令标志

		-h, --help   help for signpackage
- 全局标志

	      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
	      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
	      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
	      --connTimeout duration                Timeout for client to connect (default 3s)
	      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
	      -o, --orderer string                      Ordering service endpoint
	      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
	      --tls                                 Use TLS when communicating with the orderer endpoint
	      --transient string                    Transient map of arguments in JSON encoding
- 示例代码

	下面的示例使用 `peer chaincode signpackage` 命令，基于一个已有的签名包创建一个新的具有本地 MSP 签名的包：

		~$ peer chaincode signpackage ccwith1sig.pak ccwith2sig.pak
		
		Wrote signed package to ccwith2sig.pak successfully
		2018-02-24 19:32:47.189 EST [main] main -> INFO 002 Exiting.....

- `peer chaincode upgrade`

	升级指定的链码。当交易提交后新的链码将立即替代原来的链码。

	- 使用方法

			~$ peer chaincode upgrade [flags]
	- 命令标志
  
			 -C, --channelID string               The channel on which this command should be executed
			      --collections-config string      The fully qualified path to the collection JSON file including the file name
			      --connectionProfile string       Connection profile that provides the necessary connection information for the network. Note: currently only supported for providing peer connection information
			  -c, --ctor string                    Constructor message for the chaincode in JSON format (default "{}")
			  -E, --escc string                    The name of the endorsement system chaincode to be used for this chaincode
			  -h, --help                           help for upgrade
			  -l, --lang string                    Language of chaincode, either "golang" (default), "node", or "java"
			  -n, --name string                    Name of the chaincode
			  -p, --path string                    Path to chaincode, for "golang" use relative path from $GOPATH/src, for "node" or "java" use absolute path
			      --peerAddresses stringArray      The addresses of the peers to connect to
			  -P, --policy string                  The endorsement policy associated to this chaincode
			      --tlsRootCertFiles stringArray   If TLS is enabled, the paths to the TLS root cert files of the peers to connect to. The order and number of certs specified should match the --peerAddresses flag
			  -v, --version string                 Version of the chaincode specified in install/instantiate/upgrade commands
			  -V, --vscc string                    The name of the verification system chaincode to be used for this chaincode
	- 全局标志

		      --cafile string                       Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
		      --certfile string                     Path to file containing PEM-encoded X509 public key to use for mutual TLS communication with the orderer endpoint
		      --clientauth                          Use mutual TLS when communicating with the orderer endpoint
		      --connTimeout duration                Timeout for client to connect (default 3s)
		      --keyfile string                      Path to file containing PEM-encoded private key to use for mutual TLS communication with the orderer endpoint
		      -o, --orderer string                      Ordering service endpoint
		      --ordererTLSHostnameOverride string   The hostname override to use when validating the TLS connection to the orderer.
		      --tls                                 Use TLS when communicating with the orderer endpoint
		      --transient string                    Transient map of arguments in JSON encoding
	- 示例代码
		- 下面的示例使用 `peer chaincode upgrade` 命令将版本1.0的 mycc 链码升级为版本1.1，该新版本链码包含了一个新的变量c。在启用TLS的网络中使用全局标志 `--tls` 和 `--cafile`：

				export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
				peer chaincode upgrade -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n mycc -v 1.1 -c '{"Args":["init","a","100","b","200","c","300"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
				.
				.
				.
				2018-02-22 18:26:31.433 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
				2018-02-22 18:26:31.434 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
				2018-02-22 18:26:31.435 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 005 java chaincode enabled
				2018-02-22 18:26:31.435 UTC [chaincodeCmd] upgrade -> DEBU 006 Get upgrade proposal for chaincode <name:"mycc" version:"1.1" >
				.
				.
				.
				2018-02-22 18:26:46.687 UTC [chaincodeCmd] upgrade -> DEBU 009 endorse upgrade proposal, get response <status:200 message:"OK" payload:"\n\004mycc\022\0031.1\032\004escc\"\004vscc*,\022\014\022\n\010\001\022\002\010\000\022\002\010\001\032\r\022\013\n\007Org1MSP\020\003\032\r\022\013\n\007Org2MSP\020\0032f\n \261g(^v\021\220\240\332\251\014\204V\210P\310o\231\271\036\301\022\032\205fC[|=\215\372\223\022 \311b\025?\323N\343\325\032\005\365\236\001XKj\004E\351\007\247\265fu\305j\367\331\275\253\307R\032 \014H#\014\272!#\345\306s\323\371\350\364\006.\000\356\230\353\270\263\215\217\303\256\220i^\277\305\214: \375\200zY\275\203}\375\244\205\035\340\226]l!uE\334\273\214\214\020\303\3474\360\014\234-\006\315B\031\022\010\022\006\010\001\022\002\010\000\032\r\022\013\n\007Org1MSP\020\001" >
				.
				.
				.
				2018-02-22 18:26:46.693 UTC [chaincodeCmd] upgrade -> DEBU 00c Get Signed envelope
				2018-02-22 18:26:46.693 UTC [chaincodeCmd] chaincodeUpgrade -> DEBU 00d Send signed envelope to orderer
				2018-02-22 18:26:46.908 UTC [main] main -> INFO 00e Exiting.....
		- 下面的示例在一个禁用TLS的网络上仅使用命令特定的选项：

				~$ peer chaincode upgrade -o orderer.example.com:7050 -C mychannel -n mycc -v 1.1 -c '{"Args":["init","a","100","b","200","c","300"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
				.
				.
				.
				2018-02-22 18:28:31.433 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
				2018-02-22 18:28:31.434 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
				2018-02-22 18:28:31.435 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 005 java chaincode enabled
				2018-02-22 18:28:31.435 UTC [chaincodeCmd] upgrade -> DEBU 006 Get upgrade proposal for chaincode <name:"mycc" version:"1.1" >
				.
				.
				.
				2018-02-22 18:28:46.687 UTC [chaincodeCmd] upgrade -> DEBU 009 endorse upgrade proposal, get response <status:200 message:"OK" payload:"\n\004mycc\022\0031.1\032\004escc\"\004vscc*,\022\014\022\n\010\001\022\002\010\000\022\002\010\001\032\r\022\013\n\007Org1MSP\020\003\032\r\022\013\n\007Org2MSP\020\0032f\n \261g(^v\021\220\240\332\251\014\204V\210P\310o\231\271\036\301\022\032\205fC[|=\215\372\223\022 \311b\025?\323N\343\325\032\005\365\236\001XKj\004E\351\007\247\265fu\305j\367\331\275\253\307R\032 \014H#\014\272!#\345\306s\323\371\350\364\006.\000\356\230\353\270\263\215\217\303\256\220i^\277\305\214: \375\200zY\275\203}\375\244\205\035\340\226]l!uE\334\273\214\214\020\303\3474\360\014\234-\006\315B\031\022\010\022\006\010\001\022\002\010\000\032\r\022\013\n\007Org1MSP\020\001" >
				.
				.
				.
				2018-02-22 18:28:46.693 UTC [chaincodeCmd] upgrade -> DEBU 00c Get Signed envelope
				2018-02-22 18:28:46.693 UTC [chaincodeCmd] chaincodeUpgrade -> DEBU 00d Send signed envelope to orderer
				2018-02-22 18:28:46.908 UTC [main] main -> INFO 00e Exiting.....
												
### peer version
`peer version` 命令展示节点旳版本信息，包括版本号、提交sha、go版本、 os/cpu架构以及链码信息。例如：

	peer:
	 Version: 1.4.0
	 Commit SHA: 0efc897
	 Go version: go1.11.1
	 OS/Arch: linux/amd64
	 Chaincode:
	  Base Image Version: 0.4.14
	  Base Docker Namespace: hyperledger
	  Base Docker Label: org.hyperledger.fabric
	  Docker Namespace: hyperledger
- 使用方法

	显示fabric的peer节点服务器的当前版本信息。

		~$ peer version [flags]
- 标志

		-h, --help   help for version	
		
### peer logging
管理者可以使用 `peer logging` 子命令来动态查看或配置节点旳日志等级。

- 命令语法

	`peer logging` 命令包含以下子命令

	- getlogspec
	- setlogspec
以下子命令已启用，将在将来版本中移除

	- getlevel
	- setlevel
	- revertlevels
不同的子命令选项(getlogspec, setlogspec, getlevel, setlevel, and revertlevels) 对应不同的日志级别操作。
- 使用方法

		~$ peer logging [command]
- 可用命令

		  getlevel     Returns the logging level of the requested logger.
		  getlogspec   Returns the active log spec.
		  revertlevels Reverts the logging spec to the peer's spec at startup.
		  setlevel     Adds the logger and log level to the current logging spec.
		  setlogspec   Sets the logging spec.
- 命令标志

		-h, --help   help for logging

使用 `peer logging [command] --help` 获取关于特定命令的进一步信息。

- `peer logging getlevel`

	返回所请求日志记录器的日志等级。注意，日志记录器名称应当与日志中显示的名称 精确匹配。

	- 使用方法

			~$ peer logging getlevel <logger> [flags]
	- 命令标志

			-h, --help   help for getlevel
	- 示例代码

		下面的示例获取peer日志记录器的日志等级：

			~$ peer logging getlevel peer

			2018-11-01 14:18:11.276 UTC [cli.logging] getLevel -> INFO 001 Current log level for logger 'peer': INFO	
- `peer logging revertlevels`

	将日志规范恢复为启动时的默认参数。
	
	- 使用方法
	
			~$ peer logging revertlevels [flags]
	- 命令参数
	
			-h, --help  help for revertlevels
	- 示例代码
	
		下面的示例将日志设置恢复为启动值
		
			~$ peer logging revertlevels
	
			2018-11-01 14:37:12.402 UTC [cli.logging] revertLevels -> INFO 001 Logging spec reverted to the peer	
- `peer logging setlevel`
将日志记录器和日志等级添加到当前的日志规范中。

	- 使用方法
	
			~$ peer logging setlevel <logger> <log level> [flags]
	- 命令标志
	
			-h, --help   help for setlevel
	- 示例代码
	
		下面的示例将名称具有gossip前缀的日志记录器的日志等级设置为WARNING：
	
			~$ peer logging setlevel gossip warning
			
			2018-11-01 14:21:55.509 UTC [cli.logging] setLevel -> INFO 001 Log level set for logger name/prefix 'gossip': WARNING
		如果要精确地指定一个名称，可以添加点后缀。例如下面的示例设置名为gossip的日志记录器等级为ERROR：
	
			~$ peer logging setlevel gossip. error
		
			2018-11-01 14:27:33.080 UTC [cli.logging] setLevel -> INFO 001 Log level set for logger name/prefix 'gossip

## 整体流程
### 1 生成配置(configtxgen 命令)
在一个 peer 节点

- 生成通道配置
- 生成锚点配置

### 2 创建通道(peer 命令)
在生成配置的相同 peer 节点

- 创建通道(只创建 channel 存在于排序服务)

	`peer channel create`
	
	创建后会生成一个对应 channel 的 block 文件

### 3 加入通道	
- 加入通道(加入 channel peer 才能感知)

	`peer channel join`
- 查看通道是否建立完成
	- 使用 fabric exporter 查看
	- 或者使用 peer channel list
	- 操作可以通过  cli 专属容器来操作多个 peer 

		通过环境变量区分是哪一个 peer ，环境变量名为 `CORE_PEER_ADDERSS`，注意这里的操作步骤是针对一个一个节点，而非统一节点

### 4 设置加入的 peer 为锚节点		
在添加的 peer 中指定组织锚节点		

	peer channel update -c mychannel -f ./updatechannel.txn -o orderer.example.com:7050
注意这里的	`updatechannel.txn` 就是锚节点生成出来的，事物创建方法参考`创建组织锚点配置`

### 5 部署链码(install)
一样可以通过 cli 通过环境变量来一个 peer，一个 peer 安装链码。注意这里的 peer 是服务组件总称，非背书、记账等逻辑

	peer chaincode install
### 6 实例化链码(instantiate)
一样可以通过 cli 通过环境变量来到任意一台可以有权限的 peer 实例化链码，这里和安装不同，只需要一次即可。(因为直接初始化了整个链数据  )

	peer chaincode instantiate
实例化后就可以通过 export web 或 peer 命令查询到 chaincode 已经存在和查询实例化后初始化数据对应的数据
### 7 执行链码交易(invoke)
#### 7.1 向背书节点提交预交易
	
	peer chaincode invoke
#### 7.2 背书节点通过背书将背书结果返回客户端
#### 7.3 客户端使用背书结果发送到排序服务
#### 7.4 排序服务接受交易并按照要求进行排序打包
#### 7.5 将打包结果通知 leader peer 进行记账
#### 7.6 leader 节点拿到打好的链块和记账节点进行 p2p 同步
#### 7.7 记账节点拿到区块根据背书策略进行合法性验证
#### 7.8 记账节点验证通过将区块记录到了所在的链存储中
	
## 视频中相关问题
### 证书相关问题
1. 一个 fabric 只支持1个根基证书还是多个
- user 证书的用途
	
	是给 sdk 使用的麽？还是给谁使用？如何使用？
- 可以扩展证书，但如何回收证书

	注意，使用命令生成证书的时候只是生成证书，而证书使用是系统使用，所以如果系统中删除证书的时候，只需要移除系统证书即可
- 移除证书的系统是哪个？
- 证书如何签发

	理解证书通过一个根 ca 进行签发，具体的根证书配置是通过配置文件进行配置生成(私有)。当然理论上也可以通过公有根ca来倒出其他证书，这块命令生成主要是对应的测试网络而非生产网络，生产网络主要是由 fabric ca 或者其他类似证书管理系统来做。
-  生成过程是否和ca证书有交互

	配置文件的 ca 设置就是根 ca ,私有的，然后所有其他证书，如中间件 ca 是和这个 根ca 交互
- 生成证书工具和 fabric ca 的关系

	- 一个是测试使用，通过配置文件静态生成，证书管理步骤需要全部手动
	- 一个是生产使用，动态生成并动态管理证书

### 组织和msp 问题
1. 组织和 msp 的关系, 一个组织可以对应多个 msp？

### chain code 相关问题
1. 实例化 chain code 需要在每个 peer 执行麽？

	实例化 chain code 的操作是针对链数据，而非 peer ,所以只需要一次即可

### 区块相关问题
每次 channel 的配置修改均会当作交易记录在区块中，区块中的信息包含

- 创世区块

	系统配置信息
- 制定锚节点

	系统配置信息
- 初始化链码

	链码数据
- 交易

	链码交易	

 
## 参考
- [Fabric命令手册](http://cw.hubwiz.com/card/c/fabric-command-manual/1/1/6/)
- [configtx.yaml中文注解](https://cloud.tencent.com/developer/article/1423148)		   