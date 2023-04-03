# ipfs更新
ipfs-update 工具是一个命令行实用程序，可用于安装和更新 Kubo。安装 ipfs-update 的最简单方法是使用预构建的二进制文件，详情如下。查看 [项目存储库](https://github.com/ipfs/ipfs-update#from-source) （打开新窗口）如果您更愿意从源代码构建它。
## 安装 ipfs-update
您可以从下载预构建的二进制文件 `dist.ipfs.tech` （打开新窗口）. 二进制文件也可从 [IPFS 更新 GitHub 发布页面获得](https://github.com/ipfs/ipfs-update/releases) （打开新窗口）.

### win
1. 下载 Windows 二进制文件dist.ipfs.tech （打开新窗口）.

		wget https://dist.ipfs.tech/ipfs-update/v1.9.0/ipfs-update_v1.9.0_windows-amd64.zip -Outfile ipfs-update_v1.9.0_windows-amd64.zip
2. 将文件解压缩到一个合理的位置，例如~\Apps\ipfs-update_v1.9.0：

		Expand-Archive -Path ipfs-update_v1.9.0_windows-amd64.zip -DestinationPath ~\Apps\ipfs-update_v1.9.0
3. 移动到ipfs-update_v1.9.0文件夹：

		cd Apps\ipfs-update_v1.9.0\ipfs-update\
4. 检查是否ipfs-update.exe有效：

		.\ipfs-update.exe --version

		> ipfs-update version 1.9.0
	此时，ipfs-update 就可用了。但是，强烈建议您先按照以下步骤添加 `ipfs-update.exe` 到您的 `PATH `中：
5. 打印当前工作目录并将其复制到剪贴板：

		pwd
		
		> Path
		> ----
		> C:\Users\<username>\Apps\ipfs-update_v1.9.0\ipfs-update
6. 检查 PowerShell 的配置文件是否已存在：

		Test-Path $profile
		
		> false
	如果配置文件已存在，请跳过下一步并继续执行步骤 8。

7. 创建一个新的 PowerShell 配置文件：

		New-Item -path $profile -type file –force
		
		> Mode                LastWriteTime         Length Name
		> ----                -------------         ------ ----
		> -a----        11/5/2020   6:38 PM              0 Microsoft.PowerShell_profile.ps1
8. 将第 5 步中复制的地址添加到 PowerShell 中，方法是将其添加到存储在以下文件PATH的末尾：`Microsoft.PowerShell_profile.ps1Documents\WindowsPowerShell`

		Add-Content C:\Users\<username>\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1 "[System.Environment]::SetEnvironmentVariable('PATH',`$Env:PATH+';;C:\Users\<username>\Apps\ipfs-update_v1.9.0\ipfs-update')"
9. 关闭并重新打开您的 PowerShell 窗口。
10. `PATH` 通过转到您的主文件夹并询问 `ipfs-update` 版本来测试您的设置是否正确：

		ipfs-update --version
		
		> ipfs-update version 1.9.0

提示

	如果加载配置文件时 PowerShell 下次启动时出错，请将 PowerShell 更改ExecutionPolicy为Unrestricted，如Microsoft PowerShell 文档中所述 （打开新窗口）.

### 苹果系统
1. 从下载 macOS 二进制文件 [dist.ipfs.tech](https://dist.ipfs.tech/#ipfs-update) （打开新窗口）.

		curl -O https://dist.ipfs.tech/ipfs-update/v1.9.0/ipfs-update_v1.9.0_darwin-amd64.tar.gz
2. 解压缩文件：

		tar -xvzf ipfs-update_v1.9.0_darwin-amd64.tar.gz
		
		> x ipfs-update/install.sh
		> x ipfs-update/ipfs-update
3. 移动到ipfs-update文件夹

		cd ipfs-update
4. 运行安装脚本：

		sudo bash install.sh
		
		> installed /usr/local/bin/ipfs-update
5. 检查是否 `ipfs-update` 安装正确：

		ipfs-update --version
		
		> ipfs-update version 1.9.0
		> 

### Linux
1. 下载 Linux 二进制文件 [dist.ipfs.tech](https://dist.ipfs.tech/#ipfs-update) （打开新窗口）.

		wget https://dist.ipfs.tech/ipfs-update/v1.9.0/ipfs-update_v1.9.0_linux-amd64.tar.gz
2. 解压缩文件：

		tar -xvzf ipfs-update_v1.9.0_linux-amd64.tar.gz
		
		> x ipfs-update/install.sh
		> x ipfs-update/ipfs-update
3. 移动到 `ipfs-update` 文件夹：

		cd ipfs-update
4. 并运行安装脚本：

		sudo bash install.sh
		
		> installed /usr/local/bin/ipfs-update
5. 测试是否 `ipfs-update` 安装正确：

		ipfs-update --version
		
		> ipfs-update version 1.9.0
		> 

## 安装 IPFS
ipfs-update 工具可用于安装 Kubo。

- 要安装特定的 Kubo `<version-number>`，请运行：

		ipfs-update install <version-number>
- 要安装最新版本的 Kubo，请使用latest标签：

		ipfs-update install latest

提示
		
	当 ipfs-update install 运行并且已经安装了一个版本的 IPFS 时，该版本被隐藏并且可以在以后恢复。
## 降级IPFS
使用 `revert` 函数回滚到以前版本的 Kubo：

	ipfs-update revert
`ipfs-update revert` 恢复到先前安装的 Kubo 版本。如果新安装的版本有问题并且您想切换回旧的稳定安装，这将很有用。
## 卸载更新程序
`ipfs-update` 要卸载 IPFS 更新，请从您的变量中删除二进制文件和 `PATH`。
### win
1. 找到文件的位置 `ipfs-update.exe`：

		gci -recurse -filter ipfs-update.exe -File -ErrorAction SilentlyContinue
		
		> Directory: C:\Users\<username>\Apps\ipfs-update_v1.9.0\ipfs-update
2. 删除 `ipfs-update` 目录：

		Remove-Item -Recurse -Force C:\Users\<username>\Apps\ipfs-update_v1.9.0
3. `ipfs-update` 从变量中删除目录 `PATH`。此过程因 Windows 安装而异，因此请查看 [Microsoft 文档了解详细信息](https://docs.microsoft.com/en-us/cpp/build/setting-the-path-and-environment-variables-for-command-line-builds?view=msvc-160) （打开新窗口）.

### 其他操作系统
1. 找到文件的位置 `ipfs-update`：

		sudo find / -name ipfs-update

		/usr/local/bin/ipfs-update
2. 删除文件：

		sudo rm /usr/local/bin/ipfs-update