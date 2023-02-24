# HAMT ADL
HAMT ADL 提供 [map](https://ipld.io/docs/data-model/kinds/#map-kind) 接口，同时在内部对数据进行分片。

HAMT 可以通过使用分片在许多块之间分布数据来支持非常大的数据量。

HAMT 有一个特别有趣和有用的特性，即没有滞后性——这意味着任何两个具有相同内容的 HAMT 都将达到相同的分片模式，而不管数据的插入顺序或数据结构如何增长的历史。这对确定性来说是个好消息，并且使 HAMT 在去中心化系统设计中非常有价值。
## 规范：HashMap
### 状态：规定 - 草案
## 介绍
IPLD HashMap 提供多块键/值存储，并将 [Map 类型](https://ipld.io/docs/data-model/kinds/#map-kind) 实现为 IPLD 类型系统中的高级数据布局。

IPLD HashMap 被构造为一个[Hash-array-mapped-trie (HAMT)](https://en.wikipedia.org/wiki/Hash_array_mapped_trie)，带有用于存储值的桶和 [CHAMP](https://michael.steindorfer.name/publications/oopsla15.pdf) 突变语义。CHAMP 不变和变异规则使我们能够在给定任何一组键及其值的情况下维护规范形式，而不管插入顺序和中间数据插入和删除。因此，对于任何给定的一组键及其值、一致的 IPLD HashMap 配置和块编码、根节点应该始终产生相同的内容标识符 (CID)。
## 有用的参考资料
- [Phil Bagwell 的Fast And Space Efficient Trie Searches](https://infoscience.epfl.ch/record/64394/files/triesearches.pdf) 2000 年
- 和 Phil Bagwell 的 [Ideal Hash Trees](http://lampwww.epfl.ch/papers/idealhashtrees.pdf) ，2001 年介绍了 AMT 和 HAMT 概念。
- Steinforder & Vinju 在 Oopsla 2015 上发表的 [CHAMP 论文](https://michael.steindorfer.name/publications/oopsla15.pdf)
- 原始 CHAMP 论文附带的 [Java 实现](https://github.com/msteindorfer/oopsla15-artifact/)（参见[https://github.com/msteindorfer/oopsla15-artifact/blob/master/pdb.values/src/org/eclipse/imp/pdb/facts/util/TrieMap_5Bits.java](https://github.com/msteindorfer/oopsla15-artifact/blob/master/pdb.values/src/org/eclipse/imp/pdb/facts/util/TrieMap_5Bits.java)和其他 TrieMap 文件在同一目录中）。
- [Optimizing Hash-Array Mapped Tries for Fast and Lean Immutable JVM Collections](https://blog.acolyer.org/2015/11/27/hamt/) 对一般 HAMT 数据结构和 CHAMP 细节的高级描述。
- Peergos [CHAMP 实施](https://github.com/Peergos/Peergos/blob/master/src/peergos/shared/hamt/Champ.java)。
- [ipld-hashmap](https://github.com/rvagg/js-ipld-hashmap) 带有可变 API 的 JavaScript IPLD 实现。
	- [IAMap](https://github.com/rvagg/iamap)算法的低级 JavaScript 实现具有不可变的 API，ipld-hashmap 是该库的 IPLD 原生可变前端。
- Lotus客户端使用的 Filecoin Go HAMT 实现 [go-hamt-ipld](https://github.com/filecoin-project/go-hamt-ipld) 。有关此实现与此规范的不同之处，请参阅附录。

## 概括
HAMT 算法用于构建 IPLD HashMap。这种算法在许多语言标准库中都很常见，特别是在 JVM (Clojure、Scala、Java)上，可以支持非常高效的内存中无序键/值存储数据结构。我们扩展了基本算法，增加了块弹性桶和严格的突变规则，以确保规范形式。

HAMT 算法对传入的键进行哈希，并在其树结构的每一层使用该哈希的递增子节来确定条目或到树的子节点的链接的位置。`bitWidth` 决定了在树的每一层用于索引计算的哈希位数，这样根节点使用哈希的第一个 `bitWidth` 位来计算索引，当我们在树中向下移动时，我们沿着哈希移动深度 `x bitWidth` 位。通过这种方式，一个足够随机的哈希函数将生成一个哈希，在数据结构的每个级别上提供一个新的索引。由 `bitWidth` 位组成的索引将生成[0,2<sup>bitwidth</sup>)的索引值。因此，`bitWidth`为 8 将生成0到255(含255)的索引。

因此，树中的每个节点最多可以保存2<sup>bitWidth</sup>的数据元素，我们将其存储在数组中。在 IPLD HashMap 中，我们将条目存储在桶中。一个 `Set(key, value)` 突变，其中在根节点上为 key 的散列生成的索引表示一个数组索引，该数组索引还不包含条目，我们创建一个新桶并插入键/值对条目。通过这种方式，单个节点理论上可以容纳最多2<sup>bitWidth</sup> x bucketSize 条目，其中`bucketSize` 是一个 bucket 允许包含的最大元素数量(“碰撞”)。在实践中，指数的分布不是完全随机的，所以这个最大值是理论上的。存储在节点桶中的条目以键排序的顺序存储。

如果 `Set(key, value)` 突变在一个已经包含 `bucketSize` 条目的桶中放置了一个新条目，则溢出到一个新的子节点。一个新的空节点被创建，桶中的所有现有条目以及新的键/值对条目都被插入到这个新节点中。我们从每个键的哈希值开始增加索引计算的深度，以计算新节点的数据数组中的位置。通过增加深度，我们沿着每个键的散列中的 `bitWidth` 位移动。使用足够随机的哈希函数，在前一层生成相同索引的每个键应该大致均匀地分布在新节点的数据数组中，从而产生一个包含最多 `bucketSize` 的新 `bucket` 的节点。

从哈希值的 `bitWidth` 子节生成索引值的过程为我们的树数据结构提供了高达 `(digestLength x 8) / bitWidth` 级别的深度，其中`digestLength` 是哈希函数生成的输出字节数。每个节点可以存储最多2<sup>bitWidth</sup>的子节点引用，最多 `bucketSize` 的元素可以存储在碰撞叶位置，我们可以存储大量的条目。哈希函数的随机性将决定元素的均匀分布，哈希函数的输出 `digestLength` 将决定树的最大深度。

进一步优化，降低了 HAMT 节点的存储需求。数据元素数组只被分配到足够长的长度来存储实际的条目:非空桶或到实际子节点的链接。空数组索引或 `Null` 数组索引不会用作该节点中不存在键的信号。相反，数据元素数组通过使用 `map` 位域进行压缩，其中 `map` 的每个位对应于节点中的一个索引。当生成索引时，将检查 `map` 位字段的索引位。如果没有设置位 `(0)`，则该索引不存在。如果位设置为`(1)`，则该值存在于数据元素数组中。为了确定数据元素数组的索引，我们对 `map` 位域执行位计数 `(popcount())`，直到索引位，以生成一个 `dataIndex`。通过这种方式，数据元素数组的总长度等于 `popcount(map) `(map中设置的所有位的数量)。如果 `map` 的所有位都设置了，那么数据元素数组的长度将是 2<sup>bitWidth</sup>，即每个位置将包含一个桶或一个子节点的链接。

使用 `Set(key, value)` 插入新桶包括在 `dataIndex` 位置的数据数组中拼接一个新元素，并设置 `map` 位图的索引位。将桶转换为子节点时，映射位映射将保持不变，因为索引位仍然指示该位置有一个元素。

`Get(key)` 操作在根节点上执行相同的哈希、索引和 `dataIndex` 计算，遍历到桶中以找到匹配 `key` 的条目或遍历到子节点并执行相同的索引和 `dataIndex` 计算，但在 `key`的哈希中偏移了额外的` bitWidth `位。

突变 `Delete(key)` 首先以相同的方式定位元素，`Get(key)` 如果该条目存在，则将其从包含它的桶中删除。如果在删除条目后桶为空，我们从数据元素数组中完全删除桶元素并取消设置的 `index` 位map。如果包含已删除元素的节点没有到子节点的链接并且包含 `bucketSize` 删除后的元素，则这些元素将被压缩到一个桶中并放置在父节点中以代替到该节点的链接。我们对父节点执行此检查（如果需要，则递归执行），从而将树转换为最紧凑的形式，只用桶代替所有边上最多有条目的节点 `bucketSize`。这种压实过程结合 `key` 桶中条目的排序为任何给定的 `key/value` 对集合生成规范形式的数据结构，而不管它们的插入顺序或是否添加和删除了任何中间条目。

默认情况下，IPLD HashMap 中的每个节点都存储在不同的 IPLD 块中，CID 用于子节点链接。此处介绍的模式和算法还允许内联子节点而不是链接，读取操作能够遍历单个块内的多个节点，其中它们被内联。内联 IPLD HashMap 的生产未指定，用户应注意内联会破坏规范形式保证。

## 结构
### 参数
任何给定 IPLD HashMap 的可配置参数：

- `hashAlg`


	应用于键的哈希算法，以便在整个数据结构中均匀分布条目。该算法是根据速度 `digestLength` 和随机性属性选择的（但它必须可供读者使用，因此需要共享默认值，见下文）。
- `bitWidth`

	在数据结构的每个级别使用的位数，用于确定条目的索引或数据结构的下一级的链接以继续搜索。该等式 2<sup>bitWidth</sup> 得出 HashMap 节点的数量，即存储桶的存储位置数和/或到子节点的链接。
- `bucketSize`

	条目存储桶的最大数组大小，超过  `bucketSize`  会导致创建新的子节点来替换条目存储。

### 节点属性
HashMap 数据结构中的每个节点包含：

- `data`

	一个数组，长度为 1 到 2 <sup>bitWidth</sup>。
- `map`

	一个位域，存储为字节，其中第一位 2 <sup>bitWidth</sup> 用于指示在节点的每个可能索引处是否存在桶或子节点链接。

HAMT 的一个重要属性是 `data` 数组只包含活动元素。不包含任何值的节点中的索引（在桶中或到子节点的链接中）不被存储，并且位域 `map` 用于确定 `data` 值是否存在以及使用 [popcount()](https://en.wikipedia.org/wiki/Hamming_weight). 这允许我们为每个节点存储一个最大压缩 `data` 数组。

### 模式
IPLD HashMap 的`根块`包含与所有其他块相同的属性，此外还包含指示下面的算法如何遍历和改变数据结构的配置数据。

有关此格式的定义，请参阅 [IPLD 模式](http://ipld.io.ipns.localhost:8080/docs/schemas/)。

	# 根节点布局
	type HashMapRoot struct {
	  hashAlg Int
	  bucketSize Int
	  hamt HashMapNode
	}
	
	# 非根节点布局
	type HashMapNode struct {
	  map Bytes
	  data [ Element ]
	} representation tuple
	
	type Element union {
	  | &HashMapNode link
	  | Bucket list
	} representation kinded
	
	type Bucket [ BucketEntry ]
	
	type BucketEntry struct {
	  key Bytes
	  value Any
	} representation tuple
- 笔记：

- `hashAlg` 在根块中是一个整数代码，用于标识用于将键映射到它们在结构中的位置的哈希算法。该代码应对应于[多格式表](https://github.com/multiformats/multicodec/blob/master/table.csv)中的[多哈希代码](https://github.com/multiformats/multihash)。
- `bitWidth` 在根块中必须至少为 3，使得最小 `map` 大小为 1 字节。
-` bitWidth` 没有出现在根块中，因为它是从 map 字节数组的大小推断出来的，等式为 `log2(byteLength(map) x 8)`，是 `map` 大小等式 2 <sup>bitWidth</sup>/8的倒数。对于低级语言，您可以使用更便宜的等效 `trailingZeroBits(byteLength(map)) + 3`。
- `bucketSize` 在根块中必须至少是 1.
- keys 存储在 `Byte` 表单中。
- `Element` 是一个 kinded union，它支持存储一个 `Bucket`（如 kind `list`）或指向子节点的链接（如 kind `link`）。

## 详细算法
### Get(key)
1. 设置一个 `depth` 值为 0，表示根块
- 使用 `hashAlg` 对键进行哈希
- 从哈希中取最左边的 `bitWidth` 位，偏移深度 `x bitWidth`，形成一个索引。在数据结构的每一层，我们都增加从哈希中取出的比特的部分，以便随着我们向下移动，索引包含不同的比特集。
- 如果节点 map 中的索引位为 0 ，则可以确定该键在该数据结构中不存在，因此返回一个空值(适用于实现平台)。
- 如果节点 map 中的索引位为 1，则可能存在该值。在到索引的 map 上执行 `popcount()`，以便我们计算到索引位位置的1位数。这为我们提供了 `dataIndex`，这是数据数组中的一个索引，用于查找值或插入一个新桶。
- 如果数据的 `dataIndex` 元素包含到子块的链接(CID)，则增加深度，并重复步骤3中由链接标识的子节点。
- 如果 `data` 的 `dataIndex` 元素包含一个桶(数组)，则遍历桶中的条目:
	- 如果一个条目具有我们正在寻找的键，则返回该值。
	- 如果没有条目包含我们要查找的键，则返回一个空值(适用于实现平台)。请注意，桶将按键排序，因此当扫描产生的键大于键时，扫描可以停止。

### Set(key, value)
1. 设置一个 `depth` 值为 0，表示根块
- 使用 `hashAlg` 对键进行哈希
- 从哈希中取最左边的 `bitWidth` 位，偏移深度 `x bitWidth`，形成一个索引。在数据结构的每一层，我们都增加从哈希中取出的比特的部分，以便随着我们向下移动，索引包含不同的比特集。
- 如果节点 map 中的索引位为 0，则需要在当前节点上创建一个新桶。
- 如果节点 map 中的索引位为 1，则该节点数据中存在该索引的值，该值可能是一个桶(可能已满)，也可能是到子节点的链接。
- 在到索引的 map 上执行 `popcount()`，以便我们计算到索引位位置的1位数。这为我们提供了`dataIndex`，这是数据数组中的一个索引，用于查找值或插入一个新桶。
- 如果节点 map 的索引位为 0:
	- 修改当前节点(创建副本)。
	- 在 `dataIndex` 的数据中插入一个新元素，该元素包含一个新桶(数组)，其中键/值对只有一个条目。
	- 为突变的节点创建 CID。
	- 如果 `depth` 为 0,CID 表示 HashMap 的新根块。
	- 如果 `depth` 大于 0:
		- 改变节点的父节点
		- 将突变子节点的新 CID 记录在突变父节点数据数组的适当位置。
		- 递归地继续，通过在父节点的突变副本中记录每个节点的新 CID，直到深度为 0 时，我们生成新的根块及其 CID。
- 如果节点映射的索引位为1:
	- 如果 `data` 的 `dataIndex` 元素包含到子节点的链接(CID)，则增加 `depth`:
		- 如果 `(depth x bitWidth) / 8` 现在大于 `digestLength`，则会发生 “max collisions” 失败，并将错误状态返回给用户。
		- 如果 `(depth x bitWidth) / 8` 小于散列中的字节数，则重复步骤3中识别的子节点。
	- 如果 `data` 的 `dataIndex` 元素包含一个桶(数组)，并且桶的大小小于 `bucketSize`:
		- 修改当前节点(创建副本)。
		- 如果 `key` 已经存在于新桶中，则将其对应的值更改为新桶。否则，将键/值对插入到按键排序的位置，这样桶中的所有条目都按键排序。这有助于确保规范形式。
		- 继续按照步骤 `6.c` 为当前块和每个父块创建新的 `cid`。直到我们有了一个新的根块和它的 `CID`。
	- 如果 `data` 的 `dataIndex` 元素包含一个桶(数组)，并且桶的大小为 `bucketSize`:
		- 创建一个新的空节点
		- 对于桶的每个元素，从步骤2开始，在新的空节点上执行 `Set(key, value)`，深度设置为 `depth + 1`。这将创建一个新节点，其中 `bucketSize` 元素大约均匀地分布在其数据数组中。如果所设置的所有键在 `bitWidth` 位置深度+ 1处的哈希值具有相同的`bitWidth` 位，则此操作只会导致创建多个新节点(依此类推)。一个足够随机的哈希算法应该可以防止这种情况发生。
		- 为新的子节点创建CID。
		- 改变当前节点(创建一个副本)
		- 将 `data` 的 `dataIndex` 替换为到新子节点的链接。
		- 继续按照步骤 `6.c` 为当前块和每个父块创建新的 cid。直到我们有了一个新的根块和它的 CID。
	
### Delete(key)
下面的删除算法是一个迭代操作。它也可以被看作是一种递归算法，这在节点崩溃的情况下特别有用。该算法的描述请参见 [CHAMP 论文](https://michael.steindorfer.name/publications/oopsla15.pdf)的 4.2 删除算法。请注意，链接的论文没有使用桶，因此请注意在节点中计算条目并在下面的算法中与 `bucketSize` 进行比较的重要性。

1. 设置一个 `depth` 值为 0，表示根块
- 使用 `hashAlg` 对键进行哈希
- 从哈希中取最左边的 `bitWidth` 位，偏移深度 `x bitWidth`，形成一个索引。在数据结构的每一层，我们都增加从哈希中取出的比特的部分，以便随着我们向下移动，索引包含不同的比特集。
- 如果节点映射中的索引位为0，则可以确定该键在该数据结构中不存在，因此不需要继续。
- 如果节点映射中的索引位为1，则可能存在该值。在到索引的映射上执行 `popcount()`，以便我们计算到索引位位置的1位数。这为我们提供了 `dataIndex`，这是数据数组中的一个索引，用于查找值或插入一个新桶。
- 如果 `data` 的 `dataIndex` 元素包含到子节点的链接 (CID)，则增加深度并重复步骤3中标识的子节点。
- 如果 `data` 的 `dataIndex` 元素包含一个桶(数组)，则遍历桶中的条目:
	- 如果没有条目包含我们要查找的键，则不需要继续。请注意，桶将按键排序，因此当扫描产生的键大于键时，扫描可以停止。
	- 如果一个条目具有我们正在寻找的键，我们需要删除它，并可能折叠该节点和任意数量的父节点，这取决于剩余条目的数量。这有助于确保规范形式。请注意，下面有两种可能的状态，如果两种情况都不匹配当前节点的状态，那么我们没有满足不变量，树也没有处于规范状态(即某些事情失败了):
		- 如果 `depth` 为 `0`(根节点)，或者数据数组中有这个节点的链接(它有子节点)或者这个节点中所有桶的条目数量目前大于`bucketSize + 1`，我们不需要将这个节点折叠到它的父节点中，而是可以简单地从它的桶中删除条目。
			- 改变当前节点(创建一个副本)
				- 如果位于数据数组 `dataIndex` 的桶包含多个元素，则从 `data` 的 `dataIndex` 的桶中删除该条目(其余元素必须保持按键排序)。
				- 如果位于数据数组 `dataIndex` 的桶只包含一个元素:
					- 删除数据的 `dataIndex`(这样数据现在就少了一个元素)
					- 设置 `map` 的索引位为 0
				- 为删除了条目的新节点创建 CID。
				- 将突变子节点的新 CID 记录在突变父节点数据数组的适当位置。
				- 递归地继续，通过在父节点的突变副本中记录每个节点的新 CID，直到深度为 0 时，我们生成新的根块及其CID。
		- 如果 `depth` 不为 0(不是根节点)，并且在该节点的数据数组中没有链接(它没有子节点)，并且该节点中所有桶的条目数当前等于 `bucketSize + 1`，那么当删除的条目被删除时，该节点需要被折叠成一个 `bucket (bucketSize)`，并在其父数据数组中替换到该节点的链接。
			- 创建一个新桶，并将除被移除到新桶中的条目之外的所有条目放在节点中。新桶现在包含来自该节点的所有条目，并将用于替换父节点中的节点。
			- 改变父节点(创建一个副本)。
			- 用新创建的桶替换到父数据数组中节点的链接。(注意在父节点的数据数组中的位置将依赖于键在 `depth - 1` 处的索引和从父节点的映射中计算出来的 `dataIndex`)。
			- 为新的父进程创建CID。
			- 如果父节点的深度为0，即父节点，CID 表示新的根节点。
			- 如果父节点的深度不是0，那么在父节点上重复步骤 7.2。这个过程应该沿着树一直重复到深度为0，在这个过程中可能会将多个节点折叠到其父节点中。

### Keys(),Values() 和 Entries()
这些跨越集合的迭代操作对于实现是 `可选`的。

IPLD HashMap 中条目的存储顺序完全取决于哈希算法和 `bitWidth`。因此，出于实际目的，IPLD HashMap 被认为是随机的（与按构造排序或按比较器排序相反，请参阅[IPLD 多块集合/集合类型](https://github.com/ipld/specs/blob/master/data-structures/multiblock-collections.md#collection-types%5D)）。由实现来决定用于遍历条目的树遍历顺序和算法。

一个实现应该在每次迭代中只发出一次任何给定的 `key,value` 或 `key/value` 条目对。
### 与 CHAMP 的不同之处
该算法在以下方面与 CHAMP 不同：

- CHAMP 将 `map` 分为数据映射和节点映射，用于引用本地数据元素和对子节点的本地引用。然后将数据数组分成两半，这样数据元素从左边存储，子节点链接从右边存储，并使用反向索引。这允许对完全内存中的数据结构进行重要的速度和缓存位置优化，但这些优化在分布式数据结构中不存在，或者影响可以忽略不计。
- CHAMP 不使用桶，也不使用 JVM 上 HAMT 的常见实现（例如 Clojure、Scala、Java）。仅存储条目和链接消除了在桶内进行迭代搜索和比较的需要，并允许直接遍历所需的条目。这对内存数据结构有效，但在使用分布式数据结构执行逐块遍历时用处不大，其中打包数据以减少遍历可能更为重要。

### 规范形式
为了实现任何给定的 `key/value` 对集合的规范形式，我们注意到算法中的以下属性：

- 在插入和删除操作期间，我们必须保持桶排序 `key`。
- 我们必须保留一个不变量，规定任何非根节点都不能直接或通过子节点的链接包含少于 `bucketSize + 1` 条目。通过在删除过程中严格应用这一点，我们可以概括为没有指向子节点的链接的非根节点可能包含少于 `bucketSize + 1` 条目。在删除过程中违反此规则的树中的任何非根节点必须将其条目折叠到一个 `bucketSize` 长度的桶中（即不删除条目）并插入其父节点以代替到受影响节点的链接。我们继续在树上递归地应用此规则，可能会将其他节点折叠到它们的父节点中。

## 作为 “Set” 使用
IPLD HashMap 可以重新用作“Set”：一种仅包含唯一 keys 的数据结构。value 突变中的每个都 `Set(key, value)` 固定为一些微不足道的值，例如 `true` 或 1。`Has(key)` 操作只是一个 `Get(key)` 断言返回值的操作。
## 实现默认值
实现需要带有合理的默认值并且能够创建 HashMap 而无需用户深入了解算法和所有权衡（尽管这些知识将有助于它们的最佳使用）。

这些默认值是描述性的而不是规定性的。新的实现可能会选择不同的默认值，同时承认它们将为与下面列出的默认值相同的数据生成不同的图表（以及 CID）。还可以为用户提供覆盖这些默认值的设施，以适应这些默认值不会产生最佳结果的用例。

- `hashAlg`

	用于编写 IPLD HashMaps 的默认支持的哈希算法是 `SHA2-256`（由 `multihash multicodec code` 标识0x12）。
鼓励哈希算法的可插拔性，以允许用户在他们的用例有令人信服的理由时切换开关。这种可插拔性需要提供一种接受字节并返回字节的算法。更改哈希算法的用户需要意识到这种可插入性限制了其他实现读取其数据的能力，因为匹配的哈希算法也需要在读取端提供。
- `bitWidth`

	用于写入 IPLD hashmap 的默认 `bitWidth` 是8。该值产生的数据长度为28=256。8在大多数编程语言中也很容易分割字节列表，因为它是一个简单的字节索引。然而，实现应该设计成支持读取 IPLD hashmap 时遇到的不同位宽。支持的最小 `bitWidth` 必须为 3(对于1字节映射)。没有指定最大值，但是实现者应该意识到较大的 `bitWidth` 值可能会引起互操作性问题。对于低级语言，您可以通过等效的 `1 << (bitWidth - 3)` 计算出数据长度。
- `bucketSize`

	用于写入 IPLD hashmap 的默认 `bucketSize` 为3。结合 `bitWidth` 为8，这将产生一个理论上最大的完整节点(没有子节点) `256 x 3 = 768` 个键/值对。最小支持的 `bucketSize` 应该是1。没有指定最大值，但是实现者应该意识到，如果 `bucketSize` 值非常大，可能会出现互操作性问题。

### 最大密钥大小
实现可能会为编写 IPLD HashMap 施加最大密钥大小。本规范未定义使用大于其定义的最大读取键值读取 IPLD HashMap。
### 内联值
实现可以选择将所有值写入单独的块中，并仅将 cid 存储在 IPLD HashMap 中的值位置中。另外，还可以应用粗略的大小启发式来决定内联块还是链接块。或者这个决定可以由用户通过一些 API 选择来决定。由于本规范允许在值位置中存储任意类型的存储，因此实现应该支持读取操作。
## 未来可能的改进和研究领域
### 最大深度限制
IPLD 集合的一个目标是支持任意大的数据集。此规范不满足此要求，因为散列中的位数强加了最大深度。

本规范的未来迭代可能探索：

- 输出更多位数的默认散列算法（例如 SHA2-256 等加密散列函数）。
- 一旦达到最大深度，重置 `index` 计算以从散列的开头获取位，允许或理论上无限深度数据结构。
- `bucketSize` 允许最大深度节点的灵活性。

### 桶
需要研究在 IPLD HashMap 中使用桶对数据结构布局的影响，并适当量化成本和收益。根据此类研究的结果，可能会从本规范的未来版本中删除存储桶。
### 安全
到目前为止，还没有针对 IPLD 数据结构的已知散列冲突攻击向量。可以想象，在某些用例中，用户输入能够影响 `keys` 并且与所选对象发生碰撞 `hashAlg` 是可行的。在这种情况下，可以构建一个 IPLD HashMap，从而使其 `(digestLength x 8) / bitWidth` 快速达到最大深度，进一步的冲突添加会导致错误和可能的拒绝服务。IPLD 当前的用例不适合这种拒绝服务攻击。进一步的实际应用和研究可能会改变这种理解，并决定需要具有大字节输出的哈希算法和/或没有已知冲突缺陷的加密哈希算法。

## 附录：Filecoin HAMT 变体
[Filecoin 项目](https://filecoin-project.github.io/specs/)区块链使用 HAMT，它使用与本文档中概述的相同 HAMT 和存储桶和 CHAMP 突变语义。它直接编码为 [DAG-CBOR](http://ipld.io.ipns.localhost:8080/specs/codecs/dag-cbor/)，但使用与此处指定的不同的块布局。本节记录了 Filecoin HAMT 变体与本规范不同的具体方式。IPLD HashMap 实现可能能够实现一种形式，在用户请求时提供与 Filecoin 的兼容性。

Filecoin HAMT 的参考 Go 实现由 [Lotus](https://lotu.sh/) 客户端使用，可在 [https://github.com/filecoin-project/go-hamt-ipld](https://github.com/filecoin-project/go-hamt-ipld) 获得。请注意，本文档反映了 Go 模块 v3 之前的 Filecoin HAMT 的详细信息，其最新版本在 v3.1.0 撰写本文时已包含在 Filecoin [spec-actors](https://github.com/filecoin-project/specs-actors) v3中。
### 隐式和固定参数
Filecoin HAMT 不使用显式根块 ( `HashMapRoot` ) 在数据中编码其参数。相反，预计数据的消费者会随着时间的推移从 Filecoin 规范和区块链版本控制的组合中理解参数。所有 HAMT 节点都采用相同的形式，根节点没有区别，并且实现必须在解码每个节点时带上隐式参数。

- `hashAlg`
	
	Filecoin HAMT 使用的哈希算法是 SHA2-256。
- `bitWidth`

	Filecoin HAMT 将位宽固定为 5，这意味着 HAMT 的每个节点最多可以包含 2<sup>5</sup>(32) 个包含桶或子节点链接的元素。
- `bucketSize`

	Filecoin HAMT 将其桶的最大长度固定为 3，这意味着最大完整的 HAMT 叶节点可以包含 32 x 3(96) 个键/值对。

### map(bitfield)布局
Filecoin HAMT 使用 Go [big.int](https://golang.org/pkg/math/big/#Int) 的字节布局，而不是将 HAMT 节点的映射字段编码为带有小端字节寻址的字节到元素映射的完整序列。`Int `作为位字段表示 HAMT 节点的 map。实际上，这意味着字节数组的长度可以根据位域中设置的比特而变化。实现必须使用与 `big` 相同的编码和解码规则。`Int` 来处理该字段。有关细节的探索，请参阅这个[讨论线程](https://github.com/filecoin-project/go-hamt-ipld/issues/54#issuecomment-670314664)。
### Block 布局
描述 Filecoin HAMT 的 IPLD 模式与 IPLD HashMap [模式](http://ipld.io.ipns.localhost:8080/specs/advanced-data-layouts/hamt/spec/#schema)不同，并且在 Filecoin Actors 的版本之间也有所不同，因此任何需要读取 Filecoin HAMT 块的实现都需要处理其特定布局（如果历史数据对创建/读取很重要，则两种布局).

Filecoin HAMT 的 v3 形式具有与 IPLD HashMap 相同的[底层](http://ipld.io.ipns.localhost:8080/glossary/#substrate)数据模型布局，只是缺少用于描述参数的显式根节点。Filecoin HAMT 的根相当于 `HashMapNode` ——根节点与所有其他节点相同，参数不是显式的，而是隐式的，必须从环境 Filecoin 链版本中派生。

两种形式的块布局都显示在下面的模式中。块布局的不同之处在于版本0.9到2对结构使用键控联合 `Element`（在 Filecoin 中称为“指针”），而版本3和更高版本使用 kinded 联合。`FilecoinHAMTNodeV0` 并且在下面作为不同版本的 IPLD HashMap 中的`FilecoinHAMTNodeV3` 等价物呈现。`HashMapNode` 没有相当于 `HashMapRoot`.

请注意，目前对可用于存储的类型没有限制，`value` 只要它们可以从字节中解码即可。实际上，Filecoin HAMT 用于存储内联对象而不是对象链接。

	# 相当于“HashMapNode”，但同时作为根节点和中间节点
	# v0.9-v2 form
	type FilecoinHAMTNodeV0 struct {
	  map Bytes          # field is named "bitfield" in Filecoin code
	  data [ PointerV0 ] # field is named "pointers" in Filecoin code
	} representation tuple
	
	# v3+ form
	type FilecoinHAMTNodeV3 struct {
	  map Bytes          # field is named "bitfield" in Filecoin code
	  data [ PointerV3 ] # field is named "pointers" in Filecoin code
	} representation tuple
	
	# 相当于“元素”
	# v0.9-v2 form
	type PointerV0 union {
	  | &FilecoinHAMTNodeV0 "0"
	  | Bucket "1"
	} representation keyed
	
	# v3+ form
	type PointerV3 union {
	  | &FilecoinHAMTNodeV3 link
	  | Bucket list
	} representation kinded
	
	type Bucket [ KV ]
	
	# 相当于"BucketEntry"
	type KV struct {
	  key Bytes
	  value Any
	} representation tuple