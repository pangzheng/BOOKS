# 对于 feedless 想法
## 来源
- 电报群
- 使用 killthenewsletter 发送电子邮件
- 搜索引擎 bing.com/search?format=rss&q=khayrirrw

我们倾听那些无话可说的人。话多的人其实无话可说。聆听这寂静，它有很多话要告诉你。

## 社会参与
- 作为生产者，我想监视其他用户的行为
- 作为消费者，我想通过分享/喜欢/评论文章来与其他用户互动 
- 作为消费者，我想阅读其他人对我感兴趣的文章的活动
- 作为消费者，我不想收到垃圾邮件 
- 作为消费者，我希望看到我不认识的人的评论

它应该能够通过以下方式与其他用户互动——也许与 Twitter 等现有方法保持一致：

- 评论
- 点赞、分享/转发或存档

## 用例
- 集成到 zettelkasten，建议 zettelkasten
- 关注我的个人资料
- 通过 github 提供插件支持
	- 解析 html url ->
- 社交东西如何运作？
- 您的私人 feed 的收件箱/队列
- 探索：使用 webhook 进行内容解析检查 [https://github.com/converspace/webmention/blob/master/README.md](https://github.com/converspace/webmention/blob/master/README.md)
- 评论
- 顶部的搜索字段可搜索所有内容
- 如何向研究人员推销：
	- 处理纸 x -> 定义存储桶
	- 创建笔记

		Alegorie Wolle -> Spinnen -> Weben -> Stricken Alegorie Wiese: Fremde Samen fliegen ein
- 编辑器 [https://github.com/sparksuite/simplemde-markdown-editor](https://github.com/sparksuite/simplemde-markdown-editor)
- kotlin：添加端点以建议文章
- 评分文章
- 同步到 github 
- 归档文章a：
	- a 必须链接到用户文章
	- a 在用户文章中引用
- 私人直播
	- 通知：其他用户的通知
	- 存档：你的网络
	- 收件箱：与存档文章相关的建议文章

## 存档功能
如果要归档一篇好的文章，不应该简单地将其简单放入一个列表中。根据卡片盒原理，每篇文章都应该至少与另一篇文章进行链接，以建立上下文。在动态信息中，每篇文章都应该有链接，与内容无关。链接：文章 -> 文章
## 完层功能
### 记录事件(feed)
- 创建桶
- 将 feed 添加到桶中
	- 从多个潜在的 feed（native、rss-proxy、nitter）中选择一个 feed
	- 查看 feed 条目
- 验证 feed 是否仍在工作
- 导入/导出 opml
- 使用 archive.org 的首次抓取日期恢复创建日期或者每隔一周进行校正
- 可读性 UI

### 消费者事件（feed）
- 触发事件时
- 采集 feed
	- 认证
- 对于每篇文章
	- 采集篇
		- 认证
			- 原生
				- 基本身份验证 -> 插件
		- 预渲染
			- 渲染后操作 -> 插件
	- map
		- 原生
			- 可读性
			- 音频/视频流
			- 主图
			- 得分 -> 插件
	- 保留政策（iff 私有）
	- reduce（当且仅当是私有的）
		- 过滤器

### 生产事件（桶）
- 触发事件时
- 过滤器（分配文章段）
	- 筛选分段中的文章
- map
	- 原生 
		- 添加存储桶标签 -> 插件
- reduce
	- 原生 
		- 限流器 
		- 聚合->插件
- export
	-feed

### Feed 解析器
- 原生
- web2feed
- 插件

#### 消费者
- feed
	- 能见度
	- 触发器
		- 来源变更
		- Post
		- Mq 事件
		- 调度器
	- 采集前行动
		- 验证
- 日志
- 采集后行动
	- 可读性
	- 消费者标签
	- 内容细化
		- 内容提取
			- 多媒体
		- 内容质量评分制作者
- 触发器
	- 消费者的变化
	- 调度器
- 段分配
	- 按内容
	- 按时间，例如自上次更改以来
- 输出
	- map
		- 制作人标签
	- reduce
		- 限流器
		- 聚合
	- 格式
		- 推
		- feed
			- 验证
			- expose
		- webhook 

- agent
	- 灵感：ISBN 的后续
	- 社交网络：关注您的个人资料并避免无边泳池
	- 数据创建：现有数据的新子集，对其进行细化
	- 数据细化：视频/照片的可读性提取、stt、ocr
	- 搜索：评级文章档案
	- 提醒：咖啡价格最低的时间是什么时候?
	- 提醒：路由器致命错误/具有可升级固件 [https://wiki.mikrotik.com/wiki/Manual:Upgrading_RouterOS](https://wiki.mikrotik.com/wiki/Manual:Upgrading_RouterOS)
	- 提醒：硬件制造商发布新固件
	- 存档：soundcloud 喜欢（+摘要），youtube 喜欢备份
	- 灵感：接收链接 -> 转发到存储桶

来自 [https://www.reddit.com/r/rss/comments/ppm9hh/looking_for_an_rssapp_alternative/](https://www.reddit.com/r/rss/comments/ppm9hh/looking_for_an_rssapp_alternative/)

- 抓取文章的主要图像
- 标记白名单和黑名单。
- 您可以只关注一个页面，即使它看起来不像一个集合。理论上它是随着时间的推移不同版本的集合。文章中应该有一个字段来存储映射内容
- 比如可读性。还应该指定该字段的 mimeType。
- 从 twitter、hn、yt 导入订阅
- 浏览用户体验
	- 读者 [https://www.srf.ch/audio/sounds/nell-the-flaming-lips-where-the-viaduct-looms-ein-maerchen?id=12099281](https://www.srf.ch/audio/sounds/nell-the-flaming-lips-where-the-viaduct-looms-ein-maerchen?id=12099281)
	- 像苹果播客一样提供 feed ui [https://podcasts.apple.com/us/podcast/stuff-you-should-know/id278981407](https://podcasts.apple.com/us/podcast/stuff-you-should-know/id278981407) 或 [https://philpeople.org/profiles/dominique-kuenzle](https://philpeople.org/profiles/dominique-kuenzle)
- 自定义 feed 字段在获取时选择加入/选择退出
- yt 支持
- 查询
- 保留
- feed 生成前和后流
- 带过滤器和搜索的列表
- 阅读器模式
- 验证
- 混音、引用
- 内容创建：文章是评论的代理

抽象层次

- 手动积累、手动打分：浏览大量文档并决定是否与您相关
- 请求你想要的：自动评分、自动累积：搜索引擎
- 得到你想要的：自动查询、自动评分、自动累积

部署盒

- 带 USB 存储棒的 rasp-pie
- 应用程序通过超级节点连接到这个盒子，所以你有一个本地备份
- 您可以共享一个秘密，以便人们可以连接到您的盒子（dyndns）
- 应该有几种类型的秘密
	- 能见度等级
	- 共享数据级别
	- 所有者级别

### 下一步
1. docker+存储挂载
- 使用规范 [https://datatracker.ietf.org/doc/html/rfc4287#section-6](https://datatracker.ietf.org/doc/html/rfc4287#section-6) 扩展具有自定义属性的原子 feed
- 下一个/上一个/最后一个/第一个
- pingback -> activitypub
- feed 应该签名
- feed Ids 可能被欺骗
- 登陆页面：输入 URL 站点或创建原始 Feed
- 显示站点元（区分列表与文章）
- 显示最佳 feed（原生 > 合并原生 > 生成）。根据覆盖区域+订阅按钮对生成的 feed 进行评分。猜测列并允许过滤。

### 签名 feed [https://www.w3.org/TR/2002/REC-xmldsig-core-20020212/#sec-o-Simple](https://www.w3.org/TR/2002/REC-xmldsig-core-20020212/#sec-o-Simple)

- 测试 [https://foojay.io/today/build-and-test-non-blocking-web-applications-with-spring-webflux-kotlin-and-coroutines/?__cf_chl_management_tk__=v1LiG.cGiekCjlOKqGQghLwDmgPBsfLu20wOQ5go8bw-1641886915-0-gaNycGzNCFE
]()
- 支持某种订阅（websub），但允许 slave 位于 nat 后面
- 文本文件，例如日志
- 根 note
- Article 是站点，aritcleRef 是注释，可以引用站点或代表其自身

### slave 节点使用 ws 连接到主节点
- master 存储 Slave 使用的非私有 feed
- slave 的用作路由器来请求网站 ()

### 博客体验
- 像 WordPress 一样渲染 feed
	- 列表 [https://www.srf.ch/audio/sounds](https://www.srf.ch/audio/sounds)
- 使用唯一 ID 呈现自动发现标签来识别用户？
- 视频采集应该是异步的
- 每篇文章都有一个相关的文章 feed
- 用于播客的stt [https://www.ubuntupit.com/best-open-source-speech-recognition-tools-for-linux/
](https://www.ubuntupit.com/best-open-source-speech-recognition-tools-for-linux/)
- [https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-create-an-atom-feed-for-a-private-gallery?view=vs-2022](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-create-an-atom-feed-for-a-private-gallery?view=vs-2022)
- [https://web.archive.org/web/20120307235526/http://www.kbcafe.com/rss/atom.xsd.xml](https://web.archive.org/web/20120307235526/http://www.kbcafe.com/rss/atom.xsd.xml)