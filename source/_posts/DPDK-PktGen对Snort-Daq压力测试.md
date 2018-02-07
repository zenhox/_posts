---
title: DPDK-PktGen对Snort/Daq压力测试
date: 2018-01-30 21:55:09
categories: DPDK
---

[TOC]



# DPDK-PktGen对Snort/Daq进行压力测试

> 最近两天用DPDK-PktGen和Snort进行了一次实验.目的是分析daq_afpacket的性能瓶颈.整个实验过程分为以下几个步骤:

* 环境的配置
* 用PktGen向运行Snort的实验机器线速发包
* 根据Snort的输出,计算出使用afpacket模块时snort的转发速率,进行瓶颈判断.

## 环境的配置

> 环境配置实际上是整个实验中比较麻烦的地方,主要分配两个部分.

* 被测试的机器安装和配置Snort/DAQ.
* 测试的机器安装和配置PktGen(或者其他包生成器如:MoonGen)

### Snort/DA的安装和配置

> 这以块作为独立的一部分,我已经写成一篇博客. 详情可以浏览[Snort/Daq 安装与配置](https://zenhox.github.io/2018/01/30/Snort-Daq%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE/). 

### PktGen的安装和配置

> 由于PktGen也在不断演进. 这里我们只是用它的基本功能就可以了. 这里提供一个比较老的版本, 因为它比较容易配置.点击即可下载.

* [PktGen](/file/pktgen.zip)

> 下载完毕之后,解压到相应目录.

```bash
unzip pktgen.zip /path/to/test
```

> 解压之后, 需要先编译. 但是编译需要导入 RTE_SDK 环境变量.因为pktGen是基于DPDK.  首先应该确定是否安装好了DPDK.  并确定DPDK的安装目录. 比如我的DPDK安装在
>
> /root/dpdkovs/dpdk-stable-17.08.1   所以,我在导入环境变量的时候:

```bash
export RTE_SDK=/root/dpdkovs/dpdk-stable-17.08.1/
```

> 紧接着,可以make编译安装了.

```bash
make
```

> 可是,我在编译安装的过程中, 说找不到 x86_64-default-linuxapp-gcc. 这是因为,在DPDK目录中的SDK名称是 x86_64-native-linuxapp-gcc. 所以,这个时候,我们可以修改Makefile中的内容

```bash
vim Makefile  
##找到37行,修改
RTE_TARGET ?= x86_64-default-linuxapp-gcc  => RTE_TARGET ?= x86_64-native-linuxapp-gcc 
```

> 然后应该可以顺利编译了. 获得相应的可执行文件.
>
> 这里必须提及, 一切都是假设DPDK安装正确, 并且绑定了至少两个网卡. 分配了hugepages的情况下.  请运行DPDK/usertools目录下的 dpdk-setup.py 脚本. 检查或配置DPDK网卡.

## 用PktGen向运行Snort的实验机器线速发包

### 启动Snort,并进入inline模式.

> 执行以下命令

```bash
snort -i eno49:eno50 --daq afpacket --daq-mode inline -c /etc/snort/snort.conf -Q -f 'not ip'
## -i 后面是一组网卡. 因为是inline模式, 所以必须将一组网卡作为参数.
## 得到如下运行结果
(...)

   ,,_     -*> Snort! <*-
  o"  )~   Version 2.9.8.0 GRE (Build 229)
   ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
(...)
Commencing packet processing (pid=24750)
Decoding Ethernet
```

> 这个时候说明Snort已经正确运行了. Snort持续监听着我们eno49和eno50这两个网卡.

### 用PktGen线速向运行Snort的机器发送数据包

```bash
cd /path/to/pktgen  ##切换到pktgen的目录
./build/pktgen -c 1 -- -c config -f tx -b 6  ## 利用这个目录进行线速发包
##出现如下反应,则正常运行
(...)
Checking link status.done
Port 0 Link Up - speed 10000 Mbps - full-duplex
Port 1 Link Up - speed 10000 Mbps - full-duplex
APP: Lcore 0 is writing to port 1, queue 0
Lcore  0: 14.858 Mpps, 9.985 Gbps (6 packets per chunk) in 1.2114 sec
Lcore  0: 14.871 Mpps, 9.993 Gbps (6 packets per chunk) in 1.2104 sec
Lcore  0: 14.867 Mpps, 9.991 Gbps (6 packets per chunk) in 1.2107 sec
Lcore  0: 14.872 Mpps, 9.994 Gbps (6 packets per chunk) in 1.2104 sec
(...)
## 每隔大约一秒, 提示发送包的速率.
```

### 双方运行一段时间. 停止发包与Snort监听. 获得Snort的输出结果

> 测试对象:
>
> * 0000:04:00.0 '82599ES 10-Gigabit SFI/SFP+ Network Connection' if=eno49 drv=ixgbe unused=vfio-pci
> * 0000:04:00.1 '82599ES 10-Gigabit SFI/SFP+ Network Connection' if=eno50 drv=ixgbe unused=vfio-pci
>
> 大约运行3min.  重复三次, 获得实验结果.
>
> * 首先是Memory usage summary: (三次是一样的)
>
> ```
> ===============================================================================
> Memory usage summary:
>   Total non-mmapped bytes (arena):       5427200
>   Bytes in mapped regions (hblkhd):      21590016
>   Total allocated space (uordblks):      3390032
>   Total free space (fordblks):           2037168
>   Topmost releasable block (keepcost):   30480
> ===============================================================================
> ```

* 第一次

```
====================================================================
Run time for packet processing was 129.514036 seconds
Snort processed 0 packets.
Snort ran for 0 days 0 hours 2 minutes 9 seconds
   Pkts/min:            0
   Pkts/sec:            0
====================================================================           
   Received:     11904194
   Analyzed:            0 (  0.000%)
    Dropped:    373101332 ( 96.908%)
   Filtered:     12073654 (101.424%)
Outstanding:            0 (  0.000%)
   Injected:            0
====================================================================
```

* 第二次

```
====================================================================
Run time for packet processing was 179.808720 seconds
Snort processed 0 packets.
Snort ran for 0 days 0 hours 2 minutes 59 seconds
   Pkts/min:            0
   Pkts/sec:            0
====================================================================           
Packet I/O Totals:
   Received:     16427510
   Analyzed:            0 (  0.000%)
    Dropped:    770496667 ( 97.912%)
   Filtered:     16448690 (100.129%)
Outstanding:            0 (  0.000%)
   Injected:            0
====================================================================
```

* 第三次

```
====================================================================
Run time for packet processing was 141.825489 seconds
Snort processed 0 packets.
Snort ran for 0 days 0 hours 2 minutes 21 seconds
   Pkts/min:            0
   Pkts/sec:            0
====================================================================           
Packet I/O Totals:
   Received:     13054661
   Analyzed:            0 (  0.000%)
    Dropped:    597670223 ( 97.862%)
   Filtered:     13075841 (100.162%)
Outstanding:            0 (  0.000%)
   Injected:            0
====================================================================
```

## 计算转发速率

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
* 定义转发速率v
$$ 转发速率v = \frac {接收转发的包}{耗费的时间} $$

* 由此求得三次转发速率v1, v2, v3 分别为:

$$ v_1 =91.9Kpps $$

$$ v_2 =91.4Kpps $$

$$ v_3=92.1Kpps $$

* 求得转发速率的平均值:

$$ \bar v = \frac{v_1 + v_2 + v_3}{3} =91.8Kpps $$

* 而理论上, 10Gbps的网卡,转发线速应该是

$$ v_{max} = 14.88Mpps $$

### 结论分析

> 三次测验都有着恐怖的丢包率(均达到90%以上). 导致最后的包转发率远远没有达到线速. 可知, 内核提供的afpacket模块再处理大量小包时,性能非常差, 丢包率太高. 是包转发的性能瓶颈.

