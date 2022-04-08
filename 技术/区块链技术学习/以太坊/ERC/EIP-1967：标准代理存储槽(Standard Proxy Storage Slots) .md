# EIP-1967：标准代理存储槽(Standard Proxy Storage Slots) 
标准化代理存储它们委托的逻辑合约的地址以及其他特定于代理的信息的位置。
## 抽象的
委托代理合约被广泛用于可升级性和节省气体。这些代理依赖于称为 using 的逻辑合约（也称为实现合约或主副本）`delegatecall`。这允许代理在代码委托给逻辑合约时保持持久状态（存储和平衡）。

为了避免代理和逻辑合约之间的存储使用冲突，逻辑合约的地址通常保存在一个[特定的存储槽中](https://blog.openzeppelin.com/the-state-of-smart-contract-upgrades/#unstructured-storage)，保证不会被编译器分配。该 EIP 提出了一组标准插槽来存储代理信息。这允许像区块浏览器这样的客户端正确地提取这些信息并将其显示给最终用户，并且逻辑合约可以选择性地对其进行操作。
## 动机
委托代理被广泛使用，作为支持升级和降低部署的 gas 成本的一种手段。这些代理的例子可以在 

- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/4.x/api/proxy)
- [Terminal](https://medium.com/terminaldotco/escape-hatch-proxy-efb681de108d)
- [Gnosis](https://blog.gnosis.pm/solidity-delegateproxy-contracts-e09957d0f201)
- [AragonOS](https://github.com/aragon/aragonOS/blob/dev/contracts/common/DelegateProxy.sol)
- [Melonport](https://github.com/melonproject/melon-mail/blob/782aeff9418ac8cdd80875fd6c400bf96f3b03b3/solidity/contracts/DelegateProxy.sol)
- [Limechain](https://github.com/LimeChain/UpgradeableSolidityContract/blob/14bcabc338130fb2aba2ce8bd27b885305566fce/contracts/Upgradeability/Forwardable.sol)
- [WindingTree](https://github.com/windingtree/upgradeable-token-labs/blob/af3b66096091d8282d5c9c55c33365315d85f3e1/contracts/upgradable/DelegateProxy.sol)
- [Decentraland](https://github.com/decentraland/land/blob/5154046844f6f94a5074e82abe01381e6fd7c39d/contracts/upgradable/DelegateProxy.sol)

等中找到。

但是，由于缺少用于获取代理逻辑地址的通用接口，因此无法构建对这些信息起作用的通用工具。

- 一个典型的例子是区块浏览器。

	在这里，最终用户希望与底层逻辑合约而不是代理本身进行交互。有一种从代理检索逻辑合约地址的通用方法，允许区块浏览器显示逻辑合约的 ABI，而不是代理的 ABI（请参阅[此代理-USDC](https://etherscan.io/token/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48#readProxyContract)。浏览器检查合约在可区分槽中的存储以确定如果它确实是代理，在这种情况下，它会同时显示代理和逻辑合约的信息。
- 另一个例子是逻辑合约，它明确地根据它们被代理的事实采取行动。这允许他们潜在地触发代码更新作为其逻辑的一部分，如通用可升级代理标准 ([EIP1822](https://eips.ethereum.org/EIPS/eip-1822))的情况。公共存储槽允许这些用例独立于正在使用的特定代理实现。

## 规格
选择的存储槽的主要要求是编译器绝不能选择它们来存储任何合约状态变量。否则，逻辑合约在写入自己的变量时可能会无意中覆盖代理上的此信息。

在[合约继承链被线性化之后](https://solidity.readthedocs.io/en/v0.4.21/miscellaneous.html#layout-of-state-variables-in-storage)， Solidity 根据声明它们的顺序将变量映射到存储：第一个变量被分配到第一个插槽，依此类推。动态数组和映射中的值例外，它们存储在键和存储槽串联的哈希中。Solidity 开发团队已确认存储布局将在新版本中保留。Vyper 似乎 [遵循与 Solidity 相同的策略](https://github.com/ethereum/vyper/issues/769)。请注意，用其他语言或直接以汇编语言编写的合约可能会发生冲突。

因此，针对特定于代理的信息的存储槽建议如下。它们的选择方式是为了保证它们不会与编译器分配的状态变量发生冲突，因为它们依赖于不以存储索引开头的字符串的哈希值。此外，`-1` 添加了一个偏移量，因此无法知道哈希的原像，从而进一步降低了可能的攻击机会。

可以根据需要在后续 ERC 中添加更多用于附加信息的插槽。

代理监控对于许多应用程序的安全性至关重要。因此，必须能够跟踪对实施和管理槽的更改。不幸的是，跟踪存储槽的变化并不容易。因此，建议任何改变这些槽的函数也应该发出相应的事件。这包括初始化，从 `0x0` 到第一个非零值。

### 逻辑合约地址
存储槽 `0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc` （以 获得 `bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1)`）。

保存此代理委托给的逻辑合约的地址。如果改为使用信标，则应为空。事件应通知此插槽的更改：

	event Upgraded(address indexed implementation);
### 信标合约地址
存储槽 `0xa3f0ad74e5423aebfd80d3ef4346578335a9a72aeaee59ff6cb3582b35133d50`（以 获得 `bytes32(uint256(keccak256('eip1967.proxy.beacon')) - 1)`）。

保存此代理所依赖的信标合约的地址（回退）。如果直接使用逻辑地址，则应为空，并且仅在逻辑合约槽为空时才应考虑。事件应通知此插槽的更改：

	event BeaconUpgraded(address indexed beacon);
信标用于将多个代理的逻辑地址保存在一个位置，允许通过修改单个存储槽来升级多个代理。信标合约必须实现以下功能：

	function implementation() returns (address)
基于信标的代理合约不使用逻辑合约槽。相反，他们使用信标合约槽来存储他们所连接的信标的地址。为了知道信标代理使用的逻辑合约，客户端应该：

- 读取信标逻辑存储槽的信标地址；
- 调用 `implementation()` 信标合约上的函数。

信标合约上的函数结果 `implementation()` 不应该依赖于调用者（`msg.sender`）。

### 管理员地址
存储槽 `0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103` （以获得 `bytes32(uint256(keccak256('eip1967.proxy.admin')) - 1)`）。

持有允许升级此代理的逻辑合约地址的地址（可选）。事件应通知此插槽的更改：

	event AdminChanged(address previousAdmin, address newAdmin);

## 基本原理
这个 EIP 标准化了逻辑合约地址的存储槽，而不是像 [DelegateProxy (EIP897)](https://eips.ethereum.org/EIPS/eip-897) 那样在代理合约上使用公共方法。这样做的理由是代理不应该向最终用户公开可能与逻辑合约冲突的功能。

请注意，即使在具有不同名称的函数之间也可能发生冲突，因为 ABI 仅依赖于函数选择器的四个字节。这可能导致意外错误，甚至漏洞，其中对代理合约的调用返回与预期不同的值，因为代理拦截了调用并以自己的值进行响应。

来自 Nomic Labs 的 [以太坊代理中的恶意后门](https://medium.com/nomic-labs-blog/malicious-backdoors-in-ethereum-proxies-62629adf3357)：

	Proxy 合约中的任何选择器与实现合约中的一个匹配的函数都将被直接调用，完全跳过实现代码。

	因为函数选择器使用固定数量的字节，所以总是有可能发生冲突。这对于日常开发来说不是一个问题，因为 Solidity 编译器会检测到合约中的选择器冲突，但是当选择器用于交叉合约交互时，这个问题就可以利用了. 冲突可以被滥用来创建一个看似行为良好的合约，实际上隐藏了一个后门。

代理公共功能具有潜在可利用性的事实使得有必要以不同的方式标准化逻辑合约地址。这种方法也被用作[通用可升级代理标准 (EIP1822)](https://eips.ethereum.org/EIPS/eip-1822)的一部分。

## 参考实现
此标准的参考实现可以在 [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.3.0/contracts/proxy/ERC1967/ERC1967Proxy.sol) 存储库中找到。

	/**
	 * @dev This contract implements an upgradeable proxy. It is upgradeable because calls are delegated to an
	 * implementation address that can be changed. This address is stored in storage in the location specified by
	 * https://eips.ethereum.org/EIPS/eip-1967[EIP1967], so that it doesn't conflict with the storage layout of the
	 * implementation behind the proxy.
	 * 
	 * @开发者 这个合约实现了一个可升级的代理。它是可升级的，因为调用被委托给一个可以更改的实现地址。这个地址存储在 https://eips.ethereum.org/EIPS/eip-1967[EIP1967] 指定的存储位置中，这样它就不会与代理后面实现的存储布局冲突。
	 */

	contract ERC1967Proxy is Proxy, ERC1967Upgrade {

	    /**
	     * @dev Initializes the upgradeable proxy with an initial implementation specified by `_logic`.
	     *
	     * If `_data` is nonempty, it's used as data in a delegate call to `_logic`. This will typically be an encoded
	     * function call, and allows initializating the storage of the proxy like a Solidity constructor.
	     * 
	     * @开发者 使用' _logic '指定的初始实现初始化可升级代理。如果' _data '是非空的，它被用作委托调用 '_logic' 的数据。这通常是一个编码的函数调用，并允许像 Solidity 构造函数那样初始化代理的存储。
	     */

	    constructor(address _logic, bytes memory _data) payable {
	        assert(_IMPLEMENTATION_SLOT == bytes32(uint256(keccak256("eip1967.proxy.implementation")) - 1));
	        _upgradeToAndCall(_logic, _data, false);
	    }
	
	    /**
	     * @dev Returns the current implementation address.
	     * 
	     * @ 开发者 返回当前现实地址
	     */
	     
	    function _implementation() internal view virtual override returns (address impl) {
	        return ERC1967Upgrade._getImplementation();
	    }
	}
	
	/**
	 * @dev This abstract contract provides getters and event emitting update functions for
	 * https://eips.ethereum.org/EIPS/eip-1967[EIP1967] slots.
	 * 
	 * @ 开发者 这个抽象合约为 https://eips.ethereum.org/EIPS/eip-1967[EIP1967] 插槽提供了 getter 和事件发送更新函数。
	 */
	  
	abstract contract ERC1967Upgrade {
	 
	    // This is the keccak-256 hash of "eip1967.proxy.rollback" subtracted by 1
	    // 这是“eip1967.proxy.rollback”的 kecak -256 哈希值。减去1
	 
	    bytes32 private constant _ROLLBACK_SLOT = 0x4910fdfa16fed3260ed0e7147f7cc6da11a60208b5b9406d12a635614ffd9143;
	
	    /**
	     * @dev Storage slot with the address of the current implementation.
	     * This is the keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1, and is
	     * validated in the constructor.
	     * 
	     * @开发者 存储槽与地址的当前实现。 这是“eip1967.proxy.implementation”的 kecak -256 哈希值。减去1， 并在构造函数中验证。
	     */
	     
	    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
	
	    /**
	     * @dev Emitted when the implementation is upgraded.
	     * @ 开发者 当实现升级时触发
	     */
	     
	    event Upgraded(address indexed implementation);
	
	    /**
	     * @dev Returns the current implementation address.
	     * @开发者 返回当前实现地址。
	     */
	     
	    function _getImplementation() internal view returns (address) {
	        return StorageSlot.getAddressSlot(_IMPLEMENTATION_SLOT).value;
	    }
	
	    /**
	     * @dev Stores a new address in the EIP1967 implementation slot.
	     * @ 开发者 在EIP1967实现插槽中存储一个新地址
	     */
	     
	    function _setImplementation(address newImplementation) private {
	        require(Address.isContract(newImplementation), "ERC1967: new implementation is not a contract");
	        StorageSlot.getAddressSlot(_IMPLEMENTATION_SLOT).value = newImplementation;
	    }
	
	    /**
	     * @dev Perform implementation upgrade
	     * @ 开发者 执行实现升级
	     * 
	     * Emits an {Upgraded} event.
	     * 发出一个{Upgraded} 事件
	     */
	     
	    function _upgradeTo(address newImplementation) internal {
	        _setImplementation(newImplementation);
	        emit Upgraded(newImplementation);
	    }
	
	    /**
	     * @dev Perform implementation upgrade with additional setup call.
	     * @开发者 通过额外的设置调用执行实现升级。
	     *
	     * Emits an {Upgraded} event.
	     * 发出一个{Upgraded} 事件
	     */
	     
	    function _upgradeToAndCall(
	        address newImplementation,
	        bytes memory data,
	        bool forceCall
	    ) internal {
	        _upgradeTo(newImplementation);
	        if (data.length > 0 || forceCall) {
	            Address.functionDelegateCall(newImplementation, data);
	        }
	    }
	
	    /**
	     * @dev Perform implementation upgrade with security checks for UUPS proxies, and additional setup call.
	     * @开发者 执行实现升级，对 UUPS 代理进行安全检查，并进行额外的设置调用。
	     *
	     * Emits an {Upgraded} event.
	     * 发出一个{Upgraded} 事件
	     */
	     
	    function _upgradeToAndCallSecure(
	        address newImplementation,
	        bytes memory data,
	        bool forceCall
	    ) internal {
	        address oldImplementation = _getImplementation();
	
	        // Initial upgrade and setup call
	        // 初始升级和设置调用
	        
	        _setImplementation(newImplementation);
	        if (data.length > 0 || forceCall) {
	            Address.functionDelegateCall(newImplementation, data);
	        }
	
	        // Perform rollback test if not already in progress
	        // 如果还没有进行回滚测试，请执行回滚测试
	        
	        StorageSlot.BooleanSlot storage rollbackTesting = StorageSlot.getBooleanSlot(_ROLLBACK_SLOT);
	        if (!rollbackTesting.value) {
	            // Trigger rollback using upgradeTo from the new implementation
	            // 从新的实现中使用upgradeTo触发回滚
	      
	            rollbackTesting.value = true;
	            Address.functionDelegateCall(
	                newImplementation,
	                abi.encodeWithSignature("upgradeTo(address)", oldImplementation)
	            );
	            rollbackTesting.value = false;
	            // Check rollback was effective
	            // 检查回滚是否有效
	            
	            require(oldImplementation == _getImplementation(), "ERC1967Upgrade: upgrade breaks further upgrades");
	            // Finally reset to the new implementation and log the upgrade
	            // 最后重置到新的实现并记录升级
	            _upgradeTo(newImplementation);
	        }
	    }
	
	    /**
	     * @dev Storage slot with the admin of the contract.
	     * This is the keccak-256 hash of "eip1967.proxy.admin" subtracted by 1, and is
	     * validated in the constructor.
	     * 
	     * @开发者 存储槽与管理员的合同。这是“eip1967.proxy. Admin”的 kecak-256哈希值。减去1，并在构造函数中验证。
	     */
	     
	    bytes32 internal constant _ADMIN_SLOT = 0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103;
	
	    /**
	     * @dev Emitted when the admin account has changed.
	     * @开发者 当管理帐户更改时触发。
	     */
	     
	    event AdminChanged(address previousAdmin, address newAdmin);
	
	    /**
	     * @dev Returns the current admin.
	     * @ 开发者 返回当前管理员
	     */
	    function _getAdmin() internal view returns (address) {
	        return StorageSlot.getAddressSlot(_ADMIN_SLOT).value;
	    }
	
	    /**
	     * @dev Stores a new address in the EIP1967 admin slot.
	     * @开发者 在EIP1967管理员插槽中存储一个新地址。
	     */
	     
	    function _setAdmin(address newAdmin) private {
	        require(newAdmin != address(0), "ERC1967: new admin is the zero address");
	        StorageSlot.getAddressSlot(_ADMIN_SLOT).value = newAdmin;
	    }
	
	    /**
	     * @dev Changes the admin of the proxy.
	     * @ 开发者 修改代理的管理员
	     *
	     * Emits an {AdminChanged} event.
	     * 发出一个{AdminChanged}事件。
	     */
	     
	    function _changeAdmin(address newAdmin) internal {
	        emit AdminChanged(_getAdmin(), newAdmin);
	        _setAdmin(newAdmin);
	    }
	
	    /**
	     * @dev The storage slot of the UpgradeableBeacon contract which defines the implementation for this proxy.
	     * This is bytes32(uint256(keccak256('eip1967.proxy.beacon')) - 1)) and is validated in the constructor.
	     * @开发者 可升级信标合约的存储槽，它定义了代理的实现。 这是 bytes32(uint256(kecak256 ('eip1967.proxy.beacon')) - 1))，并在构造函数中验证
	     */
	     
	    bytes32 internal constant _BEACON_SLOT = 0xa3f0ad74e5423aebfd80d3ef4346578335a9a72aeaee59ff6cb3582b35133d50;
	
	    /**
	     * @dev Emitted when the beacon is upgraded.
	     * @ 开发者 当信标升级时发出事件。
	     */
	     
	    event BeaconUpgraded(address indexed beacon);
	
	    /**
	     * @dev Returns the current beacon.
	     * @ 开发者 返回当前信标
	     */
	     
	    function _getBeacon() internal view returns (address) {
	        return StorageSlot.getAddressSlot(_BEACON_SLOT).value;
	    }
	
	    /**
	     * @dev Stores a new beacon in the EIP1967 beacon slot.
	     * @开发者 在 EIP1967 信标插槽存储一个新的信标。
	     */
	     
	    function _setBeacon(address newBeacon) private {
	        require(Address.isContract(newBeacon), "ERC1967: new beacon is not a contract");
	        require(
	            Address.isContract(IBeacon(newBeacon).implementation()),
	            "ERC1967: beacon implementation is not a contract"
	        );
	        StorageSlot.getAddressSlot(_BEACON_SLOT).value = newBeacon;
	    }
	
	    /**
	     * @dev Perform beacon upgrade with additional setup call. Note: This upgrades the address of the beacon, it does
	     * not upgrade the implementation contained in the beacon (see {UpgradeableBeacon-_setImplementation} for that).
	     *@开发者 通过额外的设置调用执行信标升级。注意:这将升级信标的地址，但不会升级信标中包含的实现(参见{UpgradeableBeacon-_setImplementation})。
	     * Emits a {BeaconUpgraded} event.
	     * 发出一个{BeaconUpgraded}事件。
	     */
	     
	    function _upgradeBeaconToAndCall(
	        address newBeacon,
	        bytes memory data,
	        bool forceCall
	    ) internal {
	        _setBeacon(newBeacon);
	        emit BeaconUpgraded(newBeacon);
	        if (data.length > 0 || forceCall) {
	            Address.functionDelegateCall(IBeacon(newBeacon).implementation(), data);
	        }
	    }
	}
	
	/**
	 * @dev This abstract contract provides a fallback function that delegates all calls to another contract using the EVM
	 * instruction `delegatecall`. We refer to the second contract as the _implementation_ behind the proxy, and it has to
	 * be specified by overriding the virtual {_implementation} function.
	 *
	 * Additionally, delegation to the implementation can be triggered manually through the {_fallback} function, or to a
	 * different contract through the {_delegate} function.
	 *
	 * The success and return data of the delegated call will be returned back to the caller of the proxy.
	 * 
	 * 这个抽象合约提供了一个回退函数，它使用 EVM 指令 'delegatcall' 将所有调用委托给另一个合约。我们将第二个合约称为代理后面的_implementation_，它必须通过重写虚函数 {_implementation} 来指定。此外，对实现的委托可以通过 {_fallback} 函数手动触发，也可以通过{_delegate} 函数手动触发另一个合约。 被委托调用的成功和返回数据将返回给代理的调用方。
	 */
	 
	abstract contract Proxy {
	
	    /**
	     * @dev Delegates the current call to `implementation`.
	     * @开发者 将当前调用委托给' implementation '。
	     *
	     * This function does not return to its internal call site, it will return directly to the external caller.
	     * 这个函数不会返回到它的内部调用点，它会直接返回到外部调用方。
	     */
	     
	    function _delegate(address implementation) internal virtual {
	        assembly {
	            // Copy msg.data. We take full control of memory in this inline assembly
	            // block because it will not return to Solidity code. We overwrite the
	            // Solidity scratch pad at memory position 0.
	            / / 复制 msg.data。我们完全控制这个内联程序集块中的内存，因为它不会返回到 solididity 代码。我们将在内存位置 0 处覆盖 Solidity 临时存储器。
	            calldatacopy(0, 0, calldatasize())
	
	            // Call the implementation.
	            // out and outsize are 0 because we don't know the size yet.
	            // 调用实现。 Out 和 outsize 都是0，因为我们还不知道大小。
	            
	            let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)
	
	            // Copy the returned data.
	            // 复制返回的数据。
	            returndatacopy(0, 0, returndatasize())
	
	            switch result
	            // delegatecall returns 0 on error.
	            // 委托调用在出错时返回0。
	            case 0 {
	                revert(0, returndatasize())
	            }
	            default {
	                return(0, returndatasize())
	            }
	        }
	    }
	
	    /**
	     * @dev This is a virtual function that should be overridden so it returns the address to which the fallback function
	     * and {_fallback} should delegate.
	     * @ 开发者 这是一个应该被重写的虚函数，因此它返回回退函数和 {_fallback} 应该委托的地址。
	     */
	    function _implementation() internal view virtual returns (address);
	
	    /**
	     * @dev Delegates the current call to the address returned by `_implementation()`.
	     *
	     * This function does not return to its internal call site, it will return directly to the external caller.
	     * @开发者 将当前调用委托给' _implementation() '返回的地址。这个函数不会返回到它的内部调用点，它会直接返回到外部调用方。
	     */
	     
	    function _fallback() internal virtual {
	        _beforeFallback();
	        _delegate(_implementation());
	    }
	
	    /**
	     * @dev Fallback function that delegates calls to the address returned by `_implementation()`. Will run if no other
	     * function in the contract matches the call data.
	     * @开发者 回退函数，委托调用由' _implementation() '返回的地址。如果契约中没有其他函数与调用数据匹配，则将运行。
	     */
	     
	    fallback() external payable virtual {
	        _fallback();
	    }
	
	    /**
	     * @dev Fallback function that delegates calls to the address returned by `_implementation()`. Will run if call data
	     * is empty.
	     * @开发者 回退函数，委托调用由' _implementation() '返回的地址。将在调用数据为空时运行。
	     */
	    receive() external payable virtual {
	        _fallback();
	    }
	
	    /**
	     * @dev Hook that is called before falling back to the implementation. Can happen as part of a manual `_fallback`
	     * call, or as part of the Solidity `fallback` or `receive` functions.
	     *
	     * If overridden should call `super._beforeFallback()`.
	     * @开发者 钩子，在返回到实现之前被调用。可以作为手动' _fallback '调用的一部分发生，也可以作为 solididity ' fallback '或' receive '函数的一部分发生。 如果被重写，应该调用' super._beforeFallback() '。
	     */
	    function _beforeFallback() internal virtual {}
	}

## 安全注意事项
这个 ERC 依赖于这样一个事实，即所选择的存储槽不是由 solidity 编译器分配的。这保证了实现合约不会意外覆盖代理运行所需的任何信息。因此，选择具有高插槽号的位置以避免与编译器分配的插槽发生冲突。此外，选择了没有已知原像的位置，以确保使用恶意制作的密钥写入映射不会覆盖它。

打算修改代理特定信息的逻辑合约必须通过写入特定的存储槽来故意这样做（就像 UUPS 的情况一样）。

## 非标准文章建议
### 函数签名攻击
《函数签名可以理解成 ABI 接口的路由命名》

函数签名攻击的思路：由于 solidity 中识别一个函数，靠的是函数签名(ABI hash 路由名)，而函数签名是函数哈希后的前4个bytes，是非常容易碰撞出来的。在一个独立的 solidity 文件中，编译器自己会去检查所有的 external 和 public 函数是否存在函数签名碰撞，而对于代理模式的合约文件，可能存在代理合约合约中的函数签名与逻辑合约中的函数签名碰撞。而一旦发生这种碰撞，代理合约中的函数就会被直接调用，而不是逻辑合约对应的函数。代理合约函数永远不应该向最终用户公开或与逻辑合约功能冲突的方法。此处计算将 keccak256 结果进行减一再转化为 bytes32 来进一步优化隐藏前像，以防止基于函数签名攻击。

针对该攻击，具体哈希后的前 4 个 bytes 函数签名的攻击思路如下：

- 由于 solidity 中识别一个函数，靠的是函数签名，而函数签名是函数哈希后的前 4 个 bytes，是非常容易碰撞出来的。
- 在一个独立的合约，编译器自己会去检查所有的 external 和 public 函数是否存在函数签名碰撞，但对于代理模式的合约，可能存在代理合约中的函数签名与逻辑合约中的函数签名碰撞。

### 钓鱼开发人员密钥攻击
针对该修改逻辑合约进行攻击已屡见不鲜，在几天前 bzx 就曾因 ["开发方被钓鱼导致私钥被盗"](https://link.zhihu.com/?target=https%3A//bzx.network/blog/prelminary-post-mortem)，进而代理合约被修改并影响到了流动性提供者，攻击者不仅转走了原本合约内的资金，还尝试转移对合约进行了approve的账户的资产。

- 攻击时间线
	- 一名 bZx 开发人员被发送到他的个人计算机上的钓鱼电子邮件，其中包含伪装成合法电子邮件附件的 Word 文档中的恶意宏，然后在他的个人计算机上运行脚本。这导致他的个人助记钱包短语被泄露。
	- 11 月 5 日，美国东部标准时间上午 8:30 左右，发生了以下事件：bZx 收到用户报告，用户余额为负，使用率很高。
	- bZx 确定 BSC 和 Polygon 上存在可疑活动，并将被盗资金追踪到以下钱包地址（见下文）。
	- 涉及 Etherscan 标记的钱包。
	- 通知圈子要求冻结 USDC。
	- 黑客升级了合约，并借用了被盗的 BZRX 代币。
- 采取了以下行动：
	- 联系 Banteg 和 Mudit Gupta 加入我们的作战室。
	- 联系了 Tether 并冻结了黑客钱包中的 USDT。（见下面的地址）
	- 联系了币安，冻结了在 BSC 上被盗的 BZRX，以防止其被转移。
	- 联系 KuCoin 并确定其中一个黑客钱包被用于转入和转出交易所。
	- 禁用 Polygon 和 BSC 上的 UI 以防止用户存款。
	- 联系了USDC，要求冻结黑客钱包中的USDC。
	- 联系 KuCoin 确认黑客 KuCoin 账户 


## 参考
- [EIP-1967: Standard Proxy Storage Slots](https://eips.ethereum.org/EIPS/eip-1967)
- [从EIP-1967谈大火的“鱿鱼游戏SQUID”安全风险](https://zhuanlan.zhihu.com/p/432100665)
- [Preliminary Post Mortem](https://bzx.network/blog/prelminary-post-mortem)

