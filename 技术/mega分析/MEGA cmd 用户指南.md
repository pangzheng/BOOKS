# MEGA cmd 用户指南
## 账户/联系人
- signup email [password] [--name="Your Name"] 

	使用填入的电子邮件注册
- confirmlink email [password] 

	使用 “注册” 过程后提供的链接确认帐户
- invite [-d|-r] dstemail [--message="MESSAGE"] 

	邀请联系人/删除邀请.
- showpcr [--in | --out] 

	显示传入和传出的联系人请求
- ipc email|handle -a|-d|-i 

	管理联系人传入的邀请.
- users[-s] [-h] [-n] [-d contact@email] 

	列出联系人
- userattr[-s attribute value|attribute] [--user=user@email] 

	列出/更新 用户属性
- passwd [oldpassword newpassword] 

	修改用户密码
- masterkey pathtosave 

	显示您的主密钥.

## 登陆/注销
- login [email [password]] | exportedfolderurl#key | session 

	登陆
- logout [--keep-session] 

	登出
- whoami[-l] 

	打印当前用户
- session 

	打印会话ID
- killsession[-a|sessionid] 

	终止用户会话 
	
## 文件游览
- cd [remotepath] 

	更改当前的远程文件夹
- lcd[localpath] 

	更改命令行控制台的本机文件夹
- ls [-lRr] [remotepath] 

	列出远程路径中的文件
- pwd 

	打印当前的远程文件夹
- lpwd 

	打印命令行控制台的当前本地文件夹
- attr remotepath [-s attribute value|-d attribute] 

	列出/更新节点属性
- du [-h] [remotepath remotepath2 remotepath3 ... ] 

	打印文件/文件夹大小
- find [remotepath] [-l] [--pattern=PATTERN] [--mtime=TIMECONSTRAIN] [-size=SIZECONSTRAIN] 

	查找与模式匹配的节点
- mount 

	列出所有主要节点

## 复制和移动文件
- mkdir[-p] remotepath 

	 创建远程目录或目录层次结构
- cp srcremotepath dstremotepath|dstemail 

	将文件/文件夹复制到新位置 (全部在远程)
- put[-c] [-q] [--ignore-quota-warn] localfile [localfile2 localfile3 ...] [dstremotepath] 

	将文件/文件夹上载到远程文件夹
- get[-m] [-q] [--ignore-quota-warn] exportedlink#key|remotepath [localpath] 

	下载远程文件/文件夹或公共链接
- preview [-s] remotepath localpath 

	下载/上传文件预览
- thumbnail[-s] remotepath localpath 

	下载/上传文件缩略图
- mv srcremotepath [srcremotepath2 srcremotepath3 ..] dstremotepath 

	将文件/文件夹移动到新位置 (全部在远程)
- rm[-r] [-f] remotepath 

	删除远程文件/文件夹
- transfers[-c TAG|-a] | [-r TAG|-a] | [-p TAG|-a] [--only-downloads | --only-uploads] [SHOWOPTIONS] 

	列出或进行共享
- speedlimit[-u|-d] [-h] [NEWLIMIT] 

	显示/修改上传/下载速率限制
- sync [localpath dstremotepath| [-dsr] [ID|localpath] 

	控制同步
- exclude [(-a|-d) pattern1 pattern2 pattern3 [--restart-syncs]] 

	同步管理排除项
- backup localpath remotepath --period="PERIODSTRING" --num-backups=N 

	设置新的备份文件夹和/或计划时间表
- backup [-lhda] [TAG|localpath] [--period="PERIODSTRING"] [--num-backups=N]) 

	查看/修改现有备份时间表

## 共享
- cp srcremotepath dstremotepath|dstemail 

	将文件/文件夹移动到新位置（全部在远程）
- export [-d|-a [--expire=TIMEDELAY]] [remotepath] 

	打印/修改当前出口的状态
- impor texportedfilelink#key [remotepath] 

	将远程链接的内容导入您的帐户
- share [-p] [-d|-a --with=user@email.com [--level=LEVEL]] [remotepath] 

	打印/修改当前共享的状态
- webdav [ [-d] remotepath [--port=PORT] [--public] [--tls --certificate=/path/to/certificate.pem --key=/path/to/certificate.key]] 

	设置通过PC /设备从MEGA帐户下载文件的功能.

## 其他
- version [-l][-c] 

	打印MEGAcmd版本和其他信息
- deleteversions [-f] (--all | remotepath1 remotepath2 ...)

	删除文件的先前版本以节省空间
- unicode 

	在交互式 Shell 中切换 Unicode 输入启用/禁用
- reload 

	强制重载用户的远程数据
- help[-f] 

	打印命令列表
- https[on|off] 

	显示 HTTPS 是否用于传输。使用 https on 来启用它
- clear

	清除屏幕
- log[-sc] level 

	打印/修改当前日志级别
- debug 

	进入 debug 调试模式（详细）
- exit | quit [--only-shell] 

	退出命令行
	
## 参考
[UserGuide.md](https://github.com/meganz/MEGAcmd/blob/master/UserGuide.md)	