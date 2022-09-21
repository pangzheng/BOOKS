# babylon官方手册-引导学习-创建游戏
## 创建游戏简介
大家好，我是 capucat，我是团队的实习生！我今年夏天的项目是使用 Babylon.js (4.1) 开发一款游戏，并通过本教程分享我的学习成果。

我创建的游戏是 Hanabi，即 Summer's Festival。这是一款基于夏季节日和纸灯笼照明的简短 3D 平台游戏。故事是关于这只猫，它的家人为村庄制作所有的灯笼和烟花。每年都有一个特殊的夏季节日，他们有一个烟花表演来开始它。目标是穿过村庄，点亮所有灯笼，然后将烟花送到山顶神社。您只有 1 小时（实时 4 分钟）和 1 个火花（20 秒寿命）。

![](./pic/startscreen.png)

在使用 Babylon.js 时，我是一个完整的初学者，所以这是一次很棒的学习经历。我很高兴能与大家分享这个过程，因为社区创造了很多令人惊叹的东西。我希望这个系列能够激励人们开始使用 Babylon.js 进行自己的游戏开发，或者只是开始探索使用 Babylon.js 可以做的事情！
## 概括
从在本地设置项目开始，本系列将引导您了解我如何创建这个游戏，链接所有相关文件。本系列并不是完美游戏的指南，而是学习过程的反映。游戏开发是一个迭代的过程，所以这个系列不是我如何开发游戏的一步一步的；但是，我已尽力将其分解为解释概念和我发现最有效的总体方法的部分。

需要注意的是，这款游戏只触及了 Babylon.js 所提供功能的表面，因此请随意修改和尝试不同的功能！

[Github 游戏仓库](https://github.com/BabylonJS/SummerFestival)
## 开始设置
### 介绍
该游戏将使用 Typescript 开发，并且需要使用 NPM 进行一些设置。项目的设置可能是最难的部分！要更简单地介绍 Babylon.js 的功能以及基本的 HTML 和 JavaScript 模板，请先阅读 [入门](https://doc.babylonjs.com/start) 部分。

此游戏示例的设置过程基于 Babylon.js 和 [babylonjs webpack](https://github.com/RaananW/babylonjs-webpack-es6) 示例项目的[NPM 支持文档](https://doc.babylonjs.com/divingDeeper/developWithBjs/npmSupport)，但有一些添加和修改。
### 第一步
#### 创建项目
首先，您需要设置项目所在的位置。

1. 使用您选择的文本编辑器，打开您的项目文件夹。我将在本系列中使用 Visual Studio Code。
- 创建一个文件夹，您将在其中存储项目文件
- 将文件夹结构设置为如下所示：
	- dist
	- public
	- src
- 主要文件
	- 进入您的 src 文件夹并创建一个 app.ts 文件
	- 进入您的公用文件夹并创建一个包含以下内容的 index.html 文件：



