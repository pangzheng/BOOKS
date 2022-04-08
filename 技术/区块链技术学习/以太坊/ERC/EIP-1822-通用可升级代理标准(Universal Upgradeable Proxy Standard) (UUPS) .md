# EIP-1822:通用可升级代理标准(Universal Upgradeable Proxy Standard) (UUPS) 
## 简单总结
标准可升级代理合约。
## 抽象
下面描述一个代理合约的标准，它与所有合约普遍兼容，并且不会造成代理合约和业务逻辑合约之间的不兼容。这是通过利用代理合约中的唯一存储位置来存储逻辑合约的地址来实现的。兼容性检查可确保成功升级。升级可以无限次执行，也可以由自定义逻辑确定。此外，提供了一种从多个构造函数中进行选择的方法，该方法不抑制验证字节码的能力。
## 动机
- 改进现有的代理实现，以改善开发人员部署和维护代理和逻辑合约的体验。
- 规范和改进验证代理合约使用的字节码的方法。
## 术语
- `delegatecall()`

	合约A中的函数，允许外部合约B（委托）修改A的存储（见下图，[Solidity 文档](https://solidity.readthedocs.io/en/v0.5.3/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries)）
- `Proxy Contract` 代理合约

	合约A存储数据，但使用外部合约 B 的逻辑 `delegatecall()`。
- `Logic Contract `逻辑合约

	合约B包含代理合约A使用的逻辑
- `Proxiable Contract` 可委托合同

	继承于 Logic Contract B以提供升级功能
	
![](./pic/proxy-diagram.png)	

## 规格
此处提出的代理合约应按原样部署，并用作任何现有合约生命周期管理方法的直接替代品。除了代理合约之外，我们还提出了 Proxiable Contract 接口/基础，它为升级建立了一个不干扰现有业务规则的模式。允许升级的逻辑可以根据需要实现。
### 代理合约
- 职能
	- `fallback`

		提议的回退函数遵循其他代理合约实现中常见的模式，例如 [Zeppelin](https://github.com/maraoz/solidity-proxy/blob/master/contracts/Dispatcher.sol) 或 [Gnosis](https://blog.gnosis.pm/solidity-delegateproxy-contracts-e09957d0f201)。
		
		但是，不是强制使用变量，而是将逻辑合约的地址存储在定义的存储位置 `keccak256("PROXIABLE")`。这消除了代理和逻辑合约中的变量之间发生冲突的可能性，从而提供了与任何逻辑合约的“通用”兼容性。
		
			function() external payable {
			    assembly { // solium-disable-line
			        let contractLogic := sload(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7)
			        calldatacopy(0x0, 0x0, calldatasize)
			        let success := delegatecall(sub(gas, 10000), contractLogic, 0x0, calldatasize, 0, 0)
			        let retSz := returndatasize
			        returndatacopy(0, 0, retSz)
			        switch success
			        case 0 {
			            revert(0, retSz)
			        }
			        default {
			            return(0, retSz)
			        }
			    }
			}
	- `constructor`

		提议的构造函数接受任意数量的任何类型的参数，因此与任何逻辑合约构造函数兼容。
		
		此外，代理合约构造函数的任意性质提供了从逻辑合约源代码中可用的一个或多个构造函数中进行选择的能力（例如，、、`constructor1`……`constructor2`等）。请注意，如果逻辑合约中包含多个构造函数，则应包含一个检查以禁止在初始化后再次调用构造函数。
		
		值得注意的是，支持多个构造函数的新增功能不会抑制对 Proxy Contract 字节码的验证，因为初始化 tx 调用数据（输入）可以先使用 Proxy Contract ABI，然后使用 Logic Contract ABI 进行解码。
		
		下面的合同显示了代理合同的拟议实施。
		
			contract Proxy {
			    // Code position in storage is keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"
			    constructor(bytes memory constructData, address contractLogic) public {
			        // save the code address
			        assembly { // solium-disable-line
			            sstore(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7, contractLogic)
			        }
			        (bool success, bytes memory _ ) = contractLogic.delegatecall(constructData); // solium-disable-line
			        require(success, "Construction failed");
			    }
			
			    function() external payable {
			        assembly { // solium-disable-line
			            let contractLogic := sload(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7)
			            calldatacopy(0x0, 0x0, calldatasize)
			            let success := delegatecall(sub(gas, 10000), contractLogic, 0x0, calldatasize, 0, 0)
			            let retSz := returndatasize
			            returndatacopy(0, 0, retSz)
			            switch success
			            case 0 {
			                revert(0, retSz)
			            }
			            default {
			                return(0, retSz)
			            }
			        }
			    }
			}

### 可委托合同
Proxiable Contract 包含在 Logic Contract 中，并提供执行升级所需的功能。兼容性检查 `proxiable` 可防止升级期间出现无法修复的更新。

	:warning: 警告：updateCodeAddress 并且 proxiable 必须存在于逻辑合约中。未能包含这些可能会阻止升级，并可能使代理合约变得完全无法使用。见下文限制危险功能

- 职能
	- `proxiable`

		兼容性检查以确保新的逻辑合约实现通用可升级代理标准。请注意，为了支持未来的实现，bytes32可以更改比较，例如 `keccak256("PROXIABLE-ERC1822-v1").`
	- `updateCodeAddress`

		将逻辑合约的地址存储 `keccak256("PROXIABLE")` 在代理合约中。

下面的合同显示了 `Proxiable` 合同的拟议实施。

	contract Proxiable {
	    // Code position in storage is keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"
	    // 存储中的代码位置是 keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"
	
	    function updateCodeAddress(address newAddress) internal {
	        require(
	            bytes32(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7) == Proxiable(newAddress).proxiableUUID(),
	            "Not compatible"
	        );
	        assembly { // solium-disable-line
	            sstore(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7, newAddress)
	        }
	    }
	    function proxiableUUID() public pure returns (bytes32) {
	        return 0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7;
	    }
	}

## 使用代理时的陷阱
使用代理合约时，应为所有逻辑合约采用以下常见的最佳实践。
### 从逻辑中分离变量
在设计新的逻辑合约时应仔细考虑，以防止升级后与代理合约的现有存储不兼容。

- 具体来说，不应修改新合约中变量实例化的顺序，任何新变量都应添加到前一个逻辑合约的所有现有变量之后

为了促进这种做法，我们建议使用一个包含所有变量的单一“基础”合约，并在后续逻辑合约中继承。这种做法大大减少了意外重新排序变量或在存储中覆盖它们的机会。

### 限制危险功能
Proxiable Contract 中的兼容性检查是一种安全机制，可防止升级到未实现通用可升级代理标准的逻辑合约。然而，正如在平价钱包黑客事件中发生的那样，仍然有可能对逻辑合约本身造成无法弥补的损害。

为了防止损坏逻辑合约，我们建议将任何可能具有破坏性的功能的权限限制为 onlyOwner，并在部署到空地址（例如，地址（1））后立即放弃逻辑合约的所有权。具有潜在破坏性的函数包括本机函数，例如`SELFDESTRUCT`，以及其代码可能源自外部的函数，例如 `CALLCODE` 和 `delegatecall()`。在下面的 ERC-20 Token 示例中，使用 `LibraryLock` 合约来防止逻辑合约被破坏。

## 例子
### Owned
在此示例中，展示了标准所有权示例，并将仅限 `updateCodeAddress` 于所有者。

	contract Owned is Proxiable {
	    // 确保一旦部署此合约，没有人可以操纵它
	    address public owner = address(1);
	
	    function constructor1() public{
	        // 确保每个*代理*合约部署时只能调用一次
	        require(owner == address(0));
	        owner = msg.sender;
	    }
	
	    function updateCode(address newCode) onlyOwner public {
	        updateCodeAddress(newCode);
	    }
	
	    modifier onlyOwner() {
	        require(msg.sender == owner, "Only owner is allowed to perform this action");
	        _;
	    }
	}

### ERC-20 代币
- 代理合约

		pragma solidity ^0.5.1;
		
		contract Proxy {
		    // Code position in storage is keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"
		    // 存储中的代码位置是 keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"
		    constructor(bytes memory constructData, address contractLogic) public {
		        // save the code address
		        // 保存代码地址
		        assembly { // solium-disable-line
		            sstore(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7, contractLogic)
		        }
		        (bool success, bytes memory _ ) = contractLogic.delegatecall(constructData); // solium-disable-line
		        require(success, "Construction failed");
		    }
		
		    function() external payable {
		        assembly { // solium-disable-line
		            let contractLogic := sload(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7)
		            calldatacopy(0x0, 0x0, calldatasize)
		            let success := delegatecall(sub(gas, 10000), contractLogic, 0x0, calldatasize, 0, 0)
		            let retSz := returndatasize
		            returndatacopy(0, 0, retSz)
		            switch success
		            case 0 {
		                revert(0, retSz)
		            }
		            default {
		                return(0, retSz)
		            }
		        }
		    }
		}
- 通证逻辑合约

		contract Proxiable {
		    // Code position in storage is keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"
		    // 存储中的代码位置是 keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"
		    
		    function updateCodeAddress(address newAddress) internal {
		        require(
		            bytes32(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7) == Proxiable(newAddress).proxiableUUID(),
		            "Not compatible"
		        );
		        assembly { // solium-disable-line
		            sstore(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7, newAddress)
		        }
		    }
		    function proxiableUUID() public pure returns (bytes32) {
		        return 0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7;
		    }
		}
		
		
		contract Owned {
		
		    address owner;
		
		    function setOwner(address _owner) internal {
		        owner = _owner;
		    }
		    modifier onlyOwner() {
		        require(msg.sender == owner, "Only owner is allowed to perform this action");
		        _;
		    }
		}
		
		contract LibraryLockDataLayout {
		  bool public initialized = false;
		}
		
		contract LibraryLock is LibraryLockDataLayout {
		    // Ensures no one can manipulate the Logic Contract once it is deployed.
		    //确保没有人可以操作逻辑契约一旦它被部署。
		    // PARITY WALLET HACK PREVENTION
		    //校验钱包黑客预防
		
		    modifier delegatedOnly() {
		        require(initialized == true, "The library is locked. No direct 'call' is allowed");
		        _;
		    }
		    function initialize() internal {
		        initialized = true;
		    }
		}
		
		contact ERC20DataLayout is LibraryLockDataLayout {
		  uint256 public totalSupply;
		  mapping(address=>uint256) public tokens;
		}
		
		contract ERC20 {
		    //  ...
		    function transfer(address to, uint256 amount) public {
		        require(tokens[msg.sender] >= amount, "Not enough funds for transfer");
		        tokens[to] += amount;
		        tokens[msg.sender] -= amount;
		    }
		}
		
		contract MyToken is ERC20DataLayout, ERC20, Owned, Proxiable, LibraryLock {
		
		    function constructor1(uint256 _initialSupply) public {
		        totalSupply = _initialSupply;
		        tokens[msg.sender] = _initialSupply;
		        initialize();
		        setOwner(msg.sender);
		    }
		    function updateCode(address newCode) public onlyOwner delegatedOnly  {
		        updateCodeAddress(newCode);
		    }
		    function transfer(address to, uint256 amount) public delegatedOnly {
		        ERC20.transfer(to, amount);
	    }
	    }		


## 参考
[EIP-1822: Universal Upgradeable Proxy Standard (UUPS) ](https://eips.ethereum.org/EIPS/eip-1822)