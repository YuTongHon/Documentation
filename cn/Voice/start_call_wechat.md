
---
title: 实现音视频通话
description: wechat call 
platform: 微信小程序
updatedAt: Thu Sep 12 2019 04:12:22 GMT+0800 (CST)
---
# 实现音视频通话
本文介绍如何使用 Agora 微信小程序 SDK 快速实现音视频通话。

## Demo 体验

Agora 提供一个微信小程序 Demo。在实现相关功能前，你可以使用 Android 或 iOS 手机上打开微信 App，扫描识别下面的二维码，快速体验 Agora 的小程序解决方案。

![](https://web-cdn.agora.io/docs-files/1568009947551)

## 开发环境要求
* 有效的 Agora 账号（免费[注册](https://dashboard.agora.io/)）
* [微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/devtools.html)
* 请确保你的微信小程序基础库支持 live-pusher 及 live-player 组件，且这两个组件在微信开发者工具中打开。
* 在微信公众平台账号的开发设置中，给予以下域名请求权限：

	```
	https://miniapp.agoraio.cn
	https://miniapp-1.agoraio.cn
	https://miniapp-2.agoraio.cn
	https://miniapp-3.agoraio.cn
	https://miniapp-4.agoraio.cn
	https://uni-webcollector.agora.io
	wss://miniapp.agoraio.cn
	```

<div class="alert note">如果你的网络环境部署了防火墙，请根据<a href="https://docs.agora.io/cn/Agora%20Platform/firewall?platform=All%20Platforms">应用企业防火墙限制</a>打开相关端口。</div>
<div class="alert info">在集成微信小程序组件之前，Agora 建议你先阅读<a href="https://developers.weixin.qq.com/miniprogram/dev/">微信小程序开发官网文档</a>。</div>



## 准备开发环境

### 开通微信小程序服务

第一次使用微信小程序支持需要先开通服务，步骤如下：

1. 登录 [Agora Dashboard](https://dashboard.agora.io/)，点击左侧导航栏的**用量** 图标 ![](https://web-cdn.agora.io/docs-files/1551250582235) 进入产品用量页面。
2. 在页面左上角展开下拉列表选择需要开通小程序支持的项目，然后点击**小程序**下的**分钟数**。

	![](https://web-cdn.agora.io/docs-files/1568008419655)
	
3. 点击**开启小程序支持**。
4. 按照屏幕提示选择你用来发送请求的服务器所在地，然后点击**应用**。

开通成功后你就可以在用量页面看到小程序的使用情况。

### 创建微信小程序组件

在微信小程序中实现音视频功能，需要使用微信的 live-player 组件和 live-pusher 组件。详见[微信小程序 API 说明](https://developers.weixin.qq.com/miniprogram/dev/api/)。

* live-player 组件
  
	 该组件用于实现微信小程序的实时音视频播放功能。开发者在创建该组件后，还需要在 js 文件中调用 API 接口对应的组件来实现该功能。声网小程序中，创建 live-player 的示例代码如下：
	 
	```javascript
	<live-player
		id="player"
		src="{{rtmp 播放地址}}"
		mode="RTC"
		bindstatechange="playerStateChange"
		object-fit="fillCrop" />
	```
		
* live-pusher 组件

   该组件用于实现微信小程序的实时音视频录制功能。开发者在创建该组件后，还需要在 js 文件中调用 API 接口对应的组件来实现该功能。声网小程序中，创建 live-pusher 的示例代码如下：
	 
	```javascript
	<live-pusher
		url="{{rtmp 推流地址}}"
		mode="RTC"
		bindstatechange="recorderStateChange" />
	```
		
### 集成微信小程序 SDK

1. 下载[声网小程序 SDK](https://docs.agora.io/cn/Agora%20Platform/downloads) 并解压。
2. 将 SDK 包中到的 **mini-app-sdk-production** 文件复制到你的小程序项目文件夹中。
3. 使用 `require` 将小程序 SDK 集成到项目中：

	```javascript
	// ../../lib/mini-app-sdk-production.js 为你的 js 文件本地路径
	const AgoraMiniappSDK = require('../../lib/mini-app-sdk-production.js');
	```

## 实现音视频通话

完成开发环境准备后，你可以参考下图中的业务流程图，在你的项目中实现通话功能。

![](https://web-cdn.agora.io/docs-files/1568008978759)

### 1. 初始化客户端对象

你需要在该步骤中填入项目的 App ID。请参考如下步骤在 Dashboard 创建项目并获取 App ID。

1. 登录 [Dashboard](https://dashboard.agora.io/)，点击左侧导航栏的**项目管理**图标 ![](https://web-cdn.agora.io/docs-files/1551254998344)。
2. 点击**创建**，按照屏幕提示设置项目名，选择一种鉴权机制，然后点击**提交**。
3. 在**项目管理**页面，你可以获取该项目的 **App ID**。

调用 `init` 方法，然后填入获取到的 App ID，初始化客户端对象。

```javascript
client.init(appId, onSuccess, onFailure);
```

### 2. 加入频道

初始化客户端后，在成功的回调中调用 `join` 方法加入频道。在该方法中填入以下参数：

* token：传入能标识用户角色和权限的 Token。可设为如下一个值：
   * NULL
   * 临时 Token。临时 Token 服务有效期为 24 小时。你可以在 Dashboard 里生成一个临时 Token，详见[获取临时 Token](https://docs.agora.io/cn/Agora%20Platform/token?platform=All%20Platforms#获取临时-token)。
   * 在你的服务器端生成的 Token。在安全要求高的场景下，我们推荐你使用此种方式生成的 Token，详见[生成 Token](../../cn/Voice/token_server.md)。

 <div class="alert note">若项目已启用 App 证书，请使用 Token。</div>

* channel：指明你想要加入的频道名。
* uid：用户 ID。整数，且需保持唯一性。

```javascript
client.join(token, channel, uid, onSuccess, onFailure);
```

### 3. 发布本地音视频流

加入频道后，调用 `publish` 方法发布本地音视频流。

```javascript
client.publish(onSuccess, onFailure);
```

### 4. 订阅远端音视频流

订阅远端流包含如下步骤：

* 调用 `on` 注册监听事件。当有人发布音视频流到频道时，会收到该事件。

	```javascript
	client.on(event, callback);
	```

* 收到事件后，在回调中调用 `subscribe` 方法订阅远端音视频流。

	```javascript
	client.subscribe(uid, onSuccess, onFailure);
	```

### 5. 更多步骤

使用小程序 SDK 过程中，你还可以根据场景需要，实现如下功能：

<details>
	<summary><font color="#3ab7f8">重新加入频道</font></summary>

如果你的项目中有切后台的场景需求，还需要调用 `rejoin` 方法做好重连机制，并在该方法中传入当前频道内所有用户的 `uid` 列表作为额外参数。其他参数的设置与 `join` 方法相同。

```javascript
client.rejoin(token, channel, uid, uids, onSuccess, onFailure);
```
	
成功重连后，SDK 会重新发布和订阅音视频流，你可以参考 [Agora Miniapp Tutorial](https://github.com/AgoraIO/Agora-Miniapp-Tutorial) 示例项目中 `reinitAgoraClient` 的实现方法更新推流及拉流地址。
</details>

### 6. 离开频道

调用 `leave` 方法离开当前频道。

```javascript
client.leave(onSuccess, onFailure);
```

### 7. 示例代码

你可以参考如下示例代码，在项目中实现想要的功能。

* 初始化客户端、加入频道、发布音视频流

	```javascript
	let client = new AgoraMiniappSDK.Client();
			this.client = client;
			// 初始化客户端对象
			client.init(APPID, () => {
				console.log(`client init success`);
				// 加入频道
				client.join(undefined, channel, uid, () => {
					console.log(`client join channel success`);
					// 注册流事件
					this.subscribeEvents(client);

					// 发布本地音频流并获取推流 url 地址
					client.publish(url => {
						console.log(`client publish success`);
					}, e => {
						console.log(`client publish failed: ${e}`);
					});
				}, e => {
					console.log(`client join channel failed: ${e}`);
				})
			}, e => {
				console.log(`client init failed: ${e}`);
			});
	```

* 注册并监听流事件

	```javascript
	// 有新的音频流加入频道
	client.on("stream-added", e => {
		let uid = e.uid;
		const ts = new Date().getTime();
		console.log(`stream ${uid} added`);
		// 订阅相应 Url 地址的音频流
		client.subscribe(uid, (url, rotation) => {
			console.log(`stream ${uid} subscribed successful`);
			// 将 Url 地址发送至 live-player
		}, e => {
			console.log(`stream subscribed failed ${e} ${e.code} ${e.reason}`);
		});
	});
	```

* 重连

	```javascript
	client.init(APPID, () => {
		// uids 为频道中已有的用户 UID 列表
		client.rejoin(undefined, channel, uid, uids, () => {
			Utils.log(`client join channel success`);
			// 获取本地推流 Url 地址
			client.publish(url => {
				Utils.log(`client publish success`);
				resolve(url);
			}, e => {
				Utils.log(`client publish failed: ${e.code} ${e.reason}`);
				reject(e)
			});
		}, e => {
			Utils.log(`client join channel failed: ${e.code} ${e.reason}`);
			reject(e)
		})
	}, e => {
		Utils.log(`client init failed: ${e} ${e.code} ${e.reason}`);
		reject(e);
	});
	```

## 运行项目

在微信开发者工具中导入你的项目后，点击**预览**，开发者工具会生成一个二维码。手机微信扫描二维码即可进入运行小程序。

## 相关文档

使用微信小程序 SDK 开发过程中，你还可以参考如下文档：

* [小程序 SDK 常见问题集](https://docs.agora.io/cn/faq/wechat)
* [错误代码和警告代码](../../cn/Voice/the_error_wechat.md)

