# geth 教学
## 入门
### 初始化
首先，使用随机主种子初始化 Clef，该种子也使用您定义的密码进行加密。密码必须至少为 10 个字符。

	clef init
### 创建账户
使用` clef newaccount ` 命令创建两个帐户，为每个帐户设置密码，并记下每个帐户的公共地址。

	Clef 输出关于 的调试消息Failed to reload keystore contents，在后面的步骤中修复

### 开始 geth
#### 网络
可以使用网络名称作为参数将 Geth 节点连接到多个不同的网络。其中包括主要的以太坊网络、您创建的私有网络以及三个使用不同共识算法的测试网络：

 - Ropsten：工作量证明测试网络
 - Rinkeby：权威证明测试网络
 - Görli：权威证明测试网络

#### 同步模式
您可以使用 `--syncmode "<mode>" ` 确定它在网络中的节点类型的参数以三种不同的同步模式之一启动 Geth 。

- `Full` 

	完整模式，下载所有区块（包括标题、交易和收据）并通过执行每个区块以增量方式生成区块链的状态。
- `Fast` 

	快速模式，下载所有区块（包括标题、交易和收据），验证所有标题，并下载状态并根据标题进行验证。
- `Snap（默认）`

	功能与快速相同，但算法更快。
- `Light` 

	轻节点模式，下载所有区块头、区块数据，并随机验证一些。

使用 `light` 同步

### 启动 clef 
启动 Clef，为我们要连接的网络设置密钥库和链 ID（goerli 为 5

	clef --keystore <GETH_DATA_DIR>/keystore --chainid 5
您会看到有关缺少密钥库的错误，我们很快就会修复它。

Linux 下默认的 Geth 数据目录是 `~/.ethereum`

### 开始 Geth
打开另一个命令行窗口并运行下面的命令，这也同时会启动 Geth RPC 接口（见下文），并将 Clef 设置为交易签名者。

	geth --goerli --syncmode "light" --http --signer=<CLEF_LOCATION>/clef.ipc
默认情况下，Linux 下 Clef 的位置是 `~/.clef`，但签名者的位置不能包含 ，`~` 因此请将其替换为您的主目录。
### 获得 ETH
除非您在 Görli 网络上的另一个帐户中有 ETH，否则您可以使用 水龙头将 ETH 发送到您的一个新帐户地址以用于本指南。
### 链接使用 IPC 或 RPC 连接到 Geth
您可以通过两种方式与 Geth 交互

- 通过 IPC 使用 JavaScript 控制台直接与节点进行交互

	IPC 允许您做更多的事情，尤其是在创建帐户和与帐户交互时，但您需要直接访问节点。
- 或者使用 RPC 通过 HTTP 远程连接到节点。

	RPC 允许远程应用程序访问您的节点，但有限制和安全考虑，并且默认情况下只允许访问 `eth` 和 `shh` 命名空间中的方法。在 RPC 文档中了解如何覆盖此设置。

### 使用 IPC
#### 连接到控制台
从另一个终端窗口连接到节点上的 IPC 控制台

	geth attach <IPC_LOCATION>
可以在网络节点 geth 进程的输出中看到 IPC 位置。默认情况下，当使用 Görli 时，它是 `IPC_LOCATION=~/.ethereum/goerli/geth.ipc`.
#### 查看账户余额
获取账户余额不需要签署交易，因此 Clef 不要求批准，Geth 返回值。

请注意，此步骤需要初始同步结束。如果收到错误消息，请返回带有网络节点 Geth 的控制台并等待它同步。您知道您的 Geth 是同步的，因为它一次只导入少量块（通常是一两个）。

	web3.fromWei(eth.getBalance("<ADDRESS_1>"),"ether")
#### 发送 ETH 到账户
从您使用 Görli faucet 添加 ETH 的帐户发送 0.01 ETH 到您创建的第二个帐户：
		
	eth.sendTransaction({from:"<ADDRESS_1>",to:"<ADDRESS_2>", value: web3.toWei(0.01,"ether")})
此操作确实需要对交易进行签名，因此在 Clef 运行的情况下转到命令行窗口以查看 Clef 提示您批准它，并且当您这样做时，会询问您发送 ETH 的帐户的密码。如果密码正确，Geth 将继续进行交易。

要检查，获取第二个帐户的帐户余额：		

	web3.fromWei(eth.getBalance("<ADDRESS_2>"),"ether")
### 使用RPC
#### 连接到 RPC
您可以使用标准 HTTP 请求通过 RPC API 连接到 Geth 节点，使用以下语法：

	curl -X POST http://<GETH_IP_ADDRESS>:8545 \
	    -H "Content-Type: application/json" \
	   --data'{"jsonrpc":"2.0", "method":"<API_METHOD>", "params":[], "id":1}'
#### 查看账户余额
获取账户余额不需要签署交易，因此 Geth 无需调用 Clef 即可返回该值。请注意，返回的值是十六进制和 WEI。要获得 ETH 值，请转换为十进制并除以 10^18。

	curl -X POST http://<GETH_IP_ADDRESS>:8545 \
	    -H "Content-Type: application/json" \
	   --data '{"jsonrpc":"2.0", "method":"eth_getBalance", "params":["<ADDRESS_1>","latest"], "id":1}'
#### 将 ETH 发送到账户
添加 ETH 的帐户发送 0.01 ETH 到您创建的第二个帐户

	curl -X POST http://<GETH_IP_ADDRESS>:8545 \
	    -H "Content-Type: application/json" \
	   --data '{"jsonrpc":"2.0", "method":"eth_sendTransaction", "params":[{"from": "<ADDRESS_1>","to": "<ADDRESS_2>","value": "0x9184e72a"}], "id":1}'
此操作确实需要签名，因此 Clef 会提示您批准它，如果您批准，则会询问您发送 ETH 的帐户密码。如果密码正确，Geth 将继续进行交易。

要检查，获取第二个帐户的帐户余额：

	curl -X POST http://<GETH_IP_ADDRESS>:8545 \
	    -H "Content-Type: application/json" \
	    --data '{"jsonrpc":"2.0", "method":"eth_getBalance", "params":["<ADDRESS_2>","latest"], "id":1}
### 开发模式
Geth 有一种开发模式，可以建立一个单节点以太坊测试网络，其中包含为在本地机器上开发而优化的选项。您可以使用 `--dev` 参数启用它。

在开发模式下启动 geth 执行以下操作：

- 使用测试创世块初始化数据目录
- 将最大对等点设置为 0
- 关闭其他节点的发现
- 将 gas 价格设置为 0
- 使用 Clique PoA 共识引擎，允许根据需要挖掘块，而不会消耗过多的 CPU 和内存
- 使用按需块生成，在交易等待挖掘时生成块

#### 在开发模式下启动 Geth
- 您可以使用该 `--datadir` 选项指定一个数据目录来维护两次运行之间的状态，否则，数据库是临时的并且在内存中：

		mkdir test-chain-dir
- 对于本指南，在开发模式下启动 geth，并启用 RPC 以便您可以将其他应用程序连接到 geth。使用基于 Web 的以太坊 IDE Remix，因此也允许其域接受跨域请求。

		geth --datadir test-chain-dir --http --dev --http.corsdomain "https://remix.ethereum.org,http://remix.ethereum.org"
- 从另一个终端窗口连接到节点上的 IPC 控制台

		geth attach <IPC_LOCATION>

- 一旦 geth 在开发模式下运行，您就可以像当 geth 以其他方式运行时一样与它交互。例如，创建一个测试帐户

		personal.newAccount()
- 并将以太币从 coinbase 转移到新账户

		eth.sendTransaction({from:eth.coinbase, to:eth.accounts[1], value: web3.toWei(0.05, "ether")})
- 查看账户余额

		eth.getBalance(eth.accounts[1])

如果你想用真实的区块时间测试你的 dapps，请在使用参数 `--dev.period` 启动开发模式时使用该选项 `--dev.period 14`。

#### 将 Remix 连接到 Geth
- 现在运行 geth，打开 https://remix.ethereum.org。
- 正常编译合约，但在部署和运行合约时，从 Environment 下拉列表中选择 Web3 Provider，并在弹出框中添加“http://127.0.0.1:8545”。
- 单击Deploy，并与合约进行交互。

您应该会看到合约创建、挖掘和交易活动。

### 专网教程
本页介绍了如何设置本地节点集群，建议如何将其设为私有，以及如何在 eth-netstat 网络监控应用程序上连接您的节点。完全受控的以太坊网络可用作网络集成测试的后端（核心开发人员致力于与网络/区块链同步/消息传播等相关的问题，或者 DAPP 开发人员测试多块和多用户场景）。

我们假设您能够 geth 按照构建说明进行构建。

- 设置多个节点

	为了在本地运行多个以太坊节点，您必须确保：
	
	- 每个实例都有一个单独的数据目录 ( `--datadir`)
	- 每个实例都在不同的端口上运行（eth 和 rpc） ( `--port` and `--http.port`)
	- 在集群的情况下，实例必须相互了解
	- ipc 端点是唯一的或 ipc 接口被禁用 ( `--ipcpath` or `--ipcdisable`)
	
	您启动第一个节点（让我们明确端口并禁用 ipc 接口）
	
		geth --datadir="/tmp/eth/60/01" -verbosity 6 --ipcdisable --port 30301 --http.port 8101 console 2>> /tmp/eth/60/01.log
	我们使用控制台启动节点，以便我们可以获取 enode url，例如：
	
		admin.nodeInfo.enode
		enode://8c544b4a07da02a9ee024def6f3ba24b2747272b64e16ec5dd6b17b55992f8980b77938155169d9d33807e501729ecb42f5c0a61018898c32799ced152e9f0d7@9[::]:30301
	`[::]` 将被解析为 `localhost ( 127.0.0.1)`。如果您的节点在本地网络上，请检查每台主机并找到您的 IP `ifconfig`（在 Linux 和 MacOS 上）：
	
		$ ifconfig|grep netmask|awk '{print $2}'
		127.0.0.1
		192.168.1.97
	如果你的 peer 不在本地网络上，你需要知道你的外部 IP 地址（使用服务）来构建 enode url。

	现在您可以使用以下命令启动第二个节点
	
		geth --datadir="/tmp/eth/60/02" --verbosity 6 --ipcdisable --port 30302 --http.port 8102 console 2>> /tmp/eth/60/02.log 
	如果您想将此实例连接到之前启动的节点，您可以使用 `admin.addPeer(enodeUrlOfFirstInstance)`

	您可以通过在 geth 控制台中键入来测试连接：

		> net.listening
		true
		> net.peerCount 
		1
		> admin.peers
		...
- 本地集群

	作为上述内容的扩展，您可以轻松生成本地节点集群。它还可以编写脚本，包括创建挖矿所需的帐户。有关 [gethcluster.sh](https://github.com/ethersphere/eth-utils) 用法和示例，请参阅 脚本和自述文件。
	
	- 设置引导节点

		节点第一次连接到网络时，它使用一个预定义的 [引导节点](https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go)。通过这些引导节点，一个节点可以加入网络并找到其他节点。在私有集群的情况下，这些预定义的引导节点没有多大用处。因此，go-ethereum 提供了一个引导节点实现，可以在您的私有网络中配置和运行。

		它可以通过命令运行
		
			> bootnode
			Fatal: Use -nodekey or -nodekeyhex to specify a private key
		可以看出，引导节点要求输入密钥。每个以太坊节点，包括一个引导节点，都由一个 enode 标识符标识。这些标识符是从密钥派生的。因此，您需要为 bootnode 提供这样的密钥。
		
		由于我们目前没有，我们可以指示引导节点在启动之前生成一个密钥（并将其存储在一个文件中）。
		
			> bootnode -genkey bootnode.key
			I0216 09:53:08.076155 p2p/discover/udp.go:227] Listening, enode://890b6b5367ef6072455fedbd7a24ebac239d442b18c5ab9d26f58a349dad35ee5783a0dd543e4f454fed22db9772efe28a3ed6f21e75674ef6203e47803da682@
			
			(exit with CTRL-C)
		存储的密钥可以通过以下方式查看
		
			> cat bootnode.key
			dc90f8f7324f1cc7ba52c4077721c939f98a628ed17e51266d01c9cd0294033a
		要指示 geth 节点使用我们自己的引导节点，请使用该 `--bootnodes` 标志。这是一个以逗号分隔的 bootnode enode 标识符列表
		
			geth --bootnodes "enode://890b6b5367ef6072455fedbd7a24ebac239d442b18c5ab9d26f58a349dad35ee5783a0dd543e4f454fed22db9772efe28a3ed6f21e75674ef6203e47803da682@[::]:30301"
		由于每次使用相同的 enode 启动 bootnode 很方便，我们可以在下次启动时给 bootnode 程序刚刚生成的密钥。
		
			bootnode -nodekey bootnode.key
			I0216 10:01:19.125600 p2p/discover/udp.go:227] Listening, enode://890b6b5367ef6072455fedbd7a24ebac239d442b18c5ab9d26f58a349dad35ee5783a0dd543e4f454fed22db9772efe28a3ed6f21e75674ef6203e47803da682@[::]:30301
		或者
		
			bootnode -nodekeyhex dc90f8f7324f1cc7ba52c4077721c939f98a628ed17e51266d01c9cd0294033a
			I0216 10:01:40.094089 p2p/discover/udp.go:227] Listening, enode://890b6b5367ef6072455fedbd7a24ebac239d442b18c5ab9d26f58a349dad35ee5783a0dd543e4f454fed22db9772efe28a3ed6f21e75674ef6203e47803da682@[::]:30301

## 安装与构建
### 安装 Geth
您可以使用多种方式安装以太坊的 Go 实现。这些包括通过您最喜欢的包管理器安装它；下载一个独立的预构建包；作为 docker 容器运行；或自己构建。本文档详细介绍了使用您喜欢的任何方式加入以太坊网络的所有可能性。可以在此处找到稳定版本的列表。
### 更新 Geth
更新 go-ethereum 非常简单。您只需要下载并安装更新版本的 geth，关闭您的节点并使用新软件重新启动。Geth 将自动使用您旧节点的数据，并同步您关闭旧软件后挖掘的最新区块。
### 从包管理器安装
- 通过 Homebrew 在 macOS 上安装
- 通过 PPA 在 Ubuntu 上安装
- 在 Windows 上安装
- 通过 pkg 在 FreeBSD 上安装
- 通过端口安装在 FreeBSD 上
- 通过在 Arch Linux 上安装 pacman

### [下载独立包](https://geth.ethereum.org/downloads/)
### 在 Docker 容器内运行
如果您更喜欢容器化进程，我们会使用我们 develop 在 DockerHub 上的分支的最新快照构建来维护 Docker 镜像。我们维护四个不同的 Docker 镜像，用于运行 Geth 的最新稳定版本或开发版本。

- ethereum/client-go:latest 是 Geth 的最新开发版本（默认）
- ethereum/client-go:stable 是 Geth 的最新稳定版本
- ethereum/client-go:{version} 是特定版本号的 Geth 的稳定版本
- ethereum/client-go:release-{version} 是特定版本系列的 Geth 的最新稳定版本

要拉取映像并启动节点，请运行以下命令：

	docker pull ethereum/client-go
	docker run -it -p 30303:30303 ethereum/client-go
我们还维护了四个不同的 Docker 镜像，用于运行各种以太坊工具的最新稳定版本或开发版本。

- ethereum/client-go:alltools-latest 是以太坊工具的最新开发版本
- ethereum/client-go:alltools-stable 是以太坊工具的最新稳定版本
- ethereum/client-go:alltools-{version} 是特定版本号的以太坊工具的稳定版本
- ethereum/client-go:alltools-release-{version} 是特定版本系列中以太坊工具的最新稳定版本

该图像具有以下自动公开的端口：

- 8545 TCP，由基于 HTTP 的 JSON RPC API 使用
- 8546 TCP，由基于 WebSocket 的 JSON RPC API 使用
- 8547 TCP，由 GraphQL API 使用
- 30303 TCP 和 UDP，由运行网络的 P2P 协议使用

请注意，如果您在 Docker 容器中运行以太坊客户端，则应将数据卷挂载为客户端的数据目录（位于/root/.ethereum容器内部），以确保在重新启动和/或容器生命周期之间保留下载的数据。

### 从源代码构建 go-ethereum
- 大多数 Linux 系统和 macOS

	Go Ethereum 是用 Go 编写的，因此要从源代码构建，您需要最新版本的 Go。本指南不包括如何安装 Go 本身，有关详细信息，请阅读Go 安装说明并从Go 下载页面获取任何需要的包。

	安装 Go 后，您可以通过以下方式将项目下载到您的 GOPATH 工作区：

		go get -d github.com/ethereum/go-ethereum
	您还可以通过以下方式安装特定版本：

		go get -d github.com/ethereum/go-ethereum@v1.9.21
	上述命令不构建任何可执行文件。为此，您可以专门构建一个：

		go install github.com/ethereum/go-ethereum/cmd/geth
	或者，您可以构建整个项目并geth通过go install ./...在工作空间ethereum/go-ethereum内的存储库根目录中 运行来与所有开发人员工具一起安装GOPATH。

	如果您使用的是 macOS 并看到与 macOS 头文件相关的错误，请使用 安装 XCode 命令行工具 `xcode-select --install`，然后重试。

	如果您遇到 `go: cannot use path@version syntax in GOPATH mode` 或类似的错误，请使用 `export GO111MODULE=on.`
- win
- freebsd
- 不使用 Go 工作流构建

### 备份和恢复
首先是最重要的信息：`记住您的密码并备份您的密钥库`。

- 数据目录

	所有内容都 geth 被写入其数据目录中。默认数据目录位置特定于平台：
	
	- mac： ~/Library/Ethereum
	- Linux： ~/.ethereum
	- win： %APPDATA%\Ethereum

	帐户存储在 `keystore` 子目录中。该目录的内容应该可以在节点、平台、实现（C++、Go、Python）之间传输。

	要配置数据目录的位置，`--datadir` 可以指定该参数。有关更多详细信息，请参阅 CLI 选项。

	请注意，[ethash dag](https://geth.ethereum.org/docs/interface/mining) 存储在 `~/.ethash(Mac/Linux)` 或 `%APPDATA%\Ethash(Windows)` 中，以便所有客户端都可以重复使用。您可以使用符号链接将其存储在不同的位置。
- 清理

	可以通过以下方式删除 Geth 的区块链和状态数据库：

		geth removedb
	这对于删除旧链并同步到新链非常有用。它只影响可以在同步时重新创建的数据目录，不涉及密钥库。
- 区块链导入/导出
	- 导出

		以二进制格式导出区块链

			geth export <filename>
		或者，如果您想随着时间的推移备份部分链，可以指定第一个和最后一个块。例如，要备份第一个纪元：

			geth export <filename> 0 29999
		请注意，在备份部分链时，文件将被追加而不是被截断。
	- 导入

		使用以下命令导入二进制格式的区块链导出

			geth import <filename>

	有关更多信息，请参阅 https://eth.wiki/en/howto/blockchain-import-and-export-instructions

	最后：`记住您的密码并备份您的密钥库`
	
### 交叉编译 Geth
注意：所有这些以及更多内容都已合并到项目 Makefile 中。您可以通过交叉构建 `make geth-<os>-<platform>` 而无需从下面了解任何这些详细信息。

开发人员通常有一个他们觉得最舒服的首选平台，并为最佳工作流程设置了所有必要的工具、库和环境。但是，通常需要为不同的 CPU 架构或完全不同的操作系统构建；但是为每个维护一个开发环境并在它们之间切换很快变得笨拙。

在这里，我们提出了一种非常简单的方法，使用最少的先决条件和完全容器化的方法将以太坊交叉编译到各种操作系统和架构，确保即使在交叉编译的复杂要求和机制之后，您的开发环境也能保持干净。

目前支持的目标平台有：

- ARMv7 Android 和 iOS
- 32 位、64 位和 ARMv5 Linux
- 32 位和 64 位 Mac OSX
- 32 位和 64 位 Windows

请注意，交叉编译不会取代发布版本。尽管生成的二进制文件通常可以在所需的平台上完美运行，但使用官方供应商提供的专用工具在本机系统上编译通常可以产生更精细优化的代码。

- 交叉编译环境
- 构建以太坊
- 微调构建
- 本地建设
- 使用 Makefile
- 调整交叉构建	

## 使用 Geth
### 命令行选项
	$ geth --help
	NAME:
	   geth - the go-ethereum command line interface
	
	   Copyright 2013-2021 The go-ethereum Authors
	
	USAGE:
	   geth [options] [command] [command options] [arguments...]
	
	VERSION:
	   1.10.10-stable-bb74230f
	
	COMMANDS:
	   account                            Manage accounts //管理账户
	   attach                             Start an interactive JavaScript environment (connect to node) //启动一个交互式 JavaScript 环境（连接到节点）
	   console                            Start an interactive JavaScript environment // 启动交互式 JavaScript 环境
	   db                                 Low level database operations //低级数据库操作
	   dump                               Dump a specific block from storage // 从存储中转储特定块
	   dumpconfig                         Show configuration values //显示配置值
	   dumpgenesis                        Dumps genesis block JSON configuration to stdout // 将创世块 JSON 配置转储到标准输出
	   export                             Export blockchain into file //将区块链导出到文件中
	   export-preimages                   Export the preimage database into an RLP stream //将原像数据库导出到 RLP 流中
	   import                             Import a blockchain file // 导入区块链文件
	   import-preimages                   Import the preimage database from an RLP stream // 从 RLP 流导入原像数据库
	   init                               Bootstrap and initialize a new genesis block //并初始化一个新的创世区块
	   js                                 Execute the specified JavaScript files // 执行指定的 JavaScript 文件
	   license                            Display license information // 显示许可证信息
	   makecache                          Generate ethash verification cache (for testing) // 生成ethash验证缓存（用于测试）
	   makedag                            Generate ethash mining DAG (for testing) // 生成 ethash 挖矿 DAG（用于测试）
	   removedb                           Remove blockchain and state databases // 删除区块链和状态数据库
	   show-deprecated-flags              Show flags that have been deprecated // 显示已弃用的标志
	   snapshot                           A set of commands based on the snapshot // 快照基于快照的一组命令
	   version                            Print version numbers // 打印版本号
	   version-check                      Checks (online) whether the current version suffers from any known security vulnerabilities // 检查（在线）当前版本是否存在任何已知的安全漏洞
	   wallet                             Manage Ethereum presale wallets // 管理以太坊预售钱包
	   help, h                            Shows a list of commands or help for one command // 显示命令列表或一个命令的帮助
	
	ETHEREUM OPTIONS: //以太坊设置
	  --config value                      TOML configuration file // TOML 配置文件
	  --datadir value                     Data directory for the databases and keystore (default: "~/.ethereum") // 数据库和密钥库的数据目录（默认：“~/.ethereum”）
	  --datadir.ancient value             Data directory for ancient chain segments (default = inside chaindata) // 祖先链段的数据目录（默认 = 内链数据）
	  --datadir.minfreedisk value         Minimum free disk space in MB, once reached triggers auto shut down (default = --cache.gc converted to MB, 0 = disabled) // 最小可用磁盘空间（以 MB 为单位），一旦达到将触发自动关闭（默认 = --cache.gc 转换为 MB，0 = 禁用）
	  --keystore value                    Directory for the keystore (default = inside the datadir) //密钥库的目录（默认 = 在 datadir 内）
	  --usb                               Enable monitoring and management of USB hardware wallets // 启用 USB 硬件钱包的监控和管理
	  --pcscdpath value                   Path to the smartcard daemon (pcscd) socket file //智能卡守护程序 (pcscd) 套接字文件的路径
	  --networkid value                   Explicitly set network id (integer)(For testnets: use --ropsten, --rinkeby, --goerli instead) (default: 1) 值显式设置网络 ID（整数）（对于测试网：使用 --ropsten、--rinkeby、--goerli 代替）（默认值：1）
	  --mainnet                           Ethereum mainnet // 以太坊主网
	  --goerli                            Görli network: pre-configured proof-of-authority test network // Görli 网络 预先配置的权威证明测试网络
	  --rinkeby                           Rinkeby network: pre-configured proof-of-authority test network // Rinkeby 网络 预先配置的权威证明测试网络
	  --ropsten                           Ropsten network: pre-configured proof-of-work test network // Ropsten 网络 预先配置的工作量证明测试网络
	  --syncmode value                    Blockchain sync mode ("fast", "full", "snap" or "light") (default: snap) // 区块链同步模式（“fast”、“full”、“snap”或“light”）（默认值：snap）
	  --exitwhensynced                    Exits after block synchronisation completes // 块同步完成后退出
	  --gcmode value                      Blockchain garbage collection mode ("full", "archive") (default: "full") // 区块链垃圾收集模式（“full”、“archive”）（默认值：“full”）
	  --txlookuplimit value               Number of recent blocks to maintain transactions index for (default = about one year, 0 = entire chain) (default: 2350000) // 最近维护交易索引的区块数（默认值 = 大约一年，0 = 整个链）（默认值：2350000）
	  --ethstats value                    Reporting URL of a ethstats service (nodename:secret@host:port) // 服务的报告 URL (nodename:secret@host:port)
	  --identity value                    Custom node name // 自定义节点名
	  --lightkdf                          Reduce key-derivation RAM & CPU usage at some expense of KDF strength // 以牺牲 KDF 强度为代价减少密钥派生 RAM 和 CPU 使用率
	  --whitelist value                   Comma separated block number-to-hash mappings to enforce (<number>=<hash>) // 值逗号分隔的块号到哈希映射来强制执行 (<number>=<hash>)
	
	LIGHT CLIENT OPTIONS: // 轻客户端设置
	  --light.serve value                 Maximum percentage of time allowed for serving LES requests (multi-threaded processing allows values over 100) (default: 0) // 服务 LES 请求所允许的最大时间百分比（多线程处理允许值超过 100）（默认值：0）
	  --light.ingress value               Incoming bandwidth limit for serving light clients (kilobytes/sec, 0 = unlimited) (default: 0) // 服务轻客户端的传入带宽限制（千字节/秒，0 = 无限制）（默认值：0）
	  --light.egress value                Outgoing bandwidth limit for serving light clients (kilobytes/sec, 0 = unlimited) (default: 0) // 服务轻客户端的传出带宽限制（千字节/秒，0 = 无限制）（默认值：0）
	  --light.maxpeers value              Maximum number of light clients to serve, or light servers to attach to (default: 100) // 要服务的轻客户端或要附加的轻服务器的最大数量（默认值：100）
	  --ulc.servers value                 List of trusted ultra-light servers // 可信超轻型服务器列表
	  --ulc.fraction value                Minimum % of trusted ultra-light servers required to announce a new head (default: 75) // 宣布新头所需的可信超轻型服务器的最小百分比（默认值：75）
	  --ulc.onlyannounce                  Ultra light server sends announcements only // 超轻服务器只发送公告
	  --light.nopruning                   Disable ancient light chain data pruning // 禁用祖先轻链数据修剪
	  --light.nosyncserve                 Enables serving light clients before syncing // 在同步前启用服务轻客户端
	
	DEVELOPER CHAIN OPTIONS: // 开发链设置
	  --dev                               Ephemeral proof-of-authority network with a pre-funded developer account, mining enabled // 具有预先资助的开发者帐户的临时权威证明网络，已启用挖矿
	  --dev.period value                  Block period to use in developer mode (0 = mine only if transaction pending) (default: 0) // 在开发者模式下使用的区块周期（0 = 仅当交易挂起时我的）（默认值：0）
	
	ETHASH OPTIONS: // eth 哈希设置
	  --ethash.cachedir value             Directory to store the ethash verification caches (default = inside the datadir) // 存储 ethash 验证缓存的目录（默认 = 在 datadir 内）
	  --ethash.cachesinmem value          Number of recent ethash caches to keep in memory (16MB each) (default: 2) // 要保存在内存中的最近 ethash 缓存的数量（每个 16MB）（默认值：2）
	  --ethash.cachesondisk value         Number of recent ethash caches to keep on disk (16MB each) (default: 3) // 要保留在磁盘上的最近 ethash 缓存的数量（每个 16MB）（默认值：3）
	  --ethash.cacheslockmmap             Lock memory maps of recent ethash caches // 锁定最近 ethash 缓存的内存映射
	  --ethash.dagdir value               Directory to store the ethash mining DAGs (default: "~/.ethash") // 存放 ethash 挖矿 DAG 的目录（默认：“~/.ethash”）
	  --ethash.dagsinmem value            Number of recent ethash mining DAGs to keep in memory (1+GB each) (default: 1) // 要保留在内存中的最近 ethash 挖掘 DAG 的数量（每个 1+GB）（默认值：1）
	  --ethash.dagsondisk value           Number of recent ethash mining DAGs to keep on disk (1+GB each) (default: 2) // 最近要保留在磁盘上的 ethash 挖掘 DAG 的数量（每个 1+GB）（默认值：2）
	  --ethash.dagslockmmap               Lock memory maps for recent ethash mining DAGs // 锁定最近 ethash 挖掘 DAG 的内存映射
	
	TRANSACTION POOL OPTIONS: // 事务池设置
	  --txpool.locals value               Comma separated accounts to treat as locals (no flush, priority inclusion) // 逗号分隔的帐户作为本地人对待（不刷新，优先包含）
	  --txpool.nolocals                   Disables price exemptions for locally submitted transactions // 禁用本地提交交易的价格豁免
	  --txpool.journal value              Disk journal for local transaction to survive node restarts (default: "transactions.rlp") // 本地事务的磁盘日志以在节点重新启动后继续存在（默认值：“transactions.rlp”）
	  --txpool.rejournal value            Time interval to regenerate the local transaction journal (default: 1h0m0s) // 重新生成本地交易日志的时间间隔（默认：1h0m0s）
	  --txpool.pricelimit value           Minimum gas price limit to enforce for acceptance into the pool (default: 1) // 为接受入池而强制执行的最低 gas 价格限制（默认值：1）
	  --txpool.pricebump value            Price bump percentage to replace an already existing transaction (default: 10) // 替换现有交易的价格波动百分比（默认值：10）
	  --txpool.accountslots value         Minimum number of executable transaction slots guaranteed per account (default: 16) // 每个账户保证的最小可执行交易槽数（默认值：16）
	  --txpool.globalslots value          Maximum number of executable transaction slots for all accounts (default: 5120) // 所有账户的最大可执行交易槽数（默认值：5120）
	  --txpool.accountqueue value         Maximum number of non-executable transaction slots permitted per account (default: 64) // 每个账户允许的最大不可执行交易槽数（默认值：64）
	  --txpool.globalqueue value          Maximum number of non-executable transaction slots for all accounts (default: 1024) // 所有账户不可执行交易槽的最大数量（默认：1024）
	  --txpool.lifetime value             Maximum amount of time non-executable transaction are queued (default: 3h0m0s) // 不可执行事务排队的最长时间（默认值：3h0m0s）
	
	PERFORMANCE TUNING OPTIONS: //性能调整选项
	  --cache value                       Megabytes of memory allocated to internal caching (default = 4096 mainnet full node, 128 light mode) (default: 1024) // 分配给内部缓存的内存兆字节（默认 = 4096 主网全节点，128 光模式）（默认值：1024）
	  --cache.database value              Percentage of cache memory allowance to use for database io (default: 50) // 用于数据库 io 的缓存内存允许百分比（默认值：50）
	  --cache.trie value                  Percentage of cache memory allowance to use for trie caching (default = 15% full mode, 30% archive mode) (default: 15) // 用于 trie 缓存的缓存内存允许百分比（默认值 = 15% 完整模式，30% 存档模式）（默认值：15）
	  --cache.trie.journal value          Disk journal directory for trie cache to survive node restarts (default: "triecache") //磁盘日志目录，用于 trie 缓存以在节点重新启动后继续存在（默认值：“triecache”）
	  --cache.trie.rejournal value        Time interval to regenerate the trie cache journal (default: 1h0m0s) // 重新生成trie缓存日志的时间间隔（默认：1h0m0s）
	  --cache.gc value                    Percentage of cache memory allowance to use for trie pruning (default = 25% full mode, 0% archive mode) (default: 25) // 用于修剪修剪的缓存内存百分比（默认值 = 25% 完整模式，0% 存档模式）（默认值：25）
	  --cache.snapshot value              Percentage of cache memory allowance to use for snapshot caching (default = 10% full mode, 20% archive mode) (default: 10) // 用于快照缓存的缓存内存允许百分比（默认值 = 10% 完整模式，20% 存档模式）（默认值：10）
	  --cache.noprefetch                  Disable heuristic state prefetch during block import (less CPU and disk IO, more time waiting for data) //在块导入期间禁用启发式状态预取（更少的 CPU 和磁盘 IO，更多的时间等待数据）
	  --cache.preimages                   Enable recording the SHA3/keccak preimages of trie keys // 启用记录特里密钥的 SHA3/keccak 原像
	
	ACCOUNT OPTIONS: // 账户选项
	  --unlock value                      Comma separated list of accounts to unlock // 逗号分隔的要解锁的帐户列表
	  --password value                    Password file to use for non-interactive password input // 用于非交互式密码输入的密码文件
	  --signer value                      External signer (url or path to ipc file) // 外部签名者（ipc 文件的 url 或路径）
	  --allow-insecure-unlock             Allow insecure account unlocking when account-related RPCs are exposed by http // 当账户相关的 RPC 被 http 暴露时，允许不安全的账户解锁
	
	API AND CONSOLE OPTIONS: // API 和控制台选项：
	  --ipcdisable                        Disable the IPC-RPC server // 禁用 IPC-RPC 服务器
	  --ipcpath value                     Filename for IPC socket/pipe within the datadir (explicit paths escape it) // 数据目录中 IPC 套接字/管道的文件名（显式路径对其进行转义）
	  --http                              Enable the HTTP-RPC server // 启用 HTTP-RPC 服务器
	  --http.addr value                   HTTP-RPC server listening interface (default: "localhost") // HTTP-RPC 服务器监听接口（默认：“localhost”）
	  --http.port value                   HTTP-RPC server listening port (default: 8545) // HTTP-RPC 服务器监听端口（默认：8545）
	  --http.api value                    API's offered over the HTTP-RPC interface // API 通过 HTTP-RPC 接口提供
	  --http.rpcprefix value              HTTP path path prefix on which JSON-RPC is served. Use '/' to serve on all paths. // 提供 JSON-RPC 的 HTTP 路径路径前缀。使用“/”在所有路径上提供服务。
	  --http.corsdomain value             Comma separated list of domains from which to accept cross origin requests (browser enforced) // 逗号分隔的域列表，从中接受跨源请求（浏览器强制执行）
	  --http.vhosts value                 Comma separated list of virtual hostnames from which to accept requests (server enforced). Accepts '*' wildcard. (default: "localhost") // 逗号分隔的虚拟主机名列表，从中接受请求（服务器强制执行）。接受“*”通配符。 （默认：“本地主机”）
	  --ws                                Enable the WS-RPC server // 启用 WS-RPC 服务器
	  --ws.addr value                     WS-RPC server listening interface (default: "localhost") // WS-RPC 服务器监听接口（默认：“localhost”）
	  --ws.port value                     WS-RPC server listening port (default: 8546) // WS-RPC 服务器侦听端口（默认值：8546）
	  --ws.api value                      API's offered over the WS-RPC interface // 通过 WS-RPC 接口提供的值 API
	  --ws.rpcprefix value                HTTP path prefix on which JSON-RPC is served. Use '/' to serve on all paths. // 提供 JSON-RPC 的 HTTP 路径前缀。使用“/”在所有路径上提供服务。
	  --ws.origins value                  Origins from which to accept websockets requests // 接受 websockets 请求的来源
	  --graphql                           Enable GraphQL on the HTTP-RPC server. Note that GraphQL can only be started if an HTTP server is started as well. // 在 HTTP-RPC 服务器上启用 GraphQL。请注意，GraphQL 只能在 HTTP 服务器启动的情况下启动。
	  --graphql.corsdomain value          Comma separated list of domains from which to accept cross origin requests (browser enforced) // 逗号分隔的域列表，从中接受跨源请求（浏览器强制执行）
	  --graphql.vhosts value              Comma separated list of virtual hostnames from which to accept requests (server enforced). Accepts '*' wildcard. (default: "localhost") // 逗号分隔的虚拟主机名列表，从中接受请求（服务器强制执行）。接受“*”通配符。 （默认：“本地主机”）
	  --rpc.gascap value                  Sets a cap on gas that can be used in eth_call/estimateGas (0=infinite) (default: 50000000) // 设置可以在 eth_call/estimateGas 中使用的 gas 上限（0=无限）（默认值：50000000）
	  --rpc.evmtimeout value              Sets a timeout used for eth_call (0=infinite) (default: 5s) // 设置用于 eth_call 的超时（0=无限）（默认值：5s）
	  --rpc.txfeecap value                Sets a cap on transaction fee (in ether) that can be sent via the RPC APIs (0 = no cap) (default: 1) // 设置可以通过 RPC API 发送的交易费用上限（以以太为单位）（0 = 无上限）（默认值：1）
	  --rpc.allow-unprotected-txs         Allow for unprotected (non EIP155 signed) transactions to be submitted via RPC // 允许通过 RPC 提交未受保护（非 EIP155 签名）的交易
	  --jspath loadScript                 JavaScript root path for loadScript (default: ".") // loadScript 的 JavaScript 根路径（默认值：“.”）
	  --exec value                        Execute JavaScript statement // 执行 JavaScript 语句
	  --preload value                     Comma separated list of JavaScript files to preload into the console // 逗号分隔的 JavaScript 文件列表以预加载到控制台
	
	NETWORKING OPTIONS: // 网络设置
	  --bootnodes value                   Comma separated enode URLs for P2P discovery bootstrap // 逗号分隔的 enode URL，用于 P2P 发现引导程序
	  --discovery.dns value               Sets DNS discovery entry points (use "" to disable DNS) // 设置 DNS 发现入口点（使用“”禁用 DNS）
	  --port value                        Network listening port (default: 30303) // 网络监听端口（默认：30303）
	  --maxpeers value                    Maximum number of network peers (network disabled if set to 0) (default: 50) // 值最大网络对等点数（如果设置为 0，则禁用网络）（默认值：50）
	  --maxpendpeers value                Maximum number of pending connection attempts (defaults used if set to 0) (default: 0) // 最大挂起连接尝试次数（如果设置为 0，则使用默认值）（默认值：0）
	  --nat value                         NAT port mapping mechanism (any|none|upnp|pmp|extip:<IP>) (default: "any") // NAT 端口映射机制 (any|none|upnp|pmp|extip:<IP>) (默认: "any")
	  --nodiscover                        Disables the peer discovery mechanism (manual peer addition) // 禁用对等发现机制（手动对等添加）
	  --v5disc                            Enables the experimental RLPx V5 (Topic Discovery) mechanism // 启用实验性 RLPx V5（主题发现）机制
	  --netrestrict value                 Restricts network communication to the given IP networks (CIDR masks) // 将网络通信限制为给定的 IP 网络（CIDR 掩码）
	  --nodekey value                     P2P node key file //P2P 节点密钥文件
	  --nodekeyhex value                  P2P node key as hex (for testing) // P2P 节点密钥为十六进制（用于测试）
	
	MINER OPTIONS: //矿工选项
	  --mine                              Enable mining //开启挖矿
	  --miner.threads value               Number of CPU threads to use for mining (default: 0) // 用于挖掘的 CPU 线程数（默认值：0）
	  --miner.notify value                Comma separated HTTP URL list to notify of new work packages // 逗号分隔的 HTTP URL 列表，用于通知新的工作包
	  --miner.notify.full                 Notify with pending block headers instead of work packages // 使用待处理的区块头而不是工作包进行通知
	  --miner.gasprice value              Minimum gas price for mining a transaction (default: 1000000000) // 挖掘交易的最低 gas 价格（默认值：1000000000）
	  --miner.gaslimit value              Target gas ceiling for mined blocks (default: 8000000) // 开采区块的目标 gas 上限（默认值：8000000）
	  --miner.etherbase value             Public address for block mining rewards (default = first account) (default: "0") // 区块挖矿奖励的公共地址（默认=第一个帐户）（默认：“0”）
	  --miner.extradata value             Block extra data set by the miner (default = client version) // 阻止矿工设置的额外数据（默认 = 客户端版本）
	  --miner.recommit value              Time interval to recreate the block being mined (default: 3s) // 重新创建正在挖掘的块的时间间隔（默认值：3s）
	  --miner.noverify                    Disable remote sealing verification // 禁用远程密封验证
	
	GAS PRICE ORACLE OPTIONS: //天然气价格选项
	  --gpo.blocks value                  Number of recent blocks to check for gas prices (default: 20) // 最近检查gas价格的块数（默认值：20）
	  --gpo.percentile value              Suggested gas price is the given percentile of a set of recent transaction gas prices (default: 60) // 建议的 gas 价格是一组最近交易 gas 价格的给定百分比（默认值：60）
	  --gpo.maxprice value                Maximum gas price will be recommended by gpo (default: 500000000000) // gpo 将推荐的最大 gas 价格（默认值：500000000000）
	  --gpo.ignoreprice value             Gas price below which gpo will ignore transactions (default: 2) // Gas 价格低于该价格 gpo 将忽略交易（默认值：2）
	
	VIRTUAL MACHINE OPTIONS: // 虚拟机选项
	  --vmdebug                           Record information useful for VM and contract debugging // 记录对 VM 和合约调试有用的信息
	
	LOGGING AND DEBUGGING OPTIONS: // 记录和调试选项
	  --fakepow                           Disables proof-of-work verification // 禁用工作量证明验证
	  --nocompaction                      Disables db compaction after import // 导入后禁用数据库压缩
	  --verbosity value                   Logging verbosity: 0=silent, 1=error, 2=warn, 3=info, 4=debug, 5=detail (default: 3) // 日志详细程度：0=silent, 1=error, 2=warn, 3=info, 4=debug, 5=detail (default: 3)
	  --vmodule value                     Per-module verbosity: comma-separated list of <pattern>=<level> (e.g. eth/*=5,p2p=4) // 每个模块的详细程度：<pattern>=<level> 的逗号分隔列表（例如 eth/*=5,p2p=4）
	  --log.json                          Format logs with JSON // 用 JSON 格式化日志
	  --log.backtrace value               Request a stack trace at a specific logging statement (e.g. "block.go:271") // 在特定日志语句（例如“block.go:271”）处请求堆栈跟踪
	  --log.debug                         Prepends log messages with call-site location (file and line number) // 使用呼叫站点位置（文件和行号）预先准备日志消息
	  --pprof                             Enable the pprof HTTP server // 启用 pprof HTTP 服务器
	  --pprof.addr value                  pprof HTTP server listening interface (default: "127.0.0.1") // pprof HTTP 服务器监听接口（默认：“127.0.0.1”）
	  --pprof.port value                  pprof HTTP server listening port (default: 6060) // pprof HTTP 服务器监听端口（默认：6060）
	  --pprof.memprofilerate value        Turn on memory profiling with the given rate (default: 524288) // 以给定的速率打开内存分析（默认值：524288）
	  --pprof.blockprofilerate value      Turn on block profiling with the given rate (default: 0) // 以给定的速率打开块分析（默认值：0）
	  --pprof.cpuprofile value            Write CPU profile to the given file // 将 CPU 配置文件写入给定文件
	  --trace value                       Write execution trace to the given file // 将执行跟踪写入给定文件
	
	METRICS AND STATS OPTIONS: // 指标和统计选项
	  --metrics                              Enable metrics collection and reporting // 启用指标收集和报告
	  --metrics.expensive                    Enable expensive metrics collection and reporting // 启用昂贵的指标收集和报告
	  --metrics.addr value                   Enable stand-alone metrics HTTP server listening interface (default: "127.0.0.1") // 启用独立指标 HTTP 服务器侦听接口（默认值：“127.0.0.1”）
	  --metrics.port value                   Metrics HTTP server listening port (default: 6060) // Metrics HTTP 服务器监听端口（默认：6060）
	  --metrics.influxdb                     Enable metrics export/push to an external InfluxDB database // 启用指标导出/推送到外部 InfluxDB 数据库
	  --metrics.influxdb.endpoint value      InfluxDB API endpoint to report metrics to (default: "http://localhost:8086") // 值 InfluxDB API 端点将指标报告给（默认：“http://localhost:8086”）
	  --metrics.influxdb.database value      InfluxDB database name to push reported metrics to (default: "geth") // InfluxDB 数据库名称将报告的指标推送到（默认值：“geth”）
	  --metrics.influxdb.username value      Username to authorize access to the database (default: "test") // 授权访问数据库的用户名（默认：“test”）
	  --metrics.influxdb.password value      Password to authorize access to the database (default: "test") // 授权访问数据库的密码（默认：“test”）
	  --metrics.influxdb.tags value          Comma-separated InfluxDB tags (key/values) attached to all measurements (default: "host=localhost") // 逗号分隔的 InfluxDB 标签（键/值）附加到所有测量值（默认值：“host=localhost”）
	  --metrics.influxdbv2                   Enable metrics export/push to an external InfluxDB v2 database // 启用指标导出/推送到外部 InfluxDB v2 数据库
	  --metrics.influxdb.token value         Token to authorize access to the database (v2 only) (default: "test") // 授权访问数据库的令牌（仅限 v2）（默认值：“test”）
	  --metrics.influxdb.bucket value        InfluxDB bucket name to push reported metrics to (v2 only) (default: "geth") // InfluxDB 存储桶名称将报告的指标推送到（仅限 v2）（默认值：“geth”）
	  --metrics.influxdb.organization value  InfluxDB organization name (v2 only) (default: "geth") // InfluxDB 组织名称（仅限 v2）（默认值：“geth”）
	
	ALIASED (deprecated) OPTIONS: // 别名（已弃用）选项：
	  --nousb                             Disables monitoring for and managing USB hardware wallets (deprecated) //禁用监视和管理 USB 硬件钱包（已弃用）
	
	MISC OPTIONS: // 其他选项：
	  --snapshot                          Enables snapshot-database mode (default = enable) // 启用快照数据库模式（默认 = 启用）
	  --bloomfilter.size value            Megabytes of memory allocated to bloom-filter for pruning (default: 2048) // 分配给布隆过滤器用于修剪的内存兆字节（默认值：2048）
	  --help, -h                          show help
	  --catalyst                          Catalyst mode (eth2 integration testing) Catalyst 模式（eth2 集成测试）
	  --override.london value             Manually specify London fork-block, overriding the bundled setting (default: 0) //手动指定 London fork-block，覆盖捆绑设置（默认值：0）
	
### 链接到网络
如果你在没有任何标志的情况下启动 geth，它将连接到以太坊主网。除了主网之外，geth 还可以识别一些测试网，您可以通过相应的标志连接到它们：

- `--ropsten`

	 Ropsten 工作量证明测试网络
- `--rinkeby`

	 Rinkeby 权威证明测试网络
- `--goerli`

	Goerli 权威证明测试网络

注意：网络选择不会保留在配置文件中。要连接到预定义的网络，您必须始终明确启用它，即使在使用该 `--config` 标志加载其他配置值时也是如此。例如：

	# Generate desired config file. You must specify testnet here. // 生成所需的配置文件。 您必须在此处指定 testnet。
	geth --goerli --syncmode "full" ... dumpconfig > goerli.toml

	# Start geth with given config file. Here too the testnet must be specified. // 使用给定的配置文件启动 geth。 这里也必须指定测试网。
	geth --goerli --config goerli.toml
	
### 如何找到对等点
Geth 不断尝试连接到网络上的其他节点，直到它有对等点为止。如果您在路由器上启用了 UPnP 或在面向 Internet 的服务器上运行以太坊，它也将接受来自其他节点的连接。

Geth 通过称为发现协议的东西找到对等点。在发现协议中，节点彼此闲聊以了解网络上的其他节点。为了开始，geth 使用一组引导节点，其端点记录在源代码中。

要在启动时,更改引导 `--bootnodes` 节点，请使用该选项并用逗号分隔节点。例如：

	geth --bootnodes enode://pubkey1@ip1:port1,enode://pubkey2@ip2:port2,enode://pubkey3@ip3:port3
### 连接的常见问题
有时你只是无法连接。最常见的原因如下：

- 您的当地时间可能不正确。参与以太坊网络需要准确的时钟。检查您的操作系统以了解如何重新同步您的时钟（例如 `sudo ntpdate -s time.nist.gov`），因为即使 12 秒太快也会导致 0 个对等点。
- 某些防火墙配置可以阻止 UDP 流量流动。您可以使用静态节点功能或 `admin.addPeer()` 在控制台上手动配置连接。

要在没有发现协议的情况下启动 geth，您可以使用该 `--nodiscover` 参数。您只希望这是您正在运行测试节点或具有固定节点的实验测试网络。

### 检查连通性
要在交互式控制台中检查客户端连接到多少个对等点，该 `net` 模块有两个属性为您提供有关对等点数量以及您是否是侦听节点的信息。

	> net.listening
	true
	> net.peerCount
	4		
要获取有关连接的对等方的更多信息，例如 IP 地址和端口号、支持的协议，请使用对象的 `peers()` 功能 `admin`。`admin.peers()` 返回当前连接的对等点列表。

	> admin.peers
	[{
	  ID: 'a4de274d3a159e10c2c9a68c326511236381b84c9ec52e72ad732eb0b2b1a2277938f78593cdbe734e6002bf23114d434a085d260514ab336d4acdc312db671b',
	  Name: 'Geth/v0.9.14/linux/go1.4.2',
	  Caps: 'eth/60',
	  RemoteAddress: '5.9.150.40:30301',
	  LocalAddress: '192.168.0.28:39219'
	}, {
	  ID: 'a979fb575495b8d6db44f750317d0f4622bf4c2aa3365d6af7c284339968eef29b69ad0dce72a4d8db5ebb4968de0e3bec910127f134779fbcb0cb6d3331163c',
	  Name: 'Geth/v0.9.15/linux/go1.4.2',
	  Caps: 'eth/60',
	  RemoteAddress: '52.16.188.185:30303',
	  LocalAddress: '192.168.0.28:50995'
	}, {
	  ID: 'f6ba1f1d9241d48138136ccf5baa6c2c8b008435a1c2bd009ca52fb8edbbc991eba36376beaee9d45f16d5dcbf2ed0bc23006c505d57ffcf70921bd94aa7a172',
	  Name: 'pyethapp_dd52/v0.9.13/linux2/py2.7.9',
	  Caps: 'eth/60, p2p/3',
	  RemoteAddress: '144.76.62.101:30303',
	  LocalAddress: '192.168.0.28:40454'
	}, {
	  ID: 'f4642fa65af50cfdea8fa7414a5def7bb7991478b768e296f5e4a54e8b995de102e0ceae2e826f293c481b5325f89be6d207b003382e18a8ecba66fbaf6416c0',
	  Name: '++eth/Zeppelin/Rascal/v0.9.14/Release/Darwin/clang/int',
	  Caps: 'eth/60, shh/2',
	  RemoteAddress: '129.16.191.64:30303',
	  LocalAddress: '192.168.0.28:39705'
	} ]
要检查 geth 使用的端口并找到您的 enode URI，请运行：

	> admin.nodeInfo
	{
	  Name: 'Geth/v0.9.14/darwin/go1.4.2',
	  NodeUrl: 'enode://3414c01c19aa75a34f2dbd2f8d0898dc79d6b219ad77f8155abf1a287ce2ba60f14998a3a98c0cf14915eabfdacf914a92b27a01769de18fa2d049dbf4c17694@[::]:30303',
	  NodeID: '3414c01c19aa75a34f2dbd2f8d0898dc79d6b219ad77f8155abf1a287ce2ba60f14998a3a98c0cf14915eabfdacf914a92b27a01769de18fa2d049dbf4c17694',
	  IP: '::',
	  DiscPort: 30303,
	  TCPPort: 30303,
	  Td: '2044952618444',
	  ListenAddr: '[::]:30303'
	}
### 自定义网络
有时您可能不需要连接到实时公共网络，您可以选择创建自己的私有测试网。如果您不需要测试外部合约而只想测试技术，这将非常有用，因为您不必与其他矿工竞争，并且很容易生成大量测试以太币来玩（将 12345 替换为任何非负数）：
	
	geth -—networkid="12345" console
也可以通过提供 `--genesis` 标志来使用来自 JSON 文件的自定义创世块运行 geth 。Genesis JSON 文件应具有以下格式：

	{
	  "alloc": {
	    "dbdbdb2cbd23b783741e8d7fcf51e459b497e4a6": { 
	        "balance": "1606938044258990275541962092341162602522202993782792835301376"
	    },
	    "e6716f9544a56c530d868e4bfbacb172315bdead": {
	        "balance": "1606938044258990275541962092341162602522202993782792835301376"
	    },
	    ...
	  },
	  "nonce": "0x000000000000002a",
	  "difficulty": "0x020000",
	  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	  "coinbase": "0x0000000000000000000000000000000000000000",
	  "timestamp": "0x00",
	  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	  "extraData": "0x",
	  "gasLimit": "0x2fefd8"
	}
### 静态节点
Geth 还支持一种称为静态节点的功能，如果您有某些总是想连接的对等点。静态节点在断开连接时重新连接。您可以通过将以下内容放入以下内容来配置永久静态节点  `<datadir>/geth/static-nodes.json`：

	[
	  "enode://f4642fa65af50cfdea8fa7414a5def7bb7991478b768e296f5e4a54e8b995de102e0ceae2e826f293c481b5325f89be6d207b003382e18a8ecba66fbaf6416c0@33.4.2.1:30303",
	  "enode://pubkey@ip:port"
	]	
您还可以在运行时通过 js 控制台使用 `admin.addPeer()` 以下命令添加静态节点 

	admin.addPeer("enode://f4642fa65af50cfdea8fa7414a5def7bb7991478b768e296f5e4a54e8b995de102e0ceae2e826f293c481b5325f89be6d207b003382e18a8ecba66fbaf6416c0@33.4.2.1:30303")
### 可信节点
Geth 支持始终允许重新连接的受信任节点，即使达到对等限制也是如此。它们可以通过配置文件永久添加 `<datadir>/geth/trusted-nodes.json` 或通过 RPC 调用临时添加。配置文件的格式与用于静态节点的格式相同。可以通过 `admin.addTrustedPeer()` js 控制台使用 RPC 调用添加节点，并使用 `admin.removeTrustedPeer()` 调用删除节点。

	admin.addTrustedPeer("enode://f4642fa65af50cfdea8fa7414a5def7bb7991478b768e296f5e4a54e8b995de102e0ceae2e826f293c481b5325f89be6d207b003382e18a8ecba66fbaf6416c0@33.4.2.1:30303")
### JavaScript 控制台
Geth JavaScript 控制台公开了完整的 [web3 JavaScript Dapp API](https://github.com/ethereum/wiki/wiki/JavaScript-API) 和进一步的管理 API。

	（注意：geth 中捆绑的 web3 版本很旧，并且不是最新的官方文档）
#### 交互式使用：控制台
geth JavaScript 控制台以 `console` 或 `attachgeth` 子命令启动。该 `console` 子命令启动 GETH 节点，然后打开控制台。该 `attach` 子命令将控制台附加到已经运行的 geth 实例。

	geth console
	geth attach
如果 geth 节点使用非默认 ipc 端点运行或者您想通过 rpc 接口连接，则附加模式接受端点。

	geth attach /some/custom/path.ipc
	geth attach http://191.168.1.1:8545
	geth attach ws://191.168.1.1:8546
请注意，默认情况下，geth 节点不会启动 HTTP 和 WebSocket 服务器，并且出于安全原因，并非通过这些接口提供所有功能。这些默认值可以在 geth 节点启动时用 `--http.api `和 `--ws.api` 参数覆盖或者用 `admin.startRPC` 和 `admin.startWS` 覆盖。

如果您需要日志信息，请从以下内容开始：

	geth console --verbosity 5 2>> /tmp/eth.log
否则将您的日志静音，以免污染您的控制台：

	geth console 2> /dev/null
Geth 支持通过 `--preload` 选项将自定义 JavaScript 文件加载到控制台。这可用于加载常用函数，或设置 web3 合约对象。

	geth console --preload "/my/scripts/folder/utils.js,/my/scripts/folder/contracts.js"
#### 非交互式使用：脚本模式
也可以将文件执行到 JavaScript 解释器。在 `console` 与 `attach` 子接受 `--exec` 这是一个 JavaScript 语句的参数。

	geth attach --exec "eth.blockNumber"
这将打印正在运行的 geth 实例的当前块号，或者通过 http 在远程节点上执行带有更复杂语句的本地脚本：

	geth attach http://geth.example.org:8545 --exec 'loadScript("/tmp/checkbalances.js")'
	geth attach http://geth.example.org:8545 --jspath "/tmp" --exec 'loadScript("checkbalances.js")'

使用 `--jspath <path/to/my/js/root>` 为您的 js 脚本设置库目录。`loadScript()` 没有绝对路径的参数将被理解为相对于该目录。

您可以通过键入 `exit` 或简单地使用退出控制台 `CTRL-C`。

#### 注意事项
go-ethereum 现在使用与 ECMAScript 5.1 兼容的 [GoJa JS VM](https://github.com/dop251/goja)。但是有一些限制：

- 承诺，`async` 不会工作。

`web3.js` 使用 [bignumber.js](https://github.com/MikeMcl/bignumber.js) 图书馆。这个库会自动加载到控制台中。

#### 计时器
除了 JS 的全部功能（根据 ECMA5）之外，以太坊 JSRE 还增加了各种计时器。它实现 `setInterval`, `clearInterval`, `setTimeout`， `clearTimeout` 您可能习惯于在浏览器窗口中使用。它还提供了 `admin.sleep(seconds)` 一个基于块的计时器的实现，`admin.sleepBlocks(n)` 它会休眠直到添加的新块的数量等于或大于 `n`，想想“等待 n 次确认”。
### 管理您的帐户
	警告 请记住您的密码。
如果您丢失了用于加密帐户的密码，您将无法访问该帐户。

	重复：没有密码就无法访问您的帐户，这里没有忘记密码选项。请不要忘了。
以太坊 CLI `geth` 通过以下 `account` 命令提供帐户管理：

	$ geth account <command> [options...] [arguments...]
管理帐户可让您

- 创建新帐户
- 列出所有现有帐户
- 将私钥导入新帐户
- 迁移到最新的密钥格式
- 以及更改密码。

它支持交互模式，当提示您输入密码以及通过给定密码文件提供密码的非交互模式。非交互模式仅适用于在测试网络或已知安全环境中的脚本使用。

确保记住您在创建新帐户（使用新帐户、更新帐户或导入帐户）时提供的密码。没有它，您将无法解锁您的帐户。

	请注意，不支持以未加密格式导出您的密钥。
密钥存储在 `<DATADIR>/keystore`. 确保定期备份您的密钥！有关 详细信息，请参阅 DATADIR 备份和恢复章节。如果给出了自定义 datadir 和 keystore 选项，则 keystore 选项优先于 datadir 选项。

密钥文件的最新格式是：`UTC--<created_at UTC ISO8601>--<address hex>.` 列出时的帐户顺序是字典顺序，但由于时间戳格式，它实际上是创建顺序

在以太坊节点之间传输整个目录或其中的单个密钥是安全的。

	请注意，如果您从不同的节点向您的节点添加密钥，帐户的顺序可能会发生变化。因此，请确保您不依赖或更改脚本或代码片段中的索引。
然后再次。 `不要忘记您的密码`

	COMMANDS:
	     list    Print summary of existing accounts // 打印现有帐户的摘要
	     new     Create a new account // 创建一个新帐户
	     update  Update an existing account // 更新现有帐户
	     import  Import a private key into a new account // 将私钥导入新账户
您可以通过获取有关子命令的信息 `geth account <command> --help`。

	$ geth account list --help
	list [command options] [arguments...]
	
	Print a short summary of all accounts // 打印所有帐户的简短摘要
	
	OPTIONS:
	  --datadir "/home/bas/.ethereum"  Data directory for the databases and keystore // 数据库和密钥库的数据目录
	  --keystore                       Directory for the keystore (default = inside the datadir) // 密钥库的目录（默认 = 在 datadir 内）
帐户也可以通过 [Javascript 控制台](https://geth.ethereum.org/docs/interface/javascript-console) 进行管理

### 例子
#### 互动使用
- 创建帐户

		$ geth account new
		Your new account is locked with a password. Please give a password. Do not forget this password.
		Passphrase:
		Repeat Passphrase:
		Address: {168bc315a2ee09042d83d7c5811b533620531f67}
- 列出自定义密钥库目录中的帐户

		$ geth account list --keystore /tmp/mykeystore/
		Account #0: {5afdd78bdacb56ab1dad28741ea2a0e47fe41331} keystore:///tmp/mykeystore/UTC--2017-04-28T08-46-27.437847599Z--5afdd78bdacb56ab1dad28741ea2a0e47fe41331
		Account #1: {9acb9ff906641a434803efb474c96a837756287f} keystore:///tmp/mykeystore/UTC--2017-04-28T08-46-52.180688336Z--9acb9ff906641a434803efb474c96a837756287f
- 使用自定义数据目录将私钥导入节点

		$ geth account import --datadir /someOtherEthDataDir ./key.prv
		The new account will be encrypted with a passphrase.
		Please enter a passphrase now.
		Passphrase:
		Repeat Passphrase:
		Address: {7f444580bfef4b9bc7e14eb7fb2a029336b07c9d}
- 账户更新密码

		$ geth account update a94f5374fce5edbc8e2a8697c15331677e6ebf0b
		Unlocking account a94f5374fce5edbc8e2a8697c15331677e6ebf0b | Attempt 1/3
		Passphrase:
		0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b
		Account 'a94f5374fce5edbc8e2a8697c15331677e6ebf0b' unlocked.
		Please give a new password. Do not forget this password.
		Passphrase:
		Repeat Passphrase:
		0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b
- 非交互式使用

	您提供一个纯文本密码文件作为 `--password` 标志的参数。文件中的数据由密码的原始字符和单个换行符组成。

	注意：不建议在命令行中直接提供密码，但您始终可以使用 shell 技巧来绕过此限制。

		$ geth account new --password /path/to/password 
		
		$ geth account import  --datadir /someOtherEthDataDir --password /path/to/anotherpassword ./key.prv

#### 创建帐户
- 创建一个新帐户

		$ geth account new
		$ geth account new --password /path/to/passwdfile
		$ geth account new --password <(echo $mypassword)
- 创建一个新帐户并打印地址。在控制台上，使用：

		> personal.newAccount()
		... you will be prompted for a password ...
		
		or
		
		> personal.newAccount("passphrase")

该帐户以加密格式保存。您必须记住此密码以在将来解锁您的帐户。

对于非交互式使用，可以使用以下 `--password` 标志指定密码：

	geth account new --password <passwordfile> 
请注意，这仅用于测试，将密码保存到文件或以任何其他方式公开是个坏主意。

#### 通过导入私钥创建帐户
- 从中导入未加密的私钥 `<keyfile>` 并创建新帐户并打印地址。  

		geth account import <keyfile>
	- 假设密钥文件包含一个未加密的私钥，作为编码为十六进制的规范 EC 原始字节。
	- 该帐户以加密格式保存，系统会提示您输入密码。
	- 您必须记住此密码以在将来解锁您的帐户。
- 对于非交互式使用，可以使用以下 `--password` 标志指定密码：

		geth account import --password <passwordfile> <keyfile>

- 注意：由于您可以直接将您的加密帐户复制到另一个以太坊实例，因此在节点之间转移帐户时不需要这种导入/导出机制。
- 警告：当您将密钥复制到现有节点的密钥库中时，您习惯的帐户顺序可能会发生变化。因此，您要确保不依赖帐户顺序或仔细检查并更新脚本中使用的索引。
- 警告：如果您将密码标志与密码文件一起使用，最好确保该文件对除您之外的任何人都不可读取或什至不可列出。您可以通过以下方式实现：

		touch /path/to/password 
		chmod 700 /path/to/password
		cat > /path/to/password
		>I type my pass here^D

#### 更新现有帐户
您可以使用 `update` 带有帐户地址或索引作为参数的子命令在命令行上更新现有帐户。您可以一次指定多个帐户。

	geth account update 5afdd78bdacb56ab1dad28741ea2a0e47fe41331 9acb9ff906641a434803efb474c96a837756287f
	geth account update 0 1 2
该帐户以加密格式保存在最新版本中，系统会提示您输入密码以解锁帐户，并提示您输入另一个密码以保存更新的文件。因此，可以使用相同的命令将已弃用格式的帐户迁移到最新格式或更改帐户的密码。成功更新后，同一密钥的所有先前格式/版本都将被删除！

#### 导入您的预售钱包
导入您的预售钱包非常容易。如果您记得您的密码是：

	geth wallet import /path/to/my/presale.wallet
将提示您输入密码并导入您的以太预售帐户。它可以与 `–password` 选项以非交互方式使用，将密码文件作为包含明文钱包密码的参数。
#### 列出帐户和检查余额
- 列出您当前的帐户

	从命令行，使用以下命令调用 CLI：

		$ geth account list
		Account #0: {5afdd78bdacb56ab1dad28741ea2a0e47fe41331} keystore:///tmp/mykeystore/UTC--2017-04-28T08-46-27.437847599Z--5afdd78bdacb56ab1dad28741ea2a0e47fe41331
		Account #1: {9acb9ff906641a434803efb474c96a837756287f} keystore:///tmp/mykeystore/UTC--2017-04-28T08-46-52.180688336Z--9acb9ff906641a434803efb474c96a837756287f
- 按创建顺序列出您的帐户。

	注意：如果您从其他节点复制密钥文件，则此顺序可能会发生变化，因此请确保您要么不依赖索引，要么确保您在复制密钥时检查并更新脚本中的帐户索引。

	- 使用控制台时：

			> eth.accounts
			["0x5afdd78bdacb56ab1dad28741ea2a0e47fe41331", "0x9acb9ff906641a434803efb474c96a837756287f"]
	- 或通过 RPC：

			# Request
			$ curl -X POST --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}' -H 'Content-type: application/json' http://127.0.0.1:8545
			
			# Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": ["0x5afdd78bdacb56ab1dad28741ea2a0e47fe41331", "0x9acb9ff906641a434803efb474c96a837756287f"]
			}
- 如果您想以非交互方式使用帐户，则需要将其解锁。您可以在命令行上使用 `--unlock` 以逗号分隔的帐户列表（十六进制或索引）作为参数的选项来执行此操作，以便您可以以编程方式为一个会话解锁帐户。
	- 如果您想通过 RPC 使用来自 Dapps 的帐户，这将非常有用。`--unlock` 将解锁第一个帐户。这在您以编程方式创建帐户时很有用，您无需知道实际帐户即可解锁。
	- 创建帐户并在帐户解锁的情况下启动节点：
	
			geth account new --password <(echo this is not secret!) 
			geth --password <(echo this is not secret!) --unlock primary --rpccorsdomain localhost --verbosity 6 2>> geth.log 
	- 您可以使用整数索引代替帐户地址，它指的是帐户列表中的地址位置（并对应于创建顺序）
		- 命令行允许您解锁多个帐户。在这种情况下，unlock 的参数是一个逗号分隔的帐户地址或索引列表。
	
				geth --unlock "0x407d73d8a49eeb85d32cf465507dd71d507100c1,0,5,e470b1a7d2c9c5c6f03bbaa8fa20db6d404a0c32"
	- 如果以非交互方式使用此结构，您的密码文件将需要包含相关帐户的相应密码，每行一个。
	- 在控制台上，您还可以在一段时间内（以秒为单位）解锁帐户（一次一个）。
	
			personal.unlockAccount(address, "password", 300)
		请注意，我们不建议在此处使用密码参数，因为控制台历史记录已被记录，因此您可能会危及您的帐户。你被警告了。
- 检查帐户余额

	要检查您的 etherbase 帐户余额

		> web3.fromWei(eth.getBalance(eth.coinbase), "ether")
		6.5
	使用 JavaScript 函数打印所有余额

		function checkAllBalances() {
		    var totalBal = 0;
		    for (var acctNum in eth.accounts) {
		        var acct = eth.accounts[acctNum];
		        var acctBal = web3.fromWei(eth.getBalance(acct), "ether");
		        totalBal += parseFloat(acctBal);
		        console.log("  eth.accounts[" + acctNum + "]: \t" + acct + " \tbalance: " + acctBal + " ether");
		    }
		    console.log("  Total balance: " + totalBal + " ether");
		};
	然后可以执行以下操作
	
		> checkAllBalances();
		  eth.accounts[0]: 0xd1ade25ccd3d550a7eb532ac759cac7be09c2719 	balance: 63.11848 ether
		  eth.accounts[1]: 0xda65665fc30803cb1fb7e6d86691e20b1826dee0 	balance: 0 ether
		  eth.accounts[2]: 0xe470b1a7d2c9c5c6f03bbaa8fa20db6d404a0c32 	balance: 1 ether
		  eth.accounts[3]: 0xf4dd5c3794f1fd0cdc0327a83aa472609c806e99 	balance: 6 ether
	由于这个函数在重启 geth 后就会消失，所以可以将常用的函数存储起来以便以后调用。该 loadScript 功能使得这个非常容易。

	首先，将 `checkAllBalances()` 函数定义保存到计算机上的文件中。例如，`/Users/username/gethload.js`。然后从交互式控制台加载文件：

		> loadScript("/Users/username/gethload.js")
		true
	该文件将修改您的 JavaScript 环境，就像您手动输入命令一样。随意尝试！

### 矿业
本文档解释了如何设置 geth 进行挖矿。以太坊维基也有一个关于挖矿的[页面](https://eth.wiki/en/fundamentals/mining)，一定要检查一下。

挖矿是创建新区块的过程。Geth 实际上一直在创建新块，但是这些块需要通过工作量证明来保护，以便其他节点接受它们。挖矿就是创造这些工作量证明的价值。

工作量证明计算可以通过多种方式执行。Geth 包括一个 CPU 矿工，它在 geth 进程中进行挖掘。我们不鼓励在以太坊主网中使用 CPU 矿工。如果您想挖掘真正的以太币，请使用 GPU 挖掘。这样做的最佳选择是 [ethminer](https://github.com/ethereum-mining/ethminer) 软件。

在开始挖矿之前，始终确保您的区块链与链完全同步，否则您将无法在正确的链上挖矿，并且您的区块奖励将没有价值。
#### GPU挖矿
ethash 算法是内存硬的，为了将 DAG 放入内存中，每个 GPU 上需要 1-2GB 的 RAM。如果你得到 `Error GPU mining. GPU memory fragmentation?` 你没有足够的内存。

- 安装 ethminer

	要获得 ethminer，您需要安装 ethminer 二进制包或从源代码构建它。有关官方 ethminer 构建/安装说明，请参阅 [https://github.com/ethereum-mining/ethminer/#build](https://github.com/ethereum-mining/ethminer/#build)。在撰写本文时，ethminer 仅提供适用于 Microsoft Windows 的二进制文件。
- 使用 ethminer 和 geth
	- 首先创建一个帐户来保存您的块奖励。

			geth account new
		- 按照提示输入一个好的密码。不要忘记您的密码。还要注意在帐户创建过程结束时打印的公共以太坊地址。在以下示例中，我们将使用  `0xC95767AC46EA2A9162F0734651d6cF17e5BfcF10` 作为示例地址。
	- 现在启动 geth 并等待它同步区块链。这将需要相当长的时间。

			geth --http --miner.etherbase 0xC95767AC46EA2A9162F0734651d6cF17e5BfcF10
	- 要监视同步，在另一个终端中，您可以 `attach` 将 geth JavaScript 控制台连接到正在运行的节点，如下所示：

			geth attach https://127.0.0.1:8545
	- 然后在 > 提示符下键入

			eth.syncing
	
	您将看到类似于下面的示例输出的内容 ,它是一个两阶段过程，如我们的常见问题解答中详细描述的那样。
	
	- 在第一阶段，“currentBlock”和“highestBlock”之间的差异将减小，直到它们几乎相等。块下载第一阶段的示例输出：

			{
			  currentBlock: 10707814,
			  highestBlock: 13252182,
			  knownStates: 0,
			  pulledStates: 0,
			  startingBlock: 3809258
			   }
	- 然后它看起来会卡住并看起来永远不会变得平等。但是你应该看到“pulledStates”上升到等于“knownStates”。当两者相等时，您就同步了。

		来自 `eth.syncing` 的响应 `false` 意味着您已同步。

	现在我们准备开始挖矿了。在新的终端会话中，运行 ethminer 并将其连接到 geth：

	- 开放式语言

			ethminer -G -P http://127.0.0.1:8545
	- CUDA

			ethminer -U -P http://127.0.0.1:8545

	`ethminer` 在端口 8545（geth 中的默认 RPC 端口）上与 geth 通信。您可以通过提供 `--http.port` [选项](https://geth.ethereum.org/docs/rpc/server)来更改此设置 geth。Ethminer 会在任何端口上找到 geth。您还需要 ethminer 使用 `-P http://127.0.0.1:3301`. 如果您想在同一台计算机上挖掘多个实例，则需要设置自定义端口。如果您在私有集群上进行测试，我们建议您改用 CPU 挖掘。

如果默认值 `ethminer` 不起作用，请尝试使用以下命令指定 OpenCL 设备： `--opencl-device X` 其中 X 为 0、1、2 等。ethminer 使用 `-M `（基准测试）运行时，您应该看到如下内容：

	Benchmarking on platform: { "platform": "NVIDIA CUDA", "device": "GeForce GTX 750 Ti", "version": "OpenCL 1.1 CUDA" }
	Benchmarking on platform: { "platform": "Apple", "device": "Intel(R) Xeon(R) CPU E5-1620 v2 @ 3.70GHz", "version": "OpenCL 1.2 " }
注意在 geth GPU 挖掘时，哈希率信息不可用。使用来检查您的算力 ethminer，`miner.hashrate` 将始终报告 0。

#### 使用 Geth 挖掘 CPU
当您启动以太坊节点时 geth，默认情况下它不是挖矿。要以挖掘模式启动它，请使用 `--mine` 命令行标志。

- 该 `--miner.threads` 参数可用于设置并行挖矿线程数（默认为处理器内核总数）。

		geth --mine --miner.threads=4
- 您还可以使用控制台在运行时启动和停止 CPU 挖掘 。`miner.start` 为矿工线程的数量采用可选参数。

		> miner.start(8)
		true
		> miner.stop()
		true

请注意，只有当您与网络同步时（因为您在共识块之上进行挖掘），挖掘真正的以太币才有意义。因此 eth 区块链下载器/同步器将延迟挖掘直到同步完成，然后挖掘将自动开始，除非你取消你的意图 `miner.stop()`。

为了赚取以太币，您必须设置您的 etherbase（或coinbase）地址。此 etherbase 默认为您的主帐户。如果你没有 etherbase 地址，则 `geth --mine` 不会启动。

- 可以在命令行上设置您的 etherbase：

		geth --miner.etherbase '0xC95767AC46EA2A9162F0734651d6cF17e5BfcF10' --mine 2>> geth.log
- 也可以在控制台上重置您的 etherbase：

		> miner.setEtherbase(eth.accounts[2])

请注意，您的 etherbase 不需要是本地帐户的地址，只需一个现有帐户即可。

- 有一个选项可以将额外数据（仅限 32 字节）添加到您的开采块中。按照惯例，这被解释为 unicode 字符串，因此您可以设置简短的虚荣标签。

		> miner.setExtra("ΞTHΞЯSPHΞЯΞ")
- 可以使用 `miner.hashrate` 检查您的哈希率，结果以 H/s（每秒哈希操作）为单位。

		> eth.hashrate
		712000
- 在您成功挖出一些区块后，您可以查看您的 etherbase 帐户的以太币余额。现在假设您的 etherbase 是本地帐户：

		> eth.getBalance(eth.coinbase).toNumber();
		'34698870000000'
- 可以使用控制台上的以下代码片段检查特定矿工（地址）开采了哪些区块：

		> function minedBlocks(lastn, addr) {
		    addrs = [];
		    if (!addr) {
		        addr = eth.coinbase
		    }
		    limit = eth.blockNumber - lastn
		    for (i = eth.blockNumber; i >= limit; i--) {
		        if (eth.getBlock(i).miner == addr) {
		            addrs.push(i)
		        }
		    }
		    return addrs
		}
		// scans the last 1000 blocks and returns the blocknumbers of blocks mined by your coinbase // 扫描最后 1000 个区块并返回您的 coinbase 挖出的区块的区块数量
		// (more precisely blocks the mining reward for which is sent to your coinbase). //（更准确地阻止发送到您的币库的挖矿奖励）。
		> minedBlocks(1000, eth.coinbase)
		[352708, 352655, 352559]

请注意，您经常会发现一个块，但它从未进入规范链。这意味着当您在本地包含您开采的区块时，当前状态将显示记入您帐户的采矿奖励，但是，一段时间后，更好的链被发现，我们切换到不包含您的区块的链，因此没有挖矿奖励记入。因此，作为监控 coinbase 余额的矿工很可能会发现它可能会波动很大。

日志显示在 5 个区块后确认本地开采的区块。目前，您可能会发现从这些日志中生成您开采的区块列表更容易、更快捷。

### 私有网络
本指南解释了如何设置多个 Geth 节点的专用网络。如果节点未连接到主网络，则以太坊网络是专用网络。在这种情况下，私有仅意味着保留或隔离，而不是受保护或安全。
#### 选择网络 ID
网络 ID 是一个整数，用于隔离以太坊对等网络。只有当两个对等方使用相同的创世块和网络 ID 时，区块链节点之间的连接才会发生。使用 `--networkid` 命令行选项设置 geth 使用的网络 ID。

主网ID为1。如果您提供与主网不同的自定义网络ID，您的节点将不会连接到其他节点并形成私有网络。如果您计划连接到您的互联网上的私有链，最好选择一个尚未使用的网络 ID。您可以在 [https://chainid.network](https://chainid.network/) 找到社区运行的以太坊网络注册表。
#### 选择共识算法
虽然主网络使用工作量证明来保护区块链，但 Geth 还支持“集团”权威证明共识算法作为私有网络的替代方案。我们强烈建议为新的私有网络部署使用“clique”，因为它比工作量证明占用的资源少得多。clique 系统还用于多个公共以太坊测试网，例如 [Rinkeby](https://www.rinkeby.io/) 和 [Görli](https://goerli.net/)。

以下是 Geth 中可用的两种共识算法之间的主要区别：

- Ethash 共识是一种工作量证明算法

	是一个允许任何愿意将资源用于挖矿的人公开参与的系统。虽然这对于公共网络来说是一个很好的属性，但区块链的整体安全性严格取决于用于保护它的资源总量。因此，对于矿工很少的私有网络来说，工作量证明是一个糟糕的选择。Ethash 挖矿“难度”会自动调整，因此新区块的创建间隔大约为 12 秒。随着网络上部署的挖矿资源越来越多，创建新区块变得更加困难，因此平均出块时间与目标出块时间相匹配。
- Clique 共识是一种权威证明系统

	只有经过授权的“签名者”才能创建新块。集团共识协议在 [EIP-225](https://eips.ethereum.org/EIPS/eip-225) 中指定 。初始授权签名者集在创世区块中配置。可以使用投票机制对签名者进行授权和取消授权，从而允许在区块链运行时更改签名者集。Clique 可以配置为针对任何阻塞时间（在合理范围内），因为它与难度调整无关。

#### 创建创世区块
每个区块链都从创世块开始。当您第一次使用默认设置运行 Geth 时，它会将主网络 genesis 提交到数据库。对于私有网络，您通常需要不同的创世块。

创世区块是使用 `genesis.json` 文件配置的。创建创世区块时，您需要为区块链确定一些初始参数：

- 启动时启用的以太坊平台功能 ( `config` )。在区块链运行时启用协议功能需要安排硬分叉。
- 初始区块气体限制 (gasLimit)。您在此处的选择会影响单个块内可以发生的 EVM 计算量。我们建议使用主以太坊网络作为指南来找到合适的数量。可以在启动后使用 `--miner.gastarget` 命令行标志调整块气体限制。
- 初始分配 ether ( alloc)。这决定了您在创世区块中列出的地址有多少以太币可用。随着链的进展，可以通过挖矿创造额外的以太币（POA 不可以）。

####  Clique 示例
这是用于权威证明网络的 `genesis.json` 文件的示例。该 `config` 部分确保所有已知的协议更改都可用，并配置用于达成共识的“Clique”引擎。

	请注意，必须通过该 `extradata` 字段配置初始签名者集。该字段是 clique 工作所必需的。

- 首先使用 `geth account` 命令创建签名者帐户密钥（多次运行此命令以创建多个签名者密钥）。

		geth account new --datadir data
	记下此命令打印的以太坊地址。

	要为您的网络创建初始额外数据，请收集签名者地址并将其编码 `extradata` 为 32 个零字节、所有签名者地址和 65 个其他零字节的串联。在下面的示例中，`extradata` 包含单个初始签名者地址 `0x7df9a875a174b3bc565e6424a0050ebc1b2d1d82`.

- 可以使用 `period` 配置选项来设置链的目标出块时间。

		{
		  "config": {
		    "chainId": 15,
		    "homesteadBlock": 0,
		    "eip150Block": 0,
		    "eip155Block": 0,
		    "eip158Block": 0,
		    "byzantiumBlock": 0,
		    "constantinopleBlock": 0,
		    "petersburgBlock": 0,
		    "clique": {
		      "period": 5,
		      "epoch": 30000
		    }
		  },
		  "difficulty": "1",
		  "gasLimit": "8000000",
		  "extradata": "0x00000000000000000000000000000000000000000000000000000000000000007df9a875a174b3bc565e6424a0050ebc1b2d1d820000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
		  "alloc": {
		    "7df9a875a174b3bc565e6424a0050ebc1b2d1d82": { "balance": "300000" },
		    "f41c74c9ae680c1aa78f42e5647a62f353b7bdde": { "balance": "400000" }
		  }
		}

#### Ethash 示例
由于 ethash 是默认的共识算法，因此无需配置其他参数即可使用它。您可以使用该 difficulty 参数影响初始挖矿难度 ，但请注意，难度调整算法会快速适应您在链上部署的挖矿资源量。

		{
		  "config": {
		    "chainId": 15,
		    "homesteadBlock": 0,
		    "eip150Block": 0,
		    "eip155Block": 0,
		    "eip158Block": 0,
		    "byzantiumBlock": 0,
		    "constantinopleBlock": 0,
		    "petersburgBlock": 0,
		    "ethash": {}
		  },
		  "difficulty": "1",
		  "gasLimit": "8000000",
		  "alloc": {
		    "7df9a875a174b3bc565e6424a0050ebc1b2d1d82": { "balance": "300000" },
		    "f41c74c9ae680c1aa78f42e5647a62f353b7bdde": { "balance": "400000" }
		  }
		}
#### 初始化 Geth 数据库
要创建使用此创世块的区块链节点，请运行以下命令。这将为您的链导入并设置规范的创世块。

	geth init --datadir data genesis.json
未来使用此数据目录运行 geth 将使用您定义的创世块。

	geth --datadir data --networkid 15
#### 调度硬分叉
随着以太坊协议开发的进展，新的以太坊功能变得可用。要在您的专用网络上启用这些功能，您必须安排硬分叉。

- 首先，选择硬分叉将激活的任何未来区块号。继续上面的 `genesis.json` 示例，假设您的网络正在运行，其当前块号为 35421。为了安排“伊斯坦布尔”分叉，我们选择块 40000 作为激活块号并修改我们的 `genesis.json` 文件以设置它：

		{
		  "config": {
		    ...
		    "istanbulBlock": 40000,
		    ...
		  },
		  ...
		}
- 为了更新到新的分支，首先确保您的私有网络上的所有 Geth 实例实际上都支持伊斯坦布尔分支（即确保您安装了最新版本的 Geth）。
- 现在关闭所有节点并重新运行 init 命令以启用新的链配置：

		geth init --datadir data genesis.json

#### 设置网络
一旦您的节点初始化为所需的创世状态，就可以设置点对点网络了。任何节点都可以用作入口点。我们建议将单个节点用作所有其他节点用于加入的集合点。该节点称为“引导节点”。

- 首先，确定引导节点将在其上运行的机器的 IP 地址。如果您使用的是 Amazon EC2 等云服务，您将在管理控制台中找到虚拟机的 IP。还请确保您的防火墙配置允许端口 30303 上的 UDP 和 TCP 流量。
- 引导节点需要知道自己的 IP 地址，以便能够将其中继给其他人。IP 是使用 `--nat` 标志设置的（插入您自己的 IP 而不是下面的示例地址）。

		geth --datadir data --networkid 15 --nat extip:172.16.254.4
- 现在使用 JS 控制台提取 bootnode 的“节点记录”。

		geth attach data/geth.ipc --exec admin.nodeInfo.enr
- 此命令应打印一个 base64 字符串，例如以下示例。其他节点将使用引导节点记录中包含的信息连接到您的对等网络。

		"enr:-Je4QEiMeOxy_h0aweL2DtZmxnUMy-XPQcZllrMt_2V1lzynOwSx7GnjCf1k8BAsZD5dvHOBLuldzLYxpoD5UcqISiwDg2V0aMfGhGlQhqmAgmlkgnY0gmlwhKwQ_gSJc2VjcDI1NmsxoQKX_WLWgDKONsGvxtp9OeSIv2fRoGwu5vMtxfNGdut4cIN0Y3CCdl-DdWRwgnZf"

	设置对等网络取决于您的要求。如果您通过 Internet 连接节点，请确保您的 bootnode 和所有其他节点都分配了公共 IP 地址，并且 TCP 和 UDP 流量都可以通过防火墙。
- 如果不需要 Internet 连接或所有成员节点都使用众所周知的 IP 连接，我们强烈建议设置 Geth 以将点对点连接限制到 IP 子网。这样做将进一步隔离您的网络并防止与其他区块链网络交叉连接，以防您的节点可从 Internet 访问。使用该` --netrestrict` 标志配置 IP 网络白名单：

		geth <other-flags> --netrestrict 172.16.254.0/24

	通过上述设置，Geth 将只允许来自 172.16.254.0/24 子网的连接，并且不会尝试连接到设置 IP 范围之外的其他节点。

#### 运行成员节点
在运行成员节点之前，您必须使用与引导节点相同的创世文件对其进行初始化。

随着引导节点的运行和外部可访问（您可以尝试 `telnet <ip> <port> `确保它确实可访问），您可以启动更多的 Geth 节点并使用 `--bootnodes` 标志通过引导节点连接它们。

要创建与引导节点在同一台机器上运行的成员节点，请选择单独的数据目录（示例 `data-2`：）和侦听端口（示例：`30305`）：

	geth --datadir data-2 --networkid 15 --port 30305 --bootnodes <bootstrap-node-record>
在成员节点运行时，您可以通过附加控制台并运行 `admin.peers` . 节点可能需要长达几秒钟的时间才能连接。

	geth attach data-2/geth.ipc --exec admin.peers

#### Clique：运行签名者
要设置 Geth 以在授权证明模式下签署区块，必须有一个签名者帐户可用。必须解锁帐户才能挖掘区块。以下命令将提示输入帐户密码，然后开始签署区块：

	geth <other-flags> --unlock 0x7df9a875a174b3bc565e6424a0050ebc1b2d1d82 --mine
您可以通过更改默认 gas 限制块收敛到（with `--miner.gastarget`）并在（with `--miner.gasprice`）处接受价格交易来进一步配置挖掘。
#### Ethash：运行矿工
对于简单私有网络中的工作量证明，单个 CPU 矿工实例足以定期创建稳定的区块流。要启动一个用于挖掘的 Geth 实例，请使用所有常用标志运行它并添加以下内容以配置挖掘：

	geth <other-flags> --mine --miner.threads=1 --miner.etherbase=0x0000000000000000000000000000000000000000
这将在单个 CPU 线程上开始挖掘区块和交易，将所有区块奖励记入由 指定的帐户 `--miner.etherbase` 。

### 指标监控  metrics
#### 指标和计时器 Meters and Timers
请注意，默认情况下禁用指标收集，以免为普通用户带来报告开销。因此 `--metrics`  必须使用该标志来启用基本指标，并且该标志 `--metrics.expensive` 可用于启用从资源消耗角度来看被认为“昂贵”的某些指标。昂贵指标的示例是每个数据包的网络流量数据。

Geth 指标系统的目标是类似于日志,我们应该能够将任意指标集合添加到代码的任何部分，而无需花哨的构造来分析它们（计数器变量、公共接口、跨 API、控制台挂钩 等等）。相反，我们应该随时随地“更新”指标，并让它们自动收集、通过 API 呈现、可查询和可视化以进行分析。

就此而言，Geth 目前实施两种类型的指标：

- 指标 metrics 

	类似于物理仪表（电、水等），它们能够测量通过的“事物”的数量以及它们通过的速度。仪表没有特定的度量单位（字节、块、malloc 等），它只计算任意事件。它可以随时报告：
	
	- 通过仪表的事件(events)总数
	- 自启动以来仪表的平均吞吐率（事件(events)/秒）
	- 加权吞吐速率在过去1，5和15分钟（事件(events)/秒）
		- （“加权”意味着最近的秒数比旧的秒数更多）
- 计时器 timers 

	指标的扩展，不仅可以测量某些事件的发生，还可以收集其持续时间。与仪表类似，计时器也可以测量任意事件，但每个事件都需要单独分配持续时间。除了仪表可以生成的所有报告外，计时器还具有：

	- Percentiles (5, 20, 50, 80, 95)，报告某些百分比的事件花费的时间少于报告的执行时间（例如 Percentile 20 = 1.5s 意味着 20% 的测量事件花费的时间少于 1.5 秒执行；本质上 80%(=100%-20%) 花费的时间超过 1.5s )
		- 百分位数 5：最短持续时间（这是尽可能快的）
		- 百分位数 50：表现良好的样本（无聊，只是提供一个想法）
		- 百分位 80：一般性能（这些应该优化）
		- 百分位数 95：最坏情况的异常值（罕见，请妥善处理）
- 计数器 Counters

	计数器保存一个可以递增和递减的 int64 值。可以查询计数器的当前值。
- 仪表 Gauges

	仪表测量单个 int64 值。此外，为了增加和减少值，就像计数器一样，可以任意设置仪表。
	
#### 创建和更新指标
- 可以在代码中轻松添加指标

		meter := metrics.NewMeter("system/memory/allocs")
		timer := metrics.NewTimer("chain/inserts")
- 为了在不创建依赖循环的情况下使用来自两个不同包的同一个仪表，可以使用 `NewOrRegisteredX()` 函数创建度量。如果没有具有此名称的仪表可用或返回现有仪表，则这将创建一个新仪表。
	
		meter := metrics.NewOrRegisteredMeter("system/memory/allocs")
		timer := metrics.NewOrRegisteredTimer("chain/inserts")
- 名称可以是任意字符串，但是由于 Geth 假定它是某个有意义的子系统层次结构，因此请相应命名。然后可以同样简单地更新指标：

		meter.Mark(n) // Record the occurrence of `n` events // 记录`n` 个事件的发生
		
		timer.Update(duration)  // Record an event that took `duration` // 记录一个花费了 `duration` 的事件
		timer.UpdateSince(time) // Record an event that started at `time` // 记录在`time` 开始的事件
		timer.Time(function)    // Measure and record the execution of `function` // 测量并记录`function`的执行

#### 查询指标
Geth 处公开所有收集的指标 `127.0.0.1:6060/debug/metrics` 。要收集指标，您需要添加 `--metrics` 标志。为了启动度量服务器，您需要指定 `--metrics.addr` 标志。

Geth 还支持将指标直接转储到涌入数据库中。为了激活它，您需要指定 `--metrics.influxdb` 标志。您可以指定 API 端点以及密码和用户名以及其他 `influxdb` 标签。
#### 可用指标
指标是一种调试工具，每个开发人员都可以根据需要自由添加、删除或修改它们。由于它们可以在提交之间更改，因此可以通过 `127.0.0.1:6060/debug/metrics` 在浏览器中打开来查询完全可用的那些。然而，有一些可能会保证长寿，所以如果你觉得它值得更一般的观众，请随意添加到下面的列表中：

	 * system/
	    * memory/
	        * allocs: number of memory allocations made // 分配的内存数量
	        * used: amount of memory currently used // 当前使用的内存量
	        * held: memory allocated on the heap //堆上分配的内存
	        * pauses: garbage collector pauses // 垃圾收集器暂停
	        * frees: number of memory allocations freed // 释放的内存分配数
	    * cpu/
	        * sysload: time spent by CPU on all processes // CPU 在所有进程上花费的时间
	        * syswait: time spent waiting on disk i/o // 等待磁盘 i/o 所花费的时间
	        * procload: time spent by CPU on this process // CPU 在这个进程上花费的时间
	        * threads: number of threads // 线程数
	        * goroutines: number of goroutines // goroutines的数量
	    * disk/
	        * readcount: number of read operations // 读操作的次数
	        * readdata: total number of bytes read // 读取的总字节数
	        * readbytes: counter of bytes read // 读取字节的计数器
	        * writecount: number of write operations // 写操作的次数
	        * writedata: total number of bytes written // 写入的总字节数
	        * writebytes: counter of bytes written // 写入的字节数
	
	* rpc/
	    * requests: number of requests // 请求的数量
	    * success: number of successful requests // 成功请求的数量
	    * failure: number of failed requests // 失败的请求数
	
	* chain/
	    * reorg/ // 重组
	        * drop: blocks dropped by reorg // 重组丢弃的块
	        * add: blocks added by reorg // 由重组添加的块
	    * head/
	        * block: currently newest block // 当前最新的块
	        * header: currently newest header // 当前最新的标头
	        * receipt: currently newest receipt // 当前最新收据
	
	* p2p/
	    * peers: number of connected peers // 连接的对等点数
	    * ingress: inbound traffic in bytes // 以字节为单位的入站流量
	    * egress: outbound traffic in bytes // 以字节为单位的出站流量
	
	* txpool/
	    * pending: currently pending transactions // 当前待处理的交易
	    * local: number of transactions send from this node // 从该节点发送的交易数量	
## Geth 开发者指南
注意：这些说明适用于想要贡献 Go 源代码更改的人。如果您只想运行以太坊，请使用常规安装说明。

本文档是以太坊 Go 实现的开发人员的切入点。这里的开发者指的是实践者：对构建、开发、调试、提交错误报告或拉取请求或向 go-ethereum 贡献代码感兴趣的人。
### 贡献
感谢您考虑帮助提供源代码！我们欢迎互联网上任何人的贡献，并感谢即使是最小的修复！

GitHub 用于跟踪问题并提供代码、建议、功能请求或文档。

如果您想为 go-ethereum 做出贡献，请分叉、修复、提交并发送拉取请求 (PR) 供维护人员审查并合并到主代码库中。如果您希望提交更复杂的更改，请查看 go-ethereum [Discord Server](https://discord.gg/invite/nthXNEv) 中的核心开发人员。确保这些更改符合项目的总体理念和/或获得一些早期反馈。这可以减少您的工作量并加快我们的审查和合并程序。

PR 需要基于 master 分支并针对该分支打开（除非通过明确的协议，您为复杂的功能分支做出了贡献）。

您的 PR 将根据[代码审查指南](https://geth.ethereum.org/docs/developers/code-review-guidelines)进行审查。

我们鼓励 PR 早期方法，这意味着即使没有修复/功能，您也可以最早创建 PR。这会让核心开发人员和其他志愿者知道你发现了一个问题。这些早期 PR 应表明“进行中”状态。
### 构建和测试
我们假设您已经安装了 Go。请使用 Go 1.13 或更高版本。我们使用 gc 工具链进行开发，您可以从[Go 下载页面 获取](https://golang.org/doc/install)。

go-ethereum 是一个 Go 模块，它使用[Go 模块系统](https://github.com/golang/go/wiki/Modules)来管理依赖项。使用GOPATH不需要构建行进。
#### 构建可执行文件
- 切换到 go-ethereum 仓库根目录。
- 您可以使用 go 工具构建所有代码，将生成的二进制文件放在 `$GOPATH/bin`.

		go install -v ./...
- go-ethereum 可执行文件可以单独构建。要仅构建 geth，请使用：

		go install -v ./cmd/geth

如果您想为与您的主机不同的架构编译 geth，请参阅我们的交叉编译指南。

#### 测试
- 测试一个包：

		go test -v ./eth
- 运行单个测试：

		go test -v ./eth -run TestMethod

	注意：这里所有带有前缀 TestMethod 的测试都会运行，所以如果你有 TestMethod，TestMethod1，那么两个测试都会运行。
- 运行基准测试，例如：

		go test -v -bench . -run BenchmarkJoin
有关更多信息，请参阅 [go test 标志](https://golang.org/cmd/go/#hdr-Testing_flags)文档。

#### 获取堆栈跟踪
如果 geth 使用该 `--pprof` 选项启动，调试 HTTP 服务器在端口 6060 上可用。您可以打开 `http://localhost:6060/debug/pprof` 以查看堆、运行例程等。单击“full goroutine stack dump”您可以生成对调试有用的跟踪。

请注意，如果您运行的多个实例 geth，则此端口仅适用于启动的第一个实例。如果你想为这些其他实例生成堆栈跟踪，你需要选择一个替代的 pprof 端口来启动它们。确保将 stderr 重定向到日志文件。

	geth -port=30300 -verbosity 5 --pprof --pprof.port 6060 2>> /tmp/00.glog
	geth -port=30301 -verbosity 5 --pprof --pprof.port 6061 2>> /tmp/01.glog
	geth -port=30302 -verbosity 5 --pprof --pprof.port 6062 2>> /tmp/02.glog
或者，如果您想杀死客户端（以防它们挂起或停止同步等）并拥有堆栈跟踪，您可以使用以下 `-QUIT` 信号 `kill`：

	killall -QUIT geth
这会将每个实例的堆栈跟踪转储到它们各自的日志文件中。

### 代码审查
将代码放入 go-ethereum 的唯一方法是发送拉取请求。这些拉取请求需要有人审查。本文档是一份指南，解释了我们对作者和审稿人 PR 的期望。
#### 术语
- `拉取请求的作者`是编写差异并将其提交给 GitHub 的实体。
- 该`团队`由对 go-ethereum 存储库具有提交权限的人员组成。
- 该`评审`是分配给审查差异的人。审稿人必须是团队成员。
- 该`代码所有者`负责通过公关修改子系统之中的人。

#### 过程
为任何 PR 做出的第一个决定是它是否值得包括在内。这个决定主要由代码所有者决定，但可以与团队成员协商。

为了做出决定，我们必须了解 PR 是关于什么的。如果描述内容不足或差异太大，请要求解释。这部分任何人都可以做。

我们希望审阅者检查 PR 的风格和功能，使用 GitHub 审阅系统向作者提供评论。审稿人应该跟进 PR，直到它处于良好状态，然后批准PR。任何代码所有者都可以合并已批准的 PR。

与作者交流时，要礼貌和尊重。
#### 代码风格
我们期待 `gofmted` 代码。对于规模巨大的贡献，我们希望作者理解并使用 [Effective Go](https://golang.org/doc/effective_go.html) 中的指南。作者应该避免 [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)页面中解释的常见错误。
#### 功能检查
对于修复问题的 PR，审阅者应该尝试重现问题并验证拉取请求是否确实修复了它。作者可以通过包含一个在没有更改的情况下失败（并通过更改）的单元测试来帮助解决这个问题。

对于添加新功能的 PR，审阅者应该尝试使用该功能并评论使用它的感觉。示例：如果 PR 添加了新的命令行标志，请使用带有该标志的程序并评论该标志是否有用。

我们期望适当的单元测试覆盖率。审阅者应验证单元测试是否涵盖了新代码。
#### CI
提交的代码必须通过所有单元测试和静态分析（“lint”）检查。我们使用 Travis CI 在 Linux、macOS 、AppVeyor 、Microsoft Windows 上测试代码。

对于失败的 CI 构建，问题可能与 PR 本身无关。此类失败通常与 flakey 测试有关。这些失败可以忽略（作者不需要修复无关的问题），但请提交 GH 问题，以便最终修复测试。
#### 提交消息
在主分支上提交消息应遵循以下规则。PR 作者不需要使用任何特定的样式，因为可以在合并时修改消息。强制提交消息样式是合并 PR 的人的责任。

我们使用的提交消息样式类似于 Go 项目使用的样式：

更改描述的第一行通常是更改的一行摘要，以主要受影响的 Go 包为前缀。它应该完成句子“此更改将 go-ethereum 修改为 _____。” 描述的其余部分详细说明并应提供更改的上下文并解释其作用。

模板：

	package/path: change XYZ
	 
	Longer explanation of the change in the commit. You can use
	multiple sentences here. It's usually best to include content
	from the PR description in the final commit message.
 
	issue notices, e.g. "Fixes #42353".
#### 特殊情况以及如何处理
作为审阅者，您可能会发现自己处于以下情况之一。以下是处理这些的方法：

- 作者没有跟进

	过一段时间（即几天后）ping 他们。如果没有进一步的响应，请关闭 PR 或自行完成工作。
- 作者坚持将重构更改与错误修复一起包括在内

	我们可以在任何更改旁边容忍小的重构。如果您在差异中感到迷失，请要求作者将重构作为独立 PR 提交或者至少作为同一 PR 中的独立提交提交。
- 作者不断拒绝您的反馈

	审阅者有权因技术原因拒绝任何更改。如果您不确定，请向团队征求第二意见。如果无法达成共识，您可以关闭 PR。
	
### 问题处理工作流程
#### （提案草案）
- 保持未决问题数低于 820
- 保持所有问题的未决问题比例低于 13%
- 有 50 个标记为需要[帮助](https://github.com/ethereum/go-ethereum/labels/help%20wanted)的问题和 50 个好的[第一个问题](https://github.com/ethereum/go-ethereum/labels/good%20first%20issue)。
- 使用形式的结构化标签 `<category>:<label>` 或者如果需要的话 `<category>:<main>/<sub>` ，例如 `area: plugins/foobuzzer`。
- 使用以下标签。区域和状态取决于应用程序和工作流程。
	- 区域
		- area: android
		- area: clef
		- area: network
		- area: swarm
		- area: whisper
	- 类型
		- type: bug
		- type: feature
		- type: documentation
		- type: discussion
	- 地位
		- status: PR review
		- status: community working on it
	- 需要
		- need: more info
		- need: steps to reproduce
		- need: investigation
		- need: decision
- 使用这些里程碑
	- [未来](https://github.com/ethereum/go-ethereum/milestone/80)- 也许有一天会实施
	- [即将推出](https://github.com/ethereum/go-ethereum/milestone/81)- 未分配给特定版本，而是在即将发布的版本之一中交付
	- <next version> - 带有版本号的下一个版本
	- <next-next version> - 下一个版本之后的版本，带有版本号
	- <下一个主要版本> - 可选。
	
	可以不为里程碑设置截止日期，但是一旦发布，请关闭它。如果您有一些悬而未决的问题，请考虑将它们移至下一个里程碑，并关闭这个里程碑。

	或者使用项目板来收集具有最终状态并涵盖多个版本的更大工作的问题。

#### 工作流程
我们每周或每两周召开一次分类会议。问题通过[标记为“status:triage”](https://github.com/ethereum/go-ethereum/issues?q=is%3Aopen+is%3Aissue+label%3Astatus%3Atriage+sort%3Acreated-asc)来预先选择，[并首先对最旧的进行排序](https://github.com/ethereum/go-ethereum/issues?q=is%3Aopen+is%3Aissue+label%3Astatus%3Atriage+sort%3Acreated-asc)。这是我们解决新问题并执行以下操作之一的时间

- 关闭它。
- 将其分配给没有结束日期的“即将推出”里程碑。
- 将其移至“未来”里程碑。
- 将其状态更改为“需要：<需要什么>”。

可选的进一步活动：

- 用适当的区域/组件标记问题。
- 向常见问题解答添加一个部分或添加一个 wiki 页面。从问题链接到它。

#### DNS 发现设置指南
本文档介绍了如何使用 devp2p 开发人员工具设置 [EIP 1459](https://eips.ethereum.org/EIPS/eip-1459)节点列表。本指南的重点是为以太坊主网和公共测试网创建一个公共列表，但如果您想为专用网络设置基于 DNS 的发现，您也会发现这很有帮助。

当与发现 DHT 的连接不可用时，基于 DNS 的节点列表可以用作后备选项。在本指南中，我们将通过抓取发现 DHT 来创建节点列表，然后在选定的 DNS 名称下发布生成的节点集。

- 安装 devp2p 命令

	cmd/devp2p 是一个开发工具，不包含在 Geth 发行版中。您可以使用以下命令安装此命令 `go ge`：

		go get github.com/ethereum/go-ethereum/cmd/devp2p
	要创建签名密钥，您可能还需要该 `ethkey` 实用程序。

		go get github.com/ethereum/go-ethereum/cmd/ethkey
- 爬取 v4 DHT

	我们的第一步是编译所有可达节点的列表。cmd/devp2p 中的 DHT 爬虫是一个批处理过程，它运行一段时间。您应该安排此命令定期运行。要创建节点列表，请运行

devp2p discv4 crawl -timeout 30m all-nodes.json
这会遍历 DHT 并将所有找到的节点的集合存储在all-nodes.json文件中。同一命令的后续运行将重新验证以前发现的节点记录，将新发现的节点添加到集合中，并删除不再活动的节点。每次运行都会提高节点集的质量，因为重新验证的数量与集合中的每个节点一起跟踪。

通过过滤创建子列表
一旦all-nodes.json创建并且该集合包含大量节点，就可以使用该devp2p nodeset filter命令提取有用的节点子集。此命令将节点集文件作为参数并应用作为命令行标志给出的过滤器。

要创建过滤节点集，首先创建一个新目录来保存输出集。您可以使用任何目录名称，但最好使用 DNS 域名作为此目录的名称。

mkdir mainnet.nodes.example.org
然后，要创建仅包含以太坊主网节点的输出集，请运行

devp2p nodeset filter all-nodes.json -eth-network mainnet > mainnet.nodes.example.org/nodes.json
以下过滤器标志可用：

-eth-network ( mainnet | ropsten | rinkeby | goerli ) 选择一个以太坊网络。
-les-server 选择 LES 服务器节点。
-ip <mask> 将节点限制在给定的 IP 范围内。
-min-age <duration> 将结果限制为在给定持续时间内一直存在的节点。
创建 DNS 树
要将节点列表转换为 DNS 节点树，需要对该列表进行签名。为此，您需要一个密钥对。要以正确的格式创建密钥文件，您可以使用 cmd/ethkey 实用程序。请选择一个好的密码来加密磁盘上的密钥。

ethkey generate dnskey.json
现在用于devp2p dns sign更新节点列表的签名。如果您的列表的目录名称与您要发布的名称不同，请使用该-domain标志指定 DNS 名称。此命令将提示输入密钥文件密码并更新树签名。

devp2p dns sign mainnet.nodes.example.org dnskey.json
生成的 DNS 树元数据存储在 mainnet.nodes.example.org/enrtree-info.json文件中。

发布 DNS 树
既然树已经签名，就可以将其发布到 DNS 提供商。cmd/devp2p 目前支持发布到 CloudFlare DNS 和 Amazon Route53。您还可以将 TXT 记录导出为 JSON 文件并自行发布。

要发布到 CloudFlare，首先在管理控制台中创建一个 API 令牌。cmd/devp2p 需要CLOUDFLARE_API_TOKEN环境变量中的 API 令牌。现在使用以下命令通过 CloudFlare API 上传 DNS TXT 记录：

devp2p dns to-cloudflare mainnet.nodes.example.org
请注意，此命令使用签名时指定的域名。cmd/devp2p 将删除此名称下的任何现有记录。

在 Geth 中使用 DNS 树
一旦您的树通过 DNS 名称可用，您可以告诉 geth 将它与 --discovery.dns命令行标志一起使用。使用enrtree://URL 方案引用节点树。您可以在enrtree-info.json创建的文件中 找到树的 URL devp2p dns sign。只需将 URL 作为参数传递给标志即可使用已发布的树。

geth --discovery.dns "enrtree://AMBMWDM3J6UY3M32TMMROUNLX6Y3YTLVC3DC6HN2AVG5NHNSAXDW6@mainnet.nodes.example.org"	

## 参考
[https://geth.ethereum.org/docs/getting-started](https://geth.ethereum.org/docs/getting-started)