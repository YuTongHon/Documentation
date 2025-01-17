
---
title: 自定义视频采集和渲染
description: 
platform: iOS,macOS
updatedAt: Wed Sep 25 2019 09:24:01 GMT+0800 (CST)
---
# 自定义视频采集和渲染
## 功能介绍

实时音视频传输过程中，Agora SDK 通常会启动默认的音视频模块进行采集和渲染。在以下场景中，你可能会发现默认的音视频模块无法满足开发需求：

- app 中已有自己的音频或视频模块
- 希望使用非 Camera 采集的视频源，如录屏数据
- 需要使用自定义的美颜库有或前处理库
- 某些视频采集设备被系统独占。为避免与其它业务产生冲突，需要灵活的设备管理策略

基于此，Agora Native SDK 支持使用自定义的音视频源或渲染器，实现相关场景。本文介绍如何实现自定义视频采集和渲染。

## 实现方法

在开始自定义视频源和渲染器前，请确保你已在项目中实现了基本的音视频通话或直播功能，详见如下文档：
- iOS：[开始音视频通话](../../cn/Video/start_call_ios.md)或[开始互动直播](../../cn/Video/start_live_ios.md)
- macOS：[开始音视频通话](../../cn/Video/start_call_mac.md)或[开始互动直播](../../cn/Video/start_live_mac.md)

### 自定义视频采集

Agora Native SDK 目前提供 Push 方式和 MediaIO 两种方式实现自定义的视频源。其中：

- Push 方式下，SDK 默认不会对采集传入的音频数据做消噪等处理。如有音频消噪需求，需要开发者自行实现。
- MediaIO 方式下，自定义视频源可以搭配自定义渲染器使用，实现更为丰富的场景。

#### Push 方式

参考如下步骤，在你的项目中使用 Push 方式实现自定义视频源功能：

1. 在 `joinChannelByToken` 前通过调用 `setExternalVideoSource` 指定外部视频采集设备。
2. 指定外部采集设备后，开发者自行管理视频数据采集和处理。
3. 完成视频数据处理后，再通过 `pushExternalVideoFrame` 发送给 SDK 进行后续操作。

**API 时序图**

参考下图时序在你的项目中自定义视频采集。

![](https://web-cdn.agora.io/docs-files/1569393983389)

**示例代码**

参考下文代码在你的项目中自定义视频采集。

```swift
// Swift
// swift
// 推入数据类型为 CVPixelBufferRef
let videoFrame = AgoraVideoFrame()
videoFrame.format = 12
// [NSDate date].timeIntervalSince1970 为当前时间戳；1000 表示每秒钟 1000 帧
videoFrame.time = CMTimeMakeWithSeconds([NSDate date].timeIntervalSince1970, 1000)
videoFrame.textureBuf = "Your CVPixelBufferRef"
videoFrame.ratation = 0

// 推入数据类型为 rawData
let videoFrame = AgoraVideoFrame()
videoFrame.format = "your data fromat"
// [NSDate date].timeIntervalSince1970 为当前时间戳；1000 表示每秒钟 1000 帧
videoFrame.time = CMTimeMakeWithSeconds([NSDate date].timeIntervalSince1970, 1000)
videoFrame.data = "your rawData"
videoFrame.strideInPixels = "your stride"
videoFrame.height = "your height"
videoFrame.dataBuf = "your rawData"
videoFrame.ratation = 0

agoraKit.pushExternalVideoFrame(videoFrame)
```

```objective-c
// Objective-C
// 推入数据类型为 CVPixelBufferRef
AgoraVideoFrame *videoFrame = [[AgoraVideoFrame alloc] init];
videoFrame.format = 12;
// [NSDate date].timeIntervalSince1970 为当前时间戳；1000 表示每秒钟 1000 帧
videoFrame.time = CMTimeMakeWithSeconds([NSDate date].timeIntervalSince1970, 1000);
videoFrame.textureBuf = "Your CVPixelBufferRef";
videoFrame.ratation = 0;

// 推入数据类型为 rawData
AgoraVideoFrame *videoFrame = [[AgoraVideoFrame alloc] init];
videoFrame.format = "your data fromat";
// [NSDate date].timeIntervalSince1970 为当前时间戳；1000 表示每秒钟 1000 帧
videoFrame.time = CMTimeMakeWithSeconds([NSDate date].timeIntervalSince1970, 1000);
videoFrame.data = "your rawData";
videoFrame.strideInPixels = "your stride";
videoFrame.height = "your height";
videoFrame.dataBuf = "your rawData";
videoFrame.ratation = 0;

[agoraKit pushExternalVideoFrame: videoFrame];
```

同时，我们在 Github 提供一个开源的 [Agora-Video-Source-iOS](https://github.com/AgoraIO/Advanced-Video/tree/master/Capture-Raw-Video-Data/Agora-Video-Source-iOS) 示例项目。你可以前往下载，或参考 [VideoChatViewController.swift](https://github.com/AgoraIO/Advanced-Video/blob/master/Capture-Raw-Video-Data/Agora-Video-Source-iOS/Agora%20Video%20Source/VideoChat/VideoChatViewController.swift) 文件中的源代码。

**API 参考**

- [`setExternalVideoSource`](https://docs.agora.io/cn/Video/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/setExternalVideoSource:useTexture:pushMode:)
- [`pushExternalVideoFrame`](https://docs.agora.io/cn/Video/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/pushExternalVideoFrame:)


#### MediaIO 方式

Agora 通过 MediaIO 提供 `AgoraVideoSourceProtocol` 协议和 `AgoraVideoFrameConsumer` 类，你可以通过该类设置采集的视频数据格式，并控制外部视频的采集过程。

参考如下步骤，在你的项目中使用 MediaIO 方式实现自定义视频源功能：

1. 实现 `AgoraVideoSourceProtocol` 协议。Agora 通过 `AgoraVideoSourceProtocol` 协议下的各回调设置视频数据格式，并控制采集过程：
	- 收到 `bufferType` 回调后，在该回调的返回值中指定想要采集的视频数据格式；
	- 收到 `shouldInitialize` 回调后，保存该回调中的 `AgoraVideoFrameConsumer` 对象。Agora 通过 `AgoraVideoFrameConsumer` 对象发送和接收自定义的视频数据；
	- 收到 `shouldStart` 回调后，通过 `AgoraVideoFrameConsumer` 对象向 SDK 发送视频帧；
	- 收到 `shouldStop` 回调后，停止使用 `AgoraVideoFrameConsumer` 对象向 SDK 发送视频帧；
	- 收到 `shouldDispose` 回调后，释放 `AgoraVideoFrameConsumer` 对象。
2. 继承实现的 `AgoraVideoSourceProtocol` 协议，构建一个自定义的视频源对象。
3. 调用 `setVideoSource` 方法，将自定义的视频源对象设置给 AgoraRtcEngineKit。
4. 根据场景需要，调用 `startPreview`、`joinChannelByToken` 等方法预览或发送自定义采集的视频数据。

**API 调用时序**

参考下图时序使用 MediaIO 在你的项目中实现自定义视频采集。

![](https://web-cdn.agora.io/docs-files/1569398846255)

**示例代码**

1. 遵守 AgoraVideoSourceProtocol 协议， 并实现接口，构建自定义的 Video Source 类。

	```swift
	// Swift
	// 协议中的变量
		 var consumer: AgoraVideoFrameConsumer?
	// 调用 Consumer 的方法，将视频数据推入 Agora SDK

		 // 推入r awData 类型
		 consumer.consumeRawData("your rawData", withTimestamp: CMTimeMake(1, 15), format: "your data format", size: size, rotation: rotation)

		 // 推入 CVPixelBuffer
		 consumer.consumePixelBuffer("your pixelBuffer", withTimestamp: CMTimeMake(1, 15), rotation: rotation)

	// 协议中的方法
	1. 视频采集使用的 Buffer 类型
		func bufferType() -> AgoraVideoBufferType {
				return bufferType
		}

	2. 在初始化视频源（shouldInitialize）中, 初始化自定义的 Video Source
		func shouldInitialize() -> Bool {
		}

	3. 自定义视频源开始采集视频数据，并通过 Consumer 推入视频数据
		func shouldStart() {
		}

	4. 自定义视频源停止采集视频数据
		func shouldStop() { 
		}

	5. 在释放自定义视频源
		func shouldDispose() {
		}
	```

	```objective-c
	// Objective-C
	// 协议中的变量
	@synthesize consumer;
	// 调用 Consumer 的方法，将视频数据推入 Agora SDK

		// 推入 rawData 类型
		[consumer consumeRawData: "your rawData" withTimestamp: CMTimeMake(1, 15) format: "your data format" size: size rotation: rotation];

		// 推入 CVPixelBuffer
		[consumer consumePixelBuffer: "your pixelBuffer" withTimestamp: CMTimeMake(1, 15) rotation: rotation];

	// 协议中的方法
	1. 视频采集使用的 Buffer 类型
		- (AgoraVideoBufferType)bufferType {
				return AgoraVideoBufferTypePixelBuffer;
		}

	2. 在初始化视频源（shouldInitialize）中, 初始化自定义的 Video Source
		- (BOOL)shouldInitialize {
				return YES;
		}

	3. 自定义视频源开始采集视频数据，并通过 Consumer 推入视频数据
		- (void)shouldStart {
		}

	4. 自定义视频源停止采集视频数据
		- (void)shouldStop {
		}

	5. 在释放自定义视频源
		- (void)shouldDispose {
		}
	```


2. 将遵守了 AgoraVideoSourceProtocol 协议的自定义 VideoSource 对象设给 AgoraRtcEngineKit。

	```swift
	// Swift
	agoraKit.setVideoSource(videoSource)
	```

	```objective-c
	// Objective-C
	[agoraKit setVideoSource: videoSource];
	```
	
同时，我们在 Github 提供一个开源的 [Agora-Custom-Media-Device-iOS](https://github.com/AgoraIO/Advanced-Video/tree/master/Custom-Media-Device/Agora-Custom-Media-Device-iOS) 示例项目。你可以下载体验，或查看 [AgoraCamera.swift](https://github.com/AgoraIO/Advanced-Video/blob/master/Custom-Media-Device/Agora-Custom-Media-Device-iOS/Agora-Custom-Media-Device/CustomMediaDevices/AgoraCamera.swift) 文件中的源代码。
	
**API 参考**

* [`setVideoSource`](https://docs.agora.io/cn/Video/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/setVideoSource:)
* [`AgoraVideoSourceProtocal`](https://docs.agora.io/cn/Video/API%20Reference/oc/Protocols/AgoraVideoSourceProtocol.html)
* [`AgoraVideoFrameConsumer`](https://docs.agora.io/cn/Video/API%20Reference/oc/Protocols/AgoraVideoFrameConsumer.html)

### 自定义渲染器

通过 MediaIO 采集到的视频数据，还可以搭配 Agora 的 `AgoraVideoSinkProtocol` 协议使用，实现自定义渲染功能。

参考如下步骤，在你的项目中使用 MediaIO 方式实现自定义渲染器功能：

1. 实现 `AgoraVideoSinkProtocol` 协议。Agora 通过 `AgoraVideoSinkProtocol` 协议下的各回调设置视频数据格式，并控制渲染过程：
	- 收到 `bufferType` 和 `pixelFormat` 回调后，在对应回调的返回值中设置你想要渲染的数据类型；
	- 根据收到的 `shouldInitialize`、`shouldStart`、`shouldStop`、`shouldDispose` 回调，控制视频数据的渲染过程；
	- 实现一个对应渲染数据类型的 `AgoraVideoFrameConsumer` 对象，以获取视频数据。
2. 继承实现的 `AgoraVideoSinkProtocol` 协议，构建一个自定义的渲染器。
3. 调用 `setLocalVideoRenderer` 或 `setRemoteVideoRenderer`，用于本地渲染或远端渲染。
4. 根据场景需要，调用 `startPreview`、`joinChannelByToken` 等方法预览或发送自定义渲染的视频数据。

**API 调用时序**

参考下图时序使用 MediaIO 在你的项目中实现自定义视频渲染。

![](https://web-cdn.agora.io/docs-files/1569399758500)

**示例代码**

参考下文代码使用 MediaIO 在你的项目中实现自定义视频渲染。

1. 遵守 AgoraVideoSinkProtocol 协议， 并实现接口，构建自定义的 Video Renderer 类。

	```swift
	// Swift
	// 协议中的方法
	1. 希望 Agora SDK 抛出的视频 Buffer 类型
		func bufferType() -> AgoraVideoBufferType {
				return bufferType
		}

		希望 Agora SDK 抛出的视频数据格式
		func pixelFormat() -> AgoraVideoPixelFormat {
				return pixelFormat
		}

	2. 初始化自定义的 Video Renderer
		func shouldInitialize() -> Bool {
				return true
		}

	3. 启动自定义的 Video Renderer   
		func shouldStart() {

		}

	4. Agora SDK 停止抛出视频数据
		func shouldStop() {

		}

	5. 自定义的 Video Renderer 可以被释放   
		func shouldDispose() {

		}

	6. Agora SDK 通过该接口抛出 CVPixelBuffer 类型的视频数据, 自定义 Video Renderer 可以获取数据进行渲染
		func renderPixelBuffer(_ pixelBuffer: CVPixelBuffer, rotation: AgoraVideoRotation) {
		}

		Agora SDK 通过该接口抛出 rawData 类型的视频数据, 自定义 Video Renderer 可以获取数据进行渲染
		func renderRawData(_ rawData: UnsafeMutableRawPointer, size: CGSize, rotation: AgoraVideoRotation) {
		}
	}
	```

	```objective-c
	// Objective-C
	// 协议中的方法
	1. 希望 Agora SDK 抛出的视频 Buffer 类型
		- (AgoraVideoBufferType)bufferType {
				return AgoraVideoBufferTypePixelBuffer;
		}

		希望 Agora SDK 抛出的视频数据格式
		- (AgoraVideoPixelFormat)pixelFormat {
			 return AgoraVideoPixelFormatI420;
		}

	2. 初始化自定义的 Video Renderer
		- (BOOL)shouldInitialize {
			return YES;
		}

	3. 启动自定义的 Video Renderer 
		- (void)shouldStart {
		}

	4. Agora SDK 停止抛出视频数据
		- (void)shouldStop {
		}

	5. 自定义的 Video Renderer 可以被释放   
		- (void)shouldDispose {
		}

	6. Agora SDK 通过该接口抛出 CVPixelBuffer 类型的视频数据, 自定义 Video Renderer 可以获取数据进行渲染
		- (void)renderPixelBuffer:(CVPixelBufferRef _Nonnull)pixelBuffer rotation:(AgoraVideoRotation)rotation {
		}

		Agora SDK 通过该接口抛出 rawData 类型的视频数据, 自定义 Video Renderer 可以获取数据进行渲染
		- (void)renderRawData:(void * _Nonnull)rawData size:(CGSize)size rotation:(AgoraVideoRotation)rotation {
		}
	```

2. 将遵守了  AgoraVideoSourceProtocol 协议的自定义 VideoRenderer 对象设给 AgoraRtcEngineKit。

	```swift
	// Swift
	agoraKit.setLocalVideoRenderer(videoRenderer)
	agoraKit.setRemoteVideoRenderer(videoRenderer, forUserId: uid)
	```
	
	```objective-c
	// Objective-C
	[agoraKit setLocalVideoRenderer: videoRenderer];
	[agoraKit setRemoteVideoRenderer: videoRenderer, uid];
	```

同时，我们在 Github 提供一个开源的 [Agora-Custom-Media-Device-iOS](https://github.com/AgoraIO/Advanced-Video/tree/master/Custom-Media-Device/Agora-Custom-Media-Device-iOS) 示例项目。你可以下载体验，或查看 [AgoraRenderView.swift](https://github.com/AgoraIO/Advanced-Video/blob/master/Custom-Media-Device/Agora-Custom-Media-Device-iOS/Agora-Custom-Media-Device/CustomMediaDevices/AgoraRenderView.swift) 文件中的源代码。

	
**API 参考**

* [`setLocalVideoRenderer`](https://docs.agora.io/cn/Video/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/setLocalVideoRenderer:)
* [`setRemoteVideoRenderer`](https://docs.agora.io/cn/Video/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/setRemoteVideoRenderer:forUserId:)
* [`AgoraVideoSinkProtocal`](https://docs.agora.io/cn/Video/API%20Reference/oc/Protocols/AgoraVideoSinkProtocol.html)
* [`AgoraVideoFrameConsumer`](https://docs.agora.io/cn/Video/API%20Reference/oc/Protocols/AgoraVideoFrameConsumer.html)

## 开发注意事项

自定义视频采集和渲染场景中，需要开发者具有采集或渲染视频的能力：

- 自定义视频采集场景中，你需要自行管理视频数据的采集和处理。
- 自定义视频渲染场景中，你需要自定管理视频数据的处理和显示。

## 相关文档

如果你还想在项目中实现自定义的音频采集和渲染功能，请参考文档[自定义音频采集和渲染](../../cn/Video/custom_audio_apple.md)。
