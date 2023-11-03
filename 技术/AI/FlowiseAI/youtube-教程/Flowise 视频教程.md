# Flowise 教程
## 教程1-介绍和安装
Flowise 是一个使用 langchain 和 node.js 构建的 GPT 开源工具，使用易于使用的用户界面轻松构建 Langchain 应用程序.它还支持 API 调用
### 安装
参考官方文档
### 简化使用翻译模版
参考其他文档
### 参考
[Flowise AI Tutorial #1 - Introduction & Setup](https://www.youtube.com/watch?v=tD6fwQyUIJE)

## 教程2-创建 ChatFlows
创建新的 Flow 中使用的 Flowisea 组件，如果想要了解详细的信息，请访问 [https://js.langchain.com/docs](https://js.langchain.com/docs)

想要从任何 Flow 获取到响应，必须拥有两种不同节点，

- Chain 节点

	从应用程序生成某种输出的最基本节点，包含很多种 chain 节点，包括 LLM Chain 这种基础与与大模型交互的节点模型，还有高级的 Conversation Chain 这种与 LLM 来回交互的节点模型，它的用途是用来记住过去的对话
- Agent 节点 

	Agent 节点与 Chain 节点的区别就在于，Agent 节点可以利用其他第三方工具(如互联网接口、计算器接口等)来进行支持回答，而 Chain 节点只会使用 LLM 模型自带数据

###  Chain 节点
####  基于LLM 模型设置
- 创建一个 LLM Chian 节点

	![](./pic/llmchain.png)
- 再添加一个 LLM 模型节点

	该模型比较适合一次性交互的场景，比如生成什么，或者翻译什么
	
	- 参数
		- ModeName

				text-davinci-003 
		- Temperatre(可设置 0-1,越接近 1 给 AI 的想象力越高)

				0.7
	![](./pic/llmchain1.png) 
- 链接 LLM 和 LLM Chain

	![](./pic/llmchain2.png)
- 添加 prompt

	![](./pic/llmchain3.png)
- 设置 prompt 的模版

	![](./pic/llmchain5.png)	
- 链接 prompt 与 LLM Chain

	![](./pic/llmchain4.png)
- 记录就可以测试了
- 模版变量

	可以在模版中用大括号来设置变量，比如这里用户输入什么就用什么当成主题制作笑话
	
	![](./pic/llmchain6.png)
- 模版变量-设定值

	![](./pic/llmchain7.png)
	![](./pic/llmchain8.png)
	![](./pic/llmchain9.png)
- 模版变量-从用户输入获取

	![](./pic/llmchain10.png)
	![](./pic/llmchain11.png)
	![](./pic/llmchain12.png)
	
	![](./pic/llmchain13.png)

#### 基于 Chat 模型设置
- 创建一个 LLM Chian 节点

	![](./pic/llmchain.png)
- 再添加一个 Chat 模型节点

	该模型比较适合做角色扮演
	
	- 参数
		- ModeName

				gpt-3.5-turbo
		- Temperatre(可设置 0-1,越接近 1 给 AI 的想象力越高)

				0.7

	![](./pic/llmchain14.png)
- 添加 chat prompt

	![](./pic/llmchain15.png)	
- 设置 prompt 的模版	

	![](./pic/llmchain16.png)
- 角色扮演对话

	![](./pic/llmchain17.png)		
	
### Conversation Chain
与 LLM Chain 区别在于， Conversation Chain 更适合做聊天链节点，可以提供记忆上下文来回答问题。

- 上面的例子仅保留 Chat 模型节点

	![](./pic/llmchain18.png)	
- 添加 Conversation Chain 节点

	![](./pic/llmchain20.png)	
- 添加记忆节点

	用来在缓存中记录，可以使用默认值

	![](./pic/llmchain19.png)
- 测试 AI 记忆

	![](./pic/llmchain21.png)	

### Agent 节点
### Conversation Agent
- 创建 Agent

	![](./pic/ConversationAgent.png)
- 创建 Chat 模型

	![](./pic/ConversationAgent1.png)				
- 添加记忆节点

	用来在缓存中记录，可以使用默认值

	![](./pic/llmchain19.png)
- 创建工具节点
	- google 搜索引擎节点 

		![](./pic/ConversationAgent2.png)
	- 计算器节点

		![](./pic/ConversationAgent3.png)
	- 连线

		![](./pic/ConversationAgent4.png)
- 测试

	![](./pic/ConversationAgent5.png)		
							
### 参考
[Flowise AI Tutorial #2 - Creating ChatFlows (LLM Chains, Chat Models & Agents)](https://www.youtube.com/watch?v=fn4GCZuiwdk)
## 教程3-文件载入、分割、embeddings
和向量数据库
### GPT 分析基础
- 现实的理想

	![](./pic/Flieloaders.png)
- GPT 的样子
	
	为了回答问题，需要将童谣放到 GPT 的上下文中，类似这样
	
	![](./pic/Flieloaders1.png)
- 问题

	在于 Token 限制，为了解决这个问题，上下文理想状态只需要将有用信息的上下文加载来进行信息压缩

### 文字拆分器
- 拆分原理

	![](./pic/textsplitter.png)
- 文档

	langchain 的文档和文档文件定义不同，langchain 的文档是文本拆分器获取的文本块的行定义，包含
	
	- 拆分后的行内容
	- metadata
		- flename
		- 页数
		- 其他 
	
	
	![](./pic/Documents.png)

### Embeddings
将文件拆分成文档后，就需要存储到向量数据库中，但首先需要将文档信息变成向量数组,而向量数组是 AI 可以识别的东西。

而将文档信息变成向量信息的方法就是调用 Embedding 函数，一种嵌入式算法。注意在这个是对不同的大语言模型，该模型是不同的。比如 openai 模型，就要调用 openAI embedding 函数

![](./pic/embeddings.png)
### 向量数据库
向量数据库就是将向量数组、文件和元数据作为记录储存在其中

![](./pic/verctorStores.png)

除此之外，向量存储还将相似的文本片段和相似的嵌入式分组到集群中

所以要搜索谁是玛丽亚，就需要去向量数据库搜索玛丽亚相关性最高的信息列表给 GPT，默认 Flowisea 仅使用前4条返回结果当成上下文，大大减少了所需的 Token 数量

![](./pic/verctorStores1.png)
### 文档问答 QA 节点
- 创建 QA chain

	![](./pic/ConversationalRetrievalQAChain.png)
- 创建 chat 模型和 memory

	![](./pic/ConversationalRetrievalQAChain1.png)
- 创建向量数据库

	这里使用内存数据库，生产可能需要转换其他数据库
	
	![](./pic/ConversationalRetrievalQAChain2.png)
- 创建文档加载器

	有很多，这里使用 pdf 加载器
	![](./pic/ConversationalRetrievalQAChain3.png)
- 文字拆分器

	这里使用递归分割器
	
	- 参数
		- ChunkSize(块大小)

			这里默认1000，我们使用200，这个块越小，匹配的数据相对精确和少可以节省 token
		- Chunk Overlap(块重叠)

			这个意思是，块之前和之后一部分块字符也要引入，这里写20	 
	
	![](./pic/ConversationalRetrievalQAChain4.png)	
	![](./pic/ConversationalRetrievalQAChain5.png)	
- embeddings

	![](./pic/ConversationalRetrievalQAChain6.png)	
	![](./pic/ConversationalRetrievalQAChain7.png)		
### 参考
[Flowise AI Tutorial #3 - File Loaders, Text Splitters, Embeddings & Vector Stores](https://www.youtube.com/watch?v=kMtf9sNIcao)

## 教程4-向量数据库插入和查询(基于 Pinecone)
### 问题
上一节使用的是 Flowise 自带的内存向量数据库，这个数据库的好处在于方便使用，简单易懂。但也存在问题

- 基于内存的向量数据库，需要每次触发都要运行完整的流程，这对小文档(500单词以内)是非常有利的但是对于大文档，每次查询需要先加载再查询就相当耗时
- 基于内存的数据库在服务器异常的时候会导致数据丢失，导致服务中断

基于以上问题，生产级需要使用更好用的向量数据库，将向量数据库的操作进行读写分离，这样在 LLM 请求的时候，只从向量数据库中读取存在的向量，增加效率。

下面教程选择了 Pinecone(向量数据库 saas) 作为我们向量数据库，首先到 [pinecone.io](pinecone.io) 创建一个账户

### 创建索引的页面
![](./pic/pinecone.png)

- 创建索引
	- 参数
		- `index name`
			- 索引名称，全局唯一 
		- `Dimensions`
			- 向量精度，这里默认使用 1536 (openai 精度参数)
		- `metric`
			- 指标默认
		- `pod type`
			- pod 类型，这里默认    
			 
		![](./pic/pinecone1.png)
	- 创建过程
		- 初始化 	
			
			![](./pic/pinecone2.png)
		- 启动完成
			- 索引列表
			
				![](./pic/pinecone3.png)
			- 索引详情
			
				![](./pic/pinecone4.png)
-  apikeys
	- 创建
		
		![](./pic/pinecone-flow5.png)
	- 获取

		![](./pic/pinecone-flow6.png)
	- flowise 设置 apikeys
		- 创建
		
			![](./pic/pinecone-flow7.png)
		- 填入参数

			![](./pic/pinecone-flow8.png)
		- 对照参数

			![](./pic/pinecone-flow9.png)		  

### 创建提取文档 Flow
该 Flow 的目的是将文档转换成向量并存储到向量数据库中
	
- 创建 Flow 名称

	![](./pic/pinecone-flow.png)
- 创建一个交流检索 QA Chain	 
	- 这里需要3个链接
	
	![](./pic/pinecone-flow1.png)
- 创建第一个链接语言模型

	![](./pic/pinecone-flow2.png)	
- 建立第二个链接记忆体

	![](./pic/pinecone-flow3.png)		
- 建立向量数据库 
	- 注意这里选择的是 upsert 文档节点
		- 注意这里多了2个链接  

		![](./pic/pinecone-flow4.png)		
	- 设置参数
		- Connect Credential

			选择上面设置的 pinecone apikey 授权
		- pinecone index

			设置同 pinecone index name 相同的值	
		![](./pic/pinecone-flow10.png)	
	- 创建第一个链接文档
		- 注意这里需要1个链接
		
			![](./pic/pinecone-flow11.png)	
		- 创建一个文档分割器

			![](./pic/pinecone-flow12.png)
	- 创建第二个链接 Embedding 函数

		![](./pic/pinecone-flow13.png)
- 完整 Flow

	![](./pic/pinecone-flow16.png)		

### 测试提取文档 Flow 载入数据
- 加载文档

	![](./pic/pinecone-flow14.png)
- 保存

	![](./pic/pinecone-flow15.png)
- 查看 pinecone 向量值

	![](./pic/pinecone-flow17.png)	
- 询问 AI 一个问题用来加载数据

	![](./pic/pinecone-flow18.png)
- 再查看向量值
	- 注意每问一个新问题，向量就会增加 

	![](./pic/pinecone-flow19.png)
			
### 创建基于 pinecone 向量库的 QA 系统
- 创建一个 QA Flow

	![](./pic/pinecone-flow20.png)
- 创建一个交流检索 QA Chain	 
	- 这里需要3个链接
	
	![](./pic/pinecone-flow1.png)
- 创建第一个链接语言模型

	![](./pic/pinecone-flow2.png)	
- 建立第二个链接记忆体

	![](./pic/pinecone-flow3.png)			
- 建立向量数据库 
	- 注意这里选择的是 load existing index 文档节点
		- 注意这里多了1个链接  

		![](./pic/pinecone-flow21.png)
		
		- 设置参数
			- Connect Credential
	
				与 upsert 节点相同
			- pinecone index
	
				与 upsert 节点相同
		- 创建一个链接 Embedding 函数

			![](./pic/pinecone-flow22.png)
- 完整

	![](./pic/pinecone-flow24.png)	
- 询问问题

	![](./pic/pinecone-flow23.png)

### 整体系统体验问题
- 向量数据库是 paas 系统，需要更换成本地自部署系统才可以提速和降低成本
- 文档切割的大小影响了回答，如果是比较长的系统性说明，文档切割需要更大一些，比如默认 1000 ，这样回答会比较完整，但是这样会导致浪费一些 gpt token
- 回答整体不如内存数据库精确，问题需要再查询		

### 参考
[Flowise AI Tutorial #4 - Vector Store Injest & Query (incl. Pinecone)](https://www.youtube.com/watch?v=m0nr1_pnAxc)
## 教程5-云部署-雷达
暂不翻译
### 参考
[Flowise AI Tutorial #5 - Deploying to Render](https://www.youtube.com/watch?v=Fxyc6-frgrI)
## 教程6-提示词链
提示词链允许多个链和模型为应用程序输出，创建高级人工智能成为可能，下面就使用3个链来的例子来展示 demo 的能力

我们要创建一个应用使用的链，在该链中包含3个链

- 第一个成份链 Agent 设定提示模版

	根据公共假期名称提供匹配食谱对应的成分

		You are an Al assistant which will respond with a suitable main ingredient for a recipe based on a public holiday provided by the user.
		
		Public Holiday:{holiday}

		你是一个人工智能助手，它将根据用户提供的公共假日为食谱提供合适的主要成分。
		
		公共假日:{假日}
- 第二个厨师链 Agent 设定提示模版

	根据公共假期名称和第一个模型，为连锁店提供的主要成分生成一个唯一的食谱 

		You are an experienced chef that creates unique food recipes based on a public holiday and a main ingredlent that matches that holiday.
		
		Public Holiday:{holiday}
		
		Main Ingredient:{ingredient}
		
		你是一个经验丰富的厨师，创造独特的食物食谱的基础上，公共假日和一个主要的成分，以配合那个节日。
		
		公共假日:{假日}
		主要成分:{成分}
- 第三个美食评论链 Agent 设定提示模版
	
	根据连锁店提供的分步说明的食谱和成分列表，让评论家 agent 对连锁店菜谱进行评论
		
		You are a food critic who will review a food recipe based on a public holiday.

		Please note that only comment information should be included in the output, and no other information should be outputted.
		
		The output information should be presented in Chinese.
		
		Public Holiday: {holiday}
		Recipe: {recipe}
		
		你是一位美食评论家，将根据一个公共假日来评价食谱。

		请注意，输出信息只包含评论信息，其他信息均不输出。
		
		输出信息应以中文形式呈现。
		
		公共假日：{holiday}
		食谱：{recipe}
		
### 创建 Demo  prompt 链
- 创建 Flow

	![](./pic/prompt-chain.png)
- 成分链
	- 制作成分链
	
		![](./pic/prompt-chain1.png)
	- 将节日变量设置为 chat 问题
	
		![](./pic/prompt-chain2.png)
- 厨师链
	- 先修改成分链的输出为输出预期	
		
		![](./pic/prompt-chain5.png)
	- 再做厨师链
	
		![](./pic/prompt-chain3.png)
	- 链接两条链

		![](./pic/prompt-chain6.png)
	- 再设置节日变量和成分变量

		![](./pic/prompt-chain4.png)
	- 测试

		![](./pic/prompt-chain7.png)
	- 查看 Flowise AI 控制台日志

		![](./pic/prompt-chain8.png)
- 评论家链
	- 将厨师链修改成输出预期

		![](./pic/prompt-chain9.png)
	- 制作评论家链

		![](./pic/prompt-chain11.png)
	- 链接两条链

		![](./pic/prompt-chain12.png)
	- 设置节日和菜单变量

		![](./pic/prompt-chain10.png)
	- 测试

		![](./pic/prompt-chain13.png)	
- 系统问题

	输出的信息有点奇怪，总体是根模型有关，我们将评论家模型切换到 chatgpt
			
	![](./pic/prompt-chain14.png)
- 整体结构

	![](./pic/prompt-chain15.png)

### 参考
[Flowise AI Tutorial #6 - Prompt Chaining](https://www.youtube.com/watch?v=IIwhWyIkWkU)

## 教程7-使用嵌入 API 将聊天机器人添加到网站
这个教程用例介绍了完整的客服机器人训练到使用，具体参考之前的文档，这里不重复

注意，这里为了保证客服文档的向量使用的有效性，将块大小设置为 500，覆盖设置为 100 ，这个参数需要进行测试，我们当前是 1000:20
### 参考
[Flowise AI Tutorial #7 - Adding Chatbots to Websites using Embed API](https://www.youtube.com/watch?v=XOeCV1xyN48)

## 教程8-API 和 APIkeys 
本教程介绍了 Flowise AI 通过 API 和 APIkey 与应用程式整合，将 Flowise AP 内嵌到程序内部使用
### 参考
[Flowise AI Tutorial #8 - API Endpoints, API Keys (with Node)](https://www.youtube.com/watch?v=LhN560DhlzU)
## 教程9-使用本地模型、localAI、GPT4ALL
本教程是使用免费的开源模型，使用本地模型的好处

- 免费
- 无需联网
- 对话保证私有

### 安装 localAI 准备
- 代码库 clone

	https://github.com/go-skynet/LocalAI


### 参考
[Flowise AI Tutorial #9 - Using Local Models, LocalAI, GPT4ALL](https://www.youtube.com/watch?v=0B0oIs8NS9k)
				



老师,我发您看哈
chatGPT
midjourney
ideogram.ai
pika(https://www.pika.art/)
runway 或runwayGEN2(Gen2是runway里重要的工具,runway所有工具能装一起吗?...)
D-iD(https://www.d-id.com/)
clipdrop(https://clipdrop.co/)
lalamu(https://lalamu.studio/demo)
hithaw(https://www.hitpaw.com/)
elevenlabs(https://elevenlabs.io/)
			
		

		
								