---
title: Vue中的 nextTick实现
categories: css
# tags: [javascript]
---

思考一下 下面的代码

```javascript
  setTimeout(() => console.log("timer") , 0);
  var arr = new Array(1e7);
  console.log(arr.length);
  while ( arr.length ) arr.pop();
  console.log(arr.length);
```

结论：
1.js定时器的时间可能是不准的
2.在一次事件循环中,同步操作执行完,异步操作(宏任务与微任务)是不会执行的,直到主线程为等待状态 异步 操作才能被执行

Vue中的 nextTick实现 简化版

```javascript
let callbacks = [];
function nextTick ( callback , ctx ) {
        setTimeout(function () {
            let    i = -1;
            while ( ++i < callbacks.length ) {
                callbacks[ i ].call(ctx)
            }

            callbacks.length = 0
        } , 0);
    callbacks.push(callback)
}
```

实际上 这段代码是有问题的 因为我们每调用一次nextTick 实际就执行了一次新的定时器
那么我们需要给定时器加一个锁,让定时器没有触发之前不再执行新的定时器

```javascript
let callbacks = [];
let wait = false;
function nextTick ( callback , ctx ) {
    if ( !wait ) {
        wait = true;
        setTimeout(function () {
            wait = false;
            let   i = -1;
            while ( ++i < callbacks.length ) {
                callbacks[ i ].call(ctx)
            }

            callbacks.length = 0
        } , 0);
    }

    callbacks.push(callback)
}
```

对Promise支持 用法 this.$nextTick().then()

```javascript
let callbacks = [];
let wait = false;
function nextTick ( callback , ctx ) {
    if ( !wait ) {
        wait = true;
        setTimeout(function () {
            wait = false;
            let i = -1;
            while ( ++i < callbacks.length ) {
                callbacks[ i ]()
            }

            callbacks.length = 0
        } , 0);
    }
    var resolve$1 = null;

    callbacks.push(function () {
        if ( callback ) {
            callback.call(ctx)
        } else if ( resolve$1 ) {
            resolve$1()
        }
    });

    if ( !callback && typeof Promise !== "undefined" ) {
        return new Promise(resolve => {
            resolve$1 = resolve
        })
    }
}
```

为什么要这么做？
为什么这么做，我们的callback就能在更新完成后被执行？
先理解一下 Watcher是什么
什么是Watcher
Vue给我们暴露的一个接口 $watch（也可以叫做 user watcher 或者侦听器）
从$watch开始简单的理解就是 当我侦听一个或多个值变化的时候,去执行一个callback

Watcher的作用
1.配合Observer对象收集依赖 
2.侦听依赖的值变化时执行callback


除了$watch 外的另一种Watcher
那么,实际上 在Vue内部还有一种Watcher就是renderWatcher ，当侦听模板中依赖的所有值( touch 不到的值除外 )变化时,触发 _render()  （触发虚拟dom生成 -> 然后对之前的虚拟dom进行differ算法-> 根据differ的算法结果对真实dom进行更新->缓存本次虚拟dom用于下一次对比->清理上一次生成的虚拟dom）

简单实现一下 更新队列

```javascript
let watchers = [];
function flushWatcherQueue () {
    //去重
    watchers.uniqueByID();

    // Sort queue before flush.
    // This ensures that:
    // 1. Components are updated from parent to child. (because parent is always
    //    created before the child)
    // 2. A component's user watchers are run before its render watcher (because
    //    user watchers are created before the render watcher)
    // 3. If a component is destroyed during a parent component's watcher run,
    //    its watchers can be skipped.
    watchers.sort(function ( a , b ) {
        return a.id < b.id
    });

    let   i = -1;
    while ( ++i < watchers.length ) {
        watchers[ i ].run();
    }

    watchers.length = 0
}
```

来一个实际例子

```javascript
export default {
    template : "<span> {{b}} </span>" ,

    data(){
        return {
            a : "" ,
            b : ""
        }
    } ,

    watch : {
        a( v ){
            console.log(v)
        }
    } ,

    mounted(){
        this.a = "123";
        //this.a.__ob__.subs === [ $watcher]
        // watchers.push(...this.a.__ob__.subs)
        // nextTick(flushWatcherQueue);

        this.a = "234";
        //this.a.__ob__.subs === [ $watcher]
        // watchers.push(...this.a.__ob__.subs)

        this.b = "abc";
        // this.b.__ob__.subs === [renderWatcher]
        // watchers.push(...this.b.__ob__.subs)

        this.$nextTick(new Function);

        this.b = "bcd";
        // this.b.__ob__.subs === [renderWatcher]
        // watchers.push(...this.b.__ob__.subs)

    }
}
```