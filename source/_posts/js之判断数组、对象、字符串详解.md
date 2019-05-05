---
title: js之判断数组、对象、字符串详解
categories: javascript
# tags: [javascript]
---

平常写代码时经常会需要判断数据类型，但是老年人记性不是太好，经常会忘掉具体的写法，所以总结在这里备忘。

##### 判断数组

此方法是ES6新增的一个方法，可以很方便的判断对象是否为数组。

```javascript
Array.isArray(arr)
//返回true or false
```

没有ES6的话，可以使用以下方法判断数组的原型对象的类型string。

```javascript
Object.prototype.toString.call(arr)
//返回“[object Array]”
```

##### 判断对象

判断对象与数组相比没有新增的语法糖api去判断，相比之下会麻烦一点

```javascript
Object.prototype.toString.call(object)
//返回“[object Object]”
```



##### 判断字符串

判断字符串

```javascript
typeof this.value === 'string'
//返回true or false
```

你可能会问为什么数组和对象不能这么判断，那是因为数组和对象的值都是object！