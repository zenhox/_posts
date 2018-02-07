---
title: 2-OpenFlow通用抽象-上
date: 2018-02-02 15:39:23
categories: SDN
---

[TOC]



# 深入解析OpenFlow系列-上

> 基于SDNLAB-未来网络学院<深度解析OpenFlow>系列课程.

## OpenFlow中的一些术语

> 这里引用一组定义的解释, 一些解释可能会在后面的学习中深入体会.

| 术语            | 解释                                       |
| ------------- | :--------------------------------------- |
| Byte          | 一个8bit的序列,是网络处理的基本单元.                    |
| Packet        | 以太帧,包括header和payload                     |
| Port          | packets进出OpenFlow pipeline的地方, 由物理端口,Switch定义的逻辑端口,OpenFlow协议定义的reserved端口组成. |
| Flow table    | pipeline的一个stage,包含flow entries.         |
| Flow entry    | flow table 中的元素,用于packets 查找和处理,包括: {用于匹配一组match fields,描述匹配顺序(优先级)的priority, 跟踪记录packets的一组counters, 一组待用的instructions.} |
| Match Field   | packet匹配查找时使用的域,包括packet header, 输入端口,和metadata值, 一个match field可以是通配符(匹配任意值),也可以是bitmasked型. |
| Metadata      | 一个Maskable register value,用于table之间的信息传递. 可以理解为在表之间做信息传递的寄存器的值. |
| Instruction   | 当一个packet匹配某个flow entry时,对应的instruction描述特定的OpenFlow处理. 一个instruction可以直接修改pipeline processing 过程,比如直接把packets发给另一个flow table, 也可以包含一组actions用来添加到action set, 或包含一个action list 立即应用给packet. |
| Action        | 转发packet到某一个端口或者修改packet的一个操作. 比如减小TTL值. Actions也可作为flow entry对应的instruction的一部分. 或者在group entry对应的一个action bucket中. Action可以被添加到packet的action set中. 也可以立即应用给匹配的packet. |
| Action set    | 与packet 相关的一组action, 当每个flow table处理packet时,会添加action到action set, 当instruction set 表示packet会送出 pipeline时, action set会被执行在这个packet上. |
| Group         | 是一个action buckets列表, 从其中选择一个或者多个bucket用在每个packet上. |
| Action bucket | 一组actions 以及相关参数, 在Group中定义.             |
| Tag           | 通过push 和pop actions 插入packet或从packet删除的一个header. |
| Outermost tag | 离packet起始位置最近的一个tag                      |
| Controller    | 通过Openflow协议与OFS交互的设备                    |
| Meter         | 一个switch基本单位,能测量和控制packets速率,如果packets速率或byte速率经过meter超出预定义的范围,meter会激活一个meter band, 如果meter band drop packet, 就是所谓的Rate Limiter |



##  通用转发抽象模型

![抽象模型](/img/抽象模型.png)

* 除了定义了FlowTable,还定义了GroupTable,和MeterTable.
* Instruction: 更像计算机中的程序控制指令,而Action更像对数据本身的运算,即对数据包本身的操作.

###  Flow Table处理流程一览

![FlowTable处理流程](/img/FlowTable处理流程.png)

* 网络数据包进入
  * 清空它对应的Action set()
  * 初始化pipeline fields. 类似于metadata
  * 从table0开始匹配
* 开始匹配
  * 如果匹配成功(操作a)
    * 更新流表项的计数器
    * 执行流表项指令instruction:
      * 更新action set{}
      * 是否跳转到其他流表
  * 如果匹配失败
    * 会和table miss 流表项进行匹配. 
      * 如果满足table miss项
        * 执行匹配成功相关操作a
      * 如果依然匹配失败.
        * 丢弃数据包
* 跳转与否
  * 一个表匹配完毕, 该数据包可能有三中结果. 
    * 不匹配任何流表项(包括table miss) => 被丢弃
    * 匹配到流表项, 指令为跳转到table n => 执行跳转,继续匹配.
    * 匹配到流表项, 没有跳转指令 => 执行与数据包对应的action set{}

### 多级流表处理一览

> 刚刚学习了单个表的处理流程, 现在来看看多级流表整体是怎样一个流程.

![多级流表处理](/img/多级流表处理.png)

### Flow Table - 流表子模型

![流表子模型](/img/流表子模型.png)

* Match + Action => Flow Entry => Flow Table => 多级流表.
* 采用多级流表, 是因为如果只有一个流表的话, 表项会太多, 得添加每一种可能的情况, 而采用多级流表, 不仅每一个表的容量变小,而且处理也比较灵活.
* 一个Entry
  ![Entry](/img/entry.png)

### Flow Entry - 流表项

> 随着OF的发展, 流表项的内容也越来越丰富. 

#### 流表项

![流表项](/img/流表项.png)

* 匹配域: 一组网络数据包协议域的组合,用来识别该条表项对应的Flow,也叫待匹配的内容.


* 优先级: 定义这个流表项的匹配优先级,当同时有多条流表项匹配时,优先选择优先级高的流表项.

> 所以匹配域+优先级 就可以唯一标识一条Flow, 称为Flow ID.

##### 匹配域

![匹配域](/img/匹配域.png)

> 但是这种匹配模式,实际上并不是很灵活,OF在1.2版本加入了OXM的TLV匹配域描述;  选择不同类型匹配域的方法是,在匹配域的开头, 添加一个ingress field.
>
> 0 表示标准类型, 1 表示OXM类型. 
>
> 所以, 整个匹配域的结构应该是:
>
> 16位表示类型,其中包括标准和OXM的选择.
>
> 16位表示匹配域长度
>
> 8位表示oxm字段, 如果不是oxm类型,就是0.
>
> 8位表示pad

* TLV:  type-length-value的模式 描述一种匹配协议字段.
* 前32bit的结构:

![32bit](/img/32bit.png)

* 匹配类:
  * OFPXMC_NXM_0
  * OFPXMC_NXM_1
  * OFPXMC_OPENFLOW_BASIC
  * OFPXMX_PACKET_REGS
  * OFPXMC_EXPERMENTER

* 类字段: (描述匹配域的具体类型)

  * 一类是包头的信息, 如以太网的目的地址,等等.
  * pipeline的匹配域.  如输入端口, metadata等等.

* 掩码标志:

  * 只有一位,1表示有掩嘛.

* 载荷长度:

  * 定义了TLV格式的整个填充长度, 不包括后面的填充字节.

* 也许看一个例子,会更好一点:

  * 无掩码目的以太网地址.

  ![例子1](/img/例子1.png)

  * 带掩码的目的以太网地址.

    ![例子2](/img/例子2.png)

