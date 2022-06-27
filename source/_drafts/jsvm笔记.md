---
title: jsvm笔记
categories:
  - 笔记
tags:
---

# 更新记录

# 正文

## JSVM简介

## 编码测试程序部分

编码的测试程序入口在`H264AVCEncoderLibTest.cpp`, `main`函数非常简单，主要就是创建并初始化一个`H264AVCEncoderTest`对象，执行编码之后销毁。主要的处理在`init`函数和`go`函数。

### init函数

本阶段主要是进行编码参数的设置，然后进行编码资源的初始化。

#### 编码参数初始化

主要是读取一些命令行参数，然后根据命令行参数设置相应内容，这里直接引用printHelp函数的内容:

```c++
void EncoderCodingParameter::printHelp()
{
    printf("\n supported options:\n\n");
    printf("  -pf     Parameter File Name\n\n");

    printf("  -bf     BitStreamFile\n");
    printf("  -frms   Number of total frames\n");
    printf("  -gop    GOPSize - GOP size (2,4,8,16,32,64, default: 1)\n");
    printf("  -iper   Intra period (default: -1) : must be a power of 2 of GOP size (or -1)\n");
    printf("  -numl   Number Of Layers\n");
    printf("  -cabac  CABAC for all layers as entropy coding mode\n");
    printf("  -vlc    VLC for all layers as entropy coding mode\n");
    printf("  -ecmf   (Layer) (entropy_coding_mod_flag)\n");
    printf("  -org    (Layer) (original file)\n");
    printf("  -rec    (Layer) (reconstructed file)\n");
    printf("  -ec     (Layer) (entropy coding mode)\n");
    printf("  -rqp    (Layer) (ResidualQP)\n");
    printf("  -mqp    (Layer) (Stage) (MotionQP)\n");
    printf("  -lqp    (Layer) (ResidualAndMotionQP)\n");
    printf("  -meqplp (Layer) (MotionQPLowpass)\n");
    printf("  -ilpred (Layer) (InterLayerPredictionMode)\n");
    printf("  -blid   (Layer) (BaseLayerId)\n");
    printf("  -bcip   Constrained intra prediction for base layer (needed for single-loop) in scripts\n");
    //S051{
    printf("  -anasip (Layer) (SIP Analysis Mode)[0: persists all inter-predictions, 1: forbids all inter-prediction.] (File for storing bits information)\n");
    printf("  -encsip (Layer) (File with stored SIP information)\n");
    //S051}
    //JVT-W052 bug_fixed
    printf("  -icsei   (IntegrityCheckSEIEnableFlag)[0: IntegrityCheckSEI is not applied, 1: IntegrityCheckSEI is applied.]\n");
    //JVT-W052 bug_fixed
    //JVT-U085 LMI
    printf("  -tlnest (TlevelNestingFlag)[0: temporal level nesting constraint is not applied, 1: the nesting constraint is applied.]\n");
    //JVT-U116 JVT-V088 JVT-W062 LMI
    printf("  -tlidx (Tl0DepRepIdxSeiEnable)[0: tl0_dep_rep_idx is not present, 1: tl0_dep_rep_idx is present.]\n");
    //JVT-U106 Behaviour at slice boundaries{
    printf("  -ciu    (Constrained intra upsampling)[0: no, 1: yes]\n");
    //JVT-U106 Behaviour at slice boundaries}
    printf("  -kpm       (mode) [0:only for FGS(default), 1:FGS&MGS, 2:always]\n");
    printf("  -mgsctrl   (mode) [0:normal encoding(default), 1:EL ME, 2:EL ME+MC]\n");

    printf("  -eqpc   (layer) (value)         sets explicit QP cascading mode for given layer [0: no, 1: yes]\n");
    printf("  -dqp    (layer) (level) (value) sets delta QP for given layer and temporal level (in explicit mode)\n");
    printf("  -aeqpc  (value)                 sets explicit QP cascading mode for all layers  [0: no, 1: yes]\n");
    printf("  -adqp   (level) (value)         sets delta QP for all layers and given temporal level (in explicit mode)\n");
    printf("  -xdqp   (DQP0) (DDQP1) (DDQPN)  sets delta QP for all layers (in explicit mode)\n");

    printf("  -mbaff  (layer) (Mb Adaptive Frame Field Coding)  \n");
    printf("  -paff   (layer) (Picture Adadptive Frame Field Coding)   \n");

    // JVT-AD021 {
    printf("  -ml   (mode) [0:disabled(default), 1:multi-layer lambda selection, 2:mode1x0.8]   \n");
    // JVT-AD021 }
    printf("  -h       Print Option List \n");
    printf("\n");
}
```

​	这里直接用命令行配置我暂时没成功过，还是用-pf设置配置文件可能可读性更高，也更好用

#### 编码资源初始化

在读取配置完成之后，就可以确定编码的层数（对应命令行参数的-numl，问题：此处的层级按什么分类的？初步推测：此处的layer是空域和质量域一起组成的，时域对应的命令行参数是-gop）。根据编码的层数，会创建相应的输入输出的io句柄。

最后，创建编码器的句柄，初始化start code即可。

### go函数

xInitCodingOrder

初始化了时域的编码顺序，空域暂时没看到

![image-20220615163209294](/home/wuxiaohan/.config/Typora/typora-user-images/image-20220615163209294.png)

时域id的设置方法，由于这部分内容都是固定大小，其实可以提前算好

xWriteSEI

xWritePrefixUnit

SliceEncoder::encodeMbAffSliceSVC

MbEncoder::encodeMacroblockSVC

