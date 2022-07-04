# Openzepplin API-通用(Token)/跨链
## 通用(Token)
多个令牌标准共有的功能。

- [ERC2981](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981)：与 ERC721 和 ERC1155 兼容的 NFT 版税。
	- 对于 ERC721，请考虑 [ERC721Royalty](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#ERC721Royalty)在铸造时从存储中指定版税信息。

### 合约
#### ERC2981
	import "@openzeppelin/contracts/token/common/ERC2981.sol";
实施 NFT 版税标准，一种检索版税支付信息的标准化方式。

可以通过 [_setDefaultRoyalty](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setDefaultRoyalty-address-uint96-) 为所有令牌 ID 全局指定版税信息或通过 [_setTokenRoyalty](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setTokenRoyalty-uint256-address-uint96-)为特定令牌 ID 单独指定版税信息。后者优先于前者。

版税被指定为销售价格的一小部分。[_feeDenominator](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_feeDenominator--) 是可覆盖的，但默认为 10000，这意味着费用默认以基点指定。

ERC-2981 仅指定了一种发送版税信息的方式，并不强制其付款。请参阅 EIP 中的[基本原理](https://eips.ethereum.org/EIPS/eip-2981#optional-royalty-payments)。市场预计将自愿支付特许权使用费和销售额，但请注意，该标准尚未得到广泛支持。

从 v4.5 开始可用。

- 功能
	-  [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-supportsInterface-bytes4-)

		公共方法，见 [IERC165.supportsInterface](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)。
		
		- 返回值类型 bool
	- [royaltyInfo(_tokenId, _salePrice)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-royaltyInfo-uint256-uint256-)

		公共方法，根据可能以任何交换单位计价的销售价格，返回应支付的特许权使用费以及向谁支付。特许权使用费金额以相同的交换单位计价并应支付。
		
		- 返回值类型 address
	- [_feeDenominator()](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_feeDenominator--)

		内部方法，用来解释设定的费用 [_setTokenRoyalty](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setTokenRoyalty-uint256-address-uint96-)和[_setDefaultRoyalty](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setDefaultRoyalty-address-uint96-)作为售价的一部分的分母。默认为 10000，因此费用以基点表示，但可以通过覆盖进行自定义。
		
		- 返回值类型 uint96
	- [_setDefaultRoyalty(receiver, feeNumerator)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setDefaultRoyalty-address-uint96-)

		内部方法，设置此合约中所有 id 将默认使用的版税信息。

		- 要求：
			- `receiver` 不能是零地址。
			- `feeNumerator` 不能大于费用分母
	- [_deleteDefaultRoyalty()](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_deleteDefaultRoyalty--)

		内部方法，删除默认版税信息。
	- [_setTokenRoyalty(tokenId, receiver, feeNumerator)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setTokenRoyalty-uint256-address-uint96-)

		内部方法，设置特定令牌 ID 的版税信息，覆盖全局默认值。

		- 要求：
			- `tokenId` 必须已经铸造。
			- `receiver` 不能是零地址。
			- `feeNumerator` 不能大于费用分母。
	- [_resetTokenRoyalty(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_resetTokenRoyalty-uint256-)

		内部方法，将令牌 ID 的版税信息重置为全局默认值。

## 跨链
该目录提供构建块来提高智能合约的跨链方法。

- [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled)是一种抽象，包含访问器和修饰符，用于在接收跨链消息时控制执行流程。

### CrossChainEnabled 专业
以下专业化为特定网桥 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 提供抽象的实现。[CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled)这可用于复合跨链感知组件，例如 [AccessControlCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlCrossChain).
#### CrossChainEnabledAMB [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/amb/CrossChainEnabledAMB.sol)
	import "@openzeppelin/contracts/crosschain/amb/CrossChainEnabledAMB.sol";
[AMB](https://docs.tokenbridge.net/amb-bridge/about-amb-bridge) 专业化或 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled)抽象化。

自 2020 年 2 月起，以下链之间可使用 AMB 桥接器：

- [ETH ⇌ xDai](https://docs.tokenbridge.net/eth-xdai-amb-bridge/about-the-eth-xdai-amb)
- [ETH ⇌ qDai](https://docs.tokenbridge.net/eth-qdai-bridge/about-the-eth-qdai-amb)
- [ETH ⇌ ETC](https://docs.tokenbridge.net/eth-etc-amb-bridge/about-the-eth-etc-amb)
- [ETH ⇌ BSC](https://docs.tokenbridge.net/eth-bsc-amb/about-the-eth-bsc-amb)
- [ETH ⇌ POA](https://docs.tokenbridge.net/eth-poa-amb-bridge/about-the-eth-poa-amb)
- [BSC ⇌ xDai](https://docs.tokenbridge.net/bsc-xdai-amb/about-the-bsc-xdai-amb)
- [POA ⇌ xDai](https://docs.tokenbridge.net/poa-xdai-amb/about-the-poa-xdai-amb)
- [Rinkeby ⇌ xDai](https://docs.tokenbridge.net/rinkeby-xdai-amb-bridge/about-the-rinkeby-xdai-amb)
- [Kovan ⇌ Sokol](https://docs.tokenbridge.net/kovan-sokol-amb-bridge/about-the-kovan-sokol-amb)

从 v4.6 开始可用。/ 合约 CrossCha

- 功能
	- [constructor(bridge)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledAMB-constructor-address-)
	- [_isCrossChain()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledAMB-_isCrossChain--)
	- [_crossChainSender()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledAMB-_crossChainSender--)

#### CrossChainEnabledArbitrumL1 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/arbitrum/CrossChainEnabledArbitrumL1.sol)
	import "@openzeppelin/contracts/crosschain/arbitrum/CrossChainEnabledArbitrumL1.sol";
[Arbitrum](https://arbitrum.io/) 专业化或 [CrossChainEnabledL1](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 端（主网）的抽象。

此版本应仅部署在 L1 上以处理源自 L2 的跨链消息。另一方面，使用 [CrossChainEnabledArbitrumL2](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledArbitrumL2).

桥接合约由仲裁团队提供和维护。您可以在 [Arbitrum 的开发者文档](https://developer.offchainlabs.com/docs/useful_addresses)中的 rinkeby 测试网上找到该合约的地址 。

从 v4.6 开始可用。

- 功能
	- [constructor(bridge)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledArbitrumL1-constructor-address-)
	- [_isCrossChain()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledArbitrumL1-_isCrossChain--)

		内部方法，看 [CrossChainEnabled._isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_isCrossChain--)
		
		- 返回值类型 bool
	- [_crossChainSender()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledArbitrumL1-_crossChainSender--)

		内部方法，看 [CrossChainEnabled._crossChainSender](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_crossChainSender--)
		
		- 返回值类型 address

#### CrossChainEnabledArbitrumL2 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/arbitrum/CrossChainEnabledArbitrumL2.sol)
	import "@openzeppelin/contracts/crosschain/arbitrum/CrossChainEnabledArbitrumL2.sol";
[Arbitrum](https://arbitrum.io/)专业化或 [CrossChainEnabledL2](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 端的抽象（ar​​bitrum）。

此版本仅应部署在 L2 上以处理源自 L1 的跨链消息。另一方面，使用 [CrossChainEnabledArbitrumL1](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledArbitrumL1)

Arbitrum L2 包含 ArbSys 固定地址的合约。因此，这种特化 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 不包括构造函数。

从 v4.6 开始可用。

- 功能
	- [_isCrossChain()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledArbitrumL2-_isCrossChain--)
	
		内部方法，看 [CrossChainEnabled._isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_isCrossChain--)
		
		- 返回值类型 bool
	- [_crossChainSender()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_crossChainSender--)

		内部方法，看 [CrossChainEnabled._crossChainSender](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_crossChainSender--)
		
		- 返回值类型 address

#### CrossChainEnabledOptimism	[github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/optimism/CrossChainEnabledOptimism.sol)
	import "@openzeppelin/contracts/crosschain/optimism/CrossChainEnabledOptimism.sol";
[Optimism](https://www.optimism.io/)专业化或 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled)抽象。

messenger (`CrossDomainMessenger`) 合约由 Optimism 团队提供和维护。您可以在 [Optimism monorepo 的部署](https://github.com/ethereum-optimism/optimism/tree/develop/packages/contracts/deployments)部分中找到该合约在主网和 kovan 上的地址。

从 v4.6 开始可用。

- 功能
	- [constructor(messenger)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledOptimism-constructor-address-)
	- [_isCrossChain()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledOptimism-_isCrossChain--)

		内部方法，看 [CrossChainEnabled._isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_isCrossChain--)
		
		- 返回值类型 bool
	- [_crossChainSender()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledOptimism-_crossChainSender--)

		内部方法，看 [CrossChainEnabled._crossChainSender](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_crossChainSender--)
		
		- 返回值类型 address

#### CrossChainEnabledPolygonChild [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/polygon/CrossChainEnabledPolygonChild.sol)
	import "@openzeppelin/contracts/crosschain/polygon/CrossChainEnabledPolygonChild.sol";
[Polygon](https://polygon.technology/)专业化或 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 抽象子边（Polygon/mumbai）。

此版本只应部署在子链上，以处理源自父链的跨链消息。

fxChild 合约由 Polygon 团队提供和维护。您可以在Polygon 的 [Fx-Portal 文档](https://docs.polygon.technology/docs/develop/l1-l2-communication/fx-portal/#contract-addresses)中找到此合约Polygon和mumbai的地址 。

从 v4.6 开始可用。

- 功能
	- [constructor(fxChild)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledPolygonChild-constructor-address-)
	- [_isCrossChain()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledPolygonChild-_isCrossChain--)

		内部方法，看 [CrossChainEnabled._isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_isCrossChain--)
		
		- 返回值类型 bool
	- [_crossChainSender()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledPolygonChild-_crossChainSender--)

		内部方法，看 [CrossChainEnabled._crossChainSender](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_crossChainSender--)
		
		- 返回值类型 address
	- [processMessageFromRoot(_, rootMessageSender, data)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabledPolygonChild-processMessageFromRoot-uint256-address-bytes-)

		外部方法，用于接收和中继源自 fxChild 的消息的外部入口点。不可重入性对于避免跨链调用能够通过使用用户定义的参数循环来模拟任何人至关重要。注意：如果 _fxChild 调用任何其他执行委托调用的函数，则安全性可能会受到损害。
		
### 跨链库
除了 ` {CrossChainEnable} `抽象之外，跨链感知也可以通过库获得。这些库可用于构建复杂的设计，例如能够与多个网桥交互的合约。
#### LibAMB [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/amb/LibAMB.sol)
	import "@openzeppelin/contracts/crosschain/amb/LibAMB.sol";
使用 [AMB](https://docs.tokenbridge.net/amb-bridge/about-amb-bridge) 系列桥接器的跨链感知合约的原语。

- 功能
	- [isCrossChain(bridge)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibAMB-isCrossChain-address-)

		内部方法，返回当前函数调用是否是由 中继的跨链消息的结果bridge。
		
		- 返回值类型 bool
	- [crossChainSender(bridge)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibAMB-crossChainSender-address-)

		内部方法，通过 .返回触发当前跨链消息的发送者地址bridge。
		
		- 返回值类型 address

		[isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibAMB-isCrossChain-address-) 在尝试恢复发件人之前应检查，因为 `NotCrossChainCall` 如果当前函数调用不是跨链消息的结果，它将恢复。

#### LibArbitrumL1 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/arbitrum/LibArbitrumL1.sol)
	import "@openzeppelin/contracts/crosschain/arbitrum/LibArbitrumL1.sol";
[Arbitrum](https://arbitrum.io/) 的跨链感知合约的原语。

此版本应仅在 L1 上用于处理源自 L2 的跨链消息。另一方面，使用 [LibArbitrumL2](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL2)

- 功能
	- [isCrossChain(bridge)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL1-isCrossChain-address-)

		内部方法，返回当前函数调用是否是由 `bridge`.
	- [crossChainSender(bridge)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL1-crossChainSender-address-)
		
		内部方法，通过返回触发当前跨链消息的发送者地址 `bridge`。

		[isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL1-isCrossChain-address-)在尝试恢复发件人之前应检查，因为 `NotCrossChainCall` 如果当前函数调用不是跨链消息的结果，它将恢复。

#### LibArbitrumL2 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/arbitrum/LibArbitrumL2.sol)
	import "@openzeppelin/contracts/crosschain/arbitrum/LibArbitrumL2.sol";
[Arbitrum](https://arbitrum.io/) 的跨链感知合约的原语。

此版本应仅在 L2 上用于处理源自 L1 的跨链消息。另一方面，使用 [LibArbitrumL1](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL1).

- 功能
	- [isCrossChain(arbsys)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL2-isCrossChain-address-)

		- 返回值类型 bool 
	- [crossChainSender(arbsys)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL2-isCrossChain-address-)

		内部方法，通过  `arbsys` 返回触发当前跨链消息的发送者地址。
		
		- 返回值类型 address
		
		[isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibArbitrumL2-isCrossChain-address-)在尝试恢复发件人之前应检查，因为 `NotCrossChainCall` 如果当前函数调用不是跨链消息的结果，它将恢复。

#### LibOptimism [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/crosschain/optimism/LibOptimism.sol)
	import "@openzeppelin/contracts/crosschain/optimism/LibOptimism.sol";
[Optimism](https://www.optimism.io/) 的跨链感知合约的原语。有关此处使用的功能，请参阅[文档](https://community.optimism.io/docs/developers/bridge/messaging/#accessing-msg-sender) 。

- 功能
	- [isCrossChain(messenger)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibOptimism-isCrossChain-address-)

		内部  `messenger` 方法，返回当前函数调用是否是由 中继的跨链消息的结果。
	- [crossChainSender(messenger)](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibOptimism-crossChainSender-address-)	
		通过 `messenger` 返回触发当前跨链消息的发送者地址。

		[isCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#LibOptimism-isCrossChain-address-) 在尝试恢复发件人之前应检查，因为 `NotCrossChainCall` 如果当前函数调用不是跨链消息的结果，它将恢复。







	
		
	
	
## 替换
令牌