# Opensea Creature 学习笔记
## 部署合约
- 申请 Rikeby 测试链以太
	- 登陆 MetaMask 
	- 选择  Rinkeby
	- 登陆 [Rinkeby Ether 龙头](https://faucet.rinkeby.io/)
	- 注册 facebook 或者 tweet ,我是用的是 tweet 比较好找链接
	- 将自己要充值的钱包地址写到 tweet，公开后，得到对应地址，将地址贴到  [Rinkeby Ether 龙头](https://faucet.rinkeby.io/) 充值
	- 每天最多申请 7.5 个以太

	![](./pic/rikeby.png)		
- 只需获取[仓库代码](https://github.com/ProjectOpenSea/opensea-creatures)
		
		mkdir github && \
		cd github && \
		git clone https://github.com/ProjectOpenSea/opensea-creatures && \
		cd opensea-creatures
- 获得以太坊 API 代理调用权限	
	- [Alchemy API 密钥](https://dashboard.alchemyapi.io/signup?referral=affiliate:e535c3c3-9bc4-428f-8e27-4b70aa2e8ca5)
		- 注册
	
			![](./pic/alchemy16.png)
			![](./pic/alchemy17.png)
		- 认证邮箱
		- 生成应用
		
			![](./pic/alchemy18.png)
		- 选择模式
	
			![](./pic/alchemy19.png) 
			![](./pic/alchemy20.png) 
		- 选择缩放策略
	
			![](./pic/alchemy21.png) 
		- 注册完成
	
			![](./pic/alchemy22.png)
		- 创建 key(注意前段和后端建议使用不同的 key 好统计请求和压力)

			![](./pic/alchemy23.png)
		- 提取 Key，只用 v2 后面的数据

			![](./pic/alchemy23.png)				
	- [Infura.io](https://infura.io/)		
		- 注册
		
			![](./pic/infura2.png)	
			![](./pic/infura3.png)	
		- 认证邮箱
		- 生成应用

			![](./pic/infura4.png)
		- 提取 key, v3 后面的

			![](./pic/infura5.png)	
	- 生成环境变量
	
			vi .env
				export INFURA_KEY="<your_infura_project_id>"
				export ALCHEMY_KEY="<your_alchemy_project_id>"
				export MNEMONIC="<metamask 助记词>"
				export NETWORK="rinkeby"
	- 载入环境变量

			source .env <- 环境变量泄露助记词
	- 查看环境变量

			env
		![](./pic/infura6.png)			
	- 设置 `.gitignore` 不同步 `.env`和其他，防止助记词泄露

			build/
			flattened/
			node_modules/
			.env* <- 隐藏助词
			*.DS_Store
- 安装 npm
	
	[参考精通以太坊](https://github.com/pangzheng/BOOKS/blob/master/%E6%8A%80%E6%9C%AF/%E9%93%BE%E6%8A%80%E6%9C%AF%E5%AD%A6%E4%B9%A0/%E4%BB%A5%E5%A4%AA%E5%9D%8A/%E7%B2%BE%E9%80%9A%E4%BB%A5%E5%A4%AA%E5%9D%8A.md)
- 安装 nvm

	因为 node 版本要切换	
- 安装 truffle
	
	[参考精通以太坊](https://github.com/pangzheng/BOOKS/blob/master/%E6%8A%80%E6%9C%AF/%E9%93%BE%E6%8A%80%E6%9C%AF%E5%AD%A6%E4%B9%A0/%E4%BB%A5%E5%A4%AA%E5%9D%8A/%E7%B2%BE%E9%80%9A%E4%BB%A5%E5%A4%AA%E5%9D%8A.md)
- 安装 yarn
	
		brew install yarn
- 查询 Creature node 版本

		vi package.json
		
		...
		"engines": {
	    "node": "^12.18.x",
	    "yarn": "~1.22.4"
		...
- 使用 nvm 查看、安装和切换 node 版本
	- 查看远程版本	
	
			nvm ls-remote
	- 安装对应版本,版本根据 `package.json` 安装

			nvm install v12.18.4		 
	 - 切换版本

			nvm use v12.18.4
	- 查看当前版本

			nvm current
			v12.18.4
	- 安装 npm 插件
	
			npm install
		- 遇到问题  `npm ERR! Cannot read property '0' of undefined`

				npm install
				npm ERR! Cannot read property '0' of undefined
				
				npm ERR! A complete log of this run can be found in:
				npm ERR!     /Users/Prometheus/.npm/_logs/2021-05-18T09_30_05_765Z-debug.log
				Prometheus@pangzhengdeMacBook-Pro opensea-creatures % cat /Users/Prometheus/.npm/_logs/2021-05-18T09_30_05_765Z-debug.log
			- 解决

					mv node_modules node_modules.bak
	- 安装其他 `package.json` 依赖

			yarn install
		or
			
			yarn
		- 遇到 git 443 超时，请使用 shadowsocket 翻墙代理
			- 使用
				
					git config --global http.proxy 'socks5://127.0.0.1:1080'
					git config --global https.proxy 'socks5://127.0.0.1:1080
			- 卸载

					git config --global --unset http.proxy
- 使用 Truffle 进行部署
	- 编译合约

			truffle compile					
	- 部署			
			
			% truffle deploy --network rinkeby

			Compiling your contracts...
			===========================
			✔ Fetching solc version list from solc-bin. Attempt #1
			> Everything is up to date, there is nothing to compile.
			
			
			
			Migrations dry-run (simulation)
			===============================
			> Network name:    'rinkeby-fork'
			> Network id:      4
			> Block gas limit: 10000000 (0x989680)
			
			
			1_initial_migrations.js
			=======================
			
			   Deploying 'Migrations'
			   ----------------------
			   > block number:        8606715
			   > block timestamp:     1621336141
			   > account:             0xD61c47B00BB534993f6A79A6561c7a6184aDF616
			   > balance:             14.999680456
			   > gas used:            159772 (0x2701c)
			   > gas price:           2 gwei
			   > value sent:          0 ETH
			   > total cost:          0.000319544 ETH
			
			   -------------------------------------
			   > Total cost:         0.000319544 ETH
			
			
			2_deploy_contracts.js
			=====================
			
			   Deploying 'Creature'
			   --------------------
			   > block number:        8606717
			   > block timestamp:     1621336183
			   > account:             0xD61c47B00BB534993f6A79A6561c7a6184aDF616
			   > balance:             14.995625596
			   > gas used:            2000116 (0x1e84f4)
			   > gas price:           2 gwei
			   > value sent:          0 ETH
			   > total cost:          0.004000232 ETH
			
			   -------------------------------------
			   > Total cost:         0.004000232 ETH
			
			
			Summary
			=======
			> Total deployments:   2
			> Final cost:          0.004319776 ETH
			
			
			
			
			
			Starting migrations...
			======================
			> Network name:    'rinkeby'
			> Network id:      4
			> Block gas limit: 10000000 (0x989680)
			
			
			1_initial_migrations.js
			=======================
			
			   Deploying 'Migrations'
			   ----------------------
			   > transaction hash:    0x82998fe5a3764f69243816655b3beaf73c359f5859f7fe5c25aa61ebda6548d7
			   > Blocks: 2            Seconds: 17
			   > contract address:    0x7Ae44d45426728518914Ed3A7c37c909BbdE85Ee
			   > block number:        8606720
			   > block timestamp:     1621336226
			   > account:             0xD61c47B00BB534993f6A79A6561c7a6184aDF616
			   > balance:             14.99647856
			   > gas used:            176072 (0x2afc8)
			   > gas price:           20 gwei
			   > value sent:          0 ETH
			   > total cost:          0.00352144 ETH
			
			
			   > Saving migration to chain.
			   > Saving artifacts
			   -------------------------------------
			   > Total cost:          0.00352144 ETH
			
			
			2_deploy_contracts.js
			=====================
			
			   Deploying 'Creature'
			   --------------------
			   > transaction hash:    0xcffdeeca50b26e1536a9dcca71d2beac220c55fc2cf5ff3a22351fce3242dfa3
			   > Blocks: 1            Seconds: 9
			   > contract address:    0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0
			   > block number:        8606722
			   > block timestamp:     1621336256
			   > account:             0xD61c47B00BB534993f6A79A6561c7a6184aDF616
			   > balance:             14.9530882
			   > gas used:            2123804 (0x20681c)
			   > gas price:           20 gwei
			   > value sent:          0 ETH
			   > total cost:          0.04247608 ETH
			
			
			   > Saving migration to chain.
			   > Saving artifacts
			   -------------------------------------
			   > Total cost:          0.04247608 ETH
			
			
			Summary
			=======
			> Total deployments:   2
			> Final cost:          0.04599752 ETH
		- 遇到问题 ` Cannot read property 'includes' of null`
				 
				Compiling your contracts...
				===========================
				✖ Fetching solc version list from solc-bin. Attempt #1
				✖ Fetching solc version list from solc-bin. Attempt #2
				✔ Fetching solc version list from solc-bin. Attempt #3
				Error: TypeError: Cannot read property 'includes' of null
				    at VersionRange.load (/usr/local/lib/node_modules/truffle/build/webpack:/packages/compile-solidity/compilerSupplier/loadingStrategies/VersionRange.js:200:1)
				    at processTicksAndRejections (internal/process/task_queues.js:97:5)
				Truffle v5.3.5 (core: 5.3.5)
				Node v12.18.4
			- 测试

					truffle console --network rinkeby
				有问题请检查助记词(`Error: Mnemonic invalid or undefined`),通过重新加载 .env 文件解决、API key 等，
				
				最后返现是公司与 api 网络提供商的网络问题,最后无奈,切换到 infura api 服务
			 		 			
		如果您已经在使用 Infura API，则也可以使用 `INFURA_KEY` 环境变量代替 `ALCHEMY_KEY`
	- 查看游览器合约是否存在

		部署到 Rinkeby 网络后，将在 Rinkeby 上签订一份合约，该合约将在 [Rinkeby Etherscan](https://rinkeby.etherscan.io/) 
		
		使用 Creature 合约地址查看，游览器，地址查询部署日志，
		
				2_deploy_contracts.js
				=====================
			
			   Deploying 'Creature'
			   --------------------
			   > transaction hash:    0xcffdeeca50b26e1536a9dcca71d2beac220c55fc2cf5ff3a22351fce3242dfa3 <--- 交易地址
			   > Blocks: 1            Seconds: 9
			   > contract address:    0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0  <---新建 Creature 合约地址
			   > block number:        8606722
			   > block timestamp:     1621336256
			   > account:             0xD61c47B00BB534993f6A79A6561c7a6184aDF616
			   > balance:             14.9530882
			   > gas used:            2123804 (0x20681c)
			   > gas price:           20 gwei
			   > value sent:          0 ETH
			   > total cost:          0.04247608 ETH
		- 合约地址查询

			注意，这里的 Form 地址就是你的 MetaMask 钱包地址 `0xD61c47B00BB534993f6A79A6561c7a6184aDF616`
		
			![](./pic/Creature.png)
			
				https://rinkeby.etherscan.io/address/<contract_address>
		- 交易hash 查询

			![](./pic/Creature1.png)
			
				https://rinkeby.etherscan.io/tx/<Transaction Hash>
		- 查看事件

			- 创建合约 000...地址
			- 拥有者地址就是你的 MetaMask
			
			![](./pic/Creature2.png)	

## 铸币
### 加载铸币环境变量
	- 添加铸币环境变量到 `.env`
	
		![](./pic/Creature3.png)
		
			export OWNER_ADDRESS="0xD61c47B00BB534993f6A79A6561c7a6184aDF616" <--钱包地址
			export NFT_CONTRACT_ADDRESS="0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0" <---上面的Creature合约地址
	- 加载变量	
		
			source .env
	- 查看
	
			env
			
			...
			OWNER_ADDRESS=0xD61c47B00BB534993f6A79A6561c7a6184aDF616
			NFT_CONTRACT_ADDRESS=0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0
			FACTORY_CONTRACT_ADDRESS=
			NETWORK=rinkeby
			...
### 铸币脚本
- 查看铸币脚本

		vi scripts/mint.js
		...
		const NFT_CONTRACT_ADDRESS = process.env.NFT_CONTRACT_ADDRESS;
		const OWNER_ADDRESS = process.env.OWNER_ADDRESS;
		const NETWORK = process.env.NETWORK;
		const NUM_CREATURES = 13; <---修改数量为 13
		const NUM_LOOTBOXES = 4;
		const DEFAULT_OPTION_ID = 0;
		...
		 if (FACTORY_CONTRACT_ADDRESS) {  <---因为我们没有工厂合约
		    const factoryContract = new web3Instance.eth.Contract(
		      FACTORY_ABI,
		      FACTORY_CONTRACT_ADDRESS,
		      { gasLimit: "1000000" }
		    );
		
		    // Creatures issued directly to the owner.
		    for (var i = 0; i < NUM_CREATURES; i++) {
		      const result = await factoryContract.methods
		        .mint(DEFAULT_OPTION_ID, OWNER_ADDRESS)
		        .send({ from: OWNER_ADDRESS });
		      console.log("Minted creature. Transaction: " + result.transactionHash);
		    }
		
		    // Lootboxes issued directly to the owner.
		    for (var i = 0; i < NUM_LOOTBOXES; i++) {
		      const result = await factoryContract.methods
		        .mint(LOOTBOX_OPTION_ID, OWNER_ADDRESS)
		        .send({ from: OWNER_ADDRESS });
		      console.log("Minted lootbox. Transaction: " + result.transactionHash);
		    }
		  } else if (NFT_CONTRACT_ADDRESS) {  <---所以只生效 NFT 合约
		    const nftContract = new web3Instance.eth.Contract( <-- 首先生成合约实例
		      NFT_ABI,
		      NFT_CONTRACT_ADDRESS,
		      { gasLimit: "1000000" }
		    );
		
		    // Creatures issued directly to the owner. <--- 然后循环 13 次，生成13个独立的铸币合约
		    for (var i = 0; i < NUM_CREATURES; i++) {
		      const result = await nftContract.methods
		        .mintTo(OWNER_ADDRESS)               
		        .send({ from: OWNER_ADDRESS });
		      console.log("Minted creature. Transaction: " + result.transactionHash); <--- 如果成功则打印
		    }
		  } else {
		    console.error(
		      "Add NFT_CONTRACT_ADDRESS or FACTORY_CONTRACT_ADDRESS to the environment variables"
		    );
- 运行铸币脚本
	
		% node scripts/mint.js
		web3-shh package will be deprecated in version 1.3.5 and will no longer be supported.
		web3-bzz package will be deprecated in version 1.3.5 and will no longer be supported.
		Minted creature. Transaction: 0xd3e78aafd1b54c5f39190b3bb90905631636ea35e1dffdc8b3245c9ac5aa5e6e
		Minted creature. Transaction: 0x13d180d52528ed78532de200fa58ccdeeacaa25dd4bf6f6c723b18597ac0da13
		Minted creature. Transaction: 0x48e7dd8cb8e11d35e3f4ff98b846668e742f72eecf5657c4f142444d30dc0b31
		Minted creature. Transaction: 0xdda2960ceeb4749207e66a165f542ba199a08b3e629eae0f5d7c89fb8d886ee5
		Minted creature. Transaction: 0xadd8604ca4e8e755a20ef77a18d98075943c8677cc1fb07e0c66d62d29b56f65
		Minted creature. Transaction: 0x3b67a4df2c583e9e51efee7b7826e066e49bcecc2def1fd90698972e39ea0027
		Minted creature. Transaction: 0x054ad99192986e901dc106acaa7e732dbb9c1bc680a35e52370350f7ee912401
		Minted creature. Transaction: 0x0ebe41448659fe35f67d1cf2abb4e7daf95e64fe7c70f66341e63cc49265d9c9
		Minted creature. Transaction: 0xd3b11d3475d0b9485fa6681c8ef4f55285a0cbdd056837478693929f5c5a2c79
		Minted creature. Transaction: 0x8660aa374dd725bad73fdf6d1d776461aff56e75e17fe805f5092ef69c03978f
		Minted creature. Transaction: 0x9a378d1271a9bf49203011eaa19b8d2176b0e36f8be6ca91222567606c512024
		Minted creature. Transaction: 0x642a24da292aa3e705739da65b229d35b0e8ce4efe90c4b4da3f362801cf991e
		Minted creature. Transaction: 0xb19bd6f4d0285b4554bdc656bce07a76ad97ecaae3a93ef36ae0e0c9f744202e
	
		^C	<--- 这个脚本不会自己停止，看到合约到达预期后，需要 control+c
- 查看游览器	
		
	![](./pic/Creature4.png)	

## 添加元数据
部署合约后，将需要一种方法来使每个单独的项目正确显示在 OpenSea（以及支持不可替代令牌的其他网站）上。这是链外元数据发挥作用的地方！ERC721 合约中的每个令牌标识符都将具有对应的元数据 URI，该元数据 URI 返回有关项目的其他重要信息，例如项目的名称，图像，描述等。

- 例子:

	首先打开 [https://opensea-creatures-api.herokuapp.com/api/creature/3](https://opensea-creatures-api.herokuapp.com/api/creature/3) 查看结果
	
		{
		attributes: [
		{
		trait_type: "Base",
		value: "narwhal"
		},
		{
		trait_type: "Eyes",
		value: "sleepy"
		},
		{
		trait_type: "Mouth",
		value: "cute"
		},
		{
		trait_type: "Level",
		value: 4
		},
		{
		trait_type: "Stamina",
		value: 90.2
		},
		{
		trait_type: "Personality",
		value: "Boring"
		},
		{
		display_type: "boost_number",
		trait_type: "Aqua Power",
		value: 10
		},
		{
		display_type: "boost_percentage",
		trait_type: "Stamina Increase",
		value: 5
		},
		{
		display_type: "number",
		trait_type: "Generation",
		value: 1
		}
		],
		description: "Friendly OpenSea Creature that enjoys long swims in the ocean.",
		external_url: "https://openseacreatures.io/3",
		image: "https://storage.googleapis.com/opensea-prod.appspot.com/creature/3.png",
		name: "Dave Starbelly"
		}
- 如何添加例子的对象到合约里？

		vi contracts/Creature.sol
		
		// SPDX-License-Identifier: MIT

		pragma solidity ^0.8.0;
		
		import "./ERC721Tradable.sol";
		
		/**
		 * @title Creature
		 * Creature - a contract for my non-fungible creatures.
		 */
		contract Creature is ERC721Tradable {
		    constructor(address _proxyRegistryAddress)
		        ERC721Tradable("Creature", "OSC", _proxyRegistryAddress)
		    {}
		
		    function baseTokenURI() override public pure returns (string memory) {
		        return "https://creatures-api.opensea.io/api/creature/"; <--- 这里可以使用自己的 API 替换
		    }
		
		    function contractURI() public pure returns (string memory) {
		        return "https://creatures-api.opensea.io/contract/opensea-creatures"; 
		    }
		}
- 数据类型说明
	- 主参数解析
		- image
		
			这是商品图片的网址。几乎可以是任何类型的图像（包括SVG，它将由 OpenSea 甚至 MP4 缓存到 PNG 中），并且可以是 [IPFS](https://github.com/ipfs/is-ipfs) URL 或路径。我们建议使用 350 x 350 的图像。
		- image_data
		
			原始 SVG 图像数据（如果要动态生成图像）（不建议）。仅在不包含 image 参数的情况下使用此选项
		- external_url
		
			这是 URL，将显示在 OpenSea 资产 image 下方并允许用户离开 OpenSea 并在您的网站上查看项目
		- description
		
			该项目的人类可读描述。支持降价
		- name
		
			项目名称
		- attributes
		
			这些是项目的属性，这些属性将显示在该项目的OpenSea页面上。（见下文）
		- background_color
		
			项目在OpenSea上的背景颜色。必须为六位字符的十六进制，且没有前置＃
		- animation_url
		
			该项目的多媒体附件的 URL。
			
			- 支持文件扩展名 GLTF，GLB，WEBM，MP4，M4V，OGV 和 OGG
			- 以及仅音频扩展名MP3，WAV和OGA
		
			animation_url 还支持 HTML 页面，使您可以使用 JavaScript 画布，WebGL 等来构建丰富的体验。不支持访问浏览器扩展
		- youtube_url
		
			YouTube视频的网址
	- Attributes(属性)

		为了使您的项目更具趣味性，我们还允许您向元数据添加自定义“属性”，这些属性将显示在每个资产的下方。例如，以下是[OpenSea 生物](https://testnets.opensea.io/assets/0x7dca125b1e805dc88814aed7ccc810f677d3e1db/25)之一的属性。

		元数据中包含以下属性数组

			...
			{
			"attributes": [
			    {
			      "trait_type": "Base", 
			      "value": "Starfish"
			    }, 
			    {
			      "trait_type": "Eyes", 
			      "value": "Big"
			    }, 
			    {
			      "trait_type": "Mouth", 
			      "value": "Surprised"
			    }, 
			    {
			      "trait_type": "Level", 
			      "value": 5
			    }, 
			    {
			      "trait_type": "Stamina", 
			      "value": 1.4
			    }, 
			    {
			      "trait_type": "Personality", 
			      "value": "Sad"
			    }, 
			    {
			      "display_type": "boost_number", 
			      "trait_type": "Aqua Power", 
			      "value": 40
			    }, 
			    {
			      "display_type": "boost_percentage",  <-
			      "trait_type": "Stamina Increase", 
			      "value": 10
			    }, 
			    {
			      "display_type": "number", 
			      "trait_type": "Generation", 
			      "value": 2
			    }
			  ]
			}
		这 `trait_type` 是特征的名称，是特征 `value` 的值，并且 `display_type` 是一个字段，指示您希望如何显示它。对于 `string` 特质，您不必担心 `display_type`
	- Numeric Traits(display_type 数字特征)

		对于数字特征，OpenSea 当前支持三个不同的选项，

		- `number`（在下图中的右下方）
		- `boost_percentage`（在上图中的左下方）
		- `boost_number`（类似于 `boost_percentage` 但不显示百分号）
		
		如果您输入 `value` 的是数字而未设置 `display_type`，则该特征将显示在 “排名” 部分中（在上图中的右上方）。
		
		添加可选选项可 `max_value` 为数字特征的可能值设置上限。它默认为 OpenSea 迄今为止在合约资产上看到的最大数量。如果设置为一个  `max_value`，请确保不要传递更高的值 `value`。
	- Date Traits(日期特征)

		OpenSea 还支持 `date` `display_type`。此类型的特征将显示在“排名”和“统计”附近的右列中。传递 unix 时间戳作为值。
		
			{
			      "display_type": "date", 
			      "trait_type": "birthday", 
			      "value": 1546360800
			   }							
		![](./pic/opensea21.png)
		
		如果您不想 `trait_type` 为特定特征拥有一个，则可以在特征中仅包含一个值，并将其设置为通用属性。例如
		
			{
			    "value": "Happy"
			  }
		![](./pic/opensea22.png)	  				  		
		
		也支持 [Enjin元数据样式](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md#erc-1155-metadata-uri-json-schema)
		
			{
			    "properties": {
			        "base": "starfish",
			        "rich_property": {
			            "name": "eyes",
			            "value": "big",
			            "display_value": "Big",
			        }
			                ...
			    }
			}
		虽然建议不要在链上存储元数据，但是如果您认为元数据对于您的用例很重要，请通过 support@opensea.io 与我们联系，我们将与您联系并提出问题和后续步骤。
- 部署源数据服务器

	元数据API：您可以将其托管在 IPFS，云存储或您自己的服务器上。在 Python 和 NodeJS 中都创建了一个示例API服务器：

	- [Python 示例 API 服务器](https://github.com/ProjectOpenSea/metadata-api-python)
	- [NodeJS示例API服务器](https://github.com/ProjectOpenSea/metadata-api-nodejs)

## 如何将铸币展示到 Opensea 上
### 普通打开
- 打开物品地址 

		https://testnets.opensea.io/assets/<your_contract_address>/<token_id>
	例第13个
	
		https://testnets.opensea.io/assets/0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0/13
	![](./pic/Creature6.png)
- 可以强制刷新
		
		https://testnets-api.opensea.io/api/v1/asset/<your_contract_address>/<token_id>/?force_update=true
	例
		
		https://testnets.opensea.io/assets/0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0/13?force_update=true

### 调试打开
- 调试元数据
	会显示全数据
	
		https://testnets-api.opensea.io/asset/<your_contract_address>/<your_token_id>
	例	

		https://testnets-api.opensea.io/asset/0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0/13
	![](./pic/Creature9.png)	
- 加强制刷新
	
	强制刷新状态，会显示全数据
	
		https://testnets-api.opensea.io/asset/<your_contract_address>/<your_token_id>?force_update=true/validate/
	
	![](./pic/Creature8.png)		
- 更新数据

	这样没更新不会变动数据，这样节省带宽
	
		https://testnets-api.opensea.io/asset/<your_contract_address>/<your_token_id>/validate/
	例
	
		https://testnets-api.opensea.io/asset/0x838C12eeC79Ca2d4e6bFc781bFBD375Bd63cA9b0/13/validate/
	或者生产
	
		https://api.opensea.io/asset/<your_contract_address>/<your_token_id>/validate/

	没有更新仅显示空数据			
	
	![](./pic/Creature7.png)

### 创建商店	
现在该在 OpenSea 上创建自己的自定义店面了。OpenSea 店面创建者单击[此处](https://testnets.opensea.io/get-listed/step-two)转到店面创建者。里面填入 Creature 合约号。

![](./pic/Creature10.png)

点击

![](./pic/Creature11.png)

### 销售一个商品
- 首先查看铸造脚本来看，铸造的每一枚 NFT 都是放到我自己的钱包里
	
		vi scripts/mint.js
		
		......
		 else if (NFT_CONTRACT_ADDRESS) {
		    const nftContract = new web3Instance.eth.Contract(
		      NFT_ABI,
		      NFT_CONTRACT_ADDRESS,
		      { gasLimit: "1000000" }
		    );
		
		    // Creatures issued directly to the owner.
		    for (var i = 0; i < NUM_CREATURES; i++) {
		      const result = await nftContract.methods
		        .mintTo(OWNER_ADDRESS)		<----铸造者地址
		        .send({ from: OWNER_ADDRESS }); <----拥有者地址
		      console.log("Minted creature. Transaction: " + result.transactionHash);
		    }
		......
- 现在将出售我们可爱的螃蟹先生，点击进入螃蟹单个物品，并点击销售

	![](./pic/Creature12.png)
- 先从固定价格开始，设置价格，然后发布清单

	![](./pic/Creature13.png)
	
	- 修改页面数据
		- 生成修改请求
			
			![](./pic/Creature14.png)
		- 需要钱包签名操作
	
			![](./pic/Creature15.png)
		- 初始化钱包(只有第一次需要)
	
			![](./pic/Creature16.png)
			![](./pic/Creature17.png)
		- 钱包签名	
			![](./pic/Creature18.png)
		- 完成

			![](./pic/Creature19.png)
			![](./pic/Creature20.png)
			![](./pic/Creature21.png)				
		- 过期时间

			固定价格默认是没有过期的，可以设置
- 价格下降的竞拍设置
	
	![](./pic/Creature22.png)	
	
	- 生成修改请求

		![](./pic/Creature23.png)
	
		- 设置参数包括
			- 开始价格，最高价格
			- 收盘价格，最低价格
			- 截止时间，随时间移动，价格会按比例向收盘价变化
			- 公开/隐私购买(只卖个固定人) 
- ebay 拍卖
	- 生成修改请求

		![](./pic/Creature24.png)
		
		- 设置参数包括
			- 起拍价格，最低价格
			- 低价，最高价格，隐藏的，到了就成交
			- 截止时间，随时间结束，价高得
			- 公开/隐私购买(只卖个固定人) 
- 捆绑销售
	- 选择一个未拍卖，点击捆绑

		![](./pic/Creature25.png)
	- 选择捆绑物品

		![](./pic/Creature26.png)
	- 选择竞拍物属性

		![](./pic/Creature28.png)	
		
		- 名称
		- 短描述
		- 可选价格下降拍卖
		- 选择拍卖开始时间
- 选择自己的 ERC 20 交易
	- 选择币种

		![](./pic/Creature29.png)	

## 主网上线
### 查询 gas 价格
- 查询网站

	https://ethgasstation.info/
- wei/gas 

	从图得知，2021 年 1 gas 平均 90 wei
	![](./pic/mainnet.png)
- 设置推送上线 wei

		vi .env
		
		....
		live: {
	      network_id: 1,
	      provider: function () {
	        return new HDWalletProvider(MNEMONIC, mainnetNodeUrl);
	      },
	      gas: 5000000,
	      gasPrice: 5000000000, <---- 修改这里价格
	      networkCheckTimeout: 10000000,
	      },		 	 
		....
- 设置环境变量

		vi .env
		souce .env		 
- 主网发布

		% truffle deploy --network live		 	
- 其他均和测试网络使用一样

## 自定义店面
- 选择编辑按钮

	![](./pic/collection.png)
	![](./pic/collection1.png)
- 进入编辑店面选项	
	
	![](./pic/collection2.png)
	
	- 编辑选项支持
		- 商店图标
		- 特色图片(可选)
		- 横幅图片(可选)
		- 商店名称(检查是否重名，数据库唯一)
		- 商店网址
		- 商店描述，支持 Md,1000字符串
		- 增加商店类型标签
			- 艺术
			- 收藏
		- 链接
			- 社交媒体，如脸书，小鸟
			- 通讯，如电报、discord
		- 特许使用权
			- 其他人购买你的 NFT 后转售提成(合约如何实现？)
				- 设置百分比
				- 设置被支付的钱包，可以选择支付到铸币合约地址，这样可以监控付款？
			- 付款 token
				- 支持代币
			- 展示排版
				- 三种
			- 显示和敏感信息 
			- 合作者
				- 可以增加他们的地址，更改特许权费用支付地址             
					
## 嵌入开发
- 单商品
	- 点击共享，找寻迁入
	
		![](./pic/collection3.png)
	- 粘贴嵌入代码，仅支持生产环境，所以使用的是生产链接
		   
		    <nft-card
		    contractAddress="0xa75a3f5b447d3d929e5b2ca157cfe7046bc15b37"
		    tokenId="33">
		    </nft-card>
		    <script src="https://unpkg.com/embeddable-nfts/dist/nft-card.min.js"></script>
	- 效果
	
		![](./pic/collection4.png)
		
		- 点击购买跳转  Opensea 销售页面
	
			![](./pic/collection5.png)
		- 点击商店标签，跳转专属商店		    					
			![](./pic/collection6.png)
- 多商品

	老版本支持，新版本没有找到入口			
			
## 问题
- opensea 上的代理账户是什么？为什么只能是一个？
- 什么是 `Alchemy ` 或 `Infura `

	这2个算是以太坊的 API 基础设施提供商，提供商用级别的以太坊 API 调用后台，当然还体统更多服务，具体查看[对应文档](https://github.com/pangzheng/BOOKS/blob/master/%E6%8A%80%E6%9C%AF/%E5%8C%BA%E5%9D%97%E9%93%BE%E6%8A%80%E6%9C%AF%E5%AD%A6%E4%B9%A0/%E4%BB%A5%E5%A4%AA%E5%9D%8A/%E4%BB%80%E4%B9%88%E6%98%AF%20%60ALCHEMY%60%20%E5%92%8C%20%60INFURA%60.md)
- 销售为什么需要签名？

	需要查看合约代码
- 钱包如何调用	 


## 参考
[How to create NFTs and sell them on OpenSea](https://www.youtube.com/watch?v=lbXcvRx0o3Y)	