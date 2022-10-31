# stable-diffusion-webui 文本倒置(训练)
## 什么是文本反转？
文本反转允许您在自己的图片上训练一小部分神经网络，并在生成新图片时使用结果。在这种情况下，嵌入是您训练的神经网络的一小部分的名称。

训练的结果是一个 .pt 或 .bin 文件（前者是原作者使用的格式，后者是扩散器库使用的格式），其中嵌入。

有关什么是文本反转的更多详细信息，请参见原始站点：[https://textual-inversion.github.io/](https://textual-inversion.github.io/)
## 使用预训练的嵌入
将嵌入放入 `embeddings` 嵌入目录并在提示中使用其文件名。您不必重新启动程序即可使其正常工作。

例如，这是我在 WD1.2 模型上训练的 [Usada Pekora](https://drive.google.com/file/d/1MDSmzSbzkIcw5_aw_i79xfO3CRWQDl-8/view?usp=sharing) 的嵌入，在 53 张图片（119 张增强）上进行了 19500 步，每个标记设置有 8 个向量。

它生成的图片：

![](./pic/train.png)

	portrait of usada pekora
	Steps: 20, Sampler: Euler a, CFG scale: 7, Seed: 4077357776, Size: 512x512, Model hash: 45dee52b
您可以在一个提示中组合多个嵌入
![](./pic/train1.png)	

	portrait of usada pekora, mignon
	Steps: 20, Sampler: Euler a, CFG scale: 7, Seed: 4077357776, Size: 512x512, Model hash: 45dee52b
要非常小心您使用的嵌入模型：它们与您在训练期间使用的模型配合得很好，而在不同的模型上则不太好。 例如，这里是上面的嵌入和 vanilla 1.4 稳定扩散模型：
![](./pic/train2.png)

	portrait of usada pekora
	Steps: 20, Sampler: Euler a, CFG scale: 7, Seed: 4077357776, Size: 512x512, Model hash: 7460a6fa

## 训练嵌入
### 文本反转选项卡
在用户界面中训练嵌入的实验性支持。

- 创建一个新的空嵌入，选择包含图像的目录，在其上训练嵌入
- 该功能非常原始，使用风险自负
- 经过几万步后，我能够重现我在其他存储库中将动漫艺术家训练为风格的结果
- 适用于半精度浮点数，但需要进行实验以查看结果是否一样好
- 如果你有足够的内存，使用 `--no-half --precision full` 运行更安全
- 用于自动运行图像预处理的 UI 部分
- 您可以中断和恢复训练而不会丢失任何数据（AdamW 优化参数除外，但似乎现有的存储库都没有保存这些，所以一般认为它们并不重要）
- 不支持批量大小或梯度累积
- 应该不可能使用 `--lowvram` 和 `--medvram` 标志运行它

### 参数说明
#### 创建嵌入
- `name`

	创建的嵌入的文件名。在提及嵌入时，您还将在提示中使用此文本。
- `Initialization text`

	您创建的嵌入最初将填充此文本的向量。如果您使用“tree”作为初始化文本创建一个名为“zzzz1234”的向量嵌入，并在没有训练的情况下在提示中使用它，那么提示“a zzzz1234 by monet”将产生与“a tree by monet”相同的图片。

- `Number of vectors per token`(每个 token 的向量数)

	嵌入的大小。这个值越大，关于嵌入的主题信息就越多，但它会从你的提示允许中带走更多的单词。在稳定扩散的情况下，提示中的 token 限制为 75 个。如果您在提示中使用包含 16 个向量的嵌入，则将为您留出 75 - 16 = 59 的空间。另外根据我的经验，向量的数量越多，获得好的结果所需的图片就越多。

#### 预处理
从一个目录中获取图像，处理它们以准备好进行文本反转，并将结果写入另一个目录。这是一个方便的功能，如果您愿意，您可以自己预处理图片。

- `Source directory`：有图片的目录
- `Destination directory`：将写入结果的目录
- `Create flipped copies`：对于每个图像，还要写入其镜像副本
- `Split oversized images into two`：如果图像太高或太宽，调整它的大小以使短边匹配所需的分辨率，并从中创建两张可能相交的图片。
- `Use BLIP caption as filename`：使用来自询问器的 BLIP 模型为文件名添加标题。

#### 训练嵌入
- `Embedding`

	从此下拉列表中选择要训练的嵌入。
- `Learning rate`

	训练应该多快。将此参数设置为高值的危险在于，如果设置得太高，可能会破坏嵌入。如果您在训练信息文本框中看到 Loss: nan，则表示您失败并且嵌入已失效。使用默认值，这不应该发生。可以使用以下语法在此设置中指定多个学习率： 0.005:100, 1e-3:1000, 1e-5 - 前 100 步的 lr 为 0.005，然后 1e-3 直到 1000 步，然后1e-5 直到结束。
- `Dataset directory`

	包含用于训练的图像的目录。它们都必须是方形的。
- `Log directory`

	样本图像和部分训练的嵌入副本将写入此目录。
- `Prompt template file`

	带有提示的文本文件，每行一个，用于训练模型。请参阅目录 `textual_inversion_templates` 中的文件，了解您可以使用这些文件做什么。训练样式时使用 `style.txt`，训练对象嵌入时使用 ` subject.txt`。文件中可以使用以下标签：

	- `[name]`：嵌入的名称
	- `[filewords]`：数据集中图像文件名中的单词。有关更多信息，请参见下文。
- `Max steps`

	完成这么多步后，训练将停止。一个步骤是向模型显示一张图片（或一批图片，但目前不支持批量）并用于改进嵌入。如果您中断训练并在以后恢复训练，则会保留步数。
- `Save images with embedding in PNG chunks`

	每次生成图像时，它都会与最近记录的嵌入相结合，并以既可以作为图像共享的格式保存到 image_embeddings，又可以放入嵌入文件夹并加载。
- `Preview prompt`

	如果不为空，该提示将用于生成预览图片。如果为空，将使用来自培训的提示。
	
#### 文件词
`[filewords]` 是提示模板文件的标签，允许您将文件名中的文本插入提示。 默认情况下，文件的扩展名以及文件名开头的所有数字和破折号 (-) 都会被删除。 所以这个文件名：`000001-1-a man in suit.png` 将成为提示文本：`a man in suit`。 文件名中文本的格式保持不变。

可以使用选项 `Filename word regex` 和 `Filename join string` 来更改文件名中的文本：例如，使用 word regex = `\w+` 和 join string =` , `，上面的文件将生成以下文本：`a, man, in, suit` . 正则表达式用于从文本中提取单词（它们是` ['a', 'man', 'in', 'suit', ]`），并将连接字符串 (', ') 放在这些单词之间以创建一个文本： `a, man, in, suit`

也可以创建一个与图像文件名相同的文本文件（`000001-1-a man in suit.txt`），然后将提示文本放在那里。 将不使用文件名和正则表达式选项
#### 第三方回购
我使用这些存储库成功地训练了嵌入：

- [nicolai256](https://github.com/nicolai256/Stable-textual-inversion_win)
- [lstein](https://github.com/invoke-ai/InvokeAI)

其他选择是在 colabs 上进行培训和/或使用扩散器库，我对此一无所知。

### 在线查找嵌入
- [huggingface concepts library](https://cyberes.github.io/stable-diffusion-textual-inversion-models/) - 很多不同的嵌入，但是
- [16777216c](16777216c) - NSFW，一个神秘陌生人的动漫艺术家风格。
- [cattoroboto](https://gitlab.com/cattoroboto/waifu-diffusion-embeds) - anon 的一些动画嵌入。
- [viper1](https://gitgud.io/viper1/stable-diffusion-embeddings) - NSFW，毛茸茸的女孩。
- [anon's embeddings](https://mega.nz/folder/7k0R2arB#5_u6PYfdn-ZS7sRdoecD2A)- NSFW，动漫艺术家。
- [rentry](https://rentry.org/embeddings)- 一个页面，链接到来自许多来源的嵌入。

## Hypernetworks 超网络
超网络是一种新颖的（明白了吗？）概念，用于在不触及任何权重的情况下微调模型。

当前训练超网络的方法是在文本反转选项卡中。

训练的工作方式与文本倒置相同。

唯一的要求是使用非常非常低的学习率，例如 0.000005 或 0.0000005。
### 达姆达指南
一位匿名用户写了一个使用超网络的图片指南：[https://rentry.org/hypernetwork4dumdums](https://rentry.org/hypernetwork4dumdums)
### 训练时从 VRAM 中卸载 VAE 和 CLIP
设置选项卡上的此选项允许您以较慢的预览图片生成为代价节省一些内存。 

## 参考
[Textual Inversion](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Textual-Inversion)	

	