# Stable Diffusion 训练相关
## 模型相
### 如何下载
快速 Torrent 下载

- 命令行(注意这个是ubuntu 体系)

		apt update
		apt install -y aria2
		aria2c "<magnet URL here>"

### 原生模型
- 原生 1.5 [81761151] [a9263745]

	 主要模型是 `v1-5-pruned-emaonly.ckpt`，这意味着它只有 EMA 权重。
	
	- v1-5-pruned-emaonly.ckpt [81761151] | SHA256 cc6cb27103417325ff94f52b7a5d2dde45a7515b25c255d8e396c90014281516

		Stable-Diffusion-v1-5 检查点使用 Stable-Diffusion-v1-2 检查点的权重进行初始化，随后在“laion-aesthetics v2 5+”上以 512x512 分辨率在 595k 步上进行微调，并降低 10% 文本调节以改进无分类器的指导抽样。

		- Torrent
		
				magnet:?xt=urn:btih:2daef5b5f63a16a9af9169a529b1a773fc452637&dn=v1-5-pruned-emaonly.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
		- web 下载
	
		从 HuggingFace 下载。CDN 加速，但需要登录并与存储库共享您的联系信息。模型信息 [这里](https://huggingface.co/runwayml/stable-diffusion-v1-5#original-github-repository)
		
			https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.ckpt
	- v1-5-pruned.ckpt [a9263745] | SHA256 e1441589a6f3c5a53f5f54d0975a18a7feb7cdf0b0dee276dfc3331ae376a053 
	
		这个版本的模型只对原生训练有用。 如果你不知道这是什么，你就不需要这个。
		
		如果您只想生成图像，或训练 `dreambooth/textual inversion/hypernetworks`，请获取 v1-5-pruned-emaonly.ckpt。
		
		- Torrent
		
				magnet:?xt=urn:btih:38c2b7a8abf55604ae8a5f060ffda2e028075af3&dn=v1-5-pruned.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
		- web 下载

				https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned.ckpt
- VAE

	Stability AI 发布了两个用于稳定扩散的自动编码器。[github 地址](https://huggingface.co/stabilityai/sd-vae-ft-mse-original#improved-autoencoders)
	
	- Stable Diffusion VAE ft-mse-original

		从 ft-EMA 恢复并使用 EMA 权重，并使用重新加权损失训练了另外 280k 步，更强调 MSE 重建（产生一些“更平滑”的输出）。 为了保持与现有模型的兼容性，仅对解码器部分进行了微调； 检查点可以用作现有自动编码器的替代品。

		Repo 和信息（有比较）[https://huggingface.co/stabilityai/sd-vae-ft-mse-original](https://huggingface.co/stabilityai/sd-vae-ft-mse-original)

		VAE 与此列表中的其他型号不同。 它不是一个可以单独用于生成图像的完整模型，而是可以与其他模型一起使用的组件。如果您不知道这是什么或为什么想要它，请不要理会它。
		
		- web 下载

				https://huggingface.co/stabilityai/sd-vae-ft-mse-original/resolve/main/vae-ft-mse-840000-ema-pruned.ckpt
	- Stable Diffusion VAE ft-ema-original

		从原始检查点恢复，经过 313198 步训练并使用 EMA 权重。 为了保持与现有模型的兼容性，仅对解码器部分进行了微调； 检查点可以用作现有自动编码器的替代品。
回购和信息（有比较）[https://huggingface.co/stabilityai/sd-vae-ft-ema-original](https://huggingface.co/stabilityai/sd-vae-ft-ema-original)

		从原始 kl-f8 VAE 检查点恢复，经过 313198 步训练并使用 EMA 权重。
		
		- web 下载

				https://huggingface.co/stabilityai/sd-vae-ft-ema-original/resolve/main/vae-ft-ema-560000-ema-pruned.ckpt
- Stable Diffusion (1.5 )修复

	Repo [https://huggingface.co/runwayml/stable-diffusion-inpainting](https://huggingface.co/runwayml/stable-diffusion-inpainting)
	旨在与 [https://github.com/runwayml/stable-diffusion](https://github.com/runwayml/stable-diffusion) 一起使用

	使用 Stable-Diffusion-v-1-2 的权重初始化 Stable-Diffusion-Inpainting。 首先是 595k 步常规训练，然后是“laion-aesthetics v2 5+”分辨率为 512x512 的 440k 步修复训练和 10% 的文本调节下降，以改进无分类器无分类器引导采样。 对于修复，UNet 有 5 个额外的输入通道（4 个用于编码的蒙版图像，1 个用于蒙版本身），其权重在恢复非修复检查点后初始化为零。 在训练期间，我们生成合成蒙版，并以 25% 的比例对所有内容进行蒙版。
	
	[github 地址](https://github.com/runwayml/stable-diffusion#stable-diffusion)
	
		SHA256 c6bbc15e3224e6973459ba78de4998b80b50112b0ae5b5c67113d56b4e366b19
	- Torrent
	
			magnet:?xt=urn:btih:b523a9e71ae02e27b28007eca190f41999c2add1&dn=sd-v1-5-inpainting.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce
	- web 下载

			https://huggingface.co/runwayml/stable-diffusion-inpainting/resolve/main/sd-v1-5-inpainting.ckpt
- 原生 1.4

		sd-v1-4-pruned.ckpt [4af45990]
		sd-v1-4.ckpt [7460a6fa] | SHA256 fe4efff1e174c627256e44ec2991ba279b3816e364b49f9be2abc0b3ff3f8556
		sd-v1-4-full-ema.ckpt [06c50424] | SHA256 14749efc0ae8ef0329391ad4436feb781b402f4fece4883c7ad8d10556d8a36a
	- Torrent

			magnet:?xt=urn:btih:3A4A612D75ED088EA542ACAC52F9F45987488D1C&tr=udp://tracker.opentrackr.org:1337
	- web 下载

			https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/resolve/main/sd-v1-4.ckpt
	- google 网盘

			https://drive.google.com/file/d/1wHFgl0ivCmIZv88hVZXkb8oy9qCuaBGA/view
			
### Waifu Diffusion
在 Danbooru 上受过训练，略带 NSFW。

- Waifu Diffusion VAE kl-f8-anime2

	SD 1.4 VAE 在 250k 动漫风格图像上的微调。 据说可以改善眼睛和手指的重建。

	- 信息 https://twitter.com/haruu1367/status/1579286947519864833
	- repo https://huggingface.co/hakurei/waifu-diffusion-v1-4

	VAE 与此列表中的其他型号不同。 它不是一个可以单独用于生成图像的完整模型，而是可以与其他模型一起使用的组件。
如果您不知道这是什么或为什么想要它，请不要理会它。

	- web 下载

			https://huggingface.co/hakurei/waifu-diffusion-v1-4/resolve/main/vae/kl-f8-anime2.ckpt
- Waifu Diffusion VAE kl-f8-anime

	SD 1.4 VAE 在 250k 动漫风格图像上的微调。 据说可以改善眼睛和手指的重建。
	
	- 信息 https://twitter.com/haruu1367/status/1579286947519864833
	-  repo  https://huggingface.co/hakurei/waifu-diffusion-v1-4

	VAE 与此列表中的其他型号不同。 它不是一个可以单独用于生成图像的完整模型，而是可以与其他模型一起使用的组件。
如果您不知道这是什么或为什么想要它，请不要理会它。

	- web 下载

			https://huggingface.co/hakurei/waifu-diffusion-v1-4/resolve/main/vae/kl-f8-anime.ckpt
- Waifu Diffusion v1.3 Full [84692140] [4470c325] [e1de58a9] [3e1a125f]

	在 600,000 张高分辨率 Danbooru 图像上训练 10 个 Epoch。

	float16 版本小于 float32（2GB 与 4GB）。 始终使用 float16（除非您的 GPU 不支持它），因为它使用较少的磁盘空间和 RAM。[Release notes](https://gist.github.com/harubaru/f727cedacae336d1f7877c4bbe2196e1)
	
		wd-v1-3-float16.ckpt [84692140] | SHA256 4afab9126057859b34d13d6207d90221d0b017b7580469ea70cee37757a29edd
		wd-v1-3-float32.ckpt [4470c325] | SHA256 9dade826203c7ee369881b5dc20d34298fa644c1f137568e09fbc4b9a0d3e817
		wd-v1-3-full.ckpt [e1de58a9] | SHA256 23ba8d0411c211d3d14903d46613bc088924e1453ed1c6428ce86bde54a37d27
		wd-v1-3-full-opt.ckpt [3e1a125f] | SHA256 10912b9a6d773ea7c299c0563d10538ada04ade81837362b6c0c67be4df937c1
	
	- wd-v1-3-float16.ckpt [84692140] Web
		-  Torrent

				magnet:?xt=urn:btih:AWJJJZNFOOK7R2XXXGZ4GFNKUEU2TSFP&dn=wd-v1-3-float16.ckpt&xl=2132889245&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce
		- web 下载

				https://huggingface.co/hakurei/waifu-diffusion-v1-3/resolve/main/wd-v1-3-float16.ckpt
	- wd-v1-3-float32.ckpt [4470c325] Web
		-  Torrent

				magnet:?xt=urn:btih:PK4UPXMW5TC7O2MKF7YJMIDX4NRBYO5R&dn=wd-v1-3-float32.ckpt&xl=4265384157&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce
		- web 下载

				https://huggingface.co/hakurei/waifu-diffusion-v1-3/resolve/main/wd-v1-3-float32.ckpt
	- wd-v1-3-full.ckpt [e1de58a9] 

		完整的 EMA 文件大小较大，因为它包含仅用于训练的附加数据。

		如果您不知道这是什么，请使用 float16 或 float32 版本。

		- web 下载

				https://huggingface.co/hakurei/waifu-diffusion-v1-3/resolve/main/wd-v1-3-full.ckpt
	- wd-v1-3-full-opt.ckpt [3e1a125f] 

		完整选项的文件大小更大，因为它包含仅用于训练的附加数据。

		如果您不知道这是什么，请使用 float16 或 float32 版本。
		
		- web 下载
		
				https://huggingface.co/hakurei/waifu-diffusion-v1-3/resolve/main/wd-v1-3-full-opt.ckpt
	
	EMA 权重介绍在 [这里](https://huggingface.co/hakurei/waifu-diffusion-v1-3#waifu-diffusion-v13)
- Waifu Diffusion v1.3 beta	
	- Waifu Diffusion v1.3 beta epoch09 [a0d90b41] [134eaffb] [e90a9d02]
	- Waifu Diffusion v1.3 beta epoch08 [55537280] [7dd9e8e1] [d12b4159]
	- Waifu Diffusion v1.3 beta epoch07 [3ad68e9a] [e4150f29] [39312118]
	- Waifu Diffusion v1.3 beta epoch06 [281c28bb] [3a672a8a] [d970eaf1]
	- Waifu Diffusion v1.3 beta epoch05 [25f7a927] [3563d59f] [9453d3e5]
	- Waifu Diffusion v1.3 beta epoch04 [958859c3] [84d80299] [994d2e0f]
	- Waifu Diffusion v1.3 beta epoch03 [63e9eb31] [22ca2722] [d8b28983]
- Waifu Diffusion v1.2 [0b8c694b] [45dee52b]

	使用 56k Danbooru 图像在 Stable Diffusion v1.4 上训练 5 个 epoch。训练了 56,000 张 Danbooru 图像 5 个 epoch。

		wd-v1-2-full-ema-pruned.ckpt [0b8c694b] | SHA256 5512bc271c31fd9433494f3fced009d3b27515cca5ef1a1ed13bce136f8e8cad
		wd-v1-2-full-ema.ckpt [45dee52b] | SHA256 89138c3a7783a2b17730090c06e2e1950fb421068cc5f4a5d6c4ba1040a7e1bb

	- wd-v1-2-full-ema-pruned.ckpt [0b8c694b]

		修剪全尺寸模型以生产此模型。 基本上，所有不必要的数据都被删除了。
		
		- Torrent
			
				magnet:?xt=urn:btih:153590fd7e93ee11d8db951451056c362e3a9150&dn=wd-v1-2-full-ema-pruned.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
	- Full EMA Torrent [45dee52b]

		[EMA 和 裁剪模型之间没有区别](https://cyberes.github.io/stable-diffusion-models/#waifu-diffusion-ema-vs-pruned)
		
		-  Torrent
		
				magnet:?xt=urn:btih:f45cecf4e9de86da83a78dd2cccd7f27d5557a52&dn=wd-v1-2-full-ema.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
		- web
		
			创作者提供了网络下载，但速度很慢。
		
				https://thisanimedoesnotexist.ai/downloads/wd-v1-2-full-ema.ckpt

### Trinart
### trinart_stable_diffusion_v2
另一个动漫微调。 旨在将 SD 推向动漫/漫画风格。 似乎比 Waifu Diffusion 更“风格化”和“艺术化”，如果这有任何意义的话。

60,000 步版本是原始版本，115,000 和 95,000 版本是 60,000 加额外训练。 如果样式轻推太多，请使用 60,000 步版本。[请参阅下面的比较](https://cyberes.github.io/stable-diffusion-models/#model-comparison)

- web 下载
	- 60,000 Steps
	
			https://huggingface.co/naclbit/trinart_stable_diffusion_v2/resolve/main/trinart2_step60000.ckpt
	- 95,000 Steps
	
			https://huggingface.co/naclbit/trinart_stable_diffusion_v2/resolve/main/trinart2_step95000.ckpt
	- 115,000 Steps
	
			https://huggingface.co/naclbit/trinart_stable_diffusion_v2/resolve/main/trinart2_step115000.ckpt

### trinart_characters_19-2m_stable_diffusion_v1
一个基于stable diffusion  v1 的模型，由大约 1920 万张动漫/漫画风格的图像（包括预卷增强图像）以及大约 50,000 张图像进行最终微调。 该模型在 SDv1 的给定模型规格中寻求艺术风格多功能性和解剖质量之间的最佳平衡点。

这与从 9 月初到 10 月 14 日在 AI Novelist/TrinArt 服务中发布的版本 1 模型相同。

过滤数据集以排除 NSFW 或不安全的内容。 该模型在艺术风格和解剖质量之间寻求最佳平衡点。

repo [https://huggingface.co/naclbit/trinart_characters_19.2m_stable_diffusion_v1](https://huggingface.co/naclbit/trinart_characters_19.2m_stable_diffusion_v1)

- web 下载
	- 原模型
	
			https://huggingface.co/naclbit/trinart_characters_19.2m_stable_diffusion_v1/resolve/main/trinart_characters_it4_v1.ckpt
	- VAE模型

			https://huggingface.co/naclbit/trinart_characters_19.2m_stable_diffusion_v1/resolve/main/autoencoder_kl-f8-trinart_characters.ckpt

- trinart2_step115000.ckpt [f1c7e952]
- trinart2_step95000.ckpt [cf0bd941]
- trinart2_step60000.ckpt [6ecd8e48]
- trinart_stable_diffusion_epoch3.ckpt [9d7f05fc]

---------？？-----------
### SmirkingFace
SmirkingFace 是一个项目，旨在用高质量的色情内容微调稳定扩散模型，并允许它生成各种各样的色情内容。

如果您不确定需要哪种模型，EB 1.0 EMA 模型在许多情况下往往表现良好。 EBL 1.0 的训练时间最长，但需要代码才能使我们修改后的模型工作（在我们的 github 上发布）。

注意：我们所有的模型都包括由 Stability AI 发布的更新的 MSE VAE。

- Info https://www.patreon.com/posts/released-models-73689798
- Usage instructions with previews https://www.patreon.com/posts/73775030
- Word lists https://www.patreon.com/posts/word-lists-73913464

#### EB 1.0 [7f115f17] [7010a578] [339b9a04]
使用原始稳定扩散架构的模型在各种姿势的各种色情照片上进行训练。

- Info https://www.patreon.com/posts/released-models-73689798
- Usage instructions with previews https://www.patreon.com/posts/73775030

SF_EB_1.0_ema_vae.ckpt [7f115f17]

	SF_EB_1.0_ema_vae.ckpt [7f115f17] SHA256 245cac02fa7243a83ff5f7cf78407a505308e8b6f2dacdff2ad43a1846514f3d
- Torrent

		magnet:?xt=urn:btih:a0bcc397b2c3c101755245fd9b26ae1aa8d5d580&dn=SF_EB_1.0_ema_vae.zip&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://anonfiles.com/32F7ecE0y5/SF_EB_1.0_ema_vae_zip
	or

		https://gofile.io/d/MYOKl8

SF_EB_1.0_noema_vae.ckpt [7010a578]

	SF_EB_1.0_noema_vae.ckpt [7010a578] SHA256 344719ce863db1464ee81f2dc61375c10fcb9fd8b72afd69ad07b091fc834170

- Torrent

		magnet:?xt=urn:btih:3d73f553ce18ec0bae3811edb37a904c5e82c56d&dn=SF_EB_1.0_noema_vae.zip&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://anonfiles.com/t1Iee0E4y0/SF_EB_1.0_noema_vae_zip
	or

		https://gofile.io/d/MYOKl8

SF_EB_1.0_pruned_vae.ckpt [339b9a04]
SF_EB_1.0_pruned_vae.ckpt [339b9a04] SHA256 c50ab94df348e43ad001bd18bcbf4c0f239afc408b134d24ce74b5b7c86b3e0d

- Torrent

		magnet:?xt=urn:btih:875cb7ced0127c67f466a08de8945b5f936fc020&dn=SF_EB_1.0_pruned_vae.zip&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://anonfiles.com/icT4e1Eeyf/SF_EB_1.0_pruned_vae_zip
	or 

		https://gofile.io/d/MYOKl8

#### EBL 1.0 [758fda47]
使用扩展架构（1.8B 参数）的模型，从 EB 1.0 恢复。 接受过各种姿势各异的色情照片的训练。

不能使用 webui 开箱即用（至少目前是这样）。

- Code https://github.com/smirkingface/stable-diffusion
- Info https://www.patreon.com/posts/released-models-73689798
- 使用 auto 的 webui 运行 https://www.patreon.com/posts/running-extended-73768212
- 带有预览的一般使用说明 https://www.patreon.com/posts/73775030

SF_EBL_1.0_vae.ckpt [758fda47] SHA256 e237715adaf1b3e43eb2eee792fe82a90d01a45beb79d38807f5ca92b7a277db

- Torrent

		magnet:?xt=urn:btih:8c4cafe3b148c4337e381dbc0a18fc03a2eb7ac2&dn=SF_EBL_1.0_vae.zip&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://anonfiles.com/I6y3feEay4/SF_EBL_1.0_vae_zip
	or 

		https://gofile.io/d/MYOKl8

### gg1342_testrun1_pruned.ckpt [43076286] [2ccc3e58]
对 280 张 NSFW 图像（裸体单身女性）和另外 80 张几个虚构人物的 SFW 图像进行训练的测试运行。

	gg1342_testrun1_pruned.ckpt [43076286] | SHA256 ebe2f8dfeed0b87080a37e94bf0aca8800fc10691826a5e76259faf509106246
	gg1342_testrun1_pruned-float16.ckpt [2ccc3e58]

- Torrent

		magnet:?xt=urn:btih:c95e266e15e13cf0e2d69b29338a89a94d736546&dn=gg1342_testrun1_pruned.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce

### Pyro's Blowjob Model v1.0 [9b5251e8]
真实

当前基于 dreambooth 的口交模型很烂（呵呵），所以我决定对此做一个“真实”的尝试并以本地方式对其进行微调（例如 WaifuDiffusion 对他们的模型所做的）

- 基本型号：SD 1.5
- 2899 个输入图像
- 23192 步 x 批量大小 4
- LR 5e-06

信息和可能更新的版本：[https://rentry.org/pyros-sd-model](https://rentry.org/pyros-sd-model)

- Torrent

		magnet:?xt=urn:btih:5c5d96dfb4eda3d4eb0f8f67c695b8b8ac241060&dn=pyros-bj-v1-0.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zerobytes.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.srv00.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.encrypted-data.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://anonfiles.com/6123m6F6y9/pyros-bj-v1-0_ckpt
	or 

		https://mega.nz/file/lLtjCLwb#KgXPDzbTotcb0_quzzBMm6DaDCuSFIaF8CXxw1WsEs8

### Easter e3 [9c5ade34]
一般淫荡模特《Easter.ckpt》！ 使用来自 gelbooru 和 e621 的大约 180k 图像进行训练。

包括 NSFW 和利基内容。 我对模型的输出不承担任何责任！

- 模型在前面似乎比后面好。
- 建议的否定提示：“Letterbox”。
- 这个名字来源于它倾向于混合兔子和鸡蛋。

目前在 epoch 3。

- Torrent

		magnet:?xt=urn:btih:79d1adf55ab8336e095215f3d293fa3c29d57528&dn=easter_e3.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zerobytes.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.srv00.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.encrypted-data.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://mega.nz/file/l3RRiIzI#hHGR6rYAJ0-weE8kef7bBUq5u2WjI3oTP92A7atTINw

### Hentai Diffusion
repo https://huggingface.co/Deltaadams/Hentai-Diffusion/tree/main

据说每周都会更新，所以检查一下 repo，它可能比这个列表更新。

以下修剪过的种子已删除 EMA 权重。

该模型基于 Waifu Diffusion 1.2，并在来自 R34 和 gelbooru 的 150,000 张图像上进行了训练。

对更隐蔽的姿势进行了集中训练，例如蹲下和背对观察者，同时注重改善手部。

建议在提示中使用来自站点的标签。 在提示的开头添加 1girl 将有助于获得更好的结果。

### HD-16.ckpt [e2ec4647]
repo  https://huggingface.co/Deltaadams/Hentai-Diffusion/tree/main

据说每周都会更新，所以检查一下 repo，它可能比这个列表更新。

	SHA256 4c9317fe064386b0d8645375d0f8a0059455cfee0b84d2e280d5cc2fc9e88c21

- web

		https://huggingface.co/Deltaadams/Hentai-Diffusion/resolve/main/HD-16.ckpt

### RD1412.ckpt [2140af02] [5b87f0e6] [4fdde306]
Repo https://huggingface.co/Deltaadams/Hentai-Diffusion/tree/main

据说每周都会更新，所以检查一下 repo，它可能比这个列表更新。

	RD1412-pruned-fp16.ckpt [2140af02] | SHA256 c02f07fc72ea7281a14c94469428a497b142f70f6b5b4ea20435fbf5ef3f1731
	RD1412-pruned-fp32.ckpt [5b87f0e6] | SHA256 c318e7155f30af74785503d8ebf2856813e234032bd5c043c814e2f8baa4d67e
	RD1412.ckpt [4fdde306] | SHA256 477b860a6859838e99d81fa5d3345e717e11fe8bd0e7eb32bce7541d57807a4d

- Torrent RD1412-pruned-fp16.ckpt [a1f5a67e]

		magnet:?xt=urn:btih:da8986f9059ce4f64f84e7390eb542558b2cd466&dn=RD1412-pruned-fp16.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt.oiyo.tk%3a6969%2fannounce&tr=https%3a%2f%2ftracker.nanoha.org%3a443%2fannounce&tr=https%3a%2f%2ftracker.lilithraws.org%3a443%2fannounce
- Torrent RD1412-pruned-fp32.ckpt [37b5398c]

		magnet:?xt=urn:btih:ab4c2d7308a3fa694f7409407399a1cc5d4c7ed9&dn=RD1412-pruned-fp32.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt.oiyo.tk%3a6969%2fannounce&tr=https%3a%2f%2ftracker.nanoha.org%3a443%2fannounce&tr=https%3a%2f%2ftracker.lilithraws.org%3a443%2fannounce
- Web Full EMA RD1412.ckpt [4fdde306]

		https://huggingface.co/Deltaadams/Hentai-Diffusion/resolve/main/RD1412.ckpt

#### RD1212.ckpt [a1f5a67e] [37b5398c] [3b3459c8]
Repo https://huggingface.co/Deltaadams/Hentai-Diffusion/tree/main

据说每周都会更新，所以检查一下 repo，它可能比这个列表更新。
	
	HenDiff_RD1212-pruned-fp16.ckpt [a1f5a67e] | SHA256 8e46950f3d700886ced3842e4f05c31a1ab30b856668e9bc990940e60ad21be1
	HenDiff_RD1212-pruned-fp32.ckpt [37b5398c] | SHA256 c303f408d538544765b109aef0e2f165864cab720365a1d3a1d166237965095a
	RD1212.ckpt [3b3459c8] | SHA256 23c56765c35a9b205e52f64b46c6311279cc3a56e6d17eba32779ff590bdc3d7

- Torrent HenDiff_RD1212-pruned-fp16.ckpt [a1f5a67e]

		magnet:?xt=urn:btih:f4e78d085169d2077a316bd9b75723812c1ab429&dn=HenDiff_RD1212-pruned-fp16.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt.oiyo.tk%3a6969%2fannounce&tr=https%3a%2f%2ftracker.nanoha.org%3a443%2fannounce&tr=https%3a%2f%2ftracker.lilithraws.org%3a443%2fannounce
- Torrent HenDiff_RD1212-pruned-fp32.ckpt [37b5398c]

		magnet:?xt=urn:btih:2a6b60f454dcf89b81e7db034fcb1536b774628c&dn=HenDiff_RD1212-pruned-fp32.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt.oiyo.tk%3a6969%2fannounce&tr=https%3a%2f%2ftracker.nanoha.org%3a443%2fannounce&tr=https%3a%2f%2ftracker.lilithraws.org%3a443%2fannounce
- Web Full EMA RD1212.ckpt [3b3459c8]

		https://huggingface.co/Deltaadams/Hentai-Diffusion/resolve/main/RD1212.ckpt
- Torrent Full EMA RD1212.ckpt [3b3459c8]

		magnet:?xt=urn:btih:D0B89A0516205157EA0CBDDBBB49BC60C611A3B7&dn=RD1212.ckpt&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce

### Bare Feet / Full Body b4_t16_noadd [03e33bb4] [2bd8166c] [9012c514]
仅限高质量标签（“精选”模型）。

预览网格 [https://i.imgur.com/2sJGz3j.jpeg](https://i.imgur.com/2sJGz3j.jpeg)

一个非常 WIP 的检查点，专注于赤脚 (BF) 全身 (FB) 裸体女性图像 - 如果您正在尝试准确的照片级逼真的脚或女性生殖器，则最适合使用，这两者都是普通 SD 难以处理的
非常WIP

极度专业化，失去了很多通用能力和风格化能力。

- Torrent bf_fb_v3_t4_b16_noadd-ema-pruned-fp16.ckpt [03e33bb4]

		magnet:?xt=urn:btih:9530a8a0b43f83366216ab853b4419aa2056da58&dn=bf_fb_v3_t4_b16_noadd-ema-pruned-fp16.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.skyts.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt.oiyo.tk%3a6969%2fannounce
- Torrent bf_fb_v3_t4_b16_noadd-ema-pruned-fp32.ckpt [2bd8166c]

		magnet:?xt=urn:btih:1f6bab17c548e35ac2a412e3e9119e5f4e00bb50&dn=bf_fb_v3_t4_b16_noadd-ema-pruned-fp32.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.skyts.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt.oiyo.tk%3a6969%2fannounce

### Lewd Diffusion
#### Lewd Diffusion 70k 2e [f4030c43]
- LD-70k-2e-pruned.ckpt
- 来自 danbooru 的 7 万张露骨图片
- 2个 epochs 时代
- 基础模型 Waifu Diffusion v1.2

		数据集：https://drive.google.com/drive/folders/1f_BYi88LLTZUzBHkUz8PDgw6l7M7swkd?usp=sharing
		数据集统计：https://docs.google.com/spreadsheets/d/1BzNSXyT4fhiM64DwIJSCyAXuhRQ9fkxqcr-t1frIYkc/edit

	SHA256 54faf6de03fb9a6fe4d8af163b16133cd7cd045d45915178c602e4b51a92036c
- Torrent

		magnet:?xt=urn:btih:U5RICVYDEJL6LIJJWFKQOIVO5GMGCJNW&dn=last-pruned.ckpt&xl=3852165809&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce

#### Lewd Diffusion 70k 1e [950d323b]
- LD-70k-1e-pruned.ckpt
- 70k explicit from danbooru
- 1 epoch
- Base model Waifu Diffusion v1.2

		Dataset: https://drive.google.com/drive/folders/1f_BYi88LLTZUzBHkUz8PDgw6l7M7swkd?usp=sharing
		Dataset stats: https://docs.google.com/spreadsheets/d/1BzNSXyT4fhiM64DwIJSCyAXuhRQ9fkxqcr-t1frIYkc/edit

- Torrent

		magnet:?xt=urn:btih:fca8782a5a9861a6beb1aa3b48938bd1da1a665e&dn=LD-70k-1e-pruned.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce

#### Lewd Diffusion v0 [07734b46]
- Lewd-diffusion-pruned.ckpt
- Danbooru21 NSFW epoch0
- 40K Images
- 1 Full Epoch
- Base model Waifu Diffusion v1.2

		Dataset: https://drive.google.com/drive/folders/1f_BYi88LLTZUzBHkUz8PDgw6l7M7swkd?usp=sharing

- Torrent

		magnet:?xt=urn:btih:f6976fbe3b9f93469bb62eb0c4950643b09f1f83&dn=Lewd-diffusion-pruned.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce

### Yiffy
#### yiffy-e18.ckpt [50ad914b]
基于 yiffy-e15，使用来自 e621 的 210k 图像数据集再训练 3 个 epoch。

这可能是相当长一段时间内的最后一个 epoch，因为在对网络的其他部分（例如自定义 VAE、变量 res 等）进行进一步改进之前，我们认为没有必要进一步训练，因为模型已经收敛到 体面的风格现在。

重要提示：在训练过程中，显式被拼写为显式。 使用拼写错误的版本会产生更好的公鸡。 对不起：<

标签计数 https://mega.nz/file/ZkkTEYJY#X_j191NtHJRp0BKJusTqmWrs1-AdAILy99mqSAMpWbo or https://pixeldrain.com/u/TkwJU3WG

	SHA256 64e242ae67cb1fc85110a68242c936fa6f3e33077f2bfed48c4769959dff6c61
	MD5 dbe25794e24af183565dc45e9ec99713
- Torrent

		magnet:?xt=urn:btih:b177dd04ae7062b541c82ad26f897e0a9fa514f4&dn=yiffy-e18.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce
- Web

		https://sexy.canine.wf/file/yiffy-ckpt/yiffy-e18.ckpt

#### yiffy-e15.ckpt [4bb305c0]
基于 yiffy-e13，使用来自 e621 的 210k 图像数据集再训练 2 个 epoch。

重要提示：在训练过程中，显式被拼写为显式。 使用拼写错误的版本会产生更好的公鸡。 对不起：<

标签计数  https://mega.nz/file/ZkkTEYJY#X_j191NtHJRp0BKJusTqmWrs1-AdAILy99mqSAMpWbo or https://pixeldrain.com/u/TkwJU3WG

	SHA256 f0d49adddcb9f1030cb13e4eed21d12f27cdd3d98a9740002d0ccdb4a9fa61f1
	MD5 cc354964edbcbb7758cb359743aa1791

- Torrent

		magnet:?xt=urn:btih:2b8d5f308244eddf56d4a350df84d63045e65dd6&dn=yiffy-e15.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
- Web

		https://sexy.canine.wf/file/yiffy-ckpt/yiffy-e15.ckpt
	or

		https://pixeldrain.com/u/qkRKKpqg
	or

		https://iwiftp.yerf.org/Furry/Software/Stable%20Diffusion%20Furry%20Finetune%20Models/Finetune%20models/yiffy-e15.ckpt


#### yiffy-e13.ckpt [778b38ae]
- 前 4 个 epoch 在大约 70k 图像上进行了 lama 填充训练（这是我们所有头痛的原因，因为网络在边缘发现了一个模式并开始在任何地方复制它）
- 接下来的 6 个 epoch 在大约 120k 图像上进行了训练，随机裁剪和较低的 LR
- 最后一个时期是在不同的数据集上完成的，不大于 150k

要提示：在训练过程中，显式被拼写为显式。 使用拼写错误的版本会产生更好的公鸡。 对不起：<

标签计数  https://mega.nz/file/ZkkTEYJY#X_j191NtHJRp0BKJusTqmWrs1-AdAILy99mqSAMpWbo or https://pixeldrain.com/u/TkwJU3WG

		MD5 bfff21ca92fb218e52ab47aea8bac582
		SHA256 3d3f7cc7876ce79697038c620ba723d785be26a18a9734ea10a31fae63992b65

- Torrent

		magnet:?xt=urn:btih:6d749325cbdcf1fc044483fb0d53c233b60735dc&dn=yiffy-e13.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
- Web

		https://iwiftp.yerf.org/Furry/Software/Stable%20Diffusion%20Furry%20Finetune%20Models/Finetune%20models/yiffy-e13.ckpt
### Furry
#### Furry_epoch4.ckpt [323f8dd8]
在来自 e621 的 300k 图像上进行训练。

要提示： https://mega.nz/file/co0UlQ5Z#vERcoYTWGJguTsXmysbLq1NL_xBS8txQhVvPI5E3QKE or https://pixeldrain.com/u/FQwRjyyk

	MD5 f8ef45a295ef4966682f6e8fc2c6830d
	SHA256 4160c57f98f1727f5a52cba8c844656fc9061311f9c37daf45e8e0ebe913c987

- Torrent

		magnet:?xt=urn:btih:a9635389ae4c5583b0cc76ec8f6dce35438b3016&dn=furry_epoch4.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
- Web

		https://pixeldrain.com/u/dtYiYN7g
	or

		https://iwiftp.yerf.org/Furry/Software/Stable%20Diffusion%20Furry%20Finetune%20Models/Finetune%20models/furry_epoch4.ckpt

#### Furry_epoch1.ckpt [0c891127]
Trained on 300k images from e621.
Tag counts: https://mega.nz/file/co0UlQ5Z#vERcoYTWGJguTsXmysbLq1NL_xBS8txQhVvPI5E3QKE or https://pixeldrain.com/u/FQwRjyyk

- Torrent

		magnet:?xt=urn:btih:d62bc9a088b206565005cab915a58fd26da1802e&dn=furry_epoch1.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
- Web

		https://iwiftp.yerf.org/Furry/Software/Stable%20Diffusion%20Furry%20Finetune%20Models/Finetune%20models/furry_epoch1.ckpt
#### Furry_epoch0.ckpt [8c19ee5a]
Trained on 300k images from e621.
Tag counts: https://mega.nz/file/co0UlQ5Z#vERcoYTWGJguTsXmysbLq1NL_xBS8txQhVvPI5E3QKE or https://pixeldrain.com/u/FQwRjyyk

- Web

		https://iwiftp.yerf.org/Furry/Software/Stable%20Diffusion%20Furry%20Finetune%20Models/Finetune%20models/furry_epoch0_ckpt

### Zack3D_Kinky-v1.ckpt [1a75d5c6]
这是一个从 SD v1.4 训练的 NSFW 模型，非常适合一般的毛茸茸，但专门研究艺术的古怪方面，例如
变形，乳胶，触手，粘液，野性，束缚，`Transformation, latex, tentacles, goo, ferals, bondage` 还有很多我不记得的东西。 使用超过 10 万张图片，所有美学都经过过滤。
提示是原始的 E621 标签，所以带有下划线。

	SHA1 9f54dcdbdb2e4c969a9d1b70ba34f02227b8ce10
	MD5 aa944b1ecdaac60113027a0fdcda4f1b
	SHA256 3FD17AF06C610990A957A801A1831F88EF5E45D9C3C8E551557B8B9C6C1FC2D5

- Torrent

		magnet:?xt=urn:btih:807a71d3ed3f887e41c492cf24fbd3c6f5a81534&dn=Zack3D_Kinky-v1.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2fopen.tracker.cl%3a1337%2fannounce
- Web

		https://pixeldrain.com/u/DEocAHsx

### Anal Vore AVHumanFurryPony7.ckpt [68e2e32d]
肛交. 7 epochs continued from Zack3D_Kinky-v1.

Tags https://mega.nz/file/FmxSnRZa#I4JgsLTiXWoFUoDrQBWfVNnooUUOxqkFHEOPuCE1Hdc
用例 prompts https://mega.nz/file/hu5QnJgJ#DFmvHDuEpm7OdDkQD_wIRrCo7x8j3lqYkhC4aS-egvs

- Web

		https://mega.nz/file/4jR2WAIZ#p0A7eorXgI_ywC06zQiLcPwE3QdStEa86wwEdrDIE7A

### R34
#### r34_e4.ckpt [6e45cf2c]
第五纪。
Tag counts: https://mega.nz/file/4sFn0AxQ#pHDvIE5eKJC3NSNpgdEw-ve0WSlWjtsTvfpe7YaYh1Q

- Torrent

		magnet:?xt=urn:btih:ed9f0e3f849d7119107ef4e072c6abeb129e1a51&dn=r34_e4.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2fpublic.tracker.vraphim.com%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2ffe.dealclub.de%3a6969%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce
- Web

		https://mega.nz/file/yJgDUCzA#zOD2yeE6QLBqPEjEpIi2b4FWOlb64yVUveOd_eW6teI

#### r34_e3.ckpt [57ea6f43]
r34 e3（第四个纪元）。 相同的数据集，在艺术家标签前加上“by”，艺术家识别有所提高（仍然需要变得更好），字符识别有所提高。

- Web

		https://mega.nz/file/WVggzbRa#4-pcaLeLN02bdvzUwCn2QQKoLxCC7copAJHYyG4aOB0

#### r34_e2.ckpt [778b68b1]
第三个时代，又是一个相当不错的改进。 数据集增长了约 3K 图像，其中删除了 5K 并添加了 8K。

- Torrent

		magnet:?xt=urn:btih:ef93096b839a54bff77c585b1ba2518bc264d0b4&dn=r34_e2.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.filemail.com%3a6969%2fannounce
- Web

		https://mega.nz/file/3JQ13JTZ#sdByZl4rXp7uxxIDutGs5GHg5RkGZfLrTA5wQEfkDJE

#### r34_e1.ckpt [f9000e4e]
我收到了大约 30K 的图像，然后从优秀的艺术家那里添加了 20K，并显着降低了 LR 以反映我的批量大小，我认为它得到了显着改善，但我没有保存 epoch0 中的任何提示/设置来比较它。

	SHA256 6ca821f381687796052aa027f60a2dff1aeb001a0c4a2e1f1a1fef4fe89e809a

- Torrent

		magnet:?xt=urn:btih:aa575688ad99cf4bf8e5ee8ab026de9ef61f2d19&dn=r34_e1.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt.oiyo.tk%3a6969%2fannounce&tr=https%3a%2f%2ftracker.nanoha.org%3a443%2fannounce&tr=https%3a%2f%2ftracker.lilithraws.org%3a443%2fannounce
- Web

		https://mega.nz/file/OdxnkAaQ#-9VBG4jyPqlQIcRRnojtNWvhdJQFbmmfGU9I2QGWV5o

#### r34_150k_epoch0.ckpt [7dc34283] [7c3721c3]
使用来自 rule34.xxx 的 150K 图像进行训练。 1个时代。

Tag counts: https://mega.nz/file/UhsWjBCI#XOvN31pil7_g9YgtzpchpJwyOnVHxrE-dlhxBLYku0I

	r34_150k_epoch0-pruned-fp16.ckpt [7dc34283] | SHA256 f8c7da0d6a98ebd641cb67c24f33510e2c86d1dfa104c10279c7e5cb49d535fd
	r34_150k_epoch0.ckpt [7c3721c3] | SHA256 de2ad17e4af8871cdd42bd03de4faf28983c0ef8702323bbc3d382510d367a91

- r34_150k_epoch0-pruned-fp16.ckpt [7dc34283] Torrent

		magnet:?xt=urn:btih:a8f34e4f7f0bc5298516082b41b5e920b8cc483c&dn=r34_150k_epoch0-pruned-fp16.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.0x.tf%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fbt2.archive.org%3a6969%2fannounce
- r34_150k_epoch0.ckpt [7c3721c3] Torrent

		magnet:?xt=urn:btih:PVTASWQ5GXC4NXGKAZU5LVXJZX3PR6R2&dn=r34_150k_epoch0.ckpt&xl=11142098457&tr=udp%3A%2F%2Ftracker.pomf.se%3A80%2Fannounce
- r34_150k_epoch0.ckpt [7c3721c3] Web

		https://mega.nz/file/fdpVCZqB#C3G1_Qs9K65oh_9RHIJoiO1eE5Ns07mvqVdVYXq9uoo

### Gape
#### gape60 [25396b85]
- 第 60 个 epoch（120k 步）
- 在 danbooru、gelbooru、sankaku 上训练的张开/大插入/其他模型：所有张开的肛门、张开的猫、大插入、巨大的假阳具、拳交、脱垂，一直到 + 其他一些。

Info https://rentry.org/gapemodel

- Torrent

		magnet:?xt=urn:btih:b892f6e7e6e287ca2ea59d2c575fdf5ae1f82d92&dn=gape60.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.zerobytes.xyz%3A1337%2Fannounce&tr=udp%3A%2F%2Ftracker.zemoj.com%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.pomf.se%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.monitorit4.me%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.moeking.me%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
- Web

		https://mega.nz/file/38RxVBiK#ocmQb_ahp078Imfcvc5SrAvJePOqvhNVBKfZIIeP74Q

#### gape22 [d69e1a23]
- 大概是第22个纪元。
- 在 sankaku 和 e621 数据上训练的 gaping 模型：所有 gaping_anus、gaping_pussy、large_penetration、fisting、prolapse 等（省略了一些 shit）。基于 yiffy15

Info https://rentry.org/gapemodel

- Web

		https://mega.nz/file/29hmjYaA#GDdvgwJKKNFLMiaYA2RpuxFk7ormSrS32c05-s3JJCo
- Torrent

		magnet:?xt=urn:btih:9d7aabfbac88940a0f93b076f3ce30bb4b06b7e8&dn=gape22_yiffy15.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.pomf.se%3A80%2Fannounce&tr=udp%3A%2F%2Fp4p.arenabg.com%3A1337%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Fopen.demonii.com%3A1337%2Fannounce&tr=udp%3A%2F%2Fmovies.zsw.ca%3A6969%2Fannounce&tr=udp%3A%2F%2Fipv4.tracker.harry.lu%3A80%2Fannounce&tr=udp%3A%2F%2Fexplodie.org%3A6969%2Fannounce&tr=udp%3A%2F%2Fexodus.desync.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fbt.oiyo.tk%3A6969%2Fannounce&tr=https%3A%2F%2Ftracker.nanoha.org%3A443%2Fannounce&tr=https%3A%2F%2Ftracker.lilithraws.org%3A443%2Fannounce&tr=https%3A%2F%2Ftr.burnabyhighstar.com%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
		
#### gape18
- 大概是第18个纪元。
- 在 sankaku 和 e621 数据上训练的 gaping 模型: all of gaping_anus, gaping_pussy, large_penetration, fisting, prolapse, etc (some shit omitted). based on yiffy15.

Info https://rentry.org/gapemodel

- Torrent

		magnet:?xt=urn:btih:9c0fac3978e650e0fa18c18ee74dc35479774c08&dn=gape18_yiffy15.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.pomf.se%3A80%2Fannounce&tr=udp%3A%2F%2Fp4p.arenabg.com%3A1337%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Fopen.demonii.com%3A1337%2Fannounce&tr=udp%3A%2F%2Fmovies.zsw.ca%3A6969%2Fannounce&tr=udp%3A%2F%2Fipv4.tracker.harry.lu%3A80%2Fannounce&tr=udp%3A%2F%2Fexplodie.org%3A6969%2Fannounce&tr=udp%3A%2F%2Fexodus.desync.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fbt.oiyo.tk%3A6969%2Fannounce&tr=https%3A%2F%2Ftracker.nanoha.org%3A443%2Fannounce&tr=https%3A%2F%2Ftracker.lilithraws.org%3A443%2Fannounce&tr=https%3A%2F%2Ftr.burnabyhighstar.com%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce

### sd-wikiart-v2.ckpt. [8f32b8df]
sd-wikiart-v2 是一个stable diffusion 模型，已在 wikiart 数据集上进行了微调，以生成不同风格和流派的艺术图像。

当前模型已在来自 wikiart 数据集的 81K 文本图像对上以 1e-05 的学习率为 1 epoch 进行了微调。只有模型的注意力层被微调。这样做是为了避免灾难性的遗忘，该模型可以在给定特定提示的情况下生成艺术图像，但仍保留其大部分先前的知识。

Info https://huggingface.co/valhalla/sd-wikiart-v2

- Torrent

		magnet:?xt=urn:btih:474493890c2532031fefc8ba3c44d8fed61f287a&dn=sd-wikiart-v2.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce

### Pachu-Diffusion [54d5d199]
这个经过微调的 Stable Diffusion v1.5 模型在 Pachu M. Torres 的作品上进行了 8000 次迭代训练。 使用 ShivamShrirao/diffusers 进行训练，具有全精度、预先保存损失、训练文本编码器功能和来自 Stability AI 的新 1.5 MSE VAE。 总共使用了 4120 个正则化/类图像。 使用提示“艺术风格”、50 DDIM 步骤和 7 的 CFG 生成正则化图像。

在提示中使用令牌 pachu 艺术风格来获得效果。

Repo https://huggingface.co/ProGamerGov/Pachu-Diffusion

- Web

		https://huggingface.co/ProGamerGov/Pachu-Diffusion/resolve/main/pachu_artwork_style_v1_iter8000.ckpt

### TestFurry_5.ckpt [b1f1855e]
小型模型，使用 40k 混合毛茸茸和人类（约 15k 人类和 25k 毛茸茸）的数据集进行训练。没有审查制度，没有泡泡，没有漫画。 Gelbooru/e621 混合标签。 5 epoch（还没有完成但可以做很好的结果）NSFW！！

- 标签订单推荐 - 类型（如果毛茸茸的，则毛茸茸的），角色，来源，作者，种族，性别， (furry if furry), character, source, author, race, gender,其他描述标签。

训练多达 225 个标签，带有倒数第二层（AUTO GUI 中的 CLIP 2）

- Torrent

		magnet:?xt=urn:btih:0c0bbbf3182305d6e7e4707e566f2c60cf4e890f&dn=TestFurry_5.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce

### cookie_sd_pony_run_a12 [2ce7dcf5] [67ff5385]
这是“Cookie A12”模型。

在 300k Derpibooru 图像上训练，分数为 100，共 2172700 步。

新的功能;

- 改进的 VAE
- 改进的 CLIP 文本编码器
- 以任何纵横比生成图像
- 以高精度生成任何 derpibooru 艺术家或 OC。
- 生成 Anthro、Humanized、EqG、Dragon？，无论 derpibooru 中存在什么。
- 生成没有“魔法词”的高质量结果 (https://u.smutty.horse/mjzakocpxtf.png)
- 没有审查标签或数据，NSFW+SFW。
- 笔记：

	保持 CLIP 跳过。 如果您跳过任何图层，改进的 CLIP 将会中断。

- Samples:
	- https://u.smutty.horse/mjzakocpxtf.png
	- https://drive.google.com/drive/folders/1nOZKI2iXpzjusvlCsZxF7NxKZYjvKOWE?usp=share_link

- Torrent float16

		magnet:?xt=urn:btih:54d2d1d8f36d9ee192a3b84e6e6c0ddb68f4e891&dn=cookie_sd_pony_run_a12_datasetv5_300k_imgs_fp16.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zerobytes.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.encrypted-data.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web float16

		https://sexy.canine.wf/file/furry-ckpt/cookie_sd_pony_run_a12_datasetv5_300k_imgs_fp16.ckpt
- Web float32

		https://sexy.canine.wf/file/furry-ckpt/cookie_sd_pony_run_a12_datasetv5_300k_imgs_fp32.ckpt
- Web

		https://mega.nz/folder/rIZwEaLZ#AwCpM7Zhu8TFgISQhAE8tw/folder/LRxjWBaJ

### pony-diffusion
#### pony-diffusion-v2 [5bdc40aa]
pony-diffusion 是一种潜在的 text-to-image 的 diffusion 模型，它通过微调以高质量的小马和毛茸茸的 SFW 和 NSFW 图像为条件。

警告：与此模型的 v1 相比，v2 更能够生成 NSFW 内容，因此建议在提示中使用“ safe ”标签，并结合否定提示用于您可能想要抑制的图像特征。 v2 模型也有轻微的 3d 偏差，因此应使用“3d”或“sfm”等负面提示来更好地匹配 v1 输出。

Repo https://huggingface.co/AstraliteHeart/pony-diffusion-v2

- Web

		https://mega.nz/file/Va0Q0B4L#QAkbI2v0CnPkjMkK9IIJb2RZTegooQ8s6EpSm1S4CDk
#### pony-diffusion-v1 [9070b574]
在 Waifu Diffusion 的早期检查点之上进行微调，在大约 80k 小马文本图像对（使用来自 derpibooru 的标签）上的 4 个 epoch 的学习率为 5.0e-6，它们的分数都大于 500 并且属于安全类别 或暗示。

Info https://huggingface.co/AstraliteHeart/pony-diffusion

	SHA256 507dd62efcb01db04aeb1b68eb28912d1fe692c04637e463cc43e68752729627

- Web

		https://mega.nz/file/ZT1xEKgC#Xxir5udMmU_mKaRZAbBkF247Yk7DqCr01V0pDzSlYI0

### mio-wd-v1.2-e24-ex-ad [125f9ece]
在 Mio (nichijou) 上训练了 500 个 (aprox) 图像的小型模型。 24 个时代。

Info: https://huggingface.co/chavinlo/mio-naganohara-waifu-diffusion

	SHA256 4744ed99c119096fa144df9aa8c869d933e3b6a239786462c5cc399ff1d78698

- Torrent

		magnet:?xt=urn:btih:MS2YZ762U2IJZBG2GSEADRAA5KAOQCVQ&dn=mio-wd-v1.2-e24-ex-ad&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce
- Web

		https://huggingface.co/chavinlo/mio-naganohara-waifu-diffusion/resolve/main/epoch%3D000023-pruned.ckpt

### fubuki-ld-v1-e13-ex-ad [2af6d20f]
使用 Hololive 的 5,000 张 Fubuki 图像训练的中型模型。 13 Epochs Pruned，3.8GB，包括样本。

- Torrent

		magnet:?xt=urn:btih:D37QX2URIZ7QWMBCCLMZG6THZF5GLDB2&dn=fubuki-ld-v1-e13-ex-ad&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce

### asuka-ld-v1-e4-ex-ad [87074080]
小型模型训练了来自新世纪福音战士的 17,000 张明日香图像。 4 Epochs, Pruned, 3.6GB，不包括样本。

- Torrent

		magnet:?xt=urn:btih:4GBOL6SMWLZWT2CBUWVMSF5NCGD47CGZ&dn=asuka-ld-v1-e4-ex-ad&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce
### tomoko-kuroki-ld-v1-e20-ex-ad [6095e7ab]
在未知数量的黑木智子图像上训练的小型模型。 20 Epochs, Pruned, 3.5GB, 不包括样本。

- Torrent

		magnet:?xt=urn:btih:3OFJJ76Y2K4W47RJ7EXFKI3BAZPPVTQM&dn=tomoko-kuroki-ld-v1-e20-ex-ad&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce

### 70gg30LD70k.ckpt [402dc090]
gg1342_testrun1_pruned.ckpt 和 LD-70k-1e-pruned.ckpt 之间的合并，比例为 70-30。

此列表不适用于合并，您可以使用 WebUI 自己制作。

这个在这里是因为它经常被引用（并被要求）。

- Torrent

		magnet:?xt=urn:btih:5667722f227d1534232182de77730f8aa7ff8212&dn=70gg30LD70k.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fzecircle.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fyahor.ftp.sh%3A6969%2Fannounce&tr=udp%3A%2F%2Fvibe.sleepyinternetfun.xyz%3A1738%2Fannounce&tr=udp%3A%2F%2Fv2.iperson.xyz%3A6969%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker2.dler.org%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.tcp.exchange%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.swateam.org.uk%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.publictracker.xyz%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce
		
### wd1-2_sd1-4_merged.ckpt [2037c511]
Waifu Diffusion v1.2 和 Stable Diffusion v1.4 之间的合并，比例未知。

此列表不适用于合并，您可以使用 WebUI 自己制作。

这个在这里是因为它经常被引用（并被要求）。

- Torrent

		magnet:?xt=urn:btih:a1515e051c61ac55e64a7961da956dc2f5c00e90&dn=wd1-2_sd1-4_merged.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.moeking.me%3A6969%2Fannounce&tr=udp%3A%2F%2Fp4p.arenabg.com%3A1337%2Fannounce&tr=udp%3A%2F%2Fopen.demonii.com%3A1337%2Fannounce&tr=udp%3A%2F%2Fipv4.tracker.harry.lu%3A80%2Fannounce&tr=udp%3A%2F%2Fexplodie.org%3A6969%2Fannounce&tr=udp%3A%2F%2Fexodus.desync.com%3A6969%2Fannounce&tr=https%3A%2F%2Ftracker.nanoha.org%3A443%2Fannounce&tr=https%3A%2F%2Ftracker.lilithraws.org%3A443%2Fannounce&tr=https%3A%2F%2Ftr.burnabyhighstar.com%3A443%2Fannounce&tr=http%3A%2F%2Ftracker.mywaifu.best%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.dler.org%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.bt4g.com%3A2095%2Fannounce


### Hiten
基于 Waifu Diffusion v1.2，对艺术家 Hiten 进行了培训。 更多信息[请查看](https://huggingface.co/BumblingOrange/Hiten)。 我听说人们说这个模型最好与 Waifu Diffusion 或 trinart2 合并，因为它可以改善颜色。

要使用该模型，请将 Hiten 插入到您的提示中。 为了获得更好的结果，请在 Hiten 之后附加 `girl_anime_8k_wallpaper`（类标记）（例如：`Hiten` 的 `1girl by Hiten girl_anime_8k_wallpaper`）。

- web 下载

		https://huggingface.co/BumblingOrange/Hiten/resolve/main/Hiten%20girl_anime_8k_wallpaper_4k.ckpt

### WD v1.2 和 SD v1.4 合并
WD 合并回 SD，我不知道比例是多少。 这是最早的合并之一。

- Torrent

		magnet:?xt=urn:btih:UFIV4BI4MGWFLZSKPFQ5VFLNYL24ADUQ&dn=wd1-2_sd1-4_merged.ckpt&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce

### NovelAI 泄露模型
10 月 7 日，NoveAI 的服务器被 4chan 的某个人入侵。 他偷走了他们的 Stable Diffusion 和 GPT 模型。 没有用户信息/PII。

- 第1部分

	55GB，包含 NovelAI 使用的主要模型，位于 stableckpt 文件夹中。 我建议使用您的 torrent 客户端来准确下载您想要的内容或 使用此[脚本](https://github.com/Cyberes/stable-diffusion-models/blob/main/download-novelai-models.sh)。 您可以使用本[指南](https://rentry.org/sdg_FAQ#how-do-i-setup-the-leaked-novelai-model)进行设置
	
	- Torrent
	
			magnet:?xt=urn:btih:5bde442da86265b670a3e5ea3163afad2c6f8ecc&dn=novelaileak&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce
- Part 2

	103GB，包含更多 GPT 模型和开发中的稳定扩散模型。 你在这里真的不需要任何东西。
	
	- Torrent
		
			magnet:?xt=urn:btih:a20087e7807f28476dd7b0b2e0174981709d89cd&dn=novelaileakpt2&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=https%3A%2F%2Ftracker.nanoha.org%3A443%2Fannounce

### 未列出的型号
那里有很多未列出的 `.ckpt` 种子。 使用 DHT 搜索引擎搜索 “.ckpt”

[https://bt4g.org/search/.ckpt](https://bt4g.org/search/.ckpt)
### Dreambooth
HuggingFace 发布社区提交的 Dreambooth 模型。 在这里查看它们。

[https://cyberes.github.io/stable-diffusion-dreambooth-library/](https://cyberes.github.io/stable-diffusion-dreambooth-library/)

它每天自动更新两次。
#### Hiten girl_anime_8k_wallpaper_4k.ckpt [7831dea3]
- 用于合并。
- 基于艺术家 Hiten 的定制 Dreambooth 模型。 数据集是 6 个训练图像和 34 个正则化图像。 训练了 4000 步。
- 使用 Waifu Diffusion v1.2 模型作为基础。
- 要使用该模型，只需在提示中插入名称“Hiten”。 使用的类令牌是“girl_anime_8k_wallpaper”。 在 Hiten 之后附加类标记以获得更强的结果。 例如：“1girl by Hiten girl_anime_8k_wallpaper”

Info: https://huggingface.co/BumblingOrange/Hiten

	SHA256 | 59cb5b6f2b7b9373a602d93c802ee841c367db5166a758e8d1d32464c9db1dd5

- Web
	
		https://huggingface.co/BumblingOrange/Hiten/resolve/main/Hiten%20girl_anime_8k_wallpaper_4k.ckpt

#### nanachiDB-42imgs-5000steps.ckpt
yiffy-e15 上的 nanachi 42 图像
关键词：nanachiDB cute_furry_girl

- Web

		https://mega.nz/file/xE9gFQYK#f61_2_OvDSOd4VRW3W9EoLpImwCBf1hauUFhW-iNtRw

#### rizaDB-54imgs-4500steps.ckpt
ryza dreambooth 模型（WD 1.3 beta 上的 54 个图像）
关键字：rizaDB Anime_girl（是的，我拼错了名字）
注意：默认可以生成 nsfw 结果

- web

		https://mega.nz/file/dc1EDKCa#NTXIyMRCpFfS8BsW3s-VeT950ApRTzE8_aVciLmx1bM

### bea-14000-pruned-fp16.ckpt [802a681b]
- Bea (Pokemon S/S) NSFW DreamBooth 模型
- 数据集大小：24 个手工挑选、手工裁剪的同人图，50% SFW，50% NSFW
- 步数：14000
- 学习率：6.5e-07
- 触发短语（标识符 + 类别）：sks girl
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。
- 训练数据包含在洪流中。

- Torrent

		magnet:?xt=urn:btih:18c7932594fcf2b67434103d3af422a92ebac625&dn=dreambooth-bea-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.skyts.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce

		magnet:?xt=urn:btih:18c7932594fcf2b67434103d3af422a92ebac625&dn=dreambooth-bea-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.skyts.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce

### 2b-10000-pruned-fp16.ckpt [8cf4478f]
- YoRHa 2B (尼尔) DreamBooth NSFW
- 数据集大小：18 个手工挑选、手工裁剪的同人图，其中大约一半是 NSFW
- 步数：10000
- 学习率：6.5e-07
- 触发短语（标识符 + 类别）：sks girl
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。
- 训练数据包含在洪流中。

- Torrent

		magnet:?xt=urn:btih:14f5414d184f36c1932f310661eb82e14165931b&dn=dreambooth-2b-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.skyts.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce

#### misato-10000-pruned-fp16.ckpt [c51e4c77]
- Misato Katsuragi (Casual Outfit) (新世纪福音战士) DreamBooth NSFW
- 数据集大小：21 个手工挑选、手工裁剪的同人图，其中大约一半是 NSFW
- 步数：10000
- 学习率：6.5e-07（我认为？）
- 基本型号：WD 1.3 epoch 8
- 触发短语（标识符 + 类别）：sks katsuragi_misato
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。
- 包含在洪流中的训练图像。

- Torrent

		magnet:?xt=urn:btih:b4751304bbb04c719233436f92806cf278da74ab&dn=dreambooth-misato-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.skyts.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce

#### hilda-v2-8000-pruned-fp16.ckpt [d2c8eef1]
- 希尔达-v2-8000-修剪-fp16.ckpt [d2c8eef1]
- 希尔达 (Pokemon) DreamBooth NSFW
- 数据集大小：24 个手工挑选、手工裁剪的同人图，其中大约一半是 NSFW
- 步数：8000
- 基本型号：WD 1.3 epoch 8
- 触发短语（标识符 + 类别）：sks hilda_(pokemon)
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。

- Torrent

		magnet:?xt=urn:btih:37c7fc96d630dafc14c3e46526507ca66e760afc&dn=dreambooth-hilda-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce

#### nagatoro-4000-pruned-fp16.ckpt [8c5ff341]
- 长瀞DreamBooth NSFW
- 数据集大小：16 个手工挑选、手工裁剪的同人图，其中大约一半是 NSFW
- 学习率：2.9e-07 (imgs*0.18)
- 步数：4000
- 触发短语（标识符 + 类别）：sks nagatoro_hayase
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。

- Torrent

		magnet:?xt=urn:btih:fc70fc0613601df43ffc7e7921250debf925db7b&dn=dreambooth-nagatoro-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce

#### gawr_gura-4000-pruned-fp16.ckpt [cbd4da60]
- Gawr Gura (Hololive) DreamBooth 型号 NSFW
- 数据集大小：24 个手工挑选、手工裁剪的同人图，其中大约一半是 NSFW
- 学习率：4.3e-07 (imgs*0.18)
- 步数：4000
- 触发短语（标识符 + 类别）：sks gawr_gura
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。

- Torrent
	
		magnet:?xt=urn:btih:9200076f1cc41258a8069a6c44e948f5f6cd515b&dn=dreambooth-gura-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce

#### mori_calliope-4000-pruned-fp16.ckpt [6cb50083]
- Calliope Mori (Hololive) DreamBooth 型号 NSFW
- 数据集大小：14 个手工挑选、手工裁剪的同人图，其中大约一半是 NSFW
- 学习率：2.5e-07 (imgs*0.18)
- 步数：4000
- 触发短语（标识符 + 类别）：sks mori_calliope
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。

- Torrent

		magnet:?xt=urn:btih:969cabc39c8363aeec824f5530bb0749b6452621&dn=dreambooth-calli-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce

#### towa-4000-pruned-fp16.ckpt [d578d3c1]
- Tokoyami Towa (Hololive) DreamBooth 型号 NSFW
- 数据集大小：15 张同人图，其中大约一半是 NSFW
- 学习率：2.7e-07 (imgs*0.18)
- 步数：4000
- 班级：tokoyami_towa
- 所有训练图像都将角色描述为至少 18 岁 - 我不对使用此模型文件的生成结果负责。

- Torrent

		magnet:?xt=urn:btih:207fef192613543d219a861b37e04c138a7a8597&dn=dreambooth-towa-nsfw&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce
				
### Gawr_Gura_450img.ckpt [e7883abf]
- 基于外孚扩散 v1.2
- 自定义 Dreambooth 模型基于 Hololive Vtuber Gawr Gura 的相似性。 数据集是 450 张训练图像和 900 张正则化图像。 训练了 3000 步。
- 要使用该模型，只需在提示中插入名称“Gawr Gura”即可。

Repo: https://huggingface.co/BumblingOrange/GuraLv400

- Web

		https://huggingface.co/BumblingOrange/GuraLv400/resolve/main/Gawr_Gura_450img.ckpt

### Emilia_CustReg_3k.ckpt [2660cf9a]
- 基于外孚扩散 v1.2
- 自定义 Dreambooth 模型基于 Re:Zero 中的 Emilia 的相似性。 数据集是 16 个训练图像和 11 个正则化图像。 训练了 3000 步。
- 要使用该模型，只需在提示中插入名称“Emilia”即可。 使用的类标记是
- EX: "A photo of Emilia white_hair_girl_violet_eyes"

Repo: https://huggingface.co/BumblingOrange/Emilia

- web

		https://huggingface.co/BumblingOrange/Emilia/resolve/main/Emilia_CustReg_3k.ckpt

### Star Fox Krystal
- 在 yiffy18 上训练了 2500 步的 90 张粉丝艺术图像。 规则图像是 4 种常见人类物种的约 200 代低质量世代的均匀混合，具有不同的性别（我希望在即将发布的指南中解释一个新想法）。
- 必须在您的提示符中使用以下文本，否则它什么也不做！
- 触发短语 thatbluefoxfromstarfoxkrystal anthro

		SHA256 13adea314c8653de231c454fa9e911d80ab04640caf30d752802c9b201d4c243

- Web

		https://mega.nz/file/6x4zETbS#E22vQhjjK9CmBQXkzZgNFkIF4-TWGsyE0GRFP7pk-kE
	or

		https://pixeldrain.com/u/EH2mvEux

### StudioGhibli [10c6aa67]
提示是 studio_ghibli_anime_style 风格

Repo https://huggingface.co/IShallRiseAgain/StudioGhibli

- Web

		https://huggingface.co/IShallRiseAgain/StudioGhibli/resolve/main/StudioGhibliV4.ckpt

### Arcane Vi vimod.ckpt [e02601f3]
在提示中使用 sks vi 或 vi。

	SHA256 2db4e932c1d7e3ac4b8c9d94d9b9c98487941f3b6c5617973351f898374d7aae

- Web

		https://huggingface.co/jinofcoolnes/VImodel/resolve/main/vimod.ckpt

#### hyperbreasts: v3
Hyper Breasts Dreambooth 模型 [NSFW] v3

- 1.2k 图像
- 10,000 步
- 学习率1e-6

这个训练的整体图像较少，但质量更高，尺寸更大。 比 v2 更大的结果。
关键词 hyperbreasts

- Torrent

		magnet:?xt=urn:btih:nf5mg3yvhho5rc4euyrxz4s62ejyrp2d&dn=dreambooth-hyper-breasts-v3&xl=2133006880&fc=2

#### hyperpreg: v2 (realism and anime)
关键词 hyperpreg

- Torrent

		magnet:?xt=urn:btih:xeaxmzyhmdxooukwdf3ejjdb2yngjgin&dn=dreambooth-hyper-preg-v2&xl=2133006686&fc=2 

#### hyperpreg: v1 (realism)
hyper_preg_2.7k-lr5e-7-15ks-fp16.ckpt

Hyper Preg Dreambooth 模型 NSFW

- 2.7k 图像
- 15,000 步
- 学习率 5e-7
- 基本型号 sd-v1-4
- 关键词 hyperpreg
- class round belly

- Torrent

		magnet:?xt=urn:btih:wlu5ltx6nf637e3au2zk4r2o2y6oi3ol&dn=dreambooth-hyper-preg&xl=2133006686&fc=2

#### Kurisu
- 栗栖 NSFW
- 触发短语（标识符 + 类别）：sks kurisu
- 步数 - 4000
- 图片 - 16 个，带有一些 NSFW

- Torrent

		magnet:?xt=urn:btih:2380decbde8386095241729e38cb6bbead93c0c2&dn=kurisu_final_pruned.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce

#### Kurisu 视觉小说官方艺术家 (Huke) [eda12c8e]
kurisu_vn_final_4k_pruned.ckpt

- 可以做 NSFW
- 触发短语（标识符 + 类别）：sks kurisu_vn
- 步数 - 4000（尝试了 6k 并且过度拟合）
- 来自 Steins;Gate、Huke 艺术家的 12 张 SFW 图像
- 关于这个模型的快速说明，如果你想生成不同的字符并在 auto 的 ui 中使用“highres fix”，请将去噪强度设置为高以防止由于艺术风格而产生的伪影，最好 >0.85。 根据需要调整 CFG，5-7 是一个好的开始。

- Torrent

		magnet:?xt=urn:btih:04c933900719a5833fbe07581ffa6c5114d3b7b6&dn=kurisu_vn_final_4k_pruned.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce

#### dreambooth_lain_girl.ckpt [e7629bf8]
Prompt token lain girl

- Torrent

		magnet:?xt=urn:btih:8fb4f0fe0ce052b5d6f0d1221a69f04aa2ea406d&dn=dreambooth_lain_girl.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://mega.nz/file/VK0U0ALD#YDfGgOu8rquuR5FbFxmzKD5hzxO1iF0YQafN0ipw-Ck

### Wayne Barlowe [e0c27550]
受训于艺术家韦恩·巴洛 (Wayne Barlowe) 创作的 31 件 SFW 艺术品

- 触发短语：DBWayneBarlowe
- 步数：4000
- 图片：
- 型号：SD_1.5_pruned
- 使用宗教、恶魔导向的提示

		SHA512 30fd8065ff69438c0ef96e063490f6ed4a8c7b41ac8433a36e592f285937f34cc45212d7c44846161a3b5f7977d9700778bf8d566beb653e56babf079f570302

- Torrent

		magnet:?xt=urn:btih:a488fb7b0403988df282b422c3d9383e17bbc646&dn=DBWayneBarlowe_JM.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=https%3a%2f%2ftracker.nanoha.org%3a443%2fannounce&tr=https%3a%2f%2ftracker.lilithraws.org%3a443%2fannounce&tr=https%3a%2f%2ftr.burnabyhighstar.com%3a443%2fannounce
- Web

		https://gofile.io/d/VPm7wu
	or

		https://pixeldrain.com/u/9pPL2ikF

### Blacked POV blowjob [8467f44f]
dreambooth-bkdbj-4000step-pruned-fp16.ckpt

- Blacked POV 口交模型，在来自 blacked.com 的 70 张精选照片上训练了 4000 步，使用 Dreambooth 在 SD 1.4 模型上进行了训练。 提示关键字：bkdbj
- 使用的所有训练图像都将女演员描述为至少 18 岁 - 我不对使用此模型文件或与其合并的模型的任何生成结果负责。 我不会为此模型提供任何支持或进一步更新。 它是一个“one trick pony”模型，因为它只能处理一种 NSFW 场景。 有效的生成大小为 512x512 或高达 512x640。

- Torrent

		magnet:?xt=urn:btih:836422af364efb784e61a049c1257ea1230509b7&dn=dreambooth-bkdbj-4000step-pruned-fp16.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.pomf.se%3a80%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce
- Web

		https://gofile.io/d/aMfZIV

### Zeipher F222 Female Nude (better anatomy)
From: https://ai.zeipher.com/

所有这些都还在训练中，不是最终的

检查他们的网站/不和谐的下载和潜在的更新

- Torrent F222

		magnet:?xt=urn:btih:GR3IGMJDPJPW3B4WRT5B7SAN7CEBHWSZ&dn=f222&tr=http%3A%2F%2Ftracker.openbittorrent.com%2Fannounce
- Web

		https://redirect.ai.zeipher.com/b1t50kc
以前 F111. F99.

### Zeipher BB (Blowjob images)
From: https://ai.zeipher.com/

所有这些都还在训练中，不是最终的

检查他们的网站/不和谐的下载潜在更新。

### LOKEAN_PUPPYSTYLE_POV.ckpt [e02601f3]
- 逼真的异性狗风格图像。
- 出于某种原因，与 Arcane Vi vimod.ckpt 和 lokean Missionary 共享简短的哈希。 可能是由于它的训练方式 - 哈希只是文件开头附近的一小部分。 它们是不同的模型。
- 使用 LOKEAN_PUPPYSTYLE_POV 激活。
- 信息和预览图片 https://rentry.org/kwai

- Web

		https://agencykwai.vip/SD/LOKEAN_PUPPYSTYLE_POV.ckpt

### LOKEAN_MISSIONARY_POV.ckpt [e02601f3]
- 逼真的异性传教士图像。
- 出于某种原因，与 Arcane Vi vimod.ckpt 和 lokean 小狗风格共享短哈希。 可能是由于它的训练方式 - 哈希只是文件开头附近的一小部分。 它们是不同的模型。
- 使用 LOKEAN_MISSIONARY_POV 激活。
- 信息和预览图片 https://rentry.org/kwai

- web

		https://agencykwai.vip/SD/LOKEAN_MISSIONARY_POV.ckpt

### Reverse cowgirl s4629674.ckpt [e285da0d]
逼真的反向女牛仔色情模型
提示语: s4629674

- Torrent

		magnet:?xt=urn:btih:7dd135c05ce2693fbcaf0e6e2f9975dfa871f745&dn=s4629674.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zemoj.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://mega.nz/file/Q4tDRTAJ#GvgsNp_AnCuniY_rpdrerDQtVaGO90dQlyB__wT4dMM

### elden-ring-diffusion [16d77205]
- 这是根据 Elden Ring 的游戏艺术训练的微调 Stable Diffusion。
- 在提示中使用令牌 elden 环样式以获得效果。

repo https://huggingface.co/nitrosocke/elden-ring-diffusion
	
	检查 repo 以获取潜在的更新版本。
- Web

		https://huggingface.co/nitrosocke/elden-ring-diffusion/resolve/main/eldenring-v2-pruned.ckpt

### modern-disney-diffusion [ccf3615f]
这是根据流行动画工作室的截图训练的微调稳定扩散模型。

在提示中使用代币现代迪士尼风格来获得效果。

Repo https://huggingface.co/nitrosocke/mo-di-diffusion

	检查 repo 以获取潜在的更新版本。
- Web

		https://huggingface.co/nitrosocke/mo-di-diffusion/resolve/main/moDi-v1-pruned.ckpt

### Arcane-Diffusion [318a302e]
这是根据电视节目奥术训练的微调稳定扩散模型。

在提示中使用标记奥术样式以获得效果。

Repo https://huggingface.co/nitrosocke/Arcane-Diffusion

	检查 repo 以获取潜在的更新版本。
- Web

		https://huggingface.co/nitrosocke/Arcane-Diffusion/resolve/main/arcane-diffusion-v3.ckpt

### classic-anim-diffusion [be7ddafc]
是根据流行动画工作室的截图训练的微调稳定扩散模型。

在提示中使用令牌经典迪士尼风格以获得效果。

Repo https://huggingface.co/nitrosocke/classic-anim-diffusion

	检查 repo 以获取潜在的更新版本。
- Web

		https://huggingface.co/nitrosocke/classic-anim-diffusion/resolve/main/classicAnim-v1.ckpt

### bukkake [b4a14787]
bukkake_20_training_images_2020_max_training_steps_woman_class_word.ckpt

SD1.4 + Dreambooth

一个很好的测试提示是“颜射女人的肖像，可爱的女孩，性感的，美丽的女孩，照片，尼康 d4，f/2.8，70 毫米，裸照" `portrait of bukkake woman, cute girl, sexy, beautiful girl, photograph, nikon d4, f/2.8, 70mm, topless`

提示：如果您尝试使用 txt2img 来在逼真的面部上实现任何类型的逼真暨，请不要使用面部恢复。 当前可用的两种面部恢复模型（GPFGAN 和 Codeformer）都可以完全去除或破坏面部的精液。 希望另一个 coomer 会提出一个新的面部修复模型，它可以忽略脸上的液体并仍然修复下面的面部

- Torrent

		magnet:?xt=urn:btih:90df9e64cd31b686c858b9d50b54efafd6a96984&dn=bukkake_20_training_images_2020_max_training_steps_woman_class_word.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2fvibe.sleepyinternetfun.xyz%3a1738%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker1.bt.moack.co.kr%3a80%2fannounce&tr=udp%3a%2f%2ftracker.zerobytes.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.theoks.net%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.swateam.org.uk%3a2710%2fannounce&tr=udp%3a%2f%2ftracker.srv00.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.publictracker.xyz%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.monitorit4.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.lelux.fi%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.encrypted-data.xyz%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.army%3a6969%2fannounce
- Web

		https://mega.nz/file/olV0yDrR#-5ShWX92PP6r8wV6jHMHQjOmEGnq82xY6-J048GOf9s
		
### DCAU [2fd6057b]
DC动画宇宙的模型。 目前它只接受过蝙蝠侠动画系列的训练，但计划包括所有节目。

Repo (with samples) https://huggingface.co/IShallRiseAgain/DCAU

	有关潜在的较新版本，请参阅 repo。
提示目前是 Batman_the_animated_series.

- Web

		https://huggingface.co/IShallRiseAgain/DCAU/resolve/main/DCAUV1.ckpt

### Raven anime.ckpt
“动漫”模型 - 一个较新的模型。 为了获得更大的灵活性，它有点训练不足。 它可以创建更复杂的图像，稳定性更高，但动漫风格更强。
标签和类名比“西方”模型需要更多的强调/关注。 试试（RavenRoth 女人：1.2）。 它有点训练不足以获得更大的灵活性 - 可能需要在提示中使用“灰色皮肤”。 （RavenRoth 女人：1.2）灰色皮肤。

标签和类名: RavenRoth woman

- Web

		https://mega.nz/folder/4KlHFQaL#NBqANoiG3E7mz3apn85Q0g

### Raven Western.ckpt
这是我创建的早期 Dreambooth 模型。 它的风格更符合少年泰坦卡通和区域艺术。

我注意到负面提示如何导致模型偏离卡通/区域艺术风格，所以如果你发现负面提示，请小心。

负面提示：'lowres，（坏解剖），坏手，文本，错误，缺少手指，多余数字，更少数字，裁剪，最差质量，jpeg 伪影，>签名，水印，用户名，模糊，艺术家姓名 `'lowres, (bad anatomy), bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, jpeg artifacts, >signature, watermark, username, blurry, artist name`'作品 出色地。

我认为 AI 认为粗线条艺术风格是“低质量的”，而 Raven 则认为是“变形”或“变异”，因为皮肤是灰色的。

尝试降低 CFG 比例（接近 5）以获得更大的灵活性。 您甚至可能想尝试 [RavenRoth Anime_girl]

标签和类名：RavenRoth Anime_girl

标签和类名: RavenRoth anime_girl

- Web

		https://mega.nz/folder/4KlHFQaL#NBqANoiG3E7mz3apn85Q0g

### oshinobu-pruned.ckpt
- 数据集大小：63 个类似于动漫角色设计的同人图（只有 3 或 4 个是 NSFW）
- 步数：9450（~6 epochs）
- 基础模型：Waifu Diffusion v1.3
- 触发短语（标识符 + 类别）：illustration of a oshinobu blonde_loli
- 由于使用了训练数据，生成的图像保持了她典型的圆形脸红（大多数时候）。
- 现在，我想我使用了大量的步骤和图像，但是当我训练这个（一周前）时，我不知道我到底在做什么。

- Torrent

		magnet:?xt=urn:btih:bd00f28f19715632b62eae8660cbffb14d606c20&dn=oshinobu-pruned.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce

### henreader_final_pruned.ckpt [e5803978]
触发短语（标识符 + 类别）：sks henreader_db

- Torrent

		magnet:?xt=urn:btih:2ea2f859da0dd0b48d39f7296bee20434b40e92b&dn=henreader_final_pruned.ckpt&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce

### oshino_shinobu_final_pruned.ckpt
触发短语（标识符 + 类别）: photo of a sks oshino_shinobu_db

- Torrent

		magnet:?xt=urn:btih:ed81724913f0cf899c6eed4a896ff1480b243761&dn=oshino_shinobu_final_pruned.ckpt&tr=udp%3a%2f%2ftracker.moeking.me%3a6969%2fannounce&tr=udp%3a%2f%2fopentracker.i2p.rocks%3a6969%2fannounce&tr=udp%3a%2f%2fopen.demonii.com%3a1337%2fannounce&tr=udp%3a%2f%2fexodus.desync.com%3a6969%2fannounce&tr=udp%3a%2f%2fmovies.zsw.ca%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.dler.org%3a6969%2fannounce&tr=udp%3a%2f%2fipv4.tracker.harry.lu%3a80%2fannounce&tr=https%3a%2f%2fopentracker.i2p.rocks%3a443%2fannounce&tr=udp%3a%2f%2fexplodie.org%3a6969%2fannounce&tr=udp%3a%2f%2fopen.stealth.si%3a80%2fannounce&tr=udp%3a%2f%2fp4p.arenabg.com%3a1337%2fannounce&tr=udp%3a%2f%2ftracker.tiny-vps.com%3a6969%2fannounce&tr=udp%3a%2f%2fpublic.tracker.vraphim.com%3a6969%2fannounce&tr=udp%3a%2f%2f9.rarbg.com%3a2810%2fannounce&tr=udp%3a%2f%2ftracker2.dler.org%3a80%2fannounce&tr=udp%3a%2f%2ftracker.torrent.eu.org%3a451%2fannounce&tr=udp%3a%2f%2ffe.dealclub.de%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=http%3a%2f%2ftracker.openbittorrent.com%3a80%2fannounce&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337%2fannounce
		
		
---------？？-----------
				
### 升级者
#### Lollypop
对于拟人化的人物。

- Torrent

		magnet:?xt=urn:btih:JDZHD4PQVPHJU35C32GRJMSLYSR7YRHS&dn=lollypop.pth&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce

#### Remacri 升频器
对于风景。

- Torrent	
		
		magnet:?xt=urn:btih:TNSKQM7JWWWOTIVEYS4GPSG2L2HFKSI7&dn=4x_foolhardy_Remacri.pth&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce

#### SwinIR
据开发人员介绍，SwinIR 实现了：

在双三次/轻量级/真实世界图像 SR、灰度/彩色图像去噪、灰度/彩色 JPEG 压缩伪影减少方面的最先进性能。

在 [GitHub 上获取 >](https://github.com/JingyunLiang/SwinIR/releases)

### Face Restorers 面部修复师
GFPGAN 旨在开发用于真实世界人脸恢复的实用算法。

[github](https://github.com/TencentARC/GFPGAN/releases)

### Textual Inversion 文本倒置
HuggingFace 发布社区提交的文本反演模型。 我创建了一个单独的页面来预览和下载它们。

[https://cyberes.github.io/stable-diffusion-textual-inversion-models](https://cyberes.github.io/stable-diffusion-textual-inversion-models/)

它每天自动更新两次。
### 模型比较	
- 基础模型对比

	![](./pic/anime-model-comparison.jpg)
- Waifu 双模型对比 EMA vs Pruned

	![](./pic/waifu-diffusion-ema-vs-pruned.png)
		
## Stable Diffusion Textual(文本) Inversion(反转) Embeddings(嵌入)
页面每天自动更新。 最后更新于 2022 年 11 月 4 日星期五。

[HuggingFace 文本反转概念库](https://huggingface.co/sd-concepts-library)的浏览器。 目前 sd-concepts-library 中有 762 个文本反转嵌入。 这些旨在与 AUTOMATIC1111 的 [SD WebUI](https://github.com/AUTOMATIC1111/stable-diffusion-webui)一起使用。

嵌入直接从 HuggingFace 存储库下载。 显示的图像是输入，而不是输出。 想要快速测试概念？ 试试 HuggingFace 上的 [Stable Diffusion Conceptualizer](https://huggingface.co/spaces/sd-concepts-library/stable-diffusion-conceptualizer)。 有关文本反转的[更多信息](https://huggingface.co/docs/diffusers/main/en/training/text_inversion)。

显示的图像是输入，而不是输出。

### 下载列表
[Stable Diffusion Textual Inversion Embeddings](https://cyberes.github.io/stable-diffusion-textual-inversion-models/)

### 特别关注
- anime-background-style
- anime-background-style-v2

### Stable Diffusion 第三方文本反转概念库
每日更新，2022 到 11月4号

- 当前有 1724人维护
- 762个模型

浏览社区教授的对象和样式以 Stable Diffusion 并在您的提示中使用它们！

- [Run Stable Diffusion with all concepts pre-loaded](https://huggingface.co/spaces/sd-concepts-library/stable-diffusion-conceptualizer) 

	运行 Stable Diffusion 直观地浏览文本反转概念库并使用库中所有 100 多个经过训练的概念
- [Training Colab](https://colab.research.google.com/github/huggingface/notebooks/blob/main/diffusers/sd_textual_inversion_training.ipynb) 

	通过 Textual Inversion 仅用 3-5 个示例向其教授新概念，从而个性化模型库 👩‍🏫（在 Colab 中，您可以将它们直接上传到公共图书馆）
- [Inference Colab](https://colab.research.google.com/github/huggingface/notebooks/blob/main/diffusers/stable_conceptualizer_inference.ipynb) 

	使用学习的概念运行 Stable Diffusion，一次一个🖼️（包括那些私有的或不在库中的）

该图书馆经过审核，带有色情、暴力或血腥的内容将被删除。

## Stable Diffusion DreamBooth 型号
页面每天自动更新。 最后更新于 2022 年 11 月 4 日星期五。

[HuggingFace DreamBooth](https://huggingface.co/sd-dreambooth-library) 库的浏览器。 目前 `sd-dreambooth-library` 中有 101 个 DreamBooth 模型。

要将这些与 AUTOMATIC1111 的 [SD WebUI](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 一起使用，您必须对其进行转换。 下载所需模型的存档，然后使用此[脚本](https://gist.github.com/jachiam/8a5c0b607e38fcc585168b90c686eb05)创建 `.cktp` 文件。

确保您已安装 `git-lfs`。 如果没有，请执行 `sudo apt install git-lfs`。 您还需要使用 `git lfs install` 来初始化 LFS。

显示的图像是输入，而不是输出。
	
		
## 参考
- 标准模型
	- [Stable Diffusion Models](https://cyberes.github.io/stable-diffusion-models/)
- Textual Inversion Embeddings
	-  [Stable Diffusion Textual Inversion Embeddings](https://cyberes.github.io/stable-diffusion-textual-inversion-models/)
	- [HuggingFace 文本反转概念库](https://huggingface.co/sd-concepts-library)
- DreamBooth Models
	- [DreamBooth Models](https://cyberes.github.io/stable-diffusion-dreambooth-library/) 