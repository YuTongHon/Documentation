
---
title: 直播 SEI 相关问题 
description: 
platform: All Platforms
updatedAt: Mon Jul 08 2019 16:07:04 GMT+0800 (CST)
---
# 直播 SEI 相关问题 
## 声网 SEI 规范

在默认情况下，声网进行服务端转码推流时，会在转码后的 H264/H265 的 SEI 帧中，增加当前视频的编码信息。该信息为 Json 格式的字符串，具体示例如下：

```json

{
    "canvas": {
        "w": 640,
        "h": 360,
        "bgnd": "#000000"
    },
    "regions": [{
        "uid": 1,
        "alpha": 1.0,
        "zorder": 1,
        "volume": 50,
        "x": 0,
        "y": 0,
        "w": 320,
        "h": 360
    }, {
        "uid": 2,
        "alpha": 1.0,
        "zorder": 1,
        "volume": 89,
        "x": 320,
        "y": 0,
        "w": 320,
        "h": 360
    }],
    "ver": "20180828",
    "ts": 1535385600000,
    "app_data": ""
}

```

各项参数定义如下：
* canvas：画布信息，画布的参数信息如下；
   * w：画布的宽度，单位为像素。主播在 APP 设置的 LiveTranscoding 中的 width 信息；
   * h：画布的高度，单位为像素。主播在 APP 设置的 LiveTranscoding 中的 height 信息；
   * bgnd：画布的背景颜色，RGB 格式，为 16 进制代码表示的字符串。主播在 APP 设置的 LiveTranscoding 中的 backgroundColor 信息；
* regions：主播信息及主播布局信息，为 region 的列表。主播在 APP 设置的 LiveTranscoding 中的 transcodingUsers 信息。region 的参数信息如下；
   * suid：（可选）该区域对应主播的 String 型 User account。该参数适用于启用了 String 型 User account 的主播；
   * uid：该区域对应主播的 ID。主播在 APP 设置的 TranscodingUser 中的 uid 信息；
   * alpha：该区域的透明度，取值范围 [0.0, 1.0]。主播在 APP 设置的 TranscodingUser 中的 alpha 信息；
   * zorder：该区域的层级，取值范围 [1, 100]。主播在 APP 设置的 TranscodingUser 中的 zOrder 信息；
   * volume：该区域对应主播的音量大小，取值范围 [0, 255]；
   * x：该区域在画布中对应的 x 坐标。主播在 APP 设置的 TranscodingUser 中的 x 信息；
   * y：该区域在画布中对应的 y 坐标。主播在 APP 设置的 TranscodingUser 中的 y 信息；
   * w：该区域的宽度。主播在 APP 设置的 TranscodingUser 中的 width 信息；
   * h：该区域的高度。主播在 APP 设置的 TranscodingUser 中的 height 信息；
* ver：版本信息，当前版本为 20190611；
* ts：生成该信息时的时间戳，单位 ms；
* app_data：自定义信息。主播在 APP 设置的 LiveTranscoding 中的 transcodingExtraInfo 信息；

#### SEI 构成

下面是一段 SEI 帧的内容：

```
0000  0664bd7b 22617070 5f646174 61223a22  .d.{"app_data":"
0010  222c2263 616e7661 73223a7b 2262676e  ","canvas":{"bgn
0020  64223a22 23666666 66666622 2c226822  d":"#ffffff","h"
0030  3a363430 2c227722 3a333630 7d2c2272  :640,"w":360},"r
0040  6567696f 6e73223a 5b7b2261 6c706861  egions": [{"alpha
0050  223a3235 352c2268 223a3634 302c2275  ":255,"h":640,"u
0060  6964223a 33313031 32373137 39312c22  id":3101271791,"
0070  766f6c75 6d65223a 32382c22 77223a33  volume":28,"w":3
0080  36302c22 78223a30 2c227922 3a302c22  60,"x":0,"y":0,"
0090  7a6f7264 6572223a 317d5d2c 22747322  zorder":1}],"ts"
00a0  3a313533 37393630 32333537 38332c22  :1537960235783,"
00b0  76657222 3a223230 31383038 3238227d  ver":"20180828"}
```

字段说明：

* 06：SEI 帧；
* 64：用户定义的帧类型，这里声网定义 SEI 类型为 100；
* bd：帧长度。如果帧长度超过 255，例如 922，则表示为 ffffff9d;
* 其余部分：帧内容；

#### Q&A

Q：是不是只要在这儿用了SEI，就不能用信令传布局了？SDK传的是同一个字段，传递方式（信令或SEI）只能二选一？

A：这个是服务端推流时，在 H264/H265 的 SEI 帧中添加的信息，跟 APP 上行发送的数据不是一个概念。与 APP 上行发送的数据唯一相关的是 app_data 字段。

在新直播系统中，有效的方式仅有 LiveTranscoding 这个配置信息，旧直播系统的相关接口不再生效。

Q：布局信息不是通过信令传的嘛？这边为啥也要写上布局信息？不只是音量信息？ 

A：声网一直以来都会在合图推流中，发送 SEI 相关信息，但是一直没有进行规范化。此次修改规范化了 SEI 格式，并增加了音量信息，向前兼容。

加入布局信息的原因是，观众端有可能需要当前的布局信息，进行窗口边框的描绘。
