---
title: 把程序改为并行化放在spark运行的注意事项
date: 2017-02-07 19:38:30
categories: coding
tags: Spark
---

把普通的程序改成并行化放在spark上运行，把其中遇到的简单问题整理了一下。

<!--more-->

author: [@Huji][1]


## 读取文件

读取linux文件系统上的文件地址要写作如下：
```
myFile = pd.read_csv("file:///home/myroot/notebook/huji/advisor/Data/station.csv")
```
**注意**：这里pandas的read_csv不能读取hdfs上面的文件

sc读取hdfs的时候可以用
```
myFile = sc.textFile("hj/Data/station.csv")
```
或者
```
myFile = sc.textFile("hdfs://10.15.198.204:8020/user/root/hj/Data/station.csv")
```


----------


## map函数加入参数

我们都知道map可以把rdd中的每个元素进行一个转换操作，例如
```
lines = sc.textFile("data.txt")
lineLengths = lines.map(lambda s: len(s))
```
但是更多的时候我们还需要加入别的参数，可以有如下的方法予以解决[^stack1][^stack2]：

- 通过在flatMap中使用匿名函数来实现

```
json_data_rdd.flatMap(lambda j: processDataLine(j, arg1, arg2))
```
或者
```
f = lambda j: processDataLine(dataline, arg1, arg2)
json_data_rdd.flatMap(f)
```

- 还可以用下面这种方法生成processDataLine
```
def processDataLine(arg1, arg2):
    def _processDataLine(dataline):
        return ... # Do something with dataline, arg1, arg2
    return _processDataLine

json_data_rdd.flatMap(processDataLine(arg1, arg2))
```

- `toolz`库还提供了高效的`curry`装饰器:

```
from toolz.functoolz import curry

@curry
def processDataLine(arg1, arg2, dataline): 
    return ... # Do something with dataline, arg1, arg2

json_data_rdd.flatMap(processDataLine(arg1, arg2))
```

- 采用`functools.partial`的方法将参数传入函数中
```
def worker(V, element):
    element *= V.value
from functools import partial

def SomeMethod(sc):
    someValue = rand()
    V = sc.broadcast(someValue)
    A = sc.parallelize().map(partial(worker, V=V))
```


----------

## 并行化自定义对象

我自己是创建一个station的类，然后构建了一个包含数个station对象的list，对其进行parallelize，然后对这一rdd对象做map操作，这时候就会报错说python的damon中没有station这个属性。

出现这个问题的原因是我定义的station对象只是在driver，但是各个worker并不知道这一个对象的定义，因此我需要单独将其定义在一个文件中，然后确保它能够分布到各个worker上。

下面是一个简单的例子[^stack3]，这个例子当中，如果并行化的对象是内置的数据类型，如int之类的，是没有任何问题的，但如果涉及到自己定义的node对象，那么map操作（marker2那句）就会报错。

```
from pyspark import SparkContext 

sc = SparkContext("local","WordCountBySparkKeyword")

def func(x):
    if x==2:
        return [2, 3, 4]
    return [1]

rdd = sc.parallelize([2])
rdd = rdd.flatMap(func) # rdd.collect() now has [2, 3, 4]
rdd = rdd.flatMap(func) # rdd.collect() now has [2, 3, 4, 1, 1]

print rdd.collect() # gives expected output

# Class I'm defining
class node(object):
    def __init__(self, value):
        self.value = value

    # Representation, for printing node
    def __repr__(self):
        return self.value


def foo(x):
    if x.value==2:
        return [node(2), node(3), node(4)]
    return [node(1)]

rdd = sc.parallelize([node(2)])
rdd = rdd.flatMap(foo)  #marker 2

print rdd.collect() # rdd.collect should contain nodes with values [2, 3, 4, 1, 1]
```

详细的解决步骤为：
(1) 首先创建一个单独的模块`node.py`，其中包含`node`的定义
(2) 在主程序文件中引入node类
```
from node import node
```
(3) 确保该模块被分布至各个节点（其实这个不写这句话也行，但是在执行程序的命令里面要加上这个文件，后面还会提到）
```
sc.addPyFile("node.py")
```


----------
## 关于print输出的问题

原始单机跑Python文件的时候，我经常在函数里面会写一些print之类的输出语句来方便调试，但是在改为并行化的程序之后，只有master上运行的代码段的输出会显示出来，很多并行化的任务会在worker上运行，所以我们并不能看到输出，需要输出的话，可能要对rdd进行take操作，然后取出其中的数值进行输出。


----------
## 共享变量的问题

在map函数中往往会用到一些大的dataframe，那么到底是本地读取还是先读取好之后broadcast出去呢？目前我觉得还是选择broadcast比较好。

实测如果把文件放到每台机器上，然后在函数中每次读取的话，运行耗时是读取一次然后broadcast的两倍左右。

使用broadcast的时候，注意通过value取出其内容，如下面的例子：

```
stock = pd.read_csv("file:///advisor/Data/new_stock201301.csv", dtype={'netid':str})
# 将stock变量广播至各节点
broadcastedStock = sc.broadcast(stock)
origRdd = origRdd.map(lambda x: myFunc(x, broadcastedStock.value))
origRdd.collect()
# 手动释放
broadcastedStock.unpersist()
```
## spark-submit提交任务

可以参考如下语句提交任务
```
PYSPARK_PYTHON=python3 spark-submit --master=yarn-client --num-executors=7 --executor-cores=6 --driver-memory=2g --executor-memory=10g --files bikeUtil.py,weatherHoliday.py,stationFlow.py,predictor.py main.py
```

[^stack1]: http://stackoverflow.com/questions/26959221/pyspark-broadcast-variables-from-local-functions

[^stack2]: https://stackoverflow.com/questions/33019420/spark-rdd-mapping-with-extra-arguments/33020269#33020269

[^stack3]: http://stackoverflow.com/questions/32792271/flatmap-over-list-of-custom-objects-in-pyspark


  [1]: https://hujichn.github.io/