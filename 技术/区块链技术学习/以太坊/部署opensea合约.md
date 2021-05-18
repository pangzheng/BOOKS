# 部署 opensea 合约
opensea 文档说的合约，最新版本已经更新了,下面是2021年5月最新的 [readme 文档](https://github.com/ProjectOpenSea/opensea-creatures)
## OpenSea生物：初学者 ERC721，ERC1155 和工厂合同
### ERC721 / ERC1155合同样本
其中包括一个非常简单的示例 ERC721/ERC1155，目的是演示与 OpenSea 市场的集成。我们包括用于铸造物品的脚本。

此外，该合同将 OpenSea 用户的代理帐户列入白名单，以便他们能够自动在 OpenSea 上交易 ERC721 项目（无需支付额外的批准费用）。在 OpenSea 上每个用户都有一个他们控制的 “代理” 帐户，最终由交换合同调用以交易其项目。（请注意，此添加并不意味着 OpenSea 本身就可以访问这些项目，只是让用户可以根据需要更轻松地列出它们）
### 工厂合同
除了这些模板 721/1155 合同之外，我们还提供样本工厂合同，用于进行尚未铸造的物品的免 gas 预售。有关更多信息，请参见 https://docs.opensea.io/docs/opensea-initial-item-sale-tutorial。
### 要求
- 节点版本

	确保您正在运行符合 `engines` 要求的 `package.json` 节点版本，或者安装节点版本管理器 [nvm](https://github.com/creationix/nvm) 并运行 `nvm use` 以使用正确的节点版本。

### 安装
- 运行 

		yarn

	如果您在构建依赖项时遇到错误，并且您使用的是 Mac，请运行以下代码，删除 `node_modules` 文件夹，然后重新执行以下操作 `yarn install`：

		xcode-select --install # 安装命令行工具（如果尚未安装）.
		sudo xcode-select --switch /Library/Developer/CommandLineTools # 打开命令工具行
		sudo npm explore npm -g -- npm install node-gyp@latest # Update node-gyp 更新 node-gyp

### 部署中
- 部署到 Rinkeby 网络
	- 要访问 Rinkeby testnet 节点获取密钥
		- 您需要注册 [Alchemy](https://dashboard.alchemyapi.io/signup?referral=affiliate:e535c3c3-9bc4-428f-8e27-4b70aa2e8ca5) 并获得免费的 API 密钥。单击“查看密钥”，然后在后面复制URL的一部分 `v2/`。
		- 也可以使用 [Infura](https://infura.io/)。只需将 `ALCHEMY_KEY` 下面更改为即可 `INFURA_KEY`。
	- 使用您的 API 密钥和 Metamask 钱包的助记符（确保您正在使用适合用于测试目的的 Metamask 种子短语），运行：

			export ALCHEMY_KEY="<your_alchemy_project_id>"
			export MNEMONIC="<metmask_mnemonic>"
			DEPLOY_CREATURES_SALE=1 yarn truffle deploy --network rinkeby
		
### 铸造令牌
部署到 Rinkeby 网络后，将在 Rinkeby 上签订一份合同，该合同将在 [Rinkeby Etherscan](https://rinkeby.etherscan.io/) 上可见。例如，这是[最近部署的合同](https://rinkeby.etherscan.io/address/0xeba05c5521a3b81e23d15ae9b2d07524bc453561)。

运行铸造脚本时，应将此合同地址和 Metamask 帐户的地址设置为环境变量。如果[部署了CreatureFactory](https://github.com/ProjectOpenSea/opensea-creatures/blob/master/migrations/2_deploy_contracts.js#L38)，如上面的示例部署步骤所述，则您需要在下面指定其地址，因为它是 NFT 合同的所有者，并且只有它具有 `mint `(薄荷糖)权限。在这种情况下，您将不需要`NFT_CONTRACT_ADDRESS`，因为我们所需要的只是这里带有 `mint`(薄荷)权限的合同。

	export OWNER_ADDRESS="<my_address>"
	export NFT_CONTRACT_ADDRESS="<deployed_contract_address>"
	export FACTORY_CONTRACT_ADDRESS="<deployed_factory_contract_address>"
	export NETWORK="rinkeby"
	node scripts/mint.js

### 诊断常见问题
如果您正在运行的修改版本，`sell.js` 但未获得预期的行为，请检查以下内容：

-  `expirationTime` 是将来吗？如果否，请将其更改为将来的某个时间。
- `expirationTime` 是小数秒吗？如果是，请将时间四舍五入到最接近的秒数。
- 输入地址是否全部为字符串？如果否，请将其转换为字符串。
- 输入地址是否经过校验和？您可能需要使用地址的校验和版本。
- 您计算机的内部时钟是否准确？如果不是，请尝试在本地启用自动时钟调整，或者按照[本教程](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html)的说明更新 Amazon EC2 实例。
- 您是否有因全局安装的节点程序包而导致的任何冲突？如果是，请尝试 `yarn remove -g truffle; yarn`
- 您运行的节点版本是否符合中的 `engines` 要求 `package.json`？如果没有，请尝试 `nvm use; rm -rf node_modules; yarn`

## 关于 OpenSea 生物配件
这是一份 ERC-1155 合同样本，目的是演示与 OpenSea 加密收藏品市场的集成。我们还包括：

- 一份工厂合同，用于制作未铸造物品的销售订单（允许免费 gas和免费 `mint`(薄荷) 的预售）。
- 可配置的战利品合同，用于出售 ERC-1155 项目的随机集合。

除了上述 OpenSea ERC721 样本合同的功能之外，ERC1155

- 每个合同支持多个创建者，其中只有创建者才能创建更多副本
- 支持战利品箱中预先铸造的物品供您选择

## 配置战利品箱
打开 `CreatureAccessoryLootbox.sol`

- 进行更改 `Class` 以反映您的稀有程度。 
- 进行更改 `NUM_CLASSES` 以反映您有多少个类（用于在 Solidity 中调整固定长度数组的大小）
- 在中 `constructor`，`OptionSettings` 为您的每个级设置。为此，如示例中所示，`setOptionSettings`使用
	- 您的选项 ID
	- 打开包装盒时要发出的物品数
	- 接收每个类的概率数组（基点，因此为10,000的整数）。应加起来为10k且值递减。
- 然后按照以下说明进行部署！购买后将自动打开该框。如果您想使用户可以交易战利品（无需购买即可自动打开），请通过 contact@opensea.io（或更好的是Discord）与我们联系。

## 为什么某些标准方法会被覆盖？
该合同优先 `isApprovedForAll` 于将 OpenSea 用户的代理帐户列入白名单的方法。这意味着他们可以自动在 OpenSea 上交易您的 ERC-1155 项目（无需支付额外的批准费用）。在 OpenSea 上，每个用户都有一个他们控制的“代理”帐户，最终由交换合同调用以交易其项目。

请注意，这种添加并不意味着 OpenSea 本身就可以访问这些项目，只是让用户可以根据需要更轻松地列出它们！

### 要求
- 节点版本

	确保您正在运行符合 `engines` 要求的 `package.json` 节点版本，或安装节点版本管理器 [nvm](https://github.com/creationix/nvm) 并运行 `nvm use` 以使用正确的节点版本。

### 安装
- 运行

		yarn

### 部署中
- 部署到 Rinkeby 网络。
	- 要访问 Rinkeby testnet 节点获取密钥
		- 您需要注册 [Alchemy](https://dashboard.alchemyapi.io/signup?referral=affiliate:e535c3c3-9bc4-428f-8e27-4b70aa2e8ca5) 并获得免费的 API 密钥。单击“查看密钥”，然后在后面复制URL的一部分 `v2/`。
		- 也可以使用 [Infura](https://infura.io/)。只需将 `ALCHEMY_KEY` 下面更改为即可 `INFURA_KEY`。
	- 使用您的 API 密钥和 Metamask 钱包的助记符（确保您正在使用适合用于测试目的的 Metamask 种子短语），运行：

			export ALCHEMY_KEY="<your_alchemy_project_id>" #或者可以使用 infura key
			export MNEMONIC="<metmask_mnemonic>"
			DEPLOY_CREATURES_SALE=1 yarn truffle deploy --network rinkeby
- 部署到主网以太坊网络。

	确保您的钱包中至少有几美元的以太币。然后运行
	
		yarn truffle migrate --network live		
	在日志中查找您新部署的合同地址！
- 在OpenSea上查看您的物品

	OpenSea 将自动收取您合同上的转账。您可以通过访问来访问资产 `https://opensea.io/assets/CONTRACT_ADDRESS/TOKEN_ID`。
	
	要一次将所有元数据加载到商品上
	
	- 生产网络
	
		请访问 [https://opensea.io/get-listed](https://opensea.io/get-listed) 并输入您的地址以将元数据加载到 OpenSea 中！
	- 测试网络
	
		如果您已经部署了 Rinkeby 测试网络，甚至可以通过以下网址对此进行测试：[https://rinkeby.opensea.io/get-listed](https://rinkeby.opensea.io/get-listed)

### 故障排除
- 不编译

	在本地安装松露：`yarn add truffle`。然后运行 `yarn truffle migrate ...`。

	您还可以通过运行调试仅编译步骤 `yarn truffle compile`。
- 它没有部署任何东西！

	这通常是由于 `truffle-hdwallet` 提供程序无法连接。转到 [Alchemy仪表板](https://dashboard.alchemyapi.io/signup?referral=affiliate:e535c3c3-9bc4-428f-8e27-4b70aa2e8ca5)（或infura.io）并创建一个新项目。使用“项目ID”作为新的项目 `ALCHEMY_KEY`，并确保您导出上面的命令行变量。

### ERC1155 实施
为了实施 ERC1155 标准，这些合同使用 [Horizo​​n Games](https://horizongames.net/)的《多令牌标准》，该协议可在[npm](https://www.npmjs.com/package/multi-token-standard)和[github](https://github.com/arcadeum/multi-token-standard) 上获得，并获得MIT许可证。	
## 运行本地测试
在一个终端窗口中，运行：

	yarn run ganache-cli
Ganache启动后，请在另一个终端窗口中运行以下命令

	yarn run test	
	



