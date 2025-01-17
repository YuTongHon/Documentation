
---
title: Use String User Accounts
description: 
platform: iOS,macOS
updatedAt: Wed Sep 18 2019 03:28:32 GMT+0800 (CST)
---
# Use String User Accounts
## Introduction
Many apps use string usernames. To reduce development costs, Agora adds support for string user accounts. Users can now directly use their string usernames as user accounts to join the Agora channel.

For other APIs, Agora uses the integer user ID for identification. Agora maintains a mapping table object that contains the string user account and integer user ID. You can get the user ID by passing in the user account, and vice versa.

To ensure smooth communication, all the users in a channel should use the same type of user account, that is, either the integer user ID, or the string user account.



## Implementation

Ensure that you understand the steps and code logic for implmenting the basic real-time communication functions. For details, see the following documents:

- iOS: [Start a Call](../../en/Voice/start_call_ios.md) or [Start a Live Broadcast](../../en/Voice/start_live_ios.md).
- macOS: [Start a Call](../../en/Voice/start_call_mac.md) or [Start a Live Broadcast](../../en/Voice/start_live_mac.md).

Follow the steps to join an Agora channel with a string user account:

- After initializing the AgoraRtcEngine object, call the `registerLocalUserAccount` method to register a loca user account.
- Call the `joinChannelByUserAccount` method to join a channel with the registered user account.
- Call the `leaveChannel` method when you want to leave the channel.

### API Call Sequence

The following diagram shows how to join a channel with a string user account:

![](https://web-cdn.agora.io/docs-files/1568711985554)

**Note**:

- The `userAccount` parameter in the `registerLocalUserAccount` and `joinChannelByUserAccount` methods is mandatory. Do not set it as null.

- The `registerLocalUserAccount` method is optional. To join a channel with a user account, you can choose either of the following:

  - Call the `registerLocalUserAccount` method to create a user account, and then the `joinChannelByUserAccount` method to join the channel.
  - Call the `joinChannelByUserAccount` method to join the channel.

  The difference between the two is that for the former, the time elapsed between calling the `joinChannelByUserAccount` method and joining the channel is shorter than the latter.

- For other APIs, Agora uses the integer user ID for identification. You can call the  `getUserInfoByUid` or `getUserInfoByUserAccount` method to get the corresponding user ID or user account without maintaining the map.

### Sample Code

You can also refer to the following code snippets and implement string usernames in your project:

```swift
func joinChannel() {
  // Registers the local user account before joining the channel.
  let myStringId = "someStringId"
  agoraKit.registerLocalUserAccount(userAccount: myStringId, appId: myAppId)
  // Joins the channel with the registered user account.
  agoraKit.joinChannel(byUserAccount: myStringId, token: Token, channelId: "demoChannel1") {
    sid, uid, elapsed) in
  }
}
```

We provide an open-source [String-Account](https://github.com/AgoraIO/Advanced-Video/tree/master/String-Account) demo project that implements string user accounts on Github. You can try the demo and view the source code of the `joinChannel` methods in the [VideoChatViewController.swift](https://github.com/AgoraIO/Advanced-Video/blob/master/String-Account/Agora-String-Account-iOS-Swift/Agora%20iOS%20Tutorial/VideoChat/VideoChatViewController.swift) file.

### API Reference

- [`registerLocalUserAccount`](https://docs.agora.io/en/Voice/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/registerLocalUserAccount:appId:)
- [`joinChannelByUserAccount`](https://docs.agora.io/en/Voice/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/joinChannelByUserAccount:token:channelId:joinSuccess:)
- [`getUserInfoByUid`](https://docs.agora.io/en/Voice/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/getUserInfoByUid:withError:)
- [`getUserInfoByUserAccount`](https://docs.agora.io/en/Voice/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/getUserInfoByUserAccount:withError:)
- [`didRegisteredLocalUser`](https://docs.agora.io/en/Voice/API%20Reference/oc/Protocols/AgoraRtcEngineDelegate.html#//api/name/rtcEngine:didRegisteredLocalUser:withUid:)
- [`didUpdatedUserInfo`](https://docs.agora.io/en/Voice/API%20Reference/oc/Protocols/AgoraRtcEngineDelegate.html#//api/name/rtcEngine:didUpdatedUserInfo:withUid:)

## Considerations
- Do not mix parameter types within the same channel. If you use SDKs that do not support string usernames, only integer user IDs can be used in the channel. The following Agora SDKs support string user accounts:
  - The Native SDK: v2.8.0 and later.
  - The Web SDK: v2.5.0 and later.
- If you change your app usernames into string user accounts, ensure that all app clients are upgraded to the latest version.
- If you use string user accounts to join the channel, ensure that the token generation script on your server is updated to the latest version, and that you use the same user account or its corresponding integer user ID to generate a token. Call the `getUserInfoByUserAccount` method to get the user ID that corresponds to the user account.
- If the Native SDK and Web SDK join the same channel, ensure that the user identification types are the same. For the Web SDK, the `uid` parameter can be set either as a number or as a string.
