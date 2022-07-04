# Openzepplin API-ERC 1155
## ERC 1155
这套接口和合约都与 [ERC1155 Multi Token Standard](https://eips.ethereum.org/EIPS/eip-1155)相关。

EIP 由三个接口组成，它们扮演着不同的角色，在 [IERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155) 这里可以找到  [IERC1155MetadataURI](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155MetadataURI) 和[IERC1155Receiver](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver)。

[ERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155) 通过依赖替换机制为所有Token类型使用相同的 URI，实现了强制[IERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155)接口以及可选扩展，从而显着降低了[IERC1155MetadataURI](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155MetadataURI) gas 成本。

此外，还有多个自定义扩展，包括：

- 指定可以暂停所有用户的Token传输的地址 ([ERC1155Pausable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Pausable))。
- 销毁自己的Token（[ERC1155Burnable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Burnable)）。

这套核心合约被设计为 unopinionated，允许开发人员访问 ERC1155 中的内部函数（例如 [_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)），并以他们喜欢的方式将它们公开为外部函数。另一方面，[ERC1155 预设](https://docs.openzeppelin.com/contracts/4.x/erc1155#Presets)（例如 [ERC1155PresetMinterPauser](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#token/ERC1155/presets.md#ERC1155PresetMinterPauser)）是使用乐观的模式设计的，为开发人员提供即用型、可部署的合约。

## 核心
### IERC1155 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/IERC1155.sol)
	import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
符合 ERC1155 的合约的必需接口，如 [EIP](https://eips.ethereum.org/EIPS/eip-1155) 中所定义。从 v3.1 开始可用。

- 功能
	- [balanceOf(account, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-balanceOf-address-uint256-)

		外部方法 ,`id`返回 `account ` 拥有的Token类型的Token数量。
		
		- 要求
			- `account` 不能是零地址
		- 返回值类型 uint256
	- [balanceOfBatch(accounts, ids)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-balanceOfBatch-address---uint256---)

		外部方法，[批处理](https://docs.openzeppelin.com/contracts/4.x/erc1155#batch-operations) [balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-balanceOf-address-uint256-) 版本。
		
		- 要求
			- `accounts`和 `ids` 必须具有相同的长度。
		- 返回值类型 uint256[]
	- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-setApprovalForAll-address-bool-)

		根据 ，授予或撤销 `operator` 转移调用者 `approved` Token的权限，

		发出一个 [ApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)事件。
		
		- 要求
			- `operator` 不能是调用者。
	- [isApprovedForAll(account, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-isApprovedForAll-address-address-)

		如果 `operator` 批准转移 `account` 的Token，则返回 true 。

		见 [setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-setApprovalForAll-address-bool-)。
		
		- 返回值类型 bool
	- [safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)

		外部方法，将Token `id` 的  `amount`  Token从 `from ` 转移到 `to`。
		
		发出 [TransferSingle](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)事件。
		
		- 要求
			- `to` 不能是零地址
			- 如果调用者不是`from`，则它必须已被批准`from` 通过 `setApprovalForAll`
			- `from` 必须有 `id`至少 `amount` 的Token余额。
			- 如果 `to` 指的是智能合约，它必须实现 [IERC1155Receiver.onERC1155Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-)并返回接受魔法值。
	- [safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)

		外部方法，批处理 [safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-) 版本。

		发出 [TransferBatch](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)事件。

		- 要求
			- `ids` 和 `amounts` 必须具有相同的长度。
			- 如果 `to` 指的是智能合约，它必须实现 [IERC1155Receiver.onERC1155BatchReceived](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155BatchReceived-address-address-uint256---uint256---bytes-)并返回接受魔法值。
	- IERC165
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)
- 事件
	- [TransferSingle(operator, from, to, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)

		`operator`操作当 `value`Token类型的Token `id` 从 `from ` 转移到 `to` 时发出 。
	- [TransferBatch(operator, from, to, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)

		等效于多个 [TransferSingle](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)事件，其中 `operator` 和 `from` 对`to`所有传输都相同。
	- [ApprovalForAll(account, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)

		根据. `account_ operator_ approved`
	- [URI(value, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)

		当Token类型的 URI `id` 更改为 `value` 时发出，如果它是非编程 URI。

		如果 [URI](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)为 `id` 发出事件，则标准[保证](https://eips.ethereum.org/EIPS/eip-1155#metadata-extensions)将  `value`  等于返回的值[IERC1155MetadataURI.uri](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155MetadataURI-uri-uint256-)。

### IERC1155MetadataURI [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/extensions/IERC1155MetadataURI.sol)
	import "@openzeppelin/contracts/token/ERC1155/extensions/IERC1155MetadataURI.sol";
可选 ERC1155MetadataExtension 接口的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-1155#metadata-extensions)中所定义。

从 v3.1 开始可用。	

- 功能
	- [uri(id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155MetadataURI-uri-uint256-)

		外部方法，返回Token类型的 URI`id`。

		如果 `{id}` URI 中存在子字符串，则必须由具有实际Token类型 ID 的客户端替换。
	- IERC1155 功能相同
		- [balanceOf(account, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-balanceOf-address-uint256-)
		- [balanceOfBatch(accounts, ids)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-balanceOfBatch-address---uint256---)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-setApprovalForAll-address-bool-)
		- [isApprovedForAll(account, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-isApprovedForAll-address-address-)
		- [safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
	- IERC165
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)
- 事件
	- IERC1155
		- [TransferSingle(operator, from, to, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)
		- [TransferBatch(operator, from, to, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)
		- [ApprovalForAll(account, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)
		- [URI(value, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)

### ERC1155 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/ERC1155.sol)
	import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
基本标准多Token的实现。参见 [https://eips.ethereum.org/EIPS/eip-1155]() 最初基于 Enjin 的代码：[https ://github.com/enjin/erc-1155](https://github.com/enjin/erc-1155)。从 v3.1 开始可用。

- 功能
	- [constructor(uri_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-constructor-string-)

		公共方法，见 [_setURI](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setURI-string-)。
	- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-supportsInterface-bytes4-)

		公共方法，见 [IERC165.supportsInterface](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)。
		
		- 返回值类型  bool
	- [uri(_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-uri-uint256-)


		公共方法，见[IERC1155MetadataURI.uri](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155MetadataURI-uri-uint256-)。
		
		此实现为所有Token类型返回相同的 URI。它依赖于 [EIP 中定义](https://eips.ethereum.org/EIPS/eip-1155#metadata)的Token类型 ID 替换机制。
		
		调用此函数的客户端必须将 `{id}` 子字符串替换为实际的Token类型 ID。
		
		- 返回值类型 string
	- [balanceOf(account, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOf-address-uint256-)

		公共方法，见[IERC1155.balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-balanceOf-address-uint256-)。
		
		- 要求
			- `account` 不能是零地址。
		- 返回值类型 uint256 
	- [balanceOfBatch(accounts, ids)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOfBatch-address---uint256---)

		公共方法，见[IERC1155.balanceOfBatch](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-balanceOfBatch-address---uint256---)。

		- 要求
			- `accounts` 和 `ids` 必须具有相同的长度。
		- 返回值类型 uint256 
	- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-setApprovalForAll-address-bool-)

		公共方法，见 [IERC1155.setApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-setApprovalForAll-address-bool-)
	- [isApprovedForAll(account, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-isApprovedForAll-address-address-)

		公共方法，见 [IERC1155.isApprovedForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-isApprovedForAll-address-address-)。
		
		- 返回值类型 bool
	- [safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)

		公共方法，见[IERC1155.safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)
	- [safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)

		公共方法，见[IERC1155.safeBatchTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)。
	- [_safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeTransferFrom-address-address-uint256-uint256-bytes-)

		内部方法，将  `amount` Token类型的Tokenid从 `from ` 转移到 `to`。

		发出[TransferSingle](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)事件。

		- 要求
			- `to` 不能是零地址。
			- `from` 必须有 `id` 至少 `amount `的Token余额。
			- 如果 `to` 指的是智能合约，它必须实现 [IERC1155Receiver.onERC1155Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-)并返回接受魔法值。
	- [_safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeBatchTransferFrom-address-address-uint256---uint256---bytes-)

		内部方法，批处理 ` _safeTransferFrom ` 版本。

		发出 [TransferBatch](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)事件。

		- 要求
			- 如果 `to` 指的是智能合约，它必须实现 [IERC1155Receiver.onERC1155BatchReceived](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155BatchReceived-address-address-uint256---uint256---bytes-)并返回接受魔法值。
	- [_setURI(newuri)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setURI-string-)

		通过依赖 [EIP 中定义](https://eips.ethereum.org/EIPS/eip-1155#metadata)的Token类型 ID 替换机制，为所有Token类型设置一个新的 URI 。

		通过这种机制，`{id}` URI 中出现的任何子字符串或所述 URI 处 JSON 文件中的任何数量都将被具有Token类型 ID 的客户端替换。

		例如，[https://token-cdn-domain/{id}.json](https://token-cdn-domain/%7Bid%7D.json) 客户端会将 URI 解释 [https://token-cdn-domain/000000000000000000000000000000000000000000000000000000000004cce0.json](https://token-cdn-domain/000000000000000000000000000000000000000000000000000000000004cce0.json) 为Token类型 ID 0x4cce0。

		见 [uri](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-uri-uint256-)。
		
		因为这些 URI 不能由 [uri](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-uri-uint256-) 事件有意义地表示，所以这个函数不会发出任何事件。
	- [_mint(to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)

		内部方法，创建 `amount`Token类型的Token `id`，并将它们分配给 `to`。

		- 要求
			- `to` 不能是零地址
			- 如果 `to` 指的是智能合约，它必须实现 [IERC1155Receiver.onERC1155Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-)并返回接受魔法值。

		发出 [TransferSingle](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-) 事件。
	- [_mintBatch(to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mintBatch-address-uint256---uint256---bytes-)

		内部方法，[批处理](https://docs.openzeppelin.com/contracts/4.x/erc1155#batch-operations) [_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)版本。

		发出 [TransferBatch](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)事件。

		- 要求
			- `ids` 和 `amounts` 必须具有相同的长度。
			- 如果 `to` 指的是智能合约，它必须实现[IERC1155Receiver.onERC1155BatchReceived](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155BatchReceived-address-address-uint256---uint256---bytes-)并返回接受魔法值。
	- [_burn(from, id, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burn-address-uint256-uint256-)

		内部方法，销毁 `from` 拥有的Tokenid和 `amount`  的Token

		发出 [TransferSingle](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-) 事件。

		- 要求
			- `from` 不能是零地址。
			- `from` 必须有 `amount` 个Token
	- [_burnBatch(from, ids, amounts)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burnBatch-address-uint256---uint256---)

		内部方法，[批处理](https://docs.openzeppelin.com/contracts/4.x/erc1155#batch-operations) [_burn](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burn-address-uint256-uint256-)版本。

		发出 [TransferBatch](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)事件。

		- 要求
			- `ids`和 ·amounts·必须具有相同的长度。
	- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setApprovalForAll-address-address-bool-)

		内部方法，批准 `operator` 对所有 `owner`Token进行操作

		发出一个 [ApprovalForAll](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)事件。
	- [_beforeTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)

		内部方法，在任何Token传输之前调用的钩子。这包括铸造和销毁，以及批量变体。

		在单个变体和批处理变体上调用相同的钩子。对于单次传输，`ids` 和 `amounts` 数组的长度将为 1。

		调用条件（每个 `id` 和 `amount`对）
		
		- 当 `from` 和 `to` 都非零时，`amount` 的Tokenid从 `from` 将被转移到 `to`。
		- 当 `from` 为零时，`amount` 的Tokenid将为 `to`.
		- 当 `to` 为零时，`from`  的 `amount` Token将被销毁。
		- `from` 和 `to` 永远都不为零。
		- `ids` 和 `amounts` 具有相同的非零长度。

		要了解有关钩子的更多信息，请前往[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)。
	- [_afterTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_afterTokenTransfer-address-address-address-uint256---uint256---bytes-)

		内部方法，在任何Token传输后调用的挂钩。这包括铸造和销毁，以及批量变体。

		在单个变体和批处理变体上调用相同的钩子。对于单次传输，`id` 和 `amount` 数组的长度将为 1。

		- 调用条件（每个 `id` 和 `amount` 对）
			- 当 `from` 和 `to` 都非零时，`amount` 的Tokenid从 `from` 将被转移到 `to`。
			- 当 `from` 为零时，`amount` 的Tokenid将为 `to`.
			- 当 `to` 为零时，`from`  的 `amount` Token将被销毁。
			- `from` 和 `to` 永远都不为零。
			- `ids` 和 `amounts` 具有相同的非零长度。	
		要了解有关钩子的更多信息，请前往[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)。
- 事件
	- IERC1155
		- [TransferSingle(operator, from, to, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)
		- [TransferBatch(operator, from, to, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)
		- [ApprovalForAll(account, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)
		- [URI(value, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)

### IERC1155Receiver [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/IERC1155Receiver.sol)
	import "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";
从 v3.1 开始可用。

- 功能
	- [onERC1155Received(operator, from, id, value, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-)

		外部方法，处理单个 ERC1155 Token的接收。 `safeTransferFrom` 此函数在余额更新后结束时调用。

		要接受传输，这必须返回 `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))` （即 0xf23a6e61，或它自己的函数选择器）。
		
		- 返回值类型 bytes 4
	- [onERC1155BatchReceived(operator, from, ids, values, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155BatchReceived-address-address-uint256---uint256---bytes-)

		外部方法，处理多个 ERC1155 Token的接收。`safeBatchTransferFrom` 此函数在余额更新后结束时调用。

		要接受传输，它必须返回 `bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))` （即 0xbc197c81，或它自己的函数选择器）。
	
	- IERC165 功能相同
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)

### ERC1155Receiver [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/utils/ERC1155Receiver.sol)
	import "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Receiver.sol";
从 v3.1 开始可用。

- 功能
	- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Receiver-supportsInterface-bytes4-)

		公开方法，见 [ERC165.supportsInterface](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)。
	- IERC1155接收器 相同功能
		- [onERC1155Received(operator, from, id, value, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-)
		- [onERC1155BatchReceived(operator, from, ids, values, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155Receiver-onERC1155BatchReceived-address-address-uint256---uint256---bytes-)

## 扩展
### ERC1155Pausable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/extensions/ERC1155Pausable.sol)
	import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Pausable.sol";
ERC1155 Token具有可暂停的Token转移、铸造和销毁。

对于在评估期结束之前阻止交易或在出现大错误时冻结所有Token转移的紧急开关等场景很有用。

从 v3.1 开始可用。

- 功能	
	- [_beforeTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Pausable-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)

		内部方法，见 [ERC1155._beforeTokenTransfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)。
		
		- 要求
			- 不得暂停合同。
	- 可暂停功能相同
		- [constructor()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-constructor--)
		- [paused()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-paused--)
		- [_requireNotPaused()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_requireNotPaused--)
		- [_requirePaused()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_requirePaused--)
		- [_pause()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_pause--)
		- [_unpause() ](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_unpause--)
	- ERC1155 功能相同
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-supportsInterface-bytes4-)
		- [uri(_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-uri-uint256-)
		- [balanceOf(account, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOf-address-uint256-)
		- [balanceOfBatch(accounts, ids)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOfBatch-address---uint256---)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-setApprovalForAll-address-bool-)
		- [isApprovedForAll(account, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-isApprovedForAll-address-address-)
		- [safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [_safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_setURI(newuri)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setURI-string-)
		- [_mint(to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)
		- [_mintBatch(to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mintBatch-address-uint256---uint256---bytes-)
		- [_burn(from, id, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burn-address-uint256-uint256-)
		- [_burnBatch(from, ids, amounts)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burnBatch-address-uint256---uint256---)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setApprovalForAll-address-address-bool-) 
		- [_afterTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_afterTokenTransfer-address-address-address-uint256---uint256---bytes-)
- 事件
	- 暂停功能相同
		- [Paused(account)](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-Paused-address-)
		- [Unpaused(account) ](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-Unpaused-address-)
	- IERC1155功能相同
		- [TransferSingle(operator, from, to, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)
		- [TransferBatch(operator, from, to, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)
		- [ApprovalForAll(account, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)
		- [URI(value, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)

### ERC1155 Burnable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/extensions/ERC1155Burnable.sol)
	import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";
对此的扩展 [ERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155) 允许Token持有者销毁他们自己的Token和他们已被批准使用的Token。

从 v3.1 开始可用。

- 功能
	- [burn(account, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Burnable-burn-address-uint256-uint256-)
	
		公开方法，单个销毁方法
	- [burnBatch(account, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Burnable-burnBatch-address-uint256---uint256---)

		公开方法，批量销毁方法
	- ERC1155 功能相同
		- [constructor(uri_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-constructor-string-) 
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-supportsInterface-bytes4-)
		- [uri(_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-uri-uint256-)
		- [balanceOf(account, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOf-address-uint256-)
		- [balanceOfBatch(accounts, ids)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOfBatch-address---uint256---)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-setApprovalForAll-address-bool-)
		- [isApprovedForAll(account, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-isApprovedForAll-address-address-)
		- [safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [_safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_setURI(newuri)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setURI-string-)
		- [_mint(to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)
		- [_mintBatch(to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mintBatch-address-uint256---uint256---bytes-)
		- [_burn(from, id, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burn-address-uint256-uint256-)
		- [_burnBatch(from, ids, amounts)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burnBatch-address-uint256---uint256---)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setApprovalForAll-address-address-bool-) 
		- [_beforeTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)
		- [_afterTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_afterTokenTransfer-address-address-address-uint256---uint256---bytes-)
- 事件
	- IERC1155功能相同
		- [TransferSingle(operator, from, to, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)
		- [TransferBatch(operator, from, to, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)
		- [ApprovalForAll(account, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)
		- [URI(value, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)
	
### ERC1155Supply [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/extensions/ERC1155Supply.sol)
	import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Supply.sol";
ERC1155 的扩展，增加了对每个 id 的总供应量的跟踪。

对于必须明确识别可替代和不可替代Token的场景很有用。注意：虽然 totalSupply 为 1 可能意味着对应的是 NFT，但不能保证不会铸造具有相同 id 的其他Token。

- 功能
	- [totalSupply(id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Supply-totalSupply-uint256-)

		公共方法，具有给定 id 的Token总数。
		
		- 返回值类型 uint256
	- [exists(id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Supply-exists-uint256-)

		公共方法，指示是否存在具有给定 id 的任何Token。
		
		- 返回值类型 bool
	- [_beforeTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Supply-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)

		内部方法，见 [ERC1155._beforeTokenTransfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)。
	- ERC1155 功能相同
		- [constructor(uri_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-constructor-string-) 
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-supportsInterface-bytes4-)
		- [uri(_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-uri-uint256-)
		- [balanceOf(account, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOf-address-uint256-)
		- [balanceOfBatch(accounts, ids)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOfBatch-address---uint256---)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-setApprovalForAll-address-bool-)
		- [isApprovedForAll(account, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-isApprovedForAll-address-address-)
		- [safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [_safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_setURI(newuri)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setURI-string-)
		- [_mint(to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)
		- [_mintBatch(to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mintBatch-address-uint256---uint256---bytes-)
		- [_burn(from, id, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burn-address-uint256-uint256-)
		- [_burnBatch(from, ids, amounts)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burnBatch-address-uint256---uint256---)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setApprovalForAll-address-address-bool-) 
		- [_afterTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_afterTokenTransfer-address-address-address-uint256---uint256---bytes-)
- 事件
	- IERC1155功能相同
		- [TransferSingle(operator, from, to, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)
		- [TransferBatch(operator, from, to, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)
		- [ApprovalForAll(account, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)
		- [URI(value, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)
	
### ERC1155URIStorage [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/extensions/ERC1155URIStorage.sol)
	import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155URIStorage.sol";
具有基于存储的Token URI 管理的 ERC1155 Token。受 ERC721URIStorage 扩展的启发

从 v4.6 开始可用。

- 功能
	- [uri(tokenId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155URIStorage-uri-uint256-)

		公共方法，见 [IERC1155MetadataURI.uri](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155MetadataURI-uri-uint256-)。

		如果设置了后者，则此实现返回 `the_baseURI` 和特定于Token的 uri的串联

		这会启用以下行为
		
		- 如果 `_tokenURIs[tokenId]` 设置，则结果是和 `_baseURI` 的连接（请记住 `_tokenURIs[tokenId]`，`_baseURI` 默认情况下为空）；
		- 如果 `_tokenURIs[tokenId]` 未设置，那么我们回退到 `super.uri()` 在大多数情况下将包含 `ERC1155._uri`;
		- 如果 `_tokenURIs[tokenId]` 未设置，并且如果父合约没有设置 `uri` 值，则结果为空。

		返回值类型 string
	- [_setURI(tokenId, tokenURI)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155URIStorage-_setURI-uint256-string-)

		内部方法，设置 `tokenURI` 为 tokenId 的 tokenURI 。
	- [_setBaseURI(baseURI)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155URIStorage-_setBaseURI-string-)

		内部方法，设置 `baseURI` 为 `_baseURI` 所有Token
	- ERC1155 功能相同
		- [constructor(uri_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-constructor-string-) 
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-supportsInterface-bytes4-)
		- [balanceOf(account, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOf-address-uint256-)
		- [balanceOfBatch(accounts, ids)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-balanceOfBatch-address---uint256---)
		- [setApprovalForAll(operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-setApprovalForAll-address-bool-)
		- [isApprovedForAll(account, operator)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-isApprovedForAll-address-address-)
		- [safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_safeTransferFrom(from, to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeTransferFrom-address-address-uint256-uint256-bytes-)
		- [_safeBatchTransferFrom(from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_safeBatchTransferFrom-address-address-uint256---uint256---bytes-)
		- [_setURI(newuri)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setURI-string-)
		- [_mint(to, id, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)
		- [_mintBatch(to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_mintBatch-address-uint256---uint256---bytes-)
		- [_burn(from, id, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burn-address-uint256-uint256-)
		- [_burnBatch(from, ids, amounts)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_burnBatch-address-uint256---uint256---)
		- [_setApprovalForAll(owner, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_setApprovalForAll-address-address-bool-) 
		- [_beforeTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)
		- [_afterTokenTransfer(operator, from, to, ids, amounts, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155-_afterTokenTransfer-address-address-address-uint256---uint256---bytes-)
- 事件
	- IERC1155功能相同
		- [TransferSingle(operator, from, to, id, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferSingle-address-address-address-uint256-uint256-)
		- [TransferBatch(operator, from, to, ids, values)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-TransferBatch-address-address-address-uint256---uint256---)
		- [ApprovalForAll(account, operator, approved)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-ApprovalForAll-address-address-bool-)
		- [URI(value, id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#IERC1155-URI-string-uint256-)
	
## 预设
这些合约是上述功能的预配置组合。它们可以通过继承或作为模型来复制和粘贴其源代码。
## 实用程序
### ERC1155Holder [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC1155/utils/ERC1155Holder.sol)
	import "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";
从 v3.1 开始可用。

- 功能
	- [onERC1155Received(_, _, _, _, _)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Holder-onERC1155Received-address-address-uint256-uint256-bytes-)

		公共方法
		
		- 返回值类型 bytes4
	- [onERC1155BatchReceived(_, _, _, _, _)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Holder-onERC1155BatchReceived-address-address-uint256---uint256---bytes-)

		公共方法
		
		- 返回值类型 bytes4
	- ERC 1155 接收器功能相同
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155#ERC1155Receiver-supportsInterface-bytes4-) 