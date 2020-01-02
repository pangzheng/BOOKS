# Virtualenv
## 1 简介
virtualenv 为每个不同项目提供一份 Python 安装。它并没有真正安装多个 Python 副本，但是它确实提供了一种巧妙的方式来让各项目环境保持独立。

## 2 命令行安装
### 2.1 MAC
    sudo pip install virtualenv
### 2.2 创建项目环境
    $ mkdir myproject
    $ cd myproject
    $ virtualenv venv
    New python executable in venv/bin/python
    Installing distribute............done.   
### 2.3 控制台进入venv 环境
    . venv/bin/activate
    # 在新的 venv 安装pip
    pip install Flask     
## 3 PyCharm 使用 ven
### 3.1 Configure
1. 打开PyCharm
2. 在 PyCharm 界面点击 `Configure --> Preferences`
### 3.2 Preferences
1. 在弹出的窗口搜索，`project`、`interpreter` 等关键字均可
2. 然后找到 `Project Interpreter`
### 3.3 Project Interpreter
- 空项目
    1. 点击该界面的最右侧的一个锯齿形状的按钮
    2. 在下拉列表中有个 "Create VirtualEnv" 选项
    3. 在弹出的对话框中输入要配置的环境信息
        a. Name中输入名称，如flask
        - Location选择
        
                选择配置好的virtualenv的默认目录，如/Users/oper/.virtualenvs/flask
        - Base interpreter：
    
                默认就好(我的是：/System/Library/Frameworks/Python.framework/Versions/2.7/bin/python2.7）
- 创建venv项目
    - 将上面的 "Create VirtualEnv"  改成 add
## 4 参考
[PyCharm中使用virtualenv](https://segmentfault.com/a/1190000003758895)    
                
    