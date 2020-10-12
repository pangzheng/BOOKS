# nvdia ngx
## 1 NGX概述
NVIDIA NGX 使您可以轻松地将基于 AI 的预构建功能集成到您的应用程序中。NVIDIA 将添加新功能，并随着时间的推移更新现有功能。更新现有功能后，NGX 基础结构将在使用该功能的所有客户端上更新该功能。

组成系统的三个主要组件：

- NGX SDK

	NGX SDK提供 CUDA 和 DX11/12 ，供应用程序访问 AI 功能。
- NGX 核心运行时

	所有运行时模块均随支持 RTX 硬件的 NVIDIA 图形驱动程序一起提供。在高级驱动程序安装期间，该模块称为 NGX Core。
- NGX 更新模块

	该模块确保 NGX 集成的应用程序始终使用最新版本的 NGX 功能。## 2 入门
### 2.1. 要求
确保满足以下要求：

- 从 [NVIDIA开发人员专区](https://developer.nvidia.com/rtx/ngx) 下载SDK 。
- 注册后，您将收到一个安装程序，它将安装SDK资料和示例。默认情况下，SDK将安装到 `c:\ProgramData\NVIDIA Corporation\NGX SDK` 目录。

为了运行 NGX 应用程序，需要执行以下操作：

- 操作系统 Windows 10 v1709（2017年秋季 Creators Update 64位）或更高版本的 Windows PC
- 硬件 NVIDIA RTX GPU（Quadro，Titan或GeForce）
- 最新的 NVIDIA 图形驱动程序，最低 410.63

将 NGX SDK 集成到您的应用程序中的最小开发环境是：

- 带有 Windows 10 SDK 的 Microsoft Visual Studio 2015 SP3（10.0.10586）

### 2.2 建立样本
要构建 SDK 样本，请在中打开 NGX 样本解决方案文件 `Samples\NGX Samples\`，然后从 `Build Menu` 菜单中选择 `Build Menu` 。从配置下拉列表中选择 `Release x64`。

构建会将示例可执行文件和所有必需的DLL放入`x64\Release`。要在执行一个或多个示例时在调试器中逐步查看源代码，请选择`Debug x64` 配置。在这种情况下，将构建的样本和所有必需的 DLL 放入 `x64\Debug`。

### 2.3 运行样本
这些示例将构建四个可执行文件，它们位于 `<NGX SDK>\Samples\NGX Samples\x64\Release` 目录：

- isr.exe：Image UpRes(图像处理)
- vsr.exe：Video UpRes(视频处理)
- inpaint.exe：In-Painting(绘图处理)
- Slowmo.exe：Video slo-mo(视频慢镜头处理)

它们都是命令行应用程序，并且以类似的方式运行。不使用命令运行可执行文件以查看选项。

`InPainting` 示例的示例命令类似于：

	x64\Release>inpaint.exe --input 
	..\..\SampleImages\inpaint_input.png --mask 
	..\..\SampleImages\inpaint_mask.png --model 0  --output d:\test.png

## 3 将NGX与您的应用程序集成
### 3.1 将 NGX 添加到您的应用程序
NGX SDK 附带两个头文件：

- `nvsdk_ngx.h`
- `nvsdk_ngx_defs.h`

这两个文件都位于 `<NGX SDK>\include` 夹。

在您的项目中，确保您包括 `nvsdk_ngx.h`。除了包括 NGX 头文件之外，您的项目还应链接到：

- `nvsdk_ngx_s.lib` （如果是静态运行时库（`/MT`）在您的项目中使用了链接）或
- `nvsdk_ngx_d.lib`（如果动态运行时库（`/MD`）在您的项目中使用了链接）

这两个文件都位于 `<NGX SDK>\lib\x64` 文件夹（NGX仅作为64位库提供）。

在开发过程中，复制 `nvngx_*.dll` 从 `<NGX SDK>\bin\features` 到可执行文件或 DLL 所在的文件夹，以便 NGX 运行时可以找到 DLL。有关应如何与应用程序一起分发 NGX 的信息，[请参阅与应用程序一起分发NGX功能](https://docs.nvidia.com/rtx/ngx/programming-guide/index.html#distributing)。

### 3.2 初始化 NGX
NGX SDK 支持 D3D11，D3D12 和 CUDA API。一般来说，每个 NGX 功能都支持一个 API，但是 NGX 并没有限制可以支持的API 数量。公开可用的初始功能支持 CUDA。有关更多信息，请参阅[NGX功能](https://docs.nvidia.com/rtx/ngx/programming-guide/index.html#ngx-features)，该功能描述了每个功能支持的所有API。

	警告：
	NGX API 不是线程安全的。客户端应用程序必须确保根据需要强制执行线程安全性。从多个线程调用 NGX API 可能导致无法预料的行为。对于使用 DirectX 的功能，NGX API 会保留直接 D3D11 上下文的状态，但是 D3D12 命令列表并非如此。对于使用DirectX 12 和 D3D12 命令列表的所有应用程序，客户端应用程序必须根据需要管理命令列表状态。

NGX SDK 中的所有调用对于受支持的 API（D3D11，D3D12和CUDA）都是相似的，因此可以使用与以下示例代码匹配或非常相似的代码来初始化所有功能。确保您使用的调用与您的应用程序正在使用的API相匹配。API 之间的任何互操作，例如 D3D11 到CUDA，都必须由 NGX SDK 之外的应用程序处理。

要初始化 SDK 实例，请使用以下方法之一：

	// NVSDK_NGX_Init
	// -------------------------------------
	// 
	// InApplicationId:
	//      Unique Id provided by NVIDIA
	//
	// InApplicationDataPath:
	//      Folder to store logs and other temporary files (write access required)
	//
	// InDevice: [d3d11/12 only]
	//      DirectX device to use
	//
	// DESCRIPTION:
	//      Initializes new SDK instance.
	//
	#ifdef __d3d11_h__
	NVSDK_NGX_API NVSDK_NGX_Result  NVSDK_CONV NVSDK_NGX_D3D11_Init(unsigned long long InApplicationId, const wchar_t *InApplicationDataPath, ID3D11Device *InDevice, NVSDK_NGX_Version InSDKVersion = NVSDK_NGX_Version_API);
	#elif defined __d3d12_h__
	NVSDK_NGX_API NVSDK_NGX_Result  NVSDK_CONV NVSDK_NGX_D3D12_Init(unsigned long long InApplicationId, const wchar_t *InApplicationDataPath, ID3D12Device *InDevice, NVSDK_NGX_Version InSDKVersion = NVSDK_NGX_Version_API);
	#else
	NVSDK_NGX_API NVSDK_NGX_Result  NVSDK_CONV NVSDK_NGX_CUDA_Init(unsigned long long InApplicationId, const wchar_t *InApplicationDataPath,NVSDK_NGX_Version InSDKVersion = NVSDK_NGX_Version_API);
	#endif

	注意：所有 NGX 调用都将 “应用程序ID” 作为第一个参数，主要用于具有特定于应用程序调整功能的功能。

在 NVIDIA 为您分配应用程序ID之前，请使用0。在发布集成了 NGX 的产品之前，请在此处通知 NVIDIA 您正在使用 NGX：`https://developer.nvidia.com/sw-notification`，NVIDIA 将为您提供 NGX 兼容的应用程序ID，以集成到您的应用程序中

创建 SDK 实例后，根据应用程序的需要创建功能。在 1.0.0 版中，支持以下基于深度学习的功能

	enum NVSDK_NGX_Feature
	{ 
	    // DLInPainting
	    NVSDK_NGX_Feature_InPainting,
	    // DLISR
	    NVSDK_NGX_Feature_ImageSuperResolution,
	    // DLSlowMo
	    NVSDK_NGX_Feature_SlowMotion,
	    // DLVSR
	    NVSDK_NGX_Feature_VideoSuperResolution,
	    // New features go here
	    NVSDK_NGX_Feature_Count
	};
注意

	并非所有 NGX 功能都具有 D3D11/D3D12 和 CUDA 实现。检查返回码或阅读特定功能的文档，以确定可用的实现。

#### 3.2.1 验证中
成功初始化表明目标系统能够运行 NGX 功能。但是，每个功能都可能具有其他依赖性，例如，最低驱动程序版本等等；因此，最好检查您要使用的特定功能是否可用。为此，NGX 提供了 `NVSDK_NGX_Parameter` 该接口可用于查询 NGX 运行时提供的只读参数，并可使用以下 API 获得该接口：

	// NVSDK_NGX_GetParameters
	// ----------------------------------------------------------
	//
	// OutParameters:
	//      Parameters interface used to set any parameter needed by the SDK
	//
	// DESCRIPTION:
	//      This interface allows simple parameter setup using named fields.
	//      For example one can set width by calling Set(NVSDK_NGX_Parameter_Width,100) or
	//      provide CUDA buffer pointer by calling Set("Color.GPUAllocation",cudaBuffer)
	//      For more details please see sample code. Please note that allocated memory
	//      will be freed by NGX so free/delete operator should NOT be called.
	//
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D11_GetParameters(NVSDK_NGX_Parameter **OutParameters);
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D12_GetParameters(NVSDK_NGX_Parameter **OutParameters);
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_CUDA_GetParameters(NVSDK_NGX_Parameter **OutParameters);	

例如，检查 `NVSDK_NGX_Feature_InPainting` 在系统上可用，请发出以下命令：

	int InPaintingSupported = 0;
	NVSDK_NGX_Parameter *Params = nullptr;
	NVSDK_NGX_CUDA_GetParameters(&Params);
	Params->Get(NVSDK_NGX_Parameter_InPainting_Available,&InPaintingSupported );
	if(InPaintingSupported)
	{
	  // OK to use InPainting feature
	}

### 3.3 设置和获取参数
每个功能都需要在创建功能之前设置某些参数。这可以通过使用 `NVSDK_NGX_Parameter` 接口。

参数指定为 `{name,value}` 对，例如：

	NVSDK_NGX_Parameter *Params = nullptr;
	NVSDK_NGX_CUDA_GetParameters(&Params);
	Params->Set(NVSDK_NGX_Parameter_Width,100);
	
有关应为特定功能设置哪些参数的更多详细信息，请参见 [NGX功能](https://docs.nvidia.com/rtx/ngx/programming-guide/index.html#ngx-features)。

### 3.4 暂存缓冲区设置
设置完参数后，对于每个功能，请通过调用以下命令检查需要多少 GPU 暂存器

	// NVSDK_NGX_GetScratchBufferSize
	// ----------------------------------------------------------
	//
	// InFeatureId:
	//      AI feature in question
	//
	// InParameters:
	//      Parameters used by the feature to help estimate scratch buffer size
	//
	// OutSizeInBytes:
	//      Number of bytes needed for the scratch buffer for the specified feature.
	//
	// DESCRIPTION:
	//      SDK needs a buffer of a certain size provided by the client in
	//      order to initialize AI feature. Once feature is no longer
	//      needed buffer can be released. It is safe to reuse the same
	//      scratch buffer for different features as long as minimum size
	//      requirement is met for all features. Please note that some
	//      features might not need a scratch buffer so return size of 0
	//      is completely valid.
	//
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D11_GetScratchBufferSize(NVSDK_NGX_Feature InFeatureId, const NVSDK_NGX_Parameter *InParameters, size_t *OutSizeInBytes);
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D12_GetScratchBufferSize(NVSDK_NGX_Feature InFeatureId, const NVSDK_NGX_Parameter *InParameters, size_t *OutSizeInBytes);
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_CUDA_GetScratchBufferSize(NVSDK_NGX_Feature InFeatureId, const NVSDK_NGX_Parameter *InParameters, size_t *OutSizeInBytes);	

该应用程序负责分配所请求大小的暂存缓冲区，并在创建特定功能时将其作为参数传递（更多详细信息可以在 NGX SDK 的源代码示例中找到）。

	注意： SDK可以返回 0 作为特定功能所需的暂存缓冲区大小。

### 3.5 特征创建
分配临时缓冲区并设置所有必要参数后，创建功能。以下代码可用于创建功能：

	// NVSDK_NGX_CreateFeature
	// -------------------------------------
	//
	// InCmdList:[d3d12 only]
	//      Command list to use to execute GPU commands. Must be:
	//      - Open and recording 
	//      - With node mask including the device provided in NVSDK_NGX_D3D12_Init
	//      - Execute on non-copy command queue.
	// InDevCtx: [d3d11 only]
	//      Device context to use to execute GPU commands
	//
	// InFeatureID:
	//      AI feature to initialize
	//
	// InParameters:
	//      List of parameters 
	// 
	// OutHandle:
	//      Handle which uniquely identifies the feature. If feature with
	//      provided parameters already exists the "already exists" error code is returned.
	//
	// DESCRIPTION:
	//      Each feature needs to be created before it can be used. 
	//      Refer to the sample code to find out which input parameters
	//      are needed to create specific feature.
	//
	
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D11_CreateFeature(ID3D11DeviceContext *InDevCtx, NVSDK_NGX_Feature InFeatureID, const NVSDK_NGX_Parameter *InParameters, NVSDK_NGX_Handle **OutHandle);
	
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D12_CreateFeature(ID3D12GraphicsCommandList *InCmdList, NVSDK_NGX_Feature InFeatureID, const NVSDK_NGX_Parameter *InParameters, NVSDK_NGX_Handle **OutHandle);
	
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_CUDA_CreateFeature(NVSDK_NGX_Feature InFeatureID, const NVSDK_NGX_Parameter *InParameters, NVSDK_NGX_Handle **OutHandle);

注意： Feature handles 不使用参考计数。如果请求创建具有与现有功能匹配的参数的新功能，则 SDK 将返回一个已经存在错误代码。

### 3.6 功能评估
通过对特定深度学习模型执行推理来评估功能。这可以通过调用：

		// NVSDK_NGX_EvaluateFeature
		// -------------------------------------
		//
		// InCmdList:[d3d12 only]
		//      Command list to use to execute GPU commands. Must be:
		//      - Open and recording 
		//      - With node mask including the device provided in NVSDK_NGX_D3D12_Init
		//      - Execute on non-copy command queue.
		// InDevCtx: [d3d11 only]
		//      Device context to use to execute GPU commands
		//
		// InFeatureHandle:
		//      Handle representing feature to be evaluated
		// 
		// InParameters:
		//      List of parameters required to evaluate feature
		//
		// InCallback:
		//      Optional callback for features which might take longer
		//      to execute. If specified SDK will call it with progress
		//      values in range 0.0f - 1.0f. Client application can indicate
		//      that evaluation should be cancelled by setting OutShouldCancel
		//      to true.
		//
		// DESCRIPTION:
		//      Evaluates given feature using the provided parameters and
		//      pre-trained NN. Please note that for most features
		//      it can be beneficial to pass as many input buffers and parameters
		//      as possible (for example provide all render targets like color, albedo, normals, depth etc)
		//
		typedef void (NVSDK_CONV *PFN_NVSDK_NGX_ProgressCallback)(float InCurrentProgress, bool &OutShouldCancel);
		
		NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D11_EvaluateFeature(ID3D11DeviceContext *InDevCtx, const NVSDK_NGX_Handle *InFeatureHandle, const NVSDK_NGX_Parameter *InParameters, PFN_NVSDK_NGX_ProgressCallback InCallback = nullptr);
		
		NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D12_EvaluateFeature(ID3D12GraphicsCommandList *InCmdList, const NVSDK_NGX_Handle *InFeatureHandle, const NVSDK_NGX_Parameter *InParameters, PFN_NVSDK_NGX_ProgressCallback InCallback = nullptr);
		
		NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_CUDA_EvaluateFeature(const NVSDK_NGX_Handle *InFeatureHandle, const NVSDK_NGX_Parameter *InParameters, PFN_NVSDK_NGX_ProgressCallback InCallback = nullptr);

警告：

	DirectX：NGX 将修改 D3D12 命令列表状态，因此，调用过程应在调用 NGX 评估功能之前和之后保存或恢复其自己的 D3D 状态。
	
在评估要素输入缓冲区（例如颜色，反照率，法线，深度等）时，必须将其作为参数提供（要么 ID3D * 资源或 CUDA 内存缓冲区）。有关示例代码，请参见[NGX功能](https://docs.nvidia.com/rtx/ngx/programming-guide/index.html#ngx-features)。

注意

	某些功能不是实时的，因此可能需要几秒钟才能完成评估（例如，具有高分辨率视频源的 DLSlowMo）。进度回调应用于向用户提供反馈，并根据需要允许取消。

### 3.7 功能释放
当不再需要某个功能时，应通过调用以下方法将其释放：

	// NVSDK_NGX_Release
	// -------------------------------------
	// 
	// InHandle:
	//      Handle to feature to be released
	//
	// DESCRIPTION:
	//      Releases feature with a given handle.
	//      Handles are not reference counted so
	//      after this call it is invalid to use provided handle.
	//
	
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D11_ReleaseFeature(NVSDK_NGX_Handle *InHandle);
	
	
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D12_ReleaseFeature(NVSDK_NGX_Handle *InHandle);
	
	
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_CUDA_ReleaseFeature(NVSDK_NGX_Handle *InHandle);	

释放后，功能将无法再使用	

### 3.8 关掉
要释放 SDK 实例及其分配的所有资源，请使用以下方法：

	// NVSDK_NGX_Shutdown
	// -------------------------------------
	// 
	// DESCRIPTION:
	//      Shuts down the current SDK instance and releases all resources.
	//
	
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D11_Shutdown();
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_D3D12_Shutdown();
	NVSDK_NGX_API NVSDK_NGX_Result NVSDK_CONV NVSDK_NGX_CUDA_Shutdown();

## 4 在您的应用程序中分发 NGX 功能
NGX SDK 包含该功能的 DLL，这些 DLL 必须随您的应用程序一起分发。这些基本 DLL 位于 `./bin/features` 文件夹，应与应用程序的可执行文件（如果要构建插件，则为DLL）安装在同一文件夹中。

注意：您只需要分发应用程序正在使用的功能的 DLL。例如，如果您的应用程序仅使用 `InPainting`，则应仅包括 `nvngx_inpaint.dll` 与其余的二进制文件。

功能和 dl l对应列表

- AI UpRes -- Stills
	
	nvngx_isr.dll
- AI UpRes -- Video

	nvngx_vsr.dll
- AI InPainiting

	nvngx_inpaint.dll
- AI SloMo

	nvngx_slomo.dll	
	
应用程序的安装程序应以与应用程序其他组件相同的方式对待这些 DLL，并在卸载时将其删除。

## 5. NGX功能
### 5.1 DLSS-深度学习超级采样（仅DirectX）
DLSS 是由 NVIDIA Research 开发的一项新技术，可将非常高质量的图像锐化，增强和可选的超分辨率应用于 3D 引擎和游戏渲染的帧。此技术使用自动编码器从帧中提取高维特征，然后以增加的细节和更高的分辨率重建图像。

DLSS 当前处于封闭 Beta 版中。NVIDIA 正在与合作伙伴合作，以在一系列 DirectX 游戏中启用 DLSS。有关更多信息，请通过`NGXSupport@nvidia.com` 与我们联系。

### 5.2 DLISR-图像超分辨率（仅限CUDA）
- 此功能做什么？

	DLISR 功能使用深度学习推理来生成其他像素，从而将图像的分辨率提高了2倍，4倍或8倍。
	
	此功能的预期性能将以每帧秒数为单位，因此，此版本最适合静止图像或脱机处理。
- NGX DLISR 样本

	NGX SDK 中包含的 NGX ISR 示例应用程序演示了如何从较低分辨率的 8位 RGB 输入图像生成 8位 RGB格式的高分辨率图像。本示例使用 `getRgbImage()` 和 `putRgbImage()` 在 OpenCV 之上实现的辅助函数，用于基于文件扩展名读取和写入最常见的图像格式。

以下命令读取  `input.png` 文件并使用x2缩放因子生成图像以 `output.png`。比例因子可以是2、4或8。

	isr.exe --input input.png -–factor 2 --output output.png
	
- 支持参数

|参数|描述| 
---|---|---
Width |输入图像CUDA缓冲区的宽度
Height |输入图像CUDA缓冲区的高度
Scratch |分配的暂存缓冲区 cudaMalloc()
Scratch_SizeInBytes |暂存缓冲区大小。
Color |线性内存的 CUDA 缓冲区（cudaMalloc()）包含输入图像的8位RGB像素。
Color_Format |输入图像的色彩精度， `NVSDK_NGX_Buffer_Format_RGB8UI`
Color_SizeInBytes |输入图像的大小（以字节为单位）。
Output |线性内存的CUDA缓冲区（cudaMalloc()）返回输出图像的8位RGB像素（4x，16x或256x，即源缓冲区的大小）
Output_Format |	输出图像的色彩精度， `NVSDK_NGX_Buffer_Format_RGB8UI`
Output_SizeInBytes |输出图像的大小（以字节为单位）。
Scale |超分辨率比例因子（2,4或8）

- 样例代码

	以下代码段显示了如何在NGX中使用DLISR功能。涉及以下步骤：
	
	1. 使用创建 NGX 上下文 `NVSDK_NGX_CUDA_Init()`。
	2. 获取用于使用以下命令设置输入参数的结构 `NVSDK_NGX_CUDA_GetParameters()`。
	3. 使用以下命令获取 DLISR 算法所需的暂存缓冲区大小： `NVSDK_NGX_CUDA_GetScratchBufferSize()`，这取决于输入视频的宽度，高度和所选的比例因子。
	5. 使用创建临时缓冲区 `cudaMalloc()`。
	6. 使用创建 NGX DLISR 功能 `NVSDK_NGX_CUDA_CreateFeature()`。
	7. 使用运行 DLISR `NVSDK_NGX_CUDA_EvaluateFeature()`。
	8. 使用以下方法释放 NGX DLISR 功能 `NVSDK_NGX_CUDA_ReleaseFeature()`。
	9. 使用释放NGX资源 `NVSDK_NGX_CUDA_Shutdown()`。

			// Initialize NGX.
			CK_NGX(NVSDK_NGX_CUDA_Init(app_id, L"./", NVSDK_NGX_Version_API));
			
			// Get the parameter block.
			CK_NGX(NVSDK_NGX_CUDA_GetParameters(&params));
			
			// Verify feature is supported
			int Supported = 0;
			params->Get(NVSDK_NGX_Parameter_ImageSuperResolution_Available, &Supported);
			if (!Supported)
			{
			    std::cerr << "NVSDK_NGX_Feature_ImageSuperResolution Unavailable on this System" << std::endl;
			    return 1;
			}
			
			// Set the default hyperparameters for inference.
			params->Set(NVSDK_NGX_Parameter_Width, in_image_width);
			params->Set(NVSDK_NGX_Parameter_Height, in_image_height);
			params->Set(NVSDK_NGX_Parameter_Scale, myAppParams.uprez_factor);
			
			// Get the scratch buffer size and create the scratch allocation.
			// (if required)
			size_t byteSize{ 0u };
			void *scratchBuffer{ nullptr };
			CK_NGX(NVSDK_NGX_CUDA_GetScratchBufferSize(NVSDK_NGX_Feature_ImageSuperResolution, params, &byteSize));
			cudaMalloc(&scratchBuffer, byteSize > 0u ? byteSize : 1u); 
			
			// Update the parameter block with the scratch space metadata.:
			params->Set(NVSDK_NGX_Parameter_Scratch, scratchBuffer);
			params->Set(NVSDK_NGX_Parameter_Scratch_SizeInBytes, (uint32_t)byteSize);
			
			// Create the feature
			CK_NGX(NVSDK_NGX_CUDA_CreateFeature(NVSDK_NGX_Feature_ImageSuperResolution, params, &DUHandle));
			
			// Pass the pointers to the GPU allocations to the
			// parameter block along with the format and size.
			params->Set(NVSDK_NGX_Parameter_Color_SizeInBytes, in_image_row_bytes * in_image_height);
			params->Set(NVSDK_NGX_Parameter_Color_Format, NVSDK_NGX_Buffer_Format_RGB8UI);
			params->Set(NVSDK_NGX_Parameter_Color, in_image_dev_ptr);
			params->Set(NVSDK_NGX_Parameter_Output_SizeInBytes, out_image_row_bytes * out_image_height);
			params->Set(NVSDK_NGX_Parameter_Output_Format, NVSDK_NGX_Buffer_Format_RGB8UI);
			params->Set(NVSDK_NGX_Parameter_Output, out_image_dev_ptr);
			
			//Execute the feature.
			CK_NGX(NVSDK_NGX_CUDA_EvaluateFeature(DUHandle, params, NGXTestCallback));
			
			//Tear down the feature.
			CK_NGX(NVSDK_NGX_CUDA_ReleaseFeature(DUHandle));

### 5.3 DLVSR（仅限CUDA）
- 此功能做什么？

	视频超分辨率（VSR）是一种从低分辨率（LR）视频帧构造高分辨率（HR）视频帧的技术，从而改善了帧内的细节并消除了由低分辨率成像过程引起的伪像相机。

	VSR 可用于各种应用程序，例如提高以较低分辨率存档的旧内容的分辨率或生成使用高清分辨率摄像机捕获的用于高分辨率广播（UHD）的视频。

	常规的放大算法通过使用插值算法来放大视频，但是无法重新生成可能由 HR 摄像机捕获的细节。

	深度学习视频超分辨率（DLVSR）使用机器学习方法从较低分辨率的视频生成高分辨率的视频，并引入无法使用常规放大算法生成的细节。
	
	预期的性能应为每帧几毫秒，因此，此版本最适合实时应用。
- NGX DLVSR 样本

	NGX SDK 中包含的 NGX VSR 示例应用程序：

	- 演示了直接从 mp4 容器中的压缩文件从低分辨率视频生成高分辨率视频的过程。
	- 利用 FFmpeg 多路分解器，并使用 GPU 上的 NVDEC 引擎对其进行解码。它可以快速解码视频文件，生成 RGB 帧和输出，可以直接发送到 NGX VSR 插件。然后，它将 VSR 输出编码，并将其放入 mp4 容器中。

		有关 GPU 上 NVDEC/NVENC 引擎支持的解码/编码分辨率的更多信息，请参阅 [NVIDIA Video Codec SDK](https://developer.nvidia.com/nvidia-video-codec-sdk)。如果 NVDEC/NVENC 不支持分辨率或编解码器，请使用软件解码器，将生成的 RGB 帧传输到视频内存中，以发送到 NGX VSR 插件，然后将生成的高分辨率 RGB 传输到系统内存中，以用于软件编码器。

以下命令读取 `input.mp4` 文件，并生成具有 x2 缩放因子的视频，以 `output.mp4`。比例因子可以是2或3。

	DLVSR.exe --input input.mp4 –factor 2 -output output.mp4

- 支持参数
	
	|参数|描述| 
	---|---|---
	Width |输入视频的宽度
	Height |输入视频的高度
	Scratch |分配的 CUDA 暂存缓冲区
	Scratch_SizeInBytes | CUDA 暂存缓冲区大小（以字节为单位）
	Color |线性内存的 CUDA 缓冲区。
	Color_Format |输入图像的色彩精度， `NVSDK_NGX_Buffer_Format_RGB16F` 或 `NVSDK_NGX_Buffer_Format_RGB32F`
	Color_SizeInBytes |输入视频的大小（以字节为单位）。
	Output | CUDA 输出视频暂存缓冲区
	Output_Format |	输出视频的色彩精度， `NVSDK_NGX_Buffer_Format_RGB16F` 或 `NVSDK_NGX_Buffer_Format_RGB32F`
	Output_SizeInBytes |输出视频的大小（以字节为单位）。
	Scale |超分辨率比例因子（2,3）

- 样例代码

以下代码段显示了如何在NGX中使用DLVSR功能。涉及以下步骤：

- 使用创建 NGX 上下文 `NVSDK_NGX_CUDA_Init()`。
- 获取用于使用以下命令设置输入参数的结构 `NVSDK_NGX_CUDA_GetParameters()`。
- 使用以下命令获取 DLSLOMO 算法所需的暂存缓冲区大小： `NVSDK_NGX_CUDA_GetScratchBufferSize()`，这取决于输入视频的宽度，高度和所选的比例因子。
- 使用创建刮痕 `cudaMalloc()`。
- 使用创建 NGX DLSLOMO 功能 `NVSDK_NGX_CUDA_CreateFeature()`。
- 使用运行 DLSLOMO `NVSDK_NGX_CUDA_EvaluateFeature()`。
- 使用以下方法释放 NGX DLSLOMO 功能 `NVSDK_NGX_CUDA_ReleaseFeature()`。
- 使用释放 NGX 资源 `NVSDK_NGX_CUDA_Shutdown()`


		// IMPORTANT: ALWAYS CHECK FOR RETURN CODES TO ENSURE NGX CALLS ARE NOT FAILING
		
		// Should be done when application is initialized
		NVSDK_NGX_Result Status = NVSDK_NGX_CUDA_Init(MyApplicationID, MyRWAccessFolder,  NVSDK_NGX_Version_API));
		if(NVSDK_NGX_FAILED(Status))
		{
		  // Check error code, if NGX is not available on this machine disable it
		}
		
		// Once render target size is known we can create feature
		NVSDK_NGX_Handle *FeatureHandle = nullptr;
		NVSDK_NGX_Parameter *Params = nullptr;
		Status = NVSDK_NGX_CUDA_GetParameters(&Params);
		if(NVSDK_NGX_FAILED(Status)) { // Handle error };
		Params->Set(NVSDK_NGX_Parameter_Width, Width);
		Params->Set(NVSDK_NGX_Parameter_Height, Height);
		Params->Set(NVSDK_NGX_Parameter_Scale, ScaleFactor); // Must be 2x or 3x
		size_t ByteSize;
		Status = NVSDK_NGX_CUDA_GetScratchBufferSize(NVSDK_NGX_Feature_VideoSuperResolution, 
		Params, &ByteSize);
		if(NVSDK_NGX_FAILED(Status)) { // Handle error };
		void *ScratchBuffer = MyEngine::CreateCUDAResource(ByteSize);
		Params->Set(NVSDK_NGX_Parameter_Scratch, ScratchBuffer);
		Params->Set(NVSDK_NGX_Parameter_Scratch_SizeInBytes, (uint32_t)ByteSize);
		Status = NVSDK_NGX_CUDA_CreateFeature(NVSDK_NGX_Feature_VideoSuperResolution, Params, &FeatureHandle);
		if(NVSDK_NGX_FAILED(Status)) { // Handle error };
		
		// When needed call to upscale your input
		// IMPORTANT: input needs to be 3 channel fp32 in range 0.0-255.0
		Params->Set(NVSDK_NGX_Parameter_Color_SizeInBytes, ColorImageSize);
		Params->Set(NVSDK_NGX_Parameter_Color_Format, NVSDK_NGX_Buffer_Format_RGB32F);
		Params->Set(NVSDK_NGX_Parameter_Color, ColorCUDABuffer);
		Params->Set(NVSDK_NGX_Parameter_Output_SizeInBytes, UpscaledImageSize);
		Params->Set(NVSDK_NGX_Parameter_Output_Format, NVSDK_NGX_Buffer_Format_RGB32F);
		Params->Set(NVSDK_NGX_Parameter_Output, UpscaledCUDABuffer);
		Status = NVSDK_NGX_CUDA_EvaluateFeature(FeatureHandle, Params);
		if(NVSDK_NGX_FAILED(Status)) { // Handle error };
		
		// When feature is no longer needed
		Status = NVSDK_NGX_CUDA_ReleaseFeature(FeatureHandle);
		if(NVSDK_NGX_FAILED(Status)) { // Handle error };
		
		// During shutdown
		Status = NVSDK_NGX_CUDA_Shutdown();
		if(NVSDK_NGX_FAILED(Status)) { // Handle error };

### 5.4 DLSLOWMO（仅限CUDA）
- 此功能做什么？

	DLSLOWMO 功能使用深度学习从原始输入视频创建慢动作视频。此功能在原始输入视频的后续帧之间引入中间帧。
- NGX DLSLOMO 样本

	NGX SDK 中包含的 NGX DLSLOWMO 示例应用程序演示了如何通过直接从 mp4 容器中的压缩文件通过 mp4 格式的视频中的每对帧之间的深度学习推断来生成中间帧。此示例使用 FFmpeg 多路分解器，并使用 GPU 上的 NVDEC 引擎对其进行解码。它可以快速解码视频文件，生成 RGB 帧和输出，可以直接发送到 NGX VSR 插件。

有关 GPU 上 NVDEC 引擎解码分辨率的更多信息，请参阅 [NVIDIA Video Codec SDK](https://developer.nvidia.com/nvidia-video-codec-sdk)。如果 NVDEC 不支持分辨率或编解码器，请使用软件解码器并将生成的 RGB 帧传输到视频内存中，以发送到 NGX VSR 插件。

以下命令读取 `input.mp4` 文件，并生成新的慢动作视频，其输出帧数是 `output.mp4` 的两倍。帧数可以是任何整数，但受可用 GPU 内存的限制。

	slomo.exe --input input.mp4 -–frames 2 --output output.mp4

- 支持参数
	
	|参数|描述| 
	---|---|---
	Width |视频输入帧CUDA缓冲区的宽度。
	Height |视频输入帧CUDA缓冲区的高度。
	Scratch |分配的 CUDA 暂存缓冲区
	Scratch_SizeInBytes | CUDA 暂存缓冲区大小（以字节为单位）
	Input1 |线性内存的CUDA缓冲区（与 cudaMalloc（））包含平面 CHW 格式的第一个输入帧的 FP32 RGB 像素。
	Input1_Format |	第一个输入框的色彩精度， NVSDK_NGX_Buffer_Format_RGB32F。
	Input1_SizeInBytes |	第一个输入帧字节的大小。
	Input2 |线性内存的CUDA缓冲区（与 cudaMalloc（））包含平面 CHW 格式的第二个输入帧的 FP32 RGB 像素。
	Input2_Format |第二个输入框的色彩精度， NVSDK_NGX_Buffer_Format_RGB32F。
	Input2_SizeInBytes |	第二个输入帧的大小（以字节为单位）。
	OutputX | 线性内存的CUDA缓冲区（与 cudaMalloc（））以平面CHW格式返回输出帧X的FP32 RGB像素。
	OutputX_Format |色彩精度输出框X NVSDK_NGX_Buffer_Format_RGB32F。
	OutputX_SizeInBytes |输出帧X的大小（以字节为单位）
	NumFrames |在原始视频的两个帧之间生成的新帧数。

- 样例代码

	以下代码段显示了如何在NGX中使用DLSLOMO功能。涉及以下步骤：

	- 使用创建 NGX 上下文 `NVSDK_NGX_CUDA_Init()`。
	- 获取用于使用以下命令设置输入参数的结构 `NVSDK_NGX_CUDA_GetParameters()`。
	- 使用以下命令获取 DLSLOMO 算法所需的暂存缓冲区大小： `NVSDK_NGX_CUDA_GetScratchBufferSize()`，这取决于输入视频的宽度，高度和所选的比例因子。
	- 使用创建刮痕 `cudaMalloc()`。
	- 使用创建 NGX DLSLOMO 功能 `NVSDK_NGX_CUDA_CreateFeature()`。
	- 使用运行 DLSLOMO `NVSDK_NGX_CUDA_EvaluateFeature()`。
	- 使用以下方法释放 NGX DLSLOMO 功能 `NVSDK_NGX_CUDA_ReleaseFeature()`。
	- 使用释放NGX资源 `NVSDK_NGX_CUDA_Shutdown()`。

			CK_NGX(NVSDK_NGX_CUDA_Init(m_ulAppId, m_wcDataPath, m_SDKVersion));
			CK_NGX(NVSDK_NGX_CUDA_GetParameters(&m_pParams));
			
			m_pParams->Set(NVSDK_NGX_Parameter_Width, Width);
			m_pParams->Set(NVSDK_NGX_Parameter_Height, Height);
			m_pParams->Set(NVSDK_NGX_Parameter_NumFrames, Frames);
			        CK_NGX(NVSDK_NGX_CUDA_GetScratchBufferSize(NVSDK_NGX_Feature_SlowMotion, m_pParams, &m_uScratchSize));
			
			if (m_uScratchSize)
			{
			    CK_CUDA(cudaMalloc(&m_ScratchBuffer, m_uScratchSize));
			    m_pParams->Set(NVSDK_NGX_Parameter_Scratch, m_ScratchBuffer);
			    m_pParams->Set(NVSDK_NGX_Parameter_Scratch_SizeInBytes, m_uScratchSize);
			}
			
			CK_NGX(NVSDK_NGX_CUDA_CreateFeature(NVSDK_NGX_Feature_SlowMotion, m_pParams, &m_hSloMo));
			
			m_pParams->Set(NVSDK_NGX_Parameter_Input1_SizeInBytes, InputSize);
			m_pParams->Set(NVSDK_NGX_Parameter_Input1_Format, NVSDK_NGX_Buffer_Format_RGB32F);
			m_pParams->Set(NVSDK_NGX_Parameter_Input1, Input1);
			m_pParams->Set(NVSDK_NGX_Parameter_Input2_SizeInBytes, InputSize);
			m_pParams->Set(NVSDK_NGX_Parameter_Input2_Format, NVSDK_NGX_Buffer_Format_RGB32F);
			m_pParams->Set(NVSDK_NGX_Parameter_Input2, Input2);
			
			for (uint32_t i = 0; i < Frames; i++)
			{
			     const std::string outputBufName{ std::string(NVSDK_NGX_Parameter_Output) + std::to_string(i) };
			     const std::string outputBufFormat{ outputBufName + std::string(".") + std::string(NVSDK_NGX_Parameter_Format) };
			     const std::string outputBufSizeInBytes{ outputBufName + std::string(".") + std::string(NVSDK_NGX_Parameter_SizeInBytes) };
			     m_pParams->Set(outputBufName.c_str(), Output[i].get());
			     m_pParams->Set(outputBufFormat.c_str(), NVSDK_NGX_Buffer_Format_RGB32F);
			     m_pParams->Set(outputBufSizeInBytes.c_str(), OutputSize);
			}
			CK_NGX(NVSDK_NGX_CUDA_EvaluateFeature(m_hSloMo, m_pParams, NGXTestCallback)); 
			
			CK_NGX(NVSDK_NGX_CUDA_Shutdown());

### 5.5 图像修补（仅限CUDA）
- 此功能做什么？

	DLINPAINTING 功能使用深度学习来删除由蒙版指定的图像部分，并将丢失的像素替换为从周围像素推断出的像素。
	
	- 模式0 最适合风景
	- 模式1 是一般的宽幅图片类型
- NGX 修补样本

	NGX SDK 中包含的 NGX InPaint 示例应用程序演示了如何在8位RGB图像中使用遮罩和深度学习推断来替换图像像素。本示例使用 `getRgbImage()` 和 `putRgbImage()` 在 OpenCV 之上实现的辅助函数，用于基于文件扩展名读取和写入最常见的图像格式。

以下命令读取 `input.png` 文件和 `mask.png` 删除并替换在 Paint 模型中使用的原始图像中的遮罩指定的像素 0。

	inpaint.exe --input input.png --mask mask.png --model 0

- 支持参数

	DLINPAINTING支持以下参数：	
	
	|参数|描述| 
	---|---|---
	Width |视频输入帧CUDA缓冲区的宽度。
	Height |视频输入帧CUDA缓冲区的高度。
	Scratch |分配的 CUDA 暂存缓冲区
	Scratch_SizeInBytes | CUDA 暂存缓冲区大小（以字节为单位）
	Input1 |线性内存的CUDA缓冲区（与 cudaMalloc（））包含输入图像的8位RGB像素
	Input1_Format |	第一个输入图像的色彩精度， NVSDK_NGX_Buffer_Format_RGBA8UI。
	Input1_SizeInBytes |	第一个输入图像帧的大小（以字节为单位）
	Input2 |线性内存的CUDA缓冲区（与 cudaMalloc（））包含输入图像的8位RGB像素
	Input2_Format |第二个输入图像的色彩精度， NVSDK_NGX_Buffer_Format_RGBA8UI。
	Input2_SizeInBytes |	第二个输入图像帧的大小（以字节为单位）
	Output | 线性内存的CUDA缓冲区（与 cudaMalloc（））返回输出图像的8位RGB像素。
	Output_Format |色彩精度输出图像 NVSDK_NGX_Buffer_Format_RGBA8UI
	Output_SizeInBytes |输出图像的大小（以字节为单位）。
	Model |修复模型（0或1）
	
- 样例代码

	以下代码片段显示了如何在NGX中使用DLINPAINTING功能。涉及以下步骤：

	- 使用创建 NGX 上下文 `NVSDK_NGX_CUDA_Init（）`。
	- 获取用于使用以下命令设置输入参数的结构 `NVSDK_NGX_CUDA_GetParameters（）`。
	- 使用以下命令获取 DLINPAINTING 算法所需的暂存缓冲区大小： `NVSDK_NGX_CUDA_GetScratchBufferSize（`），这取决于输入视频的宽度，高度和所选的比例因子。
	- 使用创建刮痕 `cudaMalloc（`）。
	- 使用以下命令创建 NGX DLINPAINTING 功能 `NVSDK_NGX_CUDA_CreateFeature（）`。
	- 使用运行 DLINPAINTING `NVSDK_NGX_CUDA_EvaluateFeature（）`。
	- 使用以下命令释放 NGX DLINPAINTING 功能 `NVSDK_NGX_CUDA_ReleaseFeature（）`。
	- 使用释放NGX资源 `NVSDK_NGX_CUDA_Shutdown（）`。

			// Initialize NGX
			CK_NGX(NVSDK_NGX_CUDA_Init(app_id, L"./", NVSDK_NGX_Version_API));
			
			// Get the parameter block.
			CK_NGX(NVSDK_NGX_CUDA_GetParameters(&params));
			
			// Verify feature is supported
			int Supported = 0;
			params->Get(NVSDK_NGX_Parameter_InPainting_Available, &Supported);
			if (!Supported)
			{
			    std::cerr << "NVSDK_NGX_Feature_InPainting Unavailable on this System" << std::endl;
			    return 1;
			 }
			
			 // Set the default hyperparameters for inferrence.
			 params->Set(NVSDK_NGX_Parameter_Width, image_width);
			 params->Set(NVSDK_NGX_Parameter_Height, image_height);
			 params->Set(NVSDK_NGX_Parameter_Model, myAppParams.model);
			
			 // Get the scratch buffer size and create the scratch allocation.
			 size_t byteSize{ 0u };
			 void *scratchBuffer{ nullptr };
			 if NVSDK_NGX_FAILED(NVSDK_NGX_CUDA_GetScratchBufferSize(NVSDK_NGX_Feature_InPainting, params, &byteSize))
			{
			    std::cerr << "Error Getting NGX Scratch Buffer Size. " << std::endl;
			    return 1;
			}
			cudaMalloc(&scratchBuffer, byteSize > 0u ? byteSize : 1u); 
			
			// Update the parameter block with the scratch space metadata.
			params->Set(NVSDK_NGX_Parameter_Scratch, scratchBuffer);
			params->Set(NVSDK_NGX_Parameter_Scratch_SizeInBytes, (uint32_t)byteSize);
			
			CK_NGX((NVSDK_NGX_CUDA_CreateFeature(NVSDK_NGX_Feature_InPainting, params, &DUHandle));
			
			// Pass the pointers to the GPU allocations to the parameter block along with the format and size.
			params->Set(NVSDK_NGX_Parameter_Input1, in_image_dev_ptr);
			params->Set(NVSDK_NGX_Parameter_Input1_Format, NVSDK_NGX_Buffer_Format_RGBA8UI);
			params->Set(NVSDK_NGX_Parameter_Input1_SizeInBytes, in_image_row_bytes * in_image_height);
			
			params->Set(NVSDK_NGX_Parameter_Input2, in_mask_dev_ptr);
			params->Set(NVSDK_NGX_Parameter_Input2_Format, NVSDK_NGX_Buffer_Format_RGBA8UI);
			params->Set(NVSDK_NGX_Parameter_Input2_SizeInBytes, in_image_row_bytes * in_image_height);
			
			params->Set(NVSDK_NGX_Parameter_Output, out_image_dev_ptr);
			params->Set(NVSDK_NGX_Parameter_Output_Format, NVSDK_NGX_Buffer_Format_RGBA8UI);
			params->Set(NVSDK_NGX_Parameter_Output_SizeInBytes, in_image_row_bytes * in_image_height);
			
			// Execute the feature.
			CK_NGX(NVSDK_NGX_CUDA_EvaluateFeature(DUHandle, params, NGXTestCallback));
			
			// Tear down the feature.
			CK_NGX(NVSDK_NGX_CUDA_ReleaseFeature(DUHandle));

## 6.故障排除
### 6.1 错误代码
如果在NGX执行期间检测到错误，则将报告以下错误代码之一：

- NVSDK_NGX_Result_FAIL_Feature不支持

	当前硬件不支持该功能。
- NVSDK_NGX_Result_FAIL_PlatformError

	例如，平台错误，请检查d3d12调试层日志以获取更多信息。
- NVSDK_NGX_Result_FAIL_Feature已经存在

	具有给定参数的功能已经存在。
- NVSDK_NGX_Result_FAIL_FeatureNotFound

	具有提供的句柄的功能不存在。
- NVSDK_NGX_Result_FAIL_InvalidParameter

	提供了无效的参数。
- NVSDK_NGX_Result_FAIL_ScratchBufferTooSmall

	提供的缓冲区太小。
- NVSDK_NGX_Result_FAIL_NotInitialized

	SDK未正确初始化。
- NVSDK_NGX_Result_FAIL_UnsupportedInputFormat

	用于输入/输出缓冲区的不受支持的格式。
- NVSDK_NGX_Result_FAIL_RW标志缺失

	功能输入/输出需要RW访问（UAV）（特定于d3d11 / d3d12）。
- NVSDK_NGX_Result_FAIL_MissingInput

	要素是使用特定输入创建的，但评估时未提供。
- NVSDK_NGX_Result_FAIL_UnableToInitializeFeature

	功能配置错误或在系统上不可用。
- NVSDK_NGX_Result_FAIL_OutOfDate

	NGX运行时库需要较新的版本，请将NVIDIA显示驱动程序更新为当前版本。
- NVSDK_NGX_Result_FAIL_OutOfGPUMemory

	功能需要比系统上可用的GPU内存更多的内存。
- NVSDK_NGX_Result_FAIL_UnsupportedFormat

	功能不支持在输入缓冲区中使用的格式。
	
## 参考
[NGX 目录](https://docs.nvidia.com/rtx/ngx/programming-guide/index.html#introduction)

[在滴滴云 GPU 服务器上使用NVIDIA NGX环境搭建](https://help.didiyun.com/hc/kb/article/1366546/)			