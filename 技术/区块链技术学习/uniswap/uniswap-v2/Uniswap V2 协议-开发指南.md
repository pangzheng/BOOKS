# uniswap v2-指南
## 指南
### 接口集成
#### 使用 API
在本指南中，我们将创建一个使用和显示来自 Uniswap 子图的数据的 Web 界面。目标是提供一个设置的快速概览，您可以扩展该设置以围绕 Uniswap 数据创建自己的 UI 和分析。

许多不同的库可用于创建接口和与子图 graphql 端点的连接，但在本指南中，我们将使用 [React](https://reactjs.org/) 作为接口，使用 [Apollo Client](https://www.apollographql.com/docs/react/) 发送查询。我们还将使用 yarn 进行依赖管理。

- 设置和安装
	
	我们需要为应用程序创建基本骨架。我们将为此使用 [create-react-app](https://reactjs.org/docs/create-a-new-react-app.html)。我们还将添加我们需要的依赖项。在命令行中导航到您的根位置并运行：
	
		yarn create react-app uniswap-demo
		cd uniswap-demo
		yarn add  apollo-client apollo-cache-inmemory apollo-link-http graphql graphql-tag @apollo/react-hooks
		yarn start 
	在您的浏览器中，您应该会看到默认的 React 应用程序正在运行。在文本编辑器 `App.js` 中打开 `src` 并用这个精简的样板替换内容。我们将在我们进行时添加此内容。

		import React from 'react'
		import './App.css'
		
		function App() {
		  return <div></div>
		}
		
		export default App
- Graphql 客户端

	我们需要设置一些中间件，以便向 Uniswap 子图发出请求并接收数据。为此，我们将使用 Apollo 并创建一个 graphql 客户端来处理它。

	- 1 添加如下所示的导入并实例化一个新的客户端实例。请注意我们如何在此处使用指向 Uniswap 子图的链接。

			import React from 'react'
			import ReactDOM from 'react-dom'
			import App from './App'
			import registerServiceWorker from './registerServiceWorker'
			import './index.css'
			import { ApolloProvider } from 'react-apollo'
			import { client } from './App'
			
			ReactDOM.render(
			  <ApolloProvider client={client}>
			    <App />
			  </ApolloProvider>,
			  document.getElementById('root')
			)
			registerServiceWorker()
	- 2 我们还需要添加一个上下文，以便 Apollo 可以正确处理请求。在您的 `index.js` 文件中导入正确的提供程序并将根包装在其中，如下所示：

			import React from 'react'
			import ReactDOM from 'react-dom'
			import App from './App'
			import registerServiceWorker from './registerServiceWorker'
			import './index.css'
			import { ApolloProvider } from 'react-apollo'
			import { client } from './App'
			
			ReactDOM.render(
			  <ApolloProvider client={client}>
			    <App />
			  </ApolloProvider>,
			  document.getElementById('root')
			)
			registerServiceWorker()
- 编写查询

	接下来，我们将构建查询并获取数据。在本例中，我们将获取有关 Uniswap V2 上 Dai Token的一些数据。我们将获得当前价格以及所有Token对的总流动性。我们将在此查询中使用 Dai 地址作为 id。我们还将获取 ETH 的美元价格，以帮助为 Dai 数据创建美元转换。

	- 首先，我们需要定义查询本身。我们将用于 `gql` 将查询字符串解析为 GraphQL AST 标准。将帮助程序导入 `gql` 应用程序并使用它来创建查询。将以下内容添加到您的App.js文件中：

			import gql from 'graphql-tag'
			
			const DAI_QUERY = gql`
			  query tokens($tokenAddress: Bytes!) {
			    tokens(where: { id: $tokenAddress }) {
			      derivedETH
			      totalLiquidity
			    }
			  }
			`
			
			const ETH_PRICE_QUERY = gql`
			  query ethPrice {
			    bundle(id: "1") {
			      ethPrice
			    }
			  }
		我们使用 1 捆绑包的 id ，因为子图中只有一个硬编码捆绑包。
- 获取数据

	现在我们准备好使用这些查询从 Uniswap V2 子图中获取数据。为此，我们可以使用 `useQuery` 使用我们的客户端实例获取数据的钩子，并为我们提供有关请求状态的实时信息。为此，请将以下内容添加到您的 `App.js` 文件中：
	
		import { useQuery } from '@apollo/react-hooks'
		
		const { loading, error, data: ethPriceData } = useQuery(ETH_PRICE_QUERY)
		const {
		  loading: daiLoading,
		  error: daiError,
		  data: daiData,
		} = useQuery(DAI_QUERY, {
		  variables: {
		    tokenAddress: '0x6b175474e89094c44da98b954eedeac495271d0f',
		  },
		})
	请注意，我们使用 Dai Token地址来获取有关 Dai 的数据。

- 格式化响应数据

	现在我们有了数据，我们可以对其进行格式化并将其显示在 UI 中。首先，我们解析返回数据以获得我们想要的实际数据。然后我们将使用它来获取 Dai 的美元价格。最后，我们将把这些数据插入 UI 本身。

	这些查询将为每个查询返回一个响应对象。在每一个中，我们都对我们在查询定义中定义的根字段感兴趣。对于 `daiData` 响应，我们将其定义为`tokens`，对于 `ethPriceData` 查询，我们将其定义为 `ethPrice`。在每一个中，我们都会得到一系列结果。因为我们只查询单个实体，所以我们将引用 `0` 数据数组中的索引。

	将以下行添加到您的 `App.js` 文件中以解析响应：
	
		const daiPriceInEth = daiData && daiData.tokens[0].derivedETH
		const daiTotalLiquidity = daiData && daiData.tokens[0].totalLiquidity
		const ethPriceInUSD = ethPriceData && ethPriceData.bundles[0].ethPrice
- 显示 UI

	最后，我们可以使用我们解析的响应数据来聚合 UI。我们将分两步进行。

	- 1 首先，我们将创建加载状态。要检测查询是否仍在等待响应，我们可以引用我们已经定义的加载变量。我们将添加两种加载状态，
		- 一种用于 Dai 价格
		- 一种用于 Dai 总流动性。这些可能会快速闪烁，因为查询的时间很快。
	- 2 填充加载的数据。一旦我们检测到查询已经完成加载，我们就可以用真实数据填充 UI。

	为此，在 `App.js` 文件的返回函数中添加以下行：
	
		return (
		  <div>
		    <div>
		      Dai price:{' '}
		      {ethLoading || daiLoading
		        ? 'Loading token data...'
		        : '$' +
		          // parse responses as floats and fix to 2 decimals
		          (parseFloat(daiPriceInEth) * parseFloat(ethPriceInUSD)).toFixed(2)}
		    </div>
		    <div>
		      Dai total liquidity:{' '}
		      {daiLoading
		        ? 'Loading token data...'
		        : // display the total amount of DAI spread across all pools
		          parseFloat(daiTotalLiquidity).toFixed(0)}
		    </div>
		  </div>
		)		
- 后续步骤

	这应该会呈现一个非常基本的页面，其中包含有关 Uniswap 中 Dai Token的这两个统计信息。这是您可以使用 Uniswap 子图做什么的一个非常基本的示例，我们鼓励您构建更复杂和有趣的工具！

	您可以访问我们的[分析站点](https://uniswap.info/)以查看更高级的分析页面，或访问 [github](https://github.com/Uniswap/uniswap-info) 以获取使用 Uniswap 子图创建 UI 的更详细示例。
- 回顾

	最后你的 `App.js` 文件应该是这样的
	
		import React, { useEffect } from 'react'
		import './App.css'
		import { ApolloClient } from 'apollo-client'
		import { InMemoryCache } from 'apollo-cache-inmemory'
		import { HttpLink } from 'apollo-link-http'
		import { useQuery } from '@apollo/react-hooks'
		import gql from 'graphql-tag'
		
		export const client = new ApolloClient({
		  link: new HttpLink({
		    uri: 'https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v2',
		  }),
		  fetchOptions: {
		    mode: 'no-cors',
		  },
		  cache: new InMemoryCache(),
		})
		
		const DAI_QUERY = gql`
		  query tokens($tokenAddress: Bytes!) {
		    tokens(where: { id: $tokenAddress }) {
		      derivedETH
		      totalLiquidity
		    }
		  }
		`
		
		const ETH_PRICE_QUERY = gql`
		  query bundles {
		    bundles(where: { id: "1" }) {
		      ethPrice
		    }
		  }
		`
		
		function App() {
		  const { loading: ethLoading, data: ethPriceData } = useQuery(ETH_PRICE_QUERY)
		  const { loading: daiLoading, data: daiData } = useQuery(DAI_QUERY, {
		    variables: {
		      tokenAddress: '0x6b175474e89094c44da98b954eedeac495271d0f',
		    },
		  })
		
		  const daiPriceInEth = daiData && daiData.tokens[0].derivedETH
		  const daiTotalLiquidity = daiData && daiData.tokens[0].totalLiquidity
		  const ethPriceInUSD = ethPriceData && ethPriceData.bundles[0].ethPrice
		
		  return (
		    <div>
		      <div>
		        Dai price:{' '}
		        {ethLoading || daiLoading
		          ? 'Loading token data...'
		          : '$' +
		            // parse responses as floats and fix to 2 decimals
		            (parseFloat(daiPriceInEth) * parseFloat(ethPriceInUSD)).toFixed(2)}
		      </div>
		      <div>
		        Dai total liquidity:{' '}
		        {daiLoading
		          ? 'Loading token data...'
		          : // display the total amount of DAI spread across all pools
		            parseFloat(daiTotalLiquidity).toFixed(0)}
		      </div>
		    </div>
		  )
		}
		
		export default App

#### 自定义链接
- 查询参数

	Uniswap 前端支持 URL 查询参数，以允许自定义链接到 Uniswap 前端。用户和开发人员可以使用这些查询参数通过自定义预填充设置链接到 Uniswap 前端。
	
	每个页面都有可以设置的特定可用 URL 参数。全局参数可以在所有页面上使用。
	
	在错误页面上使用的参数不会影响前端设置。未使用 URL 参数设置的参数将设置为标准前端默认值。
	
	- 全局

		范围|类型|描述
		---|---|---
		theme|`string`|将它们设置为深色或浅色模式
	
		- theme
			
			主题可以设置为light或dark
		- 示例

			[https://app.uniswap.org/#/swap?theme=dark](https://app.uniswap.org/#/swap?theme=dark)
	- Swap

		范围|类型|描述
		---|---|---
		inputCurrency|`address`|将交换为输出Token的输入Token。
		outputCurrency|`address or ETH`|输入Token将被交换的输出Token。
		exactAmount|`number`|购买或出售的自定义Token数量。
		exactField|`string`|为其设置自定义Token数量的字段。必须是input或output。
		
		- 默认

			ETH 默认为输入Token。当为输入或输出选择不同的Token时，ETH 将默认为相反的选定Token。
		- 约束

			地址必须是有效的 ERC20 地址。滑点和金额值必须是前端接受的有效数字（否则错误将阻止交换）。滑点可以为 0 或在 10->9999 bips 范围内（转换为 0%、0.01%- > 99% ）
			
			当选择 ETH 作为输出Token时，用户也必须选择一个不是 ETH 的 `inputCurrency` (以防止两个字段都填充ETH)
		- 设置金额

			两个参数，`exactField` 和 `exactAmount` 可用于设置要出售或购买的特定Token数量。这两个字段都必须在 URL 中设置，否则对设置没有影响。	
		- 示例

			[https://app.uniswap.org/#/swap?exactField=input&exactAmount=10&inputCurrency=0x0F5D2fB29fb7d3CFeE444a200298f468908cC942](https://app.uniswap.org/#/swap?exactField=input&exactAmount=10&inputCurrency=0x0F5D2fB29fb7d3CFeE444a200298f468908cC942)
	- pool

		Pool 页面由 2 个子路由组成：`add`, `remove`.
		
		- 增加流动性

			范围|类型|描述
			---|---|---
			Token0|`address`|从池中提取流动性。（必须是带有现有Token的 ERC20 地址）
			Token1|`address `|从池中提取流动性。（必须是带有现有Token的 ERC20 地址）
		
			- 例

				[https://app.uniswap.org/#/add/v2/0x6B175474E89094C44Da98b954EedeAC495271d0F/0xdAC17F958D2ee523a2206206994597C13D831ec7](https://app.uniswap.org/#/add/v2/0x6B175474E89094C44Da98b954EedeAC495271d0F/0xdAC17F958D2ee523a2206206994597C13D831ec7)
		- 移除流动性	

			范围|类型|描述
			---|---|---
			Token0|`address`|从池中提取流动性。（必须是带有现有Token的 ERC20 地址）
			Token1|`address `|从池中提取流动性。（必须是带有现有Token的 ERC20 地址）
		
			- 例		

				[https://app.uniswap.org/#/remove/0x6B175474E89094C44Da98b954EedeAC495271d0F-0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2](https://app.uniswap.org/#/remove/0x6B175474E89094C44Da98b954EedeAC495271d0F-0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2)

#### iframe 集成
Uniswap 可以在其他站点中用作 iframe。iframe 显示了 uniswap 前端站点的精确版本，并且可以具有自定义的预填充设置。

- 为什么你可能想要这个

	出于多种原因，将 Uniswap 站点直接集成到您的 Web 应用程序中可能很有用。

	该界面允许用户购买、出售、发送或提供 ERC20 Token的流动性。如果您的应用程序围绕这些 ERC20 Token提供服务，则 iframe 集成可能会很有用。（例如，用户可以通过您网站上的 Uniswap iframe 购买 DAI，然后允许用户在您的网站上借出该 DAI ）。

	如果您的应用程序需要用户获取一些Token以使用某些服务（例如，允许用户购买“REP”Token，以便他们可以在 Augur Dapp 上参与预测市场） ，它也会很有用。
- iframe 与自定义 UI

	iframe 集成的一个好处是您的网站将自动跟上网站的任何改进/添加。设置初始集成后，由于交换站点会随着时间的推移而更新，因此不需要进一步的工作来获取更新。

	例子
	
		<iframe
		  src="https://app.uniswap.org/#/swap?exactField=input&exactAmount=10&inputCurrency=0x6b175474e89094c44da98b954eedeac495271d0f"
		  height="660px"
		  width="100%"
		  style="
		    border: 0;
		    margin: 0 auto;
		    margin-bottom: .5rem;
		    display: block;
		    border-radius: 10px;
		    max-width: 960px;
		    min-width: 300px;
		  "
		/>

	可以在 FOAM 网站查看[集成示例](https://map.foam.space/#/at/?lng=-74.0045300&lat=40.6771800&zoom=5.00)，要查看 iframe，请单击右上角的下拉菜单，然后单击 `Get FOAM`。
- 添加到您的网站

	要在您的网站中包含 Uniswap iframe，只需在您的网站代码中添加一个 iframe 元素并链接到 Uniswap 前端。

	链接到 ETH < - > DAI 交换页面看起来像这样。要链接到您选择的Token，请将后面的地址替换为您要链接到的Token `outputCurrency` 的Token地址。	
	
		<iframe
		  src="https://app.uniswap.org/#/swap?outputCurrency=0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359"
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
		/>
	您可以使用 URL 查询参数自定义所选页面、所选自定义Token等。请参阅[自定义链接](https://docs.uniswap.org/protocol/V2/guides/interface-integration/custom-interface-linking)

### 智能合约集成
#### 智能合约快速入门
为以太坊开发智能合约涉及一系列用于生成和测试在[以太坊虚拟机 (EVM)](https://eth.wiki/en/concepts/evm/ethereum-virtual-machine-(evm)-awesome-list)上运行的字节码的链下工具。一些工具还包括将此字节码部署到以太坊网络和测试网的工作流程。这些工具有很多选择。本指南将引导您使用一组特定的工具 ( `truffle` ++ `npm`)编写和测试与 Uniswap 协议交互的简单智能合约`mocha`

- 要求

	要遵循本指南，您必须安装以下内容：

	- [nodejs >= v12.x & npm >= 6.x](https://nodejs.org/en/)
- 引导项目

	您可以从头开始，但使用诸如 `truffle` 引导空项目之类的工具会更容易。创建一个空目录并 `npx truffle init` 在该目录中运行以将默认的 [Truffle box](https://www.trufflesuite.com/boxes) 拆箱
	
		mkdir demo
		cd demo
		npx truffle init
- 设置 NPM

	为了引用 Uniswap V2 合约，您应该使用我们部署的包含核心和外围智能合约和接口的 npm 工件。要添加 npm 依赖项，我们首先初始化 npm 包。我们可以 `npm init` 在同一目录下运行创建 `package.json` 文件。您可以接受所有默认值并稍后更改。
	
		npm init	
- 添加依赖

	现在我们有了一个 npm 包，我们可以添加我们的依赖项。让我们同时添加 [@uniswap/v2-core](https://www.npmjs.com/package/@uniswap/v2-core) 和 [@uniswap/v2-periphery](https://www.npmjs.com/package/@uniswap/v2-periphery) 包。

		npm i --save @uniswap/v2-core
		npm i --save @uniswap/v2-periphery		
	如果您检查 `node_modules/@uniswap` 目录，您现在可以找到 Uniswap V2 合约

		moody@MacBook-Pro ~/I/u/demo> ls node_modules/@uniswap/v2-core/contracts
		UniswapV2ERC20.sol    UniswapV2Pair.sol     libraries/
		UniswapV2Factory.sol  interfaces/           test/
		moody@MacBook-Pro ~/I/u/demo> ls node_modules/@uniswap/v2-periphery/contracts/
		UniswapV2Migrator.sol  examples/              test/
		UniswapV2Router01.sol  interfaces/
		UniswapV2Router02.sol  libraries/	
	这些包包括智能合约源代码和构建工件。
-  写自己的合约

	我们现在可以开始编写示例合约了。对于编写 Solidity，我们建议使用带有 Solidity 插件的 IntelliJ 或 VSCode，但您可以使用任何文本编辑器。让我们编写一个合约，返回给定Token对的一定数量的流动性份额的价值。
	
	首先创建几个文件：

		mkdir contracts/interfaces
		touch contracts/interfaces/ILiquidityValueCalculator.sol
		touch contracts/LiquidityValueCalculator.sol
	这将是我们实现的合约的接口。把它放进去 `contracts/interfaces/ILiquidityValueCalculator.sol`

		pragma solidity ^0.6.6;
		
		interface ILiquidityValueCalculator {
		    function computeLiquidityShareValue(uint liquidity, address tokenA, address tokenB) external returns (uint tokenAAmount, uint tokenBAmount);
		}
	现在让我们从构造函数开始。您需要知道 `UniswapV2Factory` 部署在哪里才能计算Token对的地址并查找流动性份额的总供应量以及储备金的数量。我们可以将其存储为传递给构造函数的地址。

	工厂地址在主网和所有测试网上都是不变的，所以在你的合约中让这个值成为一个常量可能很诱人，但由于我们需要对合约进行单元测试，所以它应该是一个参数。访问此变量时，您可以使用 solidity 不可变来节省 gas。

		pragma solidity ^0.6.6;
		
		import './interfaces/ILiquidityValueCalculator.sol';
		
		contract LiquidityValueCalculator is ILiquidityValueCalculator {
		    address public factory;
		    constructor(address factory_) public {
		        factory = factory_;
		    }
		}
	现在我们需要能够查看Token对的流动性总供应量及其Token余额。让我们把它放在一个单独的函数中。为了实现它，我们必须：

	- 查找配对地址
	- 获取对的储备金
	- 获取Token对流动性的总供应量
	- 按照 tokenA、tokenB 的顺序对储备金进行排序

	对此 [UniswapV2Library](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/library) 有一些有用的方法。

		pragma solidity ^0.6.6;
		
		import './interfaces/ILiquidityValueCalculator.sol';
		import '@uniswap/v2-periphery/contracts/libraries/UniswapV2Library.sol';
		import '@uniswap/v2-core/contracts/interfaces/IUniswapV2Pair.sol';
		
		contract LiquidityValueCalculator is ILiquidityValueCalculator {
		    function pairInfo(address tokenA, address tokenB) internal view returns (uint reserveA, uint reserveB, uint totalSupply) {
		        IUniswapV2Pair pair = IUniswapV2Pair(UniswapV2Library.pairFor(factory, tokenA, tokenB));
		        totalSupply = pair.totalSupply();
		        (uint reserves0, uint reserves1,) = pair.getReserves();
		        (reserveA, reserveB) = tokenA == pair.token0() ? (reserves0, reserves1) : (reserves1, reserves0);
		    }
		}
	最后，我们只需要计算份额值。我们将把它作为练习留给读者

		pragma solidity ^0.6.6;
		
		import './interfaces/ILiquidityValueCalculator.sol';
		import '@uniswap/v2-periphery/contracts/libraries/UniswapV2Library.sol';
		import '@uniswap/v2-core/contracts/interfaces/IUniswapV2Pair.sol';
		
		contract LiquidityValueCalculator is ILiquidityValueCalculator {
		    address public factory;
		    constructor(address factory_) public {
		        factory = factory_;
		    }
		
		    function pairInfo(address tokenA, address tokenB) internal view returns (uint reserveA, uint reserveB, uint totalSupply) {
		        IUniswapV2Pair pair = IUniswapV2Pair(UniswapV2Library.pairFor(factory, tokenA, tokenB));
		        totalSupply = pair.totalSupply();
		        (uint reserves0, uint reserves1,) = pair.getReserves();
		        (reserveA, reserveB) = tokenA == pair.token0() ? (reserves0, reserves1) : (reserves1, reserves0);
		    }
		
		    function computeLiquidityShareValue(uint liquidity, address tokenA, address tokenB) external override returns (uint tokenAAmount, uint tokenBAmount) {
		        revert('TODO');
		    }
		}	
- 测试合约

	为了测试您的合约，您需要：
	
	- 建立一个测试网
	- 部署 UniswapV2Factory
	- 为一对部署至少 2 个 ERC20 Token
	- 为工厂创建一对
	- 部署你的 `LiquidityValueCalculator` 合约
	- 称呼 `LiquidityValueCalculator#computeLiquidityShareValue`
	- 用断言验证结果

	由 `truffle test` 命令自动为您处理。

	`build` 请注意您应该只在单元测试的目录中部署预编译的 Uniswap 合约。这是因为 solidity 将元数据哈希附加到已编译的合约工件中，其中包括合约源代码路径的哈希，并且在其他机器上编译不会产生完全相同的字节码。这是有问题的，因为在 Uniswap V2 中，我们使用 `v2-periphery` 中字节码的哈希值 [UniswapV2Library](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/libraries/UniswapV2Library.sol#L24) 来计算配对地址。

	要获取用于部署 UniswapV2Factory 的字节码，您可以通过以下方式导入文件：
	
		const UniswapV2FactoryBytecode = require('@uniswap/v2-core/build/UniswapV2Factory.json').bytecode
	我们建议使用标准 `ERC20@openzeppelin/contracts` 来部署 ERC20。

	您可以在[此处](https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript)阅读有关使用 Truffle 部署合约和编写测试的更多信息 。

- 编译和部署合约

	分别在[此处]()和[此处](https://www.trufflesuite.com/docs/truffle/getting-started/running-migrations)了解有关使用 Truffle 编译和部署合约的更多信息 。	
	
#### 实施交换
从智能合约进行交易时，要记住的最重要的事情是需要访问外部价格来源。没有这一点，交易可能会因大量损失而提前交易。

阅读[安全注意事项](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/trading-from-a-smart-contract#safety-considerations)了解更多信息。

- 使用路由器

	安全交换Token的最简单方法是使用[路由器](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02)，它提供了多种安全交换不同资产的方法。您会注意到，每个交换到/从确切数量的 ETH/Token的排列都有一个函数。

	首先，您必须使用外部价格源来计算您要调用的函数的安全参数。这是出售确切输入时收到的最低金额或购买确切输出金额时您愿意支付的最高金额

	同样重要的是要确保你的合约控制足够的 ETH/Token来进行交换，并已批准路由器撤回这么多Token。

	查看[定价](https://docs.uniswap.org/protocol/V2/concepts/advanced-topics/pricing#pricing-trades)页面，了解有关获取价格的更深入讨论。

- 例子

	想象一下，你想从你的智能合约中用 50 DAI 换取尽可能多的 ETH。
	
	- `transferFrom`

		在交换之前，我们的智能合约需要控制 50 DAI。最简单的方法是调用 `transferFrom` DAI 并将所有者设置为 `msg.sender`
				
			uint amountIn = 50 * 10 ** DAI.decimals();
			require(DAI.transferFrom(msg.sender, address(this), amountIn), 'transferFrom failed.');
	- `approve`

		现在我们的合约拥有 50 个 DAI，我们需要批准[路由器](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02)来提取这个 DAI	
		
			require(DAI.approve(address(UniswapV2Router02), amountIn), 'approve failed.');
	- `swapExactTokensForETH`

		现在我们准备好交换
		
			// amountOutMin必须从某种语言及中获取
			address[] memory path = new address[](2);
			path[0] = address(DAI);
			path[1] = UniswapV2Router02.WETH();
			UniswapV2Router02.swapExactTokensForETH(amountIn, amountOutMin, path, msg.sender, block.timestamp);
- 安全注意事项

	由于以太坊交易发生在对抗性环境中，因此可以利用不执行安全检查的智能合约获利。如果智能合约假设 Uniswap 上的当前价格是“公平”价格而没有进行安全检查，那么它很容易受到操纵。
	
	例如，不良行为者可以轻松地在交换之前和之后插入交易（“三明治”攻击），导致智能合约以更差的价格进行交易，从而以交易者为代价从中获利，然后将合约恢复到其原始状态。（一个重要的警告是，这些类型的攻击可以通过在流动性极强的池中进行交易和/或以低价值进行缓解。）

	防止这些攻击的最佳方法是使用外部价格馈送或“价格预言机”。最好的“预言机”只是交易者对当前价格的链下观察，可以作为安全检查传递到交易中。这种策略最适合用户代表自己发起交易的情况。

	但是，当无法使用链下价格时，应使用链上预言机。为给定情况确定最佳预言机不是本指南的一部分，但有关 Uniswap V2 预言机方法的更多详细信息，请参阅[预言机](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/oracles)。
	
#### 提供流动性
- 简介

	从智能合约提供流动性时，要记住的最重要的事情是 `以当前准备金率以外的任何比率存入池中的Token都容易被套利`。
	
	例如，如果一对中 `x:y` 的比率为 `10:2`（即价格为 5），并且有人天真地以 5:2（价格为 2.5）添加流动性，则合约将简单地接受所有Token（将价格更改为 3.75 并开放市场进行套利），但仅发行池Token，使发送方有权以适当的比率发送资产数量，在本例中为 5:1。为了避免向套利者捐款，必须以当前价格增加流动性。幸运的是，很容易确保满足这个条件！
- 使用

	安全地将流动性添加到池中的最简单方法是使用[路由器](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02)，它提供了简单的方法来安全地向池中添加流动性。如果要将流动性添加到 ERC-20/ERC-20 对，请使用 [addLiquidity](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#addliquidity)。如果涉及 WETH，请使用 [addLiquidityETH](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/router-02#addliquidityeth)。

	这些方法都要求调用者承诺对当前价格的信念 `amount*Desired`，这是在参数中编码的。通常，可以相当安全地假设当前的公平市场价格大约是当前的准备金率（因为套利）。
	
	所以，如果用户想向一个池子添加 1 个 ETH，并且池子当前的 DAI/WETH 比率为 200/1，那么计算出 200 个 DAI 必须与 ETH 一起发送是合理的，这是一个隐含的承诺 200 DAI/1 WETH 的价格。
	
	但是，重要的是要注意，这必须在提交交易之前计算。不安全_从交易中查找准备金率并将其作为价格信念，因为该比率可以被廉价操纵，对您不利。

	但是，仍然有可能提交一个交易，该交易包含一个关于价格的信念，该信念最终是错误的，因为在交易被确认之前，真实市场价格发生了较大的变化。
	
	出于这个原因，有必要传递一组额外的参数来编码调用者对价格变化的容忍度。这些 `amount*Min` 参数通常应设置为计算的期望价格的百分比。
	
	因此，在 1% 的容差水平下，如果用户发送一笔交易，其中包含 1 ETH 和 200 DAI，`amountETHMin` 则应设置为例如 0.99 ETH，并且`amountTokenMin` 应设置为 198 DAI。这意味着，在最坏的情况下，流动性将以 198 DAI/1 ETH 和 202.02 DAI/1 ETH (200 DAI/.99 ETH) 之间的比率增加。

	一旦进行了价格计算，重要的是要确保您的合约 
	
	- a) 控制的Token/ETH 至少与作为 `amount*Desired` 参数传递的一样多
	- b) 并且已批准路由器提取这么多Token。

#### 构建预言机
要在 Uniswap V2 上构建价格预言机，您必须首先了解用例的要求。一旦您了解了所需的平均价格类型，就需要尽可能频繁地存储该Token对的累积价格变量并使用累积价格变量的两个或多个观察值来计算平均价格。

- 需求了解

	要了解您的要求，您应该首先研究以下问题的答案：

	- 数据新鲜度重要吗？即：平均价格必须包括当前价格吗？
	- 近期价格比历史价格更重要吗？即：当前价格是否比历史价格更重要？

	记下您对以下讨论的回答。

- 预言机策略
	- 固定窗口

		在数据新鲜度不重要且近期价格与历史价格权重相等的情况下，每个周期存储一次累积价格（例如每 24 小时一次）就足够了。

		计算这些数据点的平均价格会为您提供“固定窗口”，可以在每个周期过去后更新。我们在[这里](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/examples/ExampleOracleSimple.sol)写了一个这样的示例预言机 。

		本例不限制固定窗口的最大尺寸，即只要求窗口尺寸大于1个周期（如24小时）。
	- 滑动窗口

		在数据新鲜度很重要的情况下，您可以使用一个滑动窗口，在该窗口中，累积价格变量的测量频率超过每个周期一次。
		
		您可以使用 Uniswap 累积价格变量计算至少两种[移动平均线](https://www.investopedia.com/terms/m/movingaverage.asp#types-of-moving-averages)。
		
		- [简单的移动平均线](https://www.investopedia.com/terms/s/sma.asp) 
		
			对每个价格测量赋予相同的权重。我们在 [这里](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/examples/ExampleSlidingWindowOracle.sol) 构建了一个滑动窗口预言机的例子。
		- [指数移动平均线](https://www.investopedia.com/terms/e/ema.asp)

			更重视最近的价格测量。我们还没有为这种类型的预言机编写的示例。
		
		您可能希望在近期价格比历史价格更重要的情况下使用`指数移动平均线`，例如在清算的情况下。但是，请注意与对所有价格测量均等加权相比，对近期价格赋予更多权重会使预言机的操纵成本更低。
	- 计算平均价格
	
		要计算给定两个累积价格观察值的平均价格，请取周期开始和结束时累积价格之间的差值，然后除以它们之间经过的时间（以秒为单位）。这将产生一个 [定点无符号 Q112x112](https://en.wikipedia.org/wiki/Fixed-point_arithmetic#Notation) 数字，表示一种资产相对于另一种资产的价格。该数字表示为 a `uint224`，其中高 112 位表示整数，低 112 位表示小数。
	
		对包含 `price0CumulativeLast` 和 `price1CumulativeLast` ，分别是 `token1/token0` 和 `token0/token1` 的储备比率金。即，`token0` 的价格用 `token1/token0` 表示，而 `token1` 的价格用 `token0/token1`表示。
- 获取最新的累计价格
	
	如果您希望计算历史价格累积观察值和当前累积价格之间的平均价格，您应该使用当前区块的累积价格值。如果当前区块中的累积价格尚未更新，例如因为当前区块中的Token对没有任何流动性事件 ( `mint/ burn/ swap` )，您可以反事实计算累积价格。
	
	我们提供了一个用于预言机合约的库，该库具有 [UniswapV2OracleLibrary#currentCumulativePrices](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/libraries/UniswapV2OracleLibrary.sol#L16) 获取当前区块累积价格的方法。此方法返回的当前累积价格是反事实计算的，这意味着它不需要调用该Token对 `#sync` 上的相对 `gas-expensive` 方法。无论当前块中是否已经执行了交换，它都是正确的。
- 溢出注意事项

	` UniswapV2Pair` 的累计价格变量被设计为最终溢出，即 `price0CumulativeLast` 和 `price1CumulativeLast` 和 `blockTimestampLast` 将溢出到 0。

	这不应该对您的预言机设计造成问题，因为价格平均计算涉及累积价格变量的两个单独观察值之间的差异（即减法）。`uint256` 只要在最大`2^32` 秒数或约 136 年的时间段内进行观察，两个累积价格值之间的减法将得到一个适合范围内的数字。

	`blockTimestampLast` 仅存储在一个 `uint32` . 出于与上述相同的原因，该对可以通过仅存储 `block.timestamp % uint32(-1)`. 这是可行的，因为在更新累积价格时，该Token对只关心每个流动性事件之间经过的时间，预计该时间总是少于 `2^32` 秒。

	在您自己的预言机中计算经过的时间时，您可以简单地将 `block.timestamp` 观察结果存储为 `uint256`，并避免处理溢出数学来计算观察之间经过的时间。这就是 [ExampleSlidingWindowOracle](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/examples/ExampleSlidingWindowOracle.sol) 处理观察时间戳的方式。

	- 集成预言机

		要将预言机集成到您的合约中，您必须确保预言机对累积价格变量的观察保持最新。只要你的预言机是最新的，你就可以依靠它来产生平均价格。使您的预言机保持最新的过程称为“维护”。
	- 预言机维护 

		为了衡量一个时期内的平均价格，预言机必须有一种方法来参考一个时期开始和结束时的累积价格。这样做的推荐方法是将这些价格存储在预言机合约中，并足够频繁地调用预言机以存储最新的累积价格。

		可靠的预言机维护是一项艰巨的任务，并且在拥塞时可能成为故障点。相反，请考虑将此功能直接构建到您自己的智能合约的关键调用中，或激励其他方的预言机维护调用。

	- 免维护选项

		通过使用存储证明，可以避免在周期开始时定期存储此累积价格。但是，这种方法有局限性，特别是在 Gas 成本和可以测量平均价格的最大时间段方面。如果您想尝试这种方法，您可以关注 [Keydonix](https://github.com/Keydonix/uniswap-oracle/) 的这个存储库。

		Keydonix 开发了一个基于 Uniswap v2 的通用价格供给预言机，它支持任意时间窗口（最多 256 个块）并且不需要任何主动维护。

#### 闪电交换
闪电交换是 Uniswap V2 的一个组成部分。事实上，在幕后，所有交换实际上都是闪电交换！这仅仅意味着配对合约在强制接收到足够的输入Token之前将输出Token发送给接收者。这有点不典型，因为人们可能期望一对确保在交货前收到付款。

但是，由于以太坊交易是原子的，如果事实证明合约在交易结束时没有收到足够的Token来使自己完成完整业务逻辑，最终可以回滚整个交换。要了解这一切是如何工作的，让我们从检查 `swap` 函数的接口开始：

	function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data);
为了举例，假设我们正在处理 DAI/WETH 对，其中 DAI 为 `token0`，WETH 为`token1`。`amount0Out` 和 `amount1Out` 指定 `msg.sender` 希望该Token对发送到 `to` 地址的 DAI 和 WETH 数量（其中一个可能为 0）。

此时你可能想知道合约是如何接收Token的。对于典型的（非闪电）交换，实际上有责任  `msg.sender` 确保在调用之前已经向该Token对发送 `swap` 了足够的 `WETH` 或 `DAI` （在交易的上下文中，这一切都由路由合约巧妙地处理）。

但是在执行闪电交换时，调用前不需要将Token发送到合约用来交换。它们必须在 `to` 地址上由对触发的回调函数中发送。

- 触发闪电交换

	为了区分“典型”交易案例和闪电交换案例，对使用 `data` 参数。具体来说，
	
	- 如果 `data.length` 等于 0，则合约假设已经收到付款并简单地将Token转移到该 `to` 地址。
	- 但如果 `data.length` 大于 0，则合约转移Token，然后在 `to` 地址上调用以下函数：

			function uniswapV2Call(address sender, uint amount0, uint amount1, bytes calldata data);
	这种识别策略背后的逻辑很简单：绝大多数有效的闪存交换用例都涉及与外部协议的交互。传递指示这些交互如何发生的信息（函数参数、安全参数、地址等）的最佳方式是通过 `data` 参数。预计数据将在 `uniswaapv2call` 内被 `abi` 解码。在极少数不需要数据的情况下，调用方应该确保数据。`length` 等于 1(即将单个垃圾字节编码为字节)，然后在 `uniswaapv2call` 中忽略此参数。

配对调用 `uniswapV2Call`，`sender` 为 `swap` 参数设置 `msg.sender` . `amount0` 和 `amount1` 是简单的 `amount0Out`和 `amount1Out`。

- 使用 uniswapV2Call

	`uniswapV2Call` 在所有功能中都应检查几个条件：

		function uniswapV2Call(address sender, uint amount0, uint amount1, bytes calldata data) {
		  address token0 = IUniswapV2Pair(msg.sender).token0(); // 获取 token0 的地址
		  address token1 = IUniswapV2Pair(msg.sender).token1(); // 获取 token1 的地址
		  assert(msg.sender == IUniswapV2Factory(factoryV2).getPair(token0, token1)); // 确保 msg.sender 是一个 V2 对
		  // 函数的其余部分在这里!
		}

	前 2 行简单地从对中获取Token地址，第 3 行确保这`msg.sender` 是一个实际的 Uniswap V2 对地址。

- 还款

	在 `uniswapV2Call` 结束时，合约必须返回足够的Token给该Token对以使其完整。具体而言，这意味着交换后的Token对储备金的乘积，扣除 0.3% LP 费用发送的所有Token金额，必须大于之前。

	- 多个 Token 

		在提取的Token不是退回的Token的情况下（即在闪电交换中请求了 DAI，而退回了 WETH，反之亦然），费用简化为简单的互换情况。这意味着 `getAmountIn` 应该使用标准定价函数来计算，例如，必须返回的 WETH 数量以换取请求的 DAI 数量。

		这种类型的费用计算给出了一个轻微的优势给调用者,作为相应的费用来自偿还Token总是会略低于偿还费用来源于直接的Token,与取款金额,然后直接返回相比，由于金额之间的差异需要偿还交换。费用的大致比较是，互换费用约为 30 个基点，而直接还款则为 30.09 个基点。
	- 单个 Token 

		在提取的Token与返回的Token相同的情况下（即在闪存交换中请求 DAI，使用，然后返回，或 WETH 反之亦然），必须满足以下条件：

			DAIReservePre - DAIWithdrawn + (DAIReturned * .997) >= DAIReservePre

		用对提取金额征收的“费用”来重写这个公式可能更直观（尽管 Uniswap 总是对输入金额收取费用，在这种情况下是返回金额，这里我们可以简化为有效费用提取的金额）。如果我们重新排列，公式如下所示：

		- `(DAIReturned * .997) - DAIWithdrawn >= 0`
		- `DAIReturned >= DAIWithdrawn / .997`

		因此，提取金额的有效费用为 `.003 / .997 ≈ 0.3009027%。`
- 资源

	有关闪存交换的进一步探索，请参阅[白皮书](https://docs.uniswap.org/whitepaper.pdf)
- 例子

	提供了一个完整功能的闪存交换示例：[ExampleFlashSwap.sol](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/examples/ExampleFlashSwap.sol)
- 界面

		import '@uniswap/v2-core/contracts/interfaces/IUniswapV2Callee.sol';
		
		pragma solidity >=0.5.0;
		
		interface IUniswapV2Callee {
		  function uniswapV2Call(address sender, uint amount0, uint amount1, bytes calldata data) external;
		}

#### 配对地址
- `getPair`

	获取地址对的最明显方法是在工厂调用 [getPair](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#getpair)。
	
	- 如果该对存在，则此函数将返回其地址
	- 否则address(0)( 0x0000000000000000000000000000000000000000)。
	- 确定一对是否存在的“规范”方法。
	- 需要链上查找。
- `CREATE2`

	多亏了工厂里的[一些方法](https://github.com/Uniswap/uniswap-v2-core/blob/master/contracts/UniswapV2Factory.sol#L32).由于 [CREATE2](https://eips.ethereum.org/EIPS/eip-1014)，我们还可以在不进行链上查找的情况下计算对地址。以下值是这种技术所需要的:
	
	类型|描述
	---|---
	`address `|[工厂地址](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/factory#address)
	`salt`|`keccak256(abi.encodePacked(token0, token1))`
	`keccak256(init_code)`|`0x96e8ac4277198ff8b6f785478aa9a39f403cb768dd02cbee326c3e7da348845f`

	- `token0` 必须严格小于 `token1` 按排序顺序	
	- 可以离线计算
	- 需要执行 `keccak256` 的能力
- 例
	- 坚固性

			address factory = 0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f;
			address token0 = 0xCAFE000000000000000000000000000000000000; // change me!
			address token1 = 0xF00D000000000000000000000000000000000000; // change me!
			
			address pair = address(uint(keccak256(abi.encodePacked(
			  hex'ff',
			  factory,
			  keccak256(abi.encodePacked(token0, token1)),
			  hex'96e8ac4277198ff8b6f785478aa9a39f403cb768dd02cbee326c3e7da348845f'
			)))); 

#### 支持元交易
所有 Uniswap V2 池Token都支持通过[许可功能批准元交易](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20#permit)。这消除了在与池Token进行编程交互之前对阻塞批准事务的需要。

- ERC-712

	在普通 ERC-20 Token合约中，所有者只能通过直接调用 `msg.sender` 用于许可本身的函数来注册批准。使用元批准，所有权和许可来自调用者（有时称为中继者）传递给函数的签名。由于使用以太坊私钥对数据进行签名可能是一项棘手的工作，Uniswap V2 依赖 [ERC-712](https://eips.ethereum.org/EIPS/eip-712)（一种具有广泛社区支持的签名标准）来确保用户安全和钱包兼容性。

	- 域分隔符

			keccak256(
			  abi.encode(
			    keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
			    keccak256(bytes(name)),
			    keccak256(bytes('1')),
			    chainId,
			    address(this)
			  )
			);
		- `name` 总是`Uniswap V2`，见[名字](https://docs.uniswap.org/protocol/V2/reference/smart-contracts/pair-erc-20#name)。
		- `chainId` 由 [ERC-1344](https://ethereum-magicians.org/t/eip-1344-add-chain-id-opcode/1131) chainid 操作码确定。
		- `address(this)` 是对的地址，请参阅 [Pair Addresses](https://docs.uniswap.org/sdk/2.0.0/guides/getting-pair-addresses)。
- 许可哈希类型

		keccak256('Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)');`




	

		
		

	
	

 		

						

					