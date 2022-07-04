# Openzepplin API-访问控制
## 访问控制
该目录提供了限制谁可以访问合约功能或何时可以访问的方法。

- [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) 

	提供了一个通用的基于角色的访问控制机制。可以创建多个分层角色并将每个角色分配给多个帐户。
- [Ownable](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable)

	是一种更简单的机制，具有可以分配给单个帐户的单个所有者“角色”。这种更简单的机制对于快速测试很有用，但有生产问题的项目可能会超过它。

### 授权
#### Ownable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/access/Ownable.sol)

	import "@openzeppelin/contracts/access/Ownable.sol";
提供基本访问控制机制的合约模块，其中有一个帐户（所有者）可以被授予对特定功能的独占访问权限。

默认情况下，所有者帐户将是部署合约的帐户。稍后可以使用 更改 [transferOwnership](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-transferOwnership-address-)。

该模块通过继承使用。它将使修饰符可用，该修饰符 `onlyOwner` 可应用于您的函数以限制其对所有者的使用。

- 修饰符
	- [onlyOwner()](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-onlyOwner--)

		如果由所有者以外的任何帐户调用，则抛出。
-  功能
	- [constructor()](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-constructor--) 

		内部方法，初始化将部署者设置为初始所有者的合约。
	- [owner()](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-owner--)

		公共方法，返回当前所有者的地址。返回值类型：address
	- [renounceOwnership()](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-renounceOwnership--)

		公共方法，没有所有者的合同。将无法再调用 onlyOwner 函数。只能由当前所有者调用。
			
		放弃所有权将使合同没有所有者，从而删除仅所有者可用的任何功能。
	- [transferOwnership(newOwner)](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-transferOwnership-address-)

		公共方法，将合约的所有权转移到一个新账户 ( newOwner)。只能由当前所有者调用
	- [_transferOwnership(newOwner)](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-_transferOwnership-address-)

		内部方法，将合约的所有权转移到一个新账户 ( newOwner)。内部功能无访问限制。
- 事件
	- [OwnershipTransferred(previousOwner, newOwner)](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable-OwnershipTransferred-address-address-)

		事件

#### IAccessControl [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/access/IAccessControl.sol)

	import "@openzeppelin/contracts/access/IAccessControl.sol";
声明为支持 ERC165 检测的 AccessControl 的外部接口。

- 功能
	- [hasRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-hasRole-bytes32-address-)

		外部方法，`true` 如果 `account` 已被授予，则返回 `role`。
		
		- 返回值类型 bool
	- [getRoleAdmin(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-getRoleAdmin-bytes32-)
		
		外部方法，返回控制 `role` 的管理员角色。见 [grantRole](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-grantRole-bytes32-address-) 和 [revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-revokeRole-bytes32-address-)。要更改角色的管理员，请使用 [AccessControl._setRoleAdmin.](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setRoleAdmin-bytes32-bytes32-) 
		
		- 返回值类型 bytes32
	- [grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-grantRole-bytes32-address-)

		外部方法，授予 `.role_account`。如果 `account` 尚未被授予 `role` ，则发出一个 [RoleGranted](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-) 事件。

		要求：

		- 调用方必须具有 `role` 管理员角色。
	- [revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-revokeRole-bytes32-address-)

		外部方法，从 `role` 撤销 `account`。如果 `account` 已被授予 `role` ，则发出一个 [RoleRevoked](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-) 事件。
		
		要求：

		- 调用方必须具有 `role` 管理员角色。
	- [renounceRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-renounceRole-bytes32-address-)

		外部方法， 取消呼叫帐号的`role`。

		角色通常通过 [grantRoleand](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-grantRole-bytes32-address-) 进行管理 [revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-revokeRole-bytes32-address-)：此功能的目的是为帐户提供一种机制，如果它们受到损害（例如当受信任的设备放错地方时），它们就会失去其特权。

		如果调用帐户已被授予 `role` ，则发出一个 [RoleRevoked](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-) 事件。

		要求：

		- 调用方必须是 `account`。
- 事件
	- [RoleAdminChanged(role, previousAdminRole, newAdminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleAdminChanged-bytes32-bytes32-bytes32-)

		`newAdminRole` 设置为 `role` 管理员角色时发出事件，替换原有事件 `previousAdminRole`

		`DEFAULT_ADMIN_ROLE` 是所有角色的起始管理员，尽管 [RoleAdminChanged](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleAdminChanged-bytes32-bytes32-bytes32-) 没有发出信号。

		从 v3.1 开始可用。
	- [RoleGranted(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-)

		当 `account` 被授予时发出 `role` 事件。

		`sender` 是发起合约调用的账户，一个管理员角色持有者，除非使用 [AccessControl._setupRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setupRole-bytes32-address-).
	- [RoleRevoked(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-)

		`account` 撤销时发出 `role`。

		`sender` 是发起合约调用的帐户： 
		
		- 如果使用 `revokeRole`，它是管理员 role 承载者 
		- 如果使用 `renounceRole`，它是 role 承载者（即 `account`）

#### AccessControl [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/access/AccessControl.sol)
	import "@openzeppelin/contracts/access/AccessControl.sol";
允许子合约实现基于角色的访问控制机制的合同模块。这是一个轻量级版本，它不允许枚举角色成员，除非通过链下方式访问合约事件日志。一些应用程序可能会受益于链上可枚举性，对于这些情况，请参阅 [AccessControlEnumerable](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable)。

角色由其 `bytes32` 标识符引用。这些应该在外部 API 中公开并且是唯一的。实现此目的的最佳方法是使用 `public constant` 哈希摘要：

	bytes32 public constant MY_ROLE = keccak256("MY_ROLE"); 
角色可用于表示一组权限。要限制对函数调用的访问，请使用 [hasRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-hasRole-bytes32-address-)

	function foo() public {
	    require(hasRole(MY_ROLE, msg.sender));
	    ...
	}
角色可以通过 [grantRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-grantRole-bytes32-address-)和 [revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-)函数动态地授予和撤销。每个角色都有一个关联的管理员角色，并且只有具有角色管理员角色的帐户才能调用 grantRole 和 revokeRole 方法。

默认情况下，所有角色的管理员角色都是 `DEFAULT_ADMIN_ROLE`，这意味着只有具有该角色的帐户才能授予或撤销其他角色。可以通过使用  `_setRoleAdmin` 创建更复杂的角色关系

也是它自己的 `DEFAULT_ADMIN_ROLE` 管理员：它有权授予和撤销此角色。应采取额外的预防措施来保护已授予它的帐户。

- 修饰符
	- [onlyRole(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-onlyRole-bytes32-)

		检查帐户是否具有特定角色的修饰符。使用包含所需角色的标准化消息进行还原，还原原因的格式由以下正则表达式给出：

			/^AccessControl: account (0x[0-9a-f]{40}) is missing role (0x[0-9a-f]{64})$/
		从 v4.1 开始可用。
		
		- 返回值类型 bytes32

- 功能
	- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-supportsInterface-bytes4-)
	
		公共方法，见 [IERC165.supportsInterface](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)。
		
		- 返回值类型  bool
	- [hasRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-hasRole-bytes32-address-)

		公共方法，如果 `account` 已被授予`true` ，则返回 `role`。
		
		- 返回值类型  bool
	- [_checkRole(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_checkRole-bytes32-)

		 内部方法，如果缺少 `_msgSender()` ，`role` 则使用标准消息还原。覆盖此函数会更改 [onlyRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-onlyRole-bytes32-) 修饰符的行为。
		
		恢复消息的格式在  [_checkRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_checkRole-bytes32-address-) 中描述。
		
		从 v4.6 开始可用。
	- [_checkRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_checkRole-bytes32-address-)

		内部方法，如果缺少 `account`，`role` 则使用标准消息还原 。还原原因的格式由以下正则表达式给出：
		
			/^AccessControl: account (0x[0-9a-f]{40}) is missing role (0x[0-9a-f]{64})$/
	- [getRoleAdmin(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-getRoleAdmin-bytes32-)
	
		公开方法，返回控制 `role` 的管理员角色。见 [grantRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-grantRole-bytes32-address-)和 [revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-)。

		要更改角色的管理员，请使用 [_setRoleAdmin](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setRoleAdmin-bytes32-bytes32-).
		
		- 返回值类型 bytes32
	- [grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-grantRole-bytes32-address-)

		公开方法，授予. `role_account`,如果 `account` 尚未被授予 `role`，则发出一个 [RoleGranted](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-) 事件。

		- 要求：
			- 调用方必须具有 `role` 管理员角色。
	- [revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-)

		公开方法，从 `account` 撤销 `role`。如果 `account` 已被授予 `role`，则发出一个 [RoleRevoked](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-)事件。

		- 要求：
			- 调用方必须具有role管理员角色。
	- [renounceRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-renounceRole-bytes32-address-)

		公开方法，从调用 `role` 帐户撤消。

		角色通常通过 [grantRoleand](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-grantRole-bytes32-address-) 和 [revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-)进行管理：此功能的目的是为帐户提供一种机制，如果它们受到损害（例如当受信任的设备放错地方时），它们就会失去其特权。

		如果调用 `role` 帐户已被撤销，则发出一个 [RoleRevoked](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-) 事件。

		- 要求
			- 调用方必须是 `account`
	- [_setupRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setupRole-bytes32-address-)
		- 此功能已弃用，取而代之的是 [_grantRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_grantRole-bytes32-address-).
		
		内部方法，授予. `role_account`

		如果 `account` 尚未被授予 `role`，则发出一个 [RoleGranted](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-) 事件。请注意，与 [grantRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-grantRole-bytes32-address-) 不同的是，此函数不对调用帐户执行任何检查。
		
		只有在为系统设置初始角色时，才应从构造函数中调用此函数。以任何其他方式使用此功能都有效地规避了 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl).
	- [_setRoleAdmin(role, adminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setRoleAdmin-bytes32-bytes32-)

		内部方法，设置 `adminRole` 为 `role` 的管理员角色。发出 [RoleAdminChanged](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleAdminChanged-bytes32-bytes32-bytes32-)事件。
	- [_grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_grantRole-bytes32-address-)

		内部方法，授予. `role_account` ,内部功能无访问限制。
	- [_revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_revokeRole-bytes32-address-)
	
		内部方法， 从 `role` 撤销 `account`。内部功能无访问限制。
- 事件
	- 访问控制相同方法
		- [RoleAdminChanged(role, previousAdminRole, newAdminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleAdminChanged-bytes32-bytes32-bytes32-) 
		- [RoleGranted(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-)
		- [RoleRevoked(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-)

#### AccessControlCrossChain  [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/access/AccessControlCrossChain.sol)

	import "@openzeppelin/contracts/access/AccessControlCrossChain.sol";
[AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) 支持跨链访问管理的扩展。对于每个角色，扩展实现了一个等效的“别名”角色，用于限制来自其他链的调用。

例如，如果一个函数 `myFunction`被 `onlyRole(SOME_ROLE)` 保护，并且如果一个地址 `x` 具有角色 `SOME_ROLE`，它就可以直接调用 `myFunction` 。然而，另一条链上同一地址的钱包或合约将无法调用此函数。为了做到这一点，它需要有角色 `_crossChainRoleAlias(SOME_ROLE)`。

需要这种别名来防止多个合约位于不同链上的同一地址但由冲突实体控制。

从 v4.6 开始可用。

- 功能
	- [_checkRole(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlCrossChain-_checkRole-bytes32-)

		内部方法，见 [AccessControl._checkRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_checkRole-bytes32-address-)。
	- [_crossChainRoleAlias(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlCrossChain-_crossChainRoleAlias-bytes32-)

		内部方法， 返回对应的别名角色 `role`。
		
		- 返回值类型 bytes32
	- 跨链启用相同方法
		- [_isCrossChain()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_isCrossChain--)
		- [_crossChainSender()](https://docs.openzeppelin.com/contracts/4.x/api/crosschain#CrossChainEnabled-_crossChainSender--)
	- 访问控制相同方法
		- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-supportsInterface-bytes4-)
		- [hasRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-hasRole-bytes32-address-)
		- [_checkRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_checkRole-bytes32-address-)
		- [getRoleAdmin(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-getRoleAdmin-bytes32-)
		- [grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-grantRole-bytes32-address-)
		- [revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-)
		- [renounceRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-renounceRole-bytes32-address-)
		- [_setupRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setupRole-bytes32-address-)
		- [_setRoleAdmin(role, adminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setRoleAdmin-bytes32-bytes32-)
		- [_grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_grantRole-bytes32-address-)
		- [_revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_revokeRole-bytes32-address-)
- 事件
	- 访问控制相同方法
		- [RoleAdminChanged(role, previousAdminRole, newAdminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleAdminChanged-bytes32-bytes32-bytes32-)
		- [RoleGranted(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-)
		- [RoleRevoked(role, account, sender)	](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-)

#### IAccessControlEnumerable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/access/IAccessControlEnumerable.sol)

	import "@openzeppelin/contracts/access/IAccessControlEnumerable.sol";
声明为支持 ERC165 检测的 `AccessControlEnumerable` 的外部接口

- 功能
	- [getRoleMember(role, index)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControlEnumerable-getRoleMember-bytes32-uint256-)

		外部方法，返回具有 `role` 的帐户之一。`index` 必须是介于 0 和  [getRoleMemberCount](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControlEnumerable-getRoleMemberCount-bytes32-) 之间的值，不包括在内。

		角色持有者没有以任何特定方式排序，它们的顺序可能随时更改。
		
		- 返回值类型 address

		使用 [getRoleMemberand](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControlEnumerable-getRoleMember-bytes32-uint256-)和 [getRoleMemberCount](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControlEnumerable-getRoleMemberCount-bytes32-)时，请确保在同一个块上执行所有查询。有关更多信息，请参阅以下 [论坛帖子](https://forum.openzeppelin.com/t/iterating-over-elements-on-enumerableset-in-openzeppelin-contracts/2296)
	- [getRoleMemberCount(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControlEnumerable-getRoleMemberCount-bytes32-)

		外部方法，返回拥有 `role`. 可以与 [getRoleMember](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControlEnumerable-getRoleMember-bytes32-uint256-) 一起使用来枚举一个角色的所有承载者。
		
		- 数返回值类型 uint256
	- 访问控制相同方法
		- [hasRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-hasRole-bytes32-address-)
		- [getRoleAdmin(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-getRoleAdmin-bytes32-)
		- [grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-grantRole-bytes32-address-)
		- [revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-revokeRole-bytes32-address-)
		- [renounceRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-renounceRole-bytes32-address-)
- 事件
	- 访问控制相同方法
		- [RoleAdminChanged(role, previousAdminRole, newAdminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleAdminChanged-bytes32-bytes32-bytes32-)
		- [RoleGranted(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-)
		- [RoleRevoked(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-)

#### AccessControlEnumerable
	import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
其扩展 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) 允许枚举每个角色的成员

- 功能
	- [supportsInterface(interfaceId)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-supportsInterface-bytes4-)

		公共方法，见 [IERC165.supportsInterface](https://docs.openzeppelin.com/contracts/4.x/api/utils#IERC165-supportsInterface-bytes4-)。
		
		- 返回值类型 bool
	- [getRoleMember(role, index)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-getRoleMember-bytes32-uint256-)

		公共方法，返回具有 `role` 的帐户之一。`index` 必须是介于 0 和 [getRoleMemberCount](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-getRoleMemberCount-bytes32-)之间的值，不包括在内。

		角色持有者没有以任何特定方式排序，它们的顺序可能随时更改。
		
		使用 [getRoleMemberand](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-getRoleMember-bytes32-uint256-)和[getRoleMemberCount](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-getRoleMemberCount-bytes32-)时，请确保在同一个块上执行所有查询。有关更多信息，请参阅以下 论坛帖子 。

		- 返回值类型 adderss
	- [getRoleMemberCount(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-getRoleMemberCount-bytes32-)

		公共方法，返回拥有 role . 可以与 [getRoleMember](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-getRoleMember-bytes32-uint256-) 一起使用来枚举一个角色的所有承载者。.
		
		- 返回值类型 uint256
	- [_grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-_grantRole-bytes32-address-)

		内部方法，重载 [_grantRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-_grantRole-bytes32-address-) 以跟踪可枚举的成员资格
	- [_revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-_revokeRole-bytes32-address-)

		内部方法，重载 [_revokeRole](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlEnumerable-_revokeRole-bytes32-address-) 以跟踪可枚举的成员资格
	- 访问控制相同方法
		- [hasRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-hasRole-bytes32-address-)
		- [_checkRole(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_checkRole-bytes32-)
		- [_checkRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_checkRole-bytes32-address-)
		- [getRoleAdmin(role)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-getRoleAdmin-bytes32-)
		- [grantRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-grantRole-bytes32-address-)
		- [revokeRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-revokeRole-bytes32-address-)
		- [renounceRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-renounceRole-bytes32-address-)
		- [_setupRole(role, account)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setupRole-bytes32-address-)
		- [_setRoleAdmin(role, adminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setRoleAdmin-bytes32-bytes32-)
- 事件
	- 访问控制相同方法
		- [RoleAdminChanged(role, previousAdminRole, newAdminRole)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleAdminChanged-bytes32-bytes32-bytes32-)
		- [RoleGranted(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleGranted-bytes32-address-address-)
		- [RoleRevoked(role, account, sender)](https://docs.openzeppelin.com/contracts/4.x/api/access#IAccessControl-RoleRevoked-bytes32-address-address-)
		
		