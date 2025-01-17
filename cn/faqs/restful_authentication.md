
---
title: HTTP 基本认证
description: 
platform: All Platforms
updatedAt: Fri Aug 16 2019 12:16:23 GMT+0800 (CST)
---
# HTTP 基本认证
## 场景描述

在使用 RESTful API 前，你需要在 HTTP 请求头部中填入 `Authorization` 字段，进行认证。本文提供 Java 和 Swift 语言生成 Authorization 字段的示例代码，请参考代码获取相应的 `Authorization` 值。

你需要在代码中填入 Customer ID 和  Customer Certificate。登录 [Dashboard](https://dashboard.agora.io)，点击右上角账户名，进入下拉菜单 RESTful API 页面，即可获取 Customer ID 和 Customer Certificate。

> Customer ID 和 Customer Certificate 仅用于访问 Restful API。

## 示例代码

### Java

- 获取 Authorization 值

	```java
	// 填入你获取到的 Customer ID 和 Customer Certificate
	String plainCredentials = "customerId:customerCertificate";
	// base64Credentials 就是你要的 Authorization 值，是一个使用 Base64 算法编码的 Credential
	String base64Credentials = new String(Base64.encodeBase64(plainCredentials.getBytes()));
	```

- HTTP 认证

	```java
	Request request = new Request.Builder()
	...
		// 在 HTTP 请求头部填入获取到的 Authorization 字段
		.addHeader("Authorization", "Basic <base64Credentials>")
	...
	```

### Swift

- 获取 Authorization 值

	```swift
	let username = "customerId"
	let password = "customerCertificate"
	let loginString = String(format: "%@:%@", username, password)
	let loginData = loginString.data(using: String.Encoding.utf8)!
	// base64LoginString 就是你要的 Authorization 值，是一个使用 Base64 算法编码的 LoginString
	let base64LoginString = loginData.base64EncodedString()
	```

- HTTP 认证

	```swift
	// 在 HTTP 请求头部填入获取到的 Authorization 字段
	let headers = ["Authorization", "Basic <base64LoginString>"]
	```

## 开发注意事项

发送 HTTP 请求时，Authorization 字段的格式为：`Basic <base64Credentials>` 或 `Basic <base64LoginString>`。
