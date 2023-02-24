## 概述
IPLD 数据模型是 IPLD 的核心部分。它描述了 IPLD 中可表示的数据——例如，布尔值、整数、文本字符串、映射和列表等。

数据模型抽象地描述了这些数据范围。

数据模型与您对 JSON 的了解和期望大致相似——但我们添加了一些更重要的数据类型：字节和链接（内容可寻址、基于散列的链接）。有关这些的更多详细信息，请参见[种类页面](https://ipld.io/docs/data-model/kinds/)。

有了对数据模型的这种理解，我们就可以做一些事情，比如指定 [路径](https://ipld.io/docs/data-model/pathing/) 和 [遍历](https://ipld.io/docs/data-model/traversal/) 符合它的任何数据，这让我们可以编写“通用”代码来检查和操作数据模型中的数据。

然后 IPLD 的其他部分更详细地指定如何将数据移入和移出：

- [编解码器](https://ipld.io/docs/codecs/)

	准确指定数据模型中的这些信息如何来回转录为序列化字节。
- [IPLD 模式](https://ipld.io/docs/schemas/)

	在数据模型之上提供了额外的可选工具，可以进一步细化、描述和限制可接受数据值的范围（以及执行某些类型的基本透镜和转换）。
- [ADL](https://ipld.io/docs/advanced-data-layouts/) 

	Advanced Data Layouts 是一种插件系统，它允许将复杂数据显示为“普通”数据模型，这对于编程访问很有用。

## 动机
目前在内容寻址数据结构中广泛使用的不是一种 `块格式`，而是多种 `串行格式`。我们假设将来会看到更多而不是更少的这种块格式。很明显，使用这些数据结构的一种合理且更具前瞻性的方法是与串行格式无关。

数据模型定义了基本类型的通用表示，这些基本类型可以 `很容易地用通用编程语言表示` ，并且`可以在最常见和最成功的序列化格式中找到`。通过围绕这些最常见的关键元素进行选择和标准化，我们希望能够为最大数量的现有串行数据构建最顺畅的桥梁，并为图书馆实现低摩擦，图书馆应该能够使用熟悉的方式构建程序员首选语言的本机类型。

### 数据模型中的页面
- [节点](https://ipld.io/docs/data-model/node/)
- [种类](https://ipld.io/docs/data-model/kinds/)	
- [路径](https://ipld.io/docs/data-model/pathing/)
- [遍历](https://ipld.io/docs/data-model/traversal/)		
