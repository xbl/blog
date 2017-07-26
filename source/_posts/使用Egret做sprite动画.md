---
title: 使用Egret做sprite动画
date: 2016-05-12
tags: 
---
> egret 貌似没有想cocos那样很方便的sprite动画，官方也没有例子，找了很多文章都是使用flash制作完成再导入到项目，终于发现可以借助DragonBones来完成；

## 使用DragonBones

* 首先创建一个新的动画，将素材拖进资源里

![资源](/assets/images/sprite/sucai.png)

* 然后将图片拖至画布中，注意，一定要放在画布的左上角，此时时间轴上会多出一个层，我们可以在这个层的时间轴上点击右键添加关键帧（k帧），与flash的方法相近就不多讲了；![](/assets/images/sprite/donghua.png)


* 导出选择MovieClip，可以看到导出的文件包含一个json文件和一张合并好的图片，将这两个文件copy到咱们的项目中的`resource/assets`目录![](/assets/images/sprite/output.png)

​	注意：json文件如下，mc对应是动画的名字，在程序项目中也是需要用到的，当然也可以修改；

``` json
{
    "file": "texture.png", 
    "res": {
        "run_0": {
            "x": 1, 
            "y": 1, 
            "w": 200, 
            "h": 213
        }, 
        "run_12": {
            "x": 203, 
            "y": 1, 
            "w": 191, 
            "h": 212
        }
    }, 
    "mc": {
        "test": {
            "frameRate": 24, 
            "events": [ ], 
            "frames": [
                {
                    "y": -2, 
                    "res": "run_0", 
                    "duration": 12, 
                    "x": 1
                }, 
                {
                    "y": 0, 
                    "res": "run_12", 
                    "duration": 12, 
                    "x": 13
                }
            ], 
            "labels": [
                {
                    "end": 24, 
                    "name": "run", 
                    "frame": 1
                }
            ]
        }
    }
}
```

## 在项目中使用动画文件

* 刷新项目，修改`default.res.json`文件，这就不多说了，Egret提供了`MovieClip`的方法，看关键代码：
  
  ``` typescript
  this.mcFactory = new egret.MovieClipDataFactory(RES.getRes("wolfJson"),RES.getRes("texture"));
  // 获取mcName，当填写错误不会报错，这里的mcName就是json中mc对应的key值
  this.mc = new egret.MovieClip(this.mcFactory.generateMovieClipData("wolf"));
  // run是动画的名字，-1表示无限循环
  this.mc.gotoAndPlay("run", -1);
  this.addChild(this.mc);
  ```
  
  ​

