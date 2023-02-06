# Safe SDK 详细说明
## 入门
开始创建 safe 应用程序的基本资源

欢迎！我们很高兴您有兴趣创建 safe 应用程序。该团队正在努力提供工具，让您更轻松地构建可与 Safe 交互的应用程序。

主要资源包括：

- [界面套件](https://docs.gnosis-safe.io/learn/safe-tools/sdks/safe-apps/get-started#ui-kit)
- [SDK 包](https://docs.gnosis-safe.io/learn/safe-tools/sdks/safe-apps/get-started#sdk-packages)
- [基本要求](https://docs.gnosis-safe.io/learn/safe-tools/sdks/safe-apps/get-started#basic-requirements)

### 界面套件
如果您要从头开始创建 safe 应用程序，我们提供了一个可重用的 React 组件包，使您可以轻松构建具有近乎原生外观的 safe 应用程序，同时仍允许开发人员在 safe 应用程序中使用他们的品牌。

- [检查故事书中所有可用组建](https://components.gnosis-safe.io/)
- [检查 UI 套件库](https://github.com/gnosis/safe-react-components)

### SDK包
这是我们 safe 应用程序集成的主要部分之一。您已经拥有一个 dapp 或者您正在考虑创建一个新的 dapp，您会发现依赖我们的集成之一来轻松地与 Safe 应用程序进行通信很有用。在这个包中，您会发现与非常常见的包（如 `Web3Modal`、`Blocknative onboard.js` 或 `web3-react`）的集成，您可能已经在您的项目中使用了这些包。对于那些创建新的 dapp 的人，我们 [CRA 模版](https://github.com/gnosis/safe-apps-sdk/tree/master/packages/cra-template-safe-app) 的使用所有必要的配置来启动基本结构，这将加快这个过程。[更多的信息](https://docs.gnosis-safe.io/learn/safe-tools/sdks/safe-apps/build)。
### 基本要求
如果你已经有一个 dapp，这些是一些强制性要求，可以使你的应用程序成为一个safe的应用程序。如果没有这个基本配置，dapp 将无法按预期与 Safe 一起工作。

如果您使用我们 [CRA 模版](https://github.com/gnosis/safe-apps-sdk/tree/master/packages/cra-template-safe-app) 的来启动您的safe应用程序，则此基本要求已包含在内。

- 显现

	您的应用必须 `manifest.json` 在根目录中公开一个具有以下结构的文件：
	
		{
		  "name": "YourAppName",
		  "description": "A description of what your app do",
		  "iconPath": "myAppIcon.svg"
		}


		注意：iconPath 它是 Safe 将尝试加载您的应用程序图标的公共相对路径。对于此示例，它应该是 https://yourAppUrl/myAppIcon.svg。
- 夸域

	在某些时候，我们需要能够 `manifest.json` 从我们的应用程序访问。为此，需要通过将`.manifest.json`的 `CORS headers` 设置为跨站请求开放。
	
	所需的标头是：
	
		"Access-Control-Allow-Origin": "\*",
		"Access-Control-Allow-Methods": "GET",
		"Access-Control-Allow-Headers": "X-Requested-With, content-type, Authorization"
- React 开发

	可以使用本地 React 开发服务器。为此，您需要设置  `CORS headers`  并确保使用与用于测试的 safe 接口相同的协议（http 或 https）。
- 用于开发的 CORS

	为此，我们建议使用 [react-app-rewired](https://www.npmjs.com/package/react-app-rewired)。要启用库，请更新 `scripts` 部分 `package.json`：

		"scripts": {
		  "start": "react-app-rewired start",
		  "build": "react-app-rewired build",
		  "test": "react-app-rewired test"
		},
	此外，您需要 `config-overrides.js` 在项目的根目录中创建文件以配置 `CORS headers`  。该文件的内容应该是：

		/* config-overrides.js */
		
		module.exports = {
		 //运行开发时创建 webpack 开发服务器配置的函数
		 //服务器使用 'npm run start' 或 'yarn start'
		 //示例:在 https 中设置 dev 服务器使用指定的证书。
		  devServer: function (configFunction) {
		   // 返回 create-react-app 用来生成 Webpack Development Server 配置的替换函数。
		   // "configFunction"是通常用来生成Webpack Development服务器配置的函数
		   // 你可以用它来创建一个初始配置，然后进行修改，而不必从头开始创建配置。
		    return function (proxy, allowedHost) {
		      // 通过调用带有proxy/allowedHost参数的configFunction来创建默认配置
		      const config = configFunction(proxy, allowedHost);
		
		      config.headers = {
		        'Access-Control-Allow-Origin': '*',
		        'Access-Control-Allow-Methods': 'GET',
		        'Access-Control-Allow-Headers': 'X-Requested-With, content-type, Authorization',
		      };
		
		      // 返回定制的 Webpack 开发服务器配置。
		      return config;
		    };
		  },
		};
- SSL

	要启用 SSL， `react-scripts` 必须将 `HTTPS` 环境变量设置为` true`. 这可以在 `package.json` 文件中通过将 `scripts` 部分调整为：

		"scripts": {
		  "start": "HTTPS=true react-app-rewired start",
		},
	在大多数情况下，由  `react-scripts` 提供的 SSL 证书无效，因此需要在您的浏览器中将其标记为受信任。为此，在单独的选项卡（不在safe界面中）中打开 safe 应用程序并接受证书/忽略警告。
	
## SDK 包
可在[开发人员工具](https://github.com/safe-global/safe-apps-sdk)上找到多个软件包，以便更轻松地将第三方应用程序（Safe Apps）与[safe](https://app.safe.global/)集成。查看下图，看看哪种套餐更适合您：

![](./pic/safe5.png)
### 包	
包|发布|描述
---|---|---
[cra-template-safe-app](https://github.com/safe-global/safe-apps-sdk/tree/master/packages/cra-template-safe-app)||用于快速引导 safe 应用程序的 cra 模版，从头创建应用
[safe-apps-react-sdk](https://github.com/safe-global/safe-apps-sdk/tree/master/packages/safe-apps-react-sdk) |npm 4.6.2| safe-apps-sdk 包装器，带有 react 钩子
[safe-apps-sdk](https://github.com/safe-global/safe-apps-sdk/tree/master/packages/safe-apps-sdk)|npm 7.8.0|开发工具包，所有集成的基础包
[safe-apps-provider](https://github.com/safe-global/safe-apps-sdk/tree/master/packages/safe-apps-provider) |npm 0.15.1|可与通用 web3 库一起使用的通用程序，如 web3.js/ethers
[safe-apps-wagmi](https://github.com/safe-global/safe-apps-sdk/tree/master/packages/safe-apps-wagmi)|npm 2.1.0|用于safe 应用程度的 [wagmi 连接器](https://github.com/wagmi-dev/wagmi)
[safe-apps-web3modal](https://github.com/gnosis/safe-apps-sdk/tree/master/packages/safe-apps-web3modal)|npm 17.0.4|[web3modal](https://github.com/Web3Modal/web3modal) 的包装器，当应用加载到 safe 应用时，会自动连接 safe 应用程序
[@web3-onboard/gnosis](https://github.com/blocknative/web3-onboard/tree/v2-web3-onboard-develop/packages/gnosis)||[web3-onboard](https://github.com/blocknative/web3-onboard)v1.26.0 中包含safe 应用程序支持，可以在[这里](https://github.com/blocknative/web3-onboard/blob/v2-web3-onboard-develop/packages/gnosis/README.md)查看配置
[@web3-react/gnosis-safe](https://github.com/Uniswap/web3-react/tree/main/packages/gnosis-safe)||默认情况下，web3-react 已经包含了 safe 应用程序连接器，可以查看文档
## 测试您的安全应用程序
### 如何测试安全应用程序？
在开发您的安全应用程序时，您可以直接使用[生产界面](https://app.safe.global/)对其进行测试。那里也有一些像 Goerli 这样的测试网。

一旦您的应用程序上线，即使您在本地运行它，您也可以将其作为自定义应用程序导入到 Safe 应用程序。为此，您应该选择“Apps”选项卡：

![](./pic/safe6.png)

使用 `Add custom app` 按钮并使用链接添加您的应用程序：

![](./pic/safe7.png)

## 发布您的安全应用程序
### 发布流程
- 如何将您的安全应用程序交到用户手中

	一旦您完成开发和测试您的安全应用程序，您就可以让一些实际用户对其进行测试，只需向他们发送指向托管 safe 应用程序的链接并要求他们将其添加为自定义应用程序即可。[手册](https://help.gnosis-safe.io/en/articles/4022030-add-a-custom-safe-app)介绍了如何添加自定义应用程序。
- 将您的 Safe App 列在 Safe 中

	如果您希望您的 Safe App 出现在 Safe 中，它必须满足以下条件：

	- 1）智能合约必须被审计

		安全是重中之重。如果您的 Safe App 包含您自己的智能合约，您应该提供外部审计结果文件。如果智能合约是由第三方创建的，您应该使用经过适当审计的智能合约。
	- 2) 您的 Safe App 必须在根目录包含一个 manifest.json 文件，其中包含以下数据：
		- `"name": "Name of your Safe App"`
		
			您的 Safe App 的名称，最多 50 个字符。
		- `"iconPath": "your_logo.svg"`

			应用程序徽标的相对文件路径。图标必须是至少 128 x 128 像素的方形 SVG 图像。
		- `"description": "This is the Safe app description."`

			几句话描述您的申请，最多 200 个字符

			[github](https://github.com/safe-global/safe-apps-sdk/blob/master/packages/cra-template-safe-app/template/public/manifest.json)上找到示例清单文件。此外，您可以在 IPFS 上找到 safe 应用程序的[示例](https://ipfs.io/ipfs/QmTgnb1J9FDR9gimptzvaEiNa25s92iQy37GyqYfwZw8Aj/)。

			请记住，应该在 上正确配置 CORS ，以便我们可从 [mentioned](https://docs.gnosis-safe.io/learn/safe-tools/sdks/safe-apps/get-started#cors)获取 `manifest.json` 的信息。
	- 3）应用程序自动连接到保险箱

		当用户打开应用程序时，它应该自动选择 Safe 作为钱包。如果用户之前使用另一个钱包在 Safe 之外打开应用程序，请确保检查案例。
	- 4) Safe 团队审核了 Safe App

		该要求不适用于像主 dApp 一样托管在同一域中的经过实战测试的应用程序。

		虽然我们无法对您的 safe 应用程序进行适当的审核，但我们仍然希望查看源代码以提出问题或提出改进建议。因此，无论您的 Safe App 是开源还是封闭源代码，`请向我们发送公共存储库的链接或私有代码存储库的邀请`。

		我们还想对应用程序进行粗略的功能审查，因此请向我们提供高级测试计划/功能列表，以便我们的 QA 团队确保一切按预期在生产中运行。也欢迎视频演练。
	- 5) 帮助我们解码您的安全应用程序交易

		我们希望尽可能以人类可读的方式显示与 safe 应用程序的交互。为此，我们需要您的safe应用与之交互的合约的合约 ABI。执行此操作的理想方法是通过验证您的合约 [sourcify](https://github.com/ethereum/sourcify)，我们可以利用它来解码与这些合约交互的交易。

		或者，您可以向我们提供 JSON 文件形式的 ABI 或指向 Etherscan 上已验证合约的链接，以便我们可以为您的应用程序交互实施交易解码。
		
- 验证您的应用程序满足这些要求后，在我们的存储库中创建一个 issue 问题 [https://github.com/gnosis/safe-apps-list](https://github.com/gnosis/safe-apps-list)
- 正式发布及以后

	在我们审核并集成您的 Safe App 后，该 App 将首先在 Safe 的中可用，供您进行最终审核。然后我们会与您联系以协调发布和联合公告。
在发布后的任何时候，如果您或您的用户在使用 Safe App 时遇到问题，或者您想发布对现有 Safe App 的更新，请通过 [discord](https://chat.safe.global/)与我们联系
