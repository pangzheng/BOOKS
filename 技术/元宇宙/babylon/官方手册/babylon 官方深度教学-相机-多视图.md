# babylon 官方深度教学-相机-多视图
## 介绍
[WebGL Multiview](https://www.khronos.org/registry/webgl/extensions/OVR_multiview2/) 扩展允许在单个渲染过程中渲染多个视图（例如，VR 场景的每只眼睛）。这可以使渲染速度提高大约 1.5 到 2.0 倍。

目前并非所有浏览器都支持多视图。如果支持，则应该存在多视图功能。

	scene.getEngine().getCaps().multiview;
注意：多视图渲染渲染到纹理数组而不是标准纹理。在使用自定义着色器、效果或后处理应用后处理时，这可能会导致意外问题（例如，高光层将不起作用）。
## 与 VRExperienceHelper 一起使用
可以通过将 `useMultiview` 选项设置为 来启用多视图为 `true`。

	scene.createDefaultVRExperience({ useMultiview: true });

- [样例](https://playground.babylonjs.com/#SRV2A0)

		var createScene = function () {
		    var scene = new BABYLON.Scene(engine);
		    var camera = new BABYLON.Camera("camera1", BABYLON.Vector3.Zero(), scene);
		
		    BABYLON.SceneLoader.Append("https://www.babylonjs.com/Scenes/hillvalley/", "HillValley.babylon", scene, function () {
		        // 支持多视角的摄像机
		        scene.createDefaultVRExperience({useMultiview: true});
		    });
		
		    return scene;
		};
## 与 VRDeviceOrientationFreeCamera 一起使用在屏幕上显示
通过 [VRCameraMetrics](https://doc.babylonjs.com/typedoc/classes/babylon.vrcamerametrics) 对象中的选项启用：

		// 启用多视图
		var multiviewMetrics = BABYLON.VRCameraMetrics.GetDefault();
		multiviewMetrics.multiviewEnabled = true;
		// 创建摄像头
		var multiviewCamera = new BABYLON.VRDeviceOrientationFreeCamera("", new BABYLON.Vector3(-10, 5, 0), scene, undefined, multiviewMetrics);

- [样例](https://playground.babylonjs.com/#EZDZZV)

		var createScene = function () {
		    var scene = new BABYLON.Scene(engine);
		    var camera = new BABYLON.Camera("camera1", BABYLON.Vector3.Zero(), scene);
		
		    BABYLON.SceneLoader.Append("https://www.babylonjs.com/Scenes/hillvalley/", "HillValley.babylon", scene, function () {
		        // 启用多视图
		        var multiviewMetrics = BABYLON.VRCameraMetrics.GetDefault();
		        multiviewMetrics.multiviewEnabled = true;
		
		        // 创建摄像头
		        var multiviewCamera = new BABYLON.VRDeviceOrientationFreeCamera("", new BABYLON.Vector3(-10, 5, 0), scene, undefined, multiviewMetrics);
		        scene.activeCamera = multiviewCamera;
		        multiviewCamera.attachControl(canvas, true);
		    });
		
		    return scene;
		};

## 如何使用多视图
Babylon.js 可以渲染同一场景的多个视图。
## 有源相机
场景有一个 `activeCamera 属性来定义视点。但是您可以使用如下代码定义许多活动摄像机：

	scene.activeCameras.push(camera);
	scene.activeCameras.push(camera2);			
## Viewports
如果要使用多个摄像机，则需要为每个摄像机指定一个 Viewports ：

	camera.viewport = new BABYLON.Viewport(0.5, 0, 0.5, 1.0);
	camera2.viewport = new BABYLON.Viewport(0, 0, 0.5, 1.0);
它由以下构造函数定义：

	BABYLON.Viewport = function (x, y, width, height);
其中 x, y 是 Viewports 的左下角，后跟其宽度和高度。x、y、宽度和高度的值是介于 0 和 1 之间的数字，分别代表屏幕宽度和高度的分数

- [样例](https://playground.babylonjs.com/#4JXV32)

		var createScene = function() {
			var scene = new BABYLON.Scene(engine);
			scene.clearColor = new BABYLON.Color3( .5, .5, .5);
		
			// 摄像机1
			var camera1 = new BABYLON.ArcRotateCamera("camera1",  3 * Math.PI / 8, 3 * Math.PI / 8, 15, new BABYLON.Vector3(0, 2, 0), scene);
			camera1.attachControl(canvas, true);
		
		    // 摄像机2
		    var camera2 = new BABYLON.ArcRotateCamera("camera1",  5 * Math.PI / 8, 5 * Math.PI / 8, 30, new BABYLON.Vector3(0, 2, 0), scene);
			camera2.attachControl(canvas, true);
		
		    // 创建2个 Viewports
		    camera1.viewport = new BABYLON.Viewport(0, 0.5, 1, 1);
		    camera2.viewport = new BABYLON.Viewport(0, 0, 1, 0.5);
		    
		    scene.activeCameras.push(camera1);
		    scene.activeCameras.push(camera2);
		  
			// 设置2个光源
			var light1 = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(1, 0.5, 0), scene);
			light1.intensity = 0.7;
			var light2 = new BABYLON.HemisphericLight("light2", new BABYLON.Vector3(-1, -0.5, 0), scene);
			light2.intensity = 0.8;
		  
		
			/*********************创建盒子***************/
			
			//多面色材质
			var faceColors = [];
			faceColors[0] = BABYLON.Color3.Blue();
			faceColors[1] = BABYLON.Color3.White()
			faceColors[2] = BABYLON.Color3.Red();
			faceColors[3] = BABYLON.Color3.Black();
			faceColors[4] = BABYLON.Color3.Green();
			faceColors[5] = BABYLON.Color3.Yellow();
		 
		 	// 创建盒子
			var box = BABYLON.MeshBuilder.CreateBox("Box", {faceColors:faceColors, size:2}, scene, true);
		      box.material = new BABYLON.StandardMaterial("", scene);
		      
		      /*******************盒子创建完毕*****************************************/
		
			/**********************创建和绘制坐标轴***************************/
			
			// 设置坐标轴函数
			var showAxis = function(size) {
				var makeTextPlane = function(text, color, size) {
				var dynamicTexture = new BABYLON.DynamicTexture("DynamicTexture", 50, scene, true);
				dynamicTexture.hasAlpha = true;
				dynamicTexture.drawText(text, 5, 40, "bold 36px Arial", color , "transparent", true);
				var plane = new BABYLON.Mesh.CreatePlane("TextPlane", size, scene, true);
				plane.material = new BABYLON.StandardMaterial("TextPlaneMaterial", scene);
				plane.material.backFaceCulling = false;
				plane.material.specularColor = new BABYLON.Color3(0, 0, 0);
				plane.material.diffuseTexture = dynamicTexture;
				return plane;
		    };
		  	// 创建 x 轴
			var axisX = BABYLON.Mesh.CreateLines("axisX", [ 
				new BABYLON.Vector3.Zero(), new BABYLON.Vector3(size, 0, 0), new BABYLON.Vector3(size * 0.95, 0.05 * size, 0), 
				new BABYLON.Vector3(size, 0, 0), new BABYLON.Vector3(size * 0.95, -0.05 * size, 0)
				], scene);
			axisX.color = new BABYLON.Color3(1, 0, 0);
			var xChar = makeTextPlane("X", "red", size / 10);
			xChar.position = new BABYLON.Vector3(0.9 * size, -0.05 * size, 0);
		    
		    // 创建 y 轴	
		    var axisY = BABYLON.Mesh.CreateLines("axisY", [
		        new BABYLON.Vector3.Zero(), new BABYLON.Vector3(0, size, 0), new BABYLON.Vector3( -0.05 * size, size * 0.95, 0), 
		        new BABYLON.Vector3(0, size, 0), new BABYLON.Vector3( 0.05 * size, size * 0.95, 0)
		        ], scene);
		    axisY.color = new BABYLON.Color3(0, 1, 0);
		    var yChar = makeTextPlane("Y", "green", size / 10);
		    yChar.position = new BABYLON.Vector3(0, 0.9 * size, -0.05 * size);
		    
		    // 创建 z 轴
		    var axisZ = BABYLON.Mesh.CreateLines("axisZ", [
		        new BABYLON.Vector3.Zero(), new BABYLON.Vector3(0, 0, size), new BABYLON.Vector3( 0 , -0.05 * size, size * 0.95),
		        new BABYLON.Vector3(0, 0, size), new BABYLON.Vector3( 0, 0.05 * size, size * 0.95)
		        ], scene);
		    axisZ.color = new BABYLON.Color3(0, 0, 1);
		    var zChar = makeTextPlane("Z", "blue", size / 10);
		    zChar.position = new BABYLON.Vector3(0, 0.05 * size, 0.9 * size);
			};
		  
			showAxis(6);
		 
			/***************************************************************/
			return scene;
		  
		}

					