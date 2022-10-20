# Stable Diffusion web UI
基于 Gradio 库的浏览器界面，用于稳定扩散。

![](./pic/txt2img_Screenshot.png)
检查[自定义脚本](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Custom-Scripts) wiki 页面以获取用户开发的额外脚本。
## 功能
[带有图像的详细功能展示](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Features)：

- 原始 txt2img 和 img2img 模式
- 一键安装并运行脚本（但还是要安装python和git）
- 外涂
- 修补
- 提示矩阵
- 稳定扩散高档
- Attention，指定模型应该更加关注的文本部分
	- 穿 ((tuxedo)) 的男人 - 会更注意燕尾服
	- 穿西装的男人 (tuxedo:1.21) - 替代语法
	- 选择文本并按 ctrl+up 或 ctrl+down 自动调整对所选文本的关注（代码由匿名用户提供）
- Loopback，多次运行img2img处理
- X/Y 图，一种绘制具有不同参数的图像的二维图的方法
- 文本倒置
	- 有尽可能多的嵌入，并为它们使用任何你喜欢的名称
	- 使用多个嵌入，每个标记具有不同数量的向量
	- 适用于半精度浮点数
- 附加选项卡：
	- GFPGAN，修复人脸的神经网络
	- CodeFormer，人脸恢复工具，可替代 GFPGAN
	- RealESRGAN，神经网络升频器
	- ESRGAN，具有大量第三方模型的神经网络升级器
	- SwinIR 和 Swin2SR（[见此处](https://github.com/AUTOMATIC1111/stable-diffusion-webui/pull/2092)），神经网络升频器
	- LDSR，潜扩散超分辨率提升
- 调整纵横比选项
- 采样方法选择
	- 调整采样器 eta 值（噪声乘数）
	- 更高级的噪音设置选项
- 随时中断处理
- 4GB 显卡支持（也有 2GB 工作的报告）
- 批次的正确种子
- 提示长度验证
	- 在您键入时获取令牌中的提示长度
	- 如果某些文本被截断，生成后会收到警告
- 生成参数
	- 您用于生成图像的参数与该图像一起保存
	- 在 PNG 块中用于 PNG，在 EXIF 中用于 JPEG
	- 可以将图像拖动到PNG信息选项卡以恢复生成参数并自动将它们复制到UI中
	- 可以在设置中禁用
- 设置页面
- 从 UI 运行任意 python 代码（必须使用 --allow-code 运行才能启用）
- 大多数 UI 元素的鼠标悬停提示
- 可以通过文本配置更改 UI 元素的默认值/混合/最大值/步长值
- 随机艺术家按钮
- 平铺支持，用于创建可以像纹理一样平铺的图像的复选框
- 进度条和实时图像生成预览
- 否定提示，一个额外的文本字段，允许您列出您不想在生成的图像中看到的内容
- 样式，一种保存部分提示并稍后通过下拉轻松应用它们的方法
- 变化，一种生成相同图像但差异很小的方法
- 种子大小调整，一种生成相同图像但分辨率略有不同的方法
- CLIP 询问器，一个尝试从图像中猜测提示的按钮
- 提示编辑，改变提示中代的方法，说开始制作西瓜，中途切换到动漫女孩
- Batch Processing，使用 img2img 处理一组文件
- Img2img 替代品
- Highres Fix，一个方便的选项，可以一键生成高分辨率图片而不会出现通常的失真
- 即时重新加载检查点
- Checkpoint Merger，一个允许您将两个检查点合并为一个的选项卡
- 具有来自社区的许多扩展的[自定义脚本](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Custom-Scripts)
- [Composable-Diffusion](https://energy-based-model.github.io/Compositional-Visual-Generation-with-Composable-Diffusion-Models/)，一种同时使用多个提示的方法
	- 使用大写 AND 分隔提示
	- 还支持提示的权重：`a cat :1.2 AND a dog AND a penguin :2.2`
- 提示没有令牌限制（原始稳定扩散最多可让您使用 75 个令牌）
- DeepDanbooru 集成，为动画提示创建 danbooru 样式标签（将 --deepdanbooru 添加到命令行参数）
- [xformers](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Xformers)，选择卡的主要速度提高：（将 --xformers 添加到命令行参数）

## 安装和运行
确保满足所需的[依赖](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Dependencies)项并遵循适用于 [NVidia](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Install-and-Run-on-NVidia-GPUs)（推荐）和 [AMD](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Install-and-Run-on-AMD-GPUs) GPU 的说明。

或者使用 Google Colab：

- [Colab](https://colab.research.google.com/drive/1kw3egmSn-KgWsikYvOMjJkVDsPLjEMzl)，由 Akaibu 维护
- [Colab](https://colab.research.google.com/drive/1Iy-xW9t1-OQWhb0hNxueGij8phCyluOh)，由我原创，已过时

### 在 Windows 上自动安装
- 安装 [Python 3.10.6](https://www.python.org/downloads/windows/)，选中“将 Python 添加到 PATH”
- 安装 [git](https://git-scm.com/download/win)。
- 下载 stable-diffusion-webui 存储库，例如通过运行 `git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git`。
- 将 `model.ckpt` 放在 `models` 目录中（请参阅[依赖](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Dependencies)项了解获取它的位置）。
- （可选）将 `GFPGANv1.4.pth` 与 `webui.py` 放在基本目录中（请参阅[依赖](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Dependencies)项了解获取它的位置）。
- 以普通非管理员用户身份从 Windows 资源管理器运行 `webui-user.bat`

### Linux 上的自动安装
- 安装依赖项

		# Debian-based:
		sudo apt install wget git python3 python3-venv
		# Red Hat-based:
		sudo dnf install wget git python3
		# Arch-based:
		sudo pacman -S wget git python3
- 要在 `/home/$(whoami)/stable-diffusion-webui/` 中安装，请运行

		bash <(wget -qO- https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh)
- 在 Apple Silicon 上安装

	在[这里](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Installation-on-Apple-Silicon)找到说明。原文:
	
	虽然 Web UI 运行良好，但在 Apple Silicon 上运行此分支时仍然存在某些问题。 唯一有效的 2 个采样器（在撰写本文时）是 Euler 和 DPM2 - 所有其他采样器都会导致黑屏。 升级工作，但仅使用真实 ESRGAN 模型。

	人们在完成安装时也遇到了问题，因为他们以前安装了 Python 或 Miniconda。 这会导致脚本对安装环境和所有相关文件的位置感到困惑。

	如果是这种情况，请遵循本指南。
	
	- 自动安装

		首先，您需要使用 [Homebrew](https://brew.sh/) 安装所需的依赖项。

			brew install cmake protobuf rust python git wget

		该脚本可以从[这里](https://github.com/dylancl/stable-diffusion-webui-mps/blob/master/setup_mac.sh)下载，或者按照下面的说明进行操作。

		- 打开终端.app
		- 运行以下命令

				$ cd ~/Documents/
				$ curl https://raw.githubusercontent.com/dylancl/stable-diffusion-webui-mps/master/setup_mac.sh -o setup_mac.sh
				$ chmod +x setup_mac.sh
				$ ./setup_mac.sh
		- 按照终端窗口中的说明进行操作。
	- 用法

		安装后，您现在将在 `stable-diffusion-webui` 目录中找到 `run_webui_mac.sh`。 运行此脚本以使用 `./run_webui_mac.sh` 启动 Web UI。 此脚本自动激活 conda 环境，从存储库中提取最新更改，并启动 Web UI。 退出时，conda 环境被停用	
	
	
	
## 相关
- Stable Diffusion - https://github.com/CompVis/stable-diffusion, https://github.com/CompVis/taming-transformers
- k-diffusion - https://github.com/crowsonkb/k-diffusion.git
- GFPGAN - https://github.com/TencentARC/GFPGAN.git
- CodeFormer - https://github.com/sczhou/CodeFormer
- ESRGAN - https://github.com/xinntao/ESRGAN
- SwinIR - https://github.com/JingyunLiang/SwinIR
- Swin2SR - https://github.com/mv-lab/swin2sr
- LDSR - https://github.com/Hafiidz/latent-diffusion
- Ideas for optimizations - https://github.com/basujindal/stable-diffusion
- Doggettx - Cross Attention layer optimization - https://github.com/Doggettx/stable-diffusion, original idea for prompt editing.
- InvokeAI, lstein - Cross Attention layer optimization - https://github.com/invoke-ai/InvokeAI (originally http://github.com/lstein/stable-diffusion)
- Rinon Gal - Textual Inversion - https://github.com/rinongal/textual_inversion (we're not using his code, but we are using his ideas).
- Idea for SD upscale - https://github.com/jquesnelle/txt2imghd
- Noise generation for outpainting mk2 - https://github.com/parlance-zz/g-diffuser-bot
- CLIP interrogator idea and borrowing some code - https://github.com/pharmapsychotic/clip-interrogator
- Idea for Composable Diffusion - https://github.com/energy-based-model/Compositional-Visual-Generation-with-Composable
- Diffusion-Models-PyTorch
- xformers - https://github.com/facebookresearch/xformers
- DeepDanbooru - interrogator for anime diffusers https://github.com/KichangKim/DeepDanbooru

 