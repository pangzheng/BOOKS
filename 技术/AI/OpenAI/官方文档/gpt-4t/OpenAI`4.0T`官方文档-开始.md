# OpenAI`4.0T`官方文档-开始
## 介绍 
OpenAI API 几乎可以应用于任何任务。我们提供一系列具有不同功能和价位的模型，并且能够微调定制模型。
### 资源
- 在[游乐场](https://platform.openai.com/playground?mode=chat)上做实验
- 阅读 [API 参考](https://platform.openai.com/docs/api-reference)
- 访问[帮助中心](https://help.openai.com/en/)
- 查看当前 API [状态](https://status.openai.com/)
- 查看 [OpenAI 开发者论坛](https://community.openai.com/)
- 了解我们的[使用政策](https://openai.com/policies/usage-policies)

在 OpenAI，保护用户数据是我们使命的基础。我们不会通过 API 来训练我们的模型的输入和输出。[请访问我们的 API 数据隐私页面](https://openai.com/enterprise-privacy)了解更多信息。

### 关键概念
#### Text generation models(文本生成模型)
OpenAI 的文本生成模型（通常称为生成预训练 Transformer 或简称 “GPT” 模型），如 GPT-4 和 GPT-3.5，已经过训练来理解自然和形式语言。像 GPT-4 这样的模型允许文本输出来响应其输入。这些模型的输入也称为 “prompts(提示)”。设计提示本质上是如何  “(program)编程” GPT-4 等模型，通常是通过提供说明或一些如何成功完成任务的示例。GPT-4 等模型可用于多种任务，包括内容或代码生成、摘要、对话、创意写作等。请阅读我们的介绍性[文本生成指南](https://platform.openai.com/docs/guides/text-generation)和[提示工程指南](https://platform.openai.com/docs/guides/prompt-engineering)来了解更多信息。
#### Assistants(助理)
Assistants 是指实体，在 OpenAI API 的情况下，它们由 GPT-4 等大型语言模型提供支持，能够为用户执行任务。这些助手根据模型上下文窗口中嵌入的指令进行操作。他们通常还可以使用允许助理执行更复杂任务的工具，例如运行代码或从文件中检索信息。在我们的 [Assistant API 概述](https://platform.openai.com/docs/assistants)中了解有关助手的更多信息。
#### Embeddings
Embeddings 是一段数据（例如某些文本）的矢量表示，旨在保留其内容和/或其含义的各个方面。在某些方面相似的数据块往往比不相关的数据具有更紧密的嵌入。OpenAI 提供文本嵌入模型，该模型将文本字符串作为输入并生成嵌入向量作为输出。嵌入对于搜索、聚类、推荐、异常检测、分类等非常有用。在我们的 [Embeddings 指南](https://platform.openai.com/docs/guides/embeddings)中阅读有更多信息。
#### Tokens
文本生成和 Embeddings 模型以称为 token 的块的形式处理文本。token 代表常见的字符序列。例如，字符串 “tokenization” 被分解为 “token” 和“ization”，而像 “the” 这样的短而常见的单词则被表示为单个token。请注意，在句子中，每个单词的第一个 token 通常以空格字符开头。查看我们的[token生成器工具](https://platform.openai.com/tokenizer)来测试特定字符串并查看它们如何转换为 token。根据粗略的经验

	1个 token 大约相当于 4 个字符或英文文本的 0.75 个单词。
要记住的一个限制是，对于文本生成模型，提示和生成的输出的总和不得超过模型的最大上下文长度。对于 Embeddings 模型（不输出 token），输入必须短于模型的最大上下文长度。每个文本生成和嵌入模型的最大上下文长度可以在模型索引中找到。

### 指南
跳转到我们的指南之一以了解更多信息。

- [快速入门教程](https://platform.openai.com/docs/quickstart)
	
	通过构建快速示例应用程序来学习
- [文本生成](https://platform.openai.com/docs/guides/text-generation)
	
	了解如何生成和处理文本
- [Assistants](https://platform.openai.com/docs/assistants/)

	了解构建助手的基础知识
- [Embeddings](https://platform.openai.com/docs/guides/embeddings)

	了解如何搜索、分类和比较文本
- [Speech to text 语音转文字](https://platform.openai.com/docs/guides/speech-to-text)

	了解如何将语音转换为文本
- [图像生成](https://platform.openai.com/docs/guides/images)

	了解如何生成或编辑图像
- [想象](https://platform.openai.com/docs/guides/vision)

	了解如何使用 GPT-4 处理图像输入
	
## 开发者快速入门
### 使用 OpenAI API 启动并运行
OpenAI API 为开发人员提供了一个简单的界面，可在其应用程序中创建智能层，并由 OpenAI 最先进的模型提供支持。Chat Completions 端点为 ChatGPT 提供支持，并提供了一种简单的方法来将文本作为输入并使用 GPT-4 等模型来生成输出。

[想直接跳到代码吗？](https://platform.openai.com/docs/api-reference)
	
	跳过快速入门并深入了解 API 参考。

本快速入门旨在帮助您设置本地开发环境并发送您的第一个 API 请求。如果您是经验丰富的开发人员或只想深入使用 OpenAI API，[GPT 指南](https://platform.openai.com/docs/guides/text-generation) 的 [API 参考](https://platform.openai.com/docs/api-reference) 是一个很好的起点。通过本快速入门，您将了解到：

- 如何设置您的开发环境
- 如何安装最新的 SDK
- OpenAI API 的一些基本概念
- 如何发送您的第一个 API 请求

如果您在入门时遇到任何挑战或有疑问，请加入我们的开发者论坛。

### 账户设置
首先，创建一个 [OpenAI 帐户](https://platform.openai.com/signup)或[登录](https://platform.openai.com/login)。接下来，[导航到 API 密钥页面](https://platform.openai.com/account/api-keys)并“创建新密钥”，可以选择命名密钥。请务必将其保存在安全的地方，并且不要与任何人共享。
### 快速入门语言选择
选择您想要开始使用 OpenAI API 的工具或语言。支持 curl、python、node.js,这里选用 curl ，[详情参考](https://platform.openai.com/docs/quickstart?context=curl)
 
Curl 是一种流行的命令行工具，开发人员使用它向 API 发送 HTTP 请求。它需要最少的设置时间，但功能不如 Python 或 JavaScript 等功能齐全的编程语言。

- 第 1 步：设置 curl
	- 安装
- 第 2 步：设置您的 API 密钥

	现在我们已经可以使用 curl 了，下一步是在终端或命令行中设置 API 密钥。您可以选择跳过此步骤，只需在请求中包含您的 API 密钥，如步骤 3 中所述。

	- mac 系统
		- 打开终端：您可以在“应用程序”文件夹中找到它或使用 Spotlight（Command + Space）进行搜索。
		- 编辑 bash 配置文件：使用命令 `nano ~/.bash_profile` 或 `nano ~/.zshrc`（对于较新的 MacOS 版本）在文本编辑器中打开配置文件。
		- 添加环境变量：在编辑器中，添加以下行，替换 `your-api-key-here` 为您的实际 API 密钥：

				export OPENAI_API_KEY='your-api-key-here'
		- 保存并退出：按 Ctrl+O 写入更改，然后按 Ctrl+X 关闭编辑器。
		- 加载您的配置文件：使用命令`source ~/.bash_profile` 或 `source ~/.zshrc` 加载更新的配置文件。
		- 验证 `echo $OPENAI_API_KEY`：通过在终端中输入来验证设置。它应该显示您的 API 密钥。
	- win [详情参考](https://platform.openai.com/docs/quickstart?context=curl)
- 第 3 步：发送您的第一个 API 请求

	设置 API 密钥后，最后一步是发送第一个 API 请求。为此，下面包含了对 [Chat Completions](https://platform.openai.com/docs/api-reference/chat/create)、[Embeddings](https://platform.openai.com/docs/api-reference/embeddings/create) 和 [Images](https://platform.openai.com/docs/api-reference/images/create)  API的示例请求。由于 API 密钥是在步骤 2 中设置的，因此应通过 `$OPENAI_API_KEY` 终端或命令行自动引用它。您还可以手动替换 `$OPENAI_API_KEY` 为您的 API 密钥，但如果包含您的 API 密钥，请务必隐藏 curl 命令。

- Chat Completions

		curl https://api.openai.com/v1/chat/completions   -H "Content-Type: application/json"   -H "Authorization: Bearer $OPENAI_API_KEY"   -d '{
		    "model": "gpt-3.5-turbo",
		    "messages": [
		      {
		        "role": "system",
		        "content": "You are a poetic assistant, skilled in explaining complex programming concepts with creative flair."
		      },
		      {
		        "role": "user",
		        "content": "Compose a poem that explains the concept of recursion in programming."
		      }
		    ]
		  }'
- Embeddings

		curl https://api.openai.com/v1/embeddings   -H "Authorization: Bearer $OPENAI_API_KEY"   -H "Content-Type: application/json"   -d '{
		    "input": "The food was delicious and the waiter...",
		    "model": "text-embedding-ada-002"
		  }'
- Images

		curl https://api.openai.com/v1/images/generations   -H "Content-Type: application/json"   -H "Authorization: Bearer $OPENAI_API_KEY"   -d '{
		    "prompt": "A cute baby sea otter",
		    "n": 2,
		    "size": "1024x1024"
		  }'	
		  	  		  
聊天[完成](https://platform.openai.com/docs/api-reference/chat/create)示例仅突出了我们模型的一个优势领域：创造力。用一首格式良好的诗来解释递归（编程主题）是最好的开发人员和最好的诗人都会遇到的问题。在这种情况下，`gpt-3.5-turbo` 做到毫不费力。

### 下一步
现在您已经发出了第一个 OpenAI API 请求，是时候探索其他可能的方法了：

- 有关我们的模型和 API 的更多详细信息，请参阅我们的 [GPT 指南](https://platform.openai.com/docs/guides/text-generation)。
- 访问 [OpenAI Cookbook](https://cookbook.openai.com/)以获取深入的 API 用例示例以及常见任务的代码片段。
- 想知道 OpenAI 的模型有什么能力？查看我们的[示例提示库](https://platform.openai.com/examples)。
- 想要在不编写任何代码的情况下尝试 API 吗？开始在 [Playground](https://platform.openai.com/playground)中进行实验。
- 当您开始构建时，请牢记我们的[使用政策](https://openai.com/policies/usage-policies)。

## 模型
DevDay 推出新模型

我们很高兴地宣布推出 [GPT-4 Turbo](https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo)（128k 上下文窗口）的预览版和更新的[GPT-3.5 Turbo](https://platform.openai.com/docs/models/gpt-3-5)（16k 上下文窗口）。除此之外，这两种模型都具有改进的指令跟踪、JSON 模式、更可重复的输出和并行函数调用。了解更多。
### 概述
OpenAI API 由具有不同功能和价格点的多种模型提供支持。您还可以通过[fine-tuning(微调)](https://platform.openai.com/docs/guides/fine-tuning)，针对您的特定用例对我们的模型进行定制。

模型|描述
---|---
[GPT-4 和 GPT-4 Turbo](https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo)|一组改进 GPT-3.5 的模型，可以理解并生成自然语言或代码
[GPT-3.5](https://platform.openai.com/docs/models/gpt-3-5)|一组改进 GPT-3 的模型，可以理解并生成自然语言或代码
[GPT base](https://platform.openai.com/docs/models/gpt-base)|一组无需遵循指令即可理解并生成自然语言或代码的模型
[GPT-3 Legcy(遗产)](https://platform.openai.com/docs/models/gpt-3)|一组能够理解和生成自然语言的模型
[DALL.E](https://platform.openai.com/docs/models/dall-e)|可以在自然语言提示下生成和编辑图像的模型
[TTS(语音合成)](https://platform.openai.com/docs/models/tts)|一组可以将文本转换为听起来自然的语音的模型
[Whisper(耳语)](https://platform.openai.com/docs/models/whisper)|可以将音频转换为文本的模型
[Embeddings](https://platform.openai.com/docs/models/embeddings)|一组可以将文本转换为数字形式的模型
[Moderation](https://platform.openai.com/docs/models/moderation)|可以检测文本是否敏感或不安全的微调模型
[Deprecated(已弃用)](https://platform.openai.com/docs/deprecations)|已弃用的型号的完整列表以及建议的替代品

我们还发布了开源模型，包括 [Point-E](https://github.com/openai/point-e)、[Whisper](https://github.com/openai/whisper)、[Jukebox](https://github.com/openai/jukebox)和[CLIP](https://github.com/openai/CLIP)。

请访问我们的[模型索引](https://platform.openai.com/docs/model-index-for-researchers)，以便研究人员详细了解我们的研究论文中介绍了哪些模型以及 InstructGPT 和 GPT-3.5 等模型系列之间的差异。
### 型号持续升级
`gpt-3.5-turbo`、`gpt-4` 和 `gpt-4-32k` 指向最新型号版本。您可以通过发送请求后查看[响应对象](https://platform.openai.com/docs/api-reference/chat/object)来验证这一点。响应将包括所使用的特定模型版本（例如 `gpt-3.5-turbo-0613`）。

我们还提供静态模型版本，开发人员可以在引入更新模型后继续使用至少三个月。随着模型更新的新节奏，我们还让人们能够贡献评估，以帮助我们针对不同用例改进模型。如果您有兴趣，请查看 [OpenAI Evals](https://github.com/openai/evals) 存储库。

以下型号是临时快照，我们已经宣布了它们的弃用日期及其替换日期。如果您想使用最新的型号版本，请使用标准型号名称，例如 `gpt-4` 或 `gpt-3.5-turbo`。

型号名称|停产日期|更换型号
---|---|---
gpt-3.5-turbo-0613|2024 年 6 月 13 日|gpt-3.5-turbo-1106
gpt-3.5-turbo-0301|2024 年 6 月 13 日|gpt-3.5-turbo-1106
gpt-4-0314|2024 年 6 月 13 日|gpt-4-0613
gpt-4-32k-0314	|2024 年 6 月 13 日|gpt-4-32k-0613

在我们的[弃用页面](https://platform.openai.com/docs/deprecations)上了解有关模型弃用的更多信息。
### GPT-4 和 GPT-4 Turbo
GPT-4 是一个大型多模态模型（接受文本或图像输入并输出文本），由于其更广泛的常识和先进的推理能力，它可以比我们以前的任何模型更准确地解决难题。GPT-4 可在 OpenAI API 中向[付费客户提供](https://help.openai.com/en/articles/7102672-how-can-i-access-gpt-4)。与此类似 `gpt-3.5-turbo`，`GPT-4` 针对聊天进行了优化，但也适用于使用Chat Completions API 的传统完成任务。在我们的 [GPT 指南](https://platform.openai.com/docs/guides/text-generation)中了解如何使用 GPT-4 。

模型|描述|上下文窗口|训练数据
---|---|---|---|---|
gpt-4-1106-preview|GPT-4Turbo,最新的 GPT-4 模型具有改进的指令跟踪、JSON 模式、可重现的输出、并行函数调用等。最多返回 4,096 个输出标记。此预览模型尚不适合生产流量。了解更多。|128,000 个token|截至 2023 年 4 月
gpt-4-vision-preview|带视觉的 GPT-4 Turbo ,除了所有其他 GPT-4 Turbo 功能之外，还能够理解图像。最多返回 4,096 个输出标记。这是预览模型版本，尚不适合生产流量。了解更多。|128,000 个token|截至 2023 年 4 月
GPT-4|目前指向 `gpt-4-0613`. 查看型号的不断升级。|8,192 个 token|截至 2021 年 9 月
gpt-4-32k|目前指向 `gpt-4-32k-0613`. 查看型号的不断升级。|32,768 个 token|截至 2021 年 9 月
GPT-4-0613|2023 年 6 月 13 日的快照 `gpt-4`，改进了函数调用支持。|8,192 个 token|截至 2021 年 9 月
GPT-4-32K-0613|2023 年 6 月 13 日的快照 `gpt-4-32k`，改进了函数调用支持。|32,768 个 token|截至 2021 年 9 月
GPT-4-0314遗产|2023 年 3 月 14 日的快照，`gpt-4` 支持函数调用。此模型版本将于2024 年 6 月 13 日弃用。|8,192 个 token|截至 2021 年 9 月
GPT-4-32K-0314遗产|2023 年 3 月 14 日的快照，`gpt-4-32k` 支持函数调用。此模型版本将于2024 年 6 月 13 日弃用。|32,768 个 token|截至 2021 年 9 月

对于许多基本任务，GPT-4 和 GPT-3.5 模型之间的差异并不显着。然而，在更复杂的推理情况下，GPT-4 比我们之前的任何模型都更有能力。
### GPT-3.5
GPT-3.5 模型可以理解并生成自然语言或代码。我们在 GPT-3.5 系列中功能最强大且最具成本效益的模型已 gpt-3.5-turbo 针对使用[Chat Completions API](https://platform.openai.com/docs/api-reference/chat) 的聊天进行了优化，但也适用于传统的完成任务。

模型|描述|上下文窗口|训练数据
---|---|---|---
gpt-3.5-turbo-1106|更新了 GPT 3.5 Turbo|最新的 GPT-3.5 Turbo 模型具有改进的指令跟踪、JSON 模式、可重现的输出、并行函数调用等。最多返回 4,096 个输出标记。了解更多。|16,385 个token|截至 2021 年 9 月
GPT-3.5-turbo|目前指向 `gpt-3.5-turbo-0613` . 将指向 `gpt-3.5-turbo-1106` 2023年12月11日开始。请参阅持续型号升级。|4,096 个token|截至 2021 年 9 月
gpt-3.5-turbo-16k|目前指向 `gpt-3.5-turbo-0613`. 将指向 `gpt-3.5-turbo-1106` 2023年12月11日开始。请参阅持续型号升级。|16,385 个token|截至 2021 年 9 月
gpt-3.5-turbo-instruct|与旧版完成端点类似 `text-davinci-003` 但兼容，但与Chat Completions端点不兼容。|4,096 个token|截至 2021 年 9 月
GPT-3.5-turbo-0613遗产	| 2023 年 6 月 13 日的快照。`gpt-3.5-turbo` 将于2024 年 6 月 13 日弃用。|4,096 个token|截至 2021 年 9 月
gpt-3.5-turbo-16k-0613遗产|2023 年 6 月 13 日的快照。`gpt-3.5-16k-turbo`将于2024 年 6 月 13 日弃用。|16,385 个token|截至 2021 年 9 月
GPT-3.5-turbo-0301 遗产|2023 年 3 月 1 日的快照。`gpt-3.5-turbo`将于2024 年 6 月 13 日弃用。|4,096 个token|截至 2021 年 9 月
text-davinci-003遗产|与居里、巴贝奇或 ada 模型相比，可以以更好的质量和一致性完成语言任务。将于2024 年 1 月 4 日弃用。|4,096 个token|截至 2021 年 6 月
text-davinci-002​​ 遗产|与类似的功能，`text-davinci-003` 但通过监督微调而不是强化学习进行训练。将于2024 年 1 月 4 日弃用。|4,096 个token|截至 2021 年 6 月
code-davinci-003遗产|针对代码完成任务进行了优化。将于2024 年 1 月 4 日弃用。|8,001 个token|截至 2021 年 6 月

我们建议使用 `gpt-3.5-turbo` 其他 GPT-3.5 型号，因为它的成本更低，性能更高。
### DALL·E
DALL·E 是一个人工智能系统，可以根据自然语言的描述创建逼真的图像和艺术。DALL·E 3 目前支持根据提示创建具有特定尺寸的新图像的功能。DALL·E 2 还支持编辑现有图像或创建用户提供的图像的变体的功能。

[DALL·E 3](https://openai.com/dall-e-3) 可通过我们的 [图像 API](https://platform.openai.com/docs/guides/images/introduction)与[DALL·E 2](https://openai.com/blog/dall-e-api-now-available-in-public-beta)一起使用。您可以通过 [ChatGPT Plus](https://chat.openai.com/) 尝试 DALL·E 3 。

模型|描述
---|---
dall-e-3	|DALL·E 3新的 2023 年 11 月发布的最新 DALL·E 模型。[了解更多](https://openai.com/blog/new-models-and-developer-products-announced-at-devday)。
dall-e-2	|之前的 DALL·E 模型于 2022 年 11 月发布。DALL·E 的第二次迭代具有比原始模型更真实、更准确且分辨率提高 4 倍的图像。
### TTS 语音合成
TTS 是一种人工智能模型，可将文本转换为听起来自然的语音文本。我们提供两种不同的模型变量，`tts-1` 针对实时文本到语音用例进行了优化，和 `tts-1-hd` 针对质量进行了优化。这些模型可以与音频 API 中的语音端点一起使用。

模型|描述
---|---
tts-1|Text-to-speech 1(文字转语音1),最新的文本转语音模型，针对速度进行了优化。
tts-1-hd|Text-to-speech 1 HD(文字转语音 1 高清),最新的文本转语音模型，针对质量进行了优化。
### Whisper(耳语)
Whisper 是一种通用语音识别模型。它是在各种音频的大型数据集上进行训练的，也是一个多任务模型，可以执行多语言语音识别以及语音翻译和语言识别。Whisper v2-large 模型目前可通过我们的 API 和 whisper-1 模型名称获取。

目前， [Whisper 的开源版本](https://github.com/openai/whisper)和通过我们的 API 提供的版本没有区别。然而，通过我们的 API，我们提供了优化的推理过程，这使得[通过我们的 API](https://platform.openai.com/docs/guides/speech-to-text) 运行 Whisper 比通过其他方式运行要快得多。有关 Whisper 的更多技术细节，您可以[阅读论文](https://arxiv.org/abs/2212.04356)。
### Embeddings
Embeddings 是文本的数字表示，可用于衡量两段文本之间的相关性。我们的第二代 Embeddings 模型 `text-embedding-ada-002` 旨在以一小部分成本取代之前的 16 个第一代 Embeddings 模型。Embeddings 对于搜索、聚类、推荐、异常检测和分类任务非常有用。您可以在[公告博客文章](https://openai.com/blog/new-and-improved-embedding-model)中阅读有关我们最新 Embeddings 模型的更多信息。
### Moderation(审核)
审核模型旨在检查内容是否符合 OpenAI 的使用[政策](https://openai.com/policies/usage-policies)。这些模型提供分类功能，可查找以下类别的内容：仇恨、仇恨/威胁、自残、性、性/未成年人、暴力和暴力/图形。您可以在我们的[审核指南](https://platform.openai.com/docs/guides/moderation/overview)中了解更多信息。

审核模型接受任意大小的输入，该输入会自动分解为 4,096 个 token 的块。如果输入超过 32,768 个token，则会使用截断，在极少数情况下，可能会在审核检查中省略少量 token。

每个对审核端点的请求的最终结果显示每个类别的最大值。例如，如果一块 4K token 的类别分数为 0.9901，另一块的分数为 0.1901，则结果将在 API 响应中显示 0.9901，因为它更高。

模型|描述|最大 token
---|---|---
`text-moderation-latest(文本审核最新)`|最有能力的审核模型。准确度会比稳定模型略高。|32,768
`text-moderation-stable(文本审核稳定)`｜几乎与最新型号一样功能，但稍旧。｜32,768
### GPT 基础
GPT 基础模型可以理解并生成自然语言或代码，但未接受以下指令的训练。这些模型旨在替代我们原来的 GPT-3 基本模型，并使用旧版 Completions API。大多数客户应使用 GPT-3.5 或 GPT-4。

模型|描述|最大 token|训练数据
---|---|---|---|
babbage-002|GPT-3ada和babbage基本型号的替代品。|16,384 个token|截至 2021 年 9 月
davinci-002|GPT-3curie和davinci基本型号的替代品。|16,384 个token|截至 2021 年 9 月
### GPT-3遗产
GPT-3 模型可以理解并生成自然语言。这些模型被更强大的 GPT-3.5 代模型所取代。然而，原始的 GPT-3 基本模型（davinci、curie、ada和babbage）是当前唯一可进行微调的模型。

模型|描述|最大token|训练数据
---|---|---|---|---|
text-curie-001|比达芬奇能力更强、速度更快、成本更低。|2,049 个token|截至 2019 年 10 月
text-babbage-001|能够完成简单的任务、速度非常快且成本较低。|2,049 个token|截至 2019 年 10 月
text-ada-001|能够执行非常简单的任务，通常是 GPT-3 系列中最快的型号，并且成本最低。|2,049 个token|截至 2019 年 10 月
davinci|最有能力的 GPT-3 模型。可以完成其他模型可以完成的任何任务，而且质量通常更高。|2,049 个token|截至 2019 年 10 月
curie|非常有能力，但比达芬奇更快、成本更低。|2,049 个token|截至 2019 年 10 月
babbage|能够完成简单的任务、速度非常快且成本较低。|2,049 个token|截至 2019 年 10 月
ada|能够执行非常简单的任务，通常是 GPT-3 系列中最快的型号，并且成本最低。|2,049 个token|截至 2019 年 10 月
### 我们如何使用您的数据
你的数据就是你的数据。

自 2023 年 3 月 1 日起，发送到 OpenAI API 的数据将不会用于训练或改进 OpenAI 模型（除非您明确选择加入）。选择加入的好处之一是，随着时间的推移，模​​型可能会越来越适合您的用例。

为了帮助识别滥用行为，API 数据最多可保留 30 天，之后将被删除（除非法律另有要求）。对于拥有敏感应用程序的值得信赖的客户，零数据保留可能是可用的。在零数据保留的情况下，请求和响应主体不会持久保存到任何日志记录机制中，并且仅存在于内存中以便为请求提供服务。

请注意，此数据政策不适用于 OpenAI 的非 API 消费者服务，例如 ChatGPT 或 DALL·E Labs。

### 按端点的默认使用策略
端点|用于训练的数据|默认保留|符合零保留资格
---|---|---|---|---|
/v1/chat/completions\*|不|30天|是的，图像输入除外*
/v1/files|不|直至被客户删除|不
/v1/assistants|不|直至被客户删除|不
/v1/threads|不|30天|不
/v1/threads/messages|不|30天|不
/v1/threads/runs|不|30天|不
/v1/threads/runs/steps|不|30天|不
/v1/images/generations|不|30天|不
/v1/images/edits	|不|30天|不
/v1/images/variations|不|30天|不
/v1/embeddings|不|30天|是的
/v1/audio/transcriptions|不|零数据保留|-
/v1/audio/translations|不|零数据保留|-
/v1/audio/speech|不|30天|-
/v1/fine_tuning/jobs|不|直至被客户删除|不
/v1/fine-tunes|不	直至被客户删除|不
/v1/moderations|不|零数据保留|-
/v1/completions|不|30天|是的
/v1/edits|不|30天|是的

*通过模型输入的图像gpt-4-vision-preview不符合零保留条件。

有关详细信息，请参阅我们的[API 数据使用政策](https://openai.com/policies/api-data-usage-policies)。要了解有关零保留的更多信息，请联系我们的[销售团队](https://openai.com/contact-sales)。

### 模型端点兼容性
端点|最新型号
---|---|
/v1/assistants|除受支持之外的所有型号 `gpt-3.5-turbo-0301`。retrieval 工具需要 `gpt-4-1106-preview` 或 `gpt-3.5-turbo-1106`。
/v1/audio/transcriptions|whisper-1
/v1/audio/translations|whisper-1
/v1/audio/speech|tts-1,tts-1-hd
/v1/chat/completions| `gpt-4`和注明日期的模型版本 `gpt-4-1106-preview`,`gpt-4-32k` 注明日期的模型版本`gpt-4-vision-preview`, `gpt-3.5-turbo`和注明日期的模型版本,`gpt-3.5-turbo-16k`、微调版本注明日期的模型版本是 `gpt-3.5-turbo`
/v1/completions (Legacy)|gpt-3.5-turbo-instruct, babbage-002,davinci-002
/v1/embeddings|	text-embedding-ada-002
/v1/fine_tuning/jobs|gpt-3.5-turbo, babbage-002,davinci-002
/v1/moderations|	text-moderation-stable,text-moderation-latest
/v1/images/generations|dall-e-2,dall-e-3

此列表不包括我们所有[已弃用](https://platform.openai.com/docs/deprecations)的型号。

## 教程
通过逐步构建真正的 AI 应用程序，开始使用 OpenAI API。

- [带有 embeddings 的网站问答](https://platform.openai.com/docs/tutorials/web-qa-embeddings)

	了解如何构建可以回答有关您网站的问题的人工智能。
- [使用 Whisper 转录会议纪要](https://platform.openai.com/docs/tutorials/meeting-minutes)

	了解如何使用 Whisper 和 GPT-4 创建自动会议纪要生成器。
- [即将推出](https://platform.openai.com/docs/tutorials/)

	了解如何构建和部署理解多个知识库的人工智能聊天机器人。

寻找更多想法？查看我们的[示例库](https://platform.openai.com/examples)或GitHub 上的 [OpenAI Cookbook](https://cookbook.openai.com/)。

## 参考
[官方文档-开始使用](https://platform.openai.com/docs/overview)


	