---
title: How SDN shape NetWorking?
date: 2018-01-27 09:45:48
categories: SDN
---

# How SDN shape Network
> 这篇文章是基于 [Nick McKeown在2011的演讲](https://www.youtube.com/watch?v=c9-K5O_qYgA). Nick McKeown是斯坦福大学的教授,是SDN创始人之一. 他历年来的演讲,可以说是追溯SDN的起源和演进的关键.

## SDN 引起服务器结构的改变.

> SDN引起的结构变化,和计算机的演进过程很相似.

### 计算机的演进

* 在上世纪80年代, 计算机主要是 IBM 主导的, 计算机的结构如下图所示.
是由:
	* 特定的硬件
	* 特定的操作系统
	* 特定的应用
构成. 但是这样结构够过于封闭,一旦集成,很难更新, 产业规模很小,发展缓慢.

* 后来, 微型通用处理器出现后, 计算机有了通用的处理器,通用的操作系统,以及,高度可定制的软件应用. 演进就非常快.
![计算机转变](/img/计算机的演进.png)

### 网络服务器,是否可以像计算机一样演进呢?

* 原来的网络服务器架构,也是特殊化,的封闭的集成,很难扩展,革新缓慢.如下图所示.

* 通过SDN,  我们可以使用商业化通用的芯片, 控制面通过Open Interface控制底层的数据面,上层应用通过Open Interface在控制面编程,实现了控制面和数据面的解耦.
![服务器的改变](/img/服务器的演进.png)

## SDN 下的架构是怎样的?

* 首先我们需要有Open Interface,来处理包转发.
* 其次,我们应该有至少一个Net OS. 
![SDN下架构](/img/SDN下架构.png)

### 分布式的操作系统
> 网络结构一般是分布式的结构,需要一个操作系统来实时展示网络的拓扑结构,来管理整个网络.  一个网络操作系统是很重要的. 举个例子:

>  OSPF: 开放式最短路径优先, 是描述网络链路的方法, 是对链路状态路由协议的一种实现, 隶属内部网管协议,运作于自治系统内部, 内部采用Dijkstra's 来计算最短路径树. 数据库LSDB 用来保存当前的网络拓扑结构.....

> RFC 是一种IETF 是发布的一系列备忘录. 

* OSPF 在RFC 用245页来描述.
* 其中101页 是用来描述一个分布式的系统
* 系统建立好之后, 核心的 Dijkstra's 算法,只用了4页就描述好了.

> 由此可见,一个通用的网络系统构建好的话, 我们实现相应的功能会非常简单.

### 开放的接口--- OpenFlow Forwarding Abstraction

> 在OS 和数据面之间,还应该Open Interface 来管理 包转发. 这就是我们所说的 OpenFlow 协议. OpenFlow 协议是怎样一个概念? 以下图为例:

![OpenFlow](/img/OpenFlow.png)

* 可以看到, OS通过管理 FlowTable 来管理包转发. 通过动态的添加相应的Entries, 来实现对不同的包,不同的处理.
* 每一个Flow header实际上是一个五元组, 也就是所谓的 MicroFlow.包括协议类型,目的地址,源地址等. 用于对flow进行标记.
* 每一个Entry 由 Match 和 Action 组成, 意思是,如果flow 与 Match中的header 匹配,那么就采取, Match后面相应的Action , 例如将这个flow中的包,转发到某一个网络端口..

![FlowTable](/img/Match.png)
![FlowTable](/img/Action.png)

## How SDN shape networking

### Empower network owners/operators

* 他使网络工作者的能力变强,可以实现更多的功能. Nick 教授举了个例子----负载均衡.

> 负载均衡是指,当大量的 网络请求发送到服务器集群, 通过一定的策略,将这些请求均匀的分布到每一个服务器, 使每一个服务器的 负载相对较低.

![负载均衡](/img/负载均衡.png)

* 刚刚大学毕业的学生,只用了500行代码,就在SDN架构下,实现了负载均衡.
* 可见, SDN下, 进行网络功能的开发是非常便捷, 提高了网络工作者的能力.

### Increase the pace of innovation

> 增加了演进的速度. 是以软件的发展速度演进. 如果我们对于硬件有了很好的接口,如果这些接口非常狭窄, 我们就可以在软件上进行完整的仿真测试, 而不是花费数百万,组建Lab 进行实验(在此之前,很多大公司就是这样测试的). 因为在商业交换机和路由器上,做完整的仿真实验,往往是不实际的.这样限制了网络的演进. 如果我们有相对狭窄的接口, 我们就可以更好的测试.

* Fast: 我们可以更快的仿真.  因为较窄的接口, 我们甚至可以在一台笔记本上, 在 Kernel space 仿真10个交换机, 在UserSpace, 仿真Host. 而不必买数千个交换机,进行实际的仿真测试.

* Rapid transfer: 可以更快的转移到 实际运行的网络. (只需要把代码移植过去就可以)

* Code availiable : 可以更好的分享和使用现成的代码.

![仿真测试](/img/仿真测试.png)

### 多种多样的生产链路

* 由于开发新的功能变得容易, 会产生大量的软件提供商. 加快了行业的多样化, 使网络功能迅速丰富起来.

### Build a robust foundation 

> 由于控制面与数据面的解耦, 只需在控制面做相应的调整,就可以实现相应的复杂的网络功能.

> 在传统网络中, 一些简单的问题却很复杂, 比如:

* 比如Header Space 的分析.
	* Can A talk to B? 

* 而利用<Match,Action>,我们对应相应的头部,就可以采取相应的动作,来决定他的去向, 是否可以从A到B.

![Model](/img/Model.png)

* 将一个包映射成一个网络空间的一个点, 而在交换机,路由器上的FlowTable中的每一个Entry, 都由Match,Action组成. 而Match 往往是一个空间.

* 如果一个Flow的header所对应的点, 存在于Match所对应的空间, 则采取,这个空间相应的Action. 这就是它的基本想法.

* 通过这样的方法,我们可以轻松的定义不同的Flow改采取不同的Action, 在此基础上, 我们可以开发出更加复杂的功能,比如防火墙....

## 总结

> SDN 在数据面上,提供了相对狭窄的接口,这些接口具体的实现是OPenFlow 协议. 这个协议引入了 FlowTable 的概念, 每个Table 有多个Entry组成, 每个Entry 的结构是 <Match,Action>. 在底层之上, 有至少一个网络操作系统,用来管理整个网络,调用这些底层的接口, 在OS上,我们可以开发相应的App. 
来实现相应的功能,例如之前提到的 负载均衡.  SDN是我们在控制面进行相应的编程, 而不必考虑底层相应的细节, 就可以开发出复杂的网络功能. 可以快的进行仿真实验, 并只需把相应的代码移植到现有网络,就可以完成现有网络的功能扩展.