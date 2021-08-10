# Solidity-Base
## 智能合约简介
Solidity 意义上的合约是驻留在以太坊区块链上特定地址的代码（其功能）和数据（其状态）的集合。
### 一个简单的智能合约
让我们从一个基本示例开始，该示例设置变量的值并将其公开以供其他合约访问。如果您现在不了解所有内容也没关系，我们稍后会详细介绍。

例

	// SPDX-License-Identifier: GPL-3.0 //第一行告诉源代码是根据 GPL 3.0 版许可的。在发布源代码是默认设置的设置中，机器可读的许可证说明符很重要。
	pragma solidity >=0.4.16 <0.9.0; //指定源代码是为 Solidity 0.4.16 版或更高版本的语言编写的，但不包括 0.9.0 版。这是为了确保合约无法使用新的（破坏性的）编译器版本进行编译，因为它的行为可能会有所不同。 Pragma(https://docs.soliditylang.org/en/develop/layout-of-source-files.html#pragma) 是编译器关于如何处理源代码的常用指令（例如pragma once）。
	
	contract SimpleStorage {
	    uint storedData; //声明了一个名为 storedData 的状态变量，其类型为 uint（256 位无符号整数）。可以将其视为数据库中的单个插槽，可以通过调用管理数据库的代码的函数来查询和更改它。
	
	    // 该合约定义了可用于修改或检索变量值的 set 和 get 函数 
	    function set(uint x) public {
	        storedData = x;
	    }
	
	    function get() public view returns (uint) {
	        return storedData;
	    }
	}


要访问当前合约的成员（如状态变量），您通常不添加 `this.` 前缀，只需直接通过其名称访问它。与其他一些语言不同，省略它不仅仅是风格问题，它会导致访问成员的方式完全不同，稍后会详细介绍。

除了（由于以太坊构建的基础设施）允许任何人存储世界上任何人都可以访问的单个号码之外，该合约并没有做太多事情，而没有（可行的）方法来阻止您发布该号码。任何人都可以 `set` 使用不同的值再次呼叫并覆盖您的号码，但该号码仍存储在区块链的历史记录中。稍后，您将看到如何施加访问限制，以便只有您可以更改号码。

- 警告

	使用 Unicode 文本时要小心，因为外观相似（甚至相同）的字符可能具有不同的代码点，因此被编码为不同的字节数组。
- 笔记

	所有标识符（合约名称、函数名称和变量名称）都限于 ASCII 字符集。可以将 UTF-8 编码的数据存储在字符串变量中。
	
### 子货币示例
以下合约实现了最简单的加密货币形式。该合约只允许其创建者创建新的硬币（不同的发行方案是可能的）。任何人都可以相互发送硬币，而无需使用用户名和密码进行注册，您只需要一个以太坊密钥对。

	// SPDX-License-Identifier: GPL-3.0
	pragma solidity ^0.8.4;
	
	contract Coin {
	    // 关键字“public”使变量定义可从其他合约访问
	    address public minter;
	    mapping (address => uint) public balances;
	
	    // 事件允许客户端对您声明的特定合约更改做出反应
	    event Sent(address from, address to, uint amount);
	
	    // 构造函数代码仅在合约创建时运行
	    constructor() {
	        minter = msg.sender;
	    }
	
	    // 向一个地址发送一定数量的新创建的硬币只能由合约创建者调用(铸币)
	    function mint(address receiver, uint amount) public {
	        require(msg.sender == minter);
	        balances[receiver] += amount;
	    }
	
	    // 错误允许您提供有关操作失败原因的信息。
	    //它们被返回给函数的调用者。
	    error InsufficientBalance(uint requested, uint available);
	
	   // 从任何调用者发送一定数量的现有硬币到一个地址(发送方法)
	    function send(address receiver, uint amount) public {
	        if (amount > balances[msg.sender])
	            revert InsufficientBalance({
	                requested: amount,
	                available: balances[msg.sender]
	            });
	
	        balances[msg.sender] -= amount;
	        balances[receiver] += amount;
	        emit Sent(msg.sender, receiver, amount);
	    }
	}
	
- 第一行 `address public minter;` 解答	

	`address public minter` 该行声明了一个 [address](https://docs.soliditylang.org/en/develop/types.html#address) 类型的状态变量。
	
	`address` 类型是一个 160 位值，不允许任何算术运算。它适用于存储合约地址或属于[外部帐户](https://docs.soliditylang.org/en/develop/introduction-to-smart-contracts.html#accounts)的密钥对的公共部分的哈希值。
	
	该关键字 `public` 自动生成一个函数，允许从合约外部访问状态变量的当前值。没有这个关键字，其他合约就无法访问该变量。由编译器生成的函数的代码是等价于以下代码（当前忽略 `external` 和 `view`）：
	
		function minter() external view returns (address) { return minter; }
	你可以添加一个类似上面的函数，但是你会有一个同名的函数和状态变量。您不需要这样做，编译器会为您计算出来。
- 下一行 `mapping (address => uint) public balances;`

	创建了一个公共状态变量，但它是一种更复杂的数据类型。该[映射 map](https://docs.soliditylang.org/en/develop/types.html#mapping-types)类型映射地址[无符号整数 unsigned integers](https://docs.soliditylang.org/en/develop/types.html#integers)。

	映射可以被看作是虚拟初始化的[哈希表](https://en.wikipedia.org/wiki/Hash_table)，这样每个可能的键从一开始就存在并映射到一个字节表示全为零的值。但是，既不可能获得映射的所有键的列表，也无法获得所有值的列表。记录您添加到映射中的内容或在不需要的上下文中使用它。或者更好的是，保留一个列表或者使用更合适的数据类型。

	在映射的情况下，由 `public` 关键字创建的 [getter 函数](https://docs.soliditylang.org/en/develop/contracts.html#getter-functions) 更复杂。它看起来像下面这样：

		function balances(address _account) external view returns (uint) {
		    return balances[_account];
		}
	您可以使用该功能查询单个账户的余额。
- 第三行 `event Sent(address from, address to, uint amount);`

	该行声明了一个[事件](https://docs.soliditylang.org/en/develop/contracts.html#events)，它在函数的最后一行 `send` 。以太坊客户端（例如 Web 应用程序）可以侦听区块链上发出的这些事件，而无需花费太多成本。一旦发出，侦听器就会接收参数 `from`、`to` 和 `amount`，这使得跟踪交易成为可能。
	
	要侦听此事件，您可以使用以下 JavaScript 代码，该代码使用 [web3.js](https://github.com/ethereum/web3.js/) 创建 `Coin` 合约对象，并且任何用户界面都会 `balances` 从上面调用自动生成的函数：
	
		Coin.Sent().watch({}, '', function(error, result) {
		    if (!error) {
		        console.log("Coin transfer: " + result.args.amount +
		            " coins were sent from " + result.args.from +
		            " to " + result.args.to + ".");
		        console.log("Balances now:\n" +
		            "Sender: " + Coin.balances.call(result.args.from) +
		            "Receiver: " + Coin.balances.call(result.args.to));
		    }
		})
- `constructor`

	这个 [constructor-构造函数](https://docs.soliditylang.org/en/develop/contracts.html#constructor)是一个特殊的函数，在合约创建过程中执行，之后无法调用。 在这种情况下，它会永久存储创建合同的人的地址。 `msg` 变量（连同 `tx` 和 `block`）是一个特殊的全局变量，它包含允许访问区块链的属性。 `msg.sender` 始终是当前（外部）函数调用的来源地址。
	
	构成合约以及用户和合约可以调用的函数是 `mint` 和 `send`。
-  `mint 函数-创币函数`	
	
	`mint` 函数将一定数量的新创建的硬币发送到另一个地址。
	
	- `require 函数`

		调用定义了在不满足时还原所有更改的条件。在这个例子中， `require(msg.sender == minter)`; 确保只有合约的创建者才能调用 `mint`。一般来说，创建者可以铸造任意数量的代币，但在某些时候，这会导致一种称为“溢出”的现象。
	- `balances[receiver] += amount`
		请注意，由于默认的 [Checked 算法](https://docs.soliditylang.org/en/develop/control-structures.html#unchecked)，溢出，即任意精度算术中的 `balances[receiver] + amount` 大于 `uint (2**256 - 1)` 的最大值。对于报表 `balances[receiver] += amount;` 也是如此。 如果表达式 `balances[receiver] += amount`; 则交易将恢复。
- `erro`

	[错误](https://docs.soliditylang.org/en/develop/contracts.html#errors) 允许您向调用者提供有关条件或操作失败原因的更多信息。 错误与 [revert 语句](https://docs.soliditylang.org/en/develop/control-structures.html#revert-statement)一起使用。 `revert` 语句无条件地中止和恢复类似于 `require 函数`的所有更改，但它还允许您提供错误的名称和将提供给调用者（并最终提供给前端应用程序或块浏览器）的附加数据以便可以更轻松地调试或应对故障。
- `send 函数`

	任何人（已经拥有其中一些硬币的人）都可以使用 `send` 函数将硬币发送给其他任何人。 如果发送方没有足够的硬币可以发送，则 `require` 调用将失败并向发送方提供适当的错误消息字符串。
- 非常注意

	如果您使用此合约将硬币发送到某个地址，则在区块链浏览器上查看该地址时将看不到任何内容，因为您发送硬币的记录和更改的余额仅存储在该特定硬币的数据存储中合同。 通过使用事件，您可以创建一个“区块链浏览器”来跟踪新币的交易和余额，但您必须检查币合约地址而不是币所有者的地址。	
					
### 区块链基础
区块链作为一个概念对于程序员来说并不难理解。 原因是大多数复杂性（挖掘、[散列](https://en.wikipedia.org/wiki/Cryptographic_hash_function)、[椭圆曲线加密](https://en.wikipedia.org/wiki/Elliptic_curve_cryptography)、[点对点网络](https://en.wikipedia.org/wiki/Peer-to-peer)等）只是为了为平台提供一组特定的功能和承诺。 一旦你接受了这些给定的功能，你就不必担心底层技术或你是否必须知道亚马逊的 AWS 在内部是如何工作的才能使用它？
#### 交易
区块链是一个全球共享的交易数据库。这意味着每个人都可以通过参与网络来读取数据库中的条目。如果您想更改数据库中的某些内容，您必须创建一个必须被所有其他人接受的所谓的事务。事务一词意味着您要进行的更改（假设您要同时更改两个值）要么根本没有完成，要么完全应用。此外，当您的事务被应用到数据库时，没有其他事务可以改变它。

例如，假设有一个表格，其中列出了所有电子货币账户的余额。如果请求从一个帐户转移到另一个帐户，则数据库的交易性质确保如果从一个帐户中减去该金额，则始终将其添加到另一个帐户中。如果由于某种原因无法将金额添加到目标帐户，则源帐户也不会被修改。

此外，交易始终由发送者（创建者）进行加密签名。这使得保护对数据库特定修改的访问变得简单。在电子货币的例子中，简单的检查确保只有持有账户密钥的人才能从中转账
#### 块
要克服的一个主要障碍是（用比特币术语）所谓的“双花攻击”：如果网络中存在两笔交易都想清空账户，会发生什么？只有一个交易是有效的，通常是第一个被接受的交易。问题在于“第一”在对等网络中并不是一个客观的术语。

对此的抽象答案是您不必关心。将为您选择一个全球接受的交易顺序，解决冲突。交易将被捆绑到所谓的“块”中，然后它们将在所有参与节点之间执行和分配。如果两笔交易相互矛盾，最后成为第二笔的交易将被拒绝，不会成为区块的一部分。

这些块在时间上形成线性序列，这就是“区块链”一词的来源。区块以相当规律的时间间隔添加到链中——对于以太坊来说，大约每 17 秒一次。

作为“订单选择机制”（称为“挖掘”）的一部分，可能会不时恢复块，但仅在链的“尖端”。在特定块的顶部添加的块越多，该块被还原的可能性就越小。因此，您的交易可能会被还原甚至从区块链中删除，但您等待的时间越长，它就越不可能。

- 注意

	交易不能保证包含在下一个区块或任何特定的未来区块中，因为这不是由交易的提交者决定，而是由矿工决定交易包含在哪个区块中。

	如果您想安排合约的未来调用，您可以使用闹钟或类似的 oracle 服务。

### 以太坊虚拟机
#### 概述
以太坊虚拟机或 EVM 是以太坊智能合约的运行时环境。 它不仅是沙盒化的而且实际上是完全隔离的，这意味着在 EVM 中运行的代码无法访问网络、文件系统或其他进程。 智能合约甚至对其他智能合约的访问也很有限。
#### 帐户
以太坊中有两种账户共享相同的地址空间：

- 由公私钥对（即人类）控制的外部账户
- 由与账户一起存储的代码控制的合约账户

外部账户的地址是由公钥决定的，而合约的地址是在合约创建时决定的（它是从创建者地址和从该地址发送的交易数量派生出来的，所谓的“ 随机数”）。

无论帐户是否存储代码，EVM 对这两种类型都一视同仁。

每个帐户都有一个持久的键值存储，将 256 位字映射到 256 位字，称为`存储`。

此外，每个账户都有一个 `Ether `余额（确切地说，在“Wei”中，`1 ether` 是 `10**18 wei`）可以通过发送包含 Ether 的交易来修改。
### 交易
交易是从一个账户发送到另一个账户的消息（可能相同或为空，见下文）。 它可以包括二进制数据（称为“有效载荷”）和以太币。

如果目标帐户包含代码，则执行该代码并将有效负载作为输入数据提供。

如果没有设置目标账户（交易没有接收方或接收方设置为空），则交易创建一个新合约。 如前所述，该合约的地址不是零地址，而是从发送者及其发送的交易数量（“nonce”）派生的地址。 此类合约创建交易的有效载荷被视为 EVM 字节码并执行。 本次执行的输出数据作为合约代码永久保存。 这意味着为了创建合约，您不会发送合约的实际代码，而是在执行时返回该代码的实际代码。

- 笔记

	在创建合约时，其代码仍然是空的。 因此，在构造函数完成执行之前，您不应回调正在构造的合约。

### Gas
在创建时，每笔交易都会被收取一定数量的 gas，其目的是限制执行交易所需的工作量并同时为此次执行支付费用。 在 EVM 执行交易的同时，gas 根据特定规则逐渐耗尽。

`gas price `是由交易创建者设定的值，他必须从发送账户预先支付 `gas_price * gas`。 如果执行后还剩下一些 gas，则以同样的方式返还给交易创作者。

如果 gas 在任何时候用完（即它会是负数），就会触发一个 out-of-gas 异常，这将恢复对当前调用帧中的状态所做的所有修改。
### 存储、内存和堆栈
以太坊虚拟机具有三个可以存储数据的区域——存储、内存和堆栈，这将在以下段落中进行解释。

- 存储

	每个帐户都有一个称为存储的数据区域，它包含
	
	- 函数调用
	- 事务
	
	之间的持久化。存储是将 256 位字映射到 256 位字的键值存储。不可能从合约中枚举存储，读取成本相对较高，初始化和修改存储的成本更高。由于此成本，您应该将持久存储中存储的内容最小化为合约需要运行的内容。在合同之外存储数据，如派生计算、缓存和聚合。合约不能读取或写入除自身之外的任何存储。
- 内存

	第二个数据区称为内存，其中一个合约为每个消息调用获取一个新清除的实例。内存是线性的，可以在字节级别寻址，但读取的宽度限制为 256 位，而写入的宽度可以为 8 位或 256 位。当访问（读取或写入）以前未触及的内存字（即字内的任何偏移量）时，内存会扩展一个字（256 位）。在扩建时，必须支付 gas 费用。内存越大，成本越高（按二次方扩展）1。
- 堆栈

	EVM 不是寄存器机而是堆栈机，因此所有计算都在称为堆栈的数据区域上执行。它的最大大小为 1024 个元素，包含 256 位的字。对堆栈的访问以下列方式被限制在顶端：
	
	- 可以将最顶部的 16 个元素之一复制到堆栈顶部
	- 或者将最顶部的元素与其下方的 16 个元素之一交换
	- 所有其他操作从堆栈中取出最顶部的两个（或一个或多个，取决于操作）元素并将结果压入堆栈。
	- 当然，可以将堆栈元素移动到存储或内存中，以便更深入地访问堆栈，但不可能在不首先移除堆栈顶部的情况下访问堆栈中更深处的任意元素。指令系统

### 指令系统
EVM 的指令集保持最小以避免可能导致共识问题的不正确或不一致的实现。 所有指令都对基本数据类型、256 位字或内存片（或其他字节数组）进行操作。 存在通常的算术、位、逻辑和比较操作。 条件跳转和无条件跳转都是可能的。 此外，合约可以访问当前块的相关属性，例如其编号和时间戳。

有关完整列表，请参阅作为内联汇编文档一部分的[操作码列表](https://docs.soliditylang.org/en/develop/yul.html#opcodes)。
### 消息调用
合约可以通过消息调用的方式调用其他合约或向非合约账户发送 Ether。消息调用类似于事务，因为它们具有源、目标、数据负载、以太、气体和返回数据。事实上，每个事务都包含一个顶级消息调用，而后者又可以创建更多的消息调用。

合约可以决定应该通过内部消息调用发送多少剩余的 gas 以及它想要保留多少。如果在内部调用（或任何其他异常）中发生了气体耗尽异常，这将通过放入堆栈的错误值来表示。在这种情况下，只有与调用一起发送的气体才会被用完。

在 Solidity 中，在这种情况下，调用合约默认会导致手动异常，因此异常会“冒泡”调用堆栈。

如前所述，被调用的合约（可以与调用者相同）将收到一个新清除的内存实例，并有权访问调用有效负载 - 它将在称为 calldata 的单独区域中提供。执行完成后，它可以返回数据，这些数据将存储在调用者预先分配的调用者内存中的某个位置。所有这些调用都是完全同步的。

调用的深度限制为 1024，这意味着对于更复杂的操作，应该首选循环而不是递归调用。此外，消息调用中只能转发 63/64th 的gas，这导致实际中的深度限制略低于1000。
### Delegatecall/Callcode 和库
存在一个消息调用的特殊变体，名为 `delegatecall` 与消息调用相同，除了目标地址处的代码在调用合约的上下文中执行的 `msg.sender` 和 `msg.value` 不会改变外。

这意味着合约可以在运行时从不同地址动态加载代码。 存储、当前地址和余额仍然是指调用合约，只是代码取自被调用地址。

这使得在 Solidity 中实现“库”功能成为可能：可重用的库代码可以应用于合约的存储，例如 为了实现复杂的数据结构。
### 日志
可以将数据存储在特殊索引的数据结构中，该结构一直映射到块级别。 Solidity 使用这个称为日志的功能来实现[事件](https://docs.soliditylang.org/en/develop/contracts.html#events)。 合约创建后无法访问日志数据，但可以从区块链外部高效访问它们。 由于日志数据的某些部分存储在[布隆过滤器](https://en.wikipedia.org/wiki/Bloom_filter)中，因此可以以高效且加密安全的方式搜索这些数据，因此不下载整个区块链的网络对等点（所谓的“轻客户端”）仍然可以找到这些日志。
### 创造
合约甚至可以使用特殊的操作码创建其他合约（即它们不像交易那样简单地调用零地址）。 这些创建调用和普通消息调用之间的唯一区别是执行有效载荷数据并将结果存储为代码，调用者/创建者在堆栈上接收新合约的地址。
### 停用和自毁
从区块链中删除代码的唯一方法是该地址的合约执行 `selfdestruct `-自毁。 存储在该地址的剩余 Ether 被发送到指定的目标，然后存储和代码从状态中删除。 理论上删除合约听起来是个好主意，但它有潜在的危险，好像有人将以太币发送到删除的合约中，以太币就永远丢失了。

- 警告

	即使合约被 `selfdestruct`，它仍然是区块链历史的一部分，并且可能被大多数以太坊节点保留。 因此，使用 `selfdestruct` 与从硬盘中删除数据不同。
- 注意

	即使合约的代码不包含对 `selfdestruct` 的调用，它仍然可以使用 `delegatecall` 或 `callcode` 执行该操作。
	
如果你想停用你的合约，你应该通过改变一些导致所有功能恢复的内部状态来禁用它们。 这使得无法使用合约，因为它会立即返回 Ether。

## 安装 solidity 编译器-[详情](https://docs.soliditylang.org/en/develop/installing-solidity.html)
- Remix
- npm/Node.js
- docker
- 传统安装
	- linux
	- macos 
	- 二进制安装

## Solidity by Example
- 投票系统

	以下合约相当复杂，但展示了 Solidity 的许多功能。它实现了一个投票合约。当然，电子投票的主要问题是如何将投票权分配给正确的人以及如何防止操纵。我们不会在这里解决所有问题，但至少我们将展示如何进行委托投票，以便同时自动计算和完全透明。

	这个想法是
	
	- 为每个被投票者创建一个合同，为每个选项提供一个简称。
	- 然后担任主席的合约创建者将分别授予每个地址的投票权。
	- 地址后面的人可以选择自己投票或将投票委托给他们信任的人。
	- 在投票时间结束时，`winningProposal()` 将返回得票最多的提案。

	代码
	
		// SPDX-License-Identifier: GPL-3.0
		pragma solidity >=0.7.0 <0.9.0;
		/// @title Voting with delegation.
		contract Ballot {
		    // This declares a new complex type which will
		    // be used for variables later.
		    // It will represent a single voter.
		    struct Voter {
		        uint weight; // weight is accumulated by delegation
		        bool voted;  // if true, that person already voted
		        address delegate; // person delegated to
		        uint vote;   // index of the voted proposal
		    }
		
		    // This is a type for a single proposal.
		    struct Proposal {
		        bytes32 name;   // short name (up to 32 bytes)
		        uint voteCount; // number of accumulated votes
		    }
		
		    address public chairperson;
		
		    // This declares a state variable that
		    // stores a `Voter` struct for each possible address.
		    mapping(address => Voter) public voters;
		
		    // A dynamically-sized array of `Proposal` structs.
		    Proposal[] public proposals;
		
		    /// Create a new ballot to choose one of `proposalNames`.
		    constructor(bytes32[] memory proposalNames) {
		        chairperson = msg.sender;
		        voters[chairperson].weight = 1;
		
		        // For each of the provided proposal names,
		        // create a new proposal object and add it
		        // to the end of the array.
		        for (uint i = 0; i < proposalNames.length; i++) {
		            // `Proposal({...})` creates a temporary
		            // Proposal object and `proposals.push(...)`
		            // appends it to the end of `proposals`.
		            proposals.push(Proposal({
		                name: proposalNames[i],
		                voteCount: 0
		            }));
		        }
		    }
		
		    // Give `voter` the right to vote on this ballot.
		    // May only be called by `chairperson`.
		    function giveRightToVote(address voter) public {
		        // If the first argument of `require` evaluates
		        // to `false`, execution terminates and all
		        // changes to the state and to Ether balances
		        // are reverted.
		        // This used to consume all gas in old EVM versions, but
		        // not anymore.
		        // It is often a good idea to use `require` to check if
		        // functions are called correctly.
		        // As a second argument, you can also provide an
		        // explanation about what went wrong.
		        require(
		            msg.sender == chairperson,
		            "Only chairperson can give right to vote."
		        );
		        require(
		            !voters[voter].voted,
		            "The voter already voted."
		        );
		        require(voters[voter].weight == 0);
		        voters[voter].weight = 1;
		    }
		
		    /// Delegate your vote to the voter `to`.
		    function delegate(address to) public {
		        // assigns reference
		        Voter storage sender = voters[msg.sender];
		        require(!sender.voted, "You already voted.");
		
		        require(to != msg.sender, "Self-delegation is disallowed.");
		
		        // Forward the delegation as long as
		        // `to` also delegated.
		        // In general, such loops are very dangerous,
		        // because if they run too long, they might
		        // need more gas than is available in a block.
		        // In this case, the delegation will not be executed,
		        // but in other situations, such loops might
		        // cause a contract to get "stuck" completely.
		        while (voters[to].delegate != address(0)) {
		            to = voters[to].delegate;
		
		            // We found a loop in the delegation, not allowed.
		            require(to != msg.sender, "Found loop in delegation.");
		        }
		
		        // Since `sender` is a reference, this
		        // modifies `voters[msg.sender].voted`
		        sender.voted = true;
		        sender.delegate = to;
		        Voter storage delegate_ = voters[to];
		        if (delegate_.voted) {
		            // If the delegate already voted,
		            // directly add to the number of votes
		            proposals[delegate_.vote].voteCount += sender.weight;
		        } else {
		            // If the delegate did not vote yet,
		            // add to her weight.
		            delegate_.weight += sender.weight;
		        }
		    }
		
		    /// Give your vote (including votes delegated to you)
		    /// to proposal `proposals[proposal].name`.
		    function vote(uint proposal) public {
		        Voter storage sender = voters[msg.sender];
		        require(sender.weight != 0, "Has no right to vote");
		        require(!sender.voted, "Already voted.");
		        sender.voted = true;
		        sender.vote = proposal;
		
		        // If `proposal` is out of the range of the array,
		        // this will throw automatically and revert all
		        // changes.
		        proposals[proposal].voteCount += sender.weight;
		    }
		
		    /// @dev Computes the winning proposal taking all
		    /// previous votes into account.
		    function winningProposal() public view
		            returns (uint winningProposal_)
		    {
		        uint winningVoteCount = 0;
		        for (uint p = 0; p < proposals.length; p++) {
		            if (proposals[p].voteCount > winningVoteCount) {
		                winningVoteCount = proposals[p].voteCount;
		                winningProposal_ = p;
		            }
		        }
		    }
		
		    // Calls winningProposal() function to get the index
		    // of the winner contained in the proposals array and then
		    // returns the name of the winner
		    function winnerName() public view
		            returns (bytes32 winnerName_)
		    {
		        winnerName_ = proposals[winningProposal()].name;
	    		}
		}

	可能的改进，目前，需要进行很多交易才能将投票权分配给所有参与者。你能想到更好的方法吗？
- 拍卖系统

	在本节中，我们将展示在以太坊上创建一个完全盲目的拍卖合约是多么容易。我们将从公开拍卖开始，每个人都可以看到进行的投标，然后将此合同扩展为盲拍卖，在投标期结束之前无法看到实际出价。
	
	- 简单的公开拍卖

		以下简单拍卖合同的总体思路是每个人都可以在投标期间发送他们的投标。出价已经包括汇款/以太币以将投标人绑定到他们的出价中。如果提高了最高出价，则之前出价最高的人会收回他们的钱。投标期结束后，必须手动调用合同，受益人才能收到钱 - 合同无法自行激活。
		
			// SPDX-License-Identifier: GPL-3.0
			pragma solidity ^0.8.4;
			contract SimpleAuction {
			    // Parameters of the auction. Times are either
			    // absolute unix timestamps (seconds since 1970-01-01)
			    // or time periods in seconds.
			    address payable public beneficiary;
			    uint public auctionEndTime;
			
			    // Current state of the auction.
			    address public highestBidder;
			    uint public highestBid;
			
			    // Allowed withdrawals of previous bids
			    mapping(address => uint) pendingReturns;
			
			    // Set to true at the end, disallows any change.
			    // By default initialized to `false`.
			    bool ended;
			
			    // Events that will be emitted on changes.
			    event HighestBidIncreased(address bidder, uint amount);
			    event AuctionEnded(address winner, uint amount);
			
			    // Errors that describe failures.
			
			    // The triple-slash comments are so-called natspec
			    // comments. They will be shown when the user
			    // is asked to confirm a transaction or
			    // when an error is displayed.
			
			    /// The auction has already ended.
			    error AuctionAlreadyEnded();
			    /// There is already a higher or equal bid.
			    error BidNotHighEnough(uint highestBid);
			    /// The auction has not ended yet.
			    error AuctionNotYetEnded();
			    /// The function auctionEnd has already been called.
			    error AuctionEndAlreadyCalled();
			
			    /// Create a simple auction with `_biddingTime`
			    /// seconds bidding time on behalf of the
			    /// beneficiary address `_beneficiary`.
			    constructor(
			        uint _biddingTime,
			        address payable _beneficiary
			    ) {
			        beneficiary = _beneficiary;
			        auctionEndTime = block.timestamp + _biddingTime;
			    }
			
			    /// Bid on the auction with the value sent
			    /// together with this transaction.
			    /// The value will only be refunded if the
			    /// auction is not won.
			    function bid() public payable {
			        // No arguments are necessary, all
			        // information is already part of
			        // the transaction. The keyword payable
			        // is required for the function to
			        // be able to receive Ether.
			
			        // Revert the call if the bidding
			        // period is over.
			        if (block.timestamp > auctionEndTime)
			            revert AuctionAlreadyEnded();
			
			        // If the bid is not higher, send the
			        // money back (the revert statement
			        // will revert all changes in this
			        // function execution including
			        // it having received the money).
			        if (msg.value <= highestBid)
			            revert BidNotHighEnough(highestBid);
			
			        if (highestBid != 0) {
			            // Sending back the money by simply using
			            // highestBidder.send(highestBid) is a security risk
			            // because it could execute an untrusted contract.
			            // It is always safer to let the recipients
			            // withdraw their money themselves.
			            pendingReturns[highestBidder] += highestBid;
			        }
			        highestBidder = msg.sender;
			        highestBid = msg.value;
			        emit HighestBidIncreased(msg.sender, msg.value);
			    }
			
			    /// Withdraw a bid that was overbid.
			    function withdraw() public returns (bool) {
			        uint amount = pendingReturns[msg.sender];
			        if (amount > 0) {
			            // It is important to set this to zero because the recipient
			            // can call this function again as part of the receiving call
			            // before `send` returns.
			            pendingReturns[msg.sender] = 0;
			
			            if (!payable(msg.sender).send(amount)) {
			                // No need to call throw here, just reset the amount owing
			                pendingReturns[msg.sender] = amount;
			                return false;
			            }
			        }
			        return true;
			    }
			
			    /// End the auction and send the highest bid
			    /// to the beneficiary.
			    function auctionEnd() public {
			        // It is a good guideline to structure functions that interact
			        // with other contracts (i.e. they call functions or send Ether)
			        // into three phases:
			        // 1. checking conditions
			        // 2. performing actions (potentially changing conditions)
			        // 3. interacting with other contracts
			        // If these phases are mixed up, the other contract could call
			        // back into the current contract and modify the state or cause
			        // effects (ether payout) to be performed multiple times.
			        // If functions called internally include interaction with external
			        // contracts, they also have to be considered interaction with
			        // external contracts.
			
			        // 1. Conditions
			        if (block.timestamp < auctionEndTime)
			            revert AuctionNotYetEnded();
			        if (ended)
			            revert AuctionEndAlreadyCalled();
			
			        // 2. Effects
			        ended = true;
			        emit AuctionEnded(highestBidder, highestBid);
			
			        // 3. Interaction
			        beneficiary.transfer(highestBid);
			    }
			}
	- 盲拍

		之前的公开拍卖在下文中扩展为盲拍。盲拍的优点是在投标期结束时没有时间压力。在透明的计算平台上创建盲目拍卖听起来似乎很矛盾，但密码学可以解决问题。

		在投标期间，投标人实际上不会发送他们的投标，而只是它的散列版本。由于目前认为实际上不可能找到两个（足够长的）哈希值相等的值，因此投标人承诺投标。投标期结束后，投标人必须公开他们的投标：他们发送未加密的值，合同检查哈希值是否与投标期间提供的值相同。

		另一个挑战是如何使拍卖同时具有约束力和盲目性：防止投标人赢得拍卖后不发送资金的唯一方法是让他们将其与投标一起发送。由于以太坊中的价值转移无法被屏蔽，因此任何人都可以看到价值。

		下面的合约通过接受任何大于最高出价的值来解决这个问题。由于这当然只能在显示阶段进行检查，因此某些出价可能无效，这是故意的（它甚至提供了一个明确的标志来放置具有高价值转移的无效出价）：投标人可以通过放置几个高价或低无效出价。
		
			// SPDX-License-Identifier: GPL-3.0
			pragma solidity ^0.8.4;
			contract BlindAuction {
			    struct Bid {
			        bytes32 blindedBid;
			        uint deposit;
			    }
			
			    address payable public beneficiary;
			    uint public biddingEnd;
			    uint public revealEnd;
			    bool public ended;
			
			    mapping(address => Bid[]) public bids;
			
			    address public highestBidder;
			    uint public highestBid;
			
			    // Allowed withdrawals of previous bids
			    mapping(address => uint) pendingReturns;
			
			    event AuctionEnded(address winner, uint highestBid);
			
			    // Errors that describe failures.
			
			    /// The function has been called too early.
			    /// Try again at `time`.
			    error TooEarly(uint time);
			    /// The function has been called too late.
			    /// It cannot be called after `time`.
			    error TooLate(uint time);
			    /// The function auctionEnd has already been called.
			    error AuctionEndAlreadyCalled();
			
			    // Modifiers are a convenient way to validate inputs to
			    // functions. `onlyBefore` is applied to `bid` below:
			    // The new function body is the modifier's body where
			    // `_` is replaced by the old function body.
			    modifier onlyBefore(uint _time) {
			        if (block.timestamp >= _time) revert TooLate(_time);
			        _;
			    }
			    modifier onlyAfter(uint _time) {
			        if (block.timestamp <= _time) revert TooEarly(_time);
			        _;
			    }
			
			    constructor(
			        uint _biddingTime,
			        uint _revealTime,
			        address payable _beneficiary
			    ) {
			        beneficiary = _beneficiary;
			        biddingEnd = block.timestamp + _biddingTime;
			        revealEnd = biddingEnd + _revealTime;
			    }
			
			    /// Place a blinded bid with `_blindedBid` =
			    /// keccak256(abi.encodePacked(value, fake, secret)).
			    /// The sent ether is only refunded if the bid is correctly
			    /// revealed in the revealing phase. The bid is valid if the
			    /// ether sent together with the bid is at least "value" and
			    /// "fake" is not true. Setting "fake" to true and sending
			    /// not the exact amount are ways to hide the real bid but
			    /// still make the required deposit. The same address can
			    /// place multiple bids.
			    function bid(bytes32 _blindedBid)
			        public
			        payable
			        onlyBefore(biddingEnd)
			    {
			        bids[msg.sender].push(Bid({
			            blindedBid: _blindedBid,
			            deposit: msg.value
			        }));
			    }
			
			    /// Reveal your blinded bids. You will get a refund for all
			    /// correctly blinded invalid bids and for all bids except for
			    /// the totally highest.
			    function reveal(
			        uint[] memory _values,
			        bool[] memory _fake,
			        bytes32[] memory _secret
			    )
			        public
			        onlyAfter(biddingEnd)
			        onlyBefore(revealEnd)
			    {
			        uint length = bids[msg.sender].length;
			        require(_values.length == length);
			        require(_fake.length == length);
			        require(_secret.length == length);
			
			        uint refund;
			        for (uint i = 0; i < length; i++) {
			            Bid storage bidToCheck = bids[msg.sender][i];
			            (uint value, bool fake, bytes32 secret) =
			                    (_values[i], _fake[i], _secret[i]);
			            if (bidToCheck.blindedBid != keccak256(abi.encodePacked(value, fake, secret))) {
			                // Bid was not actually revealed.
			                // Do not refund deposit.
			                continue;
			            }
			            refund += bidToCheck.deposit;
			            if (!fake && bidToCheck.deposit >= value) {
			                if (placeBid(msg.sender, value))
			                    refund -= value;
			            }
			            // Make it impossible for the sender to re-claim
			            // the same deposit.
			            bidToCheck.blindedBid = bytes32(0);
			        }
			        payable(msg.sender).transfer(refund);
			    }
			
			    /// Withdraw a bid that was overbid.
			    function withdraw() public {
			        uint amount = pendingReturns[msg.sender];
			        if (amount > 0) {
			            // It is important to set this to zero because the recipient
			            // can call this function again as part of the receiving call
			            // before `transfer` returns (see the remark above about
			            // conditions -> effects -> interaction).
			            pendingReturns[msg.sender] = 0;
			
			            payable(msg.sender).transfer(amount);
			        }
			    }
			
			    /// End the auction and send the highest bid
			    /// to the beneficiary.
			    function auctionEnd()
			        public
			        onlyAfter(revealEnd)
			    {
			        if (ended) revert AuctionEndAlreadyCalled();
			        emit AuctionEnded(highestBidder, highestBid);
			        ended = true;
			        beneficiary.transfer(highestBid);
			    }
			
			    // This is an "internal" function which means that it
			    // can only be called from the contract itself (or from
			    // derived contracts).
			    function placeBid(address bidder, uint value) internal
			            returns (bool success)
			    {
			        if (value <= highestBid) {
			            return false;
			        }
			        if (highestBidder != address(0)) {
			            // Refund the previously highest bidder.
			            pendingReturns[highestBidder] += highestBid;
			        }
			        highestBid = value;
			        highestBidder = bidder;
			        return true;
			    }
			}		
- 安全远程购买

	目前远程购买商品需要多方相互信任。最简单的配置涉及卖方和买方。买方希望收到卖方的物品，而卖方希望获得金钱（或等价物）作为回报。有问题的部分是这里的装运：无法确定物品是否到达买方。

	有多种方法可以解决这个问题，但都以一种或另一种方式不足。在以下示例中，双方必须将项目价值的两倍作为托管放入合同中。一旦发生这种情况，这笔钱将被锁定在合同中，直到买方确认他们收到了物品。之后，买家将获得价值（他们押金的一半）的返还，而卖家则获得三倍的价值（他们的押金加上价值）。这背后的想法是，双方都有解决问题的动力，否则他们的钱就会被永远锁定。

	这个合约当然不能解决问题，但提供了如何在合约中使用类似状态机的结构的概述。
	
			// SPDX-License-Identifier: GPL-3.0
			pragma solidity ^0.8.4;
			contract Purchase {
			    uint public value;
			    address payable public seller;
			    address payable public buyer;
			
			    enum State { Created, Locked, Release, Inactive }
			    // The state variable has a default value of the first member, `State.created`
			    State public state;
			
			    modifier condition(bool _condition) {
			        require(_condition);
			        _;
			    }
			
			    /// Only the buyer can call this function.
			    error OnlyBuyer();
			    /// Only the seller can call this function.
			    error OnlySeller();
			    /// The function cannot be called at the current state.
			    error InvalidState();
			    /// The provided value has to be even.
			    error ValueNotEven();
			
			    modifier onlyBuyer() {
			        if (msg.sender != buyer)
			            revert OnlyBuyer();
			        _;
			    }
			
			    modifier onlySeller() {
			        if (msg.sender != seller)
			            revert OnlySeller();
			        _;
			    }
			
			    modifier inState(State _state) {
			        if (state != _state)
			            revert InvalidState();
			        _;
			    }
			
			    event Aborted();
			    event PurchaseConfirmed();
			    event ItemReceived();
			    event SellerRefunded();
			
			    // Ensure that `msg.value` is an even number.
			    // Division will truncate if it is an odd number.
			    // Check via multiplication that it wasn't an odd number.
			    constructor() payable {
			        seller = payable(msg.sender);
			        value = msg.value / 2;
			        if ((2 * value) != msg.value)
			            revert ValueNotEven();
			    }
			
			    /// Abort the purchase and reclaim the ether.
			    /// Can only be called by the seller before
			    /// the contract is locked.
			    function abort()
			        public
			        onlySeller
			        inState(State.Created)
			    {
			        emit Aborted();
			        state = State.Inactive;
			        // We use transfer here directly. It is
			        // reentrancy-safe, because it is the
			        // last call in this function and we
			        // already changed the state.
			        seller.transfer(address(this).balance);
			    }
			
			    /// Confirm the purchase as buyer.
			    /// Transaction has to include `2 * value` ether.
			    /// The ether will be locked until confirmReceived
			    /// is called.
			    function confirmPurchase()
			        public
			        inState(State.Created)
			        condition(msg.value == (2 * value))
			        payable
			    {
			        emit PurchaseConfirmed();
			        buyer = payable(msg.sender);
			        state = State.Locked;
			    }
			
			    /// Confirm that you (the buyer) received the item.
			    /// This will release the locked ether.
			    function confirmReceived()
			        public
			        onlyBuyer
			        inState(State.Locked)
			    {
			        emit ItemReceived();
			        // It is important to change the state first because
			        // otherwise, the contracts called using `send` below
			        // can call in again here.
			        state = State.Release;
			
			        buyer.transfer(value);
			    }
			
			    /// This function refunds the seller, i.e.
			    /// pays back the locked funds of the seller.
			    function refundSeller()
			        public
			        onlySeller
			        inState(State.Release)
			    {
			        emit SellerRefunded();
			        // It is important to change the state first because
			        // otherwise, the contracts called using `send` below
			        // can call in again here.
			        state = State.Inactive;
			
			        seller.transfer(3 * value);
			    }
			}	
- 小额支付渠道  

	在本节中，我们将学习如何构建支付渠道的示例实现。它使用加密签名在同一方之间安全、即时且无交易费用地重复传输 Ether。例如，我们需要了解如何签名和验证签名，以及设置支付渠道。

	- 创建和验证签名

		想象一下 Alice 想要向 Bob 发送一定数量的 Ether，即 Alice 是发送者，而 Bob 是接收者。

		Alice 只需要在链外（例如通过电子邮件）向 Bob 发送加密签名的消息，这类似于写支票。

		Alice 和 Bob 使用签名来授权交易，这可以通过以太坊上的智能合约实现。Alice 将构建一个简单的智能合约，让她传输 Ether，但她不会自己调用函数来发起支付，而是让 Bob 这样做，从而支付交易费用。

		该合同将按以下方式运作：

		- Alice 部署 `ReceiverPays` 合约，附加足够的以太币来支付将要支付的款项
		- Alice 通过使用她的私钥签署消息来授权付款
		- Alice 将加密签名的消息发送给 Bob。消息不需要保密（稍后解释），发送它的机制无关紧要
		- Bob 通过将签名的消息提交给智能合约来索取他的付款，它验证消息的真实性，然后释放资金。
	
		开始
		
		- 创建签名
	
			Alice 不需要与以太坊网络交互来签署交易，整个过程是完全离线的。在本教程中，我们将使用 [web3.js](https://github.com/ethereum/web3.js)和 [MetaMask](https://metamask.io/) 在浏览器中签署消息，使用 [EIP-762](https://github.com/ethereum/EIPs/pull/712) 中描述的方法，因为它提供了许多其他安全优势。
			
			- 笔记	
			
				`web3.eth.personal.sign` 预先考虑该消息的到签名数据的长度。因为我们先散列，所以消息总是正好是 32 字节长，因此这个长度前缀总是相同的。=
			- 注意
	
				这个签名方法可能产生夸链攻击，可以使用它的升级版本
			- 签署什么
	
				对于履行付款的合同，签署的消息必须包括：
				
				- 收件人的地址。
				- 要转移的金额。
				- 防止重放攻击。
				
				重放攻击是指重新使用已签名的消息来声明对第二个操作的授权。为了避免重放攻击，我们使用与以太坊交易本身相同的技术，即所谓的随机数，即账户发送的交易数量。智能合约检查一个随机数是否被多次使用。
				
				另一种类型的重放攻击可能发生在所有者部署 `ReceiverPays` 智能合约，支付一些费用，然后销毁合约。后来，他们决定再次部署 `RecipientPays` 智能合约，但新合约不知道之前部署中使用的随机数，因此攻击者可以再次使用旧消息。
				
				Alice 可以通过在消息中包含合约地址来防止这种攻击，并且只有包含合约地址本身的消息才会被接受。您可以 `claimPayment()` 在本节末尾的完整合约功能的前两行中找到一个示例。
			- 包装参数
	
				既然已经确定了要在签名消息中包含哪些信息，就可以将消息放在一起、散列并签名。为简单起见连接数据。该 [ethereumjs-ABI](https://github.com/ethereumjs/ethereumjs-abi) 库提供了一个调用的函数 `soliditySHA3`，模仿密实度的行为 `keccak256` 功能应用到使用参数编码`abi.encodePacked`。这是一个为 `ReceiverPays` 示例创建正确签名的 JavaScript 函数：
				
					// recipient is the address that should be paid.
					// amount, in wei, specifies how much ether should be sent.
					// nonce can be any unique number to prevent replay attacks
					// contractAddress is used to prevent cross-contract replay attacks
					function signPayment(recipient, amount, nonce, contractAddress, callback) {
					    var hash = "0x" + abi.soliditySHA3(
					        ["address", "uint256", "uint256", "address"],
					        [recipient, amount, nonce, contractAddress]
					    ).toString("hex");
					
					    web3.eth.personal.sign(hash, web3.eth.defaultAccount, callback);
					}
			- 整体签名以下是修改后的 JavaScript 代码：
			
					function constructPaymentMessage(contractAddress, amount) {
					    return abi.soliditySHA3(
					        ["address", "uint256"],
					        [contractAddress, amount]
					    );
					}
					
					function signMessage(message, callback) {
					    web3.eth.personal.sign(
					        "0x" + message.toString("hex"),
					        web3.eth.defaultAccount,
					        callback
					    );
					}
					
					// contractAddress is used to prevent cross-contract replay attacks.
					// amount, in wei, specifies how much Ether should be sent.
					
					function signPayment(contractAddress, amount, callback) {
					    var message = constructPaymentMessage(contractAddress, amount);
					    signMessage(message, callback);
					}		
		- 在 Solidity 中恢复消息签名者

			通常，ECDSA 签名由两个参数组成， `r` 和 `s` . 以太坊中的签名包括名为的第三个参数 `v`，您可以使用它来验证哪个帐户的私钥用于签署消息，以及交易的发送者。Solidity 提供了一个内置函数 [ecrecover](https://docs.soliditylang.org/en/develop/units-and-global-variables.html#mathematical-and-cryptographic-functions)，它接受一条消息以及`r`,`s` 和 `v` 参数，并返回用于签署消息的地址。

			- 提取签名参数

				web3.js 生成的签名是 `r`,` s`和 `v`的串联，因此第一步是将这些参数分开。您可以在客户端执行此操作，但在智能合约中执行此操作意味着您只需要发送一个签名参数而不是三个。将字节数组拆分为其组成部分是一团糟，因此我们使用[内联汇编](https://docs.soliditylang.org/en/develop/assembly.html)来完成 `splitSignature` 函数中的工作（本节末尾完整合约中的第三个函数）。
			- 计算消息哈希

				智能合约需要确切知道签署了哪些参数，因此它必须根据参数重新创建消息并将其用于签名验证。函数 `prefixed` 和 `recoverSigner`在 `claimPayment`函数中执行此操作。
		- 完整合约

				// SPDX-License-Identifier: GPL-3.0
				pragma solidity >=0.7.0 <0.9.0;
				contract ReceiverPays {
				    address owner = msg.sender;
				
				    mapping(uint256 => bool) usedNonces;
				
				    constructor() payable {}
				
				    function claimPayment(uint256 amount, uint256 nonce, bytes memory signature) public {
				        require(!usedNonces[nonce]);
				        usedNonces[nonce] = true;
				
				        // this recreates the message that was signed on the client
				        bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));
				
				        require(recoverSigner(message, signature) == owner);
				
				        payable(msg.sender).transfer(amount);
				    }
				
				    /// destroy the contract and reclaim the leftover funds.
				    function shutdown() public {
				        require(msg.sender == owner);
				        selfdestruct(payable(msg.sender));
				    }
				
				    /// signature methods.
				    function splitSignature(bytes memory sig)
				        internal
				        pure
				        returns (uint8 v, bytes32 r, bytes32 s)
				    {
				        require(sig.length == 65);
				
				        assembly {
				            // first 32 bytes, after the length prefix.
				            r := mload(add(sig, 32))
				            // second 32 bytes.
				            s := mload(add(sig, 64))
				            // final byte (first byte of the next 32 bytes).
				            v := byte(0, mload(add(sig, 96)))
				        }
				
				        return (v, r, s);
				    }
				
				    function recoverSigner(bytes32 message, bytes memory sig)
				        internal
				        pure
				        returns (address)
				    {
				        (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);
				
				        return ecrecover(message, v, r, s);
				    }
				
				    /// builds a prefixed hash to mimic the behavior of eth_sign.
				    function prefixed(bytes32 hash) internal pure returns (bytes32) {
				        return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
				    }
				}
	- 编写一个简单的支付渠道

		Alice 现在构建了一个简单但完整的支付渠道实现。支付渠道使用加密签名来安全、即时且无需交易费用地重复传输以太币。

		- 什么是支付渠道？

			支付渠道允许参与者在不使用交易的情况下重复转移以太币。这意味着您可以避免与交易相关的延迟和费用。我们将探索两方（Alice 和 Bob）之间的简单单向支付渠道。它包括三个步骤：

			- Alice 用 Ether 为智能合约提供资金。这“打开”了支付渠道。
			- Alice签署消息，指明欠接收者多少以太币。每次付款都会重复此步骤。
			- Bob “关闭”支付渠道，提取他的部分以太币并将剩余部分发送回发送方。
		- 笔记

			只有第 1 步和第 3 步需要以太坊交易，第 2 步意味着发件人通过链下方法（例如电子邮件）将加密签名的消息传输给收件人。这意味着只需要两个事务即可支持任意数量的传输。

		Bob 可以保证收到他的资金，因为智能合约托管 Ether 并遵守有效的签名消息。智能合约还强制执行超时，因此即使收件人拒绝关闭通道，Alice 也可以保证最终收回她的资金。支付渠道的参与者可以决定将其开放多长时间。对于短期交易，例如为每分钟的网络访问支付网吧费用，支付渠道可能会在有限的时间内保持开放。另一方面，对于经常性付款，例如向员工支付每小时工资，付款渠道可能会保持开放数月或数年。

		- 开通支付渠道

			为了打开支付通道，Alice 部署了智能合约，附加了要托管的以太币，并指定了预期的接收者和通道存在的最长持续时间。这是 `SimplePaymentChannel` 合约中的功能 ，在本节末尾。
		- 付款

			Alice 通过向 Bob 发送签名消息来付款。此步骤完全在以太坊网络之外执行。消息由发送者加密签名，然后直接传输给接收者。

			每条消息都包含以下信息

			- 智能合约的地址，用于防止跨合约重放攻击。
			- 到目前为止欠接收方的以太币总量。

			在一系列转账结束时，支付渠道仅关闭一次。因此，只有一条发送的消息被赎回。这就是为什么每条消息都指定了所欠以太币的累计总量，而不是单个小额支付的金额。收件人自然会选择赎回最近的消息，因为那是总数最高的消息。不再需要每条消息的随机数，因为智能合约只接受一条消息。智能合约的地址仍然用于防止用于一个支付渠道的消息被用于不同的渠道。
		- 关闭支付渠道
			- 当 Bob 准备好接收他的资金时，是时候通过调用 `close` 智能合约上的函数来关闭支付通道了。
			- 关闭通道会向接收者支付他们所欠的以太币并销毁合约，将任何剩余的以太币发送回 Alice。要关闭通道，Bob 需要提供一条由 Alice 签名的消息。
			
			智能合约必须验证消息是否包含来自发件人的有效签名。进行此验证的过程与收件人使用的过程相同。Solidity 的功能 `isValidSignature` 和 `recoverSigner` 工作方式与上一节中的 JavaScript 对应物一样，后者的功能是从 `ReceiverPays` 合约中借用的。
			
			只有支付渠道接收方可以调用该 `close` 函数，他自然会传递最近的支付消息，因为该消息携带的欠款总额最高。如果允许发件人调用此函数，则他们可以提供较低金额的消息，并欺骗收件人以摆脱欠款。
			
			该函数验证签名消息与给定参数匹配。如果一切顺利，接收者会收到他们的一部分以太币，而发送者会通过 `selfdestruct`. 您可以 `close` 在完整合约中查看该功能。
		- 渠道到期

			Bob 可以随时关闭支付渠道，但如果他们不这样做，Alice 需要一种方法来收回她的托管资金。一个过期时间设置在合同部署的时间。一旦到了那个时间，Alice 就可以 call `claimTimeout` 收回她的资金。您可以 在完整合约中查看 `claimTimeout`  功能。

			调用此函数后，Bob 将无法再接收任何 Ether，因此 Bob 在到期之前关闭通道很重要
		- 完整合约

				// SPDX-License-Identifier: GPL-3.0
				pragma solidity >=0.7.0 <0.9.0;
				contract SimplePaymentChannel {
				    address payable public sender;      // The account sending payments.
				    address payable public recipient;   // The account receiving the payments.
				    uint256 public expiration;  // Timeout in case the recipient never closes.
				
				    constructor (address payable _recipient, uint256 duration)
				        payable
				    {
				        sender = payable(msg.sender);
				        recipient = _recipient;
				        expiration = block.timestamp + duration;
				    }
				
				    /// the recipient can close the channel at any time by presenting a
				    /// signed amount from the sender. the recipient will be sent that amount,
				    /// and the remainder will go back to the sender
				    function close(uint256 amount, bytes memory signature) public {
				        require(msg.sender == recipient);
				        require(isValidSignature(amount, signature));
				
				        recipient.transfer(amount);
				        selfdestruct(sender);
				    }
				
				    /// the sender can extend the expiration at any time
				    function extend(uint256 newExpiration) public {
				        require(msg.sender == sender);
				        require(newExpiration > expiration);
				
				        expiration = newExpiration;
				    }
				
				    /// if the timeout is reached without the recipient closing the channel,
				    /// then the Ether is released back to the sender.
				    function claimTimeout() public {
				        require(block.timestamp >= expiration);
				        selfdestruct(sender);
				    }
				
				    function isValidSignature(uint256 amount, bytes memory signature)
				        internal
				        view
				        returns (bool)
				    {
				        bytes32 message = prefixed(keccak256(abi.encodePacked(this, amount)));
				
				        // check that the signature is from the payment sender
				        return recoverSigner(message, signature) == sender;
				    }
				
				    /// All functions below this are just taken from the chapter
				    /// 'creating and verifying signatures' chapter.
				
				    function splitSignature(bytes memory sig)
				        internal
				        pure
				        returns (uint8 v, bytes32 r, bytes32 s)
				    {
				        require(sig.length == 65);
				
				        assembly {
				            // first 32 bytes, after the length prefix
				            r := mload(add(sig, 32))
				            // second 32 bytes
				            s := mload(add(sig, 64))
				            // final byte (first byte of the next 32 bytes)
				            v := byte(0, mload(add(sig, 96)))
				        }
				
				        return (v, r, s);
				    }
				
				    function recoverSigner(bytes32 message, bytes memory sig)
				        internal
				        pure
				        returns (address)
				    {
				        (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);
				
				        return ecrecover(message, v, r, s);
				    }
				
				    /// builds a prefixed hash to mimic the behavior of eth_sign.
				    function prefixed(bytes32 hash) internal pure returns (bytes32) {
				        return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
				    }
				}
		- 笔记

			该函数 `splitSignature` 不使用所有安全检查。真正的实现应该使用经过更严格测试的库，例如此代码的 `openzepplin` 版本
		- 验证付款

			与上一节不同，支付渠道中的消息不会立即兑现。收件人会跟踪最新消息并在需要关闭支付渠道时将其赎回。这意味着收件人对每封邮件执行自己的验证至关重要。否则无法保证收件人最终能够获得付款。
			
			收件人应使用以下过程验证每条消息：
			
			- 验证消息中的合约地址是否与支付渠道匹配。
			- 验证新总数是否为预期金额。
			- 验证新的总数不超过托管的以太币数量。
			- 验证签名是否有效并且来自支付渠道发件人。
			
			我们将使用 [ethereumjs-util](https://github.com/ethereumjs/ethereumjs-util) 库来编写此验证。最后一步可以通过多种方式完成，我们使用 JavaScript。以下代码借用了 `constructPaymentMessage` 上述签名 JavaScript 代码中的函数：
			
				// this mimics the prefixing behavior of the eth_sign JSON-RPC method.
				function prefixed(hash) {
				    return ethereumjs.ABI.soliditySHA3(
				        ["string", "bytes32"],
				        ["\x19Ethereum Signed Message:\n32", hash]
				    );
				}
				
				function recoverSigner(message, signature) {
				    var split = ethereumjs.Util.fromRpcSig(signature);
				    var publicKey = ethereumjs.Util.ecrecover(message, split.v, split.r, split.s);
				    var signer = ethereumjs.Util.pubToAddress(publicKey).toString("hex");
				    return signer;
				}
				
				function isValidSignature(contractAddress, amount, signature, expectedSigner) {
				    var message = prefixed(constructPaymentMessage(contractAddress, amount));
				    var signer = recoverSigner(message, signature);
				    return signer.toLowerCase() ==
				        ethereumjs.Util.stripHexPrefix(expectedSigner).toLowerCase();
				}				
- 模块化合同

	构建合同的模块化方法可帮助您降低复杂性并提高可读性，这将有助于在开发和代码审查期间识别错误和漏洞。如果您单独指定和控制行为或每个模块，则您必须考虑的交互只是模块规范之间的交互，而不是合约的所有其他移动部分。在下面的示例中，合约使用[库](https://docs.soliditylang.org/en/develop/contracts.html#libraries)的 `move` 方法来检查地址之间发送的余额是否符合您的预期。通过这种方式，该库提供了一个独立的组件，可以正确跟踪帐户余额。很容易验证 `Balances` ,库永远不会产生负余额或溢出，并且所有余额的总和在合约的整个生命周期内都是不变的。
	
		// SPDX-License-Identifier: GPL-3.0
		pragma solidity >=0.5.0 <0.9.0;
		
		library Balances {
		    function move(mapping(address => uint256) storage balances, address from, address to, uint amount) internal {
		        require(balances[from] >= amount);
		        require(balances[to] + amount >= balances[to]);
		        balances[from] -= amount;
		        balances[to] += amount;
		    }
		}
		
		contract Token {
		    mapping(address => uint256) balances;
		    using Balances for *;
		    mapping(address => mapping (address => uint256)) allowed;
		
		    event Transfer(address from, address to, uint amount);
		    event Approval(address owner, address spender, uint amount);
		
		    function transfer(address to, uint amount) public returns (bool success) {
		        balances.move(msg.sender, to, amount);
		        emit Transfer(msg.sender, to, amount);
		        return true;
		
		    }
		
		    function transferFrom(address from, address to, uint amount) public returns (bool success) {
		        require(allowed[from][msg.sender] >= amount);
		        allowed[from][msg.sender] -= amount;
		        balances.move(from, to, amount);
		        emit Transfer(from, to, amount);
		        return true;
		    }
		
		    function approve(address spender, uint tokens) public returns (bool success) {
		        require(allowed[msg.sender][spender] == 0, "");
		        allowed[msg.sender][spender] = tokens;
		        emit Approval(msg.sender, spender, tokens);
		        return true;
		    }
		
		    function balanceOf(address tokenOwner) public view returns (uint balance) {
		        return balances[tokenOwner];
		    }
		}

			
				
				




				 				