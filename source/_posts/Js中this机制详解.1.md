---
title: Js中this机制详解
categories: javascript
# tags: [javascript]
---

**this机制的四种规则**

this到底绑定或者引用的是哪个对象环境决定于函数被调用的地方。而函数的调用有不同的方式，在不同的方式中调用决定this引用的是哪个对象是由四种规则确定的。我们一个个来看。

**默认绑定全局变量**

这条规则是最常见的，也是默认的。当函数被单独定义和调用的时候，应用的规则就是绑定全局变量。如下：



```javascript
function fn() {
    console.log( this.a );
}
var a = 2;
fn(); // 2 -- fn单独调用，this引用window
```

**隐式绑定**

隐式调用的意思是，函数调用时拥有一个上下文对象，就好像这个函数是属于该对象的一样。例如：



```javascript
function fn() {
    console.log( this.a );
}
var obj = {
    a: 2,
    fn: fn
};
obj.fn(); // 2 -- this引用obj。
```

需要说明的一点是，最后一个调用该函数的对象是传到函数的上下文对象（绕懵了）。如：



```javascript
 function fn() {
     console.log( this.a );
 }
 var obj2 = {
     a: 42,
     fn: fn
 };
 var obj1 = {
     a: 2,
     obj2: obj2
 };
 obj1.obj2.fn(); // 42 -- this引用的是obj2.
```

 

还有一点要说明的是，失去隐式绑定的情况，如下：



```javascript
 function fn() {
     console.log( this.a );
 }
 var obj = {
     a: 2,
     fn: fn
 };
 var bar = obj.fn; // 函数引用传递
 var a = "全局"; // 定义全局变量
 bar(); // "全局"
```

 

如上，第8行虽然有隐式绑定，但是它执行的效果明显是把fn赋给bar。这样bar执行的时候，依然是默认绑定全局变量，所以输出结果如上。

**显示绑定**

学过bind()\apply()\call()函数的都应该知道，它接收的第一个参数即是上下文对象并将其赋给[this](http://www.w3cmark.com/)。看下面的例子：



```javascript
 function fn() {
     console.log( this.a );
 }
 var obj = {
     a: 2
 };
 fn.call( obj ); // 2
```

 

如果我们传递第一个值为简单值，那么后台会自动转换为对应的封装对象。如果传递为null，那么结果就是在绑定默认全局变量，如：



```javascript
 function fn() {
      console.log( this.a );
  }
  var obj = {
      a: 2
  };
 var a = 10;
 fn.call( null); // 10
```

 

**new新对象绑定**

如果是一个构造函数，那么用new来调用，那么绑定的将是新创建的对象。如：



```javascript
function fn(a) {
    this.a = a;
}
var bar = new fn( 2 );
console.log( bar.a );// 2
```

 

注意，一般构造函数名首字母大写，这里没有大写的原因是想提醒读者，构造函数也是一般的[函数](http://www.w3cmark.com/)而已。

上面介绍的四种关于this绑定的4中情况和规则，现实写代码的过程中肯定比这要多和复杂，但是无论多复杂多乱，它们都是混合应用上面的几个规则和情况而已。