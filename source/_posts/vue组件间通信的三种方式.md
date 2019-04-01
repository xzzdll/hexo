---
title: vue组件间通信的三种方式
categories: javascript
# tags: [javascript]
---

vue是一种mvvm框架，它相对于jquery可能比较大的差异点之一就在于组件之间的通信了。这里提供三种不同情况下vue组件的通信方式。



#### 一. vue父子组件通信

vue父子组件通信可以用Vue.$emit自定义事件来解决。



```
// 父组件
<single-address @edit-address="editAddress"></single-address>
// 子组件
methods: {
 editAddress () {
  this.$emit('edit-address', false)
 }
}
```

当然也可以使用props方式解决。



```
// 父组件
<one-address :addressitems="addressitems"></one-address>
// 子组件
<div>{{ addressitems.partment }}{{ addressitems.address }}</div>
export default {
  props: {
    addressitems: Object
  }
}
```

#### 二. 非父子组件通信

非父子组件通信同样也可以用Vue.$emit自定义事件来解决



```
var bus = new Vue()
// 组件A
bus.$emit('id-selected', 1)
// 组件B
bus.$on('id-selected', function (id) {
 console.log(id)
})
```

#### 三. vue跨组件跨模块通信

虽然父子组件，非父子组件间的通信vue都可以有办法解决，但是如果项目结构复杂化以后，这样的自定义事件变多以后代码难以管理，所以还是建议使用vuex。

 接下来就讲一讲vuex这个东西吧。vuex，官方的说法就是Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。如果之前没有接触过应用状态管理这一块的内容的话，可能光看文档是很难看明白的。按个人的理解来说，vuex最大的作用就是把部分父子组件之间频繁的通信过程简单化，所以，对于一些父子组件通信并不频繁的情况来说，并不应该使用vuex，当然这也意味着，如果你对父子组件之间的通信还不是很明确的话，可以先学习一下这一块的知识。

 vuex有四个核心概念，其中state和getters主要是用于数据的存储与输出，而mutations和actions是用于提交事件并修改state中的数据。

 也是以购物车的业务来举例，首先由于vue的限制，页面中所有的数据都需要一个初始化，这样才能在之后业务逻辑中响应式地修改数据（修改数据的同时变化页面内容）。这里我们可以在vue组件中使用ajax获取数据，获取到的数据直接赋给state中的数据：

```
// global.Domain.centerUrl是我自己定义的全局变量，这种就可以更方便的管理前端数据请求接口的地址了
      getDataFromBackend () {
        this.$http({
          method: 'get',
          url: global.Domain.centerUrl + 'order'
        }).then(function (response) {
          let res = response.data
          // state.order是因为我对整个应用进行了分模块的设计，order是我其中的一个模块，后面会讲如何划分模块
          // $store是vue实例的全局属性，这个属性可以直接获取vuex状态树中的所有state数据
          this.$store.state.order.orderitems = res.orderitems
        })
      }
```

getters的用法与计算属性基本一致，computed会用的话那这个也基本就会用了。

 接下来就是重头戏mutations了。mutations原意是突变，这里可以理解为立即修改吧。也就是说，mutations可以修改state中的数据，但是有一个限制，那就是只能同步修改，而不能异步，你想要按个定时器过2秒钟修改state，那对不起了您哪，不行。但是，想要异步修改state，也不是不行，光靠mutations一个人是不行的了，还得要靠actions，而且actions还不能直接操作state，他需要异步提交给mutations，然后再由mutations同步修改state数据。听上去挺麻烦的一件事情啊，还是直接看代码吧。

```
  mutations: {
    correctIfcollect (state) {
      state.ifcollectwrap = true
    },
    modifyIfcollect (state) {
      state.ifcollectwrap = false
    },
    // 收藏成功事件
    collectSuccess (state) {
      state.ifcollect = true
      state.collectmsg = '收藏成功'
    },
    // 取消收藏事件
    collectCancel (state) {
      state.ifcollect = false
      state.collectmsg = '取消收藏'
    }
  },
  actions: {
    // actions有一个必要的参数commit，用于提交事件到mutations中
    collectPopup ({commit, state}) {
      let oTimer = ''
      clearTimeout(oTimer)
      if (!state.ifcollect) {
        commit('collectSuccess')
        commit('correctIfcollect')
        oTimer = setTimeout(function () {
          commit('modifyIfcollect')
        }, 2000)
      } else {
        commit('collectCancel')
        commit('correctIfcollect')
        oTimer = setTimeout(function () {
          commit('modifyIfcollect')
        }, 2000)
      }
    }
  }
    // 假如有数据需要从组件传到actions中时，需要两个大括号，第一个大括号放commit或者state这类vuex参数，而第二个大括号放传进来的参数
    // 这里使用vuex actions调用了两个mutations的方法实现一个功能，是因为第二个函数中有一个参数是需要第一个函数提供的
    changePos ({commit}, {index}) {
      commit('changeHandle', index)
      commit('changePos', index)
    }
```

总的来说，在我看懂了vuex以后，我总是觉得有一个vuex来分担一下data的功能那是再好不过了，这样不光能够更好地管理我的应用，同时也能够提高我.vue单文件的可读性。

 那么这里就有一个问题了，我们都知道一个稍微大一点的项目，那数据量都是很多的，如果整个项目里的数据都写在一个vuex文件里的话就显得整个文件太大太重了，这与我们想要提高文件的可读性是相悖的。所以我们需要有一个分模块的意识来管理vuex，管理应用的状态。

 而vuex也正好提供了这个功能（vuex还真的是够强大），在具体使用的过程中，我会先把那些需要与其他组件交互的组件中的数据都提取出来放到vuex里面管理，而那些完全没有交互的数据还是放在组件的data中，例如标识该组件的标题数据等等。然后，这里我们就可以用到vuex的模块管理功能，愉快地把数据按照组件之间的功能联系来分离，并把它们拆分到一个个小文件中啦。

```
// 这是vuex总的管理文件，将各模块统一在一起，从而将每一个分支都连接到vuex这个总的状态树上
// 引入vue
import Vue from 'vue'
// 引入vuex
import Vuex from 'vuex'
// 引入应用状态管理
import goodshandleStore from './goodshandle'
import orderStore from './order'
import addressStore from './address'
import shopcarStore from './shopcar'
import merchantStore from './merchant'
import couponStore from './coupon'

Vue.use(Vuex)

export default new Vuex.Store({
  modules: {
    handle: goodshandleStore,
    order: orderStore,
    address: addressStore,
    shopcar: shopcarStore,
    merchant: merchantStore,
    coupon: couponStore
  }
})
```