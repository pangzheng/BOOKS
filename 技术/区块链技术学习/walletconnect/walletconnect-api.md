# walletconnect-api
## 以太坊 Json-RPC API 方法
### `personal_sign`
该方法是计算以太坊特定的签名

	sign(keccack256("\x19Ethereum Signed Message:\n" + len(message) + message)))。
通过在消息中添加前缀，可以将计算出的签名识别为以太坊特定的签名。这可以防止恶意 DApp 可以签署任意数据（例如交易）并使用签名来冒充受害者的滥用行为。

注意请参阅 `ecRecover` 以验证签名。

- 参数

	消息，账号

	- `DATA` 	// N Bytes - 要签名的消息
	- `DATA`	// 20 字节 - 地址。
- 返回
	- `DATA`	//签名
- 例

		// 请求
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "method": "personal_sign",
		  "params":["0xdeadbeaf","0x9b2055d370f73ec7d8a03e965129118dc8f5bf83"],
		}
		
		// 返回
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "result": "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
		}

### eth_sign
该方法计算以太坊特定的签名

	sign(keccak256("\x19Ethereum Signed Message:\n" + len(message) + message)))。

通过在消息中添加前缀，可以将计算出的签名识别为以太坊特定的签名。这可以防止恶意 DApp 可以签署任意数据（例如交易）并使用签名来冒充受害者的滥用行为。

注意签名的地址必须是解锁的。

- 参数

	消息，账号

	- `DATA` 	// N Bytes - 要签名的消息
	- `DATA`	// 20 字节 - 地址。
- 返回
	- `DATA`	//签名
- 例

		// 请求
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "method": "eth_sign",
		  "params": ["0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", "0xdeadbeaf"],
		}
		
		// 返回
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "result": "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
		}
`eth_sign` 可以在[此处](https://gist.github.com/bas-vk/d46d83da2b2b4721efb0907aecdb7ebd)找到如何使用 solidity ecrecover 验证计算签名的示例。该合约部署在测试网 Ropsten 和 Rinkeby 上。

### eth_signTypedData
以下形式计算以太坊特定的签名

	 keccak256("\x19Ethereum Signed Message:\n" + len(message) + message))
通过在消息中添加前缀，可以将计算出的签名识别为以太坊特定的签名。这可以防止恶意 DApp 可以签署任意数据（例如交易）并使用签名来冒充受害者的滥用行为。

注意签名的地址必须是解锁的。

- 参数

	消息，账号

	- `DATA`, 20 字节 - 地址。
	- `DATA`, N Bytes - 要签名的消息，包含类型信息、域分隔符和数据

	参数例

		[
		  "0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826",
		  {
		    "types": {
		      "EIP712Domain": [
		        {
		          "name": "name",
		          "type": "string"
		        },
		        {
		          "name": "version",
		          "type": "string"
		        },
		        {
		          "name": "chainId",
		          "type": "uint256"
		        },
		        {
		          "name": "verifyingContract",
		          "type": "address"
		        }
		      ],
		      "Person": [
		        {
		          "name": "name",
		          "type": "string"
		        },
		        {
		          "name": "wallet",
		          "type": "address"
		        }
		      ],
		      "Mail": [
		        {
		          "name": "from",
		          "type": "Person"
		        },
		        {
		          "name": "to",
		          "type": "Person"
		        },
		        {
		          "name": "contents",
		          "type": "string"
		        }
		      ]
		    },
		    "primaryType": "Mail",
		    "domain": {
		      "name": "Ether Mail",
		      "version": "1",
		      "chainId": 1,
		      "verifyingContract": "0xCcCCccccCCCCcCCCCCCcCcCccCcCCCcCcccccccC"
		    },
		    "message": {
		      "from": {
		        "name": "Cow",
		        "wallet": "0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826"
		      },
		      "to": {
		        "name": "Bob",
		        "wallet": "0xbBbBBBBbbBBBbbbBbbBbbbbBBbBbbbbBbBbbBBbB"
		      },
		      "contents": "Hello, Bob!"
		    }
		  }
		]
- 返回
	- `DATA` //签名数据
- 例

		// 请求
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "method": "eth_signTypedData",
		  "params": ["0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", {see above}],
		}
		'
		
		// 返回
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "result": "0x4355c47d63924e8a72e509b65029052eb6c299d53a04e167c5775fd466751c9d07299936d304c153f6443dfa05f40ff007d72911b6f72307f996231605b915621c"
		}

### eth_sendTransaction(发送未签名交易)
如果数据字段包含代码，则创建新的消息调用交易或合同创建。

- 请求
	- 参数
		- `Object`				// 交易对象
		- `from`					// `DATA`, 20 Bytes - 发送交易的地址。
		- `to`					// `DATA`, 20 Bytes - （创建新合约时可选）交易指向的地址。
		- `data`					// `DATA` 合约的编译代码或调用的方法签名和编码参数的哈希值。详情见以太坊合约 ABI
		- `gas`					//`QUANTITY`- (可选，默认值： 90000 )为交易执行提供的气体的整数。它将返回未使用的气体。
		- `gasPrice`				// `QUANTITY`- (可选，默认值：待定)用于每个付费gas的gasPrice的整数
		- `value`					// `QUANTITY`- （可选）与此交易一起发送的值的整数
		- `nonce`				// `QUANTITY`- （可选）随机数的整数。这允许覆盖您自己的使用相同 nonce 的待处理事务。
	- 参数例
	
			[
			  {
			    "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
			    "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
			    "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675",
			    "gas": "0x76c0", // 30400
			    "gasPrice": "0x9184e72a000", // 10000000000000
			    "value": "0x9184e72a", // 2441406250
			    "nonce": "0x117" // 279
			  }
			]
	- 例子

			// 请求
			{
			  "id": 1,
			  "jsonrpc": "2.0",
			  "method": "eth_sendTransaction",
			  "params":[{see above}],
			}				
- 返回
	- 参数
		- `DATA`, 32 Bytes - 交易哈希，如果交易尚不可用，则为零哈希。
		- 使用 [eth_getTransactionReceipt](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionreceipt)获取合约地址，在交易被挖掘后，当你创建合约时。
	- 例子
		
			// 返回
			{
			  "id": 1,
			  "jsonrpc": "2.0",
			  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
			}

### eth_signTransaction(签名交易)
`eth_sendRawTransaction` 签署可以在以后使用提交到网络的交易 

- 请求
	- 参数
		- `Object`				// 交易对象
		- `from`					// `DATA`, 20 Bytes - 发送交易的地址。
		- `to`					// `DATA`, 20 Bytes - （创建新合约时可选）交易指向的地址。
		- `data`					// `DATA` 合约的编译代码或调用的方法签名和编码参数的哈希值。详情见以太坊合约 ABI
		- `gas`					//`QUANTITY`- (可选，默认值： 90000 )为交易执行提供的气体的整数。它将返回未使用的气体。
		- `gasPrice`				// `QUANTITY`- (可选，默认值：待定)用于每个付费gas的gasPrice的整数
		- `value`					// `QUANTITY`- （可选）与此交易一起发送的值的整数
		- `nonce`				// `QUANTITY`- （可选）随机数的整数。这允许覆盖您自己的使用相同 nonce 的待处理事务。
	- 参数例

			[
			  {
			    "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
			    "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
			    "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675", \\签署的交易数据
			    "gas": "0x76c0", // 30400
			    "gasPrice": "0x9184e72a000", // 10000000000000
			    "value": "0x9184e72a", // 2441406250
			    "nonce": "0x117" // 279
			  }
			]
	- 例子

			// 请求
			{
			  "id": 1,
			  "jsonrpc": "2.0",
			  "method": "eth_signTransaction",
			  "params":[{see above}],
			}
			
### eth_sendRawTransaction(发送签名交易)
为已签名的交易创建新的消息调用交易或合同创建。

- 参数
	- `DATA`	//签名的交易数据。
- 返回
	- 参数
		- `DATA`, 32 Bytes - 交易哈希，如果交易尚不可用，则为零哈希。
		- 使用 [eth_getTransactionReceipt](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionreceipt)获取合约地址，在交易被挖掘后，当你创建合约时。
- 例子

		// 请求
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "method": "eth_sendRawTransaction",
		  "params":[
		    "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f07244567"
		  ],
		}
		
		// 返回
		{
		  "id": 1,
		  "jsonrpc": "2.0",
		  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
		}
		
## 客户端 API
### 注册事件订阅

	function on(
	  event: string,
	  callback: (error: Error | null, payload: any | null) => void
	): void;
事件

- `connect`
- `disconnect`
- `session_request`
- `session_update`
- ` call_request`
- `wc_sessionRequest`
- `wc_sessionUpdate`

### 会话相关
- 创建新会话 (`session_request`)

		async function createSession(): Promise<void>;
- 批准会话请求(`connect`)

		function approveSession({
		  chainId: number, // Required
		  accounts: string[] // Required
		}): void;
- 拒绝会话请求(`disconnect`)
		
		function rejectSession({
		  message: 'OPTIONAL_ERROR_MESSAGE'
		}): void;
- 更新会话(`session_update`)
		
		function updateSession({
		  chainId: number, // Required
		  accounts: string[] // Required
		}): void;
- 终止会话(`disconnect`)
		
		function killSession(): void;

### eth 相关					
- 发送交易(`eth_sendTransaction`)

		async function sendTransaction({
		  from: string, // Required
		  to: string, // Required
		  gas: string, // Required
		  gasPrice: string, // Required
		  value: string, // Required
		  data: string, // Required
		  nonce: string, // Required
		}): Promise<string>;	
		
	返回交易hash
- 签署交易(`eth_signTransaction`)

		async function signTransaction({
		  from: string, // Required
		  to: string, // Required
		  gas: string, // Required
		  gasPrice: string, // Required
		  value: string, // Required
		  data: string, // Required
		  nonce: string, // Required
		}): Promise<string>;
	返回签署交易
- 签署消息(`eth_sign`)

		async function signMessage(params: string[]): Promise<string>;
	返回签名
- 签署个人消息(`personal_sign`)

		async function signPersonalMessage(params: string[]): Promise<string>; 
	返回签名
- 签署类型数据(`eth_signTypedData`)

		async function signTypedData(params: any[]): Promise<string>;
	返回签名

### 其他	
- 发送自定义请求

		async function sendCustomRequest(payload: IJsonRpcRequest): Promise<any>;
	返回 json-rpc 响应
- 批准调用请求

		function approveRequest({
		  id: number, // 必须
		  result: any, // 必须
		}): void;
- 拒绝调用请求

		function rejectRequest({
		  id: 1,
		  error: {
		    message: "OPTIONAL_ERROR_MESSAGE"
		  }
		}): void;		
		
## 桥接服务器 API
- Test Hello World
	- 请求
		
			GET https://bridge.walletconnect.org/hello
		
	- 返回

			Response:
			  Status: 200
			  Content-Type: text/plain; charset=utf-8
			  
			  Body: Hello World, this is WalletConnect v1.0
- 获取服务器信息
	- 请求

			GET https://bridge.walletconnect.org/info
	- 返回	
		  
			  Response:
			  Status: 200
			  Content-Type: application/json; charset=utf-8
			  
			Body:
			  {
			    "name": "WalletConnect Bridge Server",
			    "repository": "walletconnect-bridge",
			    "version": "1.0"
	 		   }
 - 订阅推送通知
	- 请求
		 
			 POST https://bridge.walletconnect.org/subscribe
			  Content-Type: application/json
			  Body:
			  {
			    "topic": <client_id>,
			    "webhook": <push_notification_webhook>
			  }
	- 返回	
		  
			  Response:
			  Status: 200
			  Content-Type: application/json; charset=utf-8
			  Body:
			  {
			    "success": true
			  }

## 推送服务器 API
- registry 推送通知
	- 请求
		  
			  POST <YOUR_PUSH_SERVER_URL>/new
			  Content-Type: application/json
			  Body:
			  {
			    "bridge": <bridge_url>,
			    "topic": <client_id>,
			    "type": <push_type>,
			    "token": <push_token>,
			    "peerName": <peer_name>,
			    "language": <language_code>,
			  }
	- 返回	
		 
			  Response:
			  Status: 200
			  Content-Type: application/json; charset=utf-8
			  Body:
			  {
			    "success": true
			  }
- 触发推送通知
	- 请求

			  POST <YOUR_PUSH_SERVER_URL>/push
			  Content-Type: application/json
			  Body:
			  {
			    "topic": <client_id>
			  }
	- 返回

			  Response:
			  Status: 200
			  Content-Type: application/json; charset=utf-8
			  Body:
			  {
			      "successs": true
			  }			

## 参考
- [客户端](https://docs.walletconnect.com/client-api)
- [桥接服务器](https://docs.walletconnect.com/bridge-server)
- [推送服务器](https://docs.walletconnect.com/push-server)
- [以太坊 json-rpc-api](https://docs.walletconnect.com/json-rpc-api-methods/ethereum)
	
			