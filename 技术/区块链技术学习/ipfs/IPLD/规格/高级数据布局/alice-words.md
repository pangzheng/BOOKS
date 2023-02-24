# alice-words
该测试构建了一个 HAMT ，对 [Lewis Carroll 的爱丽丝梦游仙境](https://www.gutenberg.org/files/11/11-h/11-h.htm) 第一章中出现的所有单词的位置进行编目。

- 参考文本在 [words.txt](http://ipld.io.ipns.localhost:8080/specs/advanced-data-layouts/hamt/fixture/alice-words/words.txt) 中可用。
- 使用简单的正则表达式匹配单词 `/\w+/`。
- HAMT 使用 `bitWidth `of 5（每个节点最多 32 个条目或桶）和 `bucketSize `of 3.
- 块使用 DAG-CBOR 编码，CID 使用 SHA2-256 多重哈希。
- 每个条目都以单词为关键字（根据 `Bytes` [HAMT规范](http://ipld.io.ipns.localhost:8080/specs/advanced-data-layouts/hamt/spec/)），值以内联方式存储。
- 值是单词每次出现的行和列位置的数组，按出现顺序排列。
- 值采用以下形式：
		
		type Value [Datum]
		type Datum struct {
		  line Int
		  column Int
		}
- 编译后：
	- 这种 HAMT 的根 CID 应该是 `bafyreic672jz6huur4c2yekd3uycswe2xfqhjlmtmm5dorb6yoytgflova`. 该根被列为包含块的 CAR 的根。
	- 将有 36 个块：1 个根和 25 个非根，与 HAMT 规范中的模式匹配。
- 档案：
	- [hamt.car](http://ipld.io.ipns.localhost:8080/specs/advanced-data-layouts/hamt/fixture/alice-words/hamt.car) - 包含 36 个块的 CAR
	- [hamt.json](http://ipld.io.ipns.localhost:8080/specs/advanced-data-layouts/hamt/fixture/alice-words/hamt.json) - 包含在 HAMT 中的数据的排序 JSON 表示，具有单词到文本位置的映射。
	- [words.txt](http://ipld.io.ipns.localhost:8080/specs/advanced-data-layouts/hamt/fixture/alice-words/words.txt) - 爱丽丝梦游仙境第一段用于生成数据的源文本。