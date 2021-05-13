---
title: 生命周期相关的实例方法 之 vm.$forceUpdate 、vm.$destroy
date: 2021-05-09 11:29:27
tags: ['Vue实例方法','Vue深入浅出读书笔记']
categories : Vue 
toc: true
---
### 概述
生命周期相关的实例方法有4个：vm.$mount、vm.$forceUpdate 、vm.$destroy、vm.$nextTick

本文主要讲述 vm.$forceUpdate 、vm.$destroy,这两个方法从lifecycleMixin中挂载到Vue构造函数的prototype 属性上

```js
export function lifecycleMixin(Vue){
    Vue.prototype.$forceUpdate=function(){}
    Vue.prototype.$destroy=function(){}
}
```


### vm.$forceUpdate
##### 使用说明
vm.$forceUpdate 的作用是强制Vue实例重新渲染。

**组件就是Vue.js实例**，vm.$forceUpdate 仅仅影响实例本身以及插入插槽内容的子组件，而不是所有组件

##### 实现原理
执行watcher的update方法，就可以使Vue.js实例重新渲染。

```js
 Vue.prototype.$forceUpdate=function(){
     const vm=this;
     if(vm._watcher){
         vm._watcher.update()
     }
 }
```
**vm._watcher是Vue.js实例的watcher,每当组件内依赖的数据发生变化时，都会自动触发Vue.js实例中_watcher的update**


总结：强制更新的原理就是调用vue实例的watcher 的update方法

### vm.$destroy 
##### 使用说明
vm.$destroy 的作用是完全销毁一个实例，它会清理该实例与其他实例的连接，并解绑全部的指令及监听器，同时会触发beforeDestroy 和 destroyed 的钩子函数

##### 实现原理
销毁实例逻辑:  
(0)开始清理前
```js
Vue.prototype.$destroy=function(){
    const vm=this;
    if(vm._isBeginDestroyed){
        return
    }
    callHook(vm,'beforeDestroy')
    vm._isBeginDestroyed=true
}
```
（1）清理当前组件与父组件之间的链接
操作：将当前组件实例从父组件实例的$children属性中删除即可。

```js
const parent=vm.$parent
if(parent && !parent._isBeginDestroyed && !vm.$options.abstract){
    remove(parent.$children,vm)
}

function remove(arr,item){
    if(arr.length){
        const index=arr.indexOf(item);
        if(index>-1){
            array.splice(index,1)
        }
    }
}
```
> remove函数不是使用循环遍历的方式，而是下标结合splice 方法将元素从数组中删除 【优雅技巧get】

（2）移除watcher 

**实例watcher**
    状态会收集依赖(watcher)，当状态变化时通知依赖。当Vue.js实例被销毁时，将watcher 从Dep列表中移除

```js
if(vm._watcher){
    vm._watcher.teardown()
}
```
watcher 的teardown 方法的作用是从所有依赖列表中将自己移除。【此时移除的只是vue实例上的watcher,还有用户自定义watcher 未被移除】



Vue2.0开始，变化侦测的粒度调整为中等粒度，只会发送通知到组件级别，然后组件使用虚拟DOM进行渲染（组件就是Vue实例）

Vue.js 实例上，有一个watcher 就是vm._watcher 它会监听这个组件中用到的所有状态，将这个组件内用到的所有状态的依赖列表都会收集到vm._watcher 中。当状态发生变化时，会通知vm._watcher ，然后这个watcher 再调用虚拟DOM进行重新渲染


**用户自定义watcher**
移除用户自定义watcher 同样也是使用teardown方法

每当创建watcher 实例时，都会将watcher 实例添加到vm._watchers中
```js
export default class Watcher{
    constructor(vm,exOrFn,cb){
        vm._watchers.push(this)
    }
}
```
因此用户创建的watcher 都在vm._watcher中，当移除的时候遍历数组调用watcher的teardown方法即可

```js
let i=vm.watchers.length
while(i--){
    vm._watchers[i].teardown()
}
```

(3)Vue实例添加_isDestroyed属性表示Vue实例已被销毁
```js
vm._isDestroyed=true
```

(4)指令解绑  
vm.$destroy 执行时，Vue.js不会将已经渲染到页面中的DOM节点移除，但会将模板中的所有指令解绑 
```js
vm.__patch__(vm._vnode,null)
```
(5)触发destroyed 构造函数
```js
callHook(vm,'destroyed')
```
(6)移除实例上的所有事件监听器
vm.$off()

**完整代码**
```js
Vue.prototype.$destroy=function(){
    const vm=this;
    if(vm._isBeginDestroyed){
        return
    }
    callHook(vm,'beforeDestroy')
    vm._isBeginDestroyed=true
    // 删除自己与父级之间的链接
    const parent=vm.$parent
    if(parent && !parent._isBeginDestroyed && !vm.$options.abstract){
        remove(parent.$children,vm)
    }
    // 从watcher 监听的所有状态的依赖列表中移除watcher
    if(vm._watcher){
    vm._watcher.teardown()
    }
    let i=vm.watchers.length
    while(i--){
        vm._watchers[i].teardown()
    }
    vm._isDestroyed=true
    // 在vnode 树上触发destroy构造函数解绑指令
    vm.__patch__(vm._vnode,null)
    //触发destroyed 构造函数
    callHook(vm,'destroyed')
    //移除所有的事件监听器
    vm.$off()
}
```