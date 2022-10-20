# conda
适用于任何语言（Python、R、Ruby、Lua、Scala、Java、JavaScript、C/C++、Fortran 等）的包、依赖项和环境管理。

Conda 是一个开源包管理系统和环境管理系统，可在 Windows、macOS、Linux 和 z/OS 上运行。 Conda 可以快速安装、运行和更新软件包及其依赖项。 Conda 在本地计算机上轻松创建、保存、加载和切换环境。它是为 Python 程序创建的，但它可以打包和分发任何语言的软件。

Conda 作为包管理器可帮助您查找和安装包。如果你需要一个包需要不同版本的 Python，你不需要切换到不同的环境管理器，因为 conda 也是一个环境管理器。只需几个命令，您就可以设置一个完全独立的环境来运行不同版本的 Python，同时继续在正常环境中运行常用版本的 Python。

在其默认配置中，conda 可以在 repo.anaconda.com 上安装和管理由 Anaconda® 构建、审查和维护的数千个包。

Conda 可以与 Travis CI 和 AppVeyor 等持续集成系统结合使用，为您的代码提供频繁的自动化测试。

conda 包和环境管理器包含在 Anaconda 和 Miniconda 的所有版本中。

Conda 还包含在 Anaconda Enterprise 中，为 Python、R、Node.js、Java 和其他应用程序堆栈提供现场企业包和环境管理。 Conda 也可以在社区频道 conda-forge 上找到。您可能还会在 PyPI 上获得 conda，但这种方法可能不是最新的。

## conda 入门
Conda 是一个强大的包管理器和环境管理器，您可以在 Windows 的 Anaconda Prompt 或 macOS 或 Linux 的终端窗口中与命令行命令一起使用。

这个 20 分钟的 conda 入门指南可让您试用 conda 的主要功能。 完成本指南后，您应该了解 conda 的工作原理。

还请参见：Anaconda Navigator 入门，这是一个图形用户界面，可让您在类似 Web 的界面中使用 conda，而无需输入手动命令。 比较每个入门指南，看看您喜欢哪个程序。
### 开始 conda
#### windows
- 在“开始”菜单中，搜索并打开“Anaconda Prompt”。

	![](./pic/anaconda-prompt.png)
	
在 Windows 上，以下所有命令都输入到 Anaconda Prompt 窗口中。

#### MacOS
- 打开 Launchpad，然后单击终端图标。

在 macOS 上，以下所有命令都输入到终端窗口中。

#### Linux
- 打开一个终端窗口。

在 Linux 上，以下所有命令都输入到终端窗口中。

### 管理 conda
通过键入以下内容验证 conda 是否已安装并在您的系统上运行：

	conda --version
Conda 显示您已安装的版本号。 您无需导航到 Anaconda 目录。

例如:

	% conda --version
	conda 4.12.0	
如果您收到错误消息，请确保在安装后关闭并重新打开终端窗口或者立即执行。 然后验证您是否登录到用于安装 Anaconda 或 Miniconda 的同一用户帐户。

将 conda 更新到当前版本。 键入以下内容

	conda update conda
Conda 比较版本，然后显示可安装的内容。

如果有更新版本的 conda 可用，请键入 y 进行更新：

	Proceed ([y]/n)? y
我们建议您始终将 conda 更新到最新版本。
### 管理环境
Conda 允许您创建单独的环境，其中包含不会与其他环境交互的文件、包及其依赖项。

当您开始使用 conda 时，您已经有了一个名为 base 的默认环境。 但是，您不想将程序放入您的基础环境中。 创建单独的环境以使您的程序彼此隔离。

- 创建一个新环境并在其中安装一个包。

	我们将环境命名为 `snowflakes ` 并安装包 BioPython。 在 Anaconda 提示符或终端窗口中，键入以下内容：
	
		conda create --name snowflakes biopython
	Conda 检查 BioPython 需要哪些附加包（“依赖项”），并询问您是否要继续：
	
		Proceed ([y]/n)? y
	键入“y”并按 Enter 继续。	
- 要使用或“activate”新环境，请键入以下内容
	- 仅适用于 conda 4.6 及更高版本。 
		- Windows:

				conda activate snowflakes
		- macOS and Linux

				conda activate snowflakes
	
	现在您处于 `snowflakes` 环境中，您键入的任何 conda 命令都将转到该环境，直到您将其停用。
- 要查看所有环境的列表，请键入

		conda info --envs
	将出现一个环境列表，类似于以下内容：
	
		conda environments:

		    base           /home/username/Anaconda3
		    snowflakes   * /home/username/Anaconda3/envs/snowflakes						
## 参考
[conda docs](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html#starting-conda)		    
	

	
		
	