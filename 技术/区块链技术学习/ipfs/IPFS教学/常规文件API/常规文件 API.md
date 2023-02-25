# 常规文件 API
## 介绍文件 API 1
- IPFS：星际文件系统

	IPFS 是一种点对点 (P2P) 网络协议，用于在分布式 Web 上共享数据。您可以将其视为具有一些独特特性的文件系统，这些特性使其成为安全、分散共享的理想选择。
	
	如果您还没有这样做，我们鼓励您查看我们的去中心化网络内容寻址教程，以了解有关去中心化网络的所有信息以及它与您习惯的网络的比较。在那里，您将了解有关内容寻址、加密散列、内容标识符 (CID) 和与同行共享的所有知识，您需要了解所有这些才能充分利用本教程。

### 文件 API 与 DAG API
您可以使用 IPFS 存储多种类型的数据。如果您已经探索了我们的 [P2P 数据链接](https://proto.school/basics)与[内容寻址或在去中心化 Web 上写博客](https://proto.school/blog)教程，您已经了解了如何 使用 DAG API 在网络上存储基元、对象和数组。

[DAG API 允许您在 IPFS 中使用 IPLD](https://github.com/ipld/ipld) （星际关联数据）提供的独特且通用的原始数据结构。您可以在 js-ipfs（IPFS 的 JavaScript 实现）中识别它的方法，因为它们采用以下格式：

	ipfs.dag.someMethod()
DAG API 是向 IPFS 节点添加数据的最通用和最灵活的方法，但在共享文件这一非常常见的用例中，它并不是最有效的。如果你想分享一张小猫的照片或者一个更大的文件，比如你最喜欢的名人狗的搞笑视频，该怎么办？您将如何将这些文件添加到网络并为您的朋友提供一种查看它们的方式？每个文件应该如何放置在有向无环图 (DAG) 中、单个块中或拆分成块？这些是超出 DAG API 范围的优化细节。尽管可以使用 DAG API 将文件添加到 IPFS 节点，但这将是一项劳动密集型任务。

另一方面，文件 API 是为更具体的用例定制的。文件 API 准备要放置在网络中的文件，并确保 IPFS 知道如何访问它们并有效地处理它们。文件 API 由两部分组成：

- 我们将在本教程中介绍的常规文件 API 
- 和可变文件系统 (MFS)。

### 常规文件 API 与 MFS 文件 API
如果您阅读了我们的可变文件系统教程，您可能会想，“我已经学会了如何在 IPFS 上处理文件。这会有什么不同？”

可变文件系统 (MFS) 提供了一个 API，旨在复制熟悉的文件系统操作，例如 `mkdir、ls、cp` 等，模仿您在计算机上组织文件和目录的方式。然而，IPFS 中内容的寻址方式使其成为一个不可变的文件系统。文件或目录的地址取决于其内容，因此对文件或目录的任何更改都将导致一个全新的地址。MFS Files API 在一个看起来很熟悉的文件系统上工作，该文件系统具有常规路径——比如 `/some/stuf` ——在本地 IPFS 节点中，这隐藏了不可变内容寻址的复杂性。

尽管 MFS 非常有用，但它提供的抽象隐藏了 IPFS 的一些内部工作原理。我们将在此处讨论的常规文件 API 是一种在 IPFS 中管理文件的基本方法。它将 MFS 的强大抽象换成一种方案，该方案可帮助您了解文件系统中实际发生的情况。在常规文件 API 中，您会找到 `add、ls、cat `和`get`等方法。

与可以在 js-ipfs 中通过格式识别的  `ipfs.files.someMethod()` MFS API 不同，常规文件 API 使用采用格式的顶级方法 `ipfs.someMethod()`。

目前，这两个版本的文件 API 之间的区别有点令人困惑，但 IPFS 团队目前正在努力改进它们，以使整个文件 API 更加用户友好。当事情发生变化时，我们一定会更新我们的教程！

## 处理文件 2
出于安全考虑，Web 浏览器不允许我们直接更改计算机文件系统中的文件。因此，您需要将一个或多个文件上传到您可以在整个教程中使用的浏览器。

在每个挑战中，您会看到您可以通过拖放或从文件资源管理器中选择文件来从计算机上传文件。如果您仔细查看代码编辑器中的  `run`  函数，您会注意到它现在有一个 `files` 参数。当您从计算机上传文件时，我们将确保它们作为 `files` 数组传递到函数中。只要您不刷新浏览器，这些文件在本教程的下一课中仍然可以访问，但您还可以选择上传不同的文件以供每节课使用。

该数组中的每个元素 `files` 都是一个对象，由浏览器的 [文件 API](https://developer.mozilla.org/en-US/docs/Web/API/File)（不要与 IPFS 文件 API 混淆）File定义。除了上传文件的内容外，对象还包含以下属性：

- `File`
	- `name `（上传文件的名称）
	- `lastModified`（上次修改上传文件的日期）
	- `size`（上传文件的大小）
	- `type `（上传文件的类型）

为了练习，让我们从您的计算机上传一个或多个文件，并查看浏览器作为数组接收到的 `files` 内容。

- [demo](https://proto.school/regular-files-api/02)
	- 首先，通过拖拽或点击从文件资源管理器中进行选择来上传一个或多个文件。接下来，在代码编辑器中，删除 `run` 函数中返回文件前面的注释标记(//)。

			const run = async (files) => {
			   return files
			}
			return run
	- 查看结果

		查看下面的数据以查看 `files` 数组中现在可访问的数据。请注意，这些文件目前仅在浏览器中。在下一课中，我们将看到如何将它们添加到 IPFS。
		
			[
			  {
			    "name": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
			    "lastModified": 1394259002000,
			    "lastModifiedDate": "2014-03-08T06:10:02.000Z",
			    "webkitRelativePath": "",
			    "size": 6368596,
			    "type": "image/png"
			  }
			]

## 添加文件 3
### 在 ProtoSchool 中处理文件
在我们的 ProtoSchool 教程中，每次您在课程中点击“提交”按钮时，我们都会在浏览器中为您创建一个新的 IPFS 节点。每当您在我们的课程中看到 `ipfs.someMethod()` 时 ，ipfs 都是一个引用您的 IPFS 实例的变量，也称为节点。这个 IPFS 节点不会从一节课转移到另一节课，这就是为什么你会看到我们经常在挑战中预先填充一些代码，以使你的新 IPFS 节点的状态与你节点的最终状态相同之前的挑战。

我们在幕后创建这些 IPFS 节点，以便您可以专注于每节课的内容。但是，在 ProtoSchool 之外，您需要更一致的体验和可以重复访问的节点（或多个节点）。为此，您可以自己在浏览器中[初始化 js-ipfs](https://github.com/ipfs/js-ipfs/blob/master/docs/BROWSERS.md) 或者通过安装 IPFS 并在终端中运行守护进程来在本地托管您自己的节点。当您准备好进行试验时，您可以访问 IPFS 文档站点以了解如何 [安装 IPFS](https://docs.ipfs.io/install/) 和 [初始化您的节点](https://docs.ipfs.io/how-to/command-line-quick-start/#initialize-the-repository)。

如前所述，本教程中讨论的方法是[ IPFS 文件 API](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md)的一部分。查看文档以获取更具体的详细信息，例如每个 API 方法的选项。
### 添加单个文件 add
为了使文件在 IPFS 网络上可用，您首先需要将其添加到特定的 IPFS 节点。重要的是要记住，因为 IPFS 是一个点对点的、去中心化的系统，添加一个文件并不意味着将它上传到某个地方的远程服务器。假设您在自己的机器上使用一个节点，这个过程更像是从您的计算机中选择一个文件并为其添加一个标签，上面写着“我在 IPFS 上共享！我的名字是 __ 。来找我吧！ ” 该标签包括从文件内容派生的内容标识符 (CID)，作为一种地址，其他对等方可以使用该地址来查找特定文件，而不管它托管在谁的计算机上。

当您将文件添加到 IPFS 时，您将把它放在您自己的节点中，并使其在您的节点运行时可供网络上的对等方访问。只要拥有它的人（比如你！）连接到网络，它就会一直可用。如果还没有其他人找到并共享您的文件，并且您关闭计算机或停止运行 IPFS 守护程序，则该内容将不再可供任何人发现。通过称为固定的过程共享您的内容的人越多，它在任何时候都可用的可能性就越大。

让我们看一下如何将文件添加到您的 IPFS 节点。我们将通过执行以下add方法来做到这一点：

	ipfs.add(data, [options])
因此，如果我们 `File` 在浏览器中有一个可爱的小猫照片对象，我们可以通过  `catPic` 变量访问它，并且我们想将它添加到我们的 IPFS 节点，我们可以像 `data` 这样将它传递到 `add` 方法中：

	const result = await ipfs.add(catPic)
变量的值 `result` 是以下格式的对象：

	{
	    path: String,
	    cid: CID,
	    size: Number,
	    mode: Number
	}
### 添加多个文件 addAll 
如果您有不止一张可爱的动物照片要添加到节点，请改用该 `addAll`方法：

	ipfs.addAll([catPic, dogPic, giraffePic])
因为该 `addAll` 方法返回一个 [Async Iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of)，所以您只能一个接一个地迭代它的值。如果你需要返回所有的值，你可以将每个值保存到一个数组中，然后返回该数组。

要迭代 `Async Iterable` 的所有值，我们可以使用 `for await...of` 循环：

	const result = []
	
	for await (const resultPart of ipfs.addAll([catPic, dogPic, giraffePic])) {
	    result.push(resultPart)
	}
	
	return result
为了使事情更容易，我们可以使用 [it-all](https://www.npmjs.com/package/it-all) 自动执行此操作的包：

	// all 函数来自 it-all 包，在我们的代码挑战中是全局可用的(就像ipfs一样)
	
	const result = await all(ipfs.addAll([catPic, dogPic, giraffePic]))
变量的值 result 是一个对象数组（每个对象的格式都与 `add ` 的输出相同），每个对象对应一个添加到 IPFS 的文件，格式如下：

	{
	    path: String,
	    cid: CID,
	    size: Number,
	    mode: Number
	}

`cid` 是一个 CID 对象（Content Identifier）的值，一个从每个文件的内容生成的唯一地址。（要更深入地了解 CID 的生成方式及其重要性，请查看我们的 [去中心化 Web 内容寻址教程](https://proto.school/content-addressing)。）在以后的课程中，我们将学习如何使用此值检索内容一份文件。

`add` 和 `addAll` 方法接受 File 对象之外的其他数据格式，并提供了许多高级选项，用于设置块大小和哈希算法，在添加文件时固定文件等等。在本教程中，我们将重点介绍基础知识，但您可以查看 `add` 或 `addAll` 方法的完整文档以了解更多信息。

- [demo](https://proto.school/regular-files-api/03)

	使用该方法将浏览器中的文件（在 `files`下面的数组中可用）添加 `addAll` 到您的 IPFS 节点。或者您可以使用该 `add` 方法上传单个文件。返回结果。

	提示：

	- 您可以将对象数组传递 `File` 给 `addAll` 方法，也可以将单个 `File` 对象传递给 `add` 方法。
	- 使用 `addAll` 方法时，您需要将所有结果连接到一个数组中，因为该方法返回一个 `Async Iterable`. 您可以使用循环 `for await...of` 或函数 `all`来执行此操作。
	- 通过该方法上传单个文件时 `add`，请确保只将文件传递给该 `add` 方法，而不是传递整个 `files` 数组。

	代码
	
	- 模式1
		
			/* global ipfs, all */
			
			const run = async (files) => {
			  const result = await all(ipfs.addAll(files))
			
			  return result
			}
			return run
	- 模式2

			const run = async (files) => {
			
			  // or using for await...of loop
			  const result = []
			 
			  for await (const resultPart of ipfs.addAll(files)) {
			   result.push(resultPart)
			  }
			
			  // or when uploading one file
			  // const result = await ipfs.add(files[0])
			
			  return result
			}
			return run
	- 模式3

			const run = async (files) => {
			
			  const result = await ipfs.add(files[0])
			
			  return result
			}
			return run
	检查结果

		您返回了下面的对象。输出很长，因为 CID 在 Object 内部表示为 ，但如果您向下滚动，我们将为您提供更精简的视图
	
			{
			  "path": "QmRajrhBHhkenmQYjxuMD1ysWLxCShXRrNJzUqhBH3jtxV",
			  "cid": {
			    "codec": "dag-pb",
			    "version": 0,
			    "hash": {
			      "0": 18,
			      "1": 32,
			      "2": 48,
			      "3": 47,
			      "4": 3,
			      "5": 85,
			      "6": 148,
			      "7": 117,
			      "8": 122,
			      "9": 126,
			      "10": 224,
			      "11": 32,
			      "12": 150,
			      "13": 114,
			      "14": 35,
			      "15": 138,
			      "16": 132,
			      "17": 15,
			      "18": 117,
			      "19": 46,
			      "20": 53,
			      "21": 48,
			      "22": 16,
			      "23": 178,
			      "24": 67,
			      "25": 74,
			      "26": 39,
			      "27": 136,
			      "28": 15,
			      "29": 147,
			      "30": 65,
			      "31": 87,
			      "32": 181,
			      "33": 238
			    }
			  },
			  "size": 6370155,
			  "mode": 420
			}
		为了简化输出，我们可以使用 `toString()`  属性上的 `cid` 方法 以字符串格式获取 CID：
			
			QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn。
		在以后的课程中，我们将始终展示此简化版本以使其更易于阅读，如下所示。

		您返回了下面的对象。请特别注意该 `cid` 值，因为稍后我们将需要它来再次访问该文件。匹配此 `path` 文件 `cid`  ，但我们将在以后的课程中看到这并不总是正确的
		
			{
			  "path": "QmRajrhBHhkenmQYjxuMD1ysWLxCShXRrNJzUqhBH3jtxV",
			  "cid": "CID('QmRajrhBHhkenmQYjxuMD1ysWLxCShXRrNJzUqhBH3jtxV')",
			  "size": 6370155,
			  "mode": 420
			}

## 读取文件内容 4
在上一课中，您看到添加到 IPFS 的每个文件都有其 `cid` 从其内容派生的唯一性。这个 [CID (Content Identifier)](https://proto.school/data-structures/04)，可以像访问文件的地址一样使用。如果您知道文件的 CID，则可以使用 `cat` 常规 IPFS 文件 API 提供的方法（类似于您之前在 Unix 风格的系统中看到的方法）来检索其内容，如下所示：

	ipfs.cat(ipfsPath, [options])
IPFS `path` 可以采用多种形式（您可以在 [Files API](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfscatipfspath-options) 文档中阅读它们）。在本课中，我们将使用最简单的一个：

	调用方法 cid 时我们收到的文件对象中的 present 。
您可能还记得上一课中 `ipfs.add` ，该 `add` 方法返回了您添加到 IPFS 的每个文件的 `path` 和 `cid`值。稍后我们将详细了解这两个值之间的差异。

该 `cat` 方法首先在您自己的节点中搜索请求的文件，如果在那里找不到它，它将尝试在更广泛的 IPFS 网络上找到它。由于加密散列的工作方式，您正在搜索的内容保证与找到的文件相匹配，无论哪个节点托管它。

该 `cat` 方法返回一个 `Async Iterable`，它迭代文件的数据块，即缓冲区。

一个 `Buffer` 只是字节的原始集合，因此不会对编码或它包含的数据类型做出任何假设。但是，如果我们知道正在检索的文件是纯文本文件，例如 `a .txt`，我们可以通过调用 JavaScript 方法 `.toString()` 将其缓冲内容转换为 UTF-8 字符串（对这些原始字节的解释）。

但是由于我们有同一个文件的多个缓冲区，我们需要在将它们转换为字符串之前将它们重新组合（连接）到一个缓冲区中。该 `it-to-buffer` 包可以做到这一点：遍历文件的所有块并为我们将它们放回一起。

因此，如果您在 IPFS 节点中拥有文本文件的 CID，则可以将文件的内容检索为可读字符串，如下所示：

	// toBuffer变量全局可用 (就像ipfs一样)
	
	const bufferedContents = await toBuffer(ipfs.cat('QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ')) // returns a Buffer
	const stringContents = bufferedContents.toString() // 返回一个字符串
当您准备好在现实世界中尝试此操作时，您应该注意到上面的示例可能会导致大量内存使用，具体取决于正在读取的文件的内容。如果您正在处理大文件并发现是这种情况，您可能希望跳过使用该 `it-to-buffer` 包，而是迭代地处理每个数据块。IPFS 现在回归的主要原因`Async Iterables` 是提供一个内置选项来处理潜在的性能问题。在  ProtoSchool 教程中，我们的代码挑战使用小文件，因此我们可以连接所有内容而无需担心性能。

- [Demo](https://proto.school/regular-files-api/04)

	对于这个挑战，我们已经为您添加了一个纯文本文件到您的 IPFS 节点，它的 CID 是以下字符串：

		QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ
	使用该 `cat` 函数，检索文件的内容并将它们作为字符串返回。

	提示：

	我们提供的  `CID`  是一个字符串，所以需要放在引号内。`.toString()`另外，不要忘记在返回结果之前将文本文件的缓冲内容转换为字符串。

	您需要连接所有数据块，因为该 `ipfs.cat` 方法返回一个 `Async Iterable.` 您可以使用该包 `it-to-buffer` 来实现此目的。
	
		/* global ipfs, toBuffer */
	
		const run = async () => {
		    const fileContents = await toBuffer(ipfs.cat('QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ'))
		    const message = fileContents.toString()
		
		    return message
		}
		return run
	返回结果
	
		You did it!

## 在目录中添加文件 5
当我们在第 3 课中将文件添加到 IPFS 时，我们看到 `addAll` 和 `add` 方法为每个文件返回`path、cid`和 `size`值，并且 `path`和 `cid ` 字符串值相同。要稍后访问这些文件中的每一个，我们需要知道它的 CID。您是否注意到没有提供文件名？

`add`和 `addAll` 方法确实提供了一种稍微更人性化的文件引用方式，但前提是我们使用特殊 `wrapWithDirectory` 选项为添加的文件提供路径和文件名。通过这样做，我们可以为单个文件创建 CID，如我们之前所见，还可以为它们的包含目录创建 CID，它可以用作路径的一部分，以便稍后通过文件名检索这些文件。这个选项将允许我们用 `ls`和 `get`方法做一些有趣的事情，我们将在接下来的课程中介绍。

要使用该 `wrapWithDirectory` 选项，您需要按如下方式调用 `add` 或 `addAll` 方法：

	ipfs.add(file, { wrapWithDirectory: true }) // 添加一个文件
	ipfs.addAll([file1, file2], { wrapWithDirectory: true }) // 添加多个文件在数组
当我们以前使用该 `add` 方法时，我们只能从浏览器传入一个  `File` 对象。但是，要用目录包装文件，我们还需要为每个文件提供一个路径，其中包含我们想要的名称。为此，我们必须用一个对象替换方法 `file` 中的参数 `add` ，其结构如下：

	{
	    path: 'file.txt', // 指定路径的字符串，包括文件名
	    content: file // 在本例中，是 File 对象
	}
要一次添加多个文件，我们只需要将一组对象（如上所示）传递给该 `addAll` 方法。例如，我们可以将两张猫图片添加到 IPFS 节点上的目录中，如下所示：

	ipfs.addAll([
	    {
	        path: 'adorable-kitty.jpg',
	        content: catPic1
	    },
	    {
	        path: 'cat-drinking-milk.jpg',
	        content: catPic2
	    }
	], { wrapWithDirectory: true })
`add` 或 `addAll` 方法将返回关于我们添加的每个文件的信息，以及为了添加具有正确路径的文件而必须创建的任何目录的详细信息。例如，在上面的例子中，`addAll` 方法将返回一个包含三个元素的数组，一个用于添加的每个文件，另一个用于创建的包装目录。

请注意，我们也可以使用相同的 `addAll` 方法通过将它们添加到每个文件的路径来创建一个或多个子目录。例如：

	ipfs.addAll([
	    {
	        path: 'cats/adorable-kitty.jpg',
	        content: catPic1
	    },
	    {
	        path: 'cats/cat-drinking-milk.jpg',
	        content: catPic2
	    },
	    {
	        path: 'dogs/dog-on-a-table.jpg',
	        content: dogPic1
	    }
	], { wrapWithDirectory: true })
在这种情况下，我们最终会在结果数组中得到六个对象，分别用于添加的三个文件

- 一个用于新包装目录
- 一个用于新目录
- 一个用于新 `cats` 目录。
- 一个用于新  `dogs` 目录。

### 目录是不可变的
重要的是要注意，我们以这种方式创建的目录的行为与常规文件系统不同。如果我们使用 `add` 或者 `addAll` 将一些文件包装到一个目录中，我们就不能简单地将 `add` 新文件添加到该目录中。我们刚刚创建的目录的内容是最终的和不可变的。

这是因为 IPFS 使用内容寻址的方式：不同的内容会导致不同的密码哈希（内容标识符），无论我们是在处理目录还是文件本身。

但是，如果您忘记将文件添加到刚创建的目录中怎么办？您必须再次调用 `addAll` 该方法，并使用该目录包含所有您想要的文件，从而生成一个带有新 CID 的新目录。

但是，重要的是要注意，如果您这样做，文件实际上不会被存储多次。不像在你的电脑上，你可能会不小心下载同一个文件两次，将它存储在两个不同的目录中，并使你的存储需求加倍，IPFS 网络本质上根据文件的 CID 知道文件的内容，因此只能保留一个CID 引用的数据的副本。它只是对该数据（CID，您可以将其视为链接或地址）的引用，它存储在您创建的每个目录中。这种效率是 IPFS 的主要优势之一。

	与其将使用 `{ wrapWithDirectory: true }` 创建的目录视为传统文件夹，不如将它们视为命名快捷方式更有用，我们将在接下来的课程中看到这一点。
### 替代
如果您正在寻找一种更好地模仿传统文件系统的体验，您应该探索 [可变文件系统（MFS）](https://proto.school/mutable-file-system)，这是 IPFS 中内置的一个工具，可以让您像通常在基于名称的文件系统中一样处理文件——您可以添加、删除、移动和编辑 MFS 文件，并为您处理更新链接和哈希的所有工作。它是一种抽象，让您可以像处理可变数据一样处理不可变数据。

- [demo](https://proto.school/regular-files-api/05)

	使用 `addAll ` 将多个文件添加到您的 IPFS 节点，用于 `{ wrapWithDirectory: true }` 将它们放在一个目录中。因为你的目标是顶级目录，而不是子目录，所以 `path` 每个文件的名字应该只是它的名字。

	提示：

	- 请务必参考上面的示例，了解指示每个文件所需路径所需的对象结构，以及如何将多个文件作为数组传入。
	- 尝试 [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 或 [forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) 数组方法循环遍历 `files` 数组中的每个文件并访问其名称为 `file.name`.
	- 您需要将所有结果连接到一个数组中，因为该 `ipfs.addAll` 方法返回一个 `Async Iterable`. 您可以使用循环 `for await...of` 或 `all`函数。

	代码
	
	- 模式1

			/* global ipfs, all */
			const run = async (files) => {
			
			  // 使用 Array.map():
			
			  let fileObjectsArray = files.map((file) => {
			    return {
			      path: file.name,
			      content: file
			    }
			  })
			  const results = await all(ipfs.addAll(fileObjectsArray, { wrapWithDirectory: true }))
			
			  return results
			
			}
			return run
	- 模式2

			/* global ipfs, all */
			const run = async (files) => {
			
			  // Using Array.map():
			
			  let fileObjectsArray = files.map((file) => {
			    return {
			      path: file.name,
			      content: file
			    }
			  })
			
			  // Alternatively, using Array.forEach():
			  //
			  // let fileObjectsArray = []
			  //
			  // files.forEach((file) => {
			  //   let fileObject = {
			  //     path: file.name,
			  //     content: file
			  //   }
			  //
			  //   fileObjectsArray.push(fileObject)
			  // })
			
			  const results = await all(ipfs.addAll(fileObjectsArray, { wrapWithDirectory: true }))
			
			  return results
			
			}
			return run
	- 检查结果

		以下是 `addAl`l 该方法返回的结果。请注意，每个文件都有一个对象，每个由选项创建的目录都有一个对象`{ wrapWithDirectory: true }`（在本例中，只是带有 path 的顶级目录''）。

		因为您使用了 `{ wrapWithDirectory: true }` 选项，所以现在每个文件的 `path` 都是您提供的文件名，而不是匹配文件的 cid. 在以后的课程中，您将能够结合使用这些人类可读的路径和目录的 CID 来访问文件的内容。

		`addAll` 当方法返回它们时，我们只能访问添加的文件和目录的 CID 。当您将来使用此方法时，您可能希望将它们保存起来以备后用。


			[
			  {
			    "path": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
			    "cid": "CID('QmRajrhBHhkenmQYjxuMD1ysWLxCShXRrNJzUqhBH3jtxV')",
			    "size": 6370155,
			    "mode": 420
			  },
			  {
			    "path": "",
			    "cid": "CID('Qmey5eWQ1L9bVeQxvbb7VH6UdDh62fbMjrpeCAVaEuWQ2G')",
			    "size": 6370256,
			    "mode": 493
			  }
			]

## 列出目录中的文件 6 
现在我们已经将一些文件包装在 IPFS 节点的目录中，让我们学习如何检查其内容。如果您经常使用命令行，那么您就会熟悉该 ls 命令。IPFS 提供了一种类似的 ls 方法来列出目录的内容。但是，与 `cat` 我们之前看到的方法一样，`ls` 实际上会先在您自己的节点上查找请求的目录，然后在需要时搜索更广泛的网络。由于加密散列的工作方式，具有相同 CID 的两个目录保证具有相同的内容，而不管它们由哪个对等点托管。

`ls` 您可以像这样调用该方法：

	ipfs.ls(ipfsPath)
参数 `ipfsPath` 可以采用多种格式，其中最简单的是 CID。cid（请记住，这与我们使用 `add` 或 `addAll` 方法时看到的返回给我们的字符串值相同。）例如：

	ipfs.ls("Qmeybqr2GaiUyGSRWX3dhS2Qz6VTVBXzBiYiFcKpYFJ7tH")
您可以在 [Regular Files API](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfslsipfspath) 的文档中探索 [ls](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfslsipfspath) 其他的 `ipfsPath` 格式化选项。

因为该 `ls` 方法返回一个 [Async Iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of)，所以您只能一个接一个地迭代这些值。如果你需要返回所有的值，你可以将每个值保存到一个数组中，然后返回该数组。

要遍历所有值，我们可以使用循环 `for await...of`：

	const result = []
	
	for await (const resultPart of ipfs.ls('/catPics')) {
	    result.push(resultPart)
	}
	
	return result
为了使事情更容易，我们可以使用 [it-all](https://www.npmjs.com/package/it-all) 自动执行此操作的包：

	//all 函数来自 it-all 包，在我们的代码挑战中是全局可用的(就像ipfs一样)
	
	const result = await all(ipfs.ls('/catPics'))
该 `result` 变量现在是一个对象数组，一个对象对应于找到的每个文件或目录，其结构如下：

	{
	  "cid": Object,
	  "path": String,
	  "name": String,
	  "depth": Number,
	  "size": Number,
	  "type": String // 可以是 "file" 或 "directory",
	  "mode": Number
	}
请注意，在 MFS API 中有一个不同的 `ls` 方法(称为 `ipfs.files.ls()` 而不是 `ipfs.ls` )，其属性略有不同，您可以在我们的MFS教程中了解更多信息。

- [demo](https://proto.school/regular-files-api/06)

	请注意，在MFS API中有一个不同的 ls 方法称为 `ipfs.files.ls()`而不是 `ipfs.ls`，其属性略有不同，您可以在我们的MFS教程中了解更多信息

	用于 `ls` 列出您在上一个挑战中创建的目录的内容。

	提示：您在上一个挑战中创建的目录的 CID 已存储在下面起始代码中 `directory` 的 CID 变量中。因为该方法返回的结果中的最后一个元素 `addAll` 始终是包装目录，所以我们的代码找到最后一个元素并保存它的 `cid` 值或 CID。
	
		/* global ipfs, all */
		
		const run = async (files) => {
		  const fileObjectsArray = files.map((file) => { return { path: file.name, content: file }})
		  const addedFiles = await all(ipfs.addAll(fileObjectsArray, { wrapWithDirectory: true }))
		  const directoryCID = addedFiles[addedFiles.length - 1].cid
		
		  return all(ipfs.ls(directoryCID))
		}
		return run
	- 检查结果

		以下是顶级目录的 `ls` 方法返回的结果。请注意，这里有一些我们在方法返回的数据中没有看到的 `addAll` 新字段。另外，看看`cid` 和 `path` 值现在有何不同。每个文件的  `cid`  是文件本身的 CID，而 `path` 是顶级目录的 CID，后跟文件名。
		
			[
			  {
			    "cid": "CID('QmRajrhBHhkenmQYjxuMD1ysWLxCShXRrNJzUqhBH3jtxV')",
			    "path": "Qmey5eWQ1L9bVeQxvbb7VH6UdDh62fbMjrpeCAVaEuWQ2G/Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
			    "name": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
			    "depth": 1,
			    "size": 6368596,
			    "type": "file",
			    "mode": 420
			  }
			]

## 读取目录中的文件 7
到目前为止，我们在访问文件时都是使用文件或目录的 CID 作为路径。然而，既然我们已经了解到我们可以将多个文件包装到一个目录中，我们就有了一种新的文件寻址方式。

正如您在我们关于选项的课程中看到的 `wrapWithDirectory ` 那样，我们可以将一个或多个文件添加到新目录，并且在这样做时，我们需要为我们添加的每个文件提供一个名称。结果，`addAll` 方法调用返回一个数组，其中包含 `cid` 创建的每个文件和目录的值。

如前所述，将使用 `{ wrapWithDirectory: true }` 创建的目录视为命名快捷方式而不是传统的文件夹很有用。原因如下：

想象一下，我们 `addAll` 这样调用方法，以添加两个被目录包裹的文件......

	ipfs.addAll([
	    {
	        path: 'lists/kitty-pic-list.txt',
	        content: catList
	    },
	    {
	        path: 'cat-drinking-milk.jpg',
	        content: catPic
	    }
	], { wrapWithDirectory: true })
...并且该 `addAll` 方法返回了这个结果：

	[
	  {
	    "path": "lists/kitty-pic-list.txt",
	    "cid": CID('Qmey7KyqDwo8BfAoVsyLbybQ8LTN3RGbvvY1zV5PeumTLV'),
	    "size": 19021
	  },
	  {
	    "path": "lists",
	    "cid": CID('QmPT14mWCteuybfrfvqas2L2oin1Y2NCbwzTh9cc33GF1t'),
	    "size": 1093102
	  },
	  {
	    "path": "cat-drinking-milk.jpg",
	    "cid": CID('QmexwNKUeJPmmNR7n4wSzQXrVuyeuQcQikHCHg5xM3mtRq'),
	    "size": 912035
	  },
	  {
	    "path": "",
	    "cid": CID('QmP1j6shbCikCSfnQR7MzJrYdgM6ALpXJAUvkGJFrrwNew'),
	    "size": 1181341
	  }
	]
要读取文件 `kitty-pic-list.txt` 的内容，我们现在有四个寻址选项：

- 使用文件自己的唯一 `cid`，就像我们之前所做的那样：

		ipfs.cat("Qmey7KyqDwo8BfAoVsyLbybQ8LTN3RGbvvY1zV5PeumTLV")
- 使用文件的完整 IPFS 路径（注意如何 `/ipfs/` 添加到 CID 之前）：

		ipfs.cat("/ipfs/Qmey7KyqDwo8BfAoVsyLbybQ8LTN3RGbvvY1zV5PeumTLV")
- 使用包含 `cid` 文件所在目录 ( lists) 和文件相对于该目录的路径的 IPFS 路径：

		ipfs.cat("/ipfs/QmPT14mWCteuybfrfvqas2L2oin1Y2NCbwzTh9cc33GF1t/kitty-pic-list.txt")
- 使用包含 `cid` 顶级目录和文件相对于该目录的路径的 IPFS 路径：

		ipfs.cat("/ipfs/QmP1j6shbCikCSfnQR7MzJrYdgM6ALpXJAUvkGJFrrwNew/lists/kitty-pic-list.txt")

`wrapWithDirectory` 最后两个选项允许我们在 IPFS 路径中包含人类可读的文件名，因此提供了一个有趣的原因，即使我们只添加一个文件，我们也可能选择使用该选项。

请注意，每当我们使用 IPFS 路径 `/ipfs/`（而不仅仅是 CID）来引用文件时，在开头包含它很重要。这是必要的，因为还有其他网络协议使用相同的 CID 但具有不同的路径结构，例如 `libp2p `或 `IPNS`。

- [demo](https://proto.school/regular-files-api/07)

	使用该 `cat` 方法，返回名为 `success.txt ` 的文件的文本内容，该文件存储在直接位于另一个目录中的 `fun`  目录中。

	与这些目录关联的 CID 是：

	- 顶级目录：QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy
	- fun 目录：QmPT14mWCteuybfrfvqas2L2oin1Y2NCbwzTh9cc33GM1r

	由于您不知道文件本身的 CID，因此您需要通过组合目录的 CID 和文件相对于该目录的路径来定义其 IPFS 路径。

	根据我们的文件结构，路径可以表示为：

	- `fun/success.txt`, 相对于顶级目录
	- `success.txt`, 相对于 `fun` 子目录

	有两种有效的方法可以解决这一挑战。随你挑！

	提示：

	- 在下面的代码中，我们已经处理好在返回结果之前将  `bufferedContents`  转换为字符串，就像您之前使用 `.toString()`.
	- 不要忘记 `/ipfs/` 在 IPFS 路径字符串前添加
	- 您需要将所有数据连接到一个缓冲区中，因为该 `ipfs.cat` 方法返回一个 Async Iterable. 你可以使用包 `it-to-buffer`。

	编码
	
	- 方法1

			/* global ipfs, toBuffer */
			
			const run = async () => {
				
			  // 使用顶级架构 cid
			  const bufferedContents = await toBuffer(ipfs.cat("/ipfs/QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy/fun/success.txt"))
			
			  return bufferedContents.toString()
			}
			return run

	- 方法2

			/* global ipfs, toBuffer */
			
			const run = async () => {
	
			  // 使用子目录 cid
			const bufferedContents = await toBuffer(ipfs.cat("/ipfs/QmPT14mWCteuybfrfvqas2L2oin1Y2NCbwzTh9cc33GM1r/success.txt"))
			
			  return bufferedContents.toString()
			}
			return run
	- 返回结果

			You did it!
			
## 获取目录下的所有文件 8 
需要了解有关 IPFS 文件或目录的更多信息，而不仅仅是其内容？虽然 `cat` 仅检索文件的缓冲内容，但我们可以通过调用 `get` 如下方法来访问有关文件或目录的其他元数据：

	ipfs.get(ipfsPath)
结果是一组对象，每个文件或目录一个，具有以下结构：

	{
	    cid: Object,
	    name: String,
	    path: String,
	    depth: Number,
	    size: Number,
	    type: String, //可以是 "file" 或 "dir"
	    content: Buffer, // 如果存在， type 必须等于 "file"
	}
与 `cat` 和 一样，如果在本地节点上找不到，`ls` 该方法将在 IPFS 网络上  `get`  搜索请求的文件或目录。

### `get` 与 `cat` 或 `ls`的用例
### 返回元数据和内容
该 `get` 方法返回所有文件和子目录的元数据，以及所有文件的内容。

请注意，（如上所示）`get` 返回的数据的结构几乎与该 `ls` 方法的结果相同，除了我们现在除了元数据之外还看到返回的内容（如果目标是文件）。

对于文件，除了我们可以使用 `cat` 检索的缓冲内容之外，还 `get` 返回元数据。

就像 `ls,get` 会返回一个 `Async Iterable`，所以我们仍然需要 `it-all` 在它的返回值上使用包。

	await all(ipfs.get(ipfsPath))
#### 递归返回目录内容
`get` 与这些其他方法之间的另一个非常重要的区别是，当在目录上运行时，它将递归地检索内容，这意味着生成的数组将包含指定目录本身以及指定目录中存在的每个子目录和每个文件的条目目录，甚至子目录中的那些。

相比之下，该ls方法返回一个数组，其中仅包含指定目录（无论是文件还是子目录）中顶级内容的元数据。它不返回指定目录本身的条目，也不返回子目录内容的条目。

该 `get` 方法可以：

- 递归检索目录中的每个文件和子目录
- 告诉我们哪些条目是目录，哪些是文件
- 通知我们文件的大小
- 查明文件在目录结构中的深度
- 并提供许多其他细节

- [demo](https://proto.school/regular-files-api/08)

	对于最后一个挑战，我们在您的 IPFS 节点中创建了以下文件结构：

		/
		/shrug.txt
		/smile.txt
		/fun/
		/fun/success.txt

	顶级目录的CID是："QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy"

	使用该 `get` 方法返回有关顶级目录中包含的所有文件和子目录的数据。

	提示：一定要返回方法创建的数组 `get`。
	
		const run = async () => {
		  let result = await all(ipfs.get('QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy'))
		
		  return result
		}
		return run
	
	- 检查结果

		下面是在顶级目录上调用 `get`  该方法的结果。（通常情况下，由于包含缓冲文件内容，结果会更加密集，但我们有意创建了微小的文本文件来限制这种影响。）

		请注意，因为我们使用 `{ wrapWithDirectory: true }` 来创建这些文件，所以这里的每个项目 `path` 都是由顶级目录的 CID 加上项目的相对路径定义的，并且每个文件或子目录都有一个人类可读的 `name`. 只有顶级目录本身有一个与其 `cid`和 `path` 匹配的值，它们都是 name 相同的 CID。

			[
			  {
			    "cid": "CID('QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy')",
			    "path": "QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy",
			    "name": "QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy",
			    "depth": 1,
			    "size": 0,
			    "type": "dir",
			    "mode": 493
			  },
			  {
			    "cid": "CID('QmPT14mWCteuybfrfvqas2L2oin1Y2NCbwzTh9cc33GM1r')",
			    "path": "QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy/fun",
			    "name": "fun",
			    "depth": 2,
			    "size": 0,
			    "type": "dir",
			    "mode": 493
			  },
			  {
			    "cid": "CID('QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ')",
			    "path": "QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy/fun/success.txt",
			    "name": "success.txt",
			    "depth": 3,
			    "size": 11,
			    "type": "file",
			    "content": {},
			    "mode": 420
			  },
			  {
			    "cid": "CID('QmQDHitBegfht9eKo7ZJ7S3haq1QVAysjUZg8tmYdPJmSx')",
			    "path": "QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy/shrug.txt",
			    "name": "shrug.txt",
			    "depth": 2,
			    "size": 13,
			    "type": "file",
			    "content": {},
			    "mode": 420
			  },
			  {
			    "cid": "CID('Qmbfrc4cF2X4KXbHuqD593SLnR2xj6hULYTnrj65wKWaKm')",
			    "path": "QmcmnUvVV31txDfAddgAaNcNKbrtC2rC9FvkJphNWyM7gy/smile.txt",
			    "name": "smile.txt",
			    "depth": 2,
			    "size": 2,
			    "type": "file",
			    "content": {},
			    "mode": 420
			  }
			]
		
		




								

