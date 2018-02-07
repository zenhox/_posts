---
title: Python中的字典和集合
date: 2018-02-05 11:10:21
categories: Python
---

# Python中的字典

> Python中的字典类似于Java中的HashMap. 是一个通过键值对存储的容器. 

## 创建字典

> 创建字典的方法有很多种.

### 直接创建

```python
dic1 = {'a':1,'b':2,'c':3}   ## {} 是字典的标志, key和value通过冒号匹配
```

### 通过内置BIF:dict()创建

* ```python
  dic = dict(one=1,two=2,three=3)  ##需要注意的是, one. two...不能加引号.
  ```

* ```python
  dic = dict([('one',1),('two',2),('three',3)])  ## 组成列表
  ```

* ```python
  dic = dict({'one':1,'two':2,'three':3})   ## dict()直接包裹一下....
  ```

* ```python
  dic = dict(zip(['one','two','three'],[1,2,3]))  ## 利用zip方法, 等效于第二种
  ```

> 推荐使用: 第一种, 简洁 明了!



## 访问字典

> 字典的访问. 

### 下标访问法

```python
dic = dict(one=1,two=2,three=3)
dic['one']
	1
```

### 通过内置方法访问

> 通过下标访问有时候会有问题, 比如访问一个不存在的key会报错. 但是使用get方法就不会.

* ```python
  dic.get('one')
  	1
  dic.get('four')   ## 访问不存在的key时,会返回一个None
  ```

* 通过keys() 访问所有的键

```python
dic.keys()
	['three', 'two', 'one']
```

* 通过values() 得到所有的值

```python
dic.values()
	[3, 2, 1]
```

* 通过items()得到所有的键值对

```python
dic.items()
	[('three', 3), ('two', 2), ('one', 1)]
```



## 遍历字典

> 经常需要遍历一个字典, 有了不同的访问方法,自然有对应的遍历方法.  好在Python有无比强大的for语句. 遍历显得异常简单

```python
for each in dic:
    ### 这个访问的可不是items, 而只是keys!
```

```python
for each in dic.keys():
    ###
```

~~~python
for each in dic.values():
    ###
~~~

```python
for each in dic.items():
    ### 这个访问的是item. 每个item是一个元组. each[0]是key. each[1]是value
```



## 其他常用操作

### 清空字典

```python
dic.clear()
```

### 拷贝字典

> 通过标签之间的赋值只是相互传递了一个引用.

```python
cp_dic = dic.copy()
```

### pop和popitem

* pop()是给定键弹出相应的值.

```python
dic.pop('one')
```

* popitem()是弹出一项

```python
dic.popitem()
```

### setdefault()

> setdefault()方法和get()方法有点相似. 但是setdefault()在字典中找不到相应的键时会自动添加, 而值为None.

```python
dic.setdefault('five')
dic
	{'five': None, 'two': 2, 'one': 1}
```

### update()

> 利用update()来更新字典.

```python
dic.update(one=11)
dic
	{'three': 3, 'two': 2, 'one': 11}
```



# Python中的集合

> 集合,在我的世界里, 你就是唯一.

* 集合是无序的,不能通过索引集合中的某一个元素.
* 集合中的每一个元素是唯一的, 不会出现重复的元素.
* 集合会把元素自动排好序.

## 创建集合

* 直接创建

```python
set = {2,3,1,5,7,6,1,2,3}
set
	set([1, 2, 3, 5, 6, 7])
```

* 通过BIF创建

```python
list1 = [2,3,1,5,7,6,1,2,3]
set1 = set(list1)  ##只限于python3. python2会报错.
set1 
	{1, 2, 3, 5, 6, 7}
list1 = list(set1)  ##只限于python3. python2会报错.
list1
	[1,2,3,5,6,7]	
```

## 访问集合

> 由于集合中的元素是无序的,多以并不能像序列那样用下标来进行访问, 但是可以使用迭代把集合中的数据一个一个的都取出来.

```python
set1 = {1,2,3,4,3,1,2,0}
for each in set1:
    print(each,end='')
    
	0 1 2 3 4
```

## 不可变集合

> 有时候希望集合中的数据具有稳定性,像元组一样的不能随意的改变.我么可以定义不可变的集合,这里使用frozenset()

```python
set1 = frozenset([1,2,3,1,2,3,4])
set1
	frozenset({1, 2, 3, 4})
```



​                                                                                        