# Uniswap V2 协议-技术参考
## 技术参考
### API
#### API 概述
本节介绍 Uniswap 子图以及如何与之交互。Uniswap 子图随着时间的推移索引来自 Uniswap 合约的数据。它组织了有关交易对、Token、Uniswap 整体等的数据。每当在 Uniswap 上进行交易时，子图都会更新。子图运行在 [Graph](https://thegraph.com/) 协议的托管服务上，可以公开查询。

- 资源
	- [Subgraph Explorer](https://thegraph.com/explorer/subgraph/uniswap/uniswap-v2) - 用于为开发人员查询数据和端点的沙箱。
	- [Uniswap V2 Subgraph](https://github.com/Uniswap/uniswap-v2-subgraph) - 已部署子图的源代码。
- 用途

	子图提供 Uniswap 当前状态的快照，并跟踪历史数据。它目前用于为 [uniswap.info](https://uniswap.info/) 供电。它不打算用作构建交易的数据源（应直接引用合约以获得最可靠的实时数据）。
- 查询

	要了解有关查询子图的更多信息，请参阅 [Graph](https://thegraph.com/docs/about/introduction) 的文档。
- 版本

	[Uniswap V2 子图](https://thegraph.com/explorer/subgraph/uniswap/uniswap-v2) 仅跟踪 Uniswap V2 上的数据。有关 Uniswap V1 的信息，请参阅 [V1 子图](https://thegraph.com/explorer/subgraph/graphprotocol/uniswap)。

#### 实体
实体定义子图的模式，并表示可以查询的数据。每个实体中都有一组字段，用于存储与该实体相关的有用信息。下面是 Uniswap 子图中可用实体的列表，以及可用字段的描述。

要查看所有实体的交互式沙箱，请参阅 [Graph Explorer](https://thegraph.com/explorer/subgraph/uniswap/uniswap-v2)。

每个实体都定义了一个值类型，它始终是基本的 AssemblyScript 类型或者由 The Graph 的自定义 TypeScript 库提供的自定义类型。有关值类型的更多信息，请参见[此处](https://thegraph.com/docs/assemblyscript-api#api-reference)。

- Uniswap Factory

	Uniswap 工厂实体负责存储所有 Uniswap 对的聚合信息。它可用于查看有关总流动性、交易量、Token对数量等的统计信息。子图中只有一个 UniswapFactory 实体。

	字段名称|值类型|描述
	---|---|---|
	id| ID|工厂地址
	pairCount| Int|Uniswap 工厂创建的交易对数量
	totalVolumeUSD| BigDecimal|	所有Token对的所有时间美元交易量（衍生美元）
	totalVolumeETH| BigDecimal| 所有Token对中 ETH 的所有时间交易量（衍生出 ETH）
	totalLiquidityUSD| BigDecimal|存储金为衍生美元金额的所有Token对的总流动性
	totalLiquidityETH| BigDecimal|存储金为派生 ETH 数量的所有Token对的总流动性
	txCount|	BigInt|所有交易对的所有时间交易
- Token

	在包含该Token的所有对中存储特定Token的聚合信息。
	
	字段名称|值类型|描述
	---|---|---|
	id| ID|Token地址
	symbol| String|象征符号
	name| String|Token名称
	decimals| BigInt|Token小数
	tradeVolume| BigDecimal|在所有Token对中一直交易的Token数量
	tradeVolumeUSD| BigDecimal|美元Token的总交易量（仅适用于流动性高于最低阈值的Token）
	untrackedVolumeUSD| BigDecimal|跨Token对交易的美元Token数量（无最低流动性阈值）
	txCount| BigInt|所有时间成对的交易量，包括Token
	totalLiquidity| BigDecimal|在所有Token对中作为流动性提供的Token总量
	derivedETH| BigDecimal|每个Token的 ETH
- Pair

	关于一对的信息。包括对中每个Token的引用、交易量信息、流动性信息等。配对实体反映配对智能合约，还包含有关使用的汇总信息
	
	字段名称|值类型|描述
	---|---|---|
	id| ID|对合约地址
	factory| UniswapFactory|引用 Uniswap 工厂实体
	token0| Token|对存储在对合约中的 token0 的引用
	token1| Token|对存储在对合约中的 token1 的引用
	reserve0| BigDecimal|Token0储备
	reserve1| BigDecimal|Token1的储备
	totalSupply| BigDecimal|分配给 LP 的流动性Token的总供应量
	reserveETH| BigDecimal|以 ETH 的数量存储的Token对的总流动性
	reserveUSD| BigDecimal|以美元金额存储的Token对总流动性金额
	trackedReserveETH| BigDecimal|仅追踪金额的总流动性（见追踪金额）
	token0Price| BigDecimal|每个 token1 值多少 token0
	token1Price| BigDecimal|每个 token0 值多少 token1
	volumeToken0| BigDecimal|在这对上交换的 token0 数量
	volumeToken1| BigDecimal|在这对上交换的 token1 数量
	volumeUSD| BigDecimal|该Token对中以美元存储的所有时间交换的总金额（仅在美元流动性高于最低阈值时跟踪）
	untrackedVolumeUSD| BigDecimal|以美元存储的该Token对中的总交换量，没有最低流动性阈值
	txCount| BigInt|这对交易的所有时间数量
	createdAtTimestamp| BigInt|合约创建时间戳
	createdAtBlockNumber| BigInt|创建合约以太坊区块高度
	liquidityPositions|[LiquidityPosition]|流动性提供者数组，用作 LP 实体的参考
- User

	为任何向 Uniswap 上的矿池提供流动性的地址创建一个用户实体。此实体可用于跟踪用户的未平仓头寸。可以引用 LiquidyPosition 实体以获取有关每个位置的特定数据。
	
	字段名称|值类型|描述
	---|---|---|
	id|ID|用户地址
	liquidityPositions|[LiquidityPosition]|用户已打开的所有流动性头寸数组
	usdSwapped| BigDecimal|交换的美元总价值
- LiquidityPositiion

	该实体用于存储有关用户流动性头寸的数据。此信息以及来自Token对本身的信息可用于提供头寸大小、Token存款等。
	
	字段名称|值类型|描述
	---|---|---|
	id| ID|用破折号连接的用户地址和配对地址
	user| User|参考用户
	pair| Pair|提供对流动性的参考
	liquidityTokenBalance| BigDecimal|为该职位铸造的 LP Token数量
- Transaction

	为每个包含 Uniswap 合约内交互的以太坊交易创建交易实体。该子图跟踪 Uniswap 核心合约上的 Mint、Burn 和 Swap 事件。每个事务包含 3 个数组，其中至少一个数组的长度为 1
	
	字段名称|值类型|描述
	---|---|---|
	id| ID|以太坊交易哈希
	blockNumber| BigInt|交易在那个高度区块
	timestamp| BigInt|交易时间戳
	mints|[Mint]|交易中的铸造事件数组，0或更大
	burns|[Burn]|交易中的烧毁事件数组，0或更大
	swaps|[Swap]|交易中的交易事件数组，0或更大
- Mint

	为 Uniswap 核心合约上每个发出的 Mint 事件创建 Mint 实体。Mint 实体存储有关事件的关键数据，例如Token数量、谁发送了交易、谁收到了流动性等等。该实体可用于跟踪Token对的流动性规定。
	
	字段名称|值类型|描述
	---|---|---
	id| ID|交易造币厂数组中的交易哈希加索引|
	transaction| Transaction|对交易 Mint 的引用包含在|
	timestamp| BigInt|Mint 的时间戳，用于对最近的流动性规定进行排序|
	pair| Pair|参考对
	to| Bytes|流动性Token的接受者
	liquidity| BigDecimal|铸造的流动性Token数量
	sender| Bytes|启动流动性提供的地址
	amount0| BigDecimal|提供的token0数量
	amount1| BigDecimal|提供的token1数量
	logIndex| BigInt|已发出事务事件中的索引
	amountUSD| BigDecimal|token0 金额加上 token1 金额的衍生美元价值
	feeTo| Bytes|	收款人地址（如果收费）
	feeLiquidity| BigDecimal|发送给费用接收者的流动性金额（如果收费）
- Burn

	为 Uniswap 核心合约上每个发出的 Burn 事件创建 Burn 实体。Burn 实体存储有关事件的关键数据，例如Token数量、谁销毁了 LP Token、谁收到了Token等。该实体可用于跟踪Token对的流动性移除
	
	字段名称|值类型|描述
	---|---|---	
	id| ID|交易销毁数组中的交易哈希加索引
	transaction| Transaction|对交易 Burn 的引用包含在
	timestamp| BigInt|Burn 的时间戳，用于对最近的流动性移除进行排序
	pair| Pair|参考对
	to| Bytes|Token的接受者
	liquidity| BigDecimal|被烧毁的流动性Token数量
	sender| Bytes|启动流动性移除的地址
	amount0| BigDecimal|移除的token0数量
	amount1| BigDecimal|移除的token1数量
	logIndex| BigInt|已发出事务事件中的索引
	amountUSD| BigDecimal|token0 金额加上 token1 金额的衍生美元价值
	feeTo| BigInt|收款人地址（如果收费）
	feeLiquidity| BigDecimal|发送给收款人的Token数量（如果收费）
- swap

	为一对中的每个Token交换创建交换实体。Swap 实体可用于获取交换大小（以Token和美元计）、发送者、接收者等信息。有关金额的更多信息，请参阅swap概览页面。
	
	字段名称|值类型|描述
	---|---|---
	id| ID|事务交换数组中的事务哈希加索引|
	transaction| Transaction|交易互换的参考被包含在|
	timestamp| BigInt|交换的时间戳，用于排序查找|
	pair| Pair|参考对
	sender| Bytes|启动交换的地址|
	amount0In| BigDecimal|已售出的 token0 数量|
	amount1In| BigDecimal|已售出的 token1 数量|
	amount0Out| BigDecimal|收到的token0数量|
	amount1Out| BigDecimal|收到的token1数量|
	to| Bytes|输出Token的接收者|
	logIndex| Bytes|事务内的事件索引|
	amountUSD| BigDecimal|以美元出售的Token数量|
- Bundle

	该捆绑包用作以美元为单位的衍生 ETH 价格的全球存储。由于Token对之间没有保证的通用基础Token，因此美元价格的全球参考对于推导其他美元价值很有用。Bundle 实体存储 ETH<-> 稳定币对价格的更新加权平均值。这为可以在子图中的其他地方使用的 ETH 的美元价格提供了强有力的估计。
	
	字段名称|值类型|描述
	---|---|---
	id| ID|常数 
	ethPrice| BigDecimal|基于稳定币对得出的 ETH 美元价格

#### 历史实体
该子图跟踪按天分组的汇总信息，以提供有关 Uniswap 日常活动的见解。虽然[时间旅行查询](https://blocklytics.org/blog/ethereum-blocks-subgraph-made-for-time-travel/)可用于与过去的值进行直接比较，但查询分组数据的成本要高得多。出于这个原因，子图使用合约事件提供的时间戳跟踪分组在每日存储桶中的信息。这些实体可用于查询特定日期的总交易量、特定日期的Token价格等。

对于每个 DayData 类型，每天都会创建一个新实体。	

- UniswapDayData

	跟踪聚合到每日存储桶中的所有对的数据。	

	字段名称|值类型|描述
	---|---|---
	id| ID|一天开始的 unix 时间戳 / 86400 给出唯一的日期索引
	date| Int|一天开始的Unix时间戳
	dailyVolumeETH| BigDecimal|当天所有Token对的总交易量，存储为衍生的 ETH 数量
	dailyVolumeUSD| BigDecimal|当天所有Token对的总交易量，存储为衍生的美元金额
	totalVolumeETH| BigDecimal|截至今天（包括今天），ETH 中所有Token对的所有时间交易量
	totalLiquidityETH| BigDecimal|截至今天（包括今天），ETH 中所有Token对的总流动性
	totalVolumeUSD| BigDecimal|截至当天（含当天）所有Token对的所有时间交易量（以美元计）
	totalLiquidityUSD| BigDecimal|截至当天（含当天）所有Token对的总流动性（以美元计）
	maxStored| Int|用于存储大多数流动性Token的参考，用于历史流动性图表
	mostLiquidTokens|[TokenDayData!]|Uniswap 中流动性最强的Token
	txCount|	BigInt|全天的交易数量
- Pair Day Data

	跟踪每天的配对数据。
	
	字段名称|值类型|描述
	---|---|---
	id| ID|将合约地址和日期 ID（unix / 86400 中的日期开始时间戳）与破折号连接起来
	date| Int|一天开始的 Unix 时间戳
	pairAddress| Bytes|配对合约地址
	token0| Token|对 token0 的引用
	token1| Token|对 token1 的引用
	reserve0| BigDecimal|token0 的储备（在对的每次交易期间更新）|
	reserve1| BigDecimal|token1 的储备（在对的每次交易期间更新）
	totalSupply| BigDecimal|分配给 LP 的流动性Token的总供应量
	reserveUSD| BigDecimal|	Token 0 的储备加上Token 1 存储为派生的美元金额
	dailyVolumeToken0| BigDecimal|全天交换的token0总量
	dailyVolumeToken1| BigDecimal|全天交换的token1总量
	dailyVolumeUSD| BigDecimal|全天对内的总交易量
	dailyTxns| BigInt|全天配对交易量
- TokenDayData

	跟踪包含Token的所有对中聚合的Token数据
	
	字段名称|值类型|描述
	---|---|---
	id| ID|Token地址和日期 ID（unix / 86400 中的日期开始时间戳）与破折号连接
	date| Int|一天开始的Unix时间戳
	token| Token|对Token实体的引用
	dailyVolumeToken| BigDecimal|全天在所有Token对之间交换的Token数量
	dailyVolumeETH| BigDecimal|	全天在所有Token对之间交换的Token数量存储为衍生的 ETH 数量
	dailyVolumeUSD| BigDecimal|全天在所有Token对之间交换的Token数量存储为衍生的美元数量
	dailyTxns| BigInt|在所有交易对中使用此Token的交易量
	totalLiquidityToken|BigDecimal|Token数量 存放在所有Token对中的Token数量
	totalLiquidityETH|BigDecimal|Token数量 存放在所有Token对中的Token数量，以 ETH 数量存储
	totalLiquidityUSD|BigDecimal|以美元金额存储在所有Token对中的Token数量
	priceUSD|BigDecimal|衍生美元的Token价格
	maxStored|Int|	以最高Token流动性成对存放的Token数量 - 仅用作存储此Token的大多数流动对的参考
	mostLiquidPairs|[PairDayData]|与此Token的流动性最大的配对

#### 查询
可以查询子图以检索有关 Uniswap、对、Token、交易、用户等的重要信息。此页面将提供常见查询的示例。

要尝试这些查询并运行您自己的查询，请访问[子图沙箱](https://thegraph.com/explorer/subgraph/uniswap/uniswap-v2)。

- Global Data

	要查询全局数据，可以传入 Uniswap 工厂地址并从可用字段中进行选择。
	
	- Global Stats 

		以美元计的所有时间量，以美元计的总流动性，所有时间的交易计数。
		
			{
			 uniswapFactory(id: "0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f"){
			   totalVolumeUSD
			   totalLiquidityUSD
			   txCount
			 }
			}
	- Global Historical lookup

		要获取过去状态的快照，请使用 The Graph 的块查询功能并在前一个块处进行查询。请参阅这篇文章以获取有关[从时间戳中获取块号](https://blocklytics.org/blog/ethereum-blocks-subgraph-made-for-time-travel/)的更多信息。这可以用来计算诸如 24 小时体积之类的东西。
		
		{
		 uniswapFactory(id: "0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f", block: {number: 10291203}){
		   totalVolumeUSD
		   totalLiquidityUSD
		   txCount
		 }
		}
- Pair Data
	- Pair Overview

		获取具有共同值的对的当前状态的快照。此示例获取 DAI/WETH 对。

			{
			 pair(id: "0xa478c2975ab1ea89e8196811f51a7b7ade33eb11"){
			     token0 {
			       id
			       symbol
			       name
			       derivedETH
			     }
			     token1 {
			       id
			       symbol
			       name
			       derivedETH
			     }
			     reserve0
			     reserve1
			     reserveUSD
			     trackedReserveETH
			     token0Price
			     token1Price
			     volumeUSD
			     txCount
			 }
			}
	- All pairs in Uniswap

		截至目前，图表将每个查询的实体返回量限制为 1000。要获取 Uniswap 上的所有对，请使用循环和 graphql 跳过查询来获取 1000 对的多个块。查询看起来像这样（其中 skip 是传递给您的查询的一些递增变量）。

			{
			 query pairs($skip: Int!) {
			   pairs(first: 1000, skip: $skip) {
			     id
			   }
			 }
			}
	- Most liquid pairs

		按流动性排序以获得 Uniswap 中流动性最强的Token对。

			{
			 pairs(first: 1000, orderBy: reserveUSD, orderDirection: desc) {
			   id
			 }
			}
	- Recent Swaps within a Pair

		通过获取交换事件并传入对地址来获取对的最后 100 次交换。您通常还需要Token信息。
		
			{
			swaps(orderBy: timestamp, orderDirection: desc, where:
			 { pair: "0xa478c2975ab1ea89e8196811f51a7b7ade33eb11" }
			) {
			     pair {
			       token0 {
			         symbol
			       }
			       token1 {
			         symbol
			       }
			     }
			     amount0In
			     amount0Out
			     amount1In
			     amount1Out
			     amountUSD
			     to
			 }
			}
	- Pair Daily Aggregated

		日数据对于围绕实体构建图表和历史视图很有用。要获取有关每日存储桶中一对的统计信息，请查询受时间戳限制的日实体。此查询获取 DAI/WETH 对上给定 unix 时间戳后的前 100 天。

			{
			 pairDayDatas(first: 100, orderBy: date, orderDirection: asc,
			   where: {
			     pairAddress: "0xa478c2975ab1ea89e8196811f51a7b7ade33eb11",
			     date_gt: 1592505859
			   }
			 ) {
			     date
			     dailyVolumeToken0
			     dailyVolumeToken1
			     dailyVolumeUSD
			     reserveUSD
			 }
			}
- Token Data
		
	可以使用Token合约地址作为 ID 来获取Token数据。Token数据汇总在包含Token的所有对中。可以查询包含在 Uniswap 中的某个对中的任何Token。
	
	- Token Overview

		获取 Uniswap 中Token当前统计数据的快照。此查询获取 DAI 的当前统计信息。allPairs 字段获取前 200 对 DAI，按衍生美元的流动性排序。

			{
			 token(id: "0x6b175474e89094c44da98b954eedeac495271d0f"){
			   name
			   symbol
			   decimals
			   derivedETH
			   tradeVolumeUSD
			   totalLiquidity
			 }
			}
	- All Tokens in Uniswap

		与获取所有对（见上文）类似，您可以查询 Uniswap 中的所有Token。因为 Graph 服务将返回大小限制为 1000 个实体，所以使用 graphql 跳过查询。（请注意，此查询在图形沙箱中不起作用，更像是您传递给某些 graphql 中间件（如 [Apollo](https://www.apollographql.com/) ）的查询结构）

			{
			 query tokens($skip: Int!) {
			   tokens(first: 1000, skip: $skip) {
			     id
			     name
			     symbol
			   }
			 }
			}
	- Token Transactions

		要获取包含Token的交易，您需要首先获取包含Token的对数组（这可以通过Token实体上的 allPairs 字段完成。）一旦您有了一个对数组，Token就会被包含在内在，在事务查找中过滤。

		此查询获取最近 30 次涉及 DAI 的铸币、swap和销毁。allPairs 数组可能看起来像这样，其中我们包含 DAI/WETH 对地址和 DAI/USDC 对地址。
		
			allPairs = [
			 "0xa478c2975ab1ea89e8196811f51a7b7ade33eb11",
			 "0xae461ca67b15dc8dc81ce7615e0320da1a9ab8d5"
			]					

			query($allPairs: [String!]) {
			 mints(first: 30, where: { pair_in: $allPairs }, orderBy: timestamp, orderDirection: desc) {
			   transaction {
			     id
			     timestamp
			   }
			   to
			   liquidity
			   amount0
			   amount1
			   amountUSD
			 }
			 burns(first: 30, where: { pair_in: $allPairs }, orderBy: timestamp, orderDirection: desc) {
			   transaction {
			     id
			     timestamp
			   }
			   to
			   liquidity
			   amount0
			   amount1
			   amountUSD
			 }
			 swaps(first: 30, where: { pair_in: $allPairs }, orderBy: timestamp, orderDirection: desc) {
			   transaction {
			     id
			     timestamp
			   }
			   amount0In
			   amount0Out
			   amount1In
			   amount1Out
			   amountUSD
			   to
			 }
			}
	- Token Daily Aggregated
 
		像对和全局每日查找一样，Token也具有可以查询的每日实体。此查询获取 DAI 的每日信息。请注意，您可能希望按升序排序，以便在返回数组中接收从最早到最近的日期。

			{
			 tokenDayDatas(orderBy: date, orderDirection: asc,
			  where: {
			    token: "0x6b175474e89094c44da98b954eedeac495271d0f"
			  }
			 ) {
			    id
			    date
			    priceUSD
			    totalLiquidityToken
			    totalLiquidityUSD
			    totalLiquidityETH
			    dailyVolumeETH
			    dailyVolumeToken
			    dailyVolumeUSD
			 }
			}
- ETH Price

	您可以使用 Bundle 实体根据稳定币的加权平均值查询 Uniswap 中 ETH 的当前美元价格。

		{
		 bundle(id: "1" ) {
		   ethPrice
		 }
		}

### 智能合约
#### Factory
- 代码

	[UniswapV2Factory.sol](https://github.com/Uniswap/uniswap-v2-core/blob/master/contracts/UniswapV2Factory.sol)
- 地址

	UniswapV2Factory 部署在[以太坊主网上地址](https://goerli.etherscan.io/address/0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f)为 `0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f`，以及 [Ropsten](https://ropsten.etherscan.io/address/0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f)、[Rinkeby](https://rinkeby.etherscan.io/address/0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f)、[Görli](https://etherscan.io/address/0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f)和 [Kovan](https://kovan.etherscan.io/address/0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f) 测试网。它是从 [提交8160750](https://github.com/Uniswap/uniswap-v2-core/tree/816075049f811f1b061bca81d5d040b96f4c07eb) 构建的。
- 事件
	- 创建配对

		每次通过 [createPair](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#createpair) 创建对时发出。

		- `token0` 保证严格小于 `token1` 按排序顺序。
		- 最终的 `uint` 日志值将是`1`创建的第一对、2第二对等（参见 [allPairs](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#allpairs) /[getPair](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#getpair)）。
		
		参考
			
			event PairCreated(address indexed token0, address indexed token1, address pair, uint); 
- 只读函数
	- getPair

		用 `tokenA` 和 `tokenB` 地址请求，如果已经创建，则返回 `address(0)( 0x0000000000000000000000000000000000000000)`。

		- `tokenA` 和 `tokenB` 可以互换。
		- 也可以通过 SDK 确定性地计算配对地址。

		参考
		
			function getPair(address tokenA, address tokenB) external view returns (address pair);
	- allPairs

		返回通过工厂创建的第`n`个对 ( `0-indexed`) 的地址，如果还没有创建足够的对，则返回 `address(0)(0x0000000000000000000000000000000000000000)。`
		
		- 传递 `0` 创建的第一对的地址，第二对的地址1，等等
		
		参考
		
			function allPairs(uint) external view returns (address pair);
	- allPairsLength

		返回到目前为止通过工厂创建的对的总数。
		
			function allPairsLength() external view returns (uint);
	- feeTo

		请参阅[协议费用计算](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/fees)
		
			function feeTo() external view returns (address);
	- feeToSetter

		允许更改 [feeTo](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#feeto) 的地址。
		
			function feeToSetter() external view returns (address);
- 状态改变函数
	- createPair

		如果一个不存在 ，则创建一对 `tokenA` 和 `tokenB`

		- `tokenA` 和 `tokenB` 可以互换。
		- 发出 [PairCreated](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#paircreated)。
- 界面

		import '@uniswap/v2-core/contracts/interfaces/IUniswapV2Factory.sol';
		
		pragma solidity >=0.5.0;

		interface IUniswapV2Factory {
		  event PairCreated(address indexed token0, address indexed token1, address pair, uint);
		
		  function getPair(address tokenA, address tokenB) external view returns (address pair);
		  function allPairs(uint) external view returns (address pair);
		  function allPairsLength() external view returns (uint);
		
		  function feeTo() external view returns (address);
		  function feeToSetter() external view returns (address);
		
		  function createPair(address tokenA, address tokenB) external returns (address pair);
		}
- [ABI](https://unpkg.com/@uniswap/v2-core@1.0.0/build/IUniswapV2Factory.json)

		import IUniswapV2Factory from '@uniswap/v2-core/build/IUniswapV2Factory.json'

#### Pair
本文档涵盖了 Uniswap 特定的功能。有关 ERC-20 功能，请参阅 [Pair (ERC-20)](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20)。

- code		
	
	[UniswapV2Pair.sol](https://github.com/Uniswap/uniswap-v2-core/blob/master/contracts/UniswapV2Pair.sol)
- 地址

	请参阅[配对地址](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/getting-pair-addresses)
- 事件
	- mint

		每次通过 [mint](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#mint-1) 创建流动性Token时都会发出。
		
			event Mint(address indexed sender, uint amount0, uint amount1);
	- brun

		每次流动性Token通过[brun](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#burn-1)消除时都会发出
		
			event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
	- Swap

		每次通过 swap 发生[交换](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#swap-1)时发出。
		
			event Swap(
			  address indexed sender,
			  uint amount0In,
			  uint amount1In,
			  uint amount0Out,
			  uint amount1Out,
			  address indexed to
			);
	- sync

		每次通过 [mint](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#mint-1)、[burn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#burn-1)、[swap](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#swap-1)或[sync](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#sync-1)更新储备时都会发出。
		
			event Sync(uint112 reserve0, uint112 reserve1);
- 只读函数
	- MINIMUM_LIQUIDITY

		返回1000所有对。请参阅[最低流动性](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/smart-contracts#minimum-liquidity) 
		
			function MINIMUM_LIQUIDITY() external pure returns (uint);
	- factory

		返回[工厂地址](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#address)
		
			function factory() external view returns (address);
	- token0

		返回排序顺序较低的对标记的地址
		
			function token0() external view returns (address);
	- token1

		返回具有较高排序顺序的对标记的地址。
		
			function token1() external view returns (address);
	- getReserves

		返回用于定价交易和分配流动性的 `token0` 和 `token1` 的储备金。请参阅 [定价](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/pricing)。还返回对发生交互的最后一个块的 `block.timestamp ( mod 2**32)`
		
			function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
	- price0CumulativeLast

		查看[语言机](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/oracles)

			function price0CumulativeLast() external view returns (uint);
	- price1CumulativeLast

		查看[语言机](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/oracles)
		
			function price1CumulativeLast() external view returns (uint);
	- kLast

		返回最近流动性事件的储备乘积。请参阅[协议费用计算](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/fees#protocol-charge-calculation)
		
			function kLast() external view returns (uint);
- 状态改变函数
	- mint

		创建池Token。
		
		- 发行 [Mint](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#mint)、[Sync](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#sync)、[Transfer](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20#transfer)。 
		
		参照
		
			function mint(address to) external returns (uint liquidity);
	- burn

		销毁池Token。
		
		- 发行 [burn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#burn)、[Sync](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#sync)、[Transfer](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20#transfer)。 
	- swap

		交换Token。对于定期 swap，`data.length` 必须是 `0`. 另请参阅[闪电交换](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/flash-swaps)。

		- 发行 [swap](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#swap)，[sync](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#sync)。 
	- skim

		查看[白皮书](https://docs.uniswap.org/whitepaper.pdf)
		
			function skim(address to) external;
	- sync

		查看[白皮书](https://docs.uniswap.org/whitepaper.pdf)
		
		- 发行[sync](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#sync)

		参照
		
			function sync() external;
- 接口

		import '@uniswap/v2-core/contracts/interfaces/IUniswapV2Pair.sol'; 
		
		pragma solidity >=0.5.0;
		
		interface IUniswapV2Pair {
		  event Approval(address indexed owner, address indexed spender, uint value);
		  event Transfer(address indexed from, address indexed to, uint value);
		
		  function name() external pure returns (string memory);
		  function symbol() external pure returns (string memory);
		  function decimals() external pure returns (uint8);
		  function totalSupply() external view returns (uint);
		  function balanceOf(address owner) external view returns (uint);
		  function allowance(address owner, address spender) external view returns (uint);
		
		  function approve(address spender, uint value) external returns (bool);
		  function transfer(address to, uint value) external returns (bool);
		  function transferFrom(address from, address to, uint value) external returns (bool);
		
		  function DOMAIN_SEPARATOR() external view returns (bytes32);
		  function PERMIT_TYPEHASH() external pure returns (bytes32);
		  function nonces(address owner) external view returns (uint);
		
		  function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;
		
		  event Mint(address indexed sender, uint amount0, uint amount1);
		  event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
		  event Swap(
		      address indexed sender,
		      uint amount0In,
		      uint amount1In,
		      uint amount0Out,
		      uint amount1Out,
		      address indexed to
		  );
		  event Sync(uint112 reserve0, uint112 reserve1);
		
		  function MINIMUM_LIQUIDITY() external pure returns (uint);
		  function factory() external view returns (address);
		  function token0() external view returns (address);
		  function token1() external view returns (address);
		  function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
		  function price0CumulativeLast() external view returns (uint);
		  function price1CumulativeLast() external view returns (uint);
		  function kLast() external view returns (uint);
		
		  function mint(address to) external returns (uint liquidity);
		  function burn(address to) external returns (uint amount0, uint amount1);
		  function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;
		  function skim(address to) external;
		  function sync() external;
			}

- ABI			

		import IUniswapV2Pair from '@uniswap/v2-core/build/IUniswapV2Pair.json'
		
	[https://unpkg.com/@uniswap/v2-core@1.0.0/build/IUniswapV2Pair.json](https://unpkg.com/@uniswap/v2-core@1.0.0/build/IUniswapV2Pair.json)

#### Pair (ERC-20)
本文档涵盖了用于命名池Token的 ERC-20 功能。对于 Uniswap 特定的功能，请参阅 [Pair](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair)。	

- 代码

	[UniswapV2ERC20.sol](https://github.com/Uniswap/uniswap-v2-core/blob/master/contracts/UniswapV2ERC20.sol)
- 事件
	- Approval

		每次通过[批准](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#approve)或[许可](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#permit)发生批准时发出。 
		
			event Approval(address indexed owner, address indexed spender, uint value);
	- Transfer

		每次通过 [transfer](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#transfer-1)、[transferFrom](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#transferfrom)、[mint](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#mint-1)或 [burn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#burn-1)发生传输时发出
		
			event Transfer(address indexed from, address indexed to, uint value);
- 只读函数
	- name

		返回 Uniswap V2 所有对  	
		
			function name() external pure returns (string memory);
	- symbol

		返回UNI-V2所有对
		
			function symbol() external pure returns (string memory);
	- decimals

		返回18位精度所有对
		
			function decimals() external pure returns (uint8);
	- totalSupply

		返回一对池Token的总量
		
			function totalSupply() external view returns (uint);
	- balanceOf

		返回地址拥有的池Token数量
		
			function balanceOf(address owner) external view returns (uint);
	- allowance

		返回允许支出者通过 [transferFrom](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#transferfrom) 转移的地址所拥有的流动性Token数量。
		
			function allowance(address owner, address spender) external view returns (uint);
	- DOMAIN_SEPARATOR

		返回用于 [permit](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#permit) 的域分隔符
		
			function DOMAIN_SEPARATOR() external view returns (bytes32);
	- PERMIT_TYPEHASH

		返回用于 [permit](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#permit) 的 typehash
		
			function PERMIT_TYPEHASH() external view returns (bytes32);
	- nonces

		返回地址的当前随机数以供在 [permit](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#permit) 中使用
			
			function nonces(address owner) external view returns (uint);
- 状态改变函数
	- approve

		让我们用  `msg.sender` 为花钱者设置他们的津贴。

		- 发出[批准](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#approval)。

		参考
		
			function approve(address spender, uint value) external returns (bool);
	- transfer

		让我们用 `msg.sender` 将池Token发送到一个地址 
		
		- 发出[传输](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#transfer)

		参考
		
			function transfer(address to, uint value) external returns (bool);
	- transferFrom

		将池Token从一个地址发送到另一个地址。
		
		- 需要批准
		- 发出[传输](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#transfer)

		参考
		
			function transferFrom(address from, address to, uint value) external returns (bool);
	- permit

		设置通过签名授予批准的支出者的限额。

		- 请参阅[使用许可](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/supporting-meta-transactions)。
		- 发出[批准](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/Pair-ERC-20#approval)

		参考

			function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;
- 接口

		import '@uniswap/v2-core/contracts/interfaces/IUniswapV2ERC20.sol';
		
		pragma solidity >=0.5.0;

		interface IUniswapV2ERC20 {
		  event Approval(address indexed owner, address indexed spender, uint value);
		  event Transfer(address indexed from, address indexed to, uint value);
		
		  function name() external pure returns (string memory);
		  function symbol() external pure returns (string memory);
		  function decimals() external pure returns (uint8);
		  function totalSupply() external view returns (uint);
		  function balanceOf(address owner) external view returns (uint);
		  function allowance(address owner, address spender) external view returns (uint);
		
		  function approve(address spender, uint value) external returns (bool);
		  function transfer(address to, uint value) external returns (bool);
		  function transferFrom(address from, address to, uint value) external returns (bool);
		
		  function DOMAIN_SEPARATOR() external view returns (bytes32);
		  function PERMIT_TYPEHASH() external pure returns (bytes32);
		  function nonces(address owner) external view returns (uint);
		
		  function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;
		}
- ABI

		import IUniswapV2ERC20 from '@uniswap/v2-core/build/IUniswapV2ERC20.json'
		
	[https://unpkg.com/@uniswap/v2-core@1.0.0/build/IUniswapV2ERC20.json](https://unpkg.com/@uniswap/v2-core@1.0.0/build/IUniswapV2ERC20.json)	
	
#### library
- code

	[UniswapV2Library.sol](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/libraries/UniswapV2Library.sol)
- 内部功能
	- sortTokens

		对Token地址进行排序
		
			function sortTokens(address tokenA, address tokenB) internal pure returns (address token0, address token1);
	- pairFor

		无需通过 v2 SDK 进行任何外部调用即可计算一对地址
		
			function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair);
	- getReserves

		对传递的Token调用 [getReserves](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#getreserves) ，并返回按传入参数顺序排序的结果	
		
			function getReserves(address factory, address tokenA, address tokenB) internal view returns (uint reserveA, uint reserveB);
	- quote

		给定一些资产数量和储备，返回代表等值的其他资产的数量。

		- 用于在调用 [mint](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#mint-1) 之前计算最佳Token数量。

		参照
		
			function quote(uint amountA, uint reserveA, uint reserveB) internal pure returns (uint amountB);
	- getAmountOut

		给定输入资产金额，返回给定准备金的其他资产（考虑费用）的最大输出金额。

		- 在 [getAmountsOut](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountsout) 中使用

		参照
		
			function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) internal pure returns (uint amountOut);
	- getAmountIn

		返回购买给定输出资产数量（考虑费用）给定储备所需的最小输入资产数量。

		- 在 [getAmountsIn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountsin) 中使用

		参照
		
			function getAmountIn(uint amountOut, uint reserveIn, uint reserveOut) internal pure returns (uint amountIn);
	- getAmountsOut

		给定一个输入资产数量和一个Token地址数组，通过依次为路径中的每对Token地址调用 [getReserves](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getreserves) 并使用这些来调用 [getAmountOut](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountout) 来计算所有后续的最大输出Token数量。

		- 用于在调用 [swap](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#swap-1) 之前计算最佳Token数量。

		参照
		
			function getAmountsOut(uint amountIn, address[] memory path) internal view returns (uint[] memory amounts);
	- getAmountsIn

		给定一个输出资产数量和一个Token地址数组，通过依次为路径中的每对Token地址调用 [getReserves](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getreserves) 并使用这些调用 [getAmountIn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountin)来计算所有先前的最小输入Token数量。

		- 用于在调用 [swap](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#swap-1) 之前计算最佳Token数量。

#### Router01
UniswapV2Router01 不应再使用，因为发现了一个[严重程度较低的错误](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-01#getamountin)，并且某些方法不适用于收取转账费用的Token。当前的建议是使用 [UniswapV2Router02](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02)。

- 其他详情[查看](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-01)

#### Router02
由于路由器是无状态的并且不持有Token余额，因此可以在必要时安全且无需信任地更换它们。如果发现了更有效的智能合约模式，或者需要额外的功能，则可能会发生这种情况。出于这个原因，路由器的版本号从 `01` 开始 。`02 ` 则是当前推荐的版本.

- code

	[UniswapV2Router02.sol](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/UniswapV2Router02.sol)
- 地址

	UniswapV2Router02 部署在 [0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D](https://goerli.etherscan.io/address/0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D) 以太坊主网上，以及[Ropsten](https://ropsten.etherscan.io/address/0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D)、[Rinkeby](https://rinkeby.etherscan.io/address/0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D)、[Görli](https://etherscan.io/address/0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D)和[Kovan](https://kovan.etherscan.io/address/0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D)测试网。它是从[提交 6961711](https://github.com/Uniswap/uniswap-v2-periphery/tree/69617118cda519dab608898d62aaa79877a61004) 构建的。
- 只读函数
	- factory

		返回[工厂地址](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#address) 
		
			function factory() external pure returns (address);
	- WETH

		返回以太坊[主网](https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2)或 [Ropsten](https://ropsten.etherscan.io/address/0xc778417e063141139fce010982780140aa0cd5ab)、[Rinkeby](https://rinkeby.etherscan.io/address/0xc778417e063141139fce010982780140aa0cd5ab)、[Görli](https://goerli.etherscan.io/address/0xb4fbf271143f4fbf7b91a5ded31805e42b2208d6)或[Kovan](https://kovan.etherscan.io/address/0xd0a1e359811322d97991e03f863a0c30c2cf029c)测试网上的[规范 WETH 地址](https://blog.0xproject.com/canonical-weth-a9aa7d0279dd)
	- quote

		详见 [quote](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#quote)
	- getAmountOut

		详见 [getAmountOut](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountout)
	- getAmountIn

		详见 [getAmountIn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountin)
	- getAmountsOut

		详见 [getAmountsOut](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountsout)
		
			function getAmountsOut(uint amountIn, address[] memory path) public view returns (uint[] memory amounts);
	- getAmountsIn					
		
		详见 [getAmountsIn](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library#getamountsin)
- 状态改变函数
	- addLiquidity

			function addLiquidity(
			  address tokenA,
			  address tokenB,
			  uint amountADesired,
			  uint amountBDesired,
			  uint amountAMin,
			  uint amountBMin,
			  address to,
			  uint deadline
			) external returns (uint amountA, uint amountB, uint liquidity);

		为 ERC-20⇄ERC-20 池增加流动性。
		
		- 为了涵盖所有可能的情况，`msg.sender` 应该已经给路由器在 tokenA/tokenB 上至少有 `amountADesired/amountBDesired `的余量。
		- 始终根据交易执行时的价格以理想的比率添加资产。
		- 如果传递Token的池不存在，则会自动创建一个池，并准确添加 `amountADesired/amountBDesired` Token。

		名字|类型|描述
		---|---|---
		tokenA|`address `|池Token
		tokenB|`address `|池Token
		amountADesired|`uint `|如果 B/A 价格 <= amountBDesired/amountADesired（A 贬值），则作为流动性添加的Token A 的数量。
		amountBDesired|`uint `|如果 A/B 价格 <= amountADesired/amountBDesired（B 贬值），则作为流动性添加的Token B 的数量。
		amountAMin|`uint `|限制 B/A 价格在交易恢复之前可以上涨的程度。必须 <= amountADesired
		amountBMin|`uint `|限制在交易恢复之前 A/B 价格可以上涨的程度。必须 <= amountBDesired
		to|`address `|流动性Token的接收者
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		amountA|`uint `|发送到池中的 tokenA 数量
		amountB|`uint `|发送到池中的 tokenB 数量
		liquidity|`uint `|铸造的流动性Token数量
	- addLiquidityETH

			function addLiquidityETH(
			  address token,
			  uint amountTokenDesired,
			  uint amountTokenMin,
			  uint amountETHMin,
			  address to,
			  uint deadline
			) external payable returns (uint amountToken, uint amountETH, uint liquidity);
		
		使用 ETH 为 ERC-20⇄WETH 池增加流动性。
		
		- 为了覆盖所有可能的场景，`msg.sender` 应该已经给路由器一个至少为 `amountTokenDesired` 的Token。
		- 始终根据交易执行时的价格以理想的比率添加资产。
		- `msg.value` 被视为数量 ETHDesired。
		- 剩余的 ETH（如果有）将返回给 `msg.sender`.
		- 如果传递的Token和 WETH 的池不存在，则会自动创建一个池，并准确 `msg.value` 添加 `amountTokenDesired` Token。

		名字|类型|描述
		---|---|---
		token|`address `|池Token
		amountTokenDesired|`uint`|如果 WETH/Token价格 <= `msg.value/amountTokenDesired`（Token贬值），则作为流动性添加的Token数量。
		msg.value (amountETHDesired)|`uint `|如果Token/WETH 价格 <= `amountTokenDesired/ msg.value`（WETH 贬值），则作为流动性添加的 ETH 数量。
		amountTokenMin|`uint `|限制在交易恢复之前 WETH/Token价格可以上涨的程度。必须 `<= amountTokenDesired`
		amountETHMin|`uint `|限制在交易恢复之前Token/WETH 价格可以上涨的程度。必须是 `<= msg.value`
		to|`address `|流动性Token的接收者。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		amountToken|`uint `|发送到池中的Token数量
		amountETH|`uint `|转换为 WETH 并发送到池中的 ETH 数量。
		liquidity|`uint `|铸造的流动性Token数量
	- removeLiquidity

			function removeLiquidity(
			  address tokenA,
			  address tokenB,
			  uint liquidity,
			  uint amountAMin,
			  uint amountBMin,
			  address to,
			  uint deadline
			) external returns (uint amountA, uint amountB);
		从 ERC-20⇄ERC-20 池中移除流动性。
		
		- `msg.sender` 应该已经给路由器至少允许池中的流动性

		名字|类型|描述
		---|---|---
		tokenA|`address `|池Token
		tokenB|	`address `|池Token
		liquidity|`uint `|要移除的流动性Token数量。
		amountAMin|`uint `|为使交易不恢复而必须接收的最小 tokenA 数量
		amountBMin|`uint `|为使交易不恢复而必须接收的最小Token B 数量
		to|`address `| 标的资产的接收方
		deadline|`uint `|交易将恢复的 Unix 时间戳
		amountA|`uint `|收到的 tokenA 数量
		amountB|`uint `|收到的tokenB数量
	- removeLiquidityETH

			function removeLiquidityETH(
			  address token,
			  uint liquidity,
			  uint amountTokenMin,
			  uint amountETHMin,
			  address to,
			  uint deadline
			) external returns (uint amountToken, uint amountETH);
		从 ERC-20⇄WETH 池中移除流动性并接收 ETH。
		
		- `msg.sender` 应该已经给路由器至少允许池中的流动性。	
		名字|类型|描述
		---|---|---
		token|`address `|池Token
		liquidity|`uint `|要移除的流动性Token数量
		amountTokenMin|`uint `|为使交易不恢复而必须接收的最小Token数量
		amountETHMin|`uint `|为使交易不恢复而必须接收的最低 ETH 数量
		to|`address `|标的资产的接收方
		deadline|`uint `|交易将恢复的 Unix 时间戳
		amountToken|`uint `|收到的Token数量
		amountETH|`uint `|收到的 ETH 数量
	- removeLiquidityWithPermit

			function removeLiquidityWithPermit(
			  address tokenA,
			  address tokenB,
			  uint liquidity,
			  uint amountAMin,
			  uint amountBMin,
			  address to,
			  uint deadline,
			  bool approveMax, uint8 v, bytes32 r, bytes32 s
			) external returns (uint amountA, uint amountB);
		由于 [permit](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20#permit)，无需预先批准即可从 ERC-20⇄ERC-20 池中移除流动性。	
		
		名字|类型|描述
		---|---|---
		tokenA|`address `|池Token。
		tokenB|`address `|池Token。
		liquidity|`uint `|要移除的流动性Token数量。
		amountAMin|`uint `|为使交易不恢复而必须接收的最小 tokenA 数量
		amountBMin|`uint `|为使交易不恢复而必须接收的最小Token B 数量。
		to|`address `|标的资产的接收方。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		approveMax|`bool `|签名中的批准金额是流动性还是uint(-1).
		v|`uint8 `|许可签名的 v 分量。
		r|`bytes32 `|许可签名的 r 分量
		s|`bytes32 `|许可签名的 s 组件
		amountA|`uint `|收到的 tokenA 数量
		amountB|`uint `|收到的tokenB数量
	- removeLiquidityETHWithPermit

			function removeLiquidityETHWithPermit(
			  address token,
			  uint liquidity,
			  uint amountTokenMin,
			  uint amountETHMin,
			  address to,
			  uint deadline,
			  bool approveMax, uint8 v, bytes32 r, bytes32 s
			) external returns (uint amountToken, uint amountETH);
		由于 [permit](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20#permit) 从 ERC-20⇄WETTH 池中移除流动性并在未经预先批准的情况下接收 ETH 
		
		名字|类型|描述
		---|---|---
		token|`address `|池Token。
		liquidity|`uint `|要移除的流动性Token数量。
		amountTokenMin|`uint `|为使交易不恢复而必须接收的最小Token数量。
		amountETHMin|`uint `|	为使交易不恢复而必须接收的最低 ETH 数量。
		to|`address `|标的资产的接收方。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		approveMax|`bool `|签名中的批准金额是流动性还是uint(-1).
		v|`uint8 `|许可签名的 v 分量。
		r|`bytes32 `|许可签名的 r 分量。
		s|`bytes32 `|许可签名的 s 组件。
		amountToken|`uint `|收到的Token数量。
		amountETH|	`uint `|收到的 ETH 数量。
	- removeLiquidityETHSupportingFeeOnTransferTokens

			function removeLiquidityETHSupportingFeeOnTransferTokens(
			  address token,
			  uint liquidity,
			  uint amountTokenMin,
			  uint amountETHMin,
			  address to,
			  uint deadline
			) external returns (uint amountETH);	
				
		与 [removeLiquidityETH](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#removeliquidityeth) 相同，但对于收取转账费用的Token会成功。

		- `msg.sender` 应该已经给路由器至少允许池中的流动性。

		名字|类型|描述
		---|---|---
		token|池Token。
		liquidity|要移除的流动性Token数量。
		amountTokenMin|为使交易不恢复而必须接收的最小Token数量。
		amountETHMin|为使交易不恢复而必须接收的最低 ETH 数量。
		to|标的资产的接收方。
		deadline|交易将恢复的 Unix 时间戳。
		amountETH|收到的 ETH 数量。
	- removeLiquidityETHWithPermitSupportingFeeOnTransferTokens

			function removeLiquidityETHWithPermitSupportingFeeOnTransferTokens(
			  address token,
			  uint liquidity,
			  uint amountTokenMin,
			  uint amountETHMin,
			  address to,
			  uint deadline,
			  bool approveMax, uint8 v, bytes32 r, bytes32 s
			) external returns (uint amountETH);	
				
		与 [removeLiquidityETHWithPermit](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#removeliquidityethwithpermit) 相同，但对于收取转账费用的Token会成功。
		
		名字|类型|描述
		---|---|---
		token|`address `|池Token
		liquidity|`uint `|要移除的流动性Token数量。
		amountTokenMin|`uint `|为使交易不恢复而必须接收的最小Token数量
		amountETHMin|`uint `|为使交易不恢复而必须接收的最低 ETH 数量。
		to|`address `|标的资产的接收方。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		approveMax|`bool `|签名中的批准金额是流动性还是uint(-1).
		v|`uint8 `|许可签名的 v 分量。
		r|`bytes32 `|许可签名的 r 分量。
		s|`bytes32 `|许可签名的 s 组件。
		amountETH|`uint `|收到的 ETH 数量。
	- swapExactTokensForTokens

			function swapExactTokensForTokens(
			  uint amountIn,
			  uint amountOutMin,
			  address[] calldata path,
			  address to,
			  uint deadline
			) external returns (uint[] memory amounts);
		沿着由路径确定的路线，将精确数量的输入标记交换为尽可能多的输出标记。path 的第一个元素是输入Token，最后一个是输出Token，任何中间元素都表示要交易的中间对（例如，如果不存在直接对）。

		- `msg.sender` 应该已经在输入Token上给了路由器至少 amountIn 的余量

		名字|类型|描述
		---|---|---
		amountIn|`uint `|要发送的输入Token的数量
		amountOutMin|`uint `|为使交易不恢复而必须接收的最小输出Token数量。
		path| `address[] calldata`|Token地址数组。path.length必须 >= 2。每对连续地址的池必须存在并具有流动性
		to|`address `|输出Token的接收者
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		amounts|`uint[] memory`|	输入Token数量和所有后续输出Token数量。
	- swapTokensForExactTokens

			function swapTokensForExactTokens(
			  uint amountOut,
			  uint amountInMax,
			  address[] calldata path,
			  address to,
			  uint deadline
			) external returns (uint[] memory amounts);	
		沿着由路径确定的路线，为尽可能少的输入Token接收准确数量的输出Token。path 的第一个元素是输入Token，最后一个是输出Token，任何中间元素都表示要交易的中间Token（例如，如果不存在直接对）。

		- `msg.sender` 应该已经在输入Token上给了路由器至少 `amountInMax` 的余量。

		名字|类型|描述
		---|---|---
		amountOut|`uint `|要接收的输出Token数量
		amountInMax|`uint `|在交易恢复之前可能需要的最大输入Token数量
		path|`address[] calldata`|Token地址数组。`path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性
		to|`address `|输出Token的接收者
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		amounts|`uint[] memory`|输入Token数量和所有后续输出Token数量
	- swapExactETHForTokens

			function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline)
			  external
			  payable
			  returns (uint[] memory amounts);	
		沿着由路径确定的路线，为尽可能多的输出Token交换准确数量的 ETH。path 的第一个元素必须是 [WETH](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#weth)，最后一个是输出Token，任何中间元素都代表要交易的中间对（例如，如果不存在直接对）。
		
		名字|类型|描述
		---|---|---
		msg.value (amountIn)|`uint `|要发送的 ETH 数量。
		amountOutMin|`uint `|为使交易不恢复而必须接收的最小输出Token数量。
		path|`address[] calldata`|Token地址数组。 `path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性。
		to|`address `|输出Token的接收者。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		amounts|`uint[] memory`|输入Token数量和所有后续输出Token数量。
	- swapTokensForExactETH

			function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline)
			  external
			  returns (uint[] memory amounts);	
		沿着由路径确定的路线，以尽可能少的输入Token接收准确数量的 ETH。path 的第一个元素是输入Token，最后一个元素必须是 [WETH](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#weth)，并且任何中间元素都表示要交易的中间对（例如，如果不存在直接对）。

		- `msg.sender` 应该已经在输入Token上给了路由器至少 amountInMax 的余量。
		- 如果收件人地址是智能合约，它必须具有接收 ETH 的能力。

		名字|类型|描述
		---|---|---
		amountOut|`uint `|要接收的 ETH 数量。
		amountInMax|`uint `|在交易恢复之前可能需要的最大输入Token数量。
		path|`address[] calldata`|Token地址数组。`path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性。
		to|`address `|ETH 的接收者。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		amounts|`uint[] memory`|输入Token数量和所有后续输出Token数量。
	- swapExactTokensForETH

			function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)
			  external
			  returns (uint[] memory amounts);	
		沿着由路径确定的路线，用尽可能多的 ETH 交换准确数量的Token。path 的第一个元素是输入Token，最后一个元素必须是 [WETH](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#weth)，并且任何中间元素都表示要交易的中间对（例如，如果不存在直接对）。

		- 如果收件人地址是智能合约，它必须具有接收 ETH 的能力。

		名字|类型|描述
		---|---|---
		amountIn|`uint `|要发送的输入Token的数量。
		amountOutMin|`uint `|为使交易不恢复而必须接收的最小输出Token数量。
		path|`address[] calldata`|Token地址数组。`path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性。
		to|`address `|ETH 的接收者。
		deadline|`uint `|	交易将恢复的 Unix 时间戳。
		amounts|`uint[] memory`|输入Token数量和所有后续输出Token数量。
	- swapETHForExactTokens

			function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline)
			  external
			  payable
			  returns (uint[] memory amounts);
		沿着由路径确定的路线，以尽可能少的 ETH 接收准确数量的Token。path 的第一个元素必须是 [WETH](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#weth)，最后一个是输出Token，任何中间元素都代表要交易的中间对（例如，如果不存在直接对）。

		- 剩余的 ETH（如果有）将返回给 `msg.sender`.

		名字|类型|描述
		---|---|---
		amountOut|`uint `|要接收的Token数量。
		msg.value (amountInMax)|`uint `|交易恢复之前可能需要的最大 ETH 数量。
		path|`address[] calldata`|Token地址数组。`path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性。
		to|`address `|	输出Token的接收者。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
		amounts|`uint[] memory`|输入Token数量和所有后续输出Token数量。
	- swapExactTokensForTokensSupportingFeeOnTransferTokens

			function swapExactTokensForTokensSupportingFeeOnTransferTokens(
			  uint amountIn,
			  uint amountOutMin,
			  address[] calldata path,
			  address to,
			  uint deadline
			) external;

		与 [swapExactTokensForTokens](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#swapexacttokensfortokens) 相同，但对于收取转账费用的Token会成功。

		- `msg.sender` 应该已经在输入Token上给了路由器至少 amountIn 的余量。

		名字|类型|描述
		---|---|---
		amountIn|`uint `|要发送的输入Token的数量。
		amountOutMin|`uint `|为使交易不恢复而必须接收的最小输出Token数量。
		path|`address[] calldata`|Token地址数组。`path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性。
		to|`address `|输出Token的接收者。
		deadline|`uint `|	交易将恢复的 Unix 时间戳。
	- swapExactETHForTokensSupportingFeeOnTransferTokens
	
			function swapExactETHForTokensSupportingFeeOnTransferTokens(
			  uint amountOutMin,
			  address[] calldata path,
			  address to,
			  uint deadline
			) external payable;
		与 [swapExactETHForTokens](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#swapexactethfortokens)相同，但对于收取转账费用的Token会成功。
		
		名字|类型|描述
		---|---|---
		msg.value (amountIn)|`uint `|要发送的 ETH 数量。
		amountOutMin|`uint `|为使交易不恢复而必须接收的最小输出Token数量。
		path|`address[] calldata`|Token地址数组。`path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性。
		to|`address `|输出Token的接收者。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
	- swapExactTokensForETHSupportingFeeOnTransferTokens

			function swapExactTokensForETHSupportingFeeOnTransferTokens(
			  uint amountIn,
			  uint amountOutMin,
			  address[] calldata path,
			  address to,
			  uint deadline
			) external;	
		与 [swapExactTokensForETH](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#swapexacttokensforeth)相同，但对于收取转账费用的Token会成功。

		- 如果收件人地址是智能合约，它必须具有接收 ETH 的能力。
		
		名字|类型|描述
		---|---|---
		amountIn|`uint `|要发送的输入Token的数量。
		amountOutMin|`uint `|为使交易不恢复而必须接收的最小输出Token数量。
		path|`address[] calldata`|Token地址数组。`path.length` 必须 >= 2。每对连续地址的池必须存在并具有流动性。
		to|`address `|ETH 的接收者。
		deadline|`uint `|交易将恢复的 Unix 时间戳。
- 接口

		import '@uniswap/v2-periphery/contracts/interfaces/IUniswapV2Router02.sol';
		
		pragma solidity >=0.6.2;
		
		interface IUniswapV2Router01 {
		    function factory() external pure returns (address);
		    function WETH() external pure returns (address);
		
		    function addLiquidity(
		        address tokenA,
		        address tokenB,
		        uint amountADesired,
		        uint amountBDesired,
		        uint amountAMin,
		        uint amountBMin,
		        address to,
		        uint deadline
		    ) external returns (uint amountA, uint amountB, uint liquidity);
		    function addLiquidityETH(
		        address token,
		        uint amountTokenDesired,
		        uint amountTokenMin,
		        uint amountETHMin,
		        address to,
		        uint deadline
		    ) external payable returns (uint amountToken, uint amountETH, uint liquidity);
		    function removeLiquidity(
		        address tokenA,
		        address tokenB,
		        uint liquidity,
		        uint amountAMin,
		        uint amountBMin,
		        address to,
		        uint deadline
		    ) external returns (uint amountA, uint amountB);
		    function removeLiquidityETH(
		        address token,
		        uint liquidity,
		        uint amountTokenMin,
		        uint amountETHMin,
		        address to,
		        uint deadline
		    ) external returns (uint amountToken, uint amountETH);
		    function removeLiquidityWithPermit(
		        address tokenA,
		        address tokenB,
		        uint liquidity,
		        uint amountAMin,
		        uint amountBMin,
		        address to,
		        uint deadline,
		        bool approveMax, uint8 v, bytes32 r, bytes32 s
		    ) external returns (uint amountA, uint amountB);
		    function removeLiquidityETHWithPermit(
		        address token,
		        uint liquidity,
		        uint amountTokenMin,
		        uint amountETHMin,
		        address to,
		        uint deadline,
		        bool approveMax, uint8 v, bytes32 r, bytes32 s
		    ) external returns (uint amountToken, uint amountETH);
		    function swapExactTokensForTokens(
		        uint amountIn,
		        uint amountOutMin,
		        address[] calldata path,
		        address to,
		        uint deadline
		    ) external returns (uint[] memory amounts);
		    function swapTokensForExactTokens(
		        uint amountOut,
		        uint amountInMax,
		        address[] calldata path,
		        address to,
		        uint deadline
		    ) external returns (uint[] memory amounts);
		    function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline)
		        external
		        payable
		        returns (uint[] memory amounts);
		    function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline)
		        external
		        returns (uint[] memory amounts);
		    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)
		        external
		        returns (uint[] memory amounts);
		    function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline)
		        external
		        payable
		        returns (uint[] memory amounts);
		
		    function quote(uint amountA, uint reserveA, uint reserveB) external pure returns (uint amountB);
		    function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) external pure returns (uint amountOut);
		    function getAmountIn(uint amountOut, uint reserveIn, uint reserveOut) external pure returns (uint amountIn);
		    function getAmountsOut(uint amountIn, address[] calldata path) external view returns (uint[] memory amounts);
		    function getAmountsIn(uint amountOut, address[] calldata path) external view returns (uint[] memory amounts);
		}
		
		interface IUniswapV2Router02 is IUniswapV2Router01 {
		    function removeLiquidityETHSupportingFeeOnTransferTokens(
		        address token,
		        uint liquidity,
		        uint amountTokenMin,
		        uint amountETHMin,
		        address to,
		        uint deadline
		    ) external returns (uint amountETH);
		    function removeLiquidityETHWithPermitSupportingFeeOnTransferTokens(
		        address token,
		        uint liquidity,
		        uint amountTokenMin,
		        uint amountETHMin,
		        address to,
		        uint deadline,
		        bool approveMax, uint8 v, bytes32 r, bytes32 s
		    ) external returns (uint amountETH);
		
		    function swapExactTokensForTokensSupportingFeeOnTransferTokens(
		        uint amountIn,
		        uint amountOutMin,
		        address[] calldata path,
		        address to,
		        uint deadline
		    ) external;
		    function swapExactETHForTokensSupportingFeeOnTransferTokens(
		        uint amountOutMin,
		        address[] calldata path,
		        address to,
		        uint deadline
		    ) external payable;
		    function swapExactTokensForETHSupportingFeeOnTransferTokens(
		        uint amountIn,
		        uint amountOutMin,
		        address[] calldata path,
		        address to,
		        uint deadline
		    ) external;
		}
- ABI

		import IUniswapV2Router02 from '@uniswap/v2-periphery/build/IUniswapV2Router02.json'
	[https://unpkg.com/@uniswap/v2-periphery@1.1.0-beta.0/build/IUniswapV2Router02.json](https://unpkg.com/@uniswap/v2-periphery@1.1.0-beta.0/build/IUniswapV2Router02.json)	
						  
#### 常见错误
本文档涵盖了在 Uniswap V2 上构建时经常遇到的一些错误代码。

- UniswapV2: K

	这是一个经常遇到的错误，需要一些上下文来理解它。
	
	Uniswap 常数乘积公式为 `X * Y = K`。其中 X 和 Y 代表两个 ERC-20 Token各自的储备余额，“K”代表储备的乘积。“K”错误所指的正是这个“K”。
	
	本质上，“K”错误意味着尝试进行的交易以某种方式使交易对的储备金少于应有的储备金，因此交易被撤销。
	
	这可能有几个不同的原因 
- 转让Token费用

	最常见的例子是由“转账费用”Token引起的。
	
	- 转让Token包括费用

		在大多数情况下，转账Token的费用会消耗或转移每笔转账的一小部分，这样转账的接收者最终会比发送者提供的要少一些。这被称为转让的“包容性”费用。
		
		在转账Token包含费用的情况下，可以使用路由器合约中以 [“SupportingFeeOnTransfer”](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#swapexacttokensfortokenssupportingfeeontransfertokens) 结尾的相应兑换功能。这些函数通过调整 “amountOutMin” 参数以在计算不变量时检查收件人金额而不是发送金额来成功。
	- 转让Token的专用费用	

		另一种类型的转账Token“专用”费用通过在初次转账后从发送地址发送额外转账来工作。因为路由器合约在计算不变量时无法预料到这种尾随传输，所以交易将要么恢复，要么通过发送主传输而部分成功，但在尾随传输时破坏池。

		在转账Token专属费用的情况下，`SupportingFeeOnTransfer` 功能可能会起作用，但会有一些Token设计成从根本上破坏路由器。如果您在使用这些功能时仍然收到“K”错误，您可能需要创建一个适合您的Token设计的路由器合约的分叉。
- Rebasing Tokens(变基Token)

	K”错误的不太常见的实例是重新定位Token的结果。

	变基Token可以任意改变任何持有其Token的地址的余额。这通常在预先指定的时间间隔内起作用，并且由于在变基Token的经济学中使用了一些变量。

	变基Token通常以两种方式工作	
	
	- Negative Rebasing Tokens(负变基Token)

		更常见的变体是负变基Token，它会压缩Token所有者的余额。因为变基不是由传输触发的，所以路由器无法预期 rebase 何时或如何发生。一旦这样做，该Token对的储备将不平衡，并且下一个与该Token对交易的人将承担由于变基而产生的增量成本。

		不用说，一个令人羡慕的位置。

		负变基Token通过更改其Token合约以在涉及 Uniswap 路由器合约的每笔交易结束时调用交易对上的[同步](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair#sync)来解决此错误。那些对路由器合约分叉感兴趣的人应该预料到，在Token合约更新以适应您的新路由器之前，负变基Token将破坏该对。
	- Positive Rebasing Tokens(正变基Token)

		正变基Token任意增加Token持有者的余额。当一个正变基发生时，它会产生交易对中未计入的盈余。由于交易对中的额外Token下落不明，因此任何人都可以在交易对上调用 `skim()` 并有效地窃取重新平衡的正差。

		虽然正向再平衡不会破坏 Uniswap 的任何功能，但对它们感兴趣的人应该知道，在任何Token对中发现的正向余额都可以免费获取。
	- 关于变基Token

		对于那些对构建变基Token感兴趣的人，请注意：许多涉及去中心化交易和流动性供应的合约将在与您的Token交互时中断。可以在 [CHAI](https://chai.money/about.html) 中找到一个示例方法，该方法将导致在未来协议中更容易集成。CHAI 使用包含包装器内重新平衡的包装器功能，以便可兑换Token可以轻松集成到许多不同的系统中	
- UniswapV2：锁定

	LOCKED 错误是路由器合约中内置的保护措施，可防止自定义重入合约在交易结束时尝试将恶意代码返回到路由器合约中。
	
	当使用 Ganache CLI 将以太坊主网分叉到本地实例作为开发环境的一部分时，通常会遇到此错误。该错误是 Ganache-Cli 中的一个错误，truffle 团队有望在未来的版本中修复该错误。
	
	只需重新启动本地分叉即可获得临时修复。
- 无法访问存档节点
	
	这是 Metamask 或 Ganache-CLI 的错误。它通常发生在实例化本地分叉并部署合约但有一个失败的事务之后。
	
	可以通过重新启动本地分叉并重置 Metamask 来临时修复。
- UniswapV2：TRANSFER_FAILED

	这意味着核心合约无法将Token发送给收件人。这很可能是由于Token诈骗，Token所有者以允许用户购买Token但不出售Token的方式恶意禁用了转账功能。
- UniswapV2：已过期

	这是由于交易时间过长而无法广播到主网的结果。

	Uniswap 本身并不设置 gas 价格，因此大多数用户默认使用 metamask 中建议的 gas 价格。但是，有时 metamask 会出错，并将 gas 价格设置得太低。如果交换执行时间超过 20 分钟，核心合约将不允许它通过。

	找到准确的汽油价格可能是一个挑战，目前，我们喜欢 [Gas Now](https://www.gasnow.org/)。
- 行动需要积极的储备

	处理事务时出现 VM 异常：操作需要活动预留

	这可能是处理闪存交换时遇到的一个 ganache 错误。我们还没有弄清楚它的来源。
- 无法在前端批准交易

	在极少数情况下，用户无法在 Uniswap 前端批准Token。

	这是因为一些Token合约采取了措施来防御恶意合约，这些合约试图抢先获得批准并窃取用户Token。仅当用户试图将批准限额从预先分配的金额增加到更大的金额时才会发生这种情况，并且只会发生在少数Token合约中。

	解决方案是让用户手动将路由器合约批准数量设置为零，然后设置为他们想要的数量。最简单的方法是通过 Etherscan。	
	
	

				
			
		
		
	

				
					 
			
			

																								
			
				
		

		
		
	  			
						
		
		
			
 

	
	
			
	
			
			
	

			
 	
	
						
	
	
		
			

					


		

		
				
	
		
	
		
		
	
			
		
	
		
