# openzepplin 文档 4.x
## 合约
用于安全智能合约开发的库。建立在经过社区审查的代码的坚实基础之上。

- [ERC20](https://docs.openzeppelin.com/contracts/4.x/erc20) 和 [ERC721](https://docs.openzeppelin.com/contracts/4.x/erc721)等标准的实施。
- 灵活的基于[角色的许可方案](https://docs.openzeppelin.com/contracts/4.x/access-control)。
- 可重用的 Solidity [组件](https://docs.openzeppelin.com/contracts/4.x/utilities)，用于构建自定义合约和复杂的去中心化系统。

## 概述
### 安装
	$ npm install @openzeppelin/contracts
OpenZeppelin Contracts 具有[稳定的 API](https://docs.openzeppelin.com/contracts/4.x/releases-stability#api-stability)，这意味着您的合约在升级到较新的次要版本时不会意外中断。
### 用法
安装后，您可以通过导入库中的合约来使用它们：

	// contracts/MyNFT.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;

	import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
	
	contract MyNFT is ERC721 {
	    constructor() ERC721("MyNFT", "MNFT") {
	    }
	}
如果您不熟悉智能合约开发，请前往[开发智能合约](https://docs.openzeppelin.com/learn/developing-smart-contracts)以了解有关创建新项目和编译合约的信息。

为了保证您的系统安全，您应该始终按原样使用已安装的代码，既不要从在线资源复制粘贴，也不要自行修改。该库旨在仅部署您使用的合约和功能，因此您无需担心它会不必要地增加 gas 成本。
### 安全
请通过我们在 Immunefi 上的错误 [赏金计划](https://www.immunefi.com/bounty/openzeppelin)或直接向 security@openzeppelin.org 报告您发现的任何安全问题。

### 学到更多
侧栏中的指南将教授不同的概念，以及如何使用 OpenZeppelin Contracts 提供的相关合约：

- [访问控制](https://docs.openzeppelin.com/contracts/4.x/access-control)

	决定谁可以在您的系统上执行什么操作
- [Token](https://docs.openzeppelin.com/contracts/4.x/tokens)

	创建可交易资产或收藏品，如众所周知的 [ERC20](https://docs.openzeppelin.com/contracts/4.x/erc20) 和 [ERC721](https://docs.openzeppelin.com/contracts/4.x/erc721) 标准。
- [实用程序](https://docs.openzeppelin.com/contracts/4.x/utilities)

	通用有用的工具，包括非溢出数学、签名验证和去信任支付系统。

完整的 [API](https://docs.openzeppelin.com/contracts/4.x/api)也有完整的文档记录，在开发智能合约应用程序时可以作为很好的参考。您也可以在[社区论坛](https://forum.openzeppelin.com/)寻求帮助或关注 Contracts 的发展。

最后，您可能想查看我们[博客](https://blog.openzeppelin.com/guides/)上的指南，其中涵盖了几个常见用例和良好实践。以下文章提供了很好的背景阅读，但请注意，随着生态系统中的工具继续快速发展，一些引用的工具已经发生了变化。

- [以太坊智能合约搭便车指南](https://blog.openzeppelin.com/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05)

	将帮助您了解可用于智能合约开发的各种工具，并帮助您设置环境。
- [以太坊编程的简要介绍](https://blog.openzeppelin.com/a-gentle-introduction-to-ethereum-programming-part-1-783cc7796094)

	第 1 部分提供了非常有用的介绍性信息，包括来自以太坊平台的许多基本概念。
- 如需更深入的了解，您可以阅读为您的[以太坊应用程序设计架构指南](https://blog.openzeppelin.com/designing-the-architecture-for-your-ethereum-application-9cec086f8317)，其中讨论了如何更好地构建您的应用程序及其与现实世界的关系。

## 扩展合约
大多数 OpenZeppelin 合约预计将通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)使用：您将在编写自己的合约时从它们继承。

这是常见的 `is` 语法，例如  `contract MyToken is ERC20`。

与 `contracts` 不同，Solidity `librarys` 不是继承自语法而是依赖于 [using for](https://solidity.readthedocs.io/en/latest/contracts.html#using-for) 语法。

OpenZeppelin 合约有一些 `library`: 大多数都在 [Utils](https://docs.openzeppelin.com/contracts/4.x/api/utils) 目录中。
### 覆盖
继承通常用于将父合约的功能添加到您自己的合约中，但这并不是它所能做的全部。您还可以使用 `overrides` 更改父级的某些部分的行为方式。

例如，假设您想要更改 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl)，以便 [revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-) 不再调用。这可以使用覆盖来实现：

	// contracts/ModifiedAccessControl.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/access/AccessControl.sol";
	
	contract ModifiedAccessControl is AccessControl {
	    // 覆盖 revokeRole 方法
	    function revokeRole(bytes32, address) public override {
	        revert("ModifiedAccessControl: cannot revoke roles");
	    }
	}

然后旧 `revokeRole` 的被我们的覆盖替换，任何对它的调用都将立即恢复。我们无法从合约中删除该函数，但恢复所有调用就足够了。

### 调用 `super`
有时您想扩展一个父合约的行为，而不是直接将其更改为其他行为。这就是 `super` 来负责的。

`super` 关键字将允许您调用父合约中定义的函数，即使它们被覆盖。此机制可用于向函数添加额外的检查、发出事件或以其他方式添加您认为合适的功能。

有关覆盖如何工作的更多信息，请访问 [Solidity 官方文档](https://solidity.readthedocs.io/en/latest/contracts.html#index-17)。

以下是 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl)的[revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-) 方法不能用于撤销的修改版本 `DEFAULT_ADMIN_ROLE`

	// contracts/ModifiedAccessControl.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/access/AccessControl.sol";
	
	contract ModifiedAccessControl is AccessControl {
	    function revokeRole(bytes32 role, address account) public override {
	        require(
	            role != DEFAULT_ADMIN_ROLE,
	            "ModifiedAccessControl: cannot revoke default admin role"
	        );
	
	        super.revokeRole(role, account);
	    }
	}
最后的 `super.revokeRole` 语句将调用 `AccessControl` 的 `revokeRole` 原始版本，如果没有覆盖，将运行相同的代码。


从 v3.0.0 开始，`view` 函数不在 `virtual` OpenZeppelin 中，因此不能被覆盖。我们正在考虑在即将发布的版本中[取消此限制](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2154)。让我们知道这是否是您关心的事情！	
### 使用钩子
有时，为了扩展父合约，您需要覆盖多个相关函数，这会导致代码重复并增加错误的可能性。

例如，考虑 [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20) 以 [IERC721Receiver](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#IERC721Receiver). 您可能认为覆盖[transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-transfer-address-uint256-)和 [transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-transferFrom-address-address-uint256) 就足够了，但是 [_transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_transfer-address-address-uint256-)和[_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_mint-address-uint256-)呢？为了避免你不得不处理这些细节，我们引入了 `hooks`。

`hooks` 只是在某些操作发生之前或之后调用的函数。它们提供了一个集中点来挂钩和扩展原始行为。

以下是在`ERC20` 中使用 [_beforeTokenTransfer](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_beforeTokenTransfer-address-address-uint256-) 钩子实现  `IERC721Receiver` 模式的方法：

	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
	
	contract ERC20WithSafeTransfer is ERC20 {
	    function _beforeTokenTransfer(address from, address to, uint256 amount)
	        internal virtual override
	    {
	        super._beforeTokenTransfer(from, to, amount);
	
	        require(_validRecipient(to), "ERC20WithSafeTransfer: invalid recipient");
	    }
	
	    function _validRecipient(address to) private view returns (bool) {
	        ...
	    }
	
	    ...
	}
以这种方式使用钩子可以使代码更干净、更安全，而不必依赖对父级内部的深入了解。

Hooks 是 OpenZeppelin Contracts v3.0.0 的新功能，我们渴望了解您打算如何使用它们！到目前为止，唯一可用的钩子是 `_beforeTransferHook`, 在所有 [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_beforeTokenTransfer-address-address-uint256-),[ERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#ERC721-_beforeTokenTransfer-address-address-uint256-)，[ERC777](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#ERC777-_beforeTokenTransfer-address-address-address-uint256-)和[ERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#ERC1155-_beforeTokenTransfer-address-address-address-uint256---uint256---bytes-)。

### 钩子规则
在编写使用钩子的代码以防止出现问题时，您应该遵循一些准则。它们非常简单，但请确保您遵循它们：

- 每当您  `virtual` 覆盖父挂钩时，将属性重新应用于挂钩。这将允许子合约向钩子添加更多功能。
- 始终使用 `super`. 这将确保调用继承树中的所有钩子：像这样的合约 [ERC20Pausable](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20Pausable) 依赖于这种行为。

		contract MyToken is ERC20 {
		    function _beforeTokenTransfer(address from, address to, uint256 amount)
		        internal virtual override // 在这里加入 virtual 
		    {
		        super._beforeTokenTransfer(from, to, amount); // 调用父合约钩子
		        ...
		    }
		}

## 使用升级
如果您的合约要部署可升级，例如使用 [OpenZeppelin 升级插件](https://docs.openzeppelin.com/upgrades-plugins/1.x/)，您将需要使用 OpenZeppelin 合约的可升级变体。

此变体可作为名为的单独包提供，该包 `@openzeppelin/contracts-upgradeable` 托管在存储库 [OpenZeppelin/openzeppelin-contracts-upgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable) 中。

它遵循编写[可升级合约](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable)的所有规则：构造函数被初始化函数替换，状态变量在初始化函数中初始化，并且我们还检查次要版本之间的存储不兼容性。

OpenZeppelin 提供了一整套用于部署和保护可升级智能合约的工具。查看[完整的资源列表](https://docs.openzeppelin.com/contracts/4.x/upgradeable#openzeppelin::upgrades.adoc)。
### 概述
#### 安装
	npm install @openzeppelin/contracts-upgradeable
#### 用法
该包复制了主 OpenZeppelin Contracts 包的结构，但每个文件和合约都有后缀 `Upgradeable`

	-import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
	+import "@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol";
	
	-contract MyCollectible is ERC721 {
	+contract MyCollectible is ERC721Upgradeable {
构造函数由遵循命名约定的内部初始化函数替换 `_ _{ContractName}_init`。由于这些是内部的，因此您必须始终定义自己的公共初始化器函数并调用您扩展的合约的父初始化器。

	-    constructor() ERC721("MyCollectible", "MCO") public {
	+    function initialize() initializer public {
	+        __ERC721_init("MyCollectible", "MCO");
	     }		
与多重继承一起使用需要特别注意。请参阅下面标题为“ [多重继承](https://docs.openzeppelin.com/contracts/4.x/upgradeable#multiple-inheritance)” 的部分。

设置并编译此合约后，您可以使用 [Upgrades Plugins](https://docs.openzeppelin.com/upgrades-plugins/1.x/) 部署它。以下片段显示了使用 Hardhat 的示例部署脚本。

	// scripts/deploy-my-collectible.js
	const { ethers, upgrades } = require("hardhat");
	
	async function main() {
	  const MyCollectible = await ethers.getContractFactory("MyCollectible");
	
	  const mc = await upgrades.deployProxy(MyCollectible);
	
	  await mc.deployed();
	  console.log("MyCollectible deployed to:", mc.address);
	}
	
	main();
### 进一步说明
#### 多重继承
初始化函数不像构造函数那样被编译器线性化。因此，每个 `_ _{ContractName}_init` 函数都将线性化调用嵌入到所有父初始值设定项。因此，调用其中两个 `init` 函数可能会两次初始化同一个合约。

每个合约中的函数 `_ _{ContractName}_init_unchained` 都是初始化函数减去对父初始化函数的调用，可以用来避免双重初始化问题，但不建议手动这样做。我们希望能够在升级插件的未来版本中对此进行安全检查。
#### 存储间隙
您可能会注意到每个合约都包含一个名为 `_ _gap` 的状态变量。这是在可升级合约中放置的存储中的空保留空间。它允许我们在未来自由添加新的状态变量，而不会影响与现有部署的存储兼容性。

简单地添加一个状态变量是不安全的，因为它会“向下移动”继承链中下面的所有状态变量。这使得存储布局不兼容，如[编写可升级合约](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#modifying-your-contracts)中所述。计算数组的大小，`__gap` 以便合约使用的存储量加起来总是相同的数字（在本例中为 50 个存储槽）。

## 访问控制
访问控制——即“谁被允许做这件事”——在智能合约的世界中非常重要。你的合约的访问控制可以控制谁可以铸造token、对提案进行投票、冻结转账和许多其他事情。因此，了解您如何实现它至关重要，以免其他人[窃取](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)您的整个系统。
### 所有权和  `Ownable`
最常见和最基本的访问控制形式是所有权的概念：有一个账户是 `owner` 合约的，可以在上面执行管理任务。这种方法对于只有一个管理用户的合约来说是完全合理的。

OpenZeppelin Contracts 提供 [Ownable](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable) 在您的合约中实现所有权。

	// contracts/MyContract.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/access/Ownable.sol";
	
	contract MyContract is Ownable {
	    function normalThing() public {
	        // 任何人都可以调用 normalThing()
	    }
	
	    function specialThing() public onlyOwner {
	        // 只有 owner 才能调用 specialThing()!
	    }
	}
默认情况下，`Ownable` 合约的所有者是部署它的帐户是 [owner](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-owner--)  。

`Ownable` 还可以：

- [transferOwnership](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-transferOwnership-address-) 从所有者帐户到新帐户，以及
- [renounceOwnership](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-renounceOwnership--) 对于所有者来说，放弃这种管理特权，在最初的集中管理阶段之后的常见模式已经结束。

完全删除所有者将意味着受保护的管理任务 `onlyOwner` 将不可再调用！

请注意，`一个合约也可以是另一个合约的所有者！` 例如，这为使用 [Gnosis Multisig](https://github.com/gnosis/MultiSigWallet) 或 [Gnosis Safe](https://safe.gnosis.io/) 、[Aragon DAO](https://aragon.org/)、[ERC725/uPort](https://www.uport.me/) 身份合约或您创建的完全自定义合约。

通过这种方式，您可以使用可组合性为您的合约添加额外的访问控制复杂性层。例如，您可以使用由项目负责人运行的 2-of-3 多重签名，而不是将单个常规以太坊账户（外部拥有的账户，或 EOA）作为所有者。该领域的知名项目，例如[MakerDAO](https://makerdao.com/)，使用与此类似的系统。

### 基于角色的访问控制
虽然所有权的简单性对于简单的系统或快速原型设计很有用，但通常需要不同级别的授权。您可能希望管理员有权禁止某些用户进入系统或不能创建新 token。基于角[色的访问控制](https://en.wikipedia.org/wiki/Role-based_access_control) (RBAC) 在这方面提供了灵活性。

本质上，我们将定义多个角色，每个角色都允许执行不同的操作集。例如，一个帐户可能具有“主持人”、“铸币工”或“管理员”角色，然后您将检查这些角色，而不是简单地使用 `onlyOwner`. 可以通过 `onlyRole` 修饰符强制执行此检查。另外，您将能够定义如何向帐户授予角色、撤销角色等规则。

大多数软件使用基于角色的访问控制系统：

- 一些用户是普通用户
- 一些可能是管理员或经理
- 还有一些通常具有管理权限

### 使用 `AccessControl`
OpenZeppelin Contracts 提供 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) 了实现基于角色的访问控制。它的用法很简单：对于您要定义的每个角色，您将创建一个新的角色标识符，用于授予、撤销和检查帐户是否具有该角色。

这是一个使用 token 定义“铸币者”角色的简单示例，`AccessControl` 角色允许拥有它的帐户创建新 [ERC20](https://docs.openzeppelin.com/contracts/4.x/tokens#ERC20) Token：

	// contracts/MyToken.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/access/AccessControl.sol";
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
	
	contract MyToken is ERC20, AccessControl {
	    // 为铸币角色创建一个新的角色标识
	    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
	
	    constructor(address minter) ERC20("MyToken", "TKN") {
	        // 将铸币角色授权指定账户
	        _setupRole(MINTER_ROLE, minter);
	    }
	
	    function mint(address to, uint256 amount) public {
	        // 检查调用账户是否是铸币角色
	        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
	        _mint(to, amount);
	    }
	}

[AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) 在您的系统上使用它或复制粘贴本指南中的示例之前， 请确保您完全了解它的工作原理。

虽然清晰明确，但这并不是我们无法通过 `Ownable`. 事实上，在 `AccessControl` 需要细粒度权限的场景中表现出色，可以通过定义多个角色来实现。

让我们通过定义一个 “burner” 角色来扩充我们的 ERC20 token示例，该角色允许帐户销毁token，并使用 `onlyRole` 修饰符

	// contracts/MyToken.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/access/AccessControl.sol";
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
	
	contract MyToken is ERC20, AccessControl {
	    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
	    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
	
	    constructor(address minter, address burner) ERC20("MyToken", "TKN") {
	        _setupRole(MINTER_ROLE, minter);
	        _setupRole(BURNER_ROLE, burner);
	    }
	
	    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
	        _mint(to, amount);
	    }
	
	    function burn(address from, uint256 amount) public onlyRole(BURNER_ROLE) {
	        _burn(from, amount);
	    }
	}
好干净！通过以这种方式拆分关注点，可以实现比使用更简单的所有权方法更细粒度的权限进行访问控制。限制系统的每个组件能够执行的操作称为[最小权限原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，是一种良好的安全实践。请注意，如果需要，每个帐户可能仍具有多个角色	

### 授予和撤销角色
上面的 ERC20 token示例使用 `_setupRole`，`internal` 函数在以编程方式分配角色（例如在构造期间）时很有用。但是，如果我们稍后想将“铸币者”角色授予其他帐户怎么办？

默认情况下，具有角色的帐户无法从其他帐户授予或撤销它：拥有角色所做的只是使 `hasRole` 检查通过。要动态授予和撤销角色，您需要角色管理员的帮助。

每个角色都有一个关联的角色管理员，该角色授予调用 `grantRole` 和 `revokeRole` 函数的权限。如果调用帐户具有相应的管理员角色，则可以使用这些来授予或撤销角色。多个角色可能具有相同的管理员角色，以便于管理。一个角色的管理员甚至可以是同一个角色本身，这将导致具有该角色的帐户也能够授予和撤销它。

这种机制可用于创建类似于组织结构图的复杂许可结构，但它也提供了一种管理简单应用程序的简单方法。AccessControl 包括一个特殊角色，称为 `DEFAULT_ADMIN_ROLE`，它充当所有角色的`默认管理员角色`。具有此角色的帐户将能够管理任何其他角色，除非`_setRoleAdmin` 用于选择新的管理员角色。

让我们看一下 ERC20 token示例，这次利用了默认管理员角色：

	// contracts/MyToken.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/access/AccessControl.sol";
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
	
	contract MyToken is ERC20, AccessControl {
	    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
	    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
	
	    constructor() ERC20("MyToken", "TKN") {
	        // 授予合约部署者默认的管理角色:它将能够授予和撤销任何角色
	        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
	    }
	
	    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
	        _mint(to, amount);
	    }
	
	    function burn(address from, uint256 amount) public onlyRole(BURNER_ROLE) {
	        _burn(from, amount);
	    }
	}
请注意，与前面的示例不同，没有帐户被授予“minter”或“burner”角色。但是，由于这些角色的管理员角色是默认管理员角色，并且该角色被授予 `msg.sender`，同一帐户可以调用 `grantRole` 以授予铸造或燃烧权限，并可以使用 `revokeRole` 删除它。

动态角色分配通常是理想的属性，例如在对参与者的信任可能随时间变化的系统中。它还可以用于支持诸如 [KYC](https://en.wikipedia.org/wiki/Know_your_customer) 之类的用例，其中角色承担者的列表可能事先不知道或者包含在单个事务中可能非常昂贵。
### 查询特权账户
由于帐户可能会动态[授予和撤销](https://docs.openzeppelin.com/contracts/4.x/access-control#granting-and-revoking)角色，因此并不总是可以确定哪些帐户拥有特定角色。这很重要，因为它可以证明系统的某些属性，例如管理帐户是多重签名或 DAO，或者某个角色已从所有用户中删除，从而有效地禁用任何相关功能。

在引擎下，`AccessControl` 使用 `EnumerableSet`  的 Solidity `mapping` 类型的更强大的变体，它允许密钥枚举。`getRoleMemberCount` 可用于检索具有特定角色的帐户数量，`getRoleMember` 然后可以调用以获取每个帐户的地址。

	const minterCount = await myToken.getRoleMemberCount(MINTER_ROLE);
	
	const members = [];
	for (let i = 0; i < minterCount; ++i) {
	    members.push(await myToken.getRoleMember(MINTER_ROLE, i));
	}
### 延迟操作
访问控制对于防止未经授权访问关键功能至关重要。这些功能可用于铸造token、冻结转账或执行完全改变智能合约逻辑的升级。虽然 [Ownable](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable)和 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl)可以防止未经授权的访问，但它们并没有解决行为不端的管理员攻击他们自己的系统以损害其用户的问题。

这是 [TimelockController](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController) 正在解决的问题。

TimelockController 是由提议者和执行者管理的代理。当设置为智能合约的所有者/管理员/控制者时，它确保提议者通过的调用在任何维护操作都会受到延迟。这种延迟保护了智能合约的用户，让他们有时间审查维护操作并在他们认为这样做符合他们的最大利益时退出系统。
#### 使用 TimelockController
默认情况下，部署地址的地址 [TimelockController](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController) 在时间锁上获得管理权限。此角色授予分配提议者、执行者和其他管理员的权利。

配置的第一步 TimelockController 是分配至少一个提议者和一个执行者。这些可以在构建期间或以后由具有管理员角色的任何人分配。这些角色不是独占的，这意味着一个帐户可以同时拥有这两个角色。

使用 AccessControl 接口管理角色 `bytes32`，每个角色的值都可以通过 `ADMIN_ROLE,PROPOSER_ROLE` 和 `EXECUTOR_ROLE` 常量访问。

在此之上还有一个附加功能：一旦时间锁定到期，执行 AccessControl 者角色可以向任何人开放执行提案的权限。`address(0)` 此功能虽然有用，但应谨慎使用。

此时，同时分配了提议者和执行者，时间锁就可以执行操作了。

可选的下一步是部署者放弃其管理权限并让时间锁自行管理。如果部署者决定这样做，所有进一步的维护，包括分配新的 `提议者/调度者` 或更改 时间锁 持续时间都必须遵循时间锁工作流程。这将时间锁的治理与附加到时间锁的合约的治理联系起来，并强制延迟时间锁维护操作。

	如果部署者放弃管理权限以支持 timelock 本身，则分配新的提议者或执行者将需要 timelocked 操作。这意味着，如果负责这两个角色中的任何一个的账户不可用，那么整个合约（以及它控制的任何合约）都会被无限期锁定。
通过分配提议者和执行者角色以及负责其自身管理的时间锁，您现在可以将任何合约的所有权/控制权转移到时间锁。

推荐的配置是将这两个角色授予安全治理合约（例如 DAO 或多重签名），并另外将执行者角色授予负责帮助维护操作的人员持有的一些 EOA。这些钱包无法接管时间锁，但它们可以帮助简化工作流程。
#### 最小延迟
执行的操作 TimelockController 不受固定延迟的影响，而是最小延迟。一些主要更新可能需要更长的延迟。例如，如果仅仅几天的延迟可能足以让用户审核铸币操作，那么在安排智能合约升级时使用几周甚至几个月的延迟是有意义的。

- 最小延迟（可通过 [getMinDelay](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-getMinDelay--) 方法访问
- 可以通过调用 [updateDelay](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-updateDelay-uint256-) 函数来更新。

请记住，只能通过时间锁本身访问此功能，这意味着此维护操作必须通过时间锁本身。

## token
啊，“token”：区块链最强大也是最容易被误解的工具。

token 是区块链中某物的表示。这可以是金钱、时间、服务、公司股份、虚拟宠物，任何东西。通过将事物表示为 token，我们可以允许智能合约与它们进行交互、交换、创建或销毁它们。
### token 合约与实际 token
围绕 token 的大部分混淆来自于两个概念的混淆：token合约 和 实际 token。

- token合约

	只是一个以太坊智能合约。“发送 token” 实际上意味着 “调用某人编写和部署的智能合约上的方法”。归根结底，token 合约只不过是地址到余额的映射，加上一些从这些余额中加减的方法。正是这些余额代表了 token 本身。当 token 合约中的余额不为零时，某人代表 “拥有token”。而已！这些余额可以被视为金钱、游戏中的经验值、所有权契约或投票权，并且这些 token 中的每一个都将存储在不同的 token 合约中。
- 不同种类的token

	请注意，拥有两个投票权和两个所有权契约之间有很大的区别：
	
	- 每个投票都等于所有其他投票，但房屋通常不是！这称为可替代性。
		- 可替代商品是等价且可互换的，例如以太币、法定货币和投票权。
		- 不可替代的商品是独一无二的，就像所有权契约或收藏品一样。

简而言之，

- 在处理不可替代的资产（例如您的房子）时，您关心的是您拥有哪些资产
- 而在可替代资产（例如您的银行账户对账单）中，重要的是您拥有多少

### 标准
尽管令牌的概念很简单，但它们在实现中具有各种复杂性。因为以太坊中的一切都只是一个智能合约，并且没有关于智能合约必须做什么的规则，社区已经开发了各种标准（称为 EIP 或 ERC）来记录合约如何与其他合约互操作。您可能听说过 ERC20 或 ERC721 token标准，这就是您来这里的原因。前往我们的专业指南了解更多信息：

- [ERC20](https://docs.openzeppelin.com/contracts/4.x/erc20)

	可替代资产最广泛使用的 token 标准，尽管受到其简单性的限制。
- [ERC721](https://docs.openzeppelin.com/contracts/4.x/erc721)

	不可替代token的实际解决方案，通常用于收藏品和游戏。
- [ERC777](https://docs.openzeppelin.com/contracts/4.x/erc777)

	更丰富的可替代 token 标准，支持新的用例并建立在过去的学习基础上。向后兼容 ERC20。
- [ERC1155](https://docs.openzeppelin.com/contracts/4.x/erc1155)

	一种新的多 token 标准，允许单个合约代表多个可替代和不可替代的 token，以及批量操作以提高 GAS 效率。
	
### ERC20
ERC20 token合约跟踪 [可替代token](https://docs.openzeppelin.com/contracts/4.x/tokens#different-kinds-of-tokens) 任何一个token都完全等同于任何其他token；没有任何token具有与之相关的特殊权利或行为。这使得 ERC20 token可用于 `交换货币、投票权、质押等媒介`。

OpenZeppelin Contracts 提供了许多与 ERC20 相关的合约。[API reference](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20) 您会在上面找到有关其属性和用法的详细信息。
#### 构建 ERC20 token合约
使用合约我们可以轻松创建自己的 ERC20 token合约，用于追踪假设游戏中的内部货币黄金(GLD)。这是我们的 GLD token的外观。
	
	// contracts/GLDToken.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
	
	contract GLDToken is ERC20 {
	    constructor(uint256 initialSupply) ERC20("Gold", "GLD") {
	        _mint(msg.sender, initialSupply);
	    }
	}
我们的合约经常通过 [继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance) 使用，在这里我们将重用 [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#erc20) 基本标准实现以及 

-  [name](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-name--)
- [symbol](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-symbol--)
- [decimals](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-decimals--)

可选扩展。此外，我们正在创建一个 `initialSupply`Token，它将分配给部署合约的地址。

有关 ERC20 供应机制的更完整讨论，请参阅[创建 ERC20 供应](https://docs.openzeppelin.com/contracts/4.x/erc20-supply)。

而已！部署后，我们将能够查询部署者的余额：

	> GLDToken.balanceOf(deployerAddress)
	1000000000000000000000
也可以将这些[token转移](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#IERC20-transfer-address-uint256-)到其他账户：

	> GLDToken.transfer(otherAddress, 300000000000000000000)
	> GLDToken.balanceOf(otherAddress)
	300000000000000000000
	> GLDToken.balanceOf(deployerAddress)
	700000000000000000000

#### 关于decimals
通常，您希望能够将您的token分成任意数量：例如，如果您拥有 5 GLD，您可能想发送 1.5 GLD 给朋友，并留给 3.5 GLD 自己。不幸的是，Solidity 和 EVM 不支持这种行为：只能使用整数（整数），这是一个问题。您可以发送 1 或发送 2Token，但不能发送 1.5.

为了解决这个问题，ERC20 提供了一个 [decimals](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-decimals--) 字段，用于指定令牌有多少小数位。为了能够转移 1.5 GLD，`decimals` 必须至少1，因为该数字有一个小数位。

如何做到这一点？其实很简单：一个 token 合约可以使用更大的整数值，这样一个 50 代表一个余额 5 GLD，一个15 对应一个1.5 GLD被发送，等等。

重要的是要了解它 `decimals` 仅用于显示目的。合约内部的所有算术仍然在整数上执行，不同的用户界面（钱包、交易所等）必须根据 `decimals`. 每个账户的总token供应量和余额未在中指定 GLD：您需要除以 `10 ** decimals` 得到实际 GLD 数量。

你可能想要使用的 decimals 值18，就像 Ether 和大多数正在使用的 ERC20 token合约一样，除非你有非常特殊的理由不这样做。铸造token或转移token时，您实际上将发送数字 `num GLD * (10 ** decimals)`。


默认情况下，ERC20 使用值 `18 decimals`。要使用不同的值，您需要覆盖合约中 `decimals()` 的函数。

	function decimals() public view virtual override returns (uint8) {
	  return 16;
	}
如果您想 5 使用 18 位小数的token合约发送token，调用的方法实际上是

	transfer(recipient, 5 * (10 ** 18));
#### 预设 ERC20 合约
预设 ERC20 可用，[ERC20PresetMinterPauser](https://docs.openzeppelin.com/contracts/4.x/erc20#api:presets.adoc#ERC20PresetMinterPauser). 它预设为允许

- token铸造（创建）
- 停止所有token转移（暂停）
- 持有者销毁（销毁）他们的token

该合约使用[访问控制](https://docs.openzeppelin.com/contracts/4.x/access-control)来控制对铸币和暂停功能的访问。部署合约的账户将被授予 ` minter `和 `pauser` 角色以及默认的 admin 角色。

合约无需编写任何 Solidity 代码即可部署。它可以按原样用于快速原型设计和测试，但也适用于生产环境。

#### 创建 ERC20 供应
在本指南中，您将学习如何使用自定义供应机制创建 ERC20 token。为此，我们将展示两种使用 OpenZeppelin 合约的惯用方式，您将能够将它们应用到您的智能合约开发实践中。

由基于以太坊的token实现的标准接口称为 ERC20，Contracts 包含一个广泛使用的实现：恰当命名的 ERC20 合约。与标准本身一样，该合约非常简单和准系统。实际上，如果您尝试按 ERC20 原样部署一个实例，它将毫无用处……它将没有供应！

ERC20 文档中没有定义供应的创建方式。每个token都可以自由地尝试自己的机制，从最分散的到最集中的，从最幼稚的到研究最多的等等。

- 固定供应

	假设我们想要一个固定供应量为 1000 的token，最初分配给部署合约的账户。如果您使用过 Contracts v1，您可能编写过如下代码
		
		contract ERC20FixedSupply is ERC20 {
		    constructor() {
		        totalSupply += 1000;
		        balances[msg.sender] += 1000;
		    }
		}
	从 Contracts v2 开始，这种模式不仅不被鼓励，而且被禁止。变量 `totalSupply` 和 `balances` 现在是的私有实现细节 ERC20，您不能直接写入它们。相反，有一个内部 [_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_mint-address-uint256-) 函数可以做到这一点
	
		contract ERC20FixedSupply is ERC20 {
		    constructor() ERC20("Fixed", "FIX") {
		        _mint(msg.sender, 1000);
		    }
		}
	像这样封装状态使扩展合约更安全。例如，在第一个示例中，我们必须手动保持 `totalSupply` 与修改后的余额同步，这很容易忘记。事实上，我们省略了其他一些也很容易忘记的东西：`Transfer` 标准要求的事件，一些客户依赖的事件。第二个例子没有这个错误，因为内部 `_mint` 函数会处理它。
- 奖励矿工

	内部 [_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_mint-address-uint256-) 函数是允许我们编写实现供应机制的 ERC20 扩展的关键构建块。

	我们将实施的机制是对生产以太坊区块的矿工的token奖励。在 Solidity 中，我们可以在全局变量中访问当前区块矿工的地址 `block.coinbase`。每当有人在我们的token上调用该函数时，我们都会向该地址铸造token奖励 `mintMinerReward()`。这个机制听起来很傻，但你永远不知道这会产生什么样的动态，值得分析和试验！

		contract ERC20WithMinerReward is ERC20 {
		    constructor() ERC20("Reward", "RWD") {}
		
		    function mintMinerReward() public {
		        _mint(block.coinbase, 1000);
		    }
		}	
正如我们所看到的，`_mint` 正确地执行此操作非常容易。

- 模块化机制

	合约中已经包含了一种供应机制：`ERC20PresetMinterPauser`. 这是一种通用机制，其中一组帐户被分配 `minter` 角色，授予他们调用 `mint` 函数的权限，一个外部版本的 `_mint`.

	这可用于集中式铸币，其中外部拥有的帐户（即拥有一对加密密钥的人）决定创建多少供应以及为谁创建。这种机制有非常合法的用例，例如传统的资产支持的[稳定币](https://medium.com/reserve-currency/why-another-stablecoin-866f774afede#3aea)。

	但是，具有铸币者角色的账户不需要由外部拥有，也可以是实现无信任机制的智能合约。事实上，我们可以实现与上一节相同的行为。

		contract MinerRewardMinter {
		    ERC20PresetMinterPauser _token;
		
		    constructor(ERC20PresetMinterPauser token) {
		        _token = token;
		    }
		
		    function mintMinerReward() public {
		        _token.mint(block.coinbase, 1000);
		    }
		}	
	此合约在使用实例初始 `ERC20PresetMinterPauser` 并授予该 `minter` 合约的角色时，将导致与上一节中实现的行为完全相同。使用 `ERC20PresetMinterPauser` 有趣之处在于，我们可以通过将角色分配给多个合约来轻松组合多种供应机制，而且我们可以动态地做到这一点。

	要了解有关角色和许可系统的更多信息，请访问我们的[访问控制指南](https://docs.openzeppelin.com/contracts/4.x/access-control)。
- 自动化奖励

	到目前为止，我们的供应机制是手动触发的，但 ERC20 也允许我们通过 [_beforeTokenTransfer](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_beforeTokenTransfer-address-address-uint256-) 钩子扩展令牌的核心功能（请参阅[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)）。

	添加到前面部分的供应机制，我们可以使用这个钩子为区块链中包含的每个token转移铸造矿工奖励。
	
		contract ERC20WithAutoMinerReward is ERC20 {
		    constructor() ERC20("Reward", "RWD") {}
		
		    function _mintMinerReward() internal {
		        _mint(block.coinbase, 1000);
		    }
		
		    function _beforeTokenTransfer(address from, address to, uint256 value) internal virtual override {
		        if (!(from == address(0) && to == block.coinbase)) {
		          _mintMinerReward();
		        }
		        super._beforeTokenTransfer(from, to, value);
		    }
		}	
- 包起来

	我们已经看到了两种实现 ERC20 供应机制的方法：
	
	- 内部通过 `_mint`
	- 和外部通过 `ERC20PresetMinterPauser`。

	希望这可以帮助您了解如何使用 OpenZeppelin 合约及其背后的一些设计原则，并且您可以将它们应用到您自己的智能合约中。


### ERC721
我们已经讨论了如何使用 [ERC20](https://docs.openzeppelin.com/contracts/4.x/erc20) 制作可替代的token，但如果不是所有token都相同怎么办？这出现在房地产、投票权或收藏品等情况下，由于它们的有用性、稀有性等，某些物品的价值高于其他物品。ERC721 是代表不可替代token所有权的标准，也就是说，其中每个令牌都是唯一的。

ERC721 是一个比 ERC20 更复杂的标准，具有多个可选扩展，并且被拆分为多个合约。OpenZeppelin 合约在如何组合这些方面提供了灵活性，以及​​自定义有用的扩展。查看 [API](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721) 参考以了解有关这些的更多信息。

- 构建 ERC721 token合约

	我们将使用 ERC721 来跟踪游戏中的项目，每个项目都有自己独特的属性。每当将一个奖励给玩家时，它都会被铸造并发送给他们。玩家可以自由地保留他们的token或在他们认为合适的时候与其他人交易，就像他们在区块链上的任何其他资产一样！请注意，任何帐户都可以调用 `awardItem` 铸造项目。要限制哪些帐户可以铸造项目，我们可以添加 [Access Control](https://docs.openzeppelin.com/contracts/4.x/access-control)。

	以下是token化项目的合约可能的样子：
	
		// contracts/GameItem.sol
		// SPDX-License-Identifier: MIT
		pragma solidity ^0.8.0;
		
		import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
		import "@openzeppelin/contracts/utils/Counters.sol";
		
		contract GameItem is ERC721URIStorage {
		    using Counters for Counters.Counter;
		    Counters.Counter private _tokenIds;
		
		    constructor() ERC721("GameItem", "ITM") {}
		
		    function awardItem(address player, string memory tokenURI)
		        public
		        returns (uint256)
		    {
		        uint256 newItemId = _tokenIds.current();
		        _mint(player, newItemId);
		        _setTokenURI(newItemId, tokenURI);
		
		        _tokenIds.increment();
		        return newItemId;
		    }
		}
	[ERC721URIStorage](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#ERC721URIStorage) 合约是 ERC721 的实现，包括元数据标准扩展 ( [IERC721Metadata](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#IERC721Metadata)) 以及每个令牌元数据的机制。这就是该 [_setTokenURI](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#ERC721-_setTokenURI-uint256-string-) 方法的来源：我们使用它来存储项目的元数据。

	另请注意，与 ERC20 不同，ERC721 缺少 `decimals` 字段，因为每个令牌都是不同的并且无法分区。

	- 创建新项目

			> gameItem.awardItem(playerAddress, "https://game.example/item-id-8u5h2m.json")
			Transaction successful. Transaction hash: 0x...
			Events emitted:
			 - Transfer(0x0000000000000000000000000000000000000000, playerAddress, 7)
	- 查询到的每个项目的所有者和元数据：

			> gameItem.ownerOf(7)
			playerAddress
			> gameItem.tokenURI(7)
			"https://game.example/item-id-8u5h2m.json"
	- 这 `tokenURI` 应该解析为可能类似于以下内容的 JSON 文档

			{
			    "name": "Thor's hammer",
			    "description": "Mjölnir, the legendary hammer of the Norse god of thunder.",
			    "image": "https://game.example/item-id-8u5h2m.png",
			    "strength": 20
			}
	有关 tokenURI 元数据 JSON 模式的更多信息，请查看[ERC721 规范](https://eips.ethereum.org/EIPS/eip-721)。
	
	- 注意

		该项目的信息包含在元数据中，但该信息不在链上！所以游戏开发者可以改变底层元数据，改变游戏规则！
	- 元数据上链

		如果您想将所有项目信息放在链上，您可以通过提供[Base64](https://docs.openzeppelin.com/contracts/4.x/utilities#base64)带有 JSON 模式编码的 Data URI 来扩展 ERC721 来这样做（尽管它会相当昂贵）。您还可以利用 IPFS 来存储 tokenURI 信息，但这些技术超出了本概述指南的范围。
- 预设 ERC721 合约

	预设 ERC721 可用，[ERC721PresetMinterPauserAutoId](https://docs.openzeppelin.com/contracts/4.x/erc721#api:presets.adoc#ERC721PresetMinterPauserAutoId). 它预设为
	
	- 允许使用令牌 ID 和 URI 自动生成令牌铸造（创建）
	- 停止所有令牌传输（暂停）
	- 并允许持有者燃烧（销毁）他们的令牌。

	该合约使用[访问控制](https://docs.openzeppelin.com/contracts/4.x/access-control)来控制对铸币和暂停功能的访问。部署合约的账户将被授予 minter 和 pauser 角色，以及默认的 admin 角色。

	该合约无需编写任何 Solidity 代码即可部署。它可以按原样用于快速原型设计和测试，但也适用于生产环境。

### ERC777
与 ERC20 一样，ERC777 是[可替代token](https://docs.openzeppelin.com/contracts/4.x/tokens#different-kinds-of-tokens)的标准，专注于在交易token时允许更复杂的交互。

更一般地说，但对于token来说它通过提供相当于一个 `msg.value` 字段的字段来使token和以太币更紧密地联系在一起。

该标准还带来了多项质量改进，例如消除周围的混乱 `decimals`，使用适当的事件进行铸造和燃烧等等，但其杀手级功能是接收钩子。钩子只是合约中的一个函数，在向其发送token时调用它，这意味着账户和合约可以对接收token做出反应。

`approve` 这实现了许多有趣的用例，包括

- 使用token进行原子购买（无需 `transferFrom` 在两个单独的交易中进行）
- 拒绝接收token（通过在挂钩调用上恢复）
- 将接收到的token重定向到其他地址（类似于如何 [PaymentSplitter](https://docs.openzeppelin.com/contracts/4.x/erc777#api:payment.adoc#PaymentSplitter) 这样做）等等。

此外，由于合约需要实现这些钩子才能接收token，因此任何token都不会卡在不了解 ERC777 协议的合约中，就像使用 ERC20 时发生的无数次一样。

- 如果我已经使用 ERC20 怎么办？

	该标准已涵盖您！ERC777 标准与 ERC20 向后兼容，这意味着您可以使用标准函数与这些令牌进行交互，就像它们是 ERC20 一样，同时仍然获得所有细节，包括发送挂钩。请参阅[EIP 的向后兼容性部分](https://eips.ethereum.org/EIPS/eip-777#backward-compatibility)以了解更多信息。
- 构建 ERC777 token合约

	我们将复制 [ERC20 指南](https://docs.openzeppelin.com/contracts/4.x/erc20#constructing-an-erc20-token-contract) GLD 的示例，这次使用 ERC777。与往常一样，请查看以了解有关每个功能的详细信息的更多信息。[API reference](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777)	
	
		// contracts/GLDToken.sol
		// SPDX-License-Identifier: MIT
		pragma solidity ^0.8.0;
		
		import "@openzeppelin/contracts/token/ERC777/ERC777.sol";
		
		contract GLDToken is ERC777 {
		    constructor(uint256 initialSupply, address[] memory defaultOperators)
		        ERC777("Gold", "GLD", defaultOperators)
		    {
		        _mint(msg.sender, initialSupply, "", "");
		    }
		}

	在这种情况下，我们将从 [ERC777](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#ERC777) 合约中扩展，它提供了对 ERC20 的兼容性支持的实现。API 与 的 API 非常相似 ERC777，我们将再次使用 [_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#ERC777-_mint-address-address-uint256-bytes-bytes-) 将分配 `initialSupply` 给部署者帐户。与 [ERC20 _mint](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC20#ERC20-_mint-address-uint256-)不同，这个包含一些额外的参数，但您现在可以放心地忽略这些参数。

	您会注意到两者 [name](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#IERC777-name--) 和 [symbol](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#IERC777-symbol--) 被分配，但不是 [decimals](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#ERC777-decimals--)。ERC777 规范强制包含对这些功能的支持（与 ERC20 不同，我们必须包含 [ERC20Detailed]()），也强制要求 `decimals` 始终返回固定值18，因此无需自己设置。有关 `decimals` 的作用和重要性的回顾，请参阅我们的 [ERC20 指南](https://docs.openzeppelin.com/contracts/4.x/erc20#a-note-on-decimals)。

	最后，我们需要设置 [defaultOperators](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#IERC777-defaultOperators--) 能够代表持有人转移token的特殊账户（通常是其他智能合约）。如果您不打算在令牌中使用运算符，则可以简单地传递一个空数组。请继续关注即将发布的有关 ERC777 操作员的深入指南！

	这就是一个基本的token合约！我们现在可以部署它，并使用相同的 [balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#IERC777-balanceOf-address-) 方法查询部署者的余额：
	
		> GLDToken.balanceOf(deployerAddress)
		1000
	要将token从一个帐户转移到另一个帐户，我们可以使用
	
	-  ERC20's [transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#ERC777-transfer-address-uint256-) 方法
	-  或 ERC777's [send](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#ERC777-send-address-uint256-bytes-) 方法

	它的作用非常相似，但添加了一个可选 data 字段：
	
		> GLDToken.transfer(otherAddress, 300)
		> GLDToken.send(otherAddress, 300, "")
		> GLDToken.balanceOf(otherAddress)
		600
		> GLDToken.balanceOf(deployerAddress)
		400
- 向合约发送token

	使用时的一个关键区别 [send](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC777#ERC777-send-address-uint256-bytes-) 是令牌转移到其他合约可能会返回以下消息：

		ERC777: token recipient contract has no implementer for ERC777TokensRecipient
	这是一件好事！这意味着接收合约尚未将自己注册为了解 ERC777 协议，因此禁用向其传输以防止token被永久锁定。例如，[Golem 合约](https://etherscan.io/token/0xa74476443119A942dE498590Fe1f2454d7D4aC0d?a=0xa74476443119A942dE498590Fe1f2454d7D4aC0d) 目前持有超过 35万 个 GNTtoken，价值数万美元，并且缺乏将它们从那里取出的方法。几乎每个 ERC20 支持的项目都会发生这种情况，通常是由于用户错误。

	即将发布的指南将介绍合约如何将自己注册为接收者、发送和接收挂钩以及 ERC777 的其他高级功能！

### ERC1155
ERC1155 是一种新颖的token标准，旨在从以前的标准中汲取精华，以创建与[可替代性无关](https://docs.openzeppelin.com/contracts/4.x/tokens#different-kinds-of-tokens)且[高效的token合约](https://docs.openzeppelin.com/contracts/4.x/tokens#but_first_coffee_a_primer_on_token_contracts)。

ERC1155 从 [ERC20](https://docs.openzeppelin.com/contracts/4.x/erc20)、[ERC721](https://docs.openzeppelin.com/contracts/4.x/erc721)和[ERC777](https://docs.openzeppelin.com/contracts/4.x/erc777)中汲取灵感。如果您不熟悉这些标准，请在继续之前查看他们的指南。
#### 多token标准
ERC1155 的显着特点是它使用单个智能合约一次代表多个token。这就是为什么它的 [balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155-balanceOf-address-uint256-) 功能不同于 ERC20 和 ERC777 的原因：它有一个额外的 `id` 参数，用于您要查询余额的token的标识符。

这类似于 ERC721 做事的方式，但在该标准中，token `id` 没有平衡的概念：每个token都是不可替代的，存在或不存在。ERC721 [balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#IERC721-balanceOf-address-)函数是指一个账户有多少个不同的token，而不是每个token有多少。另一方面，在 ERC1155 账户中，每个token都有不同的余额 `id`，不可替代的token是通过简单地铸造其中一个来实现的。

这种方法可以为需要多个token的项目节省大量GAS。无需为每种token类型部署新合约，单个 ERC1155 token合约可以保存整个系统状态，从而降低部署成本和复杂性。
#### 批量操作
由于所有状态都保存在单个合约中，因此可以非常有效地在单个交易中对多个token进行操作。该标准提供了两个功能

- [balanceOfBatch](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155-balanceOfBatch-address---uint256---)
- [safeBatchTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155-safeBatchTransferFrom-address-address-uint256---uint256---bytes-)

这使得查询多个余额和转移多个token变得更简单，耗气更少。

本着标准精神，我们还在非标准函数中加入了批处理操作，例如 [_mintBatch](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#ERC1155-_mintBatch-address-uint256---uint256---bytes-).
#### 构建 ERC1155 token合约
我们将使用 ERC1155 来跟踪我们游戏中的多个项目，每个项目都有自己独特的属性。我们将是所有项目铸造合约的部署者，然后我们可以将其转移给玩家。玩家可以自由地保留他们的token或在他们认为合适的时候与其他人交易，就像他们对区块链上的任何其他资产一样！

为简单起见，我们将在构造函数中铸造所有项目，但您可以在合约中添加铸造功能，以便按需铸造给玩家。

有关铸币机制的概述，请查看[创建 ERC20 供应](https://docs.openzeppelin.com/contracts/4.x/erc20-supply)。

以下是token化项目的合约可能的样子：			

	// contracts/GameItems.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
	
	contract GameItems is ERC1155 {
	    uint256 public constant GOLD = 0;
	    uint256 public constant SILVER = 1;
	    uint256 public constant THORS_HAMMER = 2;
	    uint256 public constant SWORD = 3;
	    uint256 public constant SHIELD = 4;
	
	    constructor() ERC1155("https://game.example/api/item/{id}.json") {
	        _mint(msg.sender, GOLD, 10**18, "");
	        _mint(msg.sender, SILVER, 10**27, "");
	        _mint(msg.sender, THORS_HAMMER, 1, "");
	        _mint(msg.sender, SWORD, 10**9, "");
	        _mint(msg.sender, SHIELD, 10**9, "");
	    }
	}
请注意，对于我们的游戏物品，黄金是可替代的token，而雷神之锤是不可替代的token，因为我们只铸造了一个。

[ERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#ERC1155) 合约包括可选的扩展 [IERC1155MetadataURI](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155MetadataURI)。这就是 [uri](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155MetadataURI-uri-uint256-) 函数的来源：我们使用它来检索元数据 uri。

另请注意，与 ERC20 不同，ERC1155 缺少 `decimals` 字段，因为每个令牌都是不同的并且无法分区。

部署后，我们将能够查询部署者的余额：

	> gameItems.balanceOf(deployerAddress,3)
	1000000000
我们可以将物品转移到玩家账户

	> gameItems.safeTransferFrom(deployerAddress, playerAddress, 2, 1, "0x0")
	> gameItems.balanceOf(playerAddress, 2)
	1
	> gameItems.balanceOf(deployerAddress, 2)
	0	
也可以批量转账到玩家账户，获取批量余额

	> gameItems.safeBatchTransferFrom(deployerAddress, playerAddress, [0,1,3,4], [50,100,1,1], "0x0")
	> gameItems.balanceOfBatch([playerAddress,playerAddress,playerAddress,playerAddress,playerAddress], [0,1,2,3,4])
	[50,100,1,1,1]	
可以获取元数据uri：

	> gameItems.uri(2)
	"https://game.example/api/item/{id}.json"	
uri 可以包括字符串 {id}，客户端必须用实际的令牌 id 替换，小写十六进制(没有0x前缀)，前导0填充到64十六进制字符。

对于令牌 ID2 和 uri，`https://game.example/api/item/{id}.json` 客户端将替换 {id} 为 `0000000000000000000000000000000000000000000000000000000000000002` 以在 [https://game.example/api/item/0000000000000000000000000000000000000000000000000000000000000002.json](https://game.example/api/item/0000000000000000000000000000000000000000000000000000000000000002.json).

令牌 ID 2 的 JSON 文档可能类似于：

	{
	    "name": "Thor's hammer",
	    "description": "Mjölnir, the legendary hammer of the Norse god of thunder.",
	    "image": "https://game.example/item-id-8u5h2m.png",
	    "strength": 20
	}	
有关元数据 JSON 模式的更多信息，请查看 [ERC-1155 元数据 URI JSON 模式](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md#erc-1155-metadata-uri-json-schema)。

	您会注意到该项目的信息包含在元数据中，但该信息不在链上！所以游戏开发者可以改变底层元数据，改变游戏规则！


	如果您想将所有项目信息放在链上，您可以通过提供 Base64 带有 JSON 模式编码的 Data URI 来扩展 ERC721 来这样做（尽管它会相当昂贵）。您还可以利用 IPFS 来存储 URI 信息，但这些技术超出了本概述指南的范围

#### 向合约发送token
使用时的一个关键区别 [safeTransferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155-safeTransferFrom-address-address-uint256-uint256-bytes-) 是令牌转移到其他合约可能会返回以下消息：

	ERC1155: transfer to non ERC1155Receiver implementer
这是一件好事！这意味着接收合约尚未将自己注册为了解 ERC1155 协议，因此禁用向其传输以防止token被永久锁定。例如，[Golem 合约](https://etherscan.io/token/0xa74476443119A942dE498590Fe1f2454d7D4aC0d?a=0xa74476443119A942dE498590Fe1f2454d7D4aC0d) 目前持有超过 35 万个 GNT token，价值数万美元，并且缺乏将它们从那里取出的方法。几乎每个 ERC20 支持的项目都会发生这种情况，通常是由于用户错误。

为了让我们的合约接收 ERC1155 token，我们可以继承 [ERC1155Holder](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#ERC1155Holder) 为我们处理注册的便利合约。尽管我们需要记住实现功能以允许将token从我们的合约中转移出来：

	// contracts/MyContract.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";
	
	contract MyContract is ERC1155Holder {
	}
我们还可以使用 [onERC1155Received](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-) 和 [onERC1155BatchReceived](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155Receiver-onERC1155BatchReceived-address-address-uint256---uint256---bytes-) 函数实现更复杂的场景。

#### 预设ERC1155合约
预设 ERC1155 可用，[ERC1155PresetMinterPauser](https://docs.openzeppelin.com/contracts/4.x/erc1155#api:presets.adoc#ERC1155PresetMinterPauser). 它预设为允许token铸造（创建） 

- 包括批量铸造，停止所有token转移（暂停）并允许持有者燃烧（销毁）他们的token。

该合约使用[访问控制](https://docs.openzeppelin.com/contracts/4.x/access-control)来控制对铸币和暂停功能的访问。部署合约的账户将被授予 minter 和 pauser 角色以及默认的 admin 角色。

该合约无需编写任何 Solidity 代码即可部署。它可以按原样用于快速原型设计和测试，但也适用于生产环境。

## 如何设置链上理
在本指南中，我们将了解 OpenZeppelin 的 Governor 合约是如何工作的，如何设置它，以及如何使用它来创建提案、为提案投票并执行提案，使用 Ethers.js 和 Tally 提供的工具。

在 [Governance API](https://docs.openzeppelin.com/contracts/4.x/api/governance) 中 查找详细的合约文档。
### 介绍
去中心化协议从公开发布的那一刻起就在不断发展。通常，最初的团队在第一阶段保留对这种演变的控制，但最终将其委托给利益相关者社区。该社区做出决策的过程称为链上治理，它已成为去中心化协议的核心组成部分，推动了各种决策，例如

- 参数调整
- 智能合约升级
- 与其他协议的集成
- 资金管理
- 赠款等。

该治理协议通常在称为 “Governor” 的特殊用途合约中实现。迄今为止，Compound 设计的 GovernorAlpha 和 Bravo 合约非常成功和流行，其缺点是具有不同需求的项目必须分叉代码来定制代码以满足他们的需求，这可能会带来引入安全问题的高风险。对于 OpenZeppelin 合约，我们着手构建 Governor 合约的模块化系统，这样就不需要分叉，并且可以通过使用 Solidity 继承编写小模块来适应不同的需求。您会在 OpenZeppelin Contracts 中找到开箱即用的最常见要求，但编写额外的要求很简单，此外，我们将在未来版本中根据社区的要求添加新功能。

### 兼容性
OpenZeppelin 的 Governor 系统在设计时考虑到与基于 Compound 的 GovernorAlpha 和 Bravo 的现有系统的兼容性。因此，您会发现许多模块以两种变体形式呈现，其中一种是为了与这些系统兼容而构建的。
### ERC20Votes & ERC20VotesComp
跟踪选票和投票委托的 ERC20 扩展就是这样一种情况。较短的版本是更通用的版本，因为它可以支持大于 2^96 的token供应，而 “Comp” 变体在这方面受到限制，但完全符合 GovernorAlpha 和 Bravo 使用的 COMP token的接口。两种合约变体共享相同的事件，因此仅在查看事件时它们是完全兼容的。
### Governor & GovernorCompatibilityBravo
默认情况下，OpenZeppelin Governor 合约与GovernorAlpha 或Bravo 接口不兼容，因为某些功能不同或缺失，尽管它共享所有相同的事件。但是，可以通过从 GovernorCompatibilityBravo 模块继承来选择完全兼容。如果没有这个模块，合约的部署和使用成本会更低。
### GovernorTimelockControl & GovernorTimelockCompound
当你的 Governor 合约使用时间锁时，你可以使用 OpenZeppelin 的`TimelockController` 或 `Compound` 的时间锁。根据时间锁的选择，您应该分别选择相应的 Governor 模块：`GovernorTimelockControl `或 `GovernorTimelockCompound`。这允许您将现有的 GovernorAlpha 实例迁移到基于OpenZeppelin 的 Governor，而无需更改正在使用的时间锁。
### Tally
[Tally](https://www.withtally.com/) 是一个成熟的应用程序，用于用户拥有的链上治理。它包括投票仪表板、提案创建向导、实时研究和分析以及教育内容。

对于所有这些选项，Governor 将与 Tally 兼容：用户将能够创建提案、可视化投票权和拥护者、导航提案和投票。特别是对于提案创建，项目还可以使用 Defender Admin 作为替代界面。

在本指南的其余部分，我们将重点介绍 fresh 在 OpenZeppelin Governor 功能的全新部署，而不考虑与 GovernorAlpha 或 Bravo 的兼容性。
### 设置
####Token
我们治理设置中每个账户的投票权将由 ERC20 token决定。token必须实现 ERC20Votes 扩展。此扩展将跟踪历史余额，以便从过去的快照而不是当前余额中检索投票权，这是防止双重投票的重要保护。

	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.2;
	
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
	import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
	
	contract MyToken is ERC20, ERC20Permit, ERC20Votes {
	    constructor() ERC20("MyToken", "MTK") ERC20Permit("MyToken") {}
	
	    // 下面的函数是 solididity 所需要的重写.
	
	    function _afterTokenTransfer(address from, address to, uint256 amount)
	        internal
	        override(ERC20, ERC20Votes)
	    {
	        super._afterTokenTransfer(from, to, amount);
	    }
	
	    function _mint(address to, uint256 amount)
	        internal
	        override(ERC20, ERC20Votes)
	    {
	        super._mint(to, amount);
	    }
	
	    function _burn(address account, uint256 amount)
	        internal
	        override(ERC20, ERC20Votes)
	    {
	        super._burn(account, amount);
	    }
	}
如果您的项目已经有一个不包含 ERC20Votes 且不可升级的实时令牌，您可以使用 ERC20Wrapper 将其包装在治理令牌中。这将允许token持有者通过一对一地包装他们的token来参与治理。

	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.2;
	
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
	import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Wrapper.sol";
	
	contract MyToken is ERC20, ERC20Permit, ERC20Votes, ERC20Wrapper {
	    constructor(IERC20 wrappedToken)
	        ERC20("MyToken", "MTK")
	        ERC20Permit("MyToken")
	        ERC20Wrapper(wrappedToken)
	    {}
	
	    //下面的函数是 solididity 所需要的重写。
	
	    function _afterTokenTransfer(address from, address to, uint256 amount)
	        internal
	        override(ERC20, ERC20Votes)
	    {
	        super._afterTokenTransfer(from, to, amount);
	    }
	
	    function _mint(address to, uint256 amount)
	        internal
	        override(ERC20, ERC20Votes)
	    {
	        super._mint(to, amount);
	    }
	
	    function _burn(address account, uint256 amount)
	        internal
	        override(ERC20, ERC20Votes)
	    {
	        super._burn(account, amount);
	    }
	}

投票权可以通过不同的方式确定：

- 多个 ERC20 token
- ERC721 token
- `sybil resistant identities` 等

所有这些选项都可能通过为您的 Governor 编写自定义投票模块来支持。目前 OpenZeppelin 合约中唯一可用的其他投票权来源是 [ERC721Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#ERC721Votes).

#### Governor
最初，我们将建立一个没有时间锁的 Governor。核心逻辑由 Governor 合约给出，但我们仍然需要选择：

- 1）如何确定投票权

	对于 1)，我们将使用 GovernorVotes 模块，该模块与 IVotes 实例挂钩，以在提案生效时根据账户持有的token余额来确定账户的投票权。该模块需要令牌的地址作为构造函数参数。
- 2）法定人数需要多少票

	对于 2)，我们将使用与 ERC20Votes 一起使用的 GovernorVotesQuorumFraction 将法定人数定义为检索提案投票权的区块总供应量的百分比。这需要一个构造函数参数来设置百分比。现在大多数 Governors 使用4%，所以我们将使用参数4初始化模块（这表示百分比，结果为4%）。
- 3）人们在投票时有哪些选择以及这些选票如何计数

	对于 3)，我们将使用 GovernorCountingSimple，该模块为选民提供 3 个选项：
	
	- 赞成
	- 反对
	- 和弃权

	其中只有赞成和弃权票计入法定人数。
- 以及 4) 应该使用什么类型的令牌进行投票。

这些方面中的每一个都可以通过编写自己的模块来定制，或者更容易从 OpenZeppelin Contracts 中选择一个。

除了这些模块之外，Governor 本身还有一些我们必须设置的参数。

- 投票延迟(votingDelay)

	提案创建后多长时间应该固定投票权。较大的投票延迟使用户有时间在必要时取消抵押token。
- `votingPeriod`

	提案保持开放投票的时间
	
这两个参数以块数指定。假设出块时间约为 13.14 秒，我们将设置 

- `votingDelay` = 1 天 = 6570 个块
- `votingPeriod` = 1 周 = 45992 个块。

我们也可以选择设置提案阈值。这将提案创建限制为具有足够投票权的帐户。

	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.2;
	
	import "./governance/Governor.sol";
	import "./governance/compatibility/GovernorCompatibilityBravo.sol";
	import "./governance/extensions/GovernorVotes.sol";
	import "./governance/extensions/GovernorVotesQuorumFraction.sol";
	import "./governance/extensions/GovernorTimelockControl.sol";
	
	contract MyGovernor is Governor, GovernorCompatibilityBravo, GovernorVotes, GovernorVotesQuorumFraction, GovernorTimelockControl {
	    constructor(IVotes _token, TimelockController _timelock)
	        Governor("MyGovernor")
	        GovernorVotes(_token)
	        GovernorVotesQuorumFraction(4)
	        GovernorTimelockControl(_timelock)
	    {}
	
	    function votingDelay() public pure override returns (uint256) {
	        return 6575; // 1 day
	    }
	
	    function votingPeriod() public pure override returns (uint256) {
	        return 46027; // 1 week
	    }
	
	    function proposalThreshold() public pure override returns (uint256) {
	        return 0;
	    }
	
	    // 下面的函数是solididity所需要的重写。
	
	    function quorum(uint256 blockNumber)
	        public
	        view
	        override(IGovernor, GovernorVotesQuorumFraction)
	        returns (uint256)
	    {
	        return super.quorum(blockNumber);
	    }
	
	    function getVotes(address account, uint256 blockNumber)
	        public
	        view
	        override(IGovernor, GovernorVotes)
	        returns (uint256)
	    {
	        return super.getVotes(account, blockNumber);
	    }
	
	    function state(uint256 proposalId)
	        public
	        view
	        override(Governor, IGovernor, GovernorTimelockControl)
	        returns (ProposalState)
	    {
	        return super.state(proposalId);
	    }
	
	    function propose(address[] memory targets, uint256[] memory values, bytes[] memory calldatas, string memory description)
	        public
	        override(Governor, GovernorCompatibilityBravo, IGovernor)
	        returns (uint256)
	    {
	        return super.propose(targets, values, calldatas, description);
	    }
	
	    function _execute(uint256 proposalId, address[] memory targets, uint256[] memory values, bytes[] memory calldatas, bytes32 descriptionHash)
	        internal
	        override(Governor, GovernorTimelockControl)
	    {
	        super._execute(proposalId, targets, values, calldatas, descriptionHash);
	    }
	
	    function _cancel(address[] memory targets, uint256[] memory values, bytes[] memory calldatas, bytes32 descriptionHash)
	        internal
	        override(Governor, GovernorTimelockControl)
	        returns (uint256)
	    {
	        return super._cancel(targets, values, calldatas, descriptionHash);
	    }
	
	    function _executor()
	        internal
	        view
	        override(Governor, GovernorTimelockControl)
	        returns (address)
	    {
	        return super._executor();
	    }
	
	    function supportsInterface(bytes4 interfaceId)
	        public
	        view
	        override(Governor, IERC165, GovernorTimelockControl)
	        returns (bool)
	    {
	        return super.supportsInterface(interfaceId);
	    }
	}

### 时间锁
为治理决策添加时间锁是一种很好的做法。如果用户在执行之前不同意某个决定，这允许用户退出系统。我们将结合使用 OpenZeppelin 的 TimelockController 和 GovernorTimelockControl 模块

使用时间锁时，执行提案的是时间锁，因此时间锁应该持有任何资金、所有权和访问控制角色。在 4.5 版本之前，当使用时间锁时，无法在 Governor 合约中收回资金！在 4.3 版本之前，当使用 Compound Timelock 时，时间锁中的 ETH 是不容易获取的。

TimelockController 使用我们需要了解的 AccessControl 设置来设置角色。

- Proposer 角色负责排队操作

	这是应该授予 Governor 实例的角色，并且它应该可能是系统中唯一的提议者。
- Executor 角色负责执行已经可用的操作

	我们可以将此角色分配给特殊的零地址以允许任何人执行（如果操作对时间特别敏感，则应将 Governor 设为 Executor）。
- 最后是 Admin 角色

	它可以授予和撤销之前的两个角色：这是一个非常敏感的角色，会自动授予部署者和时间锁本身，但部署者应该在设置后放弃。

### 提案生命周期
让我们来看看如何在我们新部署的 Governor 上创建和执行提案。

提案是 Governor 合约在通过后将执行的一系列操作。每个动作都由一个目标地址、编码函数调用的调用数据和要包含的 ETH 数量组成。此外，提案包括人类可读的描述。
#### 创建一个提案
假设我们要创建一个提案，以来自治理金库的 ERC20 token的形式为团队提供资助。该提案将包含一个动作，其中目标是 ERC20Token，calldata 是编码的函数调用 `transfer(<team wallet>, <grant amount>)`，并附加 0 ETH。

通常，提案将在 Tally 或 Defender 等界面的帮助下创建。在这里，我们将展示如何使用 Ethers.js 创建提案。

首先，我们获取提案操作所需的所有参数。

	const tokenAddress = ...;
	const token = await ethers.getContractAt(‘ERC20’, tokenAddress);
	
	const teamAddress = ...;
	const grantAmount = ...;
	const transferCalldata = token.interface.encodeFunctionData(‘transfer’, [teamAddress, grantAmount]);
现在我们准备调用 Govenor 的提议函数。请注意，我们不是传入一个动作数组，而是传入三个数组，分别对应于

- 目标列表
- 值列表
- calldata 数据列表

在这种情况下，它是一个单一的动作，所以很简单

	await governor.propose(
	  [tokenAddress],
	  [0],
	  [transferCalldata],
	  “Proposal #1: Give grant to team”,
	);
这将创建一个新提案其提案 ID 是通过将提案数据散列在一起获得的，并且也将在事务日志中的事件中找到。

#### 投票
一旦提案生效，代表就可以投票。请注意，拥有投票权的是受托人：如果token持有者想要参与，他们可以设置一个受信任的代表作为他们的受托人或者他们可以通过自行委托他们的投票权来成为受托人。

投票是通过 `castVote` 函数族与 Govenor 合约进行交互来进行的。选民通常会从治理 UI （例如 Tally）中调用它。

![](./pic/tally-vote.png)
#### 执行提案
投票期结束后，如果达到法定人数（参与了足够的投票权）并且多数人投票赞成，则该提案被视为成功并可以继续执行。这也可以在 Tally 项目的“管理面板”部分中完成。

![](./pic/tally-admin.png)
我们现在将看到如何使用 Ethers.js 手动执行此操作。

如果设置了时间锁，则执行的第一步是排队。您会注意到队列和执行函数都需要传入整个提案参数，而不仅仅是提案 ID。这是必要的，因为这些数据没有存储在链上，作为一种节省GAS的措施。请注意，这些参数始终可以在合约发出的事件中找到。唯一没有完整发送的参数是描述，因为仅需要其散列形式来计算提案 id。

要排队，我们调用队列函数：

	const descriptionHash = ethers.utils.id(“Proposal #1: Give grant to team”);
	
	await governor.queue(
	  [tokenAddress],
	  [0],
	  [transferCalldata],
	  descriptionHash,
	);
这将导致 governor 与时间锁合约进行交互，并在所需的延迟后将操作排队等待执行。

经过足够的时间后（根据时间锁参数），可以执行提案。如果开始时没有时间锁，则可以在提案成功后立即运行此步骤。

	await governor.execute(
	  [tokenAddress],
	  [0],
	  [transferCalldata],
	  descriptionHash,
	);
执行提案会将 ERC20 token转移给选定的接收者。

总结一下：我们建立了一个系统，其中一个金库由项目token持有者的集体决定控制，所有行动都通过链上投票执行的提案来执行。

## 为合约添加跨链支持
如果您的合约的目标是在多链操作的上下文中使用，您可能需要特定的工具来识别和处理这些跨链操作。

OpenZeppelin 提供 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 抽象合约，其中包括专用的内部函数。

在本指南中，我们将介绍一个示例用例：如何构建由外部链上的管理者控制的可升级和可铸造的 ERC20 token。
### 起点，我们的 ERC20 合约
让我们从一个小的 ERC20 合约开始，我们使用 [OpenZeppelin Contracts Wizard](https://wizard.openzeppelin.com/) 引导它，并通过拥有铸币能力的所有者进行扩展。请注意，出于演示目的，我们没有使用内置 `Ownable` 合约。

	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.4;
	
	import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
	import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
	import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
	
	contract MyToken is Initializable, ERC20Upgradeable, UUPSUpgradeable {
	    address public owner;
	
	    modifier onlyOwner() {
	        require(owner == _msgSender(), "Not authorized");
	        _;
	    }
	
	    /// @custom:oz-upgrades-unsafe-allow 构造函数
	    constructor() initializer {}
	
	    function initialize(address initialOwner) initializer public {
	        __ERC20_init("MyToken", "MTK");
	        __UUPSUpgradeable_init();
	
	        owner = initialOwner;
	    }
	
	    function mint(address to, uint256 amount) public onlyOwner {
	        _mint(to, amount);
	    }
	
	    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {
	    }
	}

该token可由合约所有者铸造和升级。

### 准备跨链操作合约。
现在让我们假设该合约将存在于一条链上，但我们希望铸造和升级由 [governor](https://docs.openzeppelin.com/contracts/4.x/governance) 另一条链上的合约执行。

例如，我们可以在 xDai 上拥有我们的token，在主网上有我们的管理者或者我们可以在主网上拥有我们的token，我们的管理者是乐观的

为了做到这一点，我们将首先添加 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 到我们的合约中。您会注意到合约现在是抽象的。

这是因为 `CrossChainEnabled` 它是一个抽象合约：它不依赖于任何特定的链，它以抽象的方式处理跨链交互。这使我们能够轻松地为不同的链重用代码。稍后我们将通过继承抽象的特定于链的实现来对其进行专门化。

	 import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
	+import "@openzeppelin/contracts-upgradeable/crosschain/CrossChainEnabled.sol";
	
	-contract MyToken is Initializable, ERC20Upgradeable, UUPSUpgradeable {
	+abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, UUPSUpgradeable, CrossChainEnabled {
完成后，我们可以使用 `onlyCrossChainSender` 提供的修饰符 `CrossChainEnabled` 来保护铸币和升级操作。

	-    function mint(address to, uint256 amount) public onlyOwner {
	+    function mint(address to, uint256 amount) public onlyCrossChainSender(owner) {
	
	-    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {
	+    function _authorizeUpgrade(address newImplementation) internal override onlyCrossChainSender(owner) {
此更改将有效地将铸币和升级操作限制在 owner 远程链上

### 专门针对特定链
一旦我们的token的抽象跨链版本准备就绪，我们就可以轻松地将其专门用于我们想要的链或者更准确地说，用于我们想要依赖的桥接系统。

这是使用许多 `CrossChainEnabled` 实现之一来完成的。

例如，如果我们的token在 xDai 上，我们的管理者在主网上，我们可以使用 xDai 上可用的 [AMB](https://docs.tokenbridge.net/amb-bridge/about-amb-bridge) 桥，地址为 [0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59](https://blockscout.com/xdai/mainnet/address/0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59)

	[...]
	
	import "@openzeppelin/contracts-upgradeable/crosschain/amb/CrossChainEnabledAMB.sol";
	
	contract MyTokenXDAI is
	    MyTokenCrossChain,
	    CrossChainEnabledAMB(0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59)
	{}
如果token在以太坊主网上，并且我们的 Govenor 在 Optimism 上，我们使用主网上的 Optimism [CrossDomainMessenger](https://community.optimism.io/docs/protocol/protocol-2.0/#l1crossdomainmessenger)，地址为[0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1](https://etherscan.io/address/0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1)。

	[...]
	
	import "@openzeppelin/contracts-upgradeable/crosschain/optimismCrossChainEnabledOptimism.sol";
	
	contract MyTokenOptimism is
	    MyTokenCrossChain,
	    CrossChainEnabledOptimism(0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1)
	{}
### 混合跨域地址很危险
在设计具有跨链支持的合约时，了解可能的回退和正在做出的安全假设至关重要。

在本指南中，我们特别关注限制对特定调用者的访问。`msg.sender` 这通常使用或 `_msgSender()` 来完成（如上所示）。然而，跨链的时候，就不是那么简单了。即使不考虑可能的桥接问题，重要的是要记住，在考虑多链空间时，相同的地址可能对应于非常不同的实体。EOA 钱包只有在钱包的私钥签署交易时才能执行操作。据我们所知，所有 EVM 链都是这种情况，因此来自此类钱包的跨链消息可以说等同于同一钱包的非跨链消息。然而，智能合约的情况却大不相同。

由于智能合约地址的计算方式以及不同链上的智能合约独立存在的事实，您可能有两个非常不同的合约存在于不同链上的同一地址。你可以想象两个具有不同签名者的多重签名钱包在不同的链上使用相同的地址。您还可以看到一个非常基本的智能钱包存在于一条链上，其地址与另一条链上的成熟调控器相同。因此，您应该小心，每当您授予特定地址的权限时，您都可以通过链控制该地址可以从中进行操作。
### 通过访问控制走得更远
在前面的示例中，我们既有 `onlyOwner()` 修饰符又有 `onlyCrossChainSender(owner)` 机制。我们没有使用 [Ownable](https://docs.openzeppelin.com/contracts/4.x/access-control#ownership-and-ownable) 模式，因为包含中的所有权转移机制并非旨在与作为跨链实体的所有者一起使用。与 [Ownable](https://docs.openzeppelin.com/contracts/4.x/access-control#ownership-and-ownable) 不同，[AccessControl](https://docs.openzeppelin.com/contracts/4.x/access-control#role-based-access-control)在捕捉细微差别方面更有效，并且可以有效地用于构建跨链感知合约。

使用 [AccessControlCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlCrossChain) 包括 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) 核心和 [CrossChainEnabled](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled) 抽象。它还包括一些绑定，以使角色管理与跨链操作兼容。

在 `mint` 函数的情况下，调用者必须具有 `MINTER_ROLE` 调用来自同一链的时间。如果调用者在远程链上，那么调用者不应该有`MINTER_ROLE`，而是“别名”版本（ `MINTER_ROLE ^ CROSSCHAIN_ALIAS`）。这通过将本地帐户与来自不同链的远程帐户严格分开来减轻上一节中描述的危险。有关更多详细信息，请参阅 [AccessControlCrossChain](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlCrossChain)文档。

	 import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
	 import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
	+import "@openzeppelin/contracts-upgradeable/access/AccessControlCrossChainUpgradeable.sol";
	
	-abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, UUPSUpgradeable, CrossChainEnabled {
	+abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, UUPSUpgradeable, AccessControlCrossChainUpgradeable {
	
	-    address public owner;
	-    modifier onlyOwner() {
	-        require(owner == _msgSender(), "Not authorized");
	-        _;
	-    }
	
	+    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
	+    bytes32 public constant UPGRADER_ROLE = keccak256("UPGRADER_ROLE");
	
	     function initialize(address initialOwner) initializer public {
	         __ERC20_init("MyToken", "MTK");
	         __UUPSUpgradeable_init();
	+        __AccessControl_init();
	+        _grantRole(_crossChainRoleAlias(DEFAULT_ADMIN_ROLE), initialOwner); // initialOwner 位于远程链上
	-        owner = initialOwner;
	     }
	
	-    function mint(address to, uint256 amount) public onlyCrossChainSender(owner) {
	+    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
	
	-    function _authorizeUpgrade(address newImplementation) internal override onlyCrossChainSender(owner) {
	+    function _authorizeUpgrade(address newImplementation) internal override onlyRole(UPGRADER_ROLE) {
这将产生以下最终代码：

	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.4;
	
	import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
	import "@openzeppelin/contracts-upgradeable/access/AccessControlCrossChainUpgradeable.sol";
	import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
	import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
	
	abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, AccessControlCrossChainUpgradeable, UUPSUpgradeable {
	    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
	    bytes32 public constant UPGRADER_ROLE = keccak256("UPGRADER_ROLE");
	
	    /// @custom:oz-upgrades-unsafe-allow constructor
	    constructor() initializer {}
	
	    function initialize(address initialOwner) initializer public {
	        __ERC20_init("MyToken", "MTK");
	        __AccessControl_init();
	        __UUPSUpgradeable_init();
	
	        _grantRole(_crossChainRoleAlias(DEFAULT_ADMIN_ROLE), initialOwner); // initialOwner is on a remote chain
	    }
	
	    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
	        _mint(to, amount);
	    }
	
	    function _authorizeUpgrade(address newImplementation) internal onlyRole(UPGRADER_ROLE) override {
	    }
	}
	
	import "@openzeppelin/contracts-upgradeable/crosschain/amb/CrossChainEnabledAMB.sol";
	
	contract MyTokenXDAI is
	    MyTokenCrossChain,
	    CrossChainEnabledAMB(0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59)
	{}
	
	import "@openzeppelin/contracts-upgradeable/crosschain/optimismCrossChainEnabledOptimism.sol";
	
	contract MyTokenOptimism is
	    MyTokenCrossChain,
	    CrossChainEnabledOptimism(0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1)
	{}
	
## 实用程序
OpenZeppelin Contracts 提供了大量有用的实用程序，您可以在项目中使用它们。这里有一些比较流行的。
### 密码学
- 检查签名链上

	[ECDSA](https://docs.openzeppelin.com/contracts/4.x/utilities#api:cryptography.adoc#ECDSA) 提供恢复和管理以太坊账户 ECDSA 签名的功能。这些通常是通过生成的 [web3.eth.sign](https://web3js.readthedocs.io/en/v1.2.4/web3-eth.html#sign)，并且是一个 65 字节的数组（bytesSolidity 中的类型），按以下方式排列：[[v (1)], [r (32)], [s (32)]].

	可以用 [ECDSA.recover](https://docs.openzeppelin.com/contracts/4.x/utilities#api:cryptography.adoc#ECDSA-recover-bytes32-bytes-) 恢复数据签名者，并将其地址进行比较以验证签名。大多数钱包将散列数据以进行签名并添加前缀 '\x19Ethereum Signed Message:\n'，因此在尝试恢复以太坊签名消息散列的签名者时，您需要使用 [toEthSignedMessageHash](https://docs.openzeppelin.com/contracts/4.x/utilities#api:cryptography.adoc#ECDSA-toEthSignedMessageHash-bytes32-).
	
		using ECDSA for bytes32;
		
		function _verify(bytes32 data, bytes memory signature, address account) internal pure returns (bool) {
		    return data
		        .toEthSignedMessageHash()
		        .recover(signature) == account;
		}	
		
	正确进行签名验证并非易事：确保您完全阅读并理解[ECDSA](https://docs.openzeppelin.com/contracts/4.x/utilities#api:cryptography.adoc#ECDSA)的文档		
	
- 验证默克尔证明

	[MerkleProof](https://docs.openzeppelin.com/contracts/4.x/utilities#api:cryptography.adoc#MerkleProof) 提供 [verify](https://docs.openzeppelin.com/contracts/4.x/utilities#api:cryptography.adoc#MerkleProof-verify-bytes32---bytes32-bytes32-)，这可以证明某个值是 [Merkle 树](https://en.wikipedia.org/wiki/Merkle_tree) 的一部分。

### Introspection 自我检查
在 Solidity 中，了解合约是否支持您想要使用的接口通常很有帮助。ERC165 是一个有助于进行运行时接口检测的标准。合约为在你的合约中实现 ERC165 和查询其他合约提供了帮助：

- [IERC165](https://docs.openzeppelin.com/contracts/4.x/utilities#api:introspection.adoc#IERC165)

	这是 ERC165 接口，它定义了[supportsInterface](https://docs.openzeppelin.com/contracts/4.x/utilities#api:introspection.adoc#IERC165-supportsInterface-bytes4-). 实现 ERC165 时，您将遵循此接口。
- [ERC165](https://docs.openzeppelin.com/contracts/4.x/utilities#api:introspection.adoc#ERC165)

	如果您想使用合约存储中的查找表来支持接口检测，请继承此合约。您可以使用以下方法注册接口 [_registerInterface(bytes4)](https://docs.openzeppelin.com/contracts/4.x/utilities#api:introspection.adoc#ERC165-_registerInterface-bytes4-)：查看示例用法作为 ERC721 实现的一部分。
- [ERC165Checker](https://docs.openzeppelin.com/contracts/4.x/utilities#api:introspection.adoc#ERC165Checker)

	ERC165Checker 简化了检查合约是否支持您关心的接口的过程。

	- 包括使用 `ERC165Checker for address`;
		- [myAddress._supportsInterface(bytes4)](https://docs.openzeppelin.com/contracts/4.x/utilities#api:introspection.adoc#ERC165Checker-_supportsInterface-address-bytes4-)
		- [myAddress._supportsAllInterfaces(bytes4[])](https://docs.openzeppelin.com/contracts/4.x/utilities#api:introspection.adoc#ERC165Checker-_supportsAllInterfaces-address-bytes4---)

		代码
			
			contract MyContract {
			    using ERC165Checker for address;
			
			    bytes4 private InterfaceId_ERC721 = 0x80ac58cd;
			
			    /**
			     * @dev将ERC721令牌从该合约转移到其他人
			     */
			    function transferERC721(
			        address token,
			        address to,
			        uint256 tokenId
			    )
			        public
			    {
			        require(token.supportsInterface(InterfaceId_ERC721), "IS_NOT_721_TOKEN");
			        IERC721(token).transferFrom(address(this), to, tokenId);
			    }
			}

### 数学
OpenZeppelin Contracts 提供的最流行的数学相关库是 [SafeMath](https://docs.openzeppelin.com/contracts/4.x/utilities#api:math.adoc#SafeMath)，它提供了保护您的合约免受上溢和下溢的数学函数。

包含合约，`using SafeMath for uint256` ;然后调用函数：

- `myNumber.add(otherNumber)`
- `myNumber.sub(otherNumber)`
- `myNumber.div(otherNumber)`
- `myNumber.mul(otherNumber)`
- `myNumber.mod(otherNumber)`

### 支付
想要在多人之间分摊一些付款？也许你有一个应用程序，将 30% 的艺术品购买发送给原始创作者，将 70% 的利润发送给当前所有者；你可以用 [PaymentSplitter](https://docs.openzeppelin.com/contracts/4.x/utilities#api:payment.adoc#PaymentSplitter)!

在 Solidity 中，盲目地向账户汇款存在一些安全问题，因为它允许账户执行任意代码。你可以在[以太坊智能合约最佳实践网站](https://consensys.github.io/smart-contract-best-practices/)上阅读这些安全问题。

解决重入和停滞问题的方法之一是，您可以使用 [PullPayment](https://docs.openzeppelin.com/contracts/4.x/utilities#api:payment.adoc#PullPayment)，而不是立即将 Ether 发送到需要它的帐户，它提供了[_asyncTransfer](https://docs.openzeppelin.com/contracts/4.x/utilities#api:payment.adoc#PullPayment-_asyncTransfer-address-uint256-) 向某物汇款并 [withdrawPayments()](https://docs.openzeppelin.com/contracts/4.x/utilities#api:payment.adoc#PullPayment-withdrawPayments-address-payable-) 稍后请求他们的功能。

如果你想托管一些资金，请查看 [Escrow](https://docs.openzeppelin.com/contracts/4.x/utilities#api:payment.adoc#Escrow) 和 [ConditionalEscrow](https://docs.openzeppelin.com/contracts/4.x/utilities#api:payment.adoc#ConditionalEscrow) 管理一些托管的 Ether 的释放。

### 收藏品
如果您需要比 Solidity 的原生数组和映射更强大的集合支持，请查看 [EnumerableSet](https://docs.openzeppelin.com/contracts/4.x/api/utils#EnumerableSet) 和 [EnumerableMap](https://docs.openzeppelin.com/contracts/4.x/api/utils#EnumerableMap). 它们类似于映射，因为它们在恒定时间内存储和删除元素并且不允许重复条目，但它们还支持枚举，这意味着您可以轻松地查询链上和链下的所有存储条目。

### 杂项
想检查地址是否为合约？使用 [Address](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address) 和[Address.isContract()](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address-isContract-address-)。

想要跟踪一些在每次需要另一个数字时递增 1 的数字？ 查看 [Counters](https://docs.openzeppelin.com/contracts/4.x/api/utils#Counters)。这对很多事情都很有用，例如创建增量标识符，如 [ERC721 指南](https://docs.openzeppelin.com/contracts/4.x/erc721)中所示。
### Base64
[Base64](https://docs.openzeppelin.com/contracts/4.x/api/utils#Base64) util 允许您将 `bytes32` 数据转换为其 Base64 `string` 表示。

这对于为 [ERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC721#IERC721Metadata-tokenURI-uint256-) 或构建 URL 安全的 tokenURI 特别有用 [ERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/ERC1155#IERC1155MetadataURI-uri-uint256-)。这个库提供了一种巧妙的方法来提供符合 URL 安全的[数据 URI](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/Data_URIs/)的字符串，以服务于链上数据结构。

考虑这是一个使用 ERC721 通过 Base64 数据 URI 发送 JSON 元数据的示例：

	// contracts/My721Token.sol
	// SPDX-License-Identifier: MIT
	
	import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
	import "@openzeppelin/contracts/utils/Strings.sol";
	import "@openzeppelin/contracts/utils/Base64.sol";
	
	contract My721Token is ERC721 {
	    using Strings for uint256;
	
	    constructor() ERC721("My721Token", "MTK") {}
	
	    ...
	
	    function tokenURI(uint256 tokenId)
	        public
	        pure
	        override
	        returns (string memory)
	    {
	        bytes memory dataURI = abi.encodePacked(
	            '{',
	                '"name": "My721Token #', tokenId.toString(), '"',
	                //用额外的ERC721元数据属性替换
	            '}'
	        );
	
	        return string(
	            abi.encodePacked(
	                "data:application/json;base64,",
	                Base64.encode(dataURI)
	            )
	        );
	    }
	}
### 多路调用
Multicall 抽象合约带有一个函数，可以将 `multicall` 多个调用捆绑在一个外部调用中。有了它，外部帐户可以执行包含多个函数调用的原子操作。这不仅对于 EOA 在单个事务中进行多次调用很有用，它也是一种在后续调用失败时恢复先前调用的方法。

考虑这个虚拟合约：	

	// contracts/Box.sol
	// SPDX-License-Identifier: MIT
	pragma solidity ^0.8.0;
	
	import "@openzeppelin/contracts/utils/Multicall.sol";
	
	contract Box is Multicall {
	    function foo() public {
	        ...
	    }
	
	    function bar() public {
	        ...
	    }
	}
这是 `multicall` 使用 Truffle 调用函数的方法，允许 foo 并 bar 在单个事务中调用

	// scripts/foobar.js
	
	const Box = artifacts.require('Box');
	const instance = await Box.new();
	
	await instance.multicall([
	    instance.contract.methods.foo().encodeABI(),
	    instance.contract.methods.bar().encodeABI()
	]);	

## 新版本和 API 稳定性
开发智能合约很困难，有时倾向于采用保守的依赖关系方法。然而，掌握新版本也非常重要：这些可能包括错误修复，或弃用旧模式以支持更新和更好的实践。

在这里，我们描述了您应该期待新版本发布的时间，以及这对您作为 OpenZeppelin Contracts 的用户有何影响。
### 发布时间表
OpenZeppelin Contracts 遵循[语义版本控制方案](https://docs.openzeppelin.com/contracts/4.x/releases-stability#versioning-scheme)。我们的目标是每 1 或 2 个月发布一次新的次要版本。

- 发布候选

	在每次发布之前，我们都会发布一个功能冻结的候选发布版本。发布候选的目的是在实际发布之前有一段时间让社区可以审查新代码。如果发现重要问题，可能需要更多的候选版本。
	
	在对发布候选者没有更多更改一周后，发布了新版本。
- 主要版本

	几个月或一年后，可能会发布一个新的主要版本。这些没有计划，但将基于需要发布重大更改，例如重新设计库的核心功能（例如3.0 中的 [访问控制](https://github.com/OpenZeppelin/openzeppelin-contracts/pulls/2112)）。由于我们重视稳定性，我们的目标是不经常发生这种情况（预计专业之间不少于六个月）。但是，当 Solidity 语言发生重大变化时，我们可能会被迫发布一个。

### API 稳定性
在 [OpenZeppelin Contracts 2.0](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v2.0.0) 版本中，我们致力于保持稳定的 API。我们的目标是在这里更准确地定义我们对稳定版和API的理解，因此库的用户可以理解这些保证，并确信他们的项目不会意外中断。

简而言之，API 是稳定的意味着如果您的项目今天可以工作，那么在小升级后它将继续这样做。新合约和功能将在次要版本中添加，但只能以向后兼容的方式添加。
### 版本控制方案
我们遵循 [SemVer](https://semver.org/)，这意味着 API 中断可能会在[主要版本](https://docs.openzeppelin.com/contracts/4.x/releases-stability#release-schedule)之间发生（这种情况并不经常发生）。
### Solidity 函数
虽然函数的内部实现可能会发生变化，但它们的语义和签名将保持不变。他们的论点的范围不会减少限制（例如，如果不允许传递 0 值，它将保持不允许），也不会取消一般状态限制（例如 `whenPaused` 修饰符）。

如果将新功能添加到合约中，它将以向后兼容的方式：它们的使用不是强制性的，并且它们不会以可能会破坏应用程序的方式扩展功能（例如，[internal](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1512) 可以将方法添加到更容易检索已经可用的信息）。

- `internal`

	这不仅扩展到 `external` 和 `public` 函数，还扩展到函数 `internal`：许多合约旨在通过继承它们来使用（例如 `Pausable、PullPayment、AccessControl`），因此通过调用这些函数来使用。类似地，由于所有 OpenZeppelin Contracts 状态变量都是 `private`，它们只能通过这种方式访问​​（例如，创建新 `ERC20` token，而不是手动修改 `totalSupply` ,`balances`和 `_mint` 调用）。

	`private` 函数对其行为、使用或存在没有任何保证。

最后，有时语言限制会迫使我们例如制作 `internal` 一个应该以 `private` 我们想要的方式实现功能的功能。这些案例将有据可查，正常的稳定性保证将不适用。

#### Libraries(库)
我们的一些 Solidity 库使用 `structs` 来处理不应直接访问的内部数据（例如 `Counter`）。Solidity 存储库中有一个[未解决](https://github.com/ethereum/solidity/issues/4637)的问题，要求提供一种语言特性来阻止上述访问，但看起来它不会很快实现。因此，我们将使用前导下划线并标记所述  `structs` 以向用户明确其内容和布局不是API 的一部分。
#### Events(事件)
不会删除任何事件，并且不会以任何方式更改它们的参数。新事件可能会在以后的版本中添加，现有事件可能会在新的、合理的情况下发出（例如，[从 2.1 开始](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/707)，`ERC20` 也会 `Approval` 在 `transferFrom` 调用时发出）。
#### Draft(草稿)
一些合约实现的 EIP 仍处于 Draft 状态，可以通过以 `draft-` 开头的文件名来识别 ，例如 `utils/cryptography/draft-EIP712.sol.` 由于它们是 Draft 的性质，这些合约的细节可能会发生变化，我们不能保证它们的稳定性。OpenZeppelin 合约的次要版本可能包含标记为 Draft 的合约的重大更改，这些更改将在[更改日志](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/CHANGELOG.md)中正式公布。包含的 EIP 被生产中的项目使用，这可能会降低它们发生重大变化的可能性。
#### GAS 成本
虽然通常会尝试降低使用 OpenZeppelin Contracts 的 gas 成本，但对此没有任何保证。特别是用户不应假设升级库版本时不会增加 gas 成本。
#### Bug 修复
为了修复错误，可能需要打破 API 稳定性保证，我们会这样做。然而这个决定不会轻易做出，并且将探索所有选项以使更改尽可能不中断。当足够时，可能导致不安全行为的合约或函数将被弃用而不是被删除（例如[#1543](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1543)和[#1550](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1550)）。
#### Solidity 编译器版本
从 0.5.0 版本开始，Solidity 团队切换到更快的发布周期，每隔几周发布一次小版本（v0.5.0 于 2018 年 11 月发布，v0.5.5 于 2019 年 3 月发布），并且每隔几周发布一次重大的重大变更版本几个月（v0.6.0 于 2019 年 12 月发布，v0.7.0 于 2020 年 7 月发布）。因此，在 OpenZeppelin Contract 的稳定性保证中包含编译器版本将迫使库要么坚持使用旧编译器，要么发布频繁的主要更新以跟上新的 Solidity 版本。

因此，最低要求的 Solidity 编译器版本不是稳定性保证的一部分，用户在使用较新版本的合约时可能需要升级他们的编译器。错误修复仍将向后移植到过去的主要版本，以便当前使用的所有版本都收到这些更新。

您可以阅读[更多](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1498#issuecomment-449191611)关于这背后的基本原理、我们考虑过的其他选择以及我们为什么要走这条路的原因。
## 参考
[https://docs.openzeppelin.com/contracts/4.x/](https://docs.openzeppelin.com/contracts/4.x/)
