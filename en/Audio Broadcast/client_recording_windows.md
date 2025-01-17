
---
title: Record the Audio from the Client
description: 
platform: Windows
updatedAt: Wed Feb 20 2019 06:49:55 GMT+0800 (CST)
---
# Record the Audio from the Client
## Introduction

You can record the audio of all the users in a call and save it on the client for future replays. 

The Agora Native SDK supports recording the audio of all the users in a channel and saves the recording into one file in the following formats: 

- WAV: Large file (lossless compression)
- AAC: Small file (lossy compression)

## Implementation

````C++
// Initialize the RtcEngineParameters object.
RtcEngineParameters rep(*lpRtcEngine);

// Start local audio recording. 
#ifdef UNICODE
 CHAR aFilePath[MAX_PATH];
 ::WideCharToMultiByte(CP_UTF8, 0, filePath, -1, aFilePath, MAX_PATH, NULL, NULL);
int nRet = rep.startAudioRecording(aFilePath, // Valid path to the local recording file.
	AUDIO_RECORDING_QUALITY_TYPE::AUDIO_RECORDING_QUALITY_HIGH // AUDIO_RECORDING_QUALITY_HIGH|MEDIUM|LOW
	);
#else
int nRet = rep.startAudioRecording(filePath, AUDIO_RECORDING_QUALITY_TYPE::AUDIO_RECORDING_QUALITY_HIGH);
#endif

// Stop audio recording. 
int nRet = rep.stopAudioRecording();

````

### API Reference

* [`startAudioRecording`](https://docs.agora.io/en/Audio%20Broadcast/API%20Reference/cpp/classagora_1_1rtc_1_1_rtc_engine_parameters.html#acb567614081900eaaf94d02b7c809af5)
* [`stopAudioRecording`](https://docs.agora.io/en/Audio%20Broadcast/API%20Reference/cpp/classagora_1_1rtc_1_1_rtc_engine_parameters.html#ac5f5a19d5f32d7f7d7d2765caafcdaec)

## Considerations

- Start the recording after joining a channel.
- The recording automatically stops if the local user leaves the channel. 
