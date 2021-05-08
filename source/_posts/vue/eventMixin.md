---
title: 事件相关的实例方法
date: 2021-05-07 16:18:36
tags: ['Vue实现原理','Vue深入浅出读书笔记']
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

##### 使用说明

作用：监听当前实例上自定义事件，事件由emit 触发。回调函数会接收所有事件触发函数的额外参数

参数说明：
```
{string | Array<string>} event
{Function} callback
```

示例:
```js
vm.$on('test',function(msg){
    console.log(msg) // 'hi'
})
vm.$emit('test','hi')
```

##### 实现原理
注册事件时将回调函数收集起来，触发事件时将收集起来的回调依次调用。
vm.$on 需要做的事，就是将回调函数收集起来

```js
Vue.prototype.$on=function(event,fn){
    const vm=this;
    if(Array.isArray(event)){
        for(let i=0;i<event.length;i++){
            this.$on(event[i],fn)
        }
    }else{
        (vm._events[event] || vm._events[event]=[]).push(fn)
    }
    return vm
}
```
> 上述代码中的 vm._events 在 new Vue 初始化时候 在Vue.js实例上创建_events 属性，用于存储事件。 vm._events=Object.create(null)


### vm.$emit(event,[...args])

##### 使用说明
触发实例上的事件，并将参数传递给监听器回调。

参数说明：
```
{string } event
[...args]
```

示例:
```js
vm.$on('test',function(msg){
    console.log(msg) // 'hi'
})
vm.$emit('test','hi')
```
##### 实现原理
vm.$on 将事件监听器回调函数都存储在vm._events 中，vm.$emit 从vm._event 中取出事件监听器回调函数列表，依次执行列表中的回调函数并传入参数

```js
Vue.prototype.$emit=function(event,...args){
    const vm=this;
    let cbs=vm._events[event]
    if(cbs){
        for(let i=0;i<cbs.length;i++){
           try{
                cbs[i].apply(vm,args)
           }catch(err){
               console.log(err)
           }
        }
    }
    return vm
}
```


### vm.$off( [event, callback] )
##### 使用说明
移除自定义事件的监听器
- 如果没有提供参数，则移除所有事件监听器
- 如果只提供了事件，则移除该事件所有的监听器
- 如果同时提供了事件与回调，则移除这个回调得监听器

参数说明：
{string | Array<string>} event
{Function} callback

##### 实现原理
根据提供的参数情况，来移除监听器
```js
Vue.prototype.$off=function(event,fn){
    const vm=this;
    if(!arguments.length){ //情况一：没有参数移除所有事件监听器
        vm._events=Object.create(null)
        return vm
    }

    if(Array.isArray(event)){ // event 为数组
        for(let i=0;i<event.length;i++){
            this.$off(event[i],fn)
        }
        return vm
    }
    const cb=this._events[event];
    if(!cbs){
        return vm
    }
    if(event){
        this._events[event]=null
        return vm
    }

}
```

### 相关链接
[发布订阅模式](https://www.cnblogs.com/lovesong/p/5272752.html)