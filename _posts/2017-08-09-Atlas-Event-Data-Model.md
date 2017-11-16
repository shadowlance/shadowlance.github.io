---
layout: post
title: Atlas Event Data Model
category: Physics
---
atlas在run2运行时，对其分析框架进行了一次更新，引入了xAOD格式。在引入此格式后，一个完整的atlas实验数据格式流程如下。
![atlas-experiment-data-chain](/assets/img/20170809/data-chain.png)
对于data处理流程，则与一般流程相似。触发判选，全重建，校准等，不再赘述。值得注意的是在全重建后，现在系统会给出3个对于实验分析者来说重要的系统输出：

- AOD: Analysis Object Data,就是我们分析所要使用的各个粒子，物理量的信息，也是分析中最为重要的信息。
- Hist: 全重建时产生的关于data的一些直方图，主要用来得出data quality,重建可信度等信息.
- ESD: 给出事例的重建细节，比如重建时某某探测器的工作状态，径迹重建的情况等。

最终得到的文件形式类似如下：
![data-format](/assets/img/20170809/data-format.png)
对于MC模拟数据的处理流程，也是和一般分析一样经过smearing，模拟重建等。最终得到的文件形式类似如下：
![data-format](/assets/img/20170809/mc-format.png)

## xAOD
引入xAOD这个格式最主要的目的是帮助框架制作人员更容易的获取data信息。xAOD格式主要有以下优点：

- 可同时被Athena和root这两个分析框架读取使用
- 可涵盖所有的重建事例
- 面向对象
- 轻巧紧凑

从xAOD更容易获得信息主要得益于其拥有辅助数据(auxdata)，通过它我们可以得到一些我们分析经常需要的分布图，也可以简单得到/编辑一些粒子和物理量信息。另外一点是它提供了一系列函数类等来让我们操作粒子和物理量信息。当然我们也可以直接使用auxdata来操作数据，但一般极力不推荐这么做，从官方接口来操作永远比使用一个不是为此而创建的数据更直观方便不易出错。

在xAOD::这个namespace中我们可完成对所有xAOD事例的访问，此namespace包含两个重要的类: EventInfo和IParticle

**EventInfo**

EventInfo主要提供了函数来获得此次实验运行/模拟运行的信息，比如runNumber(运行编号)，eventNumber(产生了多少事例)，Lumi block number(多少个lumi block)等。

**IParticle**

IParticle则是所有粒子，物理量的虚基类。里面存有诸如e(xAOD::Electron)，mu(xAOD::Muon)，tau(xAOD::TauJet)粒子、粒子在量能器中沉积能量、粒子重建径迹等。此外IParticle还提供一系列了访问auxdata的函数。

在xAOD中所有的粒子物理量都以一定的版本存储(如xAOD::Muon_v3)，在Athena分析框架中，类似xAOD::Muon就是其最新版本的别名(typedef)。比如在此xAOD::Muon就是xAOD::Muon_v3。而在root分析框架中无此机制，你只能访问类似xAOD::Muon_v3的类。

## xAOD存储方式

一个典型的xAOD文件包含层级如下：

    AOD.xxxxx.pool.root.1
        ##Shapes;1
        ##Links;1
        ##Params;1
        CollectionTree;1
            DiTauJets/
                ...
            TauJets/
            Muons/
            ...
            StausAux/
            SlowMuonsAux/
            TauJetsAux/
            ...
            ElecteonsAuxDyn.e225
            ElecteonsAuxDyn.ptcone20
            ...
        POOLContainerForm;1
        POOLContainer;1
        POOLCollectiontree;1
        MetaData;1
        MetaDataHdrForm;1
        MetaDataHdr;1

所有的实验/模拟事例均存在CollectionTree中，对于一个初步的实验分析来说这些已经足够，所以其他的tree我们可以暂时不管。在CollectionTree中类似“DiTauJets/”的就表示是一个container(通俗的说就是C++的Vector)，在其中含有这种对象的所有事例，比如这个是DataVector<xAOD::DiTauJets>。而像”StausAux/“这种后缀有”Aux”的就表示这是一个含辅助数据的container。类似ElecteonsAuxDyn.e225的就是单独列出的辅助数据了。
