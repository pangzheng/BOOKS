# Zapier+Flowies 消息采集分析工作流测试
这个任务的主要目的是通过  Zapier 中的 twitter 组件收集 twitter 需要的信息，然后再发给 AI 进行分析，最后所有数据落库保存
## 1 先觉条件
1. 熟练使用 Zapier、Flowise 
2. 部署 Flowise
	- 获取 GPT 计费 apikey
	- 获取 serpapi apikey 

### 1.1 创建 Flowise AI 评估工作流
#### 1.1.1 创建 apikey
- 点击创建 apikey

	![](./pic/zap-flowiseai12.png)
- 设置密钥名称

	![](./pic/zap-flowiseai13.png)	
- 查看结果

	![](./pic/zap-flowiseai14.png)
			
#### 1.1.2 模版实例化
1. 登录 Flowise AI 跳转到流程模版市场

	![](./pic/zap-flowiseai4.png)
- 暂时我们选择一个简单的 Agent 模版

	![](./pic/zap-flowiseai5.png)
- 检查一下模块，并点击使用该模版讲它实例化

	![](./pic/zap-flowiseai6.png)		

#### 1.1.3 配置实例
1. 配置 ChatGPT
	- 填写可用 apikey
	- 选择模型
		- gpt3.5-turbo-16k(当前最佳选择)
	- Temperature

		默认 0.9   

	![](./pic/zap-flowiseai7.png)
- 点击保存成实例
	- 点击保存
	
		![](./pic/zap-flowiseai9.png)
	- 设置实例名字

		![](./pic/zap-flowiseai10.png)
- 设置完成后

	![](./pic/zap-flowiseai11.png)
- 点击嵌入式按钮绑定 apikey

	![](./pic/zap-flowiseai15.png)
- 点击弹出菜单右上角，选中创建 apikey 环节对应的密钥名称

	![](./pic/zap-flowiseai16.png)
- 绑定成功后，内嵌和机器人均不可使用，提示

	![](./pic/zap-flowiseai17.png)
- 退出再次保存

	![](./pic/zap-flowiseai18.png)

#### 1.1.4 设置 agent 代理判断标准
- 打开文档编辑器设定你的 agent 代理评判信息和回复的规则

	类似

		你现在是一位专业的基金经理，请从金融角度评估输入的信息对于xx公司的影响，返回信息仅包含以下的信息


		"sentimentPolarity" : "正面或者负面，不要缠在其他信息",
		
		"impactLevel": 1-10,
		
		"briefComment":"简短50个字的评语"	
- 讲人格信息放入参数
	- 打开该工作流终端模块，点击添加参数

		![](./pic/zap-flowiseai19.png)
	- 把信息插入其中

		![](./pic/zap-flowiseai20.png)
	- 退出保存

		![](./pic/zap-flowiseai18.png)	

#### 1.1.5 点击测试机器人
- 点击系统内测试 bot 对话界面

	![](./pic/zap-flowiseai21.png)
- 输入模拟信息

	![](./pic/zap-flowiseai22.png)
- 得到回答

	![](./pic/zap-flowiseai23.png)	
		

### 1.2 创建 zap 表格
- 进入表格服务

	![](./pic/zap-zaptables.png)
- 点击创建

	![](./pic/zap-zaptables1.png)
- 填写表明

	这里可以使用模版或者倒入，我们新建
	
	![](./pic/zap-zaptables2.png)
- 添加字段

	![](./pic/zap-zaptables3.png)	
	
## 2 创建 zap 工作流			
### 2.1  设置启动
1. 前往 [zapier](https://zapier.com/app/dashboard)
2. 单击创建

	![](./pic/zap.png)

### 2.2 触发器设置
1. 单击搜索 twitter

	![](./pic/zap-trigger.png)
2. 选择 (SearchMention) 作为事件触发，点击下一步

	SearchMention 的意思是，任何一个是用 twitter 的人发布了新信息，就会进行触发
	
	![](./pic/zap-trigger1.png)
3. 授权账户
	- 授权提示，点击 yes

		![](./pic/zap-trigger2.png)
	- 跳转 twitter 系统，点击授权应用

		![](./pic/zap-trigger3.png)
	- 授权成功，点击继续

		![](./pic/zap-trigger4.png)	
4. 设置过滤器

	填写搜索到什么信息，进行任务触发
	
	![](./pic/zap-trigger6.png)
5. 测试
	- 跳出测试页面，点击测试

		![](./pic/zap-trigger5.png)
	- 如果是空

		到 twitter 上制作一个模拟测试数据	
	- 字段信息

			created_at Thu Aug 03 07:52:15 +0000 2023
			id 1687008465301184500
			id_str 1687008465301184513
			full_text xx环球 zapier 好
			truncated false
			display_text_range
				1 0
				2 13
			metadata
				iso_language_code zh
				result_type recent
			source
				<a href="https://mobile.twitter.com" rel="nofollow">Twitter Web App</a>
			in_reply_to_status_id
			in_reply_to_status_id_str
			in_reply_to_user_id
			in_reply_to_user_id_str
			in_reply_to_screen_name
			user
				id 1673979049323159600
				id_str 1673979049323159554
				name area shan
				screen_name shanyanzhou
				location
				description
				url
				entities
					description
						urls
				protected false
				followers_count 0
				friends_count 2
				listed_count 0
				created_at Wed Jun 28 08:58:14 +0000 2023
				favourites_count 0
				utc_offset
				time_zone
				geo_enabled false
				verified false
				statuses_count 2
				lang
				contributors_enabled false
				is_translator false
				is_translation_enabled false
				profile_background_color F5F8FA
				profile_background_image_url
				profile_background_image_url_https
				profile_background_tile false
				profile_image_url http://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png
				profile_image_url_https https://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png
				profile_link_color 1DA1F2
				profile_sidebar_border_color C0DEED
				profile_sidebar_fill_color DDEEF6
				profile_text_color 333333
				profile_use_background_image true
				has_extended_profile true
				default_profile true
				default_profile_image true
				following false
				follow_request_sent false
				notifications false
				translator_type none
				withheld_in_countries
			geo
			coordinates
			place
			contributors
			is_quote_status false
			retweet_count 0
			favorite_count 0
			favorited false
			retweeted false
			lang zh
			url https://twitter.com/shanyanzhou/status/1687008465301184513
			text xx环球 zapier 好
	- 测试完毕点击继续

		![](./pic/zap-trigger7.png)
- 完成触发器添加

	![](./pic/zap-trigger8.png)
								
### 2.3 过滤器设置
1. 点击添加过滤器

	![](./pic/zap-filter.png)
2. 设置不需要处理的信息
	- 过滤字段名
	- 过滤动作
	- 过滤值
	
	![](./pic/zap-filter2.png)
3. 点击继续

	![](./pic/zap-filter3.png)
4. 完成

	![](./pic/zap-filter4.png)
	
### 2.4 增加表格记录	
1. 点击事件添加 zap-tables 记录

	![](./pic/zap-zaptables4.png)
2. 选择之前建立的表

	![](./pic/zap-zaptables5.png)
3. 获取到对应字段

	![](./pic/zap-zaptables6.png)	
4. 填写获取 twitter api 字段数据

	![](./pic/zap-zaptables7.png)
5. 测试检查

	这里有5个字段有数据，1个没有
	
	![](./pic/zap-zaptables8.png)
6. 点击测试

	![](./pic/zap-zaptables9.png)
7. 从表格查看

	![](./pic/zap-zaptables10.png)	

### 2.5 添加 FlowiseAI
1. 点击添加事件搜索 FlowiseAI

	![](./pic/zap-flowiseai.png)
2. 添加预测事件

	![](./pic/zap-flowiseai1.png)
3. 点击添加 flowiseai

	![](./pic/zap-flowiseai2.png)
4. 添加你的 flowiseai 地址和 apikey

	![](./pic/zap-flowiseai24.png)
5. 添加成功，点击下一步

	![](./pic/zap-flowiseai3.png)
6. 选择 twitter 文字信息字段，点击下一步

	![](./pic/zap-flowiseai25.png)	
7. 设置 FlowAI 的 ID,这里选择的就是 Flow 界面的 Chatflows 里的名字

	![](./pic/zap-flowiseai27.png)	
	![](./pic/zap-flowiseai28.png)	
8. 进入测试页面，核准后点击测试

	![](./pic/zap-flowiseai26.png)
8. 查看测试结果

	![](./pic/zap-flowiseai29.png)

### 2.6 将 AI 评价写回数据库
1. 点击事件更新 zap-tables 记录

	![](./pic/zap-flowiseai30.png)
2. 选择之前建立的表

	![](./pic/zap-zaptables11.png)
3. 选择更新对应条目

	![](./pic/zap-zaptables12.png)	
4. 添加 AI 评估结果到评估字段，点击下一步

	![](./pic/zap-zaptables13.png)	
5. 点击测试

	![](./pic/zap-zaptables14.png)			

### 2.7 添加更多插件
这里不再描述
### 2.8 点击发布
![](./pic/zap1.png)

## 3 zap 工作流使用
### 3.1 查看工作流
- 查看 dashboard

	![](./pic/zap2.png)		
- 查看运行中的工作流详情
	- 查看工作流列表
	- 激活状态打开 

	![](./pic/zap3.png)

### 3.2 查看工作历史
- 点击查看历史

	![](./pic/zap4.png)
- 查看统计(有延迟)
	- 有1个工作流在执行
	- 有4个执行完毕的 task
	
	![](./pic/zap5.png)	

### 3.3 查新数据库	
- 查询数据库字段

	![](./pic/zap6.png)								
## 4 故障排错和系统监控
因为 zap 链接的是第三方的服务，所以工作流程中的每一个 SaaS 组件或程序都有可能出现问题而导致流程中断，这样导致整个业务流无法工作。

故此， zap 给出的方案是几个办法

- 客服机器人
- zap + 所有第三方服务系统监控报警系统

### 4.1 客服机器人处理
1. 点击屏幕又下脚 Chat ，打开 CS Bot 界面

	![](./pic/zap-csbot.png)
- 首先接受 zap 的安全协议,机器人才能为你服务

	![](./pic/zap-csbot2.png)
- 后面机器人会引导你使用选择或者对话的方式获取问题并尝试给出修复建议

	![](./pic/zap-csbot3.png)
- 比如我打字说明我的 twitter 的查询触发器无法接收到数据

	![](./pic/zap-csbot5.png)

### 4.2 服务和第三方应用问题
官方给了一个整体服务的状态页面，可以通过这个页面查看自己流程中的应用是否正常

[https://status.zapier.com/#app-status](https://status.zapier.com/#app-status)

- 比如查询 zap 服务

	![](./pic/zap-status.png)
- 比如查询应用
	
	![](./pic/zap-status1.png)
	
## 5 特别提示
- twitter 的订阅信息无法收到问题

	在 twitter 的信息订阅后，模拟测试发现不一定什么信息都可以收到，后来发现可能是因为 twitter api 过滤了历史信息，只会提供新信息，所以测试的时候需要提供新信息给 twitter 	
	
## 参考
- [zapier](https://zapier.com/)
		

	

								
				