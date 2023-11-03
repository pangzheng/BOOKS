# Gym Retro
项目状态：维护（预计错误修复和小更新）

Gym Retro 可让您将经典视频游戏转变为 Gym 环境以进行强化学习，并集成了约 1000 个游戏。它使用支持 Libretro API 的各种模拟器，使得添加新模拟器变得相当容易。

支持的平台：

- Windows 7、8、10
- macOS 10.13（High Sierra）、10.14（Mojave）
- Linux（manylinux1）

支持的Python：

- 3.6
- 3.7
- 3.8

每个游戏集成都包含列出游戏内变量的内存位置、基于这些变量的奖励函数、情节结束条件、关卡开始处的保存状态的文件，以及包含与这些文件配合使用的 ROM 哈希值的文件。

请注意，ROM 不包含在内，您必须自行获取。大多数 ROM 哈希值均源自其各自的 No-Intro SHA-1 和。

## 文档
文档位于 [https://retro.readthedocs.io/en/latest/](https://retro.readthedocs.io/en/latest/)

您可能应该从[入门指南开始](https://retro.readthedocs.io/en/latest/getting_started.html)。
## 贡献
请参阅 [CONTRIBUTING.md](https://github.com/openai/retro/blob/master/CONTRIBUTING.md)
## 变更日志
参见 [CHANGES.md](https://github.com/openai/retro/blob/master/CHANGES.md)

## 模拟系统
- 雅达利
	- Atari2600（通过 Stella）
- NEC
	- TurboGrafx-16/PC 引擎（通过 Mednafen/Beetle PCE Fast）
- 任天堂
	- Game Boy/Game Boy Color（来自 gambatte）
	- Game Boy Advance（通过 mGBA）
	- Nintendo Entertainment System 任天堂娱乐系统（来自 FCEUmm）
	- Super Nintendo Entertainment System 超级任天堂娱乐系统（通过 Snes9x）
- 世嘉
	- GameGear（通过 Genesis Plus GX）
	- Genesis/Mega Drive（通过 Genesis Plus GX）
	- Mster System（通过 Genesis Plus GX）

有关各个内核许可证的信息，请参阅 [LICENSES.md](https://github.com/openai/retro/blob/master/LICENSES.md) 。

## 包含的 ROM
Gym Retro 中包含以下非商业 ROM，用于测试目的：

- Anthrox [the 128 sine-dot](http://www.pouet.net/prod.php?which=2762)
- 本·瑞维斯 (Ben Ryves) 的[Sega Tween](https://pdroms.de/files/gamegear/sega-tween)
- Blind IO [Happy 10](http://www.pouet.net/prod.php?which=52716) 
- Chris Covell 的 [512 色测试演示](https://pdroms.de/files/pcengine/512-colour-test-demo)
- 由 Dekadence 设计的 [Dekadrive](http://www.pouet.net/prod.php?which=67142)
- 德里克·莱德贝特的[自动机](https://pdroms.de/files/atari2600/automaton-minigame-compo-2003)
- dox [fire](http://privat.bahnhof.se/wb800787/gb/demo/64/)
- dr88 的 [FamiCON intro](http://www.pouet.net/prod.php?which=53497)
- [空袭者](https://pdroms.de/genesis/airstriker-v1-50-genesis-game) Electrokinesis
- Vantage [丢失的弹珠](https://pdroms.de/files/gameboyadvance/lost-marbles)

## 引文
请使用以下 BibTeX 条目进行引用：

	@article{nichol2018retro,
	  title={Gotta Learn Fast: A New Benchmark for Generalization in RL},
	  author={Nichol, Alex and Pfau, Vicki and Hesse, Christopher and Klimov, Oleg and Schulman, John},
	  journal={arXiv preprint arXiv:1804.03720},
	  year={2018}
	}