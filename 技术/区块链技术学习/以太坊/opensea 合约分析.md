# opensea 合约分析

	/**
	 *Submitted for verification at Etherscan.io on 2021-03-31
	*/
	
	/**
	 *Submitted for verification at Etherscan.io on 2018-06-12
	*/
	
	pragma solidity ^0.4.13;
	
	contract Ownable {
	  address public owner;
	
	
	  event OwnershipRenounced(address indexed previousOwner);
	  event OwnershipTransferred(
	    address indexed previousOwner,
	    address indexed newOwner
	  );
	
	
	  /**
	   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
	   * account.
	   * @dev Ownable 构造函数将合约的原始 `owner` 设置为发送者 * 帐户。
	   */
	  constructor() public {
	    owner = msg.sender;
	  }
	
	  /**
	   * @dev Throws if called by any account other than the owner.
	   * @dev 如果被所有者以外的任何帐户调用，则抛出。
	   */
	  modifier onlyOwner() {
	    require(msg.sender == owner);
	    _;
	  }
	
	  /**
	   * @dev Allows the current owner to transfer control of the contract to a newOwner.
	   * @param newOwner The address to transfer ownership to.
	   * @dev 允许当前所有者将合同的控制权转移给新所有者。
		* @param newOwner 要转移所有权的地址。
	   */
	  function transferOwnership(address newOwner) public onlyOwner {
	    require(newOwner != address(0));
	    emit OwnershipTransferred(owner, newOwner);
	    owner = newOwner;
	  }
	
	  /**
	   * @dev Allows the current owner to relinquish control of the contract.
	   * @dev 允许当前所有者放弃对合同的控制。
	   */
	  function renounceOwnership() public onlyOwner {
	    emit OwnershipRenounced(owner);
	    owner = address(0);
	  }
	}
	
	
	 /**
	   * ERC 20 相关合约
	   */
	
	contract ERC20Basic {
	  function totalSupply() public view returns (uint256);
	  function balanceOf(address who) public view returns (uint256);
	  function transfer(address to, uint256 value) public returns (bool);
	  event Transfer(address indexed from, address indexed to, uint256 value);
	}
	
	contract ERC20 is ERC20Basic {
	  function allowance(address owner, address spender)
	    public view returns (uint256);
	
	  function transferFrom(address from, address to, uint256 value)
	    public returns (bool);
	
	  function approve(address spender, uint256 value) public returns (bool);
	  event Approval(
	    address indexed owner,
	    address indexed spender,
	    uint256 value
	  );
	}
	
	/**
	   * 代币接收者
	   */
	   
	contract TokenRecipient {
	    event ReceivedEther(address indexed sender, uint amount);
	    event ReceivedTokens(address indexed from, uint256 value, address indexed token, bytes extraData);
	
	    /**
	     * @dev Receive tokens and generate a log event
	     * @param from Address from which to transfer tokens
	     * @param value Amount of tokens to transfer
	     * @param token Address of token
	     * @param extraData Additional data to log
	     	* @dev 接收令牌并生成日志事件
		  	* @param 来自转移代币的地址
			* @param value 要转移的代币数量
			* @param token 令牌地址
			* @param extraData 要记录的附加数据
	     */
	    function receiveApproval(address from, uint256 value, address token, bytes extraData) public {
	        ERC20 t = ERC20(token);
	        require(t.transferFrom(from, this, value));
	        emit ReceivedTokens(from, value, token, extraData);
	    }
	
	    /**
	     * @dev Receive Ether and generate a log event
	     * @dev 接收 Ether 并生成日志事件
	     */
	    function () payable public {
	        emit ReceivedEther(msg.sender, msg.value);
	    }
	}
	
	
	
	contract ProxyRegistry is Ownable {
	
	    /* DelegateProxy implementation contract. Must be initialized. */
	    /* DelegateProxy 实现合约。 必须初始化。 */
	    address public delegateProxyImplementation;
	
	    /* Authenticated proxies by user. */
	    /* 由用户认证的代理。 */
	    mapping(address => OwnableDelegateProxy) public proxies;
	
	    /* Contracts pending access. */
	    /* 合同待访问。 */
	    mapping(address => uint) public pending;
	
	    /* Contracts allowed to call those proxies. */
	    /* 允许调用这些代理的合同。 */
	    mapping(address => bool) public contracts;
	
	    /* Delay period for adding an authenticated contract.
	       This mitigates a particular class of potential attack on the Wyvern DAO (which owns this registry) - if at any point the value of assets held by proxy contracts exceeded the value of half the WYV supply (votes in the DAO),
	       a malicious but rational attacker could buy half the Wyvern and grant themselves access to all the proxy contracts. A delay period renders this attack nonthreatening - given two weeks, if that happened, users would have
	       plenty of time to notice and transfer their assets.
	    */
	    
	    /* 添加经过身份验证的合约的延迟时间。

			为了减轻了对 Wyvern DAO（拥有此注册表）的特定类别的潜在攻击
			
			- 如果代理合约持有的资产价值在任何时候超过了 WYV 供应量的一半（DAO 中的投票）
			一个恶意但理性的攻击者可以购买 Wyvern 的一半并授予自己对所有代理合约的访问权。 
			延迟期使这种攻击没有威胁性——如果发生两周，用户将有足够的时间注意到并转移他们的资产。
		*/
		
	    uint public DELAY_PERIOD = 2 weeks;
	
	    /**
	     * Start the process to enable access for specified contract. Subject to delay period.
	     *
	     * @dev ProxyRegistry owner only
	     * @param addr Address to which to grant permissions
	     */
	     
	     /**
			* 启动进程以启用对指定合同的访问。 以延迟期为准。
			* 仅限 @dev ProxyRegistry 所有者
			* @param addr 授予权限的地址
			*/
			
	    function startGrantAuthentication (address addr)
	        public
	        onlyOwner
	    {
	        require(!contracts[addr] && pending[addr] == 0);
	        pending[addr] = now;
	    }
	
	    /**
	     * End the process to nable access for specified contract after delay period has passed.
	     *
	     * @dev ProxyRegistry owner only
	     * @param addr Address to which to grant permissions
	     */
	     
	     /**
			* 在延迟期过后结束为指定合同启用访问的过程。
			*
			* 仅限 @dev ProxyRegistry 所有者
			* @param addr 授予权限的地址
			*/
	    function endGrantAuthentication (address addr)
	        public
	        onlyOwner
	    {
	        require(!contracts[addr] && pending[addr] != 0 && ((pending[addr] + DELAY_PERIOD) < now));
	        pending[addr] = 0;
	        contracts[addr] = true;
	    }
	
	    /**
	     * Revoke access for specified contract. Can be done instantly.
	     *
	     * @dev ProxyRegistry owner only
	     * @param addr Address of which to revoke permissions
	     */    
	     
	     /**
			* 撤销指定合约的访问权限。 可以立即完成。
			*
			* 仅限 @dev ProxyRegistry 所有者
			* @param addr 撤销权限的地址
			*/
	    function revokeAuthentication (address addr)
	        public
	        onlyOwner
	    {
	        contracts[addr] = false;
	    }
	
	    /**
	     * Register a proxy contract with this registry
	     *
	     * @dev Must be called by the user which the proxy is for, creates a new AuthenticatedProxy
	     * @return New AuthenticatedProxy contract
	     */
	     
	     /**
			* 在此注册中心注册代理合约
			*
			* @dev 必须由代理的用户调用，创建一个新的 AuthenticatedProxy
			* @return 新的 AuthenticatedProxy 合约
			*/
	    function registerProxy()
	        public
	        returns (OwnableDelegateProxy proxy)
	    {
	        require(proxies[msg.sender] == address(0));
	        proxy = new OwnableDelegateProxy(msg.sender, delegateProxyImplementation, abi.encodeWithSignature("initialize(address,address)", msg.sender, address(this)));
	        proxies[msg.sender] = proxy;
	        return proxy;
	    }
	
	}
	
	contract WyvernProxyRegistry is ProxyRegistry {
	
	    string public constant name = "Project Wyvern Proxy Registry";
	
	    /* Whether the initial auth address has been set. */
	    /* 是否设置了初始认证地址。 */
	    
	    bool public initialAddressSet = false;
	
	    constructor ()
	        public
	    {
	        delegateProxyImplementation = new AuthenticatedProxy();
	    }
	
	    /** 
	     * Grant authentication to the initial Exchange protocol contract
	     *
	     * @dev No delay, can only be called once - after that the standard registry process with a delay must be used
	     * @param authAddress Address of the contract to grant authentication
	     */
	     
	     /**
			* 为初始 Exchange 协议合约授予身份验证
			*
			* @dev 无延迟，只能调用一次 - 之后必须使用有延迟的标准注册表进程
			* @param authAddress 授予认证的合约地址
			*/
			
	    function grantInitialAuthentication (address authAddress)
	        onlyOwner
	        public
	    {
	        require(!initialAddressSet);
	        initialAddressSet = true;
	        contracts[authAddress] = true;
	    }
	
	}
	
	contract OwnedUpgradeabilityStorage {
	
	  // Current implementation
	  // 当前实现
	  address internal _implementation;
	
	  // Owner of the contract
	  // 合约的所有者
	  address private _upgradeabilityOwner;
	
	  /**
	   * @dev Tells the address of the owner
	   * @return the address of the owner
	   */
	   /**
		* @dev 告诉所有者的地址
		* @return 所有者的地址
		*/
		
	  function upgradeabilityOwner() public view returns (address) {
	    return _upgradeabilityOwner;
	  }
	
	  /**
	   * @dev Sets the address of the owner
	   * @dev 设置所有者的地址
	   */
	  function setUpgradeabilityOwner(address newUpgradeabilityOwner) internal {
	    _upgradeabilityOwner = newUpgradeabilityOwner;
	  }
	
	  /**
	  * @dev Tells the address of the current implementation
	  * @return address of the current implementation
	  * @dev 告诉当前实现的地址
	  * @return 当前实现的地址
	  */
	  function implementation() public view returns (address) {
	    return _implementation;
	  }
	
	  /**
	  * @dev Tells the proxy type (EIP 897)
	  * @return Proxy type, 2 for forwarding proxy
	  * @dev 告诉代理类型 (EIP 897)
	  * @return 代理类型，2 为转发代理
	  */
	  function proxyType() public pure returns (uint256 proxyTypeId) {
	    return 2;
	  }
	}
	
	contract AuthenticatedProxy is TokenRecipient, OwnedUpgradeabilityStorage {
	
	    /* Whether initialized. */
	    /* 是否初始化。 */
	    bool initialized = false;
	
	    /* Address which owns this proxy. */
	    /* 拥有这个代理的地址。 */
	    address public user;
	
	    /* Associated registry with contract authentication information. */
	    /* 与合同认证信息相关联的注册表。 */
	    ProxyRegistry public registry;
	
	    /* Whether access has been revoked. */
	    /* 访问权限是否已被撤销。 */
	    bool public revoked;
	
	    /* Delegate call could be used to atomically transfer multiple assets owned by the proxy contract with one order. */
	    /* 委托调用可用于通过一个订单原子地转移代理合约拥有的多个资产。 */
	    enum HowToCall { Call, DelegateCall }
	
	    /* Event fired when the proxy access is revoked or unrevoked. */
	    /* 当代理访问被撤销或未撤销时触发的事件。 */
	    event Revoked(bool revoked);
	
	    /**
	     * Initialize an AuthenticatedProxy
	     *
	     * @param addrUser Address of user on whose behalf this proxy will act
	     * @param addrRegistry Address of ProxyRegistry contract which will manage this proxy
	     */
	     
	     /**
			* 初始化一个 AuthenticatedProxy
			*
			* @param addrUser 此代理将代表其行事的用户地址
			* @param addrRegistry 将管理此代理的 ProxyRegistry 合约的地址
			*/
	    function initialize (address addrUser, ProxyRegistry addrRegistry)
	        public
	    {
	        require(!initialized);
	        initialized = true;
	        user = addrUser;
	        registry = addrRegistry;
	    }
	
	    /**
	     * Set the revoked flag (allows a user to revoke ProxyRegistry access)
	     *
	     * @dev Can be called by the user only
	     * @param revoke Whether or not to revoke access
	     */
	     /**
			* 设置撤销标志（允许用户撤销 ProxyRegistry 访问）
			*
			* @dev 只能被用户调用
			* @param revoke 是否撤销访问
			*/
	    function setRevoke(bool revoke)
	        public
	    {
	        require(msg.sender == user);
	        revoked = revoke;
	        emit Revoked(revoke);
	    }
	
	    /**
	     * Execute a message call from the proxy contract
	     *
	     * @dev Can be called by the user, or by a contract authorized by the registry as long as the user has not revoked access
	     * @param dest Address to which the call will be sent
	     * @param howToCall Which kind of call to make
	     * @param calldata Calldata to send
	     * @return Result of the call (success or failure)
	     */
	     /**
			* 从代理合约执行消息调用
			*
			* @dev 可以被用户调用，也可以被注册中心授权的合约调用，只要用户没有撤销访问
			* @param dest 呼叫将被发送到的地址
			* @param howToCall 要拨打什么调用
			* @param calldata 要发送的调用数据
			* @return 调用结果（成功或失败）
			*/
			
	    function proxy(address dest, HowToCall howToCall, bytes calldata)
	        public
	        returns (bool result)
	    {
	        require(msg.sender == user || (!revoked && registry.contracts(msg.sender)));
	        if (howToCall == HowToCall.Call) {
	            result = dest.call(calldata);
	        } else if (howToCall == HowToCall.DelegateCall) {
	            result = dest.delegatecall(calldata);
	        }
	        return result;
	    }
	
	    /**
	     * Execute a message call and assert success
	     * 
	     * @dev Same functionality as `proxy`, just asserts the return value
	     * @param dest Address to which the call will be sent
	     * @param howToCall What kind of call to make
	     * @param calldata Calldata to send
	     */
	     /**
			* 执行消息调用并断言成功
			*
			* @dev 与 `proxy` 功能相同，只是断言返回值
			* @param dest 呼叫将被发送到的地址
			* @param howToCall 要拨打什么样的电话
			* @param calldata 要发送的调用数据
			*/
			
	    function proxyAssert(address dest, HowToCall howToCall, bytes calldata)
	        public
	    {
	        require(proxy(dest, howToCall, calldata));
	    }
	
	}
	
	contract Proxy {
	
	  /**
	  * @dev Tells the address of the implementation where every call will be delegated.
	  * @return address of the implementation to which it will be delegated
	  */
	  /**
		* @dev 告诉每个调用将被委托的实现地址。
		* @return 将被委派到的实现的地址
		*/
	  function implementation() public view returns (address);
	
	  /**
	  * @dev Tells the type of proxy (EIP 897)
	  * @return Type of proxy, 2 for upgradeable proxy
	  */
	  /**
		* @dev 告诉代理的类型（EIP 897）
		* @return 代理类型，2 为可升级代理
		*/
	  function proxyType() public pure returns (uint256 proxyTypeId);
	
	  /**
	  * @dev Fallback function allowing to perform a delegatecall to the given implementation.
	  * This function will return whatever the implementation call returns
	  */
	  /**
		* @dev Fallback 函数允许对给定的实现执行委托调用。
		* 此函数将返回任何实现调用返回
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
	
	
	contract OwnedUpgradeabilityProxy is Proxy, OwnedUpgradeabilityStorage {
	  /**
	  * @dev Event to show ownership has been transferred
	  * @param previousOwner representing the address of the previous owner
	  * @param newOwner representing the address of the new owner
	  */
	  /**
		* @dev 事件显示所有权已转移
		* @param previousOwner 代表前任所有者的地址
		* @param newOwner 代表新所有者的地址
		*/
	  event ProxyOwnershipTransferred(address previousOwner, address newOwner);
	
	  /**
	  * @dev This event will be emitted every time the implementation gets upgraded
	  * @param implementation representing the address of the upgraded implementation
	  */
	  /**
		* @dev 每次实现升级时都会发出此事件
		* @param implementation 表示升级实现的地址
		*/
	  event Upgraded(address indexed implementation);
	
	  /**
	  * @dev Upgrades the implementation address
	  * @param implementation representing the address of the new implementation to be set
	  */
	  /**
		* @dev 升级实现地址
		* @param implementation 表示要设置的新实现的地址
		*/
		
	  function _upgradeTo(address implementation) internal {
	    require(_implementation != implementation);
	    _implementation = implementation;
	    emit Upgraded(implementation);
	  }
	
	  /**
	  * @dev Throws if called by any account other than the owner.
	  */
	  /**
		* @dev 如果被所有者以外的任何帐户调用，则抛出。
		*/
	  modifier onlyProxyOwner() {
	    require(msg.sender == proxyOwner());
	    _;
	  }
	
	  /**
	   * @dev Tells the address of the proxy owner
	   * @return the address of the proxy owner
	   */
	   /**
			* @dev 告诉代理所有者的地址
			* @return 代理所有者的地址
			*/
	  function proxyOwner() public view returns (address) {
	    return upgradeabilityOwner();
	  }
	
	  /**
	   * @dev Allows the current owner to transfer control of the contract to a newOwner.
	   * @param newOwner The address to transfer ownership to.
	   */
	   /**
		* @dev 允许当前所有者将合同的控制权转移给新所有者。
		* @param newOwner 要转移所有权的地址。
		*/
	  function transferProxyOwnership(address newOwner) public onlyProxyOwner {
	    require(newOwner != address(0));
	    emit ProxyOwnershipTransferred(proxyOwner(), newOwner);
	    setUpgradeabilityOwner(newOwner);
	  }
	
	  /**
	   * @dev Allows the upgradeability owner to upgrade the current implementation of the proxy.
	   * @param implementation representing the address of the new implementation to be set.
	   */
	   **
		* @dev 允许可升级所有者升级代理的当前实现。
		* @param implementation 表示要设置的新实现的地址。
		*/
	  function upgradeTo(address implementation) public onlyProxyOwner {
	    _upgradeTo(implementation);
	  }
	
	  /**
	   * @dev Allows the upgradeability owner to upgrade the current implementation of the proxy
	   * and delegatecall the new implementation for initialization.
	   * @param implementation representing the address of the new implementation to be set.
	   * @param data represents the msg.data to bet sent in the low level call. This parameter may include the function
	   * signature of the implementation to be called with the needed payload
	   */
	   /**
		* @dev 允许可升级性所有者升级代理的当前实现并委托调用新实现进行初始化。
		* @param implementation 表示要设置的新实现的地址。
		* @param data 代表要在低级调用中发送的 msg.data。 此参数可能包括要使用所需负载调用的实现的函数签名
		*/
	  function upgradeToAndCall(address implementation, bytes data) payable public onlyProxyOwner {
	    upgradeTo(implementation);
	    require(address(this).delegatecall(data));
	  }
	}
	
	contract OwnableDelegateProxy is OwnedUpgradeabilityProxy {
	
	    constructor(address owner, address initialImplementation, bytes calldata)
	        public
	    {
	        setUpgradeabilityOwner(owner);
	        _upgradeTo(initialImplementation);
	        require(initialImplementation.delegatecall(calldata));
	    }
	
	}