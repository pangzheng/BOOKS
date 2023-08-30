# feedless 
feedless 是一个实验性的 feed 中间件，用于创建大多数 HTML 或feed的 RSS/ATOM/Json feed 并对其进行操作。其目标是保持网络开放和可访问并创建可共享的数据流。

您可以[自行托管](https://github.com/damoeb/feedless/blob/master/docs/self-hosting.md)或使用 [feedless.org](https://feedless.org/) 来创建和共享feed。对于简单的网络到feed用例，[RSS 代理](https://github.com/damoeb/rss-proxy)可能就足够了。

[feedless 视频](https://www.youtube.com/watch?v=PolMYwBVmzc)
## 特征
- 通过 Full(-text) 丰富内容
- 使用 [yt-dlp](https://github.com/yt-dlp) 进行媒体检测
- [Web-to-Feed](https://github.com/damoeb/feedless/blob/master/docs/web-to-feed.md)
- [Web-to-Fragment-Feed](https://github.com/damoeb/feedless/blob/master/docs/web-to-fragment-feed.md)
- 将多个 feed 聚合到 Bucket 中
- [过滤器](https://github.com/damoeb/feedless/blob/master/docs/filters.md)
- [JavaScript 支持](https://github.com/damoeb/feedless/blob/master/packages/agent/README.md) 基于 JavaScript 的网站
- 用于存档/隐私目的的内联图像
- 可使用[插件](https://github.com/damoeb/feedless/blob/master/docs/plugins.md)扩展
- 简单的[自托管](https://github.com/damoeb/feedless/blob/master/docs/self-hosting.md)
- [第三方迁移](https://github.com/damoeb/feedless/blob/master/docs/third-party-migration.md)

## 客户端模块
- 用于管理 feed 的[应用程序](https://github.com/damoeb/feedless/blob/master/packages/app-web/README.md) Angular UI ([Angular](https://github.com/damoeb/feedless/blob/master/angular.io) )
- [cli](https://github.com/damoeb/feedless/blob/master/packages/app-cli/README.md) CLI 查询文章（[节点](https://nodejs.org/)）

## 服务器模块
- [核心](https://github.com/damoeb/feedless/blob/master/packages/server-core/README.md) 无状态后端（[Spring Boot](https://spring.io/projects/spring-boot/)）
- [agent](https://github.com/damoeb/feedless/blob/master/packages/agent/README.md) Puppeteer 包装器（[nestjs](https://nestjs.com/)）

## 入门
请参阅 [自托管](https://github.com/damoeb/feedless/blob/master/docs/self-hosting.md) 或 [开发](https://github.com/damoeb/feedless/blob/master/docs/development.md)
## 变更日志
查看[变更日志](https://github.com/damoeb/feedless/blob/master/changelog.md)
## Contact
feedlessapp/at/proton/dot/me
## 相关项目
- [feedless](https://www.feedirss.com/)
- [nitter](https://github.com/zedeus/nitter)
- [invidious](https://github.com/iv-org/invidious)
- [siftrss](https://siftrss.com/)
- [xtractor](https://github.com/mohaps/xtractor)

## 链接
- RFC 4287 - [Atom 联合格式](https://github.com/damoeb/feedless/blob/master/docs/rfcs/RFC%204287%20The%20Atom%20Syndication%20Format.html)
- RFC 5005 - [Feed 分页和拱形](https://github.com/damoeb/feedless/blob/master/docs/rfcs/RFC%205005%20Feed%20Paging%20and%20Archiving.html)
- RFC 3275 - [XML 签名语法和处理](https://github.com/damoeb/feedless/blob/master/docs/rfcs/RFC%203275_%20(Extensible%20Markup%20Language)%20XML-Signature%20Syntax%20and%20Processing.html)
- [回拨协议](https://github.com/damoeb/feedless/blob/master/docs/rfcs/Pingback%201.0.html)

## 执照
- [EUPL-1.2](https://opensource.org/licenses/EUPL-1.2)

## 参考
[https://github.com/damoeb/feedless](https://github.com/damoeb/feedless)