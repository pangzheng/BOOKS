# IPLD 中的遍历
“遍历”是指遍历 IPLD 数据的行为。

- 遍历 [map] (https://ipld.io/docs/data-model/kinds/#map-kind)

	意味着迭代它的键和值......并且可能递归地遍历这些值。
- 遍历[list](https://ipld.io/docs/data-model/kinds/#list-kind)

	意味着遍历它的值......并且可能递归地遍历这些值。
- 遍历非递归的其他类型的节点

	仅意味着查看该节点。
- 遍历[links](https://ipld.io/docs/data-model/kinds/#link-kind)

	可能意味着检查该链接或者加载它，从而产生一整套可以遍历的新节点。

## 遍历是通用的
可以遍历任何 IPLD 数据。一旦数据通过[编解码器](https://ipld.io/glossary/#codec)加载到[数据模型](https://ipld.io/docs/data-model/)中，数据模型就意味着我们知道如何查看数据。由于处于数据模型中意味着我们知道它是什么[类型](https://ipld.io/docs/data-model/kinds/)的数据，这意味着我们知道如何对其进行迭代......这意味着我们知道如何遍历它以及它的所有相邻数据。
## 遍历顺序
遍历顺序一般都有一个稳定的定义。

但是，该顺序的确切含义可能会根据您正在执行的遍历类型而有所不同——如果您正在使用任何东西来指导遍历，或者如果您只是简单地遍历整个数据图。

另请参阅有关 IPLD 中一般订购的扩展文档：[Tricky Design Choices: Ordering](https://ipld.io/design/tricky-choices/ordering/)

## 遍历的种类
- 完全遍历

	“完全遍历”是指按自然顺序遍历数据图。按照其迭代器产生条目的顺序遍历地图；按照迭代器产生条目的顺序遍历列表；ETC。

	全遍历不同于[定向遍历](https://ipld.io/docs/data-model/traversal/#directed-traversal)。
- 定向遍历

	“定向遍历”意味着在掌握一些关于如何做的特定方向的同时遍历数据图。定向遍历可能仅访问图形的某些部分，或者可能根据其特定方向以特定顺序访问数据。

	定向遍历不同于[完全遍历](https://ipld.io/docs/data-model/traversal/#total-traversal)。

	有向遍历是一个通用术语，可以指任何一种对自定义代码的遍历；还有一些标准化的特性，许多 IPLD 库将提供这些特性来执行定向遍历，例如 [选择器](https://ipld.io/docs/data-model/traversal/#traversal-by-selector)
- 选择器遍历

	IPLD 选择器是一种特定机制，用于以声明方式描述如何遍历 [IPLD dag](https://ipld.io/glossary/#dag)，以及如何标记和选择遍历期间到达的一些节点。

	您可以将选择器大致视为文本数据的正则表达式，但用于 IPLD 图。

	[选择器是一种定向遍历的形式](https://ipld.io/docs/data-model/traversal/#directed-traversal)。因此，某些选择器可能会导致遍历以与对相同数据的普通完全遍历不同的顺序进行（当然，遍历的数据也比普通总遍历少得多——这就是重点他们，毕竟）。

	有关选择器中的定向遍历顺序可能与规范有何不同的示例，请考虑“字段”选择器：在此选择器子句中，选择器本身具有选择器要关注的字段的（有序！）map，并且正是这种排序将驱动对应用该子句的 map 的选择，而不是它所应用的 map 中数据的排序。（这很重要，因为即使对于任意大小的 map ，它仍然是正确的——记住，考虑到 ADLs，选择器可以应用于一个巨大的 map ，内部分片......如果我们有让我们避免它的说明！）

在[选择器规范](https://ipld.io/specs/selectors/)中查看有关选择器的更多信息。

- 各种IPLD实现中的遍历
	- 在 go：
		- [go-ipld-prime](https://github.com/ipld/go-ipld-prime) 中：
		- 整个 [traversal (godoc)](https://godoc.org/github.com/ipld/go-ipld-prime/traversal) 包