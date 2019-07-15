
---
title: 使用云代理服务
description: How to enable cloud proxy on Web SDK
platform: Web
updatedAt: Fri Jul 12 2019 07:35:47 GMT+0800 (CST)
---
# 使用云代理服务
## 功能简介

对于安全需求较高的企业用户，设置防火墙可以限制员工访问非认可的网站，保护内部信息安全。

为避免这些企业因防火墙无法使用 Agora 的服务，Agora 开发了云代理服务。用户只需要在防火墙上将特定的 IP 及端口列入白名单，就可以实现内网访问 Agora 服务。

相比配置单一的代理服务器的 IP，云代理服务更灵活稳定，因此在大型企业、医院、高校、金融等安全需求较高的机构内都有广泛的应用。

## 操作步骤

Agora Web SDK v2.5.1 及以上版本支持云代理服务。开始前请确保你已完成环境准备、安装包获取等步骤，详见[集成客户端 ](../../cn/Audio%20Broadcast/web_prepare.md)。

1. 联系 support@agora.io 申请开通云代理服务，并提供代理服务使用区域、并发规模、网络运营商等信息。

2. 将以下测试 IP 及端口添加到企业防火墙的白名单。
    
	源地址为集成了 Agora Web SDK 的客户端。

   #### 国内测试

   | 协议 | 目标地址            | 端口                  |
   | ---- | ------------- | --------------------- |
   | TCP  | 117.34.213.5  | 443, 4000, 3433, 3444 |
   | TCP  | 47.74.211.17  | 443                   |
   | TCP  | 52.80.192.229 | 443                   |
   | TCP  | 52.52.84.170  | 443                   |
   | TCP  | 47.96.234.219 | 443                   |
   | UDP  | 117.34.213.5  | 3478, 3488            |

   #### 国外测试

   | 协议 | 目标地址             | 端口                  |
   | ---- | -------------- | --------------------- |
   | TCP  | 23.236.115.138 | 443, 4000, 3433, 3444 |
   | TCP  | 47.74.211.17   | 443                   |
   | TCP  | 52.80.192.229  | 443                   |
   | TCP  | 52.52.84.170   | 443                   |
   | TCP  | 47.96.234.219  | 443                   |
   | UDP  | 23.236.115.138 | 3478, 3488            |	 
> 以上 IP 仅供测试阶段调试使用，正式上线前需要向 Agora 申请独立的云代理服务资源。

3. 按照下文的实现方法开启云代理服务，测试是否能正常实现音视频通话。

4. 测试完成后，Agora 会为你部署云代理服务正式环境，并提供相应的 IP 和端口。将 Agora 提供的 IP 和端口添加到企业防火墙的白名单。

## 实现方法

通过调用 `startProxyServer` 方法打开云代理功能，就能方便地使用云代理服务。

### 开启云代理服务

```javascript
var client = AgoraRTC.createClient({mode: 'live',codec: 'vp8'});

// 开启云代理
client.startProxyServer();
// 加入频道
client.init(key, function() {
    client.join(channelKey, channel, null, function(uid) {
        .......
    })
})
```

### 关闭云代理服务

开启云代理服务后，如果需要关闭代理：

1. 离开频道
2. 调用 `stopProxyServer`
3. 重新加入频道

```javascript
// 开启云代理并加入频道后
...
// 离开频道
client.leave();

// 关闭云代理服务
client.stopProxyServer();

// 重新加入频道
client.join(channelKey, channel, null, function(uid) {

 ...

}
```

## 工作原理
Agora 云代理的工作原理如下：
![](https://web-cdn.agora.io/docs-files/1543290381396)

* Agora SDK 在连接 Agora SD-RTN 之前，向云代理发起请求；
* 云代理返回相应代理信息；
* Agora SDK 向云代理发送数据，云代理将接收到的数据透传给 Agora SD-RTN；
* Agora SD-RTN 向云代理返回数据，云代理再将接收到的数据发送给 Agora SDK。
## 开发注意事项

-  `startProxyServer` 和 `stopProxyServer` 必须在加入频道前或离开频道后调用。
- Agora Web SDK 还提供 `setProxyServer` 和 `setTurnServer` 两个方法给用户[自行部署代理服务器](../../cn/Audio%20Broadcast/proxy_web.md)。这两个方法与 `startProxyServer` 不可同时调用，调用了其中任一个方法，再调用 `startProxyServer` 会报错，反之亦然。
-  `stopProxyServer` 会关闭所有代理服务，包括通过 `setProxyServer` 和 `setTurnServer` 设置的代理。