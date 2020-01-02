# Run SZ RZ on Mac With iTerm2
    文档信息
    创建人 庞铮
    邮件地址 zpang@dataman-inc.com
    建立时间 2015年6月30号
    更新时间 2015年7月6号
## 1 说明
windows 下 secureCRT 的上传下载，win下 secureCRT 通过 ZModem 协议在远程服务器和终端机器间上传下载文件,只需要敲个rz sz就能方便的互传文件了，无需通过 scp 或者 ftp 命令的麻烦操作。
最近[iTerm2](http://www.iterm2.com/#/section/downloads) 的开发版新出了一个 trigger 功能，通过调用 rz、sz命令实现了相同的功能，iTerm Build 1.0.0.20120724以上版本开始支持。
## 2 安装
###  2.1 安装iTerm2
iterm2是开源的mac-ssh终端工具[点击下载安装](http://www.iterm2.com/downloads.html)
###  2.2 安装Homebrew
说明:brew是苹果的一个安装命令行安装工具,类似linux的yum和apt-get[使用方法](http://brew.sh/index_zh-cn.html)

安装方法
    
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
### 2.3 安装需要插件
#### 2.3.1 安装lrzsz
安装方法

    sudo brew install lrzsz

#### 2.3.2 安装wget
安装方法
    
    sudo brew install wget
    
#### 2.3.3 安装iterm2脚本
    mkdir -p /usr/local/lrzsz && cd /usr/local/lrzsz
    sudo wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-send-zmodem.sh
    sudo wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-recv-zmodem.sh
    chmod -R 755  /usr/local/lrzsz/*
#### 2.3.4 设置iterm2触发器
打开Iterm2 → Preferences → advanced → triggers -Edit

    Regular expression: \*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/lrzsz/iterm2-send-zmodem.sh
 
    Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/lrzsz/iterm2-recv-zmodem.sh
## 3 使用方法
### 3.1 从服务器上下载download.sh
    sz download.sh
### 3.2 从本地上传download.sh
    rz ->  选择mac上文件
### 3.3 注意中文文件可能会有异常
## 4 参考
[Mac使用rz、sz远程上传下载文件](http://www.ipython.me/other/mac-using-lrzsz.html)
[iterm2-zmodem](https://github.com/mmastrac/iterm2-zmodem)
[SZ RZ on Mac with iTerm2](http://www.libiao.name/SZ-RZ-on-Mac-with-iTerm2.html)
