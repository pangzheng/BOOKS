# Rss-proxy
RSS-proxy 允许您创建几乎任何网站的 RSS 或 ATOM 提要，只需分析静态 HTML 结构即可。

RSS-proxy 允许您仅通过分析 HTML 结构来创建任何静态网站或提要（web2feed）的 ATOM 或 JSON 提要。[尝试演示](https://rssproxy.migor.org/)。它是 Feedless的替代 UI，但功能集有所减少。如果您需要全文 feeds、聚合、持久性、身份验证等高级功能，请查看 [feedless](https://github.com/damoeb/feedless/blob/master/docs/third-party-migration.md)
## 特征
- web2feed
- Feed to Feed：通过管道传输现有的原生 feed 给 rss-proxy 来过滤它们
- [过滤器](https://github.com/damoeb/feedless/blob/master/docs/filters.md)
- 自托管

## 高级功能
如果您寻找以下功能，则必须使用 [feedless](https://github.com/damoeb/feedless)

- feed 聚合
- 身份验证和多租户
- JavaScript 支持（预渲染）
- 全文提要和其他内容丰富
- 持久化
- CLI
- GraphQL API
- 插件

## 变更日志
看[这里](https://github.com/damoeb/rss-proxy/blob/master/changelog.md)
## 使用 docker 快速入门
如果您安装了 docker 或 podman，请执行此操作

	docker pull damoeb/rss-proxy:2.1
	docker run -p 8080:8080 -e APP_API_GATEWAY_URL=https://foo.bar -it damoeb/rss-proxy:2.1
`APP_API_GATEWAY_URL` 是您的外部网址，它将用作您创建的 feed 的主机。

然后在浏览器中打开 `localhost:8080`。

### 旧版本 1
如果您有兴趣运行第一个原型，那么您可以这样做。

	docker pull damoeb/rss-proxy:1
	docker run -p 3000:3000 -it damoeb/rss-proxy:1

然后在浏览器中打开localhost:3000 。

## 许可协议
该项目使用以下许可证：GNU GPLv3。

## 参考
[rss-proxy](https://github.com/damoeb/rss-proxy)