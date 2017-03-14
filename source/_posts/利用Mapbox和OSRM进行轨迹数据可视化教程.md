---
title: 利用Mapbox和OSRM进行轨迹数据可视化教程
date: 2016-06-07 14:53:02
categories: research
tags:  [bike,visualization,Mapbox,OSRM, mysql]
---


在做与交通有关的项目时，往往要做一些轨迹数据的可视化，这里介绍了利用Mapbox平台加OSRM进行轨迹数据可视化的方法。首先需要利用OSRM生成轨迹数据，然后把轨迹数据或者说是路径存储至mysql数据库，最后提取相应数据传入Mapbox，完成可视化的工作。

<!--more-->

author: [@Huji][1]

*版权归作者所有，任何形式转载请联系作者。*

## 前言

最近在做自行车调度方面的项目，希望能够呈现现有的调度轨迹，用可视化的方法让我们能够对现有的调度有一个直观的感受。选择`Mapbox`这一平台主要是看到了一家叫做urbica的可视化公司做的一个[自行车trip的可视化系统](http://urbica.co/citibike/)，这个系统的实现参考了一篇国外的硕士论文Rebalancing Citi Bike——A geospatial analysis of bike share redistribution in New York City，论文内容倒是比较简单，关键还是在实现，下面先放两张图仰慕一下效果。

![mode 1][2]

![mode 2][3]

其实现主要利用了`Mapbox`和`OSRM`，P.S. 另外还采用了[D3.js](http://d3js.org/) 来绘制一些滑动条啊、图表啊之类的，这个暂时还没研究，恩，列入to do list。

### Mapbox

Mapbox国内13年就有报道过了，感兴趣的同学可以点下面的链接。
[创立3年没要投资人一分钱，没销售人员——个性定制地图网站Mapbox是如何服务900家付费客户，并养活30号员工的][4]

也可以直接点[这里][5]前往`Mapbox`的官方网站，可以看到很多很漂亮的应用。

`Mapbox`实际上是一个平台，提供免费创建并定制个性化地图的服务，可以根据用户需要定制不同风格的地图，在上面加载你的数据图层等等。而上面的这个项目实际使用了[Mapbox GL JS](https://www.mapbox.com/mapbox-gl-js/api/) API来进行地图方面的可视化。注意，[Mapbox GL JS](https://www.mapbox.com/mapbox-gl-js/api/)是Mapbox提供的一个库，让用户可以方便地将生成的地图嵌入到自己的web应用中。当然，也支持导入到其他地图方案，如ios和android。

### OSRM

可以看到第一张图里面显示了一些自行车和调度车的轨迹，然而原始数据中只有轨迹的起止点，也就是从哪个站点出发，哪个站点还车。

**那么，如何将这样的数据匹配到路网上呢？**

这就需要用到[Open Source Routing Machine (OSRM) ](http://project-osrm.org/)。这是一个开源的路线生成引擎，给定起止点后，能够在路网中生成一条两点之间的最短路径，其基于的路网数据来自于[OpenStreetMap](https://www.openstreetmap.org/#map=5/51.500/-0.100)。

## 利用OSRM生成轨迹数据

这一段我们具体来介绍怎么利用OSRM生成轨迹数据。

ORSM开放的api[^osrm]如下
```
http://{server}/{service}/{version}/{profile}/{coordinates}[.{format}]?option=value&option=value
```

- `server` : router.project-osrm.org
- `service` : 要使用的服务的名称，可选的服务如下，我们这里用的是route

| Service        | 描述   | 
| --------   | ------  |
| route     | 求给定坐标点之间的最短路径 |  
| nearest   | 返回距离给定坐标最近的street segment |  
| table   | 计算给定坐标的distance tables | 
| match   | 将给定坐标点匹配到路网上 |
| trip   | 计算给定坐标点的最短回路 |
| tile   | 返回包含调试信息的vector tiles |

  
- `version`：版本号，这里是v1
- `profile`：在这里设置了交通方式，可选参数为driving，walking，cycling
- `coordinates`：字符串格式 {longitude},{latitude};{longitude},{latitude}[;{longitude},{latitude} ...] 或者 polyline({polyline}).
- `format`: 这个参数是可选的，默认为json，目前也没有其他值可以设置
- `option`：
下面是一些general options，但不是一定要设置：

|Option |Values |Description|
|----|--------|---------|
|bearings |{bearing};{bearing}[;{bearing} ...]| Limits the search to segments with given bearing in degrees towards true north in clockwise direction.|
|radiuses |{radius};{radius}[;{radius} ...]|  Limits the search to given radius in meters.|
|hints  |{hint};{hint}[;{hint} ...] |Hint to derive position in street network.|

在求取路径的时候，还有一些其他的options：
|Option |Values |Description|
|----|--------|---------|
|alternatives|  true, false (default)|  是否返回多条可能的路径|
|steps| true, false (default)|  是否对每段路径都返回详细的步骤，比如一些左转右转的信息|
|annotations| true, false (default)|  是否对于路径上的每个坐标点都返回附加的metadata|
|geometries |polyline (default), geojson|返回路径的格式|
|overview |simplified (default), full, false| 得到整条路径的overview，full的话就是最全的路径，simplified是根据显示级别得到的简化数据，或者是干脆不返回|
|continue_straight  |default (default), true, false|  强制要求走直线，并且不允许掉头|

这里给出最基本的python示例代码

```
import requests

def calRoutes(start, stop):
    url = 'http://router.project-osrm.org/route/v1/driving/' + start + ';' + stop + '?geometries=geojson&overview=simplified&steps=false'
    response = requests.get(url)
    data = response.json()
    routes = data["routes"][0]
    return routes

start = '120.38609,30.29742'
stop = '120.32375,30.2905'
routes = calRoutes(start, stop)
print(routes)
```

输出的结果为：

```
{'duration': 463.1, 'distance': 7129.5, 'legs': [{'distance': 7129.5, 'duration': 463.1, 'summary': '', 'steps': []}], 'geometry': {'type': 'LineString', 'coordinates': [[120.386362, 30.297321], [120.382159, 30.288818], [120.379259, 30.289752], [120.377031, 30.289848], [120.347114, 30.289887], [120.336178, 30.291638], [120.328133, 30.295271], [120.325014, 30.290114], [120.323842, 30.29065]]}}
```

其实返回的data变量是一个json格式的数据，其中包含三个部分：
- **code**：表征是否成功的变量，成功的话其值为OK
- **waypoints**: 所有的路径点
- **routes**: 包含了路径的数组

因为我们api中没有设置返回多条路径，所以默认返回一条，但是要取出这条路径要访问`data["routes"][0]`。

路径包含的信息参见上面的输出结果，包含了路径的长度，耗时，和各个节点的信息。

## 将轨迹数据（路径）存入mysql数据库

轨迹数据生成好之后，就需要进行存储，否则每次都要重新再算太麻烦了。
mysql遵守OGC的OpenGIS Geometry Model，支持Point、LineString等多种数据对象，我们这里用到的是LineString。

### 创建可以存储轨迹这类空间数据的表

当前只有MyISAM引擎的数据表支持地理空间数据的存储，所以在创建数据表的时候必须进行声明。
```
CREATE TABLE driving_routes(
  stationA VARCHAR(16),
  stationB VARCHAR(16),
  duration VARCHAR(32),
  distance VARCHAR(32),
  routes LINESTRING
)ENGINE=MyISAM;
```

创建后的表结果如下图

![create_table][6]

### 插入一条数据

插入数据要用到ST_GeomFromText的函数，将数据转化为数据库内部的几何格式。

```
INSERT INTO `driving_routes` VALUES(
'3942',
'3218',
'239.3',
'3295.2',
ST_GeomFromText('LINESTRING(120.7205165 27.98760837, 120.719859 27.987895, 120.73371 27.987744)')
);
```

### 提取数据

提取数据则相反，
```
SELECT ST_AsText(routes) from `driving_routes`;
```

其实存为linestring类型之后还可以在sql中进行一些其他操作

例如需要查找一辆车在某一段时间内是否在一段区域内经过，用点来说明的话，就是一个空间坐标点在一个特定时间段内是否包含在一个特定的矩形区域内。`MBRWithin(g1,g2) `函数应该能达到这个功能:


```
SELECT AsText(pnt) FROM `gis` WHERE MBRWithin(pnt,GeomFromText('Polygon(1 1,0 30,30 30,30 0，1 )'))
```

## 利用Mapbox进行可视化

**请将鼠标移动到图片上并点击可看到大图！！！！**

### 注册并进入studio

首先当然是注册一个mapbox的账号，点击[注册链接][7]，在这个页面输入用户名、邮箱、密码进行注册。

![sign_up][8]

完成注册后登陆mapbox，就会进入到studio的界面，如下图所示。

![studio_first][9]

被打码的区域，上面的是用户名，下面这个`access token`是所谓的访问令牌，也就是说，这是每个账户独一无二的一个码，是标明身份的一串字符，在使用Mapbox的API和库函数都需要先提供这个token。

左边一栏中有Home, Styles, Tilesets, Datasets, Status, Classic等选项卡，其功能分别是：

- Home: 主界面，各种功能的入口
- Styles: 可视化的核心，控制了地图的各种参数以及数据以何种格式进行显示。在Style中可以对每个图层进行各种细节的编辑，功能非常强大
- Tilesets: 这是创建好的各种图层数据
- Datasets: 用户自己上传的数据
- Status: 在这里可以看到我们账户的一些统计信息，例如地图的访问次数(map views)等等。我们注册的一般是免费账户，如果访问次数，流量等等到达一定限制的话是需要收费的，收费情况如下图所示，免费账户有50,000次访问，如果只是科研上用下其实一般用不掉的啦

![price][10]

### 创建一个style

进入studio之后，下面就可以创建一个style。

首先，在home页面下，点击`Go to styles`的按钮或者直接单击左侧的`Styles`选卡。

![go_to_styles][11]

进入到Styles的页面如下图所示，可以看到6个自带的Style，可以直接编辑使用，但是推荐还是点击圆圈处的`New style`按钮，以这些自带的Style为基础创建自己的Style。

![new_style][12]

在弹出的窗口中，输入自己的Style的名字，例如Test，然后选择下面的某个自带的Style作为模板，单击create。

![create_style][13]

单击后就会进入到Style的编辑页面，如下图所示。

![style_first][14]

双击地图进行放大，我们找到要编辑的区域，例如杭州，然后对某个图层进行编辑。

![style_color][15]

这里以更改道路名称的颜色为例进行讲解，单击某条道路，就会显示出此处包含的图层，这里包含文字的图层是`road_label_large`，单击此图层或者在左侧找到这一图层，就可以点开此图层的编辑页面，要更改颜色就直接在`color`那一栏选择自己喜欢的颜色即可，地图上会实时的跟随编辑进行调整。

> **注意**：这里做的所有操作都会自动保存，想撤销上一步操作的话可以按下`crtl+Z`，为了熟悉studio，你可以先随便试试各种功能，比如字体大小，线型，透明度，icon，placement等等，地图上几乎一切元素都是可以根据需要调整的。点击左侧侧边栏中油漆桶状的Style图标，回到Style界面，能够看到自己刚才创建的`Test` Style，在Test右边的选项中点击`Revert to last publish`，即可恢复成上一次**pulish**的样子。

> ![revert][16]

> **因此，pulish等于说是手动保存的意思！！没事的话不需要publish，确定修改已经完成，并且没有问题的时候才点击编辑界面的左栏右上角的publish按钮进行发布！**

### 上传自己的数据

基本的调整熟悉了之后，下一步就可以上传数据啦！

Mapbox支持多种格式的数据，如GeoJson或者CSV等等，这里我们采用GeoJson为例。把刚才得到的轨迹数据处理成GeoJson的格式[^geojson]。下面给出一个比较短的GeoJson的文件示例，其中包含两条轨迹数据，属于LineString的类型：

```
{
  "features": [
    {
      "properties": {
        "prop0": "value0"
      },
      "id": "{feature_id}",
      "geometry": {
        "coordinates": [
          [
            120.382304,
            30.288768
          ],
          [
            120.379259,
            30.289752
          ],
          [
            120.377224,
            30.289839
          ],
          [
            120.377248,
            30.30479
          ]
        ],
        "type": "LineString"
      },
      "type": "Feature"
    },
    {
      "properties": {
        "prop0": "value0"
      },
      "id": "{feature_id}",
      "geometry": {
        "coordinates": [
          [
            120.344083,
            30.309248
          ],
          [
            120.344496,
            30.303979
          ],
          [
            120.343989,
            30.303754
          ],
          [
            120.333314,
            30.303028
          ]
        ],
        "type": "LineString"
      },
      "type": "Feature"
    }
  ],
  "type": "FeatureCollection"
}
```

将上述轨迹数据存为test_routes.geojson文件，然后进行上传。上传之前可以先在这个[检测网站][17]把内容粘贴进去进行检测，如果格式正确就能绘制出轨迹。

上传数据的步骤如下：

- 先在Dataset的页面点击`New dataset`的按钮，再选择`Upload`，点击`Select a file`，在弹出的窗口中选中相应的数据文件，上传后点击`create`创建新的数据集。

![select_dataset][18]

- 点完`create`会见到下图所示的页面，选择`View detials`或者`Start editing`都可以。`View detials`的页面上再点击`Edit`也可以进入到编辑页面。

![create_dataset][19]

- 进入编辑页面如下图所示，可以看到已经有两条轨迹数据显示了出来，点击左侧栏右上角的`Export`蓝色按钮可以将数据从`Dataset`导出为`Tileset`（其实在这个页面也可以自己添加点、线、图形等等，功能非常强大，当然现在我们可以先忽略之...）

![edit][20]

- 点完`Export`，会跳出一个提示框，框中可以选择是导出到新的Tileset还是更新一个已有的，默认是导出到新的，所以只要再点输入框后面的`Export`按钮即可，之后左下角会出现处理的提示框，等图中的`processing`变成`Succeeded a few seconds ago`就可以关掉提示框，这时数据的上传和图层的建立就完成啦！

![processing][21]

> 其实可以直接在tileset里面上传geojson文件，或者直接在style中add new layer中上传geojson文件，从dataset里面上传，然后一步步创建tileset，添加到style中，这是最基本最复杂的步骤。熟悉后可以采用更直接的方法。

>  贴一段网站上的说明：
> Datasets vs Tilesets
> Tilesets are fast light, and easy to style. Datasets are lossless, editable, and can't be styled. Think of Tilesets as the delivery format for the data in your Datasets.

### 加载数据，发布style

数据上传好了，下一步就是加载。

先回到Style页面，点击`Test`后面的`Edit`，回到刚才创建的`Test` Style里面。

![style_edit_again][22]

点击左侧上方的`+ New layer`按钮，再点击`No tileset`处，就可以选择已经有的一些tileset添加到该style中，这里我们选择刚才创建的`test_routes`，然后再点击`create layer`（选择`test_routes`之后左边会出现一些filter啊之类的选项，可以做一些设置，感兴趣的话可以再查文档，这里先略过，直接点create就好。）

![add_new_layer][23]

创建完毕会出现style编辑的页面，双击地图放大到杭州区域，找到我们绘制的两条轨迹，然后可以改变其颜色，还可以做其他设置。

![publish][24]

至此，数据的加载就完成了，单击左上角的`publish`按钮就即可完成该style的发布。

### 创建可视化web端

![success][25]

发布完成后的页面如上图所示，点击`Preview, develop & use`，可以看到使用的相关信息，如下图所示。可以将网页进行分享，可以将生成的style展示到web端，嵌入到Android或者ios的app中等等。其中，我们需要的两条信息是`Style URL`和`Access token`。

![develop][26]

另外还需要在style中取得map的经纬度信息和缩放倍数信息，如下图标注的部分。

![position][27]

新建html文件，插入以下内容，按照已经准备好的四条信息替换需要的部分：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset='utf-8' />
    <title></title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <script src='https://api.tiles.mapbox.com/mapbox-gl-js/v0.23.0/mapbox-gl.js'></script>
    <link href='https://api.tiles.mapbox.com/mapbox-gl-js/v0.23.0/mapbox-gl.css' rel='stylesheet' />
    <style>
        body { margin:0; padding:0; }
        #map { position:absolute; top:0; bottom:0; width:100%; }
    </style>
</head>
<body>

<div id='map'></div>
<script>
mapboxgl.accessToken = '您的access token';
var map = new mapboxgl.Map({
    container: 'map', // container id
    style: '您的style url', //hosted style id
    center: [120.353031, 30.294875], // starting position，可替换为您自己的地图中心坐标
    zoom: 12.6 // starting zoom，可替换为您自己的地图缩放倍数
});
</script>

</body>
</html>
```

双击html文件，就可以在浏览器中看到您的web端可视化结果啦。

![finish][28]

当然，本教程只是实现了显示了两条轨迹数据在地图上的一个web端，要想实现丰富生动的可视化效果需要更多的数据，并需要更复杂的技术知识：需要将要点线结合，添加icon，在数据的propoties里面添加相关属性，给不同的属性设置不同的style等等。关于Mapbox GL JS如何使用的更多的例子请参见[example][29]，对example中的功能进行复杂的组合就可以得到类似于开篇中的效果啦~ fighting！

[^osrm]: https://github.com/Project-OSRM/osrm-backend/blob/master/docs/http.md

[^geojson]: http://www.oschina.net/translate/geojson-spec?cmp


  [1]: https://hujichn.github.io/
  [2]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/1.gif
  [3]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/2.gif
  [4]: http://www.pingwest.com/demo/mapbox/
  [5]: https://www.mapbox.com/
  [6]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/create_table.png
  [7]: https://www.mapbox.com/studio/signup/
  [8]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/sign_up.png
  [9]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/studio_first.png
  [10]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/price.png
  [11]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/go_to_styles.png
  [12]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/new_style.png
  [13]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/create_style.png
  [14]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/style_first.png
  [15]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/style_color.png
  [16]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/revert.png
  [17]: http://geojson.io/#map=14/30.2991/120.3531
  [18]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/select_dataset.png
  [19]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/create_dataset.png
  [20]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/edit.png
  [21]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/processing.png
  [22]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/style_edit_again.png
  [23]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/add_new_layer.png
  [24]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/publish.png
  [25]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/success.png
  [26]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/develop.png
  [27]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/position.png
  [28]: http://7xsoqo.com1.z0.glb.clouddn.com/mapbox/finish.png
  [29]: https://www.mapbox.com/mapbox-gl-js/examples/