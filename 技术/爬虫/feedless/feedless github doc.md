# feedless github doc
该项目用于创建 RSS 订阅源、过滤现有订阅源并对其进行转换（例如全文或媒体检测）的中间件。
## 自托管 feedless
最简单的 feedless 设置是使用 docker。
### 准备
- 准备环境标志

		mkdir feedless
		cd feedless
		echo 'POSTGRES_USER=postgres
		POSTGRES_PASSWORD=admin
		POSTGRES_DB=feedless
		APP_DATABASE_URL=jdbc:postgresql://postgres-db:5432/${POSTGRES_DB}
		' > feedless.env
- 创建网络

		docker network create -d bridge feedless-net
- 启动数据库

		docker run --env-file=feedless.env --network=feedless-net --hostname=postgres -d postgres:15

### 启动多合一镜像
- 最小设置没有无头 chrome 的镜像，因此您将无法渲染单页应用程序。

		docker run --env-file=feedless.env --network=feedless-net -it damoeb/feedless:aio
	或者，如果您需要 JavaScript 支持，请使用该 `feedless:aio-chromium` 镜像。该镜像包含用于无头渲染的 Chrome 浏览器。

		docker run --env-file=feedless.env --network=feedless-net damoeb/feedless:aio-chromium
- 检查日志。一体式镜像默认使用[单租户身份](https://github.com/damoeb/feedless/blob/master/docs/authentication.md)验证策略，可以使用环境标志进行更改。

	等到您看到 feedless 的横幅
	
		feedless-core_1   |               . .
		feedless-core_1   |  ,-           | |
		feedless-core_1   |  |  ,-. ,-. ,-| | ,-. ,-. ,-.
		feedless-core_1   |  |- |-' |-' | | | |-' `-. `-.
		feedless-core_1   |  |  `-' `-' `-' ' `-' `-' `-'
		feedless-core_1   | -'
		feedless-core_1   | 
		feedless-core_1   | feedless:core v0.1.0-e144ffe https://github.com/damoeb/feedless
- 在浏览器中打开 UI http://localhost:8080 并使用 `admin@localhost` 和`password` 登录

### 打造一体化形象
可以使用以下命令从头开始构建镜像

	shell
	gradle :buildDockerAio
	
## 身份验证选项
通过自定义标志，`APP_AUTHENTICATION` 您可以选择最适合您的身份验证策略。
### 单一租户
- 根登录

	`APP_AUTHENTICATION=root` 将启用单租户身份验证。`$APP_ROOT_EMAIL` 您可以使用和登录 UI `$APP_ROOT_SECRET_KEY`。
- 多租户
	- 通过电子邮件的 Magic Link（多租户）

		Magic Auth Links 是解耦程度最高的身份验证策略。

		`APP_AUTHENTICATION=mail` 将通过电子邮件使用魔术链接激活两阶段登录过程。为了发送电子邮件，您需要通过填写所有`APP_MAIL_*` 字段来配置 SMTP 服务器。

		此功能尚未完全实现。新用户需要某种注册（邀请链接或简单注册）。如果您认为这有用，请创建一个票证。
	- 单点登录（多租户）

		SSO 是产品部署中使用的策略 `feedless`，因为它是最无缝的。通过设置 `APP_AUTHENTICATION=sso` 身份验证是通过定义的 SSO 提供商运行的。如果 oauth 回复成功，`feedless` 将创建自己的令牌用于内部身份验证。

## 过滤表达式
feed 项目（也称为文章 articles）可以被过滤。feedless 提供一些基本运算符来构造复杂的表达式。要访问文章值，请使用 `#title`、`#url`、`#body`或`#any`

字符串已被 `"<STRING>"` 包裹。例如看看 `SimpleArticleFilterTest.kt`
### 例子
- 标题长度超过 4

		gt(len(#title), 4)
- 网址不以以下结尾 `/comments`

		not(endsWith(#url, "/comments"))
### 特性
- `#title`

	字符串，返回文章标题
- `#url`

	字符串，返回文章 url
- `#body`

	字符串，返回文章描述
- `#any`

	字符串，返回标题+url+描述串联的内容
- `#lc`

	Int、linkCount，如果存在，则返回标记中的不同 url 计数，否则返回纯文本中的 url 字符串

注意：某些属性可以从不同的来源重载 ( `#lc` )，一般来说，标记主体优先。我还没有决定如何处理插件输出的真正策略，但重载似乎是正确的方法，因为插件执行不是强制性的。在用例出现之前我会保持简单。

### Operators
- EndsWith

		endsWith(value: string, suffix: string): boolean
	检查是否以 `value` 给定 `suffix` 字符串结尾。

	- 例子

			endsWith(#url, "#comments")
- StartsWith

		startsWith(value: string, prefix: string): boolean
	检查是否以 `value` 给定 `prefix` 字符串开头。

	- 例子

			startsWith(#url, "https")
- 包含

		contains(haystack: string, needle: string): boolean
	检查 needle 字符串中是否包含 haystack 字符串。

	- 例子

			contains("Whatever I say", "ever")
- 大于（等于）
	
		gt(left: number, right: number): boolean
		gte(left: number, right: number): boolean
- 小于（等于）

		lt(left: number, right: number): boolean
		lte(left: number, right: number): boolean
- eq

		eq(left: number, right: number): boolean
- or
	
		or(left: boolean, right: boolean): boolean
- and

		and(left: boolean, right: boolean): boolean
- not

		not(value: boolean): boolean

## 插件
feedless 带有 feed 项目级别的插件机制。插件用于全文提取和媒体检测等。示例可以在 `packages/server-core/src/main/kotlin/org/migor/feedless/trigger/plugins` 以下位置找到
## 开发
待
## 从第三方框架迁移		
待
## 监控
如果您自行托管，则可以使用 loki、prometheus 和 grafana 监控此中间件。
## 网络限制
请求在 IP 级别上受到限制。主机也受到保护，免受洪水侵袭。
## Web 2 Feed
Web to Feed 算法将评估 DOM 结构。对于 SPA，您需要激活预渲染。
## Web 2 Fragment feed
从页面片段创建提要并渲染像素、标记或文本。

## 许可证
- [https://joinup.ec.europa.eu/collection/eupl/how-use-eupl](https://joinup.ec.europa.eu/collection/eupl/how-use-eupl)
- [ https://www.youtube.com/watch?v=9kGrKBOytYM]( https://www.youtube.com/watch?v=9kGrKBOytYM)
- [https://techhq.com/podcast-series/postgres-postgresql-services-contract-support-podcast-s01e04/](https://techhq.com/podcast-series/postgres-postgresql-services-contract-support-podcast-s01e04/)


## 参考
[https://github.com/damoeb/feedless/tree/master/docs](https://github.com/damoeb/feedless/tree/master/docs)	
	