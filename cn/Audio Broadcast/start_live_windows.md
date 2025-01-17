
---
title: 实现互动直播
description: 
platform: Windows
updatedAt: Wed Sep 18 2019 07:46:28 GMT+0800 (CST)
---
# 实现互动直播
本文介绍如何通过 Agora SDK 快速实现互动直播。

互动直播和实时通话的区别就在于，直播频道的用户有角色之分。你可以将角色设置为主播或者观众，其中主播可以收、发流，观众只能收流。

## Demo 体验
Agora 在 Github 上提供一个开源的实时音视频通话示例项目 [OpenLive-Windows](https://github.com/AgoraIO/Basic-Video-Broadcasting/tree/master/OpenLive-Windows)。在实现相关功能前，你可以下载并查看源代码。

## 前提条件
- Microsoft Visual Studio 2017 或以上版本
- 支持 Windows 7 或以上版本的 Windows 设备
- 有效的 Agora 账户（免费[注册](https://dashboard.agora.io)）

<div class="alert note">如果你的网络环境部署了防火墙，请根据<a href="https://docs.agora.io/cn/Agora%20Platform/firewall?platform=All%20Platforms">应用企业防火墙限制</a>打开相关端口。</div>

## 设置开发环境
本节介绍如何创建项目，并将 Agora SDK 集成至你的项目中。
### 创建项目
参考以下步骤创建一个 Windows 项目。若已有 Windows 项目，直接查看[集成 SDK](#inte)。

<details>
	<summary><font color="#3ab7f8">创建 Windows 项目</font></summary>

1. 打开 <b>Microsoft Visual Studio</b> 并点击新建项目。
2. 进入<b>新建项目</b>窗口，选择项目类型为 <b>MFC 应用程序</b>，输入项目名称，选择项目存储路径，并点击<b>确认</b>。
3. 进入<b>MFC 应用程序</b>窗口，选择应用程序类型为<b>基于对话框</b>，并点击完成。
	
</details>

<a name="inte"></a>
### 集成 SDK
参考以下步骤将 Agora SDK 集成到你的项目中。

**1. 配置项目文件**

- 根据应用场景，从 [SDK 下载](https://docs.agora.io/cn/Agora%20Platform/downloads)获取最新 SDK，解压并打开。

- 打开已下载的 SDK 文件，并将其中的 **sdk** 文件夹复制到你的项目文件夹下。

**2. 配置项目属性**

在**解决方案资源管理器**窗口中，右击项目名称并点击属性进行以下配置，配置完成后点击**确定**。

- 进入 **C/C++ > 常规 > 附加包含目录**菜单，点击**编辑**，并在弹出窗口中输入 **$(SolutionDir)include**。

- 进入**链接器 > 常规 > 附加库目录**菜单，点击**编辑**，并在弹出窗口中输入 **$(SolutionDir)**。

- 进入**链接器 > 输入 > 附加依赖项**菜单，点击**编辑**，并在弹出窗口中输入 **agora_rtc_sdk.lib**。

## 实现音视频直播
本节介绍如何实现音视频直播。视频直播的 API 使用时序见下图：

![](https://web-cdn.agora.io/docs-files/1568257965768)

### 1. 创建用户界面

为直观地体验音视频通话，需根据应用场景创建用户界面（UI)。若项目中已有用户界面，直接查看[初始化 IRtcEngine](#ini)。

在视频通话场景中，推荐在 UI 上添加以下控件：
- 主播视频窗口
- 离开频道按钮

当你使用示例项目中的 UI 设计时，你将会看到如下界面：

![](https://web-cdn.agora.io/docs-files/1568792771048)

<a name="ini"></a>
### 2. 初始化 IRtcEngine

在调用其他 Agora API 前，需要创建并初始化 IRtcEngine 对象。

该步骤中，你需要使用项目的 App ID。参考如下步骤，在 Dashboard 中创建一个项目并获取该项目的 App ID。
- 进入 [Dashboard](https://dashboard.agora.io) 并点击左侧导航栏的项目管理 ![](https://web-cdn.agora.io/docs-files/1551254998344)。
- 进入**项目管理**界面后，点击**创建**，根据屏幕提示输入项目名称，选择一种鉴权机制，并点击**提交**。
- 点击项目后的**编辑**按钮，即可查看该项目的 **App ID**。

调用 `createAgoraRtcEngine` 和 `initialize` 方法，传入获取到的 App ID，即可初始化 `IRtcEngine`。

你也可以根据需求，在初始化时实现其他功能。如注册用户加入频道和离开频道的回调。

```C++
// 创建实例。
m_lpRtcEngine = createAgoraRtcEngine();
RtcEngineContext ctx;
          
// 添加注册回调和事件。
ctx.eventHandler = &m_engineEventHandler;
  
// 输入你的 App ID。
ctx.appId = "YourAppID";
  
// 初始化 IRtcEngine。
m_lpRtcEngine->initialize(ctx);
```

```C++
// 继承 IRtcEngineEventHandler 类中的回调与事件。
class CAGEngineEventHandler :
    public IRtcEngineEventHandler
{
public:
    CAGEngineEventHandler();
    ~CAGEngineEventHandler();
    void setMainWnd(HWND wnd);
    HWND GetMsgReceiver() {return m_hMainWnd;};
  
    // 在主播调用 joinChannel 方法后，此回调会报告加入频道的主播信息。
    virtual void onJoinChannelSuccess(const char* channel, uid_t uid, int elapsed);
  
    // 在主播调用 leaveChannel 方法后，此回调会报告离开频道的主播信息。
    virtual void onLeaveChannel(const RtcStats& stat);
  
  
    // 在引擎收到第一帧远端视频流并解码成功时，会触发此回调。
    virtual void onFirstRemoteVideoDecoded(uid_t uid, int width, int height, int elapsed);
  
  
    // 在主播调用 leaveChannel 方法后，此回调会报告该主播离开频道及离线原因。
    virtual void onUserOffline(uid_t uid, USER_OFFLINE_REASON_TYPE reason);
private:
    HWND m_hMainWnd;
};
```

### 3. 设置频道模式
初始化结束后，调用 `setChannelProfile` 方法，将频道模式设为直播。

一个 IRtcEngine 只能使用一种频道模式。如果想切换为其他模式，需要先调用 `release` 方法释放当前的 IRtcEngine 实例，然后调用 `createAgoraRtcEngine` 和 `initialize` 方法创建一个新实例，再调用 `setChannelProfile` 设置新的频道模式。

```C++
// 设置频道模式为直播。
m_lpRtcEngine->setChannelProfile(CHANNEL_PROFILE_LIVE_BROADCASTING);
```

### 4. 设置用户角色
直播频道有两种用户角色：主播和观众，其中默认的角色为观众。设置频道模式为直播后，你可以在 App 中参考如下步骤设置用户角色：
- 让用户选择自己的角色是主播还是观众
- 调用 `setClientRole` 方法，然后使用用户选择的角色进行传参

注意，直播频道内的用户，只能看到主播的画面、听到主播的声音。加入频道后，如果你想切换用户角色，也可以调用 `setClientRole` 方法。

```C++
BOOL CAgoraObject::SetClientRole(CLIENT_ROLE_TYPE role, LPCSTR lpPermissionKey)
{
    // 设置用户角色。
    int nRet = m_lpAgoraEngine->setClientRole(role);
    m_nRoleType = role;
    return nRet == 0 ? TRUE : FALSE;
}
```

```C++
// 创建选择用户角色的对话框。
void CEnterChannelDlg::OnCbnSelchangeCmbRole()
{
    int nSel = m_ctrRole.GetCurSel();
    if (nSel == 0)
        CAgoraObject::GetAgoraObject()->SetClientRole(CLIENT_ROLE_BROADCASTER);
    else
        CAgoraObject::GetAgoraObject()->SetClientRole(CLIENT_ROLE_AUDIENCE);
}
```

### 5. 设置本地视图
如果你想实现一个语音直播，可以直接查看[加入频道](#join)。

成功初始化 IRtcEngine 对象后，需要在加入频道前设置本地视图，以便主播在直播中看到本地图像。参考以下步骤设置本地视图：
- 调用 `enableVideo` 方法启用视频模块。
- 调用 `setupLocalVideo` 方法设置本地视图。

```C++
// 启用视频模块。
m_lpRtcEngine->enableVideo();
 
// 设置本地视图。
VideoCanvas vc;
vc.uid = 0;
vc.view = hVideoWnd;
vc.renderMode = RENDER_MODE_TYPE::RENDER_MODE_HIDDEN;
m_lpRtcEngine->setupLocalVideo(vc);
```

<a name="join"></a>
### 6. 加入频道
完成设置角色和本地视图后（视频直播场景），你就可以调用 `joinChannel` 方法加入频道。你需要在该方法中传入如下参数：

- `channelName`: 传入能标识频道的频道 ID。输入频道 ID 相同的用户会进入同一个频道。

* token：传入能标识用户角色和权限的 Token。可设为如下一个值：
   * `NULL`
   * 临时 Token。临时 Token 服务有效期为 24 小时。你可以在 Dashboard 里生成一个临时 Token，详见[获取临时 Token](https://docs.agora.io/cn/Agora%20Platform/token?platform=All%20Platforms#获取临时-token)。
   * 在你的服务器端生成的 Token。在安全要求高的场景下，我们推荐你使用此种方式生成的 Token，详见[生成 Token](../../cn/Audio%20Broadcast/token_server.md)。

 <div class="alert note">若项目已启用 App 证书，请使用 Token。</div>

- `uid`: 本地用户的 ID。数据类型为整型，且频道内每个用户的 `uid` 必须是唯一的。若将 `uid` 设为 0，则 SDK 会自动分配一个 `uid`，并在 `onJoinChannelSuccess` 回调中报告。

```C++
// 在频道中开启 Web SDK 与 Native SDK 互通。
m_lpRtcEngine->enableWebSdkInteroperability(true);
 
 
// 加入频道。
BOOL CAgoraObject::JoinChannel(LPCTSTR lpChannelName, UINT nUID, LPCSTR lpDynamicKey)
{
    int nRet = 0;
    LPCSTR lpStreamInfo = "{\"owner\":true,\"width\":640,\"height\":480,\"bitrate\":500}";
#ifdef UNICODE
    CHAR szChannelName[128];
    ::WideCharToMultiByte(CP_ACP, 0, lpChannelName, -1, szChannelName, 128, NULL, NULL);
    nRet = m_lpAgoraEngine->joinChannel(lpDynamicKey, szChannelName, lpStreamInfo, nUID);
#else
    nRet = m_lpAgoraEngine->joinChannel(lpDynamicKey, lpChannelName, lpStreamInfo, nUID);
#endif
    if (nRet == 0)
        m_strChannelName = lpChannelName;
    return nRet == 0 ? TRUE : FALSE;
}
```

### 7. 设置远端视图
视频直播中，不论你是主播还是观众，都应该看到频道中的所有主播。在加入频道后，可通过调用 `setupRemoteVideo` 方法设置远端主播的视图。

远端主播成功加入频道后，SDK 会触发 `onFirstRemoteVideoDecoded` 回调，该回调中会包含这个主播的 `uid` 信息。在该回调中调用 `setupRemoteVideo` 方法，传入获取到的 `uid`，设置该主播的视图。

```C++
// 在引擎收到第一帧远端视频流并解码成功时，会触发此回调。
void CAGEngineEventHandler::onFirstRemoteVideoDecoded(uid_t uid, int width, int height, int elapsed)
{
    LPAGE_FIRST_REMOTE_VIDEO_DECODED lpData = new AGE_FIRST_REMOTE_VIDEO_DECODED;
    lpData->uid = uid;
    lpData->width = width;
    lpData->height = height;
    lpData->elapsed = elapsed;
    if(m_hMainWnd != NULL)
        ::PostMessage(m_hMainWnd, WM_MSGID(EID_FIRST_REMOTE_VIDEO_DECODED), (WPARAM)lpData, 0);
}
```

```C++
VideoCanvas canvas;
canvas.renderMode = RENDER_MODE_FIT;
POSITION pos = m_listWndInfo.GetHeadPosition();
......
AGVIDEO_WNDINFO &agvWndInfo = m_listWndInfo.GetNext(pos);
canvas.uid = agvWndInfo.nUID;
canvas.view = m_wndVideo[nIndex].GetSafeHwnd();
agvWndInfo.nIndex = nIndex;
 
// 设置远端视图。
CAgoraObject::GetEngine()->setupRemoteVideo(canvas);
```

### 8. 离开频道
根据场景需要，如结束通话、关闭 App 或 App 切换至后台时，调用 `leaveChannel` 离开当前通话频道。

```C++
BOOL CAgoraObject::LeaveCahnnel()
{
    m_lpAgoraEngine->stopPreview();
     
    // 离开频道。
    int nRet = m_lpAgoraEngine->leaveChannel();
    m_nSelfUID = 0;
    return nRet == 0 ? TRUE : FALSE;
}
 
 
void CAgoraObject::CloseAgoraObject()
{
    if(m_lpAgoraEngine != NULL)
        // 释放 IRtcEngine 对象。
        m_lpAgoraEngine->release();
    if(m_lpAgoraObject != NULL)
        delete m_lpAgoraObject;
    m_lpAgoraEngine = NULL;
    m_lpAgoraObject = NULL;
}
```

### 示例代码
你可以在 [OpenLive-Windows](https://github.com/AgoraIO/Basic-Video-Broadcasting/tree/master/OpenLive-Windows) 示例代码中查看完整的源码和代码逻辑。
## 运行项目
在 Windows 设备中运行该项目。当成功开始视频直播时，主播可以看到自己的画面；观众可以看到主播的画面。

