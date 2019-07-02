
---
title: 视频黑屏
description: 视频黑屏
platform: All Platforms
updatedAt: Mon Jul 01 2019 15:17:46 GMT+0800 (CST)
---
# 视频黑屏
可能出现黑屏的原因有很多，其中两个最重要的原因是：

* 网络问题：如果本地网络连接很差或者中断，就会看不到其他用户的视频。如果通话中有一方的网络出现问题，其他人也看不到这个用户的视频。
* 用户主动关闭了视频，也会出现黑屏。

## 步骤 1：自检

黑屏主要有以下三种情况：

### 本地视频黑屏远端视频正常

这种情况一般是摄像头故障或者被占用等原因导致本地视频采集出现问题，请按以下步骤排查：

* 摄像头硬件问题。打开系统自带的拍摄视频程序看是否可以录像，如果不行，需要换摄像头；
* 如果摄像头没有问题，看下摄像头权限有没有打开。Android 和 iOS 系统都有权限管理，请在系统设置中检查，同时 Android 上有些安全软件也管理权限；
* 是否有其他应用占据了摄像头。关闭其他应用然后打开我们的应用测试；重启手机再测试；
* 检查是否是用户禁用了本地视频；
* 如果是自采集，需要确认外部采集数据是否有问题。

### 本地视频正常远端视频黑屏

这种情况可能是远端采集问题或者本地下行网络原因导致，请按以下步骤排查：

* 检查用户是否禁用了远端视频；
* 如果没有禁用远端视频，建议换4G网络看下是否还存在问题来排除网络原因。

### 本地远端视频都黑屏

这种情况可能是渲染出现问题或者没有启用视频，请按以下步骤排查：

* 检查是否有调用 `enableVideo` 方法启用视频；
* 检查是否有禁用本地及远端视频；
* 如果上述都没有问题，可能是渲染问题，Windows 端检查 SDK 日志，看渲染方式（render type）是什么。如果是 D2D 渲染，很可能是显卡驱动不是最新，建议去显卡官网更新显卡至最新版。如果更新完官方驱动还是不行，建议走设置 GDI 渲染方式，即加入频道前调用下面的方法：
   * AParameter apm(*pRTCEngine);
   * nRet = apm->setInt("che.video.renderer.type", 9);
* 如果是自渲染，需要确认渲染方式是否有问题。

## 步骤 2：联系技术支持

如果按照上述步骤没有解决问题，请收集下面的信息并联系技术支持：

* 已经做过的排查步骤及结果；
* 黑屏用户的 UID；
* 黑屏的具体时间段；
* 出现问题端用户的 SDK 日志。

## 步骤 3：水晶球监控通话质量

你可以使用 Agora Dashboard 的[水晶球](https://dashboard.agora.io/analytics/call/search)功能对通话质量进行跟踪和反馈，详见[水晶球教程](https://dashboard.agora.io/analytics/call/tutorial?_ga=2.197716463.1125435494.1542623251-764614247.1539586349)。