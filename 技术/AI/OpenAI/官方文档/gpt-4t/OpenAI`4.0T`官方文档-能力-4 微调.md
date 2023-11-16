# OpenAI`4.0T`官方文档-能力-4 微调
了解如何为您的应用程序定制模型。

## 介绍
本指南面向新 OpenAI 微调 API 的用户。如果您是旧版微调用户，请参阅我们的[旧版微调指南](https://platform.openai.com/docs/guides/legacy-fine-tuning)。

微调可让您通过 API 提供以下功能，从而更充分地利用可用模型：

- 比提示更高质量的结果
- 能够训练超出提示范围的示例
- 由于提示较短而节省了token
- 更低的延迟请求

OpenAI 的文本生成模型已经过大量文本的预训练。为了有效地使用模型，我们在提示中包含说明，有时还包含几个示例。使用演示来展示如何执行任务通常称为“小样本学习”。

微调通过训练超出提示范围的更多示例来改进小样本学习，让您在大量任务上取得更好的结果。`一旦模型经过微调，您就不需要在提示中提供那么多示例`。这可以节省成本并实现更低延迟的请求。

在较高层面上，微调涉及以下步骤：

- 准备并上传训练数据
- 训练新的微调模型
- 评估结果并根据需要返回步骤 1
- 使用您的微调模型

请访问我们的[定价页面](https://openai.com/api/pricing)，了解有关微调模型训练和使用如何计费的更多信息。

### 哪些模型可以进行微调？
	GPT-4 的微调是在一个实验性访问计划中 - 合格的用户可以请求访问。
目前可对以下型号进行微调：

- gpt-3.5-turbo-1106（受到推崇的）
- gpt-3.5-turbo-0613
- babbage-002
- davinci-002
- gpt-4-0613（实验性 — 符合条件的用户将在[微调 UI](https://platform.openai.com/finetune)中看到请求访问权限的选项）

您还可以对经过微调的模型进行微调，如果您获取其他数据并且不想重复之前的训练步骤，则该模型非常有用。

我们希望 `gpt-3.5-turbo` 在结果和易用性方面成为大多数用户的正确模型，除非您要迁移旧的微调模型。

## 何时使用微调
微调 OpenAI 文本生成模型可以使它们更好地适应特定应用，但这需要仔细投入时间和精力。我们建议首先尝试通过提示工程、提示链接（将复杂的任务分解为多个提示）和[函数调用](https://platform.openai.com/docs/guides/function-calling)来获得良好的结果，主要原因是：

- 在许多任务中，我们的模型最初可能表现不佳，但可以通过正确的提示来改进结果 - 因此可能不需要进行微调
- 迭代提示和其他策略比微调迭代具有更快的反馈循环，后者需要创建数据集并运行训练作业
- 在仍然需要微调的情况下，最初的提示工程工作不会浪费 - 在微调数据中使用良好的提示（或将提示链接/工具使用与微调相结合）时，我们通常会看到最佳结果

我们的[提示词工程指南](https://platform.openai.com/docs/guides/prompt-engineering)提供了一些最有效的策略和策略的背景知识，无需微调即可获得更好的性能。您可能会发现在我们的 [Playground](https://platform.openai.com/playground) 中快速迭代提示很有帮助。

### 常见用例
微调可以改善结果的一些常见用例：

- 设置风格、基调、格式或其他定性方面
- 提高产生所需输出的可靠性
- 纠正未能遵循复杂提示的问题
- 以特定方式处理许多边缘情况
- 执行难以在提示中阐明的新技能或任务

思考这些案例的一种高级方法是 “展示而不是讲述” 更容易。在接下来的部分中，我们将探讨如何设置用于微调的数据以及微调可以提高基准模型性能的各种示例。

微调有效的另一种情况是通过替换 GPT-4 或使用较短的提示来降低成本和/或延迟，而不牺牲质量。如果您可以使用 GPT-4 获得良好的结果，那么您通常可以通过 `gpt-3.5-turbo` 对 GPT-4 补全进行微调（可能还需要缩短指令提示）来使用微调模型达到类似的质量。
## 准备你的数据集
一旦您确定微调是正确的解决方案（即您已经尽可能优化了提示并确定了模型仍然存在的问题），您将需要准备数据来训练模型。您应该创建一组多样化的演示对话，这些对话类似于您要求模型在生产中的推理时响应的对话。

数据集中的每个示例都应该是与我们的 Chat Completions API 格式相同的对话，特别是消息列表，其中每条消息都有角色、内容和可选名称。至少一些训练示例应该直接针对提示模型未按预期运行的情况，并且数据中提供的辅助消息应该是您希望模型提供的理想响应。
### 格式示例
在此示例中，我们的目标是创建一个偶尔给出讽刺性响应的聊天机器人，这是我们可以为数据集创建的三个训练示例（对话）：

	{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "What's the capital of France?"}, {"role": "assistant", "content": "Paris, as if everyone doesn't know that already."}]}
	{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "Who wrote 'Romeo and Juliet'?"}, {"role": "assistant", "content": "Oh, just some guy named William Shakespeare. Ever heard of him?"}]}
	{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "How far is the Moon from Earth?"}, {"role": "assistant", "content": "Around 384,400 kilometers. Give or take a few, like that really matters."}]}

对话式聊天格式需要微调 `gpt-3.5-turbo`。对于 `babbage-002` 和`davinci-002`，您可以遵循用于传统微调的提示完成对格式，如下所示。

	{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
	{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
	{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
### 制作提示
我们通常建议在微调之前采用您认为最适合模型的一组说明和提示，并将其包含在每个训练示例中。这应该可以让您达到最好和最一般的结果，特别是如果您的训练示例相对较少（例如一百个以下）。

如果您想缩短每个示例中重复的说明或提示以节省成本，请记住，模型的行为可能就像包含这些说明一样，并且可能很难让模型忽略那些 “baked-in” 推理时的指令。

可能需要更多的训练示例才能达到良好的结果，因为模型必须完全通过演示来学习，而无需指导说明。
### 计数建议示例
要微调模型，您需要提供至少 10 个示例。我们通常会看到 `gpt-3.5-turbo` 对 50 到 100 个训练示例进行微调会带来明显的改进，但正确的数量根据具体的用例而有很大差异。

我们建议从 50 个精心设计的演示开始，看看模型在微调后是否显示出改进的迹象。在某些情况下，这可能就足够了，但即使模型尚未达到生产质量，明显的改进也是一个好兆头，表明提供更多数据将继续改进模型。

	没有任何改进表明您可能需要重新考虑如何在超出有限示例集之前设置模型任务或重组数据。
### 训练和测试分割
收集初始数据集后，我们建议将其分为训练和测试部分。当提交包含训练和测试文件的微调作业时，我们将在训练过程中提供两者的统计数据。这些统计数据将成为您模型改进程度的初始信号。此外，尽早构建测试集将有助于确保您能够在训练后通过在测试集上生成样本来评估模型。
### token 限制
每个训练示例限制为 4096 个 token。训练时，超过此长度的示例将被截断为前 4096 个token。为了确保整个训练示例适合上下文，请考虑检查消息内容中的总 token 计数是否低于 4,000。

您可以使用 OpenAI cookbook 中的[计数 token 笔记本](https://cookbook.openai.com/examples/How_to_count_tokens_with_tiktoken.ipynb)来计算 token 计数。
### 估算成本
请参阅[定价页面](https://openai.com/pricing)，了解每 1000 个输入和输出 token 的成本详细信息（我们对属于验证数据一部分的 token 收费）。要估算特定微调作业的成本，请使用以下公式：

	每 1k 个 token 的基本成本*输入文件中的 token 数量*训练的 epoch 数量
对于包含 100,000 个token 并经过 3 个 epoch 训练的训练文件，预期成本约为 2.40 美元。
### 检查数据格式
编译数据集后，在创建微调作业之前，检查数据格式非常重要。为此，我们创建了一个简单的 Python 脚本，您可以使用它来查找潜在错误、检查 token 计数并估计微调作业的成本。

[微调数据格式验证](https://cookbook.openai.com/examples/chat_finetuning_data_prep)
了解微调数据格式
### 上传训练文件
验证数据后，需要使用[文件 API]( API ) 上传文件，以便与微调作业一起使用：

curl 

	curl https://api.openai.com/v1/files \
	  -H "Authorization: Bearer $OPENAI_API_KEY" \
	  -F purpose="fine-tune" \
	  -F file="@mydata.jsonl"
上传文件后，可能需要一些时间来处理。在处理文件时，您仍然可以创建微调作业，但直到文件处理完成后才会启动。
### 创建微调模型
确保数据集的数量和结构正确并上传文件后，下一步是创建微调作业。我们支持通过[微调 UI ](https://platform.openai.com/finetune)或以编程方式创建微调作业。

要使用 OpenAI SDK 启动微调作业：

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	from openai import OpenAI
	client = OpenAI()
	
	client.fine_tuning.jobs.create(
	  training_file="file-abc123", 
	  model="gpt-3.5-turbo"
	)
在此示例中，model 是要微调的模型的名称（`gpt-3.5-turbo、babbage-002、davinci-002` 或现有的微调模型），`training_file` 是将训练文件上传到 OpenAI API 时返回的文件 ID。您可以使用后缀参数自定义微调模型的名称。

要设置其他微调参数，例如 `validation_file` 或 `hyperparameters`，请参阅[微调 API 规范](https://platform.openai.com/docs/api-reference/fine-tuning/create)。

开始微调工作后，可能需要一些时间才能完成。您的作业可能排在我们系统中的其他作业后面，训练模型可能需要几分钟或几小时，具体取决于模型和数据集大小。模型训练完成后，创建微调作业的用户将收到一封确认电子邮件。

除了创建微调作业外，您还可以列出现有作业、检索作业状态或取消作业。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	from openai import OpenAI
	client = OpenAI()
	
	# 列出10个微调工作
	client.fine_tuning.jobs.list(limit=10)
	
	# 检索微调的状态
	client.fine_tuning.jobs.retrieve("ftjob-abc123")
	
	# 取消作业
	client.fine_tuning.jobs.cancel("ftjob-abc123")
	
	# 列出一个微调工作中的最多10个事件
	client.fine_tuning.jobs.list_events(fine_tuning_job_id="ftjob-abc123", limit=10)
	
	# 从一个微调job中列出最多10个事件。删除一个微调模型(必须是创建模型的组织的所有者)
	client.models.delete("ft:gpt-3.5-turbo:acemeco:suffix:abc123")

### 使用微调模型
作业成功后，您 `fine_tuned_model` 在检索作业详细信息时将看到该字段填充有模型名称。您现在可以将此模型指定为 [Chat Completions](https://platform.openai.com/docs/api-reference/chat) (for gpt-3.5-turbo) 或[旧版 Completions](https://platform.openai.com/docs/api-reference/completions) API (for `babbage-002` 和 `davinci-002`)中的参数，并使用 [Playground](https://platform.openai.com/playground) 向其发出请求。

工作完成后，模型应该立即可供推理使用。在某些情况下，您的模型可能需要几分钟才能准备好处理请求。如果对模型的请求超时或找不到模型名称，可能是因为您的模型仍在加载中。如果发生这种情况，请在几分钟后重试。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`
	
	from openai import OpenAI
	client = OpenAI()
	
	response = client.chat.completions.create(
	  model="ft:gpt-3.5-turbo:my-org:custom_suffix:id",
	  messages=[
	    {"role": "system", "content": "You are a helpful assistant."},
	    {"role": "user", "content": "Hello!"}
	  ]
	)
	print(completion.choices[0].message)
您可以通过传递上面和我们的 [GPT 指南](https://platform.openai.com/docs/guides/text-generation/chat-completions-api) 中所示的型号名称来开始提出请求。

### 分析您的微调模型
我们提供在训练过程中计算的以下训练指标：

- 训练损失
- 训练 token 准确性
- 测试损失
- 和测试 token 准确性

这些统计数据旨在提供健全性检查，确保训练顺利进行（损失应该减少，token 准确性应该增加）。当活动的微调作业正在运行时，您可以查看包含一些有用指标的事件对象：

	{
	    "object": "fine_tuning.job.event",
	    "id": "ftevent-abc-123",
	    "created_at": 1693582679,
	    "level": "info",
	    "message": "Step 100/100: training loss=0.00",
	    "data": {
	        "step": 100,
	        "train_loss": 1.805623287509661e-5,
	        "train_mean_token_accuracy": 1.0
	    },
	    "type": "metrics"
	}
微调作业完成后，您还可以通过[查询微调作业](https://platform.openai.com/docs/api-reference/fine-tuning/retrieve)、从 ` result_files` 中提取文件 ID，然后[检索该文件内容](https://platform.openai.com/docs/api-reference/files/retrieve-contents)来查看有关训练过程的指标。每个结果 CSV 文件都包含以下列：

- step
- train_loss
- train_accuracy
- valid_loss
- 和valid_mean_token_accuracy

例

	step,train_loss,train_accuracy,valid_loss,valid_mean_token_accuracy
	1,1.52347,0.0,,
	2,0.57719,0.0,,
	3,3.63525,0.0,,
	4,1.72257,0.0,,
	5,1.52379,0.0,,
虽然指标很有帮助，但评估微调模型中的样本可以提供最相关的模型质量感觉。我们建议在测试集上从基本模型和微调模型生成样本，并并排比较样本。理想情况下，测试集应包括您可能在生产用例中发送到模型的输入的完整分布。如果手动评估太耗时，请考虑使用我们的 [Evals 库](https://github.com/openai/evals)来自动执行未来的评估。

### 迭代数据质量
如果微调作业的结果不如预期，请考虑以下方式调整训练数据集：

- 收集示例以解决剩余问题
	- 如果模型在某些方面仍然不擅长，请添加训练示例，直接向模型展示如何正确地完成这些方面
- 仔细检查现有示例是否存在问题
	- 如果您的模型存在语法、逻辑或样式问题，请检查您的数据是否存在任何相同的问题。例如，如果模型现在说“我将为您安排这次会议”（实际上不应该如此），请查看现有示例是否教导模型说它可以做它不能做的新事情
- 考虑数据的平衡性和多样性
	- 如果数据中 60% 的助理回答说“我无法回答这个问题”，但在推理时只有 5% 的回答应该这么说，那么您可能会得到过多的拒绝
- 确保您的训练示例包含响应所需的所有信息
	- 如果我们希望模型根据用户的个人特征来赞美用户，并且训练示例包括对先前对话中未找到的特征的辅助赞美，则模型可能会学习产生幻觉信息
- 查看训练示例中的一致性/一致性
	- 如果多个人创建了训练数据，则模型性能可能会受到人们之间的一致性/一致性程度的限制。例如，在文本提取任务中，如果人们仅同意 70% 的提取片段，则模型可能无法做得更好
- 确保所有训练示例都采用相同的格式，正如推理所预期的那样

### 迭代数据量
一旦您对示例的质量和分布感到满意，您就可以考虑扩大训练示例的数量。这往往有助于模型更好地学习任务，尤其是围绕可能的“边缘情况”。每次将训练示例的数量增加一倍，我们预计都会有类似的改进。您可以通过以下方式粗略地估计通过增加训练数据大小而获得的预期质量增益：

- 对当前数据集进行微调
- 对当前数据集的一半进行微调
- 观察两者之间的质量差距

一般来说，如果必须做出权衡，较少量的高质量数据通常比较大量的低质量数据更有效。

### 迭代超参数
我们允许您指定以下超参数：

- epochs
- `learning rate multiplier`(学习率乘数)
- `batch size`(批量大小)

我们建议最初进行训练时不指定任何这些，以便我们根据数据集大小为您选择默认值，然后在您观察到以下情况时进行调整：

- 如果模型没有像预期那样遵循训练数据，则将 epoch 数量增加 1 或 2
	- 对于只有一个理想完成（或一小组相似的理想完成）的任务来说，这种情况更为常见。一些示例包括分类、实体提取或结构化解析。这些通常是您可以根据参考答案计算最终准确性指标的任务。
- 如果模型的多样性低于预期，则将 epoch 数量减少 1 或 2
	- 对于有多种可能良好完成的任务来说，这种情况更为常见
- 如果模型似乎没有收敛，请增加学习率乘数

您可以设置超参数，如下所示：

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	from openai import OpenAI
	client = OpenAI()
	
	client.fine_tuning.jobs.create(
	  training_file="file-abc123", 
	  model="gpt-3.5-turbo", 
	  hyperparameters={
	    "n_epochs":2
	  }
	)

### 微调示例
现在我们已经探索了微调 API 的基础知识，接下来让我们看看几个不同用例的微调生命周期。

- 风格和基调

	在此示例中，我们将探索如何构建一个微调模型，使模型遵循特定的风格和语气指导，超出了单独提示所能达到的范围。
	
	- 首先，我们创建一组示例消息，显示模型应该做什么，在本例中是拼写错误的单词。
	
			{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "What's the capital of France?"}, {"role": "assistant", "content": "Paris, as if everyone doesn't know that already."}]}
			{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "Who wrote 'Romeo and Juliet'?"}, {"role": "assistant", "content": "Oh, just some guy named William Shakespeare. Ever heard of him?"}]}
			{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "How far is the Moon from Earth?"}, {"role": "assistant", "content": "Around 384,400 kilometers. Give or take a few, like that really matters."}]}
		如果您想继续操作并自己创建一个微调模型，您将需要至少 10 个示例。
	- 获得可能改进模型的数据后，下一步是检查数据是否满足所有格式要求。
	
		现在我们已经格式化并验证了数据，最后的训练步骤是开始创建微调模型的工作。您可以通过 OpenAI CLI 或我们的 SDK 之一来执行此操作，如下所示：
	
		程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`
		
			from openai import OpenAI
			client = OpenAI()
			
			file = client.files.create(
			  file=open("marv.jsonl", "rb"),
			  purpose="fine-tune"
			)
			
			client.fine_tuning.jobs.create(
			  training_file=file.id,
			  model="gpt-3.5-turbo"
			)
		训练工作完成后，您将能够使用经过微调的模型。
- 结构化输出
	- 另一种非常适合微调的用例是让模型提供结构化信息，在本例中是关于体育头条新闻的：
		
			{"messages": [{"role": "system", "content": "Given a sports headline, provide the following fields in a JSON dict, where applicable: "player" (full name)", "team", "sport", and "gender".},{"role": "user", "content": "Sources: Colts grant RB Taylor OK to seek trade"},
			{"role": "assistant", "content": "{"player": "Jonathan Taylor", "team": "Colts", "sport": "football", "gender": "male" }"},]}
			{"messages": [{"role": "system", "content": "Given a sports headline, provide the following fields in a JSON dict, where applicable: "player" (full name)", "team", "sport", and "gender".},{"role": "user", "content": "OSU 'split down middle' on starting QB battle"},
			{"role": "assistant", "content": "{"player": null, "team": "OSU", "sport": "football", "gender": null }"},]}
		如果您想继续操作并自己创建一个微调模型，您将需要至少 10 个示例。
	
	- 获得可能改进模型的数据后，下一步是检查数据是否满足所有格式要求。
	
		现在我们已经格式化并验证了数据，最后的训练步骤是开始创建微调模型的工作。您可以通过 OpenAI CLI 或我们的 SDK 之一来执行此操作，如下所示：
	
		程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`
	
			from openai import OpenAI
			client = OpenAI()
			
			file = client.files.create(
			  file=open("sports-context.jsonl", "rb"),
			  purpose="fine-tune"
			)
			
			client.fine_tuning.jobs.create(
			  training_file=file.id,
			  model="gpt-3.5-turbo"
			)
	- 训练工作完成后，您将能够使用微调后的模型并提出如下所示的请求：
	
		程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`
	
			completion = client.chat.completions.create(
			  model="ft:gpt-3.5-turbo:my-org:custom_suffix:id",
			  messages=[
			    {"role": "system", "content": "Given a sports headline, provide the following fields in a JSON dict, where applicable: player (full name), team, sport, and gender"},
			    {"role": "user", "content": "Richardson wins 100m at worlds to cap comeback"}
			  ]
			)
			
			print(completion.choices[0].message)
		根据格式化的训练数据，响应应如下所示：
	
			{"player": "Sha'Carri Richardson", "team": null", "sport": "track and field", "gender": "female"}
- 函数调用

	chat completions API 支持[函数调用](https://platform.openai.com/docs/guides/function-calling)。在完成 API 中包含一长串函数可能会消耗大量提示 token ，有时模型会产生幻觉或不提供有效的 JSON 输出。
	
	使用函数调用示例微调模型可以让您：
	
	- 即使不存在完整的函数定义，也可以获得类似格式的响应
	- 获得更准确、一致的输出
	
	如图所示格式化示例，每行包含“message”列表和可选的“functions”列表：
	
		{
		    "messages": [
		        {"role": "user", "content": "What is the weather in San Francisco?"},
		        {"role": "assistant", "function_call": {"name": "get_current_weather", "arguments": "{\"location\": \"San Francisco, USA\", \"format\": \"celcius\"}"}
		    ],
		    "functions": [{
		        "name": "get_current_weather",
		        "description": "Get the current weather",
		        "parameters": {
		            "type": "object",
		            "properties": {
		                "location": {"type": "string", "description": "The city and country, eg. San Francisco, USA"},
		                "format": {"type": "string", "enum": ["celsius", "fahrenheit"]}
		            },
		            "required": ["location", "format"]
		        }
		    }]
		}
	如果您想继续操作并自己创建一个微调模型，您将需要至少 10 个示例。
	
	如果您的目标是使用更少的 token，一些有用的技术是：
	
	- 省略函数和参数描述：从函数和参数中删除描述字段
	- 省略参数：从参数对象中删除整个属性字段
	- 完全省略函数：从函数数组中删除整个函数对象
	
	如果您的目标是最大限度地提高函数调用输出的正确性，我们建议使用相同的函数定义来训练和查询微调模型。
	
	对函数调用的微调也可用于自定义模型对函数输出的响应。为此，您可以包含一条函数响应消息和一条解释该响应的辅助消息：
	
		{
		    "messages": [
		        {"role": "user", "content": "What is the weather in San Francisco?"},
		        {"role": "assistant", "function_call": {"name": "get_current_weather", "arguments": "{\"location\": \"San Francisco, USA\", \"format\": \"celcius\"}"}}
		        {"role": "function", "name": "get_current_weather", "content": "21.0"},
		        {"role": "assistant", "content": "It is 21 degrees celsius in San Francisco, CA"}
		    ],
		    "functions": [...] // 和以前一样
		}
	
### 遗留模型的迁移
对于从 `/v1/fine-tunes` 更新的 `/v1/fine_tuning/jobsAPI` 和更新的模型迁移到的用户，您可以预期的主要区别是更新的 API。 `babbage-002` 更新后的模型保留了旧的提示完成对数据格式 `davinci-002`，以确保平稳过渡。新模型将支持 4k token上下文的微调，知识截止日期为 2021 年 9 月。

`gpt-3.5-turbo` 对于大多数任务，您应该期望从 GPT 基本模型获得更好的性能。

## 常问问题
- 我什么时候应该使用微调和嵌入进行检索？

	带检索的嵌入最适合需要拥有具有相关上下文和信息的大型文档数据库的情况。

	默认情况下，OpenAI 的模型被训练为有用的多面手助理。微调可用于建立一个目标明确的模型，并展示特定的根深蒂固的行为模式。检索策略可用于在生成响应之前向模型提供相关上下文，从而为模型提供新信息。检索策略并不是微调的替代方案，实际上可以作为微调的补充。
- 我可以微调 GPT-4 或 GPT-3.5-Turbo-16k 吗？

	GPT-4 微调处于实验性访问阶段，符合条件的开发者可以通过[微调 UI](https://platform.openai.com/finetune)请求访问。目前，gpt-3.5-turbo-1106 最多支持 16K 上下文示例。
- 我如何知道我的微调模型是否实际上比基本模型更好？

	我们建议在聊天对话测试集上从基本模型和微调模型生成样本，并并排比较样本。如需更全面的评估，请考虑使用 [OpenAI 评估框架](https://github.com/openai/evals)来创建特定于您的用例的评估。
- 已经微调过的模型还可以继续微调吗？

	是的，您可以在创建微调作业时将微调模型的名称传递到 `model` 参数中。这将以微调模型为起点开始新的微调工作。
- 如何估算微调模型的成本？

	请参阅上面的估算[成本部分](https://platform.openai.com/docs/guides/fine-tuning/estimate-costs)。
- 新的微调端点是否仍然可以与权重和偏差一起使用来跟踪指标？

	不，我们目前不支持这种集成，但正在努力在不久的将来实现它。
- 我可以同时运行多少个微调作业？

	请参阅我们的[费率限制指南](https://platform.openai.com/docs/guides/rate-limits/what-are-the-rate-limits-for-our-api)，了解有关限制的最新信息。
- 速率限制如何在微调模型上发挥作用？

	经过微调的模型与它所基于的模型具有相同的共享速率限制。例如，如果您在给定时间段内使用标准`gpt-3.5-turbo` 模型的 TPM 速率限制的一半，则您微调的任何模型都 `gpt-3.5-turbo` 只能访问 TPM 速率限制的剩余一半，因为容量是在所有模型之间共享的。同类型的型号。

	换句话说，从总吞吐量的角度来看，拥有微调模型并不能为您提供更多使用我们模型的能力。