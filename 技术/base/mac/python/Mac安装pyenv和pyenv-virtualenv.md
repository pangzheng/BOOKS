# Mac安装pyenv和pyenv-virtualenv
## pyenv 是做什么的
pyenv 是用来更方便的管理/切换 python 版本的
## 安装
### 准备
- 安装 brew
- 安装所需应用

		# brew install openssl readline xz
- 安装 pyenv

		# brew install pyenv
		# echo 'eval "$(pyenv init -)"' >> ~/.zshrc
- 安装 pyenv-virtualenv				

		# git clone https://github.com/yyuu/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
- zshrc
	- 激活
	
			# echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
	- 重新启动shell，以使路径更改生效
	
			 # exec $SHELL && source ~/.zshrc
- bash
	- mac

			echo 'export PATH="/root/.pyenv/bin:$PATH"' >> ~/.bash_profile
			echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
			echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
	- linux

			echo 'export PATH="/root/.pyenv/bin:$PATH"' >> ~/.bashrc
			echo 'eval "$(pyenv init -)"' >> ~/.bashrc
			echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
		
### 命令集
- pyenv
	- pyenv install --list
    
        	查询所有可以安装的版本     
   - pyenv install 2.7.14
        
        	安装所需的版本
   - pyenv uninstall

    		卸载特定的Python版本。
	- pyenv version

			显示当前活动的Python版本
	- pyenv global 2.7.14

			Python的全局设置，整个系统生效
    - pyenv global 2.7.14

    		Python的局部设置，当前目录生效
    - pyenv local --unset
    
    		取消设置    
- pyenv-virtualenv
	- pyenv virtualenv 2.7.14 venv2714

			制定版本创建virtualenv
	- pyenv virtualenvs

			列出现有virtualenvs
	- pyenv activate virtualenv的名称

			激活pyenv virtualenv
    - pyenv deactivate
    
    		停用pyenv virtualenv
	- pyenv uninstall my-virtual-env
    
    		删除现有virtualenv

### 安装 python 3.8 在 pyenv
- 缓存

		v=3.8.0;wget https://npm.taobao.org/mirrors/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/
- 安装

		pyenv install 3.8.0
		   
## 参考
[Mac安装pyenv和pyenv-virtualenv](https://www.jianshu.com/p/13e300c63abd)    		