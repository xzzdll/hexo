---
title: 浏览器中Dom和Bom的区别
categories: javascript
# tags: [javascript]
---

## 区别

### BOM（Browser Object Model）

> BOM 即浏览器对象模型，BOM没有相关标准，**BOM的最核心对象是window对象**。window对象既为javascript访问浏览器提供API，同时在ECMAScript中充当Global对象。BOM和浏览器关系密切，浏览器很多东西可以通过javascript控制，例如打开窗口、打开选项卡、关闭页面、收藏夹等。这些功能与网页内容无关。由于没有标准，不同的浏览器实现同一功能，可以通过不同的实现方式。例如，加入收藏夹这个功能：

> 

```
IE浏览器: window.external.AddFavorite(url,title);

FireFox浏览器: window.sidebar.addPanel(title, url, "");
```

> 虽然没有统一标准，但是各个浏览器的常用功能的js代码大同小异，对于常用的功能已经有默认的标准了。

### DOM（Document Object Model）

> DOM即文档对象模型，DOM是W3C标准，**DOM的最根本对象是document（window.document）**，这个对象实际上是window对象的属性，这个对象的独特之处是这个是唯一一个既属于BOM又属于DOM的对象。DOM和文档有关，这里的文档指的是网页，也就是html文档。DOM和浏览器无关，他关注的是网页本身的内容，由于和浏览器没有多大的关系，所以标准就好定了。

## BOM与DOM的联系

![img](http://upload-images.jianshu.io/upload_images/7166236-ac9c88e8fc0cb3c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/842/format/webp)

image.png

![img](http://upload-images.jianshu.io/upload_images/7166236-8f4701a13ad06f23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/902/format/webp)