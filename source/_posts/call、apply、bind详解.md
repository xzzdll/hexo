---
title: call、apply、bind详解
categories: javascript
# tags: [javascript]
---
call、apply、bind的作用是改变函数运行时this的指向，所以先说清楚各种调用方式下的this指向。

### 三种this的指向

方法调用模式：当一个函数被保存为对象的一个方法时，如果调用表达式包含一个提取属性的动作，那么它就是被当做一个方法来调用，此时的this被绑定到这个对象。

```javascript
    var a = 1
    var obj1 = {
      a:2,
      fn:function(){
        console.log(this.a)
      }
    }
    obj1.fn()//2    
```

此时的this是指obj1这个对象，`obj1.fn()`实际上是`obj1.fn.call(obj1)`，事实上谁调用这个函数，this就是谁。补充一下，DOM对象绑定事件也属于方法调用模式，因此它绑定的this就是事件源DOM对象。如：

```javascript
document.addEventListener('click', function(e){
    console.log(this);
    setTimeout(function(){
        console.log(this);
    }, 200);
}, false);
```

点击页面，依次输出：document和window对象

解析：点击页面监听click事件属于方法调用，this指向事件源DOM对象，即`obj.fn.apply(obj)`，setTimeout内的函数属于回调函数，可以这么理解，`f1.call(null,f2)`，所以this指向window。



函数调用模式：就是普通函数的调用，此时的this被绑定到window

- 最普通的函数调用

```
function fn1(){
      console.log(this)//window
    }
fn1()
```

- 函数嵌套

```javascript
function fn1(){
    function fn2(){
        console.log(this)//window
    }
    fn2()
}
fn1()
```

- 把函数赋值之后再调用

```javascript
var a = 1
var obj1 = {
    a:2,
    fn:function(){
        console.log(this.a)
    }
}
var fn1 = obj1.fn
fn1()//1
```

obj1.fn是一个函数`function(){console.log(this.a)}`，此时fn1就是不带任何修饰的函数调用，`function(){console.log(this.a)}.call(undefined)`，按理说打印出来的 this 应该就是 undefined 了吧，但是浏览器里有一条规则：

> 如果你传的 context 就 null 或者 undefined，那么 window 对象就是默认的 context（严格模式下默认 context 是 undefined）

因此上面的this绑定的就是window，它也被称为隐性绑定。

 如果你希望打印出2，可以修改`fn1()`为`fn1.call(obj1)`，显示地绑定this为obj1

- 回调函数

```javascript
var a = 1
function f1(fn){
    fn()
    console.log(a)//1
}
f1(f2)

function f2(){
    var a = 2
}
```

改写代码如下：

```javascript
var a = 1
function f1(){
    (function (){var a = 2})()
    console.log(a)//1
}
f1()
```

仍旧是最普通的函数调用，`f1.call(undefined)`，this指向window，打印出的是全局的a。

 借此，我们终于可以解释为什么setTimeout总是丢失this了，因为它也就是一个回调函数而已。

```javascript
setTimeout(function() {
    console.log(this)//window
    function fn(){
        console.log(this)//window
    }
    fn()
}, 0);

```

构造器调用模式:new一个函数时，背地里会将创建一个连接到prototype成员的新对象，同时this会被绑定到那个新对象上

```javascript
function Person(name,age){
// 这里的this都指向实例
    this.name = name
    this.age = age
    this.sayAge = function(){
        console.log(this.age)
    }
}

var dot = new Person('Dot',2)
dot.sayAge()//2
```

### call

call 方法第一个参数是要绑定给this的值，后面传入的是一个参数列表。当第一个参数为null、undefined的时候，默认指向window。

```javascript
var arr = [1, 2, 3, 89, 46]
var max = Math.max.call(null, arr[0], arr[1], arr[2], arr[3], arr[4])//89
```

可以这么理解：

```javascript
obj1.fn() 
obj1.fn.call(obj1);

fn1()
fn1.call(null)

f1(f2)
f1.call(null,f2)
```

看一个例子：

```javascript
var obj = {
    message: 'My name is: '
}

function getName(firstName, lastName) {
    console.log(this.message + firstName + ' ' + lastName)
}

getName.call(obj, 'Dot', 'Dolby')
```

### apply

apply接受两个参数，第一个参数是要绑定给this的值，第二个参数是一个参数数组。当第一个参数为null、undefined的时候，默认指向window。

```javascript
var arr = [1,2,3,89,46]
var max = Math.max.apply(null,arr)//89
```

可以这么理解：

```javascript
obj1.fn() 
obj1.fn.apply(obj1);

fn1()
fn1.apply(null)

f1(f2)
f1.apply(null,f2)
```

是不是觉得和前面写的call用法很像，事实上apply 和 call 的用法几乎相同, 唯一的差别在于：当函数需要传递多个变量时, apply 可以接受一个数组作为参数输入, call 则是接受一系列的单独变量。

 看一个例子：

```javascript
var obj = {
    message: 'My name is: '
}

function getName(firstName, lastName) {
    console.log(this.message + firstName + ' ' + lastName)
}

getName.apply(obj, ['Dot', 'Dolby'])// My name is: Dot Dolby
```

可以看到，obj 是作为函数上下文的对象，函数 getName 中 this 指向了 obj 这个对象。参数 firstName 和 lastName 是放在数组中传入 getName 函数。

call和apply可用来借用别的对象的方法，这里以call()为例：

```javascript
var Person1  = function () {
    this.name = 'Dot';
}
var Person2 = function () {
    this.getname = function () {
        console.log(this.name);
    }
    Person1.call(this);
}
var person = new Person2();
person.getname();       // Dot
```

从上面我们看到，Person2 实例化出来的对象 person 通过 getname 方法拿到了 Person1 中的 name。因为在 Person2 中，Person1.call(this) 的作用就是使用 Person1 对象代替 this 对象，那么 Person2 就有了 Person1 中的所有属性和方法了，相当于 Person2 继承了 Person1 的属性和方法。

对于什么时候该用什么方法，其实不用纠结。如果你的参数本来就存在一个数组中，那自然就用 apply，如果参数比较散乱相互之间没什么关联，就用 call。像上面的找一组数中最大值的例子，当然是用apply合理。

### bind

和call很相似，第一个参数是this的指向，从第二个参数开始是接收的参数列表。区别在于bind方法返回值是函数以及bind接收的参数列表的使用。

- bind返回值是函数

```javascript
var obj = {
    name: 'Dot'
}

function printName() {
    console.log(this.name)
}

var dot = printName.bind(obj)
console.log(dot) // function () { … }
dot()  // Dot
```

bind 方法不会立即执行，而是返回一个改变了上下文 this 后的函数。而原函数 printName 中的 this 并没有被改变，依旧指向全局对象 window。

- 参数的使用

```javascript
function fn(a, b, c) {
    console.log(a, b, c);
}
var fn1 = fn.bind(null, 'Dot');

fn('A', 'B', 'C');            // A B C
fn1('A', 'B', 'C');           // Dot A B
fn1('B', 'C');                // Dot B C
fn.call(null, 'Dot');      // Dot undefined undefined
```

call 是把第二个及以后的参数作为 fn 方法的实参传进去，而 fn1 方法的实参实则是在 bind 中参数的基础上再往后排。

有时候我们也用bind方法实现函数珂里化，以下是一个简单的示例：

```javascript
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

在低版本浏览器没有 bind 方法，我们也可以自己实现一个。

```javascript
if (!Function.prototype.bind) {
    Function.prototype.bind = function () {
        var self = this,                        // 保存原函数
            context = [].shift.call(arguments), // 保存需要绑定的this上下文
            args = [].slice.call(arguments);    // 剩余的参数转为数组
        return function () {                    // 返回一个新函数
            self.apply(context, [].concat.call(args, [].slice.call(arguments)));
        }
    }
}
```

### 应用场景

- 求数组中的最大和最小值

```javascript
var arr = [1,2,3,89,46]
var max = Math.max.apply(null,arr)//89
var min = Math.min.apply(null,arr)//1
```

- 将类数组转化为数组

```javascript
var trueArr = Array.prototype.slice.call(arrayLike)
```

- 数组追加

```javascript
var arr1 = [1,2,3];
var arr2 = [4,5,6];
var total = [].push.apply(arr1, arr2);//6
// arr1 [1, 2, 3, 4, 5, 6]
// arr2 [4,5,6]
```

- 判断变量类型

```javascript
function isArray(obj){
    return Object.prototype.toString.call(obj) == '[object Array]';
}
isArray([]) // true
isArray('dot') // false
```

- 利用call和apply做继承

```javascript
function Person(name,age){
    // 这里的this都指向实例
    this.name = name
    this.age = age
    this.sayAge = function(){
        console.log(this.age)
    }
}
function Female(){
    Person.apply(this,arguments)//将父元素所有方法在这里执行一遍就继承了
}
var dot = new Female('Dot',2)
```

- 使用 log 代理 console.log

```javascript
function log(){
  console.log.apply(console, arguments);
}
// 当然也有更方便的 var log = console.log()
```

### 总结

call、apply和bind函数存在的区别:

bind返回对应函数, 便于稍后调用； apply, call则是立即调用。

除此外, 在 ES6 的箭头函数下, call 和 apply 将失效, 对于箭头函数来说:

- 箭头函数体内的 this 对象, 就是定义时所在的对象, 而不是使用时所在的对象;所以不需要类似于`var _this = this`这种丑陋的写法
- 箭头函数不可以当作构造函数，也就是说不可以使用 new 命令, 否则会抛出一个错误
- 箭头函数不可以使用 arguments 对象,，该对象在函数体内不存在. 如果要用, 可以用 Rest 参数代替
- 不可以使用 yield 命令, 因此箭头函数不能用作 Generator 函数，什么是Generator函数可自行查阅资料，推荐阅读阮一峰[Generator 函数的含义与用法](https://link.jianshu.com/?t=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2015%2F04%2Fgenerator.html)，[Generator 函数的异步应用](https://link.jianshu.com/?t=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fgenerator-async)