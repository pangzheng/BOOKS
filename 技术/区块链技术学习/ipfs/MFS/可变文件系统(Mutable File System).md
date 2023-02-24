# 可变文件系统(Mutable File System)
可变文件系统 (MFS) 让您可以像使用传统的基于名称的文件系统一样处理文件和目录
## 介绍 IPFS 1
### IPFS：星际文件系统
IPFS或星际文件系统是一种点对点 (P2P) 网络协议，用于在分布式网络上共享数据。顾名思义，您可以将 IPFS 视为一个文件系统，它具有一些独特的特性，使其成为安全、去中心化共享的理想选择。
### 在 IPFS 中存储和共享数据
当内容添加到 IPFS 网络时，内容存放在哪里？

作为一个点对点数据存储系统，IPFS 允许每个用户（对等点）在本地托管他们想要的任何数据。当你第一次向 IPFS 添加新内容时，你实际上只是在你自己的机器上以适合通过 IPFS 协议共享的格式设置它。通常，您会在自己的计算机上安装 IPFS 并在那里创建一个新的 IPFS 实例（也称为节点）。这就是您的数据在本地存在的位置，由内容地址 (CID) 引用。存储在 IPFS 中的数据可以采用多种形式，但最常见的用例之一是共享传统文件，我们将在本教程中详细了解。

当您有网络连接时，您可以选择与您的同伴共享您的数据或文件，但如果您是唯一拥有特定资源的人，那么当您的机器离线时，您的同伴将无法使用该资源。让多个节点托管相同的文件使它们更容易获得，使用 CID（通过加密哈希创建的唯一内容标识符）使该系统安全。我们将在以后的教程中更多地讨论共享，但现在我们将专注于如何在您自己的 IPFS 实例中处理文件。

### 可变文件系统
因为 IPFS 中的文件是按内容寻址且不可变的，所以你不能编辑文件；相反，每次更改都会创建一个新文件。`可变文件系统 (MFS)是 IPFS 中内置的一个工具`，它使您可以像通常在基于名称的文件系统中一样处理文件——您可以添加、删除、移动和编辑 MFS 文件，并完成更新链接和为您处理的哈希值。它是一种抽象，让您可以像处理可变数据一样处理不可变数据。

`files` 通过 IPFS CLI（命令行界面）和 API 中的命令访问 MFS 。在本教程中，我们将探索 [文件 API](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#the-mutable-files-api)。

如果您以前从命令行处理过文件和目录，许多 MFS 方法看起来会非常熟悉！

让我们开始探索如何在 IPFS 中处理文件！
## 检查目录的状态 2
### 在 ProtoSchool 中处理文件
在我们的 ProtoSchool 教程中，每次您在课程中点击“提交”按钮时，我们都会在浏览器中为您创建一个新的 IPFS 节点。每当您在我们的课程中看到时 `ipfs.someMethod()` ，ipfs 都是一个引用您的 IPFS 实例的变量，也称为节点。您采取的操作只会影响您自己的 IPFS 节点，而不影响属于您的对等节点。

我们正在幕后创建您的 IPFS 节点，因此您可以专注于我们课程的内容，但最终您需要学习通过安装 IPFS 并在终端中运行守护进程来在本地托管您自己的节点。当您准备好进行实验时，您可以在我们的文档中找到安装 IPFS 和初始化节点的说明。

如前所述，与可变文件系统关联的方法是 [文件 API](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#the-mutable-files-api) 的一部分，因此它们将采用 `ipfs.files.someMethod()`. 让我们来看看一个简单的方法，您甚至可以在将任何文件添加到 IPFS 节点之前就开始使用。
### 探索您的 IPFS 节点 ipfs.files.stat 
使用 IPFS 节点时，您经常需要检查文件或目录的状态。您可以使用  [ipfs.files.stat](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsfilesstatpath-options) 来执行此操作，传入您要检查的路径。

例如，要检查位于 (/) 根目录中的名为 `stuff`  本地的目录的状态，我们可以这样调用该方法：

	await ipfs.files.stat('/stuff')
此方法返回一个对象，其中包含有关我们的文件或目录的一些基本数据：

- `cid`（CID 对象）
- `size`（以字节为单位的文件或目录大小的整数,如果是目录，该值为0）
- `cumulativeSize`（一个整数，包含构成文件的 DAGNode 的大小，以字节为单位）
- `type`（可以是 `directory` 或 `file` 的字符串）
- `blocks`（如果类型是 `directory`，这是目录中的文件数；如果类型是 `file`，它是组成文件的块数）、
- `withLocality`（一个布尔值，指示是否存在位置信息）
- `local`（一个布尔值，指示查询的 dag 是否在本地完全存在）
- `sizeLocal`（一个整数，表示本地存在的数据的累积大小）

明白了！目录 `size的` 始终是 0，无论它包含多少条目，因为目录实际上只是一组指向其他文件和目录的链接。相比之下，目录的`cumulativeSize` 会随着目录内容的变化而变化。它不仅表示该目录中所有条目的文件大小，还表示描述这些条目的元数据：类型、块大小等。

重要的是要注意，`stat` 即使您还没有任何东西，您也可以使用您的 IPFS 节点。即使是空节点也有 CID。

运行 `files.stat` 以检查 IPFS 中根目录 (/) 的状态。请务必返回您的结果

#### [demo](https://proto.school/mutable-file-system/02)
- 执行代码

		/* global ipfs */
		
		const run = async () => {
		  return await ipfs.files.stat('/')
		}
		
		return run
- 结果

	这是您的根目录 (/) 的状态。输出很长，因为 CID 在 `Object` 内部表示为 ，但如果您向下滚动，我们将为您提供更精简的视图
	
		{
		  "cid": {
		    "codec": "dag-pb",
		    "version": 0,
		    "hash": {
		      "0": 18,
		      "1": 32,
		      "2": 89,
		      "3": 148,
		      "4": 132,
		      "5": 57,
		      "6": 6,
		      "7": 95,
		      "8": 41,
		      "9": 97,
		      "10": 158,
		      "11": 244,
		      "12": 18,
		      "13": 128,
		      "14": 203,
		      "15": 185,
		      "16": 50,
		      "17": 190,
		      "18": 82,
		      "19": 197,
		      "20": 109,
		      "21": 153,
		      "22": 197,
		      "23": 150,
		      "24": 107,
		      "25": 101,
		      "26": 224,
		      "27": 17,
		      "28": 18,
		      "29": 57,
		      "30": 240,
		      "31": 152,
		      "32": 187,
		      "33": 239
		    }
		  },
		  "size": 0,
		  "cumulativeSize": 4,
		  "blocks": 0,
		  "withLocality": false,
		  "type": "directory",
		  "mode": 493
		}

为了简化输出，我们可以使用属性 `toString(cid)` 上的方法以字符串格式获取 

	CID：QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn。
在以后的课程中，我们将始终展示此简化版本以使其更易于阅读，如下所示。

	{
	  "cid": "CID('QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn')",
	  "size": 0,
	  "cumulativeSize": 4,
	  "blocks": 0,
	  "withLocality": false,
	  "type": "directory",
	  "mode": 493
	}
请注意，我们的节点有一个 CID，即使它还没有内容。每个空的 IPFS 节点都有这个完全相同的 CID，因为它们不存在的内容是相同的！

## 上传文件处理 3
出于安全考虑，Web 浏览器不允许我们直接更改计算机文件系统中的文件。因此，您需要将一个或多个文件上传到您可以在整个教程中使用的浏览器。

在每个挑战中，您会看到您可以通过拖放或从文件资源管理器中选择文件来从计算机上传文件。如果您仔细查看代码编辑器中的  `run`  函数，您会注意到它现在有一个 `files ` 参数。当您从计算机上传文件时，我们将确保它们作为 `files` 数组传递到函数中。只要您不刷新浏览器，这些文件在本教程的下一课中仍然可以访问，但您还可以选择上传不同的文件以供每节课使用。

为了练习，让我们从您的计算机上传一个或多个文件，并查看浏览器作为  `files` 数组接收到的内容。

### [demo](https://proto.school/mutable-file-system/03)
- 首先，通过在下方拖放或单击从文件资源管理器中进行选择来上传一个或多个文件。
- 接下来，在代码编辑器中，删除 run 函数中 `return files` 前面的注释标记(//)。
- 第三步:检查结果

	得到下面的数据，查看现在可以在 `files` 数组中访问的数据。注意，这些文件目前只在浏览器中。在下一课中，我们将看到如何将它们添加到 IPFS。
		
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

## 将文件添加到 MFS 4
现在我们已经可以在浏览器中访问文件，让我们看看如何将它们添加到 IPFS。

要将文件添加到 IPFS，我们可以使用 MFS [files.write](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsfileswritepath-content-options)方法，如下所示：

	await ipfs.files.write(path, content, [options])
MFS `files.write` 方法可以接受 `Buffer、ReadableStream、PullStream、Blob（仅在浏览器中）` 或文件的字符串 `content` 路径（仅在 Node.js 中）形式的文件。由于浏览器中的文件对象是一种 Blob，我们可以开始了！

尽管浏览器中的文件对象碰巧知道自己的文件名，但 Blob 通常不知道，因此 IPFS 无法直接确定现有的文件名。我们必须提供所需的文件名作为 `path`.

这 `path` 是您想要在 IPFS 实例中创建的新路径，包括所需的文件名。（请注意，这是我们正在描述的目标路径，而不是文件已驻留在您的计算机上的路径。）

MFS `files.write` 方法与您可能在自己的计算机上使用过的类似方法一样，实际上是为编辑现有文件的内容而构建的。但是，我们也可以使用它来创建一个全新的文件，方法是提供布尔值选项，`{ create: true }` 以指示应该在给定路径上创建一个文件（如果该文件尚不存在）。

因此，如果我们的浏览器中有一个文件对象，可通过变量 `catPic` 访问，并且我们想将其添加到 IPFS 上的根目录并命名 `cat.jpg` ，我们可以这样做：

	await ipfs.files.write('/cat.jpg', catPic, { create: true })
如果需要，我们可以使用连接（将字符串连接在一起）来创建相同的路径。如果您的文件名是文件对象的属性，这会派上用场，在浏览器中也是如此。

	await ipfs.files.write('/' + catPic.name, catPic, { create: true })
请注意，该 `files.write` 方法不提供返回值。

稍后我们将看看如何将文件添加到根目录以外的目录。

- [demo](https://proto.school/mutable-file-system/04)

	让我们使用 MFS 将一些文件添加到 IPFS。由于 `files` 是一个数组，表示我们当前在浏览器中可用的文件，我们需要遍历该数组并使用 `files.write`  将我们在其中找到的每个文件添加到 IPFS，一次一个。（只上传了一个文件？没问题！）

	将文件放在根目录 (/) 中，并确保在添加文件时在路径中包含每个文件的名称。（提示：浏览器中的文件对象将文件名存储为`file.name`）。请务必设置您的选项，以便在给定路径中找不到文件时创建一个新文件。

		const run = async (files) => {
		  for (let file of files) {
		    await ipfs.files.write('/' + file.name, file, { create: true })
		  }
		}
		
		return run
	
	这是现在位于 IPFS 根目录中的数据
	
		[
		  {
		    "cid": "CID('QmdCTHVBdFyq9YgBn8ugEMVFupb1R7qyTjj62GxHn4R8qj')",
		    "name": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
		    "type": 0,
		    "size": 262144,
		    "mode": 420
		  }
		]	

## 查看目录的内容 5 
当我们使用 `files.write` 将文件添加到 MFS 时，该方法没有返回任何值，但我们仍然可以检查以确保一切都按预期工作。

在可变文件系统中，我们可以使用该 `files.ls` 方法检查目录。如果您曾经使用命令行列出计算机上目录的内容，应该会感到非常熟悉。

该 `files.ls` 方法如下所示：

	ipfs.files.ls([path], [options])
该方法将默认列出根目录 ( path) 的内容或者您​​可以选择指定要检查的特定 `/` （目录），例如，`/catPics`

`files.ls` 生成一组对象，每个对象对应于您正在检查的目录中包含的每个文件或目录，具有以下属性：

- `name`: 文件名
- `type`: 对象的类型（0- 文件或1- 目录）
- `size`：文件的大小，以字节为单位
- `cid`：在 IPFS 中标识您的文件的内容标识符 (CID)
- `mode`: 作为数字的 UnixFS 模式
- `mtimesecs` ：具有数字和 `nsecs`属性的对象

如果我们想检查目录的内容 `/catPics` ，我们可以这样做：

	ipfs.files.ls('/catPics')
因为该 `files.ls` 方法返回一个 [Async Iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of)，所以您只能一个接一个地迭代这些值。如果你需要返回所有的值，你可以将每个值保存到一个数组中，然后返回该数组。

要遍历所有值，我们可以使用循环 `for await...of`：

	const result = []
	
	for await (const resultPart of ipfs.files.ls('/catPics')) {
	    result.push(resultPart)
	}
	
	return result
为了使事情更容易，我们可以使用 [it-all](https://www.npmjs.com/package/it-all) 自动执行此操作的包：

	//all 函数来自 `it-all` 包，在我们的代码挑战中是全局可用的(就像 `ipfs` 一样)
	const result = await all(ipfs.files.ls('/catPics'))

- [demo](https://proto.school/mutable-file-system/05)

	让我们看看 `files.ls` 行动吧！列出根目录的内容 ( /)。

	注意：我们已经包含了将文件添加到 MFS 的代码，因此您可以专注于列出目录的内容。

	提示：不要忘记 `files.ls` 返回一个 Async Iterable，因此您需要使用 `for await...of` 循环或all函数来返回结果。

		/* global ipfs, all */
	
		const run = async (files) => {
		  // 这段代码将您上传的文件添加到 IPFS
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		
		  const rootDirectoryContents = await all(ipfs.files.ls('/'))
	
		  return rootDirectoryContents
		}
		
		return run
	或者使用循环
	
		/* global ipfs, all */
		
		const run = async (files) => {
		  // this code adds your uploaded files to IPFS
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		
		   const rootDirectoryContents = []
		   for await (const item of ipfs.files.ls('/')) {
		     rootDirectoryContents.push(item)
		   }
		  return rootDirectoryContents
		}
		return run
	
	看一看 `ls` 方法返回的完整数据
	
		[
		  {
		    "cid": "CID('QmdCTHVBdFyq9YgBn8ugEMVFupb1R7qyTjj62GxHn4R8qj')",
		    "name": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
		    "type": 0,
		    "size": 262144,
		    "mode": 420
		  }
		]

## 查看 CID 如何随着数据变化而变化 6
正如您在去中心化 Web 教程的内容寻址中了解到的那样，CID（内容标识符）通过加密散列与它们所代表的内容唯一匹配。具有相同内容的两个文件具有相同的 CID，而即使它们之间的最小差异的两个文件也具有不同的 CID。目录也是如此。每次更新文件或目录的内容时，其 CID 都会发生变化。

当您的根目录为空并且您使用 检查其状态时 [ipfs.files.stat](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsfilesstatpath-options)，您会看到以下结果：

	{
	  "cid": CID("QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn"),
	  "size": 0,
	  "cumulativeSize": 4,
	  "blocks": 0,
	  "type": "directory",
	  "withLocality": false
	}
现在您已经向其中添加了一个或多个文件，它会是什么样子？现在应该更改哪些字段？

- [demo](https://proto.school/mutable-file-system/06)

	既然您已经添加了一个或多个文件，请再次运行 `files.stat` 以检查您的根目录 ( /) 的状态。请务必返回您的结果。
			
		/* global ipfs, all */
		
		const run = async (files) => {
		  // 这段代码将您上传的文件添加到IPFS
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		  const rootDirectoryContents = await all(ipfs.files.ls('/'))
		
		  const directoryStatus = await ipfs.files.stat('/')
		
		  return directoryStatus
		}
		
		return run
	检查结果，这是更新后的根目录 (/) 的状态。请注意此数据与您在目录为空时看到的数据相比如何。
	
	- 由于新的内容 `cid` 已经改变
	- `cumulativeSize` 和 `blocks`也是如此 。


			{
			  "cid": "CID('QmZAwY5uViefCi9V783FMKfPEgXwx5HReiPgcYgGUBPGGG')",
			  "size": 0,
			  "cumulativeSize": 262314,
			  "blocks": 1,
			  "withLocality": false,
			  "type": "directory",
			  "mode": 493
			}
	因为目录实际上由指向内容的链接组成，而不是数据本身，所以目录 `size`始终是0.  `cumulativeSize` 更改是因为它不仅代表该目录中所有条目的文件大小，还代表描述这些条目的元数据：类型、块大小等。
	
## 创建目录 7
我们已经学会了如何将文件添加到根目录，但是我们如何创建一个新目录呢？同样，该过程与您在自己计算机上的命令行中可能遇到的过程非常相似。

MFS 方法 [files.mkdir](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsfilesmkdirpath-options) 在指定路径创建一个新目录。例如，要将目录添加 `images` 到我们的根目录 (/)，我们可以这样做：

	await ipfs.files.mkdir('/images')
- 可选 `parents` 属性（默认为false）指定是否应创建给定路径中的任何父目录（如果它们尚不存在）。

	我们不需要上面的内容，因为新 `images` 目录是现有目录 (/) 的直接子目录。但是，如果我们想创建一个新目录，嵌套在其他尚不存在的目录下，我们需要显式设置 `parentsto` 的值 `true`，如下所示：
	
		await ipfs.files.mkdir('/my/beautiful/images', { parents: true })
	明白了！尽管创建缺失路径的目标是相似的，但请注意我们如何使用 `{ parents: true }` 选项 `files.mkdir` 而不是 `{ create: true }` 可用的选项 `files.write`。

- [demo](https://proto.school/mutable-file-system/07)

	`stuff` 在新目录中创建一个新目录 `some`，该目录应位于根目录 (/) 中。
			
		/* global ipfs, all */
		
		const run = async (files) => {
		  // 这段代码将您上传的文件添加到 IPFS 中的根目录
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		
		  await ipfs.files.mkdir('/some/stuff', { parents: true })
		
		  let rootDirectoryContents = await all(ipfs.files.ls('/'))
		  return rootDirectoryContents
		}
		
		return run
	这是在您的根目录中返回的内容 `ls`。请注意
	
		[
		  {
		    "cid": "CID('QmdCTHVBdFyq9YgBn8ugEMVFupb1R7qyTjj62GxHn4R8qj')",
		    "name": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
		    "type": 0,
		    "size": 262144,
		    "mode": 420
		  },
		  {
		    "cid": "CID('QmVneuc3suf78aVdvFY3BW9HoiEfNxD7WB5zu1f9fbun3D')",
		    "name": "some",
		    "type": 1,
		    "size": 0,
		    "mode": 493
		  }
		]	
	
	- 目录的类型是 `1`
	- 而文件的类型是 `0`

## 移动文件或目录 8
MFS 允许您使用 方法在目录之间移动文件，就像在本地计算机上一样 [files.mv](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsfilesmvfrom-to-options)。

该方法如下所示：

	await ipfs.files.mv(from, to, [options])

- `from` 

	是您要移动的内容的源路径（或多个路径）。
- `to`

	是目标路径。

如果您的目标路径引用了尚不存在的父目录，您将需要使用该 `{ parents: true }` 选项，就像您对 `files.mkdir`.

您可以使用它 `files.mv` 来执行许多不同的操作：

	// 将单个文件移动到目录中
	await ipfs.files.mv('/source-file.txt', '/destination-directory')
	
	// 移动一个目录到另一个目录
	await ipfs.files.mv('/source-directory', '/destination-directory')
	
	// 用源文件的内容覆盖目标文件的内容
	await ipfs.files.mv('/source-file.txt', '/destination-file.txt')
要将多个文件移动到目录中，您可以在 `to` 参数前添加多个源路径：

	// 移动多个文件到一个目录
	await ipfs.files.mv('/source-file-1.txt', '/source-file-2.txt', '/source-file-3.txt', '/destination-directory')
如果您从一组源路径开始，您可以使用 JavaScript 的扩展语法将其元素扩展为所需的格式：

	await ipfs.files.mv(...fromArray, '/destination-directory')

- [demo](https://proto.school/mutable-file-system/07)

	仅将文件（无目录）从根目录移动到 `some/stuff` 目录中。

	请记住，您可以将数组 `files.mv` 作为 `from` 值传递给它。这很有用，因为它允许您只运行一次异步函数。请务必使用 `await`关键字，以便在评估目录 `/some/stuff` mv 内容之前完成调用。

	创建要传入的数组时，确保它只包含文件，而不包含目录。`files.ls` 请记住，在 IPFS 中访问的每个对象都有一个 `type` 属性，您可以使用该属性来确定它是文件还是目录；它的值 `0` 适用于文件和`1` 目录。（提示：尝试 [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) 或 [forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) 数组方法。）

	请记住， `files.mv` 需要路径，而不是文件名，因此您需要在数组中的每个文件名前加上 `/`. （提示：尝试 `maporforEach` 数组方法并合并 `name` 通过该 `files.ls` 方法可用的属性。）

	如果您在点击“提交”后遇到延迟，请重试，上传较小的文件进行处理。
	
		/* global ipfs, all */
		
		const run = async (files) => {
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		  await ipfs.files.mkdir('/some/stuff', { parents: true })
		  const rootDirectoryContents = await all(ipfs.files.ls('/'))
		
		  // 创建要移动文件路径的数组(不包括目录)
		  const filepathsToMove = rootDirectoryContents.filter(file => file.type === 0).map(file => '/' + file.name)		  
		  await ipfs.files.mv(...filepathsToMove, '/some/stuff')
		  
		 // 移动 `filepathsToMove` 中的所有文件到 `/some/stuff` 中
		  const someStuffDirectoryContents = await all(ipfs.files.ls('/some/stuff'))
		  return someStuffDirectoryContents
		}
		
		return run
	这是现在在 IPFS 中的 `/some/stuff` 目录中的数据
	
		[
		  {
		    "cid": "CID('QmdCTHVBdFyq9YgBn8ugEMVFupb1R7qyTjj62GxHn4R8qj')",
		    "name": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
		    "type": 0,
		    "size": 262144,
		    "mode": 420
		  }
		]	

## 复制文件或目录 9 
`files.mv` 与将项目移动到目标路径时从其源路径中删除项目不同的是，该 [files.cp](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsfilescpfrom-to-options) 方法允许您将文件或目录复制到新位置，同时在源中保持原样。

该方法如下所示：

	await ipfs.files.cp(...from, to, [options])
但是，您现在有两个格式化选项 `from`。您可以传入：

- 您自己节点中文件或目录的现有 MFS 路径（例如 `/my-dir/my-file.txt`）
- `/ipfs/QmWGeRAEgtsHW3ec7U4qW2CyVy7eA2mFRVbk1nb24jFyks` 由您或对等方（例如）托管的文件或目录的 IPFS 路径
请注意，IPFS 路径以 `/ipfs/CID` 开头和结尾。

如您所见 `files.mv`，`to` 是 MFS 中的目标路径，并且有一个选项 `{ parents: true }` 可用于创建尚不存在的父目录。

您可以使用它 `files.cp` 来执行许多不同的操作：

	// 将单个文件复制到目录中
	await ipfs.files.cp('/source-file.txt', '/destination-directory')
	await ipfs.files.cp('/ipfs/QmWGeRAEgtsHW3ec7U4qW2CyVy7eA2mFRVbk1nb24jFyks', '/destination-directory')

	// 将多个文件复制到一个目录中(注意两种可接受的格式，有[]或没有[])
	await ipfs.files.cp('/source-file-1.txt', '/source-file-2.txt', '/destination-directory')
	await ipfs.files.cp(['/source-file-1.txt', '/source-file-2.txt'], '/destination-directory')
	await ipfs.files.cp('/ipfs/QmWGeRAEgtsHW3ec7U4qW2CyVy7eA2mFRVbk1nb24jFyks',
	 '/ipfs/QmWGeRAEgtsHW3jk7U4qW2CyVy7eA2mFRVbk1nb24jFyre', '/destination-directory')
	await ipfs.files.cp(['/ipfs/QmWGeRAEgtsHW3ec7U4qW2CyVy7eA2mFRVbk1nb24jFyks',
	 '/ipfs/QmWGeRAEgtsHW3jk7U4qW2CyVy7eA2mFRVbk1nb24jFyre'], '/destination-directory')

	// 将一个目录复制到另一个目录
	await ipfs.files.cp('/source-directory', '/destination-directory')
	await ipfs.files.cp('/ipfs/QmWGeRAEgtsHW3ec7U4qW2CyVy7eA2mFRVbk1nb24jFyks', '/destination-directory')

明白了！如果您从 IPFS 路径复制文件而没有明确为其分配文件名，IPFS 会将其 `name` 属性设置为等于其 CID. 要指定更友好的文件名，您需要将其附加到目标路径，如下所示：

	await ipfs.files.cp('/ipfs/QmWGeRAEgtsHW3ec7U4qW2CyVy7eA2mFRVbk1nb24jFyks', '/destination-directory/fab-file.txt')

- [demo](https://proto.school/mutable-file-system/09)

	我们创建了一个包含秘密消息的文本文件，并将其存储在 CID 下的 IPFS 网络上`QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ`。将它从 IPFS 网络复制到 MFS 中的目录中，并通过将所需的文件名附加到 `/some/stuff` 目标路径中，并为其命名 `success.txt`。

	提示： IPFS 路径以 `/ipfs/CID` 开始和结束。
	
		/* global ipfs, all */
		
		const run = async (files) => {
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		  await ipfs.files.mkdir('/some/stuff', { parents: true })
		  const rootDirectoryContents = await all(ipfs.files.ls('/'))
		  const filepathsToMove = rootDirectoryContents.filter(file => file.type === 0).map(file => '/' + file.name)
		  await ipfs.files.mv(...filepathsToMove, '/some/stuff')
		
		  await ipfs.files.cp('/ipfs/QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ', '/some/stuff/success.txt')
		
		  let someStuffDirectoryContents = await all(ipfs.files.ls('/some/stuff'))
		  return someStuffDirectoryContents
		}
		
		return run
	查看 `/some/stuff` 从 IPFS
	
		[
		  {
		    "cid": "CID('QmdCTHVBdFyq9YgBn8ugEMVFupb1R7qyTjj62GxHn4R8qj')",
		    "name": "Javelin_GunneryDeckUpdate_SideV_Hobbins的副本.png",
		    "type": 0,
		    "size": 262144,
		    "mode": 420
		  },
		  {
		    "cid": "CID('QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ')",
		    "name": "success.txt",
		    "type": 0,
		    "size": 11,
		    "mode": 420
		  }
		]

## 读取文件内容 10
MFS 有一种方法可以让您显示缓冲区 [files.read](https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsfilesreadpath-options) 中文件的内容。这使我们能够轻松读取 .txt 文件的内容等。

该方法采用以下格式：

	ipfs.files.read(path, [options])
提供 `path` 的是要读取的文件的路径，它必须指向文件而不是目录。

该 `files.read` 方法返回一个 Async Iterable，它迭代文件的数据块，即缓冲区。在我们的例子中，我们最终需要使用方法将 Buffers 转换为 `toString()` 字符串 。但单个文件中的数据块需要在转换之前重新组合（连接）。该 [it-to-buffer](https://www.npmjs.com/package/it-to-buffer) 包可以遍历所有块并为我们将它们放回一起。（我们已经在我们的挑战中向您提供了这个包 `toBuffer`。）

	// toBuffer 变量全局可用(就像ipfs一样)
	
	let bufferedContents = await toBuffer(ipfs.files.read('/directory/some-file.txt'))  // 一个 buffer
	let contents = bufferedContents.toString() // 一个字符串
当您准备好在现实世界中尝试此操作时，您应该注意到上面的示例可能会导致大量内存使用，具体取决于正在读取的文件的内容。如果您正在处理大文件并发现是这种情况，您可能希望跳过使用该 `it-to-buffer` 包，而是迭代地处理每个数据块。IPFS 现在回归的主要原因`Async Iterables` 是提供一个内置选项来处理潜在的性能问题。在 ProtoSchool 教程中，我们的代码挑战使用小文件，因此我们可以连接所有内容而无需担心性能。

- [demo](https://proto.school/mutable-file-system/10)

	我们在您从 IPFS 复制的那个文件中为您隐藏了一条秘密消息 `success.txt`。用于 `files.read` 将文件内容（作为字符串）保存到`secretMessage` 我们在下面为您创建的变量中。

	提示：请记住，您需要将 返回的缓冲区转换 `files.read` 为字符串。
	
		/* global ipfs, all, toBuffer */
		
		const run = async (files) => {
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		  await ipfs.files.mkdir('/some/stuff', { parents: true })
		  let rootDirectoryContents = await all(ipfs.files.ls('/'))
		  const filepathsToMove = rootDirectoryContents.filter(file => file.type === 0).map(file => '/' + file.name)
		  await ipfs.files.mv(...filepathsToMove, '/some/stuff')
		  await ipfs.files.cp('/ipfs/QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ', '/some/stuff/success.txt')
		  let someStuffDirectoryContents = await all(ipfs.files.ls('/some/stuff'))
		
		  let secretMessage = (await toBuffer(ipfs.files.read('/some/stuff/success.txt'))).toString('utf8')
		
		  return secretMessage
		}
		
		return run
	返回，这是你在文件中发现的秘密信息:
		
		You did it!

## 删除文件或目录 11
MFS 允许您使用 `files.rm` 方法删除文件或目录 ：

	await ipfs.files.rm(...paths, [options])
`paths` 是一个或多个要删除的路径。

默认情况下，如果您尝试删除仍有内容的目录，请求将失败。要删除目录及其中包含的所有内容，您需要使用该选项 `{ recursive: true }`。

	// 删除文件
	await ipfs.files.rm('/my/beautiful/file.txt')
	
	// 删除多个文件(注意两种可接受的格式，有[]或没有[])
	await ipfs.files.rm('/my/beautiful/file.txt', '/my/other/file.txt')
	await ipfs.files.rm(['/my/beautiful/file.txt', '/my/other/file.txt'])
	
	// 删除目录(不为空)及其内容(要删除目录及其中包含的所有内容)
	await ipfs.files.rm('/my/beautiful/directory', { recursive: true })
	
	// 仅当目录为空时才删除该目录
	await ipfs.files.rm('/my/beautiful/directory')

- [demo](https://proto.school/mutable-file-system/11)

	删除 `some` 目录，包括 `stuff` 目录及其所有内容。此操作应将您的根目录留空。
	
		/* global ipfs, all */
		
		const run = async (files) => {
		  await Promise.all(files.map(f => ipfs.files.write('/' + f.name, f, { create: true })))
		  await ipfs.files.mkdir('/some/stuff', { parents: true })
		  let rootDirectoryContents = await all(ipfs.files.ls('/'))
		  const filepathsToMove = rootDirectoryContents.filter(file => file.type === 0).map(file => '/' + file.name)
		  await ipfs.files.mv(...filepathsToMove, '/some/stuff')
		  await ipfs.files.cp('/ipfs/QmWCscor6qWPdx53zEQmZvQvuWQYxx1ARRCXwYVE4s9wzJ', '/some/stuff/success.txt')
		  let someStuffDirectoryContents = await all(ipfs.files.ls('/some/stuff'))
		
		  await ipfs.files.rm('/some', { recursive: true })
		
		  let finalRootDirectoryContents = await all(ipfs.files.ls('/'))
		  return finalRootDirectoryContents
		}
		
		return run
	返回
	
	您的函数返回了一个空数组 `([])`，因为根目录中没有内容。它的状态 `(stat)` 如下所示。注意，我们回到了与开始时完全相同的CID !
	
		{
		  "cid": "CID('QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn')",
		  "size": 0,
		  "cumulativeSize": 4,
		  "blocks": 0,
		  "withLocality": false,
		  "type": "directory",
		  "mode": 493
		}

	

	
	
		


## 参考
[Mutable File System](https://proto.school/mutable-file-system)