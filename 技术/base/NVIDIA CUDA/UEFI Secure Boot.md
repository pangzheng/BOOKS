# UEFI Secure Boot
- 错误

		nvidia: module verification failed: signature and/or required key missing - tainting kernel
	
	- UEFI Secure Boot
	
		用于 RHEL7 和 SLES12 的 MLNX_OFED 中包含的所有内核模块都使用 x.509 密钥进行签名，以支持在启用安全引导时加载模块。

		- 在您的系统上注册 Mellanox 的 x.509 公钥
	
			为了在支持安全启动的操作系统在启用了安全启动的基于 UEFI 的系统上启动时支持加载 MLNX_OFED 驱动程序，应将 Mellanox x.509 公钥添加到 UEFI 安全启动密钥数据库并通过以下方式加载到系统密钥环中内核。

			按照以下步骤将 Mellanox 的 x.509 公钥添加到您的系统：

				在将 Mellanox 的 x.509 公钥添加到您的系统之前，请确保：
					- 'mokutil' 软件包已安装在您的系统上
					- 系统以 UEFI 模式启动
			- 下载 x.509 公钥。
	
					# wget http://www.mellanox.com/downloads/ofed/mlnx_signing_key_pub.der
			- 使用 mokutil 实用程序将公钥添加到 MOK 列表。
	
				您将被要求输入并确认此 MOK 注册请求的密码。
	
					# mokutil --import mlnx_signing_key_pub.der
			- 重新启动系统。
	
			shim.efi 将注意到挂起的 MOK 密钥注册请求，它将启动 MokManager.efi 以允许您从 UEFI 控制台完成注册。 您需要输入之前与此请求关联的密码并确认注册。 完成后，公钥将添加到 MOK 列表中，该列表是持久的。 一旦密钥在 MOK 列表中，它将自动传播到系统密钥环，并在启用 UEFI 安全启动时启动后续。
	
			要查看当前启动时已将哪些密钥添加到系统密钥环，请安装“keyutils”包并运行：#keyctl list %:.system_keyring
		- 从内核模块中删除签名

			可以使用“binutils”包提供的“strip”实用程序从已签名的内核模块中删除签名。

				# strip -g my_module.ko
			strip 实用程序将更改给定文件而不保存备份。该操作只能通过退出内核模块来撤消。因此，我们建议在删除签名之前备份一份副本。

			要从 MLNX_OFED 内核模块中删除签名：

			- 删除签名。

					# rpm -qa | grep -E "kernel-ib|mlnx-ofa_kernel|iser|srp|knem|mlnx-rds|mlnx-nfsrdma|mlnx-nvme|mlnx-rdma-rxe" | xargs rpm -ql | grep "\.ko$" | xargs strip -g
			- 删除签名后，模块加载时将不再显示如下消息：
				
					"Request for unknown module key 'Mellanox Technologies signing key: 61feb074fc7292f958419386ffdd9d5ca999e403' err -11"
			- 但是，请注意，将显示类似于以下内容的消息：

					"my_module: module verification failed: signature and/or required key missing - tainting kernel"
			此消息仅在没有签名或密钥不在内核密钥环中的第一次模块启动时出现。因此，此消息可能会被忽视。卸载并重新加载内核模块后重新启动系统后，将出现该消息（此消息无法消除）。

			- 使用剥离的模块更新 RHEL 系统上的 initramfs。

					mkinitrd /boot/initramfs-$(uname -r).img $(uname -r) --force