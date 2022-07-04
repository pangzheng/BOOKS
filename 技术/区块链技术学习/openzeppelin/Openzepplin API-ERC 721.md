# Openzepplin API-ERC 721
## ERC 721
这组接口、合约和实用程序都与 [ERC721 Non-Fungible Token Standard](https://eips.ethereum.org/EIPS/eip-721) 相关。

如需了解如何创建 ERC721  Token，请阅读我们的 [ERC721 指南](https://docs.openzeppelin.com/contracts/4.x/erc721)。

- EIP 指定了四个接口
	- [IERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721)

		所有合规实施所需的核心功能。
	- [IERC721Metadata](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata)

		添加名称、符号和 Token URI 的可选扩展，几乎总是包含在内。
	- [IERC721Enumerable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable)

		允许枚举链上 Token的可选扩展，通常不包括在内，因为它需要大量的气体开销。
	- [IERC721Receiver](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver)

		如果合约想要通过 `safeTransferFrom`.

- OpenZeppelin Contracts 提供了所有四个接口的实现
	- [ERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721)
		
		核心和元数据扩展，具有基本 URI 机制。
	- [ERC721Enumerable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Enumerable)

		可枚举的扩展名。
	- [ERC721Holder](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Holder)

		接收器接口的基本实现。
- 此外，还有一些其他扩展：
	- [ERC721URIStorage](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721URIStorage)

		一种更灵活但更昂贵的元数据存储方式。
	- [ERC721Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Votes)

		支持投票和投票委托。
	- [ERC721Royalty](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Royalty)

		一种按照 ERC2981 发出版税信息的方法。
	- [ERC721Pausable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Pausable)

		暂停合约操作的原语。
	- [ERC721Burnable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Burnable)

		代币持有者烧掉自己的代币的一种方式。

这套核心合约被设计为乐观，允许开发人员访问 ERC721 中的内部函数（例如 `_mint`），并以他们喜欢的方式将它们公开为外部函数。另一方面，[ERC721 预设](https://docs.openzeppelin.com/contracts/4.x/erc721#Presets)（例如 [ERC721PresetMinterPauserAutoId](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#token/ERC721/presets.md#ERC721PresetMinterPauserAutoId)）使用乐观的模式设计，为开发人员提供即用型、可部署的合约。

## 核心
### IERC721  [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/IERC721.sol)
	import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
符合 ERC721 的合约所需的接口。

- 功能
	- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-balanceOf-address-)

		外部方法，`owner` 返回的帐户中的 Token数。
		
		- 返回值类型 uint256 balance
	- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ownerOf-uint256-)

		外部方法，`tokenId` 返回 Token的所有者

		- 要求
			- `tokenId` 必须存在
		- 返回值类型 address owner
	- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-bytes-)

		外部方法，安全地将 `tokenId`  Token从 `from` 转移到`to`。首先检查合约接收者是否符合 ERC721 协议，以防止代币被永久锁定。

		- 要求
			- `from` 不能是零地址
			- `to` 不能是零地址
			- `tokenId`  Token必须存在并归 `from` 地址所有
			- 如果调用方不是 `from`，则必须通过 [approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-) 或来授权移动此 Token [setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)。
			- 如果 `to` 指的是智能合约，它必须实现 [IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-)，这被称为安全转移。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)事件
	- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-)

		外部方法，安全地将 `tokenId`  Token从 `from` 转移到`to`。首先检查合约接收者是否符合 ERC721 协议，以防止代币被永久锁定。

		- 要求
			- `from` 不能是零地址
			- `to` 不能是零地址
			- `tokenId`  Token必须存在并归 `from` 地址所有
			- 如果调用方不是 `from`，则必须通过 [approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-) 或来授权移动此 Token [setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)。
			- 如果 `to` 指的是智能合约，它必须实现 [IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-)，这被称为安全转移。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)事件
	- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-)

		外部方法，将 `tokenId`  Token从`from `转移到 `to`。不鼓励使用此方法，[safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-)尽可能使用。
		
		- 要求
			- `from` 不能是零地址
			- `to` 不能是零地址
			- `tokenId`  Token必须存在并归 `from` 地址所有
			- 如果调用方不是 `from`，则必须通过 [approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-) 或来授权移动此 Token [setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)。
		
		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)事件
	- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-)

		外部方法，授予将 Token转移到另一个帐户的权限。`tokenId` 转移 Token时清除授权。一次只能授权一个帐户，因此授权零地址会清除以前的授权。

		- 要求
			- 调用方必须拥有 Token或者是经过授权的操作员。
			- tokenId 必须存在。

		发出一个 [Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-) 事件。
	- [setApprovalForAll(operator, _approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)

		外部方法，授权或移除调用方的  `operator`  身份。 `operator`  可以调用 [transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-) 或 [safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-) 获取调用方拥有的任何 Token。

		- 要求
			- operator 不能是调用方
		
		发出一个 [ApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)事件
	- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-getApproved-uint256-)

		外部方法，`tokenId` 返回为 Token授权的帐户。

		- 要求
			- `tokenId` 必须存在
		- 返回值类型 address operator
	- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-isApprovedForAll-address-address-)

		外部方法，如果 `operator` 允许管理的所有资产，则返回 `owner`。

		看 [setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)
		
		- 返回值类型 bool
	- IERC65 相同方法 
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)
- 事件
	- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)

		当 `tokenId`  Token从 `from` 转移到 `to` 时发出。
	- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)

		`owner` 启用 `approved` 管理 `tokenId`  Token时发出。
	- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

		当 `owner` 启用或禁用 (`approved`) `operator` 以管理其所有资产时发出。

### IERC721Metadata [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/IERC721Metadata.sol)
	import "@openzeppelin/contracts/token/ERC721/extensions/IERC721Metadata.sol";
见 [https://eips.ethereum.org/EIPS/eip-721](https://eips.ethereum.org/EIPS/eip-721)

- 功能
	- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata-name--)

		外部方法，返回 Token集合名称。
		
		- 返回值类型 string
	- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata-symbol--)

		外部方法，返回 Token集合符号。
		
		- 返回值类型 string
	- [tokenURI(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata-tokenURI-uint256-)

		外部方法，返回 `tokenId`  Token的统一资源标识符 (URI)。
		
		- 返回值类型 string
	- IERC721 相同方法
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ownerOf-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-)
		- [setApprovalForAll(operator, _approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-getApproved-uint256-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-isApprovedForAll-address-address-)
	-  IERC65 相同方法 
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-) 
- 事件
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

#### IERC721Enumerable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/IERC721Enumerable.sol)
	import "@openzeppelin/contracts/token/ERC721/extensions/IERC721Enumerable.sol";
见 [https://eips.ethereum.org/EIPS/eip-721](https://eips.ethereum.org/EIPS/eip-721)

- 功能
	- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable-totalSupply--)

		外部方法，返回合约存储的代币总量。
		
		- 返回值类型 uint256
	- [tokenOfOwnerByIndex(owner, index)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable-tokenOfOwnerByIndex-address-uint256-)

		外部方法，返回其 Token列表 `owner` 的给定所拥有的 Token ID。`index`与 [balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-balanceOf-address-) 一起使用可枚举所有 `owner`的标记
		
		- 返回值类型 uint256
	- [tokenByIndex(index)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable-tokenByIndex-uint256-)

		外部方法，返回 `index` 合约存储的所有代币中给定的代币 ID。与 [totalSupply](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable-totalSupply--) 一起使用可枚举所有标记。
		
		- 返回值类型 uint256
	- IERC721 相同方法
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ownerOf-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-)
		- [setApprovalForAll(operator, _approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-getApproved-uint256-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-isApprovedForAll-address-address-)
	- IERC65 相同方法 
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-) 
- 事件
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

### ERC721 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/ERC721.sol)
	import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
ERC721 [Non-Fungible Token Standard](https://eips.ethereum.org/EIPS/eip-721) 的实施，包括 Metadata 扩展，但不包括 Enumerable 扩展，可作为 ERC721Enumerable.

- 功能
	- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-constructor-string-string-)

		公共方法，通过将`name`和`symbol`设置到 Token集合来初始化契约。
	- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-supportsInterface-bytes4-)

		公共方法，见 [IERC165.supportsInterface](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)。
		
		- 返回值类型 bool
	- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-balanceOf-address-)

		公共方法，见 [IERC721.balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-balanceOf-address-)。
		
		- 返回值类型 uint256
	- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-ownerOf-uint256-)

		公共方法，见 [IERC721.ownerOf](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ownerOf-uint256-)。
		
		- 返回值类型 address
	- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-name--)

		公共方法，见 [IERC721Metadata.name](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata-name--)。
		
		- 返回值类型 string
	- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-symbol--)

		公共方法，见 [IERC721Metadata.symbol](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata-symbol--)
		
		- 返回值类型 string
	- [tokenURI(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-tokenURI-uint256-)

		公共方法，见 [IERC721Metadata.tokenURI](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata-tokenURI-uint256-)
		
		- 返回值类型 string
	- [_baseURI()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_baseURI--)

		内部方法，计算 [tokenURI](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-tokenURI-uint256-) 的基本 URI。如果设置，则每个 Token的结果 URI 将是 `baseURI` 和的串联 `tokenId` 。默认为空，可以在子合约中被覆盖。
	- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-)

		公共方法，见 [IERC721.approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-)
	- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-getApproved-uint256-)

		公共方法，见 [IERC721.getApproved](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-getApproved-uint256-)
		
		- 返回值类型 address
	- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-)

		公共方法，见 [IERC721.setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)
	- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-isApprovedForAll-address-address-)

		公共方法，见 [IERC721.isApprovedForAl](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-isApprovedForAll-address-address-)。
		
		- 返回值类型 bool
	- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-)

		公共方法，见 [IERC721.transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-)。
	- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-)

		公共方法,见 [见IERC721.safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-)
	- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)

		公共方法，见 [IERC721.safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-)
	- [_safeTransfer(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeTransfer-address-address-uint256-bytes-)

		内部方法，安全地将代 `tokenId` 币从 `from` 转移到`to`，首先检查合约接收者是否了解 ERC721 协议，以防止代币被永久锁定。

		`data` 是附加数据，它没有指定的格式，它是在调用中发送的 `to`。

		这个内部函数等价于 [safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)，并且可以用于例如实现替代机制来执行 Token传输，例如基于签名的。

		- 要求
			- `from` 不能是零地址。
			- `to` 不能是零地址。
			- `tokenId`  Token必须存在并归 `from`.
			- 如果 `to` 指的是智能合约，它必须实现 [IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-)，这被称为安全转移。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-) 事件。
	- [_exists(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_exists-uint256-)

		内部方法，返回是否 `tokenId` 存在。

		 Token可以由其所有者或通过 [approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-) 或 [setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-) 授权的帐户管理。

		代币在铸造时开始存在 ( `_mint`)，在销毁时停止存在 ( `_burn`)。
		
		- 返回值类型 bool
	- [_isApprovedOrOwner(spender, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_isApprovedOrOwner-address-uint256-)

		内部方法，返回 `spender` 是否允许管理 `tokenId`。
		
		- 要求
			- tokenId必须存在。
		- 返回值类型 bool
	- [_safeMint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)

		内部方法，安全铸造 `tokenId` 并将其转移到 `to`
		
		- 要求
			- `tokenId` 一定不存在
			- 如果 `to` 指的是智能合约，它必须实现 [IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-)，这被称为安全转移。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-) 事件。
	- [_safeMint(to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)

		内部方法，与 [_safeMint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-) 相同，带有一个附加 `data` 参数，该参数被转发 [IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-) 给合约接收者。
	- [_mint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-)

		内部方法，铸币厂铸造 `tokenId` 和将其转移到 `to`. 不鼓励使用此方法，[_safeMint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)尽可能使用
		
		- 要求
			- `tokenId` 一定不存在。
			- `to` 不能是零地址。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)事件。
	- [_burn(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)

		内部方法，销毁 `tokenId`。销毁 Token时清除授权。

		- 要求
			- `tokenId` 必须存在。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)事件
	- [_transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_transfer-address-address-uint256-)

		内部方法，`tokenId` 从 `from` 转移到 `to`。与 [transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-) 相反，这对 `msg.sender` 没有任何限制。
		
		- 要求
			- `to` 不能是零地址。
			- `tokenId`  Token必须由 `from` 拥有。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)事件。
	- [_approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_approve-address-uint256-)

		内部方法，授权 `to`  操作 `tokenId`

		发出一个 [Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-) 事件。
	- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_setApprovalForAll-address-address-bool-)

		内部方法，授权 `operator` 对所有 `owner` 代币进行操作

		发出一个 [ApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-) 事件。
	- [_requireMinted(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_requireMinted-uint256-)

		内部方法，如果 `tokenId` 尚未铸造，则恢复。
	- [_beforeTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_beforeTokenTransfer-address-address-uint256-)

		内部方法，在任何 Token传输之前调用的钩子。这包括铸造和销毁
		
		- 调用条件
			- 当 `from` 和 `to` 都非零时，`from` 将 `tokenId` 转移到 `to`。
			- 当 `from` 为零时，`tokenId` 将被铸造到 `to`.
			- 当 `to` 为零时，`from` 将 `tokenId` 被销毁。
			- `from` 和 `to` 永远都不为零。

		要了解有关钩子的更多信息，请前往[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)。
	- [_afterTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_afterTokenTransfer-address-address-uint256-)

		内部方法，在任何代币转移后调用的挂钩。这包括铸造和销毁。
		
		- 调用条件
			- 当 `from` 和 `to` 都非零
			- `from` 和 `to` 永远都不为零。
		
		要了解有关钩子的更多信息，请前往[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)。
- 事件
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

### ERC721Enumerable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/ERC721Enumerable.sol)
	import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
这实现了 EIP 中定义的可选扩展 [ERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721)，增加了合约中所有代币 ID 以及每个账户拥有的所有代币 ID 的可枚举性。

- 功能
	- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Enumerable-supportsInterface-bytes4-)

		公共方法，见[IERC165.supportsInterface](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)。
		
		- 返回值类型 bool
	- [tokenOfOwnerByIndex(owner, index)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Enumerable-tokenOfOwnerByIndex-address-uint256-)

		公共方法，见[IERC721Enumerable.tokenOfOwnerByIndex](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable-tokenOfOwnerByIndex-address-uint256-)。
		
		- 返回值类型  unit256
	- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Enumerable-totalSupply--)

		公共方法，见 [IERC721Enumerable.totalSupply](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable-totalSupply--)。
		
		- 返回值类型 unit256
	- [tokenByIndex(index)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Enumerable-tokenByIndex-uint256-)

		公共方法，见[IERC721Enumerable.tokenByIndex](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Enumerable-tokenByIndex-uint256-)。
		
		- 返回值类型 unit256
	- [_beforeTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Enumerable-_beforeTokenTransfer-address-address-uint256-)

		内部方法，在任何 Token传输之前调用的钩子。这包括铸造和销毁。

		- 调用条件
			- 当 `from` 和 `to` 都非零时，`from` 将 `tokenId` 转移到 `to`。
			- 当 `from` 为零时，`tokenId` 将被铸造为 `to`.
			- 当 `to` 为零时，`from` 将 `tokenId` 被销毁。
			- `from` 不能是零地址。
			- `to` 不能是零地址。

			要了解有关钩子的更多信息，请前往[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)。
	- ERC 721 功能相同			
		- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-constructor-string-string-)
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-ownerOf-uint256-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-symbol--)
		- [tokenURI(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-tokenURI-uint256-)
		- [_baseURI()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_baseURI--)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-getApproved-uint256-)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-isApprovedForAll-address-address-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [_safeTransfer(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeTransfer-address-address-uint256-bytes-)
		- [_exists(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_exists-uint256-)
		- [_isApprovedOrOwner(spender, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_isApprovedOrOwner-address-uint256-)
		- [_safeMint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)
		- [_safeMint(to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)
		- [_mint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-)
		- [_burn(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)
		- [_transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_transfer-address-address-uint256-)
		- [_approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_approve-address-uint256-)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_setApprovalForAll-address-address-bool-)
		- [_requireMinted(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_requireMinted-uint256-)
		- [_afterTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_afterTokenTransfer-address-address-uint256-)
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

#### IERC721Receiver [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/IERC721Receiver.sol)
	import "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
任何想要支持来自 ERC721 资产合约的 safeTransfers 的合约的接口。

- 功能
	- [onERC721Received(operator, from, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-)

		外部方法，当一个 [IERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721)  Token id 通过  [IERC721.safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-) 通过 `operator ` 操作符 `from`，调用此函数。它必须返回它的 Solidity 选择器来确认 Token传输。如果返回任何其他值或接收方未实现接口，则将恢复传输。

		该选择器可以在 Solidity 中使用 `IERC721Receiver.onERC721Received.selector` 获得。
		
		- 返回值类型 bytes4

## 扩展
### ERC721Pausable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/ERC721Pausable.sol)
	import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Pausable.sol";
ERC721 代币具有可暂停的代币转移、铸造和销毁。

对于在评估期结束之前阻止交易或在出现大错误时冻结所有代币转移的紧急开关等场景很有用。

- 功能
	- [_beforeTokenTransfer(from, to, tokenId)	](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Pausable-_beforeTokenTransfer-address-address-uint256-)

		内部方法，见 [ERC721._beforeTokenTransfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_beforeTokenTransfer-address-address-uint256-)

		- 要求
			- 不得暂停合约
	- PAUSABLE 功能相同
		- [constructor()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-constructor--)
		- [paused()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-paused--)
		- [_requireNotPaused()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_requireNotPaused--)
		- [_requirePaused()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_requirePaused--)
		- [_pause()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_pause--)
		- [_unpause()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_unpause--)
	- ERC 721 功能相同			
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-supportsInterface-bytes4-)
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-ownerOf-uint256-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-symbol--)
		- [tokenURI(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-tokenURI-uint256-)
		- [_baseURI()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_baseURI--)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-getApproved-uint256-)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-isApprovedForAll-address-address-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [_safeTransfer(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeTransfer-address-address-uint256-bytes-)
		- [_exists(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_exists-uint256-)
		- [_isApprovedOrOwner(spender, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_isApprovedOrOwner-address-uint256-)
		- [_safeMint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)
		- [_safeMint(to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)
		- [_mint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-)
		- [_burn(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)
		- [_transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_transfer-address-address-uint256-)
		- [_approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_approve-address-uint256-)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_setApprovalForAll-address-address-bool-)
		- [_requireMinted(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_requireMinted-uint256-)
		- [_afterTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_afterTokenTransfer-address-address-uint256-)
- 事件
	-  PAUSABLE 功能相同
		- [Paused(account)](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-Paused-address-)
		- [Unpaused(account)](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-Unpaused-address-)
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

### ERC721Burnable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/ERC721Burnable.sol)

	import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
ERC721 可被销毁的代币。

- 功能
	- [burn(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Burnable-burn-uint256-)

		公共方法，烧伤 `tokenId`。见 [ERC721._burn](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)。

		- 要求
			- 调用方必须拥有 `tokenId` 或为经授权的 `operator`。
	- ERC 721 功能相同			
		- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-constructor-string-string-)
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-supportsInterface-bytes4-)
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-ownerOf-uint256-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-symbol--)
		- [tokenURI(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-tokenURI-uint256-)
		- [_baseURI()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_baseURI--)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-getApproved-uint256-)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-isApprovedForAll-address-address-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [_safeTransfer(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeTransfer-address-address-uint256-bytes-)
		- [_exists(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_exists-uint256-)
		- [_isApprovedOrOwner(spender, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_isApprovedOrOwner-address-uint256-)
		- [_safeMint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)
		- [_safeMint(to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)
		- [_mint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-)
		- [_burn(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)
		- [_transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_transfer-address-address-uint256-)
		- [_approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_approve-address-uint256-)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_setApprovalForAll-address-address-bool-)
		- [_requireMinted(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_requireMinted-uint256-)
		- [_beforeTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_beforeTokenTransfer-address-address-uint256-)
		- [_afterTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_afterTokenTransfer-address-address-uint256-)
- 事件
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

#### ERC721URIStorage [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/ERC721URIStorage.sol)
	import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
具有基于存储的 Token URI 管理的 ERC721  Token。

- 功能
	- [tokenURI(tokenId)]()
	
		公共方法，见 [IERC721Metadata.tokenURI](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Metadata-tokenURI-uint256-)
		
		- 返回类型 string
	- [_setTokenURI(tokenId, _tokenURI)]()

		内部方法，设置 `_tokenURI` 为 `tokenId` 的 tokenURI。

		- 要求
			- `tokenId` 必须存在。 
	- [_burn(tokenId)]()

		内部方法，见 [ERC721._burn](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)。此覆盖还会检查是否为 Token设置了特定于 Token的 URI，如果是，则从存储映射中删除 Token URI。
	- ERC 721 功能相同			
		- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-constructor-string-string-)
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-supportsInterface-bytes4-)
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-ownerOf-uint256-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-symbol--)
		- [_baseURI()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_baseURI--)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-getApproved-uint256-)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-isApprovedForAll-address-address-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [_safeTransfer(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeTransfer-address-address-uint256-bytes-)
		- [_exists(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_exists-uint256-)
		- [_isApprovedOrOwner(spender, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_isApprovedOrOwner-address-uint256-)
		- [_safeMint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)
		- [_safeMint(to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)
		- [_mint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-)
		- [_transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_transfer-address-address-uint256-)
		- [_approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_approve-address-uint256-)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_setApprovalForAll-address-address-bool-)
		- [_requireMinted(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_requireMinted-uint256-)
		- [_beforeTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_beforeTokenTransfer-address-address-uint256-)
		- [_afterTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_afterTokenTransfer-address-address-uint256-)
- 事件
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

### ERC721Votes [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/draft-ERC721Votes.sol)
	import "@openzeppelin/contracts/token/ERC721/extensions/draft-ERC721Votes.sol";
扩展 ERC721 以支持由 [Votes](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes) 实施的投票和委托，其中每个 NFT 算作 1 个投票单元。

代币在被授权之前不算作投票，因为必须跟踪投票，这会在每次转移时产生额外费用。代币持有者可以委托一个受信任的代表来决定如何在治理决策中使用投票权，或者他们可以委托自己作为自己的代表。

从 v4.5 开始可用。

- 功能
	- [_afterTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Votes-_afterTokenTransfer-address-address-uint256-)

		内部转移转移代币时调整投票。

		发出 `{Votes-DelegateVotesChanged}` 事件。
	- [_getVotingUnits(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Votes-_getVotingUnits-address-)

		内部，返回 `account` 的余额
	- 投票功能相同
		- [getVotes(account)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-getVotes-address-)
		- [getPastVotes(account, blockNumber)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-getPastVotes-address-uint256-)
		- [getPastTotalSupply(blockNumber)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-getPastTotalSupply-uint256-)
		- [_getTotalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-_getTotalSupply--)
		- [delegates(account)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-delegates-address-)
		- [delegate(delegatee)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-delegate-address-)
		- [delegateBySig(delegatee, nonce, expiry, v, r, s)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-delegateBySig-address-uint256-uint256-uint8-bytes32-bytes32-)
		- [_delegate(account, delegatee)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-_delegate-address-address-)
		- [_transferVotingUnits(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-_transferVotingUnits-address-address-uint256-)
		- [_useNonce(owner)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-_useNonce-address-)
		- [nonces(owner)](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-nonces-address-)
		- [DOMAIN_SEPARATOR()](https://docs.openzeppelin.com/contracts/4.x/api/governance#Votes-DOMAIN_SEPARATOR--)
	- EIP712
		- [constructor(name, version)](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-constructor-string-string-)
		- [_domainSeparatorV4()](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_domainSeparatorV4--)
		- [_hashTypedDataV4(structHash)](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_hashTypedDataV4-bytes32-) 
	- ERC 721 功能相同			
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-supportsInterface-bytes4-)
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-ownerOf-uint256-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-symbol--)
		- [_baseURI()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_baseURI--)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-getApproved-uint256-)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-isApprovedForAll-address-address-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [_safeTransfer(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeTransfer-address-address-uint256-bytes-)
		- [_exists(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_exists-uint256-)
		- [_isApprovedOrOwner(spender, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_isApprovedOrOwner-address-uint256-)
		- [_safeMint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)
		- [_safeMint(to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)
		- [_mint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-)
		- [_burn(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)
		- [_transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_transfer-address-address-uint256-)
		- [_approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_approve-address-uint256-)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_setApprovalForAll-address-address-bool-)
		- [_requireMinted(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_requireMinted-uint256-)
		- [_beforeTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_beforeTokenTransfer-address-address-uint256-)
- 事件
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)
	- 投票功能相同
		- [DelegateChanged(delegator, fromDelegate, toDelegate)](https://docs.openzeppelin.com/contracts/4.x/api/governance#IVotes-DelegateChanged-address-address-address-) 
		- [DelegateVotesChanged(delegate, previousBalance, newBalance)](https://docs.openzeppelin.com/contracts/4.x/api/governance#IVotes-DelegateVotesChanged-address-uint256-uint256-)

### ERC721Royalty [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/extensions/ERC721Royalty.sol)
	import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Royalty.sol";
使用 ERC2981 NFT 版税标准扩展 ERC721，这是一种检索版税支付信息的标准化方法。

版税信息可以通过 `{_setDefaultRoyalty}` 为所有代币 ID 全局指定或通过 `{_setTokenRoyalty}` 为特定代币 ID 单独指定。后者优先于前者。

ERC-2981 仅指定了一种发送版税信息的方式，并不强制其付款。请参阅 EIP 中的[基本原理](https://eips.ethereum.org/EIPS/eip-2981#optional-royalty-payments)。市场预计将自愿支付特许权使用费和销售额，但请注意，该标准尚未得到广泛支持。从 v4.5 开始可用。

- 功能
	- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Royalty-supportsInterface-bytes4-)
	
		公共方法，见[IERC165.supportsInterface。](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)
		
		- 返回类型 bool
	- [_burn(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Royalty-_burn-uint256-)

		内部方法，见 [ERC721._burn](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_burn-uint256-)。此覆盖还会清除 Token的版税信息。
	- ERC 721 功能相同			
		- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-constructor-string-string-)
		- [balanceOf(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-balanceOf-address-)
		- [ownerOf(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-ownerOf-uint256-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-symbol--)
		- [tokenURI(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-tokenURI-uint256-)
		- [_baseURI()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_baseURI--)
		- [approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-approve-address-uint256-)
		- [getApproved(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-getApproved-uint256-)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-setApprovalForAll-address-bool-)
		- [isApprovedForAll(owner, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-isApprovedForAll-address-address-)
		- [transferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-transferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-)
		- [safeTransferFrom(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-safeTransferFrom-address-address-uint256-bytes-)
		- [_safeTransfer(from, to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeTransfer-address-address-uint256-bytes-)
		- [_exists(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_exists-uint256-)
		- [_isApprovedOrOwner(spender, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_isApprovedOrOwner-address-uint256-)
		- [_safeMint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-)
		- [_safeMint(to, tokenId, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)
		- [_mint(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-)
		- [_transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_transfer-address-address-uint256-)
		- [_approve(to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_approve-address-uint256-)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_setApprovalForAll-address-address-bool-)
		- [_requireMinted(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_requireMinted-uint256-)
		- [_beforeTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_beforeTokenTransfer-address-address-uint256-)
		- [_afterTokenTransfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_afterTokenTransfer-address-address-uint256-)
	- ERC 2981 功能相同
		- [royaltyInfo(_tokenId, _salePrice)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-royaltyInfo-uint256-uint256-)
		- [_feeDenominator()](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_feeDenominator--)
		- [_setDefaultRoyalty(receiver, feeNumerator)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setDefaultRoyalty-address-uint96-)
		- [_deleteDefaultRoyalty()](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_deleteDefaultRoyalty--)
		- [_setTokenRoyalty(tokenId, receiver, feeNumerator)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_setTokenRoyalty-uint256-address-uint96-)
		- [_resetTokenRoyalty(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/common#ERC2981-_resetTokenRoyalty-uint256-)
- 事件
	- IERC 721 相同方法 
		- [Transfer(from, to, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Transfer-address-address-uint256-)
		- [Approval(owner, approved, tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-Approval-address-address-uint256-)
		- [ApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-ApprovalForAll-address-address-bool-)

## 预设
这些合约是上述功能的预配置组合。它们可以通过继承或作为模型来复制和粘贴其源代码。
## 实用程序
### ERC721Holder [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC721/utils/ERC721Holder.sol)
	import "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
接口的实现 [IERC721Receiver](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver)。

接受所有代币转移。确保合约能够将其代币与 [IERC721.safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-safeTransferFrom-address-address-uint256-),[IERC721.approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-approve-address-uint256-)或[IERC721.setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-setApprovalForAll-address-bool-)一起使用。

- 功能
	- [onERC721Received(_, _, _, _)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Holder-onERC721Received-address-address-uint256-bytes-)

		公共方法，见 [IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-)

		总是返回 `IERC721Receiver.onERC721Received.selector`。
		
		- 返回值类型 bytes4