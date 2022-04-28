---
title: Simulcast的新标准-rfc8853笔记
tags:
  - rfc
  - rtc
  - rtp
  - sdp
  - rfc8853
categories:
  - 笔记
date: 2022-04-18 19:18:23
---


# 更新记录

* 2022-04-18 第一次编辑

# 正文

## Simulcast基本介绍

Simulcast可以翻译成多播/同播/联播，指在RTC应用里面指经过同一个RTP会话（RTP Session）发送同一视频源的不同RTP视频流（RTP Stream）的技术，这些视频流一般拥有不同的帧率、分辨率、码流乃至于编码。

在WebRTC应用当中，启用Simulcast的客户端会同时启用多个编码器编码多个视频流，并将其发送给SFU，使得与会的其他成员可以根据自己的实际需求要求SFU向自己转发其中一路符合需要的码流。如下图所示：

![在SFU中使用联播](https://nyamori.oss-cn-shanghai.aliyuncs.com/img/Simulcast-with-WebRTC.png)

传统的联播实现采用了多SSRC的方式，也叫SDP munging。在这种实现方式下，SDP当中会出现`a=ssrc-group:SIM ssrc1 ssrc2 ssrc3`行，用于描述同一个媒体下的不同视频流。

目前来看，大多数的WebRTC应用和SFU在支持联播特性的时候使用了这种实现方式。早期的Chrome和Safari都采取了这种实现方式。这种SDP大概是这样的（在chrome://webrtc-internals/中可以看到WebRTC的调试信息）：

```
a=ssrc:2639951093 cname:hTlM+qeNGeOz6KNK
a=ssrc:2639951093 msid:bzGeV70XSWsjK6GOPxkKL3qLHglA95aUGSXO d66fcc7b-f965-4f3d-b7a3-0ae5ad472d20
a=ssrc:2230960165 cname:hTlM+qeNGeOz6KNK
a=ssrc:2230960165 msid:bzGeV70XSWsjK6GOPxkKL3qLHglA95aUGSXO d66fcc7b-f965-4f3d-b7a3-0ae5ad472d20
a=ssrc-group:FID 2639951093 2230960165
a=ssrc:2639951094 cname:hTlM+qeNGeOz6KNK
a=ssrc:2639951094 msid:bzGeV70XSWsjK6GOPxkKL3qLHglA95aUGSXO d66fcc7b-f965-4f3d-b7a3-0ae5ad472d20
a=ssrc:2639951095 cname:hTlM+qeNGeOz6KNK
a=ssrc:2639951095 msid:bzGeV70XSWsjK6GOPxkKL3qLHglA95aUGSXO d66fcc7b-f965-4f3d-b7a3-0ae5ad472d20
a=ssrc-group:FID 2639951094 2639951095
a=ssrc:2639951096 cname:hTlM+qeNGeOz6KNK
a=ssrc:2639951096 msid:bzGeV70XSWsjK6GOPxkKL3qLHglA95aUGSXO d66fcc7b-f965-4f3d-b7a3-0ae5ad472d20
a=ssrc:2639951097 cname:hTlM+qeNGeOz6KNK
a=ssrc:2639951097 msid:bzGeV70XSWsjK6GOPxkKL3qLHglA95aUGSXO d66fcc7b-f965-4f3d-b7a3-0ae5ad472d20
a=ssrc-group:FID 2639951096 2639951097
a=ssrc-group:SIM 2639951093 2639951094 2639951096
```

在这段SDP当中`a=ssrc-group:FID`是绑定FEC的描述行，而`a=ssrc-group:SIM`则是描述Simulcast信息的描述行。因此可以看到SSRC `2639951093`、`2639951094`和`2639951096`组成了一个联播组，描述了分辨率依次为低、中、高的一组视频流。

最近在实验Janus的demo的时候，在offer SDP里面就没有携带多SSRC的信息，而是变成了以下类型的SDP：

```
a=rid:h send
a=rid:m send
a=rid:l send
a=simulcast:send h;m;l
```

这种SDP看不到SSRC，但是有一个新增的属性rid，看起来描述了三种不同等级的RTP流。janus的answer SDP中也是类似的格式：

```
a=rid:h recv
a=rid:m recv
a=rid:l recv
a=simulcast:recv h;m;l
```

这种基于rid的格式现在已经有了一个RFC文档--[rfc8853](https://datatracker.ietf.org/doc/html/rfc8853)。

## rfc8853的阅读笔记

rfc8853的标题是` Using Simulcast in Session Description Protocol (SDP) and RTP Sessions`，可以看出来是通过SDP和RTP/RTCP的扩展完成了联播相关的功能。本节对文档的部分描述进行一些记录。

### 一些定义

* rid即Restriction Identifier，定义在[rfc8851](https://datatracker.ietf.org/doc/html/rfc8851)，通过`a=rid`行和offer/answer模式来使得rtp处理者可以去定位某一个特定的rtp流。同时，该行可以做一些比特率，分辨率，帧率相关的描述
* `a=simulcast`行可以指定rid的发送接收方向，`;`用于分隔属性不同的rid，`,`用于分隔同类的rid

