---
title: VimBegin
date: 2018-01-24 16:44:05
categories: Vim
tags: vim
---


# Vim 和 Vi 的入门用法介绍

> 如果你拥有飞一般的码字速度，却头痛于双手频繁在鼠标，方向键，撤销，退格（Backspace）回车，这些键盘上远离打字区的键位之间移动。 那么，VIM将是你的不二之选。我们使用vim文本编辑的器的原因之一，就是你的双手只要放在A D S F (G) (H) J K L ; 这八个标准打字着手位，就可以完成你想要做的任何复杂操作， 将100%的精力用于打字上。
>
> 而且，vim有很大的跨平台性，不仅通用于UNIX以及他的所有发型版本，在WINDOWS和MAC同样适用，你可以直接用它编写代码，你也可以把他内置到你的visual studio，或者eclipse,intellij等许多IDE中。
>
> 不夸张的说，能够熟练使用vim后，你的文本编辑速度至少是从前的1.5倍以上。
>
> 

## Vim – 编辑器之神

 vim 是 vi的升級版本。 要介紹vim， 还得从vi说起。 vi（**vi**sual editor 读作vee-eye） 在所有UNIX系统上都有着近乎相同的形式，可以说是文本编辑器的通用语言。 是一种全屏编辑器， 它可以卷页，移动光标，删除一整行，插入文本等。 对于很多新手来说，vi不仅不直观，而且麻烦—–他那多如牛毛的命令。 但是，一旦你习惯了vi的编辑方式，你就再也不想回头使用任何“简易的”编辑器了。

vim 兼容了vi所有指令，而且更直观，通用，功能更加强大。

## vim中的几种模式

刚刚打开vim，你为发现一个黑色光标在闪烁，你可能想输入一段话，例如“ I LOVE U”，可是却怎样也没有相应。为什么呢？

因为VIM 有四种模式：

- 正常模式（normal-mode）

  正常模式是vim的默认模式，在这个模式，你可以输入一系列指令来完成相应的操作（移动光标，拷贝。。）。你无法输入字符。你可以在其他模式时，按esc返回正常模式。

- 插入模式（insert-mode）

  在正常模式下，你可以输入一个插入指令（稍后会介绍）进入插入模式，这种状态和普通文本编辑器别无二致。你可以输入你想要输入的字符。

- 可视模式（visual-mode）

  在正常模式下按v进入可视模式，在这种模式下，你可以高亮的选择一段字符，进行相应的操作。

- 命令模式（command-mode）

  命令模式则多用于操作文本文件（而不是操作文本文件的内容），例如保存文件；或者用来改变编辑器本身的状态，例如设定多栏窗口、标签或者退出编辑器。

\##插入一段字符

如果你想对他（她）说 “I LOVE U 回车 FOREVER” 如何使用vim插入这段字符呢？

在正常模式下，有多种方式进入插入模式，不同模式应用于不同的场景 。

一当前光标所在位置为中心，

- 你想在当前光标后面插入字符 ——— a
- 你想在当前光标前面插入字符————i
- 你想在当前光标所在行的下一行———o (这里是新建一行，位于当前光标和原下一行之间)
- 你想在当前一行的结尾追加字符———A
- 你想在当前一行的开始添加字符——— I
- 你想在当前一行的上一行添加字符—— O

选取任何一种方式，你都可以进入插入模式， 进入插入模式后，你就可以打出 “I LOVE U”这段字符了。

你可以在任何时候使用esc返回正常模式。

## vim中移动光标

假如你不小心输入成 “I LOVE HER” 怕出乱子， 如何快速的修改呢？ 首先你要做的就是移动光标到相应位置。

在正常模式下，你可以向往常一样，按箭头移动光标。但是vim中特别提供了独特的移动光标形式。

- VIM中的上下左右

  在vim中， 使用 h(左) j（下） k（上） l（右）来移动光标。 也就是说， 移动光标的任务完全在右手上，而且不需要离开基本着手点。 刚开始可能不太适应，习惯后，你就会发现这正是vim的精华所在。

- 全局移动

  有时候你不需要一个一个字符的移动光标，而是需要在单词之间跳跃，或者在整个文本之间跳跃，下面提供了几种常用的移动方式。

  w —— 移动到下一个单词 （W无视标点符号）

  b ——- 移动到上一个单词（B无视标点符号）

  gg —— 移动到整个文件最开始的位置

  G ——–移动到整个文本最末尾的地方

  0 ——–移动到一行的开头（不推荐使用，后面有更好的方法）

  $ ———移动到一行的结尾（同样不推荐使用）

- 结合数字的指令

  在相应命令前添加数字，可以起到效果加倍的作用。例如：

  3l（右） ——— 向右移动三个字符

  2w( 跳单词) ——向右移动2个单词

  3G —————-移动到第三行。

  可能朋友们会说，第三行好找，那么第30行呢，难道要我们一行行的数吗？

  当然不需要！

  这里有个小技巧， 在正常模式下，输入 ‘ : ‘ 冒号 进入命令模式。

  `: set nu`

  输入这个命令 将对所有行标记行号。 大家不用数啦！

  顺便一提，你也可以使用命令模式跳转光标，例如

  `: 30` 跳转到第30行。

  移动光标的话，能熟练使用以上方法，就已经大大跳高了 移动光标的效率了。

## Vim中对文本的常用操作

好了,现在你可以快速移动到 ‘HER’ 的位置，那么如何修改呢？

你可以输入i或者a进入插入模式， 然后按以往的方式， 用退格键 删除重写。

但是，有没有更快的方法呢？ 没有就不叫vim了！

下面介绍一些删除指令：

- x ——— 删除当前光标下的字符
- nx ——— 删除n个当前光标下的字符
- dd———删除一整行
- ndd——– 删除下面n行
- dG———- 从光标位置删除到文本最后（并不难记，前提是你记得G是移动到文本末尾的意思）
- dgg———- 你猜？ 好吧，删除到开头

这些是比较常用的删除指令，所以，你可以移动到 HER 的 H位置，然后3x 把HER删除掉，然后重新插入你心爱的U.

所以，有没有在快一点的方法呢？ 没有就不叫vim了！

还是在正常模式下移动到HER的H 位置。

输入 `cw` 删除当前光标所在的单词，并且直接进入插入模式。

其实我还想说，他可以更快。。。。。。。。。。

- VIM中的替换指令

  `r` 替换光标处的字符， 如移动到 HER 的H ， 按下r 然后按 U 就可以把 H换成U

  `R` 替换当前光标后的n个字符。 这个时候，你在H处按R ，然后 U 空格 空格 就OK了。

好了，现在你的文本是“I LOVE U 回车 FOREVER” 。 如果，你要觉得一个FOREVER 无法表达清楚你的爱意。你需要999个FOREVER。 所以你需要一个复制粘贴指令。

- VIM中的复制粘贴

yy———–复制一整行。

p————-粘贴到当前光标的下一行。

np————你懂得 你需要的只是 999p

这个是一整行的复制，如何实现选中你想要的文本在复制呢？

这个时候visual-mode派上用场了， 移动到 FOREVER 的F 位置， 按住v进入visual模式， 按l向右移动光标（别用箭头了） 你会发现一段高亮的文本， 这个时候你再按y， 就可以复制了。

这里提以下，

之前讲到了删除命令其实更类似于常用编辑器的剪切， 他们其实被保存在了剪切版里面， 你也以使用删除 p粘贴来起到移动，或者撤销删除的操作。

- VIM的撤销（UNDO）

  也许你觉得999个FOREVER太花哨了， 你其实是一个斯文的男孩子（女）。 还是正常点好， 所以你想删除这998个FOREVER 你可以利用之前的删除命令

  `d1gg` 删除到从开始数的第一行

  当然一也可以撤销， 在VIM中，我们用u（UNDO）键 来撤销， U（将一整行恢复为原来的状态）

你会然觉得”I LOVE U 回车 FOREVER” 用两行来写，有点浪费纸，你想把两行合并，怎们做呢？

移动到第一行 按 `J` 即进行合并。

现在你的文本是 “I LOVE U FOREVER”

好了，一封感人的情书写好了，如何保存呢？

## VIM中的保存退出

- 在正常模式在按ZZ 保存并退出
- w 只是保存，不退出
- q 只是退出，不保存
- ！ 有时权限不够（常见于UNIX系统） 可以使用 wq! 来强制保存退出 或者q! 强制退出。

好了，写到这里，vim的基本操作已经讲的差不多了，意思是你应该已经能摆脱鼠标和“简易”的文本编辑器了。 勤加练习，相信你一定可以熟练使用vim的。
