---
title: 异步加载script的几种方式
categories: javascript
# tags: [javascript]
---

**前言**

关于JavaScript脚本加载的问题，相信大家碰到很多。主要在几个点——

1> 同步脚本和异步脚本带来的文件加载、文件依赖及执行顺序问题
2> 同步脚本和异步脚本带来的性能优化问题

深入理解脚本加载相关的方方面面问题，不仅利于解决实际问题，更加利于对性能优化的把握并执行。

 

先看随便一个script标签代码——

```html
<script src="js/myApp.js"></script>
```

如果放在<head>上面，会阻塞所有页面渲染工作，使得用户在脚本加载完毕并执行完毕之前一直处于“白屏死机”状态。而<body>末尾的打脚本只会让用户看到毫无生命力的静态页面，原本应该进行客户端渲染的地方却散布着不起作用的控件和空空如也的方框。拿一个测试用例——


```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>异步加载script</title>
    <script src="js/test.js"></script>
</head>
<body>
    <div>我是内容</div>
    <img src="img/test.jpg">
</body>
</html>
```



其中,test.js中的内容——

```
alert('我是head里面的脚本代码，执行这里的js之后，才开始进行body的内容渲染!');
```

我们会看到,alert是一个暂停点，此时，页面是空白的。但是要注意，此时整个页面已经加载完毕，如果body中包含某些src属性的标签(如上面的img标签)，此时浏览器已经开始加载相关内容了。总之要注意——js引擎和渲染引擎的工作时机是互斥的（一些书上叫它为UI线程）。

因此,我们需要——那些负责让页面更好看、更好用的脚本应该立即加载，而那些可以待会儿再加载的脚本稍后再加载。

 

**一、脚本延迟执行**

现在越来越流行把脚本放在页面<body>标签的尾部。这样，一方面用户可以更快地看到页面，另一方面脚本可以直接操作已经加载完成的dom元素。对于大多数脚本而言，这次“搬家”是个巨大的进步。该页面模型如下——


```html
<!DOCTYPE html>
<html>
<head lang="en">
    <!--metadata and scriptsheets go here-->
    <script src="headScript.js"></script>
</head>
<body>
    <!--content goes here-->
    <script src="bodyScript.js"></script>
</body>
</html>
```



这确实大大加快了页面的渲染时间，但是注意一点，这可能让用户有机会在加载bodyScript之前与页面交互。源于浏览器在加载完整个文档之前无法加载这些脚本，这对那些通过慢速连接传送的大型文档来说会是一大瓶颈。

理想情况下，脚本的加载应该与文档的加载同时进行，并且不影响DOM的渲染。这样，一旦文档就绪就可以运行脚本，因为已经按照script标签的次序加载了相应脚本。

我们使用defer便能够完成这样的需求，即——

```html
<script src="deferredScript.js"></script>
```

添加defer属性相当于告诉浏览器：请马上开始加载这个脚本吧，但是，请等到文档就绪且此前所有具有defer属性的脚本都结束运行之后再运行它。

这样，在head标签里放入延迟脚本，技能带来脚本置于body标签时的所有好处，又能让大文档的加载速度大幅提升。此时的页面模式便是——


```html
<!DOCTYPE html>
<html>
<head lang="en">
    <!--metadata and scriptsheets go here-->
    <script src="headScript.js"></script>
    <script src="deferredScript.js" defer></script>
</head>
<body>
    <!--content goes here-->
</body>
</html>
```



但是并非所有的浏览器都支持defer（对于一些modern浏览器，如果声明defer，其内部脚本将不会执行document.write及DOM渲染操作。IE4+均支持defer属性）。这意味着，如果想确保自己的延迟脚本能在文档加载后运行，就必须将所有延迟脚本的代码都封装在诸如jQuery之$(document).ready之类的结构中。这是值得的，因为差不多97%的访客都能享受到并行加载的好处，同时另外3%的访客仍然能使用功能完整的JavaScript。

 

**二、脚本的完全并行化**

让脚本的加载及执行再快一步，我不想等到defer脚本一个接着一个运行（defer让我们想到一种静静等待文档加载的有序排队场景），更不想等到文档就绪之后才运行这些脚本，我想要尽快加载并且尽快运行这些脚本。这里也就想到了HTML5的async属性，但是要注意，它是一种混乱的无政府状态。

例如，我们加载两个完全不相干的第三方脚本，页面没有它们也运行得很好，而且也不在乎它们谁先运行谁后运行。因此，对这些第三方脚本使用async属性，相当于一分钱没花就提升了它们的运行速度。

async属性是HTML5新增的。作用和defer类似，即允许在下载脚本的同时进行DOM的渲染。但是它将在下载后尽快执行（即JS引擎空闲了立马执行），不能保证脚本会按顺序执行。它们将在onload 事件之前完成。 

Firefox 3.6、Opera 10.5、IE 9 和 最新的Chrome 和 Safari 都支持 async 属性。可以同时使用 async 和 defer，这样IE 4之后的所有 IE 都支持异步加载，但是要注意，async会覆盖掉defer。

那么此时的页面模型如下——


```html
<!DOCTYPE html>
<html>
<head lang="en">
    <!--metadata and scriptsheets go here-->
    <script src="headScript.js"></script>
    <script src="deferredScript.js" defer></script>
</head>
<body>
    <!--content goes here-->
    <script src="asyncScript1.js" async defer></script>
    <script src="asyncScript2.js" async defer></script>
</body>
</html>
```


要注意这里的执行顺序——各个脚本文件加载，接着执行headScript.js，紧接着在DOM渲染的同时会在后台加载defferedScript.js。接着在DOM渲染结束时将运行defferedScript.js和那两个异步脚本，要注意对于支持async属性的浏览器而言，这两个脚本将做无序运行。

 

**三、可编程的脚本加载**

尽管上面两个脚本属性的功能非常吸引人，但是由于兼容性的问题，应用并不是很广泛。故此，我们更多使用脚本加载其他脚本。例如，我们只想给那些满足一定条件的用户加载某个脚本，也就是经常提到的“懒加载”。

在浏览器API层面，有两种合理的方法来抓取并运行服务器脚本——

1> 生成ajax请求并用eval函数处理响应

2> 向DOM插入script标签

后一种方式更好，因为浏览器会替我们操心生成HTTP请求这样的事。再者，eval也有一些实际问题：泄露作用域，调试搞得一团糟，而且还可能降低性能。因此，想要加载名为feture.js的脚本，我们应该使用类似下面的代码：

```javascript
var head = document.getElementsByTagName('head')[0];
var script = document.createElement('script');
script.src = 'feature.js';
head.appendChild(script);
```

当然，我们要处理回调监听，HTML5规范定义了一个可以绑定回调的onload属性。

```javascript
script.onload = function() {
    console.log('script loaded ...');
}
```

不过，IE8及更老的版本并不支持onload，它们支持的是onreadystatechange。而且，对于错误处理仍然千奇百怪。在这里，可以多参考一些流行的校本加载库，如labjs、yepnope、requirejs等。

如下，自己封装了一个简易loadjs文件——


```javascript
var loadJS = function(url,callback){
    var head = document.getElementsByTagName('head')[0];
    var script = document.createElement('script');
    script.src = url;
    script.type = "text/javascript";
    head.appendChild( script);

    // script 标签，IE下有onreadystatechange事件, w3c标准有onload事件
    // IE9+也支持 W3C标准的onload
    var ua = navigator.userAgent,
        ua_version;
    // IE6/7/8
    if (/MSIE ([^;]+)/.test(ua)) {
        ua_version = parseFloat(RegExp["$1"], 10);
        if (ua_version <= 8) {
            script.onreadystatechange = function(){
                if (this.readyState == "loaded" ){
                    callback();
                }
            }
        } else {
            script.onload = function(){
                callback();
            };
        }
    } else {
        script.onload = function(){
            callback();
        };
    }
};
```



对于document.write的方式异步加载脚本，在这里就不说了，现在很少有人这么干了，因为浏览器差异性实在是搞得头大。

要注意，使用 Image 对象异步预加载 js 文件，里面的js代码将不会被执行。

最后，谈一下requirejs中的异步加载脚本。

requirejs不会保证按顺序运行目标脚本，只是保证它们的运行次序能满足各自的依赖性要求。从而我们确保了尽快的并行加载所有脚本，并有条不紊的按照依赖性拓扑结构去执行这些脚本。

 

**四、总结**

OK，谈到这儿，异步加载脚本的陈述也就完了。我再次啰嗦一下这里的优化顺序——

1> 传统的方式，我们使用script标签直接嵌入到html文档中，这里分两种情况——

　　a> 嵌入到head标签中——要注意，这样做并不会影响文档内容中其他静态资源文件的并行加载，它影响的是，文档内容的渲染，即此时的DOM渲染就会被阻塞，呈现白屏。

　　b> 嵌入到body标签底部——为了免去白屏现象，我们优先进行DOM的渲染，再去执行脚本，但问题又来了。先说第一个问题——如果DOM文档内容比较大，交互事件绑定便有了延迟，体验便差了些。当然，我们需要根据需求而定，让重要的脚本优先执行。再说第二个问题——由于脚本文件至于body底部，导致对于这些脚本的加载相对于至于head中的脚本而言，它们的加载便有了延迟。所以，至于body底部，也并非是优化的终点。

　　c> 添加defer属性——我们希望脚本尽早的进行并行加载，我们把这批脚本依旧放入head中。脚本的加载应该与文档的加载同时进行，并且不影响DOM的渲染。这样，一旦文档就绪就可以运行脚本。所以便有了defer这样属性。但是要注意它的兼容性，对于不支持defer属性的浏览器，我们需要将代码封装在诸如jQuery之$(document).ready中。需要注意一点，所有的defer属性的脚本，是按照其出场顺序依次执行，因此，它同样严格同步。

 2> 上一点，讲的都是同步执行脚本（要注意，这些脚本的加载过程是并行的，只不过是谁先触发请求谁后触发请求的区别而已），接下来的优化点便是“并行执行脚本”，当然，我们知道，一个时间点，只有执行一个js文件，这里的“并行”是指，谁先加载完了，只要此时js引擎空闲，立马执行之。这里的优化分成两种——

　　a> 添加async这个属性——确实能够完成上面我们所说的优化点，但是它有很高的局限性，即仅仅是针对非依赖性脚本加载，最恰当的例子便是引入多个第三方脚本了。还有就是与deffer属性的合用，实在是让人大费脑筋。当然，它也存在兼容性问题。以上三个问题便导致其应用并不广泛。当使用async的时候，一定要严格注意依赖性问题。

　　b> 脚本加载脚本——很显然，我们使用之来达到“并行执行脚本”的目的。同时，我们也方便去控制脚本依赖的问题，我们便使用了如requirejs中对于js异步加载的智能化加载管理。

好，写到这儿。