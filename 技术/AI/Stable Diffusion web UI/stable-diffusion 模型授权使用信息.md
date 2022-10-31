# stable-diffusion v 1.4 模型授权使用信息

# 模型基础信息
许可证：creativeml-openrail-m

标签：

- stable-diffusion
- text-to-image

library_name：“stable-diffusion”

extra_gated_prompt：|-
  在获得此模型之前还有一步。该模型是开放访问所有人都可以使用，CreativeML OpenRAIL-M 许可证进一步指定了权利和使用。
  
  CreativeML OpenRAIL 许可证指定：
 
  1. 不得利用模型故意制作或分享非法或有害的输出或内容
  2. CompVis 对您生成的输出不主张任何权利，您可以自由使用它们并对它们的使用负责，不得违反许可中的规定
  3. 您可以重新分配权重并将模型用于商业和/或作为服务。如果您这样做，请注意您必须包含与许可证中相同的使用限制，并向所有用户共享 CreativeML OpenRAIL-M 的副本（请完整并仔细阅读许可证）
  
  请在此处阅读完整许可：https://huggingface.co/spaces/CompVis/stable-diffusion-license
  
  通过单击下面的“访问存储库”，您接受您的*联系信息*（电子邮件地址和用户名）也可以与模型作者共享。
    
---
stable-diffusion 是一种潜在的文本到图像扩散模型，能够在给定任何文本输入的情况下生成照片般逼真的图像。

**Stable-Diffusion-v-1-4** 检查点使用 [Stable-Diffusion-v-1-2] (https://steps/huggingface.co/CompVis/stable-diffusion-v-1-2-original) 检查点并随后在“laion-aesthetics v2 5+”上以 512x512 分辨率对 225k 步进行微调并减少 10% 的文本调节以改进 [classifier-free guidance sampling](https://arxiv.org/abs/2207.12598)。

#### 下载权重
- [sd-v1-4.ckpt](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/resolve/main/sd-v1-4.ckpt)
- [sd-v1-4-full-ema.ckpt](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/resolve/main/sd-v1-4-full-ema.ckpt)

这些权重旨在与原始的 [CompVis Stable-Diffusion 码库](https://github.com/CompVis/stable-diffusion) 一起使用。如果您正在寻找与 D🧨iffusers 库一起使用的模型，[来这里](https://huggingface.co/CompVis/stable-diffusion-v1-4)。

## 模型详细信息
- **开发者：** Robin Rombach, Patrick Esser
- **模型类型：** Diffusion-based text-to-image 生成模型
- **语言：** 英语
- **许可证：** 

	 [CreativeML OpenRAIL M 许可证](https://huggingface.co/spaces/CompVis/stable-diffusion-license) 是 [Open RAIL M 许可证](https://www.licenses.ai/blog/2022/8/18/naming-convention-of-responsible-ai-licenses)，改编自 [BigScience](https://bigscience.huggingface.co/) 和 [the RAIL Initiative]( https://www.licenses.ai/) 共同开展负责任的人工智能许可领域。另请参阅[关于 BLOOM Open RAIL 许可证的文章](https://bigscience.huggingface.co/blog/the-bigscience-rail-license)，我们的许可证基于该文章。
- **模型说明：** 

	这是一个模型，可用于根据文本提示生成和修改图像。它是一个 [Latent Diffusion Model](https://arxiv.org/abs/2112.10752)，使用固定的、预训练的文本编码器 ([CLIP ViT-L/14](https://arxiv.org/abs/2103.00020) )) 如 [Imagen 论文](https://arxiv.org/abs/2205.11487) 中建议的那样。
- **更多信息的资源：** [GitHub 存储库](https://github.com/CompVis/stable-diffusion)、[论文](https://arxiv.org/abs/2112.10752)。
- **引用为：**

      @InProceedings{Rombach_2022_CVPR,
          作者 = {Rombach, Robin and Blattmann, Andreas and Lorenz, Dominik and Esser, Patrick and Ommer, Bj\"orn},
          标题 = {具有潜在扩散模型的高分辨率图像合成}，
          booktitle = {计算机视觉和模式识别 (CVPR) IEEE/CVF 会议论文集}，
          月 = {六月}，
          年 = {2022}，
          页数 = {10684-10695}
 
# 用途
## 直接使用
该模型仅用于研究目的。 可能的研究领域和
任务包括

- 安全部署可能产生有害内容的模型。
- 探索和理解生成模型的局限性和偏差。
- 艺术品的产生以及在设计和其他艺术过程中的使用。
- 在教育或创意工具中的应用。
- 生成模型研究。

排除用途如下所述。

### 滥用、恶意使用和超出范围的使用
_注意：本节取自 [DALLE-MINI 模型卡](https://huggingface.co/dalle-mini/dalle-mini)，但同样适用于 Stable Diffusion v1_。

该模型不应用于故意创建或传播为人们创造敌对或疏远环境的图像。这包括生成人们可以预见的令人不安、痛苦或冒犯的图像；或传播历史或当前刻板印象的内容。    
## 范围外使用
该模型未经过训练以真实或真实地表示人或事件，因此使用该模型生成此类内容超出了该模型的能力范围。
### 滥用和恶意使用
使用该模型生成对个人残忍的内容是对该模型的滥用。这包括但不限于：

- 对人或其环境、文化、宗教等进行贬低、非人化或其他有害的表述
- 故意宣传或宣传歧视性内容或有害的刻板印象
- 未经他人同意冒充他人。
- 未经可能看到的人同意的色情内容
- 错误和虚假信息
- 令人震惊的暴力和血腥的表现
- 违反其使用条款共享受版权保护或许可的材料
- 共享内容是违反其使用条款更改受版权保护或许可材料的内容

## 限制和偏见
### 限制
- 模型没有达到完美的真实感
- 模型无法呈现清晰的文本
- 该模型在涉及组合性的更困难的任务上表现不佳，例如渲染对应于“蓝色球体顶部的红色立方体/A red cube on top of a blue sphere”的图像
- 一般的面孔和人物可能无法正确生成。
- 该模型主要使用英文字幕进行训练，在其他语言中效果不佳。
- 模型的自动编码部分是有损的
- 该模型是在大规模数据集上训练的 [LAION-5B](https://laion.ai/blog/laion-5b/)  包含成人内容并且不适合在没有额外安全机制的情况下使用产品，并且考虑因素。
- 没有使用其他措施对数据集进行重复数据删除。结果，我们观察到在训练数据中重复的图像有一定程度的记忆。可以在 [https://rom1504.github.io/clip-retrieval/](https://rom1504.github.io/clip-retrieval/) 上搜索训练数据，以可能有助于检测记忆图像。
  
### 偏见
虽然图像生成模型的能力令人印象深刻，但它们也会加强或加剧社会偏见。

在 [LAION-2B(en)](https://laion.ai/blog/laion-5b/) 的子集上训练了稳定扩散 v1，其中包括主要限于英文描述的图像。

来自使用其他语言的社区和文化的文本和图像可能没有得到充分考虑。

这会影响模型的整体输出，因为白人和西方文化通常被设置为默认值。此外，该模型生成非英语提示内容的能力明显低于英语提示。

### 训练
**训练数据**

模型开发人员使用以下数据集来训练模型：

- LAION-2B (en) 及其子集（见下一节）

**培训程序**
Stable Diffusion v1 是一种潜在扩散模型，它结合了自动编码器和在自动编码器的潜在空间中训练的扩散模型。在训练中，

- 图像通过编码器进行编码，将图像转换为潜在表示。自动编码器使用 8 的相对下采样因子，并将形状为 H x W x 3 的图像映射到形状为 H/f x W/f x 4 的潜在图像
- 文本提示通过 ViT-L/14 文本编码器进行编码。
- 文本编码器的非池化输出通过交叉注意输入到潜在扩散模型的 UNet 主干。
- 损失是添加到潜在噪声和 UNet 预测之间的重建目标。

我们目前提供三个检查点，`sd-v1-1.ckpt`、`sd-v1-2.ckpt`和`sd-v1-3.ckpt`，

训练如下，

- `sd-v1-1.ckpt`：在 [laion2B-en](https://huggingface.co/datasets/laion/laion2B-en) 上以 `256x256` 分辨率执行 237k 步。在 [laion-high-resolution](https://huggingface.co/datasets/laion/laion-high-resolution) 分辨率为 `512x512` 的 194k 步（来自 LAION-5B 的 170M 示例，分辨率为 `>= 1024x1024`）。
- `sd-v1-2.ckpt`：从`sd-v1-1.ckpt`恢复。在“laion-improved-aesthetics”（laion2B-en 的一个子集，过滤到具有原始尺寸“> = 512x512”、估计美学分数“> 5.0”和估计水印概率“< 0.5”的图像。水印估计来自 LAION-5B 元数据，美学分数是使用 [改进的美学估计器](https://github.com/christophschuhmann/improved-aesthetic-predictor) 估计的。
- `sd-v1-3.ckpt`：从`sd-v1-2.ckpt`恢复。在“laion-improved-aesthetics”上以“512x512”分辨率执行 195k 步，并减少 10% 的文本条件以改进 [无分类器指导抽样](https://arxiv.org/abs/2207.12598)。
- **硬件：** 32 x 8 x A100 GPU
- **优化器：** AdamW
- **梯度累积**：2
- **批次：** 32 x 8 x 2 x 4 = 2048
- **学习率：** 10,000 步预热到 0.0001，然后保持不变

## 评估结果
评价具有不同无分类器指导尺度（1.5、2.0、3.0、4.0、5.0、6.0、7.0、8.0）和 50 PLMS 采样

步骤显示检查点的相对改进：
![pareto](https://huggingface.co/CompVis/stable-diffusion/resolve/main/v1-variants-scores.jpg)

使用来自 COCO2017 验证集的 50 个 PLMS 步骤和 10000 个随机提示进行评估，以 512x512 分辨率进行评估。未针对 FID 分数进行优化。

## 对环境造成的影响
**稳定扩散 v1** **估计排放** 

基于该信息，我们使用 [Lacoste et al. (2019)](https://arxiv.org/abs/1910.09700)。硬件、运行时、云提供商和计算区域被用来估计碳影响。

- **硬件类型：** A100 PCIe 40GB
- **使用小时数：** 150000
- **云提供商：** AWS
- **计算区域：** 美国东部
- **排放的碳（功耗 x 时间 x 基于电网位置产生的碳）：** 11250 kg CO2 eq.

## 引用
```中文提供
    @InProceedings{Rombach_2022_CVPR,
        作者 = {Rombach, Robin and Blattmann, Andreas and Lorenz, Dominik and Esser, Patrick and Ommer, Bj\"orn},
        标题 = {具有潜在扩散模型的高分辨率图像合成}，
        booktitle = {计算机视觉和模式识别 (CVPR) IEEE/CVF 会议论文集}，
        月 = {六月}，
        年 = {2022}，
        页数 = {10684-10695}
    }
```
*此模型卡由 Robin Rombach 和 Patrick Esser 编写，基于 [DALL-E Mini 模型卡](https://huggingface.co/dalle-mini/dalle-mini)。*


## 原文
- [CompVis  stable-diffusion-v-1-4-original Copied](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original)
- [readme](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/blob/main/README.md)