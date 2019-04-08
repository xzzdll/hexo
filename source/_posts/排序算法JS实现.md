---
title: 排序算法JS实现
categories: javascript
# tags: [javascript]
---

#### 快速排序JS实现



```javascript
function demo (arr){
 if(arr.length < 2){
  return arr
 }else{
  var tmp = arr[0]
  var less = []
  var more = []
  for(var i=1;i< arr.length;i++){
   if(arr[i] < tmp){
    less.push(arr[i])
   }else{
    more.push(arr[i])
   }
  }
  return demo(less).concat(tmp).concat(demo(more))
 }
}

console.log(demo([6,5,4,3,2,1]))
```





#### 桶排序JS实现



```javascript
function demo(arr) {
    var tmp = Array(arr.length + 1);
    for (let i = 0; i < tmp.length; i++) {
        tmp[i] = 0for (let q = 0; q < arr.length; q++) {
            if (arr[q] == i) {
                tmp[i] += 1
            }
        }
    }
    var tmp2 = []
    console.log(tmp)
    for (let i = 0; i < tmp.length; i++) {
        if (tmp[i] != 0) {
            for (let q = 0; q < tmp[i]; q++) {
                tmp2.push(i)
            }
        }
    }
    return tmp2
}

console.log(demo([6, 5, 4, 3, 2, 1]))
```





#### 冒泡排序js实现

基本思想：每次比较相邻的两个元素，如果他们的顺序错误就把他们交换过来



```javascript
function demo(arr) {
  for (var q = 0; q < arr.length; q++) {
    for (var i = 0; i < arr.length -1 - q; i++) {
      if (arr[i] < arr[i + 1]) {
        var tmp = arr[i + 1]
        arr[i + 1] = arr[i]
        arr[i] = tmp
      }
    }
  }
  return arr
}

console.log(demo([6, 5, 8, 2, 4, 6, 8, 4, 2, 58, 4]))
```