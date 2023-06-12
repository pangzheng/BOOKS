## Controlnet 1.1
- [代码](https://github.com/Mikubill/sd-webui-controlnet/tree/main)
- [说明手册](https://github.com/lllyasviel/ControlNet-v1-1-nightly#model-specification)

### 安装
![](./pic/Controlnet.png)

- 安装插件	
	- 跳转 Extensions 插件安装界面
	- 选择 Available 可用
	- search 搜索 controlnet
	- 找寻插件
	- 点击 install
	- 提示安装完毕
		
			Use Installed tab to restart.	
	- 点击 Reload UI

### 下载模型
controlnet 下完插件后，还需要下很多模型来使用，所以需要从代码库查看模型地址
#### 查看代码库 url
- 扩展
- 已安装
- 查询 controlnet 的网站，如下

	 ![](./pic/Controlnet1.png)
- 进入网址读取 Readme

		https://github.com/Mikubill/sd-webui-controlnet/blob/main/README.md
- 查询模型

		https://huggingface.co/lllyasviel/ControlNet-v1-1/tree/main
- 点击进入模型库，下载所有 pth 后缀文件
	- 放入

			/home/pangzheng/sd/stable-diffusion-webui\extensions\sd-webui-controlnet\models

		 ![](./pic/Controlnet2.png)
		
			注意这里下载的是 controlnet 1.1 版本模型，如果新版本需要按照说明下载 
	- 下载 1.1 的 14 个模型

		标准 ControlNet 命名规则 (SCNNRs) 来命名所有模型
		
		![](./pic/Controlnet3.png)
		
			control_v11e_sd15_ip2p.pth
			control_v11e_sd15_shuffle.pth
			control_v11f1e_sd15_tile.pth
			control_v11f1p_sd15_depth.pth
			control_v11p_sd15_canny.pth
			control_v11p_sd15_inpaint.pth
			control_v11p_sd15_lineart.pth
			control_v11p_sd15_mlsd.pth
			control_v11p_sd15_normalbae.pth
			control_v11p_sd15_openpose.pth
			control_v11p_sd15_scribble.pth
			control_v11p_sd15_seg.pth
			control_v11p_sd15_softedge.pth
			control_v11p_sd15s2_lineart_anime.pth
		
		- control

			插件名字
		-  v11

			模型版本
		- p

			可生产
		- sd15

			原生模型1.5
		- sd15s2

			除了 1.5 模型，还需要带 `clip skip 2` 参数
		- ip2p..等

			下一个就是具体的控制方法	
		- pth

			模型格式

### 配置 Controlnet
#### 打开多个 Controlnet 单元
默认初始化只有一个 Controlnet 单元，但很多情况下要使用多个 Controlnet 处理单元配合使用，所以需要打开多个 Controlnet 单元
 
- 设置
- 找到 Controlnet 设置
- 找到 `Multi ControlNet 的最大网络数量`
	- 设置数量，一般设置3 
	
		![](./pic/ControlnetUnit.png)
- 记录保存并重启 UI
- 查看
	
	![](./pic/ControlnetUnit1.png)
		 
## [Controlnet 1.1 面部识别制作表情包](https://www.bilibili.com/video/BV1nM4y1h78e/?spm_id_from=333.788&vd_source=f6f37263151dcfb42f0a8663116bb22b)
- 下载表情图片
-  将表情图片放入 Controlnet 的单张图像中
-  下面的选项选择允许预览图 `Allow Preview`
-  预处理器
	- openpose_face
- 模型
	- contorl_v11_sd15_openpose
- 点击中间爆炸，可以在预览图生成表情样子 

	![](./pic/Controlnet-pose.png	)
- 点击启用

	![](./pic/Controlnet-pose1.png)
	
	![](./pic/Controlnet-pose2.png)

### 注意
- 使用的时候慢
	- 看日志，是在下载模型 
- 使用的时候可能会出错
	- `ValueError: too many values to unpack (expected 3)`
	- 解决办法，重启 

## [Controlnet 1.1 线稿上色](https://www.bilibili.com/video/BV1EX4y1172X/?spm_id_from=333.788&vd_source=f6f37263151dcfb42f0a8663116bb22b)
### 上色基础准备
- sd 参数
	- 将想重新上色的图片放到 Tag 反推获取咒语
	- 删除咒语中所有的颜色提示，放入咒语
	- 调整生成的图像大小和比例与上色图相同
- Controlnet 参数
	- 将想重新上色的图片放入 Controlnet 中单张图像中
	- 启动
	- 预览
	- 控制模式
		- 平衡
		
### Canny 上色
- Controlnet 参数
	- 预处理器
		- canny
	- 模型
		- `control_v11p_sd15_canny [d14c016b]`
- 运行预览

	![](./pic/canny.png)
- 点生成
	
	![](./pic/canny1.png)
	
### Invert + lineart 上色
- 预处理器
	- invert
- 模型
	- `control_v11p_sd15_lineart [43d4be0d]`
- 运行预览

	![](./pic/invert1.png)
- 点生成
	
	![](./pic/invert.png)

### Invert + lineart_anime 上色
- 预处理器
	- invert
- 模型
	- `control_v11p_sd15s2_lineart_anime [3825e83e]`
- 运行预览

	![](./pic/invert2.png)
- 点生成
	
	![](./pic/invert3.png)

### lineart_anime + lineart 上色
- 预处理器
	- lineart_anime
- 模型
	- `control_v11p_sd15_lineart [43d4be0d]`
- 运行预览

	![](./pic/lineart.png)
- 点生成
	
	![](./pic/lineart1.png)
		
###  lineart_anime + lineart_anime 上色
- 预处理器
	-  lineart_anime
- 模型
	- `control_v11p_sd15s2_lineart_anime [3825e83e]`
- 运行预览

	![](./pic/lineart2.png)
- 点生成
	
	![](./pic/lineart3.png)

### scribble_pidinet + scribble 上色
- 预处理器
	-  scribble_pidinet
- 模型
	- `control_v11p_sd15_scribble [d4ba51ff]`
- Preprocessor Resolution 增加精度，先图否则糊了
	- 1500  
- 运行预览

	![](./pic/scribble.png)
- 点生成
	
	![](./pic/scribble1.png)

### 预处理器与模型的关系
- invert 
	- 使用特点
		- 强调线稿
		- 白色黑底反色官方推荐
		- 1 处理器 vs 2 模型
	- 可使用模型 
		- `control_v11p_sd15_lineart [43d4be0d]`
			- 特点
				- 有些图会崩
				- 与线图几乎一致，不丢失细节   
		- `control_v11p_sd15s2_lineart_anime [3825e83e]` 
			- 特点
				- 强调线稿
				- 官方推荐黑白线稿 ****
				- 保留线稿赛璐璐风格黑色块保留原始深度
				- 与线图几乎一致，不丢失细节
- lineart

	传说其他几个预处理器差不多，待测试
	
	- lineart_anime
		- 使用特点
			- 1 处理器 vs 2 模型
		- 可使用模型 
			- `control_v11p_sd15_lineart [43d4be0d]`
				- 特点 
					- 去掉黑色线条颜色的色彩，替换成符合模型的色彩
					- 想弱化线条感选择 ****
 			- `control_v11p_sd15s2_lineart_anime [3825e83e]`
				- 特点同上
- canny
	- 使用特点
		- 最早模型
		- 边缘检测
		- 1 处理器 vs 2 模型
		- 不丢失细节
	- 可使用模型 
		-  `control_v11p_sd15_canny [d14c016b] `
- scribble
	
	涂鸦传说其他几个预处理器差不多，待测试
	
	- scribble_pidinet
		- 使用特点
			- 1 处理器 vs 1 模型
			- 想弱化线稿，让 AI 更多自由发挥 ****
			- 颜色更好细节更好
		- 可使用模型 
			- `control_v11p_sd15_scribble [d4ba51ff]`
				- 特点 
					- 需要将 `Preprocessor Resolution` 调节到 1500，才能产生可用线稿
					
- 自由模式

	如果增加自由模式，所有模式都可以得到更加开放的结果

	![](./pic/freemodel.png)				
        
## [Controlnet 1.1 + Tile 放大](https://www.bilibili.com/video/BV1Vm4y1t7Z5/?spm_id_from=333.788&vd_source=f6f37263151dcfb42f0a8663116bb22b)
- sd 参数
	- tag 反推
		- 将想重新放大的图片放到 Tag 反推获取咒语
	- 文生图 
		- 将获取 tag 放入咒语 
		- 调整相同的图像尺寸比例，如测试使用的是 1024*512
		- 点击高清修复
	- Controlnet 参数
		- 要放大的图片放入 Controlnet 中单张图像中
		- 启动
		- 预处理器
			- none
		- 模型
			- `control_v11f1e_sd15_tile [a371b31b]`
		- 控制模式
			- 3种任意选择下，测试自己想要的，推荐 ControlNet is more important
- 参数
	- sd
	
		![](./pic/controlnet_tile.png)
	- Controlnet

		![](./pic/controlnet_tile1.png)	
- 对比
	- 原图
	
		![](./pic/ultimate3.png) 
	- 模式
		- 平衡

			![](./pic/controlnet_tile2.png)
		- 重提示词

			![](./pic/controlnet_tile3.png)
		- 重 Controlnet

			![](./pic/controlnet_tile4.png)
			
			- 高清修复
				- 重绘幅度
					- 0.5
			
			 ![](./pic/controlnet_tile5.png)

当然还可以用多个 controlnet 单元来调节，但是我没调节出适合的
		  	
## [Controlnet 1.1 + Depth 景深](https://www.bilibili.com/video/BV1yP411X78e/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
获取图片的景深、画面主体物体轮廓、动作，生成，实现大轮廓的控制，效果稳定

![](./pic/controlnet1.1-2.png)

### 使用场景
比如宣传画是欧洲做的，要做亚洲市场的时候

![](./pic/controlnet1.1-1.png)

### 配置
- 模型文件
	- control v11p_sd15 depth.pth
- 预处理器
	- 原图

		![](./pic/controlnet1.1-3.png)
	- 前2个深度细节多一点 
		- depth_leres(LeReS深度图估算)

			![](./pic/controlnet1.1-4.png)
		- depth leres++(LeReS深度图估算++)

			![](./pic/controlnet1.1-5.png)
	- 后两个仅有轮廓
		- depth_midas(MiDaS深度图估算)

			![](./pic/controlnet1.1-6.png)
		- depth zoe(ZoE深度图估算)

			![](./pic/controlnet1.1-7.png)
 
## [Controlnet 1.1 + normal 法线 ](https://www.bilibili.com/video/BV1Mk4y1p7h1/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
获取画面主体轮廓，以及光影信息加以控制，效果稳定
### 使用场景
比如锁定房屋结构进行装修配置，比如 post 模型不好获取的姿势

![](./pic/controlnet1.1-8.png)

### 配置
- 模型文件
	- control_v11p_sd15_normalbae.pth
- 预处理器
	- 原图

		![](./pic/controlnet1.1-9.png)
	- normal_bae(*推荐*)

		![](./pic/controlnet1.1-10.png)
	- normal_midas

		老板本，获取法线是真，完全无法使用
- 参数
	- 分辨率
	
		分辨率加高有助于获取图片更多信息，底就获取少信息
		
		- 500

			![](./pic/controlnet1.1-10.png)
		- 1000 			

			![](./pic/controlnet1.1-11.png)
		- 最终结果

			![](./pic/controlnet1.1-12.png)
			
			

## 错误处理
### 下载错误
- 报错
			
		.....
		    result, is_image = preprocessor(
		  File "/data/home/pangzheng/stable-diffusion-webui/extensions/sd-webui-controlnet/scripts/utils.py", line 76, in decorated_func
		    return cached_func(*args, **kwargs)
		  File "/data/home/pangzheng/stable-diffusion-webui/extensions/sd-webui-controlnet/scripts/utils.py", line 64, in cached_func
		    return func(*args, **kwargs)
		  File "/data/home/pangzheng/stable-diffusion-webui/extensions/sd-webui-controlnet/scripts/global_state.py", line 35, in unified_preprocessor
		    return preprocessor_modules[preprocessor_name](*args, **kwargs)
		  File "/data/home/pangzheng/stable-diffusion-webui/extensions/sd-webui-controlnet/scripts/processor.py", line 211, in leres
		    result = model_leres(img, thr_a, thr_b, boost=boost)
		  File "/data/home/pangzheng/stable-diffusion-webui/extensions/sd-webui-controlnet/annotator/leres/__init__.py", line 45, in apply_leres
		    load_file_from_url(remote_model_path_leres, model_dir=base_model_path)
		  File "/data/home/pangzheng/stable-diffusion-webui/venv/lib/python3.10/site-packages/basicsr/utils/download_util.py", line 98, in load_file_from_url
		    download_url_to_file(url, cached_file, hash_prefix=None, progress=progress)
		  File "/data/home/pangzheng/stable-diffusion-webui/venv/lib/python3.10/site-packages/torch/hub.py", line 611, in download_url_to_file
		    u = urlopen(req)
		  File "/usr/lib/python3.10/urllib/request.py", line 216, in urlopen
		    return opener.open(url, data, timeout)
		  File "/usr/lib/python3.10/urllib/request.py", line 525, in open
		    response = meth(req, response)
		  File "/usr/lib/python3.10/urllib/request.py", line 634, in http_response
		    response = self.parent.error(
		  File "/usr/lib/python3.10/urllib/request.py", line 557, in error
		    result = self._call_chain(*args)
		  File "/usr/lib/python3.10/urllib/request.py", line 496, in _call_chain
		    result = func(*args)
		  File "/usr/lib/python3.10/urllib/request.py", line 749, in http_error_302
		    return self.parent.open(new, timeout=req.timeout)
		  File "/usr/lib/python3.10/urllib/request.py", line 519, in open
		    response = self._open(req, data)
		  File "/usr/lib/python3.10/urllib/request.py", line 536, in _open
		    result = self._call_chain(self.handle_open, protocol, protocol +
		  File "/usr/lib/python3.10/urllib/request.py", line 496, in _call_chain
		    result = func(*args)
		  File "/usr/lib/python3.10/urllib/request.py", line 1391, in https_open
		    return self.do_open(http.client.HTTPSConnection, req,
		  File "/usr/lib/python3.10/urllib/request.py", line 1351, in do_open
		    raise URLError(err)
		urllib.error.URLError: <urlopen error [Errno 104] Connection reset by peer>
- 分析

	huggingface 没有授权
- 解决方案
	- 授权 huggingface 
	- 手动下载
	- 重启服务
