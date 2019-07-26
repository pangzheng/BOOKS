# IPFS包数据存储
	import "github.com/ipfs/go-datastore"
## 索引
- 变量

		var ErrBatchUnsupported = errors.New("this datastore does not support batching")
	如果数据存储区实际上不支持批处理，则按批处理返回ErrBatchUnsupported。
		
		var ErrInvalidType = errors.New("datastore: invalid type error")
	当给定值与数据存储区支持的类型不兼容时，由 Put 返回ErrInvalidType。这意味着需要事先进行转换（或序列化）。
		
		var ErrNotFound = errors.New("datastore: key not found")
	当数据存储未将给定键映射到值时，Get，Has 和 Delete将返回 ErrNotFound。

- func [DiskUsage](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L142)

	DiskUsage 检查数据存储区是否为PersistentDatastore 并返回其 DiskUsage（），否则返回 0。
- func [GetBackedHas](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L209)

		func GetBackedHas(ds Read, key Key) (bool, error)
	GetBackedHas 提供默认的 Datastore.Has 实现，如下所示
	
		func (*d SomeDatastore) Has(key Key) (exists bool, err error) {
		
			return GetBackedHas(d, key)
		}					
- func [GetBackedSize](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L227)

		func GetBackedSize(ds Read, key Key) (int, error)
	GetBackedSize 提供默认的 Datastore.GetSize 实现。如下所示：
	
		func (*d SomeDatastore) GetSize(key Key) (size int, err error) {
		
			return GetBackedSize(d, key)
		}
- func [NamespaceType](https://github.com/ipfs/go-datastore/blob/master/key.go#L273)

		func NamespaceType(namespace string) string
	NamespaceType 是命名空间的第一个组件。`foo：bar`中的`foo`
- func [NamespaceValue](https://github.com/ipfs/go-datastore/blob/master/key.go#L282)

		func NamespaceValue(namespace string) string
	NamespaceValue 返回命名空间的最后一个组件。在`f:b:baz`中的`baz`
- type [Batch](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L235)

		type Batch interface {
		    Write
		
		    Commit() error
		}
	- func [NewBasicBatch](https://github.com/ipfs/go-datastore/blob/master/batch.go#L16)
	
			func NewBasicBatch(ds Datastore) Batch
- type [Batching](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L95)	

		Batching interface {
		    Datastore
		
		    Batch() (Batch, error)
		}

	批处理数据存储支持对数据库的延迟的分组更新。`Batch`没有事务语义：不保证在相同的时间内对基础数据存储区进行更新。同样，在调用 “Commit” 之前，不会将批量更新刷新到基础数据存储区。来自`TxnDatastore`的 `Txn`具有`Batch`的所有功能，但反之则不行。
- type [CheckedDatastore](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L107)

		type CheckedDatastore interface {
		    Datastore
		
		    Check() error
		}
	CheckedDatastore 是一个应该由数据存储实现的接口，可能需要检查磁盘数据的完整性。
- type [Datastore](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L34)

		type Datastore interface {
		    Read
		    Write
		    io.Closer
		}
	数据存储区表示任何键值对的存储。

	数据存储通常足以支持各种不同的存储：内存高速缓存，数据库，远程数据存储，磁盘上的平面文件等。

	一般的想法是将一个更复杂的存储设施包装在一个简单，统一的界面中，保持使用正确工具的。特别是，数据存储区可以以有趣的方式聚合其他数据存储，例如分片（分发负载）或分层访问（数据库之前的缓存）。

	虽然数据存储的编写应足以接受各种值，但某些实现无疑必须具体（例如，应将字段分解为列的SQL数据库），尤其是有效支持查询。此外，某些数据存储可能会强制执行某些类型的值（例如，需要 io.Reader，特定结构等）或序列化格式（JSON，Protobufs等）。

	重要提示：没有数据存储应该是 Panic！这是一个跨模块的接口，因此它应该具有可预测性，并通过适当的错误报告处理异常情况。因此，所有数据存储区调用都可能返回错误，应由调用方检查。
- type [GCDatastore](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L124)

		type GCDatastore interface {
		    Datastore
		
		    CollectGarbage() error
		}			
	GCDatastore 是一个应该由数据存储实现的接口，它不会通过从中删除数据来释放磁盘空间。
- type [Key](https://github.com/ipfs/go-datastore/blob/master/key.go#L33)

		type Key struct {
		    // 包含已过滤或未导出的字段 
		}
	Key 表示对象的唯一标识符。key 方案受文件系统和Google App Engine key 模型的启发。

	key 在系统中是唯一的。key 是分层的，包含越来越多的特定命名空间。因此，key 可以被视为其他 key 的“children”或“ancestors”:
	
		Key("/Comedy")
		Key("/Comedy/MontyPython")	
	此外，可以对每个命名空间进行参数化以嵌入相关的对象信息。例如，Key`name`（最具体的命名空间）可以包含对象类型:
	
		Key("/Comedy/MontyPython/Actor:JohnCleese")
		Key("/Comedy/MontyPython/Sketch:CheeseShop")
		Key("/Comedy/MontyPython/Sketch:CheeseShop/Character:Mousebender")
	- func [EntryKeys](https://github.com/ipfs/go-datastore/blob/master/key.go#L296)
	
			func EntryKeys(e []dsq.Entry) []Key	
		EntryKeys
	- func [KeyWithNamespaces](https://github.com/ipfs/go-datastore/blob/master/key.go#L63)
	
			func KeyWithNamespaces(ns []string) Key
		KeyWithNamespaces 从命名空间片段构造一个键。
	- func [NewKey](https://github.com/ipfs/go-datastore/blob/master/key.go#L38)	
	
			func NewKey(s string) Key
		NewKey 从 string 构造一个 key。它会清理价值。
	- func [RandomKey](https://github.com/ipfs/go-datastore/blob/master/key.go#L256)
	
			func RandomKey() Key
		
		RandomKey 返回随机（uuid）生成的 key	
		
			RandomKey()
			NewKey("/f98719ea086343f7b71f32ea9d9d521d")
	- func [RawKey](https://github.com/ipfs/go-datastore/blob/master/key.go#L45)
	
			func RawKey(s string) Key 				
		RawKey 在没有安全检查输入的情况下创建新密钥。小心使用。
	- func (Key) [BaseNamespace](https://github.com/ipfs/go-datastore/blob/master/key.go#L145)
	
			func (k Key) BaseNamespace() string	
		BaseNamespace 返回此键的 “base” 命名空间（path.Base（filename））
		
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").BaseNamespace()
			"Actor:JohnCleese"
	- func (Key) [Bytes](https://github.com/ipfs/go-datastore/blob/master/key.go#L85)
	
			func (k Key) Bytes() []byte
		
		Bytes 将 Key 的字符串值作为[]字节返回	
	- func（Key）[Child](https://github.com/ipfs/go-datastore/blob/master/key.go#L193)
	
			func (k Key) Child(k2 Key) Key
		
		Child 返回此 Key 的`child`键。
		
			NewKey("/Comedy/MontyPython").Child(NewKey("Actor:JohnCleese"))
			NewKey("/Comedy/MontyPython/Actor:JohnCleese")
	- func (Key) [ChildString](https://github.com/ipfs/go-datastore/blob/master/key.go#L207)
	
			func (k Key) ChildString(s string) Key
		ChildString 返回此 Key - 字符串帮助的`child`键。
			
			NewKey("/Comedy/MontyPython").ChildString("Actor:JohnCleese")
			NewKey("/Comedy/MontyPython/Actor:JohnCleese")
	- func (*Key) [Clean](https://github.com/ipfs/go-datastore/blob/master/key.go#L68)
	
			func (k *Key) Clean()					
		使用 path.Clean 清理 key。
	- func（Key）[Equal](https://github.com/ipfs/go-datastore/blob/master/key.go#L90)
	
			func (k Key) Equal(k2 Key) bool
		等于检查两个键的相等性
	- func (Key) [Instance](https://github.com/ipfs/go-datastore/blob/master/key.go#L167)
	
			func (k Key) Instance(s string) Key
			
		Instance 返回此类型键的“实例”（将值附加到命名空间）
		
			NewKey("/Comedy/MontyPython/Actor").Instance("JohnClesse")
			NewKey("/Comedy/MontyPython/Actor:JohnCleese")
	- func (Key) [IsAncestorOf](https://github.com/ipfs/go-datastore/blob/master/key.go#L214)
	
			func (k Key) IsAncestorOf(other Key) bool
		IsAncestorOf 返回此键是否是 “other” 的前缀
		
			NewKey("/Comedy").IsAncestorOf("/Comedy/MontyPython")
			true
	- func (Key) [IsDescendantOf](https://github.com/ipfs/go-datastore/blob/master/key.go#L224)
	
			func (k Key) IsDescendantOf(other Key) bool
		IsDescendantOf 返回此键是否包含另一个作为前缀。
		
			NewKey("/Comedy/MontyPython").IsDescendantOf("/Comedy")
			true
	- func (Key) [IsTopLevel](https://github.com/ipfs/go-datastore/blob/master/key.go#L232)
	
			func (k Key) IsTopLevel() bool
		IsTopLevel 返回此键是否只有一个名称空间
	- func (Key) [Less](https://github.com/ipfs/go-datastore/blob/master/key.go#L95)		
	
			func (k Key) Less(k2 Key) bool
		较少检查此键是否排序低于另一个
	- func (Key) [List](https://github.com/ipfs/go-datastore/blob/master/key.go#L119)
	
			func (k Key) List() []string
		List返回此Key的`list`表示
		
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").List()
			["Comedy", "MontyPythong", "Actor:JohnCleese"]
	- func (Key) [MarshalJSON](https://github.com/ipfs/go-datastore/blob/master/key.go#L238)
	
			func (k Key) MarshalJSON() ([]byte, error)		
		MarshalJSON 实现了 json.Marshaler 接口，键表示为JSON字符串
	- func (Key) [Name](https://github.com/ipfs/go-datastore/blob/master/key.go#L160)
	
			func (k Key) Name() string	
		Name 返回此键的“名称”（最后一个名称空间的字段）
		
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").Name()
			"JohnCleese"
	- func (Key) [Namespaces](https://github.com/ipfs/go-datastore/blob/master/key.go#L138)
	
			func (k Key) Namespaces() []string
		命名空间返回构成此 Key 的`namespaces`。
		
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").Namespaces()
			["Comedy", "MontyPython", "Actor:JohnCleese"]
	- func (Key) [Parent](https://github.com/ipfs/go-datastore/blob/master/key.go#L182)
	
			func (k Key) Parent() Key
		Parent返回此Key的`parent`键。
		
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").Parent()
			NewKey("/Comedy/MontyPython")
	- func (Key) [Path](https://github.com/ipfs/go-datastore/blob/master/key.go#L174)
	
			func (k Key) Path() Key	
		Path返回此键的“路径”（父+类型）。
		
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").Path()
			NewKey("/Comedy/MontyPython/Actor")
	- func (Key) [Reverse](https://github.com/ipfs/go-datastore/blob/master/key.go#L126)
	
			func (k Key) Reverse() Key	
		反向返回此键的反向
		
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").Reverse()
			NewKey("/Actor:JohnCleese/MontyPython/Comedy")
	- func (Key) [String](https://github.com/ipfs/go-datastore/blob/master/key.go#L80)
	
			func (k Key) String() string
		字符串是Key的字符串值
	- func (Key) [Type](https://github.com/ipfs/go-datastore/blob/master/key.go#L153)	
	
			func (k Key) Type() string
		Type返回此键的“type”（最后一个命名空间的值）
			
			NewKey("/Comedy/MontyPython/Actor:JohnCleese").Type()
			"Actor"
	- func (*Key) [UnmarshalJSON](https://github.com/ipfs/go-datastore/blob/master/key.go#L244)
	
			func (k *Key) UnmarshalJSON(data []byte) error	
			
		UnmarshalJSON 实现了 json.Unmarshaler 接口，键将解析指定为字符串键的任何值
- type [KeySlice](KeySlice)

		type KeySlice []Key
	KeySlice 将 sort.Interface 的方法附加到[] Key，按递增顺序排序。
	
	- func (KeySlice) [Len](https://github.com/ipfs/go-datastore/blob/master/key.go#L291)

			func (p KeySlice) Len() int
	- func (KeySlice) [Less](https://github.com/ipfs/go-datastore/blob/master/key.go#L292)

			func (p KeySlice) Less(i, j int) bool
	- func (KeySlice) [Swap](https://github.com/ipfs/go-datastore/blob/master/key.go#L293)

			func (p KeySlice) Swap(i, j int)
- type [LogBatch](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L210)	

		type LogBatch struct {
		    Name string
		    // 包含过滤或未导出的字段 
		}
	LogBatch 记录批处理中的所有访问
	
	- func (*LogBatch) Commit
	
			func (d *LogBatch) Commit() (err error)
		提交实现 Batch.Commit
	- func (*LogBatch) Delete
	
			func (d *LogBatch) Delete(key Key) (err error)
		删除实现 Batch.Delete	
	- func (*LogBatch) Put 
	
			func (d *LogBatch) Put(key Key, value []byte) (err error)
		put 实现 Batch.Put				
- type LogDatastore

		type LogDatastore struct {
		    Name string
		    // 包含已过滤或未导出的字段
		}
	LogDatastore 记录通过数据存储区的所有访问

	- func NewLogDatastore
	
			func NewLogDatastore(ds Datastore, name string) *LogDatastore
		NewLogDatastore 构造一个日志数据存储区
	- func (*LogDatastore) [Batch](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L215)
	
			func (d *LogDatastore) Batch() (Batch, error)
	- func (*LogDatastore) [Check](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L255)
	
			func (d *LogDatastore) Check() error
	- func (*LogDatastore) [Children](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L156)
	
			func (d *LogDatastore) Children() []Datastore
		children 实现的 shim
	- func (*LogDatastore) [Close](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L250)
	
			func (d *LogDatastore) Close() error
	- func (*LogDatastore) [CollectGarbage](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L269)
	
			func (d *LogDatastore) CollectGarbage() error
	- func (*LogDatastore) [Delete](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L186)
	
			func (d *LogDatastore) Delete(key Key) (err error)
		删除实现 Datastore.Delete
	- func（* LogDatastore）[DiskUsage](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L192)
	
			func (d *LogDatastore) DiskUsage() (uint64, error)	
		DiskUsage 实现 PersistentDatastore 接口。
	- func (*LogDatastore) [Get](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L168)
	
			func (d *LogDatastore) Get(key Key) (value []byte, err error)
	
		Get 实现 Datastore.Get
	- func (*LogDatastore) [GetSize](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L180)
	
			func (d *LogDatastore) GetSize(key Key) (size int, err error)
		GetSize 实现 Datastore.GetSize
	- func (*LogDatastore) [Has](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L174)
	
			func (d *LogDatastore) Has(key Key) (exists bool, err error)
		Has 实现 Datastore.Has
	- func (*LogDatastore) [Put](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L161)
	
			func (d *LogDatastore) Put(key Key, value []byte) (err error)
		Put 实现 Datastore.Put		
	- func (*LogDatastore) [Query](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L198)
	
			func (d *LogDatastore) Query(q dsq.Query) (dsq.Results, error)
	
		Query 实现 Datastore.Query
	- func (*LogDatastore) [Scrub](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L262)
	
			func (d *LogDatastore) Scrub() error
- type [MapDatastore](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L12)

		type MapDatastore struct {
		    // 包含已过滤或未导出的字段
		}
	MapDatastore 使用标准 Go 映射进行内部存储

	- func NewMapDatastore

			func NewMapDatastore() (d *MapDatastore)
		NewMapDatastore 构造一个 MapDatastore。 默认情况下 `_not_` 线程安全，如果你需要线程安全，请使用 `sync.MutexWrap` 换行。
	- func (*MapDatastore) [Batch](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L78)

			func (d *MapDatastore) Batch() (Batch, error)
	- func (*MapDatastore) [Close](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L82)

			func (d *MapDatastore) Close() error
	- func (*MapDatastore) [Delete](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L55)

			func (d *MapDatastore) Delete(key Key) (err error)
	Delete 实现 Datastore.Delete
	- func (*MapDatastore) [Get](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L32)

			func (d *MapDatastore) Get(key Key) (value []byte, err error)
	Get 实现 Datastore.Get
	- func (*MapDatastore) [GetSize](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L47)

			func (d *MapDatastore) GetSize(key Key) (size int, err error)
	GetSize 实现 Datastore.GetSize
	- func (*MapDatastore) [Has](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L41)

			func (d *MapDatastore) Has(key Key) (exists bool, err error)
	Has 实现 Datastore.Has
	- func (*MapDatastore) [Put](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L26)
	
			func (d *MapDatastore) Put(key Key, value []byte) (err error)
		Put 实现 Datastore.Put
	- func (*MapDatastore) [Query](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L64)
	
			func (d *MapDatastore) Query(q dsq.Query) (dsq.Results, error)
		Query 实现 Datastore.Query
- type [NullDatastore](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L88)

		type NullDatastore struct {
		}
	NullDatastore 不存储任何内容，但符合API。有用的测试。
	- func [NewNullDatastore](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L92)
	
			func NewNullDatastore() *NullDatastore
		NewNullDatastore 构造一个 null datastoe
	- func (*NullDatastore) [Batch](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L126)
	
			func (d *NullDatastore) Batch() (Batch, error)
	- func (*NullDatastore) [Close](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L130)
	
			func (d *NullDatastore) Close() error
	- func (*NullDatastore) [Delete](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L117)
	
			func (d *NullDatastore) Delete(key Key) (err error)
		Delete 实现 Datastore.Delete
	- func (*NullDatastore) [Get](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L102)
	
			func (d *NullDatastore) Get(key Key) (value []byte, err error)
		Get 实现 Datastore.Get
	- func (*NullDatastore) [GetSize](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L112)
	
			func (d *NullDatastore) GetSize(key Key) (size int, err error)
		Has 实现 Datastore.GetSize
	- func (*NullDatastore) [Has](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L107)
	
			func (d *NullDatastore) Has(key Key) (exists bool, err error)
		Has 实现 Datastore.Has
	- func (*NullDatastore) [Put](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L97)
	
			func (d *NullDatastore) Put(key Key, value []byte) (err error)
		Put 实现 Datastore.Put
	- func (*NullDatastore) [Query](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L122)
	
			func (d *NullDatastore) Query(q dsq.Query) (dsq.Results, error)
		Query 实现 Datastore.Query
- type [PersistentDatastore](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L132)

		type PersistentDatastore interface {
		    Datastore
		
		    // DiskUsage 返回数据存储使用的空间（以字节为单位.
		    DiskUsage() (uint64, error)
		}
	PersistentDatastore 是一个应该由数据存储实现的接口，可以报告磁盘使用情况。
- type [Read](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L58)

		type Read interface {
		    // Get 检索由 `key` 命名的对象`value`。
		    // 如果密钥未映射到值，则Get将返回ErrNotFound
		    Get(key Key) (value []byte, err error)
		    
		
		    // 返回是否将`key`映射到`value`。
		    // 在某些情况下，只检查是否存在可能要便宜得多
		    // 一个值，而不是检索值本身。（例如HTTP HEAD）
		    // 默认实现在`GetBackedHas`中找到
		    Has(key Key) (exists bool, err error)
		    
		
		    // GetSize返回`key`命名的`value`的大小
		    // 在某些情况下，只能获得大小的便宜得多
		    // 值而不是检索值本身
		    GetSize(key Key) (size int, err error)
		    
		
		    // Query 搜索数据存储区并返回查询结果。
		    // 可能在查询实际运行之前返回。要等待查询：
		    //
		    //   result, _ := ds.Query(q)
		    //
		    // 使用频道界面; 结果可能会在不同的时间进入
		    //   for entry := range result.Next() { ... }
		    //
		    //   // 或者等待查询完全完成
		    //   entries, _ := result.Rest()
		    //   for entry := range entries { ... }
		    //
		    Query(q query.Query) (query.Results, error)
		}
	Read 是数据存储区接口的读取接端。
- type [ScrubbedDatastore](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L116)

		type ScrubbedDatastore interface {
		    Datastore
		
		    Scrub() error
		}
	ScrubbedDatastore 是一个应该由数据存储实现的接口，数据存储要提供检查数据完整性和/或纠错的机制。
- type [Shim](https://github.com/ipfs/go-datastore/blob/master/basic_ds.go#L141)

		type Shim interface {
		    Datastore
		
		    Children() []Datastore
		}							
	Shim是一个有 Children 的数据存储区。
- type [TTL](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L158)

		type TTL interface {
		    PutWithTTL(key Key, value []byte, ttl time.Duration) error
		    SetTTL(key Key, ttl time.Duration) error
		    GetExpiration(key Key) (time.Time, error)
		}
	TTL 包含了处理具有生存时间的条目的方法。
- type [TTLDatastore](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L152)

		type TTLDatastore interface {
		    Datastore
		    TTL
		}
	TTLDatastore 是一个应该由支持过期条目的数据存储实现的接口。
- type [Txn](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L169)

		type Txn interface {
		    Read
		    Write
		
		    // 提交最终确定事务，尝试将其提交到数据存储区。
		    // 如果事务已过时，可能会返回错误。存在的		    // 错误表示数据未提交到数据存储区。
		    Commit() error
		    
		    // 丢弃抛弃事务中记录的更改而不提交
		    // 他们到基础数据存储区。在提交后调用丢弃的任何调用
		    // 已成功调用将对事务没有影响
		    // 数据存储区的状态，可以安全推迟。
		    Discard()
		}
	Txn 扩展了数据存储区类型。Txns 允许用户将数据存储区的查询和突变批量转换为原子组或事务。在成功调用 Commit 之前，对事务执行的操作将不会保留。同样，可以在成功提交之前调用 Discard 来中止事务。
- type [TxnDatastore](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L186)

		type TxnDatastore interface {
		    Datastore
		
		    NewTransaction(readOnly bool) (Txn, error)
		}
	TxnDatastore 是一个应该由支持事务的数据存储实现的接口。
- type [Write](https://github.com/ipfs/go-datastore/blob/master/datastore.go#L41)

		type Write interface {
		    // 将存储由`key`命名的对象`value`
		    //
		    // 通用数据存储区接口不会强加值类型,允许各种数据存储中间件实现（没有直接处理值）一起组合
		    //
		    // 最终，最低级别的数据存储区需要进行一些值检查或冒险获得不正确的值。暴露更多可能也是有用的应用程序的类型安全接口，并预先进行检查。
		    Put(key Key, value []byte) error
		
		
		    // Delete删除给定`key`的值
		    Delete(key Key) error
		}
	Write是数据存储区接口的写入端。
- 目录	

path|概述
---|---
[autobatch](https://godoc.org/github.com/ipfs/go-datastore/autobatch)	|Package autobatch 提供了一个 go-datastore 实现，它通过将 put 放在内存中直到满足某个阈值来自动批量写入。
[delayed](https://godoc.org/github.com/ipfs/go-datastore/delayed)|包延迟包装数据存储区，允许人为地延迟所有操作。
	[examples](https://godoc.org/github.com/ipfs/go-datastore/examples)|软件包 fs 是一个简单的数据存储区实现，它将 key 存储为目录和文件，镜像密钥。
[failstore](https://godoc.org/github.com/ipfs/go-datastore/failstore)|程序包故障库实现了一个数据存储区，它可以通过调用用户提供的错误函数来产生操作的自定义故障。
[keytransform](https://godoc.org/github.com/ipfs/go-datastore/keytransform)|包 keytransform 引入了一个 Datastore Shim，它在将密钥传递给子进程之前对其进行转换。
[mount](https://godoc.org/github.com/ipfs/go-datastore/mount)|软件包安装提供了一个数据存储区，其中包含以各种密钥前缀安装的其他数据存储区，并且是线程安全的
[namespace](https://godoc.org/github.com/ipfs/go-datastore/namespace)|Package 命名空间引入了一个名称空间 Datastore Shim，它基本上在前缀下安装整个子数据存储区。
[query](https://godoc.org/github.com/ipfs/go-datastore/query)|
[retrystore](https://godoc.org/github.com/ipfs/go-datastore/retrystore)|Package retrystore提供了一个允许重试操作的数据存储包装器。
[sync](https://godoc.org/github.com/ipfs/go-datastore/sync)|
[test](https://godoc.org/github.com/ipfs/go-datastore/test)|


## 参考
[package datastore](https://godoc.org/github.com/ipfs/go-datastore#pkg-subdirectories)			
	
				
			
														 														 			