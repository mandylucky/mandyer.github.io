---
title: Generator 函数的异步应用
tags: ['异步','ES6']
toc: true
date: 2021-05-16 07:34:04
categories: js
---
# 传统方法
异步编程的方法，大概有下面四种
- 回调函数
- 时间监听  
- 发布/订阅
- Promise 对象  

Generator 函数将JavaScript 异步编程带入了一个全新的阶段

# 基本概念
所谓"异步"，简单说就是一个任务不是连续完成的，可以理解成该任务被人为分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段。
相应地，连续的执行就叫做同步。

### 回调函数
回调函数，就是把任务的第二段单独写在一个函数里面，等到重新执行这个任务的时候，就直接调用这个函数

缺点：回调函数本身并没有问题，它的问题出现在多个回调函数嵌套。代码会横向发展，无法管理。多个异步操作形成了强耦合，只要有一个操作需要修改，它的上层回调函数和下层回调函数，可能都要跟着修改 ，这种情况称为"回调地狱"

```js
fs.readFile(fileA, 'utf-8', function (err, data) {
  fs.readFile(fileB, 'utf-8', function (err, data) {
    // ...
  });
});
```
### Promise
Promise 对象是为了解决"回调地狱"问题而提出的。它不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用。

```js
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function (data) {
  console.log(data.toString());
})
.then(function () {
  return readFile(fileB);
})
.then(function (data) {
  console.log(data.toString());
})
.catch(function (err) {
  console.log(err);
});

```
Promise 提供then方法加载回调函数，catch方法捕捉执行过程中抛出的错误。

Promise 的写法只是回调函数的改进，使用then方法以后，异步任务的两段执行看得更清楚了，除此以外，并无新意。

缺点：Promise 的最大问题是代码冗余，原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆then，原来的语义变得很不清楚。

# Generator 函数
### 协程
多个线程相互协作，完成异步任务（多任务的解决方案）

携程有点像函数，又有点像线程，它的运行流程大致如下 
- 第一步，协程A开始执行
- 第二步，协程A执行到一半，进入暂停，执行权转移到协程B
- 第三步，一段时间后，协程B交还执行权
- 第四步，协程A恢复执行

上面流程的协程A，就是异步任务，因为它分成两段（或多段）执行。

### 协程的 Generator 函数实现 
Generator 函数是协程在ES6的实现，最大特点是可以交出函数的执行权（既暂停执行）

Generator 函数就是一个异步任务的容器，异步操作需要暂停的地方都用yield 语句注明  

```js
function* gen(x) {
  var y = yield x + 2;
  return y;
}

var g = gen(1);
g.next() // { value: 3, done: false }
g.next() // { value: undefined, done: true }
```
next方法的作用是分阶段执行Generator函数。每次调用next方法，会返回一个对象，表示当前阶段的信息（value属性和done属性）

value属性是yield语句后面表达式的值，表示当前阶段的值；done属性是一个布尔值，表示 Generator 函数是否执行完毕，即是否还有下一个阶段。

### Generator 函数的数据交换和错误处理 
<span style="background-color:yellow">Generator 函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因。除此之外，它还有两个特性，使它可以作为异步编程的完整解决方案：函数体内外的数据交换和错误处理机制。</span>

next返回值的 value 属性，是 Generator 函数向外输出数据；next方法还可以接受参数，向 Generator 函数体内输入数据。

Generator 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。
```js
function* gen(x){
  try {
    var y = yield x + 2;
  } catch (e){
    console.log(e);
  }
  return y;
}

var g = gen(1);
g.next();
g.throw('出错了');
// 出错了
```

### 异步任务的封装
```js
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);
  console.log(result.bio);
}
var g = gen();
var result = g.next();

result.value.then(function(data){
  return data.json();
}).then(function(data){
  g.next(data);
});
```
虽然 Generator 函数将异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段）。

# Thunk函数 
Thunk 函数是自动执行Generator 函数的一种方法   

参数的求值策略:传名调用、传值调用【JavaScript 语言是传值调用】
```js
var x = 1;
function f(m) {
  return m * 2;
}
f(x + 5)


f(x + 5)
// 传值调用时，等同于
f(6)

f(x + 5)
// 传名调用时，等同于
(x + 5) * 2
```
### JavaScript 语言的 Thunk 函数 
