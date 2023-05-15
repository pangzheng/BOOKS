# Ubantu Linux install sd
## 准备工作
- 主机端操作
	- 云端操作 
		- 选择主机
			- 最好选择带 N 卡 gpu
			- 选择  t4 
			- 自己装驱动
		- 创建主机 
	- 服务器操作
		- 使用 root 创建使用用户
		
			stable-diffusion-webui 安装不允许使用 root 
			
				# adduser  $user
		- 授予 sudo
		
				# export EDITOR=vim;
				# visudo
				.....add.....
				## Allow root to run any commands anywhere
				root    ALL=(ALL)       ALL
				$user  ALL=(ALL)       ALL		
- 进入用户空间
	- mac 客户端 ssh 设置

			$ sudo ~/.ssh/config
			..... add .....
			# stable-diffusion-webui
			Host sd
			    Hostname $serverip
			    User $user
			    Port 22
- OS 是否支持

	CUDA 开发工具仅在某些特定的 Linux 发行版上受支持
	
	- ubunt

			# uname -m && cat /etc/*release
			x86_64
			DISTRIB_ID=Ubuntu
			DISTRIB_RELEASE=20.04
			DISTRIB_CODENAME=focal
			DISTRIB_DESCRIPTION="Ubuntu 20.04.5 LTS"
			NAME="Ubuntu"
			VERSION="20.04.5 LTS (Focal Fossa)"
			ID=ubuntu
			ID_LIKE=debian
			PRETTY_NAME="Ubuntu 20.04.5 LTS"
			VERSION_ID="20.04"
			HOME_URL="https://www.ubuntu.com/"
			SUPPORT_URL="https://help.ubuntu.com/"
			BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
			PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
			VERSION_CODENAME=focal
			UBUNTU_CODENAME=focal
	- rocky
			
			uname -m && cat /etc/*release
			
			x86_64
			Rocky Linux release 8.6 (Green Obsidian)  // 8.y  y <=6
			NAME="Rocky Linux"
			VERSION="8.6 (Green Obsidian)"
			ID="rocky"
			ID_LIKE="rhel centos fedora"
			VERSION_ID="8.6"
			PLATFORM_ID="platform:el8"
			PRETTY_NAME="Rocky Linux 8.6 (Green Obsidian)"
			....
- gcc 验证

	使用 CUDA Toolkit 进行开发需要 `gcc` 编译器。运行 CUDA 应用程序不需要它。
- 验证内核文件和头

	CUDA 驱动程序要求在安装驱动程序时以及在重建驱动程序时安装内核运行版本的内核头文件和开发包。
	
		# uname -r
		5.4.0-125-generic
			    
## 安装
- 登陆

		$ ssh sd
- 安装依赖
	- ubuntu
		- 安装 wget & git
		
				$ sudo apt install -y wget git
		- 安装 python 3.10
		
				$ sudo apt update && sudo apt upgrade -y
				$ sudo apt install software-properties-common -y
				$ sudo add-apt-repository ppa:deadsnakes/ppa
				$ sudo apt install python3.10 -y
				$ sudo apt-get -y install python3.10-venv
				$ cd /usr/bin/
				$ sudo rm -Rf python3
				$ sudo ln -s python3.10 python3
				$ python3 -V
	- rocky
		- 安装 wget & git
			
				$ sudo su - 
				# dnf install -y wget git
		- 安装 python 3.10
	
			因为无法默认安装 python 3.10 所以需要手动编译
			
				# sudo dnf install -y curl gcc openssl-devel bzip2-devel libffi-devel zlib-devel wget make
				# wget https://www.python.org/ftp/python/3.10.8/Python-3.10.8.tar.xz
				# tar -xf Python-3.10.8.tar.xz && cd cd Python-3.10.8
				# ./configure --enable-optimizations
				# make -j 2
				# make altinstall
				# python3.10 -V
					Python 3.10.8
				# python3 -V
					Python 3.6.8
				# cd /usr/bin
				# link -s /usr/local/bin/python3.10 python3
				# python3 -V
					Python 3.10.8
		- 安装编译工具
	
				# sudo dnf install -y  elfutils-libelf-devel
		- 升级 gcc
	
				# dnf -y install gcc-toolset-9-gcc gcc-toolset-9-gcc-c++ 
				# source /opt/rh/gcc-toolset-9/enable
		- 查看 gcc 版本
	
				# gcc --version		
	- 检查 GPU 驱动安装
		- ubunt 检查
				
				$ sudo lshw -C display
		- rocky 检查硬件是否支持 cuda
		
				# lspci | grep -i vga
			
					00:02.0 VGA compatible controller: Cirrus Logic GD 5446
					00:07.0 VGA compatible controller: NVIDIA Corporation TU104GL [Tesla T4] (rev a1)
				# lspci | grep -i nvidia
			
					00:07.0 VGA compatible controller: NVIDIA Corporation TU104GL [Tesla T4] (rev a1)	
	- 安装 GPU 驱动
	- 检查 gpu 驱动 	
		- 7 检查

				# nvidia-smi	
		- 驱动查询
			
				# whereis nvidia
				# modinfo nvidia
			- ubuntu

					filename:       /lib/modules/5.4.0-125-generic/kernel/drivers/video/nvidia.ko
					firmware:       nvidia/470.82.01/gsp.bin
					alias:          char-major-195-*
					version:        470.82.01
					supported:      external
					license:        NVIDIA
					srcversion:     8774ECCCFBD53AA8A183150
					alias:          pci:v000010DEd*sv*sd*bc03sc02i00*
					alias:          pci:v000010DEd*sv*sd*bc03sc00i00*
					depends:        drm
					retpoline:      Y
					name:           nvidia
					vermagic:       5.4.0-125-generic SMP mod_unload modversions
					parm:           NvSwitchRegDwords:NvSwitch regkey (charp)
					parm:           NvSwitchBlacklist:NvSwitchBlacklist=uuid[,uuid...] (charp)
					parm:           NVreg_ResmanDebugLevel:int
					parm:           NVreg_RmLogonRC:int
					parm:           NVreg_ModifyDeviceFiles:int
					parm:           NVreg_DeviceFileUID:int
					parm:           NVreg_DeviceFileGID:int
					parm:           NVreg_DeviceFileMode:int
					parm:           NVreg_InitializeSystemMemoryAllocations:int
					parm:           NVreg_UsePageAttributeTable:int
					parm:           NVreg_RegisterForACPIEvents:int
					parm:           NVreg_EnablePCIeGen3:int
					parm:           NVreg_EnableMSI:int
					parm:           NVreg_TCEBypassMode:int
					parm:           NVreg_EnableStreamMemOPs:int
					parm:           NVreg_RestrictProfilingToAdminUsers:int
					parm:           NVreg_PreserveVideoMemoryAllocations:int
					parm:           NVreg_EnableS0ixPowerManagement:int
					parm:           NVreg_S0ixPowerManagementVideoMemoryThreshold:int
					parm:           NVreg_DynamicPowerManagement:int
					parm:           NVreg_DynamicPowerManagementVideoMemoryThreshold:int
					parm:           NVreg_EnableGpuFirmware:int
					parm:           NVreg_EnableUserNUMAManagement:int
					parm:           NVreg_MemoryPoolSize:int
					parm:           NVreg_KMallocHeapMaxSize:int
					parm:           NVreg_VMallocHeapMaxSize:int
					parm:           NVreg_IgnoreMMIOCheck:int
					parm:           NVreg_NvLinkDisable:int
					parm:           NVreg_EnablePCIERelaxedOrderingMode:int
					parm:           NVreg_RegisterPCIDriver:int
					parm:           NVreg_RegistryDwords:charp
					parm:           NVreg_RegistryDwordsPerDevice:charp
					parm:           NVreg_RmMsg:charp
					parm:           NVreg_GpuBlacklist:charp
					parm:           NVreg_TemporaryFilePath:charp
					parm:           NVreg_ExcludedGpus:charp
					parm:           rm_firmware_active:charp
			- rocky 
					
					filename:       /lib/modules/4.18.0-372.26.1.el8_6.x86_64/extra/drivers/video/nvidia/nvidia.ko
					firmware:       nvidia/520.61.05/gsp.bin
					alias:          char-major-195-*
					version:        520.61.05
					supported:      external
					license:        NVIDIA
					rhelversion:    8.6
					srcversion:     5CF7EBF2C168DB3C24CBCC6
					alias:          pci:v000010DEd*sv*sd*bc06sc80i00*
					alias:          pci:v000010DEd*sv*sd*bc03sc02i00*
					alias:          pci:v000010DEd*sv*sd*bc03sc00i00*
					depends:        drm
					name:           nvidia199.232.69.194 github.global-ssl.fastly.net
140.82.114.4 github.com

					vermagic:       4.18.0-372.26.1.el8_6.x86_64 SMP mod_unload modversions
					sig_id:         PKCS#7
					signer:         NVIDIA			
- 安装  sd
	- 开始安装
	
			$ bash <(wget -qO- https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh)
	- 本地 clone 后

			$ bash stable-diffusion-webui/master/webui.sh
- 运行 sd

		$ bash stable-diffusion-webui/webui.sh
- 不更新启动
		
		$ cd stable-diffusion-webui
		$ cp webui.sh ubuntu-webui.sh
		$ vi ubuntu-webui.sh
		
		.....删除....
		# Check prerequisites
		< for preq in "${GIT}" "${python_cmd}"
		< do
		<     if ! hash "${preq}" &>/dev/null
		<     then
		<         printf "\n%s\n" "${delimiter}"
		<         printf "\e[1m\e[31mERROR: %s is not installed, aborting...\e[0m" "${preq}"
		<         printf "\n%s\n" "${delimiter}"
		<         exit 1
		<     fi
		< done
		<
		< if ! "${python_cmd}" -c "import venv" &>/dev/null
		< then
		<     printf "\n%s\n" "${delimiter}"
		<     printf "\e[1m\e[31mERROR: python3-venv is not installed, aborting...\e[0m"
		<     printf "\n%s\n" "${delimiter}"
		<     exit 1
		< fi
		<
		< printf "\n%s\n" "${delimiter}"
		< printf "Clone or update stable-diffusion-webui"
		< printf "\n%s\n" "${delimiter}"
		< cd "${install_dir}"/ || { printf "\e[1m\e[31mERROR: Can't cd to %s/, aborting...\e[0m" "${install_dir}"; exit 1; }
		< if [[ -d "${clone_dir}" ]]
		< then
		<     cd "${clone_dir}"/ || { printf "\e[1m\e[31mERROR: Can't cd to %s/%s/, aborting...\e[0m" "${install_dir}" "${clone_dir}"; exit 1; }
		<     "${GIT}" pull
		< else
		<     "${GIT}" clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git "${clone_dir}"
		<     cd "${clone_dir}"/ || { printf "\e[1m\e[31mERROR: Can't cd to %s/%s/, aborting...\e[0m" "${install_dir}" "${clone_dir}"; exit 1; }
		< fi
			
- 守护进程
		$ vi stable-diffusion-webui.sh

		.......
		#! /bin/sh

		### BEGIN INIT INFO
		# Provides:          $user
		# Short-NAMEription: the stable-diffusion webui service
		# Description:       Manage stable-diffusion webui service Lifecycle
		### END INIT INFO
		
		NAME=stable-diffusion-webui
		PIDFILE=/tmp/$NAME.pid
		DAEMON_OPTS=ubuntu-webui.sh
		UPDATE_DAEMON_OPTS=webui.sh
		DEPLOY_DIR=/home/pangzheng/stable-diffusion-webui
		
		
		PID=`ps axuf | grep launch.py | grep -v grep  | awk '{print $2}'`
		
		echo "-----------PID: $PID----------"
		
		start_method(){
		        echo -n "Starting $NAME"
		
		        if [ $PID ];then
		                echo "$NAME is running......"
		                exit 1
		        fi
		
		        cd $DEPLOY_DIR && nohup /usr/bin/bash  ${DAEMON_OPTS} &
		
		        echo " $NAME start done"
		
		        cd $DEPLOY_DIR && tail -f nohup.out
		}
		
		update_method(){
        echo -n "Starting $NAME"

        if [ $PID ];then
                echo "$NAME is running......"
                exit 1
        fi

        cd $DEPLOY_DIR && nohup /usr/bin/bash ${UPDATE_DAEMON_OPTS} &

        echo " $NAME start done"

        cd $DEPLOY_DIR && tail -f nohup.out
		}
		
		stop_method(){
		        echo -n "Stopping $NAME"
		
		        if [ ! $PID ];then
		                echo "$NAME is not running..... "
		                exit 1
		        fi
		
		        kill "$PID"
		
		        sleep 1
		
		        echo " $NAME stop is done"
		}
		
		status_method(){
		
		        if [ $PID ];then
		                echo "$NAME is running"
		        else
		                echo "$NAME is not running"
		        fi
		
		}
		
		logs_method(){
		
		        cd $DEPLOY_DIR  && tail -f nohup.out
		}
		
		case "$1" in
		  status)
		        status_method
		        ;;
		  start)
		        start_method
		        ;;
		  update)
        		  update_method
        		  ;;
		  stop)
		        stop_method
		        ;;
		  restart)
		        stop_method
		        start_method
		        ;;
		 logs)
		        logs_method
		        ;;
		  *)
		        echo "Usage: $0 {status|start|update|stop|restart}" >&2
		        exit 1
		        ;;
		esac
		
		exit 0

- 参数优化
	- 开启 xformers 和 deepdanbooru
		- 安装 Xformers

			Xformers 库是加速图像生成的一种可选方式。除了一种特定的配置外，没有适用于 Windows 的二进制文件，但您可以自己构建它。
			
			来自匿名用户的指南，尽管我认为它适用于在 Linux 上构建：
			
			关于如何构建 XFORMERS 的指南还包括如何让自己摆脱 voldy 新提交的 sm86 限制
				
				sudo apt-get install python3-dev
				cd stable-diffusion-webui
				source ./venv/bin/activate
				cd repositories
				git clone https://github.com/facebookresearch/xformers.git
				cd xformers
				git submodule update --init --recursive
				pip install -r requirements.txt
				pip install -e .

			报错

		- `error: command '/usr/bin/x86_64-linux-gnu-gcc' failed with exit code 1`
			- 解决
		
					sudo apt-get install python3-dev
		- 测试
			- 未打开 Xformers

					Launching launch.py...
					################################################################
					Python 3.10.8 (main, Oct 12 2022, 19:14:26) [GCC 9.4.0]
					Commit hash: 17a2076f72562b428052ee3fc8c43d19c03ecd1e
					Installing requirements for Web UI
					Launching Web UI with arguments:
					LatentDiffusion: Running in eps-prediction mode
					DiffusionWrapper has 859.52 M params.
					making attention of type 'vanilla' with 512 in_channels
					Working with z of shape (1, 4, 32, 32) = 4096 dimensions.
					making attention of type 'vanilla' with 512 in_channels
					Loading weights [925997e9] from /home/pangzheng/stable-diffusion-webui/models/Stable-diffusion/animefull-final-pruned-4.2G.ckpt
					Loading VAE weights from: /home/pangzheng/stable-diffusion-webui/models/Stable-diffusion/animefull-final-pruned-4.2G.vae.pt
					Applying cross attention optimization (Doggettx).
					Model loaded.
					Loaded a total of 0 textual inversion embeddings.
					Embeddings:
					Running on local URL:  http://127.0.0.1:7860
					
					To create a public link, set `share=True` in `launch()`.
					100%|██████████| 20/20 [00:04<00:00,  4.21it/s]
					Total progress: 100%|██████████| 20/20 [00:04<00:00,  4.18it/s]
			- 打开 Xformers
		
					Launching launch.py...
					################################################################
					Python 3.10.8 (main, Oct 12 2022, 19:14:26) [GCC 9.4.0]
					Commit hash: 17a2076f72562b428052ee3fc8c43d19c03ecd1e
					Installing requirements for Web UI
					Launching Web UI with arguments: --force-enable-xformers  <----这里
					LatentDiffusion: Running in eps-prediction mode
					DiffusionWrapper has 859.52 M params.
					making attention of type 'vanilla' with 512 in_channels
					Working with z of shape (1, 4, 32, 32) = 4096 dimensions.
					making attention of type 'vanilla' with 512 in_channels
					Loading weights [925997e9] from /home/pangzheng/stable-diffusion-webui/models/Stable-diffusion/animefull-final-pruned-4.2G.ckpt
					Loading VAE weights from: /home/pangzheng/stable-diffusion-webui/models/Stable-diffusion/animefull-final-pruned-4.2G.vae.pt
					Applying xformers cross attention optimization. <----这里
					Model loaded.
					Loaded a total of 0 textual inversion embeddings.
					Embeddings:
					Running on local URL:  http://127.0.0.1:7860
					
					To create a public link, set `share=True` in `launch()`.
					
					100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 20/20 [00:03<00:00,  5.33it/s] <----这里
					Total progress: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 20/20 [00:03<00:00,  5.24it/s]
			- 提速证明

				实际证明打开 Xformers 可以提速 1/4 效率
			- 参考
				- [Xformers](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Xformers)
		- deepdanbooru				
			- 设置 	
				
					vi  ~/stable-diffusion-webui/webui-user.sh
					.....
					export COMMANDLINE_ARGS="--xformers --deepdanbooru"
			- 重启启动		
					
## ubunt 安装 Nginx 登陆认证反向代理
- 安装 nginx

		sudo apt install nginx -y
- 设置密码
	- 安装工具
		
			sudo apt-get install apache2-utils -y
	- 设置登陆密码（修改密码记住重启才能生效）	
		
			$ sudo htpasswd -c /etc/nginx/.htpasswd $user
			
			New password:
			Re-type new password:
			Adding password for user $user
	- 设置代理配置
		 		
			$ sudo vi /etc/nginx/sites-available/default
			
			upstream SD_Node {
			    # Nodejs app upstream
			    server 127.0.0.1:7860;    # 只需要修改这个你要监听的端口号
			    keepalive 64;
			}
			server {
			        listen 80 default_server;
			        listen [::]:80 default_server;
			        root /var/www/html;
			
			        # Add index.php to the list if you are using PHP
			        index index.html index.htm index.nginx-debian.html;
			
			        server_name _;
			
			        location / {
			                # First attempt to serve request as file, then
			                # as directory, then fall back to displaying a 404.
			                # try_files $uri $uri/ =404;
			                auth_basic           "Please input password";
			                auth_basic_user_file /etc/nginx/.htpasswd;
			                proxy_pass http://SD_Node/;
			        }
			        
			        location /outputs {
			                auth_basic           "Please input password";
			                auth_basic_user_file /etc/nginx/.htpasswd;
			                root /home/pangzheng/stable-diffusion-webui/; #指定目录所在路径
			                autoindex on; #开启目录浏览
			                #autoindex_format html; #以html风格将目录展示在浏览器中
			                autoindex_exact_size off; #切换为 off 后，以可读的方式显示文件大小，单位为 KB、MB 或者 GB
			                autoindex_localtime on; #以服务器的文件时间作为显示的时间
			                charset utf-8,gbk; #展示中文文件名
			       }
			}
	- 设置上传大小
		- PNG Info 大画 会被 nginx 阻拦

				sudo vi /etc/nginx/nginx.conf
				
				....................
				http {
				.......add......
				client_max_body_size 50m;
				....................
				}
						
	- 重启
			
			$ sudo systemctl stop nginx
			$ sudo systemctl start nginx
			$ sudo systemctl status nginx

## 启动脚本

## 游览器使用
打开游览器，访问服务 http://serverip				

 注意使用中会有各种算法再下载
		  
## 报错处理
### Ubuntu
- venv
	- 错误
	
			ERROR: Cannot activate python venv, aborting...
	- 解决

			$ rm -Rf /home/$user/stable-diffusion-webui/venv/- venv pip
	- 错误
		
			note: This error originates from a subprocess, and is likely not a problem with pip.
	
			[notice] A new release of pip available: 22.2.2 -> 22.3
			[notice] To update, run: pip install --upgrade pip 
	- 解决		
		
			$ source ~/stable-diffusion-webui/venv/bin/activate
			$ pip install --upgrade pip
			$ deactivate
- GFPGAN
	- 错误

			RuntimeError: Couldn't install gfpgan.
			Command: "/home/$user/stable-diffusion-webui/venv/bin/python3" -m pip install git+https://github.com/TencentARC/GFPGAN.git@8d2447a2d918f8eba5a4a01463fd48e45126a379 --prefer-binary
			Error code: 1
	- 解决 
		- 解决git网络
			
				$ bash <(wget -qO- https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh)
		- or 手动 cloen 后重试
			
				$ git clone --filter=blob:none --quiet https://github.com/TencentARC/GFPGAN.git /tmp/pip-req-build-b_cnz264 		 		$ bash <(wget -qO- https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh) 
- BLIP
	- 错误

			RuntimeError: Couldn't clone BLIP.
			Command: "git" clone "https://github.com/salesforce/BLIP.git" "repositories/BLIP"  
	- 解决

			$ pwd 
			/home/$user
			$ git clone https://github.com/salesforce/BLIP.git stable-diffusion-webui/repositories/BLIP
- 启动报错
	- 错误

			.....
			Error verifying pickled file from /home/$user/.cache/huggingface/transformers/c506559a5367a918bab46c39c79af91ab88846b49c8abd9d09e699ae067505c6.6365d436cc844f2f2b4885629b559d8ff0938ac484c01a6796538b2665de96c7:
			......
	- 解决

			rm -Rf /home/$user/.cache/huggingface/transformers/c506559a5367a918bab46c39c79af91ab88846b49c8abd9d09e699ae067505c6.6365d436cc844f2f2b4885629b559d8ff0938ac484c01a6796538b2665de96c7
						
									
## 模型下载
- SD 标准模型
	- [1.4](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original)
	
		当前标准模型是 1.4 版本，最新开源模型需要去[抱脸网](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original)站下载，下载完毕后 cp 到 
	
			cp sd-v1-4.ckpt	/home/$user/stable-diffusion-webui/models/Stable-diffusion/
	- [1.5](https://huggingface.co/runwayml/stable-diffusion-v1-5) 
		- v1-5-pruned-emaonly.ckpt - 4.27GB, ema-only重量。使用较少的VRAM -适合推理
		- v1-5-pruned.ckpt - 7.7GB, ema + non-ema 权重。使用更多的VRAM -适合微调

				cp sd-v1-*  /home/$user/stable-diffusion-webui/models/Stable-diffusion/
- 二次元模型

	你懂的		

## 更新版本
## 报错处理
### Ubuntu
- pip 加速
	- 错误

		pip 慢
	- 解决方案

		将 pip 替换成阿里源
		
		- 使用用户权限进入用户目录

				cd /home/pangzheng
		- 创建 pip 配置目录
			
				mkdir .pip
		- 创建配置文件

				vi pip.conf

				## Note, this file is written by cloud-init on first boot of an instance
				## modifications made here will not survive a re-bundle.
				###
				[global]
				index-url=http://mirrors.cloud.aliyuncs.com/pypi/simple/
				
				[install]
				trusted-host=mirrors.cloud.aliyuncs.com
		- 保存退出		
- torch 
	- 错误
		
			...
			RuntimeError: Couldn't install torch.
			Command: "/usr/bin/python3" -m pip install torch==2.0.0 torchvision==0.15.1 --extra-index-url https://download.pytorch.org/whl/cu118
			Error code: 1
	- [解决方案](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/6592)

		安装正确版本的 python 3.10.x 后，我的解决方案是从 stable-diffusion-webui-master 目录中删除“venv”文件夹。。		
- TCMalloc
	- 错误

			Cannot locate TCMalloc (improves CPU memory usage)
	- [解决方案](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/10117)

			sudo apt-get install libgoogle-perftools4 libtcmalloc-minimal4 -y
- pip
	- 错误
		
			[notice] A new release of pip is available: 23.0.1 -> 23.1.2
			[notice] To update, run: pip install --upgrade pip
	- 解决
		- 进入 `stable-diffusion-webui` 目录
		- 进入虚拟机
				
				source venv/bin/activate
		- 升级 pip

				pip install --upgrade pip
- Xformers
	- 错误

		启动报错
			
			No module 'xformers'. Proceeding without it. 
	- [解决方案](https://blog.csdn.net/weixin_42045719/article/details/130071986)
		- 进入 `stable-diffusion-webui` 目录
		- 编辑	
				
				vi webui-user.sh			
				
				# 添加
				export COMMANDLINE_ARGS="--xformers" 
		- 重启安装解决
- 任务管理器错误
	- 错误

		刷新页面打开后，点击任务无法进行，且无法切换模型，打开控制台报错

			WebSocket connection to 'ws://123.56.40.96/queue/join' failed: 
	- [解决方案](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/9074)
		- 进入 `stable-diffusion-webui` 目录
		- 编辑	
				
				vi webui-user.sh			
				
				# 添加
				export COMMANDLINE_ARGS="--xformers --no-gradio-queue"
- deepdanbooru 
	- 错误
	
		点击 deepdanbooru 取图获取魔法词，安装报错	
	- 解决方案
		
		找台可以网路加速的机器，下载，复制到对应目录，重启服务
		
		- 模型地址

				https://github.com/AUTOMATIC1111/TorchDeepDanbooru/releases/download/v1/model-resnet_custom_v3.pt 
		- 下载地方

				/data/home/pangzheng/stable-diffusion-webui/models/torch_deepdanbooru/model-resnet_custom_v3.pt
- A tensor with all NaNs was produced in VAE
	- 错误

			modules.devices.NansException: A tensor with all NaNs was produced in VAE. This could be because there's not enough precision to represent the picture. Try adding --no-half-vae commandline argument to fix this. Use --disable-nan-check commandline argument to disable this check.
	- [解决方案](https://www.scratchina.com/html/aigcjishu/357.html)
		- 进入 `stable-diffusion-webui` 目录
		- 编辑	
				
				vi webui-user.sh			
				
				# 添加
				export COMMANDLINE_ARGS="--xformers --no-half-vae --no-gradio-queue" 
		- 重启安装解决
- No such file or directory: 'xdg-open'
	- 错误

		点击 ui 的目录报错
			
				FileNotFoundError: [Errno 2] No such file or directory: 'xdg-open' 							
## 参考
- [NVIDIA Drivers on Rocky Linux](https://darryldias.me/2021/nvidia-drivers-on-rocky-linux/)
- [Install NVIDIA Drivers [515.76 / 510.85.02 / 470.141.03 / 390.154 / 340.108] on CentOS Stream 9/8, RHEL 9/8, Rocky Linux 8.5](https://www.if-not-true-then-false.com/2021/install-nvidia-drivers-on-centos-rhel-rocky-linux/)