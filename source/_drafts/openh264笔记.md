---
title: openh264笔记
tags:
---

# 更新记录

# 正文

## openh264简介

## 编码流程

### 参数解析

### 实际编码处理 - SVC编码

`welsenc.cpp`-->`main`-->`ProcessEncoding`-->`EncodeFrame`-->`EncodeFrameInternal`-->`WelsEncoderEncodeExt`，此时就到了实际的编码处理。

PrepareEncodeFrame

​	WelsRcCheckFrameStatus

​	DecideFrameType

​	GetTemporalLevel

根据空间层进行循环编码

​	InitFrameCoding

​	AnalyzeSpatialPic

​	假设单slice

​		WelsLoadNal

​		SetSliceBoundaryInfo

​		WelsCodeOneSlice

​		WelsUnloadNal

​		WelsEncodeNal
