# OpenAI`4.0T`官方文档-assistants(助手)API-概述
Assistants API 允许您在自己的应用程序中构建 AI 助手。助手有指令，可以利用模型、工具和知识来响应用户查询。

Assistants API 目前支持三种类型的[工具](https://platform.openai.com/docs/assistants/tools)：

- 代码解释器
- 检索
- 函数调用

未来，我们计划发布更多 OpenAI 构建的工具并允许您在我们的平台上提供自己的工具。

您可以使用 [Assistants Playground](https://platform.openai.com/playground?mode=assistant) 或通过构建本指南中概述的分步集成来探索 Assistants API 的功能。从较高层面来看，Assistants API 的典型集成具有以下流程：

- 通过定义其自定义指令并选择模型来在 API 中创建[助手](https://platform.openai.com/docs/api-reference/assistants/createAssistant)。如果有帮助，请启用代码解释器、检索和函数调用等工具。
- 当用户开始对话时创建一个[线程](https://platform.openai.com/docs/api-reference/threads)。
- 当用户提问时将[消息](https://platform.openai.com/docs/api-reference/messages)添加到线程中。
- 在线程上[运行](https://platform.openai.com/docs/api-reference/runs)助手以触发响应。这会自动调用相关工具。

Assistants API 处于测试阶段，我们正在积极努力添加更多功能。在我们的[开发者论坛](https://community.openai.com/)中分享您的反馈！

本入门指南逐步介绍了创建和运行使用[代码解释器](https://platform.openai.com/docs/assistants/tools/code-interpreter)的助手的关键步骤。

## 助理游乐场
除了 [Assistants API](https://platform.openai.com/docs/api-reference/assistants) 之外，我们还提供 [Assistants Playground](https://platform.openai.com/playground?mode=assistant)（需要登录）。Playground 是探索 Assistants API 功能并学习如何在不编写任何代码的情况下构建自己的 Assistant 的好方法。
### 第 1 步：创建助手
助手代表一个实体，可以配置为使用多个参数响应用户的消息，例如：

- 说明：助理和模特应如何表现或回应
- 模型：您可以指定任何 GPT-3.5 或 GPT-4 模型，包括微调模型。检索工具需要 `gpt-3.5-turbo-1106` 和 `gpt-4-1106-preview` 模型。
- 工具：API 支持由 OpenAI 构建和托管的代码解释器和检索。
- 函数：API 允许您定义自定义函数签名，其行为与我们的[函数调用](https://platform.openai.com/docs/guides/function-calling)功能类似。

在此示例中，我们 [创建一个助手](https://platform.openai.com/docs/api-reference/assistants/createAssistant)，它是个人数学导师，并启用了代码解释器工具：

调用 Assistants API 要求您传递 beta HTTP 标头。如果您使用 OpenAI 的官方 Python 或 Node.js SDK，则会自动处理此问题。

	OpenAI-Beta: assistants=v1
程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	assistant = client.beta.assistants.create(
	    name="Math Tutor",
	    instructions="You are a personal math tutor. Write and run code to answer math questions.",
	    tools=[{"type": "code_interpreter"}],
	    model="gpt-4-1106-preview"
	)

### 第 2 步：创建线程
一个线程代表一个对话。我们建议在用户发起对话后立即为每个用户[创建一个线程](https://platform.openai.com/docs/api-reference/threads/createThread)。通过[创建 Messages](https://platform.openai.com/docs/api-reference/messages/createMessage) 在此线程中传递任何特定于用户的上下文和文件。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	thread = client.beta.threads.create()
线程没有大小限制。您可以向线程传递任意数量的消息。API 将使用截断等相关优化技术来确保对模型的请求适合最大上下文窗口。
### 第 3 步：向线程添加消息
消息包含文本，还可以选择允许用户上传的任何[文件](https://platform.openai.com/docs/assistants/tools/supported-files)。消息需要[添加到特定线程](https://platform.openai.com/docs/api-reference/messages/createMessage) 中。目前不支持通过消息对象[添加图像](https://platform.openai.com/docs/guides/vision)，例如使用 GPT-4 和 Vision 在聊天完成中添加图像，但我们计划在未来几个月内添加对它们的支持。您仍然可以上传图像并通过检索对其进行处理。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	message = client.beta.threads.messages.create(
	    thread_id=thread.id,
	    role="user",
	    content="I need to solve the equation `3x + 11 = 14`. Can you help me?"
	)
现在，如果您在[ Thread 中列出消息](https://platform.openai.com/docs/api-reference/messages/listMessages)，您将看到该消息在创建时添加到线程中：

	{
	  "object": "list",
	  "data": [
	    {
	      "created_at": 1696995451,
	      "id": "msg_4rb1Skx3XgQZEe4PHVRFQhr0",
	      "object": "thread.message",
	      "thread_id": "thread_34p0sfdas0823smfv",
	      "role": "user",
	      "content": [{
	        "type": "text",
	        "text": {
	          "value": "I need to solve the equation `3x + 11 = 14`. Can you help me?",
	          "annotations": []
	        }
	          }],
	        ...
### 第4步：运行助手
为了让助手响应用户消息，您需要创建一个 [Run](https://platform.openai.com/docs/api-reference/runs/createRun)。这使得助手读取线程并决定是调用工具（如果启用了）还是简单地使用模型来最好地回答查询。随着运行的进行，助手将消息附加到带有`role="assistant".`

您可以[选择在创建运行时](https://platform.openai.com/docs/api-reference/runs/createRun#runs-createrun-instructions)向助手传递其他指令，但请注意，这些指令会覆盖助手的默认指令：

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	run = client.beta.threads.runs.create(
	  thread_id=thread.id,
	  assistant_id=assistant.id,
	  instructions="Please address the user as Jane Doe. The user has a premium account."
	)

### 第5步：检查运行状态
这会在一个 [`queued `status](https://platform.openai.com/docs/api-reference/runs/object#runs/object-status)中创建 Run 。您可以定期检索运行以检查其状态，看看它是否已移至 `completed`.

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	run = client.beta.threads.runs.retrieve(
	  thread_id=thread.id,
	  run_id=run.id
	)
### 第 6 步：显示助理的响应
运行完成后，您可以[列出助手添加到线程的消息](https://platform.openai.com/docs/api-reference/messages/listMessages)。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	messages = client.beta.threads.messages.list(
	  thread_id=thread.id
	)
最后，将它们显示给用户！在此运行期间，助理向线程添加了两条新消息。下面是一个示例：

角色|内容
---|---
user	|我需要解方程3x + 11 = 14。你能帮助我吗？
assistant|当然，简·多伊。(3x + 11 = 14)要求解的方程(x)，您需要(x)在方程的一侧进行隔离。您可以按照以下方法执行此操作：
||1 等式两边同时减去11，得到(3x = 3)。
||2 然后，两边除以 3 即可解出(x)。
||3 (x)让我为您计算一下价值。
assistant|方程的解(3x + 11 = 14)为(x = 1)。

如果您想探索或显示助手及其工具的内部工作原理，您还可以检索此运行的运行步骤。