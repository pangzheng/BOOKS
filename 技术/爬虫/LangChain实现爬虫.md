# LangChain实现爬虫.md
从网络爬取内容的一些关键步骤如下：

- 搜索：查询URL（例如使用 GoogleSearchAPIWrapper）。
- 加载：将URL转换成HTML（例如使用AsyncHtmlLoader、AsyncChromiumLoader等）。
- 转换：将HTML转换为格式化文本（例如使用HTML2Text或Beautiful Soup）。

## 部署前准备
- 硬件

	使用 flowise 测试主机(ubuntu 系统)
- 启动环境

	fastapi_poe 要求使用 python3 版本，ubuntu 默认 3.8 所以可以使用
	
		$ python3 --version
		Python 3.8.10
- 安装 python virtualenv 虚拟机
	- 检查虚拟机
	
			$ virtualenv poe
	
				Command 'virtualenv' not found, but can be installed with:
				
				apt install python3-virtualenv
				Please ask your administrator.
	- 安装

			$ sudo apt install python3-virtualenv
	- 创建项目目录		

			$ mkdir  AsyncChromiumLoader/
	- 进入目录

			$ cd AsyncChromiumLoader/
	- 创建虚拟机		
	
			~/AsyncChromiumLoader$ virtualenv acl

			created virtual environment CPython3.8.10.final.0-64 in 159ms
			  creator CPython3Posix(dest=/home/pangzheng/AsyncChromiumLoader/acl, clear=False, global=False)
			  seeder FromAppData(download=False, pip=latest, setuptools=latest, wheel=latest, pkg_resources=latest, via=copy, app_data_dir=/home/pangzheng/.local/share/virtualenv/seed-app-data/v1.0.1.debian.1)
			  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator				

## 安装爬虫依赖环境
- 进入虚拟机

		$ source acl/bin/activate
- 安装依赖库

		(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ pip install -q openai langchain playwright beautifulsoup4 tiktoken
		
	这个命令会安装四个库：
		
	- `openai`
		- 用于与OpenAI的GPT模型交互。
	- `langchain`
		- 一个用于语言模型应用的工具库。
	- `playwright`
		- 一个用于自动化浏览器操作的库。
	- `beautifulsoup4`
		- 用于解析和操作网页内容的库。
	- `tiktoken`
		- 切割器，用于按照需求切割文档长度
	- `-q`
		-  参数表示安装过程中输出较少的日志信息。
- 安装浏览器以及依赖库
	- 安装依赖库

			(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$sudo playwright install-deps
			
			Get:1 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]
			Hit:2 http://mirrors.cloud.aliyuncs.com/ubuntu focal InRelease
			Get:3 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [33.7 kB]
			...
			update-alternatives: using /usr/share/icons/Adwaita/cursor.theme to provide /usr/share/icons/default/index.theme (x-cursor-theme) in auto mode
			Setting up libgtk-3-0:amd64 (3.24.20-0ubuntu1.1) ...
			Setting up humanity-icon-theme (0.6.15) ...
			Setting up ubuntu-mono (19.04-0ubuntu3) ...
			Processing triggers for systemd (245.4-4ubuntu3.21) ...
			Processing triggers for man-db (2.9.1-1) ...
			Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
			Processing triggers for dictionaries-common (1.28.1) ...
			Processing triggers for libgdk-pixbuf2.0-0:amd64 (2.40.0+dfsg-3ubuntu0.4) ...
		- 关联错误

			如果不安装，执行会报

				...
				playwright._impl._api_types.Error:
				╔══════════════════════════════════════════════════════╗
				║ Host system is missing dependencies to run browsers. ║
				║ Please install them with the following command:      ║
				║                                                      ║
				║     sudo playwright install-deps                     ║
				║                                                      ║
				║ Alternatively, use apt:                              ║
				║     sudo apt-get install libatk1.0-0\                ║
				║         libatk-bridge2.0-0\                          ║
				║         libxkbcommon0\                               ║
				║         libatspi2.0-0\                               ║
				║         libxcomposite1\                              ║
				║         libxdamage1\                                 ║
				║         libxfixes3\                                  ║
				║         libxrandr2\                                  ║
				║         libgbm1\                                     ║
				║         libpango-1.0-0\                              ║
				║         libcairo2                                    ║
				║                                                      ║
				║ <3 Playwright Team                                   ║
				╚══════════════════════════════════════════════════════╝
	- 安装浏览器
	
		这个命令会安装 Playwright 需要的浏览器。Playwright 用于自动化Web 浏览器操作，需要特定的浏览器版本来运行
	
			(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ playwright install
			Downloading Chromium 119.0.6045.9 (playwright build v1084) from https://playwright.azureedge.net/builds/chromium/1084/chromium-linux.zip
			155.8 Mb [====================] 100% 0.0s
			Chromium 119.0.6045.9 (playwright build v1084) downloaded to /home/pangzheng/.cache/ms-playwright/chromium-1084
			Downloading FFMPEG playwright build v1009 from https://playwright.azureedge.net/builds/ffmpeg/1009/ffmpeg-linux.zip
			2.6 Mb [====================] 100% 0.0s
			FFMPEG playwright build v1009 downloaded to /home/pangzheng/.cache/ms-playwright/ffmpeg-1009
			Downloading Firefox 118.0.1 (playwright build v1425) from https://playwright.azureedge.net/builds/firefox/1425/firefox-ubuntu-20.04.zip
			80.8 Mb [====================] 100% 0.0s
			Firefox 118.0.1 (playwright build v1425) downloaded to /home/pangzheng/.cache/ms-playwright/firefox-1425
			Downloading Webkit 17.4 (playwright build v1921) from https://playwright.azureedge.net/builds/webkit/1921/webkit-ubuntu-20.04.zip
			123.7 Mb [====================] 100% 0.0s
			Webkit 17.4 (playwright build v1921) downloaded to /home/pangzheng/.cache/ms-playwright/webkit-1921

- 升级 openai v1.2 

	程序升级到 Python SDK v1.2
	
		(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ pip install --upgrade openai
			
### 测试爬虫代码
- 创建测试代码

		(poe) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/fastapi_poe$ vi poe-demo.py
- 将测试代码贴入
	- 通过 bs 筛选标签的方式

			import argparse  # 导入解析命令行参数的库
			from langchain.document_loaders import AsyncChromiumLoader  # 从langchain导入异步Chromium加载器
			from langchain.document_transformers import BeautifulSoupTransformer # 从 langchain 导入 BeautifulSoupTransformer
			from langchain.text_splitter import RecursiveCharacterTextSplitter # 导入文档切割器
			
			# 爬虫部分
			# 定义一个函数来抓取网页并转换为bs
			def scrape_and_convert_to_bs(url):
			    # 爬虫
			    loader = AsyncChromiumLoader([url])  # 创建一个加载器实例
			    html = loader.load() # 加载指定URL的网页内容
			
			    # html2bs
			    bs_transformer = BeautifulSoupTransformer()
			    docs_transformed = bs_transformer.transform_documents(
			            html, tags_to_extract=["div","p"]
			    )
			
			    # 切割器，切割成 gpt 的 limit 长度
			    splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
			        chunk_size=1000, chunk_overlap=0
			    )
			    splits = splitter.split_documents(docs_transformed)
			    content = splits[0].page_content
			
			    # print(content)
			    return content  # 返回bs内容
			
			def main():
			    # 命令行参数
			    parser = argparse.ArgumentParser(description='Web scraper that converts HTML to bs text.')  # 创建命令行解析器
			    parser.add_argument('url', type=str, help='The URL to scrape')  # 添加URL参数
			    args = parser.parse_args()  # 解析命令行参数
			    
			    # 调用函数
			    bs_content = scrape_and_convert_to_bs(args.url)
			
			    # 将结果写入到文件
			    with open('bs.txt', 'w') as file:
			        print(bs_content, file=file)
			
			# 当脚本直接运行时执行
			if __name__ == "__main__":
			    main()
	- 通过 html2text 转换成 md 格式

			import argparse  # 导入解析命令行参数的库
			from langchain.document_loaders import AsyncChromiumLoader  # 从langchain导入Chromium加载器
			from langchain.document_transformers import Html2TextTransformer # 从langchain 导入 Html2Text
			from langchain.text_splitter import RecursiveCharacterTextSplitter # 导入文档切割器
			
			# 爬虫部分
			# 定义一个函数来抓取网页并转换为 Markdown
			def scrape_and_convert_to_md(url):
			    # 爬虫
			    loader = AsyncChromiumLoader([url])  # 创建一个加载器实例
			    html = loader.load()
			
			    # 转换 md
			    html2text = Html2TextTransformer()
			    docs_transformed = html2text.transform_documents(html)
		
			    # 切割器，切割成 gpt 的limit 长度
			    splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
			        chunk_size=1000, chunk_overlap=0
			    )
			    splits = splitter.split_documents(docs_transformed)
			    content = splits[0].page_content
			
			    return content  # 返回 Markdown 内容
			
			def main():
			    # 参数说明
			    parser = argparse.ArgumentParser(description='Web scraper that converts HTML to Markdown.')  # 创建命令行解析器
			    parser.add_argument('url', type=str, help='The URL to scrape')  # 添加URL参数
			    args = parser.parse_args()  # 解析命令行参数
			
			    markdown_content = scrape_and_convert_to_md(args.url)  # 运行异步函数并等待结果
			
			    # 将结果写入到文件
			    with open('md.txt', 'w') as file:
			        print(markdown_content, file=file)
			
			# 当脚本直接运行时执行
			if __name__ == "__main__":
			    main()			    
		
- 运行测试

		(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ python test2.py url
- 退出 python 虚拟机

		deactivate

## 遇到问题解决
### 问题1  waiting until "load" 加载错误
- 运行报错

		==========================html_markdown=========================
		Error: Timeout 30000ms exceeded. =========================== logs
		=========================== navigating to
		"https://www.reuters.com/world/us/us-thwarts-plot-kill-sikh-separatist-issues-
		warning-india-ft-2023-11-22/", waiting until "load"
		============================================================
- 分析原因
	- 原因可能1
		- 处理时间可能短，需要修改 timeout
	
			查看代码

				#加载浏览器部分
				...
				async def scrape_and_convert_to_md(url):
					loader = AsyncChromiumLoader([url])  # 创建一个加载器实例
					html = await loader.ascrape_playwright(url)  # 异步加载指定URL的网页内容
				...
		- 关键函数  `AsyncChromiumLoader ` 
		
			查看 `AsyncChromiumLoader ` 代码,位置在 [这里](https://github.com/langchain-ai/langchain/blob/bcf83988ecb72ae6bc0bfa3f93f4c9b1ada1d728/libs/langchain/langchain/document_loaders/chromium.py#L1)
		
				....
				async def ascrape_playwright(self, url: str) -> str:
			        """
			        使用Playwright的异步API异步抓取给定URL的内容。
			
			        参数:
			            url (str): 要抓取的URL。
			
			        返回:
			            str: 抓取的HTML内容，如果出现异常则返回错误消息。
			
			        """
			        from playwright.async_api import async_playwright
			
			        logger.info("开始抓取...")
			        results = ""
			        async with async_playwright() as p:
			            browser = await p.chromium.launch(headless=True)  
			            try:
			                page = await browser.new_page()
			                await page.goto(url)  <---- 这里
			                results = await page.content()  # 直接获取HTML内容
			                logger.info("内容已抓取")
			            except Exception as e:
			                results = f"错误: {e}"
			            await browser.close()
			        return results
			  	.....
		- 关键函数 `async_playwright`	
			- 在 playwright.dev [这里](https://playwright.dev/docs/test-timeouts)查询得到可以配置超时参数，但这个是 ts 配置方法，python 文档没有标注
			- 1 询问 GPT
				- `python playwright config 在哪里设置`  
				- 回答

						from playwright.async_api import async_playwright
						
						async def run():
						    async with async_playwright() as p:
						        # 启动浏览器的配置在这里设置
						        browser = await p.chromium.launch(
						            headless=True,  # 无头模式
						            # 其他配置参数
						        )
						        # ... 后续代码 ..
			- 2 询问 GPT
				- `配置timeout 如何配置`
				- 回答

					在这个例子中，timeout=30000 设置了 goto 方法的超时时间为 30 秒。如果页面在 30 秒内没有完成加载，将抛出超时异常。

						from playwright.async_api import async_
						
						async def run():
						    async with async_playwright() as p:
						        browser = await p.chromium.launch(headless=True)
						        page = await browser.new_page()
						        # 设置页面加载的超时时间（以毫秒为单位）
						        await page.goto('https://example.com', timeout=30000)  # 30秒超时
						        # ... 后续代码 ...
						        await browser.close()
						
						asyncio.run(run())
		- 加100秒

				vi /home/pangzheng/AsyncChromiumLoader/acl/lib/python3.8/site-packages/langchain/document_loaders/chromium.py
				...
				await page.goto('https://example.com', timeout=100000)  # 100秒超时
				...
		- 测试，还是报错
		
				(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ python test3.py https://www.reuters.com/world/us/us-thwarts-plot-kill-sikh-separatist-issues-warning-india-ft-2023-11-22/
				==========================html_markdown=========================
				Error: Timeout 100000ms exceeded. =========================== logs
				=========================== navigating to
				"https://www.reuters.com/world/us/us-thwarts-plot-kill-sikh-separatist-issues-
				warning-india-ft-2023-11-22/", waiting until "load"
				============================================================
		- 加600秒

				vi /home/pangzheng/AsyncChromiumLoader/acl/lib/python3.8/site-packages/langchain/document_loaders/chromium.py
				...
				await page.goto('https://example.com', timeout=600000)  # 600秒超时
				...
		- 测试，再加上运行时间

			成功抓到数据，执行时间是248秒多

				(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ time python test3.py https://www.reuters.com/world/us/us-thwarts-plot-kill-sikh-separatist-issues-warning-india-ft-2023-11-22/
				
				.....
				![dot image
				pixel](https://sp.analytics.yahoo.com/sp.pl?a=10000&d=Thu%2C%2023%20Nov%202023%2006%3A20%3A54%20GMT&n=-8&b=US%20thwarted%20plot%20to%20kill%20Sikh%20separatist%20in%20America%20%7C%20Reuters&.yp=10160484&f=https%3A%2F%2Fwww.reuters.com%2Fworld%2Fus%2Fus-
				thwarts-plot-kill-sikh-separatist-issues-warning-india-
				ft-2023-11-22%2F&enc=UTF-8&us_privacy=1---&yv=1.15.1&tagmgr=gtm)![](https://t.co/i/adsct?bci=3&eci=2&event_id=4e96bc18-a90e-4ac8-8e32-1ece903c7ddc&events=%5B%5B%22pageview%22%2C%7B%7D%5D%5D&integration=advertiser&p_id=Twitter&p_user_id=0&pl_id=73efae03-0209-41e1-9b69-6337c7d4afe7&tw_document_href=https%3A%2F%2Fwww.reuters.com%2Fworld%2Fus%2Fus-
				thwarts-plot-kill-sikh-separatist-issues-warning-india-
				ft-2023-11-22%2F&tw_iframe_status=0&tw_order_quantity=0&tw_sale_amount=0&txn_id=o61xp&type=javascript&version=2.3.29)![](https://analytics.twitter.com/i/adsct?bci=3&eci=2&event_id=4e96bc18-a90e-4ac8-8e32-1ece903c7ddc&events=%5B%5B%22pageview%22%2C%7B%7D%5D%5D&integration=advertiser&p_id=Twitter&p_user_id=0&pl_id=73efae03-0209-41e1-9b69-6337c7d4afe7&tw_document_href=https%3A%2F%2Fwww.reuters.com%2Fworld%2Fus%2Fus-
				thwarts-plot-kill-sikh-separatist-issues-warning-india-
				ft-2023-11-22%2F&tw_iframe_status=0&tw_order_quantity=0&tw_sale_amount=0&txn_id=o61xp&type=javascript&version=2.3.29)![](https://ad-
				delivery.net/px.gif?ch=2)![](https://ad.doubleclick.net/favicon.ico?ad=300x250&ad_box_=1&adnet=1&showad=1&size=250x250)![](https://ad-
				delivery.net/px.gif?ch=1&e=0.4169835972951028)
				
				
				==========================html_markdown=========================
				
				real	4m8.831s
				user	0m11.064s
				sys	0m2.871s
- 第二次优化

	因为 playwright 支持多浏览器调用，更换了火狐和webkit，最终对比发现 webkit 支持更好，全部响应秒级别，而且可以全部打开
	
	测试时间方法
	
		time python test3.py https://www.nbcnews.com/investigations/israels-secret-air-war-gaza-west-bank-rcna126096
	修改方法
	
		vi /home/pangzheng/AsyncChromiumLoader/acl/lib/python3.8/site-packages/langchain/document_loaders/chromium.py
	
		....
		async def ascrape_playwright(self, url: str) -> str:
        """
        Asynchronously scrape the content of a given URL using Playwright's async API.

        Args:
            url (str): The URL to scrape.

        Returns:
            str: The scraped HTML content or an error message if an exception occurs.

        """
        from playwright.async_api import async_playwright

        logger.info("Starting scraping...")
        results = ""
        async with async_playwright() as p:
            browser = await p.chromium.launch(headless=True)
            # browser = await p.firefox.launch(headless=True)
            #browser = await p.webkit.launch(headless=True)
            try:
                #context = await browser.new_context(java_script_enabled=True)
                #page = await context.new_page()
                page = await browser.new_page()
                #await page.goto(url)
                await page.goto(url,timeout=600000)

                results = await page.content()  # Simply get the HTML content
                logger.info("Content scraped")
            except Exception as e:
                results = f"Error: {e}"
            await browser.close()
        return results
	
### 问题2 页面大小超过 上下文长度 limit 限制
- 修改当前结构，使用向量对比处理(标签)
- 增加摘要拼接方法？需要查询

### 问题3 js解析异常(kugga)
https://www.kugga.com/space/post-detail/411

## 参考
- [Web scraping](https://python.langchain.com/docs/use_cases/web_scraping)
- [基于LangChain实现AI爬虫，爬取网页内容](https://www.tizi365.com/article/107.html)