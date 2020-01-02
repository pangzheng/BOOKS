# GVIM-Macvim配置

    文档信息
    创建人 庞铮
    邮件地址 zpang@dataman-inc.com
    建立时间 2015年7月8号
    更新时间 2015年7月8号
## 1 说明
Mac系统本身带vim，但是只能在终端使用，这样想检查一个.sh文件，系统默认直接执行。所以在图形界面也需要一个gvim－macvim.

## 2 安装
### 2.1 查看最新版本
[点击查看最新版本](https://github.com/macvim-dev/macvim/releases)
### 2.2 下载安装
以Snapshot 77版本为例[点击下载](https://github.com/macvim-dev/macvim/releases/download/snapshot-77/MacVim-snapshot-77.tbz),点击解压。

    mv ~/Downloads/MacVim-snapshot-77/MacVim.app ~/Applications/MacVim.app/
### 2.3 配置文件修改

    vi Applications/MacVim.app/Contents/Resources/vim/vimrc
####  2.3.1 解决乱码
默认安装打开中文会乱码，需要调整。
参数

    set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
####  2.3.2 字体大小
默认打开字体过小，需要将字体调整。
参数

    set guifont=Monaco:h16
####  2.3.3 其他参数
剩下的就按照需求自己调节，我设置的一些参数

    "设置编码
    set encoding=utf-8
    "中文编码设置
    set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
    "字体大小
    set guifont=Monaco:h16
    "自行设置内容
    "高亮设置
    syntax on
    "设置高亮搜索
    set hlsearch
    "行号
    "set nu!
    "自动折行
    set wrap
    "输入字符串就显示匹配点
    set incsearch
    "输入命令显示出来
    set showcmd
    set nocompatible
    "设置tab距离
    "保存一个TAB是4个字符
    set tabstop=4
    "按一次TAB是前进4个字符
    set softtabstop=4
    set expandtab
    "shifwidth写代码时用到，缩进为4字符
    set shiftwidth=4
    "颜色类型
    colorscheme torte
    "colorscheme shine
    set ruler 
