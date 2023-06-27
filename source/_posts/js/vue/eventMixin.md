---
title: 事件相关的实例方法
date: 2021-05-07 16:18:36
tags: ['Vue实例方法','Vue深入浅出读书笔记']
categories : Vue
toc: true
---

[vm.$on](#vm.$on)
[vm.$emit](#vm.$emit)
[vm.$off](#vm.$off)
[vm.$once](#vm.$once)

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

### <div id="vm.$on">vm.$on(event,callback)</div>

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


### <div id="vm.$emit">vm.$emit(event,[...args])</div>

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


### <div id="vm.$off">vm.$off( [event, callback] )</div>
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
    //情况一：没有参数移除所有事件监听器
    if(!arguments.length){ 
        vm._events=Object.create(null)
        return vm
    }
    //处理参数为数组的情况
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
    //情况二: 只提供了事件，移除该事件所有的监听器
    if(arguments.length ===1){ 
        this._events[event]=null
        return vm
    }
    //情况三：同时提供了事件和回调函数，只移除与fn相同的监听器
    if(fn){ 
        const cbs=vm._events[event];
        let cb;
        let i=cbs.length;
        while(i--){ //细节注意：从后往前遍历，不会影响到未遍历到的监听器
            cb=cbs[i];
            if(cb===fn || cb.fn===fn){ //cb.fn===fn 是干啥的？vm.$once中具体讲解
                cbs.splice(i,1);
                break
            }
        }
        return vm
    }

}
```

### <div id="vm.$once">vm.$once(event,callback)</div>

##### 使用说明
监听一个自定义事件，但只触发一次，在一次触发之后移除监听器

参数：
{string | Array<string>} event
{Function} callback

##### 实现原理
监听一个自定义事件，但只触发一次，在一次触发之后移除监听器。上述描述可以使用vm.$on 来监听事件， 使用vm.$off移除监听

```js
Vue.prototype.$once=function(event,fn){
    const vm=this;
    function on(...args){
        vm.$off(event,on) //移除自定义事件监听器
        fn.apply(vm,args) //触发回调
    }
    on.fn=fn //这里为啥这样呢？思考一下
    vm.$on(event,on)
    return vm
}
```

为什么会有 on.fn=fn  这行代码？
思考一个场景，用户在触发事件之前调用vm.$off,那么用户会这么写：`vm.$off('xxx',fn)`
而此时_events是这样的：`vm._events={'xxx':[on]}`,on 和 fn 不一致那么就不会溢出监听器。
结合vm.$off源码，如果on和fn不相等的时候，还会判断on.fn 和 fn 是否相等，如果相等也会移除监听器。这也就是为什么要写on.fn=fn 这行代码的原因。

```js
//vm.$off 源码片段
if(cb===fn || cb.fn===fn){
    cbs.splice(i,1);
    break
 }
```
### 设计模式
Vue 的事件使用了发布订阅模式，有关发布订阅模式的详细介绍可以看这里 [点我](https://www.cnblogs.com/lovesong/p/5272752.html)
