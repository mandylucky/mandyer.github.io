---
title: 事件相关的实例方法
date: 2021-05-07 16:18:36
tags: Vue实现原理
categories : Vue 
toc: true
---
### 概述
与事件相关的实例方法有4个，分别是:vm.$on 、 vm.$once 、vm.$off 和 vm.$emit,这四个方法是在eventsMixin中被挂载到Vue构造函数的原型中的。

Vue大致实现如下：
```js
import {eventsMixin} from './events'
function Vue(options){
    //....
     eventsMixin(Vue)
    //...
}
```
```js
export function eventsMixin(Vue){
    Vue.prototype.$on=function(event,fn){
        //todo something
    }
    Vue.prototype.$emit=function(event,fn){
        //todo something
    }
    Vue.prototype.$once=function(event,fn){
        //todo something
    }
    Vue.prototype.$off=function(event,fn){
        //todo something
    }
}
```

下面将详细介绍vm.$on 、 vm.$once 、vm.$off 、vm.$emit 实现原理

### vm.$on(event,callback) 
监听当前实例上自定义事件，事件由emit 触发。回调函数会接收所有事件触发函数的额外参数


示例:
```js
vm.$on('test',function(msg){
    console.log(msg) // 'hi'
})
vm.$emit('test','hi')
```

