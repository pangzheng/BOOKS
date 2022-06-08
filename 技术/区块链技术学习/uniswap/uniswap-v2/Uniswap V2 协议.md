# Uniswap V2 协议-概述
## 概念
- 开发者

	V2 Uniswap 协议分为两个存储库

	- [uniswap-v2-core](https://github.com/Uniswap/uniswap-v2-core)
	- [uniswap-v2-periphery](https://github.com/Uniswap/uniswap-v2-periphery)

	可以在此处找到 V2 SDK，它可以在与 Uniswap V2 协议交互时协助开发人员。

	- [uniswap-sdk](https://github.com/Uniswap/uniswap-v2-sdk)
	- [uniswap-sdk-core](https://github.com/Uniswap/uniswap-sdk-core)

### 协议概念
#### 工作原理
![](./pic/v2protocol.jpeg)
Uniswap 是一种自动流动性协议，由 [恒定乘积](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/glossary#constant-product-formula) 公式提供支持，并在[以太坊区块链](https://ethereum.org/)上的`不可升级智能合约`系统中实施。它消除了对可信中介的需求，优先考虑去中心化、抗审查和安全。Uniswap 是根据 [GPL许可](https://en.wikipedia.org/wiki/GNU_General_Public_License) 的开源软件。

每个 Uniswap 智能合约或配对都管理一个由两个 [ERC-20](https://eips.ethereum.org/EIPS/eip-20) Token储备金组成的流动资金池。

任何人都可以通过存入每个基础Token的等值来换取池Token，从而成为池的流动性提供者 (LP)。这些Token按比例跟踪总储备金中的 LP 份额，并可随时赎回基础资产。
	
![](./pic/v2protocol1.jpeg)
	
Token对充当自动做市商，只要保留 `恒定乘积` 公式，就准备好接受一个Token换另一个Token。该公式最简单地表示为 `x * y = k`，表明交易不得改变 `k` 一对储备金余额 (`x`) 和(`y`) 的乘积。因为 `k` 与交易的参考框架保持不变，它通常被称为不变量。该公式具有理想的特性，即较大的交易（相对于储备金额）以比较小的交易更差的指数执行。

在实践中，Uniswap 对交易收取 0.30% 的费用，该费用被添加到准备金中。结果，每笔交易实际上都在增加 `k`。这起到了向 LP 支付的作用，当他们烧掉流动性池中的Token以提取他们在总储备金中的一部分时，就会实现这一点。将来，此费用可能会降低到 0.25%，剩余的 0.05% 作为协议范围内的费用扣留。

![](./pic/v2protocol2.jpeg)
由于两对资产的相对价格只能通过交易来改变，Uniswap 价格与外部价格的背离创造了套利机会。这种机制确保 Uniswap 价格始终趋向于市场出清价格。
- 进一步阅读

	要了解Token交换在实践中是如何工作的，并了解交换的生命周期，请查看[交换](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/swaps)。或者，要了解流动资金池的工作原理，请参阅[Pools](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/pools)。
	
	当然，归根结底，Uniswap 协议只是在以太坊上运行的智能合约代码。要了解它们的工作原理，请前往[智能合约](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory)。
#### 生态参与者
![](./pic/v2protocol3.jpeg)
Uniswap 生态系统主要由三类用户组成：

- 流动性提供者

	流动性提供者被激励向公共流动性池贡献 [ERC-20](https://eips.ethereum.org/EIPS/eip-20) Token。
- 交易者

	交易者可以以固定的 0.30% 的[费用](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/fees)将这些Token相互交换（这将支付给流动性提供者）。
- 开发者

	开发人员可以直接与 Uniswap 智能合约集成，以支持与Token、交易界面、零售体验等的新的和令人兴奋的交互。

总的来说，这些类别之间的互动创造了一个积极的反馈循环，通过定义一种可以汇集、交易和使用Token的通用语言来推动数字经济。

##### 流动性提供者
流动性提供者或 LP 不是同质群体：

- 被动 LP 是希望被动投资资产以积累交易费用的Token持有者。
- 专业的有限合伙人专注于做市作为他们的主要策略。他们通常开发定制工具和方法来跟踪他们在不同 DeFi 项目中的流动性头寸。
- Token项目有时会选择成为 LP 为其Token创建一个流动的市场。这使得Token买卖更容易，并通过 Uniswap 解锁与其他 DeFi 项目的互操作性。
- 最后，一些 DeFi 先驱正在探索复杂的流动性提供互动，例如激励流动性、流动性作为抵押品以及其他实验性策略。

Uniswap 是项目尝试这些想法的完美协议。

##### 交易员
协议生态系统中有几类交易者：

- 投机者使用各种社区构建的工具和产品，利用从 Uniswap 协议中提取的流动性来交换Token。
- 套利机器人通过比较不同平台的价格以寻找优势来寻求利润。（虽然看起来很榨取，但这些机器人实际上有助于平衡更广泛的以太坊市场的价格并保持公平。）
- DAPP 用户在 Uniswap 上购买Token，用于以太坊上的其他应用程序。
- 通过实现交换功能（从 DEX 聚合器等产品到自定义 Solidity 脚本）在协议上执行交易的智能合约。

在所有情况下，根据协议进行的交易均需支付相同的固定费用。每一项对于提高价格的准确性和激励流动性都很重要。

##### 开发商/项目
在更广泛的以太坊生态系统中使用 Uniswap 的方式太多了，但一些例子包括：

- Uniswap 的开源、可访问性意味着有无数的 UX 实验和前端旨在提供对 Uniswap 功能的访问。您可以在大多数主要的 DeFi 仪表板项目中找到 Uniswap 功能。社区还构建了许多特定于 [Uniswap 的工具](https://github.com/Uniswap/universe)。
- 钱包通常将交换和流动性提供功能作为其产品的核心产品。
- DEX（去中心化Exchange）聚合器从许多流动性协议中提取流动性，通过拆分交易为交易者提供最优惠的价格。Uniswap 是这些项目最大的单一去中心化流动性来源。
- 智能合约开发人员使用可用的功能套件来发明新的 DeFi 工具和其他各种实验性想法。查看 [Unisocks](https://unisocks.exchange/) 或 [Zora](https://ourzora.com/) 等项目，其中包括许多其他项目。

##### Uniswap 团队和社区
Uniswap 团队与更广泛的 Uniswap 社区一起推动协议和生态系统的发展。

#### 智能合约
Uniswap V2 是一个二进制智能合约系统。核心合约为与 Uniswap 交互的各方提供基本的安全保障。外围合约与一个或多个核心合约交互，但它们本身不是核心的一部分。
##### Core
- [源代码](https://github.com/Uniswap/uniswap-v2-core)

	核心由一个单例[工厂](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/smart-contracts#factory)和许多 [对](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/smart-contracts#pairs) 组成，工厂负责创建和索引。这些合约非常少，甚至是野蛮的。这样做的简单理由是: 具有较小表面积的合约更容易推理，更不容易出错，并且在功能上更优雅。也许这种设计的最大优点是系统的许多所需属性可以直接在代码中声明，几乎没有出错的余地。然而，一个缺点是核心合约在某种程度上对用户不友好。事实上，对于大多数用例，不建议直接与这些合约交互。相反，应该使用外围合约。

	- [工厂](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/smart-contracts#factory)
	
		[参考文档](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory)
	
		工厂持有负责为对供电的通用字节码。它的主要工作是为每个独特的Token对创建一个且只有一个智能合约。它还包含开启协议收费的逻辑。
	- [对](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/smart-contracts#pairs)
		- [参考文档](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair)
		- [参考文档 (ERC-20)](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20)

		Token对有两个主要目的：
		
		- 充当自动做市商和跟踪池Token余额。
		- 它们还公开可用于构建去中心化价格预言机的数据。
- Periphery

	[源代码](https://github.com/Uniswap/uniswap-v2-periphery)

	Periphery 是一组智能合约，旨在支持与核心的特定领域交互。由于 Uniswap 的无许可性质，下面描述的合约没有特权，实际上只是可能的外围类合约宇宙的一小部分。但是，它们是如何安全有效地与 Uniswap V2 交互的有用示例。
	
	- [Library](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/smart-contracts#library)
	
		[参考文档](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library)
	
		该库为获取数据和定价提供了各种便利功能。
	- [router](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/smart-contracts#router)
	
		[参考文档](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02)
	
		使用该库的路由器完全支持前端提供交易和流动性管理功能的所有基本要求。值得注意的是，它原生支持多对交易（例如 x 到 y 到 z），将 ETH 视为一等公民，并提供用于消除流动性的元交易。

##### 设计决策
以下部分描述了 Uniswap V2 中做出的一些值得注意的设计决策。除非您有兴趣深入了解 V2 的工作原理或编写智能合约集成，否则这些内容可以安全跳过！

- 发送 Token 
	- 通常，需要Token来执行某些功能的智能合约要求潜在的交互者首先对Token合约进行批准授权
	- 然后调用一个函数，该函数又调用Token合约上的 transferFrom。这不是 V2 对接受Token的方式。相反，配对在每次交互结束时检查他们的Token余额
	- 然后，在下一次交互开始时，当前余额与存储的值不同，以确定当前交互者发送的Token数量。

	请参阅[白皮书](https://docs.uniswap.org/whitepaper.pdf)以了解为什么会出现这种情况的理由，但要点是在调用任何需要Token的方法之前必须将Token转移到该Token对,此规则的一个例外是 [Flash Swaps](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/flash-swaps)。
- WETH

	与 Uniswap V1 矿池不同，V2 对不直接支持 ETH，因此必须用 WETH 模拟 ETH⇄ERC-20 对。这种选择背后的动机是删除核心中特定于 ETH 的代码，从而产生更精简的代码库。然而，最终用户可以完全不了解这个实现细节，只需在外围包装/解包 ETH。

	路由器完全支持通过 ETH 与任何 WETH 对进行交互。

- 最低流动性

	为了改善舍入误差并增加流动性提供的理论最小刻度大小，Token对会销毁第一个 [MINIMUM_LIQUIDITY](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#minimum_liquidity) 池Token。对于绝大多数对，这将代表一个微不足道的价值。在第一次提供流动性期间，销毁会自动发生，之后总供应量将永远受到限制。
	
#### 词汇表
- Automated market maker(自动市商)

	自动做市商是以太坊上的智能合约，持有链上流动性储备。用户可以以自动做市公式设定的价格与这些储备金进行交易。
- Constant product formula(恒定乘积)

	Uniswap 使用的自动做市算法。见 [x * y=k](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/glossary#x--y--k)
- ERC20

	ERC20 Token是以太坊上的可替代Token。Uniswap 支持所有标准的 ERC20 实现
- Factory(工厂)

	为任何 ERC20/ ERC20 交易对部署独特智能合约的智能合约
- Pair(对)

	从 Uniswap V2 工厂部署的智能合约，可以在两个 ERC20 Token之间进行交易。
- Pool(池)

	一对内的流动性汇集在所有流动性提供者
- Liquidity provider / LP(流动性提供者)

	流动性提供者是将等值的两个 ERC20 Token存入一对流动性池中的人。流动性提供者承担价格风险并获得费用补偿。
- Mid price(中间价格)

	用户在给定时刻可以买卖Token的价格。在 Uniswap 中，这是两个 ERC20 Token储备的比率。
- Price impact(价格影响)

	中间价与交易执行价之间的差额。
- Slippage(滑点)

	在交易提交和执行之间，价格在交易对中的变动量。
- Core(内核)

	Uniswap 存在必不可少的智能合约。升级到新版本的核心将需要流动性迁移。
- Periphery(周边)

	有用的外部智能合约，但 Uniswap 不需要存在。新的外围合约总是可以在不迁移流动性的情况下部署。
- Flash swap(闪电交换)

	在付款之前使用所购买Token的交易。
- x * y = k

	恒定乘积公式。
- Invariant(恒定)

	恒定乘积公式中的“k”值

### 核心概念
#### Swaps
![](./pic/v2protocol2.jpeg)

- 介绍

	Uniswap 中的Token交换是一种将一个 ERC-20 Token换成另一个Token的简单方法。

	对于最终用户来说，交换是直观的：用户选择一个输入Token和一个输出Token。他们指定输入金额，协议计算他们将收到多少输出Token。然后他们一键执行交换，立即在钱包中收到输出Token。

	在本指南中，我们将在协议级别查看交换期间发生的情况，以便更深入地了解 Uniswap 的工作原理。

	Uniswap 中的交换交易不同于传统平台上的交易。Uniswap 不使用订单簿来代表流动性或确定价格。Uniswap 使用自动做市商机制提供有关利率和滑点的即时反馈。

	正如我们在[协议概述](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/how-uniswap-works)中了解到的，Uniswap 上的每一对实际上都由一个流动资金池支撑。流动资金池是智能合约，持有两种独特Token的余额，并执行有关存取它们的规则。

	这个规则就是[恒定乘积](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/glossary#constant-product-formula)公式。当任一Token被提取（购买）时，必须按比例存入（出售）另一个Token，以保持不变。

	- swap 剖析

		在最基本的层面上，Uniswap V2 中的所有交换都发生在一个函数中，恰当地命名为 swap：
		
			function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data);
- 接收Token

	从函数签名中可能很清楚，Uniswap 要求 `swap` 调用者通过参数指定他们希望接收多少个输出Token `amount{0,1}Out`，这些参数对应于所需的`token{0,1}`.

- 发送Token

	尚不清楚的是 Uniswap 如何接收Token作为交换的付款。通常，需要Token来执行某些功能的智能合约要求调用者首先对Token合约进行批准，然后调用一个函数，该函数依次调用Token合约上的 `transferFrom`。这不是 V2 对接受Token的方式。相反，配对在每次交互结束时检查他们的Token余额。然后，在下一次交互开始时，当前余额与存储的值不同，以确定当前交互者发送的Token数量。请参阅[白皮书](https://docs.uniswap.org/whitepaper.pdf)了解为什么会这样。

	要点是必须在调用交换之前将Token转移到对（该规则的一个例外是 [Flash Swaps](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/flash-swaps)）。这意味着要安全地使用该 `swap` 函数，必须从另一个智能合约调用它。替代方案（将Token转移到该Token对然后调用 `swap`）以非原子方式执行是不安全的，因为发送的Token很容易受到套利。
- 开发者资源
	- 要了解如何在智能合约中实施Token交换，请阅读智能合约中的[交易](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/trading-from-a-smart-contract)。
	- 要查看如何从界面执行交换，请阅读交易 ([SDK](https://docs.uniswap.org/sdk/2.0.0/guides/trading))

#### Pool
- 介绍

	![](./pic/v2protocol.jpeg)	

	每个 Uniswap 流动资金池都是一对 ERC20 Token的交易场所。
	
	- 矿池合约创建时，其每个 token 的余额为 0；
	- 为了让矿池开始促进交易，必须有人用每个Token的初始存款作为种子。第一个流动性提供者是设定池初始价格的人。他们被激励将相等价值的两个Token存入池中。要了解原因，请考虑第一个流动性提供者以不同于当前市场利率的比率存入Token的情况。这立即创造了一个有利可图的套利机会，很可能被外部方利用。

	当其他流动性提供者添加到现有池中时，他们必须存入与当前价格成比例的配对Token。如果他们不这样做，他们增加的流动性也有被套利的风险。如果他们认为当前价格不正确，他们可能会将其套利到他们想要的水平，并以该价格增加流动性。 
- Pool Token
	
	![](./pic/v2protocol1.jpeg)	
	
	每当流动性存入池中时，都会铸造称为流动性Token的独特Token并将其发送到提供者的地址。这些Token代表给定流动性提供者对资金池的贡献。提供的资金池流动性的比例决定了提供者收到的流动性Token的数量。如果提供者正在铸造一个新池，他们将收到的流动性Token的数量将等于 `sqrt(x * y)`，其中 `x` 和 `y `代表提供的每个Token的数量。

	每当发生交易时都会向交易发送者收取 0.3% 的费用。这笔费用在交易完成后按比例分配给池中的所有 LP。

	为了取回潜在的流动性和产生的所有费用，流动性提供者必须“烧掉”他们的流动性Token，有效地将它们换成他们在流动性池中的一部分，以及按比例分配的费用。

	由于流动性Token本身就是可交易资产，流动性提供者可以以他们认为合适的任何方式出售、转让或以其他方式使用他们的流动性Token。

	通过高级主题了解更多信息：

	- [退货](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/understanding-returns)
	- [费用](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/fees)
- 为什么是 pool

	Uniswap 的独特之处在于它不使用订单簿来推导资产价格或匹配Token的买卖双方。相反，Uniswap 使用所谓的流动资金池。

	流动性通常由个人在集中操作的订单簿上下达的离散订单来表示。希望提供流动性或做市​​的参与者必须积极管理他们的订单，不断更新订单以响应市场上其他人的活动。

	虽然订单簿是财务的基础，并且在某些用例中非常有用，但它们受到一些重要限制的影响，这些限制在应用于去中心化或区块链原生环境时尤其明显。订单簿需要中介基础设施来托管订单簿并匹配订单。这会创建控制点并增加额外的复杂性。它们还需要做市商的积极参与和管理，这些做市商通常使用复杂的基础设施和算法，限制了高级交易者的参与。订单簿是在一个交易资产相对较少的世界中发明的，因此它们对于任何人都可以创建自己的Token的生态系统并不理想，而且这些Token通常流动性低，这并不奇怪。

	Uniswap 专注于以太坊的优势，从第一原则重新构想Token交换。

	区块链原生流动性协议应利用受信任的代码执行环境、自主且永久运行的虚拟机以及开放、无需许可和包容性的访问模型，从而产生指数级增长的虚拟资产生态系统。

	重要重申，矿池只是一个智能合约，由用户调用其上的函数来操作。交换Token调用 swap 池合约实例，同时提供流动性调用 deposit。

	最终用户如何通过接口与 Uniswap 协议交互（反过来与底层合约交互），开发人员可以直接与智能合约交互并将 Uniswap 功能集成到他们自己的应用程序中，而无需依赖中介或需要许可。

- 开发者资源

	要了解如何在智能合约中汇集Token，请阅读[提供流动性](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/providing-liquidity)。	
	
#### 闪电交换(Flash Swap )
Uniswap 闪电交换允许您提取 Uniswap 上任何 ERC20 Token的全部储备金，并免费执行任意逻辑，前提是在交易结束时您：

- 用相应的Token对支付提取的 ERC20 Token
- 退还提取的 ERC20 Token以及少量费用

Flash Swap 非常有用，因为它们避免了涉及 Uniswap 的多步交易的前期资本要求和不必要的操作顺序限制。

- 例子
	- 自由资本套利
	
		闪电交换的一个特别有趣的用例是自由资本套利。众所周知，Uniswap 设计的一个组成部分是激励套利者将 Uniswap 价格交易为“公平”的市场价格。虽然从博弈论上讲是合理的，但这种策略只有那些有足够资金来利用套利机会的人才能使用。闪电交换完全消除了这一障碍，有效地使套利民主化。
	
		想象一个场景，在 Uniswap 上购买 1 ETH 的成本是 200 DAI（通过调用 `getAmountIn` 指定 1 ETH 作为精确输出来计算），而在 Oasis（或任何其他 Exchange）上，1 ETH 购买 220 DAI。对于任何拥有 200 DAI 的人来说，这种情况代表了 20 DAI 的无风险利润。不幸的是，你可能没有 200 DAI。然而，通过闪电交换，只要有能力支付 Gas 费，任何人都可以获得这种无风险的利润。
		
		- ETH 退出 Uniswap
	
			第一步是通过闪电交换从 Uniswap 中`乐观`地提取 1 ETH。这将作为我们用来执行套利的资金。请注意，在这种情况下，我们假设：
	
			- 1 ETH 是预先计算的利润最大化交易
			- 自我们计算后 Uniswap 或 Oasis 的价格没有变化
	
			可能是我们想在执行时计算链上利润最大化的交易，这对价格变动是稳健的。这可能有点复杂，具体取决于正在执行的策略。然而，一种常见的策略是针对固定的外部价格进行尽可能有利可图的交易。（这个价格可能是 Oasis 上一个或多个订单的平均执行价格。）如果 Uniswap 市场价格远远高于或低于这个外部价格，以下示例包含计算通过 Uniswap 交易的最大金额的代码利润：[ExampleSwapToPrice.sol](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/examples/ExampleSwapToPrice.sol).
		- 对外场地交易
	
			一旦我们从 Uniswap 获得了 1 ETH 的临时资金，我们现在可以在 Oasis 上用 220 DAI 进行交易。收到 DAI 后通过 `getAmountIn`,需要偿还 Uniswap 已经提到的支付 1 ETH 所需的金额是 200 DAI。 因此，在将 200 个 DAI 发送回 Uniswap 对后，您将获得 20 个 DAI 的利润！
	- 即时利用
	
		闪电交换可用于提高使用借贷协议和 Uniswap 的杠杆效率。
		
		以最简单的形式考虑 Maker：一个接受 ETH 作为抵押品并允许用它铸造 DAI 的系统，同时确保 ETH 的价值永远不会低于 DAI 价值的 150%。
	
		假设我们使用该系统存入 3 ETH 的本金并铸造最大数量的 DAI。以 1 ETH / 200 DAI 的价格，我们收到 400 DAI。从理论上讲，我们可以通过出售 DAI 以获得更多 ETH、存入该 ETH、铸造最大数量的 DAI（这次会更少）并重复直到我们达到我们想要的杠杆水平来提高这一头寸。
	
		使用 Uniswap 作为该过程中 DAI 到 ETH 组件的流动性来源非常简单。然而，以这种方式循环协议并不是特别优雅，而且可能会消耗大量气体。
	
		幸运的是闪电交换使我们能够预先提取全部 ETH 金额。如果我们想要对我们的 3 ETH 本金进行 2 倍杠杆，我们可以简单地在闪兑中请求 3 ETH 并将 6 ETH 存入 Maker。这使我们能够铸造 800 DAI。如果我们铸造了我们需要的数量来覆盖我们的快速交换（比如 605），那么剩余的部分可以作为对抗价格变动的安全边际。
- 开发者资源
	
	要了解如何在您的智能合约中集成闪兑，请阅读使用[闪电交换](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/using-flash-swaps)。

#### 预言机
- 介绍

	价格预言机是用于查看给定资产价格信息的任何工具。当您在手机上查看股票价格时，您将手机用作价格预言机。同样，您手机上的应用程序依赖于设备来检索价格信息——可能有几个，这些信息被汇总然后显示给您，最终用户。这些也是价格预言。

	在构建与 DeFi 协议集成的智能合约时，开发者不可避免地会遇到价格预言问题。在链上检索给定资产价格的最佳方法是什么？

	以太坊上的许多预言机设计都是临时实施的，具有不同程度的去中心化和安全性。正因为如此，生态系统见证了许多引人注目的黑客攻击，其中预言机实现是主要的攻击媒介。此处讨论了其中一些漏洞。

	虽然没有一种万能的解决方案，但 Uniswap V2 使开发人员能够构建高度去中心化和抗操纵的链上价格预言机，这可以解决构建健壮协议所需的许多需求。
- Uniswap V2 解决方案

	Uniswap V2 包括多项改进，以支持抗操纵的公开价格馈送。
	
	- 首先，在任何交易发生之前，每对在每个区块开始时测量（但不存储）市场价格。这个价格操作起来很昂贵，因为它是由上一个区块中的最后一笔交易设定的，无论是铸币、交换还是销毁。

		要将测量的价格设置为与全球市场价格不同步的价格，攻击者必须在前一个区块结束时进行不良交易，通常无法保证他们会在下一个区块中套利。除非攻击者能够“自私地”连续开采两个区块，否则攻击者将蒙受套利者的损失。这种类型的攻击提出了几个挑战，迄今为止尚未观察到。

	- 不幸的是，仅此还不够。如果基于这种机制产生的价格结算了重大价值，则攻击的利润可能会超过损失。

		相反，Uniswap V2 将此块结束价格添加到核心合约中的单个累积价格变量中，该变量由该价格存在的时间量加权。该变量代表整个合约历史中每一秒的 Uniswap 价格的总和。
		
		![](./pic/v2protocol4.PNG)
		
		外部合约可以使用此变量来跟踪任何时间间隔内的准确`时间加权平均价格` (TWAP)。

		TWAP 是通过在所需间隔的开始和结束时读取 ERC20 Token对的累积价格来构建的。然后可以将此累积价格的差异除以间隔长度，以创建该时期的 TWAP。
		
		![](./pic/v2protocol5.PNG)
		
		TWAP 可以直接使用也可以根据需要作为移动平均线（EMA 和 SMA）的基础。

		几点注意事项：

		- 对于 10 分钟的 TWAP，每 10 分钟采样一次。对于 1 周的 TWAP，每周采样一次。
		- 对于简单的 TWAP，操纵成本会随着 Uniswap 上的流动性增加（近似线性），并且随着平均时间的长度增加（近似线性）。
		- 攻击成本的估计相对简单。在 1 小时 TWAP 上将价格移动 5% 大约等于在 1 小时内每个区块移动 5% 的套利损失和费用。

		将 Uniswap V2 用作预言机时，需要注意一些细微差别，尤其是在涉及操纵阻力时。[白皮书](https://docs.uniswap.org/whitepaper.pdf)详细阐述了其中的一些。其他以预言机为重点的开发人员指南和文档将很快发布。

		同时，查看我们基于 Uniswap V2 构建的 24 小时 TWAP Oracle的[示例实现](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/examples/ExampleOracleSimple.sol)！
- 操纵阻力

	在特定时间段内,操纵价格的成本可以粗略估计为整个期间每个区块的套利损失和费用。对于更大的流动资金池和更长的时间段，这种攻击是不切实际,因为操纵成本通常超过风险价值。

	其他因素，例如网络拥塞，可以降低攻击成本。如需更深入地了解 Uniswap V2 价格预言机的安全性，请阅读 [Oracle Integrity](https://uniswap.org/audit.html#org87c8b91) 的安全审计部分。
- 构建一个预言机

	要了解有关构建[预言机](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/building-an-oracle)的更多信息，请查看开发人员指南中的构建 Oracle。
	
### 高级主题
#### 费用
- 流动性提供者的费用

	交换Token需要 0.3% 的费用。该费用由流动性提供者按其对流动性储备金的贡献比例分配。

	交换费用立即存入流动性储备金。这增加了流动性Token的价值，作为对所有流动性提供者其在池中的份额成正比的支付。费用通过燃烧流动性Token来收取一定比例的基础储备金。

	由于费用被添加到流动性池中，因此在每笔交易结束时不变量都会增加。在单个事务中，不变量表示 `token0_pool/token1_pool` 上一个事务的结束。

	有许多社区开发的工具来确定回报。您还可以在文档中阅读有关如何考虑 [LP 回报](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/understanding-returns) 的更多信息。

- 协议费用

	目前没有协议费用。但是将来可能会开启 0.05% 的费用。有关潜在的未来协议费用的更多信息，请参见[此处](https://uniswap.org/blog/uniswap-v2/#path-to-sustainability)。

	未来，每笔交易 0.05% 的协议范围收费可能会生效。这代表 0.30% 费用的 ⅙ (16.6̅%)。如果 [feeTo](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory/#feeto) 不是 `address(0)( 0x0000000000000000000000000000000000000000)`，则费用有效，表明 `feeTo` 是费用的接收方。

	该金额不会影响交易者支付的费用，但会影响流动性提供者收到的金额。

	不计算交换费用，这会显着增加所有用户的 gas 成本，而是在增加或删除流动性时计算费用。有关更多详细信息，请参阅[白皮书](https://docs.uniswap.org/whitepaper.pdf)。

#### 价格
- 价格是如何确定的？

	正如我们在协议[概述](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/how-uniswap-works)中了解到 Uniswap 上的每一对实际上都由一个流动资金池支撑。流动资金池是智能合约，持有两种独特Token的余额，并执行有关存取它们的规则。主要规则是[恒定乘积公式](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/glossary#constant-product-formula)。当提取（购买）Token时，必须存入（出售）一定比例的金额以保持不变。池中Token的比率，结合恒定乘积公式，最终决定了交换执行的价格。
- Uniswap 如何处理价格

	在 Uniswap V1 中，交易始终以“可能的最佳”价格执行，在执行时计算。有点令人困惑的是这个计算实际上是用两个不同的公式之一完成的，这取决于交易是否指定了准确的输入或输出量。在功能上，这两个功能之间的差异是微乎其微的，但差异的存在增加了概念的复杂性。
	
	在 V2 中支持这两个功能的最初尝试被证明是不优雅的，因此决定不在核心中提供任何定价功能。相反，对在每次交易后直接检查不变量是否得到满足（考虑费用）。这意味着与其依赖定价功能强制执行不变量，V2 对简单而透明地确保它们自己的安全，很好地分离关注点。一个下游好处是 V2 对将更自然地支持可能出现的其他类型的交易（例如，在执行时以特定价格交易）。

	在高层次上，在 Uniswap V2 中，交易必须在外围定价。好消息是，该库提供了各种旨在简化此操作的功能，并且路由器中的所有交换功能都考虑到了这一点。
- 定价交易

	在 Uniswap 上交换Token时，通常希望接收尽可能多的输出Token以获得准确的输入金额，或者支付尽可能少的输入Token以获得准确的输出金额。为了计算这些金额，合约必须查看Token对的当前储备金，以了解当前价格是多少。但是，执行此查找并依赖结果而不访问外部价格是不安全的。

	假设一个智能合约天真地想向 DAI/WETH 对发送 10 个 DAI，并在给定当前准备金率的情况下接收尽可能多的 WETH。如果在调用时，幼稚的智能合约只是查看当前价格并执行交易，它很容易受到抢先交易的影响，并且可能会遭受经济损失。
	
	要了解原因，请考虑在确认交易之前看到该交易的恶意行为者。他们可以在乐观的交换通过之前立即执行一个大幅改变 DAI/WETH 价格的交换，等待乐观的交换以低利率执行，然后进行交换以将价格改回乐观的交换之前的价格。这种攻击相当便宜且风险低，通常可以为盈利而执行。

	为了防止这些类型的攻击，提交能够了解其交换应执行的“公平”价格的交换至关重要。换句话说，交换需要访问预言机，以确保他们可以从 Uniswap 获得的最佳执行与预言机认为的“真实”价格足够接近。
	
	虽然这听起来很复杂，但预言机可以像对当前市场价格的链下观察一样简单. 由于套利，通常情况下，一对区块内储备金的比率接近“真实”市场价格。因此，如果用户在提交交易时考虑到这一点，他们可以确保由于抢先交易而造成的损失是有限的。例如，这就是 Uniswap 前端确保交易安全的方式。它计算给定观察到的块内价格的最佳输入/输出量，并使用路由器执行交换，这保证交换将以不低于x观察到的块内速率的 % 执行，其中x用户- 指定的滑点容差（默认为 0.5%）。

	当然，预言机还有其他选项，包括[原生 V2 预言机](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/oracles)。

	- 精准输入

		如果您想发送准确数量的输入Token以换取尽可能多的输出Token，您将需要使用 [getAmountsOut](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#getamountout)。等效的 SDK 函数是 [getOutputAmount](https://docs.uniswap.org/sdk/2.0.0/reference/pair#getoutputamount) 或用于滑点计算的 [minimumAmountOut](https://docs.uniswap.org/sdk/2.0.0/reference/trade#minimumamountout-since-204)。

	- 精准输出

		如果您想以尽可能少的输入Token接收准确数量的输出Token，您将需要使用 [getAmountsIn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#getamountsin)。等效的 SDK 函数是 [getInputAmount](https://docs.uniswap.org/sdk/2.0.0/reference/pair#getinputamount) 或用于滑点计算的 [maximumAmountIn](https://docs.uniswap.org/sdk/2.0.0/reference/trade#maximumamountin-since-204)
	- 交换价格

		对于这个更高级的用例，请参阅 [ExampleSwapToPrice.sol](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/examples/ExampleSwapToPrice.sol)。
		
#### 了解退货
Uniswap 通过奖励提供者与其他用户与这些池进行交易时产生的费用来激励用户为交易池增加流动性。一般来说，做市是一项复杂的活动。与单纯持有资产相比，在标的资产价格大幅持续波动期间存在亏损风险。

- 风险

	要了解与提供流动性相关的风险，您可以阅读[https://medium.com/@pintail/uniswap-a-good-deal-for-liquidity-providers-104c0b6816f2以](https://medium.com/@pintail/uniswap-a-good-deal-for-liquidity-providers-104c0b6816f2)深入了解如何概念化流动性位置。

- 文章中的例子
	- 准备条件
		- 流动性提供者将 10,000 DAI 和 100 WETH 添加到池中（总价值为 20,000 美元）的情况
		- 流动性池现在总共是 100,000 DAI 和 1,000 ETH
		- 由于提供的数量等于总流动性的 10%，合约铸造并向做市商发送“流动性Token”，使他们有权获得池中可用流动性的 10%。
	
		这些不是要交易的投机Token。它们只是一种会计或簿记工具，用于跟踪流动性提供者的欠款。如果其他人随后添加/取出硬币，则铸造/销毁新的流动性Token，以使每个人在流动性池中的相对百分比份额保持不变。
	- 假设交易

		现在让我们假设 Coinbase 上的交易价格从 100 美元到 150 美元。Uniswap 合约在一些套利之后也应该反映这种变化。交易者将添加 DAI 并移除 ETH，直到新的比率现在为 150:1。流动性提供者会怎样？
		
		该合约反映了接近 122,400 DAI 和 817 ETH （为了检查这些数字是否准确，122,400 * 817 = 100,000,000（恒定值）和 122,400 / 817 = 150 新价格）。提取我们有权获得的 10% 现在将产生 12,240 DAI 和 81.7 ETH。这里的总市值为 24,500 美元。由于做市，错过了大约 500 美元的利润。
		
		显然，没有人愿意通过慈善方式提供流动性，而且收入并不依赖于从良好交易中撤出的能力（没有翻转）。相反，所有交易量的 0.3% 按比例分配给所有流动性提供者。默认情况下，这些费用会被放回流动资金池，但可以随时收取。如果不知道中间交易的数量，就很难知道费用收入和定向运动损失之间的权衡是什么。来回交易的越多越好。

	- 为什么我的流动性比我投入的少

		为了理解为什么流动性提供者的股份价值会下降，尽管有费用收入，我们需要更仔细地研究 Uniswap 用于管理交易的公式。公式真的很简单。如果我们忽略交易费用，我们有以下几点：

		- `eth_liquidity_pool * token_liquidity_pool = constant_product`

		换句话说，交易者收到的 ETH Token数量（反之亦然）是这样计算的，即交易后，两个流动性池的乘积与交易前相同。这个公式的结果是，对于与我们拥有的流动资金池规模相比价值非常小的交易：

		- `eth_price = token_liquidity_pool / eth_liquidity_pool`
		
		结合这两个等式，假设总流动性恒定，我们可以计算出在任何给定价格下每个流动性池的规模：

		- `eth_liquidity_pool = sqrt(constant_product / eth_price)`
		- `token_liquidity_pool = sqrt(constant_product * eth_price)`

		因此，让我们看看价格变化对流动性提供者的影响。
		
		为简单起见，假设
		
		- 我们的流动性提供者向 Uniswap DAI Exchange提供 1 ETH 和 100 DAI，包含 100 ETH 和 10,000 DAI 的总量中，给他们 1% 的流动性池。
		- 这意味着 1 ETH = 100 DAI 的价格。
		- 还是忽略手续费，我们想象一下，经过一些交易，价格发生了变化；1 ETH 现在价值 120 DAI。
		
		流动性提供者股权的新价值是多少？将数字代入上面的公式，我们有：

		- `eth_liquidity_pool = 91.2871`
		- `dai_liquidity_pool = 10954.4511`

		“由于我们的流动性提供者拥有 1% 的流动性Token，这意味着他们现在可以从流动性池中索取 0.9129 ETH 和 109.54 DAI。但由于 DAI 大约相当于美元，我们可能更愿意将全部金额转换为 DAI 来理解价格变化的整体影响。在当前价格下，我们的流动性总共价值 219.09 DAI。如果流动性提供者刚刚持有他们原来的 1 ETH 和 100 DAI 怎么办？好吧，现在我们可以很容易地看到，以新价格计算，总价值为 220 DAI。因此，我们的流动性提供者通过向 Uniswap 提供流动性而不是仅仅持有他们最初的 ETH 和 DAI，损失了 0.91 DAI。

		“当然，如果价格回到流动性提供者添加流动性时的相同值，这种损失就会消失。因此，我们可以称之为无常损失。使用上面的等式，我们可以推导出一个公式以提供流动性时与现在之间的价格比率来衡量无常损失的大小。我们得到以下结果：“

		- " `impermanent_loss = 2 * sqrt(price_ratio)/(1+price_ratio)-1`"
		- “我们可以绘制出来，以大致了解不同价格比率下无常损失的规模：” 

			![](./pic/v2protocol6.PNG)

		- 或者换一种说法
			- 1.25 倍的价格变化导致相对于 HODL 损失 0.6% 
			- 1.50 倍的价格变化导致相对于 HODL 损失 2.0%
			- 1.75 倍的价格变化导致相对于 HODL 损失 3.8%
			- 2 倍的价格变化导致相对于 HODL 损失 5.7%
			- 3 倍的价格变化导致相对于 HODL 损失 13.4% 
			- 4 倍的价格变化导致相对于 HODL 损失 20.0% 
			- 5 倍的价格变化导致相对于 HODL 损失 25.5%
		- “注意，无论价格变化发生在哪个方向，损失都是相同的（即价格翻倍导致与减半相同的损失）。” 

#### 安全
- 审计和形式验证

	在 1 月 8 日至 4 月 30 日期间一个由六名工程师组成的团队审查并正式验证了 Uniswap V2 智能合约的关键组件。

	他们过去的工作包括多抵押 DAI 的智能合约开发和形式验证。

	工作范围包括：
	
	- 核心智能合约的形式化验证
	- 核心智能合约的代码审查
	- 数值误差分析
	- 外围智能合约的代码审查（在持续开发中）

	该报告还有一个“设计评论”部分，我们强烈建议您深入了解 Uniswap V2 中的某些选择。

	[阅读报告](https://uniswap.org/audit.html)
- 漏洞赏金

	Uniswap 有一个开放且持续的漏洞[赏金计划](https://uniswap.org/bug-bounty/)。
- 在 Uniswap 上构建时的注意事项

	将 Uniswap V2 集成到另一个链上系统时，必须特别注意避免安全漏洞、操纵途径和潜在的资金损失。

	作为初步说明：
	
	智能合约集成可以发生在两个级别：
	
	- 直接使用 [Pair](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair) 合约
	- 或通过 [Router](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02)

	直接交互提供了最大的灵活性，但需要做最多的工作才能做到正确。中介交互提供了更有限的能力，但更强大的安全保障。

	Uniswap V2 有两种主要的风险类别。
	
	- 第一个涉及所谓的“静态”错误。这可能包括在交换期间向一对发送过多的Token（或请求返回的Token太少）
	- 或者允许交易在内存池中停留足够长的时间，以致发送者对价格的预期不再准确。

	可以通过相当简单的逻辑检查来解决这些错误。执行这些逻辑检查是路由器的主要目的。那些直接与配对交互的人必须自己执行这些检查（在[lib](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library)的帮助下。

	第二类“动态”风险涉及运行时定价。由于以太坊交易发生在对抗性环境中，乐观编写的智能合约可以而且将会被利用以获取利润。
	
	- 例如，假设智能合约在运行时检查 Uniswap 池中的资产比率并进行交易，假设该比率代表这些资产的“公平”或“市场”价格。在这种情况下，它很容易受到操纵。
	- 例如，恶意行为者可以在乐观交易之前和之后简单地插入交易（所谓的“三明治”攻击），导致智能合约以更差的价格进行交易，以牺牲交易者为代价从中获利，然后返回以低廉的成本将合约恢复到原始状态。

	防范这些攻击的最佳方法是引入价格预言机。预言机是返回所需信息的任何设备，在这种情况下，是一对现货价格。最好的“预言机”只是交易者对当前价格的链下观察，可以作为安全检查传递到交易中。这种策略最适合用户代表自己发起交易的零售交易场所。然而，通常情况下，可信的价格观察不可用（例如，在涉及 Uniswap 的多步骤、程序化交互中）。如果没有价格预言机，这些交互将被迫以 Uniswap 上的（可能被操纵的）利率进行交易。有关预言机的 Uniswap V2 方法的详细信息，请参阅[预言机](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/oracles)。

#### 研究
自动化做市商是一个新概念，因此，新的研究经常出现。我们在这里选择了一些最周到的。

- Uniswap 的金融炼金术

	作者：戴夫·怀特、马丁·塔西、查理·诺伊斯和丹·罗宾逊

	自动化做市商是一种去中心化Exchange，让客户可以在 USDC 和 ETH 等链上资产之间进行交易。Uniswap 是以太坊上最受欢迎的 AMM。与大多数 AMM 一样，Uniswap 通过持有两种资产的储备来促进一对特定资产之间的交易。它根据其储备金规模设定它们之间的交易价格，以使价格与大盘保持一致。任何想加入特定Token对的“池”并成为流动性提供者或 LP 的人，所谓的因为他们提供流动资产供其他人交易。有限合伙人同时为两种储备贡献资产，承担一些交易风险以换取收益的一部分。

	- [Uniswap 的金融炼金术](https://research.paradigm.xyz/uniswaps-alchemy)
- Uniswap 市场分析

	作者：Guillermo Angeris、Hsien-Tang Kao、Rei Chiang、Charlie Noyes、Tarun Chitra

	Uniswap——以及其他不变的产品市场——在实践中似乎运作良好，尽管它们很简单。在本文中，我们对不变产品市场及其概括进行了简单的形式分析，表明在某些常见条件下，这些市场必须密切跟踪参考市场价格。我们还展示了 Uniswap 满足许多其他理想的特性，并通过基于代理的大规模模拟在数值上证明 Uniswap 在广泛的市场条件下是稳定的。

	- [Uniswap 市场分析](https://arxiv.org/abs/1911.03380)
- 改进的价格预言机：恒定函数做市商

	作者：吉列尔莫·安杰里斯、塔伦·奇特拉

	自动化做市商首先由 Hanson 的预测市场对数市场评分规则（或 LMSR）推广，已成为去中心化金融的重要组成部分，称为“基元”。一个特别有用的原语是衡量资产价格的能力，这个问题通常被称为定价预言问题。在本文中，我们重点分析了一大类自动化做市商，称为恒函数做市商（或 CFMM），其中包括现有的流行做市商，如 Uniswap、Balancer 和 Curve，其年交易量总计达数十亿美元。我们给出充分条件，使得在相当一般的假设下，与这些恒定功能做市商互动的代理人被激励正确报告资产的价格，并且他们可以以计算有效的方式这样做。我们还推导出了其他几个以前不知道的有用属性。其中包括 CFMM 持有的资产总价值的下限，以及保证任何代理都不能通过任何一组交易耗尽给定 CFMM 持有的资产储备的下限。

	- [改进的价格预言机：恒定函数做市商](https://arxiv.org/abs/2003.10001)
- 针尾研究

	Pintail发表的[媒体文章](https://medium.com/@pintail)。

	- [了解 Uniswap 回报](https://medium.com/@pintail/understanding-uniswap-returns-cc593f3499ef)
	- [Uniswap：对流动性提供者来说是一笔划算的交易吗？](https://medium.com/@pintail/uniswap-a-good-deal-for-liquidity-providers-104c0b6816f2)
- 几何平均市场中的流动性提供者回报

	作者：亚历克斯·埃文斯

	几何平均做市商 (G3Ms)，例如 Uniswap 和 Balancer，是由以下规则定义的一类流行的自动做市商 (AMM)：每笔交易之前和之后的 AMM 储备必须具有相同的（加权）几何平均值. 本文将几个已知的恒权 G3M 结果扩展到具有时变和潜在随机权重的 G3M 的一般情况。这些结果包括投资者为向 G3M 提供流动性而获得的流动性池 (LP) 股票的回报和无套利价格。使用这些表达式，我们展示了如何创建 G3M，其 LP 股票复制金融衍生品的收益。由此产生的对冲是模型独立的，并且对于支付函数满足弹性约束的衍生合约是精确的。这些策略允许 LP 股票复制各种交易策略和金融合约，包括标准期权。因此，G3M 被证明能够通过 LP 股票的被动头寸重新创建各种主动交易策略。

	- [几何平均市场中的流动性提供者回报](https://arxiv.org/abs/2006.08806)
- 恒定产品市场的复制投资组合

	作者：约瑟夫·克拉克

	我们推导出恒定产品市场的复制组合。这是结构上的空头波动（卖出期权），这解释了为什么需要正的交易成本来吸引流动性提供者参与。在不存在期货和期权市场的地方，这种收益可以用来创造它们。

	- [https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3550601](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3550601)
	
		

## 参考	
[Uniswap V2 协议](https://docs.uniswap.org/protocol/V2/introduction)		

	
				
	

	

			

	
	
			
	
	
						
		
		
			
	



	
	
