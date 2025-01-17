
---
title: Start a Call
description: 
platform: Electron
updatedAt: Wed Sep 18 2019 05:19:29 GMT+0800 (CST)
---
# Start a Call
Use this guide to quickly start a basic call with the Agora SDK for Electron.

## Try the demo

We provide an open-source demo project that implements [Agora Electron Quickstart](https://github.com/AgoraIO-Community/Agora-Electron-Quickstart) on GitHub. You can try the demo and view the source code.

## Prerequisites

* Node.js 6.9.1 or later
* Electron 1.8.3 or later

<div class="alert note">If you use Windows for development, ensure that you run npm install -D —arch = ia32 electron to install a 32-bit Electron.</div>
<div class="alert note">Open the specified ports in <a href="https://docs.agora.io/cn/Agora%20Platform/firewall?platform=All%20Platforms">Firewall Requirements</a> if your network has a firewall.</div>

## Set up the development environment

In this section, we will create an Electron project, integrate the SDK into the project, and install the dependency to prepare the development environment.

### Create a project

Follow the steps in [Writing Your First Electron App](https://electronjs.org/docs/tutorial/first-app) to create an Electron project. Skip to [Integrate the SDK](#integrate_sdk) if a project already exists.

<a name="integrate_sdk"></a>
### Integrate the SDK

Choose either of the following methods to integrate the Agora SDK into your project.

**Method 1: Install the SDK through npm**

1. Go to the project path, and run the following command line to install the latest version of the Electron SDK:

	`npm install agora-electron-sdk`
	
2. Import the SDK into your project with the following code:

	`import AgoraRtcEngine from 'agora-electron-sdk'`
	
**Method 2: Download the SDK from the Developer Portal**

1. Go to [SDK Downloads](https://docs.agora.io/cn/Agora%20Platform/downloads) to download the Agora SDK for Electron.
2. Save the downloaded SDk package into the root directory of your project file.
3. Import the SDK into your project with the following code:

	`import AgoraRtcEngine from 'agora-electron-sdk'`

<div class="alert note">Ensure that you use Electron 3.0.6 if you want to get the SDK by downloading it from the Developer Portal.</div>

### Switch the prebuild add-on version

By default, Agora uses Electron 1.8.3 to build the project. Switch the prebuilt add-on version in the .npmrc file according to your Electron version:

```javascript
// Downloads a prebuilt add-on with Electron 1.8.3
AGORA_ELECTRON_DEPENDENT = 2.0.0
// Downloads a prebuilt add-on with Electron 3.0.6
AGORA_ELECTRON_DEPENDENT = 3.0.6
// Downloads a prebuilt add-on With Electron 4.0.0
AGORA_ELECTRON_DEPENDENT = 4.0.0
```

### Install the dependency

Under the project path, run `nmp install` to install the dependency and trigger the npm run download command. You can also install the dependency manually.

If you want to debug with Xcode or Visual Studio, run `npm run debug` to generate project files and SDK files for the debug environment.

You have now integrated the Agora SDK for Electron into your project.

## Implement the basic call

Refer to the [Agora Electron Quickstart](https://github.com/AgoraIO-Community/Agora-Electron-Quickstart) demo project to implement various real-time communication functions in your project.

## Open-source SDK

The [Agora SDK for Electron](https://www.npmjs.com/package/agora-electron-sdk) is open source in Github. You can download it and refer to the source code. Agora welcomes contributions from developers to improve the usability of the Electron SDK.






