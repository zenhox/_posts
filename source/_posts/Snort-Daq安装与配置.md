---
title: Snort/Daq安装与配置
date: 2018-01-30 16:50:10
categories: DPDK
---

# Snort/Daq 安装与配置

> 这两天做实验, 在安装配置Snort的时候遇到不少坑. 在这里做个简单的记录, 希望对更多的人有所帮助.
>
> 环境: Ubuntu 16.04

## Snort 简单介绍

* 说实话,我对Snort 也没有深入的了解, 所以引用了 @百度百科 的解释:

> 在1998年，**Marty Roesch**先生用C语言开发了[开放源代码](https://baike.baidu.com/item/%E5%BC%80%E6%94%BE%E6%BA%90%E4%BB%A3%E7%A0%81)(Open Source)的[入侵检测系统](https://baike.baidu.com/item/%E5%85%A5%E4%BE%B5%E6%A3%80%E6%B5%8B%E7%B3%BB%E7%BB%9F)Snort.直至今天，Snort已发展成为一个多平台(Multi-Platform),实时(Real-Time)流量分析，网络IP数据包(Pocket)记录等特性的强大的网络入侵检测/防御系统(Network Intrusion Detection/Prevention System),即NIDS/NIPS.Snort符合通用公共许可(GPL——GNU General Pubic License),在网上可以通过免费下载获得Snort,并且只需要几分钟就可以安装并开始使用它。snort基于libpcap。

## 相关软件源码压缩包下载

> 这里直接提供了资源, 我就是采用以下版本的压缩包.  如果不愿意冒险,想少走一些弯路, 就可以直接在这里下载.  如果有特殊需求, 可以在[Snort官网](https://snort.org/downloads)下载其他版本.

* [snort-2.9.8.0.tar.gz](/file/snort-2.9.8.0.tar.gz)
* [daq-2.0.6.zip](/file/daq-2.0.6.zip)

## 解压与安装

* 将这两个安装包解压到合适位置. 我在这里以~/dpdkovs/lab3/ 为例子.

```bash
tar xvf snort-2.9.8.0.tar.gz /root/dpdkovs/lab3/
unzip daq-2.0.6.zip /root/dpdkovs/lab3/
#虽然用root操作是一个不好的习惯, 但是为了一切行云流水, 就将就将就吧.
```

* 安装相应的依赖,需要的依赖有很多 : build-essential , libpcap-dev , libpcre3-dev , libdumbnet-dev , flex , bison , zlib1g-dev, liblzma-dev, openssl, libssl-dev,libnghttp2-dev  

> 这些库用包管理器很容易就安装好了, 但是麻烦之处在于, 编译安装还需要 automake-1.14.1 这个库. 虽然这个库本身也可以用包管理器安装, 但是烦人之处在于, 使用包管理器安装的automake 版本是 1.15.  我曾经试过1.15, 甚至是1.14 都不能编译通过. 必须是1.14.1版本才可以. 这里直接提供 1.14.1版本的压缩包. 大家不用四处找啦~

* [automake-1.14.1.tar.gz](/file/automake-1.14.1.tar.gz)

> 为了方便, 和之前两个安装包安装到同一个目录吧.
```bash
tar xvf automake-1.14.1.tar.gz
cd automake-1.14.1/
```
> 如果就这样编译的话, 还是会报错的, 因为没有 autoconf ,是无法编译automake的, 所以编译之前,应该先安装autoconf. 顺便, 把之前提到的所有依赖全部安装好吧

```bash
## 现在是root
apt-get install  -y build-essential libpcap-dev libpcre3-dev libdumbnet-dev  bison flex autoconf zlib1g-dev liblzma-dev openssl libssl-dev 
```

```bash
## Ubuntu 16 only (not Ubuntu 14)
apt-get install -y libnghttp2-dev
```



### 开始编译安装

> 现在所有依赖基本上安装好了,可以开始一个一个安装了.

> 安装顺序 automake -> daq -> snort  其实后面两个顺序没有必然要求

* 安装automake

```bash
## 现在应该在automake 目录下, 不在的话手动切过去
./configure && make && make install 
## 将automake-1.14.1显式添加到命令路径
ln -s ./bin/automake /usr/bin/automake-1.14.1
```

* 安装daq

```bash
cd ../daq-2.0.6/
./configure && make && make install
```

* 安装snort

```bash
cd ../snort-2.9.8.0
./configure --enable-sourcefire && make && sudo make install
```

> 由于我们把所有适当的依赖提前安装好了, 所以编译过来应该顺风顺水.

## 配置Snort

> 安装好之后, 必须配置Snort才能正确使用.

* 执行下面的命令来更新 共享库. 如果跳过这步, 可能会GET AN ERROR!

```bash
ldconfig
```

* 在/usr/sbin 中创建一个软链接

```bash
ln -s /usr/local/bin/snort /usr/sbin/snort
## 有些时候, 这个目录下已经有了. 可以跳过这一步.
```

* 测试Snort. 传递 -V 参数给它, 你会得到这样的结果. 说明安装成功了! 前面的小猪是Snort的logo. 是不是和我一样可爱哈哈哈~

```bash
snort -V
   ,,_     -*> Snort! <*-
  o"  )~   Version 2.9.8.0 GRE (Build 229)
   ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
           Copyright (C) 2014-2015 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using libpcap version 1.8.1
           Using PCRE version: 8.38 2015-11-23
           Using ZLIB version: 1.2.8
```

> 以为这样就结束了?  NO~  配置Snort,让它以NIDS模式运行.  Follow me!

* 在/etc 目录下, 建立起我们的配置架构

```bash
# Create the Snort directories: 
mkdir /etc/snort 
mkdir /etc/snort/rules 
mkdir /etc/snort/rules/iplists 
mkdir /etc/snort/preproc_rules 
mkdir /usr/local/lib/snort_dynamicrules 
mkdir /etc/snort/so_rules

# Create some files that stores rules and ip lists 
touch /etc/snort/rules/iplists/black_list.rules 
touch /etc/snort/rules/iplists/white_list.rules 
touch /etc/snort/rules/local.rules 
touch /etc/snort/sid-msg.map

# Create our logging directories: 
sudo mkdir /var/log/snort 
sudo mkdir /var/log/snort/archived_logs
```

* 我们需要将一些配置文件和动态预处理器cp一份到/etc的相应目录下.这些配置文件是:

  * classiﬁcation.conﬁg 
  * ﬁle magic.conf 
  * snort.conf
  * threshold.conf
  * attribute table.dtd
  * gen-msg.map 
  * unicode.map

  ```bash
  cd ~/snort_src/snort-2.9.9.0/etc/    ## 切换到Snort源码的etc 目录
  cp *.conf* /etc/snort 
  cp *.map /etc/snort 
  cp *.dtd /etc/snort
  cd ~/snort_src/snort-2.9.9.0/src/dynamic-preprocessors/build/usr/local/lib/snort_dynamicpreprocessor  ##根据你的文件树来
  cp * /usr/local/lib/snort_dynamicpreprocessor/
  ```

> 检验下配置, 现在/etc目录下的配置树应该是这样的

```bash
## 执行tree命令 
tree /etc/snort 
/etc/snort 
|-- attribute_table.dtd 
|-- classification.config 
|-- file_magic.conf 
|-- gen-msg.map 
|-- preproc_rules
|-- reference.config 
|-- rules 
| |-- iplists 
| | |-- black_list.rules
| | |-- white_list.rules 
| |-- local.rules 
|-- sid-msg.map 
|-- snort.conf 
|-- so_rules 
|-- threshold.conf 
|-- unicode.map
```

* 现在, 我们需要配置Snort的主配置文件, /etc/snort/snort.conf.  当Snort 运行的时候以它为参数, Snort就会以NIDS的模式运行.


> 我们需要注释掉在Snort配置文件中引用的所有单独的规则文件，因为我们不是单独下载每个文件，而是使用PulledPork来管理我们的规则集，这些规则集将所有规则组合成一个文件。下面这一行将注释掉我们的snort.conf文件中的所有规则集（从540行开始有大约100行注释掉）：


```bash
sed -i "s/include \$RULE\_PATH/#include \$RULE\_PATH/" /etc/snort/snort.conf
```

> 现在我们将手动更改在snort.confﬁ乐一些设置，使用你喜欢的编辑器：

```bash
vim /etc/snort/snort.conf   ##Life is short, I select vim.  Hhhhhhhhhhhhh~
```

> 第45行，HOME_NET应该匹配你的内部网络。在下面的例子中，我们的HOME NET是192.168.122.1，带有一个24位的子网掩码（255.255.255.0):

```bash
ipvar HOME_NET 192.168.122.1/24   
## 你的可不一定是这个.
##  ifconfig | grep "inet add"  执行这个, 查查看!
## 一般是192.168.1.x 或者 10.0.0.x
```

> 在第104行开始，在snort.conf中设置以下文件路径：

```bash
var RULE_PATH /etc/snort/rules 
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules
var WHITE_LIST_PATH /etc/snort/rules/iplists 
var BLACK_LIST_PATH /etc/snort/rules/iplists
## 配置中本身有这些路径变量.  你只需要修改它们的路径值就好~
```

> 为了使测试Snort变得简单，我们要启用local.rules文件，在那里我们可以添加Snort可以提醒的规则。从546行取消注释（去掉散列符号），看起来像这样:

```bash
include $RULE_PATH/local.rules  ## 把include前面的#去掉
```

> 一旦配置文件准备好了，我们将会让Snort验证它是一个有效的文件，并且所有必要的文件都是正确的。

```bash
snort -T -i eth0 -c /etc/snort/snort.conf 
  (...) 
  Snort successfully validated the configuration! 
  Snort exiting
```

* 一旦执行到这一步,说明你的Snort 已经完全配置好了. 接下来我们可以添加自己的rule. 这就是你的事情了.

## 以inline模式监听两个网卡的流量

```bash
snort -i p2p1:p2p2 --daq afpacket --daq-mode inline -c /etc/snort/snort.conf -Q -f 'not ip'
## p2p1:p1p2 是两个网卡 按照你的网卡来
```

# 总结

> 按照惯例,做一个小的总结.  这是一个Snort的配置教程. 参考的官方的文档. 如果有任何问题, 可以写邮件到我的邮箱, 邮箱地址在我的名片下面. 
>
> Bye~
