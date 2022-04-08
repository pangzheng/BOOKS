# EIP-897: 委托代理(DelegateProxy)
## 简单总结
代理合约越来越多地被用作可升级机制和在部署特定合约的许多实例时,节省气体的方法。该标准为代理提出了一组接口，以表明它们如何工作以及它们的主要实现是什么。
## 抽象
使用将自己的逻辑委托给另一个合约的代理正在成为一种越来越流行的技术，用于智能合约可升级性和创建廉价的克隆合约。

鉴于 DelegateProxy 的简单性，我们不认为标准化任何特定的 DelegateProxy 实现有什么价值，但我们相信，就所有代理使用的接口达成一致意见，该接口允许以标准方式操作代理，这是有很大价值的。
## 标准化接口

	interface ERCProxy {
	  function proxyType() public pure returns (uint256 proxyTypeId);
	  function implementation() public view returns (address codeAddr);
	}
### 代码地址 (`implementation()`)
返回的代码地址是代理将在那个时刻为该消息委托调用的地址。

### 代理类型 (`proxyType()`)
检查代理类型是检查合约是否是代理的方法。当合约无法返回此方法或返回 0 时，可以假设该合约不是代理。

它还允许传达有关代理如何运行的更多信息。它是一个纯函数，因此使其有效地保持不变，因为它不能根据状态变化返回不同的值。

- 转发代理（id = 1）
	- 代理将始终转发到相同的代码地址。以下不变量应始终为真：一旦代理返回非零代码地址，该代码地址就永远不会改变。
- 可升级代理（id = 2）
	- 代理代码地址可以根据在代理级别或其转发逻辑中实现的一些任意逻辑进行更改。
	
## 实现
- aragonOS
	- [AppProxyUpgradeable](https://github.com/aragon/aragonOS/blob/master/contracts/apps/AppProxyUpgradeable.sol)
	- [AppProxyPinned](https://github.com/aragon/aragonOS/blob/master/contracts/apps/AppProxyPinned.sol)
	- [KernelProxy](https://github.com/aragon/aragonOS/blob/master/contracts/kernel/KernelProxy.sol)
- zeppelinOS
	- [proxy](https://github.com/zeppelinos/labs/blob/2da9e859db81a61f2449d188e7193788ca721c65/upgradeability_ownership/contracts/Proxy.sol) 

			pragma solidity ^0.4.18;
			
			/**
			 * @title Proxy
			 * @dev Gives the possibility to delegate any call to a foreign implementation.
			 * @开发者 提供将任何调用委托给外部实现的可能性。
			 */
			contract Proxy {
			
			  /**
			  * @dev Tells the address of the implementation where every call will be delegated.
			  * @开发者 告诉每个调用将被委托的实现地址。
			  * 
			  * @return address of the implementation to which it will be delegated
			  * @返回 谁将被授权执行地址
			  */
			  
			  function implementation() public view returns (address);
			
			  /**
			  * @dev Fallback function allowing to perform a delegatecall to the given implementation.
			  * This function will return whatever the implementation call returns
			  * @开发者 回退函数，允许对给定的实现执行委托调用。这个函数将返回实现调用返回的内容
			  */
			  
			  function () payable public {
			    address _impl = implementation();
			    require(_impl != address(0));
			
			    assembly {
			      let ptr := mload(0x40)
			      calldatacopy(ptr, 0, calldatasize)
			      let result := delegatecall(gas, _impl, ptr, calldatasize, 0, 0)
			      let size := returndatasize
			      returndatacopy(ptr, 0, size)
			
			      switch result
			      case 0 { revert(ptr, size) }
			      default { return(ptr, size) }
			    }
			  }
			}

## 好处
- 源代码验证

	现在在像 Etherscan 这样的浏览器中检查代理的代码时，它只显示代理本身的代码，而不是合约的实际代码。通过标准化这个结构，他们将能够显示实际的 ABI 和合约代码。
	
## 参考
- [EIP-897:DelegateProxy](https://eips.ethereum.org/EIPS/eip-897)	