---
title: Tips on data analysis by python (advanced)
date: 2016-05-13 14:19:00
categories: coding
tags: Python
---

第一篇主要是一些比较基础的问题，接下来遇到了稍微复杂一些的问题，因此整理在这个advanced tips里面，以备不时之需 :)

主要内容包括一些pandas的常用操作，利用pandas处理日期中的假期，python的异常处理、函数定义、装饰器的简单介绍等等。

<!--more-->

author: [@Huji][1]

[toc]

### 1. 对于groupby的结果分别apply函数进行计算，类似reduce的感觉

```
# 输入df，也就是其中的每一个group
def myFunc(df)：
    p = df["p"].values
    a = df["a"].values 
    # 返回两个值的话要用Series来返回
    return pd.Series({"sum": a.sum(), "rmsle": np.sqrt(np.squre(np.log(p+1)-np.log(a+1)).mean())})

dfGroupby.apply(myFunc)
```

### 2. 把字符串表示的时间进行加减，得到新字符串
```
from pandas.tslib import Timestamp
from pandas.tseries.offsets import *
eventsDate = “2013-10-22”
eventsDate = Timestamp(eventsDate)
dayBackward = str((eventsDate - DateOffset(days=1)).date())
dayForward = str((eventsDate + DateOffset(days=1)).date())
```

### 3. python中的异常处理简介

有时候网络有点崩，比如要从某网页抓取数据，经常由于网络原因中就error，这个时候真是痛心疾首，加入异常处理的逻辑可以使得程序放心运行。:）

异常处理的基本流程如下：
``` Python
try:
    Normal execution block
except A:
    Exception A handle
except B:
    Exception B handle
except:
    Other exception handle
else:
    if no exception,get here
finally:
    print("finally")  
```

其中A, B这样的异常名称不清楚的话，可以先看一下[python中的基本异常类型][2]。

还是不知道Error的类型的话，可以暂时先写
```
except Exception as ex：
    print(type(ex))
    print(ex.args)
```

这样一般可以得到Error的名称。

### 4. 由字典建立一行的DataFrame

```
firstLine = pd.DataFrame([{"rent_count": 0, "netid": netid}], index=[startTime])
```

需要注意的是，内容和index都要记得以list的形式传输

### 5. 填充缺失值 fillna
```
r['Wind SpeedMPH'].fillna(0, inplace=True)
r['VisibilityMPH'].fillna(method='ffill', inplace=True)
r['Humidity'].fillna(method='bfill', inplace=True)
```

### 6. 求出假期的方法

目前找到了两种方法，我们以美国2013年的法定节假日为例写一下示例代码：

- 第一种是Pandas中的Holiday模块提供的功能：

```
from datetime import * 
from pandas.tseries.holiday import USFederalHolidayCalendar
cal = USFederalHolidayCalendar()
cal.holidays(datetime(2013, 1, 1), datetime(2013, 12, 31))
```

输出的格式是：

![holidayUSA2013][3]

- 后一种应用范围更广，可以支持很多国家，对于US来说甚至细到州的假期，所以还是比较好用的。需要install workalendar这个库，代码如下：

```
from datetime import date
from workalendar.usa import NewYork
cal = NewYork()
cal.holidays(2013)
```
得到的结果是：

![holidayNYC2013][4]

### 7. 去掉列表中重复元素的方法

- 一维的list可以利用集合的方法,但会改变元素的顺序：
```
l1 = ['b','c','d','b','c','a','a']
l2 = list(set(l1))
print(l2)
```
要想不改变顺序可以用
```
l1 = ['b','c','d','b','c','a','a']
l2 = sorted(set(l1),key=l1.index)
print(l2)
```

- 如果不是一维的list可以用以下方法，或者遍历，原理是一样的。

```
a = [a[i] for i in range(len(a)) if a[i] not in a[:i]]
```

### 8. 统计嵌套列表里面各个list出现的次数

```
import pandas as pd

allList = [[1,2,3],[2,3,4],[1,2,3],[1,2,3]]
mDict = {'list':[],'num':[]}
print(mDict)

def calOccurrence(mList):
    if mList in mDict['list']:
        loc = mDict['list'].index(mList)
        mDict['num'][loc] = mDict['num'][loc] + 1
    else:
        mDict['list'].append(mList)
        mDict['num'].append(1)

for mList in allList:
    calOccurrence(mList)

print(pd.DataFrame(mDict))
```

输出结果是
```
        list  num
0  [1, 2, 3]    3
1  [2, 3, 4]    1
```

本来想利用pandas的groupby+count来做，但是groupby是求by的对象的hash值进行group，list对象不能求hash值也就不能groupby，所以还是用循环来维护一个字典，统计出各个list的出现次数。

### 9. Python函数定义[^pythondoc]

- 引入一个形如 **name 的参数时，它接收一个字典
- 引入一个形如 *name 的参数时，它接收一个元祖

对于如下代码：
```
def cheeseshop(kind, *arguments, **keywords):
    print("-- Do you have any", kind, "?")
    print("-- I'm sorry, we're all out of", kind)
    for arg in arguments:
        print(arg)
    print("-" * 40)
    keys = sorted(keywords.keys())
    for kw in keys:
        print(kw, ":", keywords[kw])
```

它可以像这样调用：
```
cheeseshop("Limburger", "It's very runny, sir.",
           "It's really very, VERY runny, sir.",
           shopkeeper="Michael Palin",
           client="John Cleese",
           sketch="Cheese Shop Sketch")
```

输出结果为：
```
-- Do you have any Limburger ?
-- I'm sorry, we're all out of Limburger
It's very runny, sir.
It's really very, VERY runny, sir.
----------------------------------------
client : John Cleese
shopkeeper : Michael Palin
sketch : Cheese Shop Sketch
```

### 10. python装饰器简单解析[^wrapper]

装饰器提供了一种方法，在函数和类定义的末尾插入自动运行的代码，相当于对函数进行一种扩展。

下面是一个简单地把日志输出到界面的例子
```
def logger(func):
    print("in decorator")
    def inner(*args, **kwargs):
        print("Arguments were: %s, %s" % (args, kwargs))
        return func(*args, **kwargs)
    return inner
```
请注意我们的函数inner，它能够接受任意数量和类型的参数并把它们传递给被包装的方法，这让我们能够用这个装饰器来装饰任何方法，例如：装饰下面的foo1函数。

```
@logger
def foo1(x, y=1):
     return x * y
print(foo1(5, 4))
orint(foo1(1))
```

输出结果如下：
```
in decorator
Arguments were: (5, 4), {}
20
Arguments were: (1,), {}
1
```

值得注意的是，第一次调用`foo1(5,4)`时，相当于
```
foo1 = logger(foo1)
foo1(5,4)
```

第二次调用`foo1(1)`时，foo1已经是logger(foo1)的返回值了。

所以只有第一次调用时输出了`in decorator`

在python的functools模块中，跟装饰器相关的有三个主要函数partial()，update_wrapper()和wraps()[^decorator]

**partial()**主要功能我感觉就是**把需要输入参数减少了**，例如本来把输入的数字转化为二进制的话需要`int('11',base=2)`，但是利用partial新建一个函数`int2 = partial(int, base=2)`之后，就可以直接调用`int2('11')`

**update_wrapper()**的功能是把被封装的函数的module，name，doc和dict复制到封装的函数中去。

其源码涉及到以下两个函数：

setattr()
: setattr()表示你可以通过该方法，给对象添加或者修改指定属性，setattr()接受三个参数：setattr(对象，属性，属性的值)

getattr()
: getattr用于返回一个对象属性，或者方法

**wraps()**函数相当于是用partial()把update_wrapper()又封装了一下，其实现如下。

```
def wraps(wrapped, assigned = WPAPPER_ASSIGNMENTS, updated = WRAPPER_UPDATES):
    return partial(update_wrapper, wrapped = wrapped, assigned = assigned, updated = updated)
```

利用这三个函数可以方便地实现装饰器，参考下面的代码，和一般装饰器类似，都是定义了一个自己的装饰器函数。

```
from functools import wraps
def my_decorator(f):
     @wraps(f)
     def wrapper(*args, **kwds):
         print('Calling decorated function')
         return f(*args, **kwds)
     return wrapper

@my_decorator
def example():
    '''这里是文档注释'''
    print('Called example function')

example()
print(example.__name__) # 'example'
print(example.__doc__) # '这里是文档注释'
```

其输出为

```
Calling decorated function
Called example function
example
这里是文档注释
```

值得注意的是@wraps(f)一句，它把被装饰的函数的name，doc等参数复制到装饰器内部的wrapper函数中去，所以最后两句print的输出是
```
example
这里是文档注释
```

否则输出为

```
wrapper
None
```



[^pythondoc]: http://www.pythondoc.com/pythontutorial3/controlflow.html#tut-defining

[^wrapper]: http://python.jobbole.com/81683/

[^decorator]: http://blog.jkey.lu/2013/03/15/python-decorator-and-functools-module/


  [1]: https://hujichn.github.io
  [2]: http://www.runoob.com/python/python-exceptions.html
  [3]: http://7xsoqo.com2.z0.glb.clouddn.com/python/tips/holidayUSA2013.png
  [4]: http://7xsoqo.com2.z0.glb.clouddn.com/python/tips/holidayNYC2013.png

