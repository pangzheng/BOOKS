# OpenAI`4.0T`官方文档-能力-2 函数调用
了解如何将大型语言模型连接到外部工具。
## 介绍
在 API 调用中，您可以描述函数并让模型智能地选择输出包含调用一个或多个函数的参数的 JSON 对象。

Chat Completions API 不会调用该函数；相反，模型会生成 JSON，您可以使用它来调用代码中的函数。

最新的模型 (`gpt-3.5-turbo-1106` 和 `gpt-4-1106-preview`) 经过训练，可以检测何时应调用函数（取决于输入），并使用比以前的模型更紧密地遵循函数签名的 JSON 进行响应。这种能力也带来了潜在的风险。

	我们强烈建议在代表用户采取影响世界的行动（发送电子邮件、在线发布内容、购买等）之前构建用户确认流程。

本指南重点介绍聊天完成 API 的函数调用，有关助手 API 中函数调用的详细信息，请参阅[助手工具页面](https://platform.openai.com/docs/assistants/tools)。

### 常见用例
函数调用使您能够更可靠地从模型中获取结构化数据。

例如，您可以：

- 创建通过调用外部 API 来回答问题的助手（例如 ChatGPT 插件）

	- 例如定义函数，如 `send_email(to: string, body: string)`,
	- 或 `get_current_weather(location: string, unit: 'celsius' | 'fahrenheit')`
- 将自然语言转换为 API 调用

	例如转换 “谁是我的主要客户？”  `get_customers(min_revenue: int, created_before: string, limit: int)` 调用您的内部 API
- 从文本中提取结构化数据

	例如定义一个名为 `extract_data(name: string, birthday: string)` 的函数或者 `sql_query(query: string)`
- ...以及更多！

函数调用的基本步骤顺序如下：

- 使用用户查询和[函数参数](https://platform.openai.com/docs/api-reference/chat/create#chat/create-functions) 中定义的一组函数来调用模型。
- 模型可以选择调用一个或多个函数；如果是这样，内容将是一个符合您的自定义架构的字符串化 JSON 对象（注意：模型可能会产生参数）。
- 在代码中将字符串解析为 JSON，并使用提供的参数（如果存在）调用函数。
- 通过将函数响应作为新消息附加来再次调用模型，并让模型将结果汇总返回给用户。

### 支持机型
并非所有模型版本都使用函数调用数据进行训练。

以下型号支持函数调用：

- gpt-4
- gpt-4-1106-preview
- gpt-4-0613
- gpt-3.5-turbo
- gpt-3.5-turbo-1106
- gpt-3.5-turbo-0613

此外，以下型号支持并行函数调用：

- gpt-4-1106-preview
- gpt-3.5-turbo-1106

### 并行函数调用
并行函数调用是模型同时执行多个函数调用的能力，允许并行解决这些函数调用的效果和结果。如果函数需要很长时间，这尤其有用并且可以减少 API 的往返次数。

例如，模型可能会调用函数来同时获取 3 个不同位置的天气，这将导致一条消息在数组中包含 3 个函数调用`tool_calls`，每个函数调用都有一个id. 要响应这些函数调用，请向对话中添加 3 条新消息，每条消息都包含一个函数调用的结果，并 `tool_call_id` 从 `id`引用 `tool_calls`。

在这个例子中，我们定义了一个函数 `get_current_weather`。模型多次调用该函数，并将函数响应发送回模型后，我们让它决定下一步。它回复了一条面向用户的消息，告诉用户旧金山、东京和巴黎的气温。根据查询，它可能会选择再次调用函数。

如果您想强制模型调用特定函数，可以通过设置 [tool_choice](https://platform.openai.com/docs/api-reference/chat/create#chat-create-tool_choice) 特定函数名称来实现。您还可以通过设置强制模型生成面向用户的消息 `tool_choice: "none"`。请注意，默认行为 ( `tool_choice: "auto"`) 是让模型自行决定是否调用函数以及如果调用哪个函数。

### DEMO
在一个响应中调用多个函数的示例

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	from openai import OpenAI
	import json
	
	client = OpenAI()
	
	# 示例虚拟函数硬编码以返回相同的天气
	# 在生产环境中，这可以是您的后端API或外部API
	def get_current_weather(location, unit="fahrenheit"):
	    """获取给定位置的当前天气"""
	    if "tokyo" in location.lower():
	        return json.dumps({"location": "Tokyo", "temperature": "10", "unit": "celsius"})
	    elif "san francisco" in location.lower():
	        return json.dumps({"location": "San Francisco", "temperature": "72", "unit": "fahrenheit"})
	    elif "paris" in location.lower():
	        return json.dumps({"location": "Paris", "temperature": "22", "unit": "celsius"})
	    else:
	        return json.dumps({"location": location, "temperature": "unknown"})
	
	def run_conversation():
	    # 第1步:将对话和可用功能发送给模型
	    messages = [{"role": "user", "content": "What's the weather like in San Francisco, Tokyo, and Paris?"}]
	    tools = [
	        {
	            "type": "function",
	            "function": {
	                "name": "get_current_weather",
	                "description": "Get the current weather in a given location",
	                "parameters": {
	                    "type": "object",
	                    "properties": {
	                        "location": {
	                            "type": "string",
	                            "description": "The city and state, e.g. San Francisco, CA",
	                        },
	                        "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
	                    },
	                    "required": ["location"],
	                },
	            },
	        }
	    ]
	    response = client.chat.completions.create(
	        model="gpt-3.5-turbo-1106",
	        messages=messages,
	        tools=tools,
	        tool_choice="auto",  # Auto 是默认值，但我们会明确说明
	    )
	    response_message = response.choices[0].message
	    tool_calls = response_message.tool_calls
	    
	    # 步骤2: 检查模型是否需要调用函数
	    if tool_calls:
	        # 步骤3:调用函数
	        # 注意:JSON响应可能并不总是有效的;一定要处理错误
	        available_functions = {
	            "get_current_weather": get_current_weather,
	        }  # 本例中只有一个函数，但可以有多个
	        messages.append(response_message)  # 用助理的回答延长谈话时间
	       
	        # 步骤4:将每个函数调用和函数响应的信息发送给模型
	        for tool_call in tool_calls:
	            function_name = tool_call.function.name
	            function_to_call = available_functions[function_name]
	            function_args = json.loads(tool_call.function.arguments)
	            function_response = function_to_call(
	                location=function_args.get("location"),
	                unit=function_args.get("unit"),
	            )
	            messages.append(
	                {
	                    "tool_call_id": tool_call.id,
	                    "role": "tool",
	                    "name": function_name,
	                    "content": function_response,
	                }
	            )  # 用函数响应扩展会话
	        second_response = client.chat.completions.create(
	            model="gpt-3.5-turbo-1106",
	            messages=messages,
	        )  # 从可以看到函数响应的模型中获取一个新的响应
	        return second_response
	print(run_conversation())

您可以在 OpenAI Cookbook 中找到更多函数调用示例：

- [函数调用](https://cookbook.openai.com/examples/how_to_call_functions_with_chat_models)

	从更多演示函数调用的示例中学习

### Token
在底层，函数按照模型训练过的语法注入到系统消息中。这意味着函数会根据模型的上下文限制进行计数，并作为输入 Token 进行计费。如果遇到上下文限制，我们建议限制函数的数量或为函数参数提供的文档的长度。

如果定义了许多函数，还可以使用[微调](https://platform.openai.com/docs/guides/fine-tuning/fine-tuning-examples)来减少使用的 Token 数量。