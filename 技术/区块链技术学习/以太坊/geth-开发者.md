# geth 开发者
## 对于 Dapp 开发者
###  EVM 追踪
以太坊中有两种不同类型的交易：

- 简单价值转移
- 合约执行

简单的价值转移只是将以太币从一个账户转移到另一个账户，因此从本指南的角度来看是无趣的。然而，如果交易的接收者是具有关联 EVM（以太坊虚拟机）字节码的合约账户（除了传输任何以太币，代码也将作为交易的一部分执行）。

拥有与以太坊账户相关联的代码允许交易进行任意复杂的数据存储，并通过与外部账户和合同在内部进一步交易，使它们能够对先前存储的数据采取行动。这创建了一个相互交织的合约生态系统，其中单个交易可以与数十或数百个账户进行交互。

合约执行的缺点是很难说交易实际上做了什么。交易收据确实包含一个状态代码，用于检查执行是否成功，但无法查看修改了哪些数据，也无法查看调用了哪些外部合同。为了内省一个事务，我们需要跟踪它的执行。

#### 跟踪先决条件
以最简单的形式，跟踪交易需要请求以太坊节点以不同程度的数据收集重新执行所需的交易，并让它返回汇总摘要以进行后期处理。然而，重新执行交易需要满足一些先决条件。

为了让以太坊节点重新执行交易，它需要拥有交易访问的所有历史状态：

- 接收方以及所有内部调用的合约的余额、随机数、字节码和存储
- 在执行外部以及所有内部创建的事务期间引用的块元数据
- 由包含在与被跟踪的块相同的块中的所有先前事务生成的中间状态

根据您节点的同步和修剪模式，不同的配置会产生不同的功能：

- 保留 `所有历史数据的存档` 节点可以在任何时间点跟踪任意交易。跟踪单个事务还需要在同一块中重新执行所有先前的事务。
- 一个`完全同步`的节点保留`所有历史数据` 初始同步后，只能从跟踪块的初始同步点以下交易。跟踪单个事务还需要在同一块中重新执行所有先前的事务。
- `快速同步`节点仅保留`周期性状态数据`,初始同步之后只能跟踪从块的初始同步点之后的交易。跟踪一个单个事务再次执行嗣继承所有先前的交易都在相同的块，以及直到前一存储的快照所有前面的块。
- 一个`按需`检索数据的`轻同步`节点,理论上可以跟踪所有必需的历史状态在网络中随时可用的交易。在实践中，数据可用性不是一个可行的假设。

在运行整个块或链段的批处理跟踪时，上述规则有例外。这些将在后面详述。

#### 基本痕迹
go-ethereum 可以生成的最简单的事务跟踪类型是原始 EVM 操作码跟踪。对于事务执行的每条 VM 指令，都会发出一个结构化日志条目，其中包含所有被认为有用的上下文元数据。这包括程序计数器、操作码名称、操作码成本、剩余气体、执行深度和任何 发生的错误。结构化日志还可以选择包含 执行堆栈、执行内存和合约存储的内容。

单个操作码的示例日志条目如下所示：	

	{
	  "pc":      48,
	  "op":      "DIV",
	  "gasCost": 5,
	  "gas":     64532,
	  "depth":   1,
	  "error":   null,
	  "stack": [
	    "00000000000000000000000000000000000000000000000000000000ffffffff",
	    "0000000100000000000000000000000000000000000000000000000000000000",
	    "2df07fbaabbe40e3244445af30759352e348ec8bebd4dd75467a9f29ec55d98d"
	  ],
	  "memory": [
	    "0000000000000000000000000000000000000000000000000000000000000000",
	    "0000000000000000000000000000000000000000000000000000000000000000",
	    "0000000000000000000000000000000000000000000000000000000000000060"
	  ],
	  "storage": {
	  }
	}
原始 EVM 操作码跟踪的整个输出是一个 JSON 对象，具有几个元数据字段：

- 消耗的气体
- 故障状态
- 返回值
- 以及 采用上述形式的操作码条目列表

如同

	{
	  "gas":         25523,
	  "failed":      false,
	  "returnValue": "",
	  "structLogs":  []
	}

#### 生成基本跟踪
为了生成原始 EVM 操作码跟踪，go-ethereum 提供了一些 [RPC API](https://geth.ethereum.org/docs/rpc/ns-debug) 端点，其中最常用的是 [debug_traceTransaction](https://geth.ethereum.org/docs/rpc/ns-debug#debug_tracetransaction).

以最简单的形式，`traceTransaction` 接受交易哈希作为其唯一参数，跟踪交易，聚合所有生成的数据并将其作为大型 JSON 对象返回。来自 Geth 控制台的示例调用是：

	debug.traceTransaction("0xfc9359e49278b7ba99f59edac0e3de49956e46e530a53c15aa71226b7aa92c6f")
当然也可以通过 HTTP RPC 从节点外部调用相同的调用。在这种情况下，请确保HTTP端点通过启用 `--http` 和 `debug` API命名空间通过暴露 `--http.api=debug`。

	$ curl -H "Content-Type: application/json" -d '{"id": 1, "method": "debug_traceTransaction", "params": ["0xfc9359e49278b7ba99f59edac0e3de49956e46e530a53c15aa71226b7aa92c6f"]}' localhost:8545
	
在 Rinkeby 网络上运行上述操作（节点保留足够的历史记录）将导致此[跟踪转储](https://gist.github.com/karalabe/c91f95ac57f5e57f8b950ec65ecc697f)。	

#### 调整基本轨迹
默认情况下，原始操作码跟踪器会在处理事务时发出 EVM 内发生的所有相关事件，例如

- EVM 堆栈
- EVM 内存
- 更新的存储槽

然而，某些用例可能不需要报告这些数据字段中的一些。为了满足这些用例，可以使用跟踪器的第二个选项参数省略这些大量字段：

	{
	  "disableStack": true,
	  "disableMemory": true,
	  "disableStorage": true
	}

在禁用数据字段的情况下从 Geth 控制台运行先前的跟踪器调用：

	debug.traceTransaction("0xfc9359e49278b7ba99f59edac0e3de49956e46e530a53c15aa71226b7aa92c6f", {disableStack: true, disableMemory: true, disableStorage: true})
也可以通过 HTTP RPC 从节点外部运行过滤后的跟踪器：

	$ curl -H "Content-Type: application/json" -d '{"id": 1, "method": "debug_traceTransaction", "params": ["0xfc9359e49278b7ba99f59edac0e3de49956e46e530a53c15aa71226b7aa92c6f", {"disableStack": true, "disableMemory": true, "disableStorage": true}]}' localhost:8545
	
在 Rinkeby 网络上运行上述操作将导致此[跟踪转储](https://gist.github.com/karalabe/d74a7cb33a70f2af75e7824fc772c5b4)显着缩短。		
#### 基本迹线的限制
尽管我们上面生成的原始操作码跟踪有其用途，但这种基本的跟踪方式在现实世界中是有问题的。对于大多数用例来说，为每个操作码设置单独的日志条目的级别太低，并且需要开发人员创建额外的工具来对跟踪进行后处理。此外，完整的操作码跟踪很容易达到数百兆字节，这使得它们非常耗费资源，无法离开节点并在外部进行处理。

为了避免前面提到的所有问题，go-ethereum 支持在以太坊节点内运行自定义 JavaScript 跟踪器，这些跟踪器可以完全访问 EVM 堆栈、内存和合约存储。这允许开发人员仅收集他们需要的数据，以及做任何处理的数据。请参阅下一节了解我们的自定义节点内跟踪器。
#### 修剪
Geth 默认对状态进行内存修剪，丢弃它认为不再需要维护的状态条目。这是通过 `--gcmode` 选项配置的。通常，人们会遇到状态不可用的错误。

假设您想对 block 进行跟踪 B。以下是每种情况下发生的情况：

- 您已经完成了一个快速同步的枢轴块 P，其中P <= B.
	- Geth 将通过 B 在它具有完整状态之前的最近点重放块来重新生成所需的状态。这默认为 `128` blocks max，但您可以在 `... "reexec":1000 .. }` 对跟踪器的实际调用中指定更多。
- 您已经完成了一个快速同步的枢轴块 P，其中P > B.
	- 抱歉，如果不从 genesis 重播就无法完成。
- 您已经完成了完全同步，并进行了修剪
	同 1)
- 您已经完成了完全同步，没有修剪 ( `--gcmode=archive`)
	- 不需要重放任何东西，可以立即加载状态并为请求提供服务。

### 过滤追踪
上面学习了如何创建完整的跟踪。但是，这些跟踪可以包括执行过程中每个点的 EVM 的完整状态，这是巨大的。通常您只对这些信息的一小部分感兴趣。要获得它，您可以指定一个 JavaScript 过滤器。

注意： Geth 使用的 JavaScript 解释器是 [duktape](https://duktape.org/)，仅达到 [ECMAScript 5.1 标准](https://262.ecma-international.org/5.1/)。这意味着我们不能使用 [箭头函数](https://www.w3schools.com/js/js_arrow_function.asp) 和 [模板文字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)。

#### 运行简单的跟踪
1. 创建一个 `filterTrace_1.js` 包含以下内容的文件：

		tracer = function(tx) {
		   return debug.traceTransaction(tx, {tracer:
		      '{' +
		         'retVal: [],' +
		         'step: function(log,db) {this.retVal.push(log.getPC() + ":" + log.op.toString())},' +
		         'fault: function(log,db) {this.retVal.push("FAULT: " + JSON.stringify(log))},' +
		         'result: function(ctx,db) {return this.retVal}' +
		      '}'
		   }) // return debug.traceTransaction ...
		}   // tracer = function ...

	可以直接在 JavaScript 控制台中指定这个函数，但它会很笨拙且难以编辑。
2. 运行 [JavaScript 控制台](https://geth.ethereum.org/docs/interface/javascript-console)。
3. 获取最近交易的哈希值。比如你使用Goerli网络，你可以[在这里](https://goerli.etherscan.io/)得到这样的值 。
4. 运行此命令以运行脚本：

		loadScript("filterTrace_1.js")
5. 从脚本运行跟踪器。请耐心等待，这可能需要很长时间。

		tracer("<hash of transaction>")
	输出的底部类似于：

		"3366:POP", "3367:JUMP", "1355:JUMPDEST", "1356:PUSH1", "1358:MLOAD", "1359:DUP1", "1360:DUP3", "1361:ISZERO", "1362:ISZERO",
		"1363:ISZERO", "1364:ISZERO", "1365:DUP2", "1366:MSTORE", "1367:PUSH1", "1369:ADD", "1370:SWAP2", "1371:POP", "1372:POP", "1373:PUSH1",
		"1375:MLOAD", "1376:DUP1", "1377:SWAP2", "1378:SUB", "1379:SWAP1", "1380:RETURN"]
6. 这个输出不是很可读。运行此行以获得更易读的输出，每个字符串在其自己的行中。

		console.log(JSON.stringify(tracer("<hash of transaction>"), null, 2))

您可以[在此](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)处阅读有关该`JSON.stringify` 功能的信息。如果我们只返回 `\n` 换行符的输出，这就是为什么我们需要使用 `console.log`.

#### 它是如何工作的？
我们调用与[基本跟踪](https://geth.ethereum.org/docs/dapp/tracing)相同的 `debug.traceTransaction` 函数，但使用了一个新参数。这个参数是一个字符串，它是我们使用的 JavaScript 对象。在上面的跟踪的情况下，它是：`tracer`

	{
	   retVal: [],
	   step: function(log,db) {this.retVal.push(log.getPC() + ":" + log.op.toString())},
	   fault: function(log,db) {this.retVal.push("FAULT: " + JSON.stringify(log))},
	   result: function(ctx,db) {return this.retVal}
	}
这个对象必须有三个成员函数：

- `step`, 为每个操作码调用
- `fault`, 执行有问题时调用
- ` result`, 调用以产生 `debug.traceTransaction` 执行完成后返回的结果

它可以有额外的成员。在这种情况下，我们使用 `retVal` 来存储我们将在中返回的字符串列表 `result` 。

`step` 这里的函数添加到 `retVal` 程序计数器和那里的操作码的名称。然后，在中 `result`，我们返回要发送给调用者的这个列表。

#### 实际过滤
对于实际的过滤跟踪，我们需要一个 if 语句来仅记录相关信息。

例如，如果我们对事务与存储的交互感兴趣

- 执行脚本

		tracer = function(tx) {
		      return debug.traceTransaction(tx, {tracer:
		      '{' +
		         'retVal: [],' +
		         'step: function(log,db) {' +
		         '   if(log.op.toNumber() == 0x54) ' +
		         '     this.retVal.push(log.getPC() + ": SLOAD");' +
		         '   if(log.op.toNumber() == 0x55) ' +
		         '     this.retVal.push(log.getPC() + ": SSTORE");' +
		         '},' +
		         'fault: function(log,db) {this.retVal.push("FAULT: " + JSON.stringify(log))},' +
		         'result: function(ctx,db) {return this.retVal}' +
		      '}'
		      }) // return debug.traceTransaction ...
		}   // tracer = function ...

	`step` 此处的函数查看操作的操作码编号，并且仅在操作码为 `SLOAD` or 时才推送条目 `SSTORE`（[这里是 EVM 操作码及其编号的列表](https://github.com/wolflo/evm-opcodes)）。我们本来可以用 `log.op.toString()` 它来代替，但比较数字而不是字符串会更快。

- 输出类似于以下内容：

		[
		  "5921: SLOAD",
		  .
		  .
		  .
		  "2413: SSTORE",
		  "2420: SLOAD",
		  "2475: SSTORE",
		  "6094: SSTORE"
		]

#### 堆栈信息
上面的跟踪告诉我们程序计数器 (PC) 以及程序是从存储中读取还是写入。那不是很有用。要了解更多信息，您可以使用该 `log.stack.peek` 函数查看堆栈。

- `log.stack.peek(0)` 是栈顶
- `log.stack.peek(1)` 它下面的条目等
- 返回的值 `log.stack.peek` 是 Gobig.Int 对象。

默认情况下，它们会转换为 JavaScript 浮点数，因此您需要 toString(16) 将它们作为十六进制来获取，这就是我们通常表示 256 位值（例如存储单元及其内容）的方式。

	tracer = function(tx) {
	      return debug.traceTransaction(tx, {tracer:
	      '{' +
	         'retVal: [],' +
	         'step: function(log,db) {' +
	         '   if(log.op.toNumber() == 0x54) ' +
	         '     this.retVal.push(log.getPC() + ": SLOAD " + ' +
	         '        log.stack.peek(0).toString(16));' +
	         '   if(log.op.toNumber() == 0x55) ' +
	         '     this.retVal.push(log.getPC() + ": SSTORE " +' +
	         '        log.stack.peek(0).toString(16) + " <- " +' +
	         '        log.stack.peek(1).toString(16));' +
	         '},' +
	         'fault: function(log,db) {this.retVal.push("FAULT: " + JSON.stringify(log))},' +
	         'result: function(ctx,db) {return this.retVal}' +
	      '}'
	      }) // return debug.traceTransaction ...
	}   // tracer = function ...
此功能可让您跟踪所有存储操作，并显示它们的参数。这让您更全面地了解程序与存储的交互。输出类似于：

	[
	  "5921: SLOAD 0",
	  .
	  .
	  .
	  "2413: SSTORE 3f0af0a7a3ed17f5ba6a93e0a2a05e766ed67bf82195d2dd15feead3749a575d <- fb8629ad13d9a12456",
	  "2420: SLOAD cc39b177dd3a7f50d4c09527584048378a692aed24d31d2eabeddb7f3c041870",
	  "2475: SSTORE cc39b177dd3a7f50d4c09527584048378a692aed24d31d2eabeddb7f3c041870 <- 358c3de691bd19",
	  "6094: SSTORE 0 <- 1"
	]

#### 成果
上述函数中缺少的一条信息是 `SLOAD` 操作的结果。我们进入 log 的状态是执行操作码之前的状态，因此该值尚不可知。对于更多的操作，我们可以自己解决，但是我们无法访问存储，所以在这里我们不能。

解决方案是有一个标志，`afterSload`，它只在操作码中紧跟在 a 之后才为真` SLOAD`，当我们可以在堆栈顶部看到结果时。

	tracer = function(tx) {
	      return debug.traceTransaction(tx, {tracer:
	      '{' +
	         'retVal: [],' +
	         'afterSload: false,' +
	         'step: function(log,db) {' +
	         '   if(this.afterSload) {' +
	         '     this.retVal.push("    Result: " + ' +
	         '          log.stack.peek(0).toString(16)); ' +
	         '     this.afterSload = false; ' +
	         '   } ' +
	         '   if(log.op.toNumber() == 0x54) {' +
	         '     this.retVal.push(log.getPC() + ": SLOAD " + ' +
	         '        log.stack.peek(0).toString(16));' +
	         '        this.afterSload = true; ' +
	         '   } ' +
	         '   if(log.op.toNumber() == 0x55) ' +
	         '     this.retVal.push(log.getPC() + ": SSTORE " +' +
	         '        log.stack.peek(0).toString(16) + " <- " +' +
	         '        log.stack.peek(1).toString(16));' +
	         '},' +
	         'fault: function(log,db) {this.retVal.push("FAULT: " + JSON.stringify(log))},' +
	         'result: function(ctx,db) {return this.retVal}' +
	      '}'
	      }) // return debug.traceTransaction ...
	}   // tracer = function ...
输出现在包含在 `SLOAD`. 我们也可以修改 `SLOAD` 生产线本身，但这需要更多的工作。

	[
	  "5921: SLOAD 0",
	  "    Result: 1",
	  .
	  .
	  .
	  "2413: SSTORE 3f0af0a7a3ed17f5ba6a93e0a2a05e766ed67bf82195d2dd15feead3749a575d <- fb8629ad13d9a12456",
	  "2420: SLOAD cc39b177dd3a7f50d4c09527584048378a692aed24d31d2eabeddb7f3c041870",
	  "    Result: 0",
	  "2475: SSTORE cc39b177dd3a7f50d4c09527584048378a692aed24d31d2eabeddb7f3c041870 <- 358c3de691bd19",
	  "6094: SSTORE 0 <- 1"
	]
#### 处理合约之间的调用
到目前为止，我们已经将存储视为只有 2^256 个单元。然而，事实并非如此。合约可以调用其他合约，那么涉及的存储就是其他合约的存储。我们可以在中看到当前合约的地址 `log.contract.getAddress()`。该值是执行上下文，即我们正在使用其存储的合约，即使我们使用来自另一个合约的代码（通过使用 `CALLCODE` 或 `DELEGATECODE`）。

但是，`log.contract.getAddress()` 返回一个字节数组。我们使用 `this.byteHex()` 和 `array2Hex() `将这个数组转换为我们通常用来识别合约的十六进制表示。

	tracer = function(tx) {
	      return debug.traceTransaction(tx, {tracer:
	      '{' +
	         'retVal: [],' +
	         'afterSload: false,' +
	         'callStack: [],' +
	
	         'byte2Hex: function(byte) {' +
	         '  if (byte < 0x10) ' +
	         '      return "0" + byte.toString(16); ' +
	         '  return byte.toString(16); ' +
	         '},' +
	
	         'array2Hex: function(arr) {' +
	         '  var retVal = ""; ' +
	         '  for (var i=0; i<arr.length; i++) ' +
	         '    retVal += this.byte2Hex(arr[i]); ' +
	         '  return retVal; ' +
	         '}, ' +
	
	         'getAddr: function(log) {' +
	         '  return this.array2Hex(log.contract.getAddress());' +
	         '}, ' +
	
	         'step: function(log,db) {' +
	         '   var opcode = log.op.toNumber();' +
	
	         // SLOAD
	         '   if (opcode == 0x54) {' +
	         '     this.retVal.push(log.getPC() + ": SLOAD " + ' +
	         '        this.getAddr(log) + ":" + ' +
	         '        log.stack.peek(0).toString(16));' +
	         '        this.afterSload = true; ' +
	         '   } ' +
	
	         // SLOAD Result
	         '   if (this.afterSload) {' +
	         '     this.retVal.push("    Result: " + ' +
	         '          log.stack.peek(0).toString(16)); ' +
	         '     this.afterSload = false; ' +
	         '   } ' +
	
	         // SSTORE
	         '   if (opcode == 0x55) ' +
	         '     this.retVal.push(log.getPC() + ": SSTORE " +' +
	         '        this.getAddr(log) + ":" + ' +
	         '        log.stack.peek(0).toString(16) + " <- " +' +
	         '        log.stack.peek(1).toString(16));' +
	
	         // End of step
	         '},' +
	
	         'fault: function(log,db) {this.retVal.push("FAULT: " + JSON.stringify(log))},' +
	
	         'result: function(ctx,db) {return this.retVal}' +
	      '}'
	      }) // return debug.traceTransaction ...
	}   // tracer = function ...
输出类似于：

	[
	  "423: SLOAD 22ff293e14f1ec3a09b137e9e06084afd63addf9:360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc",
	  "    Result: 360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc",
	  "10778: SLOAD 22ff293e14f1ec3a09b137e9e06084afd63addf9:6",
	  "    Result: 6",
	  .
	  .
	  .
	  "13529: SLOAD f2d68898557ccb2cf4c10c3ef2b034b2a69dad00:8328de571f86baa080836c50543c740196dbc109d42041802573ba9a13efa340",
	  "    Result: 8328de571f86baa080836c50543c740196dbc109d42041802573ba9a13efa340",
	  "423: SLOAD f2d68898557ccb2cf4c10c3ef2b034b2a69dad00:360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc",
	  "    Result: 360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc",
	  "13529: SLOAD f2d68898557ccb2cf4c10c3ef2b034b2a69dad00:b38558064d8dd9c883d2a8c80c604667ddb90a324bc70b1bac4e70d90b148ed4",
	  "    Result: b38558064d8dd9c883d2a8c80c604667ddb90a324bc70b1bac4e70d90b148ed4",
	  "11041: SSTORE 22ff293e14f1ec3a09b137e9e06084afd63addf9:6 <- 0"
	]
#### 结论
本教程仅教授了使用 JavaScript 过滤跟踪的基础知识。我们没有讨论访问内存或者如何使用db参数来知道执行时链的状态。所有这些以及更多内容都包含在参考资料中。

希望使用此工具，您会发现更容易跟踪 EVM 的行为和调试棘手的合同问题。



### GOAPI
以太坊区块链及其两个扩展协议 Whisper 和 Swarm 最初被概念化为 web3 的支柱，为称为 DApps 的新一代分布式（实际上是去中心化）应用程序提供共识、消息传递和存储主干。

实现这个 web3 梦想的第一个化身是一个命令行客户端，它为点对点协议提供了一个 RPC 接口。客户端很快就扩展了类似网络浏览器的图形用户界面，允许开发人员基于久经考验的 HTML/CSS/JS 技术编写 DApp。

由于许多 DApp 具有比浏览器环境可以处理的更复杂的要求，因此很明显，提供对 web3 支柱的编程访问将为新类别的应用程序打开大门。因此，web3 梦想的第二个化身是将我们所有的技术作为可重用组件开放给其他项目。

从 1.5 版本的 `go-ethereum`.

请注意，本指南假设您熟悉 Go 开发。它不会尝试涵盖有关 Go 项目布局、导入路径或任何其他标准方法的一般主题。如果您是 Go 新手，请考虑先阅读其[入门指南](https://github.com/golang/go/wiki#getting-started-with-go)。
#### 快速概览
我们的可重用 Go 库专注于四个主要使用领域

- 简化的客户端帐户管理
- 通过不同传输的远程节点接口
- 通过自动生成的绑定进行合同交互
- 进程内以太坊、Whisper 和 Swarm 对等节点

您可以在 Peter (@karalabe) 在 2016 年 9 月（上海）举行的 Ethereum Devcon2 开发者大会上发表题为“Import Geth: Ethereum from Go and Beyond”的演讲中观看有关这些的简要概述。此处提供[幻灯片](https://ethereum.karalabe.com/talks/2016-devcon.html)和[Peter 的 Devcon2 演讲](https://www.youtube.com/watch?v=R0Ia1U9Gxjg)

#### go 包
该 ·go-ethereum· 库直接从我们的 GitHub 存储库作为标准 Go 包的集合分发。这些包可以通过官方 Go 工具包直接使用，无需任何第三方工具。外部依赖项在本地供应到中 `vendor`，以确保自包含和代码稳定性。如果您` go-ethereum` 在自己的项目中重用 ，请遵循这些最佳实践并自己提供，以避免任何意外的 API 损坏！

的规范导入路径 `go-ethereum` 是 `github.com/ethereum/go-ethereum` ，所有包都驻留在下面,尽管有[很多](https://pkg.go.dev/github.com/ethereum/go-ethereum/#section-directories)，但您只需要关心有限的子集，每个子​​集都将在其相关部分中进行适当介绍。

您可以通过以下方式下载我们所有的软件包：

	$ go get -d github.com/ethereum/go-ethereum/...
	
### go 账户管理
要为您的本机应用程序提供以太坊集成，您应该感兴趣的第一件事是帐户管理。

尽管当前所有领先的以太坊实现都提供内置的帐户管理，但不建议将帐户保存在多个应用程序和/或多人共享的任何位置。与您不将登录凭据委托给您的 ISP（毕竟是您进入互联网的网关）的方式相同；您也不应该将您的凭据委托给以太坊节点（它是您进入以太坊网络的网关）。

在您的本机应用程序中处理用户帐户的正确方法是进行客户端帐户管理，所有内容都包含在您自己的应用程序中。通过这种方式，您可以根据需要确保对敏感数据的细粒度（或粗略）访问权限，而无需依赖任何第三方应用程序的功能和/或漏洞。

为了支持这一点，go-ethereum 提供了一个简单而全面的帐户包，为您提供所有工具，通过加密的密钥库和密码保护帐户进行适当的安全帐户管理。您可以利用 go-ethereum 加密实现的所有安全性，同时在您自己的应用程序中运行所有内容。
#### 加密密钥库
尽管在应用程序本地处理帐户确实提供了某些安全保证，但以太坊帐户的访问密钥不应以明文形式存在。因此，我们提供了一个加密的密钥库，它为您提供适当的安全保证，而无需您对相关加密原语的部分有透彻的了解。

使用加密密钥库时要知道的重要一点是，其中使用的加密原语可以在标准模式或轻型模式下运行。前者以增加计算负担和资源消耗为代价提供更高级别的安全性：

- 标准需要 256MB 内存和现代 CPU 上的 1 秒处理才能访问密钥
- light需要 4MB 内存和现代 CPU 上的 100 毫秒处理才能访问密钥

因此，标准更适合本机应用程序，但如果您的目标是更多资源受限的环境，您仍然应该了解权衡。

对于那些对加密和/或实现细节感兴趣的人，密钥库使用[高效加密标准](https://www.secg.org/sec2-v2.pdf)中 `secp256k1` 定义的椭圆曲线，由库实现并由 . 帐户以 [Web3 Secret Storage](https://github.com/ethereum/wiki/wiki/Web3-Secret-Storage-Definition) 格式存储在磁盘上。

- [libsecp256k](https://github.com/bitcoin-core/secp256k1)
- [github.com/ethereum/go-ethereum/accounts](https://godoc.org/github.com/ethereum/go-ethereum/accounts)

#### 来自 Go 的密钥库
加密密钥库由包中的 [accounts.Manager](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager) 结构体实现 [github.com/ethereum/go-ethereum/accounts](https://godoc.org/github.com/ethereum/go-ethereum/accounts)，其中还包含上述标准或轻型 安全模式的配置常量。因此，要从 Go 进行客户端帐户管理，您只需要将 `accounts` 包导入到您的代码中：

	import "github.com/ethereum/go-ethereum/accounts"
	import "github.com/ethereum/go-ethereum/accounts/keystore"
	import "github.com/ethereum/go-ethereum/common"
之后，您可以通过以下方式创建新的加密帐户管理器：

	ks := keystore.NewKeyStore("/path/to/keystore", keystore.StandardScryptN, keystore.StandardScryptP)
	am := accounts.NewManager(&accounts.Config{InsecureUnlockAllowed: false}, ks)
密钥库文件夹的路径必须是本地用户可写但其他系统用户不可读取的位置（显然出于安全原因），因此我们建议将其放置在用户的主目录中，或者更加锁定用于后端应用程序。

最后两个参数 [keystore.NewKeyStore](https://godoc.org/github.com/ethereum/go-ethereum/accounts/keystore#NewKeyStore) 是定义密钥库加密的资源密集程度的加密参数。您可以在 [accounts.StandardScryptN ,accounts.StandardScrypt,Paccounts.LightScryptN, accounts.LightScryptP](https://godoc.org/github.com/ethereum/go-ethereum/accounts#pkg-constants) 之间进行选择或指定您自己的数字（请确保您了解相关的底层密码学）。我们建议使用标准版本。
#### 帐户生命周期
为您的以太坊账户创建了一个加密的密钥库后，您可以使用这个账户管理器来满足您本地应用程序的整个账户生命周期要求,包括

- 创建新帐户
- 删除现有帐户的基本功能
- 以及更新访问凭证
- 导出现有帐户
- 喝将它们导入到另一台设备的更高级功能。

尽管密钥库定义了用于存储您的帐户的加密强度，但没有可以授予对所有帐户的访问权限的全局主密码。相反，每个帐户都是单独维护的，并以 [加密格式](https://github.com/ethereum/wiki/wiki/Web3-Secret-Storage-Definition) 单独存储在磁盘上，确保更清晰、更严格地分离凭据。

然而，这种个性意味着任何需要访问帐户的操作都需要以密码短语的形式为该特定帐户提供必要的身份验证凭据：

- 创建新帐户时，调用者必须提供密码来加密帐户。任何后续访问都需要此密码，缺少密码将永远无法使用新创建的帐户。
- 删除现有帐户时，调用者必须提供密码来验证帐户的所有权。这不是加密必要的，而是防止帐户意外丢失的保护措施。
- 更新现有帐户时，调用者必须提供当前密码和新密码。完成操作后，将无法再通过旧密码访问该帐户。
- 导出现有帐户时，调用者必须提供当前密码来解密帐户，以及在将密钥文件返回给用户之前重新加密的导出密码。这是允许在机器和应用程序之间移动帐户而无需共享原始凭据所必需的。
- 导入新帐户时，调用者必须提供要导入的密钥文件的加密密码，以及用于存储帐户的新密码。这是允许使用与用于移动它们的凭据不同的凭据存储帐户所必需的。

请注意，没有丢失密码短语的恢复机制。加密密钥库的加密属性（如果使用提供的参数）保证帐户凭据不会在任何有意义的时间内被强制执行。

#### 来自 Go 的帐户
以太坊帐户由包中的 [accounts.Account](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Account) 结构实现 [github.com/ethereum/go-ethereum/accounts](https://godoc.org/github.com/ethereum/go-ethereum/accounts)。假设我们已经有了上一节中 [accounts.Manager](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager) 调用的实例 `am`，我们可以通过少量函数调用（省略错误处理）轻松执行所有描述的生命周期操作。

	// Create a new account with the specified encryption passphrase. // 使用指定的加密密码创建一个新帐户。
	newAcc, _ := ks.NewAccount("Creation password")
	fmt.Println(newAcc)
	
	// Export the newly created account with a different passphrase. The returned// 使用不同的密码导出新创建的帐户。 此方法调用返回的数据是 JSON 编码的加密密钥文件。
	// data from this method invocation is a JSON encoded, encrypted key-file. 
	jsonAcc, _ := ks.Export(newAcc, "Creation password", "Export password")
	
	// Update the passphrase on the account created above inside the local keystore. // 更新上面在本地密钥库中创建的帐户的密码。
	_ = ks.Update(newAcc, "Creation password", "Update password")
	
	// Delete the account updated above from the local keystore. // 从本地密钥库中删除上面更新的帐户。
	_ = ks.Delete(newAcc, "Update password")
	
	// Import back the account we've exported (and then deleted) above with yet // 使用新的密码重新导入我们在上面导出（然后删除）的帐户。
	// again a fresh passphrase.
	impAcc, _ := ks.Import(jsonAcc, "Export password", "Import password")
尽管 的实例 [accounts.Account](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Account) 可用于访问有关特定以太坊帐户的各种信息，但它们不包含任何敏感数据（例如密码或私钥），而仅用作客户端代码和密钥库的标识符。

#### 签署授权
如上所述，帐户对象不持有相关以太坊帐户的敏感私钥，而只是用于标识加密密钥的占位符。所有需要授权的操作（例如交易签名）都由账户管理器在授予其访问私钥的权限后执行。

有几种不同的方式可以授权客户经理执行签名操作，每种方式都有其优点和缺点。由于不同的方法具有截然不同的安全保证，因此必须清楚每种方法的工作原理：

- 单一授权

	通过账户管理器签署交易的最简单方法是在每次需要签名时提供账户的密码，这将临时解密私钥，执行签名操作并立即丢弃解密的密钥。
	
	- 缺点

		是每次都需要从用户那里查询密码，如果经常这样做会很烦人；或者应用程序需要将密码保存在内存中，如果操作不当，可能会产生安全后果；并且根据密钥库的配置强度，不断解密密钥可能会导致不可忽视的资源需求。
- 多重授权

	通过账户管理器签署交易的一种更复杂的方法是通过其密码解锁账户一次，并允许账户管理器缓存解密的私钥，从而无需密码即可完成所有后续签名请求。缓存私钥的生命周期可以手动管理（通过明确锁定帐户备份）或自动管理（通过在解锁期间提供超时）。这种机制对于用户可能需要签署许多交易或应用程序需要在不需要用户输入的情况下签署的场景很有用。
	
		要记住的关键方面是，任何有权访问帐户管理器的人都可以在特定帐户解锁时签署交易（例如，应用程序运行不受信任的代码）。
	请注意，创建交易超出了此处的范围，因此本节的其余部分将假设我们已经有一个交易哈希要签名，并且将只关注创建授权它的加密签名。稍后将介绍创建实际交易并将授权签名注入其中。

#### 从 Go 签名
假设我们已经有一个从前面部分 [accounts.Manager](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager) 调用的实例 am，我们可以创建一个新帐户来通过它已经演示的 [NewAccount](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager.NewAccount) 方法签署交易 ；并且为了暂时避免进入交易创建，我们可以硬编码一个随机 [common.Hash](https://godoc.org/github.com/ethereum/go-ethereum/common#Hash) 签名。

	// Create a new account to sign transactions with  // 创建一个新账户来签署交易
	signer, _ := ks.NewAccount("Signer password")
	txHash := common.HexToHash("0x0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef")
随着样板的消失，我们现在可以使用上述授权机制签署交易：

	// Sign a transaction with a single authorization // 使用单一授权签署交易
	signature, _ := ks.SignHashWithPassphrase(signer, "Signer password", txHash.Bytes())
	
	// Sign a transaction with multiple manually cancelled authorizations // 签署具有多个手动取消授权的交易
	_ = ks.Unlock(signer, "Signer password")
	signature, _ = ks.SignHash(signer, txHash.Bytes())
	_ = ks.Lock(signer.Address)
	
	// Sign a transaction with multiple automatically cancelled authorizations // 签署多条自动取消授权的交易
	_ = ks.TimedUnlock(signer, "Signer password", time.Second)
	signature, _ = ks.SignHash(signer, txHash.Bytes())
您可能想知道为什么 [SignWithPassphrase](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager.SignWithPassphrase) 将一个 [accounts.Account](accounts.Account) 作为签名者，而 [Sign](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager.Sign) 只将 [common.Address](https://godoc.org/github.com/ethereum/go-ethereum/common#Address). 原因是 [accounts.Account](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Account) 对象还可能包含自定义密钥路径，允许 [SignWithPassphrase](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager.SignWithPassphrase) 使用密钥库之外的帐户进行签名；但是 [Sign](https://godoc.org/github.com/ethereum/go-ethereum/accounts#Manager.Sign) 依赖于已经在密钥库中解锁的帐户，因此它无法指定自定义路径。

### go合同绑定
	请注意，事件尚未实现，因为它们需要一些仍在审查中的 RPC 订阅功能。

以太坊平台的最初路线图或梦想是提供一个可靠的、高性能的各种语言的共识协议客户端实现，这将为 JavaScript DApps 提供一个 RPC 接口进行通信，朝着 Mist 浏览器的方向发展，用户可以通过它与区块链进行交互。

尽管这是主流采用的可靠计划，并且确实涵盖了人们提出的大量用例（主要是人们手动与区块链交互的地方），但它避开了服务器端（后端、全自动、devops）用例，其中鉴于 JavaScript 的动态特性，它通常不是首选语言。

本页面介绍了服务器端原生 Dapps 的概念：Go 语言绑定到任何编译时类型安全、高性能和最好的以太坊合约，可以从合约 ABI 和可选的 EVM 字节码完全自动生成。

本页面以对初学者更友好的教程风格编写，使人们更容易开始编写 Go 原生 Dapp。使用的概念将随着开发人员需要遇到它们而逐渐引入。但是，我们确实假设读者大体上熟悉以太坊，对 Solidity 有一定的了解并且可以编写 Go。
#### 代币合约
为了避免陷入无用的学术例子的谬论，我们将以官方的 Token 合约为基础，引入 Go 原生绑定。如果您不熟悉合同，浏览链接页面可能就足够了，详细信息暂时不相关。简而言之，合约实现了一个可以部署在以太坊之上的自定义代币。为了确保本教程在链接网站更改时不会过时，令牌合约的 Solidity 源代码也可在 [token.sol](https://gist.github.com/karalabe/08f4b780e01c8452d989).
#### 去绑定生成器
通过以太坊客户端公开的 RPC 接口，已经可以通过 Go（或事实上的任何其他语言）与以太坊区块链上的合约进行交互。然而，编写将合适的 Go 语言构造转换为 RPC 调用并返回的样板代码非常耗时且非常脆弱：只能在运行时检测到实现错误，几乎不可能发展合约，因为即使 Solidity 中的微小变化也可以移植到 Go 会很痛苦。

为了避免所有这些混乱，go-ethereum 实现引入了一个源代码生成器，可以将 Ethereum ABI 定义转换为易于使用、类型安全的 Go 包。假设您设置了有效的 Go 开发环境并且正确检出了 go-ethereum 存储库，您可以使用以下命令构建生成器：

	$ cd $GOPATH/src/github.com/ethereum/go-ethereum
	$ go build ./cmd/abigen
#### 生成绑定
生成与以太坊合约的 Go 绑定所需的唯一基本内容是合约的 ABI 定义 JSON 文件。对于我们的 Token 合约教程，您可以通过自己编译 Solidity 代码（例如通过@chriseth 的[在线Solidity编译器](https://chriseth.github.io/browser-solidity/)）来获得它，或者您可以下载我们的预编译[token.abi](https://gist.github.com/karalabe/b8dfdb6d301660f56c1b).

要生成绑定，只需调用：

	$ abigen --abi token.abi --pkg main --type Token --out token.go
参数意义

	--abi：要绑定到的合约 ABI 的强制性路径
	--pkg: 将 Go 代码放入的强制 Go 包名
	--type: 可选的 Go 类型名称以分配给绑定结构
	--out: 生成的 Go 源文件的可选输出路径（未设置 = stdout）
这将为 Token 合约生成一个类型安全的 Go 绑定。生成的代码看起来像 `token.go`，但请生成您自己的代码，因为随着更多的工作投入到生成器中，这会发生变化。
#### 访问以太坊合约
要与部署在区块链上的合约进行交互，您需要知道合约本身的 `address`  ，并需要指定 `backend` 访问以太坊的 。绑定生成器提供开箱即用的 RPC 后端，您可以通过它通过 IPC、HTTP 或 WebSockets 连接到现有的以太坊节点。

我们将使用基金会在测试网上部署的 Unicorn 代币合约来演示调用合约方法。它部署在地址 `0x21e6fc92f93c8a1bb41e2be64b4e1f88a54d3576`。

要运行下面的代码片段，请确保 Geth 实例正在运行并连接到部署了上述合约的 Morden 测试网络。另外，请将下面的 IPC 套接字路径更新为您自己的本地 Geth 节点报告的路径。

	package main
	
	import (
		"fmt"
		"log"
	
		"github.com/ethereum/go-ethereum/common"
		"github.com/ethereum/go-ethereum/ethclient"
	)
	
	func main() {
		// Create an IPC based RPC connection to a remote node // // 创建到远程节点的基于 IPC 的 RPC 连接
		conn, err := ethclient.Dial("/home/karalabe/.ethereum/testnet/geth.ipc")
		if err != nil {
			log.Fatalf("Failed to connect to the Ethereum client: %v", err)
		}
		// Instantiate the contract and display its name // 实例化合约并显示其名称
		token, err := NewToken(common.HexToAddress("0x21e6fc92f93c8a1bb41e2be64b4e1f88a54d3576"), conn)
		if err != nil {
			log.Fatalf("Failed to instantiate a Token contract: %v", err)
		}
		name, err := token.Name(nil)
		if err != nil {
			log.Fatalf("Failed to retrieve token name: %v", err)
		}
		fmt.Println("Token name:", name)
	}
输出

	Token name: Testnet Unicorn
如果您查看调用以读取令牌名称的方法 `token.Name(nil)`，它需要传递一个参数，即使原始 Solidity 合约不需要。这是一种 `*bind.CallOpts` 类型，可用于微调呼叫。

- `Pending`: 是否访问待定合约状态或当前稳定的状态
- `GasLimit`：限制调用可能消耗的计算资源

#### 使用以太坊合约进行交易
调用改变合约状态的方法（即交易）需要更多的参与，因为实时交易需要被授权并广播到网络中。与在我们附加的节点中存储帐户和密钥的传统方式相反，Go 绑定需要在本地签署交易，而不是将其委托给远程节点。这样做是为了促进以太坊社区的总体方向，在该社区中，帐户对 DApp 是私有的，而不是在它们之间共享（默认情况下）。

因此，为了允许使用合约进行交易，您的代码需要实现一个方法，该方法给出输入交易，对其进行签名并返回授权的输出交易。由于大多数用户拥有 [Web3 Secret Storage](https://github.com/ethereum/wiki/wiki/Web3-Secret-Storage-Definition) 格式的密钥，因此该 `bind` 包包含一个小型实用程序方法 ( `bind.NewTransactor(keyjson, passphrase`))，该方法可以根据密钥文件和相关密码创建授权交易者，而无需用户自己实施密钥签名。

更改前面的代码片段以将一个独角兽发送到零地址

	package main
	
	import (
		"fmt"
		"log"
		"math/big"
		"strings"
	
		"github.com/ethereum/go-ethereum/accounts/abi/bind"
		"github.com/ethereum/go-ethereum/common"
		"github.com/ethereum/go-ethereum/ethclient"
	)
	
	const key = `paste the contents of your *testnet* key json here`
	
	func main() {
		// Create an IPC based RPC connection to a remote node and instantiate a contract binding
		conn, err := ethclient.Dial("/home/karalabe/.ethereum/testnet/geth.ipc")
		if err != nil {
			log.Fatalf("Failed to connect to the Ethereum client: %v", err)
		}
		token, err := NewToken(common.HexToAddress("0x21e6fc92f93c8a1bb41e2be64b4e1f88a54d3576"), conn)
		if err != nil {
			log.Fatalf("Failed to instantiate a Token contract: %v", err)
		}
		// Create an authorized transactor and spend 1 unicorn
		auth, err := bind.NewTransactor(strings.NewReader(key), "my awesome super secret password")
		if err != nil {
			log.Fatalf("Failed to create authorized transactor: %v", err)
		}
		tx, err := token.Transfer(auth, common.HexToAddress("0x0000000000000000000000000000000000000000"), big.NewInt(1))
		if err != nil {
			log.Fatalf("Failed to request token transfer: %v", err)
		}
		fmt.Printf("Transfer pending: 0x%x\n", tx.Hash())
	}
输出

	Transfer pending: 0x4f4aaeb29ed48e88dd653a81f0b05d4df64a86c99d4e83b5bfeb0f0006b0e55b
请注意，很可能您不会有任何测试网的以太可供使用，因此上述程序将失败并出现错误。测试网将至少 2.014 个以太`0xDf7D0030bfed998Db43288C190b63470c2d18F50` 发送到基金会测试网 tipjar 以接收独角兽令牌，您将能够看到上面的代码运行而没有错误！

与上一节中仅读取合约状态的方法调用类似，交易方法也需要一个强制性的第一个参数，一个 `*bind.TransactOpts` 类型，它授权交易并可能对其进行微调：

- `From`: 调用方法的账户地址（必填）
- `Signer`：在广播之前在本地签署交易的方法（强制）
- `Nonce`：用于交易排序的帐户随机数（可选）
- `GasLimit`：限制调用可能消耗的计算资源（可选）
- `GasPrice`：显式设置运行交易的燃料价格（可选）
- `Value`：与方法调用一起转移的任何资金（可选）

两个强制字段被自动由设置 `bind` 如果 AUTH 选项使用构造包 `bind.NewTransactor` 。如果未设置 nonce 和 gas 相关字段，则绑定会自动派生它们。未设置的值假定为零。	

#### 预配置的合同会话
如前两节所述，读取和状态修改合约调用都需要一个强制性的第一个参数，该参数既可以授权也可以微调一些内部参数。然而，大多数时候我们希望使用相同的参数并使用相同的帐户发出交易，因此总是构建调用/交易选项或将它们与绑定一起传递可能变得笨拙。

为了避免这些情况，生成器还创建了专门的包装器，可以预先配置调整和授权参数，允许在不需要额外参数的情况下调用所有 Solidity 定义的方法。

这些名称类似于原始合约类型名称，只是后缀为 `Sessions`：

	// Wrap the Token contract instance into a session // 将 Token 合约实例包装成会话
	session := &TokenSession{ 
		Contract: token,
		CallOpts: bind.CallOpts{
			Pending: true,
		},
		TransactOpts: bind.TransactOpts{
			From:     auth.From,
			Signer:   auth.Signer,
			GasLimit: big.NewInt(3141592),
		},
	}
	// Call the previous methods without the option parameters // 不带选项参数调用前面的方法
	session.Name()
	session.Transfer("0x0000000000000000000000000000000000000000"), big.NewInt(1))
#### 将合约部署到以太坊
与现有合约交互很好，但让我们更上一层楼，将全新的合约部署到以太坊区块链上！然而，要做到这一点，我们用来生成绑定的合约 ABI 是不够的。我们也需要编译的字节码来允许部署它。

要获取字节码，请返回到您可以生成它的在线编译器，或者下载我们的 [token.bin](https://gist.github.com/karalabe/026548f6a5f5f97b54de). 您还需要重新运行包含字节码的 Go 生成器来创建部署代码：

	$ abigen --abi token.abi --pkg main --type Token --out token.go --bin token.bin
这将生成类似于 [token.go](https://gist.github.com/karalabe/2153b087c1f80f651fd87dd4c439fac4). 如果你快速浏览一下这个文件，你会发现 `DeployToken` 与之前的代码相比刚刚注入的一个额外的函数。除了 Solidity 指定的所有参数之外，它还需要通常的授权选项来部署合约，以及以太坊后端来部署合约。

将它们放在一起将导致：

	package main
	
	import (
		"fmt"
		"log"
		"math/big"
		"strings"
		"time"
	
		"github.com/ethereum/go-ethereum/accounts/abi/bind"
		"github.com/ethereum/go-ethereum/ethclient"
	)
	
	const key = `paste the contents of your *testnet* key json here`
	
	func main() {
		// Create an IPC based RPC connection to a remote node and an authorized transactor // 创建到远程节点和授权交易者的基于 IPC 的 RPC 连接
		conn, err := rpc.NewIPCClient("/home/karalabe/.ethereum/testnet/geth.ipc")
		if err != nil {
			log.Fatalf("Failed to connect to the Ethereum client: %v", err)
		}
		auth, err := bind.NewTransactor(strings.NewReader(key), "my awesome super secret password")
		if err != nil {
			log.Fatalf("Failed to create authorized transactor: %v", err)
		}
		// Deploy a new awesome contract for the binding demo // 为绑定演示部署一个新的绑定合约
		address, tx, token, err := DeployToken(auth, conn), new(big.Int), "Contracts in Go!!!", 0, "Go!")
		if err != nil {
			log.Fatalf("Failed to deploy new token contract: %v", err)
		}
		fmt.Printf("Contract pending deploy: 0x%x\n", address)
		fmt.Printf("Transaction waiting to be mined: 0x%x\n\n", tx.Hash())
	
		// Don't even wait, check its presence in the local pending state // 甚至不要等待，检查它在本地挂起状态中的存在
		time.Sleep(250 * time.Millisecond) // Allow it to be processed by the local node :P
	
		name, err := token.Name(&bind.CallOpts{Pending: true})
		if err != nil {
			log.Fatalf("Failed to retrieve pending name: %v", err)
		}
		fmt.Println("Pending name:", name)
并且代码按预期执行：它请求在以太坊区块链上创建一个全新的 Token 合约，我们可以等待被挖掘，或者在上面的代码中开始在挂起状态下调用它的方法:)

	Contract pending deploy: 0x46506d900559ad005feb4645dcbb2dbbf65e19cc
	Transaction waiting to be mined: 0x6a81231874edd2461879b7280ddde1a857162a744e3658ca7ec276984802183b
	
	Pending name: Contracts in Go!!!
#### 直接绑定 Solidity
如果您一直遵循本教程直到此时，您可能已经意识到每个合约修改都需要重新编译，生成的 ABI 和字节码（特别是如果您需要多个合约）分别保存到文件中，然后为它们执行绑定。在第 N 次迭代之后，这可能会变得非常麻烦，因此该 `abigen` 命令支持直接从 Solidity 源文件 (`.sol`) 进行绑定，它首先将源代码（通过`--solc`，默认为 `solc`）编译为它的组成组件并使用它进行绑定。

绑定官方代币合约 [token.sol](https://gist.github.com/karalabe/08f4b780e01c8452d989) 将需要运行：

	$ abigen --sol token.sol --pkg main --out token.go
注意：从 Solidity 构建 ( `--sol`) 与单独设置绑定组件 ( `--abi,--bin`) 是相互排斥的 `--type`，因为它们都是从 Solidity 代码中提取的，并直接生成构建结果。

直接从 Solidity 构建合约有一个很好的副作用，即 Solidity 源文件中包含的所有合约都被构建和绑定，因此如果您的文件包含许多合约源，则可以从 Go 代码中获得每一个。示例令牌可靠性文件生成 [token.go](https://gist.github.com/karalabe/c22aab73194ba7da834ab5b379621031).
#### 项目整合（即 go generate）
该 `abigen` 命令的创建方式可以与现有的 Go 工具链完美结合：无需记住将以太坊合约绑定到 Go 项目所需的确切命令，我们可以利用它 `go generate` 来记住所有细节。 

将绑定生成命令放在包定义之前的 Go 源文件中：

	//go:generate 
	abigen --sol token.sol --pkg main --out token.go
之后，无论何时修改 Solidity 合约，无需记住并运行上述命令，我们只需调用 `go generate` 包（甚至通过调用整个源代码树 `go generate ./..` .），它就会为我们正确生成新的绑定。
#### 区块链模拟器
能够从本机 Go 代码中部署和访问已部署的以太坊合约是一项非常强大的功能，但开发本机代码有一个方面甚至测试网都不适合：自动单元测试。使用 go-ethereum 内部构造可以创建测试链并对其进行验证，但是使用这种低级机制进行高级合约测试是不可行的。

为了解决使运行（和测试）原生 DApp 变得困难的最后一个问题，我们还实现了一个模拟区块链，可以像实时 RPC 后端一样将其设置为原生合约的后端：`backends.NewSimulatedBackend(genesisAccounts).`

	package main
	
	import (
		"fmt"
		"log"
		"math/big"
	
		"github.com/ethereum/go-ethereum/accounts/abi/bind"
		"github.com/ethereum/go-ethereum/accounts/abi/bind/backends"
		"github.com/ethereum/go-ethereum/core"
		"github.com/ethereum/go-ethereum/crypto"
	)
	
	func main() {
		// Generate a new random account and a funded simulator
		key, _ := crypto.GenerateKey()
		auth := bind.NewKeyedTransactor(key)
	
		sim := backends.NewSimulatedBackend(core.GenesisAccount{Address: auth.From, Balance: big.NewInt(10000000000)})
	
		// Deploy a token contract on the simulated blockchain
		_, _, token, err := DeployMyToken(auth, sim, new(big.Int), "Simulated blockchain tokens", 0, "SBT")
		if err != nil {
			log.Fatalf("Failed to deploy new token contract: %v", err)
		}
		// Print the current (non existent) and pending name of the contract
		name, _ := token.Name(nil)
		fmt.Println("Pre-mining name:", name)
	
		name, _ = token.Name(&bind.CallOpts{Pending: true})
		fmt.Println("Pre-mining pending name:", name)
	
		// Commit all pending transactions in the simulator and print the names again
		sim.Commit()
	
		name, _ = token.Name(nil)
		fmt.Println("Post-mining name:", name)
	
		name, _ = token.Name(&bind.CallOpts{Pending: true})
		fmt.Println("Post-mining pending name:", name)
	}
输出

	Pre-mining name: 
	Pre-mining pending name: Simulated blockchain tokens
	Post-mining name: Simulated blockchain tokens
	Post-mining pending name: Simulated blockchain tokens
请注意，我们不必等待本地私有链矿工或测试网矿工来整合当前待处理的交易。当我们决定挖掘下一个区块时，我们只是 Commit() 使用模拟器
	
### [移动应用程序接口](https://geth.ethereum.org/docs/dapp/mobile)
- 库捆绑	

	go-ethereum 移动库分布无论是作为一个 Android.aar 归档（含有二进制文件arm-7，arm64，x86和x64）; 或作为 iOS XCode 框架（包含 arm-7,arm64和 的二进制文件 x86）。我们目前不提供适用于 Windows 手机的库包。
- 安卓

	go-ethereum 在 Android 项目中使用的最简单方法是通过 Maven 依赖项。我们通过 Maven Central 提供所有稳定版本（从 v1.5.0 开始）的包，还通过 Sonatype OSS 存储库提供最新的开发包。
	
	...
- ios

	go-ethereum在 iOS 项目中使用的最简单方法是通过 CocoaPods依赖项。我们提供所有稳定版本（从 v1.5.3 开始）和最新开发版本的捆绑包。
	
	...
	
### 移动账户管理
要为您的移动应用程序提供以太坊集成，您应该感兴趣的第一件事是帐户管理。

尽管当前所有领先的以太坊实现都提供内置的帐户管理，但不建议将帐户保存在多个应用程序和/或多人共享的任何位置。与您不将登录凭据委托给您的 ISP（毕竟是您进入互联网的网关）的方式相同；您也不应该将您的凭据委托给以太坊节点（它是您进入以太坊网络的网关）。

在您的移动应用程序中处理用户帐户的正确方法是进行客户端帐户管理，所有内容都包含在您自己的应用程序中。通过这种方式，您可以根据需要确保对敏感数据的细粒度（或粗略）访问权限，而无需依赖任何第三方应用程序的功能和/或漏洞。

为了支持这一点，go-ethereum 提供了一个简单而全面的帐户库，它为您提供了所有工具，通过加密的密钥库和密码保护帐户进行适当的安全帐户管理。您可以利用go-ethereum 加密实现的所有安全性，同时在您自己的应用程序中运行所有内容。
#### 加密密钥库
尽管在用户自己的移动设备上本地处理用户帐户确实提供了某些安全保证，但以太坊帐户的访问密钥不应以明文形式放置。因此，我们提供了一个加密的密钥库，它为您提供适当的安全保证，而无需您对相关加密原语的部分有透彻的了解。

使用加密密钥库时要知道的重要一点是，其中使用的加密原语可以在标准模式或轻型模式下运行。前者以增加计算负担和资源消耗为代价提供更高级别的安全性：

- 标准需要 256MB 内存和现代 CPU 上的 1 秒处理才能访问密钥
- light需要 4MB 内存和现代 CPU 上的 100 毫秒处理才能访问密钥

因此，光更适合移动应用程序，但您仍应注意权衡。

对于那些对加密和/或实现细节感兴趣的人，密钥库使用高效加密标准中secp256k1定义的椭圆曲线，由库实现并由 . 帐户以Web3 Secret Storage格式存储在磁盘上。libsecp256kgithub.com/ethereum/go-ethereum/accounts
#### Android (Java) 上的密钥库
Android 上的加密密钥库由包中的 `KeyStore` 类实现 `org.ethereum.geth` 。配置常量（用于上述标准或轻量级 安全模式）位于 Geth 抽象类中，与 `org.ethereum.geth` 包类似。因此，要在 Android 上进行客户端帐户管理，您需要将两个类导入到您的 Java 代码中：

	import org.ethereum.geth.Geth;
	import org.ethereum.geth.KeyStore;
之后可以通过以下方式创建新的加密密钥库：

	KeyStore ks = new KeyStore("/path/to/keystore", Geth.LightScryptN, Geth.LightScryptP);
密钥库文件夹的路径必须是本地移动应用程序可写入但其他已安装应用程序无法读取的位置（显然出于安全原因），因此我们建议将其放在应用程序的数据目录中。如果您 `KeyStore` 在扩展 Android 对象的类中创建，您很可能可以 `Context.getFilesDir()` 通过访问该方法` this.getFilesDir()` ，因此您可以将密钥库路径设置为 `this.getFilesDir() + "/keystore`"。

KeyStore 构造函数的最后两个参数是定义密钥库加密的资源密集程度的加密参数。您可以在 之间进行选择 `Geth.StandardScryptN, Geth.StandardScryptP，Geth.LightScryptN, Geth.LightScryptP` 或指定您自己的数字（请确保您了解相关的底层密码学）。我们建议使用轻型版本。
#### iOS 上的密钥库 (Swift 3)
iOS 上的加密密钥库由框架中的 `GethKeyStore` 类实现 `Geth`。配置常量（用于上述标准或轻量级安全模式）与全局变量位于相同的命名空间中。因此，要在 iOS 上进行客户端帐户管理，您需要将框架导入到 Swift 代码中：

	import Geth
之后可以通过以下方式创建新的加密帐户管理器：

	let ks = GethNewKeyStore("/path/to/keystore", GethLightScryptN, GethLightScryptP);
密钥库文件夹的路径必须是本地移动应用程序可写但其他已安装应用程序不可读取的位置（显然出于安全原因），因此我们建议将其放在应用程序的文档目录中。您应该能够通过 检索文档目录 `let datadir = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true)[0]，`因此您可以将密钥库路径设置为 `datadir + "/keystore".`

GethNewKeyStore 工厂方法的最后两个参数是定义密钥库加密的资源密集程度的加密参数。您可以在 之间进行选择 `GethStandardScryptN, GethStandardScryptP，GethLightScryptN, GethLightScryptP` 或指定您自己的数字（请确保您了解相关的底层密码学）。我们建议使用轻型版本。
#### 帐户生命周期
为您的以太坊账户创建加密密钥库后，您可以将其用于移动应用程序的整个账户生命周期要求。这包括创建新帐户和删除现有帐户的基本功能；以及更新访问凭证、导出现有帐户和将它们导入到另一台设备的更高级功能。

尽管密钥库定义了用于存储您的帐户的加密强度，但没有可以授予对所有帐户的访问权限的全局主密码。相反，每个帐户都是单独维护的，并以加密格式 单独存储在磁盘上，确保更清晰、更严格地分离凭据。

然而，这种个性意味着任何需要访问帐户的操作都需要以密码短语的形式为该特定帐户提供必要的身份验证凭据：

- 创建新帐户时，调用者必须提供密码来加密帐户。任何后续访问都需要此密码，缺少密码将永远无法使用新创建的帐户。
- 删除现有帐户时，调用者必须提供密码来验证帐户的所有权。这不是加密必要的，而是防止帐户意外丢失的保护措施。
- 更新现有帐户时，调用者必须提供当前密码和新密码。完成操作后，将无法再通过旧密码访问该帐户。
- 导出现有帐户时，调用者必须提供当前密码来解密帐户，以及在将密钥文件返回给用户之前重新加密的导出密码。这是允许在不共享原始凭证的情况下在设备之间移动帐户所必需的。
- 导入新帐户时，调用者必须提供要导入的密钥文件的加密密码，以及用于存储帐户的新密码。这是允许使用与用于移动它们的凭据不同的凭据存储帐户所必需的。

请注意，没有丢失密码短语的恢复机制。加密密钥库的加密属性（如果使用提供的参数）保证帐户凭据不会在任何有意义的时间内被强制执行。

#### Android 帐户 (Java)
...
#### iOS 帐户 (Swift 3)
...
#### 签署授权
如上所述，帐户对象不持有相关以太坊帐户的敏感私钥，而只是用于标识加密密钥的占位符。所有需要授权的操作（例如交易签名）都由账户管理器在授予其访问私钥的权限后执行。

有几种不同的方式可以授权客户经理执行签名操作，每种方式都有其优点和缺点。由于不同的方法具有截然不同的安全保证，因此必须清楚每种方法的工作原理：

- 单一授权

	通过密钥库签署交易的最简单方法是在每次需要签名时提供帐户的密码，这将临时解密私钥，执行签名操作并立即丢弃解密的密钥。缺点是每次都需要从用户那里查询密码，如果经常这样做会很烦人；或者应用程序需要将密码保存在内存中，如果操作不当，可能会产生安全后果；并且根据密钥库的配置强度，不断解密密钥可能会导致不可忽视的资源需求。
- 多重授权

	通过密钥库对交易进行签名的一种更复杂的方法是通过其密码解锁一次账户，并允许账户管理器缓存解密的私钥，从而无需密码即可完成所有后续签名请求。缓存私钥的生命周期可以手动管理（通过明确锁定帐户备份）或自动管理（通过在解锁期间提供超时）。这种机制对于用户可能需要签署许多交易或应用程序需要在不需要用户输入的情况下签署的场景很有用。要记住的关键方面是，任何有权访问帐户管理器的人都可以在特定帐户解锁时签署交易（例如设备无人看管；应用程序运行不受信任的代码）。

请注意，创建交易超出了此处的范围，因此本节的其余部分将假设我们已经有一个交易要签名，并且只关注创建它的授权版本。创建一个真正有意义的交易将在后面介绍。

#### 在 Android (Java) 上签名
...
#### 在 iOS 上签名 (Swift 3)
...
	

	
	