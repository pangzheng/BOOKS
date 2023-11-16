# OpenAI`4.0T`官方文档-能力-7 文字转语音
了解如何将文本转换为逼真的语音
## 介绍
音频 API 提供 `speech` 基于我们的 TTS（文本转语音）模型的文本转语音端点。它带有 6 种内置声音，可用于：

- 叙述一篇书面博客文章
- 制作多种语言的语音音频
- 使用流式传输提供实时音频输出

这是声音的示例 `alloy`：

- [听音频](https://platform.openai.com/docs/guides/text-to-speech)

		请注意，我们的使用政策要求您向最终用户明确披露，他们听到的 TTS 语音是人工智能生成的，而不是人类的声音。

## 快速开始
端点 `speech` 接受三个关键输入：

- 模型名称
- 应转换为音频的文本
- 以及用于生成音频的语音

一个简单的请求如下所示：

- 说出这段文字

		curl https://api.openai.com/v1/audio/speech \
		  -H "Authorization: Bearer $OPENAI_API_KEY" \
		  -H "Content-Type: application/json" \
		  -d '{
		    "model": "tts-1",
		    "input": "Today is a wonderful day to build something people love!",
		    "voice": "alloy"
		  }' \
		  --output speech.mp3
默认情况下，终端将输出语音音频的 MP3 文件，但也可以配置为输出我们[支持的任何格式](https://platform.openai.com/docs/guides/text-to-speech/supported-output-formats)。

## 音频质量
对于实时应用程序，标准 `tts-1` 模型提供最低的延迟，但质量低于 `tts-1-hd` 模型。由于音频的生成方式和根据您的收听设备和个人的不同，在某些情况, `tts-1` 下可能会生成比 `tts-1-hd`. 音频可能没有明显的差异。
## 语音选项
尝试不同的声音（`alloy`、`echo`、`fable`、`onyx`、`nova` 和 `shimmer`），找到一种与您所需的语气和听众相匹配的声音。当前的语音针对英语进行了优化。

- [听音频](https://platform.openai.com/docs/guides/text-to-speech)

## 支持的输出格式
默认响应格式为“mp3”，但也可以使用“opus”、“aac”或“flac”等其他格式。

- Opus：用于互联网流媒体和通信，低延迟。
- AAC：用于数字音频压缩，YouTube、Android、iOS 首选。
- FLAC：用于无损音频压缩，受到音频爱好者存档的青睐。

## 支持的语言
TTS 模型在语言支持方面总体上遵循 Whisper 模型。尽管当前语音针对英语进行了优化，但 [Whisper 支持以下语言](https://github.com/openai/whisper#available-models-and-languages)并且表现良好：

南非荷兰语、阿拉伯语、亚美尼亚语、阿塞拜疆语、白俄罗斯语、波斯尼亚语、保加利亚语、加泰罗尼亚语、中文、克罗地亚语、捷克语、丹麦语、荷兰语、英语、爱沙尼亚语、芬兰语、法语、加利西亚语、德语、希腊语、希伯来语、印地语、匈牙利语、冰岛语、印度尼西亚语、意大利语、日语、卡纳达语、哈萨克语、韩语、拉脱维亚语、立陶宛语、马其顿语、马来语、马拉地语、毛利语、尼泊尔语、挪威语、波斯语、波兰语、葡萄牙语、罗马尼亚语、俄语、塞尔维亚语、斯洛伐克语、斯洛文尼亚语、西班牙语、斯瓦希里语、瑞典语、他加禄语、泰米尔语、泰语、土耳其语、乌克兰语、乌尔都语、越南语和威尔士语。

## 流式传输实时音频
语音 API 使用块传输编码提供对实时音频流的支持。这意味着可以在生成完整文件并可供访问之前播放音频。

程序升级到 Python SDK v1.2，使用 `pip install --upgrade openai`

	from openai import OpenAI
	
	client = OpenAI()
	
	response = client.audio.speech.create(
	    model="tts-1",
	    voice="alloy",
	    input="Hello world! This is a streaming test.",
	)
	
	response.stream_to_file("output.mp3")

## 常问问题
- 如何控制生成的音频的情感范围？

	没有直接的机制来控制所生成的音频的情感输出。某些因素可能会影响输出音频，例如大小写或语法，但我们对这些因素的内部测试产生了不同的结果。
- 我可以创建自己声音的自定义副本吗？

	不，这不是我们支持的。
- 我拥有输出的音频文件吗？

	是的，就像我们 API 的所有输出一样，创建它们的人拥有输出。您仍然需要告知最终用户，他们听到的是人工智能生成的音频，而不是真人与他们交谈。

