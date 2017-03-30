# ArcGIS API for JavaScript 3.x 4.x 对比分析
## 概述
目前，`ArcGIS API for JavaScript` 同时存在着`3.x`和`4.x`两个版本。

其中`3.x`版本可以看做是Esri传统的Web API的代表，自从`ArcGIS Server 9.3`推出REST API以来，`ArcGIS JavaScript API` 就作为`ArcGIS Server`前端的counterpart一直存在，虽然那个时候JS API 无论从功能还是性能上来说，都远不如基于插件版本的Flex API 和 Silverlight API强大。时过境迁，伴随着Web前端技术的飞速进步，ArcGIS 也紧跟业界步伐，不断迭代更新JS API的功能。发展到现在最新的3.20版本（截止到这篇文章的时间），JS API 二维Web地图的功能已经相当成熟稳定并且功能完备了。

随着Web前端标准的不断演进规范，基于插件的RIA技术如Flex、Silverlight已成明日黄花，而原生Web标准正在不断的成熟、规范：**HTML5**、**CSS3**、**ES6**、**WebGL**、**Web Assembly**。这些新技术规范的出现，不仅极大的提高了前端开发的生产效率；前端代码的运行效率。更为我们带来了许多新的功能。**WebGL**就是其中的典型代表，它提供了一套在浏览器中渲染三维图形而不依赖于插件的API，可以充分的利用浏览器和GPU的渲染能力。

WebGL的成熟与规范，也为前端实现三维Web地图提供了技术支撑。`ArcGIS API for JavaScript 4.0`就是在这样的技术背景下诞生的。`4.x` API要解决的核心问题就是Web GIS 3D的功能实现，而借此契机，Esri也吸取了之前版本ArcGIS JavaScript API开发过程中的一些经验教训，彻底重写了API，实现了地图逻辑和渲染逻辑的解耦，摆脱了之前API的一些历史包袱，为将来二三维地图功能的整合发展以及功能的完善打下了坚实的基础。就目前的客观事实来说，`4.x`在二维地图方面的功能还没有`3.x`版本完备、稳定，所以对于一些历史项目，Esri的推荐是继续使用`3.x`版本的API，而对于新的项目并且有三维需求，可以尝试`4.x`版本的API。目前，这两个版本Esri目前都在积极更新维护。`4.x`的最终目标是取代`3.x`版本的API，提供一个完整一致的二三维地图开发体验。


## 对比分析
### 数据和渲染的分离

`3.x`版本的API，是紧紧围绕二维地图的核心类：[Map](!https://developers.arcgis.com/javascript/3/jsapi/map-amd.html) 来进行展开的。`Map`这个巨无霸类，既是前端二维地图数据模型的承载者，也包含了地图的渲染逻辑。而在`4.x`版本，地图核心得以解耦，![](https://ooo.0o0.ooo/2017/03/26/58d78e95985d6.jpg)
在`4.x`版本中[Map](https://developers.arcgis.com/javascript/latest/api-reference/esri-Map.html)类仅仅代表地图数据逻辑的抽象，而二维地图和三维地图，分别有两个不同的类来支持Map对象的渲染实现：[MapView](https://developers.arcgis.com/javascript/latest/api-reference/esri-views-MapView.html)、[SceneView](https://developers.arcgis.com/javascript/latest/api-reference/esri-views-SceneView.html)。通过这样架构上的模型与渲染的分离，我们可以轻松的写出[二三维联动](https://developers.arcgis.com/javascript/latest/sample-code/views-synchronize/index.html)、多屏联动的地图应用。

### 二维地图实现的区别
`3.x`版本的API，由于历史原因，地图渲染一直使用的是DOM元素直接渲染，而DOM元素的频繁修改是对前端性能有极大影响的。以最常见的的缓存地图为例，缓存底图实际上是由`<img>`元素引用缓存切片拼接而来：
![](https://ooo.0o0.ooo/2017/03/26/58d7968092e10.jpg)
这样当用户操作地图进行平移缩放的时候，浏览器会产生大量不必要的DOM重排、重绘。

而在`4.x`API中，二维地图的渲染实现改为了Canvas API，如图：
![](https://ooo.0o0.ooo/2017/03/26/58d79859a0560.jpg)
使用Canvas API不仅极大的提高了前端地图的显示性能，并且还轻松解决了一直困扰`3.x`版本的地图旋转功能缺失的问题，现在，在`4.x` API的二维地图中，只需按住鼠标右键，就可以轻松旋转地图。

### 更加人性化的API

#### 读写属性的变化

传统`3.x` 版本的API，对象属性的方法一般是通过`SetXXX` 和 `GetXXX` 来进行读写。例如[map.getLevel()](https://developers.arcgis.com/javascript/3/jsapi/map-amd.html#getlevel)，[map.setLevel()](https://developers.arcgis.com/javascript/3/jsapi/map-amd.html#setlevel)。而在`4.x`版本中，对象属性的读写被统一的定义为更简洁自然的方式：`map.basemap = 'streets'`。 在新版本的API中，所有属性的读写都可以直接通过赋值的方式赋予新的值并且触发新属性的功能。例如，上面的语句可以直接切换map的底图为`streets`。这在背后使用到了ES5的新特性[Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)，所有的getter 和setter都可以触发相应的事件，来进行程序状态的更新。

#### 更加统一的构造函数与autocast

在`3.x`版本的API中，类的构造函数可以说是五花八门，在`4.x`版本中，构造函数的形式被统一为只接收一个对象参数，且可以在构造函数中设置类的所有属性，例如：
```javascript
view = new MapView({
  center: [-122, 38],
  scale: 12345678
});
```
MapView的构造函数只接收一个参数，而通过参数对象中的属性，可以设置对应`MapView`对象上的`center`和`scale`属性。而得益于构造函数的统一，`4.x`版本还有一个非常实用的语法糖**autocast**：即对于需要设置的属性，我们可以直接传入所需对象的构造函数参数，而不需要显式的传入直接对象，以上面的代码为例，`MapView`的`center`属性，实际上是一个`esri/geometry/Point`对象，而在我们的代码中，不需要显式的引入模块：
```javascript
require(["esri/geometry/Point"], function(Point) { /* code goes here */ });
```
而是直接传入`esri/geometry/Point`的构造函数参数`[-122,38]`即可。

#### 模块名称大小写统一以及简化
`3.x` 绝大部分的类名都是大写开头，但是有几个**特例**，`4.x`修复了这个问题，全部改成了大写开头，另外重命名了几个长的令人发指的类名。

| 3.x命名           | 4.x命名  |
|:-------------| :-----|
| esri/map | esri/Map |
| esri/graphic      |   esri/Graphic |
| esri/tasks/query      |    esri/tasks/Query |
|ArcGISTiledMapServiceLayer | TileLayer |
|  ArcGISDynamicMapServiceLayer |  MapImageLayer|

## 功能矩阵
关于`3.x`和`4.x`版本具体的功能对比，参考下表：

|Capability	|3.20	|4.3|
|:-------------| :-----|:----|
|3D	|不支持|	支持|
|2D	|支持	|部分支持|
|矢量切片	|支持	|支持|
|缓存切片	|支持	|支持|
|影像服务	|支持	|支持|
|动态地图服务	|支持	|支持|
|要素服务	|支持	|支持|
|Geometry Engine	|支持	|支持|
|Web Scene	|不支持|	支持
|Web Map	|支持	|部分支持|
|直接使用Portal里面的item	|不支持	|部分支持|
|编辑	|支持	|部分支持|
|时态数据	|支持|即将支持
|OGC 服务 (WMS, WMTS, WFS, KML)	|支持|	即将支持|
|打印	|支持	|支持|
|更多GIS控件 (分析, 路径, 量测)	|支持	|即将支持|

更详细的功能对比，参考以下[链接](https://developers.arcgis.com/javascript/latest/guide/functionality-matrix/index.html)