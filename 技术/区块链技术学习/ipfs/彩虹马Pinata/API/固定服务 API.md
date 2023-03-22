# 固定服务 API

	Pinning Services API 目前不符合规范，正在评估未来的支持，性能可能会有所不同 
## 固定服务 API
### IPFS 固定服务 API 规范兼容性
PFS [Pinning Servers API Spec] 是一个标准化的规范，适用于构建在 IPFS 之上的开发人员，它允许应用程序集成 pinning 服务，而无需了解该 pinning 服务的独特 API。

- 端点

	希望使用 IPFS Pinning Services API 的 Pinata 用户可以从我们专用的 API 端点执行此操作：

		https://api.pinata.cloud/psa
	有关可用端点的最新列表以及当前文档，请访问[官方端点文档](https://ipfs.github.io/pinning-services-api-spec/#tag/pins)。
- 验证

	要通过 Pinning Services API 规范对 Pinata 进行身份验证，您首先需要拥有一个 `accessToken`. [可以在key管理页面](https://pinata.cloud/keys)上创建此 API Token。
	
	当您创建新的 API 密钥时，您需要记下创建后立即显示给您的JWT(Json Web Token). 此 JWT 特定于 API 密钥并共享相同的权限。

	如果您曾经撤销此 JWT 的 API 密钥，则此 JWT 将不再对 Pinning Services API 的身份验证有效。

	Pinning Services API 规范使用 Bearer Token 方法进行身份验证，您将通过以下方式使用 JWT 提供身份验证标头：

		Authorization: Bearer JWTForYourAPIKey
- 在 IPFS 桌面应用程序中配置 Pinata

	如果您正在运行 IPFS 桌面应用程序，则可以在用户界面中配置您选择的固定服务。
	
	- 为此，请打开应用程序，转到您的首选项，然后单击添加服务：
	- 选择 Pinata 作为固定服务，然后使用您的秘密访问令牌 (JWT) 进行配置：
- 在 IPFS CLI 中配置 Pinata

	您还可以使用命令直接从 IPFS CLI 固定到 Pinata ipfs。

	- 要添加 Pinata 凭据，请使用以下命令（其中 YOUR_JWT 是上面“身份验证”部分中描述的 JWT 令牌）：

			ipfs pin remote service add pinata https://api.pinata.cloud/psa YOUR_JWT
	- 要以人类可读的名称将 CID 固定到 Pinata：

			ipfs pin remote add --service=pinata --name=war-and-peace.txt bafybeib32tuqzs2wrc52rdt56cz73sqe3qu2deqdudssspnu4gbezmhig4
	- 列出成功的 pin：

			ipfs pin remote ls --service=pinata
	- 列出挂起的 pin：

			ipfs pin remote ls --service=pinata --status=queued,pinning,failed
	- 有关更多命令和一般帮助：

			ipfs pin remote --help
