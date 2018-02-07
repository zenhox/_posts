---
title: 1-OpenFlow概述
date: 2018-01-28 10:42:20
categories: SDN
---
# 深度解析OpenFlow系列
> 基于@SDNLab未来网络学院<深度解析OpenFlow系列课程>

## (一)OpenFlow 概述

![OpenFlowLogo](/img/logo.png)
> Open : 十分开放的协议,规范. 关键是流的概念:
*  Flow : 网络数据实际上是一个流的概念. 用来描述具有相同特征的一组网络数据包的集合. 比如说,来源于同一主机,或者同一个用户.

### 为什么要学习OpenFlow

* OpenFlow是实践SDN首选. 比如我们要分析使用SDN的控制器,或者SDN的交换机(OpenVSwitch..).首先就得对OpenFLow有较好的理解.

* OpenFlow是主流的南向接口. 主流的SDN控制器都对OpenFlow 有支持. 

> 南向接口: 管理其他厂家网关或者设备的接口,即向下提供的接口. SDN控制器对网络的控制主要是通过南向接口协议实现,包括链路发现,拓扑管理,策略制定,表项下发等.

> 北向接口: 提供给其他厂家或者运营商进行介入和管理的接口,即向上提供的接口. SDN北向接口是通过控制器向上层业务应用开放的接口,其目标是使得业务应用能够便利的调用底层的网络资源和能力.

* P4现在非常热门. 而OPenFlow可以说是P4的起源. 是P4和PISA的前身.

> P4: 是一种新的网络编程语言. OpenFlow交换机还不能提供更好的可编程能力,而P4作为新的高级网络编程语言可以很好的弥补这个缺陷.

> PISA: 网络可编程芯片,采用P4编程语言,让企业直接对交换机进行编程.

### OpenFlow 的起源

* Ethane项目是OpenFLOW的前身
  * Martin Casado 领导的项目,面向企业网络管理,是SDN的雏形.
* 2008年的OpenFlow论文的发表,业界关注度达到顶峰.
  * 作者是8位著名网络教授,包括SDN创始人Nick McKenwn. 这篇论文被应用了5792次. 
* OpenFlow 比SDN出现的还要早,最初是用于做网络实验.

### OpenFlow 是什么?

* 首先: SDN != OpenFlow. 这个可以自己思考一下.

* 先从官方定义描述来理解:

ONF(开放网络基金会)的描述:
>  OpenFlow is the first standard communications interface defined between the control and forwarding layers of an SDN architecture. OpenFlow allows direct access to and manipulation of the forwarding plane of network devices.

Wikipedla的描述: 
> OpenFlow is a communications protocol that gives access to the forwarding plane of a network switch of router over the network. OpenFlow enables network controllers to determine the path of network packets across a network of switches.

* OpenFlow 不仅是一个开放的南向接口,还是一个通用转发抽象模型. 会在后面详细讨论.

* OpenFlow 也被称为"网络x86指令集". OF尝试把原来网络处理不太通用的模型,希望能通过技术的演进,得到一个相对通用,相对稳定的模型. 使整个技术上的演进,包括控制器的演进与处理器的演进互不影响.


### OpenFlow 是开放的南向接口
> 之前写到过控制面与数据面的分离,而控制面和数据面的分离是通过建立一个开放的接口(OPen Interface).来实现的.  这个通用的api, 目前主流的就是OpenFlow.

* 但是这个api 与编程语言的api,与北向的api还不太一样. 

* OpenFlow 中有一个流表的概念. 流表会在后面详细介绍,这里可以单纯的把它想象成一张表. 这个表一般由三个部分组成.
  * 匹配域

  * 动作

  * 计数器

| MAC SRC | MAC DST |  IP SRC   | IP DST | TCP DPORT | Action | Counter |
| :-----: | :-----: | :-------: | :----: | :-------: | :----: | :-----: |
|    *    |  10:20  |     *     |   *    |     *     | output |   250   |
|  10:20  |    *    | 192.128.* |   *    |    80     |  drop  |  1980   |
|  ....   |  ....   |   ....    |  ....  |   ....    |  ....  |  ....   |

> 前面5个是Match, 然后紧跟着 Action 和 Counter. 星号代表全部. 可以看出, Match代表一个特征,可以匹配到一系列包,而这一系列包被称为 一个flow. 

* 所以,OpenFlow 的编程是基于flow的编程, 而传统的网络编程时基于地址的. 这是它们的主要区别.



### 通用转发抽象模型

> OpenFlow 在其发展过程中,做了一个尝试: 希望能定义出适用于所有网络转发处理的通用的抽象模型.  

* 传统的网络数据处理模型是一个协议相关的模型. 在网络设备中,分为很多个模块, 比如: MAC地址学习, MAC转发, IP查找表, ACL (介入控制访问)表,  和端口分组表 等.  每个模块都与相应的协议邦定. 每个网络芯片完成之后,其相应的功能就固定了, 比如Mac地址学习模块不能做IP相关的事情, 而整个模型就是相关模块的简单堆砌. 

![传统的网络处理](/img/传统的处理模型.png)

* OpenFlow 尝试定义一种通用的处理模型,即协议无关的处理模型. 它将传统的各种模块统一用一个FlowTable描述. 网络设备由多个流表组成(称为:多级流表流水线处理:pipe-line).  流表就是支撑这个统一处理的抽象模型.

![OpenFlow模型](/img/OpenFlow模型.png)

* 实际上,准确的说是 switch 定义了一种统一的转发处理模型. 这里必须解释,  OpenFlow 只提供了一个通用的转发处理模型, 而网络数据处理模型不仅仅是switch上的包转发, 还有协议处理, Qos等等.   所以, 这也是OpenFlow 的缺陷, 它在协议处理等其他方面并不能 实现可编程. 他只是通向通用处理模型发展过程中的一步, 为更先进的技术 (如P4) 做出了基础.

### OpenFlow 发展历史

* 最广泛使用的是 1.3 和 1.5

![OF发展历程](/img/OF发展历程.png)

### OpenFlow 的历史意义

* 它是网络设备的API.
* 它是网络x86指令集的尝试

> Bye~
