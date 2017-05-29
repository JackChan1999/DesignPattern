---
typora-copy-images-to: img
---
## 1. MVC设计模式
### 1.1 概述

模型-视图-控制器，MVC设计模式的目的是将数据模型和视图分离开来，并以控制器作为连接两者的桥梁以实现解耦

![mvc](img/mvc.png)

| 意义   | 说明                                       |
| :--- | :--------------------------------------- |
| M    | Model，表示模型层，数据模型或业务模型，就是我们要显示给用户查看的内容    |
| V    | View，表示视图层，就是用户直接看到的界面，例如：Activity，Fragment，自定义View，还有XML布局文件 |
| C    | Controler，表示控制层，用于模型和视图间切换数据，对应于Activity业务逻辑，数据处理和UI处理 |

而J2EE中的MVC分别表示：

- M：javabean
- V：jsp
- C：servlet

三角关系的问题就是维护问题。在MVC，当你有变化的时候你需要同时维护三个对象和三个交互，这显然让事情复杂化了

Android中经常会出现数千行的Activity代码，究其原因，Android中纯粹作为View的各个XML视图功能太弱，Activity基本上都是View和Controller的合体，既要负责视图的显示又要加入控制逻辑，承担的功能过多，代码量大也就不足为奇。所以更贴切的目前常规的开发说应该是View-Model 模式，大部分都是通过Activity的协调，连接，和处理逻辑的

Activity中存在两部分内容：业务相关和界面相关，V中的内容较少，而C中的内容较多

解决方案：将Activity中的业务部分拆分----MVP，将Activity中的界面部分拆分----MVVM

### 1.2 案例分析
Android中的ListView就用到了MVC设计模式

- M：数据的集合，List&lt;UserInfo>
- V：ListView
- C：Adapter，控制数据如何显示在ListView上

而Activity控制层和视图层都有

数据模型，业务层，业务逻辑，数据层，表现层，控制层：事件路由