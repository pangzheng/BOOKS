# Flow-常见问题
## 建设者/开发商
- 在可预见的未来，预期的 TPS（每秒交易数）是多少？

	并非所有交易都是平等的，因此仅凭吞吐量数字并不能说明全部情况。Flow 的交易吞吐量取决于多种因素，例如节点可用的网络带宽和我们实现中的优化。主要因素是交易复杂性，受交易执行的指令数量、交易执行的分类账读取和写入数量以及必须检查多少签名以确认您的交易具有所需权限的影响。
	
	对于简单令牌余额转移的基准负载（基本上是加法和减法加上两次签名验证），我们当前的实现轻松维持了显着超过 100/tps 的吞吐量。然而，我们观察到主网上的交易通常要复杂得多（需要多次签名验证；许多账本读写；以及运行相对复杂的计算）。对于近期的未来，100/tps 似乎是一个现实的量级。我们正在努力维持目前在主网上运行的那种 100/tps。如果您的交易比普通的 Mainnet 交易简单得多，Flow 可能已经满足您所需的吞吐量。找出答案的最佳方法是在 TestNet 上测试特定交易的基准集。
- Flow 有区块浏览器吗？

	今天有两个区块浏览器。你可以在这里找到它们：
	
	- [https://flowscan.org/](https://flowscan.org/)
	- [https://flow.bigdipper.live/](https://flow.bigdipper.live/)
- 如何连接和查询接入节点？有哪些可用数据？

	在协议层面，您可以通过 GRPC 连接到接入节点。我们提供 JavaScript 和 Golang SDK 来为您执行此操作。

	连接到访问节点后，您可以获取有关帐户、合约、区块、集合、交易和事件的信息。您还可以执行脚本来查询 Flow 区块链的当前状态。

	- 可用数据

		账户、合约、区块、集合、交易、事件，这些类型记录在 Access API 页面上。
		
		SDK 将 Flow 数据公开为类型。例如，可以在此处找到 Go Flow Block 数据类型实现：
		
		[https://github.com/onflow/flow-go-sdk/blob/master/block.go](https://github.com/onflow/flow-go-sdk/blob/master/block.go)
		
		您可以使用 SDK 查询历史数据和获取合约代码
	- 脚本

		Flow 允许您使用以 Cadence 编程语言编写的脚本主动查询区块链的状态。您可以在此处了解有关 Cadence 的更多信息：
		
		[https://docs.onflow.org/cadence](https://docs.onflow.org/cadence)
		
		SDK 支持运行脚本并解析其输出。
		
		脚本可以访问多个合同和账户，计算价值，并使用 Cadence 的类型系统确保数据正确。他们只能访问区块链的当前状态。
	- 使用 SDK
		- JavaScript
		
			您可以在此处下载 JavaScript SDK：
		
			[https://github.com/onflow/flow-js-sdk](https://github.com/onflow/flow-js-sdk)
		
			要连接到访问节点，您需要提供 SDK 的 URL。 
			
			- 测试网：https://access-testnet.onflow.org 
			- 主网：https : //access.onflow.org
			
			下面是一些例子
			
			- 以下是在此页面底部查询访问节点的示例：
		
				[https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk](https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk)
			- 以下是创建脚本和 Cadence 数据类型的示例：
		
				[https://github.com/onflow/flow-js-sdk/tree/master/packages/types#scripts](https://github.com/onflow/flow-js-sdk/tree/master/packages/types#scripts)
			- 以下是解析脚本输出的示例：
		
				[https://github.com/onflow/flow-js-sdk/tree/master/packages/decode](https://github.com/onflow/flow-js-sdk/tree/master/packages/decode)
		- golang 
		
			您可以在此处下载 Go SDK：
			
			[https://github.com/onflow/flow-go-sdk](https://github.com/onflow/flow-go-sdk)
			
			要连接到访问节点，您需要提供 SDK 的 URL。
			
			- 测试网：access.devnet.nodes.onflow.org:9000 
			- 主网：access.mainnet.nodes.onflow.org:9000
			
			下面是一些例子
			
			- 您可以在本页底部找到查询访问节点的示例：
			
				[https://github.com/onflow/flow-go-sdk/blob/master/README.md](https://github.com/onflow/flow-go-sdk/blob/master/README.md)
			- 您可以在此处找到使用仿真器的示例，该仿真器可适用于使用访问节点：
			
				[https://github.com/onflow/flow-go-sdk/blob/master/examples/query_events/main.go](https://github.com/onflow/flow-go-sdk/blob/master/README.md)		
			- 这是使用脚本的示例：
	
				[https://github.com/onflow/flow-go-sdk#executing-a-script](https://github.com/onflow/flow-go-sdk#executing-a-script)
		- 其他编程语言
		
			有关使用另一种编程语言实现接入节点通信的示例，请参阅 Daniel Podaru 的“使用 Ruby 与 Flow 交互”：[https ://www.onflow.org/post/interact-with-flow-using-ruby](https://www.onflow.org/post/interact-with-flow-using-ruby)
- 应用如何消费事件？事件如何运作？

	流交易可以发出信息“事件”，其中包含供链下观察者使用的数据。例如，事件可用于触发后端或 UI 事件。
	
	请注意，单个事务可能会发出许多事件，如果使用的是非标准事务，事件的顺序可能会让您感到惊讶。事件参数可能是可选的，这意味着它们在某些情况下可能为零。所有这些都意味着您在解析事件时必须小心。
	
	- 定义事件
	
		事件是使用 Cadence 编程语言在 Flow 智能合约中实现的。
		
		您可以在此处找到有关 Cadence 活动的更多信息:[https://docs.onflow.org/cadence/language/events/](https://docs.onflow.org/cadence/language/events/)
		
		作为事件可以包含的信息类型的示例，请参阅抵押协议发出的事件的文档：[https://docs.onflow.org/staking/events](https://docs.onflow.org/staking/events)
	- 消费事件
	
		要使用事件，您必须查询流访问节点并指定您希望从中获取这些事件的事件类型和块范围。然后，您可以解析任何返回的事件并处理它们包含的信息。
		
		- 使用 Go
		
			使用 Go SDK 连接到访问节点后，您可以查询事件。
		
			这是记录在这里：[https://github.com/onflow/flow-go-sdk#querying-events](https://github.com/onflow/flow-go-sdk#querying-events)
		
			这里有一个用法示例：[https://github.com/onflow/flow-go-sdk/tree/master/examples/query_events](https://github.com/onflow/flow-go-sdk/tree/master/examples/query_events)
		- 使用 JavaScript

			使用 JavaScript SDK 连接到访问节点后，您可以查询事件。
		
			这是记录在这里：[https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk#getevents-usage](https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk#getevents-usage) 
			
			这里有一个使用示例：[https://github.com/onflow/kitty-items/tree/master/kitty-items-js/src/workers](https://github.com/onflow/kitty-items/tree/master/kitty-items-js/src/workers)
		- 使用其他编程语言

			目前，我们只为 Flow 提供 Go 和 JavaScript SDK。有关使用另一种编程语言访问 Flow 区块链并从中消费事件的示例，请参阅 Daniel Podaru 的“使用 Ruby 与 Flow 交互”：[https://www.onflow.org/post/interact-with-flow-using-ruby](https://www.onflow.org/post/interact-with-flow-using-ruby)
- 什么是整箱？

	Flow 客户端库 (FCL) 使应用程序能够轻松地与所有兼容 FCL 的钱包和其他服务（例如配置文件、私人信息、通知）集成。这为开发人员提供了一个强大的基础，可以用现有的构建块来组合他们的应用程序。FCL 目前支持浏览器应用，并将扩展到其他平台。
	
	使用 FCL，您可以：
	
	- 集成所有兼容钱包，无需任何自定义代码或代码注入
	- 验证用户
	- 查询用户的Flow账号
	- 发送交易（例如初始化资源、发送资产、购买等）
	- 通过钱包集成签署交易以避免密钥管理（特别有助于修女托管应用程序）
- 我如何访问主网？

	您可以在 [https://docs.onflow.org/access-api](https://docs.onflow.org/access-api) 找到有关访问 Flow 主网的详细信息。
	
	访问是通过 GRPC 接口进行的。交易也可以使用 GRPC [SendTransaction](https://docs.onflow.org/access-api#sendtransaction) 调用提交。用任何语言编写的 GRPC 客户端都应该能够通过 Access API 进行对话。我们有Go SDK和JavaScript SDK，如果您分别使用基于 Go 或基于 JS 的客户端，则可以使用它们。
- 主网 spork 和测试网 spork 有什么区别？sporks 如何影响 TopShot？

	主网和测试网是两个独立的网络。不同之处在于哪个网络将经历停机时间。由于 NBA TopShot 的主要应用存在于主网上，因此只有 Mainnet sporks 会影响主要应用的正常运行时间。
	
	目前 NBA Top Shot 在主网上运行，NBA Top Shot 测试应用在 Flow Testnet 上运行
- Flow 的 Gas 费用是多少？

	暂时，Flow 上的交易是免费的——节点运营商通过网络发放的奖励获得补偿，以确保您的交易得到正确处理。将来，您可能自己支付交易费用，但更有可能的是 dapp 或钱包会为您支付这些费用。Flow 支持除您之外的其他人为您的交易付款（如果您选择其他方式，您将获得 FLOW 并确实支付费用）。这些费用用于防止垃圾邮件，并且在大多数交易中都很低且固定。
- 开发人员什么时候可以在主网上部署？如何为主网创建我的第一个帐户？

	在测试期间，您需要将您的合同提交给 Flow 团队进行审核，然后您就可以将其提交给网络。一旦您在 devnet 上测试了您的 dapp，并且 Flow 团队审查了您的合同，您就可以部署了！
	
	信息在这里：[https://docs.onflow.org/dapp-deployment](https://docs.onflow.org/dapp-deployment)
- 如何部署合约？

	在测试期间，您需要将您的合同提交给 Flow 团队进行审核，然后您就可以将其提交给网络。
	
	此处的智能合约审查链接：[https://buildwithflow.typeform.com/to/EkfaboAx](https://buildwithflow.typeform.com/to/EkfaboAx)
- 有测试网/开发网吗？

	在 testnet/devnet 上有一个供您开发的访问节点。您可以在[此处](https://docs.onflow.org/dapp-deployment/testnet-deployment#accessing-flow-testnet)了解更多信息
- 有公共节点吗？

	是的，访问节点可以公开访问以提交交易并从区块链读取数据。如果您想访问 devnet 访问节点进行构建，您可以在[此处](https://docs.onflow.org/concepts/accessing-testnet)进行
- 是否可以将多个公钥添加到给定的帐户/地址，以便它可以由多个私钥控制？

	是的，帐户支持多个加权键，[这里](https://docs.onflow.org/cadence/language/accounts) 使用 AuthAccount's  `fun addPublicKey(_ publicKey: [UInt8])`和 ` fun removePublicKey(_ index: Int)`函数。
- 密钥和帐户如何在 Flow 上工作？

	帐户是使用关联的密钥创建的。一个帐户上可以有多个密钥。要从账户执行交易，总共需要签署 1000 个权重密钥。该帐户有一个 FLOW 余额字段。当交易流动时，该余额由协议更新。该帐户还用于存放存储和合约代码。
	
	FLOW 支持多种签名方案来为账户添加密钥。
	
	详情：[https : //docs.onflow.org/concepts/accounts-and-keys](https://docs.onflow.org/concepts/accounts-and-keys)
- 如何查看账户拥有哪些 Fungible-Tokens？

	如果您知道特定 FT 保险库类型的 FungibleToken.Balance 功能的 /public/ 存储路径，您可以借用它并检查其余额。如果您想知道一个帐户有哪些保管库，您目前必须检查已知路径的列表。[有一个问题可能在未来对此有所帮助](https://github.com/onflow/cadence/issues/208)
- 如何查看账户拥有哪些资源？

	Flow 尚未提供检查帐户上所有资源的功能，但可以执行 Cadence 脚本来检查已知存储路径上的资源。
- 如果我没有服务账户，如何创建 Flow 账户？

	生成地址的说明在[这里](https://docs.onflow.org/flow-go-sdk/creating-accounts)。您不需要服务帐户。
- 是否有关于如何访问流测试网的教程？从头开始，获取测试网、Flow 令牌等......？

	点击查看[这里](https://docs.onflow.org/dapp-deployment/testnet-deployment/)
- 您可以查询块范围之间的事件吗？

	您可以查询访问 API 以获取块范围的事件。请参阅[此处](https://docs.onflow.org/access-api)的访问 API 规范
- 我可以在哪里关注 Flow 上的功能发布？

	您可以在[此处](https://github.com/onflow/flow-go/releases)关注 Flow 节点软件版本：。

## 流用户/支持者
- Flow 有区块浏览器吗？

	今天有两个区块浏览器。你可以在这里找到它们：
	
	- [https://flowscan.org/](https://flowscan.org/)
	- [https://flow.bigdipper.live/](https://flow.bigdipper.live/)
- 我下注了，现在我看不到我的流量 - 发生了什么？

	一旦您成功完成质押或委托请求，您的代币就会发送到质押合约。您的代币不会丢失 - 它们已被质押！要查看您的活跃权益或委托，请导航到 Flow Port 上的权益和委托页面（左侧栏，或此 URL + 最后一个斜杠后的地址 ( [https://port.onflow.org/stake-delegate/](https://port.onflow.org/stake-delegate/) )
- 是否可以将多个公钥添加到给定的帐户/地址，以便它可以由多个私钥控制？

	是的，帐户支持多个加权键，[这里](https://docs.onflow.org/cadence/language/accounts) 使用 AuthAccount's  `fun addPublicKey(_ publicKey: [UInt8])`和 
`fun removePublicKey(_ index: Int) `函数。
- 密钥和帐户如何在 Flow 上工作？

	帐户是使用关联的密钥创建的。一个帐户上可以有多个密钥。要从账户执行交易，总共需要签署 1000 个权重密钥。该帐户有一个 FLOW 余额字段。当交易流动时，该余额由协议更新。该帐户还用于存放存储和合约代码。

	FLOW 支持多种签名方案来为账户添加密钥。[详情](https://docs.onflow.org/concepts/accounts-and-keys)
- 如果我没有服务账户，如何创建 Flow 账户？

	生成地址的说明在[这里](https://docs.onflow.org/flow-go-sdk/creating-accounts)。您不需要服务帐户。

## Flow 操作者
- 在可预见的未来，预期的 TPS（每秒交易数）是多少？

	并非所有交易都是平等的，因此仅凭吞吐量数字并不能说明全部情况。Flow 的交易吞吐量取决于多种因素，例如节点可用的网络带宽和我们实现中的优化。主要因素是交易复杂性，受交易执行的指令数量、交易执行的分类账读取和写入数量以及必须检查多少签名以确认您的交易具有所需权限的影响。
	
	对于简单令牌余额转移的基准负载（基本上是加法和减法加上两次签名验证），我们当前的实现轻松维持了显着超过 100/tps 的吞吐量。然而，我们观察到主网上的交易通常要复杂得多（需要多次签名验证；许多账本读写；以及运行相对复杂的计算）。对于近期的未来，100/tps 似乎是一个现实的量级。我们正在努力维持目前在主网上运行的那种 100/tps。如果您的交易比普通的 Mainnet 交易简单得多，Flow 可能已经满足您所需的吞吐量。找出答案的最佳方法是在 TestNet 上测试特定交易的基准集。
- 区块高度每秒增加 1 次吗？

	Flow 的目标是 1 秒的区块时间，但该协议仍处于开发初期，需要进一步优化来实现这一目标。截至 2021 年 2 月，主网上的区块终结率为 0.4 个区块/秒；标准偏差为 ±0.1 个块/秒。因此，平均每 2.5 秒确定一个新区块。请注意，块高度与时间仅具有松散的相关性，因为块率自然会波动。
- Flow 有区块浏览器吗？

	今天有两个区块浏览器。你可以在这里找到它们：
		
	- [https://flowscan.org/](https://flowscan.org/)
	- [https://flow.bigdipper.live/](https://flow.bigdipper.live/)
- 如何连接和查询接入节点？有哪些可用数据？

	在协议层面，您可以通过 GRPC 连接到接入节点。我们提供 JavaScript 和 Golang SDK 来为您执行此操作。

	连接到访问节点后，您可以获取有关帐户、合约、区块、集合、交易和事件的信息。您还可以执行脚本来查询 Flow 区块链的当前状态。

	- 可用数据

		账户、合约、区块、集合、交易、事件，这些类型记录在 Access API 页面上。
		
		SDK 将 Flow 数据公开为类型。例如，可以在此处找到 Go Flow Block 数据类型实现：
		
		[https://github.com/onflow/flow-go-sdk/blob/master/block.go](https://github.com/onflow/flow-go-sdk/blob/master/block.go)
		
		您可以使用 SDK 查询历史数据和获取合约代码
	- 脚本

		Flow 允许您使用以 Cadence 编程语言编写的脚本主动查询区块链的状态。您可以在此处了解有关 Cadence 的更多信息：
		
		[https://docs.onflow.org/cadence](https://docs.onflow.org/cadence)
		
		SDK 支持运行脚本并解析其输出。
		
		脚本可以访问多个合同和账户，计算价值，并使用 Cadence 的类型系统确保数据正确。他们只能访问区块链的当前状态。
	- 使用 SDK
		- JavaScript
		
			您可以在此处下载 JavaScript SDK：
		
			[https://github.com/onflow/flow-js-sdk](https://github.com/onflow/flow-js-sdk)
		
			要连接到访问节点，您需要提供 SDK 的 URL。 
			
			- 测试网：https://access-testnet.onflow.org 
			- 主网：https : //access.onflow.org
			
			下面是一些例子
			
			- 以下是在此页面底部查询访问节点的示例：
		
				[https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk](https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk)
			- 以下是创建脚本和 Cadence 数据类型的示例：
		
				[https://github.com/onflow/flow-js-sdk/tree/master/packages/types#scripts](https://github.com/onflow/flow-js-sdk/tree/master/packages/types#scripts)
			- 以下是解析脚本输出的示例：
		
				[https://github.com/onflow/flow-js-sdk/tree/master/packages/decode](https://github.com/onflow/flow-js-sdk/tree/master/packages/decode)
		- golang 
		
			您可以在此处下载 Go SDK：
			
			[https://github.com/onflow/flow-go-sdk](https://github.com/onflow/flow-go-sdk)
			
			要连接到访问节点，您需要提供 SDK 的 URL。
			
			- 测试网：access.devnet.nodes.onflow.org:9000 
			- 主网：access.mainnet.nodes.onflow.org:9000
			
			下面是一些例子
			
			- 您可以在本页底部找到查询访问节点的示例：
			
				[https://github.com/onflow/flow-go-sdk/blob/master/README.md](https://github.com/onflow/flow-go-sdk/blob/master/README.md)
			- 您可以在此处找到使用仿真器的示例，该仿真器可适用于使用访问节点：
			
				[https://github.com/onflow/flow-go-sdk/blob/master/examples/query_events/main.go](https://github.com/onflow/flow-go-sdk/blob/master/README.md)		
			- 这是使用脚本的示例：
	
				[https://github.com/onflow/flow-go-sdk#executing-a-script](https://github.com/onflow/flow-go-sdk#executing-a-script)
		- 其他编程语言
		
			有关使用另一种编程语言实现接入节点通信的示例，请参阅 Daniel Podaru 的“使用 Ruby 与 Flow 交互”：[https ://www.onflow.org/post/interact-with-flow-using-ruby](https://www.onflow.org/post/interact-with-flow-using-ruby)
- 应用如何消费事件？事件如何运作？

	流交易可以发出信息“事件”，其中包含供链下观察者使用的数据。例如，事件可用于触发后端或 UI 事件。
	
	请注意，单个事务可能会发出许多事件，如果使用的是非标准事务，事件的顺序可能会让您感到惊讶。事件参数可能是可选的，这意味着它们在某些情况下可能为零。所有这些都意味着您在解析事件时必须小心。
	
	- 定义事件
	
		事件是使用 Cadence 编程语言在 Flow 智能合约中实现的。
		
		您可以在此处找到有关 Cadence 活动的更多信息:[https://docs.onflow.org/cadence/language/events/](https://docs.onflow.org/cadence/language/events/)
		
		作为事件可以包含的信息类型的示例，请参阅抵押协议发出的事件的文档：[https://docs.onflow.org/staking/events](https://docs.onflow.org/staking/events)
	- 消费事件
	
		要使用事件，您必须查询流访问节点并指定您希望从中获取这些事件的事件类型和块范围。然后，您可以解析任何返回的事件并处理它们包含的信息。
		
		- 使用 Go
		
			使用 Go SDK 连接到访问节点后，您可以查询事件。
		
			这是记录在这里：[https://github.com/onflow/flow-go-sdk#querying-events](https://github.com/onflow/flow-go-sdk#querying-events)
		
			这里有一个用法示例：[https://github.com/onflow/flow-go-sdk/tree/master/examples/query_events](https://github.com/onflow/flow-go-sdk/tree/master/examples/query_events)
		- 使用 JavaScript

			使用 JavaScript SDK 连接到访问节点后，您可以查询事件。
		
			这是记录在这里：[https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk#getevents-usage](https://github.com/onflow/flow-js-sdk/tree/master/packages/sdk#getevents-usage) 
			
			这里有一个使用示例：[https://github.com/onflow/kitty-items/tree/master/kitty-items-js/src/workers](https://github.com/onflow/kitty-items/tree/master/kitty-items-js/src/workers)
		- 使用其他编程语言

			目前，我们只为 Flow 提供 Go 和 JavaScript SDK。有关使用另一种编程语言访问 Flow 区块链并从中消费事件的示例，请参阅 Daniel Podaru 的“使用 Ruby 与 Flow 交互”：[https://www.onflow.org/post/interact-with-flow-using-ruby](https://www.onflow.org/post/interact-with-flow-using-ruby)
- 我下注了，现在我看不到我的流量 - 发生了什么？

	一旦您成功完成质押或委托请求，您的代币就会发送到质押合约。您的代币不会丢失 - 它们已被质押！要查看您的活跃权益或委托，请导航到 Flow Port 上的权益和委托页面（左侧栏，或此 URL + 最后一个斜杠后的地址 ( [https://port.onflow.org/stake-delegate/](https://port.onflow.org/stake-delegate/) )
- 当我收到此错误时该怎么办：节点启动问题 - 无法处理块提议：协议状态的无效扩展？

	听起来您没有擦除 `data` 文件夹？关闭节点，删除 `data` 文件夹并重新启动节点。
- 我的节点运行时可以看到哪些错误？

	没有错误应该被认为是可以接受的。如果有不断重复出现的情况，请提请我们注意，如果我们确定它们不是真正的错误，我们将调整日志以反映。

- 有测试网/开发网吗？

	在 testnet/devnet 上有一个供您开发的访问节点。您可以[在此处](https://docs.onflow.org/dapp-deployment/testnet-deployment#accessing-flow-testnet)了解更多信息
- 有公共节点吗？

	是的，访问节点可以公开访问以提交交易并从区块链读取数据。如果您想访问 devnet 访问节点进行构建，您可以[在此处](https://docs.onflow.org/concepts/accessing-testnet)进行
- 是否可以将多个公钥添加到给定的帐户/地址，以便它可以由多个私钥控制？

	是的，帐户支持多个加权键，这里 使用 AuthAccount's `fun addPublicKey(_ publicKey: [UInt8])`和 `fun removePublicKey(_ index: Int)` 函数。
- 密钥和帐户如何在 Flow 上工作？

	帐户是使用关联的密钥创建的。一个帐户上可以有多个密钥。要从账户执行交易，总共需要签署 1000 个权重密钥。该帐户有一个 FLOW 余额字段。当交易流动时，该余额由协议更新。该帐户还用于存放存储和合约代码。

	FLOW 支持多种签名方案来为账户添加密钥。[详情](https://docs.onflow.org/concepts/accounts-and-keys)
- 如果我没有服务账户，如何创建 Flow 账户？

	生成地址的说明[在这里](https://docs.onflow.org/flow-go-sdk/creating-accounts)。您不需要服务帐户。
- 您可以查询块范围之间的事件吗？

	您可以查询访问 API 以获取块范围的事件。请参阅[此处](https://docs.onflow.org/access-api)的访问 API 规范
- 我可以在哪里关注 Flow 上的功能发布？

	您可以在[此处](https://github.com/onflow/flow-go/releases)关注 Flow 节点软件版本：。					