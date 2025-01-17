
---
title: Custom Video Source and Renderer
description: 
platform: Android
updatedAt: Wed Sep 25 2019 08:36:23 GMT+0800 (CST)
---
# Custom Video Source and Renderer
## Introduction

By default, the Agora SDK uses default audio and video modules for capturing and rendering in real-time communications. 

However, the default modules might not meet your development requirements, such as in the following scenarios:

- Your app has its own audio or video module.
- You want to use a non-camera source, such as recorded screen data.
- You need to process the captured video with a pre-processing library for functions such as image enhancement.
- You need flexible device resource allocation to avoid conflicts with other services.

This article tells you how to use the Agora Native SDK to customize the video source and renderer.

## Implementation

Before customizing the video source or renderer, ensure that you hava implemented the basic real-time communication functions in your project. For details, see [Start a Call](../../en/Interactive%20Broadcast/start_call_android.md) or [Start a Live Broadcast](../../en/Interactive%20Broadcast/start_live_android.md).

### Customize video source

The Agora SDK provides two methods to customize the video source: Push mode and MediaIO.

- With Push mode, the SDK conducts no data processing to the audio frame, such as noise reduction. Implement noise reduction on your own if you have such requirements.
- With MediaIO, Agora provides the `IVideoSource` and `IVideoSink` classes. You can create more varied use cases by using these two interfaces.

#### Push mode

Refer to the following steps to customize the video source in Push mode:

1. Call the `setExternalVideoSource` method before `joinChannel` to enable the external video source.
2. Manage video data capturing and processing on your own.
3. Send the video data back to the SDK using the `pushExternalVideoFrame` method.

**API call sequence**

Refer to the following diagram to customize the video source in your project.

![](https://web-cdn.agora.io/docs-files/1569392881143)

**Sample code**

Refer to the following sample code to customize the video source in your project.

```java
// Notify the SDK that you want to use the external video source.
rtcEngine.setExternalVideoSource(
    true，      // Whether to use the external video source.
    false,       // Whether to use Texture as the output video data format.
    true         // Whether to use Push mode.
);
  
// Push the processed video data back to the SDK.
rtcEngine.pushExternalVideoFrame(new AgoraVideoFrame(
    // Pass in the video format, width, and height of the video frame in the struct type.
    // See AgoraVideoFrame for details
));
```

We provide an open-source [Agora-Video-Source-Android](https://github.com/AgoraIO/Advanced-Video/blob/master/Capture-Raw-Video-Data/Agora-Video-Source-Android) demo project on Github. You can try the demo and view the source code in the [VideoChatViewActivity.java](https://github.com/AgoraIO/Advanced-Video/blob/master/Capture-Raw-Video-Data/Agora-Video-Source-Android/app/src/main/java/io/agora/tutorials/customizedvideosource/VideoChatViewActivity.java) file.

**API reference**

- [`isTextureEncodeSupported`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a60c16364ab588a38f5155d9c94eaf800)
- [`setExternalVideoSource`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a2d9966c52798ab62ed941fa865e926cd)
- [`pushExternalVideoFrame`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a6e7327f4449800a2c2ddc200eb2c0386)

#### MediaIO

With MediaIO, Agora provides the `IVideoSource` class and the `IVideoFrameConsumer` class to set the video format of the external video data and control the capturing process.

Refer to the following steps to customize the video source with MediaIO in your project:

1. Implement IVideoSource. Agora manages external video data capturing with the callbacks in the IVideoSource class:
	- After receiving the `getBufferType` callback, specify the format of the video data in the return value of this callback.
	- After receiving the `onInitialize` callback, save the `IVideoFrameConsumer` object in this callback. Agora receives and sends the customized video data with the `IVideoFrameConsumer` object.
	- After receiving the `onStart` callback, send the captured video data to the SDK with the `IVideoFrameConsumer` object.
	- After receiving the `onStop` callback, stop sending the video data.
	- After receiving the `onDispose` callback, release the `IVideoFrameConsume`r object.
2. Inherit the `IVideoSource` class to create a custom video source object.
3. Call the `setVideoSource` method to set the custom video source to RtcEngine.
4. Call the `startPreview` or `joinChannel` method to preview or send the customized video data according to your scenario.

**API call sequence**

Refer to the following diagram to customize the video source in your project:

![](https://web-cdn.agora.io/docs-files/1569201794546)

**Sample code**

Refer to the following sample code to customize the video source in your project:

```java
IVideoFrameConsumer mConsumer;
boolean mHasStarted;

// Create a VideoSource instance.
VideoSource source = new VideoSource() {
	@Override
	public int getBufferType() {
		// Get the current frame type. 
		// The SDK uses different methods to process different frame types.
		// If you want to switch to another VideoSource type, create another instance.
		// There are three video frame types: BYTE_BUFFER(1); BYTE_ARRAY(2); TEXTURE(3)
		return BufferType.BYTE_ARRAY;
	}

	@Override
 	public boolean onInitialize(IVideoFrameConsumer consumer) {
		// Consumer was created by the SDK.
		// Save it in the lifecycle of the VideoSource.
		mConsumer = consumer;
	}

	@Override
 	public boolean onStart() {
		mHasStarted = true;
	}

	@Override
  	public void onStop() {
		mHasStarted = false;
	}

	@Override
 	public void onDispose() {
		// Release the consumer.
		mConsumer = null;
	}
};

// Change the inputting video stream to the VideoSource instance.
rtcEngine.setVideoSource(source);

// After receiving the video frame data, use the consumer class to send the data.
// Choose differnet methods according to the frame type.
// For example, the current frame type is byte array, i.e. NV21.
if (mHasStarted && mConsumer != null) {
	mConsumer.consumeByteArrayFrame(data, AgoraVideoFrame.NV21, width, height, rotation, timestamp);
}
```

We provide an open-source [Custom-Media-Device-Android](https://github.com/AgoraIO/Advanced-Video/tree/master/Custom-Media-Device/Agora-Custom-Media-Device-Android) demo project on Github. You can try the demo and view the source code in the [ViewSharingCapturer.java](https://github.com/AgoraIO/Advanced-Video/blob/master/Custom-Media-Device/Agora-Custom-Media-Device-Android/app/src/main/java/io/agora/rtc/mediaio/app/shareScreen/source/ViewSharingCapturer.java) file.

**API Reference**

* [`setVideoSource`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#aa240e991d12b5240fc5fd362cbc0d521)
* [`IVideoSource`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/interfaceio_1_1agora_1_1rtc_1_1mediaio_1_1_i_video_source.html)
* [`IVideoFrameConsumer`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/interfaceio_1_1agora_1_1rtc_1_1mediaio_1_1_i_video_frame_consumer.html)

**Reference**

The video source and renderer can be customized by switching on/off the video frame input based on the media engine callback. This method can be used if an app has its own video module and only needs the Agora SDK for real-time communications. See [Customize the Video Source with the Agora Component](../../en/Interactive%20Broadcast/custom_advanced_android.md).

### Custom video renderer

Video data captured with MediaIO can be rendered with the customized video renderer using the `IVideoSink` class.

Refer to the following steps to customize the video renderer with MediaIO in your project:

1. Implement IVideoSink. Agora manages external video data rendering with the callbacks in the `IVideoSink` class:
	- After receiving the `getBufferType` and `getPixelFormat` callbacks, specify the format of video data in the return value of the respective callback.
	- Manage the rendering process of the video data with the `onInitialize`, `onStart`, `onStop`, `onDispose`, and `getEglContextHandle` callbacks. 
	- Implement an `IVideoFrameConsumer` object with the specified video format to get the video data.
2. Inherit the `IVideoSink` class to create a customized sink object.
3. Call the `setLocalVideoRenderer` or `setRemoteVideoRenderer` method for local or remote rendering according to your scenario.
4. Call the `startPreview` or `joinChannel` method to preview or send the rendered video data.

**API call sequence**

Refer to the following diagram to customize the video sink in your project.

![](https://web-cdn.agora.io/docs-files/1569397613198)

**Sample code**

Refer to the following diagram to customize the video sink in your project.

```java
IVideoSink sink = new IVideoSink() {
	@Override
	public boolean onInitialize () {
		return true;
	}

	@Override
	public boolean onStart() {
		return true;
	}
 
	@Override
	public void onStop() {
	}
 
	@Override
	public void onDispose() {
	}
 
	@Override
	public long getEGLContextHandle() {
	}
 
	@Override
	public int getBufferType() {
		return BufferType.BYTE_ARRAY;
	}
 
	@Override
	public int getPixelFormat() {
		return PixelFormat.NV21;
	}
	
    // Call this method to send the video data to the sink
    // Choose the respective callback according to the video data format that you want.
    @Override
    public void consumeByteArrayFrame(byte[] data, int format, int width, int height, int rotation, long timestamp) {
     
    // The video sink renders the video.
    }
 
 
    public void consumeByteBufferFrame(ByteBuffer buffer, int format, int width, int height, int rotation, long timestamp) {
     
    // The video sink renders the video.
    }
 
 
    public void consumeTextureFrame(int textureId, int format, int width, int height, int rotation, long timestamp, float[] matrix) {
     
    // The video sink renders the video.
    }
}

rtcEngine.setLocalVideoRenderer(sink);
```

We provide an open-source [Custom-Media-Device-Android](https://github.com/AgoraIO/Advanced-Video/tree/master/Custom-Media-Device/Agora-Custom-Media-Device-Android) demo project on Github. You can try the demo and view the source code in the [PrivateTextureHelper.java](https://github.com/AgoraIO/Advanced-Video/blob/master/Custom-Media-Device/Agora-Custom-Media-Device-Android/app/src/main/java/io/agora/rtc/mediaio/app/videoSource/source/PrivateTextureHelper.java) file.

**API Reference**
* [`setLocalVideoRenderer`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#ab10fd6d8dd89a5bca09b115ecd9e3416)
* [`setRemoteVideoRenderer`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a0da32c040cb9d987df2950b83459ba56)
* [`IVideoSink`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/interfaceio_1_1agora_1_1rtc_1_1mediaio_1_1_i_video_sink.html)
* [`IVideoFrameConsumer`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/java/interfaceio_1_1agora_1_1rtc_1_1mediaio_1_1_i_video_frame_consumer.html)

**Reference**
Agora provides a helper class and sample code for developers to customize the video renderer. See [Customize the Video Renderer with Agora Components](../../en/Interactive%20Broadcast/custom_advanced_android.md).


## Consideration

Customizing the video source and sink requires you to manage video data recording and playback on your own.

- When customizing the video source, you need to capture and process the video data on your own.
- When customizing the video sink, you need to process and render the video data on your own.

## Reference

Refer to [Custom Audio Source and Sink](../../en/Interactive%20Broadcast/custom_audio_android.md) if you want to customize the audio source and sink in your project.

