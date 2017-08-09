---
layout: post
title: Atlas Event Data Model
tag: physics, atlas experiment
---
atlas在run2运行时，对其分析框架进行了一次更新，引入了xAOD格式。此格式主要有以下优点：

- 可同时被Athena和root这两个分析框架读取使用
- 可涵盖所有的重建事例
- 面向对象
- 轻巧紧凑

在引入此格式后，一个完整的atlas实验数据格式流程如下。
![atlas-experiment-data-chain](/assets/img/20170809/data-chain.png)
对于data处理流程，则与一般流程相似。触发判选，全重建，校准等，不再赘述。值得注意的是在全重建后，有3个对于实验分析者来说重要的系统输出：

- AOD: Analysis Object Data,就是我们分析所要使用的各个粒子，物理量的信息，也是分析中最为重要的信息。
- Hist: 全重建时产生的关于data的一些直方图，主要用来得出data quality,重建可信度等信息.
- ESD: 给出事例的重建细节，比如重建时某某探测器的工作状态，径迹重建的情况等。

最终得到的文件形式类似如下：
![data-format](/assets/img/20170809/data-format.png)
对于MC模拟数据的处理流程，也是和一般分析一样经过smearing，模拟重建等。最终得到的文件形式类似如下：
![data-format](/assets/img/20170809/mc-format.png)

## xAOD
