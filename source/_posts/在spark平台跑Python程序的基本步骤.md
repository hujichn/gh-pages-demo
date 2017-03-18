---
title: 在spark平台跑Python程序的基本步骤
date: 2016-04-10 16:35:09
categories: coding
tags: [Spark, Python]
---

实验室的spark平台装好之后，写了一篇简单的使用教程供大家参考，以k-means作为例子。

<!--more-->

author: [@Huji][1]

## 前期准备

 1. 下载并安装[Xshell][2]
 2. 利用Xshell连接至Master节点:
- 首先新建连接，输入名称，如Master，输入主机的ip地址,点击确定。
  ![xshell-connect][3]

- 建立好会话后点击连接
  ![xshell-connect2][4]

- 输入用户名：root，密码：nesciipc@509，连接成功后会看到如下界面。
  ![xshell-connected][5]


 3. 登录jupyter，地址为http://10.15.198.204:7777。点击进入你的文件夹或者创建一个新文件夹然后进入，可以点击upload按钮上传你自己的python文件和数据文件。
  ![upload-py][6]

这里我们上传一个kmeans.py，内容为：

```
from pyspark import SparkContext
from pyspark.mllib.clustering import KMeans, KMeansModel
from numpy import array
from math import sqrt

sc = SparkContext(appName="kmeans")

# Load and parse the data
data = sc.textFile("hj/kmeans_data.txt")
parsedData = data.map(lambda line: array([float(x) for x in line.split(' ')]))

# Build the model (cluster the data)
clusters = KMeans.train(parsedData, 2, maxIterations=10,
        runs=10, initializationMode="random")

# Evaluate clustering by computing Within Set Sum of Squared Errors
def error(point):
    i = clusters.predict(point)
    center = clusters.centers[i]
    print("(" + str(point[0]) + "," + str(point[1]) + "," + str(point[2]) + ")" + "blongs to cluster " + str(i+1))
    # print("Cluster Number:" + str(len(clusters.centers)))
    return sqrt(sum([x**2 for x in (point - center)]))

WSSSE = parsedData.map(lambda point: error(point)).reduce(lambda x, y: x + y)
print("Within Set Sum of Squared Error = " + str(WSSSE))

for mCenter in clusters.centers:
    print(mCenter)
# Save and load model
# clusters.save(sc, "myModelPath")
# sameModel = KMeansModel.load(sc, "myModelPath")
```
> 注意：直接上传py文件的时候要记得import SparkContext，还要自己创建sc。

再上传一个kmeans_data.txt

```
0.0 0.0 0.0
0.1 0.1 0.1
0.2 0.2 0.2
9.0 9.0 9.0
9.1 9.1 9.1
9.2 9.2 9.2
```

我们的**目标**是：

使用Kmeans的方法进行聚类，被聚类的对象是空间中的点，每个点的参数就是kmeans_data.txt中每一行数值组成的三维坐标。

## 程序运行

 1. 将数据文件复制到hdfs上
- 第一步输入cd /home/myroot/notebook/huji，这里的huji对应于你在jupyter创建的文件夹名称。
- ls显示这个目录下的文件，可以看到有一个我们上传的数据文件kmeans_data.txt
- 下一步在hdfs上创建一个目录，可以随便自己取名，这里取为hj
- 将kmeans_data.txt复制到hj文件夹下
- 用ls命令看一下hj文件夹下，已经有该数据了。
  ![file-prepare][7]

 2. 运行python程序，在程序文件所在的文件中，输入下面命令即可。

```
PYSPARK_PYTHON=python3 spark-submit kmeans.py
```

**注意**：懒得每次运行都输入的话，可以在当前目录下新建一个sh文件，例如`start_kmeans.sh`，把上面这句命令粘贴进去，以后到这个目录下输入以下命令即可运行代码。
```
./start_kmeans.sh
```


我们的程序文件`kmeans.py`放在huji文件夹中，所以在这个目录下直接输入就好。

> 注意：如果上面第一步复制数据文件完毕以后，不慎退出了shell，那么下次再打开需要点击左上角`文件`-`打开`，选择上次建立的连接，如`Master`，再次连接，连上之后再次输入`cd /home/myroot/notebook/huji`进入程序文件所在目录。

## 输出结果

得到的结果如下图所示，其中三个点被聚到了cluster2，三个点被聚到了cluster1，两个cluster的中心点分别是[0.1,0.1,0.1]和[9.1,9.1,9.1]。


![输出结果][8]

## 其他

1. 程序修改：
这个工作可以在jupyter里面完成，点击传上去的py文件，就能够打开文件内容的页面，直接编辑，然后`ctrl+s`保存即可。

 ![code-edit][9]

2. 查找文件：
    linux下查找开头为@的文件的命令是

    ```
    find / -maxdepth 1 -name "@*" 
    ```

    要是查找当前目录就把斜杠改为点


3. 要删除hdfs上的数据等hadoop操作可以看[hadoop常用命令][10]


  [1]: https://hujichn.github.io
  [2]: https://www.netsarang.com/xshell_download.html
  [3]: http://7xsoqo.com2.z0.glb.clouddn.com/spark/pic/tutorial/xshell-connect.png
  [4]: http://7xsoqo.com2.z0.glb.clouddn.com/spark/pic/tutorial/xshell-connect2.png
  [5]: http://7xsoqo.com2.z0.glb.clouddn.com/spark/pic/tutorial/xshell-connected.png
  [6]: http://7xsoqo.com2.z0.glb.clouddn.com/spark/pic/tutorial/upload-py.png
  [7]: http://7xsoqo.com2.z0.glb.clouddn.com/spark/pic/tutorial/file-prepare.PNG
  [8]: http://7xsoqo.com2.z0.glb.clouddn.com/spark/pic/tutorial/output.png
  [9]: http://7xsoqo.com2.z0.glb.clouddn.com/spark/pic/tutorial/code-edit.png
  [10]: http://blog.csdn.net/bell2008/article/details/9639833