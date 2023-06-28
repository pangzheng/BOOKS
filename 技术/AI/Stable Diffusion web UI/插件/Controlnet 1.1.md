# Controlnet 1.1
- [代码](https://github.com/Mikubill/sd-webui-controlnet/tree/main)
- [说明手册](https://github.com/lllyasviel/ControlNet-v1-1-nightly#model-specification)

## 安装
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

## 下载模型
controlnet 下完插件后，还需要下很多模型来使用，所以需要从代码库查看模型地址
### 查看代码库 url
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

## 配置 Controlnet
### 打开多个 Controlnet 单元
默认初始化只有一个 Controlnet 单元，但很多情况下要使用多个 Controlnet 处理单元配合使用，所以需要打开多个 Controlnet 单元
 
- 设置
- 找到 Controlnet 设置
- 找到 `Multi ControlNet 的最大网络数量`
	- 设置数量，一般设置3 
	
		![](./pic/ControlnetUnit.png)
- 记录保存并重启 UI
- 查看
	
	![](./pic/ControlnetUnit1.png)

## Controlnet 功能讲解 
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
			
## [Controlnet 1.1 + canny 线稿](https://www.bilibili.com/video/BV1Wk4y1L7AK/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
获取画面线条，然后重新上色、材质、光照等，所以效果是与上面的模型不一样，
### 使用场景
比如上面说到的漫画上色、或者人物服装换色
### 配置
- 模型文件
	- control_v11p_sd15_canny.pth
- 预处理器
	- 原图

		![](./pic/controlnet1.1-13.png)
	- canny(*推荐*)

		![](./pic/controlnet1.1-14.png)
	- invert (from white bg & black line)

		生成黑白反色的控制图，暂时没想到什么地方用
		![](./pic/controlnet1.1-15.png)	
- 参数
	
	在 canny 处理器下，可以通过调低高阈值获取画面更多的细节线条，这样来加大对生成图的更多限制
	
	![](./pic/controlnet1.1-16.png)	


## [Controlnet 1.1 + MLSD 直线模型 ](https://www.bilibili.com/video/BV1Wk4y1L7AK/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
获取画面直线条线搞，如果是曲线就会忽略
### 使用场景
可以做一些建筑设计相关，比如室内设计
### 配置
- 模型文件
	- control_v11p_sd15_canny.pth
- 预处理器
	- 原图(用法线的图对比)

		![](./pic/controlnet1.1-9.png)
	- mlsd(*推荐*)

		![](./pic/controlnet1.1-17.png)
	- invert (from white bg & black line)

		生成黑白反色的控制图，暂时没想到什么地方用
		![](./pic/controlnet1.1-18.png)	
- 参数
	
	在 mlsd 处理器下，通过分辨率增高，反而细节变少了，对细节误差较大，所以可能产生更多噪点影响准确度，如下
	
	![](./pic/controlnet1.1-19.png)	
	
	![](./pic/controlnet1.1-20.png)
	
	- MLSD Value Threshold
	
		霍夫变换阈值越小，细节更重
		
		- 0.1

			![](./pic/controlnet1.1-17.png)
		- 0.01 

			![](./pic/controlnet1.1-21.png)
	- MLSD Distance Threshold

		霍夫变换距离阈值，增大密集线条逐渐减少
		
		![](./pic/controlnet1.1-22.png)

## [Controlnet 1.1 + scribble 涂鸦](https://www.bilibili.com/video/BV1Uu411p76u/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
涂鸦有两种模式

- 合成涂鸦式
- 交互式手绘涂鸦

### 使用场景
文化创作的概念设计

### 配置
- 模型文件
	- control_v11p_sd15_scribble.pth
- 预处理器
	- 准备数据
		- 原图
	
			![](./pic/controlnet1.1-23.png)
		- 参数

				DPM2 a Karras	
		- 想要修改的咒语，正向咒语不加其他的
		
				In the cinema 	
		- 合成涂鸦式	
			- scribble_pidinet
				- 控制画面
				
					![](./pic/controlnet1.1-24.png)
				- 合成
					
					![](./pic/controlnet1.1-25.png)	
			- scribble_hed
				- 控制画面
				
					![](./pic/controlnet1.1-26.png)
				- 合成
					
					![](./pic/controlnet1.1-27.png)
			- scribble_xdog
				- 控制画面
				
					![](./pic/controlnet1.1-28.png)
				- 合成
					
					![](./pic/controlnet1.1-29.png)		
		- 手绘涂鸦
			
			就是直接拿笔画
			
			- 手绘涂鸦画板	
			
				![](./pic/controlnet1.1-30.png)	
			- 手绘涂鸦
	
				![](./pic/controlnet1.1-31.png)
			- 合成结果
	
				![](./pic/controlnet1.1-32.png)		  	
		- invert (from white bg & black line)

		生成黑白反色的控制图，暂时没想到什么地方用

## Controlnet 1.1 + Soft Edge 软边缘 
对比线图，这个模型更在乎轮廓线图

性能可以大致记为

- 鲁棒性

		SoftEdge_PIDI_safe > SoftEdge_HED_safe >> SoftEdge_PIDI > SoftEdge_HED
- 最高结果质量

		SoftEdge_HED > SoftEdge_PIDI > SoftEdge_HED_safe > SoftEdge_PIDI_safe
		
### 使用场景
换装？

### 配置
- 模型文件
	- control_v11p_sd15_softedge.pth
- 预处理器
	- 原图(用法线的图对比)

		![](./pic/controlnet1.1-13.png)
	- 咒语
	
			1girl, subway station,  	
	- SoftEdge_HED
		- 控制图

			![](./pic/controlnet1.1-33.png)
		- 结果

			![](./pic/controlnet1.1-34.png)  
	- SoftEdge_PIDI
		- 控制图

			![](./pic/controlnet1.1-35.png)
		- 结果

			![](./pic/controlnet1.1-36.png)  
	- SoftEdge_HED_safe
		- 控制图

			![](./pic/controlnet1.1-37.png)
		- 结果

			![](./pic/controlnet1.1-38.png)  
	- SoftEdge_PIDI_saf
		- 控制图

			![](./pic/controlnet1.1-39.png)
		- 结果

			![](./pic/controlnet1.1-40.png)  

## [Controlnet 1.1 + Segmentation 分割 ](https://www.bilibili.com/video/BV18o4y1b73p/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
可以通过  ADE20K 或 COCO 颜色标注画面物体模块内容，获取画面物体色块分割标注。为了加深理解，加上了分块检模型的 demo [OneFormer](https://huggingface.co/spaces/shi-labs/OneFormer)

![](./pic/controlnet1.1-48.png)

### 使用场景
可以做一些建筑设计相关，比如室内设计
### 配置
- 模型文件
	- control_v11p_sd15_seg.pth
- 预处理器
	- 原图(用法线的图对比)

		![](./pic/controlnet1.1-41.jpeg) 
	- 反推咒语

			no humans, scenery, sky, outdoors, building, window, tree, reflection, night, plant, balcony, day, blue sky, water, lamppost, house, door
	- 换了一个主模型

			xsarchitecturalv3com_v31.ckpt [7bf72c55d3]
	- seg_ofade20k
		- 控制图

			![](./pic/controlnet1.1-42.png)
		- 结果

			![](./pic/controlnet1.1-43.png) 
	- seg_ofcoco
		- 控制图

			![](./pic/controlnet1.1-44.png)
		- 结果

			![](./pic/controlnet1.1-45.png)
	- seg_ufade20k
		- 控制图

			![](./pic/controlnet1.1-46.png)
		- 结果

			![](./pic/controlnet1.1-47.png)

## [ControlNet 1.1 Openpose 姿势 ](https://www.bilibili.com/video/BV1iL411v7ii/?spm_id_from=333.788&vd_source=f6f37263151dcfb42f0a8663116bb22b)
通过姿势捕捉器，捕捉原图的姿势，用于生成新图
### 使用场景
捕捉一些特殊的姿势用给自己的角色，因为只对人物有效，所以场景有限
### 配置
- 模型文件
	- controlv11psd15_openpose.pth
- 预处理器
	- Openpose body
		- 身体姿势 
	- Openpose hand
		- 手姿势 
	- Openpose face
		- 脸姿势 
	- Openpose body + Openpose hand
		- 身体和手姿势 
	- Openpose body + Openpose face
		- 身体和脸姿势 
	- Openpose hand + Openpose face
		- 手和脸姿势 
	- Openpose body + Openpose hand + Openpose face
		- 全部姿势 
- 预处理器  
	- openpose
		- 老版本，只有姿态没有其他补偿
	- openpose_face
		- 姿态+脸补偿
	- openpose_faceonly
		- 只有脸补偿
	- openpose_hand
		- 姿态+手补偿
	- openpose_full
		- 姿态+手脸补偿
		
### 全身姿势
- 姿势图选择
	-  ![](./pic/controlnet1.1-87.png) 
- 文生图
	- Sd
		- 保持默认 
	- controlnet
		- 姿态参考图图片放入 Controlnet 中单张图像中
		- 启动
		- 最佳像素
		- Control Type
			- openpose 
		- 预处理器  
			- openpose_full
		- Control Weight
			- 1 
		- 控制模式
			- Balanced
	- controlnet 参数
		-  ![](./pic/controlnet1.1-88.png) 
- 生成结果
	-  ![](./pic/controlnet1.1-89.png)  
		 
### 脸部捕捉加部分身体
- 姿势图选择
	-  ![](./pic/controlnet1.1-90.png) 
- 其他参数不变
- 生成结果
	-  ![](./pic/controlnet1.1-91.png) 	              

## [ControlNet 1.1 Shuffle 洗牌](https://www.bilibili.com/video/BV1Dh4y1g7a9/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
打散颜色
### 使用场景
这里主要的用途是可以将一张图的颜色融合到另一张图，这里感觉比较适合跨模型, 给图其他颜色风格

 ![](./pic/controlnet1.1-86.png) 
### 配置
- 模型文件
	- controlv11esd15_shuffle.pth
- 预处理器
	- shuffle	
- 咒语

		castle, soldier, armor, 
- 原图
	- ![](./pic/controlnet1.1-81.png)   
- 混色图
	- ![](./pic/controlnet1.1-82.png) 
	
### 混色
 - 参数
	- 转图到图生图 
	- sd
		- 咒语  
			
				castle, soldier, armor, 
		- 种子锁死

				2951692755		
	- controlNet
		- 修改图片放入 Controlnet 中单张图像中
		- 启动
		- 最佳像素
		- Control Type
			- Shuffle
		- Control Weight
			- 0.8 (小于)
		- Starting Control Step(启动)
			- 0.3 
		- 控制模式
			- Balanced
	- sd参数

		 ![](./pic/controlnet1.1-84.png)
	- controlnet 参数

		 ![](./pic/controlnet1.1-85.png)
	- 结果

		![](./pic/controlnet1.1-83.png)

注意这里造型还是有变化，如果不想造型有变化，可以附加多个 controlnet 来控制，比如

- canny
- lineart			

## ControlNet 1.1 Instruct Pix2Pix
这个模型是一个实验模型，可以将画面通过 `make it into X` 语句进行变化，我觉得更偏画面整体效果，比如相同的图像变成春晓秋冬、早晚这样的
### 使用场景
- 场景的天气变化
	-  比如春夏秋冬
- 场景的时间变化
	- 白天晚上

### 配置
- 模型文件
	- control_v11e_sd15_ip2p [c4bb465c]
- 预处理器
	- 空
- 咒语

		stilt house, 
- 原图(先生成一个房子)

	![](./pic/controlnet1.1-71.png) 

### 场景的天气变化
 - 参数
	- 图生图 
	- sd
		- 咒语  
			
				make it into spring
				make it into summer
				make it into autumn
				make it into winter
	- controlNet
		- 修改图片放入 Controlnet 中单张图像中
		- 启动
		- 最佳像素
		- Control Type
			- IP2P
		- Control Weight
			- 1   
		- 控制模式
			- Balanced
	- sd参数

		 ![](./pic/controlnet1.1-73.png)
	- controlnet 参数

		 ![](./pic/controlnet1.1-74.png)
	- 结果

		![](./pic/controlnet1.1-75.png)
		![](./pic/controlnet1.1-76.png)
		![](./pic/controlnet1.1-77.png)   
		![](./pic/controlnet1.1-72.png) 
	 
### 场景的时间变化
- 咒语	
	
		make it into morning
		make it into Noon
		make it into night
- 结果

	![](./pic/controlnet1.1-78.png)
	![](./pic/controlnet1.1-79.png)
	![](./pic/controlnet1.1-80.png)   

可以通过调节 Control Weight 调节效果	

## ControlNet 1.1 Inpaint 修图
修复图中的元素变成自己想要的
### 使用场景
- 消除图像元素
	-  比如去除人物要背景
- 变更画面内容
	- 换一个脸
	- 换衣服什么的
	- 在天空上增加一些鸟或云
- 扩展画面元素
	- 类似于 outpaint     
	
### 配置
- 模型文件
	- control_v11p_sd15_inpaint [ebff9138]
- 预处理器
	- inpaint_global_harmonious
	- inpaint_only
	- inpaint_only+lama 

### [消除图像元素](https://www.bilibili.com/video/BV1oW4y1S7ca/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
- 参数
	- 文生图 
	- sd
		- 咒语写修改部分相关的
			- 这里为空  
		- 其他参数不变
	- controlNet
		- 修改图片放入 Controlnet 中单张图像中
		- 点选画笔将替换遮罩
		- 启动
		- 最佳像素
		- 预处理器
			-  非 inpaint_only+lama 的其他两个模型
		- Control Type
			- Inpaint
		- Control Weight
			- 2 (max)   
		- 控制模式
			- My prompt is more important
	- sd参数

		 ![](./pic/controlnet1.1-61.png)
	- controlnet 参数

		 ![](./pic/controlnet1.1-62.png)
	- xyz 参数  

		 ![](./pic/controlnet1.1-63.png)
- 结果
	- 原图

		 ![](./pic/controlnet1.1-60.png)
	- 蒙版范围

		 ![](./pic/controlnet1.1-64.png)
	- 三个预处理器对比

		![](./pic/controlnet1.1-58.png)

### [变更画面内容](https://www.bilibili.com/video/BV1mu411p739/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
- 参数
	- 文生图 
	- sd
		- 咒语写修改部分相关的
			- 这里填写 `Coffee shop`  
		- 其他参数不变
	- controlNet
		- 修改图片放入 Controlnet 中单张图像中
		- 点选画笔将替换遮罩
		- 启动
		- 最佳像素
		- 预处理器
			-  非 inpaint_only+lama 的其他两个模型
		- Control Type
			- Inpaint
		- Control Weight
			- 2 (max)   
		- 控制模式
			- My prompt is more important
	- sd参数

		 ![](./pic/controlnet1.1-61.png)
	- controlnet 参数

		 ![](./pic/controlnet1.1-62.png)
- 结果
	- 原图

		 ![](./pic/controlnet1.1-60.png)
	- 蒙版范围

		 ![](./pic/controlnet1.1-65.png)
	- 生成结果

		![](./pic/controlnet1.1-66.png)

这图后面再通过 

- hires.fix 生成的到前空间
- 再用图生图中的 loopback + 低比例重绘

具体看视频

### [扩展画面元素](https://www.bilibili.com/video/BV1dm4y1i7mh/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
- 准备
	- 文生图 
		- 使用 AI 进行文生图 
- 扩展
	- 参数
		- 将文生图的图像和咒语都转移到图生图 
		- sd
			- Resize mode
				- 填充
			- 宽高比调节成要扩展的比率
				- 比如原图是 512*512 现在生成宽图 1024 * 512 
			- 其他不变  
		- controlNet
			- 修改图片放入 Controlnet 中单张图像中
			- 启动
			- 最佳像素
			- Control Type
				- Inpaint
			- 预处理器
				- inpaint_only+lama 
			- Control Weight
				- 1  
			- 控制模式
				- ControlNet is more important
			- Resize Mode
				- Resize and Fill  
		- sd参数
	
			 ![](./pic/controlnet1.1-67.png)
		- controlnet 参数
	
			 ![](./pic/controlnet1.1-68.png)
- 结果
	- 原图

		 ![](./pic/controlnet1.1-69.png)
	- 生成结果

		![](./pic/controlnet1.1-70.png)

## ControlNet 1.1 Tile 图像超分辨率
该模型可以以多种方式使用。

- 忽略图像中的细节并生成新的细节。
- 如果局部瓦片语义和提示不匹配，则忽略全局提示，并根据局部上下文引导扩散。

因为该模型可以生成新的细节并忽略现有的图像细节，所以我们可以使用该模型去除不良细节并添加细化的细节。例如，消除由图像大小调整引起的模糊。

### 使用场景
- 补充细节

	 利用 tile 细节补充的方式，帮助将图片的细节补充的更全面，使用文生图模式
- 图像超分辨率

	与补充细节很像，把模糊的的图像修复成高清图像，但不同的是，对于生成出来的图片要尽量还原原图，所以使用的是图生图模式，而非文生图模式，对安防可能有一定作用	 
- 图像放大

	就是将小图片放大，再放大的时候，tile 还会对细节进行修复
	
	![](./pic/controlnet1.1-50.png)
- 细节修复

	修复图中画崩的部分
- 从草图到成品

	利用 Tile 的细节补充，可以将粗糙的草图变成和精致的成品图，比如游戏道具、技能的按钮设定
- 整个画面通过色块调整

	可以通过图生图和色块调节整体画面的色彩，比如从白天变黑天，但我觉得这个是非正规用法，不做介绍，有兴趣查看[视频-6:21](https://www.bilibili.com/video/BV1bo4y1L753/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
		
### 配置
- 模型文件
	- control_v11f1e_sd15_tile [a371b31b]
- 预处理器
	- tile_resample(官方推荐)
	- tile_colorfix+sharp
	- tile_colorfix

### [Tile 放大](https://www.bilibili.com/video/BV1Vm4y1t7Z5/?spm_id_from=333.788&vd_source=f6f37263151dcfb42f0a8663116bb22b)
- sd 参数
	- tag 反推
		- 将想重新放大的图片放到 Tag 反推获取咒语
	- 图生图 
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

### 超分辨率
- sd 参数
	- tag 反推
		- 将想超分的图片放到 Tag 反推获取咒语
		- 选取最重要的词
	- 图生图 
		- 将获取的 tag 放入咒语 
		- 调整相同的图像尺寸比例
		- 缩放模式
			- 选择裁剪
		- 提示词相关性(CFG Sclae)
			- 15-18
		- 重绘幅度(Denoising)
			- 0.6-0.8
		- 随机种子
			- -1 
	- Controlnet 参数
		- 将图片放入 Controlnet 中单张图像中
		- 启动
		- 开启像素完美
		- 打开预览
		- control type
			- tile 
			- 预处理器
				- `tile_resample`
			- 模型
				- `control_v11f1e_sd15_tile [a371b31b]`  
		- control mode

			根据需求调整，我这里选择的是平衡
			
			- balanced
		- 缩放模式
			- 选择裁剪 
- 参数
	- sd
	
		![](./pic/controlnet1.1-51.png)	     
	- Controlnet

		![](./pic/controlnet1.1-52.png)	 
- 对比
	- 原图
	
		![](./pic/controlnet1.1-53.png)	 
	- 修复

		![](./pic/controlnet1.1-54.png)	 

### [细节修复](https://www.bilibili.com/video/BV1bo4y1L753/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)
没有选择 inpaint 是因为，可以保留原图的特征，参考下图

![](./pic/controlnet1.1-55.png)

使用方法与放大与超分类似，其他不在重述，仅描述下不同地方

- 图生图
	- SD 参数
		- 局部重绘
	
			使用图生图中的局部重绘功能，来重绘制图中崩坏部分
		- 重回幅度
			- 选择最大 1 	 		

### [从草图到成品](https://www.bilibili.com/video/BV1AL411a7Yk/?spm_id_from=333.337.search-card.all.click&vd_source=f6f37263151dcfb42f0a8663116bb22b)

![](./pic/controlnet1.1-56.png)

![](./pic/controlnet1.1-57.png)

使用方法与放大与超分类似，其他不在重述，仅描述下不同地方

- 原搞准备

	准备你想用来参考生图的原搞
- 参数
	- 选择文生图 
	- SD 参数
		- 给一个基础的咒语，类似
			- 正向
			
					masterpiece, best quality+物体描述词，比如 blue trrasure chest
			- 反向
			
					(worst quality, low quality:1.4) ,
		- 高清修复
			- 放大算法
				- lanczos
			- 采样
				- 50
			- 重绘
				- 0.3     			
	-  Controlnet	 
		- 将画好的图放进去
		- 启动
		- 开启像素完美
		- 打开预览
		- control type
			- tile 
			- 预处理器
				- `tile_resample`
			- 模型
				- `control_v11f1e_sd15_tile [a371b31b]`  
		- control mode
			- My prompt is more important(题词更重要) 
		- 打开 xyz 脚本
			- 设置 control net 权重,测试12次，0.3-1权重
				- 0.3-1[12]  
		
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

	
# 较好的 Controlnet 整套视频
- [坚韧的橡树](https://www.bilibili.com/video/BV1Dh4y1g7a9/?spm_id_from=333.788&vd_source=f6f37263151dcfb42f0a8663116bb22b)
- [Control Net v1.1 Is Here!](https://www.youtube.com/watch?v=0X7XQ04GUH4)	
