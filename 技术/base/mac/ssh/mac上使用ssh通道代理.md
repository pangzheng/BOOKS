# Mac OSX ssh隧道使用方法
    文档信息
    创建人 庞铮
    邮件地址 zpang@dataman-inc.com
    建立时间 2015年6月28号
    更新时间 2016年3月22号
## 前言
>刚刚转到mac上工作，原先在win的ssh隧道使用的是SecureCRT上的，现在转到mac上，虽然也有CRT但应该有更廉价的方法。so...

## 方法一（ssh命令方法）

打开终端，然后直接用命令行

`ssh -i /path/to/key -D 1080 -C -N remoteuser@remote.host`

* -D 设置动态转发端口号；
* -C 启用压缩；
* -N 不执行远程shell命令(ssh2支持)，登录后不会有提示行；
* -i 优先使用秘钥key 而不是密码；

## 方法二（Mac GUI软件） 
点击下载[ssh tunnel manager](http://www.macupdate.com/app/mac/10128/ssh-tunnel-manager)按照步骤安装完毕后启动添加跳板服务器通道设置。

***需要注意几个问题***

* 代理使用的是Socks4;
* 高级选项建议点选Handle authentication\Compress;
* 加密建议选择aes256-cbc;
* 建议不选择auto connecet否则启动就会自动连接;

#### 关于多私钥问题
[ssh tunnel manager](http://www.macupdate.com/app/mac/10128/ssh-tunnel-manager)不支持密钥选择，如果多密钥或者密钥不在默认.ssh目录下此程序将无法正常运转。

##### 使用ssh自带解决办法
新增ssh的配置文件，并修改权限

`touch ~/.ssh/config`

`chmod 600 ~/.ssh/config`

修改config文件的内容
    
    Host *.workdomain.com            #hostname or ip
    IdentityFile ~/.ssh/id_rsa.work  #密钥所在地  
    User lee                         #用户名
    Port 22                          #端口号
 
    Host github.com
    IdentityFile ~/.ssh/id_rsa.github
    User git
    Port 5500
    
    Host 10.3.10.* 10.3.20.*
    User jyliu
    Port 22
    IdentityFile ~/.ssh/pri.pem
    ProxyCommand nc -x 127.0.0.1:1080 %h %p
    
这样在登陆的时候，ssh会根据登陆不同的域来读取相应的私钥文件
ssh 10.3.10.33 可以直接通过隧道1080端口直接登录到内网主机

##### config文件参数说明
    Host *
    选项“Host”只对能够匹配后面字串的计算机有效。“*”表示 所有的计算机。
    
    ForwardAgent no
    “ForwardAgent”设置连接是否经过验证代理（如果存在）转发给远程计 算机。
    
    ForwardX11 no
    “ForwardX11”设置X11连接是否被自动重定向到安全的通道和显示集 （DISPLAY set）。
    
    RhostsAuthentication no
    “RhostsAuthentication”设 置是否使用基于rhosts的安全验证。
    
    RhostsRSAAuthentication no
    “RhostsRSAAuthentication” 设置是否使用用RSA算法的基于rhosts的安全验证。
    
    RSAAuthentication yes
    “RSAAuthentication” 设置是否使用RSA算法进行安全验证。
    
    PasswordAuthentication yes
    “PasswordAuthentication” 设置是否使用口令验证。
    
    FallBackToRsh no
    “FallBackToRsh”设置如果用ssh连接出现错误是否自动 使用rsh。
    
    UseRsh no
    “UseRsh”设置是否在这台计算机上使用“rlogin/rsh”。
    
    BatchMode no
    “BatchMode”如果设为“yes”，passphrase/password（交互式输入口令）的提示将被禁止。当不能交互式输入 口令的时候，这个选项对脚本文件和批处理任务十分有用。
    
    CheckHostIP yes
    “CheckHostIP”设置ssh是 否查看连接到服务器的主机的IP地址以防止DNS欺骗。建议设置为“yes”。
    
    StrictHostKeyChecking no
    “StrictHostKeyChecking” 如果设置成“yes”，ssh就不会自动把计算机的密匙加入“$HOME/.ssh/known_hosts”文件，并且一旦计算机的密匙发生了变化，就 拒绝连接。
    
    IdentityFile ~/.ssh/identity
    “IdentityFile”设置从哪个文件读取用户的 RSA安全验证标识。
    
    Port 22
    “Port”设置连接到远程主机的端口。
    
    Cipher blowfish
    “Cipher” 设置加密用的密码。
    
    EscapeChar ~
    “EscapeChar”设置escape字符。


###参考资料
[Mac OSX上的ssh tunnel 代理设置备忘](http://www.chedong.com/blog/archives/001403.html)

[ssh_config 文件配置详解](http://blog.chinaunix.net/uid-8389195-id-1741604.html)
