# Uniswap v1 协议
Uniswap V1 是该协议的第一个版本，于 [2018 年 11 月](https://twitter.com/haydenzadams/status/1058376395108376577)在 Devcon 4 上推出。由于其无需许可的性质，它将与以太坊一样存在。

Uniswap 协议在设计时考虑到了简单性，为在以太坊上无缝Exchange ERC20 Token提供了一个接口。通过消除不必要的提取租金和中间商，它可以实现更快、更有效的Exchange。在它进行权衡的地方，权力下放、审查阻力和安全性是优先考虑的。

Uniswap 是开源的，作为一种公共产品发挥作用。没有中央Token或平台税率。对早期投资者、采用者或开发商没有特殊待遇。 Token 上市是开放和免费的。所有智能合约功能都是公开的，所有升级都是可选的。

该站点将作为 Uniswap 的项目概述——解释它是如何工作的，如何使用它以及如何在它之上构建。这些文档正在积极处理中，更多信息将持续添加。

## V1 功能
- 使用 [Uniswap工厂](https://github.com/Uniswap/uniswap-v1/blob/master/contracts/uniswap_factory.vy)添加对任何 ERC20 Token 的支持
- 加入流动性池以收取 ETH-ERC20 对的税率
- [使用恒定乘积公式](https://github.com/runtimeverification/verified-smart-contracts/blob/uniswap/uniswap/x-y-k.pdf)对流动性敏感的自动定价
- 将 ETH 交易为任何 ERC20，无需包装
- 在单笔交易中将任何 ERC20 换成任何 ERC20
- 在单笔交易中交易和转账到不同的地址
- 任何去中心化exchange的提供最低 gas 成本
- 支持私人和自定义 uniswap exchange
- 使用 ENS 从任何钱包购买 ERC20 Token
- 用 Vyper [编写](https://github.com/runtimeverification/verified-smart-contracts/tree/uniswap/uniswap)的经过部分验证的智能合约
- 移动优化的开源[前端实现](https://github.com/Uniswap/uniswap-interface)
- 通过[以太坊基金会资助](https://blog.ethereum.org/2018/08/17/ethereum-foundation-grants-update-wave-3/)

### 资源
- [网站](https://uniswap.org/)
- [GitHub](https://github.com/Uniswap)
- [twitter](https://twitter.com/Uniswap)
- [reddit](https://www.reddit.com/r/Uniswap)
- [电子邮件](contact@uniswap.org)
- [白皮书](https://hackmd.io/s/HJ9jLsfTz)

### 这个怎么运作
Uniswap 由一系列 ETH-ERC20 Exchange合约组成。每个 ERC20 Token 只有一个Exchange合约。如果一个 Token 还没有 exchange，任何人都可以使用 Uniswap 工厂合约创建它。该工厂用作公共注册表，用于查找添加到系统中的所有 token 和Exchange地址。

每个 exchange 合约都持有 ETH 及其相关 ERC20 Token 的储备金。任何人都可以成为 exchange 合约的流动性提供者并为其储备金做出贡献。这与买卖不同；它需要存入等值的 ETH 和相关的 ERC20 Token。流动性汇集在所有提供者之间，内部 “Token 池”（ERC20）用于跟踪每个提供者的相对贡献。Token 池是在流动性存入系统时铸造的并且可以随时销毁以提取一定比例的储备金。

exchange 合约是 ETH-ERC20 对之间的自动做市商。交易者可以通过增加一个的流动性储备金并从另一个的储备金中提取来在两个方向之间进行Exchange。由于 ETH 是所有 ERC20 exchange 合约的通用Token对，它可以用作中介，允许在单笔交易中直接进行 ERC20-ERC20 交易。如果用户想要在与用于进行交易的地址不同的地址接收购买的 Token，则可以指定收件人地址。

Uniswap 使用 “恒定乘积” 做市公式，该公式根据 ETH 和 ERC20 储备金的相对规模以及传入交易改变该比率的数量来设定汇率。购买 ERC20 Token 出售 ETH ，会增加 ETH 储备金的规模并减少 ERC20 储备金的规模。这会改变准备金率，增加 ERC20 Token 相对于 ETH 的价格以进行后续交易。交易相对于储备金的总规模越大，价格滑点就会越多。本质上，exchange 合约使用公开金融市场来决定Token对的相对价值并将其用作做市策略。

从每笔交易中扣除少量流动性提供者税率( 0.30% )并添加到准备金中。虽然 ETH-ERC20 准备金率不断变化，但税率可确保总的综合准备金规模随着每笔交易而增加。这起到了向流动性提供者支付的作用，当他们烧掉他们的池 Token 以提取他们在总储备金中的一部分时，他们会收集这些款项。价格波动带来的有保证的套利机会应该会推动系统稳定的交易流，并增加产生的税率收入。

由于 Uniswap 完全在链上，因此价格可能会在交易签署和包含在区块中之间发生变化。交易者可以通过指定卖出订单的最低买入量或买入订单的最高卖出量来约束价格波动。这就像一个限价单，如果没有成交，它会自动取消。也可以设置交易截止日期，如果订单执行速度不够快，将取消订单。

每个 Token 只能向工厂注册一个 exchange 的原因是鼓励供应商将他们的流动性集中到一个储备金中。然而，Uniswap 已经内置了对 ERC20 到 ERC20 交易的支持，在交易的一侧使用来自工厂的公共池，在另一侧使用自定义的用户指定池。自定义池可以有基金经理，使用替代定价机制，取消流动性提供者税率，整合复杂的三维基于 fomo 的庞氏骗局等等。他们只需要实现 Uniswap 接口并接受 ETH 作为中介资产。自定义池不具有与公共池相同的安全属性。建议用户仅与经过审计的开源智能合约进行交互。

升级抗审查、去中心化的智能合约很困难。如果对系统进行了重大改进，将发布新版本。流动性提供者可以选择迁移到新系统或留在旧系统中。如果可能的话，新版本将向后兼容，并能够与旧版本进行 ERC20 到 ERC20 之间的交易，类似于自定义池。

### 如何使用它
[uniswap.org](https://uniswap.org/) 是 Uniswap 协议的登录页面。它描述了项目并将用户引导到他们需要去的地方。

Uniswap 智能合约存在于以太坊上。任何人都可以直接与他们互动。

Uniswap 前端是一个开源界面，旨在改善与智能合约交互时的用户体验。任何人都可以使用源代码来托管界面或构建自己的界面。托管接口独立于 Uniswap，并应遵守其管辖法律和法规。
## 连接到 Uniswap
Uniswap 智能合约存在于以太坊区块链上。使用 [ethers.js](https://github.com/ethers-io/ethers.js/) 或 [web3.js](https://github.com/ethereum/web3.js) 将您的网站连接到以太坊。用户将需要支持 web3 的浏览器。

在桌面上，这意味着使用 MetaMask 扩展或类似的东西。在移动设备上，兼容 web3 的浏览器包括 [Trust Wallet](https://trustwalletapp.com/)和 [Coinbase Wallet](https://wallet.coinbase.com/)。请参阅 [ethereum.org](https://ethereum.org/use/#_3-what-is-a-wallet-and-which-one-should-i-use) 了解更多信息。
### 工厂合约
Uniswap [工厂合约](https://github.com/Uniswap/uniswap-v1/blob/master/contracts/uniswap_factory.vy) 可用于为任何尚未拥有的 ERC20 Token 创建Exchange 合约。它还充当已添加到系统中的 ERC20 token的注册表以及与它们相关联的交易所。

工厂合约可以使用[工厂地址](https://etherscan.io/address/0xc0a47dFe034B400B47bDaD5FecDa2621de6c4d95)和 ABI 实例化

- 工厂地址

		// mainnet 主网
		const factory = '0xc0a47dFe034B400B47bDaD5FecDa2621de6c4d95'
		
		// testnets 测试网
		const ropsten = '0x9c83dCE8CA20E9aAF9D3efc003b2ea62aBC08351'
		const rinkeby = '0xf5D915570BC477f9B8D6C0E980aA81757A3AaC36'
		const kovan = '0xD3E51Ef092B2845f10401a0159B2B96e8B6c3D30'
		const görli = '0x6Ce570d02D73d4c384b46135E87f8C592A8c86dA'
- 工厂接口

	在 web3 中创建工厂接口需要工厂地址和工厂 ABI
	
		const factoryABI = [
		  {
		    name: 'NewExchange',
		    inputs: [
		      { type: 'address', name: 'token', indexed: true },
		      { type: 'address', name: 'exchange', indexed: true },
		    ],
		    anonymous: false,
		    type: 'event',
		  },
		  {
		    name: 'initializeFactory',
		    outputs: [],
		    inputs: [{ type: 'address', name: 'template' }],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 35725,
		  },
		  {
		    name: 'createExchange',
		    outputs: [{ type: 'address', name: 'out' }],
		    inputs: [{ type: 'address', name: 'token' }],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 187911,
		  },
		  {
		    name: 'getExchange',
		    outputs: [{ type: 'address', name: 'out' }],
		    inputs: [{ type: 'address', name: 'token' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 715,
		  },
		  {
		    name: 'getToken',
		    outputs: [{ type: 'address', name: 'out' }],
		    inputs: [{ type: 'address', name: 'exchange' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 745,
		  },
		  {
		    name: 'getTokenWithId',
		    outputs: [{ type: 'address', name: 'out' }],
		    inputs: [{ type: 'uint256', name: 'token_id' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 736,
		  },
		  {
		    name: 'exchangeTemplate',
		    outputs: [{ type: 'address', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 633,
		  },
		  {
		    name: 'tokenCount',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 663,
		  },
		]
		
		const factoryContract = new web3.eth.Contract(factoryABI, factoryAddress)

### Exchange 合约
- 获取 Exchange 合约地址

	每个 ERC20 Token 都有一个单独的Exchange合约。工厂合约中的 `getExchange` 方法可用于查找与 ERC20 token地址关联的以太坊地址。
	
		const exchangeAddress = factoryContract.methods.getExchange(tokenAddress)
	如果返回值是 `0x0000000000000000000000000000000000000000` 则 Token 还没有Exchange
- Exchange	接口

	在 web3 中创建Exchange接口需要Exchange地址和Exchange ABI：
	
		const exchangeABI = [
		  {
		    name: 'TokenPurchase',
		    inputs: [
		      { type: 'address', name: 'buyer', indexed: true },
		      { type: 'uint256', name: 'eth_sold', indexed: true },
		      { type: 'uint256', name: 'tokens_bought', indexed: true },
		    ],
		    anonymous: false,
		    type: 'event',
		  },
		  {
		    name: 'EthPurchase',
		    inputs: [
		      { type: 'address', name: 'buyer', indexed: true },
		      { type: 'uint256', name: 'tokens_sold', indexed: true },
		      { type: 'uint256', name: 'eth_bought', indexed: true },
		    ],
		    anonymous: false,
		    type: 'event',
		  },
		  {
		    name: 'AddLiquidity',
		    inputs: [
		      { type: 'address', name: 'provider', indexed: true },
		      { type: 'uint256', name: 'eth_amount', indexed: true },
		      { type: 'uint256', name: 'token_amount', indexed: true },
		    ],
		    anonymous: false,
		    type: 'event',
		  },
		  {
		    name: 'RemoveLiquidity',
		    inputs: [
		      { type: 'address', name: 'provider', indexed: true },
		      { type: 'uint256', name: 'eth_amount', indexed: true },
		      { type: 'uint256', name: 'token_amount', indexed: true },
		    ],
		    anonymous: false,
		    type: 'event',
		  },
		  {
		    name: 'Transfer',
		    inputs: [
		      { type: 'address', name: '_from', indexed: true },
		      { type: 'address', name: '_to', indexed: true },
		      { type: 'uint256', name: '_value', indexed: false },
		    ],
		    anonymous: false,
		    type: 'event',
		  },
		  {
		    name: 'Approval',
		    inputs: [
		      { type: 'address', name: '_owner', indexed: true },
		      { type: 'address', name: '_spender', indexed: true },
		      { type: 'uint256', name: '_value', indexed: false },
		    ],
		    anonymous: false,
		    type: 'event',
		  },
		  {
		    name: 'setup',
		    outputs: [],
		    inputs: [{ type: 'address', name: 'token_addr' }],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 175875,
		  },
		  {
		    name: 'addLiquidity',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'min_liquidity' },
		      { type: 'uint256', name: 'max_tokens' },
		      { type: 'uint256', name: 'deadline' },
		    ],
		    constant: false,
		    payable: true,
		    type: 'function',
		    gas: 82605,
		  },
		  {
		    name: 'removeLiquidity',
		    outputs: [
		      { type: 'uint256', name: 'out' },
		      { type: 'uint256', name: 'out' },
		    ],
		    inputs: [
		      { type: 'uint256', name: 'amount' },
		      { type: 'uint256', name: 'min_eth' },
		      { type: 'uint256', name: 'min_tokens' },
		      { type: 'uint256', name: 'deadline' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 116814,
		  },
		  {
		    name: '__default__',
		    outputs: [],
		    inputs: [],
		    constant: false,
		    payable: true,
		    type: 'function',
		  },
		  {
		    name: 'ethToTokenSwapInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'min_tokens' },
		      { type: 'uint256', name: 'deadline' },
		    ],
		    constant: false,
		    payable: true,
		    type: 'function',
		    gas: 12757,
		  },
		  {
		    name: 'ethToTokenTransferInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'min_tokens' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		    ],
		    constant: false,
		    payable: true,
		    type: 'function',
		    gas: 12965,
		  },
		  {
		    name: 'ethToTokenSwapOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_bought' },
		      { type: 'uint256', name: 'deadline' },
		    ],
		    constant: false,
		    payable: true,
		    type: 'function',
		    gas: 50455,
		  },
		  {
		    name: 'ethToTokenTransferOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_bought' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		    ],
		    constant: false,
		    payable: true,
		    type: 'function',
		    gas: 50663,
		  },
		  {
		    name: 'tokenToEthSwapInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_sold' },
		      { type: 'uint256', name: 'min_eth' },
		      { type: 'uint256', name: 'deadline' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 47503,
		  },
		  {
		    name: 'tokenToEthTransferInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_sold' },
		      { type: 'uint256', name: 'min_eth' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 47712,
		  },
		  {
		    name: 'tokenToEthSwapOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'eth_bought' },
		      { type: 'uint256', name: 'max_tokens' },
		      { type: 'uint256', name: 'deadline' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 50175,
		  },
		  {
		    name: 'tokenToEthTransferOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'eth_bought' },
		      { type: 'uint256', name: 'max_tokens' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 50384,
		  },
		  {
		    name: 'tokenToTokenSwapInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_sold' },
		      { type: 'uint256', name: 'min_tokens_bought' },
		      { type: 'uint256', name: 'min_eth_bought' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'token_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 51007,
		  },
		  {
		    name: 'tokenToTokenTransferInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_sold' },
		      { type: 'uint256', name: 'min_tokens_bought' },
		      { type: 'uint256', name: 'min_eth_bought' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		      { type: 'address', name: 'token_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 51098,
		  },
		  {
		    name: 'tokenToTokenSwapOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_bought' },
		      { type: 'uint256', name: 'max_tokens_sold' },
		      { type: 'uint256', name: 'max_eth_sold' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'token_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 54928,
		  },
		  {
		    name: 'tokenToTokenTransferOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_bought' },
		      { type: 'uint256', name: 'max_tokens_sold' },
		      { type: 'uint256', name: 'max_eth_sold' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		      { type: 'address', name: 'token_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 55019,
		  },
		  {
		    name: 'tokenToExchangeSwapInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_sold' },
		      { type: 'uint256', name: 'min_tokens_bought' },
		      { type: 'uint256', name: 'min_eth_bought' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'exchange_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 49342,
		  },
		  {
		    name: 'tokenToExchangeTransferInput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_sold' },
		      { type: 'uint256', name: 'min_tokens_bought' },
		      { type: 'uint256', name: 'min_eth_bought' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		      { type: 'address', name: 'exchange_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 49532,
		  },
		  {
		    name: 'tokenToExchangeSwapOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_bought' },
		      { type: 'uint256', name: 'max_tokens_sold' },
		      { type: 'uint256', name: 'max_eth_sold' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'exchange_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 53233,
		  },
		  {
		    name: 'tokenToExchangeTransferOutput',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'uint256', name: 'tokens_bought' },
		      { type: 'uint256', name: 'max_tokens_sold' },
		      { type: 'uint256', name: 'max_eth_sold' },
		      { type: 'uint256', name: 'deadline' },
		      { type: 'address', name: 'recipient' },
		      { type: 'address', name: 'exchange_addr' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 53423,
		  },
		  {
		    name: 'getEthToTokenInputPrice',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [{ type: 'uint256', name: 'eth_sold' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 5542,
		  },
		  {
		    name: 'getEthToTokenOutputPrice',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [{ type: 'uint256', name: 'tokens_bought' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 6872,
		  },
		  {
		    name: 'getTokenToEthInputPrice',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [{ type: 'uint256', name: 'tokens_sold' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 5637,
		  },
		  {
		    name: 'getTokenToEthOutputPrice',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [{ type: 'uint256', name: 'eth_bought' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 6897,
		  },
		  {
		    name: 'tokenAddress',
		    outputs: [{ type: 'address', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1413,
		  },
		  {
		    name: 'factoryAddress',
		    outputs: [{ type: 'address', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1443,
		  },
		  {
		    name: 'balanceOf',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [{ type: 'address', name: '_owner' }],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1645,
		  },
		  {
		    name: 'transfer',
		    outputs: [{ type: 'bool', name: 'out' }],
		    inputs: [
		      { type: 'address', name: '_to' },
		      { type: 'uint256', name: '_value' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 75034,
		  },
		  {
		    name: 'transferFrom',
		    outputs: [{ type: 'bool', name: 'out' }],
		    inputs: [
		      { type: 'address', name: '_from' },
		      { type: 'address', name: '_to' },
		      { type: 'uint256', name: '_value' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 110907,
		  },
		  {
		    name: 'approve',
		    outputs: [{ type: 'bool', name: 'out' }],
		    inputs: [
		      { type: 'address', name: '_spender' },
		      { type: 'uint256', name: '_value' },
		    ],
		    constant: false,
		    payable: false,
		    type: 'function',
		    gas: 38769,
		  },
		  {
		    name: 'allowance',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [
		      { type: 'address', name: '_owner' },
		      { type: 'address', name: '_spender' },
		    ],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1925,
		  },
		  {
		    name: 'name',
		    outputs: [{ type: 'bytes32', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1623,
		  },
		  {
		    name: 'symbol',
		    outputs: [{ type: 'bytes32', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1653,
		  },
		  {
		    name: 'decimals',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1683,
		  },
		  {
		    name: 'totalSupply',
		    outputs: [{ type: 'uint256', name: 'out' }],
		    inputs: [],
		    constant: true,
		    payable: false,
		    type: 'function',
		    gas: 1713,
		  },
		]
		
		const exchangeContract = new web3.eth.Contract(exchangeABI, exchangeAddress)

### Token 合约
一些 Uniswap 交互需要直接调用 ERC20 Token 合约而不是与它们关联的 Exchange。
#### 获取 Token
工厂合约中的 `getToken` 方法可用于查找与 Exchange 合约关联的 ERC20  Token 地址。将 ERC20 Token 添加到 Uniswap 或检查 Token 合约的有效性没有进入障碍。前端接口应该维护一个有效的 ERC20 Token 列表，用户可以安全地交易或允许用户粘贴到任意地址。

	const tokenAddress = factoryContract.methods.getToken(exchangeAddress)
如果返回值是 `0x0000000000000000000000000000000000000000` 输入地址，则 Uniswap 不可Exchange。
#### Token
在 web3 中创建 Token 接口需要 Token 地址和 Token ABI

	const tokenABI = [
	  {
	    name: 'Transfer',
	    inputs: [
	      { type: 'address', name: '_from', indexed: true },
	      { type: 'address', name: '_to', indexed: true },
	      { type: 'uint256', name: '_value', indexed: false },
	    ],
	    anonymous: false,
	    type: 'event',
	  },
	  {
	    name: 'Approval',
	    inputs: [
	      { type: 'address', name: '_owner', indexed: true },
	      { type: 'address', name: '_spender', indexed: true },
	      { type: 'uint256', name: '_value', indexed: false },
	    ],
	    anonymous: false,
	    type: 'event',
	  },
	  {
	    name: '__init__',
	    outputs: [],
	    inputs: [
	      { type: 'bytes32', name: '_name' },
	      { type: 'bytes32', name: '_symbol' },
	      { type: 'uint256', name: '_decimals' },
	      { type: 'uint256', name: '_supply' },
	    ],
	    constant: false,
	    payable: false,
	    type: 'constructor',
	  },
	  {
	    name: 'deposit',
	    outputs: [],
	    inputs: [],
	    constant: false,
	    payable: true,
	    type: 'function',
	    gas: 74279,
	  },
	  {
	    name: 'withdraw',
	    outputs: [{ type: 'bool', name: 'out' }],
	    inputs: [{ type: 'uint256', name: '_value' }],
	    constant: false,
	    payable: false,
	    type: 'function',
	    gas: 108706,
	  },
	  {
	    name: 'totalSupply',
	    outputs: [{ type: 'uint256', name: 'out' }],
	    inputs: [],
	    constant: true,
	    payable: false,
	    type: 'function',
	    gas: 543,
	  },
	  {
	    name: 'balanceOf',
	    outputs: [{ type: 'uint256', name: 'out' }],
	    inputs: [{ type: 'address', name: '_owner' }],
	    constant: true,
	    payable: false,
	    type: 'function',
	    gas: 745,
	  },
	  {
	    name: 'transfer',
	    outputs: [{ type: 'bool', name: 'out' }],
	    inputs: [
	      { type: 'address', name: '_to' },
	      { type: 'uint256', name: '_value' },
	    ],
	    constant: false,
	    payable: false,
	    type: 'function',
	    gas: 74698,
	  },
	  {
	    name: 'transferFrom',
	    outputs: [{ type: 'bool', name: 'out' }],
	    inputs: [
	      { type: 'address', name: '_from' },
	      { type: 'address', name: '_to' },
	      { type: 'uint256', name: '_value' },
	    ],
	    constant: false,
	    payable: false,
	    type: 'function',
	    gas: 110600,
	  },
	  {
	    name: 'approve',
	    outputs: [{ type: 'bool', name: 'out' }],
	    inputs: [
	      { type: 'address', name: '_spender' },
	      { type: 'uint256', name: '_value' },
	    ],
	    constant: false,
	    payable: false,
	    type: 'function',
	    gas: 37888,
	  },
	  {
	    name: 'allowance',
	    outputs: [{ type: 'uint256', name: 'out' }],
	    inputs: [
	      { type: 'address', name: '_owner' },
	      { type: 'address', name: '_spender' },
	    ],
	    constant: true,
	    payable: false,
	    type: 'function',
	    gas: 1025,
	  },
	  {
	    name: 'name',
	    outputs: [{ type: 'bytes32', name: 'out' }],
	    inputs: [],
	    constant: true,
	    payable: false,
	    type: 'function',
	    gas: 723,
	  },
	  {
	    name: 'symbol',
	    outputs: [{ type: 'bytes32', name: 'out' }],
	    inputs: [],
	    constant: true,
	    payable: false,
	    type: 'function',
	    gas: 753,
	  },
	  {
	    name: 'decimals',
	    outputs: [{ type: 'uint256', name: 'out' }],
	    inputs: [],
	    constant: true,
	    payable: false,
	    type: 'function',
	    gas: 783,
	  },
	]

	const tokenContract = new web3.eth.Contract(tokenABI, tokenAddress)

## 池流动性，形式化模型
Uniswap 流动资金池是自治的，并使用恒定乘积  `( x * y = k) ` 做市商。该模型被形式化，智能合约实现通过了轻量级形式验证

- [正式规范](https://github.com/runtimeverification/verified-smart-contracts/blob/uniswap/uniswap/x-y-k.pdf)
- [轻量验证](https://github.com/runtimeverification/verified-smart-contracts/tree/uniswap/uniswap/results)

### 创建 Exchange
该 `createExchange` 功能用于为尚未拥有的 ERC20  Token 部署 Exchange 合约

	factory.methods.createExchange(tokenAddress).send()
创建 Exchange 后，可以使用 [getExchange](https://docs.uniswap.org/protocol/V1/guides/connect-to-uniswap/#get-exchange-address) 检索地址 
### 交换(Exchange)储备		
每个 Exchange 合约都持有 ETH 及其相关 ERC20 Token 的流动性储备金。

- ETH 储备金

	与 ERC20 token交易所相关的 ETH 储备金是 Exchange 智能合约的 ETH 余额

		const ethReserve = web3.eth.getBalance(exchangeAddress)
- ERC 20 储备金

	与 ERC20 Token Exchange 相关的 ERC20 储备金是 Exchange 智能合约的 ERC20 余额
	
		const tokenReserve = tokenContract.methods.balanceOf(exchangeAddress)

### 增加流动性
任何人都可以通过调用该 `addLiquidity` 函数加入 Uniswap 流动资金池

	exchange.methods.addLiquidity(min_liquidity, max_tokens, deadline).send({ value: ethAmount })
增加流动性需要将等值的 ETH 和 ERC20 token存入 ERC20 token的相关交易合约中。

第一个加入池的流动性提供者通过存入他们认为是等值的 ETH 和 ERC20 token来设定初始汇率。如果该比率关闭，套利交易者将以牺牲初始流动性提供者为代价使价格达到平衡。

所有未来的流动性提供者都使用他们存入时的汇率存入 ETH 和 ERC20。如果汇率不好，就会有一个有利可图的套利机会来修正价格。
#### 参数
- `ethAmount` 
	
	`ethAmount`  发送到的 `addLiquidity` 是将存入流动性储备金的 ETH 数量。它应该是流动性提供者希望存入储备金的总价值的 50%。

	由于流动性提供者必须以当前汇率存入，Uniswap 智能合约用于 `ethAmount` 确定必须存入的 ERC20 Token 数量。该 Token 金额是流动性提供者希望存入的总价值的剩余 50%。
-  `max_tokens`	
	
	由于交易签署和在以太坊上执行之间的汇率可能会发生变化，因此 `max_tokens` 用于限制该汇率可能波动的金额。对于第一个流动性提供者，`max_tokens` 是所存 Token 的数量。
- `min_liquidity` 

	铸造流动性 Token 是为了跟踪每个流动性提供者贡献的总储备的相对比例。`min_liquidity` 与流动性 Token 的铸造率结合使用 `max_tokens` 和 `ethAmount` 限制铸造率。对于第一个流动性提供者，`min_liquidity` 不做任何事情，可以设置为 0。

事务 `deadline` 用于设置一个时间，在该时间之后事务不能再执行。这限制了 “免费选项” (free option) 问题，以太坊矿工可以持有已签署的交易并根据市场走势执行它们。
### 移除流动性
流动性提供者使用该 `removeLiquidity` 功能提取他们的准备金部分

	exchange.methods.removeLiquidity(amount, min_eth, min_tokens, deadline).send()
流动性的提取比例与提取时的准备金相同。如果汇率不好，就会有一个有利可图的套利机会来修正价格。
#### 参数
- `amount` 

	指定将被销毁的流动性token的数量。将此金额除以总流动性token供应量得出提供者提取的 ETH 和 ER20 储备金的百分比。
- `min_eth` 和 `min_tokens` 

	由于汇率在交易签署和在以太坊上执行之间可能会发生变化，用于限制该汇率可能会波动的金额。 
- `addLiquidity` , `deadline` 

	用于设置事务不能再执行的时间。	

## 交易 Token
在 Uniswap 中，每个 ERC20 Token 都有一个单独的 Exchange 合约。这些 Exchange 持有 ETH 及其相关 ERC20 的储备金。用户无需等待在订单簿中匹配，而是可以随时针对储备金进行交易。储备金汇集在分散的流动性提供者网络之间，这些流动性提供者在每笔交易中收取费用。

定价是自动的，基于 `x * y = k` 做市公式，该公式根据两个储备金的相对规模和传入交易的规模自动调整价格。由于所有 Token 都将 ETH 作为一个公共对共享，因此它被用作任何 ERC20 ⇄ ERC20 对之间直接交易的中介资产。
### ETH ⇄ ERC20 计算
在 ETH 和 ERC20 Token 之间进行交易时确定价格所需的变量是：

- ERC20 Exchange 的 ETH 储备金规模
- ERC20 Exchange 的 ERC20 储备金大小
- 销售量（输入）或购买量（输出）

#### 买入金额-卖单
对于卖单（精确输入），计算买入量（输出）：

	// 出售 ETH 换取 ERC20
	const inputAmount = userInputEthValue
	const inputReserve = web3.eth.getBalance(exchangeAddress)
	const outputReserve = tokenContract.methods.balanceOf(exchangeAddress).call()
	
	// 出售 ERC20 换取 ETH
	const inputAmount = userInputTokenValue
	const inputReserve = tokenContract.methods.balanceOf(exchangeAddress).call()
	const outputReserve = web3.eth.getBalance(exchangeAddress)
	
	// 输出可买数量
	const numerator = inputAmount * outputReserve * 997
	const denominator = inputReserve * 1000 + inputAmount * 997
	const inputAmount = numerator / denominator 
	
#### 卖出金额-买单
对于买单（精确输出），计算成本（输入）：

	//用 ETH 购买 ERC20
	const outputAmount = userInputTokenValue
	const inputReserve = web3.eth.getBalance(exchangeAddress)
	const outputReserve = tokenContract.methods.balanceOf(exchangeAddress).call()
	
	// 使用 ERC20 购买 ETH
	const outputAmount = userInputEthValue
	const inputReserve = tokenContract.methods.balanceOf(exchangeAddress).call()
	const outputReserve = web3.eth.getBalance(exchangeAddress)
	
	// 成本计算
	const numerator = outputAmount * inputReserve * 1000
	const denominator = (outputReserve - outputAmount) * 997
	const inputAmount = numerator / denominator + 1
	
#### 流动性提供者的费用
价格公式中包含 0.3% 的流动性提供者费用。可以这样计算：

	fee = inputAmount * 0.003
#### 汇率
汇率只是输出量除以输入量

	const rate = outputAmount / inputAmount

### ERC20 ⇄ ERC20 计算
在两个 ERC20 Token 之间进行交易时确定价格所需的变量是：

- 输入 ERC20 Exchange 的 ETH 储备金大小
- 输入 ERC20 Exchange 的 ERC20 储备金大小
- 输出 ERC20 Exchange 的 ETH 储备金大小
- 输出 ERC20 Exchange 的 ERC20 储备金大小
- 销售量（输入）或购买量（输出）

#### 买入金额-卖单
对于卖单（精确输入），计算买入量（输出）：

	// TokenA (ERC20) 到 ETH 转换
	const inputAmountA = userInputTokenAValue
	const inputReserveA = tokenContractA.methods.balanceOf(exchangeAddressA).call()
	const outputReserveA = web3.eth.getBalance(exchangeAddressA)
	
	const numeratorA = inputAmountA * outputReserveA * 997
	const denominatorA = inputReserveA * 1000 + inputAmountA * 997
	const outputAmountA = numeratorA / denominatorA
	
	// ETH 到 TokenB (ERC20)  转换
	const inputAmountB = outputAmountA
	const inputReserveB = web3.eth.getBalance(exchangeAddressB)
	const outputReserveB = tokenContract.methods.balanceOf(exchangeAddressB).call()
	
	const numeratorB = inputAmountB * outputReserveB * 997
	const denominatorB = inputReserveB * 1000 + inputAmountB * 997
	const outputAmountB = numeratorB / denominatorB
#### 卖出金额-买单
对于买单（精确输出），计算成本（输入）

	// 使用 ETH 购买 TokenB  (ERC20)
	const outputAmountB = userInputEthValue
	const inputReserveB = web3.eth.getBalance(exchangeAddressB)
	const outputReserveB = tokenContractB.methods.balanceOf(exchangeAddressB).call()
	
	// 成本计算
	const numeratorB = outputAmountB * inputReserveB * 1000
	const denominatorB = (outputReserveB - outputAmountB) * 997
	const inputAmountB = numeratorB / denominatorB + 1
	
	// 用 TokenA  (ERC20) 购买 ETH
	const outputAmountA = userInputEthValue
	const inputReserveA = tokenContractA.methods.balanceOf(exchangeAddressA).call()
	const outputReserveA = web3.eth.getBalance(exchangeAddressA)
	
	// 成本计算
	const numeratorA = outputAmountA * inputReserveA * 1000
	const denominatorA = (outputReserveA - outputAmountA) * 997
	const inputAmountA = numeratorA / denominatorA + 1
#### 流动性提供者费用
在输入 Exchange 从 TokenA (ERC20) 兑换为 ETH 需要 0.30% 的流动性提供者费用。将剩余的 ETH 兑换成 TokenB (ERC20) 需要另外支付 0.3% 的流动性提供者费用

	const exchangeAFee = inputAmountA * 0.003
	const exchangeBFee = inputAmountB * 0.003
由于用户只输入 Token A(ERC20) ，因此可以将其表示为

	const combinedFee = inputAmountA * 0.00591	
#### 汇率
汇率只是输出量除以输入量。

	const rate = outputAmountB / inputAmountA	

### 截止日期
许多 Uniswap 功能包括一个事务，该事务 `deadline` 设置一个时间，在该时间之后事务将不再执行。这限制了矿工长时间持有已签署交易并根据市场走势执行交易。它还减少了由于 Gas 价格问题而需要很长时间才能执行的交易的不确定性。

截止日期是通过将所需的时间量（以秒为单位）添加到最新的以太坊区块时间戳来计算的。

	web3.eth.getBlock('latest', (error, block) => {
	  deadline = block.timestamp + 300 // 交易在 300秒(5分钟) 后到期
	})
### Recipients(收件人)
Uniswap 允许交易者交换 token 并将输出转移到新 `recipient` 地址。这允许一种付款类型，其中付款人发送一个 Token，收款人接收另一个 Token 。

## 自定义链接
### 查询参数
Uniswap 前端支持 URL 查询参数，以允许自定义链接到 Uniswap 交换。用户和开发人员可以使用这些查询参数通过自定义预填充设置链接到 Uniswap Exchange。

每个页面都有可以设置的特定可用 URL 参数。全局参数可以在所有页面上使用。

在错误页面上使用的参数不会影响交换设置。未使用 URL 参数设置的参数将设置为标准交换默认值。
###  全局参数
参数|类型|描述
---|---|---
theme|`String `|将它们设置为深色或浅色模式

- theme

	主题可以设置为 light 或 dark
- 示例用法

	`https://app.uniswap.org/#/swap?theme=dark&use=v1`

### 交换页面
参数|类型|描述
---|---|---
inputCurrency|`address`|输入Token，将交换为输出Token的输入Token。
outputCurrency|`address` or `ETH`|输出Token，输入Token将被交换的输出Token。
slippage|`number `|交易期间使用的最大滑点（以 bips 为单位）
exactAmount|`number `|购买或出售的自定义 Token 数量
exactField|`string `|为其设置自定义 Token 数量的字段。必须是 `input` 或 `output`
#### 默认值
输入Token默认为 ETH 。当为输入或输出选择不同的 Token 时，ETH 将默认为相反的选定Token
#### 约束
地址必须是有效的 ERC20 地址。滑点和金额值必须是 Exchange 接受的有效数字（否则错误将阻止交换）。

滑点可以为 `0` 或在 `10- > 9999 bips` 范围内（转换为 0%、0.01%- > 99% ）

当选择 ETH 作为输出Token时，用户也必须选择一个不是 ETH 的 `inputCurrency` (以防止两个字段都填充ETH)
#### 设定金额
两个参数 `exactField` 和 `exactAmount` 可用于设置要出售或购买的特定token数量。这两个字段都必须在 URL 中设置，否则对设置没有影响。
#### 示例
https://app.uniswap.org/#/swap?exactField=input&exactAmount=10&inputCurrency=0x0F5D2fB29fb7d3CFeE444a200298f468908cC942?use=v1

### 发送页面
发送页面具有与交换页面相同的选项，外加一个附加参数 `recipient`

参数|类型|描述
---|---|---
recipient|`address`|发送交易的收件人地址
#### 示例
https://app.uniswap.org/#/send?recipient=0x74Aa01d162E6dC6A657caC857418C403D48E2D77?use=v1

### 池页面
Pool 页面由 3 个子路由组成：`add-liquidity`、`remove-liquidity`、`create-exchange`
#### 参加流动性
参数|类型|描述
---|---|---
ethAmount|`number`|存入池中的 ETH 数量。
token|`address`|添加流动性的池的 ERC20 地址
tokenAmount|`number`|要存入池中的选定token数量。
#### 示例
https://app.uniswap.org/#/add-liquidity?ethAmount=2.34&token=0x42456D7084eacF4083f1140d3229471bbA2949A8&tokenAmount=300?use=v1

### 移除流动性
参数|类型|描述
---|---|---
poolTokenAddress|`address`|从池中提取流动性。（必须是现有交易所的 ERC20 地址）
poolTokenAmount|`number`|从流动性池中提取的池token数量。
#### 示例
https://app.uniswap.org/#/remove-liquidity?poolTokenAmount=1.23&use=v1

### 创建 Exchange
参数|类型|描述
---|---|---
tokenAddress|`address `|用于创建 Exchange 的 ERC20 Token。必须是没有现有 Exchange 的有效 ERC20 Token
#### 示例
https://app.uniswap.org/#/swap?use=v1&create-exchange?tokenAddress=0x0F5D2fB29fb7d3CFeE444a200298f468908cC942

### 自定义路由
自定义  Token 路由仍然可以与 URL 参数结合使用。URL 参数在设置层次结构中高于自定义路由。

使用自定义 Token 路由和 URL 参数的示例。

https://app.uniswap.org/#/swap/0x0F5D2fB29fb7d3CFeE444a200298f468908cC942?exactField=input&exactAmount=10&use=v1

## iframe 集成
Uniswap 可以在其他站点中用作 iframe。iframe 显示 app.uniswap.org 网站的确切版本，并且可以具有自定义的预填充设置。
### 为什么你可能想要这个
出于多种原因，将 Uniswap 站点直接集成到您的 Web 应用程序中可能很有用。

v1.app.uniswap.org 允许用户购买、出售、发送或提供 ERC20 Token 的流动性。如果您的应用程序围绕这些 ERC20 Token 提供服务，则 iframe 集成可能会很有用。（例如，用户可以通过您网站上的 Uniswap iframe 购买 DAI，然后允许用户在您的网站上借出该 DAI ）。

如果您的应用程序需要用户获取一些 Token 以使用某些服务（例如，允许用户购买“REP”token，以便他们可以在 Augur Dapp 上参与预测市场） ，它也会很有用。
### iframe 与自定义 UI
iframe 集成的一个好处是您的站点将自动跟上 v1.app.uniswap.org 站点的任何改进/添加。设置初始集成后，由于交换站点会随着时间的推移而更新，因此不需要进一步的工作来获取更新。
### 现场示例
可以在 FOAM 网站上找到 Iframe 集成的示例 [https://map.foam.space/](https://map.foam.space/)

要查看 iframe，请单击右上角的下拉菜单，然后单击“获取  foam”。
### 添加到您的网站
要在您的网站中包含 Uniswap iframe，只需在您的网站代码中添加 iframe 元素并链接到 Uniswap 交换。

链接到 ETH < - > DAI 交换页面看起来像这样。要链接到您选择的 Token，请将 `outputCurrency` 之后的地址替换为您要链接到的Token 的 Token 地址。

	<iframe
	  src="https://app.uniswap.org/#/swap?use=v1?outputCurrency=0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359"
	  height="660px"
	  width="100%"
	  style="
	    border: 0;
	    margin: 0 auto;
	    display: block;
	    border-radius: 10px;
	    max-width: 600px;
	    min-width: 300px;
	  "
	  id="myId"
	/>
您可以使用 URL 查询参数自定义所选页面、所选自定义Token等。请参阅[自定义链接](https://docs.uniswap.org/protocol/V1/guides/custom-linking)

##  Token 上市
您感兴趣的 Token 可能不包含在 `https://app.uniswap.org/#/swap?use=v1` 上的 Token 下拉列表中，但是所有部署了 uniswap Exchange 的 Token 都支持前端。

可以通过三种方式与默认列表中尚未包含的 Token 进行交互。

1. 将 Token 地址粘贴到搜索框中

	如果列表中不包含 Token，请尝试将 Token 地址粘贴到搜索框中。它将使用您要查找的 Token 填充下拉列表。
2. 自定义

	[https://app.uniswap.org/#/swap?use=v1](https://app.uniswap.org/#/swap?use=v1)支持自定义链接到具有 Uniswap Exchange 的所有 Token 。有关如何链接的详细信息，请参阅[自定义链接](https://docs.uniswap.org/protocol/V1/guides/custom-linking)。

	例如，要使用未列出的 Token 填充输出 Token 字段，我们可以在 URL 中指定 outputCurrency 并传入 Token 的地址，如下所示：

		https://app.uniswap.org/#/swap?use=v1?outputCurrency=0xfA3E941D1F6B7b10eD84A0C211bfA8aeE907965e

### Token 详情和资产
Token 信息（包括小数、符号、名称等）直接从token合约中提取。徽标图像是从 TrustWallet 中提取的。如果您希望更新您的 Token 徽标，请向 TrustWallet 资产回购 [https://github.com/trustwallet/assets](https://github.com/trustwallet/assets)提出拉取请求。

## ABI 说明
### 工厂
- `initializeFactory`

	参数|描述
	---|---
	template| Exchange 模板的以太坊地址
	
	- 智能合约

			initializeFactory(template: address)
	- web3

			factoryContract.methods.initializeFactory(template).send()
- `createExchange`

	参数|类型|描述
	---|---|---	
	token|`address`|ERC20 Token 的以太坊地址
	
	返回|描述
	---|---
	address|Uniswap  Exchange 的以太坊地址
	
	- 智能合约

			createExchange(token: address): address
	- web3

			factoryContract.methods.createExchange(token).send()
- `getToken`

	参数|类型|描述
	---|---|---	
	exchange| `address`|Uniswap 交易所的以太坊地址
	
	返回|描述
	---|---
	address|ERC20 token的以太坊地址
	
	- 智能合约

			@constant
			getToken(exchange: address): address
	- web3

			factoryContract.methods.getToken(exchange).call()
- `getTokenWithId`

	参数|类型|描述
	---|---|---	
	token_id| uint256|ERC20 Token 的 Uniswap ID
	
	返回|描述
	---|---
	address|ERC20 token的以太坊地址
	
	- 智能合约

			@constant
			getTokenWithId(token_id: uint256): address
	- web3

			factoryContract.methods.getTokenWithId(token_id).call()

### Exchange
- `setup`

	参数|描述
	---|---
	token_addr|ERC20 Token 的以太坊地址
	
	- 智能合约

			// 只能在 createExchange() 期间由工厂合约调用
			setup(token_addr: address):
	- web3

			// 只能在 createExchange() 期间由工厂合约调用
			exchangeContract.methods.setup((token: String)).send()
- `addLiquidity`

	参数|类型|描述
	---|---|---	
	msg.value| uint256|添加的 ETH 数量
	min_liquidity| uint256|最低流动性
	max_tokens| uint256|添加的最大 ERC20 Token 
	deadline| uint256|交易截止日期
	
	返回|描述
	---|---
	uint256 |铸造的流动性 Token 数量
	
	- 智能合约

			@payable
			addLiquidity(
			    min_liquidity: uint256,
			    max_tokens: uint256,
			    deadline: uint256
			): uint256
	- web3

			exchangeContract.methods.addLiquidity(min_liquidity, max_tokens, deadline).send({ value: ethValue })
- `removeLiquidity `
	
	参数|类型|描述
	---|---|---	
	amount| uint256|燃烧的流动性数量
	min_eth| uint256|移除最低 ETH
	min_tokens| uint256|移除最低 ERC20 Token 
	deadline| uint256|交易截止日期
	
	返回|描述
	---|---
	uint256 |移除的 ETH 数量
	uint256|移除的 ERC20 token数量
	
	- 智能合约

			removeLiquidity(
			    amount: uint256;
			    min_eth: uint256,
			    min_tokens: uint256,
			    deadline: uint256
			): (uint256, uint256)
	- web3

			exchangeContract.methods.removeLiquidity(amount, min_eth, min_tokens, deadline).send()
- `default `

	参数|类型|描述
	---|---|---	
	msg.value| uint256|出售的 ETH 数量
	
	- 智能合约

			# Default function in Vyper replaces the "fallback" function in Solidity
			@payable
			__default__():
	- web3

			web3.eth.sendTransaction({ value: ethAmount })
- `ethToTokenSwapInput `

	参数|类型|描述
	---|---|---	
	msg.value| uint256|出售的 ETH 数量
	min_tokens| uint256|最少购买 ERC20 token
	deadline| uint256|交易截止日期
	
	返回|描述
	---|---
	uint256 |购买的 ERC20 Token数量
	
	- 智能合约

			@payable
				ethToTokenSwapInput(
				    min_tokens: uint256,
				    deadline: uint256
				): uint256
	- web3

			exchangeContract.methods.ethToTokenSwapInput(min_liquidity, max_tokens, deadline).send({ value: ethValue })
- `ethToTokenTransferInput `

	参数|类型|描述
	---|---|---	
	msg.value| uint256|出售的 ETH 数量
	min_tokens| uint256|最少购买 ERC20 token
	deadline| uint256|交易截止日期
	recipient| address|接收 ERC20 token的地址
	
	返回|描述
	---|---
	uint256 |购买的 ERC20 Token 数量
	
	- 智能合约

			@payable
				ethToTokenTransferInput(
				    min_tokens: uint256,
				    deadline: uint256,
				    recipient: address
				): uint256
	- web3

			exchangeContract.methods
			  .ethToTokenTransferInput(min_liquidity, max_tokens, deadline, recipient)
			  .send({ value: ethValue })
- `ethToTokenSwapOutput`

	参数|类型|描述
	---|---|---	
	msg.value| uint256|卖出的最大 ETH
	tokens_bought | uint256|购买的 ERC20 Token 数量
	deadline| uint256|交易截止日期
	
	返回|描述
	---|---
	uint256 |出售的 ETH 数量
	
	- 智能合约

			@payable
				ethToTokenSwapOutput(
				    tokens_bought: uint256,
				    deadline: uint256
				): uint256
	- web3

			exchangeContract.methods.ethToTokenSwapOutput(tokens_bought, deadline).send({ value: ethValue })
- `ethToTokenTransferOutput `

	参数|类型|描述
	---|---|---	
	msg.value| uint256|卖出的最大 ETH
	tokens_bought | uint256|购买的 ERC20 Token 数量
	deadline| uint256|交易截止日期
	recipient| address| 接收 ERC20 Token 的地址
	
	返回|描述
	---|---
	uint256 |出售的 ETH 数量
	
	- 智能合约

			@payable
				ethToTokenTransferOutput(
				    tokens_bought: uint256,
				    deadline: uint256,
				    recipient: address
				): uint256
	- web3

			exchangeContract.methods
			  .ethToTokenTransferOutput(tokens_bought, deadline, (recipient: String))
			  .send({ value: ethValue })
- `tokenToEthSwapInput `

	参数|类型|描述
	---|---|---	
	tokens_sold | uint256|售出的 ERC20 token数量
	min_eth | uint256|最低购买 ETH
	deadline| uint256|交易截止日期
	
	返回|描述
	---|---
	uint256 |购买的 ETH 数量
	
	- 智能合约

			tokenToEthSwapInput(
			    tokens_sold: uint256,
			    min_eth: uint256,
			    deadline: uint256
			): uint256
	- web3

			exchangeContract.methods.tokenToEthSwapInput(tokens_sold, min_eth, deadline).send()
- `tokenToEthTransferInput `

	参数|类型|描述
	---|---|---	
	tokens_sold | uint256|售出的 ERC20 token数量
	min_eth | uint256|最低购买 ETH
	deadline| uint256|交易截止日期
	recipient| address|接收 ETH 的地址
	
	返回|描述
	---|---
	uint256 |购买的 ETH 数量
	
	- 智能合约

			tokenToEthTransferInput(
			    tokens_sold: uint256,
			    min_eth: uint256,
			    deadline: uint256,
			    recipient: address
			): uint256
	- web3

			exchangeContract.methods.tokenToEthTransferInput(tokens_sold, min_eth, deadline, recipient).send()
- `tokenToEthSwapOutput`

	参数|类型|描述
	---|---|---
	eth_bought| uint256|购买的 ETH 数量
	max_tokens| uint256|卖出的最大 ERC20 Token
	deadline| uint256| 交易截止日期
	
	返回|描述
	---|---
	uint256 |售出的 ERC20 Token 数量
	
	- 智能合约

			tokenToEthSwapOutput(
			    eth_bought: uint256,
			    max_tokens: uint256,
			    deadline: uint256
			): uint256
	- web3

			exchangeContract.methods.tokenToEthSwapOutput(eth_bought, max_tokens, (deadline: Integer)).send()
- `tokenToEthTransferOutput `

	参数|类型|描述
	---|---|---
	eth_bought| uint256|购买的 ETH 数量
	max_tokens| uint256|卖出的最大 ERC20 Token
	deadline| uint256| 交易截止日期
	recipient| address|接收 ETH 的地址
	
	返回|描述
	---|---
	uint256 |售出的 ERC20 Token 数量
	
	- 智能合约

			tokenToEthTransferOutput(
			    eth_bought: uint256,
			    max_tokens: uint256,
			    deadline: uint256,
			    recipient: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToEthTransferOutput(eth_bought, max_tokens, (deadline: Integer), (recipient: String)) .send()
- `tokenToTokenSwapInput `

	参数|类型|描述
	---|---|---
	tokens_sold | uint256 |已售出的输入 ERC20 token数量
	min_tokens_bought | uint256 |购买的最低产出 ERC20 Token
	min_eth_bought | uint256 | 作为中介购买的最低 ETH
	deadline | uint256 |交易截止日期
	token_addr| address|输出ERC20 Token 地址
	
	返回|描述
	---|---
	uint256 |售出的 ERC20 Token 数量
	
	- 智能合约

			tokenToTokenSwapInput(
			    tokens_sold: uint256,
			    min_tokens_bought: uint256,
			    min_eth_bought: uint256,
			    deadline: uint256,
			    token_addr: address
			): uint256
	- web3

			exchangeContract.methods
			 .tokenToTokenSwapInput(tokens_sold, min_tokens_bought, min_eth_bought, deadline, token_addr).send()
- `tokenToTokenTransferInput `

	参数|类型|描述
	---|---|---
	tokens_sold | uint256 |已售出的输入 ERC20 token数量
	min_tokens_bought | uint256 |购买的最低产出 ERC20 Token
	min_eth_bought | uint256 | 作为中介购买的最低 ETH
	deadline | uint256 |交易截止日期
	recipient| address |接收输出 ERC20 Token 的地址
	token_addr| address|输出 ERC20 Token 地址
	
	返回|描述
	---|---
	uint256 |售出的 ERC20 Token 数量
	
	- 智能合约

			tokenToTokenTransferInput(
			    tokens_sold: uint256,
			    min_tokens_bought: uint256,
			    min_eth_bought: uint256,
			    deadline: uint256,
			    recipient: address
			    token_addr: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToTokenTransferInput(tokens_sold, min_tokens_bought, min_eth_bought, deadline, recipient, token_addr)
			  .send()
- `tokenToTokenSwapOutput `

	参数|类型|描述
	---|---|---
	tokens_bought| uint256|购买的输出 ERC20 Token 数量
	max_tokens_sold| uint256|购买的最大输入 ERC20 Token 
	max_eth_sold| uint256|作为中介购买的最大 ETH
	deadline| uint256|交易截止日期
	token_addr| uint256|输出ERC20 Token 地址
	
	返回|描述
	---|---
	uint256 |已售出的输入 ERC20 Token 数量
	
	- 智能合约

			tokenToTokenSwapOutput(
			    tokens_bought: uint256,
			    max_tokens_sold: uint256,
			    max_eth_sold: uint256,
			    deadline: uint256,
			    token_addr: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToTokenSwapOutput(tokens_bought, max_tokens_sold, max_eth_sold, deadline, token_addr).send()
- `tokenToTokenTransferOutput `

	参数|类型|描述
	---|---|---
	tokens_bought| uint256|购买的输出 ERC20 Token 数量
	max_tokens_sold| uint256|购买的最大输入 ERC20 Token
	max_eth_sold| uint256|作为中介购买的最大 ETH
	deadline| uint256|交易截止日期
	recipient| uint256|接收输出 ERC20 Token 的地址
	token_addr| uint256|输出ERC20 Token 地址
	
	返回|描述
	---|---
	uint256 |已售出的输入 ERC20 Token 数量
	
	- 智能合约

			tokenToTokenTransferOutput(
			    tokens_bought: uint256,
			    max_tokens_sold: uint256,
			    max_eth_sold: uint256,
			    deadline: uint256,
			    recipient: address,
			    token_addr: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToTokenTransferOutput(tokens_bought, max_tokens_sold, max_eth_sold, deadline, recipient, token_addr)
			  .send()
- `tokenToExchangeSwapInput `

	参数|类型|描述
	---|---|---
	tokens_bought| uint256 |已售出的输入 ERC20 token数量
	min_tokens_bought | uint256 |购买的最低产出 ERC20 Token 
	min_eth_bought | uint256 |作为中介购买的最低 ETH
	deadline| uint256 |交易截止日期
	exchange_addr | uint256 |输出 ERC20 Token兑换地址
	
	返回|描述
	---|---
	uint256 |购买的输出 ERC20 Token 数量
	
	- 智能合约

			tokenToTokenSwapInput(
			    tokens_sold: uint256,
			    min_tokens_bought: uint256,
			    min_eth_bought: uint256,
			    deadline: uint256,
			    exchange_addr: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToTokenSwapInput(tokens_sold, min_tokens_bought, min_eth_bought, deadline, exchange_addr)
			  .send()
- `tokenToExchangeTransferInput `

	参数|类型|描述
	---|---|---	
	tokens_sold| uint256|已售出的输入 ERC20 token数量
	min_tokens_bought| uint256|购买的最低产出 ERC20 token
	min_eth_bought| uint256|作为中介购买的最低 ETH
	deadline| uint256|交易截止日期
	recipient| address|接收输出 ERC20 Token 的地址
	exchange_addr| address|输出ERC20 Token兑换地址
	
	返回|描述
	---|---
	uint256 |购买的输出 ERC20 Token 数量
	
	- 智能合约

			tokenToExchangeTransferInput(
			    tokens_sold: uint256,
			    min_tokens_bought: uint256,
			    min_eth_bought: uint256,
			    deadline: uint256,
			    recipient: address
			    exchange_addr: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToExchangeTransferInput(tokens_sold, min_tokens_bought, min_eth_bought, deadline, recipient, exchange_addr)
			  .send()	
- `tokenToExchangeSwapOutput`	

	参数|类型|描述
	---|---|---	
	tokens_bought | uint256|购买的输出 ERC20 token 数量
	max_tokens_sold | uint256|购买的最大输入 ERC20 token
	max_eth_sold | uint256|作为中介购买的最大 ETH
	deadline | uint256|交易截止日期
	exchange_addr | address|输出 ERC 20token 兑换地址
	
	返回|描述
	---|---
	uint256 |已售出的输入 ERC20 token 数量
	
	- 智能合约

			tokenToExchangeSwapOutput(
			    tokens_bought: uint256,
			    max_tokens_sold: uint256,
			    max_eth_sold: uint256,
			    deadline: uint256,
			    exchange_addr: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToExchangeSwapOutput(tokens_bought, max_tokens_sold, max_eth_sold, deadline, exchange_addr)
			  .send() 
- `tokenToExchangeTransferOutput`

	参数|类型|描述
	---|---|---	
	tokens_bought | uint256|购买的输出 ERC20 token 数量
	max_tokens_sold | uint256|购买的最大输入 ERC20 token
	max_eth_sold | uint256|作为中介购买的最大 ETH
	deadline | uint256|交易截止日期
	recipient | address|接收输出 ERC20 token 的地址
	exchange_addr | address|输出 ERC2 token 兑换地址
	
	返回|描述
	---|---
	uint256 |已售出的输入 ERC20 token 数量
	
	- 智能合约

			tokenToExchangeTransferOutput(
			    tokens_bought: uint256,
			    max_tokens_sold: uint256,
			    max_eth_sold: uint256,
			    deadline: uint256,
			    recipient: address,
			    exchange_addr: address
			): uint256
	- web3

			exchangeContract.methods
			  .tokenToExchangeTransferOutput(tokens_bought, max_tokens_sold, max_eth_sold, deadline, recipient, exchange_addr)
			  .send()
- `getEthToTokenInputPrice `

	参数|类型|描述
	---|---|---	
	eth_sold| uint256|出售的 ETH 数量
	
	返回|描述
	---|---
	uint256 |可购买的 ERC20  Token 数量
	
	- 智能合约

			@constant
			getEthToTokenInputPrice(eth_sold: uint256): uint256
	-  web3

			exchangeContract.methods.getEthToTokenInputPrice(eth_sold).call()
- `getEthToTokenOutputPrice `	

	参数|类型|描述
	---|---|---	
	eth_sold | uint256|购买的 ERC20 Token 数量
	
	返回|描述
	---|---
	uint256 |必须出售的 ETH 数量
	
	- 智能合约

			@constant
			getEthToTokenOutputPrice(tokens_bought: uint256): uint256
	- web3	

			exchangeContract.methods.getEthToTokenOutputPrice(tokens_bought).call()
- `getTokenToEthInputPrice `			
	
	参数|类型|描述
	---|---|---	
	tokens_sold | uint256|售出的 ERC20 Token 数量
	
	返回|描述
	---|---
	uint256 |可购买的 ETH 数量
	
	- 智能合约

			@constant
			getTokenToEthInputPrice(tokens_sold: uint256): uint256
	- web3

			exchangeContract.methods.getTokenToEthInputPrice(tokens_sold).call()
- `getTokenToEthOutputPrice `

	参数|类型|描述
	---|---|---	
	eth_bought | uint256|购买的 ETH 数量
	
	返回|描述
	---|---
	uint256 |必须出售的 ERC20 token 数量
	
	- 智能合约

			@constant
			getTokenToEthOutputPrice(eth_bought: uint256): uint256
	- web3

			exchangeContract.methods.getTokenToEthOutputPrice(eth_bought).call()
- `tokenAddress`

	返回|描述
	---|---
	address |在 Exchange 出售的 ERC20 token 的地址
	
	- 智能合约

			@constant
			tokenAddress(): address
	- web3
			  
			exchangeContract.methods.tokenAddress().call()
- `factoryAddress `

	返回|描述
	---|---
	address |创建 Exchange 的工厂地址
	
	- 智能合约

			@constant
			factoryAddress(): address
	- web3
	
			exchangeContract.methods.factoryAddress().call()
- `name`

	返回|描述
	---|---
	bytes32 |流动性 Token 名称
	
	- 智能合约

			# 所有的 Exchange 合约都有相同的名字
			@constant
			name(): bytes32 // Uniswap V1
	- web3

			exchangeContract.methods.tokenAddress().call()
- `symbol`

	返回|描述
	---|---
	bytes32 |流动性 Token 的象征
	
	- 智能合约

			# 所有的 Exchange 合约都有相同 symbol
			@constant
			symbol(): bytes32 // UNI-V1
	- web3

			exchangeContract.methods.tokenAddress().call()
- `decimals `

	返回|描述
	---|---
	uint256 |流动性 Token 的小数
	
	- 智能合约

			# 所有的 Exchange 合约都有相同的小数点
			@constant
			decimals(): uint256 // 18
	- web3

			exchangeContract.methods.decimals().call()
- `balanceOf `

	参数|类型|描述
	---|---|---	
	_owner| address|以太坊地址
	
	返回|描述
	---|---
	uint256 |地址的流动性 Token 余额
	
	- 智能合约

			@constant
			balanceOf(_owner: address): uint256
	- web3

			exchangeContract.methods.balanceOf(_owner).call()
- `transfer`

	参数|类型|描述
	---|---|---	
	`_to` | address|收件人地址
	`_value`| uint256|转账金额
	
	返回|描述
	---|---
	bool |如果成功则为真。失败时恢复或错误
	
	- 智能合约

			transfer(
			    _to: address,
			    _value : uint256
			): bool
	- web3
			
			exchangeContract.methods.transfer(_to, _value).send()
- `transferFrom `

	参数|类型|描述
	---|---|---	
	`_from ` | address |发件人地址
	`_to`| address |收件人地址
	`_value `| uint256|转账金额
	
	返回|描述
	---|---
	bool |如果成功则为真。失败时恢复或错误
	
	- 智能合约

			transferFrom(
			    _from: address,
			    _to: address,
			    _value : uint256
			): bool
	- web3

			exchangeContract.methods.transferFrom(_from, _to, _value).send()
- `approve`

	参数|类型|描述
	---|---|---
	`_spender `| address|获批消费人地址
	`_value `|uint256|消费津贴
	
	返回|描述
	---|---
	bool |如果成功则为真。失败时恢复或错误
	
	- 智能合约

			approve(
			    _spender: address,
			    _value: uint256
			): bool
	- web3
	
			exchangeContract.methods.approve(_spender, _value).send()
- `allowance `

	参数|类型|描述
	---|---|---
	_owner| address|流动性 Token 所有者地址
	_spender| uint256|获批消费者地址
	
	返回|描述
	---|---
	uint256 |消费津贴
	
	- 智能合约

			allowance(
			    _owner: address,
			    _spender: address
			): uint256
	- web3
	
			exchangeContract.methods.allowance(_owner, _spender).call()

### 接口
#### Factory
- Solidity

		interface UniswapFactoryInterface {
		    // 公共变量
		    address public exchangeTemplate;
		    uint256 public tokenCount;
		    // 创建 Exchange
		    function createExchange(address token) external returns (address exchange);
		    // 获取  Exchange 和 Token 信息
		    function getExchange(address token) external view returns (address exchange);
		    function getToken(address exchange) external view returns (address token);
		    function getTokenWithId(uint256 tokenId) external view returns (address token);
		    // 从不使用
		    function initializeFactory(address template) external;
		}
- Vyper

		contract UniswapFactoryInterface():
		    # Create Exchange
		    def createExchange(token: address) -> address: modifying
		    # Public Variables
		    def exchangeTemplate() -> address: constant
		    def tokenCount() -> uint256: constant
		    # Get Exchange and Token Info
		    def getExchange(token_addr: address) -> address: constant
		    def getToken(exchange: address) -> address: constant
		    def getTokenWithId(token_id: uint256) -> address: constant
		    # Initialize Factory
		    def initializeFactory(template: address): modifying

#### Exchange
- Solidity

		interface UniswapExchangeInterface {
		    // 在该 Exchange 出售的 ERC20 Token 的地址
		    function tokenAddress() external view returns (address token);
		    
		    //Uniswap 工厂地址
		    function factoryAddress() external view returns (address factory);
		    
		    // 提供流动性
		    function addLiquidity(uint256 min_liquidity, uint256 max_tokens, uint256 deadline) external payable returns (uint256);
		    function removeLiquidity(uint256 amount, uint256 min_eth, uint256 min_tokens, uint256 deadline) external returns (uint256, uint256);
		    
		    // 获取价格
		    function getEthToTokenInputPrice(uint256 eth_sold) external view returns (uint256 tokens_bought);
		    function getEthToTokenOutputPrice(uint256 tokens_bought) external view returns (uint256 eth_sold);
		    function getTokenToEthInputPrice(uint256 tokens_sold) external view returns (uint256 eth_bought);
		    function getTokenToEthOutputPrice(uint256 eth_bought) external view returns (uint256 tokens_sold);
		    
		    // 用 eth 交易 erc 20
		    function ethToTokenSwapInput(uint256 min_tokens, uint256 deadline) external payable returns (uint256  tokens_bought);
		    function ethToTokenTransferInput(uint256 min_tokens, uint256 deadline, address recipient) external payable returns (uint256  tokens_bought);
		    function ethToTokenSwapOutput(uint256 tokens_bought, uint256 deadline) external payable returns (uint256  eth_sold);
		    function ethToTokenTransferOutput(uint256 tokens_bought, uint256 deadline, address recipient) external payable returns (uint256  eth_sold);
		    
		    // 用 ERC20 交易 ETH
		    function tokenToEthSwapInput(uint256 tokens_sold, uint256 min_eth, uint256 deadline) external returns (uint256  eth_bought);
		    function tokenToEthTransferInput(uint256 tokens_sold, uint256 min_eth, uint256 deadline, address recipient) external returns (uint256  eth_bought);
		    function tokenToEthSwapOutput(uint256 eth_bought, uint256 max_tokens, uint256 deadline) external returns (uint256  tokens_sold);
		    function tokenToEthTransferOutput(uint256 eth_bought, uint256 max_tokens, uint256 deadline, address recipient) external returns (uint256  tokens_sold);
		    
		    // 用  ERC20 交易 ERC20
		    function tokenToTokenSwapInput(uint256 tokens_sold, uint256 min_tokens_bought, uint256 min_eth_bought, uint256 deadline, address token_addr) external returns (uint256  tokens_bought);
		    function tokenToTokenTransferInput(uint256 tokens_sold, uint256 min_tokens_bought, uint256 min_eth_bought, uint256 deadline, address recipient, address token_addr) external returns (uint256  tokens_bought);
		    function tokenToTokenSwapOutput(uint256 tokens_bought, uint256 max_tokens_sold, uint256 max_eth_sold, uint256 deadline, address token_addr) external returns (uint256  tokens_sold);
		    function tokenToTokenTransferOutput(uint256 tokens_bought, uint256 max_tokens_sold, uint256 max_eth_sold, uint256 deadline, address recipient, address token_addr) external returns (uint256  tokens_sold);
		    
		    // 交易 ERC20 到自定义池
		    function tokenToExchangeSwapInput(uint256 tokens_sold, uint256 min_tokens_bought, uint256 min_eth_bought, uint256 deadline, address exchange_addr) external returns (uint256  tokens_bought);
		    function tokenToExchangeTransferInput(uint256 tokens_sold, uint256 min_tokens_bought, uint256 min_eth_bought, uint256 deadline, address recipient, address exchange_addr) external returns (uint256  tokens_bought);
		    function tokenToExchangeSwapOutput(uint256 tokens_bought, uint256 max_tokens_sold, uint256 max_eth_sold, uint256 deadline, address exchange_addr) external returns (uint256  tokens_sold);
		    function tokenToExchangeTransferOutput(uint256 tokens_bought, uint256 max_tokens_sold, uint256 max_eth_sold, uint256 deadline, address recipient, address exchange_addr) external returns (uint256  tokens_sold);
		    
		    //流动性 Token 的 ERC20 兼容性
		    bytes32 public name;
		    bytes32 public symbol;
		    uint256 public decimals;
		    function transfer(address _to, uint256 _value) external returns (bool);
		    function transferFrom(address _from, address _to, uint256 value) external returns (bool);
		    function approve(address _spender, uint256 _value) external returns (bool);
		    function allowance(address _owner, address _spender) external view returns (uint256);
		    function balanceOf(address _owner) external view returns (uint256);
		    function totalSupply() external view returns (uint256);
		    
		    // Never use
		    function setup(address token_addr) external;
		}
- vyper

		contract UniswapExchangeInterface():
		    # Public Variables
		    def tokenAddress() -> address: constant
		    def factoryAddress() -> address: constant
		    # Providing Liquidity
		    def addLiquidity(min_liquidity: uint256, max_tokens: uint256, deadline: timestamp) -> uint256: modifying
		    def removeLiquidity(amount: uint256, min_eth: uint256(wei), min_tokens: uint256, deadline: timestamp) -> (uint256(wei), uint256): modifying
		    # Trading
		    def ethToTokenSwapInput(min_tokens: uint256, deadline: timestamp) -> uint256: modifying
		    def ethToTokenTransferInput(min_tokens: uint256, deadline: timestamp, recipient: address) -> uint256: modifying
		    def ethToTokenSwapOutput(tokens_bought: uint256, deadline: timestamp) -> uint256(wei): modifying
		    def ethToTokenTransferOutput(tokens_bought: uint256, deadline: timestamp, recipient: address) -> uint256(wei): modifying
		    def tokenToEthSwapInput(tokens_sold: uint256, min_eth: uint256(wei), deadline: timestamp) -> uint256(wei): modifying
		    def tokenToEthTransferInput(tokens_sold: uint256, min_eth: uint256(wei), deadline: timestamp, recipient: address) -> uint256(wei): modifying
		    def tokenToEthSwapOutput(eth_bought: uint256(wei), max_tokens: uint256, deadline: timestamp) -> uint256: modifying
		    def tokenToEthTransferOutput(eth_bought: uint256(wei), max_tokens: uint256, deadline: timestamp, recipient: address) -> uint256: modifying
		    def tokenToTokenSwapInput(tokens_sold: uint256, min_tokens_bought: uint256, min_eth_bought: uint256(wei), deadline: timestamp, token_addr: address) -> uint256: modifying
		    def tokenToTokenTransferInput(tokens_sold: uint256, min_tokens_bought: uint256, min_eth_bought: uint256(wei), deadline: timestamp, recipient: address, token_addr: address) -> uint256: modifying
		    def tokenToTokenSwapOutput(tokens_bought: uint256, max_tokens_sold: uint256, max_eth_sold: uint256(wei), deadline: timestamp, token_addr: address) -> uint256: modifying
		    def tokenToTokenTransferOutput(tokens_bought: uint256, max_tokens_sold: uint256, max_eth_sold: uint256(wei), deadline: timestamp, recipient: address, token_addr: address) -> uint256: modifying
		    def tokenToExchangeSwapInput(tokens_sold: uint256, min_tokens_bought: uint256, min_eth_bought: uint256(wei), deadline: timestamp, exchange_addr: address) -> uint256: modifying
		    def tokenToExchangeTransferInput(tokens_sold: uint256, min_tokens_bought: uint256, min_eth_bought: uint256(wei), deadline: timestamp, recipient: address, exchange_addr: address) -> uint256: modifying
		    def tokenToExchangeSwapOutput(tokens_bought: uint256, max_tokens_sold: uint256, max_eth_sold: uint256(wei), deadline: timestamp, exchange_addr: address) -> uint256: modifying
		    def tokenToExchangeTransferOutput(tokens_bought: uint256, max_tokens_sold: uint256, max_eth_sold: uint256(wei), deadline: timestamp, recipient: address, exchange_addr: address) -> uint256: modifying
		    # Get Price
		    def getEthToTokenInputPrice(eth_sold: uint256(wei)) -> uint256: constant
		    def getEthToTokenOutputPrice(tokens_bought: uint256) -> uint256(wei): constant
		    def getTokenToEthInputPrice(tokens_sold: uint256) -> uint256(wei): constant
		    def getTokenToEthOutputPrice(eth_bought: uint256(wei)) -> uint256: constant
		    # Pool Token ERC20 Compatibility
		    def balanceOf() -> address: constant
		    def allowance(_owner : address, _spender : address) -> uint256: constant
		    def transfer(_to : address, _value : uint256) -> bool: modifying
		    def transferFrom(_from : address, _to : address, _value : uint256) -> bool: modifying
		    def approve(_spender : address, _value : uint256) -> bool: modifying
		    # Setup
		    def setup(token_addr: address): modifying

#### ERC20 Token
- Solidity

		interface ERC20Interface {
		    function totalSupply() public view returns (uint);
		    function balanceOf(address tokenOwner) public view returns (uint balance);
		    function allowance(address tokenOwner, address spender) public view returns (uint remaining);
		    function transfer(address to, uint tokens) public returns (bool success);
		    function approve(address spender, uint tokens) public returns (bool success);
		    function transferFrom(address from, address to, uint tokens) public returns (bool success);
		    // 选项
		    function name() external view returns (string);
		    function symbol() external view returns (string);
		    function decimals() external view returns (string);
		
		    event Transfer(address indexed from, address indexed to, uint tokens);
		    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
		}
- vyper

		contract ERC20Interface():
		    def totalSupply() -> uint256: constant
		    def balanceOf(_owner: address) -> uint256: constant
		    def allowance(_owner : address, _spender : address) -> uint256: constant
		    def transfer(_to : address, _value : uint256) -> bool: modifying
		    def approve(_spender : address, _value : uint256) -> bool: modifying
		    def transferFrom(_from : address, _to : address, _value : uint256) -> bool: modifying
		    # optional
		    def name() -> bytes32: constant
		    def symbol() -> bytes32: constant
		    def decimals() -> uint256: constant						
	
			
	
		
	
							

	
	



			
	
	

				  	
	
	
	
	
	

	
	
	
	
	

	
		
	
					
		
	


			
	
						
	







	
	

					