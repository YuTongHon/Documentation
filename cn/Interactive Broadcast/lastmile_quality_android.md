
---
title: 通话前检测网络质量
description: 通话前的网络质量检测。
platform: Android
updatedAt: Thu Sep 26 2019 02:07:52 GMT+0800 (CST)
---
# 通话前检测网络质量
## 功能描述

在加入频道或切换角色为主播前，进行网络质量探测，可以判断或预测用户当前的网络状况是否良好，可以满足音频码率或者当前选定的视频属性的目标码率。

在对网络质量要求高的场景下，Agora 建议在加入频道前进行探测，保证通信顺畅。

## 实现方法

开始检测网络质量前，请确保你已在项目中实现了基本的音视频通信或直播功能。详见[开始音视频通话](../../cn/Interactive%20Broadcast/start_call_android.md)或[开始互动直播](../../cn/Interactive%20Broadcast/start_live_android.md)。

在用户加入频道或上麦前，调用 `startLastmileProbeTest` 进行网络质量探测，向用户反馈上下行网络的带宽、丢包、网络抖动和往返时延。

启用该方法后，SDK 会依次返回如下 2 个回调：
- `onLastmileQuality`：约 2 秒内返回。该回调通过打分反馈上下行网络质量，更贴近主观感受
- `onLastmileProbeResult`：约 30 秒内返回。该回调通过客观数据反馈上下行网络质量，更客观

### API 调用时序

参考如下时序在你的项目中进行通话前网络质量探测。

![](https://web-cdn.agora.io/docs-files/1569463642819)

### 示例代码

参考下文示例代码在你的项目中进行通话前网络质量探测。

```java
// 配置一个 LastmileProbeConfig 实例。参数可参考 API 文档
LastmileProbeConfig config = new LastmileProbeConfig(){};
// 确认探测上行网络质量
config.probeUplink =  true;
// 确认探测下行网络质量
config.probeDownlink = true;
// 期望的最高发送码率，单位为 Kbps，范围为 [100, 5000]
config.expectedUplinkBitrate = 1000;
// 期望的最高接收码率，单位为 Kbps，范围为 [100, 5000]
config.expectedDownlinkBitrate = 1000;
// 加入频道前开始 Last-mile 网络探测
rtcEngine.startLastmileProbeConfig(config);

// 位于全局 IRtcEngineEventHandler 中
// 开始 Last-mile 网络探测后，约 2 秒后会发生该回调
public void onLastmileQuality(int quality)

// 位于全局 IRtcEngineEventHandler 中
// 开始 Last-mile 网络探测后，约 30 秒后会发生该回调
public void onLastmileProbeResult(LastmileProbeResult) {
	// (1)可以选择在回调内部结束测试。在测试结束前，Agora 建议不要调用其他 API 方法
	rtcEngine.stopLastmileProbeTest();
}

// (2)也可以选择其他时候结束测试。在测试结束前，Agora 建议不要调用其他 API 方法
rtcEngine.stopLastmileProbeTest();
```

同时，我们在 Github 提供一个开源的 [OpenVideoCall-Android](https://github.com/AgoraIO/Basic-Video-Call/tree/master/Group-Video/OpenVideoCall-Android) 示例项目。你可以下载体验，或者查看 [NetworkTestActivity.java](https://github.com/AgoraIO/Basic-Video-Call/blob/master/Group-Video/OpenVideoCall-Android/app/src/main/java/io/agora/openvcall/ui/NetworkTestActivity.java) 文件中的源代码。

### API 参考

- [`startLastmileProbeTest`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a81c6541685b1c4437d9779a095a0f871)
- [`stopLastmileProbeTest`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#ae21243b8da8bda9ee5f3a00621cbf959)
- [`onLastmileQuality`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_i_rtc_engine_event_handler.html#a2887941e3c105c21309bd2643372e7f5)
- [`onLastmileProbeResult`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_i_rtc_engine_event_handler.html#ad74a9120325bfeccdec4af4611110281)

## 开发注意事项

- Last-mile 测试必须在加入通话频道之前。在结束测试之前，Agora 不建议调用其他 API 方法。
- `onLastmileQuality` 回调第一次报告的结果有一定概率是 unknown, 可通过之后的几次回调获得结果。
- 纯语音产品使用 48 Kbps 的固定探测码率；视频产品会根据当前选定的视频属性调整探测码率。
