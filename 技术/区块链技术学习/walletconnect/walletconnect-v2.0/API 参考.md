## API 参考
### 客户端 API-SDK

	abstract class Client {
	  // ---------- 方法 ----------------------------------------------- //
	
	  // 使用持久存储和网络连接初始化客户端
	  public abstract init(params: {
	    controller?: boolean;
	    metadata?: AppMetadata;
	    relayProvider?: string;
	  }): Promise<void>;
	
	  // 提议者向响应者提议会话
	  public abstract connect(params: {
	    permissions: SessionPermissions;
	    pairing?: Sequence;
	  }): Promise<Sequence>;
	  
	  // 让响应者接收来自提议者的会话提议
	  public abstract pair(params: { uri: string }): Promise<Sequence>;
	
	  //让响应者批准会话提案
	  public abstract approve(params: {
	    proposal: SessionProposal;
	    response: SessionResponse;
	  }): Promise<Sequence>;
	  
	  // 响应者拒绝会话提议
	  public abstract reject(params: {
	    proposal: SessionProposal;
	    reason: Reason;
	  }): Promise<void>;
	  
	  // 让响应者升级会话权限
	  public abstract upgrade(params: {
	    topic: string;
	    permissions: SessionPermissions;
	  }): Promise<void>;
	  
	  // 让响应者更新会话状态
	  public abstract update(params: {
	    topic: string;
	    state: SessionState;
	  }): Promise<void>;
	
	  // 供提议者请求 JSON-RPC
	  public abstract request(params: {
	    topic: string;
	    request: RequestArguments;
	    chainId?: string;
	  }): Promise<any>;
	  
	  // 让响应者响应 JSON-RPC
	  public abstract respond(params: {
	    topic: string;
	    response: JsonRpcResponse;
	  }): Promise<void>;
	
	  //用于 ping 并验证对等方是否在线
	  public abstract ping(params: {
	    topic: string;
	  }): Promise<void>;
	  
	  // 用于 ether 发送通知
	  public abstract notify(params: {
	    topic: string;
	    notification: Notification;
	  }): Promise<void>;
	  
	  //用于 ether 断开会话
	  public abstract disconnect(params: {
	    topic: string;
	    reason: Reason;
	  }): Promise<void>;
	
	  // ---------- 事件 ----------------------------------------------- //
	  //订阅配对建议
	  public abstract on("pairing_proposal", (pairingProposal: PairingProposal) => {}): void;
	
	  // 订阅会话提案
	  public abstract on("session_proposal", (sessionProposal: SessionProposal) => {}): void;
	
	  //订阅会话请求
	  public abstract on("session_request", (sessionRequest: SessionRequest) => {}): void;
	
	  //订阅会话通知
	  public abstract on("session_notification", (sessionNotification: SessionNotification) => {}): void;
	}
	
	interface Sequence {
	  topic: string;
	}
	
	interface AppMetadata {
	  name: string;
	  description: string;
	  url: string;
	  icons: string[];
	}
	
	interface SessionPermissions {
	  blockchain: {
	    chains: string[];
	  };
	  jsonrpc: {
	    methods: string[];
	  };
	}
	
	interface SessionProposal {
	  topic: string;
	  relay: {
	    protocol: string;
	    params?: any;
	  };
	  proposer: {
	    publicKey: string;
	    metadata: AppMetadata;
	  };
	  signal: {
	    method: "pairing";
	    params: Sequence;
	  };
	  permissions: SessionPermissions;
	  ttl: number;
	}
	
	interface SessionState {
	  accounts: string[];
	}
	
	interface SessionResponse {
	  state: SessionState;
	}
	
	interface Reason {
	  code: number;
	  message: string;
	}
	
	interface RequestArguments {
	  method: string;
	  params: any;
	}
	
	interface JsonRpcRequest {
	  id: number;
	  jsonrpc: "2.0";
	  method: string;
	  params: any;
	}
	
	interface JsonRpcResult {
	  id: number;
	  jsonrpc: "2.0";
	  result: any;
	}
	
	interface JsonRpcError {
	  id: number;
	  jsonrpc: "2.0";
	  error: {
	    code: number;
	    message: string;
	  };
	}
	
	type JsonRpcResponse = JsonRpcResult | JsonRpcError;
	
	interface Notification {
	  type: string;
	  data: any;
	}
	
	interface SessionRequest {
	  topic: string;
	  request: JsonRpcRequest;
	  chainId?: string;
	}
	
	interface SessionNotificaiton {
	  topic: string;
	  notification: Notification;
	}
	
	interface PairingProposal {
	  topic: string;
	  relay: {
	    protocol: string;
	    params?: any;
	  };
	  proposer: {
	    publicKey: string;
	  };
	  signal: {
	    method: "pairing";
	    params: {
	      uri: string;
	    };
	  };
	  permissions: {
	    jsonrpc: {
	      methods: string[];
	    };
	  };
	  ttl: number;
	}

### 注册表 API
Registry API 当前提供以下功能：

- [Listings API](https://docs.walletconnect.com/2.0/api/registry-api#listings-api) - 允许获取 [WalletConnect 云注册表](https://walletconnect.com/registry)中列出的钱包和 dApp 。
- [logo API](https://docs.walletconnect.com/2.0/api/registry-api#logo-api) - 为给定的注册表项提供不同大小的logo资产。

#### 列表 API
默认情况下，列表端点返回提供类型的所有数据。您可以使用以下查询参数返回分页数据或按名称搜索特定列表：

名称|描述
---|---
entries|指定将返回多少条目（必须与页面参数一起使用）
page|指定当前页面（必须与条目参数一起使用）
search|返回名称与提供的搜索查询匹配的列表


-  `GET /api/v1/wallets`

	返回一个 JSON 对象，其中包含公共注册表中列出的所有钱包

	- 请求
	
			curl https://registry.walletconnect.com/api/v1/wallets
	- 返回
		
			 {
			  "count": 142,
			  "listings": [
			     "1ae92b26df02f0abca6304df07debccd18262fdf5fe82daa81593582dac9a369": {
			       "id": "1ae92b26df02f0abca6304df07debccd18262fdf5fe82daa81593582dac9a369",
			       "name": "Rainbow",
			   ...
- `GET /api/v1/dapps`

	返回一个 JSON 对象，其中包含公共注册表中列出的所有 dApp。

	- 请求
		
			curl https://registry.walletconnect.com/api/v1/dapps
	- 返回	
		
			 {
			   "count": 155,
			   "listings": [
			       "d2ae9c3c2782806fd6db704bf40ef0238af9470d7964ae566114a033f4a9a110": {
			         "id": "d2ae9c3c2782806fd6db704bf40ef0238af9470d7964ae566114a033f4a9a110",
			         "name": "Etherscan",
			           ...
- `GET /api/v1/all`

	返回一个 JSON 对象，其中包含公共注册表中列出的所有条目。

	- 请求
		
			curl https://registry.walletconnect.com/api/v1/all
	- 返回		
		
			{
			   "count": 290,
			   "listings": [
			       "1ae92b26df02f0abca6304df07debccd18262fdf5fe82daa81593582dac9a369": {
			         "id": "1ae92b26df02f0abca6304df07debccd18262fdf5fe82daa81593582dac9a369",
			         "name": "Rainbow",
			         ...
			       },
			       "d2ae9c3c2782806fd6db704bf40ef0238af9470d7964ae566114a033f4a9a110": {
			         "id": "d2ae9c3c2782806fd6db704bf40ef0238af9470d7964ae566114a033f4a9a110",
			         "name": "Etherscan",
			         ...

#### LOGO API
- `GET /api/v1/logo/:size/:id`
	- `size` 参数可以是以下之一：`sm | md | lg`(小｜中｜大)
	- `id` 参数对应于 `id` 列表 API 返回的注册表项的字段			         
	`id` 根据 `size` ,返回 logo 的图像源，

			# 解析大 (`lg`) 格式的 Etherscan 徽标。
			curl 'https://registry.walletconnect.com/api/v1/logo/lg/d2ae9c3c2782806fd6db704bf40ef0238af9470d7964ae566114a033f4a9a110'
			
			# -> https://imagedelivery.net/...

### 中继服务器 API
#### WebSocket API (JSON-RPC)
- Subscribe
	- 接口

			// 请求 (Client -> Server)
			interface WakuSubscribeRequest {
			  id: number;
			  jsonrpc: "2.0";
			  method: "waku_subscribe";
			  params: {
			    topic: string;
			  };
			}
			
			// 响应 (Server -> Client)
			interface WakuSubscribeResponse {
			  id: number;
			  jsonrpc: "2.0";
			  result: string;
			}
	- 例子

			// 请求 (Client -> Server)
			{
			  "id": 1,
			  "jsonrpc": "2.0",
			  "method": "waku_subscribe",
			  "params": {
			    "topic": "<TOPIC_ID>",
			  }
			}
			
			// 响应 (Server -> Client)
			{
			  "id": 1,
			  "jsonrpc": "2.0",
			  "result": "<SUBSCRIPTION_ID>"
			}		
- Publish
	- 接口
	
			// 请求 (Client -> Server)
			interface WakuPublishRequest {
			  id: number;
			  jsonrpc: "2.0";
			  method: "waku_publish";
			  params: {
			    topic: string;
			    message: string;
			    ttl: number;
			  };
			}
			
			// 响应 (Server -> Client)
			interface WakuPublishResponse {
			  id: number;
			  jsonrpc: "2.0";
			  result: true;
			}
	- 例子

			// 请求 (Client -> Server)
			{
			  "id": 2,
			  "jsonrpc": "2.0",
			  "method": "waku_publish",
			  "params": {
			    "topic": "<TOPIC_ID>",
			    "message": "<MESSAGE_PAYLOAD>",
			    "ttl": 86400
			  }
			}
			
			// 响应 (Server -> Client)
			{
			  "id": 2,
			  "jsonrpc": "2.0",
			  "result": true
			}		
- Subscription
	- 接口

			// 请求 (Server -> Client)
			interface WakuSubscriptionRequest {
			  id: number;
			  jsonrpc: "2.0";
			  method: "waku_subscription";
			  params: {
			    id: string;
			    data: {
			      topic: string;
			      message: string;
			    };
			  };
			}
			
			// 响应 (Client -> Server)
			interface WakuSubscriptionResponse {
			  id: number;
			  jsonrpc: "2.0";
			  result: true;
			}
	- 例子

			// 请求 (Server -> Client)
			{
			  "id": 3,
			  "jsonrpc": "2.0",
			  "method": "waku_subscription",
			  "params": {
			    "id": "<SUBSCRIPTION_ID>",
			    "data": {
			      "topic": "<TOPIC_ID>",
			      "message": "<MESSAGE_PAYLOAD>",
			    }
			  }
			}
			
			// 响应 (Client -> Server)
			{
			  "id": 3,
			  "jsonrpc": "2.0",
			  "result": true
			}
- Unsubscribe
	- 接口

			// 请求 (Client -> Server)
			interface WakuUnsubscribeRequest {
			  id: number;
			  jsonrpc: "2.0";
			  method: "waku_unsubscribe";
			  params: {
			    topic: string;
			    id: string;
			  };
			}
			
			// 响应 (Server -> Client)
			interface WakuUnsubscribeResponse {
			  id: number;
			  jsonrpc: "2.0";
			  result: true;
			}
	- 例子

			// 请求 (Client -> Server)
			{
			  "id": 4,
			  "jsonrpc": "2.0",
			  "method": "waku_unsubscribe",
			  "params": {
			    "topic": "<TOPIC_ID>",
			    "id": "<SUBSCRIPTION_ID>",
			  }
			}
			
			// 响应 (Server -> Client)
			{
			  "id": 4,
			  "jsonrpc": "2.0",
			  "result": true
			}

#### HTTP API
- Test Hello World

		  GET https://relay.walletconnect.com/hello
		
		  响应:
		  Status: 200
		  Content-Type: text/plain; charset=utf-8
		  Body: Hello World, this is WalletConnect v2.0
- 推送通知

		  POST https://relay.walletconnect.com/subscribe
		  Content-Type: application/json
		  Body:
		  {
		    "topic": <client_id>,
		    "webhook": <push_notification_webhook>
		  }
		
		  响应:
		  Status: 200
		  Content-Type: application/json; charset=utf-8
		  Body:
		  {
		    "success": true
		  }

### 项目编号
#### 如何实施
项目 ID 通过 URL 参数使用。

使用的网址参数：

- projectId：您的项目 ID 可以从 walletconnect.com 获得

示例网址：

`https://relay.walletconnect.com/?projectId=c4f79cc821944d9680842e34466bfbd`

这可以从 `WalletConnectClient` 客户端使用构造函数 `projectId` 中的实例化。

	import WalletConnectClient from "@walletconnect/client";
	const client = await WalletConnectClient.init({
	  projectId: "c4f79cc821944d9680842e34466bfb",
	});

#### 白名单
由于大部分钱包和 dapp 代码将在客户端，项目 ID 的安全性取决于钱包的用户代理和 HTTP Origin 的正确实现。

- 应用

	钱包的用户代理。
	
	TODO 插入用户代理允许列表的屏幕截图
	
	- Kotlin
	- Swift
- 网站

	网站[起源](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin)。

	TODO 插入 HTTP 源允许列表的屏幕截图 

### 错误
原因|错误代码
---|---
项目id不存在|401
存在且无效|403	

	
		  

