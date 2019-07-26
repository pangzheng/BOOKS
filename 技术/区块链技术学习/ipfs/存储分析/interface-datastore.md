# interface-datastore
## 实现
### 后端实现
- Memory: [src/memory](https://github.com/ipfs/interface-datastore/blob/master/src/memory.js)
- level: [datastore-level](https://github.com/ipfs/js-datastore-level) (支持任何levelup 兼容的后端)
- File System: [datstore-fs](https://github.com/ipfs/js-datastore-fs)

### 包装实现
- Mount: [datastore-core/src/mount](https://github.com/ipfs/js-datastore-core/tree/master/src/mount.js)
- Keytransform: [datstore-core/src/keytransform](https://github.com/ipfs/js-datastore-core/tree/master/src/keytransform.js)
- Sharding: [datastore-core/src/sharding](https://github.com/ipfs/js-datastore-core/tree/master/src/sharding.js)
- Tiered: [datstore-core/src/tiered](https://github.com/ipfs/js-datastore-core/blob/master/src/tiered.js)
- Namespace: [datastore-core/src/namespace](https://github.com/ipfs/js-datastore-core/tree/master/src/namespace.js)

如果您想要与 [go-ds-flatfs](https://github.com/ipfs/go-ds-flatfs) 相同的功能，请使用带有fs的分片 

	const FsStore = require('datastore-fs')
	const ShardingStore = require('datastore-core').ShardingDatatstore
	const NextToLast = require('datastore-core').shard.NextToLast
	
	const fs = new FsStore('path/to/store')
	
	// flatfs 就像 go-flatfs
	const flatfs = await ShardingStore.createOrOpen(fs, new NextToLast(2))
## Install
	$ npm install interface-datastore	
## Usage
### 包装存储
	const MemoryStore = require('interface-datastore').MemoryDatastore
	const MountStore = require('datastore-core').MountDatastore
	const Key = require('interface-datastore').Key
	
	const store = new MountStore({ prefix: new Key('/a'), datastore: new MemoryStore() })
### 测试套件
在 [src/tests.js](https://github.com/ipfs/interface-datastore/blob/master/src/tests.js)

	describe('mystore', () => {
	  require('interface-datastore/src/tests')({
	    async setup () {
	      return instanceOfMyStore
	    },
	    async teardown () {
	      // cleanup resources
	    }
	  })
	})
## API
### key
为了更好地抽象如何寻址值，有一个 Key 类用作标识符。从a Buffer 或 string 很容易的创建 key

	const a = new Key('a')
	const b = new Key(new Buffer('hello'))
关键方案的灵感来自文件系统和 Google App Engine key 模型。key 在系统中是唯一的。它们通常是分层的，包含越来越多的特定命名空间。因此，key 可以被视为其他key 的“children”或“ancestors”：	

	new Key('/Comedy')
	new Key('/Comedy/MontyPython')
此外，可以对每个命名空间进行参数化以嵌入相关的对象信息。例如，Key name（最具体的命名空间）可以包含对象类型

	new Key('/Comedy/MontyPython/Actor:JohnCleese')
	new Key('/Comedy/MontyPython/Sketch:CheeseShop')
	new Key('/Comedy/MontyPython/Sketch:CheeseShop/Character:Mousebender')
### 方法
确切的类型在 [src/index.js](https://github.com/ipfs/interface-datastore/blob/master/src/index.js)

这些方法将出现在每个数据存储区中。Keyalways 始终表示上述 Key 类型的实例。每个数据存储区都是 Value 类型的通用数据库，但目前所有后备实现仅针对此实现 [Buffer](https://nodejs.org/docs/latest/api/buffer.html)。

#### has(key) -> Promise<Boolean>
	key: Key
检查给定 key 是否存在

	const exists = await store.has(new Key('awesome'))
	console.log('is it there', exists)
#### put(key, value) -> Promise
	key: Key
	value: Value	
使用给定 key 存储值

	await store.put(new Key('awesome'), new Buffer('datastores'))
	console.log('put content')
#### get(key) -> Promise<Value>
	key: Key
检索存储在给定 key 下的值。

	const value = await store.get(new Key('awesome'))
	console.log('got content: %s', value.toString())
	// => 得到内容：数据存储
#### delete(key) -> Promise
	key: Key
删除存储在给定key下的内容。

	await store.delete(new Key('awesome'))
	console.log('deleted awesome content :(')
#### query(query) -> Iterable
	query: Query
在存储中搜索某些值。返回一个 [Iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)，每个项目为 a Value

	// 检索来自存储__all__值
	let list = []
	for await (const value of store.query({})) {
	  list.push(value)
	}
	console.log('ALL THE VALUES', list)
- Query

	具有以下可选属性的表单中的对象
	
	- prefix: string （可选） - 仅返回键以此前缀开头的值
	- filters: Array<Filter<Value>> （可选） - 根据这些功能过滤结果
	- orders: Array<Order<Value>> （可选） - 根据这些功能对结果进行排序
	- limit: Number （可选） - 只返回这么多记录
	- offset: Number （可选） - 在开头跳过这么多记录
	- keysOnly: Boolean （可选） - 仅返回键，无值。

#### batch()
这将返回一个对象，可以使用该对象将多个操作链接在一起，它们仅在调用时执行 commit

	const b = store.batch()
	
	for (let i = 0; i < 100; i++) {
	  b.put(new Key(`hello${i}`), new Buffer(`hello world ${i}`))
	}
	
	await b.commit()
	console.log('put 100 values')
- put(key, value)

		key: Key
		value: Value
	将 put 操作排队到商店。
- delete(key)

		key: Key
	将删除操作排队到商店。

### commit() - > Promise
将所有排队操作写入底层存储。调用它后不应使用批处理对象。
#### open() - > Promise
打开数据存储区，只有在商店之前关闭时才需要这样做，否则由构造函数处理。
#### close() - > Promise
关闭数据存储区，应始终调用此方法以确保清理资源。

 		 					