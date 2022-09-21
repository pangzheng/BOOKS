# babylon 官方深度教学-相机-相机碰撞
## 相机碰撞
### 相机、网格碰撞和重力
你玩过 FPS（第一人称射击）游戏吗？在本教程中，我们将模拟相同的摄像机运动：摄像机在地板上，与地面发生碰撞，并可能与场景中的任何对象发生碰撞。
### 我怎样才能做到这一点 ？
要复制这个动作，我们必须做 3 个简单的步骤：

1. 定义和应用重力

	首先要做的是定义我们的重力矢量，定义 G 力。

	Babylon.js [场景](https://doc.babylonjs.com/typedoc/classes/babylon.scene) 有一个[重力](https://doc.babylonjs.com/typedoc/classes/babylon.scene#gravity)属性，可以应用于您之前在代码中定义的任何相机。这将沿着指定的方向和速度移动相机（Vector3对象），除非相机的椭球体（参见下面的 #2）与该方向上的另一个网格（例如您的地面网格）发生碰撞，并且 `checkCollisions` 设置为 `true`。
	
		scene.gravity = new BABYLON.Vector3(0, -0.15, 0);
	要将场景的重力应用到您的相机，请将 `applyGravity` 属性设置为 `true`
	
		camera.applyGravity = true;
	在现实世界中，重力是一种向下施加的力（好吧，有点）——即，沿 Y 轴的负方向。在地球上，这个力大约是 9.81m/s²。下落的物体在下落时会加速，所以完全达到这个速度需要 1 秒，然后在 2 秒后速度达到 19.62m/s，在 3 秒后达到 29.43m/s，以此类推。在大气中，风阻力最终匹配这个力并且速度停止增加（“终端速度”）。

	但是，Babylon.js 遵循一个更简单的引力模型—— `scene.gravity` 代表一个恒定的速度，而不是加速度，它以 `单位/帧` 而不是 `米/秒` 来衡量。在渲染每一帧时，您应用此重力的相机将沿每个轴移动x向量的值（通常设置为 0 ，z 但您可以在任何方向上都有“重力”！），直到检测到碰撞。

	虽然 Babylon.js 单位没有直接的物理等效物，但在默认相机视野下，`1 单位 = 1 米` 的近似值是一个相当标准的假设。因此，如果您想近似标称地球重力，您需要对每秒渲染的帧数做出一些假设，并计算一个合适的向量：
	
		const assumedFramesPerSecond = 60;
		const earthGravity = -9.81;
		scene.gravity = new BABYLON.Vector3(0, earthGravity / assumedFramesPerSecond, 0);
	由于这是每帧计算一次，因此相机实际上并没有“移动”，而是沿着重力矢量的方向进行微小的“跳跃”。如果您依靠碰撞检测来确定相机（或者更确切地说，为此目的附加到它的网格）是否已“进入”或“退出”某个其他网格（例如，您的“地面”层来感应下落的角色并重置游戏玩法）。
	
	根据您选择的重力、起始高度以及“触发”网格的位置和高度，相机可能会直接跳过触发网格而不会“相交”它。一定要检查以确保至少有一个 `scene.gravity` 的倍数添加到起始仰角将与你的触发网格相交。

	如果您需要更准确地表示重力（或其他）力，您可以使用与 [Babylon 集成的物理引擎](https://doc.babylonjs.com/divingDeeper/physics/usingPhysicsEngine)，或添加[您自己的物理引擎](https://doc.babylonjs.com/divingDeeper/physics/addPhysicsEngine).
		
	一个警告：添加物理代替品并为同一个对象设置启用碰撞可能会导致意外行为
2. 定义一个椭球

	下一个重要步骤是定义我们相机周围的椭圆体。这个椭球代表我们玩家的尺寸：当一个网格与这个椭球接触时会引发一个碰撞事件，防止我们的相机离这个网格太近：		
	
	![](./pic/ellipsoid.png)
	
	Babylon.js 相机上的 `ellipsoid` 属性默认为大小 (0.5, 1, 0.5)。更改这些值将使您更高、更大、更小、更薄，具体取决于调整后的轴。在下面的示例中，我们将使相机的椭球比默认椭球更宽更深：
	
		// 设置摄像机周围的椭球(例如玩家的尺寸)
		camera.ellipsoid = new BABYLON.Vector3(1, 1, 1);
	请注意，相机的椭球体是偏移的，以始终使视点位于椭球体的顶部。您可以通过更新 [ellipsoidOffset](https://doc.babylonjs.com/typedoc/classes/babylon.freecamera#ellipsoidoffset) 属性来控制此行为。
	
	计算如下：
	
		finalPosition = position - vec3(0, ellipsoid.y, 0) + ellipsoidOffset
3.  应用碰撞

	最后一步是声明我们对感知场景中的碰撞感兴趣
	
		// 开启碰撞
		scene.collisionsEnabled = true;
		camera.checkCollisions = true;
	并声明哪些网格可能与我们的相机发生冲突：
	
		ground.checkCollisions = true;
		box.checkCollisions = true;	
- [教程中使用的场景](https://playground.babylonjs.com/#4HUQQ#1713)

		// 场景
		var createScene = function () {
		    var scene = new BABYLON.Scene(engine);
		
		    // 灯光
		    var light0 = new BABYLON.DirectionalLight("Omni", new BABYLON.Vector3(-2, -5, 2), scene);
		    var light1 = new BABYLON.PointLight("Omni", new BABYLON.Vector3(2, -5, -2), scene);
		
		    // 创建需要一个相机碰撞
		    var camera = new BABYLON.FreeCamera("FreeCamera", new BABYLON.Vector3(8, -8, -16), scene);
		    camera.setTarget(new BABYLON.Vector3(0, -8, 0));
		    camera.attachControl(canvas, true);
		    camera.minZ = 0.45;
		
		    //创建地面
		    var ground = BABYLON.Mesh.CreatePlane("ground", 20.0, scene);
		    //材质
		    ground.material = new BABYLON.StandardMaterial("groundMat", scene);
		    ground.material.diffuseColor = new BABYLON.Color3(1, 1, 1);
		    ground.material.backFaceCulling = false;
		    //位置
		    ground.position = new BABYLON.Vector3(5, -10, -15);
		    // 方向
		    ground.rotation = new BABYLON.Vector3(Math.PI / 2, 0, 0);
		
		    // 创建简单的板条箱
		    var box = BABYLON.Mesh.CreateBox("crate", 2, scene);
		    // 材质
		    box.material = new BABYLON.StandardMaterial("Mat", scene);
		    box.material.diffuseTexture = new BABYLON.Texture("textures/crate.png", scene);
		    box.material.diffuseTexture.hasAlpha = true;
		    box.position = new BABYLON.Vector3(5, -9, -10);
		
		    //为场景设置重力(G力，在y轴)
		    scene.gravity = new BABYLON.Vector3(0, -0.9, 0);
		
		    // 为场景开放碰撞
		    scene.collisionsEnabled = true;
		
		    //将碰撞和重力应用到相机
		    camera.checkCollisions = true;
		    camera.applyGravity = true;
		
		    // 设置相机周围的椭球体（例如你的播放器的大小）
		    camera.ellipsoid = new BABYLON.Vector3(1, 1, 1);
		
		    // 最后，说出哪个网格是可碰撞的
		    ground.checkCollisions = true;
		    box.checkCollisions = true;
		
		    return scene;
		}
	您的相机将落在 y 轴上，直到它与地面相撞。而且，当您将它移得太靠近时，您的相机会与盒子发生碰撞。

4. 物体与物体碰撞

	您可以通过使用 [ellipsoid](https://doc.babylonjs.com/typedoc/classes/babylon.mesh#ellipsoid) 属性和 [moveWithCollisions(velocity)](https://doc.babylonjs.com/typedoc/classes/babylon.mesh#movewithcollisions)函数对网格做同样的事情。当发现当前网格与任何激活了 [checkCollisions](https://doc.babylonjs.com/typedoc/classes/babylon.mesh#checkcollisions) 的网格之间没有碰撞时，此函数将尝试根据给定速度移动网格。

	您还可以使用网格的 [ellipsoidOffset](https://doc.babylonjs.com/typedoc/classes/babylon.mesh#ellipsoidoffset) 在网格上移动椭圆体（默认情况下，椭圆体以网格为中心）。

		var speedCharacter = 8;
		var gravity = 0.15;
		var character = Your mesh;
		
		character.ellipsoid = new BABYLON.Vector3(0.5, 1.0, 0.5);
		character.ellipsoidOffset = new BABYLON.Vector3(0, 1.0, 0);
		
		var forwards = new BABYLON.Vector3(parseFloat(Math.sin(character.rotation.y)) / speedCharacter, gravity, parseFloat(Math.cos(character.rotation.y)) / speedCharacter);
		forwards.negate();
		character.moveWithCollisions(forwards);
		// or
		var backwards = new BABYLON.Vector3(parseFloat(Math.sin(character.rotation.y)) / speedCharacter, -gravity, parseFloat(Math.cos(character.rotation.y)) / speedCharacter);
		character.moveWithCollisions(backwards);	
	- [物体碰撞](https://playground.babylonjs.com/#1M79PT#1)

			const createScene = function () {
			    const scene = new BABYLON.Scene(engine);
			    const camera = new BABYLON.ArcRotateCamera("camera1", -Math.PI / 2, Math.PI / 3.5, 25, new BABYLON.Vector3(0, 0, 0));
			    camera.attachControl(canvas, true);
			    const light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
			    light.intensity = 0.7;
			
			    /* 设置风景
			    _____________________*/
			
			    //地面
			    const ground = BABYLON.MeshBuilder.CreateGround("ground", {width: 20, height: 20}, scene);
			    ground.material = new BABYLON.StandardMaterial("groundMat", scene);
			    ground.material.diffuseColor = new BABYLON.Color3(1, 1, 1);
			    ground.material.backFaceCulling = false;
			
			
			    const randomNumber = function (min, max) {
					if (min == max) {
						return (min);
					}
					const random = Math.random();
					return ((random * (max - min)) + min);
				};
			    
			    // 盒子
			    const box = new BABYLON.MeshBuilder.CreateBox("crate", {size: 2}, scene);
			    box.material = new BABYLON.StandardMaterial("Mat", scene);
			    box.material.diffuseTexture = new BABYLON.Texture("textures/crate.png", scene);
			    box.checkCollisions = true;
			
			    // 设置盒子位置
			    const boxNb = 6;
			    let theta = 0;
			    const radius = 6;
			    box.position = new BABYLON.Vector3((radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.cos(theta + randomNumber(-0.1 * theta, 0.1 * theta)), 1, (radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.sin(theta + randomNumber(-0.1 * theta, 0.1 * theta)));
			    
			    // 循环创建盒子
			    const boxes = [box];
			    for (let i = 1; i < boxNb; i++) {
			        theta += 2 * Math.PI / boxNb;
			        const newBox = box.clone("box" + i);
			        boxes.push(newBox);
			        newBox.position = new BABYLON.Vector3((radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.cos(theta + randomNumber(-0.1 * theta, 0.1 * theta)), 1, (radius + randomNumber(-0.5 * radius, 0.5 * radius)) * Math.sin(theta + randomNumber(-0.1 * theta, 0.1 * theta)));
			    }
			    /* 结束创建风景 */
			
			    // 创建 mesh 
			    const base = new BABYLON.Mesh("pivot");
			    
			    const headDiam = 1.5;
			    const bodyDiam = 2; 
			    const head = BABYLON.MeshBuilder.CreateSphere("h", {diameter: headDiam});
			    head.parent = base;
			    const body = BABYLON.MeshBuilder.CreateSphere("b", {diameter: bodyDiam});
			    body.parent = base;
			    head.position.y = 0.5 * (headDiam + bodyDiam);
			    base.position.y = 0.5 * bodyDiam;
			
			    const extra = 0.25;
			    base.ellipsoid = new BABYLON.Vector3(0.5 * bodyDiam, 0.5 * (headDiam + bodyDiam), 0.5 * bodyDiam); //x and z must be same value
			    base.ellipsoid.addInPlace(new BABYLON.Vector3(extra, extra, extra));
			    const offsetY = 0.5 * (headDiam + bodyDiam) - base.position.y
			    base.ellipsoidOffset = new BABYLON.Vector3(0, offsetY, 0);
			    
			    // 创建可见椭球周围的相机
			    const a = base.ellipsoid.x;
			    const b = base.ellipsoid.y;
			    const points = [];
			    for(let theta = -Math.PI/2; theta < Math.PI/2; theta += Math.PI/36) {
			        points.push(new BABYLON.Vector3(0, b * Math.sin(theta) + offsetY, a * Math.cos(theta)));
			    }
			
			    const ellipse = [];
			    ellipse[0] = BABYLON.MeshBuilder.CreateLines("e", {points:points}, scene);
			    ellipse[0].color = BABYLON.Color3.Red();
			    ellipse[0].parent = base;
			    const steps = 12;
			    const dTheta = 2 * Math.PI / steps; 
			    for(let i = 1; i < steps; i++) {
			            ellipse[i] = ellipse[0].clone("el" + i);
			            ellipse[i].parent = base;
			            ellipse[i].rotation.y = i * dTheta;
			    }
			
			    //键盘移动
			    const forward = new BABYLON.Vector3(0, 0, 1);
			    let angle = 0;
			    let matrix = BABYLON.Matrix.Identity();
			
			    //指示方向的线
			    let line = BABYLON.MeshBuilder.CreateLines("f", {points: [base.position.add(new BABYLON.Vector3(0, 3, 0)), base.position.add(new BABYLON.Vector3(0, 3, 0)).add(forward.scale(3))], updatable: true});
			    scene.onKeyboardObservable.add((kbInfo) => {
					switch (kbInfo.type) {
						case BABYLON.KeyboardEventTypes.KEYDOWN:
							switch (kbInfo.event.key) {
			                    case "a":
			                    case "A":
			                        angle -= 0.1;
			                        BABYLON.Matrix.RotationYToRef(angle, matrix);
			                        BABYLON.Vector3.TransformNormalToRef(forward, matrix, forward);
			                        base.rotation.y = angle;
			                    break
			                    case "d":
			                    case "D":
			                        angle += 0.1;
			                        BABYLON.Matrix.RotationYToRef(angle, matrix);
			                        BABYLON.Vector3.TransformNormalToRef(forward, matrix, forward);
			                        base.rotation.y = angle;
			                    break
			                    case "w":
			                    case "W":
			                        base.moveWithCollisions(forward.scale(0.1));
			                    break
			                    case "s":
			                    case "S":
			                        base.moveWithCollisions(forward.scale(-0.1));
			                    break
			                }
						break;
					}
			        line = BABYLON.MeshBuilder.CreateLines("f", {points: [base.position.add(new BABYLON.Vector3(0, 3, 0)), base.position.add(new BABYLON.Vector3(0, 3, 0)).add(forward.scale(3))], instance:line});
				});
			
			    return scene;
			};
- 弧形旋转相机

	它还可以检查碰撞，但不是沿着障碍物滑动，而是在 `ArcRotateCamera` 发生碰撞时相机不会移动。

	要激活碰撞，只需设置 `camera.checkCollisions` 为 `true`。您还可以定义碰撞半径
	
		camera.collisionRadius = new BABYLON.Vector3(0.5, 0.5, 0.5);
- 其他类型的碰撞

	太好了，现在您可以开发真正的 FPS 游戏了！但也许您想知道一个网格何时与另一个网格发生冲突？如果您对此感兴趣，可以前往此处：[Mesh Collisions](https://doc.babylonjs.com/divingDeeper/mesh/interactions/mesh_intersect)。