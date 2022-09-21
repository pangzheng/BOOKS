# babylon 官方深度教学-相机-相机介绍
## 相机介绍
Babylon.js 可以使用多种相机，其中最常用的两种可能是 Universal 相机和 arcRotate 相机以及越来越多的虚拟现实相机。

- 通用相机 UniversalCamera

	第一人称射击类游戏的首选，适用于所有键盘、鼠标、触控和游戏手柄。使用此相机，您可以检查碰撞并应用重力。
- 弧形旋转相机 Arc Rotate Cameras 

	其作用类似于围绕目标运行的卫星，始终指向目标位置。
- 跟随相机 followCamera

	这会将网格作为目标并在其移动时跟随它。提供免费相机版本 `followCamera` 和圆弧旋转版本 `arcFollowCamera`。
- Anaglyph Camera 

	扩展 Universal 和 Arc Rotate Cameras 的使用范围，用于红色和青色 3D 眼镜。
- 设备方向相机

	这是设计用于对向前或向后和向左或向右倾斜的设备做出反应。
- 虚拟操纵杆相机

	用于控制相机或其他场景项目的屏幕 2D 图形。
- 虚拟现实相机

	系列适用于 VR 设备的相机

可以使用一些方法来更改相机对输入的反应方式以及使用多个相机。

## 相机介绍
在 Babylon.js 中可用的众多摄像头中，最常用的三个可能是用于

- “第一人称”运动的 [Universal Camera](https://doc.babylonjs.com/typedoc/classes/babylon.universalcamera) 
- 轨道摄像头 [ArcRotateCamera](https://doc.babylonjs.com/typedoc/classes/babylon.arcrotatecamera) 
- 用于现代虚拟现实体验的 [WebXRCamera](https://doc.babylonjs.com/typedoc/classes/babylon.webxrcamera) 。

要允许用户输入，必须使用以下方法将相机连接到画布：

	camera.attachControl(canvas, true);
- 第二个参数是可选的，默认为 `false`，这会阻止对画布事件的默认操作。设置为 `true` 允许画布默认操作。
- 对于[游戏手柄](https://doc.babylonjs.com/divingDeeper/input/gamepads)可用作控制器。
- 对于触摸控制，需要[PEP](https://github.com/jquery/PEP)或[hand.js](https://github.com/Deltakosh/handjs)

### 万能相机 Universal Camera
这是在 2.3 版的 Babylon.js 中引入的，由键盘、鼠标、触摸或[游戏手柄](https://doc.babylonjs.com/divingDeeper/input/gamepads)控制，具体取决于所使用的输入设备，无需指定控制器。这扩展并取代了仍然可用的 [Free Camera](https://doc.babylonjs.com/typedoc/classes/babylon.freecamera)、[Touch Camera](https://doc.babylonjs.com/typedoc/classes/babylon.touchcamera) 和 [Gamepad Camera](https://doc.babylonjs.com/typedoc/classes/babylon.gamepadcamera) 。

Universal Camera 现在是 Babylon.js 使用的默认相机，如果您想在场景中拥有类似 FPS 的控件，它是您的最佳选择。[babylonjs.com](https://www.babylonjs.com/) 上的所有演示都使用通用相机。将 Xbox 控制器插入您的 PC，您将能够使用它来浏览大多数演示。

默认操作是：

- 键盘——左右方向键左右移动相机，上下方向键前后移动；
- 鼠标 - 以相机为原点围绕轴旋转相机；
- 触摸 - 左右滑动可左右移动相机，上下滑动可前后移动；
- [游戏手柄](https://doc.babylonjs.com/divingDeeper/input/gamepads)- 对应于设备。

可选操作是：

- 鼠标中键 MouseWheel - 鼠标上的滚轮或触摸板上的滚动动作
- [鼠标中键 demo](https://playground.babylonjs.com/#DWPQ9R#1)

在 Playground 中使用键需要您在渲染区域内单击以使其获得焦点。

- 构建通用摄像机

		// 参数:名称、位置、场景
		var camera = new BABYLON.UniversalCamera("UniversalCamera", new BABYLON.Vector3(0, 0, -10), scene);
		
		//将摄像机对准一个特定的位置。在这种情况下，场景源
		camera.setTarget(BABYLON.Vector3.Zero());
		
		// 将相机固定在画布上
		camera.attachControl(canvas, true);	
- [Universal Camera Example](https://playground.babylonjs.com/#DWPQ9R#1)

	    // 创建场景
	    var createScene = function () {
	    // 这样就创建了一个基本的巴比伦场景对象(非网格)
	    var scene = new BABYLON.Scene(engine);
	
	    // 这创建并定位了一个通用相机(非网格)
	    var camera = new BABYLON.UniversalCamera("camera1", new BABYLON.Vector3(0, 5, -10), scene);
	
	     // 启用鼠标滚轮输入。
	     camera.inputs.addMouseWheel();
	    
	    // 改变鼠标滚轮Y轴来控制场景中摄像机的高度:
	    //camera.inputs.attached["mousewheel"].wheelYMoveRelative = BABYLON.Coordinate.Y;
	    
	    // 修改鼠标滚轮Y轴方向:
	    // camera.inputs.attached["mousewheel"].wheelPrecisionY = -1;
	
	    // 这将相机指向场景源
	    camera.setTarget(BABYLON.Vector3.Zero());
	
	    // 这样就把相机固定在画布上了
	    camera.attachControl(true);
	
	    // 这就产生了一盏灯，瞄准 0,1,0-天空(非网格)
	    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
	
	    // 默认强度为1。让我们把灯光调暗一点
	    light.intensity = 0.7;
	
	    // 创建内置的“球体”形状。
	    var sphere = BABYLON.MeshBuilder.CreateSphere("sphere", {diameter: 2, segments: 32}, scene);
	
	    // 将球体向上移动1/2高度
	    sphere.position.y = 1;
	
	    // 创建内置的“地面”形状。
	    var ground = BABYLON.MeshBuilder.CreateGround("ground", {width: 6, height: 6}, scene);
	
	    return scene;
	
		};

### 弧形旋转相机
该摄像机始终指向给定的目标位置，并且可以以目标为旋转中心围绕该目标旋转。它可以用光标和鼠标控制，也可以用触摸事件控制。

将这台相机想象成一个围绕其目标位置运行的摄像机，或者更形象地想象成一颗围绕地球运行的卫星。它相对于目标（“地球”）的位置可以通过三个参数设置：

- [alpha](https://doc.babylonjs.com/typedoc/classes/babylon.arcrotatecamera#alpha)（纵向旋转，以弧度为单位）
- [beta](https://doc.babylonjs.com/typedoc/classes/babylon.arcrotatecamera#beta)（纬度旋转，以弧度为单位）
- [radius](https://doc.babylonjs.com/typedoc/classes/babylon.arcrotatecamera#radius) 半径（到目标的距离）。

这是一个插图：

![](./pic/camalphabeta.jpeg)

由于技术原因，设置 beta 为 0 或 PI 可能会导致问题，因此在这种情况下，beta 偏移 0.1 弧度（约 0.6 度）。

- beta 顺时针方向增加
- 逆时针方向 alpha 增加。

相机的位置也可以从一个向量中设置，这将覆盖 alpha,beta 和的任何当前值 radius。这比计算所需的角度要容易得多。

无论是使用键盘、鼠标还是触摸滑动

- 左/右方向变化alpha
- 上/下方向变化beta

以下可选 `ArcRotateCamera` 属性也很方便：

- `zoomToMouseLocation`

	如果设置为 `true` 将导致鼠标滚轮以当前鼠标位置为中心放大或缩小，而不是固定的 `camera.target` 位置。这使得探索大场景的各个角落变得容易。设置此项意味着鼠标滚轮输入控制器将在鼠标滚轮缩放期间更改 `camera.target` 位置。当这是 `true` 使用鼠标滚轮的缩放操作时，同时进行缩放和少量平移。
- `wheelDeltaPercentage` 

	如果设置为非零值，将导致缩放量设置为相机半径的百分比。这意味着当您靠近目标对象时变焦会减慢，这很好，因为这意味着您可以在近距离探索对象时更精确地确定相机位置。
	
- 构建圆弧旋转相机

		// 参数:名称，alpha, beta，半径，目标位置，场景
		var camera = new BABYLON.ArcRotateCamera("Camera", 0, 0, 10, new BABYLON.Vector3(0, 0, 0), scene);
		
		// 定位摄像头覆盖alpha, beta，半径
		camera.setPosition(new BABYLON.Vector3(0, 0, 20));
		
		// 这样就把相机固定在画布上了
		camera.attachControl(canvas, true);
- [例子](https://playground.babylonjs.com/#SRZRWV#839)

		var createScene = function () {
		
		    // 这创建了一个基本的巴比伦场景对象(非网格)
		    var scene = new BABYLON.Scene(engine);
		
			/********** 圆弧旋转相机示例 **************************/
		
		    // 创建，角度，距离和目标的相机
			var camera = new BABYLON.ArcRotateCamera("Camera", 0, 0, 10, new BABYLON.Vector3(0, 0, 0), scene);
		
		    // 这就定位了摄像机
		    camera.setPosition(new BABYLON.Vector3(0, 0, -10));
		
		    // 这样就把相机固定在画布上了
		    camera.attachControl(canvas, true);
		
			/**************************************************************/
		
		    //这创建了一个光，瞄准 0,1,0 天空(非网格)。
		    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
			
			// 材料
			var redMat = new BABYLON.StandardMaterial("red", scene);
			redMat.diffuseColor = new BABYLON.Color3(1, 0, 0);
			redMat.emissiveColor = new BABYLON.Color3(1, 0, 0);
			redMat.specularColor = new BABYLON.Color3(1, 0, 0);
			
			var greenMat = new BABYLON.StandardMaterial("green", scene);
			greenMat.diffuseColor = new BABYLON.Color3(0, 1, 0);
			greenMat.emissiveColor = new BABYLON.Color3(0, 1, 0);
			greenMat.specularColor = new BABYLON.Color3(0, 1, 0);
			
			var blueMat = new BABYLON.StandardMaterial("blue", scene);
			blueMat.diffuseColor = new BABYLON.Color3(0, 0, 1);
			blueMat.emissiveColor = new BABYLON.Color3(0, 0, 1);
			blueMat.specularColor = new BABYLON.Color3(0, 0, 1);
			
			//形状
			var plane1 = BABYLON.MeshBuilder.CreatePlane("plane1", {size: 3, sideOrientation: BABYLON.Mesh.DOUBLESIDE}, scene);
			plane1.position.x = -3;
			plane1.position.z = 0;
			plane1.material = redMat;
			
			var plane2 = BABYLON.MeshBuilder.CreatePlane("plane2", {size: 3, sideOrientation: BABYLON.Mesh.DOUBLESIDE});
			plane2.position.x = 3;
			plane2.position.z = -1.5;
			plane2.material = greenMat;
			
			var plane3 = BABYLON.MeshBuilder.CreatePlane("plane3", {size: 3, sideOrientation: BABYLON.Mesh.DOUBLESIDE});
			plane3.position.x = 3;
			plane3.position.z = 1.5;
			plane3.material = blueMat;
			
			var ground = BABYLON.MeshBuilder.CreateGround("ground1", {width: 10, height: 10, subdivisions: 2}, scene);
			
		    return scene;
		
		};
		
默认情况下，ArcRotateCamera 也可以使用 `Ctrl+ 鼠标左键` 进行平移 。您可以通过在`useCtrlForPanning`为来设置使用鼠标右键：

	camera.attachControl(canvas, noPreventDefault, useCtrlForPanning);
如果需要，您还可以通过设置完全停用平移：

	camera.panningSensibility = 0;
这是一个[演示](https://playground.babylonjs.com/#3B5W22#29)其中一些内容的演示，以及 ArcRotateCamera 的其他功能：

	/**
	 * ArcRotateCamera 带有偏移演示
	 * by Dave Solares (PolygonalSun)
	 * 
	 * 这个演示的目的是展示如何使用带有视觉偏移的 ArcRotateCamera。虽然前提很简单，但执行起来可能有点复杂。我使用的方法是首先 “平移相机” 到选中的网格中心。接下来，我将平移的 x 和 y 作为相机的偏移量。
	 * 重置是有效地做相反的，虽然我做它不同的借用代码从 FreeCamera 的键盘移动。使用该代码，我只是平移回它的目标在中心的位置，并删除偏移。我添加了一些基本的动画，只是为了给你的选择增加一些深度，但它们并没有机械的目的。
	 * 红色的球代表相机的目标位置。如果你在屏幕上看到它，你就没有聚焦在网格上。当你双击，你会改变你的焦点和红球“占有”另一个球。任何选定的球体都将有一个红色的轮廓。
	 * 
	 * 控制
	* 双击球体:
	* 高亮球体和设置为目标旋转周围
	* 
	* 双击空白处:
	* 设置目标为中心的半径为10
	*
	* 右键拖
	* 平移(行为正常的ArcRotateCamera)，不打破目标锁定
	* 
	* 左键键拖:
	* 旋转(行为正常的ArcRotateCamera)
	*/
	
	// 主动选择网格作为相机的目标
	var activeMesh = null;
	var targetLocation = null;
	var animationsActive = true;
	var waitForPanning = false;
	
	/**
	 * 创建场景中可拾取的球体
	 */
	var createMeshes = function (scene) {
	    let dim = 3;
	    let sphereNum = 0;
	    let sphereNode = new BABYLON.Mesh("spheres", scene);
	
	    for (let i = -dim; i < dim; i++) {
	        for (let j = -dim; j < dim; j++) {
	            for (let k = -dim; k < dim; k++) {
	                let name = `sphere${sphereNum++}`;
	                let sphere = BABYLON.MeshBuilder.CreateSphere(name, {diameter: 2, segments: 32}, scene);
	                let variance = Math.floor(Math.random()*5)
	                sphere.position = new BABYLON.Vector3(i*12 + variance,j*12 + variance,k*12 + variance);
	                sphere.outlineWidth = 0.1;
	                sphereNode.addChild(sphere);
	            }
	        }
	    }
	};
	
	/**********动画部分**********/
	/**
	 * 将目标球体淡入场景
	 */
	 
	var fadeInTarget = function() {
	    const fadeFrames = [];
	    const frameRate = 10;
	    const fadeTarget = new BABYLON.Animation("fadeInTarget", "alpha", frameRate, BABYLON.Animation.ANIMATIONTYPE_FLOAT);
	
	    fadeFrames.push({
	        frame: 0,
	        value: 0
	    });
	
	    fadeFrames.push({
	        frame: frameRate/4,
	        value: 0.75
	    });
	
	    fadeTarget.setKeys(fadeFrames);
	    targetLocation.material.animations = [];
	    targetLocation.material.animations.push(fadeTarget);
	    scene.beginAnimation(targetLocation.material, 0, frameRate, false);
	};
	
	/**
	 * 移动目标球体到选中的网格/目标
	 */
	var moveToMesh = function (mesh, scene) {
	    const moveFrames = [];
	    const fadeFrames = [];
	    const frameRate = 10;
	    const origin = targetLocation.position.clone();
	    const destination = mesh.position.clone();
	    const moveTarget = new BABYLON.Animation("moveTarget", "position", frameRate, BABYLON.Animation.ANIMATIONTYPE_VECTOR3);
	    const fadeTarget = new BABYLON.Animation("fadeTarget", "alpha", frameRate, BABYLON.Animation.ANIMATIONTYPE_FLOAT);
	    const mergeEase = new BABYLON.CircleEase();
	    mergeEase.setEasingMode(BABYLON.EasingFunction.EASINGMODE_EASEINOUT);
	    moveTarget.setEasingFunction(mergeEase);
	
	    moveFrames.push({
	        frame: 0,
	        value: origin
	    });
	
	    moveFrames.push({
	        frame: frameRate,
	        value: destination
	    });
	
	    fadeFrames.push({
	        frame: 0,
	        value: 0.75
	    });
	
	    fadeFrames.push({
	        frame: frameRate,
	        value: 0
	    });
	
	    moveTarget.setKeys(moveFrames);
	    fadeTarget.setKeys(fadeFrames);
	    targetLocation.material.animations = [];
	    targetLocation.animations = [];
	    targetLocation.animations.push(moveTarget);
	    targetLocation.material.animations.push(fadeTarget);
	    scene.beginAnimation(targetLocation, 0, frameRate, false);
	    scene.beginAnimation(targetLocation.material, 0, frameRate, false);
	};
	/******** 结束动画部分 **********/
	
	/**
	 * 重置相机的目标，使它回到它的视点的中心
	 */
	var resetCameraTarget = function (camera) {
	    if (activeMesh) {
	        /**
	         * 基本上，我们将反向进行我们设置补偿时所做的工作。
	         * 我们的目标只是设置我们的相机以默认半径10为中心。
	         * 我们首先得到当前半径和期望半径之间的差值。
	         * 接下来，我们设置我们的 localDirection 为我们想要在相对空间中平移相机。
	         * 我们计算和移动相机的目标在绝对空间，然后删除我们的偏移。
	         */
	        let diff = camera.radius - 10;
	        let relPos = activeMesh.getPositionInCameraSpace(camera);
	        let localDirection = new BABYLON.Vector3(relPos.x, relPos.y, diff);
	        let viewMatrix = camera.getViewMatrix();
	        let transformMatrix = camera.getTransformationMatrix();
	        let transformedDirection = BABYLON.Vector3.Zero();
	        
	        viewMatrix.invertToRef(transformMatrix);
	        localDirection.multiplyInPlace(new BABYLON.Vector3(1,1,1));
	        BABYLON.Vector3.TransformNormalToRef(localDirection, transformMatrix, transformedDirection);
	        camera.target.subtractInPlace(transformedDirection);
	
	        camera.targetScreenOffset.x = 0;
	        camera.targetScreenOffset.y = 0;
	
	        activeMesh.renderOutline = false;
	        activeMesh = null;
	        camera.radius = 10;
	    }
	    targetLocation.material.alpha = 0.75;
	    targetLocation.position = camera.target;
	};
	
	/**
	 * 设置相机的目标到一个特定的点，并配置偏移来移动它回到位置。
	 */
	var setCameraOffset = function (camera, mesh) {
	    // If we have a mesh to set that hasn't already been set
	    if (mesh && mesh !== activeMesh) {
	        // Disable outline for previous mesh
	        if (activeMesh) {
	            activeMesh.renderOutline = false;
	        }
	
	        if (!animationsActive) {
	            targetLocation.material.alpha = 0;
	        }
	
	        /** 
	         * 这是很重要的一部分。
	         * getpositionincamerspace 函数将给我们网格的位置，就像我们将相机平移到它一样。
	         * 然后我们取这个值，并设置我们的偏移到该位置的相对 x 和 y，并使用 z 作为我们的半径。
	         * 通过复制 alpha 和 beta 角度，我们有效地执行一个3D平移，然后立即抵消相机回到原来的位置。
	         * */
	        let relPos = mesh.getPositionInCameraSpace(camera);
	        let alpha = camera.alpha;
	        let beta = camera.beta;
	
	        mesh.renderOutline = true;
	        camera.target = mesh.position.clone();
	        camera.targetScreenOffset.x = relPos.x;
	        camera.targetScreenOffset.y = relPos.y;
	        camera.radius = relPos.z;
	        camera.alpha = alpha;
	        camera.beta = beta;
	
	        activeMesh = mesh;
	    }
	};
	
	var createScene = async function () {
	    // 这样就创建了一个基本的场景对象(非网格)
	    var scene = new BABYLON.Scene(engine);
	
	    // 这将创建并定位一个 ArcRotateCamera
	    var camera = new BABYLON.ArcRotateCamera("camera", Math.PI/3, Math.PI/3, 10, BABYLON.Vector3.Zero(), scene);;
	
	    // 这将相机指向场景源
	    camera.setTarget(BABYLON.Vector3.Zero());
	
	    // 这样就把相机固定在画布上了
	    camera.attachControl(canvas, true);
	
	    // 这创建了一个光，瞄准0,1,0 -到天空(非网格)
	    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
	
	    // 默认强度为1。让我们把灯光调暗一点
	    light.intensity = 0.7;
	    light.specular = new BABYLON.Color3(0, 0, 0);
	
	    // 创建所有的球体，可以选择
	    createMeshes(scene);
	
	    // 创建目标球体作为焦点位置的视觉指示器
	    targetLocation = BABYLON.MeshBuilder.CreateSphere("targetLoc", {diameter: 0.75, segments: 32}, scene);
	    targetLocation.position = camera.target;
	    let targetMat = new BABYLON.StandardMaterial("targetMat", scene);
	    targetMat.diffuseColor = BABYLON.Color3.Red();
	    targetMat.alpha = 0.75;
	    targetLocation.material = targetMat;
	
	    // GUI
	    let advancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI("UI");
	
	    let targetText = new BABYLON.GUI.TextBlock();
	    targetText.horizontalAlignment = BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_LEFT;
	    targetText.verticalAlignment = BABYLON.GUI.Control.VERTICAL_ALIGNMENT_TOP;
	    targetText.textHorizontalAlignment = BABYLON.GUI.TextBlock.HORIZONTAL_ALIGNMENT_LEFT;
	    targetText.text = "Target: none";
	    targetText.color = "red";
	    targetText.fontSize = 24;
	    targetText.width = "200px";
	    targetText.height = "30px";
	    if (canvas.width < 500) {
	        targetText.isVisible = false;
	    }
	    advancedTexture.addControl(targetText);
	
	    let buttonPanel = new BABYLON.GUI.StackPanel();
	    buttonPanel.verticalAlignment = BABYLON.GUI.Control.VERTICAL_ALIGNMENT_TOP;
	    advancedTexture.addControl(buttonPanel);  
	
	    // 动画切换按钮
	    let animButton = BABYLON.GUI.Button.CreateSimpleButton("anim", "Toggle Animations");
	    animButton.width = "300px";
	    animButton.height = "30px";
	    animButton.color = "white";
	    animButton.background = "grey";
	    animButton.horizontalAlignment = BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_RIGHT;
	    animButton.onPointerClickObservable.add((eventData) => {
	        if (animationsActive) {
	            animButton.textBlock.color = "red";
	        }
	        else {
	            animButton.textBlock.color = "white";
	        }
	        animationsActive = !animationsActive 
	    });
	    buttonPanel.addControl(animButton);
	
	    // 重置相机位置按钮
	    let resetButton = BABYLON.GUI.Button.CreateSimpleButton("reset", "Reset Camera");
	    resetButton.width = "300px";
	    resetButton.height = "30px";
	    resetButton.color = "white";
	    resetButton.background = "grey";
	    resetButton.horizontalAlignment = BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_RIGHT;
	    resetButton.onPointerClickObservable.add((eventData) => {
	        camera.target = BABYLON.Vector3.Zero();
	        camera.alpha = Math.PI/3;
	        camera.beta = Math.PI/3;
	        camera.radius = 10;
	
	        if (animationsActive && activeMesh) { fadeInTarget(); }
	        camera.targetScreenOffset.x = 0;
	        camera.targetScreenOffset.y = 0;
	
	        activeMesh.renderOutline = false;
	        activeMesh = null;
	        targetLocation.material.alpha = 0.75;
	        targetLocation.position = camera.target;
	        
	        targetText.text = "Target: none";
	    });
	    buttonPanel.addControl(resetButton);
	
	    // 这一部分的存在仅仅是因为我们需要考虑平移惯性。
	    scene.beforeRender = () => {
	        if (waitForPanning && camera.inertialPanningX === 0 && camera.inertialPanningY === 0) {
	            let mesh = activeMesh;
	            activeMesh = null;
	            setCameraOffset(camera, mesh);
	            waitForPanning = false;
	        }
	    };
	
	    // 双击:如果你双击网格，高亮并设置该网格为目标
	    // 否则，将目标重置为视野中心，半径为10
	    scene.onPointerObservable.add((eventData) => {
	        let mesh = eventData.pickInfo.pickedMesh;
	        if (mesh) {
	            if (animationsActive) { moveToMesh(mesh, scene); }
	            setCameraOffset(camera, mesh);
	            targetText.text = `Target: ${mesh.name}`;
	        }
	        else {
	            if (animationsActive && activeMesh) { fadeInTarget(); }
	            resetCameraTarget(camera);
	            targetText.text = "Target: none";
	        }
	    }, BABYLON.PointerEventTypes.POINTERDOUBLETAP);
	
	    // 为了防止丢失使用网格作为目标，我们跟踪我们正在使用的活动网格，并在pan完成后重新设置为目标网格
	    scene.onPointerObservable.add((eventData) => {
	        if (activeMesh) {
	            // 如果我们还在移动，等待移动结束，然后重置
	            if (camera.inertialPanningX !== 0 || camera.inertialPanningY !== 0) {
	                waitForPanning = true;
	            }
	            // 如果松开鼠标右键
	            else if ((eventData.event.button === 2 && eventData.event.pointerType === "mouse") ||
	                // 或者停止他们的接触
	                (eventData.event.buttons === 0 && eventData.event.pointerType === "touch") ||
	                // 或者让我们在按 Ctrl 的时候移动一个指针按钮
	                eventData.event.ctrlKey) {
	                    // 重置网格偏移量，这样我们就不会失去目标
	                    let mesh = activeMesh;
	                    activeMesh = null;
	                    setCameraOffset(camera, mesh);
	                }
	        }
	    }, BABYLON.PointerEventTypes.POINTERUP);
	
	    scene.onKeyboardObservable.add((eventData) => {
	        // 如果我们还在移动，等待移动结束，然后重置
	        if (camera.inertialPanningX !== 0 || camera.inertialPanningY !== 0) {
	                waitForPanning = true;
	        }
	        // 因为我们可以将Ctrl和拖动平移相结合，所以我们还需要考虑先释放Ctrl
	        else if (activeMesh && (eventData.event.keyCode === 17 || eventData.event.key === "Control")) {
	            let mesh = activeMesh;
	            activeMesh = null;
	            setCameraOffset(camera, mesh);
	        }
	    }, BABYLON.KeyboardEventTypes.KEYUP);
	
	    return scene;
	};
	
### 跟随相机
[FollowCamera](https://doc.babylonjs.com/typedoc/classes/babylon.followcamera) 按照它上面说的做。给它一个网格作为目标，它将从当前的任何位置移动到一个目标位置，从中可以查看目标。当目标移动时，跟随相机也会移动。

Follow Camera 的初始位置在创建时设置，然后使用三个参数设置目标位置：

- [radius](https://doc.babylonjs.com/typedoc/classes/babylon.followcamera#radius) : 到目标的距离
- [heightOffset](https://doc.babylonjs.com/typedoc/classes/babylon.followcamera#heightOffset)：目标上方的高度；
- [rotationOffset](https://doc.babylonjs.com/typedoc/classes/babylon.followcamera#rotationOffset) : 在 xy 平面上围绕目标的目标角度。

相机移动到目标位置的速度通过其加速度 ( [cameraAcceleration](https://doc.babylonjs.com/typedoc/classes/babylon.followcamera#cameraacceleration) ) 设置到最大速度 ( [maxCameraSpeed](https://doc.babylonjs.com/typedoc/classes/babylon.followcamera#maxcameraspeed) )。

- 构建跟随相机

		// 参数:名称、位置、场景
		var camera = new BABYLON.FollowCamera("FollowCam", new BABYLON.Vector3(0, 10, -10), scene);
		
		// 摄像机到目标的目标距离
		camera.radius = 30;
		
		// 相机在目标局部原点(中心)以上的目标高度
		camera.heightOffset = 10;
		
		// 相机在xy平面上围绕目标的局部原点(中心)的目标旋转
		camera.rotationOffset = 0;
		
		// 相机从当前位置移动到目标位置时的加速度
		camera.cameraAcceleration = 0.005;
		
		// 停止加速的速度
		camera.maxCameraSpeed = 10;
		
		// 这样就把相机固定在画布上了
		camera.attachControl(canvas, true);
		
		// 注意::在目标创建后设置相机目标，注意从babylonjs v 2.5更改
		// 创建 targetMesh。
		camera.target = targetMesh; //2.4版本及更早版本
		camera.lockedTarget = targetMesh; //2.5版本开始
	
跟随相机 [演示 demo](https://playground.babylonjs.com/#SRZRWV#6)

	var createScene = function () {
	
	    // 创建场景
	    var scene = new BABYLON.Scene(engine);
	
		/********** 跟随相机例子**************************/
	
	    // 这创建和最初的位置跟踪相机	
	    var camera = new BABYLON.FollowCamera("FollowCam", new BABYLON.Vector3(0, 10, -10), scene);
		
		// 摄像机到目标的目标距离
		camera.radius = 30;
		
		// 相机在目标局部原点(中心)以上的目标高度
		camera.heightOffset = 10;
		
		// 相机在xy平面上围绕目标的局部原点(中心)的目标旋转
		camera.rotationOffset = 0;
		
		// 相机从当前位置移动到目标位置时的加速度
		camera.cameraAcceleration = 0.005
		
		// 停止加速的速度
		camera.maxCameraSpeed = 10
		
		// camera.target 在创建目标器后设置
	    
		// 这样就把相机固定在画布上了
	    camera.attachControl(canvas, true);
		
		/**************************************************************/
	
	    // 这创建了一个光，瞄准 0,1,0 天空(非网格)。
	    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
		
		//材质
	     var mat = new BABYLON.StandardMaterial("mat1", scene);
	     mat.alpha = 1.0;
	     mat.diffuseColor = new BABYLON.Color3(0.5, 0.5, 1.0);
	     var texture = new BABYLON.Texture("https://i.imgur.com/vxH5bCg.jpg", scene);
	     mat.diffuseTexture = texture;
		
		// 盒子的每一边都有不同的面来显示相机的旋转
		var hSpriteNb =  3;  // 3 sprites per row
	      var vSpriteNb =  2;  // 2 sprite rows	
		
		var faceUV = new Array(6);
	
	  	for (var i = 0; i < 6; i++) {
	    	faceUV[i] = new BABYLON.Vector4(i/hSpriteNb, 0, (i+1)/hSpriteNb, 1 / vSpriteNb);
	  	}
		  
		// 跟随形状
		var box = BABYLON.MeshBuilder.CreateBox("box", {size: 2, faceUV: faceUV }, scene);
		box.position = new BABYLON.Vector3(20, 0, 10);
		box.material = mat;
	
		// 创建文具灰色盒子的固体粒子系统，以显示盒子和相机的运动
	     var boxesSPS = new BABYLON.SolidParticleSystem("boxes", scene, {updatable: false});
	    
	    // 功能定位灰框
	    var set_boxes = function(particle, i, s) {
	        particle.position = new BABYLON.Vector3(-50 + Math.random()*100, -50 + Math.random()*100, -50 + Math.random()*100); 
	    }
	    
	    // 增加400 box
	    boxesSPS.addShape(box, 400, {positionFunction:set_boxes});  
	    var boxes = boxesSPS.buildMesh(); // mesh of boxes
	
	      /*****************设定相机目标 ************************/ 
		camera.lockedTarget = box;
		/**************************************************************/
		
		
		//  盒运动变量
		var alpha = 0;
		var orbit_radius = 20
		
		
		// 移动盒子，让摄像机跟随它	
		scene.registerBeforeRender(function () {
	     alpha +=0.01;
		 box.position.x = orbit_radius*Math.cos(alpha);
		 box.position.y = orbit_radius*Math.sin(alpha);
		 box.position.z = 10*Math.sin(2*alpha);
		 
		 // 当相机跟随盒子时，改变它的视角
		 camera.rotationOffset = (18*alpha)%360;
	    });
		
	    return scene;
	
	};
	
### 立体相机
[AnaglyphUniversalCamera](https://doc.babylonjs.com/typedoc/classes/babylon.anaglyphuniversalcamera)  和 AnaglyphArcRotateCamera扩展了 Universal 和 Arc Rotate 相机的使用范围，可用于红色和青色 3D 眼镜。他们使用后处理过滤技术。

- 构建立体UniversalCamera

		// 参数:名称、位置、眼域、场景
		var camera = new BABYLON.AnaglyphUniversalCamera("af_cam", new BABYLON.Vector3(0, 1, -15), 0.033, scene);
- 构建立体 ArcRotateCamera

		// 参数:名称、alpha、beta、半径、目标、眼空间、场景
		var camera = new BABYLON.AnaglyphArcRotateCamera("aar_cam", -Math.PI / 2, Math.PI / 4, 20, BABYLON.Vector3.Zero(), 0.033, scene);

	该 `eyeSpace` 参数设置左眼视图和右眼视图之间的偏移量。戴上 3D 眼镜后，您可能想试验一下这个浮点值。

	您可以通过访问详细解释它的 [Wikipedia 页面](https://en.wikipedia.org/wiki/Anaglyph_3D)了解有关立体图的所有信息
	
### 设备方向相机
[DeviceOrientationCamera](https://doc.babylonjs.com/typedoc/classes/babylon.deviceorientationcamera) 专门设计用于对设备方向事件做出反应，例如现代移动设备向前、向后、向左或向右倾斜。

- 构建设备方向相机

		// 参数:名称、位置、场景
		var camera = new BABYLON.DeviceOrientationCamera("DevOr_camera", new BABYLON.Vector3(0, 0, 0), scene);
		
		// 将摄像机对准一个特定的位置
		camera.setTarget(new BABYLON.Vector3(0, 0, -10));
		
		// 设置相机移动和旋转的灵敏度
		camera.angularSensibility = 10;
		camera.moveSensibility = 10;
		
		//  将相机固定在画布上
		camera.attachControl(canvas, true);
- [设备相机案例](https://playground.babylonjs.com/#SRZRWV#3)

		var createScene = function () {
		
		    // 这样就创建了一个基本的场景对象(非网格)
		    var scene = new BABYLON.Scene(engine);
		
		   /********** 设备方向摄像机示例 **************************/
		
		    // 这将创建并定位一个设备定向摄像头
		    var camera = new BABYLON.DeviceOrientationCamera("DevOr_camera", new BABYLON.Vector3(0, 0, 0), scene);
		
		    // 这将相机指向场景源
		    camera.setTarget(new BABYLON.Vector3(0, 0, 10));
		
		    //  这样就把相机固定在画布上了
		    camera.attachControl(canvas, true);
		   /**************************************************************/
		
		    // 这创建了一个光，瞄准 0,1,0 天空(非网格)。
		    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
			
			// 材料
			var redMat = new BABYLON.StandardMaterial("red", scene);
			redMat.diffuseColor = new BABYLON.Color3(1, 0, 0);
			redMat.emissiveColor = new BABYLON.Color3(1, 0, 0);
			redMat.specularColor = new BABYLON.Color3(1, 0, 0);
			
			var greenMat = new BABYLON.StandardMaterial("green", scene);
			greenMat.diffuseColor = new BABYLON.Color3(0, 1, 0);
			greenMat.emissiveColor = new BABYLON.Color3(0, 1, 0);
			greenMat.specularColor = new BABYLON.Color3(0, 1, 0);
			
			var blueMat = new BABYLON.StandardMaterial("blue", scene);
			blueMat.diffuseColor = new BABYLON.Color3(0, 0, 1);
			blueMat.emissiveColor = new BABYLON.Color3(0, 0, 1);
			blueMat.specularColor = new BABYLON.Color3(0, 0, 1);
			
			// 形状
			var plane1 = new BABYLON.Mesh.CreatePlane("plane1", 3, scene, true, BABYLON.Mesh.DOUBLESIDE);
			plane1.position.x = -3;
			plane1.position.z = 0;
			plane1.material = redMat;
			
			var plane2 = new BABYLON.Mesh.CreatePlane("plane2", 3, scene, true, BABYLON.Mesh.DOUBLESIDE);
			plane2.position.x = 3;
			plane2.position.z = -1.5;
			plane2.material = greenMat;
			
			var plane3 = new BABYLON.Mesh.CreatePlane("plane3", 3, scene, true, BABYLON.Mesh.DOUBLESIDE);
			plane3.position.x = 3;
			plane3.position.z = 1.5;
			plane3.material = blueMat;
			
			var ground = BABYLON.Mesh.CreateGround("ground1", 10, 10, 2, scene);
			
		    return scene;
		
		};

### 虚拟操纵杆相机
`VirtualJoysticksCamera` 专门设计用于对 Virtual Joystick 事件做出反应。虚拟操纵杆是屏幕上的 2D 图形，用于控制相机或其他场景项目。

注意：本相机需要第三方文件 [hand.js](https://archive.codeplex.com/?p=handjs)。

- demo 视频

	需要翻墙点击 [这里](https://www.youtube.com/embed/53Piiy71lB0) 
- 完整样本

	这是一个完整的示例，它加载了 Espilit 演示并将默认摄像头切换到虚拟操纵杆摄像头
	
		document.addEventListener("DOMContentLoaded", startGame, false);
		function startGame() {
		  if (BABYLON.Engine.isSupported()) {
		    var canvas = document.getElementById("renderCanvas");
		    var engine = new BABYLON.Engine(canvas, true);
		
		    BABYLON.SceneLoader.Load("Espilit/", "Espilit.babylon", engine, function (newScene) {
		
		      var VJC = new BABYLON.VirtualJoysticksCamera("VJC", newScene.activeCamera.position, newScene);
		      VJC.rotation = newScene.activeCamera.rotation;
		      VJC.checkCollisions = newScene.activeCamera.checkCollisions;
		      VJC.applyGravity = newScene.activeCamera.applyGravity;
		
		      // 等待纹理和着色器准备好
		      newScene.executeWhenReady(function () {
		        newScene.activeCamera = VJC;
		        // 将相机连接到画布输入
		        newScene.activeCamera.attachControl(canvas);
		        // 一旦场景被加载，只需注册一个渲染循环来渲染它
		        engine.runRenderLoop(function () {
		          newScene.render();
		        }),
		      }),
		    }, function (progress) {
		    // 要做的事情:把进度反馈给用户。
		    }),
		  }
		}

如果您切换回另一台相机，请不要忘记先调用 `dispose()`。`VirtualJoysticks` 正在 3D WebGL 画布上创建一个 2D 画布，用青色和黄色圆圈绘制操纵杆。如果您忘记调用  `dispose()` ，则 2D 画布将保留并继续处理触摸事件。

### VR 设备定向相机
`VRDeviceOrientationFreeCamera` 、[VRDeviceOrientationArcRotateCamera](https://doc.babylonjs.com/typedoc/classes/babylon.vrdeviceorientationfreecamera)和 [VRDeviceOrientationGamepadCamera](https://doc.babylonjs.com/typedoc/classes/babylon.vrdeviceorientationarcrotatecamera) 是一组更新的相机，它们扩展了上面的相机以处理来自 VR 设备的设备方向。

- [示例（需要 VR 设备）](https://playground.babylonjs.com/#SRZRWV#4)：

		var createScene = function () {
		
		    // 这样就创建了一个基本的巴比伦场景对象(非网格)
		    var scene = new BABYLON.Scene(engine);
		
		    /********** 设备方向摄像机示例 **************************/
		
		    // 这将创建并定位一个设备定向摄像头	
		    var camera = new BABYLON.VRDeviceOrientationFreeCamera("DevOr_camera", new BABYLON.Vector3(0, 0, 0), scene);
		
		    // 这将相机指向场景源
		    camera.setTarget(new BABYLON.Vector3(0, 0, 10));
		
		    // 这样就把相机固定在画布上了
		    camera.attachControl(canvas, true);
		   /**************************************************************/
		
		    // 这创建了一个光，瞄准 0,1,0 天空(非网格)。
		    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
			
			//材质
			var redMat = new BABYLON.StandardMaterial("red", scene);
			redMat.diffuseColor = new BABYLON.Color3(1, 0, 0);
			redMat.emissiveColor = new BABYLON.Color3(1, 0, 0);
			redMat.specularColor = new BABYLON.Color3(1, 0, 0);
			
			var greenMat = new BABYLON.StandardMaterial("green", scene);
			greenMat.diffuseColor = new BABYLON.Color3(0, 1, 0);
			greenMat.emissiveColor = new BABYLON.Color3(0, 1, 0);
			greenMat.specularColor = new BABYLON.Color3(0, 1, 0);
			
			var blueMat = new BABYLON.StandardMaterial("blue", scene);
			blueMat.diffuseColor = new BABYLON.Color3(0, 0, 1);
			blueMat.emissiveColor = new BABYLON.Color3(0, 0, 1);
			blueMat.specularColor = new BABYLON.Color3(0, 0, 1);
			
			// 形状
			var plane1 = new BABYLON.Mesh.CreatePlane("plane1", 3, scene, true, BABYLON.Mesh.DOUBLESIDE);
			plane1.position.x = -3;
			plane1.position.z = 0;
			plane1.material = redMat;
			
			var plane2 = new BABYLON.Mesh.CreatePlane("plane2", 3, scene, true, BABYLON.Mesh.DOUBLESIDE);
			plane2.position.x = 3;
			plane2.position.z = -1.5;
			plane2.material = greenMat;
			
			var plane3 = new BABYLON.Mesh.CreatePlane("plane3", 3, scene, true, BABYLON.Mesh.DOUBLESIDE);
			plane3.position.x = 3;
			plane3.position.z = 1.5;
			plane3.material = blueMat;
			
			var ground = BABYLON.Mesh.CreateGround("ground1", 10, 10, 2, scene);
			
		    return scene;
		
		};
- 构建一个 VR 设备方向自由相机
	
		// 参数:名称，位置，场景，补偿失真，vrCameraMetrics
		var camera = new BABYLON.VRDeviceOrientationFreeCamera("Camera", new BABYLON.Vector3(-6.7, 1.2, -1.3), scene);
- 构建 VR 设备方向弧形旋转相机

		// 参数:名称，alpha, beta，半径，目标，场景，补偿失真，vrCameraMetrics
		var camera = new BABYLON.VRDeviceOrientationArcRotateCamera("Camera", Math.PI / 2, Math.PI / 4, 25, new BABYLON.Vector3(0, 0, 0), scene);
- 构建 VR 设备方向游戏手柄相机

		// 参数:名称，位置，场景，补偿失真，vrCameraMetrics
		var camera = new BABYLON.VRDeviceOrientationGamepadCamera("Camera", new BABYLON.Vector3(-10, 5, 14));		
### WebVR 免费摄像头
注意： WebVR 已弃用！请改用 WebXR

[WebVRFreeCamera](https://doc.babylonjs.com/typedoc/classes/babylon.webvrfreecamera) 是旧版虚拟现实相机，现在旨在与不支持 WebXR 标准的旧版 VR 设备浏览器一起使用。

	// 参数:名称、位置、场景、webVROptions
	var camera = new BABYLON.WebVRFreeCamera("WVR", new BABYLON.Vector3(0, 1, -15), scene);
该相机值得拥有自己的页面：使用 [WebVR 相机](https://doc.babylonjs.com/divingDeeper/cameras/webVRCamera)。

### 飞行相机
[FlyCamera](https://doc.babylonjs.com/typedoc/classes/babylon.flycamera) 模仿 3D 空间中的自由运动，想象“空间中的幽灵”。它带有一个逐渐纠正滚动的选项，以及一个模拟倾斜转弯的选项。

其默认值为：

- 键盘 -A和D键左右移动相机。W和键可S前后移动。和键E上下Q移动它。
- 鼠标 - 以相机为原点，围绕 Pitch 和 Yaw (X, Y) 轴旋转相机。按住鼠标右键可使相机以相机为原点围绕 Roll (Z) 轴旋转。

		
- 构建飞行相机

		// 参数:名称、位置、场景
		var camera = new BABYLON.FlyCamera("FlyCamera", new BABYLON.Vector3(0, 5, -10), scene);
		
		// 像飞机一样旋转，具有更快的滚转修正和倾斜转弯。
		// 默认是100。数字越高，修正速度越慢。
		camera.rollCorrect = 10;
		// 默认是假的。
		camera.bankedTurn = true;
		// 默认为90°弧度在银行将滚动相机的距离。
		camera.bankedTurnLimit = Math.PI / 2;
		// 偏航(转弯)对滚转(倾斜转弯)有多大影响?
		// 偏航(转弯)对滚转(倾斜转弯)有多大影响?
		camera.bankedTurnMultiplier = 1;
		
		//这样就把相机固定在画布上了
		camera.attachControl(canvas, true);
		
### 剪裁平面和无限透视
Babylon.js 中的相机具有指定要渲染的场景中的可视范围的[剪切平面](https://doc.babylonjs.com/divingDeeper/scene/clipPlanes)。超出该范围的任何内容都不会被渲染。例如，将相机的远裁剪平面设置为 100，如下所示：

	camera.maxZ = 100;
不会从相机渲染超过 100 个单位的任何东西。近剪裁平面也是如此。例如，将近剪裁平面设置为 10，如下所示：

	camera.minZ = 10;
不会渲染距离相机 10 个单位以内的任何东西。

在某些情况下，您可能不想剪辑场景的渲染。您可能希望您的场景有效地渲染到无限远。这可以通过将远裁剪平面设置为 0 来完成，如下所示：

	camera.maxZ = 0;
是一个友好的警告，将远裁剪平面设置为无穷大会降低深度精度，因此请谨慎使用！

### 自定义输入
相机依靠用户输入来移动相机。如果您对 Babylon.js 提供的相机预设感到满意，请坚持使用。

如果您想根据用户偏好更改用户输入，请自定义现有预设之一或使用自定义输入机制。这些摄像机具有专为这些高级场景设计的输入管理器。阅读[自定义相机输入](https://doc.babylonjs.com/divingDeeper/cameras/customizingCameraInputs)以了解有关调整相机输入的更多信息。
### 校正透视投影
如果您正在执行诸如建筑渲染之类的应用程序，您可能会遇到需要补偿垂直线的透视倾斜。让我们考虑这种情况：您正在从人眼的角度渲染一座高楼。自然地，垂直线会汇聚到一个消失点，就像在这个 [playground-demo](https://playground.babylonjs.com/#L20FJ4#15) 上：

![](./pic/tilted-vertical.jpeg)

	var createScene = function () {
	    // 这样就创建了一个基本的场景对象(非网格)
	    var scene = new BABYLON.Scene(engine);
	
	    // 这创建并定位了一个自由相机(非网格)
	    var camera = new BABYLON.FreeCamera("camera1", new BABYLON.Vector3(0, 0, -10), scene);
	
	    // 这将相机指向场景源
	    camera.setTarget(new BABYLON.Vector3(0, 2, 0));
	
	    console.log("vertical angle : " + BABYLON.Tools.ToDegrees(camera.rotation.x));
	    // 这样就把相机固定在画布上了
	    camera.attachControl(canvas, true);
	    
	    // 这创建了一个光，瞄准 0,1,0 天空(非网格。
	    var light = new BABYLON.DirectionalLight("light", new BABYLON.Vector3(0.5, 0, 1), scene);
	
	    // 默认强度为1。让我们把灯光调暗一点
	    light.intensity = 0.7;
	
	    // 我们内置的“球体”形状。
	    var box = BABYLON.MeshBuilder.CreateBox("Box", {width: 1, height: 5}, scene);
	
	    box.position.y = 2.5;
	    box.position.x = -2;
	
	    var box1 = BABYLON.MeshBuilder.CreateBox("Box", {width: 1, height: 5}, scene);
	
	    box1.position.y = 2.5;
	    box1.position.x = 2;
	
	    var mat = new BABYLON.StandardMaterial("mat", scene);
	    box.material = mat;
	    box1.material = mat;
	
	    // 我们内置的“地面”形状。
	    var ground = BABYLON.MeshBuilder.CreateGround("ground", {width: 6, height: 6}, scene);
	
	    // GUI
	    var advancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI("UI");
	
	    var panel = new BABYLON.GUI.StackPanel();
	    panel.width = "220px";
	    panel.horizontalAlignment = BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_RIGHT;
	    panel.verticalAlignment = BABYLON.GUI.Control.VERTICAL_ALIGNMENT_CENTER;
	    advancedTexture.addControl(panel);
	
	    var header = new BABYLON.GUI.TextBlock();
	    header.text = "tilt: " + (BABYLON.Tools.ToDegrees(camera.projectionPlaneTilt) | 0) + " deg";
	    header.height = "30px";
	    header.color = "white";
	    panel.addControl(header); 
	
	    var slider = new BABYLON.GUI.Slider();
	    slider.minimum = - Math.PI / 2;
	    slider.maximum = Math.PI / 2;
	    slider.value = camera.projectionPlaneTilt;
	    slider.height = "20px";
	    slider.width = "200px";
	    slider.onValueChangedObservable.add(function(value) {
	        header.text = "tilt: " + (BABYLON.Tools.ToDegrees(value) | 0) + " deg";
	        if (camera) {
	            camera.projectionPlaneTilt = value;
	        }
	    });
	    panel.addControl(slider);
	
	    var button = BABYLON.GUI.Button.CreateSimpleButton("but", "Apply Vertical Correction");
	    button.width = 1.2;
	    button.height = "40px";
	    button.color = "white";
	    button.background = "green";
	    button.paddingTop = "10px";
	    button.onPointerClickObservable.add(() => {
	        if (camera) {
	            camera.applyVerticalCorrection();
	            slider.value = camera.projectionPlaneTilt;
	            header.text = "tilt: " + (BABYLON.Tools.ToDegrees(camera.projectionPlaneTilt) | 0) + " deg";
	        }
	    })
	    panel.addControl(button);    
	
	    // 添加参考网格
	    const gridCanvas = document.createElement("canvas");
	    const parent = document.getElementById("canvasZone");
	    parent.appendChild(gridCanvas);
	    parent.style.position = "relative";
	    gridCanvas.style.position = "absolute";
	    gridCanvas.style.width = "100%";
	    gridCanvas.style.height = "100%";
	    gridCanvas.style.top = "0px";
	    gridCanvas.style.left = "0px";
	    gridCanvas.style.pointerEvents = "none";
	    
	    var w = gridCanvas.clientWidth;
	    var h = gridCanvas.clientHeight;
	    
	    var ctx = gridCanvas.getContext('2d');
	    ctx.canvas.width  = w;
	    ctx.canvas.height = h;
	
	    for (x=0;x<=w;x+=20) {
	        for (y=0;y<=h;y+=20) {
	            ctx.moveTo(x, 0);
	            ctx.lineTo(x, h);
	            ctx.stroke();
	            ctx.moveTo(0, y);
	            ctx.lineTo(w, y);
	            ctx.stroke();
	        }
	    }
	
	    return scene;
	};	

虽然这是现实的，但它可能在视觉上没有吸引力。如果这些线之间的角度保持很低，则考虑使用 校正透视校正可能会很有趣 `camera.applyVerticalCorrection()`。此方法将自动计算垂直校正以应用于当前摄像机俯仰角：

![](./pic/corrected-vertical.jpeg)

如果你想进一步控制其他相机投影平面的倾斜，你可以弄乱这个 `camera.projectionPlaneTilt` 属性。有关更多信息，请参阅此[论坛帖子](https://forum.babylonjs.com/t/add-vertical-shift-to-3ds-max-exporter-babylon-cameras/17480/16)。

	
	
					









