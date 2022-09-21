# babylon 官方深度教学-动画-设计动画
## 设计剪辑
第一步是决定你想在剪辑中看到什么，那就是表现。这给了表演者和它的动画。

在这个游戏中，一个盒子，表演者，每秒在两个地方之间滑动一次。可以从任何角度查看该盒子。

第一阶段设计是在关键时间点勾勒出需要的东西，有点像 gif 动画设计。

![](./pic/ani1.jpeg)
一秒钟后，盒子应该在它的新位置，一秒钟后在它的开始位置。然后不断重复这个序列。

在 Babylon.js 中，动画正在改变盒子沿 x 轴的位置，它的 x 位置是一个浮点数，并且动画应该循环。在代码中，沿 x 方向滑动项目的动画变为

	const frameRate = 10;
	// 新建动画
	const xSlide = new BABYLON.Animation("xSlide", "position.x", frameRate, BABYLON.Animation.ANIMATIONTYPE_FLOAT, BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE);

关键帧位于 0、1 和 2 秒。要找到 t 秒后的帧号，请将时间乘以帧速率，即 `tx frameRate`。

在这种情况下，关键帧的帧号为 0、`1x frameRate` 和 `2x frame Rate`。

从 `x = 2` 开始盒子并将其滑动到 `x = -2`，在 0、1 和 2 秒后盒子的 x 位置值分别为 `2、-2` 和 `2`。

关键帧被设置为具有帧（编号）和值属性的 JavaScript 对象数组，并添加到动画中，如

	// 设置帧队列
	const keyFrames = [];
	
	// 插入帧数据
	keyFrames.push({
	    frame: 0,
	    value: 2,
	});
	
	keyFrames.push({
	    frame: frameRate,
	    value: -2,
	});
	
	keyFrames.push({
	    frame: 2 * frameRate,
	    value: 2,
	});
	
	// 动画绑定帧
	xSlide.setKeys(keyFrames);	
动画现在已完全制作，可以应用到盒子上

	box.animations.push(xSlide);
表演（Animatable ）开始于

	scene.beginAnimation(box, 0, 2 * frameRate, true);
	
- [组合例子](https://playground.babylonjs.com/#7V0Y1I)

		const createScene = () => {
		
		    // 创建场景
		    const scene = new BABYLON.Scene(engine);
		    
		    // 创建摄像头
		    const camera = new BABYLON.ArcRotateCamera("Camera", - Math.PI / 2, Math.PI / 4, 10, BABYLON.Vector3.Zero());
		    // 摄像头控制
		    camera.attachControl(canvas, true);
			
		     // 创建设置两个灯光和强度	
		    const light1 = new BABYLON.DirectionalLight("DirectionalLight", new BABYLON.Vector3(0, -1, 1));
		    const light2 = new BABYLON.HemisphericLight("HemiLight", new BABYLON.Vector3(0, 1, 0));   
		    light1.intensity =0.75;
		    light2.intensity =0.5;
		
		    // 创建盒子和设置位置
		    const box = BABYLON.MeshBuilder.CreateBox("box", {});
		    box.position.x = 2;
		
		    // 动画
		    // 设置每秒帧
		    const frameRate = 10;
		
		    // 创建动画
		    const xSlide = new BABYLON.Animation("xSlide", "position.x", frameRate, BABYLON.Animation.ANIMATIONTYPE_FLOAT, BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE);
		
		    // 创建关键帧数组
		    const keyFrames = []; 
		
		    // 向数组插入数据
		    keyFrames.push({
		        frame: 0,
		        value: 2
		    });
		
		    keyFrames.push({
		        frame: frameRate,
		        value: -2
		    });
		
		    keyFrames.push({
		        frame: 2 * frameRate,
		        value: 2
		    });
		    // 关键帧数组绑定到动画
		    xSlide.setKeys(keyFrames);
		    // 盒子动画绑定盒子
		    box.animations.push(xSlide);
		    // 开始动画
		    scene.beginAnimation(box, 0, 2 * frameRate, true);
		
			return scene;
		};

## 反转动画
有趣的提示，beginAnimation 方法的第二个和第三个参数是 keyFrames 列表中的起始帧和结束帧。如果你反转这两个值，动画将反向播放

