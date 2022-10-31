# Stable Diffusion 更新和使用心得
## 更新的 Python 脚本
自从上一篇文章以来，我已经清理了我用来调用 Stable Diffusion 的 Python 脚本。新版本现在允许直接从 CLI 传入所有参数——无需手动编辑 Python 代码。

它增加了一个要求，一个名为“Click”的 Pip 包，它使解析 CLI 参数变得非常简单。确保您已激活虚拟环境（`virtualenv/Scripts/Activate.ps1` 或 `virtualenv\Scripts\activate.bat`），然后运行：

	pip install click

..你会准备好

更新后的 Python 脚本如下所示：	

	import click
	from diffusers import StableDiffusionOnnxPipeline
	import numpy as np
	
	@click.command()
	@click.option("-p", "--prompt", required=True, type=str)
	@click.option("-w", "--width", required=False, type=int, default=512)
	@click.option("-h", "--height", required=False, type=int, default=512)
	@click.option("-st", "--steps", required=False, type=int, default=25)
	@click.option("-g", "--guidance-scale", required=False, type=float, default=7.5)
	@click.option("-s", "--seed", required=False, type=int, default=None)
	def run(
	    prompt: str, 
	    width: int, 
	    height: int, 
	    steps: int, 
	    guidance_scale: float, 
	    seed: int):
	
	    pipe = StableDiffusionOnnxPipeline.from_pretrained(
	        "./stable_diffusion_onnx", 
	        provider="DmlExecutionProvider"
	    )        
	
	    # Generate our own latents so that we can provide a seed.
	    seed = np.random.randint(np.iinfo(np.int32).max) if seed is None else seed
	    latents = get_latents_from_seed(seed, width, height)
	
	    print(f"\nUsing a seed of {seed}")
	    image = pipe(prompt, height=height, width=width, num_inference_steps=steps, guidance_scale=guidance_scale, latents=latents).images[0]
	    image.save("output.png")
	
	def get_latents_from_seed(seed: int, width: int, height:int) -> np.ndarray:
	    # 1 is batch size
	    latents_shape = (1, 4, height // 8, width // 8)
	    # Gotta use numpy instead of torch, because torch's randn() doesn't support DML
	    rng = np.random.default_rng(seed)
	    image_latents = rng.standard_normal(latents_shape).astype(np.float32)
	    return image_latents
	
	if __name__ == '__main__':
	    run()
如果这更符合您的风格，您也可以直接在 [GitHub](https://github.com/pingzing/stable-diffusion-playground/blob/main/text2img.py) 上找到它。

它最多需要六个参数，其中一个是必需的：

- `-p` 或 `--prompt` 是必需的，并且是您要从中生成图像的文本提示。
- `-w` 或 `--width` 是可选的，默认为 512，并且必须能被 8 整除。
- `-h` 或 `--height` 是可选的，默认为 512，并且必须能被 8 整除
- `-st` 或 `--steps` 是可选的，默认为 25，是在提示符下执行的迭代次数。一般来说，这个数字越高，输出的质量就越好。
- `-g` 或 `--guidance-scale` 是可选的，默认为 7.5，这是 AI 对您的提示和创造性的重视程度。 0 意味着 AI 将拥有大量的创作自由。 20 或更高意味着它试图严格遵守提示。
- `-s` 或 `--seed` 是可选的，默认为随机生成的 32 位整数，并且是用作生成随机性的种子的值。具有相同种子的相同提示将产生相同的输出。

通过这些修改，您现在可以像这样调用脚本：

	./text2img.py -st 25 -p "A happy cat in a cyberpunk garden, cartoony style, digital painting, artstation, concept art, smooth, sharp focus, illustration, 8k"
	
![](./pic/cyberpunk-cat.png)

## 禁用安全检查器
您可能已经注意到，有时，您的输出图像将只是一个空白的黑色方块，而不是生成有用的东西。 这不是错误，也不是错误——这是因为 Stable Diffusion 的内置安全检查器检测到了 NSFW 或其他令人反感的内容。

现在，如果您发现这是一个有用的功能，您可以通过在 Python 脚本中执行以下操作来检测它并打印出一条消息：

	result = pipe(prompt, height=height, width=width, num_inference_steps=steps, guidance_scale=guidance_scale, latents=latents)
	image = result.images[0]
	is_nsfw = result.has_nsfw_concept
	if is_nsfw: 
	    print("Oh no! NSFW output detected!")
	
	image.save("output.png")
..但对于我的用例，我只在本地运行它，我并不关心人工智能是否偶尔会产生一些问题。 作为额外的奖励，我观察到如果我禁用安全检查器，我会获得相当显着的加速——大约在 20% 到 40% 之间，这通常会缩短我的运行时间大约一分钟。 不错！ 因此，如果您想禁用安全检查器，您所要做的就是在管道声明之后添加以下行：

	# .... etc
	pipe = StableDiffusionOnnxPipeline.from_pretrained(
	    "./stable_diffusion_onnx", 
	    provider="DmlExecutionProvider"
	)    
	
	# Add this line here!
	pipe.safety_checker = lambda images, **kwargs: (images, [False] * len(images))
	# ... etc
这是一个小技巧——我们正在搞乱管道的内部结构，这些内部结构并不是真的要在外部使用，但是动态语言会变成动态语言。我们将 pipe 的 `safe_checker` 成员替换为基本上是一个无条件返回 false 的虚拟函数。

现在，不再有黑色方块！请注意，您现在很可能会生成您可能不想在工作中打开的东西。

## 使用不同的调度器
Stable Diffusion 可以使用多种不同的采样方法，`diffusers ` 包在内部将其称为“调度器 schedulers”。坦率地说，所有这些的细节我都没有仔细研究过。简短的版本是它们输出的特征，特别是在较少的步数下，往往会有所不同。因此，有时使用不同的调度程序会很有用。要使用不同的，您必须手动构建它，然后将其传递给` from_pretrained() `的调用。例如：

	# Up in your imports, add the DDIMScheduler from diffusers
	import click
	import diffusers import StableDiffusionOnnxPipeline, DDIMScheduler
	import numpy as np
	
	# Skipping a few lines for brevity...
	
	# Constructing the DDIMScheduler scheduler manually:
	scheduler = DDIMScheduler(beta_start=0.00085, beta_end=0.012, beta_schedule="scaled_linear", num_train_timesteps=1000)
	
	# And telling the created pipe to use it:
	    pipe = StableDiffusionOnnxPipeline.from_pretrained(
	        "./stable_diffusion_onnx", 
	        provider="DmlExecutionProvider",
	        scheduler=scheduler
..但是，如果您按原样运行，它将无法正常工作。 你会得到一个神秘的错误 “expected np.int64, got np.int32”。

解决这个问题需要两件事，第一件事是非常 hacky。

- 第一件事

	我们需要去修改我们本地版本的 Stable Diffusion 的 Onnx 管道。 为了找到它，请在您设置了虚拟环境的任何文件夹中查看 `virtualenv\Lib\site-packages\diffusers\pipelines\stable_diffusion\`。

	在那里，找到 `pipeline_stable_diffusion_onny.py`。 这就是我们的目标。 打开它，向下到第 133 行。我们将把它从：
	
		# OLD
		sample=latent_model_input, timestep=np.array([t]), encoder_hidden_states=text_embeddings
		
		# NEW
		sample=latent_model_input, timestep=np.array([t], dtype=np.int64), encoder_hidden_states=text_embeddings
	我们现在在调用 `np.array()` 时指定 `dtype`。

	请记住，如果您重新创建虚拟环境、重新安装 `diffusers`包或更新` diffusers `包，此更改将无法保留。 无论如何，我完全希望在下一个版本的`diffusers `中这种需求会消失。

- 第二件事

	一旦修改了 `diffusers` 包，您需要对我们声明调度程序的方式进行微小的更改。 让我们重用我们的 DDIM 调度程序示例。

	而不是这样做：
	
		# Wrong
		scheduler = DDIMScheduler(beta_start=0.00085, beta_end=0.012, beta_schedule="scaled_linear", num_train_timesteps=1000)
	
		# Right
		scheduler = DDIMScheduler(beta_start=0.00085, beta_end=0.012, beta_schedule="scaled_linear", num_train_timesteps=1000, tensor_format="np")
		
	（我相信这是因为我们使用的是 Onnx 而不是 Torch，我们需要告诉调度程序使用 Numpy 的张量格式，而不是 Torch 的。我认为。我不是这方面的专家。）

完成第一件事和第二件事后，您现在应该可以使用其他调度程序了。构建它们的示例可以在 [HuggingFace](https://github.com/huggingface/diffusers/tree/main/src/diffusers/pipelines/stable_diffusion) 的 `diffusers` [存储库](https://github.com/huggingface/diffusers/tree/main/src/diffusers/pipelines/stable_diffusion) 中找到。

有关所有这些放在一起的示例，请查看[我在 GitHub 中的版本](https://github.com/pingzing/stable-diffusion-playground/blob/main/text2img.py)。

免责声明：我自己只尝试过 DDIM 调度程序——我的 GPU 有点供电不足，我主要只是想运行一些能以更少的步骤产生可接受的结果的东西。如果您在让其他人运行方面取得任何成功，请随时发表评论！

## 包起来
我想这就是我所拥有的一切。上次脚本的 CLI 化版本，禁用安全检查器以提高速度（以及可能的色情输出），并在使用 Onnx 管道时启用其他调度程序。修了几天还不错。

还要感谢上一篇文章评论中的 ponut64，以及 AzuriteCoin 确认 Onnx 调度程序修复。

一个额外的想法：我将来可能做的一件事是增强我的小 CLI 脚本，以允许调用者选择使用哪个调度程序。我将不得不再玩一点，但如果你对这样的事情感兴趣，请注意这个空间（不要只是自己破解）

## 参考
- [上一篇文章中 Python 脚本的更新版本，现在使用 CLI args](https://www.travelneil.com/stable-diffusion-updates.html#updated-python-script)
- [如何禁用安全检查器（以及为什么它有时会导致黑色方块）](https://www.travelneil.com/stable-diffusion-updates.html#disabling-the-safety-checker)
- [如何使用不同的调度器（例如 DDIM 调度器）](https://www.travelneil.com/stable-diffusion-updates.html#using-different-schedulers)
- [Stable Diffusion Updates](https://www.travelneil.com/stable-diffusion-updates.html)






















## 参考
[Stable Diffusion Updates](https://www.travelneil.com/stable-diffusion-updates.html)