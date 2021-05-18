---
title: 数据相关的实例方法
date: 2021-05-07 16:41:39
tags: ['Vue实例方法','Vue深入浅出读书笔记']
categories : Vue 
toc: true
---

[vm.$watch](#vm.$watch)
[vm.$set](#vm.$set)
[vm.$delete](#vm.$delete)
### 概述
数据相关的实例方法有3个：vm.$watch、vm.$set、vm.$delete,他们是在stateMixin 中挂载到Vue原型中的

Vue大致实现如下：
```js
import {stateMixin} from './xxx'
function Vue(options){
    //....
     stateMixin(Vue)
    //...
}
```

```js
import {set,del} from '../observer/index';
export function stateMixin(Vue){
    Vue.prototype.$set=set;
    Vue.prototype.$delete=del;
    Vue.prototype.$watch=function(expOrFn,cb,options){}
}
```

下面将详细介绍 vm.$watch、vm.$set、vm.$delete 实现原理

### <div id="vm.$watch">vm.$watch(expOrFn,callback,[options])</div>
##### 使用说明
用于观察一个表达式或者computed函数在Vue.js实例上的变化，变化后触发回调传入new value 和 old value。第一个参数如果是表达式只接受以点分隔eg:a.b.c,如果复杂的表达式，可以用函数代替表达式。   


参数说明：  

{string | Function} expOrFn
{Function | Object} callback
{Object} [options] deep | immediate

deep:为了发现对象内部值的变化,deep:true
immediate:立即触发回调，immediate:true


返回值：{Function} unwatch

示例:
```js
vm.$watch('a.b.c',function(newVal,oldVal){
    
})

```
##### 实现原理
TODO 待补充...

---

### <div id="vm.$set">vm.$set(target,key,value)</div>
##### 使用说明
在对象上设置一个属性，如果object 是响应式的，保证属性被创建后也是响应式的，并且触发视图更新。该方法用于解决Vue.js无法监听object添加新属性的问题

参数说明：

{Object | Array} target
{string | number} key
{any} value

返回值：{Function} unwatch

示例:

```js
let obj={a:'a'}
this.$set(obj,'key','val')
```
##### 实现原理
```js
export function set(target,key,value){
    //情况一：数组
    if(Array.isArray(target) && isValidArrayIndex(key)){
        trget.length=Max.max(target.length,key);
        target.splice(key,1,value)
        return value
    }
    //情况二：已存在属性
    if(target.hasOwnProperty(key)){
        target[key]=value
        return value
    }
    //情况三：新增属性
    const ob=target.__ob__ //__ob__属性用于判断是否是响应式对象
    if(target._isVue || (ob&&ob.vmCount)){
        //警告不可以是 vue实例 或者 根数据对象
        return value
    }
    if(!ob){
        target[key]=value
        return value
    }
    definReactive(ob.value,key,value)//使用definReactive将新增属性转为响应式数据
    ob.dep.notify() //向target的依赖触发变化通知
    return val
}
```
总结：数组类型直接用spice 方法,对象类型分两种情况(1)已存在属性，直接修改属性值 (2)新增属性 ，分为两种情况 对象不是响应式数据 直接修改属性值，对象是响应式数据 使用definReactive 将新增属性换为响应式数据，然后调用dep.notify通知依赖
---
### <div id="vm.$delete">vm.$delete(target,key)</div>
##### 使用说明
vm.$delete 方法用来删除数据中的某个属性。如果对象是响应式的要触发视图更新，该方法用于解决Vue.js无法监听object删除新属性的问题

参数说明：
{Object|Array} target
{string | number} key/index

示例:
##### 实现原理

```js
export function del(target,key){
    //数组类型
    if(Array.isArray(target) && isValidArrayIndex(key)){
        target.splice(key,1)
        return
    }
    //对象类型
    if(target._isVue || (ob && ob.vmCount)){
        //警告对象不可以是vue实例，也不可以是根数据
        return
    }
    if(!hasOwn(target,key)){//key 不存在直接return
        return
    }
    const ob=target.__ob__
    delete target[key]
    if(!ob){ //如果不是相应数据无需通知变化
        return
    }
    ob.dep.notify()
}
```
总结：数组类型直接用spice 方法，对象类型 分为两种情况(1)不是响应式数据 只做删除操作 (2)是响应式数据删除后触发dep.notify 进行通知,异常处理有两个，（1）target 不可以为跟数据或者vue实例（2）对象中不存在目标key 直接return