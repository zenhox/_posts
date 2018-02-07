---
title: 有趣的Python函数
date: 2018-02-04 20:33:30
categories: Python
---

# 有趣的Python函数

> 学了一会儿Python函数, 感觉挺有趣的. 对于我这个脚本语言初学者, 有一些地方着实应该做做笔记.

## 函数文档

* 想要对函数做解释, Python中不用传统的注释符#, 而是用引号. 有趣的是,这些被引号框住的代码,不会被执行,但是会和函数一起被保留. 可以通过特殊的方式得到这些文档.

```python
def add(n1,n2):
    """TODO: add n1 and n2.
    :returns: sum of n1 and n2

    """
    pass
    return n1 + n2

add.__doc__
'TODO: add n1 and n2.\n    :returns: sum of n1 and n2\n\n    '

help(add)
Help on function add in module __main__:
    
add(n1, n2)
    TODO: add n1 and n2.
    :returns: sum of n1 and n2
(END)
```

## 关键字参数

> 有时记不清参数顺序,可以指定关键字赋值.

```python
add(n1=2,n2=3)
```

## 收集参数

> 有时候我们不知道参数有多少个

```python
def add(* nums):
## 参数个数可以通过 len(nums)得到
    sum = 0
    for each in nums:
        sum += each
    return sum
## 建议如果还有其他参数,建议把其他参数设置为默认参数
```

## 函数的返回值

* Python中任何函数都会有返回值. 即使没有显式的return. Python也会返回一个None~
* Python中的返回值可以组成一个列表/元组. 通过这样的方式返回多个返回值.

## 全局变量和局部变量

* 函数内部的变量是局部变量, 函数外部的变量是全局变量.
* 需要特别注意的是, 如果在函数内部引用了全局变量. 是可以的. 但是不要尝试去修改全局变量的值. 因为, 修改的只是全局变量的一个拷贝. 全局变量本身是纹丝不动的. 
* 如果真的必要修改全局变量. 可以在使用全局变量前:

```python
glb = 10
def fun():
    global glb
    glb = 20
```

## 内嵌函数

> 函数内部可以定义内嵌函数. 内嵌函数仅限于函数本身使用, 在函数外部是无法访问一个函数的内嵌函数的.

### 闭包(closure)

* 定义:  内部函数,调用外部函数 的参数. 则这个内部函数就是一个闭包.

~~~python
def funX(x):
  def funY(y):
    return x * y
  return funY
i = funX(8)
i(5)
	40
funX(8)(5)
	40
~~~

* 另外需要注意的是, 内部函数中,只能引用外部函数的变量,但是不能修改它. 如果要修改,有两种办法:

  * 在Python2中, 只能通过用容器包裹外部函数变量. 因为容器类型不是存放在栈里面,所以不会被屏蔽掉.如:

  * ```python
    def funX():
        x = [5]
        def funY(y):
            return x[0] * y
        return funY
    ```

  * 在Python3中,引入了nonlocal关键字, 其作用和global类似:

  * ```python
    def funX():
        x = 5
        def funY(y):
            nonlocal x
            return x * y
        return funY
    ```



## 匿名函数

> Python允许使用lambda关键字来创建匿名函数. 原因是:

* Python写一些执行脚本时,使用lambda就可以省下定义函数的过程, 比如说只是写个简单的脚本来管理服务器时,就不需要专门定义一个函数后再写调用, 使用lambda就可以使得代码更加精简.
* 对于一些比较抽象并且整个程序执行下来只需要调用一两次的函数,有时候起个名字也是比较头疼的问题,使用lambda就不需要考虑命名的问题了.
* 简化代码的可读性,由于阅读普通函数经常需要调到开头def定义的位置,使用lambda可以省去这样的步骤.

使用方法如下:

```python
# 普通函数
def add(x,y):
    return x+y

# 匿名函数
g = lambda x,y : x + y  ## 冒号左面是参数, 右面是返回值
g(3,4)
	7
```

## 常用的BIF

### filter()  过滤器

> 我们每天会接触到大量的数据, 过滤器的作用就显得非常重要了.

* filter() 有两个参数, 第一个参数是一个函数,第二个参数是需要被过滤的数据.当然,第一个参数可以为None:

  * 当第一个参数为None时, 直接将第二个参数中为True的值筛选出来.

  * ```python
    filter(None,[1,0,True,False])
    	[1,True]
    ```

  * 当第一个参数为一个函数时,将被过滤数据集中的每一个数据作为这个函数的参数进行运算,筛选出返回值是True的数据.

  * ~~~python
    def odd(x):
        return x%2
    filter(odd,[1,0,3,4,8,7,13,16])
    	[1,3,7,13]
    ~~~



### map() 映射

> map()函数也是一个函数参数,一个列表参数.

* map()的作用是,将数据集中的每一个数据作为函数参数的参数进行运算,直到所有的列表运算完毕,返回由运算结果所组成的列表. 例如:

* ~~~python
  def double(x):
      return x*2
  temp = map(double,[1,0,3,4,8,7,13,16])
  	[2, 0, 6, 8, 16, 14, 26, 32]
  ~~~

  ​