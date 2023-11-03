# langchain-安装
## 正式发布
要安装 LangChain 运行：

- pip

		pip install langchain
- conda
	
		conda install langchain -c conda-forge

这将安装 LangChain 的最低要求。LangChain 的很多价值在于将其与各种模型提供程序、数据存储等集成。默认情况下，未安装执行此操作所需的依赖项。然而，还有另外两种安装 LangChain 的方法可以引入这些依赖项。

要安装常见 LLM 提供商所需的模块，请运行：

	pip install langchain[llms]
要安装所有集成所需的所有模块，请运行：

	pip install langchain[all]
请注意，如果您使用zsh，则在将它们作为参数传递给命令时需要引用方括号，例如：

	pip install 'langchain[all]'

## 从源安装
如果您想从源安装，可以通过克隆存储库并确保该目录正在 `PATH/TO/REPO/langchain/libs/langchain` 运行来实现：

	pip install -e .