# babylon 官方深度教学-相机-自定义相机输入
## 如何自定义相机输入
一旦您调用相机的 [attachControl](https://doc.babylonjs.com/typedoc/classes/babylon.freecamera#attachcontrol) 方法，每个 Babylon.js 相机都会自动为您处理输入。您可以使用 [detachControl](https://doc.babylonjs.com/typedoc/classes/babylon.freecamera#detachcontrol) 方法撤销控制。大多数 Babylon.js 专家使用两步过程来激活和附加摄像头：

	// 首先，将场景的 activeCamera... 设置为您的相机。
	scene.activeCamera = myCamera;
	
	// 然后将 activeCamera 附加到画布上。参数：canvas, noPreventDefault
	scene.activeCamera.attachControl(canvas, true);
更简单的版本可能如下所示：

	myCamera.attachControl(canvas);
默认 `noPreventDefault` 设置为 `false`，这意味着 `preventDefault()` 在所有画布鼠标点击和触摸事件上都会自动调用它。

Babylon.js v2.4 引入了一种不同的方法来管理相机输入，以提供一种面向输入可组合性的方法。您现在可以使用输入管理器，每个输入都可以看作是特定于该相机系列和给定输入类型（鼠标、键盘、游戏手柄、设备方向等）的插件。

使用输入管理器，您可以添加、删除、启用或禁用相机可用的任何输入。您还可以轻松实现自定义输入机制或覆盖现有的。

输入管理器可通过相机的[输入](https://doc.babylonjs.com/typedoc/classes/babylon.freecamera#inputs)属性获得。例如：

	var camera = new BABYLON.FreeCamera("sceneCamera", new BABYLON.Vector3(0, 1, -15), scene);
	var inputManager = camera.inputs;
## 配置输入
大多数输入提供设置以自定义灵敏度并使其适应您自己的场景。

每个输入都提供了管理器上可用的短名称。目标是在使用您的输入时提供友好的语法。

	var camera = new BABYLON.FreeCamera("sceneCamera", new BABYLON.Vector3(0, 1, -15), scene);
	camera.inputs.add(new BABYLON.FreeCameraGamepadInput());
	camera.inputs.attached.gamepad.gamepadAngularSensibility = 250;
## 添加现有输入
输入管理器 `ArcRotateCamera` 和 `FreeCamera` 公开用于添加内置输入的速记函数

	var camera = new BABYLON.FreeCamera("sceneCamera", new BABYLON.Vector3(0, 1, -15), scene);
	camera.inputs.addGamepad();
如果愿意，还可以添加您自己的输入的实例（我们将在本文末尾介绍如何实现您自己的输入）。

	var camera = new BABYLON.FreeCamera("sceneCamera", new BABYLON.Vector3(0, 1, -15), scene);
	camera.inputs.add(new BABYLON.FreeCameraGamepadInput());
## 启用或禁用输入
当您调用 `attachControl` 相机时，您正在激活附加到输入管理器的所有输入。`detachControl` 同样，您可以通过调用相机来关闭所有输入。

如果你想暂时禁用一个输入，你可以 `detachControl` 直接在输入上调用......像这样：

	var camera = new BABYLON.FreeCamera("sceneCamera", new BABYLON.Vector3(0, 1, -15), scene);
	camera.inputs.attached.mouse.detachControl();
	camera.inputs.addGamepad();
`attachInput` 然后，当您想再次打开它时

	camera.inputs.attachInput(camera.inputs.attached.mouse);
## 删除输入
有时您需要一个非常具体的输入机制。在这种情况下，最好的方法可能是清除所有输入并仅在场景中添加您可能需要的输入

	var camera = new BABYLON.FreeCamera("sceneCamera", new BABYLON.Vector3(0, 1, -15), scene);
	camera.inputs.clear();
	camera.inputs.addMouse();
您还可以从输入管理器中删除单个输入。您可以按实例或按类型名称删除它们

	var camera = new BABYLON.FreeCamera("sceneCamera", new BABYLON.Vector3(0, 1, -15), scene);
	// 按实例移除
	camera.inputs.remove(camera.inputs.attached.mouse);
	// 按类型移除
	camera.inputs.removeByType("FreeCameraKeyboardMoveInput");
## 实现你自己的输入
您的输入法被创建为一个函数对象。然后，您必须为输入函数对象调用的具有必需名称的多个方法编写代码。方法名称和用途是：

	// 这个函数必须返回相机的类名，它可以用来序列化你的场景
	getClassName();
	
	// 这个函数必须返回一个简单的名称，这个名称将被注入到输入管理器中，例如 "mouse" 将变成camera.input.attached.mouse
	getSimpleName();
	
	// 他的函数必须激活你的输入事件。即使您的输入不需要 DOM 元素元素和 noPreventDefault 必须出现并用作参数名。
	// 返回 void。
	attachControl(element, noPreventDefault);
	
	// 分离控件必须停用输入，并释放所有指针、闭包或事件监听器元素，这些元素必须作为参数名出现。
	// 返回 void。
	detachControl(element);
	
	// 这个可选的函数会被每个渲染帧调用，如果你想同步你的输入到渲染，不需要使用 requestAnimationFrame。如果需要的话，这是一个应用计算的好地方。
	// 返回 void。
	checkInputs();

## HammerJS 输入
如果 `ArcRotateCamera` 的内置触摸控件对您来说不够用，您可以使用著名的触摸手势库 `HammerJS` 实现自己的相机输入。

[https://hammerjs.github.io/](https://hammerjs.github.io/)

我们有一个如何使用 HammerJS 模拟类似于 Google Earth 控件的示例。为此，我们使用 BabylonJS 的 `ArcRotateCamera`。摄像机锁定在 Y 轴上，水平平移沿 X 轴移动摄像机，垂直平移沿 Z 轴移动摄像机。alpha 平移时会考虑相机的角度。通过捏合您可以更改 `radius` 相机的位置，以便您可以放大和缩小。您可以通过两指旋转手势更改 `alpha` 相机的角度，因此基本上您可以围绕相机目标旋转。

示例应用： [https ://demos.babylonjs.xyz/hammerjs-example/#/](https://demos.babylonjs.xyz/hammerjs-example/#/)

GitHub 仓库： [https ://github.com/RolandCsibrei/babylonjs-hammerjs-arc-rotate-camera](https://github.com/RolandCsibrei/babylonjs-hammerjs-arc-rotate-camera)

该示例包含 `utils\ArcRotateCameraHammerJsInput.ts` 哪些实现 `ICameraInput<ArcRotateCamera>`。输入有多个参数，并且非常不言自明。请参考输入源代码并设置适合您需要的灵敏度和阈值。首先需要安装 HammerJS，请参考 [https://hammerjs.github.io/getting-started/](https://hammerjs.github.io/getting-started/)并导入

	import "hammerjs";
`camera.inputs` 要使用新输入，请在创建相机后将其添加到您的输入中。为避免一个输入与另一个输入冲突，请删除`ArcRotateCameraPointersInputfrom camera.inputs` 。创建输入后，您可以设置它的参数。默认值（请参阅 [https://github.com/RolandCsibrei/babylonjs-hammerjs-arc-rotate-camera/blob/680cf12155924a818faac5ff9d7f0a0271bb632b/src/utils/ArcRotateCameraHammerJsInput.ts#L21](https://github.com/RolandCsibrei/babylonjs-hammerjs-arc-rotate-camera/blob/680cf12155924a818faac5ff9d7f0a0271bb632b/src/utils/ArcRotateCameraHammerJsInput.ts#L21)）适用于一般的触摸屏显示器，因此您可能必须根据需要设置它们。

    // 删除鼠标输入
    camera.inputs.removeByType('ArcRotateCameraPointersInput')

    // 添加锤子js输入
    const hammerJsInput = new ArcRotateCameraHammerJsInput()
    
    // 现在你可以设置你喜欢的参数，让我们将 zoomSensitivity 加倍(默认为1)
    hammerJsInput.zoomSensitivity = 2
    // 将输入添加到相机
    camera.inputs.add(hammerJsInput)
随意使用这个 Input 类作为您自己的基于 HammerJS 的输入的启动器
## 使用 Javascript
这将更改用于向左、向右、向前和向后移动相机以及在其当前位置旋转的法线键映射。

首先，删除默认的键盘输入

	camera.inputs.removeByType("FreeCameraKeyboardMoveInput");
现在创建新的输入法 `FreeCameraKeyboardRotateInput`

	var FreeCameraKeyboardRotateInput = function () {
	  this._keys = [];
	  this.keysLeft = [37];
	  this.keysRight = [39];
	  this.sensibility = 0.01;
	};
添加获取名称方法：

	FreeCameraKeyboardRotateInput.prototype.getClassName = function () {
	  return "FreeCameraKeyboardRotateInput";
	};
	FreeCameraKeyboardRotateInput.prototype.getSimpleName = function () {
	  return "keyboardRotate";
	};
以及附加和分离方法：

	FreeCameraKeyboardRotateInput.prototype.attachControl = function (noPreventDefault) {
	  var _this = this;
	  var engine = this.camera.getEngine();
	  var element = engine.getInputElement();
	  if (!this._onKeyDown) {
	    element.tabIndex = 1;
	    this._onKeyDown = function (evt) {
	      if (_this.keysLeft.indexOf(evt.keyCode) !== -1 || _this.keysRight.indexOf(evt.keyCode) !== -1) {
	        var index = _this._keys.indexOf(evt.keyCode);
	        if (index === -1) {
	          _this._keys.push(evt.keyCode);
	        }
	        if (!noPreventDefault) {
	          evt.preventDefault();
	        }
	      }
	    };
	    this._onKeyUp = function (evt) {
	      if (_this.keysLeft.indexOf(evt.keyCode) !== -1 || _this.keysRight.indexOf(evt.keyCode) !== -1) {
	        var index = _this._keys.indexOf(evt.keyCode);
	        if (index >= 0) {
	          _this._keys.splice(index, 1);
	        }
	        if (!noPreventDefault) {
	          evt.preventDefault();
	        }
	      }
	    };
	
	    element.addEventListener("keydown", this._onKeyDown, false);
	    element.addEventListener("keyup", this._onKeyUp, false);
	    BABYLON.Tools.RegisterTopRootEvents(canvas, [{ name: "blur", handler: this._onLostFocus }]);
	  }
	};
	
	FreeCameraKeyboardRotateInput.prototype.detachControl = function () {
	  var engine = this.camera.getEngine();
	  var element = engine.getInputElement();
	  if (this._onKeyDown) {
	    element.removeEventListener("keydown", this._onKeyDown);
	    element.removeEventListener("keyup", this._onKeyUp);
	    BABYLON.Tools.UnregisterTopRootEvents(canvas, [{ name: "blur", handler: this._onLostFocus }]);
	    this._keys = [];
	    this._onKeyDown = null;
	    this._onKeyUp = null;
	  }
	};
（可选）添加检查输入

	FreeCameraKeyboardRotateInput.prototype.checkInputs = function () {
	  if (this._onKeyDown) {
	    var camera = this.camera;
	    // Keyboard
	    for (var index = 0; index < this._keys.length; index++) {
	      var keyCode = this._keys[index];
	      if (this.keysLeft.indexOf(keyCode) !== -1) {
	        camera.cameraRotation.y += this.sensibility;
	      } else if (this.keysRight.indexOf(keyCode) !== -1) {
	        camera.cameraRotation.y -= this.sensibility;
	      }
	    }
	  }
	};
最后，将这个新的输入法添加到相机输入中：

	camera.inputs.add(new FreeCameraKeyboardRotateInput());

[Rotate Free Camera 例子](https://playground.babylonjs.com/#KHQBRL)


	var createScene = function () {
	
	    // 这样就创建了一个基本的场景对象(非网格)
	    var scene = new BABYLON.Scene(engine);
	
	    // 这创建并定位了一个自由相机(非网格)
	    var camera = new BABYLON.FreeCamera("camera1", new BABYLON.Vector3(0, 5, -10), scene);
	
	    // 这将相机指向场景源
	    camera.setTarget(BABYLON.Vector3.Zero());
	
	    // 这样就把相机固定在画布上了
	    camera.attachControl(canvas, true);
	
	    // 这创建了一个光，瞄准 0,1,0 天空(非网格)。
	    var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
	
	    // 默认强度为1。让我们把灯光调暗一点
	    light.intensity = 0.7;
	
	    // 我们内置的“球体”形状。参数:名称，细分，大小，场景
	    var sphere = BABYLON.Mesh.CreateSphere("sphere1", 16, 2, scene);
	
	    // 将球体向上移动1/2高度
	    sphere.position.y = 1;
	
	    // 我们内置的“地面”形状。参数:名称，宽度，深度，细分，场景
	    var ground = BABYLON.Mesh.CreateGround("ground1", 6, 6, 2, scene);
	
	    // 让我们移除默认键盘:
	    camera.inputs.removeByType("FreeCameraKeyboardMoveInput");
	
	    // 创建我们自己的键盘:
	    var FreeCameraKeyboardRotateInput = function () {
	            this._keys = [];
	            this.keysLeft = [37];
	            this.keysRight = [39];
	            this.sensibility = 0.01;
	    }
	
	    // 连接键盘事件
	    FreeCameraKeyboardRotateInput.prototype.attachControl = function (noPreventDefault) {
	        var _this = this;
	        var engine = this.camera.getEngine();
	            var element = engine.getInputElement();
	            
	        if (!this._onKeyDown) {
	            element.tabIndex = 1;
	            this._onKeyDown = function (evt) {
	                if (_this.keysLeft.indexOf(evt.keyCode) !== -1 ||
	                    _this.keysRight.indexOf(evt.keyCode) !== -1) {
	                    var index = _this._keys.indexOf(evt.keyCode);
	                    if (index === -1) {
	                        _this._keys.push(evt.keyCode);
	                    }
	                    if (!noPreventDefault) {
	                        evt.preventDefault();
	                    }
	                }
	            };
	            
	            this._onKeyUp = function (evt) {
	                if (_this.keysLeft.indexOf(evt.keyCode) !== -1 ||
	                    _this.keysRight.indexOf(evt.keyCode) !== -1) {
	                    var index = _this._keys.indexOf(evt.keyCode);
	                    if (index >= 0) {
	                        _this._keys.splice(index, 1);
	                    }
	                    if (!noPreventDefault) {
	                        evt.preventDefault();
	                    }
	                }
	            };
	
	            element.addEventListener("keydown", this._onKeyDown, false);
	            element.addEventListener("keyup", this._onKeyUp, false);
	            BABYLON.Tools.RegisterTopRootEvents(canvas, [
	                { name: "blur", handler: this._onLostFocus }
	            ]);
	        }
	    };
	
	    // 解锁
	    FreeCameraKeyboardRotateInput.prototype.detachControl = function () {
	        if (this._onKeyDown) {
	            var engine = this.camera.getEngine();
	            var element = engine.getInputElement();
	            element.removeEventListener("keydown", this._onKeyDown);
	            element.removeEventListener("keyup", this._onKeyUp);
	            BABYLON.Tools.UnregisterTopRootEvents(canvas, [
	                { name: "blur", handler: this._onLostFocus }
	            ]);
	            this._keys = [];
	            this._onKeyDown = null;
	            this._onKeyUp = null;
	        }
	    };
	
	    // 这个函数由系统在每一帧上调用
	    FreeCameraKeyboardRotateInput.prototype.checkInputs = function () {
	        if (this._onKeyDown) {
	            var camera = this.camera;
	            // 键盘
	            for (var index = 0; index < this._keys.length; index++) {
	                var keyCode = this._keys[index];
	                if (this.keysLeft.indexOf(keyCode) !== -1) {
	                    camera.cameraRotation.y += this.sensibility;
	                }
	                else if (this.keysRight.indexOf(keyCode) !== -1) {
	                    camera.cameraRotation.y -= this.sensibility;
	                }
	            }
	        }
	    };
	    FreeCameraKeyboardRotateInput.prototype.getTypeName = function () {
	        return "FreeCameraKeyboardRotateInput";
	    };
	    FreeCameraKeyboardRotateInput.prototype._onLostFocus = function (e) {
	        this._keys = [];
	    };
	    FreeCameraKeyboardRotateInput.prototype.getSimpleName = function () {
	        return "keyboardRotate";
	    };
	
	    // 操控设置链接到相机:
	    camera.inputs.add(new FreeCameraKeyboardRotateInput());
	
	    return scene;
	
	};

## 使用 TS
使用 TypeScript，您可以实现接口 `ICameraInput`：

	interface ICameraInput<TCamera extends BABYLON.Camera> {
	  // 输入管理器将填充父摄像机
	  camera: TCamera;
	
	  // 这个函数必须返回相机的类名，它可以用于序列化场景
	  getClassName(): string;
	
	  // 这个函数必须返回输入管理器中注入的简单名称，例如“mouse”将变成 camera.input .attached.mouse
	  getSimpleName(): string;
	
	  // 如果你的输入不需要DOM元素，这个函数必须激活你的输入事件
	  attachControl: (element: HTMLElement, noPreventDefault?: boolean) => void;
	
	  // 分离控件必须停用输入并释放所有指针、闭包或事件监听器
	  detachControl: (element: HTMLElement) => void;
	
	  // 这个可选的函数将被调用为每个渲染帧，如果你想同步你的输入到渲染，不需要使用 requestAnimationFrame。如果需要的话，这是一个应用计算的好地方
	  checkInputs?: () => void;

## 如何散步并环顾四周
以下示例自定义通用相机的键盘和鼠标输入。通过此更改，您可以使用箭头键在场景中向前和向后走动并旋转以向左和向右看。使用鼠标，您可以环顾四周以及上方和下方。

在示例中，有两个视口，上面的一个在您移动和环顾四周时提供第一人称视图。下一个给出了相机和它周围的碰撞体积的表示。

请记住在使用箭头键之前单击场景

[Walk and Look Camera 例子](https://playground.babylonjs.com/#CTCSWQ#945)

	var createScene = function () {
	    var scene = new BABYLON.Scene(engine);
	
	    var light = new BABYLON.DirectionalLight("Omni", new BABYLON.Vector3(-2, -5, 2), scene);
	
	    // 添加相机，显示为一个圆锥和周围的碰撞体积
	    var camera = new BABYLON.UniversalCamera("MyCamera", new BABYLON.Vector3(0, 1, 0), scene);
	    camera.minZ = 0.0001;
	    camera.attachControl(canvas, true);
	    camera.speed = 0.02;
	    camera.angularSpeed = 0.05;
	    camera.angle = Math.PI/2;
	    camera.direction = new BABYLON.Vector3(Math.cos(camera.angle), 0, Math.sin(camera.angle));
	    
	    // 添加viewCamera，提供第一人称射击视图
	    var viewCamera = new BABYLON.UniversalCamera("viewCamera", new BABYLON.Vector3(0, 3, -3), scene);
	    viewCamera.parent = camera;
	    viewCamera.setTarget(new BABYLON.Vector3(0, -0.0001, 1));
	    
	    // 激活这两个相机
	    scene.activeCameras.push(viewCamera);
	    scene.activeCameras.push(camera);
	
	    // 添加两个视窗
	    camera.viewport = new BABYLON.Viewport(0, 0.5, 1.0, 0.5);
	    viewCamera.viewport = new BABYLON.Viewport(0, 0, 1.0, 0.5);  
	    
	    // 创建锥状形状的假相机
	    var cone = BABYLON.MeshBuilder.CreateCylinder("dummyCamera", {diameterTop:0.01, diameterBottom:0.2, height:0.2}, scene);
	    cone.parent = camera;
	    cone.rotation.x = Math.PI/2;
	
	    /* 建立了风景 */
	    // 地面设置
	    var ground = BABYLON.MeshBuilder.CreateGround("ground", {width: 20, height: 20}, scene);
	    // 地面材质
	    ground.material = new BABYLON.StandardMaterial("groundMat", scene);
	    ground.material.diffuseColor = new BABYLON.Color3(1, 1, 1);
	    ground.material.backFaceCulling = false;
	
	    // clone domain，设置大小和位置
	    var lowerGround = ground.clone("lowerGround");
	    lowerGround.scaling.x = 4;
	    lowerGround.scaling.z = 4;
	    lowerGround.position.y = -16;
	    // 设置材质
	    lowerGround.material = ground.material.clone("lowerMat");
	    lowerGround.material.diffuseColor = new BABYLON.Color3(0, 1, 0);
	
	    // 设置随机数
	    var randomNumber = function (min, max) {
			if (min == max) {
				return (min);
			}
			var random = Math.random();
			return ((random * (max - min)) + min);
		};
	
	    // 创建盒子	
	    var box = new BABYLON.MeshBuilder.CreateBox("crate", {size: 2}, scene);
	    box.material = new BABYLON.StandardMaterial("Mat", scene);
	    box.material.diffuseTexture = new BABYLON.Texture("textures/crate.png", scene);
	    // 设置碰撞
	    box.checkCollisions = true;
	    
	    // 设置参数
	    var boxNb = 6;
	    var theta = 0;
	    var radius = 6;
	    
	    // 盒子的位置
	    box.position = new BABYLON.Vector3((radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.cos(theta + randomNumber(-0.1 * theta, 0.1 * theta)), 1, (radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.sin(theta + randomNumber(-0.1 * theta, 0.1 * theta)));
          
          // 设置克隆盒子	
	    var boxes = [box];
	    for (var i = 1; i < boxNb; i++) {
	        theta += 2 * Math.PI / boxNb;
	        var newBox = box.clone("box" + i);
	        boxes.push(newBox);
	        
	        // 盒子的新位置
	        newBox.position = new BABYLON.Vector3((radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.cos(theta + randomNumber(-0.1 * theta, 0.1 * theta)), 1, (radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.sin(theta + randomNumber(-0.1 * theta, 0.1 * theta)));
	    }
	    /* 场景创建完毕 */
	
	    //重力和碰撞
	    // 场景重力和碰撞
	    scene.gravity = new BABYLON.Vector3(0, -0.9, 0);
	    scene.collisionsEnabled = true;
	
	    // 摄像机重力和碰撞
	    camera.checkCollisions = true;
	    camera.applyGravity = true;
	    
	    // 场地碰撞
	    ground.checkCollisions = true;
	    lowerGround.checkCollisions = true;
	
	    // 摄像机碰撞体设置	
	    camera.ellipsoid = new BABYLON.Vector3(0.5, 1, 0.5);
	    camera.ellipsoidOffset = new BABYLON.Vector3(0, 1, 0); 
	
	    // 创建可见椭球周围的相机
	    var a = 0.5;
	    var b = 1;
	    var points = [];
	    for(var theta = -Math.PI/2; theta < Math.PI/2; theta += Math.PI/36) {
	        points.push(new BABYLON.Vector3(0, b * Math.sin(theta), a * Math.cos(theta)));
	    }
	
	    var ellipse = [];
	    ellipse[0] = BABYLON.MeshBuilder.CreateLines("e", {points:points}, scene);
	    ellipse[0].color = BABYLON.Color3.Red();
	    ellipse[0].parent = camera;
	    ellipse[0].rotation.y = 5 * Math.PI/ 16;
	    for(var i = 1; i < 23; i++) {
	            ellipse[i] = ellipse[0].clone("el" + i);
	            ellipse[i].parent = camera;
	            ellipse[i].rotation.y = 5 * Math.PI/ 16 + i * Math.PI/16;
	    }
	    
	    
	    /* 新的输入管理相机*/
	    
	    // 首先删除默认管理
	    camera.inputs.removeByType("FreeCameraKeyboardMoveInput");
	    camera.inputs.removeByType("FreeCameraMouseInput");
	     
	    /* 键盘控制输入相关 */ 
	    // 键输入管理器使用键向前和向后移动和向左或向右看
	    var FreeCameraKeyboardWalkInput = function () {
	        this._keys = [];
	        this.keysUp = [38];
	        this.keysDown = [40];
	        this.keysLeft = [37];
	        this.keysRight = [39];
	    }
	    
	    // 添加键盘控件
	    FreeCameraKeyboardWalkInput.prototype.attachControl = function (noPreventDefault) {
	            var _this = this;
	            var engine = this.camera.getEngine();
	            var element = engine.getInputElement();
	            if (!this._onKeyDown) {
	                element.tabIndex = 1;
	                this._onKeyDown = function (evt) {                 
	                    if (_this.keysUp.indexOf(evt.keyCode) !== -1 ||
	                        _this.keysDown.indexOf(evt.keyCode) !== -1 ||
	                        _this.keysLeft.indexOf(evt.keyCode) !== -1 ||
	                        _this.keysRight.indexOf(evt.keyCode) !== -1) {
	                        var index = _this._keys.indexOf(evt.keyCode);
	                        if (index === -1) {
	                            _this._keys.push(evt.keyCode);
	                        }
	                        if (!noPreventDefault) {
	                            evt.preventDefault();
	                        }
	                    }
	                };
	                this._onKeyUp = function (evt) {
	                    if (_this.keysUp.indexOf(evt.keyCode) !== -1 ||
	                        _this.keysDown.indexOf(evt.keyCode) !== -1 ||
	                        _this.keysLeft.indexOf(evt.keyCode) !== -1 ||
	                        _this.keysRight.indexOf(evt.keyCode) !== -1) {
	                        var index = _this._keys.indexOf(evt.keyCode);
	                        if (index >= 0) {
	                            _this._keys.splice(index, 1);
	                        }
	                        if (!noPreventDefault) {
	                            evt.preventDefault();
	                        }
	                    }
	                };
	                element.addEventListener("keydown", this._onKeyDown, false);
	                element.addEventListener("keyup", this._onKeyUp, false);
	            }
	        };
	
	
	        // 添加键盘分离控制
	        FreeCameraKeyboardWalkInput.prototype.detachControl = function () {
	            var engine = this.camera.getEngine();
	            var element = engine.getInputElement();
	            if (this._onKeyDown) {
	                element.removeEventListener("keydown", this._onKeyDown);
	                element.removeEventListener("keyup", this._onKeyUp);
	                BABYLON.Tools.UnregisterTopRootEvents(canvas, [
	                    { name: "blur", handler: this._onLostFocus }
	                ]);
	                this._keys = [];
	                this._onKeyDown = null;
	                this._onKeyUp = null;
	            }
	        };
	
	        // 键盘运动控制通过检查输入
	        FreeCameraKeyboardWalkInput.prototype.checkInputs = function () {
	            if (this._onKeyDown) {
	                var camera = this.camera;
	                for (var index = 0; index < this._keys.length; index++) {
	                    var keyCode = this._keys[index];
	                    var speed = camera.speed;
	                    if (this.keysLeft.indexOf(keyCode) !== -1) {
	                        camera.rotation.y -= camera.angularSpeed;
	                        camera.direction.copyFromFloats(0, 0, 0);                
	                    }
	                    else if (this.keysUp.indexOf(keyCode) !== -1) {
	                        camera.direction.copyFromFloats(0, 0, speed);               
	                    }
	                    else if (this.keysRight.indexOf(keyCode) !== -1) {
	                        camera.rotation.y += camera.angularSpeed;
	                        camera.direction.copyFromFloats(0, 0, 0);
	                    }
	                    else if (this.keysDown.indexOf(keyCode) !== -1) {
	                        camera.direction.copyFromFloats(0, 0, -speed);
	                    }
	                    if (camera.getScene().useRightHandedSystem) {
	                        camera.direction.z *= -1;
	                    }
	                    camera.getViewMatrix().invertToRef(camera._cameraTransformMatrix);
	                    BABYLON.Vector3.TransformNormalToRef(camera.direction, camera._cameraTransformMatrix, camera._transformedDirection);
	                    camera.cameraDirection.addInPlace(camera._transformedDirection);
	                }
	            }
	        };
	
	        // 添加 onLostFocus 函数
	        FreeCameraKeyboardWalkInput.prototype._onLostFocus = function (e) {
	            this._keys = [];
	        };
	        
	        // 为控件Name添加两个必需的函数
	        FreeCameraKeyboardWalkInput.prototype.getClassName = function () {
	            return "FreeCameraKeyboardWalkInput";
	        };
	
	        FreeCameraKeyboardWalkInput.prototype.getSimpleName = function () {
	            return "keyboard";
	        };
	    
	    // 向相机添加新的密钥输入管理器。
	     camera.inputs.add(new FreeCameraKeyboardWalkInput());
	 
	  /* 键盘控制输入相关结束 */ 
	
	
	  /* 鼠标和触摸控制输入相关 */ 
	  
	    // 创建鼠标管理器，使用鼠标(触摸)搜索周围包括上面和下面
	    var FreeCameraSearchInput = function (touchEnabled) {
	        // 如果触控开放
	        if (touchEnabled === void 0) { touchEnabled = true; }
		        this.touchEnabled = touchEnabled;
		        // ？
		        this.buttons = [0, 1, 2];
		        // 鼠标输入的敏感度，默认 2000,越小灵敏度越高
		        this.angularSensibility = 2000.0;
		        // 限制？？
		        this.restrictionX = 100;
		        this.restrictionY = 60;
	    }
	
	    // 添加附件控件，该控件还包含响应鼠标输入的代码
	    FreeCameraSearchInput.prototype.attachControl = function (noPreventDefault) {
	        var _this = this;
	        // 创建渲染引擎
	        var engine = this.camera.getEngine();
	        // 创建 element
	        var element = engine.getInputElement();
	        // 设置角
	        var angle = {x:0, y:0};
	        
	        if (!this._pointerInput) {
	            this._pointerInput = function (p, s) {
	                var evt = p.event;
	                // 排除多种非交互逻辑
	                if (!_this.touchEnabled && evt.pointerType === "touch") {
	                    return;
	                }
	                if (p.type !== BABYLON.PointerEventTypes.POINTERMOVE && _this.buttons.indexOf(evt.button) === -1) {          
	                    return;
	                }
	                
	                // 如果捕捉事件为按下
	                if (p.type === BABYLON.PointerEventTypes.POINTERDOWN) {          
	                    try {
	                        evt.srcElement.setPointerCapture(evt.pointerId);
	                    }
	                    catch (e) {
	                        // 与错误无关。将继续执行。
	                    }
	                    _this.previousPosition = {
	                        x: evt.clientX,
	                        y: evt.clientY
	                    };
	                    if (!noPreventDefault) {
	                        evt.preventDefault();
	                        element.focus();
	                    }
	                }
	                
	                // 如果捕捉事件为抬起
	                else if (p.type === BABYLON.PointerEventTypes.POINTERUP) {          
	                    try {
	                        evt.srcElement.releasePointerCapture(evt.pointerId);
	                    }
	                    catch (e) {
	                        // 与错误无关。
	                    }
	                    _this.previousPosition = null;
	                    if (!noPreventDefault) {
	                        evt.preventDefault();
	                    }
	                }
	                
	                // 如果捕捉事件是移动
	                else if (p.type === BABYLON.PointerEventTypes.POINTERMOVE) {            
	                    if (!_this.previousPosition || engine.isPointerLock) {
	                        return;
	                    }
	                    var offsetX = evt.clientX - _this.previousPosition.x;
	                    var offsetY = evt.clientY - _this.previousPosition.y;                   
	                    angle.x +=offsetX;
	                    angle.y -=offsetY;  
	                    if(Math.abs(angle.x) > _this.restrictionX )  {
	                        angle.x -=offsetX;
	                    }
	                    if(Math.abs(angle.y) > _this.restrictionY )  {
	                        angle.y +=offsetY;
	                    }       
	                    if (_this.camera.getScene().useRightHandedSystem) {
	                        if(Math.abs(angle.x) < _this.restrictionX )  {
	                            _this.camera.cameraRotation.y -= offsetX / _this.angularSensibility;
	                        }
	                    }
	                    else {
	                        if(Math.abs(angle.x) < _this.restrictionX )  {
	                            _this.camera.cameraRotation.y += offsetX / _this.angularSensibility;
	                        }
	                    }
	                    if(Math.abs(angle.y) < _this.restrictionY )  {
	                        _this.camera.cameraRotation.x += offsetY / _this.angularSensibility;
	                    }
	                    _this.previousPosition = {
	                        x: evt.clientX,
	                        y: evt.clientY
	                    };
	                    if (!noPreventDefault) {
	                        evt.preventDefault();
	                    }
	                }
	            };
	        }
	        
	        this._onSearchMove = function (evt) {       
	            if (!engine.isPointerLock) {
	                return;
	            }       
	            var offsetX = evt.movementX || evt.mozMovementX || evt.webkitMovementX || evt.msMovementX || 0;
	            var offsetY = evt.movementY || evt.mozMovementY || evt.webkitMovementY || evt.msMovementY || 0;
	            if (_this.camera.getScene().useRightHandedSystem) {
	                _this.camera.cameraRotation.y -= offsetX / _this.angularSensibility;
	            }
	            else {
	                _this.camera.cameraRotation.y += offsetX / _this.angularSensibility;
	            }
	            _this.camera.cameraRotation.x += offsetY / _this.angularSensibility;
	            _this.previousPosition = null;
	            if (!noPreventDefault) {
	                evt.preventDefault();
	            }
	        };
	        
	        this._observer = this.camera.getScene().onPointerObservable.add(this._pointerInput, BABYLON.PointerEventTypes.POINTERDOWN | BABYLON.PointerEventTypes.POINTERUP | BABYLON.PointerEventTypes.POINTERMOVE);
	        element.addEventListener("mousemove", this._onSearchMove, false);
	    };
	
	    // 添加分离控制
	    FreeCameraSearchInput.prototype.detachControl = function () {
	        var engine = this.camera.getEngine();
	        var element = engine.getInputElement();
	        if (this._observer && element) {
	            this.camera.getScene().onPointerObservable.remove(this._observer);
	            element.removeEventListener("mousemove", this._onSearchMove);
	            this._observer = null;
	            this._onSearchMove = null;
	            this.previousPosition = null;
	        }
	    };
	
	    // 为名称添加两个必需的函数
	    FreeCameraSearchInput.prototype.getClassName = function () {
	        return "FreeCameraSearchInput";
	    };
	
	    FreeCameraSearchInput.prototype.getSimpleName = function () {
	        return "MouseSearchCamera";
	    };
	
	    // 向相机添加新的鼠标输入管理器
	    camera.inputs.add(new FreeCameraSearchInput());
	
	    
	
	    return scene;
	}
	  



		





