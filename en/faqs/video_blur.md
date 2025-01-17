
---
title: The video is blurry in a call.
description: 视频模糊
platform: All Platforms
updatedAt: Mon Jul 01 2019 15:11:51 GMT+0800 (CST)
---
# The video is blurry in a call.
Blurry videos may be caused by low bitrates and resolution ratios. 

## Step 1: Self-check

Check the following:

* Check `videoProfile`. If possible, set `videoProfile` to a higher level to see whether the video is clearer.
* Check the stream type of the receiver. If the stream type is low, call the `setRemoteVideoStreamType` method to switch from a low stream to high stream.
* Switch to 4G or another Wi-Fi network to ensure that the blurry video is not caused by poor Internet connections.
* Turn off all pre-processing options.

## Step 2: Contact Agora Customer Support

If the issue persists, contact Agora customer support and submit the issue with the following information:
* The uid of the user who sees the blurry video.
* The time frame during which the blurry video appears.
* SDK logs and screen recording files of the user.

## Step 3: Monitor the Quality of Experience in Agora Analytics in Dashboard

You can check the statistics of every call in [Agora Analytics](https://dashboard.agora.io/analytics/call/search) in Dashboard. For more information, see [Agora Analytics Tutorial](https://dashboard.agora.io/analytics/call/tutorial?_ga=2.197716463.1125435494.1542623251-764614247.1539586349).
