---
title: Python中的列表
date: 2018-02-03 16:30:06
categories: Python
---

# Python中的列表

> 惭愧. 跑大二才学习Python.   借用python社区名人Bruce Eckel 的名言:
```
Life is short, You need python.
```

> Python社区的理想就是创建一款干净,简单,完美的语言. 学习了两三天, 真的感觉Python的一切设计,都是为了解决曾经让程序员感觉痛苦的问题. 
>
> 看到Python的列表, 再想想曾经的学习过的数组. 我的天, 这不是我一直苦苦寻找的东西吗? 
>
> 由于列表和以前学习过的数组有很大的差异性, 在此做了一些笔记, 留作以后复用的指南吧~



## 鱼龙混杂

> 列表的最大特性莫过于没有类型之分,只是一些相关元素的简单堆砌. 例如: int可以和str混在一个容器当中.



## 向列表中添加元素

> 列表是一个对象,既然是一个对象,就有相应的行为. 其中append , extend, insert可以用来添加元素.

* append(element):
  * 适用场景: 在尾部添加一个元素.
  * 缺点: 不能添加多个元素
* extend(element或者另一个列表)
  * 可以将需要追加的多个元素,制成一个列表作为extend的参数,将它们添加到列表尾部.
* insert(element)
  * 适用: 在列表中间的某一个位置添加元素.



## 从列表中删除元素

> 在列表中删除元素, 一种是根据元素名(值)的删除. 一种是根据下标的删除.

* remove(element)
  * 特点: remove是列表对象的一个成员函数. 只能根据元素名(值)删除元素.
* del list[x]
  * 特点: del是一个语句, 不是列表的函数. 
  * 可以通过del list来删除整个列表



## 列表分片

> 这个和matlab向量语法很类似.  list[start : end : interval]
>
> 如果start不写,表示列表最开始, end不写,表示到列表最后. interval不写, 表示间隔为1,即依次输出.

```python
list = range(0,5,1)  ##创建[0,1,2,3,4]
list
	[0,1,2,3,4]
list[1:3]
	[1,2]
list[:3]
[0, 1, 2]
list[:]
	[0, 1, 2, 3, 4]
list[::2]
	[0, 2, 4]
list[::-1]  ##获得反向列表
	[4, 3, 2, 1, 0]
```

### 分片拷贝

> 利用列表分片得到初始化的新列表, 只是原来列表的部分或者全部元素的拷贝. 不会随着源列表的改变而改变.
>
> 而直接变量名赋值的新列表,相当于引用. 会随着源列表的变化而变化.例如

```python
list1 = range(1,5)
list2 = list1[:]
list3 = list1
list1.reverse()
list1
	[4, 3, 2, 1]
list2
	[1, 2, 3, 4]
list3
	[4, 3, 2, 1]
```



## 列表操作符

### 大小比较

* 两个列表比较大小和C系strcmp类似, 是一个元素一个元素比较. 一旦有一个对应位置两个列表元素不相等, 那么比较结束,返回结果.

### 列表相加

* 列表之间的相加,是两个列表的合并. 注意, 列表只能和列表相加, 不能通过相加把元素添加到列表尾部.

###  列表乘法

* 当列表与数组相乘,表示将列表扩展相应的倍数.

### in 和 not in

* 如何判断某变量在不在列表中呢? 使用:
  *  element in list[可以使用列表分片]
  *  element not in list

## 多维列表

> 列表也可以像数组一样,实现多维性.

```python
list[x][y]
```



## 列表中的成员

> 列表中有多少小伙伴呢? 不妨让python自己告诉我们:

```
dir(list)
['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__delslice__', 
'__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getslice__', 
'__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__iter__', '__le__', '__len__', '__lt__',
 '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', 
'__rmul__', '__setattr__', '__setitem__', '__setslice__', '__sizeof__', '__str__', 
'__subclasshook__', 'append', 'count', 'extend', 'index', 'insert', 'pop', '
remove', 'reverse', 'sort']

```

> 可以发现,我们之前使用的 remove, append, insert,extend都在里面. 还有几种常用的成员:

* list.pop()   弹出一个元素   ==> 当栈一样的使用
* list.count(element) 统计element元素的个数
* list.index(element) 得到element的下标
  * list.index(element,start,stop) 可以通过三个参数限定搜索范围
* list.reverse()  反转列表
* list.sort(func,key,reverse) 对列表排序  
  * func 排序算法, 默认是归并
  * key 排序关键字
  * reverse 是否逆序排.  默认是false. 如果需要从大到小排序, 则 list.sort(reverse = True)

* 求列表的长度? 可以通过 len(list)得到列表的长度. len是一个BIF.

# 关于元组
> 由于Python的列表过于强大, 所以python的创始人引入了元祖的概念, 关于元组, 和列表最大的区分就是不能随意的修改里面的元素. 必须通过copy的方式,创建一个新的元组.另外, 声明方式也比较特殊.

## 元组的声明
* 元祖的声明常用小括号表示例如:
```python
tuple = (1,2,3)
```
* 但是,元组的关键字并不是小括号, 而是逗号. 例如, 其实去掉括号后
```python
tuple = 1,2,3
tuple
	(1, 2, 3)

```
* 而当我们想要申明一个单元素元组的时候,就不能简单的 tuple = (1).  这里的tuple其实是一个int.  而应该 tuple = 1, 或者 tuple = (1,)
