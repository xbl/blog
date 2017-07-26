---
title: Egret 使用Tiled地图
date: 2016-05-12
tags: 
---

> ​	与sprite动画一样，Tiled的资料非常少，通过这几天的研究大致能做一个45°的游戏，虽然支持Tiled但是作为第三方库来支持，使用起来还不是很方便；

## Tiled使用

​	目前Egret支持的【正常】、【45度】和【Hexagonal】，还不支持【45度交错】模式，如果要做45°的游戏经过我的经验应该使用【Hexagonal】模式；

​	创建的时候要设置的宽和高不要一样，这样才是45°的视图，配置如下：![](/assets/images/map/mapProp.png)

通常我们需要两层就可以满足对地图的基本制作：

1. 地图贴图层；
2. 碰撞层；

### 贴图层的使用

​	我们将素材都支持成菱形的，与`Tile Width` 和`Tile Height`一致，可以通过Texture Merger 帮助我们合并成一张图片，选择【地图】->【新图块】，这里的【块宽高】同样要与`TileWidth` 一致；

### 碰撞层

​	我们需要制作两张不同颜色的菱形，区分哪些地图是可以创建的，那些是不允许创建，与贴图的方法一致，唯一不同的是需要在【Custom Properties】中添加自定义属性来区分即可，贴完之后要将碰撞层透明的设置为0，这样就不会影响地图的外观；

**注意：素材要与地图文件放在同一个目录，否则地图导出时的路径会有问题；**

## 在项目中使用

​	我们先将导出的素材和`.tmx`文件Copy到项目resource目录

### 使用第三方库

下载git代码：

``` shell
git clone https://github.com/egret-labs/egret-game-library.git
```

将`tiled/libsrc` Copy 到项目外层，然后配置`egretProperties.json`

``` json
{
	"native": {
		"path_ignore": []
	},
	"publish": {
		"web": 0,
		"native": 1,
		"path": "bin-release"
	},
	"egret_version": "3.0.8",
	"modules": [
		{
			"name": "egret"
		},
		{
			"name": "game"
		},
		{
			"name": "tween"
		},
		{
			"name": "res"
		},
		{
			"name": "tiled",
			"path": "../libsrc"
		}
	]
}
```



代码实现：

``` typescript
var warp = this;
// 获取到地图数据，由于.tmx就是xml所以可以将文件放到preload中，类型改为text
var data: any = egret.XML.parse(RES.getRes("mapTmx"));
var mapAttr:MapAttributes = new MapAttributes(data.attributes);

var mapWidth: number = mapAttr.width * mapAttr.tilewidth,
    mapHeight: number = (mapAttr.height + 1) * mapAttr.tileheight / 2;
// 初始化地图
this.tmxTileMap = new tiled.TMXTilemap(mapWidth,mapHeight,data,this.url);
this._buidingContainer = new BuidingContainer(this.tmxTileMap);
// 渲染地图
this.tmxTileMap.render();
var grasslandLayer;
// 获取地图所有layer
this.tmxTileMap.getLayers().forEach(function(layer,i): void {
// 这里很诡异，不能使用value.name，需要使用_name才能获取到
switch(layer._name) {
  case "meta_collision":
  warp.collisionLayer = layer;
  break;
  case "grassland_layer":
  grasslandLayer = layer;
  break;
}
```

​	与官方的demo差不多，坑点在与地图渲染后获取每个layer的name并不会马上得到，这里是异步操作，但库并没有提供一个完成渲染的事件可供回调，所以只能通过内部方法`._name`来获取；

## 坑点

​	使用的坑点还是挺多的，简单整理了几点：

1. 地图图片不能使用官方提供的资源预加载方法；
2. Tiled库提供了`TMXHexagonalRenderer`用于像素坐标与网格坐标的互换，然而并不好用，计算出来的结果是错误的，无奈需要自己写算法；
3. 地图与`ScrollView`必须要包裹一层容器，否则地图会无限向下滚动；
4. 关于Egret的深度坑，与网页中的`z-index`不同，在容器只有一个子元素的时候，即便通过`.addChildAt(obj, 9999)`子元素所在的层仍然是1，因为在容器没有这么多深度的时候设置更高的深度无效，所以如果每添加一个节点都需要自己手动排序，好在这并不难；
5. 图像图层貌似在【Hexagonal】模式下不能正常使用；