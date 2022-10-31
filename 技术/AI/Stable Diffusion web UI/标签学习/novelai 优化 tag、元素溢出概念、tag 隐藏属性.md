# novelai 优化词条、元素溢出概念、词条隐藏属性
## 顺序和优化逻辑说明部分
### 重要说明
- 顺序的在 AI 理解的重要程度中非常重要，永远是前面的优先级最高，比如
	- 场景在前就是场景为主
	- 人物在前就是人物为主 
- 参数说明
	- 加权部分说明
	
		两个元素的颜色
		
		- 加权 novelai 使用大括号 `{}`,sd 使用小括号 `()`
			- 如果绿色和黑色各占50%，那么绿色加权，绿色就会更多一些可能是60%，但黑色占比维持 50%，这样总体权重就是110%。
		- 减权 novelai 使用中括号 `[]` 
			- 如果要黑色减少权重，还是需要进行减权标注。 
	- 组合词条

		下划线和空格一样的效果	 
- 描述相同元素尽量在一起，比如本例发型描述和眼睛描述
- 颜色定义
	- <font color=red>图像质量</font>
	- <font color= Gold>人物元素</font>
	- <font color= Green>服装饰品</font>
	- <font color= BlueViolet>场景元素</font>
	-  <font color= Blue>画面特效</font>
	-  <font color= DeepPink>画风滤镜</font>

### 词条语句格式解读
- 第一部分  镜头组词条( 要么开头要么结尾，需要测试) 

	包含元素 <font color=red>图像质量</font> <font color= Blue>画面特效</font>  <font color= BlueViolet>场景元素</font> 

	-----------
	
	<font color=red>masterpiece, best quality, masterpiece, best quality, masterpiece, {{best quality}}, highly detailed, ultra-detailed, illustration,</font> <font color= Blue>Depth of field</font> , <font color= BlueViolet> cold </font> , 

	-----------
	 <font color=red>杰作，最佳品质，高度细致，超精细，插图，</font><font color= Blue>景深, </font><font color= BlueViolet>冷</font> 		
- 第二部分 角色组词条
	
	包含元素  <font color= Gold>人物元素</font> <font color= BlueViolet>场景元素</font>    <font color= Green>服装饰品</font>
	
	-----------
	<font color= Gold>{solo}</font>,<font color= BlueViolet>{{{Dark prison}}}, </font> <font color= Gold>{{{1 girl}}}, black short_hair, bob haircut, </font> <font color= Green>{{long_sleeves}}, off_shoulder,</font> <font color= Gold>detailed eyes, {{Hematic red eyes}}, {{tareme}}, {{crazy}}, expressionless,</font> <font color= Green> {{prisoner's gard}}, Handcuffs, </font> <font color= BlueViolet>Chain,</font> <font color= Green>Hammer, miniskirt, {metal jewelry}, {{{heavy metal}}}, Foot cuffs, cross-laced_footwear, choker, </font>	 
	
	-----------
	<font color= Gold>独奏</font> , <font color= BlueViolet>黑暗监狱 </font><font color= Gold> 1个女孩，黑色短发，波波发型，</font> <font color= Green> 长袖子，露肩，</font>  <font color= Gold> 精致入微的眼睛，血色的眼睛，</font>  <font color= Green>重金属，脚镣，绑带靴，项链</font>
	 
- 第三部分 场景组词条

	包含元素 <font color= BlueViolet>场景元素</font>
	
	-----------
	<font color= BlueViolet>{{Monster Energy}}, {Demon Array}, {{{666}}},[Dry ice fog]</font>
	
	-----------
	
	<font color= BlueViolet>魔抓，恶魔法阵，666，干冰雾气</font>

### 正向词条
描述你需要结果的词条

- 通用

	((best quality)), (masterpiece), extremely detailed CG unity 4k wallpaper, 
- 精细人物描述	
	
	((a gril)), solo,  (beautiful detailed eyes), (beautiful detailed white hair),gothic, pocket watch,full-center calotte,witch hat

- 场景描述	
	
	pen词条ram array, night, (full moon), (star), starlight, arcade, fog, crystal, 

### 反向词条
描述不想要结果的词条
- 通用

	lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry, bad feet,
- 精细描述
	
	artist name,more than 2 thighs,nsfw,multiple breasts, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry, bad feet, single color, ((((ugly)))), (((duplicate))), ((morbid)), ((mutilated)), (((tranny))), (((trans))), (((trannsexual))), (hermaphrodite), (out of frame), extra fingers, mutated hands, ((poorly drawn hands)), ((poorly drawn face)), (((mutation))), (((deformed))), ((ugly)), blurry, ((bad anatomy)), (((bad proportions))), ((extra limbs)), (((disfigured))), out of frame, (bad anatomy), gross proportions, (malformed limbs), ((missing arms)), ((missing legs)), (((extra arms))), (((extra legs))), mutated hands,(fused fingers), (too many fingers), (((long neck))), more than one people, more than one light, more than one head, more than one face, more than one body

### 参数
- steps(步数):45
- method(模型):Euler a
- CFG(与描述语言关系):11

### 主体描写说明顺序
- 人物动作，描写轮廓
- 人物性别和数量，描写主体
- 精致五官细节，再写细节
- 装饰，衣服等，描写非主体细节

## 画面大小说明
- 画面小，词条小，否则画面无法承载这么多描述，
- 画面大，就需要更多词条，否则就会溢出。

## 词条的一些规则
- 比如两个冲突词条，只会生成1个

## 溢出（污染）
整体溢出解决方法是2个

- 调小图大小
- 增加更多词条，最好是背景词条

### 溢出描述
- 颜色污染
	- 问题 
	
		各部分串色，如头发和衣服串色
	- 解决

		增加相同颜色的容器来接住溢出颜色，比如红色增加红旗
- 元素污染	
	- 问题
		
		图像尺寸大，生成出不想要的元素，比如1个女孩生成出2个，比如多个肢体，这是因为画面有空白，和权重无关
	- 解决

		这个就需要描述更多元素，使用上空白画面，所以多出来的，用复合场景画面的人来标称，比如怪物和人偶，不想增加容器，就要见小画面尺寸
		
## 词条隐藏属性
因为库的样本训练，所以词条默认带其他描述属性如下

- 黑杰克，自带扑克牌属性
- 修女，自带头巾属性
- 人偶, 镜头属性：人物居中，正坐，仰视


## 参考
[【novelai】进阶教程：优化词条串、元素溢出概念、词条的隐藏属性（录播）](https://www.bilibili.com/video/BV1Ae4y177c3/?share_source=copy_web&vd_source=37c252c9599c22f77b7b782cf7c8d5c5)
		 	

		 