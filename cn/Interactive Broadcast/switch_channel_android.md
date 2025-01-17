
---
title: 快速切换直播频道
description: 
platform: Android
updatedAt: Mon Sep 23 2019 09:03:43 GMT+0800 (CST)
---
# 快速切换直播频道
## 功能描述

通常来说，如果直播观众希望切换到另外一个直播频道，需要依次调用如下方法：

- `leaveChannel`，用以离开当前频道
- `joinChannel`，用以加入新的频道

Agora Native SDK 从 v2.9.0 开始，提供新接口 `switchChannel`，帮助直播频道的观众更快实现直播频道切换。

## 实现方法

实现快速切换频道前，请确保你已在项目中实现了互动直播功能，详见开始[互动直播](../../cn/Interactive%20Broadcast/start_live_android.md)。

互动直播过程中，观众加入频道后只需调用 `switchChannel` 方法，就可以快速切换到另外一个直播频道。你需要在该方法中传入想要加入的新频道的 Token 和频道名。

成功调用 `switchChannel` 方法切换到其他频道后，本地会先收到离开原频道的回调 `onLeaveChannel`，再收到成功加入新频道的回调 `onJoinChannelSuccess`。

### API 调用时序

下图展示切换频道的 API 调用时序：

![](https://web-cdn.agora.io/docs-files/1569229417437)

### 示例代码

你可以参考下面的示例代码片段，调用 `switchChannel` 在项目中实现快速切换频道：

```java
mRtcEngine.switchChannel(YOUR_TOKEN, mRoomName);
```

同时，我们在 Github 提供一个开源的 [Quick-Switch-Android](https://github.com/AgoraIO/Advanced-Video/tree/master/Quick-Switch-Channel/Quick-Switch-Android) 示例项目，你也可以前往下载体验并参考 [WorkThread.java](https://github.com/AgoraIO/Advanced-Video/blob/master/Quick-Switch-Channel/Quick-Switch-Android/app/src/main/java/io/agora/switchlive/rtc/WorkerThread.java) 文件中 `switchChannel` 函数的源代码。

### API 参考

- [switchChannel](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a72f13225defc1b14dfb29820a0495da2)

## 开发注意事项

`switchChannel` 方法仅适用于直播频道中的观众用户。
