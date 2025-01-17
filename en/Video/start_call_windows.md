
---
title: Start a Call
description: 
platform: Windows
updatedAt: Wed Sep 18 2019 07:44:22 GMT+0800 (CST)
---
# Start a Call
Use this guide to quickly start a basic voice/video call with the Agora SDK for Windows.

## Try the demo
We provide an open-source [Agora-Windows-Tutorial-1to1](https://github.com/AgoraIO/Basic-Video-Call/tree/master/One-to-One-Video/Agora-Windows-Tutorial-1to1) demo project that implements the basic one-to-one video call on GitHub. You can try the demo and view the source code.

## Prerequisites
- Microsoft Visual Studio 2017 or later
- A Windows device running Windows 7 or later
- A valid Agora account. ([Sign up](https://dashboard.agora.io) for free)

<div class="alert note">Open the specified ports in <a href="https://docs.agora.io/cn/Agora%20Platform/firewall?platform=All%20Platforms">Firewall Requirements</a> if your network has a firewall.</div>

## Set up the development environment
In this section, we will create a Windows project and integrate the SDK into the project.

### Create a project
Now, let's build a project from scratch. Skip to [Integrate the SDK](#inte) if a project already exists.

<details>
	<summary><font color="#3ab7f8">Create a Windows project</font></summary>
	
1. Open <b>Microsoft Visual Studio</b> and click <b>Create new project</b>.
2. On the <b>New Project</b> panel, choose <b>MFC Application</b> as the project type, input the project name, choose the project location, and click <b>OK</b>.
3. On the <b>MFC Application</b> panel, choose <b>Application type > Dialog based</b>, and click <b>Finish</b>.
	
</details>

<a name="inte"></a>
### Integrate the SDK
Follow these steps to integrate the Agora SDK into your project.

**1. Configure the project files**

- Go to [SDK Downloads](https://docs.agora.io/en/Agora%20Platform/downloads), download the latest version of the Agora SDK for Windows, and unzip the downloaded SDK package.
- Copy the **sdk** folder of the downloaded SDK package to your project files.

**2. Configure the project properties**

Right-click the project name in the **Solution Explorer** window, click **Properties** to configure the following project properties, and click **OK**.

- Go to the **C/C++ > General > Additional Include Directories** menu, click **Edit**, and input **$(SolutionDir)include** in the pop-up window.
- Go to the **Linker > General > Additional Library Directories** menu, click **Edit**, and input **$(SolutionDir)** in the pop-up window.
- Go to the **Linker > Input > Additional Dependencies** menu, click **Edit**, and input **agora_rtc_sdk.lib** in the pop-up window.

## Implement the basic call
This section introduces how to use the Agora SDK to make a video call. The following figure shows the API call sequence of a basic one-to-one video call.

![](https://web-cdn.agora.io/docs-files/1568261927105)

### 1. Create the UI

Create the user interface (UI) for the one-to-one call in your project. Skip to [Initialize IRtcEngine](#ini) if you already have a UI in your project.

For a video call, we recommend adding the following elements into the UI:
- The local video view
- The remote video view
- The end-call button

When you use the UI setting of the demo project, you can see the following interface:

![](https://web-cdn.agora.io/docs-files/1568792157006)

<a name="ini"></a>
### 2. Initialize IRtcEngine
Create and initialize the IRtcEngine object before calling any other Agora APIs.

In this step, you need to use the App ID of your project. Follow these steps to create an Agora project in Dashboard and get an App ID.

1. Go to [Dashboard](https://dashboard.agora.io) and click the **Project Management** icon ![](https://web-cdn.agora.io/docs-files/1551254998344)  on the left navigation panel. 
2. Click **Create** and follow the on-screen instructions to set the project name, choose an authentication mechanism, and click **Submit**. 
3. On the **Project Management** page, find the **App ID** of your project. 

Call the `createAgoraRtcEngine` method and the `initialize` method, and pass in the App ID to initialize the IRtcEngine object.

You can also listen for callback events, such as when the local user joins the channel, and when the first video frame of a remote user is decoded.

```C++
CAgoraObject *CAgoraObject::GetAgoraObject(LPCTSTR lpAppId)
{
    if (m_lpAgoraObject == NULL)
        m_lpAgoraObject = new CAgoraObject();
    if (m_lpAgoraEngine == NULL)
        // Create the instance.
        m_lpAgoraEngine = (IRtcEngine *)createAgoraRtcEngine();
    if (lpAppId == NULL)
        return m_lpAgoraObject;
    RtcEngineContext ctx;
    ctx.eventHandler = &m_EngineEventHandler;
#ifdef UNICODE
    char szAppId[128];
    ::WideCharToMultiByte(CP_ACP, 0, lpAppId, -1, szAppId, 128, NULL, NULL);
    // Input your App ID.
    ctx.appId = szAppId;
#else
    ctx.appId = lpAppId;
#endif
    // Initialize the RtcEngine object.
    m_lpAgoraEngine->initialize(ctx);
    return m_lpAgoraObject;
}
```

```C++
// Inherit the events and callbacks of IRtcEngineEventHandler.
class CAGEngineEventHandler :
    public IRtcEngineEventHandler
{
public:
    CAGEngineEventHandler(void);
    ~CAGEngineEventHandler(void);
    void SetMsgReceiver(HWND hWnd = NULL);
    HWND GetMsgReceiver() {return m_hMainWnd;};
  
    // Listen for the onJoinChannelSuccess callback.
    // This callback occurs when the local user successfully joins the channel.
    virtual void onJoinChannelSuccess(const char* channel, uid_t uid, int elapsed);
  
 
    // Listen for the onLeaveChannel callback.
    // This callback occurs when the local user successfully leaves the channel.
    virtual void onLeaveChannel(const RtcStats& stat);
  
    // Listen for the onFirstRemoteVideoDecoded callback.
    // This callback occurs when the first video frame of a remote user is received and decoded after the remote user successfully joins the channel.
    // You can call the setupRemoteVideo method in this callback to set up the remote video view.
    virtual void onFirstRemoteVideoDecoded(uid_t uid, int width, int height, int elapsed);
  
    // Listen for the onUserOffline callback.
    // This callback occurs when the remote user leaves the channel or drops offline.
    virtual void onUserOffline(uid_t uid, USER_OFFLINE_REASON_TYPE reason);
private:
    HWND        m_hMainWnd;
};
```

### 3. Set the local video view
If you are implementing a voice call, skip to [Join a channel](https://confluence.agoralab.co/display/TEKP/Windows+1+to+1#Windows1to1-4.Joinachannel).
After initializing the IRtcEngine object, set the local video view before joining the channel so that you can see yourself in the call. Follow these steps to configure the local video view: 
- Call the `enableVideo` method to enable the video module. 
- Call the `setupLocalVideo` method to configure the local video display settings. 

```C++
// Enable the video module.
m_lpAgoraObject->GetEngine()->enableVideo();
  
VideoCanvas vc;
vc.uid = 0;
vc.view = m_wndLocal.GetSafeHwnd();
vc.renderMode = RENDER_MODE_FIT;
// Set the local video view.
m_lpAgoraObject->GetEngine()->setupLocalVideo(vc);
```

### 4. Join a channel
After initializing the IRtcEngine object and setting the local video view (for a video call), you can call the `joinChannel` method to join a channel. In this method, set the following parameters:

- `channelName`: Specify the channel name that you want to join.

* token: Pass a token that identifies the role and privilege of the user.  You can set it as one of the following values:
  * `NULL`.
  * A temporary token generated in Dashboard. A temporary token is valid for 24 hours. For details, see [Get a Temporary Token](https://docs.agora.io/en/Agora%20Platform/token?platform=All%20Platforms#get-a-temporary-token).
  * A token generated at the server. This applies to scenarios with high-security requirements. For details, see [Generate a token from Your Server](../../en/Video/token_server.md).
  
 <div class="alert note">If your project has enabled the app certificate, ensure that you provide a token.</div>

- `uid`: ID of the local user that is an integer and should be unique. If you set `uid` as 0,  the SDK assigns a user ID for the local user and returns it in the `onJoinChannelSuccess` callback.

```C++
BOOL CAgoraObject::JoinChannel(LPCTSTR lpChannelName, UINT nUID,LPCTSTR lpToken)
{
    int nRet = 0;
#ifdef UNICODE
    CHAR szChannelName[128];
    ::WideCharToMultiByte(CP_UTF8, 0, lpChannelName, -1, szChannelName, 128, NULL, NULL);
    char szToken[128];
    ::WideCharToMultiByte(CP_UTF8, 0, lpToken, -1, szToken, 128, NULL, NULL);
    if(0 == _tcslen(lpToken))
        nRet = m_lpAgoraEngine->joinChannel(NULL, szChannelName, NULL, nUID);
    else
        // Join a channel with a token.
        nRet = m_lpAgoraEngine->joinChannel(szToken, szChannelName, NULL, nUID);
#else
    if(0 == _tcslen(lpToken))
        nRet = m_lpAgoraEngine->joinChannel(NULL, lpChannelName, NULL, nUID);
    else
        nRet = m_lpAgoraEngine->joinChannel(lpToken, lpChannelName, NULL, nUID);
#endif
    if (nRet == 0)
        m_strChannelName = lpChannelName;
    return nRet == 0 ? TRUE : FALSE;
}
```

### 5. Set the remote video view
In a video call, you should be able to see other users too. This is achieved by calling the `setupRemoteVideo` method after joining the channel.

Shortly after a remote user joins the channel, the SDK gets the remote user's ID in the `onFirstRemoteVideoDecoded` callback. Call the `setupRemoteVideo` method in the callback, and pass in the `uid` to set the video view of the remote user.

```C++
// Listen for the onFirstRemoteVideoDecoded callback.
// This callback occurs when the first video frame of a remote user is received and decoded after the remote user successfully joins the channel.
LRESULT CAgoraTutorialDlg::OnFirstRemoteVideoDecoded(WPARAM wParam, LPARAM lParam)
{
    LPAGE_FIRST_REMOTE_VIDEO_DECODED lpData = (LPAGE_FIRST_REMOTE_VIDEO_DECODED)wParam;
    VideoCanvas vc;
    vc.renderMode = RENDER_MODE_FIT;
    vc.uid = lpData->uid;
    vc.view = m_wndRemote.GetSafeHwnd();
  
    // Set the remote video view.
    m_lpAgoraObject->GetEngine()->setupRemoteVideo(vc);
    delete lpData;
    return 0;
}
```

### 6. Leave the channel
Call the `leaveChannel` method to leave the current call according to your scenario, for example, when the call ends, when you need to close the app, or when your app runs in the background.

```C++
BOOL CAgoraObject::LeaveChannel()
{
    m_lpAgoraEngine->stopPreview();
    // Leave the current channel.
    int nRet = m_lpAgoraEngine->leaveChannel();
    return nRet == 0 ? TRUE : FALSE;
}
 
 
void CAgoraObject::CloseAgoraObject()
{
    if(m_lpAgoraEngine != NULL)
        // Release the IRtcEngine object.
        m_lpAgoraEngine->release();
    if(m_lpAgoraObject != NULL)
        delete m_lpAgoraObject;
    m_lpAgoraEngine = NULL;
    m_lpAgoraObject = NULL;
}
```

### Sample code
You can find the sample code logic in [AgoraTutorialDlg.cpp](https://github.com/AgoraIO/Basic-Video-Call/blob/master/One-to-One-Video/Agora-Windows-Tutorial-1to1/AgoraTutorial/AgoraTutorialDlg.cpp) file in the [Agora-Windows-Tutorial-1to1](https://github.com/AgoraIO/Basic-Video-Call/tree/master/One-to-One-Video/Agora-Windows-Tutorial-1to1) demo project.
## Run the project
Run the project on your Windows device. You can see both the local and remote video views when you successfully start a one-to-one video call in your app.
