# OpenAI`4.0T`官方文档-能力-8 语音转文字
了解如何将音频转换为文本
## 介绍
音频 API 提供两个语音转文本端点，`transcriptions` 和 `translations` 基于我们最先进的开源大型 v2 Whisper 模型。它们可用于：

- 将音频转录成音频所使用的任何语言。
- 将音频翻译并转录成英语。

文件上传当前限制为 25 MB，并且支持以下输入文件类型：`mp3、mp4、mpeg、mpga、m4a、wav和webm`。

## 快速开始
### 转录
转录 API 将您想要转录的音频文件以及音频转录所需的输出文件格式作为输入。我们目前支持多种输入和输出文件格式。

- 转录音频

		curl --request POST \
		  --url https://api.openai.com/v1/audio/transcriptions \
		  --header 'Authorization: Bearer TOKEN' \
		  --header 'Content-Type: multipart/form-data' \
		  --form file=@/path/to/file/openai.mp3 \
		  --form model=whisper-1
	
默认情况下，响应类型将为 json，其中包含原始文本。

	{
	  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger.
	....
	}
	翻译:想象一下您曾经有过的最疯狂的想法，并且您很好奇它如何扩展到 100 倍、1,000 倍……

音频 API 还允许您在请求中设置其他参数。例如，如果您想设置 `response_format` 格式为 `text`，您的请求将如下所示：

- 其他选项

		curl --request POST \
		  --url https://api.openai.com/v1/audio/transcriptions \
		  --header 'Authorization: Bearer TOKEN' \
		  --header 'Content-Type: multipart/form-data' \
		  --form file=@openai.mp3 \
		  --form model=whisper-1 \
		  --form response_format=text

[API参考](https://platform.openai.com/docs/api-reference/audio)包含可用参数的完整列表。

### 翻译
翻译 API 将任何受支持语言的音频文件作为输入，并在必要时将音频转录为英语。这与我们的 `/Transcriptions` 端点不同，因为输出不是原始输入语言，而是翻译为英语文本。

- 翻译音频

		curl --request POST   --url https://api.openai.com/v1/audio/translations   --header 'Authorization: Bearer TOKEN'   --header 'Content-Type: multipart/form-data'   --form file=@/path/to/file/german.mp3   --form model=whisper-1

在本例中，输入的音频是德语，输出的文本如下所示：

	Hello, my name is Wolfgang and I come from Germany. Where are you heading today?
	你好，我叫沃尔夫冈，来自德国。今天你要去哪里？
目前我们仅支持翻译成英文。

### 支持的语言
目前，我们通过 `transcriptions` 和 `translations` 端点[支持以下语言](https://github.com/openai/whisper#available-models-and-languages)：

南非荷兰语、阿拉伯语、亚美尼亚语、阿塞拜疆语、白俄罗斯语、波斯尼亚语、保加利亚语、加泰罗尼亚语、中文、克罗地亚语、捷克语、丹麦语、荷兰语、英语、爱沙尼亚语、芬兰语、法语、加利西亚语、德语、希腊语、希伯来语、印地语、匈牙利语、冰岛语、印度尼西亚语、意大利语、日语、卡纳达语、哈萨克语、韩语、拉脱维亚语、立陶宛语、马其顿语、马来语、马拉地语、毛利语、尼泊尔语、挪威语、波斯语、波兰语、葡萄牙语、罗马尼亚语、俄语、塞尔维亚语、斯洛伐克语、斯洛文尼亚语、西班牙语、斯瓦希里语、瑞典语、他加禄语、泰米尔语、泰语、土耳其语、乌克兰语、乌尔都语、越南语和威尔士语。

虽然底层模型是在 98 种语言上进行训练的，但我们只列出了[单词错误率](https://en.wikipedia.org/wiki/Word_error_rate)(WER)小于 50% 的语言，这是语音到文本模型准确性的行业标准基准。该模型将返回上面未列出的语言的结果，但质量较低。

### 更长的输入
默认情况下，Whisper API 仅支持小于 25 MB 的文件。如果您的音频文件比该长度长，则需要将其分成 25 MB 或更少的块或者使用压缩音频格式。为了获得最佳性能，我们建议您避免在句子中间中断音频，因为这可能会导致丢失一些上下文。

处理此问题的一种方法是使用 [PyDub 开源 Python](https://github.com/jiaaro/pydub) 包来分割音频：

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	from pydub import AudioSegment
	
	song = AudioSegment.from_mp3("good_morning.mp3")
	
	# PyDub handles time in milliseconds
	ten_minutes = 10 * 60 * 1000
	
	first_10_minutes = song[:ten_minutes]
	
	first_10_minutes.export("good_morning_10.mp3", format="mp3")
OpenAI 不保证 PyDub 等第三方软件的可用性或安全性。

### 提示词
您可以使用[提示词](https://platform.openai.com/docs/api-reference/audio/createTranscription#audio/createTranscription-prompt)来提高 Whisper API 生成的转录本的质量。该模型将尝试匹配提示的样式，因此如果提示也这样做，则更有可能使用大写和标点符号。然而，当前的提示系统比我们的其他语言模型受到更多限制，并且仅提供对生成的音频的有限控制。以下是提示如何在不同情况下提供帮助的一些示例：

- 提示词对于纠正模型在音频中经常错误识别的特定单词或首字母缩略词非常有帮助。

	例如，以下提示词改进了单词 DALL·E 和 GPT-3 的转录，这些词之前写为 “GDP 3” 和 “DALI”：“转录是关于 OpenAI，它使 DALL·E、GPT-3 和ChatGPT 等技术、，希望有一天能够建立一个造福全人类的AGI 系统”
- 要保留分割为多个片段的文件的上下文，您可以使用前一个片段的转录本来提示模型。这将使转录更加准确，因为模型将使用之前音频中的相关信息。该模型将仅考虑提示的最后 224 个 token，并忽略之前的任何内容。对于多语言输入，Whisper 使用自定义 tokenizer。对于仅英语输入，它使用标准 GPT-2 分词器，这两种分词器都可以通过 [开源 Whisper Python](https://github.com/openai/whisper/blob/main/whisper/tokenizer.py#L361) 包访问。
- 有时，模型可能会跳过记录中的标点符号。您可以通过使用包含标点符号的简单提示来避免这种情况：

		"Hello, welcome to my lecture."
		“您好，欢迎来到我的讲座。”
- 该模型还可能会遗漏音频中常见的填充词。如果您想在成绩单中保留填充词，您可以使用包含它们的提示：

		 "Umm, let me think like, hmm... Okay, here's what I'm, like, thinking."
		 “嗯，让我这样想，嗯......好吧，这就是我的想法。”
- 有些语言可以用不同的方式书写，例如简体中文或繁体中文。默认情况下，模型可能并不总是使用您想要的成绩单写作风格。您可以通过使用您喜欢的写作风格的提示词来改进这一点。

## 提高可靠性
正如我们在提示部分中探讨的那样，使用 Whisper 时面临的最常见挑战之一是模型通常无法识别不常见的单词或首字母缩略词。为了解决这个问题，我们重点介绍了在这些情况下提高 Whisper 可靠性的不同技术：

### 使用提示词参数
第一种方法涉及使用可选的提示词参数来传递正确拼写的字典。

由于 Whisper 没有使用指令跟踪技术进行训练，因此其运行方式更像是基本 GPT 模型。请务必记住，Whisper 仅考虑提示的前 244 个标记。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	transcribe(filepath, prompt="ZyntriQix, Digique Plus, CynapseFive, VortiQore V8, EchoNix Array, OrbitalLink Seven, DigiFractal Matrix, PULSE, RAPT, B.R.I.C.K., Q.U.A.R.T.Z., F.L.I.N.T.")
虽然它会提高可靠性，但该技术仅限于 244 个字符，因此您的 SKU 列表需要相对较小，才能成为可扩展的解决方案。

### 使用 GPT-4 进行后处理
第二种方法涉及使用 GPT-4 或 GPT-3.5-Turbo 的后处理步骤。

我们首先通过变量提供 GPT-4 的指令 `system_prompt`。与我们之前对提示词参数所做的类似，我们可以定义我们的公司和产品名称。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`
	
	system_prompt = "You are a helpful assistant for the company ZyntriQix. Your task is to correct any spelling discrepancies in the transcribed text. Make sure that the names of the following products are spelled correctly: ZyntriQix, Digique Plus, CynapseFive, VortiQore V8, EchoNix Array, OrbitalLink Seven, DigiFractal Matrix, PULSE, RAPT, B.R.I.C.K., Q.U.A.R.T.Z., F.L.I.N.T. Only add necessary punctuation such as periods, commas, and capitalization, and use only the context provided."
	
	def generate_corrected_transcript(temperature, system_prompt, audio_file):
	    response = client.chat.completions.create(
	        model="gpt-4",
	        temperature=temperature,
	        messages=[
	            {
	                "role": "system",
	                "content": system_prompt
	            },
	            {
	                "role": "user",
	                "content": transcribe(audio_file, "")
	            }
	        ]
	    )
	    return response['choices'][0]['message']['content']
	
	corrected_text = generate_corrected_transcript(0, system_prompt, fake_company_filepath)
如果您在自己的音频文件上尝试此操作，您会发现 GPT-4 设法纠正了记录中的许多拼写错误。由于其更大的上下文窗口，该方法可能比使用 Whisper 的提示参数更具可扩展性，并且更可靠，因为 GPT-4 可以以 Whisper 无法实现的方式进行指示和引导，因为缺乏后续指令。

