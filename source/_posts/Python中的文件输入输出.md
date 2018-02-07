---
title: Python中的文件输入输出
date: 2018-02-06 09:31:04
categories: Python
---

# Python 中的文件输入输出

## 打开文件

* open(file, mode = 'r', buffering = -1, encoding = None, errors = None, newLine = None, closefd = True, opener = None)

> open()这个函数有很多参数, 但作为初学者的我们, 只需要先关注第一个和第二个参数即可. 第一个参数是传入的文件名, 第二个参数是指定文件打开的模式.

### 打开模式

| 打开模式 |        执行操作         | 打开模式 |           执行操作           |
| :--: | :-----------------: | :--: | :----------------------: |
| 'r'  |        只读模式         | 'w'  |     写入模式,会覆盖已经存在的文件      |
| 'x'  | 如果文件已经存在,使用此模式会产生异常 | 'a'  | 写入模式打开,如果文件已经存在,则在末尾追加写入 |
| 'b'  |     以二进制模式打开文件      | 't'  |       以文本模式代开(默认)        |
| '+'  | 可读可写模式(可添加到其他模式中使用) | 'U'  |         通用换行符支持          |

> 使用open() 打开文件之后, 它会返回一个文件对象,拿到这个文件对象, 就可以读取或者修改这个文件.



## 文件对象的操作方法

|      文件对象的方法      |                   执行操作                   |
| :---------------: | :--------------------------------------: |
|      close()      |                   关闭文件                   |
|   read(size=-1)   | 从文件读取size个字符,当未给定size或者给定负值的时候,地区剩余的所有字符,然后作为字符串返回 |
|    readline()     |               从文件中读取一整行字符串               |
|    write(str)     |               将字符串str写入文件                |
|  writelines(seq)  |    向文件写入字符串序列seq, seq应该是一个返回字符串的可迭代对象    |
| seek(offset,from) | 在文件中移动文件指针,从from(0代表文件的起始位置,1代表当前位置,2代表文件末尾)偏移offset个字节 |
|      tell()       |               返回当前在文件中的位置                |

### 文件的关闭

* 如果是C语言,文件的关闭显得异常重要.但是Python中有垃圾回收机制,会在文件对象的引用计数降至零的时候自动关闭文件,所以在Python的编程里,如果忘记关闭文件并不会造成内存泄漏那么危险的结果.
* 但是,Python可能会缓存你写入的数据,如果突然的断电等情况,这些缓存的数据还没来得急写入而丢失. 而显示的关闭文件,可以将缓存的数据强制写入文件,所以,为了安全起见,要养成使用完文件之后立即关闭文件.

### 文件的读取

* 文件指针:  读取文件的指针.起到定位的作用. 读取过程中会不断移动文件指针, 可以通过tell()知道文件指针的位置,通过seek()调整文件指针的位置.  读取完一次后再次读取, 不会读出任何信息, 这是因为文件指针已经指向了文件末尾, 可以通过f.seek(0,0) 将指针调整到开头.
* read() 可以读取, 从文件读取size个字符,当未给定size或者给定负值的时候,地区剩余的所有字符,然后作为字符串返回
* readline()读取到换行符.
* Python的列表很强大, 通过 list(f). 可以将文件内容一行一行的存放在列表中.
* 按行遍历文件的方法

```python
for each_line in f:
    print(each_line)
```

### 文件的写入

> 如果要写入文件, 请确保之前的打开模式有'w' 或者 'a', 否则会出错.

* 一半我们这样写入文件:

```python
f = open('test','w')   ### 注意, 'w'模式下, 文件原来内容会被删除.  请考虑使用'a' 模式
f.write("write a line\n")   
f.close()
```



## 一个练习

> 教材提供了一个练习, 的确, 还是动手实战一番显得更加实在一点, 任务目标如下:

* 将文件(record2.txt)中的数据分割并按照以下规则保存起来:
  * 将小甲鱼的对话单独保存为boy.txt的文件(去掉"小甲鱼:").
  * 将小客服的对话单独保存为girl.txt的文件(去掉"小客服:").
* 这个是[record2.txt](/file/record2.txt)
* 这是实例代码:


```python
#setencoding=utf-8

## Split the record2

def judge(line):
    '''judge the line
    if it's 小甲鱼, then return 0
    else return 1
    '''
    str_line =  str(line)
    index = str_line.find('小甲鱼')
    if index == 0:
        return 0
    else:
        return 1


f = open('record2.txt','r')

f1 = open('boy.txt','w')
f2 = open('girl.txt','w')

s = f.readline()
while s != '':
    index = judge(s)
    if index  == 0:
        index2 = s.find('鱼')
        ss = s[index2+2:]
        f1.write(ss)
    else:
        index2 = s.find('服')
        ss = s[index2+2:]

f.close()
f1.close()
f2.close()
print('over')
```

* 这个是运行结果:

```
哦?
哈哈哈,我看到了呀,我还发微博了呢~
OK~
T_T
~
```

```
 小甲鱼,有个好评很好笑哈.
"有了小甲鱼,以后妈妈再也不用担心我的学习了~"
嗯嗯,我看到了你的微博了呀~
哪个有条回复"左手拿着小甲鱼,右手拿着打火机,哪里不会点哪里,so easy"
```



## pickle: 对象流输入输出

> pickle 可以非常容易地将列表,字典等这类复杂地数据类型存储为文件.  
>
> 将所有pythonde的对象转化为二进制的形式存放,这个过程称为pickling
>
> 将二进制形式转换回对象的过程称为unpickling.

### 把对象写入文件

```python
import pickle

my_list = [123,3.14,'小甲鱼',['another list']]
pickle_file = open('mylist.pkl','wb')  ##一定要是可写的,并且以二进制形式打开'b'
pickle.dump(mylist,pickle_file)   ##将mylist 写入文件
pickle_file.close()
```

### 从文件读取对象

```python
import pickle

pickle_file = open('mylist.pkl','rb')   ##依然是二进制打开
my_list = pickle.load(pickle_file)
print(my_list)
```

