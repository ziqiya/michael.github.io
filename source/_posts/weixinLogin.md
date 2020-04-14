---
title: 微信小程序登录
cover: /covers/weixinLogin.jpg
date: 2020-04-01 15:43:50
category: 技术
tags: [微信小程序, Taro]
excerpt: 关于微信小程序登录的新方法探索
---

## 前言

&emsp;&emsp;之前我在项目中负责微信小程序的登录模块，微信小程序开放平台上提供的是以下做法。

![微信登录流程](/images/posts/weixinLogin/weixin.jpg)

&emsp;&emsp;这个做法是比较官方正式的做法，但是我们团队里用的是另一种流程方法，现在也放在这里对比讨论一下可行性。

## 新的想法

&emsp;&emsp;以下为我们的微信小程序登录流程：

![我的微信登录](/images/posts/weixinLogin/myLogin.jpg)

&emsp;&emsp;这种方式的核心思想是后端把 `session_key` 和 `openid` 传给前端,虽然<b>官网上不推荐这么做</b>:

![官网建议](/images/posts/weixinLogin/tips.jpg)

&emsp;&emsp;不过后端小伙伴已经这么写了，我就联调试试看，结果发现，基本上的流程都是能走通，但是安全性不高，用户可以通过 storage 获得 `session_key`，而且多端登录还有个 bug。

&emsp;&emsp;这个是微信登录的坑，那就是微信小程序的登录需要经过微信端的认证，如果把 `session_key` 放在前端的话，多端登录会有问题，因为我们知道微信登录的 wx.login 每次调用后，原先的 `session_key` 都会失效，这样就无法保证多端设备中 `session_key` 同步性。虽然 Taro.login 对 wx.login 进行了封装，如果 `session_key` 没过期就不会执行 wx.login，也就不会导致原来的`session_key` 过期，但是有一种特殊情况，就是如果在前一个设备上的 `session_key` 已经过期，但是我先在新设备上登录了，刷新了线上的 `session_key`，这时候旧设备上检测的 `checkSession` 还是 success，但是却不是最新的 `session_key`，这时候就会请求失败，一直登录不成功，除非清缓存。

&emsp;&emsp;如此一来就限制了`session_key`只能存在后端，因为在前端是无法根据当前设备本地的`session_key`判断其是否有效的，`checkSession`只能检测线上的`session_key`是否有效，如果是保存在后端就能保证`session_key`的唯一性和实时性。

&emsp;&emsp;所以还是要走微信小程序官网上的流程：把`session_key`存在后端。
