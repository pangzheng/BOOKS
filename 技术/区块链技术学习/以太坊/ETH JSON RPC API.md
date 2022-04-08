# ETH JSON RPC API
[JSON](https://json.org/) 是一种轻量级的数据交换格式。它可以表示数字、字符串、有序的值序列以及名称/值对的集合。

[JSON-RPC](http://www.jsonrpc.org/specification)是一种无状态、轻量级的远程过程调用 (RPC) 协议。该规范主要定义了几种数据结构及其处理规则。它与传输无关，因为这些概念可以在同一进程中、通过套接字、通过 HTTP 或在许多不同的消息传递环境中使用。它使用 JSON ( [RFC 4627](https://www.ietf.org/rfc/rfc4627.txt) ) 作为数据格式。

- Geth 1.4 具有实验性的发布/订阅支持。有关更多信息，[请参阅此页面](https://github.com/ethereum/go-ethereum/wiki/RPC-PUB-SUB)。
- Parity 1.6 具有实验性的 pub/sub 支持有关更多信息，[请参阅此内容](https://github.com/paritytech/parity/wiki/JSONRPC-Eth-Pub-Sub-Module)。
- Hyperledger Besu 1.3 支持发布/订阅。有关更多信息，[请参阅此内容](https://besu.hyperledger.org/en/stable/HowTo/Interact/APIs/RPC-PubSub/)。

## JavaScript API
要从 JavaScript 应用程序内部与以太坊节点对话，请使用 [web3.js](https://github.com/ethereum/web3.js) 库，它为 RPC 方法提供了一个方便的接口。

有关更多详细信息，请参阅 [JavaScript API](https://eth.wiki/clients/javascript-api)
## JSON-RPC Endpoint
客户端|URL
---|---
C++	|http://localhost:8545
Go|http://localhost:8545
Py|http://localhost:4000
Parity|http://localhost:8545
Hyperledger Besu|http://localhost:8545
### Go Eth 客户端
您可以使用以下 `--rpc` 标志启动 HTTP `JSON-RPC`

	geth --rpc
使用以下命令更改默认端口 (8545) 和列表地址 (localhost)

	geth --rpc --rpcaddr <ip> --rpcport <portnumber>
如果从浏览器访问 RPC，则需要使用适当的域集启用 CORS。否则，JavaScript 调用会受到同源策略的限制，请求将失败：

	geth --rpc --rpccorsdomain "http://localhost:3000"
JSON RPC 也可以使用命令从 [geth 控制台](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console) 启动 `admin.startRPC(addr, port)`

- Go Eth 对 JSON-RPC 支持
	-  json-rpc 2.0
	-  批处理
	-  http
	-  ipc
	-  ws

### C++
### python
### java
### 十六进制值编码
目前有两种通过 JSON 传递的关键数据类型：

- 未格式化的字节数组
- 数量

两者都使用十六进制编码传递，但对格式有不同的要求

- 编码 `QUANTITIES`（整数、数字）时：编码为十六进制，前缀为“0x”，最紧凑的表示（有轻微的例外：零应表示为“0x0”）。例子：
	- 0x41（十进制 65）
	- 0x400（十进制为 1024）
	- 错误：0x（应该总是至少有一个数字 - 零是“0x0”）
	- 错误：0x0400（不允许前导零）
	- 错误：ff（必须以 0x 为前缀）
- 编码 `UNFORMATTED DATA` （无格式数据）（字节数组、帐户地址、哈希、字节码数组）时：编码为十六进制，前缀为“0x”，每字节两个十六进制数字。例子：
	- 0x41（大小 1，“A”）
	- 0x004200（大小 3，“\0B\0”）
	- 0x（大小 0，“”）
	- 错误：0xf0f0f（必须是偶数位数）
	- 错误：004200（必须以 0x 为前缀）

目前[cpp-ethereum](https://github.com/ethereum/cpp-ethereum)、[go-ethereum](https://github.com/ethereum/go-ethereum)和 [parity](https://github.com/paritytech/parity) 通过 http 和 IPC（unix socket Linux 和 OSX/Windows 上的命名管道）提供 JSON-RPC 通信。go-ethereum 1.4 版、Parity 1.6 版和 Hyperledger Besu 1.3 版以后都支持 websocket

### 默认块参数
以下方法有一个额外的默认块参数：

- eth_getBalance
- eth_getCode
- eth_getTransactionCount
- eth_getStorageAt
- eth_call

当请求作用于以太坊的状态时，最后一个默认块参数决定了块的高度。

`defaultBlock` 参数可以使用以下选项：

- `HEX String` - 一个整数块号
- `String "earliest"` 对于最早/创世区块
- `String "latest"` - 最新开采的区块
- `String "pending"` - 对于挂起的状态/交易

### curl 示例解释
下面的 curl 选项可能会返回节点内容类型的响应，这是因为 `--data` 选项将内容类型设置为 `application/x-www-form-urlencoded` 。如果您的节点确实有问题，请通过在调用开始时放置 `-H “Content-Type: application/json”` 来手动设置标头。

这些示例也不包括 URL/IP 和端口组合，它必须是给 curl 例如 127.0.0.1:8545 的最后一个参数

## JSON-RPC 方法
### 节点相关
- web3_clientVersion

	返回当前客户端版本
	
	- 参数

		没有任何
	- 返回

		String - 当前客户端版本
	- 例
			
			// Request 请求
			curl -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}'
			
			// Result 返回
			{
			  "id":67,
			  "jsonrpc":"2.0",
			  "result": "Mist/v0.9.3/darwin/go1.4.1"
			}
- web3_sha3

	返回给定数据的 Keccak-256（注意不是标准化的 SHA3-256）
	
	- 参数
		- DATA 

			要转换为 SHA3 哈希的数据

				params: [
				  "0x68656c6c6f20776f726c64"
				]
	- 返回
		- DATA 给定字符串的 SHA3 结果。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"web3_sha3","params":["0x68656c6c6f20776f726c64"],"id":64}'
			
			// Result
			{
			  "id":64,
			  "jsonrpc": "2.0",
			  "result": "0x47173285a8d7341e5e972fc677286384f802f8ef42a5ec5f03bbfa254cb01fad"
			}
- net_version

	返回当前网络 ID。

	- 参数

		没有任何
	- 返回

		String - 当前网络 ID。

		- "1": 以太坊主网
		- "2"：现代测试网（已弃用）
		- "3": Ropsten 测试网
		- "4": Rinkeby 测试网
		- "42": 硬测试网
	- 例子
		
			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"net_version","params":[],"id":67}'
			
			// Result
			{
			  "id":67,
			  "jsonrpc": "2.0",
			  "result": "3"
			}
- net_peerCount

	返回当前连接到客户端的对等点数。

	- 参数

		没有任何
	- 返回

		`QUANTITY` - 连接的对等点数的整数(16进制)。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":74}'
			
			// Result
			{
			  "id":74,
			  "jsonrpc": "2.0",
			  "result": "0x2" // 2
			}
- net_listening

	如果客户端正在主动侦听网络连接，则返回 `true`。

	- 参数

		没有任何
	- 返回

		聆听时返回布尔结构 `true` ，否则 `false`。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"net_listening","params":[],"id":67}'
			
			// Result
			{
			  "id":67,
			  "jsonrpc":"2.0",
			  "result":true
			}
- eth_protocolVersion

	返回当前的以太坊协议版本。

	- 参数

		没有任何
	- 返回

		String - 当前的以太坊协议版本
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_protocolVersion","params":[],"id":67}'
			
			// Result
			{
			  "id":67,
			  "jsonrpc": "2.0",
			  "result": "54"
			}
- eth_syncing

	返回一个对象，其中包含有关同步状态或的数据 false。
	
	- 参数

		没有任何
	- 返回

		`Object|Boolean`, 一个带有同步状态数据的对象或者当不同步时`FALSE`：

		- `startingBlock: QUANTITY`- 导入开始的块（只会在同步到达他的头部后重置）
		- `currentBlock: QUANTITY`- 当前区块，与 eth_blockNumber 相同
		- `highestBlock: QUANTITY`- 估计的最高块
	- 例子
	
			// Request // 请求
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
			
			// Result // 返回
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": {
			    startingBlock: '0x384',
			    currentBlock: '0x386',
			    highestBlock: '0x454'
			  }
			}
			// Or when not syncing //不同步时
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": false
			}
- eth_coinbase

	返回客户端 coinbase 地址。

	- 参数

		没有任何
	- 返回
	
		DATA, 20 字节 - 当前币库地址。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_coinbase","params":[],"id":64}'
			
			// Result
			{
			  "id":64,
			  "jsonrpc": "2.0",
			  "result": "0x407d73d8a49eeb85d32cf465507dd71d507100c1"
- eth_call

	立即执行新消息调用，无需在区块链上创建交易。

	- 参数
		- Object - 交易调用对象
		- from: DATA, 20 Bytes -（可选）发送交易的地址。
		- to: DATA, 20 Bytes - 交易指向的地址。
		- gas：QUANTITY -提供用于事务执行的气体的（可选）整数。eth_call 消耗零气体，但某些执行可能需要此参数。
		- gasPrice: QUANTITY - (可选) 用于每个付费gas 的gasPrice 的整数
		- value：QUANTITY -值（可选）整数与此事务中发送
		- data: DATA - (可选) 方法签名和编码参数的哈希值。有关详细信息，请参阅 [Solidity 文档中的 Ethereum Contract ABI](https://solidity.readthedocs.io/en/latest/abi-spec.html)
		- QUANTITY|TAG- 整数块编号，或字符串"latest","earliest"或"pending"，请参阅默认块参数
	- 返回
		- DATA - 执行合约的返回值。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_call","params":[{see above}],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x"
			}
			
### 矿工相关
- eth_mining

	`true` 如果客户端正在积极挖掘新块，则返回。

	- 参数

		没有任何
	- 返回

		客户的回报是挖掘返回 true，否则 false。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_mining","params":[],"id":71}'
			
			// Result
			{
			  "id":71,
			  "jsonrpc": "2.0",
			  "result": true
			}
- eth_hashrate

	返回节点正在挖掘的每秒哈希数。

	- 参数

		没有任何
	- 返回

		`QUANTITY` - 每秒的哈希数。
	- 例子
	
		// Request
		curl -X POST --data '{"jsonrpc":"2.0","method":"eth_hashrate","params":[],"id":71}'
		
		// Result
		{
		  "id":71,
		  "jsonrpc": "2.0",
		  "result": "0x38a"
		}
		
### 账户地址相关
- eth_accounts

	返回客户端拥有的地址列表。
	
	- 参数

		没有任何
	- 返回

		`Array of DATA`, 20 Bytes - 客户端拥有的地址。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"]
			}
- eth_getBalance

	返回给定地址的账户余额。

	- 参数
		- `DATA`, 20 Bytes - 检查余额的地址。
		- `QUANTITY|TAG`- 整数块编号，或字符串 `"latest","earliest"` 或 `"pending"`，请参[阅默](https://eth.wiki/json-rpc/API#the-default-block-parameter)认块参数

				params: [
				   '0x407d73d8a49eeb85d32cf465507dd71d507100c1',
				   'latest'
				]
	- 返回

		`QUANTITY` - wei 当前余额的整数。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1", "latest"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x0234c8a3397aab58" // 158972490234375000
			}
- eth_getTransactionCount

	返回从一个地址发送的交易数量

	- 参数
		- DATA, 20 字节 - 地址。
		- QUANTITY|TAG- 整数块编号，或字符串"latest","earliest"或"pending"，请参阅默认块参数
	- 参数例子

			params: [
			   '0x407d73d8a49eeb85d32cf465507dd71d507100c1',
			   'latest' // state at the latest block
			]
	- 返回

			QUANTITY - 从该地址发送的交易数量的整数。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1","latest"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x1" // 1
			}

### 合约相关
- eth_getCode

	返回给定合约地址的代码。
	
	- 参数
		- DATA, 20 字节 - 地址
		- QUANTITY|TAG- 整数块编号，或字符串"latest","earliest"或"pending"，请参阅默认块参数
	- 参数例子

			params: [
			   '0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b',
			   '0x2'  // 2
			]
	- 返回

		DATA - 来自给定地址的代码。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getCode","params":["0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b", "0x2"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x600160008035811a818181146012578301005b601b6001356025565b8060005260206000f25b600060078202905091905056"
			}
- eth_getStorageAt 

	从给定的合约地址存储位置返回值

	- 参数
		- `DATA`, 20 Bytes - 存储地址。
		- `QUANTITY` - 存储位置的整数。
		- `QUANTITY|TAG`- 整数块编号，或字符串 `"latest","earliest"`或`"pending"`，请参[阅默](https://eth.wiki/json-rpc/API#the-default-block-parameter)认块参数
	- 返回

		`DATA` - 该存储位置的值。
	- 例子
		- 合约创建存储和存储数据
		
			计算正确位置取决于要检索的合约的存储。考虑在 `0x295a70b2de5e3953354a6a8344e616ed314d7251` 合约下面，按地址下按地址部署合同`0x391694e7e0b0cce554cb130d723a9d27458f9298`。

				contract Storage { 
				    uint pos0;
				    mapping(address => uint) pos1; //创建存储
				
				    function Storage() { 		//初始化存储数据
				        pos0 = 1234;  // 固定数据
				        pos1[msg.sender] = 5678; //动态数据
				    }
				}
		- 读取合约数据		
		
			检索 pos0 的值很简单：
			
			- 请求
			
					curl -X POST --data '{"jsonrpc":"2.0", "method": "eth_getStorageAt", "params": ["0x295a70b2de5e3953354a6a8344e616ed314d7251", "0x0", "latest"], "id": 1}' localhost:8545
			- 返回

					{"jsonrpc":"2.0","id":1,"result":"0x00000000000000000000000000000000000000000000000000000000000004d2"}
		- 检索 maping 的元素,元素在maping中的位置计算如下：

				keccack(LeftPad32(key, 0), LeftPad32(map position, 0))

			这意味着要检索 `pos1[“0x391694e7e0b0cce554cb130d723a9d27458f9298”] `上的存储，我们需要计算位置：

				keccak(decodeHex("000000000000000000000000391694e7e0b0cce554cb130d723a9d27458f9298" + "0000000000000000000000000000000000000000000000000000000000000001"))
			可以使用 web3 库自带的 geth 控制台进行计算：

				> var key = "000000000000000000000000391694e7e0b0cce554cb130d723a9d27458f9298" + "0000000000000000000000000000000000000000000000000000000000000001"
				undefined
				> web3.sha3(key, {"encoding": "hex"})
				"0x6661e9d6d8b923d5bbaab1b96e1dd51ff6ea2a93520fdc9eb75d059238b8c5e9"
			- 现在获取存储：
				- 请求

						curl -X POST --data '{"jsonrpc":"2.0", "method": "eth_getStorageAt", "params": ["0x295a70b2de5e3953354a6a8344e616ed314d7251", "0x6661e9d6d8b923d5bbaab1b96e1dd51ff6ea2a93520fdc9eb75d059238b8c5e9", "latest"], "id": 1}' localhost:8545
				- 返回

						{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000000000000000000000000000000000000000162e"}
						
### 签名相关
- eth_sign

	sign 方法计算以太坊消息签名(非交易签名)：
	
		sign(keccak256("\x19Ethereum Signed Message:\n" + len(message) + message)))。
	通过向消息添加前缀，可以将计算出的签名识别为以太坊特定的签名。这可以防止恶意 DApp 可以签署任意数据（例如交易）并使用签名来冒充受害者的滥用。

	请注意，签名的地址必须解锁

	- 参数

		账号、留言

		- DATA, 20 字节 - 地址
		- DATA, N Bytes - 要签名的消息
	- 返回

		DATA： 签名
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sign","params":["0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", "0xdeadbeaf"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
			}
	`eth_sign` 可以在[此处](https://gist.github.com/bas-vk/d46d83da2b2b4721efb0907aecdb7ebd)找到如何使用 solidity ecrecover 来验证所计算的签名的示例。该合约部署在测试网 Ropsten 和 Rinkeby 上。
- eth_signTransaction

	使用 `eth_sendRawTransaction` 签署可以稍后提交到网络的交易。

	- 参数
		- `Object` - 交易对象
		- `from: DATA`, 20 Bytes - 发送交易的地址。
		- `to: DATA`, 20 Bytes -（创建新合约时可选）交易的地址。
		- `gas：QUANTITY` - （可选，默认：90000）提供了一种用于事务执行的气体的整数。它将返回未使用的气体。
		- `gasPrice：QUANTITY` - （可选，默认：待确定）的gasPrice的整数用于每个付费气，卫。
		- `value：QUANTITY` -值（可选）整数与此事务中发送，在卫。
		- `data: DATA` - 合约的编译代码或调用的方法签名和编码参数的哈希值。有关详细信息，请参阅[以太坊合约 ABI](https://eth.wiki/json-rpc/API/Ethereum-Contract-ABI)。
		- `nonce：QUANTITY` -一个随机数（可选）整数。这允许覆盖您自己的使用相同随机数的待处理事务。
	- 返回

		`DATA`, 已签名的交易对象。
	- 例子

			// Request
			curl -X POST --data '{
			    "id": 1,
			    "jsonrpc": "2.0",
			    "method": "eth_signTransaction",
			    "params": [{
			        "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675",
			        "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
			        "gas": "0x76c0",
			        "gasPrice": "0x9184e72a000",
			        "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
			        "value": "0x9184e72a"
			    }]
			}'
			
			// Result
			{
			    "id": 1,
			    "jsonrpc": "2.0",
			    "result": "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
			}
			
### gas 相关
- eth_gasPrice

	以 wei 为单位返回每个 gas 的当前价格。

	- 参数

		没有任何
	- 返回
	
		QUANTITY - 当前 gas 价格的整数，单位为 wei。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":73}'
			
			// Result
			{
			  "id":73,
			  "jsonrpc": "2.0",
			  "result": "0x1dfd14000" // 8049999872 Wei
			}
- eth_estimateGas

	生成并返回完成交易所需的燃料量的估计值。交易不会被添加到区块链中。请注意，由于包括 EVM 机制和节点性能在内的各种原因，估计值可能明显高于交易实际使用的气体量。

	- 参数

		参见 [eth_call](https://eth.wiki/json-rpc/API#eth_call) 参数，期望所有属性都是可选的。如果没有指定 gas 限制，geth 使用待处理块中的块 gas 限制作为上限。因此，当 gas 量高于挂起块的 gas 限制时，返回的估计可能不足以执行调用/交易。
	- 返回

		QUANTITY - 使用的气体量。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_estimateGas","params":[{see above}],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x5208" // 21000
			}
			
### 交易相关
- eth_sendTransaction

	如果数据字段包含代码，则创建新的消息调用事务或合约创建。

	- 参数
		- Object - 交易对象
		- from: DATA, 20 Bytes - 发送交易的地址。
		- to: DATA, 20 Bytes -（创建新合约时可选）交易的地址。
		- gas：QUANTITY - （可选，默认：90000）提供了一种用于事务执行的气体的整数。它将返回未使用的气体。
		- gasPrice：QUANTITY - （可选，默认：待确定）的gasPrice的整数用于每个付费气体
		- value：QUANTITY -值（可选）整数与此事务中发送
		- data: DATA - 合约的编译代码或调用的方法签名和编码参数的哈希值。有关详细信息，请参阅以太坊合约 ABI
		- nonce：QUANTITY -一个随机数（可选）整数。这允许覆盖您自己的使用相同随机数的待处理事务。

				params: [{
				  "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
				  "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
				  "gas": "0x76c0", // 30400
				  "gasPrice": "0x9184e72a000", // 10000000000000
				  "value": "0x9184e72a", // 2441406250
				  "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"
				}]
	- 返回
	
		DATA, 32 Bytes - 交易散列，如果交易尚不可用则为零散列。

		使用 `eth_getTransactionReceipt` 获取合约地址，在交易被挖掘后，当您创建合约时。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{see above}],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
			}
- eth_sendRawTransaction

	为已签名的交易创建新的消息调用交易或合约创建。

	- 参数
		- DATA, 签署的交易数据。
	- 参数例子
				
				params: ["0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"]
	- 返回

		DATA, 32 Bytes - 交易散列，如果交易尚不可用则为零散列。

	使用 `eth_getTransactionReceipt` 获取合约地址，在交易被挖掘后，当您创建合约时。
	
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":[{see above}],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
			}
			
- eth_getTransactionByHash

	返回交易哈希请求的交易信息。

	- 参数
		- DATA, 32 Bytes - 交易的哈希值
	- 参数说明

			params: [
			   "0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b"
			]
	- 返回

		Object- 一个交易对象，或者null当没有发现交易时：
		
		- blockHash: DATA, 32 Bytes - 该交易所在区块的哈希值。null当其未决时。
		- blockNumber: QUANTITY- 此交易所在的区块号。null当它挂起时。
		- from: DATA, 20 Bytes - 发件人的地址。
		- gas: QUANTITY- 发送者提供的气体。
		- gasPrice: QUANTITY- 发送方在 Wei 提供的 gas 价格。
		- hash: DATA, 32 Bytes - 交易的哈希值。
		- input: DATA- 数据与交易一起发送。
		- nonce: QUANTITY- 在此之前发件人进行的交易数量。
		- to: DATA, 20 Bytes - 接收者的地址。null当它是一个合约创建交易时。
		- transactionIndex: QUANTITY- 区块中交易索引位置的整数。null当它待定时。
		- value: QUANTITY- 以魏转移的价值。
		- v: QUANTITY- ECDSA 恢复 ID
		- r: DATA, 32 字节 - ECDSA 签名 r
		- s: DATA, 32 Bytes - ECDSA 签名
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b"],"id":1}'
			
			// Result
			{
			  "jsonrpc":"2.0",
			  "id":1,
			  "result":{
			    "blockHash":"0x1d59ff54b1eb26b013ce3cb5fc9dab3705b415a67127a003c3e61eb445bb8df2",
			    "blockNumber":"0x5daf3b", // 6139707
			    "from":"0xa7d9ddbe1f17865597fbd27ec712455208b6b76d",
			    "gas":"0xc350", // 50000
			    "gasPrice":"0x4a817c800", // 20000000000
			    "hash":"0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b",
			    "input":"0x68656c6c6f21",
			    "nonce":"0x15", // 21
			    "to":"0xf02c1c8e6114b1dbe8937a39260b5b0a374432bb",
			    "transactionIndex":"0x41", // 65
			    "value":"0xf3dbb76162000", // 4290000000000000
			    "v":"0x25", // 37
			    "r":"0x1b5e176d927f8e9ab405058b2d2457392da3e20f328b16ddabcebc33eaac5fea",
			    "s":"0x4ba69724e8f69de52f0125ad8b3c5c2cef33019bac3249e2c0a2192766d1721c"
			  }
			}
- eth_getTransactionByBlockHashAndIndex

	通过区块哈希和交易索引位置返回有关交易的信息。

	- 参数
		- DATA, 32 Bytes - 一个区块的哈希值。
		- QUANTITY - 交易索引位置的整数。

				params: [
				   '0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331',
				   '0x0' // 0
				]
	- 返回

		参见 eth_getTransactionByHash
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockHashAndIndex","params":["0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b", "0x0"],"id":1}'

		结果见 eth_getTransactionByHash
- eth_getTransactionByBlockNumberAndIndex

	按块号和交易索引位置返回有关交易的信息。

	- 参数
		- QUANTITY|TAG- 块编号，或字符串"earliest","latest"或"pending"，如默认块参数。
		- QUANTITY - 交易指数位置。

				params: [
				   '0x29c', // 668
				   '0x0' // 0
				]
		- 返回

			参见 eth_getTransactionByHash
		- 例子

				// Request
				curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockNumberAndIndex","params":["0x29c", "0x0"],"id":1}'

			结果见 eth_getTransactionByHash	
- eth_getTransactionReceipt

	通过交易哈希返回交易的收据。

	请注意，收据不可用于待处理的交易。

	- 参数
		- DATA, 32 Bytes - 交易的哈希值
	- 参数例子
			
			params: [
			   '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238'
			]
	- 返回

		Object- 交易收据对象或未找到收据时为null：
		
		- transactionHash : DATA, 32 Bytes - 交易的哈希值。
		- transactionIndex: QUANTITY- 区块中交易索引位置的整数。
		- blockHash: DATA, 32 Bytes - 该交易所在区块的哈希值。
		- blockNumber: QUANTITY- 此交易所在的区块号。
		- from: DATA, 20 Bytes - 发件人的地址。
		- to: DATA, 20 Bytes - 接收者的地址。如果是合约创建交易，则为 null。
		- cumulativeGasUsed : QUANTITY - 在区块中执行此交易时使用的气体总量。
		- gasUsed : QUANTITY - 仅此特定交易使用的 gas 量。
		- contractAddress : DATA, 20 Bytes - 创建的合约地址，如果交易是合约创建，否则null.
		- logs: Array- 此事务生成的日志对象数组。
		- logsBloom: DATA, 256 Bytes - 用于轻客户端快速检索相关日志的布隆过滤器。
		它还返回或者：

		- root: DATA32 字节的交易后 stateroot (pre Byzantium)
		- status:QUANTITY要么1（成功）要么0（失败）
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'
			
			// Result
			{
			"id":1,
			"jsonrpc":"2.0",
			"result": {
			     transactionHash: '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238',
			     transactionIndex:  '0x1', // 1
			     blockNumber: '0xb', // 11
			     blockHash: '0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b',
			     cumulativeGasUsed: '0x33bc', // 13244
			     gasUsed: '0x4dc', // 1244
			     contractAddress: '0xb60e8dd61c5d32be8058bb8eb970870f07233155', // or null, if none was created
			     logs: [{
			         // logs as returned by getFilterLogs, etc.
			     }, ...],
			     logsBloom: "0x00...0", // 256 byte bloom filter
			     status: '0x1'
			  }
			}
			
### 块相关
- eth_blockNumber

	返回最近块的编号。
	
	- 参数

		没有任何
	- 返回

		QUANTITY - 客户端所在的当前块号的整数。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}'
			
			// Result
			{
			  "id":83,
			  "jsonrpc": "2.0",
			  "result": "0x4b7" // 1207
			}
- eth_getBlockTransactionCountByHash

	从匹配给定块哈希的块中返回块中的事务数。

	- 参数
		- DATA, 32 Bytes - 一个区块的哈希值
	- 参数例子

			params: [
			   '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238'
			]
	- 返回
		- QUANTITY - 此区块中交易数量的整数。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0xb" // 11
			}
- eth_getBlockTransactionCountByNumber

	返回与给定块编号匹配的块中的事务数。

	- 参数
		- QUANTITY|TAG- 块号的整数，或字符串"earliest"，"latest"或"pending"，如默认块参数。
	- 参数例子
	
			params: [
			   '0xe8', // 232
			]
	- 返回

		QUANTITY - 此区块中交易数量的整数。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByNumber","params":["0xe8"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0xa" // 10
			}
- eth_getUncleCountByBlockHash

	从匹配给定块哈希的块中返回块中叔叔的数量。

	- 参数
		- DATA, 32 Bytes - 一个区块的哈希值
	- 参数例子

			params: [
			   '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238'
			]
	- 返回

		QUANTITY - 此区块中叔叔的数量的整数
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x1" // 1
			}
- eth_getUncleCountByBlockNumber

	从匹配给定块号的块中返回块中叔块的数量。

	- 参数
		- QUANTITY|TAG- 区块编号的整数，或字符串“latest”、“earliest”或“pending”，参见默认区块参数
	- 参数例子

			params: [
			   '0xe8', // 232
			]
	- 返回

		QUANTITY - 此区块中叔叔的数量的整数。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockNumber","params":["0xe8"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x1" // 1
			}
- eth_getBlockByHash

	通过哈希返回返回所在块信息。
	
	- 参数
		- DATA, 32 Bytes - 一个区块的哈希值。
		- Boolean- 如果true它返回完整的交易对象，如果false只有交易的哈希值。
	
				params: [
				   '0xdc0818cf78f21a8e70579cb46a43643f78291264dda342ae31049421c82d21ae',
				   false
				]
	- 返回

		Object- 一个块对象，或者null当没有找到块时：
		
		- number: QUANTITY- 块号。null当它的待处理块时。
		- hash: DATA, 32 Bytes - 块的哈希值。null当它的待处理块时。
		- parentHash: DATA, 32 Bytes - 父区块的哈希值。
		- nonce: DATA, 8 Bytes - 生成的工作量证明的哈希值。null当它的待处理块时。
		- sha3Uncles: DATA, 32 Bytes - 块中叔块数据的 SHA3。
		- logsBloom: DATA, 256 Bytes - 块日志的布隆过滤器。null当它的待处理块时。
		- transactionsRoot: DATA, 32 Bytes - 块的事务树的根。
		- stateRoot: DATA, 32 Bytes - 块的最终状态树的根。
		- receiptsRoot: DATA, 32 Bytes - 块的回执树的根。
		- miner: DATA, 20 Bytes - 获得挖矿奖励的受益人的地址。
		- difficulty: QUANTITY- 此块难度的整数。
		- totalDifficulty: QUANTITY- 直到这个区块的链总难度的整数。
		- extraData: DATA- 此块的“额外数据”字段。
		- size: QUANTITY- 整数此块的大小（以字节为单位）。
		- gasLimit: QUANTITY- 此块中允许的最大气体。
		- gasUsed: QUANTITY- 此区块中所有交易使用的总gas。
		- timestamp: QUANTITY- 块被整理时的 unix 时间戳。
		- transactions: Array- 交易对象数组，或 32 字节交易哈希，取决于最后一个给定的参数。
		- uncles: Array- 叔叔哈希数组。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByHash","params":["0xdc0818cf78f21a8e70579cb46a43643f78291264dda342ae31049421c82d21ae", false],"id":1}'
			
			// Result
			{
			{
			  "jsonrpc": "2.0",
			  "id": 1,
			  "result": {
			    "difficulty": "0x4ea3f27bc",
			    "extraData": "0x476574682f4c5649562f76312e302e302f6c696e75782f676f312e342e32",
			    "gasLimit": "0x1388",
			    "gasUsed": "0x0",
			    "hash": "0xdc0818cf78f21a8e70579cb46a43643f78291264dda342ae31049421c82d21ae",
			    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
			    "miner": "0xbb7b8287f3f0a933474a79eae42cbca977791171",
			    "mixHash": "0x4fffe9ae21f1c9e15207b1f472d5bbdd68c9595d461666602f2be20daf5e7843",
			    "nonce": "0x689056015818adbe",
			    "number": "0x1b4",
			    "parentHash": "0xe99e022112df268087ea7eafaf4790497fd21dbeeb6bd7a1721df161a6657a54",
			    "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
			    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
			    "size": "0x220",
			    "stateRoot": "0xddc8b0234c2e0cad087c8b389aa7ef01f7d79b2570bccb77ce48648aa61c904d",
			    "timestamp": "0x55ba467c",
			    "totalDifficulty": "0x78ed983323d",
			    "transactions": [
			    ],
			    "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
			    "uncles": [
			    ]
			  }
			}
- eth_getBlockByNumber

	按块编号返回所查块信息。

	- 参数
		- QUANTITY|TAG- 块号的整数，或字符串"earliest"，"latest"或"pending"，如默认块参数。
		- Boolean- 如果true它返回完整的交易对象，如果false只有交易的哈希值。
	- 参数说明

			params: [
			   '0x1b4', // 436
			   true
			]
	- 返回

		参见 eth_getBlockByHash
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1b4", true],"id":1}'
			
			// 返回
			
		详细见 eth_getBlockByHash
- eth_getWork

	返回当前块的哈希值、seedHash 和要满足的边界条件（“目标”）。

	- 参数

		没有任何
	- 返回

		Array - 具有以下属性的数组：

		- DATA, 32 Bytes - 当前区块头 pow-hash
		- DATA, 32 Bytes - 用于 DAG 的种子哈希。
		- DATA, 32 Bytes - 边界条件（“目标”），2^256/难度。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getWork","params":[],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":"2.0",
			  "result": [
			      "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
			      "0x5EED00000000000000000000000000005EED0000000000000000000000000000",
			      "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
			    ]
			}
- eth_getUncleByBlockHashAndIndex

	通过哈希和叔块索引位置返回有关块的叔块的信息。

	- 参数
		- DATA, 32 Bytes - 一个区块的哈希值。
		- QUANTITY - 大叔的索引位置。
	- 参数例	
			
			params: [
			   '0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b',
			   '0x0' // 0
			]
	- 返回

		参见 eth_getBlockByHash
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockHashAndIndex","params":["0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b", "0x0"],"id":1}'

		结果见eth_getBlockByHash

		注意：叔叔不包含单个交易。
- eth_getUncleByBlockNumberAndIndex

	按编号和叔块索引位置返回有关块的叔块的信息。

	- 参数
		- QUANTITY|TAG- 块编号，或字符串"earliest","latest"或"pending"，如默认块参数。
		- QUANTITY - 叔的索引位置。
	- 参数例子

			params: [
			   '0x29c', // 668
			   '0x0' // 0
			]
	- 返回

		参见eth_getBlockByHash

		注意：叔叔不包含单个交易。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockNumberAndIndex","params":["0x29c", "0x0"],"id":1}'
	
		结果见eth_getBlockByHash
		
### 过滤器相关
- eth_newFilter

	根据过滤器选项创建过滤器对象，以在状态更改（日志）时发出通知。
	
	要检查状态是否已更改，请调用 eth_getFilterChanges。

	- 关于指定主题过滤器的说明

		主题是顺序相关的。具有主题 [A, B] 的日志的事务将被以下主题过滤器匹配：

		- `[]` “任何事物”
		- `[A]` “A 在第一位置（以及之后的任何位置）的东西”
		- `[null, B]` “任何在第一个位置和 B 在第二个位置的东西（以及之后的任何东西）”
		- `[A, B]` “A 在第一个位置和 B 在第二个位置（以及之后的任何位置）”
		- `[[A, B], [A, B]]` “（A OR B）在第一个位置和（A OR B）在第二个位置（以及之后的任何位置）”
	- 参数
		- `Object` - 过滤器选项：
		- `fromBlock：QUANTITY|TAG` - （可选，默认：`"latest"`）整数块编号或 `"latest"` 在过去的开采块或者 `"pending"`，`"earliest"` 对于尚未开采的交易。
		- `toBlock：QUANTITY|TAG` - （可选，默认：`"latest"` ）整数块编号或 `"latest"` 在过去的开采块或者 `"pending"，"earliest"` 对于尚未开采的交易。
		- `address: DATA|Array`, 20 Bytes -（可选）合约地址或日志应源自的地址列表。
		- `topics: Array of DATA`, - （可选）32 字节 `DATA` 主题数组。主题是顺序相关的。每个主题也可以是带有“或”选项的 DATA 数组。
	- 参数例子
			
			params: [{
			  "fromBlock": "0x1",
			  "toBlock": "0x2",
			  "address": "0x8888f1f195afa192cfee860698584c030f4c9db1",
			  "topics": ["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b", null, ["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b", "0x0000000000000000000000000aff3454fce5edbc8cca8697c15331677e6ebccc"]]
			}]
	- 返回

		QUANTITY - 过滤器 ID。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newFilter","params":[{"topics":["0x12341234"]}],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x1" // 1
			}
- eth_newBlockFilter

	在节点中创建过滤器，以在新块到达时进行通知。

	要检查状态是否已更改，请调用 eth_getFilterChanges。

	- 参数
		
		没有任何
	- 返回

		QUANTITY - 过滤器 ID。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newBlockFilter","params":[],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":  "2.0",
			  "result": "0x1" // 1
			}
- eth_newPendingTransactionFilter

	在节点中创建过滤器，以在新的待处理事务到达时进行通知。

	要检查状态是否已更改，请调用 eth_getFilterChanges。

	- 参数

		没有任何
	- 返回

		QUANTITY - 过滤器 ID。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newPendingTransactionFilter","params":[],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":  "2.0",
			  "result": "0x1" // 1
			}
- eth_uninstallFilter

	卸载具有给定 id 的过滤器。不再需要 watch 时应始终调用。此外，当一段时间内未使用eth_getFilterChanges请求时，过滤器会超时。

	- 参数

		`QUANTITY` - 过滤器 ID。
	- 参数例子

			params: [
			  "0xb" // 11
			]
	- 返回

		`Boolean`-  如果 true 过滤器被成功卸载，否则false。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_uninstallFilter","params":["0xb"],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": true
			}
- eth_getFilterChanges

	过滤器的轮询方法，它返回自上次轮询以来发生的日志数组。

	- 参数

		QUANTITY - 过滤器 ID。
	- 参数例子

			params: [
			  "0x16" // 22
			]
	- 返回

		Array - 日志对象数组，如果自上次轮询以来没有任何变化，则为空数组。

		- 对于使用 `eth_newBlockFilter` 返回创建的过滤器是块哈希（DATA, 32 字节），例如["0x3454645634534..."].
		- 对于使用 `eth_newPendingTransactionFilter` 返回创建的过滤器是交易哈希（DATA, 32 字节），例如["0x6345343454645..."]。
		- 对于使用 `eth_newFilter` 日志创建的过滤器是具有以下参数的对象：
			- `removed: TAG-true`当日志被删除时，由于链重组。`false` 如果它是一个有效的日志。
			- `logIndex: QUANTITY` - 块中日志索引位置的整数。`null`当它的待处理日志时。
			- `transactionIndex: QUANTITY` - 创建的交易索引位置日志的整数。`null` 当它的待处理日志时。
			- `transactionHash: DATA`, 32 Bytes - 创建此日志的事务的哈希值。`null` 当它的待处理日志时。
			- `blockHash: DATA`, 32 Bytes - 此日志所在块的哈希值。`null` 当它挂起时。`null` 当它的待处理日志时。
			- `blockNumber: QUANTITY` - 此日志所在的块号。`null`当它挂起时。`null`当它的待处理日志时。
			- `address: DATA`, 20 Bytes - 此日志的来源地址。
			- `data: DATA` - 包含日志的一个或多个 32 字节非索引参数。
			- `topics: Array of DATA` - 0 到 4 32 字节DATA索引日志参数的数组。（实事求是：第一个主题是事件签名的散列（例如Deposit(address,bytes32,uint256)），除非您使用说明anonymous符声明了事件。）
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getFilterChanges","params":["0x16"],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":"2.0",
			  "result": [{
			    "logIndex": "0x1", // 1
			    "blockNumber":"0x1b4", // 436
			    "blockHash": "0x8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcfdf829c5a142f1fccd7d",
			    "transactionHash":  "0xdf829c5a142f1fccd7d8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcf",
			    "transactionIndex": "0x0", // 0
			    "address": "0x16c5785ac562ff41e2dcfdf829c5a142f1fccd7d",
			    "data":"0x0000000000000000000000000000000000000000000000000000000000000000",
			    "topics": ["0x59ebeb90bc63057b6515673c3ecf9438e5058bca0f92585014eced636878c9a5"]
			    },{
			      ...
			    }]
			}
- eth_getFilterLogs

	返回与给定 id 匹配的所有日志的数组。

	- 参数
		- QUANTITY - 过滤器 ID。
	- 参数说明

			params: [
			  "0x16" // 22
			]
	- 返回

		参见 eth_getFilterChanges
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getFilterLogs","params":["0x16"],"id":74}'
		结果见 eth_getFilterChanges
- eth_getLogs

	返回与给定过滤器对象匹配的所有日志的数组。

	- 参数
		- `Object` - 过滤器选项：
		- `fromBlock：QUANTITY|TAG` - （可选，默认：`"latest"`）整数块编号或 `"latest"` 在过去的开采块或者 `"pending"，"earliest"`对于尚未开采的交易。
		- `toBlock：QUANTITY|TAG` - （可选，默认：`"latest"`）整数块编号或 `"latest"` 在过去的开采块或者`"pending"，"earliest"`对于尚未开采的交易。
		- `address: DATA|Array`, 20 Bytes -（可选）合约地址或日志应源自的地址列表。
		- `topics: Array of DATA`, - （可选）32 字节 `DATA` 主题数组。主题是顺序相关的。每个主题也可以是带有“或”选项的 DATA 数组。
		- `blockhash: DATA`, 32 Bytes - (optional, future ) 随着 EIP-234 的添加，`blockHash` 将是一个新的过滤器选项，它限制返回到具有 32 字节哈希的单个块的日志 `blockHash` 。使用 `blockHash` 等同于 `fromBlock==toBlock` 带有 hash 的块号`blockHash`。如果 `blockHash` 存在于过滤条件中，则既不允许 `fromBlock` 也不允许 `toBlock` 。
	- 参数例子

			params: [{
			  "topics": ["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b"]
			}]
	- 返回

		参见 eth_getFilterChanges
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getLogs","params":[{"topics":["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b"]}],"id":74}'
		结果见 eth_getFilterChanges
		
### 编译相关
- eth_getCompilers

	返回客户端中可用编译器的列表。

	- 参数

		没有任何	
	- 返回

		Array - 可用编译器数组。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getCompilers","params":[],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": ["solidity", "lll", "serpent"]
			}
- eth_compileSolidity

	返回已编译的 Solidity 代码。

	- 参数
		- String - 源代码。

				params: [
				   "contract test { function multiply(uint a) returns(uint d) {   return a * 7;   } }",
				]
	- 返回

		DATA - 编译后的源代码。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_compileSolidity","params":["contract test { function multiply(uint a) returns(uint d) {   return a * 7;   } }"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": {
			      "code": "0x605880600c6000396000f3006000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114602e57005b603d6004803590602001506047565b8060005260206000f35b60006007820290506053565b91905056",
			      "info": {
			        "source": "contract test {\n   function multiply(uint a) constant returns(uint d) {\n       return a * 7;\n   }\n}\n",
			        "language": "Solidity",
			        "languageVersion": "0",
			        "compilerVersion": "0.9.19",
			        "abiDefinition": [
			          {
			            "constant": true,
			            "inputs": [
			              {
			                "name": "a",
			                "type": "uint256"
			              }
			            ],
			            "name": "multiply",
			            "outputs": [
			              {
			                "name": "d",
			                "type": "uint256"
			              }
			            ],
			            "type": "function"
			          }
			        ],
			        "userDoc": {
			          "methods": {}
			        },
			        "developerDoc": {
			          "methods": {}
			        }
			      }
			
			}
- eth_compileLLL

	返回已编译的 LLL 代码。

	- 参数
		- String - 源代码。
	- 参数例

			params: [
			   "(returnlll (suicide (caller)))",
			]
	- 返回

		DATA - 编译后的源代码。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_compileLLL","params":["(returnlll (suicide (caller)))"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b6021600435602b565b8060005260206000f35b600081600702905091905056" // the compiled source code
			}

- eth_compileSerpent

	返回编译的蛇代码。

	- 参数
		- String - 源代码。
	- 参数例子
		
			params: [
			   "/* some serpent */",
			]
	- 返回

		DATA - 编译后的源代码。

	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"eth_compileSerpent","params":["/* some serpent */"],"id":1}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b6021600435602b565b8060005260206000f35b600081600702905091905056" // the compiled source code
			}			
			
### 挖矿相关			
- eth_submitWork

	用于提交工作量证明解决方案。

	- 参数
		- DATA, 8 Bytes - 找到的随机数（64 位）
		- DATA, 32 Bytes - 标头的 pow-hash（256 位）
		- DATA, 32 Bytes - 混合摘要（256 位）
	- 参数例子

			params: [
			  "0x0000000000000001",
			  "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
			  "0xD1FE5700000000000000000000000000D1FE5700000000000000000000000000"
			]
	- 返回

		`Boolean` - 如果 true 提供的解决方案有效，则返回，否则返回 `false`。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0", "method":"eth_submitWork", "params":["0x0000000000000001", "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef", "0xD1GE5700000000000000000000000000D1GE5700000000000000000000000000"],"id":73}'
			
			// Result
			{
			  "id":73,
			  "jsonrpc":"2.0",
			  "result": true
			}
- eth_submitHashrate

	用于提交挖矿算力。

	- 参数
		- Hashrate, 哈希率的十六进制字符串表示（32 字节）
		- ID, String - 标识客户端的随机十六进制（32 字节）ID
	- 参数例子

			params: [
			  "0x0000000000000000000000000000000000000000000000000000000000500000",
			  "0x59daa26581d0acd1fce254fb7e85952f4c09d0915afd33d3886cd914bc7d283c"
			]
	- 返回
	
		`Boolean`- 如果提交成功则返回 `true` ，否则返回 `false`。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0", "method":"eth_submitHashrate", "params":["0x0000000000000000000000000000000000000000000000000000000000500000", "0x59daa26581d0acd1fce254fb7e85952f4c09d0915afd33d3886cd914bc7d283c"],"id":73}'
			
			// Result
			{
			  "id":73,
			  "jsonrpc":"2.0",
			  "result": true
			}

### 悄悄话协议相关	
- shh_post

	发送悄悄话信息。

	- 参数
		- `Object` - 悄悄话帖子对象：
		- `from: DATA`, 60 Bytes -（可选）发送者的身份。
		- `to: DATA`, 60 Bytes -（可选）接收者的身份。当出现时，悄悄话将加密消息，以便只有接收者才能解密。
		- `topics: Array of DATA` - `DATA` 主题数组，供接收者识别消息。
		- `payload: DATA` - 消息的有效载荷。
		- `priority: QUANTITY` - 范围内的优先级整数，从… (?)。
		- `ttl: QUANTITY` - 以秒为单位的整数生存时间。
	- 参数例子

			params: [{
			  from: "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1",
			  to: "0x3e245533f97284d442460f2998cd41858798ddf04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a0d4d661997d3940272b717b1",
			  topics: ["0x776869737065722d636861742d636c69656e74", "0x4d5a695276454c39425154466b61693532"],
			  payload: "0x7b2274797065223a226d6",
			  priority: "0x64",
			  ttl: "0x64",
			}]
	- 返回

		`Boolean`- 如果提交成功则返回 `true` ，否则返回 `false`
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_post","params":[{"from":"0xc931d93e97ab07fe42d923478ba2465f2..","topics": ["0x68656c6c6f20776f726c64"],"payload":"0x68656c6c6f20776f726c64","ttl":0x64,"priority":0x64}],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":"2.0",
			  "result": true
			}

- shh_version

	返回当前的悄悄话协议版本。

	- 参数

		没有任何
	- 返回

		String - 当前的悄悄话协议版本
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_version","params":[],"id":67}'
			
			// Result
			{
			  "id":67,
			  "jsonrpc": "2.0",
			  "result": "2"
			}
- shh_newIdentity

	在客户端创建新的悄悄话身份。

	- 参数

		没有任何
	- 返回

		DATA, 60 Bytes - 新标识的地址。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_newIdentity","params":[],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0xc931d93e97ab07fe42d923478ba2465f283f440fd6cabea4dd7a2c807108f651b7135d1d6ca9007d5b68aa497e4619ac10aa3b27726e1863c1fd9b570d99bbaf"
			}
- shh_hasIdentity

	检查客户端是否持有给定身份的私钥。

	- 参数
		- DATA, 60 Bytes - 要检查的身份地址。
	- 参数例子
	
			params: [
			  "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"
			]
	- 返回

		`Boolean`- 如果提交成功则返回 `true` ，否则返回 `false`
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_hasIdentity","params":["0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": true
			}
- shh_newGroup

	？

	- 参数

		没有任何
	- 返回

		DATA, 60 Bytes - 新组的地址。(?)
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_newGroup","params":[],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": "0xc65f283f440fd6cabea4dd7a2c807108f651b7135d1d6ca90931d93e97ab07fe42d923478ba2407d5b68aa497e4619ac10aa3b27726e1863c1fd9b570d99bbaf"
			}
- shh_addToGroup

	？
	
	- 参数

		DATA, 60 Bytes - 要添加到组 (?) 的身份地址。
	- 参数例子

			params: [
			  "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"
			]
	- 返回

		`Boolean`- 如果提交成功则返回 `true` ，否则返回 `false`
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_addToGroup","params":["0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc": "2.0",
			  "result": true
			}
- shh_newFilter

	创建过滤器以在客户端收到与过滤器选项匹配的悄悄话消息时进行通知。

	- 参数
		- `Object` - 过滤器选项：
		- `to: DATA`, 60 Bytes -（可选）接收者的身份。当存在时，如果客户端持有此身份的私钥，它将尝试解密任何传入的消息。
		- `topics: Array of DATA-DATA` 传入消息的主题应该匹配的主题数组。您可以使用以下组合：
			- `[A, B] = A && B`
			- `[A, [B, C]] = A && (B || C)`
			- `[null, A, B] = ANYTHING && A && B null` 用作通配符
	- 例子

			params: [{
			   "topics": ['0x12341234bf4b564f'],
			   "to": "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"
			}]
	- 返回

		QUANTITY - 新创建的过滤器。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_newFilter","params":[{"topics": ['0x12341234bf4b564f'],"to": "0x2341234bf4b2341234bf4b564f..."}],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":"2.0",
			  "result": "0x7" // 7
			}
- shh_uninstallFilter

	卸载具有给定 id 的过滤器。不再需要 watch 时应始终调用。此外，当一段时间内没有使用 `shh_getFilterChanges` 请求时，过滤器超时。

	- 参数
		- `QUANTITY` - 过滤器 ID。
	- 参数例子

			params: [
			  "0x7" // 7
			]
	- 返回

		`Boolean`- 如果提交成功则返回 `true` ，否则返回 `false`
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_uninstallFilter","params":["0x7"],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":"2.0",
			  "result": true
			}
- shh_getFilterChanges

	悄悄话过滤器的轮询方法。返回自上次调用此方法以来的新消息。

	注意调用 `shh_getMessages` 方法，将重置此方法的缓冲区，这样您就不会收到重复的消息。

	- 参数
		- QUANTITY - 过滤器 ID。
	- 参数例子

			params: [
			  "0x7" // 7
			]
	- 返回
		- `Array` - 自上次轮询以来收到的消息数组：
			- `hash: DATA`, 32 Bytes (?) - 消息的哈希值。
			- `from: DATA`, 60 Bytes - 如果指定了发件人，则为消息的发件人。
			- `to: DATA`, 60 Bytes - 消息的接收者，如果指定了接收者。
			- `expiry: QUANTITY `- 此消息应该过期的时间整数（以秒为单位）（？）。
			- `ttl: QUANTITY` - 消息应该以秒为单位在系统中浮动的时间整数 (?)。
			- `sent: QUANTITY` - 发送消息时的 unix 时间戳整数。
			- `topics: Array of DATA-DATA` 消息包含的主题数组。
			- `payload: DATA` - 消息的有效载荷。
			- `workProved: QUANTITY` - 此消息在发送之前所需的工作的整数 (?)。
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_getFilterChanges","params":["0x7"],"id":73}'
			
			// Result
			{
			  "id":1,
			  "jsonrpc":"2.0",
			  "result": [{
			    "hash": "0x33eb2da77bf3527e28f8bf493650b1879b08c4f2a362beae4ba2f71bafcd91f9",
			    "from": "0x3ec052fc33..",
			    "to": "0x87gdf76g8d7fgdfg...",
			    "expiry": "0x54caa50a", // 1422566666
			    "sent": "0x54ca9ea2", // 1422565026
			    "ttl": "0x64", // 100
			    "topics": ["0x6578616d"],
			    "payload": "0x7b2274797065223a226d657373616765222c2263686...",
			    "workProved": "0x0"
			    }]
			}

- shh_getMessages

	获取与过滤器匹配的所有消息。与 shh_getFilterChanges 此不同的是返回所有消息。
	
	- 参数
		- QUANTITY - 过滤器 ID。
	- 参数说明

			params: [
			  "0x7" // 7
			]
	- 返回

		参见 shh_getFilterChanges
	- 例子

			// Request
			curl -X POST --data '{"jsonrpc":"2.0","method":"shh_getMessages","params":["0x7"],"id":73}'
		结果见 shh_getFilterChanges

## 弃用功能
- db_putString

	在本地数据库中存储一个字符串。

	请注意，此功能已弃用，将来会被删除。
- db_getString

	从本地数据库返回字符串。

	请注意，此功能已弃用，将来会被删除。
- db_putHex

	在本地数据库中存储二进制数据。

	请注意，此功能已弃用，将来会被删除。
- db_getHex

	从本地数据库返回二进制数据。

	请注意，此功能已弃用，将来会被删除。
	
## 参考
[https://eth.wiki/json-rpc/API](https://eth.wiki/json-rpc/API)		 