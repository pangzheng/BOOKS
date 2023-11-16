# OpenAI`4.0T`官方文档-能力-6 视觉
了解如何使用 GPT-4 来理解图像
## 介绍
GPT-4 with Vision（有时称为 [GPT-4V](https://openai.com/research/gpt-4v-system-card)或 `gpt-4-vision-preview` 在 API 中）允许模型接收图像并回答有关图像的问题。从历史上看，语言模型系统受到单一输入模式（文本）的限制。对于许多用例来说，这限制了 GPT-4 等模型的使用领域。

目前，所有可以通过 `gpt-4-vision-preview` 模型和 Chat Completions API [访问 GPT- 4 的开发人员](https://help.openai.com/en/articles/7102672-how-can-i-access-gpt-4)都可以使用具有视觉功能的 GPT-4，该 API 已更新为支持图像输入。请注意，[Assistants API](https://platform.openai.com/docs/api-reference/assistants) 目前不支持图像输入。

重要的是要注意以下几点：

- 带视觉的 GPT-4 模型的行为与 GPT-4 没有什么不同，除了我们用于模型的系统提示之外
- 具有视觉功能的 GPT-4 并不是一个在文本任务上表现较差的不同模型，因为它具有视觉功能，它只是添加了视觉功能的 GPT-4
- 具有视觉功能的 GPT-4 是模型的一组增强功能

目前，GPT-4 with Vision 不支持 `message.name` 参数、`functions/tools`、`response_format` 参数，并且我们当前设置了一个较低的 `max_tokens` 默认值，您可以覆盖该默认值。

## 快速开始
可以通过两种主要方式向模型提供图像：

- 传递图像的链接或直接在请求中传递 base64 编码的图像
- 图像可以在 `user `、`system` 和  `assistant` 消息中传递。目前我们不支持第一条 `system` 消息中的图像，但将来可能会改变。

- 这张图片里有什么？
		
		curl https://api.openai.com/v1/chat/completions \
		  -H "Content-Type: application/json" \
		  -H "Authorization: Bearer $OPENAI_API_KEY" \
		  -d '{
		    "model": "gpt-4-vision-preview",
		    "messages": [
		      {
		        "role": "user",
		        "content": [
		          {
		            "type": "text",
		            "text": "What’s in this image?"
		          },
		          {
		            "type": "image_url",
		            "image_url": {
		              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
		            }
		          }
		        ]
		      }
		    ],
		    "max_tokens": 300
		  }'
		  
该模型最擅长回答有关图像中存在的内容的一般问题。虽然它确实理解图像中对象之间的关系，但尚未优化以回答有关图像中某些对象位置的详细问题。

例如，

- 您可以询问它汽车是什么颜色或者根据冰箱里的东西提出一些关于晚餐的想法
- 但如果您向它展示房间的图像并询问它椅子在哪里，它可能不会回答问题正确。

当您探索视觉理解可以应用于哪些用例时，请务必牢记[模型的局限性](https://platform.openai.com/docs/guides/vision/limitations)。

- [用视觉理解视频](https://cookbook.openai.com/examples/gpt_with_vision_for_video_understanding)

	了解如何结合使用 GPT-4 和 Vision 来理解 OpenAI Cookbook 中的视频

## 上传 Base 64 编码图像
如果您本地有一个图像或一组图像，则可以将它们以 Base 64 编码格式传递给模型，以下是实际操作的示例：

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`
	
	import base64
	import requests
	
	# OpenAI API Key
	api_key = "YOUR_OPENAI_API_KEY"
	
	# 函数对图像进行编码
	def encode_image(image_path):
	  with open(image_path, "rb") as image_file:
	    return base64.b64encode(image_file.read()).decode('utf-8')
	
	# 到图像的路径
	image_path = "path_to_your_image.jpg"
	
	# 获取base64字符串
	base64_image = encode_image(image_path)
	
	headers = {
	  "Content-Type": "application/json",
	  "Authorization": f"Bearer {api_key}"
	}
	
	payload = {
	  "model": "gpt-4-vision-preview",
	  "messages": [
	    {
	      "role": "user",
	      "content": [
	        {
	          "type": "text",
	          "text": "What’s in this image?"
	        },
	        {
	          "type": "image_url",
	          "image_url": {
	            "url": f"data:image/jpeg;base64,{base64_image}"
	          }
	        }
	      ]
	    }
	  ],
	  "max_tokens": 300
	}
	
	response = requests.post("https://api.openai.com/v1/chat/completions", headers=headers, json=payload)
	
	print(response.json())

## 多图像输入
Chat Completions API 能够接收并处理采用 Base64 编码格式或图像 URL 形式的多个图像输入。该模型将处理每张图像并使用所有图像的信息来回答问题。

- 多图像输入
		
		curl https://api.openai.com/v1/chat/completions \
		  -H "Content-Type: application/json" \
		  -H "Authorization: Bearer $OPENAI_API_KEY" \
		  -d '{
		    "model": "gpt-4-vision-preview",
		    "messages": [
		      {
		        "role": "user",
		        "content": [
		          {
		            "type": "text",
		            "text": "What are in these images? Is there any difference between them?"
		          },
		          {
		            "type": "image_url",
		            "image_url": {
		              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
		            }
		          },
		          {
		            "type": "image_url",
		            "image_url": {
		              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
		            }
		          }
		        ]
		      }
		    ],
		    "max_tokens": 300
		  }'
在这里，模型显示了同一图像的两个副本，并且可以独立回答有关两个或每个图像的问题。

## 低保真度或高保真度图像理解
通过控制 `detail` 具有三个选项

- `low`
- `high`
- 或 `auto `

的参数，您可以控制模型如何处理图像并生成其文本理解。默认情况下，模型将使用 `auto` 将查看图像输入大小并决定是否应使用 `low` 或 `high` 设置的设置。

- `low` 将禁用“高分辨率”模型。

	该模型将收到图像的低分辨率 512px x 512px 版本，并以 65 个token 的预算表示该图像。这使得 API 能够返回更快的响应，并在不需要高细节的用例中消耗更少的输入 token。
- `high` 将启用“高分辨率”模式

	该模式首先允许模型查看低分辨率图像，然后根据输入图像大小将输入图像创建为 512 像素正方形的详细裁剪。每个详细作物使用两倍的 token 预算（65 个token），总共 129 个token。

- 选择细节级别
		
		curl https://api.openai.com/v1/chat/completions \
		  -H "Content-Type: application/json" \
		  -H "Authorization: Bearer $OPENAI_API_KEY" \
		  -d '{
		    "model": "gpt-4-vision-preview",
		    "messages": [
		      {
		        "role": "user",
		        "content": [
		          {
		            "type": "text",
		            "text": "What’s in this image?"
		          },
		          {
		            "type": "image_url",
		            "image_url": {
		              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
		              "detail": "high"
		            }
		          }
		        ]
		      }
		    ],
		    "max_tokens": 300
		  }'
## 管理图像
与助手 API 不同，聊天完成 API 没有状态。这意味着您必须自己管理传递给模型的消息（包括图像）。如果您想多次将相同的图像传递给模型，则每次向 API 发出请求时都必须传递该图像。

对于长时间运行的对话，我们建议通过 URL 而不是 base64 传递图像。还可以通过提前将图像尺寸缩小到小于预期的最大尺寸来改善模型的延迟。对于低分辨率模式，我们预计图像尺寸为 512 像素 x 512 像素。对于高休息模式，图像的短边应小于 768 像素，长边应小于 2,000 像素。

图像经过模型处理后，将从 OpenAI 服务器中删除且不会保留。[我们不使用通过 OpenAI API 上传的数据来训练我们的模型。](https://openai.com/enterprise-privacy)

## 局限性
虽然具有视觉功能的 GPT-4 功能强大并且可以在许多情况下使用，但了解该模型的局限性也很重要。以下是我们意识到的一些限制：

- 医学图像：该模型不适合解释 CT 扫描等专业医学图像，也不应用于提供医疗建议。
- 非英语：在处理包含非拉丁字母文本（例如日语或韩语）的图像时，模型可能无法获得最佳性能。
- 大文本：放大图像中的文本以提高可读性，但避免裁剪重要细节。
- 旋转：模型可能会误解旋转/颠倒的文本或图像。
- 视觉元素：模型可能难以理解颜色或样式（如实线、虚线或点线）变化的图形或文本。
- 空间推理：该模型难以完成需要精确空间定位的任务，例如识别国际象棋位置。
- 准确性：在某些情况下，模型可能会生成不正确的描述或标题。
- 图像形状：模型难以处理全景和鱼眼图像。
- 元数据和调整大小：模型不处理原始文件名或元数据，图像在分析之前会调整大小，从而影响其原始尺寸。
- 计数：可以给出图像中对象的近似计数。
- 验证码：出于安全原因，我们实施了一个系统来阻止验证码的提交。

## 计算成本
与文本输入一样，图像输入以 token 计量和收费。给定图像的 token 成本由两个因素决定：

- 图像的大小以及每个 image_url 块上的 `detail` 选项。
	- 所有图像 `detail: low` 每张花费 85 个token。
	- `detail: high` 图像首先被缩放以适合 2048 x 2048 的正方形，并保持其纵横比。然后，对它们进行缩放，使图像的最短边长为 768 像素。
- 最后，我们计算图像由多少个 512px 的正方形组成。每个方格需要170 个token。另外85 个token始终会添加到最终总数中。

以下是一些演示上述内容的示例。

- 模式下的 1024 x 1024 方形图像 `detail: high` 需要 765 个token
	- 1024 小于 2048，因此没有初始调整大小。
	- 最短边是 1024，因此我们将图像缩小到 768 x 768。
	- 需要 4 512px 方形图块来表示图像，因此最终的 token 成本为 170 * 4 + 85 = 765。
- 模式下的 2048 x 4096 图像 detail: high 需要 1105 个token
	- 我们将图像缩小到 1024 x 2048 以适合 2048 的正方形。
	- 最短边是 1024，因此我们进一步缩小到 768 x 1536。
	- 需要 6 512px 图块，因此最终的token成本为170 * 6 + 85 = 1105。
- 大多数情况下，一张 4096 x 8192 的图像`detail: low`需要 85 个token
	- 无论输入大小如何，低细节图像都是固定成本。

## 常问问题
- 我可以在 `gpt-4` 微调图像功能吗？
	
	不，我们目前不支持 `gpt-4` 微调图像功能。
- 我可以用来 `gpt-4`生成图像吗？

	不，您可以用来 `dall-e-3` 生成图像并 `gpt-4-vision-preview` 理解图像。
- 我可以上传什么类型的文件？

	我们目前支持 PNG (.png)、JPEG（.jpeg 和 .jpg）、WEBP (.webp) 和非动画 GIF (.gif)。
- 我可以上传的图片大小有限制吗？

	是的，我们将图像上传限制为每张图像 20MB。
- 我可以删除我上传的图片吗？

	不会，模型处理完图像后，我们会自动为您删除该图像。
- 在哪里可以了解有关 GPT-4 with Vision 的注意事项的更多信息？

	您可以在 [GPT-4 with Vision](https://openai.com/contributions/gpt-4v) 系统卡中找到有关我们的评估、准备和缓解工作的详细信息。

	我们进一步实施了一个系统来阻止验证码的提交。
- GPT-4 with Vision 的速率限制如何运作？

	我们在 token 级别处理图像，因此我们处理的每个图像都会计入您的每分钟 token (TPM) 限制。有关用于确定每个图像的标记计数的公式的详细信息，请参阅计算成本部分。
- 带有 Vision 的 GPT-4 可以理解图像元数据吗？

	不，模型不接收图像元数据。
- 如果我的图像不清楚怎么办？

	如果图像不明确或不清楚，模型将尽力解释它。然而，结果可能不太准确。一个好的经验法则是，如果普通人无法以低/高分辨率模式下使用的分辨率看到图像中的信息，那么模型也看不到。