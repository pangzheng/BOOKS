# langchain-开始使用
## 介绍
LangChain 是一个用于开发由语言模型支持的应用程序的框架。它使应用程序能够：

- 具有上下文感知能力

	将语言模型与上下文源连接起来（提示说明、几个镜头示例、响应的内容等）
- 原因

	依靠语言模型进行推理（关于如何根据提供的上下文回答、采取什么操作等）

LangChain 的主要价值道具有：

- 组件

	用于处理语言模型的抽象，以及每个抽象的实现集合。无论您是否使用 LangChain 框架的其余部分，组件都是模块化且易于使用的
- 现成的链

	用于完成特定更高级别任务的组件的结构化组装
	
现成的 LangChain 让您可以轻松上手。对于复杂的应用程序，组件可以轻松定制现有链和构建新链。

## 开始使用
[以下](https://python.langchain.com/docs/get_started/installation)是如何安装 LangChain、设置环境并开始构建。

我们建议您按照我们的[快速入门](https://python.langchain.com/docs/get_started/quickstart)指南构建您的第一个 LangChain 应用程序来熟悉该框架。

注意：这些文档适用于 [LangChain Python 包](https://github.com/hwchase17/langchain)。有关 JS/TS 版本 [LangChain.js](https://github.com/hwchase17/langchainjs) 的文档，请前往[此处](https://js.langchain.com/docs)。

## 模块
LangChain 为以下模块提供标准的、可扩展的接口和外部集成，按照从最简单到最复杂的顺序列出：

- [模型输入/输出](https://python.langchain.com/docs/get_started/introduction#model-io)

	与语言模型的接口
- [retrieval](https://python.langchain.com/docs/get_started/introduction#retrieval)

	与应用程序特定数据的接口
- [chains](https://python.langchain.com/docs/get_started/introduction#chains)

	构建调用序列
- [Agents](https://python.langchain.com/docs/modules/agents/)

	让链根据给定的高级指令选择使用哪些工具
- [memory](https://python.langchain.com/docs/modules/memory/)

	在链运行之间保留应用程序状态
- [callbacks](https://python.langchain.com/docs/modules/callbacks/)

	记录并流式传输任何链的中间步骤

## 例子、生态和资源
### [用例](https://python.langchain.com/docs/use_cases/question_answering/)
常见端到端用例的演练和最佳实践，例如：

- [文档问答](https://python.langchain.com/docs/use_cases/question_answering/)
- [聊天机器人](https://python.langchain.com/docs/use_cases/chatbots/)
- [分析结构化数据](https://python.langchain.com/docs/use_cases/qa_structured/sql/)
- 以及更多...

### [手册](https://python.langchain.com/docs/guides/)
了解使用 LangChain 进行开发的最佳实践。
### [生态](https://python.langchain.com/docs/integrations/providers/)
LangChain 是丰富的工具生态系统的一部分，它与我们的框架集成并建立在其之上。查看我们不断增长的[集成](https://python.langchain.com/docs/integrations/providers/)和[依赖存储库列表](https://python.langchain.com/docs/additional_resources/dependents)。
### [额外资源](https://python.langchain.com/docs/additional_resources/)
我们的社区充满了多产的开发人员、富有创造力的建设者和出色的教师。查看 [YouTube 教程](https://python.langchain.com/docs/additional_resources/youtube)，了解社区人员提供的精彩教程，并查看 [Gallery](https://github.com/kyrolabs/awesome-langchain) ，了解由 [KyroLabs](https://kyrolabs.com/) 人员编制的精彩 LangChain 项目列表。
### [社区](https://python.langchain.com/docs/community)
前往[社区导航器](https://python.langchain.com/docs/community)寻找提出问题、分享反馈、结识其他开发人员并梦想 LLM 未来的地方。

## API 参考
请前往[参考](https://api.python.langchain.com/)部分，获取 LangChain Python 包中所有类和方法的完整文档。