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

		(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ pip install -q openai langchain playwright beautifulsoup4
		
	这个命令会安装四个库：
		
	- `openai`
		- 用于与OpenAI的GPT模型交互。
	- `langchain`
		- 一个用于语言模型应用的工具库。
	- `playwright`
		- 一个用于自动化浏览器操作的库。
	- `beautifulsoup4`
		- 用于解析和操作网页内容的库。
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

		import argparse  # 导入解析命令行参数的库
		import asyncio  # 导入异步I/O库
		import html2text  # 导入HTML转Markdown的库
		from langchain.document_loaders import AsyncChromiumLoader  # 从langchain导入异步Chromium加载器
		import requests # 导入用于HTTP请求的库
		
		# AI 部分
		Axxxxx9ea771b9"
		headers = {"Authorization": "Bearer At2WtopwADjayghNc16y/xxxxxxxxxxxxx="}
		
		def query(payload):
		    response = requests.post(API_URL, headers=headers, json=payload)# 向API发送POST请求并获取响应
		    return response.json() # 将响应转换为JSON格式
		
		# 爬虫部分
		# 定义一个异步函数来抓取网页并转换为Markdown
		async def scrape_and_convert_to_md(url):
		    loader = AsyncChromiumLoader([url])  # 创建一个加载器实例
		    html = await loader.ascrape_playwright(url)  # 异步加载指定URL的网页内容
		    h = html2text.HTML2Text()  # 创建HTML到Markdown的转换器
		    markdown = h.handle(html)  # 将HTML内容转换为Markdown
		    return markdown  # 返回Markdown内容
		
		def main():
		    parser = argparse.ArgumentParser(description='Web scraper that converts HTML to Markdown.')  # 创建命令行解析器
		    parser.add_argument('url', type=str, help='The URL to scrape')  # 添加URL参数
		    args = parser.parse_args()  # 解析命令行参数
		
		    loop = asyncio.get_event_loop()  # 获取当前事件循环
		    markdown_content = loop.run_until_complete(scrape_and_convert_to_md(args.url))  # 运行异步函数并等待结果
		    print("==========================html_markdown=========================")  # 打印Markdown内容
		    print(markdown_content)  # 打印Markdown内容
		    print("==========================html_markdown=========================")  # 打印Markdown内容
		
		    # AI 部分
		#    output = query({
		#        "question": markdown_content, # 将Markdown内容作为问题发送到API
		#    })
		#    print(output)
		
		# 当脚本直接运行时执行
		if __name__ == "__main__":
		    main()

- 运行测试

		(acl) pangzheng@iZt4n86o3ror4rdm8mmpiyZ:~/AsyncChromiumLoader$ python test2.py url
- 退出 python 虚拟机

		deactivate

## 遇到问题解决
### 问题1 无头浏览器30秒超时

### 问题2 页面大小超过 上下文长度 limit 限制
- 查询了下，当前通用解决方案是数据汇总需要引入向量数据库，那么整体的设计就完全不同了

### 问题3 js解析异常(kugga)
https://www.kugga.com/space/post-detail/411

## 参考
- [Web scraping](https://python.langchain.com/docs/use_cases/web_scraping)
- [基于LangChain实现AI爬虫，爬取网页内容](https://www.tizi365.com/article/107.html)