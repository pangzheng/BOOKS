# OpenAI`4.0T`官方文档-assistants(助手)API-工具
让助手可以访问 OpenAI 托管的工具，例如代码解释器和知识检索，或使用函数调用构建您自己的工具。使用 OpenAI 托管的工具需要支付额外费用 - 请访问我们的[帮助中心文章](https://help.openai.com/en/articles/8550641-assistants-api)，了解有关这些工具如何定价的更多信息。

	Assistants API 处于测试阶段，我们正在积极努力添加更多功能。在我们的开发者论坛中分享您的反馈！
## 代码解释器
代码解释器允许 Assistants API 在沙盒执行环境中编写和运行 Python 代码。该工具可以处理具有不同数据和格式的文件，并生成具有数据和图形图像的文件。代码解释器允许您的助手迭代运行代码以解决具有挑战性的代码和数学问题。当 Assistant 编写无法运行的代码时，它可以通过尝试运行不同的代码来迭代此代码，直到代码执行成功。
### 启用代码解释器
 `code_interpreter`传入`tools` Assistant 对象的参数以启用代码解释器

	curl https://api.openai.com/v1/assistants \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "instructions": "You are a personal math tutor. When asked a math question, write and run code to answer the question.",
	    "tools": [
	      { "type": "code_interpreter" }
	    ],
	    "model": "gpt-4-1106-preview"
	  }'
然后，模型根据用户请求的性质决定何时在运行中调用代码解释器。这种行为可以通过助手中的提示来促进 `instructions`（例如，“编写代码来解决这个问题”）。
### 将文件传递给代码解释器
代码解释器可以解析文件中的数据。当您想要向助手提供大量数据或允许用户上传自己的文件进行分析时，这非常有用。

使用此助手的所有运行都可以访问在助手级别传递的文件：

	# 上传一个带有“助手”目的的文件
	curl https://api.openai.com/v1/files \
	  -H "Authorization: Bearer $OPENAI_API_KEY" \
	  -F purpose="assistants" \
	  -F file="@mydata.csv"
	
	# 使用文件ID创建助手
	curl https://api.openai.com/v1/assistants \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "instructions": "You are a personal math tutor. When asked a math question, write and run code to answer the question.",
	    "tools": [{"type": "code_interpreter"}],
	    "model": "gpt-4-1106-preview",
	    "file_ids": ["file_123abc456"]
	  }'
文件也可以在线程级别传递。这些文件只能在特定线程中访问。使用文件上传端点[上传文件](https://platform.openai.com/docs/api-reference/files/create)，然后将文件 ID 作为消息创建请求的一部分传递：

	curl https://api.openai.com/v1/threads/THREAD_ID/messages \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "role": "user",
	    "content": "I need to solve the equation `3x + 11 = 14`. Can you help me?",
	    "file_ids": ["file_123abc456"]
	  }'
文件的最大大小为 512 MB。代码解释器支持多种文件格式，包括`.csv、.pdf、.json`等。有关支持的文件扩展名（及其相应的 MIME 类型）的更多详细信息，请参阅下面的[支持的文件](https://platform.openai.com/docs/assistants/tools/supported-files)部分。
### 读取代码解释器生成的图像和文件
API 中的代码解释器还输出文件，例如生成图像图表、CSV 和 PDF。生成的文件有两种类型：

- 图片
- 数据文件（例如csv包含助手生成的数据的文件）


当代码解释器生成图像时，您可以在助手消息响应 `file_id` 字段中查找并下载该文件：

	{
	    "id": "msg_OHGpsFRGFYmz69MM1u8KYCwf",
	    "object": "thread.message",
	    "created_at": 1698964262,
	    "thread_id": "thread_uqorHcTs46BZhYMyPn6Mg5gW",
	    "role": "assistant",
	    "content": [
	    {
	      "type": "image_file",
	      "image_file": {
	        "file_id": "file-WsgZPYWAauPuW4uvcgNUGcb"
	      }
	    }
	  ]
	  # ...
	}
然后可以通过将文件 ID 传递给 Files API 来下载文件内容：

	curl https://api.openai.com/v1/files/file-abc123/content   -H "Authorization: Bearer $OPENAI_API_KEY"   --output image.png
当代码解释器引用文件路径（例如，“下载此 csv 文件”）时，文件路径将作为注释列出。您可以将这些注释转换为下载文件的链接：

	{
	  "id": "msg_3jyIh3DgunZSNMCOORflDyih",
	  "object": "thread.message",
	  "created_at": 1699073585,
	  "thread_id": "thread_ZRvNTPOoYVGssUZr3G8cRRzE",
	  "role": "assistant",
	  "content": [
	    {
	      "type": "text",
	      "text": {
	        "value": "The rows of the CSV file have been shuffled and saved to a new CSV file. You can download the shuffled CSV file from the following link:\n\n[Download Shuffled CSV File](sandbox:/mnt/data/shuffled_file.csv)",
	        "annotations": [
	          {
	            "type": "file_path",
	            "text": "sandbox:/mnt/data/shuffled_file.csv",
	            "start_index": 167,
	            "end_index": 202,
	            "file_path": {
	              "file_id": "file-oSgJAzAnnQkVB3u7yCoE9CBe"
	            }
	          }
	          ...
### Code Interpreter 的输入输出日志
通过列出调用 Code Interpreter 的 Run 的步骤，您可以检查 Code Interpreter 的代码`input` 和 `outputs`日志：

	curl https://api.openai.com/v1/threads/THREAD_ID/runs/RUN_ID/steps \
	  -u :$OPENAI_API_KEY
响应

	{
	  "object": "list",
	  "data": [
	    {
	      "id": "step_DQfPq3JPu8hRKW0ctAraWC9s",
	      "object": "thread.run.step",
	      "type": "tool_calls",
	      "run_id": "run_kme4a442kme4a442",
	      "thread_id": "thread_34p0sfdas0823smfv",
	      "status": "completed",
	      "step_details": {
	        "type": "tool_calls",
	        "tool_calls": [
	          {
	            "type": "code",
	            "code": {
	              "input": "# Calculating 2 + 2\nresult = 2 + 2\nresult",
	              "outputs": [
	                {
	                  "type": "logs",
	                  "logs": "4"
	                }
	                        ...
	 }
### 知识检索
检索可以利用模型外部的知识来增强助手，例如专有产品信息或用户提供的文档。文件上传并传递给助手后，OpenAI 将自动对文档进行分块、索引和存储嵌入，并实施矢量搜索以检索相关内容来回答用户查询。
#### 启用检索
`tools`的 `Retrieval retrieval` 传入 Assistant 的参数以启用 

	curl https://api.openai.com/v1/assistants \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "instructions": "You are a customer support chatbot. Use your knowledge base to best respond to customer queries.",
	    "tools": [{"type": "retrieval"}],
	    "model": "gpt-4-1106-preview"
	  }'
### 怎么运行的
然后，模型根据用户消息决定何时检索内容。Assistants API 会自动在两种检索技术之间进行选择：

- 它要么在短文档的提示中传递文件内容，要么
- 对较长的文档执行矢量搜索

目前，检索通过将所有相关内容添加到模型调用的上下文中来优化质量。我们计划引入其他检索策略，使开发人员能够在检索质量和模型使用成本之间选择不同的权衡。

### 上传文件以供检索
与代码解释器类似，文件可以在助手级别或线程级别传递
	
	# 上传一个带有“助手”目的的文件
	curl https://api.openai.com/v1/files \
	  -H "Authorization: Bearer $OPENAI_API_KEY" \
	  -F purpose="assistants" \
	  -F file="@knowledge.pdf"
	
	# 将文件添加到助手中
	curl https://api.openai.com/v1/assistants \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "instructions": "You are a customer chatbot. Use your knowledge base to best respond to customer queries.",
	    "tools": [{"type": "retrieval"}],
	    "model": "gpt-4-1106-preview"
	    "file_ids": ["file_123abc456"]
	  }'
文件也可以添加到线程中的消息中。这些文件只能在该特定线程内访问。上传文件后，您可以在创建消息时传递该文件的 ID：
	
	curl https://api.openai.com/v1/threads/thread_34p0sfdas0823smfv/messages \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "role": "user",
	    "content": "I can't find in the PDF manual how to turn off this device.",
	    "file_ids": ["file_123abc456"]
	  }'
最大文件大小为 512MB。检索支持多种文件格式，包括 `.pdf、.doc、.md`等。有关支持的文件扩展名（及其相应的 MIME 类型）的更多详细信息，请参阅下面的[支持的文件](https://platform.openai.com/docs/assistants/tools/supported-files)部分。

### 删除文件
要从助手中删除文件，您可以将该文件与助手分离：

	curl https://api.openai.com/v1/assistants/ASSISTANT_ID/files/FILE_ID \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -X DELETE
从助手中分离文件也会从检索索引中删除该文件。

### 文件引用
当代码解释器在消息中输出文件路径时，您可以使用该 `annotations` 字段将它们转换为相应的文件下载。有关如何执行此操作的示例，请参阅注释部分。

	{
	    "id": "msg_3jyIh3DgunZSNMCOORflDyih",
	    "object": "thread.message",
	    "created_at": 1699073585,
	    "thread_id": "thread_ZRvNTPOoYVGssUZr3G8cRRzE",
	    "role": "assistant",
	    "content": [
	      {
	        "type": "text",
	        "text": {
	          "value": "The rows of the CSV file have been shuffled and saved to a new CSV file. You can download the shuffled CSV file from the following link:\n\n[Download Shuffled CSV File](sandbox:/mnt/data/shuffled_file.csv)",
	          "annotations": [
	            {
	              "type": "file_path",
	              "text": "sandbox:/mnt/data/shuffled_file.csv",
	              "start_index": 167,
	              "end_index": 202,
	              "file_path": {
	                "file_id": "file-oSgJAzAnnQkVB3u7yCoE9CBe"
	              }
	            }
	          ]
	        }
	      }
	    ],
	    "file_ids": [
	      "file-oSgJAzAnnQkVB3u7yCoE9CBe"
	    ],
	        ...
	  },

## 函数调用
与 Chat Completions API 类似，助手 API 支持函数调用。函数调用允许您向助手描述函数，并让它智能地返回需要调用的函数及其参数。当 Assistants API 调用函数时，它会在 Run 期间暂停执行，您可以提供函数回调的结果以继续 Run 执行。
### 定义函数
首先，在创建助手时定义您的功能：

	curl https://api.openai.com/v1/assistants \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "instructions": "You are a weather bot. Use the provided functions to answer questions.",
	    "tools": [{
	      "type": "function",
	      "function": {
	        "name": "getCurrentWeather",
	        "description": "Get the weather in location",
	        "parameters": {
	          "type": "object",
	          "properties": {
	            "location": {"type": "string", "description": "The city and state e.g. San Francisco, CA"},
	            "unit": {"type": "string", "enum": ["c", "f"]}
	          },
	          "required": ["location"]
	        }
	      }	
	    },
	    {
	      "type": "function",
	      "function": {
	        "name": "getNickname",
	        "description": "Get the nickname of a city",
	        "parameters": {
	          "type": "object",
	          "properties": {
	            "location": {"type": "string", "description": "The city and state e.g. San Francisco, CA"}
	          },
	          "required": ["location"]
	        }
	      }	
	    }],
	    "model": "gpt-4-1106-preview"
	  }'
### 读取助手调用的函数
当您使用[触发该功能的用户消息启动](https://platform.openai.com/docs/api-reference/runs/createRun)运行时，运行将进入一个 `pending` 状态。处理完成后，运行将进入一种·requires_action· 状态，您可以通过[检索 Run](https://platform.openai.com/docs/api-reference/runs/getRun) 来验证该状态。该模型可以使用[并行函数调用](https://platform.openai.com/docs/guides/function-calling/parallel-function-calling)来提供多个函数同时调用：

	{
	  "id": "run_3HV7rrQsagiqZmYynKwEdcxS",
	  "object": "thread.run",
	  "assistant_id": "asst_rEEOF3OGMan2ChvEALwTQakP",
	  "thread_id": "thread_dXgWKGf8Cb7md8p0wKiMDGKc",
	  "status": "requires_action",
	  "required_action": {
	    "type": "submit_tool_outputs",
	    "submit_tool_outputs": {
	      "tool_calls": [
	        {
	          "id": "call_Vt5AqcWr8QsRTNGv4cDIpsmA",
	          "type": "function",
	          "function": {
	            "name": "getCurrentWeather",
	            "arguments": "{\"location\":\"San Francisco\"}"
	          }
	        },
	        {
	          "id": "call_45y0df8230430n34f8saa",
	          "type": "function",
	          "function": {
	            "name": "getNickname",
	            "arguments": "{\"location\":\"Los Angeles\"}"
	          }
	        }
	      ]
	    }
	  },
	...
### 提交函数输出
然后，您可以通过[提交您调用的函数的工具输出](https://platform.openai.com/docs/api-reference/runs/submitToolOutputs)来完成运行。传递上面对象 `tool_call_id` 中的`required_action` 引用以将输出匹配到每个函数调用。
	
	curl https://api.openai.com/v1/threads/thread_34p0sfdas0823smfv/runs/run_123/submit_tool_outputs \
	  -u :$OPENAI_API_KEY \
	  -H 'Content-Type: application/json' \
	  -H 'OpenAI-Beta: assistants=v1' \
	  -d '{
	    "tool_outputs": [{
	      "tool_call_id": "call_Vt5AqcWr8QsRTNGv4cDIpsmA",
	      "output": "{"temperature": "22", "unit": "celsius"}"
	    }, {
	      "tool_call_id": "call_45y0df8230430n34f8saa",
	      "output": "{"nickname": "LA"}"
	    }]
	  }'
提交输出后，运行将进入 `queued` 继续执行之前的状态。

## 支持的文件
对于text/MIME 类型，编码必须是utf-8、utf-16或 之一ascii。

文件格式|MIME类型|代码解释器|恢复
---|---|---|---|
.c|text/x-c|支持|支持		
.cpp|text/x-c++|支持|支持		
.csv|application/csv|支持|		
.docx|application/vnd.openxmlformats-officedocument.wordprocessingml.document|支持|支持
.html|text/html|支持|支持
.java|text/x-java|支持|支持
.json|application/json|支持|支持		
.md|text/markdown|支持|支持		
.pdf|application/pdf|支持|支持		
.php|text/x-php|支持|支持		
.pptx|application/vnd.openxmlformats-officedocument.presentationml.presentation|支持|支持
.py	|text/x-python|支持|支持		
.py	|text/x-script.python|支持|支持		
.rb	|text/x-ruby|支持|支持		
.tex|text/x-tex|支持|支持		
.txt|text/plain|支持|支持		
.css|text/css|支持|			
.jpeg|image/jpeg	|支持|		
.jpg|image/jpeg|支持|			
.js|	text/javascript|支持|			
.gif|image/gif|支持|			
.png|image/png	|支持|		
.tar|application/x-tar	|支持|		
.ts|	application/typescript|支持|		
.xlsx|application/vnd.openxmlformats-officedocument.spreadsheetml.sheet|支持|
.xml|application/xml or "text/xml"|支持|	
.zip|application/zip|支持|			
