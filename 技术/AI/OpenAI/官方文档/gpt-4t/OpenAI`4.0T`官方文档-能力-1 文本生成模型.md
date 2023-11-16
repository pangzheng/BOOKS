# OpenAI`4.0T`官方文档-能力-1 文本生成模型
## 文本生成模型
- DevDay 推出新功能

	文本生成模型现在支持 JSON 模式和可重复输出。我们还推出了Assistants API，使您能够在我们的文本生成模型之上构建类似代理的体验。通过指定gpt-4-1106-preview 作为模型名称，GPT-4 Turbo 可以在预览版中使用。

OpenAI 的文本生成模型（通常称为生成式预训练 Transformer 或大型语言模型）经过训练可以理解自然语言、代码和图像。这些模型提供文本输出来响应其输入。这些模型的输入也称为“提示”。设计提示本质上是如何“编程”大型语言模型，通常是通过提供说明或一些如何成功完成任务的示例。

使用 OpenAI 的文本生成模型，您可以构建应用程序来：

- 文件草案
- 编写计算机代码
- 回答有关知识库的问题
- 分析文本
- 为软件提供自然语言界面
- 一系列科目的导师
- 翻译语言
- 模拟游戏角色

随着 的发布 `gpt-4-vision-preview`，您现在可以构建还可以处理和理解图像的系统。

- [通过图像输入探索 GPT-4](https://platform.openai.com/docs/guides/vision)

	查看视觉指南​​了解更多详细信息。
- [GPT-4 Turbo](https://platform.openai.com/playground?mode=chat&model=gpt-4-1106-preview)

	在操场上试用 GPT-4 Turbo。

要通过 OpenAI API 使用这些模型之一，您将发送包含输入和 API 密钥的请求，并接收包含模型输出的响应。我们最新的模型 `gpt-4` 和 `gpt-3.5-turbo` 是通过Chat Completions API 端点访问的。

||模型|API端点
---|---|---|
较新型号 (2023–)|gpt-4（和 gpt-4 Turbo）、gpt-3.5-turbo|https://api.openai.com/v1/chat/completions
更新基本型号（2023）|babbage-002, davinci-002|https://api.openai.com/v1/completions
旧型号（2020–2022）|text-davinci-003, text-davinci-002, davinci, curie, babbage, ada||https://api.openai.com/v1/completions

您可以在[聊天游乐场](https://platform.openai.com/playground?mode=chat)中尝试各种模型。如果您不确定要使用哪个模型，请使用 `gpt-3.5-turbo` 或`gpt-4`。

### Chat Completions API
聊天模型将消息列表作为输入，并返回模型生成的消息作为输出。尽管聊天格式旨在使多轮对话变得容易，但它对于没有任何对话的单轮任务也同样有用。

Chat Completions API 调用示例如下所示：

	curl https://api.openai.com/v1/chat/completions \
	  -H "Content-Type: application/json" \
	  -H "Authorization: Bearer $OPENAI_API_KEY" \
	  -d '{
	    "model": "gpt-3.5-turbo",
	    "messages": [
	      {
	        "role": "system",
	        "content": "You are a helpful assistant."
	      },
	      {
	        "role": "user",
	        "content": "Who won the world series in 2020?"
	      },
	      {
	        "role": "assistant",
	        "content": "The Los Angeles Dodgers won the World Series in 2020."
	      },
	      {
	        "role": "user",
	        "content": "Where was it played?"
	      }
	    ]
	  }'
  
要了解更多信息，您可以查看聊天 API 的完整 [API 参考文档](https://platform.openai.com/docs/api-reference/chat)。

主要输入是 messages 参数。消息必须是消息对象的数组，其中每个对象都有一个角色（“系统”、“用户”或“assistant（助理）”）和内容。对话可以短至一条消息，也可以来回多次。

通常，对话首先由系统消息格式化，然后是交替的用户消息和assistant（助理）消息。

- `系统消息( "role": "system")` 有助于设置助手的行为

	例如，您可以修改助手的个性或提供有关其在整个对话过程中应如何表现的具体说明。但请注意，系统消息是可选的，并且没有系统消息的模型的行为可能类似于使用通用消息，例如“你是一个有用的助手”。
- `用户消息( "role": "user",)` 提供 assistant（助理）响应的请求或评论。
- `assistant（助理）消息("role": "assistant")` 存储以前的assistant（助理）响应，但也可以由您编写以给出所需行为的示例。

当用户指令引用之前的消息时，包含对话历史记录非常重要。

在上面的示例中，用户的最后一个问题是“在哪里播放的？” 仅在有关 2020 年世界职业棒球大赛的先前消息的上下文中才有意义。由于模型没有过去请求的记忆，因此所有相关信息必须作为每个请求中的对话历史记录的一部分提供。如果对话无法满足模型的 token 限制，则需要以某种方式[缩短](https://platform.openai.com/docs/guides/prompt-engineering/tactic-for-dialogue-applications-that-require-very-long-conversations-summarize-or-filter-previous-dialogue)对话。

要模仿 ChatGPT 中迭代返回文本的效果，请将[流参数](https://platform.openai.com/docs/api-reference/chat/create#chat/create-stream)设置为 true。

#### Chat Completions响应格式
Chat Completions API 响应示例如下所示：

	{
	  "choices": [
	    {
	      "finish_reason": "stop",
	      "index": 0,
	      "message": {
	        "content": "The 2020 World Series was played in Texas at Globe Life Field in Arlington.",
	        "role": "assistant"
	      }
	    }
	  ],
	  "created": 1677664795,
	  "id": "chatcmpl-7QyqpwdfhqwajicIEznoc6Q47XAyW",
	  "model": "gpt-3.5-turbo-0613",
	  "object": "chat.completion",
	  "usage": {
	    "completion_tokens": 17,
	    "prompt_tokens": 57,
	    "total_tokens": 74
	  }
	}
`assistant(助理)` 的回复可以通过以下方式提取(python)-需要使用 `pip install --upgrade openai`

	response['choices'][0]['message']['content']
每个响应都将包含一个 `finish_reason`. 可能的值为 `finish_reason`：

- `stop`

	API 返回完整消息，或由通过 stop 参数提供的停止序列之一终止的消息
- `length`

	由于 `max_tokens` 参数或 `token` 限制，模型输出不完整
- `function_call`

	模型决定调用一个函数
- `content_filter`

	由于我们的内容过滤器中的token而省略了内容
- `null`

	API 响应仍在进行中或不完整

根据输入参数，模型响应可能包括不同的信息。

### JSON mode
使用Chat Completions的常见方法是通过在系统消息中指定，指示模型始终返回对您的用例有意义的 JSON 对象。虽然这在某些情况下确实有效，但有时模型可能会生成无法解析为有效 JSON 对象的输出。

为了防止这些错误并提高模型性能，在调用 `gpt-4-1106-preview` 或时 `gpt-3.5-turbo-1106`，可以将[response_format](https://platform.openai.com/docs/api-reference/chat/create#chat-create-response_format) 设置为 `{ "type": "json_object" }` 启用 JSON 模式。启用 JSON 模式时，模型仅限于生成解析为有效 JSON 对象的字符串。

重要笔记：

- 使用 JSON 模式时，请`始终`指示模型通过对话中的某些消息（例如通过系统消息）生成 `JSON`。如果您不包含生成 JSON 的显式指令，则模型可能会生成无休止的空白流，并且请求可能会持续运行，直到达到 `token` 限制。`"JSON"` 为了帮助确保您不会忘记，如果该字符串没有出现在上下文中的某个位置，API 将抛出错误。
- `finish_reason` 如果是 `length `，则模型返回的消息中的 JSON 可能是部分的（即被截断，这表示生成超出`max_tokens` 或会话超出 `token` 限制。为了防止这种情况，请 `finish_reason` 在解析响应之前进行检查。
- JSON 模式不保证输出匹配任何特定模式，仅保证其有效且解析无错误。

curl

	curl https://api.openai.com/v1/chat/completions \
	  -H "Content-Type: application/json" \
	  -H "Authorization: Bearer $OPENAI_API_KEY" \
	  -d '{
	    "model": "gpt-3.5-turbo-1106",
	    "response_format": { "type": "json_object" },
	    "messages": [
	      {
	        "role": "system",
	        "content": "You are a helpful assistant designed to output JSON."
	      },
	      {
	        "role": "user",
	        "content": "Who won the world series in 2020?"
	      }
	    ]
	  }'

在此示例中，响应包含一个 JSON 对象，如下所示：

	"content": "{\"winner\": \"Los Angeles Dodgers\"}"`
请注意，当模型生成参数作为函数调用的一部分时，JSON 模式始终处于启用状态。

### 可重复的输出
默认情况下，Chat Completions是不确定的（这意味着模型输出可能因请求而异）。话虽这么说，我们通过让您访问 [seed](https://platform.openai.com/docs/api-reference/chat/create#chat-create-seed) 参数和 [system_fingerprint](https://platform.openai.com/docs/api-reference/completions/object#completions/object-system_fingerprint) 响应字段来提供对确定性输出的一些控制。

要跨 API 调用接收（大部分）确定性输出，您可以：

- 将 [seed](https://platform.openai.com/docs/api-reference/chat/create#chat-create-seed )参数设置为您选择的任何整数，并在您想要确定性输出的请求中使用相同的值。
- 确保所有其他参数（如 `prompt` 或 `temperature`）在请求中完全相同。

有时，由于 OpenAI 对我们端的模型配置进行必要的更改，确定性可能会受到影响。为了帮助您跟踪这些更改，我们公开了 [system_fingerprint](https://platform.openai.com/docs/api-reference/chat/object#chat/object-system_fingerprint) 字段。如果该值不同，您可能会因我们对系统所做的更改而看到不同的输出。

#### [确定性输出](https://cookbook.openai.com/examples/deterministic_outputs_with_the_seed_parameter)
探索 OpenAI 食谱中的新种子参数
### 管理 token
语言模型以称为 token 的块的形式读取和写入文本。在英语中，token 可以短至一个字符，也可以长至一个单词（例如，`a` 或 `apple`），而在某些语言中，token 甚至可以短于一个字符，甚至长于一个单词。

例如，字符串 `"ChatGPT is great!"` 被编码为六个token：`["Chat", "G", "PT", " is", " great", "!"]`。

API 调用中的 token 总数会影响：

- 您的 API 调用费用是多少（按 `token` 支付）
- API 调用需要多长时间，因为写入更多 token 需要更多时间
- 您的 API 调用是否有效，因为 token 总数必须低于模型的最大限制（4097  个 token `gpt-3.5-turbo`）

输入和输出 token 都计入这些数量。

例如，如果您的 API 调用在消息输入中使用了 10 个token，而您在消息输出中收到了 20 个token，则您将需要支付 30 个 token。但请注意，对于某些模型，输入中的 `token` 与输出中的 `token` 的每个 `token` 的价格是不同的（有关更多信息，请参阅[定价页面](https://openai.com/pricing)）。

要查看 API 调用使用了多少 token，请检查 `usage` API 响应中的字段（例如 `response['usage']['total_tokens']`）。

聊天模型喜欢 `gpt-3.5-turbo` 和 `gpt-4` 使用 token，其方式与完成 API 中可用的模型相同，但由于其基于消息的格式，因此更难以计算对话将使用多少 token。

#### 深潜-计算聊天 API 调用的 token
下面是一个示例函数，用于计算传递给 `gpt-3.5-turbo-0613` 的消息的 token 。

消息转换为 token 的确切方式可能因模型而异。因此，当未来的模型版本发布时，该函数返回的答案可能只是近似的。ChatML [文档](https://github.com/openai/openai-python/blob/main/chatml.md) 解释了 OpenAI API 如何将消息转换为 token，并且对于编写您自己的函数可能很有用。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	def num_tokens_from_messages(messages, model="gpt-3.5-turbo-0613"):
	  """返回消息列表使用的 token 数。"""
	  try:
	      encoding = tiktoken.encoding_for_model(model)
	  except KeyError:
	      encoding = tiktoken.get_encoding("cl100k_base")
	  if model == "gpt-3.5-turbo-0613":  # 注意:未来的模型可能会偏离这一点
	      num_tokens = 0
	      for message in messages:
	          num_tokens += 4  # 每条消息遵循 <im_start> {role/name}\n{content}<im_end>\n
	          for key, value in message.items():
	              num_tokens += len(encoding.encode(value))
	              if key == "name":  #如果有名称，则省略角色
	                  num_tokens += -1  # 角色总是必需的，并且总是 1 个 token
	      num_tokens += 2  # 每个回复都以 `<im_start>assistant` 开头
	      return num_tokens
	  else:
	      raise NotImplementedError(f"""num_tokens_from_messages() is not presently implemented for model {model}.
	  See https://github.com/openai/openai-python/blob/main/chatml.md for information on how messages are converted to tokens.""")

接下来，创建一条消息并将其传递给上面定义的函数以查看 token 计数，这应该与 API 使用参数返回的值匹配：

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	messages = [
	  {"role": "system", "content": "You are a helpful, pattern-following assistant that translates corporate jargon into plain English."},
	  {"role": "system", "name":"example_user", "content": "New synergies will help drive top-line growth."},
	  {"role": "system", "name": "example_assistant", "content": "Things working well together will increase revenue."},
	  {"role": "system", "name":"example_user", "content": "Let's circle back when we have more bandwidth to touch base on opportunities for increased leverage."},
	  {"role": "system", "name": "example_assistant", "content": "Let's talk later when we're less busy about how to do better."},
	  {"role": "user", "content": "This late pivot means we don't have time to boil the ocean for the client deliverable."},
	]
	
	model = "gpt-3.5-turbo-0613"
	
	print(f"{num_tokens_from_messages(messages, model)} prompt tokens counted.")
	# 应该显示~126 total_tokens

要确认上面函数生成的数字与 API 返回的数字相同，请创建一个新的Chat Completions：

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	# 来自OpenAI API的示例 token 计数
	from openai import OpenAI
	client = OpenAI()
	
	response = client.chat.completions.create(
	  model=model,
	  messages=messages,
	  temperature=0,
	)
	
	print(f'{response.usage.prompt_tokens} prompt tokens used.')
要在不调用 API 的情况下查看文本字符串中有多少个 token，请使用 OpenAI 的 [tiktoken](https://github.com/openai/tiktoken) Python 库。[示例代码](https://cookbook.openai.com/examples/how_to_count_tokens_with_tiktoken) 可以在 OpenAI Cookbook 的关于如何使用 tiktoken 计算 token 的指南中找到。

传递到 API 的每条消息都会消耗内容、角色和其他字段中的 token 数量，以及一些用于幕后格式化的额外 token。这在未来可能会略有改变。

如果对话有太多 token 而无法满足模型的最大限制（例如，gpt-3.5-turbo 超过 4097 个token），您将必须截断、省略或以其他方式缩小文本，直到适合为止。请注意，如果从消息输入中删除消息，模型将丢失它的所有知识。

请注意，很长的对话更有可能收到不完整的回复。例如，长度为 4090 个 token 的 gpt-3.5-turbo 对话将在仅 6 个 token 后就被切断。

### 参数详情-频率和存在处罚
[Chat Completions API ](https://platform.openai.com/docs/api-reference/chat/create) 和[Legacy Completions API](https://platform.openai.com/docs/api-reference/completions)中发现的频率和存在惩罚可用于降低对 token 重复序列进行采样的可能性。它们的工作原理是通过附加贡献直接修改 logits（非标准化对数概率）。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	mu[j] -> mu[j] - c[j] * alpha_frequency - float(c[j] > 0) * alpha_presence

在哪里：

- `mu[j]`

	是第 j 个 token 的 logits
- `c[j]`

	是在当前位置之前对该 `token` 进行采样的频率
- `float(c[j] > 0)`

	如果为 1，`c[j] > 0`否则为 0
- `alpha_frequency`

	是频率惩罚系数
- `alpha_presence`

	是存在惩罚系数

正如我们所看到的，存在惩罚是一种一次性的附加贡献，适用于至少被采样一次的所有 token，而频率惩罚是与特定 token 已经被采样的频率成正比的贡献。

如果目的只是稍微减少重复样本，那么惩罚系数的合理值约为 0.1 到 1。如果目标是强烈抑制重复，那么可以将系数增加到 2，但这会明显降低样本的质量。负值可用于增加重复的可能性。

### Completions API legacy
完成 API 端点于 2023 年 7 月收到最终更新，并且具有与新的Chat Completions端点不同的界面。输入不是消息列表，而是称为 `prompt ` 的自由格式文本字符串。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	from openai import OpenAI
	client = OpenAI()
	
	response = client.completions.create(
	  model="gpt-3.5-turbo-instruct",
	  prompt="Write a tagline for an ice cream shop."
	)
请参阅完整的[API 参考文档](https://platform.openai.com/docs/api-reference/completions)以了解更多信息。

#### token 对数概率
完成 API 可以提供与每个输出 token 的最可能 token 相关​​联的有限数量的日志概率。此功能通过使用 [logprobs](https://platform.openai.com/docs/api-reference/completions/create#completions/create-logprobs) 字段来控制。在某些情况下，这对于评估模型对其输出的置信度很有用。
#### 插入文本
除了被视为前缀的标准提示之外，完成端点还支持通过提供后缀来插入文本。当编写长文本、段落之间的转换、遵循大纲或引导模型走向结局时，这种需求自然会出现。这也适用于代码，并且可用于插入到函数或文件的中间。

- 深潜-插入文本

	为了说明后缀上下文如何影响生成的文本，请考虑提示 “Today I decided to make a big change(今天我决定做出重大改变。)” 人们可以通过多种方式来完成这个句子。
	
	但如果我们现在提供故事的结局：“I’ve gotten many compliments on my new hair!(我的新头发得到了很多赞美)”，预期的完成就变得清晰了。

	I went to college at Boston University. After getting my degree, I decided to make a change. `A big change!`
	
		我在波士顿大学上大学。获得学位后，我决定做出改变。一个大变化！
	
	`I packed my bags and moved to the west coast of the United States.`
	
		我收拾好行李，搬到了美国西海岸。
	
	Now, I can’t get enough of the Pacific Ocean!

		现在，我对太平洋还不够了解！

通过为模型提供额外的上下文，它可以变得更加可操纵。然而，这对于模型来说是一项更具约束性和挑战性的任务。为了获得最佳结果，我们建议如下：

- `使用 max_tokens > 256`

	该模型更擅长插入更长的补全。如果 `max_tokens `太小，模型可能会在连接到后缀之前被切断。请注意，即使使用较大的 `max_tokens`，您也只需按生成的 token 数量付费。

- `更喜欢 finish_reason=“stop”`。

	当模型到达自然停止点或用户提供的停止序列时，它将把 `finish_reason` 设置为“stop”。这表明该模型已成功连接到后缀，并且是完成质量的良好信号。当使用` n > 1` 或重采样时，这对于在几个完成之间进行选择尤其相关（请参阅下一点）。

- `重新采样 3-5 次`。

	虽然几乎所有补全都连接到前缀，但在更困难的情况下，模型可能很难连接后缀。我们发现，在这种情况下，重采样 3 或 5 次（或使用 best_of 的 k=3,5 ）并选择 “stop” 作为其 `finish_reason` 的样本可能是一种有效的方法。重新采样时，您通常需要更高的温度来增加多样性。

	注意：如果所有返回的样本都具有 `finish_reason == "length"`，则很可能 `max_tokens` 太小并且模型在成功连接提示和后缀之前就用完了 token。考虑在重采样之前增加 `max_tokens`。

- `尝试提供更多线索`

	在某些情况下，为了更好地帮助模型的生成，您可以通过给出一些模式示例来提供线索，模型可以遵循这些模式来决定自然停止的位置。

	例子1
	
		如何制作美味的热巧克力：
		
		1.将水烧开
		2. 将热巧克力放入杯子中
		3. 将沸水倒入杯子中
		4. 享用热巧克力
	例子2
	
		1 狗是忠诚的动物。
		2 狮子是凶猛的动物。
		3 海豚是顽皮的动物。
		4 马是雄伟的动物。

#### 完工响应格式
完成 API 响应示例如下所示：

	{
	  "choices": [
	    {
	      "finish_reason": "length",
	      "index": 0,
	      "logprobs": null,
	      "text": "\n\n\"Let Your Sweet Tooth Run Wild at Our Creamy Ice Cream Shack"
	    }
	  ],
	  "created": 1683130927,
	  "id": "cmpl-7C9Wxi9Du4j1lQjdjhxBlO22M61LD",
	  "model": "gpt-3.5-turbo-instruct",
	  "object": "text_completion",
	  "usage": {
	    "completion_tokens": 16,
	    "prompt_tokens": 10,
	    "total_tokens": 26
	  }
	}
在 Python 中，可以使用 `response['choices'][0]['text']` 代理提取输出。

响应格式与Chat Completions API 的响应格式类似，但还包括可选字段logprobs。

### Chat Completions vs. Completions
通过使用单个用户消息构造请求，可以使Chat Completions格式与完成格式类似。例如，可以通过以下完成提示将英语翻译成法语：

	Translate the following English text to French: "{text}"
等效的聊天提示是：

	[{"role": "user", "content": 'Translate the following English text to French: "{text}"'}]
同样，完成 API 可用于通过[相应地](https://platform.openai.com/playground/p/default-chat?model=gpt-3.5-turbo-instruct)格式化输入来模拟用户和 assistant（助理）之间的聊天。

这些 API 之间的区别在于每个 API 中可用的底层模型。 Chat Completions API 是我们功能最强大的模型 (`gpt-4`) 和最具成本效益的模型 (`gpt-3.5-turbo`)的接口。

#### 我应该使用哪种型号？
我们通常建议您使用 `gpt-4` 或 `gpt-3.5-turbo`。您应该使用哪一个取决于您使用模型执行的任务的复杂性。

- `gpt-4` 通常在广泛的[评估](https://arxiv.org/abs/2303.08774)中表现更好。特别是，`gpt-4` 能够更仔细地遵循复杂的指令。
- 相比之下，`gpt-3.5-turbo` 更有可能只遵循复杂的多部分指令的一部分。
- 编造信息的 `gpt-4` 可能性更小，这种行为被称为“幻觉”。
- `gpt-4` 还具有更大的上下文窗口，最大大小为 8,192 个 token
- `gpt-3.5-turbo`  最大是 4096
- 但是，gpt-3.5-turbo 返回延迟较低的输出和要低得多的每个 token 的成本。

我们建议在[游乐场](https://platform.openai.com/playground?mode=chat)进行试验，以调查哪些模型可以为您的使用提供最佳的性价比权衡。常见的设计模式是使用几种不同的查询类型，每种查询类型都会分派到适合处理它们的模型。

#### Prompt engineering
了解使用 OpenAI 模型的最佳实践可以对应用程序性能产生重大影响。每种故障模式以及解决或纠正这些故障模式的方法并不总是直观的。有一个与使用语言模型相关的整个领域，被称为“提示工程”，但随着该领域的发展，其范围已经超出了仅仅将提示工程设计为使用模型查询作为组件的工程系统的范围。要了解更多信息，请阅读我们的[提示词工程指南](https://platform.openai.com/docs/guides/prompt-engineering)，其中涵盖了改进模型推理、减少模型幻觉可能性等的方法。您还可以在 [OpenAI Cookbook](https://cookbook.openai.com/) 中找到许多有用的资源，包括代码示例。
### 常问问题
#### 温度参数该如何设置？
- 较低的温度值会产生更一致的输出（例如 0.2）
- 而较高的值会产生更加多样化和创造性的结果（例如 1.0）。

根据您的特定应用的一致性和创造性之间所需的权衡选择温度值。温度范围为 0 至 2。

#### 最新型号可以进行微调吗？
是的，对于某些人来说。目前，您只能微调 `gpt-3.5-turbo` 和我们更新的基本模型（`babbage-002`和`davinci-002`）。有关如何使用微调模型的更多详细信息，请参阅[微调指南](https://platform.openai.com/docs/guides/fine-tuning)。
#### 您是否存储传递到 API 的数据？
自 2023 年 3 月 1 日起，我们将您的 API 数据保留 30 天，但不再使用您通过 API 发送的数据来改进我们的模型。了解更多信息，请参阅我们的[数据使用政策](https://openai.com/policies/usage-policies)。一些端点提供[零保留](https://platform.openai.com/docs/models/default-usage-policies-by-endpoint)。
#### 如何使我的应用程序更加安全？
如果您想为 Chat API 的输出添加审核层，您可以遵循我们的[审核指南](https://platform.openai.com/docs/guides/moderation)，以防止显示违反 OpenAI 使用政策的内容。
#### 我应该使用 ChatGPT 还是 API？
[ChatGPT](https://chat.openai.com/) 为 OpenAI API 中的模型提供聊天界面以及一系列内置功能，例如集成浏览、代码执行、插件等。相比之下，使用 OpenAI 的 API 提供了更大的灵活性。