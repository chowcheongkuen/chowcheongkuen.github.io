---
layout: post
title: 七牛与网易易盾视频审核服务简单对比
date: 2021-07-17 15:56
---

一共测试了 8 个视频，8 个视频都存储在七牛云上，涵盖了正常视频、色情视频、涉政视频，文件大小从几 MB 到 300 多 MB。

## 审核速度

测试的 8 个视频里，一部分是七牛审核快，另一部分是网易易盾快。在速度上两者没有特别优胜的。

## 审核严格度

网易易盾比七牛更严格，其中 2 个有色情内容的视频，七牛都检测为正常，而网易易盾是检测为疑似色情。当然网易易盾也出现了一些正常的视频，也检测为疑似有问题。

## 技术对接限制

### 七牛

七牛视频审核回调不支持重试机制，因此提交了一个审核请求后，需要额外设置一个定时任务，在没有收到回调通知时主动请求审核结果。

### 网易易盾

网易易盾在收到审核请求，并且完成审核后，会每隔 10 分钟向回调地址推送一次审核结果，并且会重试。但是如果在推送前主动请求了审核结果，就不会再有回调通知，并且后续再次主动请求会没有审核数据。在需要及时获取审核结果的场景下，这种方式很不友好。因为一般项目都会有正式环境、预发布环境、测试环境等，其中一个环境的后端请求了数据，其他环境就没有了，对集成审核功能有很大的影响。

## 结论

如果对审核结果的及时性没特别要求，可以选择网易易盾的服务。毕竟审核相对严格，不容易出事。如果从开发对接难度上考虑的话，对接七牛会简单些。我个人不喜欢网易易盾每隔 10 分钟推送一次，已经主动请求后就没有后续结果这两个限制。

## Refs

- [七牛API调用视频审核](https://developer.qiniu.com/censor/5620/video-censor)
- [网易易盾点播视频提交接口](https://support.dun.163.com/documents/2018041903?docId=150440843651764224)
