# langchain-快速开始
## 安装
请参阅我们的[安装指南](https://python.langchain.com/docs/get_started/installation.html)。
## 环境设置
使用 LangChain 通常需要与一个或多个模型提供者、数据存储、API 等集成。在本例中，我们将使用 OpenAI 的模型 API。

首先我们需要安装他们的 Python 包：

	pip install openai
访问 API 需要 API 密钥，您可以通过创建帐户并前往[此处](https://platform.openai.com/account/api-keys)获取该密钥。一旦我们有了密钥，我们就需要通过运行以下命令将其设置为环境变量：

	export OPENAI_API_KEY="..."
如果您不想设置环境变量，可以 `openai_api_key` 在启动 OpenAI LLM 类时直接通过命名参数传递密钥：

	from langchain.llms import OpenAI
	
	llm = OpenAI(openai_api_key="...")
## 构建一个应用
现在我们可以开始构建我们的语言模型应用程序。LangChain 提供了许多可用于构建语言模型应用程序的模块。模块可以在简单的应用程序中独立使用，也可以组合起来用于更复杂的用例。

LangChain 帮助创建的最常见、最重要的链包含三个内容：

- LLM

	语言模型是这里的核心推理引擎。为了使用 LangChain，您需要了解不同类型的语言模型以及如何使用它们。
- Prompt 提示模板

	这为语言模型提供指令。这控制着语言模型的输出内容，因此了解如何构建提示和不同的提示策略至关重要。
- 输出解析器

	它们将 LLM 的原始响应转换为更可行的格式，从而可以轻松使用下游的输出。

在本入门指南中，我们将单独介绍这三个组件，然后介绍如何将它们组合起来。了解这些概念将为您使用和定制 LangChain 应用程序做好准备。大多数 LangChain 应用程序允许您配置 LLM 和/或使用的提示，因此了解如何利用这一点将是一个很大的推动因素。

## LLMs
语言模型有两种类型，在 LangChain 中称为：

- LLM

	这是一种语言模型，它将字符串作为输入并返回字符串
- ChatModels

	这是一个语言模型，它将消息列表作为输入并返回消息

LLM 的输入/输出简单易懂——一个字符串。但是 ChatModels 呢？输入是一个 `ChatMessages` 列表，输出是一个 s 列表`ChatMessage`。`ChatMessage` 有两个必需的组件：

- `content`

	这是消息的内容。
- `roleChatMessage`

	这是来自实体的角色。

LangChain 提供了几个对象来方便区分不同的角色：

- `HumanMessage`

	ChatMessage 来自人类/用户。
- `AIMessage`

	ChatMessage：来自人工智能/助手。
- `SystemMessage`

	ChatMessage 来自系统。
- `FunctionMessage`

	ChatMessage 来自函数调用。
	
如果这些角色听起来都不合适，还有一个 `ChatMessage` 类可以让您手动指定角色。有关如何最有效地使用这些不同消息的更多信息，请参阅我们的提示指南。

LangChain 为两者提供了标准接口，但了解这种差异对于为给定语言模型构建提示很有用。LangChain 提供的标准接口有两种方法：

- `predict`

	接收一个字符串，返回一个字符串
- `predict_messages`

	接收消息列表，返回消息。

让我们看看如何使用这些不同类型的模型和这些不同类型的输入。首先，我们导入一个 LLM 和一个 ChatModel

	from langchain.llms import OpenAI
	from langchain.chat_models import ChatOpenAI
	
	llm = OpenAI()
	chat_model = ChatOpenAI()
	
	llm.predict("hi!")
	>>> "Hi"
	
	chat_model.predict("hi!")
	>>> "Hi"

`OpenAI` 和对象 `ChatOpenAI` 基本上只是配置对象。您可以使用等参数初始化它们 `temperature`，然后传递它们。

接下来，让我们使用该 `predict` 方法来运行字符串输入。

	text = "What would be a good company name for a company that makes colorful socks?"
	
	llm.predict(text)
	# >> Feetful of Fun
	
	chat_model.predict(text)
	# >> Socks O'Color
最后，让我们使用该 `predict_messages` 方法来运行消息列表。

	from langchain.schema import HumanMessage
	
	text = "What would be a good company name for a company that makes colorful socks?"
	messages = [HumanMessage(content=text)]
	
	llm.predict_messages(messages)
	# >> Feetful of Fun
	
	chat_model.predict_messages(messages)
	# >> Socks O'Color
对于这两种方法，您还可以传入参数作为关键字参数。例如，您可以传入 `temperature=0` 以调整对象配置的 temperature。在运行时传入的任何值都将始终覆盖对象配置的值。

## 提示词模版
大多数 LLM 申请不会将用户输入直接传递到LLM。通常，他们会将用户输入添加到较大的文本中，称为提示模板，该文本提供有关当前特定任务的附加上下文。

在前面的示例中，我们传递给模型的文本包含生成公司名称的指令。对于我们的应用程序，如果用户只需提供公司/产品的描述，而不必担心提供模型说明，那就太好了。

PromptTemplates 正好可以帮助解决这个问题！它们捆绑了从用户输入到完全格式化的提示的所有逻辑。这可以非常简单地开始 - 例如，生成上述字符串的提示就是：

	from langchain.prompts import PromptTemplate
	
	prompt = PromptTemplate.from_template("What is a good name for a company that makes {product}?")
	prompt.format(product="colorful socks")


`What is a good name for a company that makes colorful socks?`

然而，与原始字符串格式化相比，使用它们有几个优点。您可以“部分”输出变量 

- 例如，您一次只能格式化部分变量。您可以将它们组合在一起，轻松地将不同的模板组合成一个提示。有关这些功能的说明，请参阅有关[提示的部分](https://python.langchain.com/docs/modules/model_io/prompts)以了解更多详细信息。

PromptTemplates 还可用于生成消息列表。在这种情况下，提示不仅包含有关内容的信息，还包含每条消息（其角色、在列表中的位置等）。这里，最常发生的是

-  `ChatPromptTemplate` 是 `ChatMessageTemplates` 的列表。
-  每个 `ChatMessageTemplate` 都包含有关如何格式化该 `ChatMessage` 的说明 

它的角色，以及它的内容。让我们看看下面这个：

	from langchain.prompts.chat import ChatPromptTemplate
	
	template = "You are a helpful assistant that translates {input_language} to {output_language}."
	human_template = "{text}"
	
	chat_prompt = ChatPromptTemplate.from_messages([
	    ("system", template),
	    ("human", human_template),
	])
	
	chat_prompt.format_messages(input_language="English", output_language="French", text="I love programming.")
	
	
	[
	    SystemMessage(content="You are a helpful assistant that translates English to French.", additional_kwargs={}),
	    HumanMessage(content="I love programming.")
	]
`ChatPromptTemplates` 也可以通过其他方式构建 -有关更多详细信息，请参阅[提示部分](https://python.langchain.com/docs/modules/model_io/prompts)。

## 输出解析器
OutputParsers 将 LLM 的原始输出转换为可在下游使用的格式。OutputParsers 有几种主要类型，包括：

- 从 LLM 转换文本 -> 结构化信息（例如 JSON）
- 将 ChatMessage 转换为字符串
- 将除消息之外的调用返回的额外信息（如 OpenAI 函数调用）转换为字符串。

有关这方面的完整信息，请参阅[输出解析器部分](https://python.langchain.com/docs/modules/model_io/output_parsers)

在本入门指南中，我们将编写自己的输出解析器 - 将逗号分隔列表转换为列表的解析器。

	from langchain.schema import BaseOutputParser
	
	class CommaSeparatedListOutputParser(BaseOutputParser):
	    """将LLM调用的输出解析为逗号分隔的列表。"""
	
	
	    def parse(self, text: str):
	        """解析LLM调用的输出"""
	        return text.strip().split(", ")
	
	CommaSeparatedListOutputParser().parse("hi, bye")
	# >> ['hi', 'bye']

## PromptTemplate + LLM + OutputParser
我们现在可以将所有这些组合成一条链。该链将获取输入变量，将这些变量传递给提示模板以创建提示，将提示传递给语言模型，然后通过（可选）输出解析器传递输出。这是捆绑模块化逻辑块的便捷方法。让我们看看它的实际效果！

	from langchain.chat_models import ChatOpenAI
	from langchain.prompts.chat import ChatPromptTemplate
	from langchain.schema import BaseOutputParser
	
	class CommaSeparatedListOutputParser(BaseOutputParser):
	    """将LLM调用的输出解析为逗号分隔的列表。"""
	
	
	    def parse(self, text: str):
	        """解析LLM调用的输出."""
	        return text.strip().split(", ")
	
	template = """You are a helpful assistant who generates comma separated lists.
	A user will pass in a category, and you should generate 5 objects in that category in a comma separated list.
	ONLY return a comma separated list, and nothing more."""
	
	human_template = "{text}"
	
	chat_prompt = ChatPromptTemplate.from_messages([
	    ("system", template),
	    ("human", human_template),
	])
	chain = chat_prompt | ChatOpenAI() | CommaSeparatedListOutputParser()
	chain.invoke({"text": "colors"})
	# >> ['red', 'blue', 'green', 'yellow', 'orange']


请注意，我们正在使用 `|` 语法将这些组件连接在一起。这种 `|` 语法称为 LangChain 表达式语言。要了解有关此语法的更多信息，请阅读[此处](https://python.langchain.com/docs/expression_language)的文档。

## 后续步骤
就是这个！现在我们已经了解了如何创建 LangChain 应用程序的核心构建块。所有这些组件（LLM、提示、输出解析器）都有更多的细微差别，并且还有更多不同的组件需要学习。要继续您的旅程：

- [深入研究](https://python.langchain.com/docs/modules/model_io) LLM、提示和输出解析器
- 了解其他 [关键组件](https://python.langchain.com/docs/modules)
- 阅读 [LangChain 表达式语言](https://python.langchain.com/docs/expression_language)，了解如何将这些组件链接在一起
- 查看我们的[有用指南](https://python.langchain.com/docs/guides)，了解特定主题的详细演练
- 探索[端到端用例](https://python.langchain.com/docs/use_cases)