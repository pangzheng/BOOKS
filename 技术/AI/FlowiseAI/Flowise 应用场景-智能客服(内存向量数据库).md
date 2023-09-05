# Flowise 应用场景-智能客服(内存向量数据库)
## 智能客服
### 需求
- 根据文档内容进行回答
- 文档没有的内容回答不知道
- 文档相关的内容可以有一定的联想

### 分析需求
- 分析文档功能
	- 文档属性包含
		- 网络获取 url(爬虫)
		- text
		- pdf
		- makedown
		- doc(微软)
	- 方案选择
		- 上述文档除了 text 全部需要转成 text 字符串，然后进行于智能客服上下文沟通，本次实验局限于 PDF 文档
- 多语言问题
	- 如果产品需要进行多语言兼容，那么需要提供的资料也是多语言的，当然使用单一语种也可以进行多语言回答，但是效果不佳
	- 解决方案
		- 当前解决方案是使用多语言文本多 flow 来进行隔离
		- 多语言文本单flow 还需要测试
		- 其他解决方案待观察
	- 方案选择
		- 本次实验选择多语言多 FLow 版本
- 超出 AI token 的问题
	- 当前文档大小不可控，可能会有多文档，所以超出 token 范围可能性极高，超出 token 范围后，AI 回答想要保证精确性，当今有几种解决方案
	- 解决方案
		- 向量数据库
			- 好处
				- 通过向量数据库的羽翼分析，可以进一步减小每次问题的请求 token 数量
				- 相对精确
			- 坏处
				- 增加技术复杂度，需要引入向量数据库
				- 对于比较范围话的问题，可能导致 AI 无法回答     
		- 使用支持更高 token 的大预言模型
			- 好处
				- 全文作为上下文，AI 回答会非常精确
				- 不引入新的技术点，技术复杂度降低
			- 坏处
				- 每次 token 数量是全文上下文数量，成本高       
	- 方案选择
		- 本次实验选择 FLOW 内带内存向量数据库，减少复杂度
		- 后面方案升级，引入成熟的向量数据库 	

## 准备步骤
- 准备好人工智能分析数据
- 升级最新的 Flow AI 版本，老版本缺少新模版支持
- 准备好 Flow AI 基础知识
- 准备好智能客服的人格设定          

## 实施步骤
### 创建授权
- 登陆 Flow AI

	![](./pic/FlowAI模版.png)
- 创建第三方服务认证，这里我们只和 OpenAI 交互，所以只创建 OpenAI apikey 即可

 	![](./pic/FlowAI模版1.png)	
 	![](./pic/FlowAI模版2.png)	
	![](./pic/FlowAI模版3.png)	

### 选择模版
- 找寻和你需求最接近的模版

	在这个模版模式下进行修改

	![](./pic/FlowAI模版4.png)	
- 点击保存模版	

	![](./pic/FlowAI模版5.png)	

### 修改 FLow 实例	
- 删除 Text 文件输入，否则这个模块没有数据输入的话会报错

	![](./pic/FlowAI模版6.png)	 
	![](./pic/FlowAI模版7.png)
- 更换向量数据库		 

	默认是 pinecone 第三方服务向量数据库，修改成内存向量数据库
	
	![](./pic/FlowAI模版8.png)
	
	- 记忆向量数据库的连线模式

		![](./pic/FlowAI模版9.png)
	- 删除该插件

		![](./pic/FlowAI模版10.png)
	- 添加内存向量数据库插件

		![](./pic/FlowAI模版11.png)
		![](./pic/FlowAI模版12.png)		

### 配置 Flow 插件参数
- 文字分割器
	- Chunk size
		- 1000
	- Chunk overlap
		- 200
	
	![](./pic/FlowAI模版13.png)	
- PDF 输入
	- 使用配置
		- One document per fille
	
	![](./pic/FlowAI模版14.png)
- 内嵌 openAI 选择链接认证
	- Connect Credential*
		- 选择刚才设置的 openai 授权
		
	![](./pic/FlowAI模版15.png)
- 聊天对话器
	- Connect Credential*
		- 选择刚才设置的 openai 授权
	- Model Name
		- gpt-3.5-turbo

			说明下为什么选择这个，因为这个最便宜，而且上下文长度够用，涌现也 ok，当然你也可以使用 gpt 4 或者 gpt3.5 t 16k ，但价格更高
	- Temperature
		- 0

		这里的这个温度，类似 sd 的幻想程度， 0 是没幻想 
			
	![](./pic/FlowAI模版16.png)
- 内存向量数据库

	这个配置默认即可
	
	![](./pic/FlowAI模版17.png)
- 人工智能客服人设

		I want you to act as a document that l am having a conversation with.   Yourname is "chainstroage 智能客服".   You will provide me with answers from the given info.   lfthe answer is not included, say exactly "对不起，这个问题我无法回答", and stop afterthat.   Refuse to answer any question not about the info.   Never break character.
		
	注意这里为什么使用英文和中文，因为这个 Flow 使用的是中文智能客服，所以回答语句都是中文，但是人设语句设置英文，是因为节省 token 成本，其他参数默认
	
	![](./pic/FlowAI模版18.png)
- 设置 Flow 名称

	![](./pic/FlowAI模版19.png)
- 点击保存

	![](./pic/FlowAI模版20.png)		

### 导入手册数据
- 拿出准备的产品用户手册数据，点击上传文件

	这个过程根据文档大小，时间不同，因为它要利用 ai 初始化到向量数据库中
	
	![](./pic/FlowAI模版28.png)	
	
### 嵌入式参数配置
因为这个代码是需要配置到登陆应用程序页面的，所以需要一定的配置

- 配置参考

	是用共享智能客服调节参数，参数包含
	
	- 开头聊天提示信息
		- Welcome Message
	- 聊天模版各种颜色

		这里我选默认
	- 智能客服的娃娃头像
	- 用户的对话头像
	- 等待对话框内提示信息
		- Textlnput Placeholder

	![](./pic/FlowAI模版21.png)		
	![](./pic/FlowAI模版22.png)
- 点击打开生成页面调试
	- ![](./pic/FlowAI模版23.png)
	- ![](./pic/FlowAI模版24.png)	
- 查询对应嵌入式代码配置
	- 访问嵌入式代码网站
		- [FlowiseAI/FlowiseChatEmbed](https://github.com/FlowiseAI/FlowiseChatEmbed)
		- 查看配置

			![](./pic/FlowAI模版25.png)	
	- 根据网站提示调整嵌入式代码

			<script type="module">
			    import Chatbot from "https://cdn.jsdelivr.net/npm/flowise-embed/dist/web.js"
			    Chatbot.init({
			        chatflowid: "ba3e1370-316b-435d-94dd-c6debcfd701f",
			        apiHost: "http://xx14.38:3000",
			        chatflowConfig: {
			        // topK: 2
			        },
			    theme: {
			      button: {
			        backgroundColor: "#3B81F6",
			        right: 20,
			        bottom: 20,
			        size: "medium",
			        iconColor: "white",
			        customIconSrc:
			          "https://raw.githubusercontent.com/walkxcode/dashboard-icons/main/svg/google-messages.svg",
			      },
			      chatWindow: {
			        welcomeMessage: "嗨！你好，我是 chainstorage 的智能客服，请问有什么可以帮助你的麽?",
			        backgroundColor: "#ffffff",
			        height: 700,
			        width: 400,
			        fontSize: 16,
			        poweredByTextColor: "#303235",
			        botMessage: {
			          backgroundColor: "#f7f8ff",
			          textColor: "#303235",
			          showAvatar: true,
			          avatarSrc:
			            "https://raw.githubusercontent.com/zahidkhawaja/langchain-chat-nextjs/main/public/parroticon.png",
			        },
			        userMessage: {
			          backgroundColor: "#3B81F6",
			          textColor: "#ffffff",
			          showAvatar: true,
			          avatarSrc:
			            "https://raw.githubusercontent.com/zahidkhawaja/langchain-chat-nextjs/main/public/usericon.png",
			        },
			        textInput: {
			          placeholder: "这里设置你的问题",
			          backgroundColor: "#ffffff",
			          textColor: "#303235",
			          sendButtonColor: "#3B81F6",
			        },
			      },
			    },
			  });
			</script>
- 最后在本机系统中增加一个 robots.html 文件，将代码 cp 到里面，点击打开测试

	![](./pic/FlowAI模版26.png)
- 这里查看智能客服结果

	![](./pic/FlowAI模版27.png)							
	
		