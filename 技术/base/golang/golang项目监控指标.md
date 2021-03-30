# golang 项目监控指标
## 为什么要监控 golang 内核指标
- cpu 消耗异常
- 内存泄露溢出到 oom (golang 自带 gc 无效)

## 使用工具
- prometheus 的 client_golang
	-  prmhttp.Handler
- pprof 分析方法
	- 获取堆数据
	
			curl https://services/domain-service/debug/pprof/heap > heap.out
	- 分析
			
			go tool pprof heap.out
		- 模式
	
				go tool pprof -<mode> heap.out
			- 模式
				- inuse_space
			
					表示正在分配但尚未释放的内存量
				- inuse_objects
			
					表示正在分配但尚未释放的对象数量
				- alloc_space
			
					表示已分配的内存量无论是否已释放
				- alloc_objects
					
					平均值，表示分配的对象数量，无论对象是否已释放。
		- top
	
			可以使用 top 查看内存消耗最多的
			
			- 参数说明
				- flat
	
					表示由功能分配并仍由该功能保留的内存。
				- cum
				
					表示由函数或在堆栈中调用的任何其他函数分配的内存
	- 可能不在核心层，在网络层，查看 goroutine
	
			curl https://services/domain-service/debug/pprof/goroutine > goroutine.out
			go tool pprof goroutine.out
	- 使用 `netstat ` 查看链接数是否激增
- 其他

##  golang 性能指标
### 需求
- 查看该 go 进程 cpu 占用情况,给开发优化进程 cpu 参考，给管理员提供运行程序健康状态评估
- 查看该 go 进程内存占用情况,给开发优化进程内存参考，给管理员提供运行程序健康状态评估
- 查看该 go 进程 go 特性功能使用情况，如 GC、线程、goroutine,给开发优化进程参考，给管理员提供运行程序健康状态评估

### 具体值参考 prometheus 的 client_golang
- go 指标
	- `go_goroutines`(gauge)- goroutines 当前数量
	- `go_threads`(gauge)-os 线程数
- gc 指标
	- `go_gc_duration_seconds`(summary)-垃圾回收周期摘要
	- `go_memstats_gc_cpu_fraction`(gauge)-程序启动后，GC 和程序使用可用 CPU 的时间的比
	- `go_memstats_gc_sys_bytes`(gauga)- GC 系统元数据的字节数
	- `go_memstats_last_gc_time_seconds` (gauga)- 最后一次 gc 的时间，单位秒
	- `go_memstats_next_gc_bytes` (gauga)- 下一次 gc 回收的字节数
- 内存指标
	- 系统进程 
		- `process_resident_memory_bytes`(gauge)- 进程物理内存
		- `process_virtual_memory_bytes`(gauge)- 进程虚拟内存
		- `process_virtual_memory_max_bytes`(gauge)- 进程最大虚拟内存
		- `process_max_fds`(gauge) - 最大文件打开数
		- `process_open_fds`(gauge)- 文件打开数 
	- go 内存指标  
		- `go_memstats_sys_bytes`(gauge) - go 从系统获取的字节数
		- `go_memstats_alloc_bytes`(gauge)-分配并使用的字节数
		- `go_memstats_alloc_bytes_total`(count)-已分配的字节总数，即使已释放也是如此。
		- `go_memstats_other_sys_bytes`(gauge)-用于其他系统分配的字节数
	- heap 相关 
		- `go_memstats_heap_alloc_bytes`(gauge)-heap 已分配且仍在使用的字节数
		- `go_memstats_heap_sys_bytes`(gauge)- heap从 sys 获取的字节数
		- `go_memstats_heap_idle_bytes`(gauge)-heap 等待使用的字节数
		- `go_memstats_heap_inuse_bytes`(gauge)-heap正在使用的字节数
		- `go_memstats_heap_released_bytes`(gauge)-heap 释放到操作系统的字节数
		- `go_memstats_heap_objects`(gauge)-所有分配对象数量
	- stack 相关
		- `go_memstats_stack_inuse_bytes` (gauge)-stack 已分配且仍在使用的字节数
		- `go_memstats_stack_sys_bytes`(gauge)- stack 从 sys 获取的字节数
	- mallocs 相关
		- `go_memstats_mallocs_total`(counter)- malocs 总数
	- mcache 相关
		- `go_memstats_mcache_inuse_bytes`(gauge)-mcache 结构体使用的字节数
		- `go_memstats_mcache_sys_bytes`(gauge)- sys 用于mcache 结构体使用的字节数
	- mspan	 	
		- `go_memstats_mspan_inuse_bytes`(gauge)-mspan 结构体使用的字节数
		- `go_memstats_mspan_sys_bytes`(gauge)- sys 用于 mspan 结构体使用的字节数

### grafana 模版
需要调整

- [模版地址](https://grafana.com/grafana/dashboards/10826)
	
## GIN-http 监控
### 需求
- 管理员想要知道该服务器的 QPS，用来推算服务器时段压力，过大可能导致服务异常，该指标根据每个 api 端口独立统计
- 管理员想要知道最大响应时间和平均响应时间，看服务器处理是否出现瓶颈或者程序出现问题，该指标根据每个 api 端口独立统计
- 管理员想要知道 api 请求接口响应状态，200，404，500 等，判断服务是否正常，该指标根据每个 api 端口独立统计
- 管理员想要知道 50%，90% 分位数请求耗时

#### prometheus 自带指标参考
| Name                                  | Type      | Description                         |
---|---|---|---
| `service_http_request_count_total`      | Counter   | http 的请求总数 |
| `service_http_request_duration_seconds` | Histogram | http 的请求延迟(单位为秒)  |
| `service_http_request_size_bytes`       | Summary   | http 请求大小(单位字节)       |
| `service_http_response_size_bytes`      | Summary   | http 响应大小(单位字节)        |

### 容量规划参考
按二八定律来看，如果每天 80% 的访问集中在 20% 的时间里，这 20% 时间就叫做峰值时间。

- 公式

		( 总PV数 * 80% ) / ( 每天秒数 * 20% ) = 峰值时间每秒请求数(QPS)
- 机器

		峰值时间每秒QPS / 单台机器的QPS = 需要的机器

- 公式套用
	- 问题1，每天 300w PV 的在单台机器上，这台机器需要多少 QPS？ 
	
			( 3000000 * 0.8 ) / (86400 * 0.2 ) = 139 (QPS)
	- 问题2，如果一台机器的QPS是58，需要几台机器来支持？ 
	
			139 / 58 = 3
		
## 需要基础知识
### 内存
- golang 分配内存方法分为
	- 堆(Heap)
	
		heap 是编程语言用来存储全局变量的内存。默认情况下，所有全局变量都存储在堆内存空间中。它支持动态内存分配,系统不会自动管理 heap，CPU也不会对其进行严格管理。它更像是一个自由浮动的内存区域。
		
		heap 出问题不会自动解除和回收，为了防止堆不受约束，需使手动回收或者编程语言回收(GC)删除不必要的引用对象。
		
		- 优点
			- heap 可以帮助您找到最大和最小数量
			- 垃圾回收在 heap 内存上运行以释放对象使用的内存
			- 优先级队列中也使用了 heap 方法
			- 允许访问全局变量
			- heap 对内存大小没有任何限制
		- 缺点
			- 它可以提供操作系统可以提供的最大内存
			- 计算需要更多时间
			- 内存管理在 heap 内存中更为复杂，因为它是全局使用
			- 与 stack 相比，执行需要花费太多时间
		- 使用场景

			当需要分配较大的内存块时，应使用 Heap。例如，要创建一个大型数组或大型结构，以使该变量保持较长时间，然后应将其分配到 heap 上。
	- 堆栈/栈(stack)
	
		堆栈是计算机内存的一个特殊区域，用于存储由函数创建的临时变量。在堆栈中，在运行时声明，存储和初始化变量。它是一个临时存储内存。计算任务完成后，变量的内存将被自动擦除。堆栈包含主要方法中局部变量和引用变量。它是动态分配，内存回收和所属任务功能生命周期一致。
		
		- 优点
			- “后进先出”（LIFO）方法管理数据，这是“链接列表”和“数组”无法实现的
			- 调用函数时，局部变量存储在 stack 中，一旦返回，它将自动销毁
			- 当未在该函数外部使用变量时，将使用 stack
			- 它使开发者可以控制内存的分配和释放方式
			- stack 会自动清理对象
			- 不容易损坏
			- 变量无法调整大小
		- 缺点
			- stack 内存非常有限
			- 在 stack 上创建太多对象会增加 stack 溢出的风险。
			- 随机访问是不可能的。
			- 变量存储将被覆盖，有时会导致函数或程序的不确定行为。
			- stack 将落在内存区域之外，这可能导致异常终止。 
		- 使用场景

			使用相对较小的变量，则仅在使用它们的函数处于活动状态之前才需要使用 stack，它更快，更容易 
	- 区别	
		
		参数|stack|heap
		---|---|---
		数据结构类型|stack 是线性数据结构|heap 是分层数据结构
		存取速度|高速存取|比stack慢
		空间管理|操作系统可有效管理空间，因此内存永远不会碎片化。|heap 空间使用效率不高。内存可能会变得碎片化，因为首先分配然后释放的内存块。
		访问|仅局部变量|它允许全局访问变量
		空间大小的限制|stack 大小限制取决于操作系统|对内存大小没有具体限制
		调整大小|变量无法调整大小|变量可以调整大小
		内存分配|内存在连续的块中分配|内存以任何随机顺序分配
		分配和解除分配|由编译器指令自动完成|由程序员或程序手动完成
		解除分配|不需要取消分配变量|需要显式取消分配
		成本|少|多
		实施|可以使用基于内存的动态数组、基于链表的简单数组三种方式实现 stack|可以使用数组和树来实现 heap
		主要问题|内存不足|内存碎片
		参考地点|自动编译时间指令|充足
		大小灵活性|固定尺寸|可以调整大小
		访问时间|快|慢
- 操作系统内存
	- VSZ 
	
		全称为 Virtual Memory Size，表示虚拟内存大小，没有占用物理内存，只是划分了地址空间，并没有建立与物理地址的映射, GC 调节后，这个不变
	- RSS 

		全称为 Resident Set Size，物理内存大小, GC 调节后，这个占用会提升到调节内存大小

### go 内存问题	
- 内存压力来自
	- 分配过多，数据表示不正确
	- 大量使用反射或弦
	- 使用全局变量
	- 孤立无尽的 goroutines
- 内存泄露来自
	- 创建子字符串和子切片
	- 错误使用 defer 语句
	- 未封闭的 HTTP 响应主体（或通常未封闭的资源）
	- 孤立的挂例行程序
	- [全局变量](https://medium.com/dm03514-tech-blog/sre-debugging-simple-memory-leaks-in-go-e0a9e6d63d4d)

### go cpu 问题
#### GC 优化 CPU 使用
延长默认 GC 来提升 CPU 效率，提升 API 响应速度 (优化 GC CPU 消耗的一种方法)

- GC 的阶段

	GC 分两阶段
	
	- 标记阶段
	
		运行时将遍历一个你哟功能程序在堆上引用的所有对象，并将他们标记为使用中。这组对象为实时内存(live memroy),然后堆上所有未标记的其他内容将被视为垃圾
		
		清除过程非常快，所以扫描 GC 的时间由标记组件决定
		
		标记涉及遍历程序当前指向的所有对象，因此时间与活动内存量成正比和堆总大小无关，多余的垃圾不会增加标记时间。
		
		较低的 GC 频率意味着较少的标记，意味着 cpu 使用量降低而带来更多的内存消耗。
	- 清除阶段
	
		清除程序清除未标记的垃圾数据，进行重新分配
	
	- 堆大小
	
		包含所有分配的资源，包含标记和未标记的
	- 实时内存
	
		正在运行的程序当前正在引用的所有分配，未分配没算
- 解决 GC 频率和内存占用的折中点(Pacer)

	Go 的默认 Pacer(步长器)将在每次堆大小加倍时尝试触发GC周期，通过在当前GC周期的标记终止阶段设置下一个堆触发大小来实现。因此在标记所有活动内存之后，当总堆大小是当前活动集的2倍时，它可以决定触发下一个GC。所以2x 值来自 `GOGC` 运行时用于设置触发比率的变量。这里衡量是按照程序占用系统总内存百分比，如果小于1%可以进行优化。
	- 引入 ballast
	
		初始化 ballast 设置一个该程序合理的值，这样在这个值的2倍之前将不触发 GC，用来减少 GC 周期，减少 CPU 消耗。通过降低 GC ，可以减少 CPU 消耗 30%
	- 不使用 GOGC 调整的原因
		- 内存量更精确，比率并无意义
		- 这个值设置很高可以达到相同效果，但是会容易受到堆大小的影响
		- 实时内存推理不简单，但是总内存很简单	 	
	- 降低 API 延迟(GC assists)
	
		GC assists 将 GC 周期内的内存分配负担放在负责分配 goroutine 上。如果没有这个机制，运行时将无法防止堆GC 周期内变的不受约束。当执行此代码时，通过一系列符号转换和类型检查，`goroutine` 将调用 `runtime.makeslice`，最终该调用最终将调用 `runtime.mallocgc` 来为数据分配一些内存，`runtime.mallocgc` 中 `if assistG.gcAssistBytes < 0` 代码检查 `goroutine` 是否处于分配债务中。
		
		假设 `goroutine` 在当前 GC 周期内第一次分配，它将被迫进行 GC assists( `gcAssistAlloc`) ,该功能负责一些内务处理，并最终调用 `gcAssistAlloc` 1 以执行实际的 GC 协助工作.工作步骤
		
		- 检查 goroutine 是否正在执行不可抢占的操作（即系统 goroutine）
		- 执行 GC 标记工作
		- 检查 goroutine 是否仍然有分配债务，否则返回
		- 转到2
		
		随着每个服务器负载的增加，内存分配速率将增加，这反过来又会增加GC周期的速率（通常为10或20/s/每周期），而更多的 GC 周期意味着为 API 服务的 goroutine 会进行更多的 GC 辅助工作，反过来会有更多的 API 延迟。
	
		- 债务说明
		
			它表示此 `goroutine` 在 GC 周期内分配的资源超过了 GC 的工作量，`goroutine` 必须在 GC 周期内为分配该税收，除非该税收必须在实际发生分配之前预先支付。税收与`goroutine` 尝试分配的金额成正比。这提供了一定程度的公平性，以便分配大量的 goroutine 将为这些分配付出代价。
	
- 优化思路
	- 注意 golang 应用程序在做很多 GC 工作
	- 部署了内存镇流器
	- 通过允许堆变大，它减少了GC周期
	- 由于Go GC在协助下减少了工作，API延迟得以改善
	- 镇流器分配大部分是免费的，因为它驻留在虚拟内存中
	- 镇流器比配置 GOGC 值更容易推理
	- 从小的镇流器开始，然后通过测试增加
- GC 优化需要观察的指标
	- 内存
		- GC 周期(opm)
			- 压力高峰期，GC 周期变化
			- GC 周期内 cpu 占用的情况
		- GC 触发回收效率(rps 每秒回收个数 )
		- Heap 大小(Heap-use-MIB)
	- cpu
		- GC 周期内 cpu 占用的情况
	
	
## 优化遇到问题
很多时候发现 pprof 消耗几兆，而系统或者 docker 消耗几百兆，很可能是函数造成了很多内存压力，并且Go垃圾回收器或其运行时正在消耗内存。优化功能以流式传输数据，而不是将结构保留在内存中。可以尝试使用 `runtime.GC` 和 `FreeOSMemory` 从中调用手动触发垃圾回收 `runtime/debug`，但是可能无效。

因为 go gc 后，可能会保留一段时间，预计是5分钟一个循环，这样防止再向 os 申请的消耗。再 1.12 版本后，除非有其他服务抢占内存，否则 golang 的 rss 将不会释放。可以使用 `GODEBUG=madvdontneed=1` 参数强制修改回收模式	
## 参考
- [How we tracked down (what seemed like) a memory leak in one of our Go microservices](https://blog.detectify.com/2019/09/05/how-we-tracked-down-a-memory-leak-in-one-of-our-go-microservices/)
- [Go服务监控](https://www.cnblogs.com/52fhy/p/11828448.html)
- [Go memory ballast: How I learnt to stop worrying and love the heap](https://blog.twitch.tv/en/2019/04/10/go-memory-ballast-how-i-learnt-to-stop-worrying-and-love-the-heap-26c2462549a2/)
- [go trace](https://golang.org/pkg/runtime/trace/)
- [golang 监控3+1](https://www.cnblogs.com/52fhy/p/11828448.html)
- [使用 Prometheus 监控服务器性能](https://cjting.me/2017/03/12/use-prometheus-to-monitor-server/)
- [玩转PROMETHEUS(1)–第1个例子](http://vearne.cc/archives/11085)
- [Stack vs Heap: Know the Difference](https://www.guru99.com/stack-vs-heap.html)
- [Golang 使用 Prometheus 监控 Gin 服务性能](https://studygolang.com/articles/22902)