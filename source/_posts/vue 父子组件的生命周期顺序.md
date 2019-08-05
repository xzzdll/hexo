---
title: vue 父子组件的生命周期顺序
categories: vue
# tags: [javascript]
---

​	平常我们使用vue的时候常常要跟生命周期打交道，但是父子组件的生命周期顺序是互相交叉的，这里权当做个备忘吧。

###### 一、加载渲染过程

父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount->子mounted->父mounted

###### 二、子组件更新过程

父beforeUpdate->子beforeUpdate->子updated->父updated

###### 三、父组件更新过程

父beforeUpdate->父updated

###### 四、销毁过程

父beforeDestroy->子beforeDestroy->子destroyed->父destroyed