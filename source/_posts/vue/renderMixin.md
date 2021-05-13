---
title: Vue nextTick原理
date: 2021-05-09 11:30:25
tags: ['Vue实例方法','Vue深入浅出读书笔记']
categories : Vue 
toc: true
---
### 概述
生命周期相关的实例方法有4个：vm.$mount、vm.$forceUpdate 、vm.$destroy、vm.$nextTick，本文主要讲述 vm.$nextTick。

### 使用&作用
**语法：**
`vm.$nextTick( [callback] )`


**作用：**
将回调延迟到下次 DOM 更新循环之后执行。在修改数据之后立即使用它，然后等待 DOM 更新。


**示例：**
```js
//回调函数写法
this.$nextTick(function(){
})
//Promise写法
this.$nextTick().then(function(){})
```


**"下一个DOM 更新循环"是什么意思？？？**

当状态变化时，watcher 会得到通知，然后触发虚拟DOM的渲染流程。watcher 触发渲染这个操作是异步的。 Vue.js 中有一个队列，每当需要渲染时，会将watcher 推送到这个队列中，在下一次事件循环中再让watcher 触发渲染的流程

>数据的变化到 DOM 的重新渲染是一个异步过程


**1.为什么Vue.js 使用异步更新队列**  

变化侦测的通知发送到组件，组件内用到的所有状态的变化都会通知到同一个watcher,然后对整个组件的新旧虚拟DOM进行对比并更改DOM。如果同一轮事件循环中有两个数据发生了变化，那么组件的watcher 会收到两份通知，从而进行两次渲染？当然不是！

vue为了解决上述问题，Vue的实现方式是将收到通知的watcher实例 添加到队列中缓存起来，并且在添加到队列之前检查是否已经存在相同watcher,只有不存在时才会将watcher实例添加到队列中。 然后在下一次事件循环中，Vue.js 会让队列中的Watcher 触发渲染流程并清空队列。这样就可以保证同一事件循环中，两个数据变化,watcher 最终也只执行一次渲染流程。



**2.什么是事件循环**
javaScript 是单线程非阻塞的脚本语言，这就意味着javascript 代码在执行的时候有一个主线程来处理所有任务，非阻塞是指处理异步任务时，主线程会挂起这个任务，当异步任务处理完毕，将这个事件加入事件队列。事件队列。

 异步任务有两种类型：微任务（mincrotask)、宏任务(macrotask)。不同类型的任务会被分配到不同的任务队列中。

微任务：   
Promise.then、MutationObserve

宏任务：   
setTimeout、setInterval、setImmediate、PostMessage


 当执行栈中所有任务都执行完毕后，会去检查微任务队列是否有事件存在，如果存在，则会依次执行并移除微任务队列中事件对应的回调，直到为空。
 然后去宏任务队列中取出一个任务放入执行栈，当执行栈中所有任务都执行完毕后，检查微任务队列中是否有事件存在，不断循环这个过程 ，这个循环就叫做事件循环

**3.什么是执行栈**
当我们执行一段可执行代码时，会生成对应的执行上线文，执行上下文会被添加到一个栈中，这个栈就是执行栈。   

回到前面的问题 ”下一个DOM 更新循环“的意思其实是下一次微任务执行时更新DOM。vm.$nextTick 其实是将回调添加到微任务中，如果语法不支持则降级成宏任务

因此，使用vm.$nextTick来获取更新后的DOM，则需要注意顺序问题。因为**不论是更新DOM的回调，还是vm.$nextTick 注册的回调，都是向微任务队列中添加任务，所以哪个任务先添加到队列中，就先执行哪个任务**  
 
>更新DOM的回调也是使用vm.$nextTick 来注册到微任务队列中的

如果想在vm.$nextTick 中获取更新后的DOM，则一定要在更新数据的后面使用vm.$nextTick 注册回调。如果先使用vm.$nextTick 注册回调再修改数据 是获取不到更新后的DOM的。因为在微任务队列中先执行vm.$nextTick 回调，后执行DOM更新。


在事件循环中，当执行栈中同步任务执行完之后，会先执行并清空微任务后 ，才会从宏任务队列中取出一个事件,执行下一轮。

所以添加到微任务队列中的任务执行时机优先于宏任务队列中的任务  

```js
new Vue({
    methods:{
        example:function(){
            setTimeout(()=>{
                //dom更新了
            },0)
            this.message='changed'
        }
    }
})
```
setTimeout 是宏任务，宏任务比微任务执行晚，所以即便是先注册也是先执行更新DOM，后执行setTimeout 回调  

### 实现原理  

```js
import {nextTick} from './util/index'
Vue.prototype.$nextTick=function(fn){
    return nextTick(fn,this)
} 
```
```js
const callbacks=[];
let pending=false;//用于标记是否已经向任务队列中添加一个任务
function flushCallbacks(){
    pending=false
    const copies=callbacks.slice(0);
    callbacks.length=0;
    for(let i=0;i<copies.length;i++){
        copies[i]()
    }
}

let microTimerFunc; 
let macroTimerFunc=;
let useMacroTask=false;
// 宏任务方法定义：降级顺序 setImmediate——>MessageChannel——>SetTimeout
if(typeof setImmediate !=='undefined' && isNative(setImmediate)){
    macroTimerFunc=()=>{
        setImmediate(flushCallbacks)
    }
}else if(type of MessageChannel !=='undefined' && (isNative(MessageChannel) || MessageChannel.toString ==='[object MessageChannelConstructor]' )){
    const channel=new MessageChannel();
    const port=channel.port2
    channel.port1.onmessage=flushCallbacks
    macroTimerFunc=()=>{
        port.postMessage(1)
    }
}else{
    macroTimerFunc=()=>{
        setTimout(flushCallbacks,0)
    }
}
//微任务方法定义：不支持promise 降级为宏任务
if(typeof Promise !=='undefined' && isNative(Promise)){
    const p=Promise.resolve();
    microTimerFunc=()=>{  
        p.then(flushCallbacks) //将flushCallbacks 添加到微任务队列中
    }
}else{
    microTimerFunc=macroTimerFunc
}
// withMacroTask 的作用是给回调函数做一层包装，保证在回调函数执行过程中，如果修改了数据，那么更新DOM的操作会被推倒宏任务队列中。【更新DOM的执行事件会晚于回调函数的执行事件】
export function withMacroTask(fn){
    return fn.withFask || (fn._withTask=function(){
        useMacroTask=true;
        const fn.apply(null,arguments)
        useMacroTask=false
        return res
    })
}
//没有提供回调且支持promise 时候返回promise
export function nextTick(cb,ctx){
    let _resolve
    callbacks.push(()=>{
        if(cb){
            cb.call(ctx)
        }else if(_resolve){
            _resolve(ctx)
        }
    })
    if(!pending){
        pending=true
        if(useMacroTask){
            macroTimerFunc()
        }else{
            microTimerFunc()
        }
    }
    //promise
    if(!cb && typeof Promise !=='undefined'){
        return new Promise(resolve=>{
            _resolve=resolve
        })
    }
}
//测试 
nextTick(function(){
    console.log(this.name) //mandyer
},{
    name:'mandyer'
})

```

### nextTick内部注册和执行流程
**注册流程：**
首先nextTick 被调用时，会将回调函数添加到callbacks 中，如果此时是本轮事件循环第一次使用nextTick,那么需要向任务队列中添加队列（Promise.then 作用是将任务添加到微任务队列中）。如果不是本轮事件循环中第一次调用nextTick，就说明任务队列已经已经添加了执行回调列表的任务，就不需要重复添加了。



```flow
s=>start: nextTick
e=>end: 结束
op1=>operation: 将回调添加到callbacks中
c1=>condition: 本轮事件循环中第一次使用nextTick?
op2=>operation: 向任务队列中添加任务

s->op1->c1(yes)->op2->e
s->op1->c1(no)->e
```

---
**执行流程：**
依次执行callbacks 中所有的回调

```flow
op1=>operation: 任务被执行
op2=>operation: 依次执行callbacks 中的所有回调
op3=>operation: 清空callbacks

op1->op2->op3
```

总结：
- 使用nextTick的作用是让回调再下一次DOM更新之后执行。
- Vue.js 是怎么作用让回调函数在DOM更新后执行的呢？
数据更新到视图渲染是一个异步的过程，我们手动调用nextTick 和 触发Dom更新都会调用nextTick，nextTick会将回调函数添加到callbacks 队列中，当callbacks 被执行时 先加入的会先执行，后加入的后执行 ，所以在更新dom 回调 后面加入callbacks 的回调可以获取到更新后的DOM节点
- callbacks 如何执行？
nextTick 会将执行callbacks 的任务添到任务队列中，且一次事件循环中只添加一次

