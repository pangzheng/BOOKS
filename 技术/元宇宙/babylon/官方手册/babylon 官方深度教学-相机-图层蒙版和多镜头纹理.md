# babylon 官方深度教学-相机-图层蒙版和多镜头纹理
## 使用图层蒙版的多个相机的不同网格
AlayerMask 是分配给每个网格和相机的编号。它用于位级别以指示灯光和相机是否应该照亮或显示网格。默认值 `0x0FFFFFFF` 将导致网格被任何库存灯光和相机照亮和显示。

该功能主要在多个摄像头同时处于活动状态时使用。如果您希望有一个在屏幕上始终可见并且可以选择的网格，例如一个按钮，您可以在场景中添加第二个摄像机和灯光以专门显示和点亮它。

您需要第二台相机才能看到按钮。该按钮也应该只可见一次。

请注意，默认 layerMask 以前 4 位为 0 或关闭开始。如果第二台摄像机和按钮都具有 layerMask 以下 4 个值之一，那么第二台摄像机只会看到按钮：

- 0x10000000
- 0x20000000
- 0x40000000
- 0x80000000

还应该注意的是 layerMask，任何人都看不到 a 为 0 的网格。这可能有利于隐藏东西。

要设置多相机：

	if (scene.activeCameras.length === 0){
	    scene.activeCameras.push(scene.activeCamera);
	}
	
	var secondCamera = new Babylon.Camera(...);
	secondCamera.layerMask = 0x10000000;
	scene.activeCameras.push(secondCamera);
	
	var Button = new BABYLON.Mesh(...);
	Button.layerMask = 0x10000000;

## 灯光
除非第二台摄像机的网格材质是纯自发光的，否则仍然会为按钮留下任何光线来照亮所有其他网格，以及场景中的其他光线照亮按钮。要防止场景灯光照亮按钮，请遍历现有灯光，并设置 `excludeWithLayerMask` 值：

	for (var i = scene.lights.length - 1; i >= 0; i--) {
	  scene.lights[i].excludeWithLayerMask = 0x10000000;
	}
然后使“按钮”亮起：

	var light = new BABYLON.Light(...);
	light.includeOnlyWithLayerMask = 0x10000000;
最后，如果以后可能会产生更多的灯光，可以在添加灯光时注册一个回调

	scene.onNewLightAdded = onNewLight;
这可能是：

	onNewLight = function (newLight, positionInArray, scene) {
	  newLight.excludeWithLayerMask = 0x10000000;
	};

## 枪瞄准准星示例
这是使用显示枪瞄准器的第二个正交相机的简单示例。为了简单起见，我们使用了发光材料来避免照明。只需将其复制并粘贴到任何场景中，然后调用它。选择还允许 Babylon 的 `layerMaskDialog` 扩展进行互操作。也许这些可以结合起来做一个带有测距仪的抬头坦克瞄准具。

商业质量的实现可能不会使用 `CreateBox()`，因为它会创建无论如何都无法直接看到的深度面。它还应该考虑窗口大小的变化，除非它是平板电脑应用程序。

- [瞄准器例子](https://playground.babylonjs.com/#JU1DZP)

		// 创建场景函数
		var createScene = function () {
			// 创建场景
			var scene = new BABYLON.Scene(engine);
			// 创建主 FPS 摄像机
			var camera = new BABYLON.FreeCamera("camera1", new BABYLON.Vector3(0, 5, -10), scene);
			// 朝向
			camera.setTarget(BABYLON.Vector3.Zero());
			// 默认控制打开
			camera.attachControl(canvas, true);
			// 创建场景灯光
			var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
			// 亮度
			light.intensity = 0.7;
			
			// 创建球体
			var sphere = BABYLON.Mesh.CreateSphere("sphere1", 16, 2, scene);
			// 设置位置
			sphere.position.y = 1;
			
			// 创建地面
			var ground = BABYLON.Mesh.CreateGround("ground1", 6, 6, 2, scene);
			
			// 瞄准器函数
			function addGunSight(scene) {
				
				if (scene.activeCameras.length === 0) {
					scene.activeCameras.push(scene.activeCamera);
				}
				  
				// 创建第二个瞄准器摄像机
				var secondCamera = new BABYLON.FreeCamera("GunSightCamera", new BABYLON.Vector3(0, 0, -50), scene);
					// 摄像机模式
					secondCamera.mode = BABYLON.Camera.ORTHOGRAPHIC_CAMERA;
					// 设置层
					secondCamera.layerMask = 0x20000000;
					// 激活摄像机
					scene.activeCameras.push(secondCamera);
					
				// 设置 mesh
				meshes = [];
				var h = 250;
				var w = 250;
					
				// y 纵轴瞄准器
				var y = BABYLON.Mesh.CreateBox("y", h * 0.2, scene);
				y.scaling = new BABYLON.Vector3(0.05, 1, 1);
				y.position = new BABYLON.Vector3(0, 0, 0);
				meshes.push(y);
				
				// x 横轴瞄准器	
				var x = BABYLON.Mesh.CreateBox("x", h * 0.2, scene);
				x.scaling = new BABYLON.Vector3(1, 0.05, 1);
				x.position = new BABYLON.Vector3(0, 0, 0);
				meshes.push(x);
					
				var lineTop = BABYLON.Mesh.CreateBox("lineTop", w * 0.8, scene);
				lineTop.scaling = new BABYLON.Vector3(1, 0.005, 1);
				lineTop.position = new BABYLON.Vector3(0, h * 0.5, 0);
				meshes.push(lineTop);
					
				var lineBottom = BABYLON.Mesh.CreateBox("lineBottom", w * 0.8, scene);
				lineBottom.scaling = new BABYLON.Vector3(1, 0.005, 1);
				lineBottom.position = new BABYLON.Vector3(0, h * -0.5, 0);
				meshes.push(lineBottom);
					
				var lineLeft = BABYLON.Mesh.CreateBox("lineLeft", h, scene);
				lineLeft.scaling = new BABYLON.Vector3(0.01, 1, 1);
				lineLeft.position = new BABYLON.Vector3(w * -0.4, 0, 0);
				meshes.push(lineLeft);
					
				var lineRight = BABYLON.Mesh.CreateBox("lineRight", h, scene);
				lineRight.scaling = new BABYLON.Vector3(0.01, 1, 1);
				lineRight.position = new BABYLON.Vector3(w * 0.4, 0, 0);
				meshes.push(lineRight);
				
				// 获取 mash 设置准星层	
				var gunSight = BABYLON.Mesh.MergeMeshes(meshes);
				gunSight.name = "gunSight";
				gunSight.layerMask = 0x20000000;
				gunSight.freezeWorldMatrix();
				
				// 设置准星材料	
				var mat = new BABYLON.StandardMaterial("emissive mat", scene);
				mat.checkReadyOnlyOnce = true;
				mat.emissiveColor = new BABYLON.Color3(0, 1, 0);
				// 绑定准星材料到准星	
				gunSight.material = mat;
			}
			// 使用函数
			addGunSight(scene);
			
			return scene;
		};
		
使用此处的信息并将其与[上一节](https://doc.babylonjs.com/divingDeeper/cameras/multiViewsPart2)中的 Viewports 信息相结合，我们可以创建一个更复杂的示例，其中包括从特定相机中省略网格的选项。

- [例子](https://playground.babylonjs.com/#L92PHY#217)

		/**
		 * 画中画相机视觉演示
		 * Written by Dave Solares (Twitter/GitHub - PolygonalSun)
		 * 
		 * 这个演示提供了一个摄像头在三维空间中如何工作的视觉表现，使用了图层蒙版和多个 Viewports。
		 * 
		 * 因为摄像机没有视觉表现，所以很难看到摄像机如何移动并对用户输入作出反应。 
		 * 在这个演示中，有几种方法可以操纵摄像机。 
		 * 你可以用方向键来旋转（左键+移动也可用于旋转）
		 * WASD将移动和扫射摄像机，类似于FPS控制
		 * 在游戏中有一些功能，如果你想了解更多信息，都会提到各自的文档页面
		 * 
		 * 
		 * 控制
		 * - 键盘
		 *  WASD Keys - Move/Strafe
		 *  Arrow Keys - Look Around
		 * - 鼠标
		 *  Left-Click+Move - Look Around
		 * - 点击
		 *  Drag - Look Around
		 *  Two-finger Touch - Move Forward
		 */
		
		// 一个数组，用于跟踪要从 pip 相机中省略的网格。
		var omittedMeshes = [];
		
		/**
		 * 创建并返回场景对象的函数
		 * 
		 * 这个函数利用了Viewports、Layer Masks 和 SceneLoader。
		 * 
		 * MultiViews Part 2:
		 * https://doc.babylonjs.com/divingDeeper/cameras/multiViewsPart2
		 * 
		 * 图层蒙版和多机位纹理:
		 * https://doc.babylonjs.com/divingDeeper/cameras/layerMasksAndMultiCam
		 * 
		 * 加载任何文件类型:
		 * https://doc.babylonjs.com/divingDeeper/importers/loadingFileTypes
		 * 
		 */
		
		// 创建场景函数 
		var createScene = function () {
		    // 这将创建一个基本的场景对象（非网格）。
		    var scene = new BABYLON.Scene(engine);
		
		    // 这就形成了我们的主摄像头
		    var camera = new BABYLON.FreeCamera("camera1", new BABYLON.Vector3(0, 5, 0), scene);
		
		    // 加载glTF场景。 一旦加载，开始配置
		    BABYLON.SceneLoader.Append("https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/Sponza/glTF/", "Sponza.gltf", scene, function (scene) {
		       
		       // pip 创建摄像机
		        var pipCamera = new BABYLON.FreeCamera("pipCamera", new BABYLON.Vector3(0,20,0), scene);
		        // pip 摄像机朝向
		        pipCamera.setTarget(BABYLON.Vector3.Zero());
		
		        // 我们希望保留方形 PIP 的外观，所以我们将使用主摄像头的长宽比来相应地调整尺寸。
		        // Aspect ratio < 1 = Portrait, > 1 = Landscape
		        let ar = engine.getAspectRatio(camera);
		        let pipW = (ar < 1) ? 0.3 : 0.3 * (1/ar);
		        let pipH = (ar < 1) ? 0.3 * ar : 0.3;
		        let pipX = 1 - pipW;
		        let pipY = 1 - pipH;
		
		        // 指定每台摄像机的位置和应占屏幕的数量
		        // 注意：Viewports 的所有数值都将是0到1，把它看成是屏幕的百分比。
		        camera.viewport = new BABYLON.Viewport(0, 0, 1, 1);
		        pipCamera.viewport = new BABYLON.Viewport(pipX, pipY, pipW, pipH);
		
		        // 为摄像机设置图层蒙版（以及以后的网格）。
			  // 这样做是因为 Sponza 网格中有些部分我们不想在PIP摄像机上显示（逻辑见 castRay 函数）。
			  
			  // 设置主摄像头图层蒙版，使其能够看到0x10000000和0x20000000对象
		        camera.layerMask = 0x30000000;  
		        // 设置 pip 摄像头 设置图层屏蔽，只看到 0x10000000 对象
		        pipCamera.layerMask = 0x10000000;  
		
		        // 将摄像机添加到活动摄像机列表中。 
		        // 每台摄像机必须在活动的摄像机列表中，才能以其定义的 viewport 显示。
		        scene.activeCameras.push(camera);
		        scene.activeCameras.push(pipCamera);
		
		        // 创建头部网格，以代表摄像机所看的位置。.
		        var head = createHead(scene);
			        // 我们关闭了头部网格的拾取功能，因为我们不希望它被我们的顶层射线拾取。
			        head.isPickable = false; 
			        // 设置父级
			        head.setParent(camera);
			        // 设置位置
			        head.position = BABYLON.Vector3.Zero();
		
		        // 这创建了一个光，瞄准0,1,0 -天空(非网格)
		        var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
			        //设置光强度
			        light.intensity = 0.7;
		
		        // 对于 Sponza 网格的每个部分，我们想要增加其大小并设置其图层蒙版，以便它对两个相机都可见
		        scene.meshes.forEach((mesh) => {
		        	// 设置大小
		            mesh.scaling = new BABYLON.Vector3(3,3,3);
		            // 设置图层蒙版，使网格对所有相机都可见
		            mesh.layerMask = 0x10000000; 
		        });
		
		        //设置摄像头向下看大厅和显示脸
		        camera.setTarget(new BABYLON.Vector3(1,6,0));
		
		        // 创建一个基本的天空盒
		        createSkyBox(scene);
		
		        // 创建并初始化 DeviceSourceManager
		        var dsm = initializeInput(scene, camera, pipCamera);
		    });
		
		    return scene;
		};
		
		/******************** 辅助函数 ********************/
		/**
		 * 给定一个场景对象，创建头部网格来表示一个相机
		 * 
		 * 这个头部使用了在地图材料到单个网格面页面上找到的操场的UV代码:
		 * https://doc.babylonjs.com/divingDeeper/materials/using/texturePerBoxFace
		 */
		
		// 创建头部函数
		var createHead = function (scene) {
		    // 为我们的头部创建一个纹理
		    var mat = new BABYLON.StandardMaterial("mat", scene);
		    var texture = new BABYLON.Texture("https://i.imgur.com/vxH5bCg.jpg", scene);
		    mat.diffuseTexture = texture;
		
		    // 为我们的盒子面定义uv(坐标)
		    // 6面
		    var faceUV = new Array(6);
		    faceUV[0] = new BABYLON.Vector4(0, 0.5, 1 / 3, 1);  // 右边
		    faceUV[1] = new BABYLON.Vector4(1 / 3, 0, 2 / 3, 0.5);  // 头顶
		    faceUV[2] = new BABYLON.Vector4(2 / 3, 0, 1, 0.5); // 后面
		    faceUV[3] = new BABYLON.Vector4(0, 0, 1 / 3, 0.5); // 下面
		    faceUV[4] = new BABYLON.Vector4(1 / 3, 0.5, 2 / 3, 1); // 前面
		    faceUV[5] = new BABYLON.Vector4(2 /3, 0.5, 1, 1); // 左边
		
		    // 设置
		    var options = {
		        faceUV: faceUV,
		        wrap: true
		    };
		
		    // 创建头部，旋转到适当的位置，并设置纹理。
		    let head = BABYLON.MeshBuilder.CreateBox('head', options, scene);
		    head.rotate(BABYLON.Axis.Y, Math.PI);
		    head.material = mat;
		
		    return head;
		};
		
		/**
		 * 给定一个场景，创建一个基本的天空盒来包围该区域。
		 * 
		 * 有关Skyboxes的更多信息，请查看Skyboxes文档页面:
		 * https://doc.babylonjs.com/divingDeeper/environment/skybox
		 */
		
		// 创建天空和函数 
		var createSkyBox = function (scene) {
		    // 创建天空盒
		    var skybox = BABYLON.MeshBuilder.CreateBox("skyBox", {size:1000.0}, scene);
		    	// 设置层全部可见
		    	skybox.layerMask = 0x10000000;
		    	// 创建材质
			var skyboxMaterial = new BABYLON.StandardMaterial("skyBox", scene);
			skyboxMaterial.backFaceCulling = false;
			skyboxMaterial.reflectionTexture = new BABYLON.CubeTexture("textures/skybox", scene);
			skyboxMaterial.reflectionTexture.coordinatesMode = BABYLON.Texture.SKYBOX_MODE;
			skyboxMaterial.diffuseColor = new BABYLON.Color3(0, 0, 0);
			skyboxMaterial.specularColor = new BABYLON.Color3(0, 0, 0);
			skybox.material = skyboxMaterial;
		};
		
		/**
		 * 不使用默认的附加输入管理器控件，使用 DeviceSourceManager 添加类似 fps 的控件。箭头键已经被赋予了旋转行为，为那些想在他们的实验中只使用键盘或潜在添加手柄支持的用户。		 * 
		 * Device Source Manager:
		 * https://doc.babylonjs.com/divingDeeper/input/deviceSourceManager
		 */
		 
		// 创建输入控制函数 
		var initializeInput = function (scene, camera, pipCamera) {
		
		    // 引用 DSM
		    let dsm = new BABYLON.DeviceSourceManager(scene.getEngine());
		
		    // 添加控制
		    dsm.onDeviceConnectedObservable.add((device) => {
		        // 键盘设置
		        if (device.deviceType === BABYLON.DeviceType.Keyboard) {
		            scene.onBeforeRenderObservable.add(() => {
		                let transformMatrix = BABYLON.Matrix.Zero();
		                let localDirection = BABYLON.Vector3.Zero();
		                let transformedDirection = BABYLON.Vector3.Zero();
		                let isMoving = false;
		
		                // WASD 将移动
		                if (device.getInput(65) === 1) {
		                    localDirection.x = -0.1;
		                    isMoving = true;
		                }
		                if (device.getInput(68) === 1) {
		                    localDirection.x = 0.1;
		                    isMoving = true;
		                }
		
		                if (device.getInput(87) === 1) {
		                    localDirection.z = 0.1;
		                    isMoving = true;
		                }
		                if (device.getInput(83) === 1) {
		                    localDirection.z = -0.1;
		                    isMoving = true;
		                }
		
		                // 方向键旋转
		                if (device.getInput(37) === 1) {
		                    camera.rotation.y -= 0.01;
		                }
		                if (device.getInput(39) === 1) {
		                    camera.rotation.y += 0.01;
		                }
		                if (device.getInput(38) === 1) {
		                    camera.rotation.x -= 0.01;
		                }
		                if (device.getInput(40) === 1) {
		                    camera.rotation.x += 0.01;
		                }
		
		                if (isMoving) {
		                    camera.getViewMatrix().invertToRef(transformMatrix);
		                    BABYLON.Vector3.TransformNormalToRef(localDirection, transformMatrix, transformedDirection);
		                    camera.position.addInPlace(transformedDirection);
		                    pipCamera.position.addInPlace(transformedDirection);
		                    castRay(scene, pipCamera);
		                }
		            });
		        }
		        
		        // 鼠标/点击配置
		        else if (device.deviceType === BABYLON.DeviceType.Mouse || device.deviceType === BABYLON.DeviceType.Touch) {
		            device.onInputChangedObservable.add((deviceData) => {
		                if (deviceData.inputIndex === BABYLON.PointerInput.Move && device.getInput(BABYLON.PointerInput.LeftClick) === 1) {
		                    camera.rotation.y += deviceData.movementX * 0.00175;
		                    camera.rotation.x += deviceData.movementY * 0.00175;
		                }
		            });
		
		            // 如果有两根手指压在屏幕上，则向前移动
		            if (!scene.beforeRender && device.deviceType === BABYLON.DeviceType.Touch ) {
		                scene.beforeRender = () => {
		                    let transformMatrix = BABYLON.Matrix.Zero();
		                    let localDirection = BABYLON.Vector3.Zero();
		                    let transformedDirection = BABYLON.Vector3.Zero();
		                    let isMoving = false;
		
		                    if (dsm.getDeviceSources(BABYLON.DeviceType.Touch).length === 2) {
		                        localDirection.z = 0.1;
		                        isMoving = true;
		                    }
		
		                    if (isMoving) {
		                        camera.getViewMatrix().invertToRef(transformMatrix);
		                        BABYLON.Vector3.TransformNormalToRef(localDirection, transformMatrix, transformedDirection);
		                        camera.position.addInPlace(transformedDirection);
		                        pipCamera.position.addInPlace(transformedDirection);
		                        castRay(scene, pipCamera);
		                    }
		                };
		            }
		        }
		    });
		
		    return dsm;
		};
		
		
		/**
		 * 给定一个场景、头顶的摄像机和画中画平面， 从PIP摄像机投射一条光线到球员，并检查所有的网格之间。
		 * 光线击中的每个网格都将其图层蒙版更改为 0x20000000，以便只对主摄像机可见，而对 pip 摄像机不可见。
		 * 当一个网格不再被投射的光线所选择时，它的层蒙版将被改变回 0x10000000 (对两个摄像机都可见)。
		 * 
		 * 这个函数使用射线进行网格选取。欲了解更多信息，请查看网格挑选:
		 * https://doc.babylonjs.com/divingDeeper/mesh/interactions/picking_collisions
		 */
		 
		// 投射线函数 
		var castRay = function (scene) {
		    // 因为我们是垂直向下的，我们可以用给定的向量
		    let direction = new BABYLON.Vector3(0, -1, 0);
		    let mainCamera = scene.activeCameras[0];
		    let pipCamera = scene.activeCameras[1];
		
		    // 长度将是每个相机之间的距离
		    let length = pipCamera.position.y - mainCamera.position.y;
		    let ray = new BABYLON.Ray(pipCamera.position, direction, length);
		    let hits = scene.multiPickWithRay(ray);
		
		    // 如果我们有任何挑选的网格，添加所有挑选的网格，并删除任何没有被我们的射线标记的网格
		    if (hits) {
		        
		        // 数组来保存当前选取的网格
		        let meshesToCheck = []; 
		
		        hits.forEach((hit) => {
		        	//设置图层蒙版，这样只有主摄像头可以看到网格
		            hit.pickedMesh.layerMask = 0x20000000; 
		            meshesToCheck.push(hit.pickedMesh);
		        });
		
		        let meshesToReAdd = omittedMeshes.filter(omittedMesh => meshesToCheck.indexOf(omittedMesh) < 0);
		        meshesToReAdd.forEach((omittedMesh) => {
		            omittedMesh.layerMask = 0x10000000;
		        });
		
		        omittedMeshes = meshesToCheck;
	    }
		};

