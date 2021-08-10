# Solidity-描述
## [Solidity 源文件的布局](https://docs.soliditylang.org/en/develop/layout-of-source-files.html#syntax-and-semantics)
源文件可以包含任意数量的[合同定义](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#contract-structure)、[导入指令](https://docs.soliditylang.org/en/develop/layout-of-source-files.html#import)、 [pragma 指令](https://docs.soliditylang.org/en/develop/layout-of-source-files.html#pragma)和 [struct](https://docs.soliditylang.org/en/develop/types.html#structs)、[enum](https://docs.soliditylang.org/en/develop/types.html#enums)、[function](https://docs.soliditylang.org/en/develop/contracts.html#functions)、[error](https://docs.soliditylang.org/en/develop/contracts.html#errors) 和[常量变量](https://docs.soliditylang.org/en/develop/contracts.html#constants)定义。
### SPDX 许可证标识符(// SPDX-License-Identifier: MIT)
### 编译指示(pragma)
### 导入其他源文件(import)
### 注释
## 合同的结构
Solidity 中的合约类似于面向对象语言中的类。每个合约都可以包含[状态变量](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#structure-state-variables)、[函数](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#structure-functions)、 [函数修饰符](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#structure-function-modifiers)、[事件](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#structure-events)、[错误](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#structure-errors)、[结构类型](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#structure-struct-types)和[枚举类型](https://docs.soliditylang.org/en/develop/structure-of-a-contract.html#structure-enum-types)的声明。此外，合约可以从其他合约继承。

还有称为[库](https://docs.soliditylang.org/en/develop/contracts.html#libraries)和[接口](https://docs.soliditylang.org/en/develop/contracts.html#interfaces)的特殊类型的合约。

关于合同的部分包含比此部分更多的详细信息，用于提供快速概览。

### 状态变量
状态变量是其值永久存储在合约存储中的变量

	// SPDX-License-Identifier: GPL-3.0
	pragma solidity >=0.4.0 <0.9.0;
	
	contract SimpleStorage {
	    uint storedData; // State variable
	    // ...
	}
请参阅[类型](https://docs.soliditylang.org/en/develop/types.html#types)部分了解有效的状态变量类型以及[可见性](https://docs.soliditylang.org/en/develop/contracts.html#visibility-and-getters)和获取器以了解可见性的可能选择。
### 函数
函数是代码的可执行单元。函数通常在合约内定义，但也可以在合约外定义。

	// SPDX-License-Identifier: GPL-3.0
	pragma solidity >0.7.0 <0.9.0;
	
	contract SimpleAuction {
	    function bid() public payable { // Function
	        // ...
	    }
	}
	
	// Helper function defined outside of a contract
	function helper(uint x) pure returns (uint) {
	    return x * 2;
	}
[函数调用](https://docs.soliditylang.org/en/develop/control-structures.html#function-calls)可以在内部或外部发生，并且 对其他合约具有不同级别的可见性。函数接受参数并[返回变量](https://docs.soliditylang.org/en/develop/contracts.html#function-parameters-return-variables)以在它们之间传递参数和值。
### 函数修饰符
函数修饰符可用于以声明方式修改函数的语义（请参阅契约部分中的[函数修饰符](https://docs.soliditylang.org/en/develop/contracts.html#modifiers)）。

重载，即具有相同的修饰符名称和不同的参数，是不可能的。

像函数一样，修饰符可以被覆盖。

	// SPDX-License-Identifier: GPL-3.0
	pragma solidity >=0.4.22 <0.9.0;
	
	contract Purchase {
	    address public seller;
	
	    modifier onlySeller() { // Modifier
	        require(
	            msg.sender == seller,
	            "Only seller can call this."
	        );
	        _;
	    }
	
	    function abort() public view onlySeller { // Modifier usage
	        // ...
	    }
	}
### 事件
事件是与 EVM 日志记录工具的便捷接口。

	// SPDX-License-Identifier: GPL-3.0
	pragma solidity >=0.4.21 <0.9.0;
	
	contract SimpleAuction {
	    event HighestBidIncreased(address bidder, uint amount); // Event
	
	    function bid() public payable {
	        // ...
	        emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
	    }
	}
有关如何声明事件以及如何在 dapp 中使用的信息，请参阅合约中的[事件](https://docs.soliditylang.org/en/develop/contracts.html#events)部分。
### 错误
错误允许您为失败情况定义描述性名称和数据。错误可以在 [revert 语句中使用](https://docs.soliditylang.org/en/develop/control-structures.html#revert-statement)。与字符串描述相比，错误要便宜得多，并且允许您对附加数据进行编码。您可以使用 NatSpec 向用户描述错误。

	// SPDX-License-Identifier: GPL-3.0
	pragma solidity ^0.8.4;
	
	/// Not enough funds for transfer. Requested `requested`,
	/// but only `available` available.
	error NotEnoughFunds(uint requested, uint available);
	
	contract Token {
	    mapping(address => uint) balances;
	    function transfer(address to, uint amount) public {
	        uint balance = balances[msg.sender];
	        if (balance < amount)
	            revert NotEnoughFunds(amount, balance);
	        balances[msg.sender] -= amount;
	        balances[to] += amount;
	        // ...
	    }
	}
### 结构类型
结构是自定义定义的类型，可以对多个变量进行分组（请参阅 类型中的[结构](https://docs.soliditylang.org/en/develop/types.html#structs)部分）// SPDX-License-Identifier: GPL-3.0

	pragma solidity >=0.4.0 <0.9.0;
	
	contract Ballot {
	    struct Voter { // Struct
	        uint weight;
	        bool voted;
	        address delegate;
	        uint vote;
	    }
	}
### 枚举类型
枚举可用于创建具有一组有限“常量值”的自定义类型（请参阅 类型中的[枚举](https://docs.soliditylang.org/en/develop/types.html#enums)部分）

	// SPDX-License-Identifier: GPL-3.0
	pragma solidity >=0.4.0 <0.9.0;
	
	contract Purchase {
	    enum State { Created, Locked, Inactive } // Enum
	}	
## 类型
Solidity 是一种静态类型语言，这意味着需要指定每个变量（状态和本地）的类型。Solidity 提供了几种基本类型，它们可以组合形成复杂类型。

此外，类型可以在包含运算符的表达式中相互交互。有关各种运算符的快速参考，请参阅运算符的[优先顺序](https://docs.soliditylang.org/en/develop/cheatsheet.html#order)。

Solidity 中不存在“未定义”或“空”值的概念，但新声明的变量始终具有依赖于其类型的[默认值](https://docs.soliditylang.org/en/develop/control-structures.html#default-value)。要处理任何意外值，您应该使用 revert 函数来恢复整个事务，或者返回一个元组，其中第二个 `bool` 值表示成功。
### 值类型
以下类型也称为值类型，因为这些类型的变量总是按值传递，即当它们用作函数参数或赋值时总是被复制。

- 布尔值
- 整数
- 比较
- 位操作
- 转变
- 加法、减法和乘法
- 分配

	由于运算结果的类型始终是其中一个操作数的类型，因此整数除法总是产生整数。在 Solidity 中，除法向零舍入。这意味着。`int256(-5) / int256(2) == int256(-2)`
- 模数
- 求幂
- 定点数
- 地址

	地址类型有两种风格，它们基本相同：

	- `address`：持有一个 20 字节的值（以太坊地址的大小）
	- `address payable`: 相同 `address`，但有额外的成员 `transfer` 和 `send`。

	这种区别背后的想法是，您用 `address payableaddress` 将 Ether 发送到一个地址，而不能将普通地址发送到 Ether。

	类型转换：

	允许从 `address payable` 到 `address` 的隐式转换，而从 `address` 到 `address payable` 的转换必须显式通过 `payable(<address>)`

	`address` 允许对 `uint160` 、整型文字 `bytes20` 和合同类型进行显式转换。

	只有类型 `address` 和 `contract-type` 的表达式才能通过显式转换 `payable(...)` 转换为类型 `address payable`。对于合约类型，只有当合约可以接收 Ether 时才允许这种转换，即合约具有接收或应付回退功能。请注意，`payable(0)` 是有效的并且是此规则的例外。
	
	- 笔记

		如果您需要一个类型的 `address`  变量并计划将 Ether 发送给它，则声明其类型为  `address payable` 以使此要求可见。此外，尽量尽早进行这种区分或转换
	- 运算

		`<=, <, ==, !=,>=`和`>`
		
		- 警告				
	
			例如，如果将使用较大字节大小的类型转换 `address` 为 `bytes32`，则将 `address` 被截断。为了减少 0.4.24 版及更高版本的编译器的转换歧义，您必须在转换中明确截断。以 32 字节值为例 `0x111122223333444455556666777788889999AAAABBBBCCCCDDDDEEEEFFFFCCCC`。

			您可以使用 `address(uint160(bytes20(b)))`, 结果为 `0x111122223333444455556666777788889999aAaa`，也可以使用 `address(uint160(uint256(b)))`，结果为 `0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc`。
		- 笔记

			`address` 和 `address payable` 之间的区别是在0.5.0 版中引入的。同样从那个版本开始，合约不是从地址类型派生的，但如果它们具有接收或应付回退功能，仍然可以显式转换为 `address` 或 `address payable`
- 地址成员

	有关地址的所有成员的快速参考，请参阅[地址类型的成员](https://docs.soliditylang.org/en/develop/units-and-global-variables.html#address-related)。

	- `balance` 和 `transfer`
	
		可以使用该属性查询地址的余额，`balance` 并使用以下 `transfer` 函数将 Ether（以 wei 为单位）发送到可支付地址：
	
				address payable x = address(0x123);
				address myAddress = address(this);
				if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
		`transfer` 如果当前合约的余额不够大或者以太币转账被接收账户拒绝，则该函数将失败。该 `transfer` 功能在失败时恢复。
		
		- 笔记
	
		如果 `x` 是一个合约地址，它的代码（更具体地说：它的 [Receive Ether Function](https://docs.soliditylang.org/en/develop/contracts.html#receive-ether-function)如果存在或者它的 [Fallback Function](https://docs.soliditylang.org/en/develop/contracts.html#fallback-function)如果存在）将与 `transfer` 调用一起执行（这是 EVM 的一个特性，无法阻止）。如果该执行耗尽 gas 或以任何方式失败，则以太币转移将被恢复，当前合约将异常停止。
	- `send`

		Send 是的低级对应物 `transfer`。如果执行失败，当前合约不会因异常而停止，而是 `send` 会返回 `false`。
		
		- 警告

			使用时有一些危险 `send`：
			
			- 如果调用堆栈深度为 1024（这总是可以由调用者强制），则传输失败
			- 如果接收者耗尽 gas，它也会失败。

			因此，为了进行安全的以太转账，请始终检查 `send` 的返回值 ，使用 `transfer`甚至更好：使用接收者提取资金的模式。
	- `call，delegatecall` 和 `staticcall`		
		
		为了与不遵守 ABI 的合约交互或者更直接地控制编码，函数调用提供 `call, delegatecall` 和 `staticcall`。它们都采用单个字节内存参数并返回成功条件（作为 `bool`）和返回的数据（`bytes memory`）。函数 `abi.encode, abi.encodePacked,abi.encodeWithSelector` 和` abi.encodeWithSignature `可用于对结构化数据进行编码。
		
		例如
		
			bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
			(bool success, bytes memory returnData) = address(nameReg).call(payload);
			require(success);
		- 警告

			所有这些函数都是低级函数，应谨慎使用。具体来说，任何未知的合约都可能是恶意的，如果你调用它，你会将控制权移交给该合约，该合约又可能会回调到你的合约中，因此请准备好在调用返回时更改状态变量。与其他合约交互的常规方式是在合约对象 `( x.f())` 上调用函数。
		- 笔记

			Solidity 的先前版本允许这些函数接收任意参数，并且还会以 `bytes4` 不同的方式处理第一个类型的参数 。这些边缘情况在 0.5.0 版中被删除。
		
			- 可以使用 gas 改变调整供应的 gas
	
					address(nameReg).call{gas: 1000000}(abi.encodeWithSignature("register(string)", "MyName"));
			- 可以控制提供的以太币值
	
					address(nameReg).call{value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"));
			- 可以组合这些修饰符。他们的顺序无关紧要：
	
					address(nameReg).call{gas: 1000000, value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"))
		类似地，可以使用 `delegatecall`  函数：不同之处在于仅使用给定地址的代码，所有其他方面（存储，余额，...）均取自当前合约。目的 `delegatecall` 是使用存储在另一个合约中的库代码。用户必须确保两个合约中的存储布局都适合要使用的 `delegatecall`。		
		- 笔记

			在 homestead 之前，只有一个有限的变体 `callcode` 可用，它不提供对原始值 `msg.sender` 和 `msg.value` 值的访问。此功能在 0.5.0 版本中被移除。
		
		由于拜占庭 `staticcall` 也可以使用。这与基本 `call` 相同 ，但如果被调用的函数以任何方式修改状态，则会恢复。

		所有这三个功能 `call，delegatecall` 以及 `staticcall` 非常低级别的功能，只能被用作最后的手段，因为他们打破密实的类型安全。

		该 `gas` 选项可用于所有三种方法，而该 `value` 选项仅可用于 `call` .
		
		- 笔记

			最好避免在智能合约代码中依赖硬编码的 gas 值，无论是读取还是写入状态，因为这可能有很多陷阱。此外，未来天然气的使用可能会发生变化
		
			所有合约都可以转换为 `address` 类型，因此可以使用 `address(this).balance` 查询当前合约的余额。
- 合约类型

	每个[合约](https://docs.soliditylang.org/en/develop/contracts.html#contracts)都定义了自己的类型。您可以将合约隐式转换为它们继承的合约。合同可以显式转换为 `address` 类型和从类型转换。
	
	只有当合同类型具有接收或应付回退功能时，才可能与 `address payable` 类型进行显式转换。转换仍然使用 ` address(x) `执行。 如果合同类型没有接收或应付回退功能，则可以使用 `payable(address(x))` 转换为 `address payable`。您可以在有关[地址类型](https://docs.soliditylang.org/en/develop/types.html#address)的部分中找到更多信息。
	
	- 笔记

		0.5.0 版本之前，合同直接从地址类型推导有没有区别 `address`和 `address payable`
	
	如果你声明一个合约类型的局部变量（`MyContract c`），你可以调用该合约的函数。 请注意从相同合同类型的某个地方分配它。

	您还可以实例化合约（这意味着它们是新创建的）。 您可以在“新合同”部分找到更多[详细信息](https://docs.soliditylang.org/en/develop/control-structures.html#creating-contracts)。

	合约的数据表示与 `address` 类型的数据表示相同，并且在 [ABI](https://docs.soliditylang.org/en/develop/abi-spec.html#abi) 中也使用这种类型。

	合约不支持任何运营商。

	合约类型的成员是合约的外部函数，包括标记为 `public` 的任何状态变量。

	对于合约 `C`，您可以使用 `type(C)` 来访问有关合约的[类型信息](https://docs.soliditylang.org/en/develop/units-and-global-variables.html#meta-type)。
- 固定大小的字节数组
- 动态大小的字节数组
- 地址文字

	例如，通过地址校验和测试的十六进制文字 `0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF` 是 `address` 类型。长度在 39 到 41 位之间且未通过校验和测试的十六进制文字会产生错误。您可以预先（对于整数类型）或附加（对于 bytesNN 类型）零以消除错误。

	- 笔记

		混合大小写地址校验和格式在 [EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md) 中定义。
- 有理数和整数字面量
- 字符串文字和类型
- Unicode 文字
- 十六进制文字
- 枚举

	枚举是在 Solidity 中创建用户定义类型的一种方法。它们可以与所有整数类型显式转换，但不允许隐式转换。整数的显式转换在运行时检查该值是否在枚举范围内，否则会导致 Panic 错误。枚举至少需要一个成员，并且其声明时的默认值是第一个成员。枚举不能超过 256 个成员。

	数据表示与 C 中的枚举相同：选项由从0.
	
		// SPDX-License-Identifier: GPL-3.0
		pragma solidity >=0.4.16 <0.9.0;
		
		contract test {
		    enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
		    ActionChoices choice;
		    ActionChoices constant defaultChoice = ActionChoices.GoStraight;
		
		    function setGoStraight() public {
		        choice = ActionChoices.GoStraight;
		    }
		
		    // Since enum types are not part of the ABI, the signature of "getChoice"
		    // will automatically be changed to "getChoice() returns (uint8)"
		    // for all matters external to Solidity.
		    function getChoice() public view returns (ActionChoices) {
		        return choice;
		    }
		
		    function getDefaultChoice() public pure returns (uint) {
		        return uint(defaultChoice);
		    }
		}
	- 笔记

		枚举也可以在文件级别上声明，在合同或库定义之外。
- 函数类型

	函数类型是函数的类型。函数类型的变量可以从函数中赋值，函数类型的函数参数可用于将函数传递给函数调用并从函数调用中返回函数。函数类型有两种风格 
	
	- 内部函数

		内部函数只能在当前合约内部调用（更具体地说，在当前代码单元内部，也包括内部库函数和继承函数），因为它们不能在当前合约的上下文之外执行。调用内部函数是通过跳转到其入口标签来实现的，就像内部调用当前合约的函数一样。
	- 外部函数

		外部函数由地址和函数签名组成，它们可以通过外部函数调用传递和返回。

	函数类型标记如下：							

		function (<parameter types>) {internal|external} [pure|view|payable] [returns (<return types>)]
	与参数类型相反，返回类型不能为空 - 如果函数类型不应返回任何内容，则必须省略整个 `returns (<return types>) ` 部分。
	
	默认情况下，函数类型是内部的，因此 `internal` 可以省略关键字。请注意，这仅适用于函数类型。必须为合约中定义的函数明确指定可见性，它们没有默认值。

	- 转换
	
		函数类型 `A` 是隐式转换为函数类型 `B` 当且仅当它们的参数类型是相同的，它们的返回类型是相同的，它们的内部/外部属性是相同的，并且状态可变性 `A`  比的状态易变性  `B` 更具限制性 。特别是：
		
		- `pure` 函数可以转换为 `view` 和 `non-payable` 函数
		- `view` 函数可以转换为 `non-payable` 函数
		- `payable` 函数可以转换为 `non-payable` 函数
		函数类型之间无法进行其他转换。
		
		关于 `payable` 和的规则 `non-payable` 可能有点混乱，但本质上，
		
		- 如果一个函数是 `payable` ，这意味着它也接受零以太币的支付，所以它也是 `non-payable` 。
		- 另一方面，`non-payable` 函数会拒绝发送给它的 Ether，因此 `non-payable` 函数无法转换为 `payable` 函数。
	
		如果函数类型变量未初始化，调用它会导致 [Panic 错误](https://docs.soliditylang.org/en/develop/control-structures.html#assert-and-require)。如果在使用后调用函数，也会发生同样的 `delete` 情况  。
	
		如果在 Solidity 的上下文之外使用外部函数类型，它们将被视为 `function` 类型，它将地址后跟函数标识符一起编码为单个 `bytes24` 类型。
		
		请注意，当前合约的公共功能既可以用作内部功能，也可以用作外部功能。要 `f` 用作内部函数，只需使用 `f` ，如果要使用其外部形式，请使用 `this.f`.
		
		内部类型的函数可以分配给内部函数类型的变量，而不管它在哪里定义。这包括合同和库的私有、内部和公共功能以及免费功能。另一方面，外部函数类型仅与公共和外部合约函数兼容。库被排除在外，因为它们需要 `adelegatecall` 并为其选择器使用不同的 [ABI 约定](https://docs.soliditylang.org/en/develop/contracts.html#library-selectors)。在接口中声明的函数没有定义，因此指向它们也没有意义。
	- 成员：

		外部（或公共）函数具有以下成员：
		
		- `.address` 返回函数合约的地址。
		- `.selector` 返回[ABI 函数选择器](https://docs.soliditylang.org/en/develop/abi-spec.html#abi-function-selector)
	- 笔记

		外部（或公共）函数用于具有附加成员 `.gas(uint)` 和 `.value(uint)`. 这些在 Solidity 0.6.2 中被弃用，在 Solidity 0.7.0 中 `{gas: ...}`和 `{value: ...}` 被移除。而是使用和分别指定发送给函数的气体量或 wei 量。有关更多信息，请参阅[外部函数调用](https://docs.soliditylang.org/en/develop/control-structures.html#external-function-calls)。
	- 显示如何使用成员的示例：

			// SPDX-License-Identifier: GPL-3.0
			pragma solidity >=0.6.4 <0.9.0;
			
			contract Example {
			    function f() public payable returns (bytes4) {
			        assert(this.f.address == address(this));
			        return this.f.selector;
			    }
			
			    function g() public {
			        this.f{gas: 10, value: 800}();
			    }
			}	
	- 显示如何使用内部函数类型的示例：
	
			// SPDX-License-Identifier: GPL-3.0
			pragma solidity >=0.4.16 <0.9.0;
			
			library ArrayUtils {
			    // internal functions can be used in internal library functions because
			    // they will be part of the same code context
			    function map(uint[] memory self, function (uint) pure returns (uint) f)
			        internal
			        pure
			        returns (uint[] memory r)
			    {
			        r = new uint[](self.length);
			        for (uint i = 0; i < self.length; i++) {
			            r[i] = f(self[i]);
			        }
			    }
			
			    function reduce(
			        uint[] memory self,
			        function (uint, uint) pure returns (uint) f
			    )
			        internal
			        pure
			        returns (uint r)
			    {
			        r = self[0];
			        for (uint i = 1; i < self.length; i++) {
			            r = f(r, self[i]);
			        }
			    }
			
			    function range(uint length) internal pure returns (uint[] memory r) {
			        r = new uint[](length);
			        for (uint i = 0; i < r.length; i++) {
			            r[i] = i;
			        }
			    }
			}

			contract Pyramid {
			    using ArrayUtils for *;
			
			    function pyramid(uint l) public pure returns (uint) {
			        return ArrayUtils.range(l).map(square).reduce(sum);
			    }
			
			    function square(uint x) internal pure returns (uint) {
			        return x * x;
			    }
			
			    function sum(uint x, uint y) internal pure returns (uint) {
			        return x + y;
			    }
			}
	- 另一个使用外部函数类型的例子				

			// SPDX-License-Identifier: GPL-3.0
			pragma solidity >=0.4.22 <0.9.0;
			
			
			contract Oracle {
			    struct Request {
			        bytes data;
			        function(uint) external callback;
			    }
			
			    Request[] private requests;
			    event NewRequest(uint);
			
			    function query(bytes memory data, function(uint) external callback) public {
			        requests.push(Request(data, callback));
			        emit NewRequest(requests.length - 1);
			    }
			
			    function reply(uint requestID, uint response) public {
			        // Here goes the check that the reply comes from a trusted source
			        requests[requestID].callback(response);
			    }
			}
			
			
			contract OracleUser {
			    Oracle constant private ORACLE_CONST = Oracle(address(0x00000000219ab540356cBB839Cbe05303d7705Fa)); // known contract
			    uint private exchangeRate;
			
			    function buySomething() public {
			        ORACLE_CONST.query("USD", this.oracleResponse);
			    }
			
			    function oracleResponse(uint response) public {
			        require(
			            msg.sender == address(ORACLE_CONST),
			            "Only oracle can call this."
			        );
			        exchangeRate = response;
				    }
				}
		- 笔记

			Lambda 或内联函数已计划但尚不支持。

### 引用类型
引用类型的值可以通过多个不同的名称进行修改。将此与值类型进行对比，其中每当使用值类型的变量时都会获得独立的副本。因此，必须比值类型更仔细地处理引用类型。目前，引用类型包括结构、数组和映射。如果使用引用类型，则始终必须显式提供存储该类型的数据区域：

- `memory` （ 其生命周期仅限于外部函数调用）
- `storage`（存储状态变量的位置，生命周期仅限于合约的生命周期）
- `calldata`（包含函数参数的特殊数据位置）

更改数据位置的赋值或类型转换将始终导致自动复制操作，而同一数据位置内的赋值仅在某些情况下复制存储类型。

#### 数据位置
每个引用类型都有一个额外的注释，即“数据位置”，关于它的存储位置。共有三个数据位置： memory、storage和calldata。Calldata 是一个不可修改、非持久性的区域，用于存储函数参数，其行为主要类似于内存。它是外部函数的参数所必需的，但也可用于其他变量。	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
		
						
			
										