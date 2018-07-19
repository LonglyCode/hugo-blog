---
title: python基础零碎笔记
date: 2016-01-20 11:54:22
lastmod: 2018-07-19 17:16:11
tags: ["python", "note"]

---
<!-- toc -->

> 这些是很久之前学python的时候零零散散记录了的一些笔记，现在看来很简单，整理了一下用于开篇的博文吧。python 是优雅的语言，但有些技巧和语法需要注意一下。

<!--more-->
## 1. 循环技巧：

*   在字典中循环时，关键字和对应的值可以用 **items()** 方法解读出来
*   在序列中循环时，索引位置和对应值可以使用 **enumerate()** 函数同时得到
*   同时循环两个或更多的序列，可以使用 **zip()** 整体解读

## 2. 作用域：

*   Python 的一个特别之处在于其赋值操作总是在最里层的作用域。赋值不会复制数据——只是将命名绑定到对象。删除也是如此：“del x ”只是从局部作用域的命名空间中删除命名 x

## 3. slots 方法

*   slots方法其实广为人知，听说有人用这个方法节省了很多内存。slots的作用是阻止在实例化类时为实例分配dict，默认情况下每个类都会有一个dict,通过`__dict__`访问，这个dict维护了这个实例的所有属性。当你在一个class中定义了`__slots__`后所有class的实例中只有slots中定义的attributes会出现在class的实例中。当你实例化多个对象时可以节省内存，当然副作用就是你的实例对象不能动态添加class里slots定义之外的属性。
    
```python
class User(object):
    __slots__ =("name","age")
    def __init__(self,name,age):
        self.name = name
        self.age = age
u = User("dog",22)
hasattr(u,"__dict__") #return false
u.high = 1.75 #给不存在在slots里面的属性赋值会报错
```

## 4. `__getatrribute__` 和 `__getattr__`区别

*   `__getatrribute__`用来查询任意属性，包括存在和不存在的，`__getattr__`只有查询不存在的属性的时候才会触发，重写这个方法可以用于属性访问异常处理。

## 5. 字符串

### 字符串的连接

*   这个几乎上应该是入门者必须了解的特性了:因为字符串在内存里面**不可变**，所以如果用+号连接两个字符串时，将会进行各种内存调度工作，所以连接多个字符串先将其放进序列里面，然后用"".join(seq)的方法，其中 seq 代表字符串序列。
    
```python
In[4]: temp_str ="".join( list("wo" "he" "ni"))
In[5]: temp_str
Out[5]: 'woheni'
#实际上join()方法是把前面的字符当做连接符来连接序列里面的字符串
In[6]: temp_str =";".join( list("wo" "he" "ni"))
In[7]: temp_str
Out[7]: 'w;o;h;e;n;i'
```

### 字符串的反转

    > 是一个技巧性的方法
    > a为字符串，a[::-1]用切片可以达到reverse的效果

## 6. 各种判断和比较

*   永远不要用==或者!=来比较单件, 比如 None. 使用 is 或者 is not
*   一行判断：return_value = True if a == 1 else False 和 c 里面的 a?b:c 一样
*   all([a, b, c])和a and b and c 效果是一样的

## 7. 头文字

*   通常认为用 #!/usr/bin/env python 要比 #!/usr/bin/python 更好，因为 python 解释器有时并不安装在默认路径，例如在 virtualenv 中，如果你用 python xxoo.py 来运行，那么写不写都没关系，如果要用 ./xxoo.py 那么就必须加这行，这行被称为 shebang, 用来为脚本语言指定解释器。

## 8. 字典

### 字典的创建可以有以下这几种方式

```python
In[13]: dict(a=1,b=2)
Out[13]: {'a': 1, 'b': 2}
In[14]: {"a":1,"b":2}
Out[14]: {'a': 1, 'b': 2}
In[15]: dict([("a",1),("b",2)])
Out[15]: {'a': 1, 'b': 2}
In[16]: names
Out[16]: ('a', 'b')
In[17]: dict(zip(names,range(2)))
Out[17]: {'a': 0, 'b': 1}
#用zip()来解析两个表达式，当然也可以用列表推导式来代替
```

### 字典取值

*   判断一个 key 是否在一个 dict 里面，用 has_key(key,default=none),其中第二参数可以自己定义当为空时的返回值

### 字典比较差异

*   用视图来判断两个字典的差异，即viewitems,具体用法如下:
    在 Python 2.x 里面，iteritems() 和 viewitems() 这两个方法都已经废除了，而 item() 得到的结果是和 2.x 里面 viewitems() 一致的。以下代码在python3.4中测试。

```python
n[18]: d = dict(a=1,b=2,c=4)
In[19]: d2 = dict(a=1,b=2,c=3)
In[20]: d.items()
Out[20]: dict_items([('b', 2), ('c', 4), ('a', 1)])
In[21]: v1 = d.items()
In[22]: v2 = d2.items()
In[23]: v1 | v2 #求并集
Out[23]: {('a', 1), ('b', 2), ('c', 3), ('c', 4)}
In[24]: v1 & v2 #求交集
Out[24]: {('a', 1), ('b', 2)}
#有意思的一点是视图会随着原来的字典的变化而变化
In[25]: d.update(e=8)
In[26]: d
Out[26]: {'a': 1, 'b': 2, 'c': 4, 'e': 8}
In[27]: v1
Out[27]: dict_items([('e', 8), ('b', 2), ('c', 4), ('a', 1)])
```

## 9. sorted方法和使用

1.  sorted(a, key=lambda result: result[0],reverse=True) ，这个排序居然跟 lisp 里面很像，基本抄过来的。在 key 里面定义一个匿名函数，在函数里面自定排序规则，因为输入参数默认为 a（sorted 的第一个参数），所以使用匿名函数时可以使用 map 等提取单独元素，完整方法以及参数 sorted(data, cmp=None, key=None, reverse=False)
2.  operator.itemgetter函数获取的不是值，而是定义了一个函数，通过该函数作用到对象上才能获取值。所以sorted(aa.items(),key=itemgetter(1)) 是对字典 aa 的value值进行排序，使用之前 from operator import itemgetter

## 10. yield from 
    yield from 更像是一种约定，把异步任务以协程发出去，然后等待结果返回，可以在结果返回上下文做文章。至于怎么知道结果到了，就需要事件循环来调度，某种任务协议来驱动。
ps：更像执行体和调用方直接建立了管道。

## 11. 可变对象copy
`b = range(9);a=[];a[:]=b等同于a=copy.decopy(b)`

## 12. 将某个包全局化
(不建议这么做)
`import site;site.getsitepackages()`可以看到安装系统package的位置，`.pth`结尾的文件可以将本地python包的绝对位置暴露给全局，把`.pth`文件放到 `site.getsitepackages`所返回的路径当中即可。`sys.path`返回的是`PYTHONPATH`变量指向的位置。

## 13. 关于`__import__` 
`__import__`方法其实就是import的原型，只不过前者接收的是字符串，而且在导入package.module，返回的是package而不是module，建议用importlib中的import_module替代。`sys.modules`是一个包含了当前解释器的所有导入模块的字典。

## 14. pkgutil
 如果要获取包里面的所有模块列表，不应该用 os.listdir()，而是 pkgutil 模块。

## 15. 关于闭包
至少有两种主要方式来捕获和保存状态信息,你可以在一个对象实例 (通过一个绑定方法) 或者在一个闭包中保存它。

