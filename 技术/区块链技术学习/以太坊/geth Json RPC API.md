# Geth Json RPC API
## JSON-RPC 服务器
Geth 支持所有标准的 web3 JSON-RPC API。您可以在 [Ethereum Wiki JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC) 页面上找到这些 API 的文档。

JSON-RPC 在多种传输上提供。Geth 支持基于 HTTP、WebSocket 和 Unix 域套接字的 JSON-RPC。必须通过命令行标志启用传输。

以太坊 JSON-RPC API 使用命名空间系统。RPC 方法根据其用途分为几类。所有方法名称都由名称空间、下划线和名称空间内的实际方法名称组成。例如，`eth_call` 驻留在 `eth` 命名空间中。

可以在每个命名空间的基础上启用对 RPC 方法的访问。在侧边栏中查找各个命名空间的文档。
### HTTP服务器
要启用 HTTP 服务器，请使用该 `--http` 标志。

	geth --http
默认情况下，geth 接受来自环回接口 (127.0.0.1) 的连接。默认侦听端口为 8545。您可以使用 `--http.port` 和 `--http.addr` 标志自定义地址和端口 。

	geth --http --http.port 3334
JSON-RPC 方法命名空间必须列入白名单才能通过 HTTP 服务器使用。`-32602` 如果调用未列入白名单的命名空间，则会生成带有错误代码的 RPC 错误。默认白名单允许访问 `eth` 和 `shh` 命名空间。要启用对帐户管理（“个人”）和调试（“调试”）等其他 API 的访问，它们必须通过 `--http.api` 标志进行配置。但是，我们不建议通过 HTTP 启用此类 API，因为访问这些方法会增加攻击面。

	geth --http --http.api personal,eth,net,web3
由于可以从任何本地应用程序访问 HTTP 服务器，因此服务器内置了额外的保护，以防止从网页中滥用 API。如果您想从网页启用对 API 的访问，您必须将服务器配置为接受带有该 `--http.corsdomain` 标志的跨域请求。

示例：如果您想将 [Remix](https://remix.ethereum.org/) 与 geth一起使用，请允许来自 remix 域的请求。

	geth --http --http.corsdomain https://remix.ethereum.org
使用 `--http.corsdomain '*'`，以便从任何来源获得。
### 网络套接字服务器
WebSocket 端点的配置类似于 HTTP 传输。要启用 WebSocket 访问，请使用 `--ws` 标志。默认 WebSocket 端口为 `8546`。

`--ws.addr、--ws.port `和 `--ws.api` 标志可用于自定义 WebSocket 服务器的设置。

	geth --ws --ws.port 3334 --ws.api eth,net,web3
跨域请求保护也适用于 WebSocket 服务器。使用该 `--ws.origins` 标志允许从网页访问服务器：

	geth --ws --ws.origins http://myapp.example.com
与一样 `--http.corsdomain`，使用 `--ws.origins '*'` 允许从任何来源访问。
### 工控机服务器
JSON-RPC API 也在 UNIX 域套接字上提供。该服务器默认启用，可以访问所有 J​​SON-RPC 命名空间。

默认情况下，侦听套接字放置在数据目录中。在 Linux 和 macOS 上，geth 套接字的默认位置是

	~/.ethereum/geth.ipc
在 Windows 上，IPC 是通过命名管道提供的。geth管道的默认位置是：

	\\.\pipe\geth.ipc
您可以使用该--ipcpath标志配置套接字的位置。可以使用该--ipcdisable标志禁用 IPC 。

## 实时事件
Geth v1.4 及更高版本支持使用 JSON-RPC 通知发布/订阅。这允许客户端等待事件而不是轮询它们。

它通过订阅特定事件来工作。该节点将返回订阅 ID。对于与订阅匹配的每个事件，带有相关数据的通知与订阅 ID 一起发送。

例子：

	// create subscription //创建订阅
	>> {"id": 1, "method": "eth_subscribe", "params": ["newHeads", {}]}
	<< {"jsonrpc":"2.0","id":1,"result":"0xcd0c3e8af590364c09d0fa6a1210faf5"}
	
	// incoming notifications //传入通知
	<< {"jsonrpc":"2.0","method":"eth_subscription","params":{"subscription":"0xcd0c3e8af590364c09d0fa6a1210faf5","result":{"difficulty":"0xd9263f42a87",<...>, "uncles":[]}}}
	<< {"jsonrpc":"2.0","method":"eth_subscription","params":{"subscription":"0xcd0c3e8af590364c09d0fa6a1210faf5","result":{"difficulty":"0xd90b1a7ad02", <...>, "uncles":["0x80aacd1ea4c9da32efd8c2cc9ab38f8f70578fcd46a1a4ed73f82f3e0957f936"]}}}
	
	// cancel subscription //取消订阅
	>> {"id": 1, "method": "eth_unsubscribe", "params": ["0xcd0c3e8af590364c09d0fa6a1210faf5"]}
	<< {"jsonrpc":"2.0","id":1,"result":true}
#### 注意事项
1. 通知是针对当前事件而不是过去事件发送的。如果您的用例要求您不要错过任何通知，那么订阅可能不是最佳选择。
2. 订阅需要全双工连接。Geth 以 WebSocket 和 IPC（默认启用）的形式提供此类连接。
3. 订阅与连接耦合。如果连接关闭，则删除通过此连接创建的所有订阅。
4. 通知存储在内部缓冲区中并从该缓冲区发送到客户端。如果客户端无法跟上并且缓冲通知的数量达到限制（当前为 10k），则连接将关闭。请记住，订阅某些事件可能会导致大量通知，例如在节点开始同步时侦听所有日志/块。

### 创建订阅
使用 `eth_subscribeas` 方法和订阅名称作为第一个参数的常规 RPC 调用创建订阅。如果成功，则返回订阅 ID。

- 参数
	- 订阅名称
	- 可选参数
- 例子

		>> {"id": 1, "method": "eth_subscribe", "params": ["newHeads"]}
		<< {"id": 1, "jsonrpc": "2.0", "result": "0x9cef478923ff08bf67fde6c64013158d"}

### 取消订阅
使用 `eth_unsubscribeas` 方法和订阅 ID 作为第一个参数的常规 RPC 调用取消订阅。它返回一个布尔值，指示订阅是否成功取消。

- 参数
	- 订阅号
- 例子

		>> {"id": 1, "method": "eth_unsubscribe", "params": ["0x9cef478923ff08bf67fde6c64013158d"]}
		<< {"jsonrpc":"2.0","id":1,"result":true}

### 支持的订阅
#### 新头
每次将新标头附加到链时触发通知，包括链重组。用户可以使用布隆过滤器来确定块是否包含他们感兴趣的日志。

在链重组的情况下，订阅将发出新链的所有新标头。因此订阅可以在相同的高度发出多个标题。
##### 例子

	>> {"id": 1, "method": "eth_subscribe", "params": ["newHeads"]}
	<< {"jsonrpc":"2.0","id":2,"result":"0x9ce59a13059e417087c02d3236a0b1cc"}
	
	<< {
	       "jsonrpc": "2.0",
	       "method": "eth_subscription",
	       "params": {
	         "result": {
	           "difficulty": "0x15d9223a23aa",
	           "extraData": "0xd983010305844765746887676f312e342e328777696e646f7773",
	           "gasLimit": "0x47e7c4",
	           "gasUsed": "0x38658",
	           "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
	           "miner": "0xf8b483dba2c3b7176a3da549ad41a48bb3121069",
	           "nonce": "0x084149998194cc5f",
	           "number": "0x1348c9",
	           "parentHash": "0x7736fab79e05dc611604d22470dadad26f56fe494421b5b333de816ce1f25701",
	           "receiptRoot": "0x2fab35823ad00c7bb388595cb46652fe7886e00660a01e867824d3dceb1c8d36",
	           "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
	           "stateRoot": "0xb3346685172db67de536d8765c43c31009d0eb3bd9c501c9be3229203f15f378",
	           "timestamp": "0x56ffeff8",
	           "transactionsRoot": "0x0167ffa60e3ebc0b080cdb95f7c0087dd6c0e61413140e39d94d3468d7c9689f"
	         },
	         "subscription": "0x9ce59a13059e417087c02d3236a0b1cc"
	       }
	     }
### 日志
返回包含在新导入块中并匹配给定过滤条件的日志。

如果链重组，以前发送的旧链上的日志将重新发送，并将 `removed` 属性设置为 true。来自在新链中结束的交易的日志被发出。因此，订阅可以多次发出同一事务的日志。

- 参数
	- `object` 具有以下（可选）字段
		- `address`，地址或地址数组。仅返回从这些地址创建的日志（可选）
		- `topics`，只有匹配指定主题的日志（可选）
- 例子

		>> {"id": 1, "method": "eth_subscribe", "params": ["logs", {"address": "0x8320fe7702b96808f7bbc0d4a888ed1468216cfd", "topics": ["0xd78a0cb8bb633d06981248b816e7bd33c2a35a6089241d099fa519e361cab902"]}]}
		<< {"jsonrpc":"2.0","id":2,"result":"0x4a8a4c0517381924f9838102c5a4dcb7"}
		
		<< {"jsonrpc":"2.0","method":"eth_subscription","params": {"subscription":"0x4a8a4c0517381924f9838102c5a4dcb7","result":{"address":"0x8320fe7702b96808f7bbc0d4a888ed1468216cfd","blockHash":"0x61cdb2a09ab99abf791d474f20c2ea89bf8de2923a2d42bb49944c8c993cbf04","blockNumber":"0x29e87","data":"0x00000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000003","logIndex":"0x0","topics":["0xd78a0cb8bb633d06981248b816e7bd33c2a35a6089241d099fa519e361cab902"],"transactionHash":"0xe044554a0a55067caafd07f8020ab9f2af60bdfe337e395ecd84b4877a3d1ab4","transactionIndex":"0x0"}}}

### 新的待处理交易
返回添加到挂起状态并使用节点中可用的密钥签名的所有交易的哈希值。

当先前属于规范链一部分的交易在重组后不再属于新规范链时，它会再次发出。

- 参数

	没有任何
- 例子

		>> {"id": 1, "method": "eth_subscribe", "params": ["newPendingTransactions"]}
		<< {"jsonrpc":"2.0","id":2,"result":"0xc3b33aa549fb9a60e95d21862596617c"}
		<< {
		        "jsonrpc":"2.0",
		        "method":"eth_subscription",
		        "params":{
		            "subscription":"0xc3b33aa549fb9a60e95d21862596617c",
		            "result":"0xd6fdc5cc41a9959e922f30cb772a9aef46f4daea279307bc5f7024edc4ccd7fa"
		        }
		   }
		   
### 同步
指示节点何时开始或停止同步。结果可以是指示同步已开始 (true)、已完成 (false) 的布尔值或具有各种进度指示器的对象。

- 参数

	没有任何
- 例子

		>> {"id": 1, "method": "eth_subscribe", "params": ["syncing"]}
		<< {"jsonrpc":"2.0","id":2,"result":"0xe2ffeb2703bcf602d42922385829ce96"}
		
		<< {"subscription":"0xe2ffeb2703bcf602d42922385829ce96","result":{"syncing":true,"status":{"startingBlock":674427,"currentBlock":67400,"highestBlock":674432,"pulledStates":0,"knownStates":0}}}}

## 管理员命名空间
该 `admin` API 使您可以访问多种非标准 RPC 方法，这将允许您对 Geth 实例进行细粒度控制，包括但不限于网络对等点和 RPC 端点管理。

- admin_addPeer
- admin_datadir
- admin_nodeInfo
- admin_peers
- admin_startRPC
- admin_startWS
- admin_stopRPC
- admin_stopWS

### admin_addPeer
该 `addPeer` 管理方法要求增加一个新的远程节点跟踪静态节点列表。该节点将始终尝试保持与这些节点的连接，如果远程连接断开，则每隔一段时间重新连接一次。

该方法接受单个参数，即 [enode](https://github.com/ethereum/wiki/wiki/enode-url-format) 要开始跟踪的远程对等方的URL，并返回一个 `BOOL` 指示对等方是否被接受进行跟踪或发生了某些错误。

客户端|方法调用
---|---
go|admin.AddPeer(url string) (bool, error)
console|admin.addPeer(url)
RPC|{"method": "admin_addPeer", "params": [url]}

- 例子
	
		> admin.addPeer("enode://a979fb575495b8d6db44f750317d0f4622bf4c2aa3365d6af7c284339968eef29b69ad0dce72a4d8db5ebb4968de0e3bec910127f134779fbcb0cb6d3331163c@52.16.188.185:30303")
		true

### admin_datadir
该 `datadir` 可以被询问运行 Geth 的节点当前用来存储所有数据库的绝对路径。

客户端|方法调用
---|---
go|admin.Datadir() (string, error)
console|admin.datadir
RPC|{"method": "admin_datadir"}

- 例子

		> admin.datadir
		"/home/john/.ethereum"

### admin_nodeInfo
该 `nodeInfo`  可以被询问所有已知的关于在网络粒度运行 Geth 的节点的信息。这些包括作为 [ÐΞVp2p](https://github.com/ethereum/wiki/wiki/%C3%90%CE%9EVp2p-Wire-Protocol) P2P 覆盖协议的参与者的节点本身的一般信息，以及由每个正在运行的应用程序协议（例如 `eth，les，shh，bzz`）添加的专门信息。

客户端|方法调用
---|---
go|admin.NodeInfo() (*p2p.NodeInfo, error)
console|admin.nodeInfo
RPC|{"method": "admin_nodeInfo"}

- 例子

		> admin.nodeInfo
		{
		  enode: "enode://44826a5d6a55f88a18298bca4773fca5749cdc3a5c9f308aa7d810e9b31123f3e7c5fba0b1d70aac5308426f47df2a128a6747040a3815cc7dd7167d03be320d@[::]:30303",
		  id: "44826a5d6a55f88a18298bca4773fca5749cdc3a5c9f308aa7d810e9b31123f3e7c5fba0b1d70aac5308426f47df2a128a6747040a3815cc7dd7167d03be320d",
		  ip: "::",
		  listenAddr: "[::]:30303",
		  name: "Geth/v1.5.0-unstable/linux/go1.6",
		  ports: {
		    discovery: 30303,
		    listener: 30303
		  },
		  protocols: {
		    eth: {
		      difficulty: 17334254859343145000,
		      genesis: "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3",
		      head: "0xb83f73fbe6220c111136aefd27b160bf4a34085c65ba89f24246b3162257c36a",
		      network: 1
		    }
		  }
		}
		
### admin_peers
该 `peers` 可以被询问所有已知的关于在网络粒度连接到远程节点的信息。这些包括关于节点本身作为 ÐΞVp2p P2P 覆盖协议的参与者的一般信息，以及由每个正在运行的应用程序协议（例如 `eth，les，shh，bzz` ）添加的专门信息。

客户端|方法调用
---|---
go|admin.Peers() ([]*p2p.PeerInfo, error)
console|admin.peers
RPC|{"method": "admin_peers"}

- 例子

		> admin.peers
		[{
		    caps: ["eth/61", "eth/62", "eth/63"],
		    id: "08a6b39263470c78d3e4f58e3c997cd2e7af623afce64656cfc56480babcea7a9138f3d09d7b9879344c2d2e457679e3655d4b56eaff5fd4fd7f147bdb045124",
		    name: "Geth/v1.5.0-unstable/linux/go1.5.1",
		    network: {
		      localAddress: "192.168.0.104:51068",
		      remoteAddress: "71.62.31.72:30303"
		    },
		    protocols: {
		      eth: {
		        difficulty: 17334052235346465000,
		        head: "5794b768dae6c6ee5366e6ca7662bdff2882576e09609bf778633e470e0e7852",
		        version: 63
		      }
		    }
		}, /* ... */ {
		    caps: ["eth/61", "eth/62", "eth/63"],
		    id: "fcad9f6d3faf89a0908a11ddae9d4be3a1039108263b06c96171eb3b0f3ba85a7095a03bb65198c35a04829032d198759edfca9b63a8b69dc47a205d94fce7cc",
		    name: "Geth/v1.3.5-506c9277/linux/go1.4.2",
		    network: {
		      localAddress: "192.168.0.104:55968",
		      remoteAddress: "121.196.232.205:30303"
		    },
		    protocols: {
		      eth: {
		        difficulty: 17335165914080772000,
		        head: "5794b768dae6c6ee5366e6ca7662bdff2882576e09609bf778633e470e0e7852",
		        version: 63
		      }
		    }
		}]
		
### admin_startRPC
该 `startRPC` 管理方法开始基于 HTTP 的 [JSON RPC API](https://www.jsonrpc.org/specification) 服务器无法处理客户端请求。所有参数都是可选的：

- `host`：打开侦听器套接字的网络接口（默认为 `"localhost"`）
- `port`：打开侦听器套接字的网络端口（默认为 `8545`）
- `cors`：要使用的[跨源资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)标头（默认为 `""`）
- `apis`：通过此接口提供的 API 模块（默认为 `"eth,net,web3"`）

该方法返回一个布尔标志，指定是否打开了 HTTP RPC 侦听器。请注意，任何时候都只允许一个 HTTP 端点处于活动状态。

客户端|方法调用
---|---
go|admin.StartRPC(host *string, port *rpc.HexNumber, cors *string, apis *string) (bool, error)
console|admin.startRPC(host, port, cors, apis)
RPC|{"method": "admin_startRPC", "params": [host, port, cors, apis]}

- 例子

		> admin.startRPC("127.0.0.1", 8545)
		true

### admin_startWS
该 `startWS` 管理方法开始的 WebSocket 的基于JSON RPC API 服务器无法处理客户端请求。所有参数都是可选的：

- `host`：打开侦听器套接字的网络接口（默认为 `"localhost"`）
- `port`：打开侦听器套接字的网络端口（默认为 `8546`）
- `cors`：要使用的跨源资源共享标头（默认为 `""`）
- `apis`：通过此接口提供的 API 模块（默认为 `"eth,net,web3"`）

该方法返回一个布尔标志，指定是否打开了 WebSocket RPC 侦听器。请注意，任何时候都只允许一个 WebSocket 端点处于活动状态。

客户端|方法调用
---|---
go|admin.StartWS(host *string, port *rpc.HexNumber, cors *string, apis *string) (bool, error)
console|admin.startWS(host, port, cors, apis)
RPC|{"method": "admin_startWS", "params": [host, port, cors, apis]}

- 例子

		> admin.startWS("127.0.0.1", 8546)
		true

### admin_stopRPC
该 `stopRPC` 管理方法关闭当前打开的 HTTP RPC 端点。由于节点只能运行一个 HTTP 端点，因此此方法不带任何参数，无论端点是否关闭都返回一个布尔值。

客户端|方法调用
---|---
go|admin.StopRPC() (bool, error)
console|admin.stopRPC()
RPC|{"method": "admin_stopRPC"

- 例子

		> admin.stopRPC()
		true

### admin_stopWS
该 `stopWS` 管理方法关闭当前打开的 WebSocket 的 RPC 终结点。由于节点只能运行一个 WebSocket 端点，因此此方法不带任何参数，无论端点是否关闭都返回一个布尔值。

客户端|方法调用
---|---
go|admin.StopWS() (bool, error)
console | admin.stopWS()
RPC|{"method": "admin_stopWS"

- 例子

		> admin.stopWS()
		true

## clique 命名空间
该 clique API 提供了访问 clique 共识发动机的状态。您可以使用此 API 来管理签名者投票并检查专用网络的健康状况。

- click_getSnapshot
- click_getSnapshotAtHash
- click_getSigners
- click_proposals
- click_propose
- click_discard
- clique_status

### click_getSnapshot
检索给定块中所有 clique 状态的快照。

客户端|方法调用
---|---
console |clique.getSnapshot(blockNumber)
RPC|{"method": "clique_getSnapshot", "params": [blockNumber]}

- 例子

		> clique.getSnapshot(5463755)
		{
		  hash: "0x018194fc50ca62d973e2f85cffef1e6811278ffd2040a4460537f8dbec3d5efc",
		  number: 5463755,
		  recents: {
		    5463752: "0x42eb768f2244c8811c63729a21a3569731535f06",
		    5463753: "0x6635f83421bf059cd8111f180f0727128685bae4",
		    5463754: "0x7ffc57839b00206d1ad20c69a1981b489f772031",
		    5463755: "0xb279182d99e65703f0076e4812653aab85fca0f0"
		  },
		  signers: {
		    0x42eb768f2244c8811c63729a21a3569731535f06: {},
		    0x6635f83421bf059cd8111f180f0727128685bae4: {},
		    0x7ffc57839b00206d1ad20c69a1981b489f772031: {},
		    0xb279182d99e65703f0076e4812653aab85fca0f0: {},
		    0xd6ae8250b8348c94847280928c79fb3b63ca453e: {},
		    0xda35dee8eddeaa556e4c26268463e26fb91ff74f: {},
		    0xfc18cbc391de84dbd87db83b20935d3e89f5dd91: {}
		  },
		  tally: {},
		  votes: []
		}

### click_getSnapshotAtHash
检索给定块的状态快照。

客户端|方法调用
---|---
console |clique.getSnapshotAtHash(blockHash)
RPC|{"method": "clique_getSnapshotAtHash", "params": [blockHash]}
### click_getSigners
检索指定区块的授权签名者列表。

客户端|方法调用
---|---
console |clique.getSigners(blockNumber)
RPC|{"method": "clique_getSigners", "params": [blockNumber]}
### click_propose
添加一个新的授权提议，签名者将尝试通过该提议。如果该 auth 参数为真，则本地签名者投票将给定地址包含在授权签名者集合中。随着 auth 设置 false，投票是针对地址。

客户端|方法调用
---|---
console |clique.propose(address, auth)
RPC|{"method": "clique_propose", "params": [address, auth]}
### click_proposals
返回节点正在投票的当前提案。

客户端|方法调用
---|---
console |clique.proposals()
RPC|{"method": "clique_proposals", "params": []}
### click_discard
此方法删除当前正在运行的提案。签名者不会对该地址进一步投票（支持或反对）。

客户端|方法调用
---|---
console |clique.discard(address)
RPC|{"method": "clique_discard", "params": [address]}
### clique_status
这是一种调试方法，它返回有关最近 64 个块的签名者活动的统计信息。返回的对象包含以下字段：

- `inturnPercent`: 轮流签名的区块百分比
- `sealerActivity`: 包含签名者地址和由他们签名的块数的对象
- `numBlocks`：分析的块数

客户端|方法调用
---|---
console |clique.status()
RPC|{"method": "clique_status", "params": []}

例子：

	> clique.status()
	{
	  inturnPercent: 100,
	  numBlocks: 64,
	  sealerActivity: {
	    0x42eb768f2244c8811c63729a21a3569731535f06: 9,
	    0x6635f83421bf059cd8111f180f0727128685bae4: 9,
	    0x7ffc57839b00206d1ad20c69a1981b489f772031: 9,
	    0xb279182d99e65703f0076e4812653aab85fca0f0: 10,
	    0xd6ae8250b8348c94847280928c79fb3b63ca453e: 9,
	    0xda35dee8eddeaa556e4c26268463e26fb91ff74f: 9,
	    0xfc18cbc391de84dbd87db83b20935d3e89f5dd91: 9
	  }
	}


## Debug 命名空间
该 `debug` API让你在运行时获得一些非标准的 RPC 方法，这将让你检查，调试和设置某些调试标志。

- `debug_backtraceAt`
- `debug_blockProfile`
- `debug_cpuProfile`
- `debug_dumpBlock`
	- `Example`
- `debug_gcStats`
- `debug_getBlockRlp`
- `debug_goTrace`
- `debug_memStats`
- `debug_seedHash`
- `debug_setHead`
- `debug_setBlockProfileRate`
- `debug_stacks`
- `debug_startCPUProfile`
- `debug_startGoTrace`
- `debug_stopCPUProfile`
- `debug_stopGoTrace`
- `debug_traceBlock`
	- `Example`
- `debug_traceBlockByNumber`
- `debug_traceBlockByHash`
- `debug_traceBlockFromFile`
- `debug_standardTraceBlockToFile`
- `debug_standardTraceBadBlockToFile`
- `debug_traceTransaction`
	- `Example`
	- `JavaScript-based tracing`
		- `Step`
		- `Result`
		- `Fault`
		- `Enter & Exit`
		- `Usage`
- `debug_traceCall`
	- `Example`
- `debug_verbosity`
- `debug_vmodule`
	- `Examples`
- `debug_writeBlockProfile`
- `debug_writeMemProfile`

### debug_backtraceAt
设置日志回溯位置。当设置回溯位置并在该位置发出日志消息时，执行日志语句的 goroutine 的堆栈将打印到 stderr。

该位置指定为 `<filename>:<line>`。

客户端|方法调用
---|---
console |debug.backtraceAt(string)
RPC|{"method": "debug_backtraceAt", "params": [string]}

例子：

	> debug.backtraceAt("server.go:443")
### debug_blockProfile
在给定的持续时间内打开块分析并将分析数据写入磁盘。它使用 1 的配置文件率来获取最准确的信息。如果需要不同的速率，请设置速率并使用 手动编写配置文件 `debug_writeBlockProfile`。

客户端|方法调用
---|---
console |debug.blockProfile(file, seconds)
RPC|{"method": "debug_blockProfile", "params": [string, number]}

### debug_cpuProfile
在给定的持续时间内打开 CPU 分析并将分析数据写入磁盘。

客户端|方法调用
---|---
console |debug.cpuProfile(file, seconds)
RPC|{"method": "debug_cpuProfile", "params": [string, number]}

### debug_dumpBlock
检索与区块号对应的状态并返回账户列表（包括存储和代码）。

客户端|方法调用
---|---
go|debug.DumpBlock(number uint64) (state.World, error)
console |debug.traceBlockByHash(number, [options])
RPC|{"method": "debug_dumpBlock", "params": [number]}

- 例子

		> debug.dumpBlock(10)
		{
		    fff7ac99c8e4feb60c9750054bdc14ce1857f181: {
		      balance: "49358640978154672",
		      code: "",
		      codeHash: "c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470",
		      nonce: 2,
		      root: "56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
		      storage: {}
		    },
		    fffbca3a38c3c5fcb3adbb8e63c04c3e629aafce: {
		      balance: "3460945928",
		      code: "",
		      codeHash: "c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470",
		      nonce: 657,
		      root: "56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
		      storage: {}
		    }
		  },
		  root: "19f4ed94e188dd9c7eb04226bd240fa6b449401a6c656d6d2816a87ccaf206f1"
		}

### debug_gcStats
返回 GC 统计信息。

有关返回对象的字段的信息，请参阅 [https://golang.org/pkg/runtime/debug/#GCStats](https://golang.org/pkg/runtime/debug/#GCStats)。

客户端|方法调用
---|---
console |debug.gcStats()
RPC|{"method": "debug_gcStats", "params": []}

### debug_getBlockRlp
按编号检索并返回 RLP 编码块。

客户端|方法调用
---|---
go|debug.GetBlockRlp(number uint64) (string, error)
console |debug.getBlockRlp(number, [options])
RPC|{"method": "debug_getBlockRlp", "params": [number]}
参考资料：[RLP](https://github.com/ethereum/wiki/wiki/RLP)

### debug_goTrace
在给定的持续时间内打开 Go 运行时跟踪并将跟踪数据写入磁盘。

客户端|方法调用
---|---
console |debug.goTrace(file, seconds)
RPC|{"method": "debug_goTrace", "params": [string, number]}

### debug_memStats
返回详细的运行时内存统计信息。

有关返回对象的字段的信息，请参阅 [https://golang.org/pkg/runtime/#MemStats](https://golang.org/pkg/runtime/#MemStats)。

客户端|方法调用
---|---
console |debug.memStats()
RPC|{"method": "debug_memStats", "params": []}
### debug_seedHash
按编号获取检索块的种子哈希

客户端|方法调用
---|---
go|debug.SeedHash(number uint64) (string, error)
console |debug.seedHash(number, [options])
RPC|{"method": "debug_seedHash", "params": [number]}
### debug_setHead(危险操作)
通过块号设置本地链的当前头。请注意，这是一种破坏性操作，可能会严重损坏您的链条。使用带有极度谨慎。

客户端|方法调用
---|---
go|debug.SetHead(number uint64)
console |debug.setHead(number)
RPC|{"method": "debug_setHead", "params": [number]}

参考资料： [Ethash](https://eth.wiki/en/concepts/ethash/ethash)
### debug_setBlockProfileRate
设置 goroutine 块配置文件数据收集的速率（以样本/秒为单位）。非零速率启用块分析，将其设置为零停止分析。收集的配置文件数据可以使用`debug_writeBlockProfile`.

客户端|方法调用
---|---
console |debug.setBlockProfileRate(rate)
RPC|{"method": "debug_setBlockProfileRate", "params": [number]}
### debug_stacks
返回所有 goroutine 堆栈的打印表示。请注意，此方法的 web3 包装器负责打印并且不返回字符串。

客户端|方法调用
---|---
console |debug.stacks()
RPC|{"method": "debug_stacks", "params": []}
### debug_startCPUProfile
无限期地打开 CPU 分析，写入给定文件。

客户端|方法调用
---|---
console |debug.startCPUProfile(file)
RPC|{"method": "debug_startCPUProfile", "params": [string]}
### debug_startGoTrace
开始将 Go 运行时跟踪写入给定文件。

客户端|方法调用
---|---
console |debug.startGoTrace(file)
RPC|{"method": "debug_startGoTrace", "params": [string]}
### debug_stopCPUProfile
停止正在进行的 CPU 配置文件。

客户端|方法调用
---|---
console |debug.stopCPUProfile()
RPC|{"method": "debug_stopCPUProfile", "params": []}
### debug_stopGoTrace
停止编写 Go 运行时跟踪。

客户端|方法调用
---|---
console |debug.startGoTrace(file)
RPC|{"method": "debug_stopGoTrace", "params": []}
### debug_traceBlock
该 traceBlock 方法将返回包含在此块中的所有事务的所有调用操作码的完整堆栈跟踪。请注意，此块的父级必须存在，否则将失败。

客户端|方法调用
---|---
go|debug.TraceBlock(blockRlp []byte, config. *vm.Config) BlockTraceResult
console |debug.traceBlock(tblockRlp, [options])
RPC|{"method": "debug_traceBlock", "params": [blockRlp, {}]}

参考资料： RLP

- 例子

		> debug.traceBlock("0xblock_rlp")
		{
		  gas: 85301,
		  returnValue: "",
		  structLogs: [{
		      depth: 1,
		      error: "",
		      gas: 162106,
		      gasCost: 3,
		      memory: null,
		      op: "PUSH1",
		      pc: 0,
		      stack: [],
		      storage: {}
		  },
		    /* snip */
		  {
		      depth: 1,
		      error: "",
		      gas: 100000,
		      gasCost: 0,
		      memory: ["0000000000000000000000000000000000000000000000000000000000000006", "0000000000000000000000000000000000000000000000000000000000000000", "0000000000000000000000000000000000000000000000000000000000000060"],
		      op: "STOP",
		      pc: 120,
		      stack: ["00000000000000000000000000000000000000000000000000000000d67cbec9"],
		      storage: {
		        0000000000000000000000000000000000000000000000000000000000000004: "8241fa522772837f0d05511f20caa6da1d5a3209000000000000000400000001",
		        0000000000000000000000000000000000000000000000000000000000000006: "0000000000000000000000000000000000000000000000000000000000000001",
		        f652222313e28459528d920b65115c16c04f3efc82aaedc97be59f3f377c0d3f: "00000000000000000000000002e816afc1b5c0f39852131959d946eb3b07b5ad"
		      }
		  }]

### debug_traceBlockByNumber
与 [debug_traceBlock](https://geth.ethereum.org/docs/rpc/ns-debug#debug_traceblock) 类似，`traceBlockByNumber` 接受一个块号并重放已经存在于数据库中的块。

客户端|方法调用
---|---
go|debug.TraceBlockByNumber(number uint64, config. *vm.Config) BlockTraceResult
console |debug.traceBlockByNumber(number, [options])
RPC|{"method": "debug_traceBlockByNumber", "params": [number, {}]}

参考资料： RLP
### debug_traceBlockByHash
与 debug_traceBlock 类似，traceBlockByHash 接受块哈希并重放数据库中已经存在的块。

客户端|方法调用
---|---
go|debug.TraceBlockByHash(hash common.Hash, config. *vm.Config) BlockTraceResult
console |debug.traceBlockByHash(hash, [options])
RPC|{"method": "debug_traceBlockByHash", "params": [hash {}]}

参考资料： RLP
### debug_traceBlockFromFile
与 debug_traceBlock 类似，traceBlockFromFile 接受包含块的 RLP 的文件。

客户端|方法调用
---|---
go|debug.TraceBlockFromFile(fileName string, config. *vm.Config) BlockTraceResult
console |debug.traceBlockFromFile(fileName, [options])
RPC|{"method": "debug_traceBlockFromFile", "params": [fileName, {}]}

参考资料： RLP
### debug_standardTraceBlockToFile
当首次实现基于 JS 的跟踪（见下文）时，预期用例是启用长时间运行的跟踪器，可以通过订阅通道将结果流回。这种方法的工作方式有点不同。（有关详细信息，请参阅 [PR](https://github.com/ethereum/go-ethereum/pull/17914)）

- 它在执行过程中将输出流式传输到磁盘，以免破坏节点上的内存使用量
- 它 `jsonl` 用作输出格式（以允许流式传输）
- 使用跨客户端标准化输出，即所谓的“标准 json”
	- 用途 `op` 为操作码的字符串表示，而不是 `op/opName` 对数字/串和其他小 simlar 差异。
	- 已 `refund`
	- 将内存表示为连续的数据块，而不是32像 -byte 段的列表 `debug_traceTransaction`
	
这意味着此方法仅对控制节点的调用者“有用”——至少足以在事后从文件系统读取人工制品。

该方法可用于从给定块中转储某个交易：

	> debug.standardTraceBlockToFile("0x0bbe9f1484668a2bf159c63f0cf556ed8c8282f99e3ffdb03ad2175a863bca63", {txHash:"0x4049f61ffbb0747bb88dc1c85dd6686ebf225a3c10c282c45a8e0c644739f7e9", disableMemory:true})
	["/tmp/block_0x0bbe9f14-14-0x4049f61f-099048234"]
或者来自一个区块的所有交易：

	> debug.standardTraceBlockToFile("0x0bbe9f1484668a2bf159c63f0cf556ed8c8282f99e3ffdb03ad2175a863bca63", {disableMemory:true})
	["/tmp/block_0x0bbe9f14-0-0xb4502ea7-409046657", "/tmp/block_0x0bbe9f14-1-0xe839be8f-954614764", "/tmp/block_0x0bbe9f14-2-0xc6e2052f-542255195", "/tmp/block_0x0bbe9f14-3-0x01b7f3fe-209673214", "/tmp/block_0x0bbe9f14-4-0x0f290422-320999749", "/tmp/block_0x0bbe9f14-5-0x2dc0fb80-844117472", "/tmp/block_0x0bbe9f14-6-0x35542da1-256306111", "/tmp/block_0x0bbe9f14-7-0x3e199a08-086370834", "/tmp/block_0x0bbe9f14-8-0x87778b88-194603593", "/tmp/block_0x0bbe9f14-9-0xbcb081ba-629580052", "/tmp/block_0x0bbe9f14-10-0xc254381a-578605923", "/tmp/block_0x0bbe9f14-11-0xcc434d58-405931366", "/tmp/block_0x0bbe9f14-12-0xce61967d-874423181", "/tmp/block_0x0bbe9f14-13-0x05a20b35-267153288", "/tmp/block_0x0bbe9f14-14-0x4049f61f-606653767", "/tmp/block_0x0bbe9f14-15-0x46d473d2-614457338", "/tmp/block_0x0bbe9f14-16-0x35cf5500-411906321", "/tmp/block_0x0bbe9f14-17-0x79222961-278569788", "/tmp/block_0x0bbe9f14-18-0xad84e7b1-095032683", "/tmp/block_0x0bbe9f14-19-0x4bd48260-019097038", "/tmp/block_0x0bbe9f14-20-0x1517411d-292624085", "/tmp/block_0x0bbe9f14-21-0x6857e350-971385904", "/tmp/block_0x0bbe9f14-22-0xbe3ae2ca-236639695"]

文件是在临时位置创建的，命名标准为 `block_<blockhash:4>-<txindex>-<txhash:4>-<random suffix>.` 每个操作码都会立即流式传输到文件，除了操作系统通常所做的任何缓冲之外，没有 in-geth 缓冲。

在服务器端，它还会在重新生成历史状态时添加更多信息，即 `required historical state is not avaiable` 遇到 `reexec-number` 时，因此用户可以尝试增加该设置。它还打印出剩余的块，直到它达到目标：

	INFO [10-15|13:48:25.263] Regenerating historical state            block=2385959 target=2386012 remaining=53   elapsed=3m30.990537767s
	INFO [10-15|13:48:33.342] Regenerating historical state            block=2386012 target=2386012 remaining=0    elapsed=3m39.070073163s
	INFO [10-15|13:48:33.343] Historical state regenerated             block=2386012 elapsed=3m39.070454362s nodes=10.03mB preimages=652.08kB
	INFO [10-15|13:48:33.352] Wrote trace                              file=/tmp/block_0x14490c57-0-0xfbbd6d91-715824834
	INFO [10-15|13:48:33.352] Wrote trace                              file=/tmp/block_0x14490c57-1-0x71076194-187462969
	INFO [10-15|13:48:34.421] Wrote trace file=/tmp/block_0x14490c57-2-0x3f4263fe-056924484
该 `options` 如下：

	type StdTraceConfig struct {
	  *vm.LogConfig
	  Reexec *uint64
	  TxHash *common.Hash
	}

### debug_standardTraceBadBlockToFile
此方法类似于 `debug_standardTraceBlockToFile` ，但可用于获取有关已被拒绝为无效（出于某种原因）的块的信息。
### debug_traceTransaction
OBS 在大多数场景下，`debug.standardTraceBlockToFile` 更适合追踪！

`traceTransaction` 的调试方法将尝试在网络上执行以完全相同的方式来运行事务。在最终尝试执行与给定散列对应的事务之前，它将重放在此事务之前可能已执行的任何事务。

除了交易的哈希值之外，您还可以给它一个次要的可选参数，用于指定此特定调用的选项。可能的选项是：

- `disableStorage：BOOL`。将此设置为 true 将禁用存储捕获（默认值 = false）。
- `disableMemory：BOOL`。将此设置为 true 将禁用内存捕获（默认值 = false）。
- `disableStack：BOOL`。将此设置为 true 将禁用堆栈捕获（默认值 = false）。
- `tracer：STRING`。设置此项将启用基于 JavaScript 的事务跟踪，如下所述。如果设置，前四个参数将被忽略。
- `timeout：STRING`。覆盖基于 JavaScript 的跟踪调用的默认超时 5 秒。此处描述了有效值。

客户端|方法调用
---|---
go|debug.TraceTransaction(txHash common.Hash, logger *vm.LogConfig) (*ExecutionResurt, error)
console |debug.traceTransaction(txHash, [options])
RPC|{"method": "debug_traceTransaction", "params": [txHash, {}]}
 
- 例子

		> debug.traceTransaction("0x2059dd53ecac9827faad14d364f9e04b1d5fe5b506e3acc886eff7a6f88a696a")
		{
		  gas: 85301,
		  returnValue: "",
		  structLogs: [{
		      depth: 1,
		      error: "",
		      gas: 162106,
		      gasCost: 3,
		      memory: null,
		      op: "PUSH1",
		      pc: 0,
		      stack: [],
		      storage: {}
		  },
		    /* snip 剪断 */
		  {
		      depth: 1,
		      error: "",
		      gas: 100000,
		      gasCost: 0,
		      memory: ["0000000000000000000000000000000000000000000000000000000000000006", "0000000000000000000000000000000000000000000000000000000000000000", "0000000000000000000000000000000000000000000000000000000000000060"],
		      op: "STOP",
		      pc: 120,
		      stack: ["00000000000000000000000000000000000000000000000000000000d67cbec9"],
		      storage: {
		        0000000000000000000000000000000000000000000000000000000000000004: "8241fa522772837f0d05511f20caa6da1d5a3209000000000000000400000001",
		        0000000000000000000000000000000000000000000000000000000000000006: "0000000000000000000000000000000000000000000000000000000000000001",
		        f652222313e28459528d920b65115c16c04f3efc82aaedc97be59f3f377c0d3f: "00000000000000000000000002e816afc1b5c0f39852131959d946eb3b07b5ad"
		      }
		  }]

### JavaScript-based tracing
`tracer` 在第二个参数中指定选项可启用基于 JavaScript 的跟踪。在这种模式下，`tracer` 被解释为一个 JavaScript 表达式，该表达式预期评估为必须公开 `result` 和 `fault` 方法的对象。

存在 3 种附加方法，即：`step、enter` 和 `exit` 。您必须提供 `step` 或 `enterAND exit`（即这两个必须一起公开）。如果您选择这样做，您可以公开所有三个。

- step

	step 是一个函数，它接受两个参数 log 和 db 并在 EVM 的每个步骤或发生错误时调用，因为跟踪指定的事务。

	- `log` 具有以下字段：
		- `op`: 对象，一个表示当前操作码的 OpCode 对象
		- `stack`：array[big.Int]，EVM 执行栈
		- `memory`: 对象，表示合约内存空间的结构体
		- `contract`: 对象，代表执行当前操作的账户的对象

		以及以下方法：
		
		- `getPC()` - 返回一个带有当前程序计数器的数字
		- `getGas()` - 返回一个带有剩余气体量的数字
		- `getCost()` - 以数字形式返回操作码的成本
		- `getDepth()` - 以数字形式返回执行深度
		- `getRefund()` - 以数字形式返回要退还的金额
		- `getError()` - 如果发生错误，则返回有关错误的信息，否则返回 `undefined`
	
		如果 error 非空，则应忽略所有其他字段。
	
		为提高效率，`log` 在每个执行步骤中重复使用相同的对象，并使用当前值进行更新；确保复制要在当前调用之外保留的值。例如，这个 step 函数将不起作用：
	
			function(log) {
			  this.logs.append(log);
			}
		但是这个步骤函数将：
	
			function(log) {
			  this.logs.append({gas: log.getGas(), pc: log.getPC(), ...});
			}
			
		- `log.op` 有以下方法：
			- `isPush()` - 如果操作码是 PUSHn，则返回 true
			- `toString()` - 返回操作码的字符串表示
			- `toNumber()` - 返回操作码的编号
		- `log.memory` 有以下方法：
			- `slice(start, stop)` - 将指定的内存段作为字节切片返回
			- `getUint(offset)` - 返回给定偏移量的 32 个字节
		- `log.stack` 有以下方法：
			- `peek(idx)` - 从栈顶返回第 idx 个元素（0 是最顶部的元素）作为 big.Int
			- `length()` - 返回栈中元素的数量
		- `log.contract` 有以下方法：
			- `getCaller()` - 返回调用者的地址
			- `getAddress()` - 返回当前合约的地址
			- `getValue()` - 将调用者发送到合约的值作为 big.Int 返回
			- `getInput()` - 返回传递给合约的输入数据

	- db 有以下方法：
		- `getBalance(address)`- 返回big.Int具有指定账户余额的 a
		- `getNonce(address) `- 返回一个带有指定账户随机数的数字
		- `getCode(address)` - 返回带有指定帐户代码的字节切片
		- `getState(address, hash)` - 返回指定账户和指定哈希的状态值
		- `exists(address)` - 如果指定地址存在则返回真
	
		如果 step 函数在任何时候抛出异常或执行非法操作，则不会在任何进一步的 VM 步骤上调用它，并将错误返回给调用者。
- result

	result 是一个函数，它接受两个参数 `ctx` 和`db`，并期望返回一个 JSON 可序列化的值以返回给 RPC 调用者。

	`ctx` 是事务执行的上下文，具有以下字段：

	- `type` - 字符串，两个值之一 `CALL` 和 `CREATE`
	- `from` - 地址，交易发送者
	- `to` - 地址、交易目标
	- `input` - 缓冲区，输入交易数据
	- `gas` - 数量，交易的 gas 预算
	- `value` - big.Int，要转入wei的金额
	- `block` - 号码，区块号码
	- `output` - 缓冲区，从 EVM 返回的值
	- `gasUsed` - 数量，执行交易时使用的 `gas` 量（不包括 txdata 成本）
	- `time` - 字符串，执行运行时
- fault

	`fault` 是有两个参数，一个函数 `log` 和 `db` ，就像 `step` 当这是不报道的操作码的执行过程中发生错误时调用 `step`。该方法 `log.getError()` 具有有关错误的信息。
- Enter & Exit

	`enter` 和 `exit` 分别在进入和退出内部调用时调用。更具体地说，它们在 `CALL` 变体、`CREATE` 变体以及由 a 隐含的传输中被调用 `SELFDESTRUCT`。

	`enter` 将一个 `callFrame` 对象作为参数，它具有以下方法：
	
	- `getType()` - 返回具有调用帧类型的字符串
	- `getFrom()` - 返回调用帧发送者的地址
	- `getTo()` - 返回调用帧目标的地址
	- `getInput()` - 将输入作为缓冲区返回
	- `getGas()` - 返回一个数字，其中包含为框架提供的气体量
	- `getValue()`- `big.Int` 仅在可用时返回要转移的金额，否则返回 `undefined`

	`exit` 接收一个 `frameResult` 具有以下方法的对象：
	
	- `getGasUsed()` - 以数字形式返回整个框架中使用的气体量
	- `getOutput()` - 将输出作为缓冲区  
	- `getError()` - 如果在执行期间发生错误，则返回错误，否则返回 `undefined`
- 用法
	
	请注意，有几个值是 Golang big.Int 对象，而不是 JavaScript 数字或 JS bigint。因此，它们具有与 godocs 中描述的相同的接口。它们默认的 JSON 序列化是作为 Javascript 编号；序列化大数字准确地调用 `.String()` 它们。为方便起见，`big.NewInt(x)` 提供了 uint 并将其转换为 Go BigInt。

	用法示例，仅在每个 CALL 操作码处返回堆栈的顶部元素：

		debug.traceTransaction(txhash, {tracer: '{data: [], fault: function(log) {}, step: function(log) { if(log.op.toString() == "CALL") this.data.push(log.stack.peek(0)); }, result: function() { return this.data; }}'});

### debug_traceCall
该 `debug_traceCall` 方法允许您 `eth_call` 使用父块的最终状态作为基础在给定块执行的上下文中运行。块可以通过哈希或数字指定。它采用与 a 相同的输入对象`eth_call` 。它返回与相同的输出 `debug_traceTransaction`。可以将跟踪器指定为第三个参数，类似于 `debug_traceTransaction`。

- `Object` - 交易调用对象
	- `from: DATA`, 20 Bytes -（可选）发送交易的地址。
	- `to: DATA`, 20 Bytes - 交易指向的地址。
	- `gas：QUANTITY`-提供用于事务执行的气体的（可选）整数。eth_call 消耗零气体，但某些执行可能需要此参数。
	- `gasPrice: QUANTITY`- (可选) 用于每个付费gas 的gasPrice 的整数
	- `value：QUANTITY`-值（可选）整数与此事务中发送
	- `data: DATA` - (可选) 方法签名和编码参数的哈希值。有关详细信息，请参阅 Solidity 文档中的 Ethereum Contract ABI

客户端|方法调用
---|---
go|debug.TraceCall(args ethapi.CallArgs, blockNrOrHash rpc.BlockNumberOrHash, config *TraceConfig) (*ExecutionResult, error)
console |debug.traceCall(object, blockNrOrHash, [options])
RPC|{"method": "debug_traceCall", "params": [object, blockNrOrHash, {}]}

- 例子

	没有特定的呼叫选项：

		> debug.traceCall(null, "0x0")
		{
		  failed: false,
		  gas: 53000,
		  returnValue: "",
		  structLogs: []
		}
	跟踪具有目的地和特定发送者的调用，禁用存储和内存输出（通过 RPC 返回的数据较少）

		debug.traceCall({
			"from": "0xdeadbeef29292929192939494959594933929292",
			"to": "0xde929f939d939d393f939393f93939f393929023",
			"gas": "0x7a120",
			"data": "0xf00d4b5d00000000000000000000000001291230982139282304923482304912923823920000000000000000000000001293123098123928310239129839291010293810"
			},
			"latest", {"disableStorage": true, "disableMemory": true})
- Cur l示例：

		> curl -H "Content-Type: application/json" -X POST  localhost:8545 --data '{"jsonrpc":"2.0","method":"debug_traceCall","params":[null, "pending"],"id":1}'
		{"jsonrpc":"2.0","id":1,"result":{"gas":53000,"failed":false,"returnValue":"","structLogs":[]}}

### debug_verbosity
设置日志记录详细程度上限。将打印级别达到并包括给定级别的日志消息。

单个包和源文件的详细程度可以使用 `debug_vmodule`.

客户端|方法调用
---|---
console |debug.verbosity(level)
RPC|{"method": "debug_vmodule", "params": [number]}

### debug_vmodule
设置日志记录详细模式。

客户端|方法调用
---|---
console |debug.vmodule(string)
RPC|{"method": "debug_vmodule", "params": [string]}

- 例子

	如果要查看来自特定 Go 包（目录）和所有子目录的消息，请使用：

		> debug.vmodule("eth/*=6")
	如果您想将消息限制到特定包（例如 p2p）但排除子目录，请使用：

		> debug.vmodule("p2p=6")
	如果要查看来自特定源文件的日志消息，请使用

		> debug.vmodule("server.go=6")
	您可以组合这些基本模式。如果您想在 eth 下的包（eth/peer.go、eth/downloader/peer.go）中查看 peer.go 的所有输出以及级别 <= 5 的包 p2p 的输出，请使用：

		debug.vmodule("eth/*/peer.go=6,p2p=5")

### debug_writeBlockProfile
将 `goroutine` 阻塞配置文件写入给定文件。

客户端|方法调用
---|---
console |debug.writeBlockProfile(file)
RPC|{"method": "debug_writeBlockProfile", "params": [string]}
### debug_writeMemProfile
将分配配置文件写入给定文件。请注意，分析率不能通过 API 设置，必须使用 `--pprof.memprofilerate` 标志在命令行上设置。

客户端|方法调用
---|---
console |debug.writeMemProfile(file string)
RPC|{"method": "debug_writeBlockProfile", "params": [string]}

## eth 命名空间(重点)
Geth 为标准的 “eth”JSON-RPC 命名空间提供了多种扩展。

- eth_subscribe, eth_unsubscribe
- eth_call
	- Parameters
		1. Object - Transaction call object
		2. Quantity | Tag - Block number or the string latest or pending
		3. Object - State override set
	- Return Values
	- Simple example
	- Override example
- eth_createAccessList
	- Parameters
	- Usage
	- Response

### eth_subscribe, eth_unsubscribe
这些方法通过订阅用于实时事件。有关更多信息，请参[订阅文档](https://geth.ethereum.org/docs/rpc/pubsub)。
### eth_call
立即执行新消息调用，无需在区块链上创建交易。该 `eth_call` 方法可用于查询内部合约状态，执行编码到合约中的验证，甚至可以在不实时运行交易的情况下测试交易的效果。

- 参数
	- 该方法接受 3 个参数：
		- 一个以只读模式执行的未签名事务对象
		- 一个执行调用的块号
		- 一个可选的状态覆盖集，以允许针对修改后的链状态执行调用。
		
	1. `Object` - 交易调用对象

		该交易调用对象是强制性的，包含了所有必要的参数执行只读 EVM 合同法。

		字段|类型|字节|选项|描述
		---|---|---|---|---
		from|Address|20|可选|模拟交易的发送地址。如果没有可用的本地帐户就使用 `0x00..0`，默认则为本地密钥库中的第一个帐户或地址。
		to|Address|20|必选|交易发送到的地址。
		gas|Quantity| <8|可选|代码执行的最大 gas 限额，以避免无限循环。默认为 `2^63` 节点操作员通过 `--rpc.gascap`.
		gasPrice|Quantity|<32|可选|  在执行期间模拟为每单位 gas 支付的数量 `wei`。默认为 `1 gwei`.
		value|Quantity|<32|可选|用 `wei` 来模拟与交易一起发送。默认为 `0`.
		data|Binary|任何|可选|要发送到目标合约的二进制数据。通常是方法签名的 4 字节散列，后跟 ABI 编码参数。有关详细信息，请参阅[以太坊合约 ABI](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI)。
		
		- 例子：
		
				{
				  "from": "0xd9c9cd5f6779558b6e0ed4e6acf6b1947e7fa1f3",
				  "to":   "0xebe8efa441b9302a0d7eaecc277c09d20d684540",
				  "gas":  "0x1bd7c",
				  "data": "0xd459fc46000000000000000000000000000000000000000000000000000000000046c650dbb5e8cb2bac4d2ed0b1e6475d37361157738801c494ca482f96527eb48f9eec488c2eba92d31baeccfb6968fad5c21a3df93181b43b4cf253b4d572b64172ef000000000000000000000000000000000000000000000000000000000000008c00000000000000000000000000000000000000000000000000000000000000e0000000000000000000000000000000000000000000000000000000000000014000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000001c000000000000000000000000000000000000000000000000000000000000001c0000000000000000000000000000000000000000000000000000000000000002b85c0c828d7a98633b4e1b65eac0c017502da909420aeade9a280675013df36bdc71cffdf420cef3d24ba4b3f9b980bfbb26bd5e2dcf7795b3519a3fd22ffbb2000000000000000000000000000000000000000000000000000000000000000238fb6606dc2b5e42d00c653372c153da8560de77bd9afaba94b4ab6e4aa11d565d858c761320dbf23a94018d843772349bd9d92301b0ca9ca983a22d86a70628",
				}
	2. `Quantity | Tag` - 块号或字符串 `latest` 或 `pending`

		块号是强制性的，并且限定针对其指定的事务应该被执行的上下文（状态）。无法对重组块执行调用或早于 128 的块（除非该节点是存档节点）。
	3. `Object` - 状态覆盖集

		状态覆写集是一个可选的地址到状态映射，其中每个条目指定一些状态之前执行呼叫被短暂地覆盖。每个地址映射到一个对象，其中包含：

		字段|类型|字节|选项|描述
		---|---|---|---|---
		balance|Quantity|	<32|可选|在执行呼叫之前为帐户设置假余额。
		nonce|Quantity|<8|可选|在执行调用之前为帐户设置假随机数。
		code|Binary|	任何|可选|在执行调用之前将伪造的 EVM 字节码注入到帐户中。
		state|Object|	任何|可选|在执行调用之前，伪造键值映射以覆盖帐户存储中的所有插槽。
		stateDiff	|Object|任何|可选|	在执行调用之前，伪造键值映射以覆盖帐户存储中的各个插槽。

		状态覆盖集的目标是多方面的：

		- DApps 可以使用它来减少需要部署在链上的合约代码量。简单地返回内部状态或执行预定义验证的代码可以保持在链外并按需提供给节点。
		- 通过使用自定义方法扩展部署在链上的代码并调用它们，它可以用于智能合约分析。这避免了必须在沙箱中下载和重建整个状态来运行自定义代码。
		- 它可用于通过有选择地覆盖某些代码或状态并查看执行如何变化来调试已部署的大型合约套件中的智能合约。可能需要专门的工具。

		- 例子：

				{
				  "0xd9c9cd5f6779558b6e0ed4e6acf6b1947e7fa1f3": {
				    "balance": "0xde0b6b3a7640000"
				  },
				  "0xebe8efa441b9302a0d7eaecc277c09d20d684540": {
				    "code": "0x...",
				    "state": {
				      ""
				    }
				  }
				}
- 返回值

	该方法返回一个 `Binary` 由执行的合约调用的返回单个值组成的。
- 简单的例子

	通过在 `localhost` ( `geth --rinkeby --http`) 上公开 RPC 的同步 Rinkeby 节点，我们可以调用 [Checkpoint Oracle](https://rinkeby.etherscan.io/address/0xebe8efa441b9302a0d7eaecc277c09d20d684540) 以检索管理员列表：

		$ curl --data '{"method":"eth_call","params" [{"to":"0xebe8efa441b9302a0d7eaecc277c09d20d684540","data":"0x45848dfc"},"latest"],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
	结果是一个以太坊 ABI 编码的账户列表：

		{
		  "id":      1,
		  "jsonrpc": "2.0",
		  "result":  "0x00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000004000000000000000000000000d9c9cd5f6779558b6e0ed4e6acf6b1947e7fa1f300000000000000000000000078d1ad571a1a09d60d9bbf25894b44e4c8859595000000000000000000000000286834935f4a8cfb4ff4c77d5770c2775ae2b0e7000000000000000000000000b86e2b0ab5a4b1373e40c51a7c712c70ba2f9f8e"
		}
	为了完整起见，解码后的响应是：

		0xd9c9cd5f6779558b6e0ed4e6acf6b1947e7fa1f3,
		0x78d1ad571a1a09d60d9bbf25894b44e4c8859595,
		0x286834935f4a8cfb4ff4c77d5770c2775ae2b0e7,
		0xb86e2b0ab5a4b1373e40c51a7c712c70ba2f9f8e
- 覆盖的示例

	上面的简单示例展示了如何调用链上智能合约已经公开的方法。如果我们想访问一些它没有公开的数据怎么办？

	我们可以用一个保留相同字段（以保留相同的存储布局）但包含不同方法集的合约来删除原始检查点 [oracle](https://github.com/ethereum/go-ethereum/blob/master/contracts/checkpointoracle/contract/oracle.sol) 合约：

		pragma solidity ^0.5.10;
		
		contract CheckpointOracle {
		    mapping(address => bool) admins;
		    address[] adminList;
		    uint64 sectionIndex;
		    uint height;
		    bytes32 hash;
		    uint sectionSize;
		    uint processConfirms;
		    uint threshold;
		
		    function VotingThreshold() public view returns (uint) {
		        return threshold;
		    }
		}
	通过在 localhost ( `geth --rinkeby --http` )上公开 RPC 的同步 Rinkeby 节点，我们可以对实时 [Checkpoint Oracle](https://rinkeby.etherscan.io/address/0xebe8efa441b9302a0d7eaecc277c09d20d684540) 进行调用，但使用我们自己的版本覆盖其字节码，该版本具有投票阈值字段的访问器：

		$ curl --data '{"method":"eth_call","params":[{"to":"0xebe8efa441b9302a0d7eaecc277c09d20d684540","data":"0x0be5b6ba"}, "latest", {"0xebe8efa441b9302a0d7eaecc277c09d20d684540": {"code":"0x6080604052348015600f57600080fd5b506004361060285760003560e01c80630be5b6ba14602d575b600080fd5b60336045565b60408051918252519081900360200190f35b6007549056fea265627a7a723058206f26bd0433456354d8d1228d8fe524678a8aeeb0594851395bdbd35efc2a65f164736f6c634300050a0032"}}],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
	结果是以太坊 ABI 编码的阈值数：

		{
		  "id":      1,
		  "jsonrpc": "2.0",
		  "result":  "0x0000000000000000000000000000000000000000000000000000000000000002"
		}
	只是为了完整起见，解码后的响应是：2。

### eth_createAccessList
该方法创建一个 [EIP2930](https://eips.ethereum.org/EIPS/eip-2930) 类型 `accessList` 基于给定的 `Transaction` 。在 `accessList` 包含所有存储插槽和地址读取和写入的事务，除了发送者帐户和预编译。此方法使用与相同的 `transaction` 调用对象和 `blockNumberOrTag` 对象 `eth_call` 。`AnaccessList` 可用于解除因 `gas` 成本增加而变得无法访问的合同。

- 参数

	字段|类型|描述
	---|---|---
	transaction|Object|TransactionCall 目的
	blockNumberOrTag|Object|可选，blocknumber 或latest或pending
- 用法

		curl --data '{"method":"eth_createAccessList","params":[{"from": "0x8cd02c6cbd8375b39b06577f8d50c51d86e8d5cd", "data": "0x608060806080608155"}, "pending"],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
- 返回

	该方法 `eth_createAccessList` 返回交易使用的地址和存储密钥列表以及添加访问列表时消耗的气体。也就是说它为您提供了该交易将使用的地址和存储密钥列表以及如果包含访问列表则消耗的 gas。
	
	就像 `eth_estimateGas`，这是一个估计；当实际开采交易时该列表可能会发生变化。`accessList` 与没有访问列表的交易相比，向您的交易添加 a 不一定会导致较低的 gas 使用量。

	- 例子：
	
			{
			  "accessList": [
			    {
			      "address": "0xa02457e5dfd32bda5fc7e1f1b008aa5979568150",
			      "storageKeys": [
			        "0x0000000000000000000000000000000000000000000000000000000000000081",
			      ]
			    }
			  ]
			  "gasUsed": "0x125f8"
			}
			

## les 命名空间
该 `les` API 允许你管理 LES 服务器设置，包括客户端参数和优先级客户的付款设置。它还提供了在服务器和客户端模式下查询检查点信息的功能。

- les_serverInfo
	- Example
- les_clientInfo
	- Example
- les_priorityClientInfo
	- Example
- les_addBalance
	- Example
- les_setClientParams
	- Example
- les_setDefaultParams
	- Example
- les_latestCheckpoint
	- Example
- les_getCheckpoint
	- Example
- les_getCheckpointContractAddress
	- Example

### les_serverInfo
获取有关当前连接和总/单个允许连接容量的信息。

客户端|方法调用
---|---
go|les.ServerInfo() map[string]interface{}
console|les.serverInfo()
RPC|{"method": "les_serverInfo", "params": []}

- 例子

		> les.serverInfo
		{
		  freeClientCapacity: 16000,
		  maximumCapacity: 1600000,
		  minimumCapacity: 16000,
		  priorityConnectedCapacity: 180000,
		  totalCapacity: 1600000,
		  totalConnectedCapacity: 180000
		}

### les_clientInfo
如果 ID 列表为空，则获取指定客户端列表或所有已连接客户端的单个客户端信息（连接、余额、定价）。

客户端|方法调用
---|---
go|les.ClientInfo(ids []enode.ID) map[enode.ID]map[string]interface{}
console|les.clientInfo([id, ...])
RPC|{"method": "les_clientInfo", "params": [[id, ...]]}

- 例子

		> les.clientInfo([])
		{
		  37078bf8ea160a2b3d129bb4f3a930ce002356f83b820f467a07c1fe291531ea: {
		    capacity: 16000,
		    connectionTime: 11225.335901136,
		    isConnected: true,
		    pricing/balance: 998266395881,
		    pricing/balanceMeta: "",
		    pricing/negBalance: 501657912857,
		    priority: true
		  },
		  6a47fe7bb23fd335df52ef1690f37ab44265a537b1d18eb616a3e77f898d9e77: {
		    capacity: 100000,
		    connectionTime: 9874.839293082,
		    isConnected: true,
		    pricing/balance: 2908840710198,
		    pricing/balanceMeta: "qwerty",
		    pricing/negBalance: 206242704507,
		    priority: true
		  },
		  740c78f7d914e5c763731bc751b513fc2388ffa0b47db080ded3e8b305e68c75: {
		    capacity: 16000,
		    connectionTime: 3089.286712188,
		    isConnected: true,
		    pricing/balance: 998266400174,
		    pricing/balanceMeta: "",
		    pricing/negBalance: 55135348863,
		    priority: true
		  },
		  9985ade55b515f79f64274bf2ae440ca8c433cfb0f283fb6010bf46f796b2a3b: {
		    capacity: 16000,
		    connectionTime: 11479.335479545,
		    isConnected: true,
		    pricing/balance: 998266452203,
		    pricing/balanceMeta: "",
		    pricing/negBalance: 564116425655,
		    priority: true
		  },
		  ce65ada2c3e17d6da00cec0b3cc4c8ed5e74428b60f42fa287eaaec8cca62544: {
		    capacity: 16000,
		    connectionTime: 7095.794385419,
		    isConnected: true,
		    pricing/balance: 998266448492,
		    pricing/balanceMeta: "",
		    pricing/negBalance: 214617753229,
		    priority: true
		  },
		  e1495ceb6db842f3ee66428d4bb7f4a124b2b17111dae35d141c3d568b869ef1: {
		    capacity: 16000,
		    connectionTime: 8614.018237937,
		    isConnected: true,
		    pricing/balance: 998266391796,
		    pricing/balanceMeta: "",
		    pricing/negBalance: 185964891797,
		    priority: true
		  }
		}
### les_priorityClientInfo
获取指定 ID 范围内余额为正的客户的个别客户信息，`start` 包括、`stop` 排除。如果 `stop` 为零，则返回结果直到最后一个现有余额条目。`maxCount` 限制返回结果的数量。如果达到计数限制但范围内还有更多 ID，则结果中会包含第一个缺失的 ID，并为其分配一个空值。

客户端|方法调用
---|---
go|les.PriorityClientInfo(start, stop enode.ID, maxCount int) map[enode.ID]map[string]interface{}
console | les.priorityClientInfo(id, id, number)
RPC|{"method": "les_priorityClientInfo", "params": [id, id, number]}

- 例子
	- 2 条
		
			> les.priorityClientInfo("0x4000000000000000000000000000000000000000000000000000000000000000", "0xe000000000000000000000000000000000000000000000000000000000000000", 2)
			{
			  6a47fe7bb23fd335df52ef1690f37ab44265a537b1d18eb616a3e77f898d9e77: {
			    capacity: 100000,
			    connectionTime: 9842.11178361,
			    isConnected: true,
			    pricing/balance: 2912113588853,
			    pricing/balanceMeta: "qwerty",
			    pricing/negBalance: 206242704507,
			    priority: true
			  },
			  740c78f7d914e5c763731bc751b513fc2388ffa0b47db080ded3e8b305e68c75: {
			    capacity: 16000,
			    connectionTime: 3056.559199029,
			    isConnected: true,
			    pricing/balance: 998790060237,
			    pricing/balanceMeta: "",
			    pricing/negBalance: 55135348863,
			    priority: true
			  },
			  9985ade55b515f79f64274bf2ae440ca8c433cfb0f283fb6010bf46f796b2a3b: {}
			}
	- 100 条

			> les.priorityClientInfo("0x0000000000000000000000000000000000000000000000000000000000000000", "0x0000000000000000000000000000000000000000000000000000000000000000", 100)
			{
			  37078bf8ea160a2b3d129bb4f3a930ce002356f83b820f467a07c1fe291531ea: {
			    capacity: 16000,
			    connectionTime: 11128.247204027,
			    isConnected: true,
			    pricing/balance: 999819815030,
			    pricing/balanceMeta: "",
			    pricing/negBalance: 501657912857,
			    priority: true
			  },
			  6a47fe7bb23fd335df52ef1690f37ab44265a537b1d18eb616a3e77f898d9e77: {
			    capacity: 100000,
			    connectionTime: 9777.750592047,
			    isConnected: true,
			    pricing/balance: 2918549830576,
			    pricing/balanceMeta: "qwerty",
			    pricing/negBalance: 206242704507,
			    priority: true
			  },
			  740c78f7d914e5c763731bc751b513fc2388ffa0b47db080ded3e8b305e68c75: {
			    capacity: 16000,
			    connectionTime: 2992.198001116,
			    isConnected: true,
			    pricing/balance: 999819845102,
			    pricing/balanceMeta: "",
			    pricing/negBalance: 55135348863,
			    priority: true
			  },
			  9985ade55b515f79f64274bf2ae440ca8c433cfb0f283fb6010bf46f796b2a3b: {
			    capacity: 16000,
			    connectionTime: 11382.246766963,
			    isConnected: true,
			    pricing/balance: 999819871598,
			    pricing/balanceMeta: "",
			    pricing/negBalance: 564116425655,
			    priority: true
			  },
			  ce65ada2c3e17d6da00cec0b3cc4c8ed5e74428b60f42fa287eaaec8cca62544: {
			    capacity: 16000,
			    connectionTime: 6998.705683407,
			    isConnected: true,
			    pricing/balance: 999819882177,
			    pricing/balanceMeta: "",
			    pricing/negBalance: 214617753229,
			    priority: true
			  },
			  e1495ceb6db842f3ee66428d4bb7f4a124b2b17111dae35d141c3d568b869ef1: {
			    capacity: 16000,
			    connectionTime: 8516.929533901,
			    isConnected: true,
			    pricing/balance: 999819891640,
			    pricing/balanceMeta: "",
			    pricing/negBalance: 185964891797,
			    priority: true
			  }
			}
		
				
### les_addBalance
将签名值添加到指定客户端的令牌余额并更新其 meta 标签。余额不能低于零或高于 `2^^63-1`。返回更新前后的余额值。该 `meta` 标签可用于存储序列号或对上次处理的传入付款的引用、令牌到期信息、其他货币的余额或任何特定于应用程序的附加信息。

客户端|方法调用
---|---
go|les.AddBalance(id enode.ID, value int64, meta string) ([2]uint64, error)}
console |les.addBalance(id, number, string)
RPC|{"method": "les_addBalance", "params": [id, number, string]}

- 例子

		> les.addBalance("0x6a47fe7bb23fd335df52ef1690f37ab44265a537b1d18eb616a3e77f898d9e77", 1000000000, "qwerty")
		[968379616, 1968379616]

### les_setClientParams
如果 ID 列表为空，则为指定的已连​​接客户端列表或所有已连接客户端设置容量和定价因素。

客户端|方法调用
---|---
go|les.SetClientParams(ids []enode.ID, params map[string]interface{}) error
console |les.setClientParams([id, ...], {string: value, ...})
RPC|{"method": "les_setClientParams", "params": [[id, ...], {string: value, ...}]}

- 例子

		> les.setClientParams(["0x6a47fe7bb23fd335df52ef1690f37ab44265a537b1d18eb616a3e77f898d9e77"], {
			"capacity": 100000,
			"pricing/timeFactor": 0,
			"pricing/capacityFactor": 1000000000,
			"pricing/requestCostFactor": 1000000000,
			"pricing/negative/timeFactor": 0,
			"pricing/negative/capacityFactor": 1000000000,
			"pricing/negative/requestCostFactor": 1000000000,
		})
		null
### les_setDefaultParams
为后续连接的客户端设置默认定价因素。

客户端|方法调用
---|---
go|lles.SetDefaultParams(params map[string]interface{}) error
console| les.setDefaultParams({string: value, ...})
RPC|{"method": "les_setDefaultParams", "params": [{string: value, ...}]}

- 例子

		> les.setDefaultParams({
			"pricing/timeFactor": 0,
			"pricing/capacityFactor": 1000000000,
			"pricing/requestCostFactor": 1000000000,
			"pricing/negative/timeFactor": 0,
			"pricing/negative/capacityFactor": 1000000000,
			"pricing/negative/requestCostFactor": 1000000000,
		})
		null

### the_latestCheckpoint
获取最新已知检查点的索引和哈希值。

客户端|方法调用
---|---
go|les.LatestCheckpoint() ([4]string, error)
console | les.latestCheckpoint()
RPC|{"method": "les_latestCheckpoint", "params": []}

- 例子

		> les.latestCheckpoint
		["0x110", "0x6eedf8142d06730b391bfcbd32e9bbc369ab0b46ae226287ed5b29505a376164", "0x191bb2265a69c30201a616ae0d65a4ceb5937c2f0c94b125ff55343d707463e5", "0xf58409088a5cb2425350a59d854d546d37b1e7bef8bbf6afee7fd15f943d626a"]

### les_getCheckpoint
按索引获取检查点哈希。

客户端|方法调用
---|---
go|les.GetCheckpoint(index uint64) ([3]string, error)
console |les.getCheckpoint(number)
RPC|{"method": "les_getCheckpoint", "params": [number]}

- 例子

		> les.getCheckpoint(256)
		["0x93eb4af0b224b1097e09181c2e51536fe0a3bf3bb4d93e9a69cab9eb3e28c75f", "0x0eb055e384cf58bc72ca20ca5e2b37d8d4115dce80ab4a19b72b776502c4dd5b", "0xda6c02f7c51f9ecc3eca71331a7eaad724e5a0f4f906ce9251a2f59e3115dd6a"]

### les_getCheckpointContractAddress
获取检查点预言机合约的地址。

客户端|方法调用
---|---
go|les.GetCheckpointContractAddress() (string, error)
console |les.checkpointContractAddress()
RPC|{"method": "les_getCheckpointContractAddress", "params": []}

- 例子

		> les.checkpointContractAddress
		"0x9a9070028361F7AAbeB3f2F2Dc07F82C4a98A02a"

## 矿工命名空间
该 miner API 允许您远程控制节点的挖矿操作并设置各种挖矿特定设置。

- `miner_getHashrate`
- `miner_setExtra`
- `miner_setGasPrice`
- `miner_start`
- `miner_stop`
- `miner_setEtherbase`
- `miner_setGasLimit`

### `miner_getHashrate`
获取以 H/s 为单位的哈希率（每秒哈希操作数）。

客户端|方法调用
---|---
console |miner.getHashrate()
RPC|{"method": "miner_getHashrate", "params": []}

### miner_setExtra
设置矿工阻止时矿工可以包含的额外数据。这上限为 32 个字节。

客户端|方法调用
---|---
go|miner.setExtra(extra string) (bool, error)
console |miner.setExtra(string)
RPC|{"method": "miner_setExtra", "params": [string]}

### miner_setGasPrice
设置挖矿交易时可接受的最低天然气价格。任何低于此限制的交易都将被排除在挖矿过程之外。

客户端|方法调用
---|---
go|miner.setGasPrice(number *rpc.HexNumber) bool
console |miner.setGasPrice(number)
RPC|{"method": "miner_setGasPrice", "params": [number]}

### miner_start
使用给定的线程数启动 CPU 挖掘过程，并在需要时生成新的 DAG。

客户端|方法调用
---|---
go|miner.Start(threads *rpc.HexNumber) (bool, error)
console | miner.start(number)
RPC| {"method": "miner_start", "params": [number]}

### miner_stop
停止 CPU 挖矿操作。

客户端|方法调用
---|---
go|miner.Stop() bool
console |miner.stop()
RPC|{"method": "miner_stop", "params": []}

### miner_setEtherbase
设置以太坊，挖矿奖励将流向何处。

客户端|方法调用
---|---
go|miner.SetEtherbase(common.Address) bool
console | miner.setEtherbase(address)
RPC|{"method": "miner_setEtherbase", "params": [address]}

### miner_setGasLimit
设置矿工在挖矿时将针对的gas限制。注意：在激活 EIP-1559 的网络上，这应该设置为您想要的气体目标（即每个块平均使用的有效气体）的两倍。

客户端|方法调用
---|---
go|miner.SetGasLimit(number *rpc.HexNumber) bool
console |miner.SetGasLimit(number)
RPC|{"method": "miner_setGasLimit", "params": [number]}


## 个人命名空间
个人 API 管理密钥库中的私钥。

- `personal_importRawKey`
- `personal_listAccounts`
- `personal_lockAccount`
- `personal_newAccount`
- `personal_unlockAccount`
- `personal_sendTransaction`
- `personal_sign`
- `personal_ecRecover`

### Personal_importRawKey
将给定的未加密私钥（十六进制字符串）导入密钥库，并使用密码对其进行加密。

返回新帐户的地址。

客户端|方法调用
---|---
console | personal.importRawKey(keydata, passphrase)
RPC|{"method": "personal_importRawKey", "params": [string, string]}

### Personal_listAccounts
返回密钥库中所有密钥的所有以太坊账户地址。

客户端|方法调用
---|---
console |personal.listAccounts
RPC|{"method": "personal_listAccounts", "params": []}

- 例子

		> personal.listAccounts
		["0x5e97870f263700f46aa00d967821199b9bc5a120", "0x3d80b31a78c30fc628f20b2c89d7ddbf6e53cedc"]

### Personal_lockAccount
从内存中删除具有给定地址的私钥。该帐户不能再用于发送交易。

客户端|方法调用
---|---
console |personal.lockAccount(address)
RPC| {"method": "personal_lockAccount", "params": [string]}

### Personal_newAccount
生成新的私钥并将其存储在密钥存储目录中。密钥文件使用给定的密码加密。返回新帐户的地址。

在 geth 控制台上，newAccount 当它没有作为参数提供时，将提示输入密码。

客户端|方法调用
---|---
console |personal.newAccount()
RPC|{"method": "personal_newAccount", "params": [string]}

- 例子

		> personal.newAccount()
		Passphrase: 
		Repeat passphrase: 
		"0x5e97870f263700f46aa00d967821199b9bc5a120"

	密码短语也可以作为字符串提供。

		> personal.newAccount("h4ck3r")
		"0x3d80b31a78c30fc628f20b2c89d7ddbf6e53cedc"

### Personal_unlockAccount
使用密钥库中的给定地址解密密钥。使用 JavaScript 控制台时，密码短语和解锁持续时间都是可选的。如果密码不作为参数提供，控制台将交互提示输入密码。

未加密的密钥将保存在内存中，直到解锁持续时间到期。如果解锁时长默认为 300 秒。零秒的明确持续时间将解锁密钥，直到 geth 退出。

该账户可以使用 `eth_sign`，并 `eth_sendTransaction` 同时被解锁。

客户端|方法调用
---|---
console |personal.unlockAccount(address, passphrase, duration)
RPC|{"method": "personal_unlockAccount", "params": [string, string, number]}

- 例子

		> personal.unlockAccount("0x5e97870f263700f46aa00d967821199b9bc5a120")
		Unlock account 0x5e97870f263700f46aa00d967821199b9bc5a120
		Passphrase: 
		true
	提供密码短语和解锁持续时间作为参数：

		> personal.unlockAccount("0x5e97870f263700f46aa00d967821199b9bc5a120", "foo", 30)
		true
	如果您想输入密码并仍然覆盖默认解锁持续时间，请 `null` 作为密码传递。

		> personal.unlockAccount("0x5e97870f263700f46aa00d967821199b9bc5a120", null, 30)
		Unlock account 0x5e97870f263700f46aa00d967821199b9bc5a120
		Passphrase: 
		true

### personal_sendTransaction
验证给定的密码并提交交易。交易与 `for` 的参数相同，`eth_sendTransaction` 并包含 `from` 地址。如果密码可用于解密记录到 `tx.from` 交易的私钥，则验证、签名并发送到网络上。该账户在节点内未全局解锁，不能用于其他 RPC 调用。

客户端|方法调用
---|---
console|personal.sendTransaction(tx, passphrase)
RPC|{"method": "personal_sendTransaction", "params": [tx, string]}

请注意，在 Geth 1.5 之前，请使用它 `personal_signAndSendTransaction` 作为最初的介绍性名称，后来才重命名为当前的最终版本。

- 例子

		> var tx = {from: "0x391694e7e0b0cce554cb130d723a9d27458f9298", to: "0xafa3f8684e54059998bc3a7b0d2b0da075154d66", value: web3.toWei(1.23, "ether")}
		undefined
		> personal.sendTransaction(tx, "passphrase")
		0x8474441674cdd47b35b875fd1a530b800b51a5264b9975fb21129eeb8c18582f

### personal_sign
sign 方法计算以太坊特定的签名： `sign(keccack256("\x19Ethereum Signed Message:\n" + len(message) + message)))`。

通过向消息添加前缀，可以将计算出的签名识别为以太坊特定的签名。这可以防止恶意 DApp 可以签署任意数据（例如交易）并使用签名来冒充受害者的滥用。

请参阅 ecRecover 以验证签名。

客户端|方法调用
---|---
console|personal.sign(message, account, [password])
RPC|{"method": "personal_sign", "params": [message, account, password]}

- 例子

		> personal.sign("0xdeadbeaf", "0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", "")
		"0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"

### Personal_ecRecover
`ecRecover` 返回与用于计算签名的私钥关联的地址 `personal_sign`。

客户端|方法调用
---|---
console|personal.ecRecover(message, signature)
RPC|{"method": "personal_ecRecover", "params": [message, signature]}

- 例子

		> personal.sign("0xdeadbeaf", "0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", "")
		"0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
		> personal.ecRecover("0xdeadbeaf", "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b")
		"0x9b2055d370f73ec7d8a03e965129118dc8f5bf83"

## txpool 命名空间
该 txpoolAPI 使您可以访问多种非标准 RPC 方法，以检查包含所有当前挂起的事务以及排队等待将来处理的事务的事务池的内容。

- `txpool_content`
- `txpool_inspect`
- `txpool_status`

###  txpool_content
该 `content` 检查性可以查询列出所有交易的具体细节目前尚待列入下一个块（一个或多个），以及正在计划在未来只有执行的人。

结果是一个包含两个字段 `pending` (等待)和 `queued` (排队)的对象 。这些字段中的每一个都是关联数组，其中每个条目将一个源地址映射到一批预定的事务。这些批次本身是将随机数与实际交易相关联的映射。

请注意，可能有多个交易与同一个帐户和随机数相关联。如果用户广播多个具有不同 gas 限额（甚至完全不同的交易）的消息，就会发生这种情况。

客户端|方法调用
---|---
go|txpool.Content() (map[string]map[string]map[string]*RPCTransaction)
console | txpool.content
RPC|{"method": "txpool_content"}

- 例子

		> txpool.content
		{
		  pending: {
		    0x0216d5032f356960cd3749c31ab34eeff21b3395: {
		      806: {
		        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
		        blockNumber: null,
		        from: "0x0216d5032f356960cd3749c31ab34eeff21b3395",
		        gas: "0x5208",
		        gasPrice: "0xba43b7400",
		        hash: "0xaf953a2d01f55cfe080c0c94150a60105e8ac3d51153058a1f03dd239dd08586",
		        input: "0x",
		        nonce: "0x326",
		        to: "0x7f69a91a3cf4be60020fb58b893b7cbb65376db8",
		        transactionIndex: null,
		        value: "0x19a99f0cf456000"
		      }
		    },
		    0x24d407e5a0b506e1cb2fae163100b5de01f5193c: {
		      34: {
		        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
		        blockNumber: null,
		        from: "0x24d407e5a0b506e1cb2fae163100b5de01f5193c",
		        gas: "0x44c72",
		        gasPrice: "0x4a817c800",
		        hash: "0xb5b8b853af32226755a65ba0602f7ed0e8be2211516153b75e9ed640a7d359fe",
		        input: "0xb61d27f600000000000000000000000024d407e5a0b506e1cb2fae163100b5de01f5193c00000000000000000000000000000000000000000000000053444835ec580000000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
		        nonce: "0x22",
		        to: "0x7320785200f74861b69c49e4ab32399a71b34f1a",
		        transactionIndex: null,
		        value: "0x0"
		      }
		    }
		  },
		  queued: {
		    0x976a3fc5d6f7d259ebfb4cc2ae75115475e9867c: {
		      3: {
		        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
		        blockNumber: null,
		        from: "0x976a3fc5d6f7d259ebfb4cc2ae75115475e9867c",
		        gas: "0x15f90",
		        gasPrice: "0x4a817c800",
		        hash: "0x57b30c59fc39a50e1cba90e3099286dfa5aaf60294a629240b5bbec6e2e66576",
		        input: "0x",
		        nonce: "0x3",
		        to: "0x346fb27de7e7370008f5da379f74dd49f5f2f80f",
		        transactionIndex: null,
		        value: "0x1f161421c8e0000"
		      }
		    },
		    0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a: {
		      2: {
		        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
		        blockNumber: null,
		        from: "0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a",
		        gas: "0x15f90",
		        gasPrice: "0xba43b7400",
		        hash: "0x3a3c0698552eec2455ed3190eac3996feccc806970a4a056106deaf6ceb1e5e3",
		        input: "0x",
		        nonce: "0x2",
		        to: "0x24a461f25ee6a318bdef7f33de634a67bb67ac9d",
		        transactionIndex: null,
		        value: "0xebec21ee1da40000"
		      },
		      6: {
		        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
		        blockNumber: null,
		        from: "0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a",
		        gas: "0x15f90",
		        gasPrice: "0x4a817c800",
		        hash: "0xbbcd1e45eae3b859203a04be7d6e1d7b03b222ec1d66dfcc8011dd39794b147e",
		        input: "0x",
		        nonce: "0x6",
		        to: "0x6368f3f8c2b42435d6c136757382e4a59436a681",
		        transactionIndex: null,
		        value: "0xf9a951af55470000"
		      }
		    }
		  }
		}

### txpool_inspect
该 inspect 检查性可以查询列出所有交易的文本摘要目前正在等待列入下一个块（一个或多个），以及正在计划在未来只有执行的人。这是一种专门为开发人员量身定制的方法，可以快速查看池中的交易并发现任何潜在问题。

结果是一个包含两个字段  `pending` 和 `queued` 的对象。这些字段中的每一个都是关联数组，其中每个条目将一个源地址映射到一批预定的事务。这些批次本身是将随机数与交易摘要字符串相关联的映射。

请注意，可能有多个交易与同一个帐户和随机数相关联。如果用户广播多个具有不同 gas 限额（甚至完全不同的交易）的消息，就会发生这种情况。

客户端|方法调用
---|---
go|txpool.Inspect() (map[string]map[string]map[string]string)
console |txpool.inspect
RPC|{"method": "txpool_inspect"}

- 例子

		> txpool.inspect
		{
		  pending: {
		    0x26588a9301b0428d95e6fc3a5024fce8bec12d51: {
		      31813: "0x3375ee30428b2a71c428afa5e89e427905f95f7e: 0 wei + 500000 × 20000000000 wei"
		    },
		    0x2a65aca4d5fc5b5c859090a6c34d164135398226: {
		      563662: "0x958c1fa64b34db746925c6f8a3dd81128e40355e: 1051546810000000000 wei + 90000 gas × 20000000000 wei",
		      563663: "0x77517b1491a0299a44d668473411676f94e97e34: 1051190740000000000 wei + 90000 gas × 20000000000 wei",
		      563664: "0x3e2a7fe169c8f8eee251bb00d9fb6d304ce07d3a: 1050828950000000000 wei + 90000 gas × 20000000000 wei",
		      563665: "0xaf6c4695da477f8c663ea2d8b768ad82cb6a8522: 1050544770000000000 wei + 90000 gas × 20000000000 wei",
		      563666: "0x139b148094c50f4d20b01caf21b85edb711574db: 1048598530000000000 wei + 90000 gas × 20000000000 wei",
		      563667: "0x48b3bd66770b0d1eecefce090dafee36257538ae: 1048367260000000000 wei + 90000 gas × 20000000000 wei",
		      563668: "0x468569500925d53e06dd0993014ad166fd7dd381: 1048126690000000000 wei + 90000 gas × 20000000000 wei",
		      563669: "0x3dcb4c90477a4b8ff7190b79b524773cbe3be661: 1047965690000000000 wei + 90000 gas × 20000000000 wei",
		      563670: "0x6dfef5bc94b031407ffe71ae8076ca0fbf190963: 1047859050000000000 wei + 90000 gas × 20000000000 wei"
		    },
		    0x9174e688d7de157c5c0583df424eaab2676ac162: {
		      3: "0xbb9bc244d798123fde783fcc1c72d3bb8c189413: 30000000000000000000 wei + 85000 gas × 21000000000 wei"
		    },
		    0xb18f9d01323e150096650ab989cfecd39d757aec: {
		      777: "0xcd79c72690750f079ae6ab6ccd7e7aedc03c7720: 0 wei + 1000000 gas × 20000000000 wei"
		    },
		    0xb2916c870cf66967b6510b76c07e9d13a5d23514: {
		      2: "0x576f25199d60982a8f31a8dff4da8acb982e6aba: 26000000000000000000 wei + 90000 gas × 20000000000 wei"
		    },
		    0xbc0ca4f217e052753614d6b019948824d0d8688b: {
		      0: "0x2910543af39aba0cd09dbb2d50200b3e800a63d2: 1000000000000000000 wei + 50000 gas × 1171602790622 wei"
		    },
		    0xea674fdde714fd979de3edf0f56aa9716b898ec8: {
		      70148: "0xe39c55ead9f997f7fa20ebe40fb4649943d7db66: 1000767667434026200 wei + 90000 gas × 20000000000 wei"
		    }
		  },
		  queued: {
		    0x0f6000de1578619320aba5e392706b131fb1de6f: {
		      6: "0x8383534d0bcd0186d326c993031311c0ac0d9b2d: 9000000000000000000 wei + 21000 gas × 20000000000 wei"
		    },
		    0x5b30608c678e1ac464a8994c3b33e5cdf3497112: {
		      6: "0x9773547e27f8303c87089dc42d9288aa2b9d8f06: 50000000000000000000 wei + 90000 gas × 50000000000 wei"
		    },
		    0x976a3fc5d6f7d259ebfb4cc2ae75115475e9867c: {
		      3: "0x346fb27de7e7370008f5da379f74dd49f5f2f80f: 140000000000000000 wei + 90000 gas × 20000000000 wei"
		    },
		    0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a: {
		      2: "0x24a461f25ee6a318bdef7f33de634a67bb67ac9d: 17000000000000000000 wei + 90000 gas × 50000000000 wei",
		      6: "0x6368f3f8c2b42435d6c136757382e4a59436a681: 17990000000000000000 wei + 90000 gas × 20000000000 wei",
		      7: "0x6368f3f8c2b42435d6c136757382e4a59436a681: 17900000000000000000 wei + 90000 gas × 20000000000 wei"
		    }
		  }
		}

### txpool_status
该 status 检查性，可以被询问的交易目前正等待列入下一个块（一个或多个）的数量，以及正在计划在未来只有执行的人。

结果是一个具有两个字段 `pending` 和 `queued` 的对象，每个字段都是一个计数器，表示该特定状态下的交易数量。

客户端|方法调用
---|---
go|txpool.Status() (map[string]*rpc.HexNumber)
console | txpool.status
RPC|{"method": "txpool_status"}

- 例子

		> txpool.status
		{
		  pending: 10,
		  queued: 7
		}				