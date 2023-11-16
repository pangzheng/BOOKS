# OpenAI`4.0T`官方文档-能力-9 审核
## 概述
[审核](https://platform.openai.com/docs/api-reference/moderations)端点是一个可用于检查内容是否符合 OpenAI 的[使用政策](https://openai.com/policies/usage-policies)的工具。因此，开发人员可以识别我们的使用政策禁止的内容并采取行动，例如过滤它。

这些模型分为以下几类：

类别|描述
---|---|
hate|表达、煽动或宣扬基于种族、性别、民族、宗教、国籍、性取向、残疾状况或种姓的仇恨的内容。针对非受保护群体（例如棋手）的仇恨内容属于骚扰。
hate/threatening|仇恨内容还包括基于种族、性别、民族、宗教、国籍、性取向、残疾状况或种姓对目标群体实施暴力或严重伤害。
harassment|对任何目标表达、煽动或宣扬骚扰性语言的内容。
harassment/threatening|骚扰内容还包括对任何目标的暴力或严重伤害。
self-harm|宣扬、鼓励或描述自残行为的内容，例如自杀、自残和饮食失调。
self-harm/intent|说话者表示他们正在或打算进行自残行为的内容，例如自杀、割伤和饮食失调。
self-harm/instructions|鼓励进行自残行为（例如自杀、割伤和饮食失调）的内容，或者提供有关如何实施此类行为的说明或建议的内容。
sexual|旨在引起性兴奋的内容，例如性活动的描述，或宣传性服务（不包括性教育和健康）。
sexual/minors|涉及 18 岁以下个人的色情内容。
violence|描述死亡、暴力或身体伤害的内容。
violence/graphic|以图形细节描述死亡、暴力或身体伤害的内容。

监控 OpenAI API 的输入和输出时，可以免费使用审核端点。我们目前不允许其他用例。较长文本的准确性可能会较低。为了获得更高的准确度，请尝试将长文本分割成较小的块，每个块少于 2,000 个字符。

我们不断努力提高分类器的准确性。目前我们对非英语语言的支持有限。

## 快速开始
要获取一段文本的分类，请向[审核端点](https://platform.openai.com/docs/api-reference/moderations)发出请求，如以下代码片段所示：

- 示例：获得审核

		curl https://api.openai.com/v1/moderations \
		  -X POST \
		  -H "Content-Type: application/json" \
		  -H "Authorization: Bearer $OPENAI_API_KEY" \
		  -d '{"input": "Sample text goes here"}'

以下是端点的输出示例。它返回以下字段：

- `flagged`

	如果模型将内容分类为违反 OpenAI 的使用政策，则设置为 `true `，否则设置为 `false `。
- `categories`

	包含每个类别的二进制使用策略违规标志的字典。对于每个类别，该值是`true` 模型是否将相应类别标记为违规，否则为 `false `。
- `category_scores`

	包含模型输出的每个类别原始分数的字典，表示模型对输入违反 OpenAI 该类别政策的置信度。该值介于 0 和 1 之间，其中值越高表示置信度越高。分数不应被解释为概率。
	
- 返回例子	

	程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

		{
		  "id": "modr-XXXXX",
		  "model": "text-moderation-005",
		  "results": [
		    {
		      "flagged": true,
		      "categories": {
		        "sexual": false,
		        "hate": false,
		        "harassment": false,
		        "self-harm": false,
		        "sexual/minors": false,
		        "hate/threatening": false,
		        "violence/graphic": false,
		        "self-harm/intent": false,
		        "self-harm/instructions": false,
		        "harassment/threatening": true,
		        "violence": true,
		      },
		      "category_scores": {
		        "sexual": 1.2282071e-06,
		        "hate": 0.010696256,
		        "harassment": 0.29842457,
		        "self-harm": 1.5236925e-08,
		        "sexual/minors": 5.7246268e-08,
		        "hate/threatening": 0.0060676364,
		        "violence/graphic": 4.435014e-06,
		        "self-harm/intent": 8.098441e-10,
		        "self-harm/instructions": 2.8498655e-11,
		        "harassment/threatening": 0.63055265,
		        "violence": 0.99011886,
		      }
		    }
		  ]
		}
OpenAI 将不断升级审核端点的底层模型。因此，依赖的自定义策略 `category_scores` 可能需要随着时间的推移重新校准。