---
title: this
date: 2021-05-11 07:44:54
tags: ['你不知道的js读书笔记','js基础']
categories : js
toc: true
---
# 概述
this是一个特别的关键字，被自动定义在所在函数的作用域中。this提供了一种更优雅的方式隐式”传递“一个对象的引用。

# 关于this的误解
1. this指向自身(❌)
2. this指向函数的词法作用域(❌)

**那this到底是什么?**
this实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用


# 调用位置
**调用位置就是函数在代码中被调用的位置**
**寻找调用位置：**先分析调用栈，找到栈中第二个元素就是真正的调用位置

```js
function baz(){
    // 当前的调用栈是baz,因此调用位置在全局作用域中
    bar()
}
function bar(){
    // 当前的调用栈是 baz->bar,因此当前调用位置在baz中
    foo()
}
function foo(){
    // 当前调用栈是 baz->bar->foo,因此当前调用位置在bar中

}
baz()
```

# 绑定规则
先找到调用位置，然后判断需要应用下面哪一条规则
### 默认绑定
独立函数调用，既不带任何修饰符的函数引用进行调用 使用默认绑定规则.  

默认绑定在非严格模式下this指向全局变量，严格模式下this 会绑定到undefined

```js
//非严格模式
function foo(){
    console.log(this.a)
}
var a=2;
foo() //2
```
```js
//严格模式
function foo(){
    "use strict"
    console.log(this.a)
}
var a=2;
foo() //undefined

```
### 隐式绑定
由上下文对象调用，绑定到那个上下文对象  
```js
function foo(){
    console.log(this.a)
}
var obj={
    a:2,
    foo:foo
}
obj.foo()//2
```
**对象属性引用链中，只有最后一层在调用位置中起作用**
```js
function foo(){
    console.log(this.a)
}
var obj2={
    a:42,
    foo:foo
}
var obj1={
    a:2,
    obj2:obj2
}
obj1.obj2.foo() //42 foo 中的this 绑定到了obj2
```

**隐式绑定函数，丢失绑定对象**
例子1：
```js
function foo(){
    console.log(this.a)
}
var obj={
    a:2,
    foo:foo
}
var bar=obj.foo;
var a='global a'
bar()//'global a'，bar 不带任何修饰函数，因此应用了默认绑定规则
```
例子2：
```js
function foo(){
    console.log(this.a)
}
function doFoo(fn){
    // fn是引用foo
    fn() //在这里调用，没有任何修饰符 走默认绑定规则
} 
var obj={
    a:2,
    foo:foo
}
var a="global a"
doFoo(obj.foo) //"global a"
```
例子3：
```js
setTimeout(obj.foo,100) //"global a"

//setTimeout函数伪代码 
function setTimeout(fn,delay){
    fn() //在这里调用，没有任何修饰符 走默认绑定规则
}

```
参数传递其实就是一种隐式赋值，因此我们传递参数是函数时也会被隐式赋值。
### 显式绑定
使用call()、apply()方法可以直接指定this的绑定对象，因此称为显式绑定
```js
function foo(){
    console.log(this.a)
}
var obj={
    a:2
}
foo.call(obj) //2
```
**显式绑定无法解决之前提到的丢失绑定的问题**

1. 硬绑定
显式的强制绑定，称之为硬绑定
```js
function bind(fn,obj){
    return function(){
        return fn.apply(obj,arguments)
    }
}
```
ES5中提供了内置方法 Function.prtotype.bind 
2. API 调用的 "上下文"
一些内置方法提供了一个可选参数，被称之为上下文，作用和bind一样确保你的回调函数使用指定的this
```js
function foo(){
    console.log(el,this.id)
}
var obj={
    id:'awesome'
}
[1,2,3].forEach(foo,obj)
//1 awesome 2 awesome 3 awesome
```
### new绑定
```js
// new 源码实现
function newFun(con,...args){
    var obj=new Object();
    obj.__proto__=con.prototype
    const res=con.apply(obj,args)
    return res instanceof Object?res:obj
} 
```
在new的源码实现中可以看到新对象会绑定构造函数的this上  


### 优先级：new>显式绑定>隐式绑定>默认绑定

### 绑定例外
1. 被忽略的this
把null 或 undefined 作为this的绑定对象传入call 、apply 或者 bind ,这些值在调用时会被忽略，实际应用的是默认绑定规则  

2. 间接引用  
创建一个函数的”间接引用“ ，在这种情况下，调用这个函数会应用默认绑定规则。

```js
function foo(){
    console.log(this.a)
}
var a=2;
var o={a:3,foo:foo}
var p={a:4}
o.foo() //3
(p.foo=o.foo)() //2
```
**赋值表达式 p.foo=o.foo的返回值是目标函数的引用，因此相当于调用的是foo()，所以会走默认绑定**

3. 软绑定
硬绑定大大降低了函数的灵活性，使用硬绑定之后无法使用使用隐式绑定或者显示绑定来修改this  
// TODO 待补充...

### 箭头函数（this词法）
箭头函数不使用this的四种绑定规则，箭头函数是没有this，是根据外层作用域来决定this。所以箭头函数的绑定无法被修改

### 题目自测，你真的懂了吗？

```js
var a = 1;
function fn(){
    "use strict"
    console.log(a) 
    var a = 2;
    function fb(){
        console.log(a); 
        console.log(this,'==this')
        console.log(this.a); 
    }
    return fb;
}

fn.call({a: 3 })();
```
<details>
<summary><mark><font color=darkred>点击查看答案及解析</font></mark></summary>
<p>打印结果：undefined、2、1 </p> 
<b>undefined解析：</b>
<p>变量提升，a 提升到函数作用域顶部</p>

<b>2解析：</b>
<p>闭包</p>

<b>1解析：</b>
<p>fn.call({a: 3 }) 返回了 fb引用 ，fn.call({a: 3 })() 等价于 fb()，因为默认绑定在非严格模式下this指向全局变量，严格模式下this 会绑定到undefined，所以非严格模式下 结果为1，严格模式下 TypeError</p>
</details>

```js
let num=20;
let obj={
    num:40,
    init:function(){
        console.log(this.num)
        function p(){
            console.log(this.num) 
        }
        p.prototype.name=function(){
            console.log(this.num)
        }
        return p
    }
} 
let p=obj.init() 
p()
new p()
```

<details>
<summary><mark><font color=darkred>点击查看答案及解析</font></mark></summary>
<div>
<b>let p=obj.init()解析</b>
<p>执行console.log(this.num) this指向obj 所以返回结果为 40</p>

<b>p()解析</b>
<p>执行let p=obj.init()返回p 函数引用,执行p() 等价于直接执行 init内的p函数且使用默认规则this 指向window， let num=20 ；声明全局变量 num ，但是 let 声明的变量不属于顶层对象的属性，因此 num 为undefined </p>

<b>new p()解析</b>
<p>首先我们知道 构造函数中的this 指向实例对象，实例对象 原型链中没有 num ，因此 p函数执行时 this.num 会打印 undefined ,原型方法 name 未被执行</p>

</div>

</details>

### 其他关联知识点 

##### [顶层对象 vs  全局变量](https://es6.ruanyifeng.com/#docs/let#%E9%A1%B6%E5%B1%82%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%B1%9E%E6%80%A7)
> 顶层对象，**在浏览器环境指的是window对象**，在 Node 指的是global对象。顶层对象的属性与全局变量挂钩，被认为是 JavaScript 语言最大的设计败笔之一。ES6 为了改变这一点，一方面规定，为了保持兼容性，**var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性**。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。