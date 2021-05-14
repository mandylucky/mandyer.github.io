---
title: Generator 生成器
tags: ['ES6']
toc: true
date: 2021-05-14 07:00:00
categories: js
---
# 概述
###  基础概念

Generator 函数有多种理解角度
- 语法上，Generator 函数是一个**状态机**，封装了多个内部状态 
- 执行Generator 函数会返回一个遍历器对象，所有Generator 也是一个**遍历器对象生成函数**，返回遍历器对象
- 形式上，Generator 函数是一个普通函数，有两个特征 一个是function 关键字和函数名直接有一个星号，二是，函数体内部使用yield（产出） 表达式，定义不同的庄爱


```js
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}
var hw = helloWorldGenerator();
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

遇到yield 表达式就会暂停，下一次调用会从上一次暂停的地方一直执行到下一个yield/return ,每次调用都会返回```{value:xxx,done:true/false}```

###  yield 表达式
**由于 Generator 函数返回的遍历器对象，只有调用next方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。yield表达式就是暂停标志。**

**yield表达式只能用在 Generator 函数里面,用在其他地方都会报错**


**遍历器对象的next方法运行逻辑：** 
1. 遇到yild 表达式，暂停执行后面操作，yield 表达式的值作为返回对象的value属性值 
2. 下一次调用next方法时，再继续往下执行，直到遇到下一个yield表达式。
3. 如果没有再遇到新的yield表达式，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值
4. 如果该函数没有return语句，则返回的对象的value属性值为undefined。

**Generator 不用yield 表达式，变成了一个单纯的暂缓执行函数**
```js
function * f(){
    console.log('执行')
}
var generator = f();

setTimeout(function () {
  generator.next()
}, 2000);
```
函数f是一个 Generator 函数，就变成只有调用next方法时，函数f才会执行。

###  与Iterator 接口的关系
- Symbol.iterator方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。
- 由于 Generator 函数就是遍历器生成函数，因此可以把 Generator 赋值给对象的Symbol.iterator属性，从而使得该对象具有 Iterator 接口。
- Generator 函数执行后，返回一个遍历器对象。该对象本身也具有Symbol.iterator属性，执行后返回自身。

```js
function* gen(){
  // some code
}
var g = gen();
g[Symbol.iterator]() === g
```

#  next 方法的参数
yield表达式本身没有返回值，或者说总是返回undefined。   
**next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值**。

# for...of 循环 
for...of循环可以自动遍历 Generator 函数运行时生成的Iterator对象，且此时不再需要调用next方法。

```js
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

# Generator.prototype.throw() 
**Generator 函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。**

```js
var g=function * (){
    try{
        yield
    }catch(e){
        console.log('内部捕获',e)
    }
}
var i=g()
try{
    i.throw('a')
    i.throw('b');
}catche(e){
    console.log('外部捕获'，e)
}
// 内部捕获 a
// 外部捕获 b
```
第一个错误被 Generator 函数体内的catch语句捕获。i第二次抛出错误，由于 Generator 函数内部的catch语句已经执行过了，不会再捕捉到这个错误了，所以这个错误就被抛出了 Generator 函数体，被函数体外的catch语句捕获。

如果 Generator 函数内部没有部署try...catch代码块，那么throw方法抛出的错误，将被外部try...catch代码块捕获。

# Generator.prototype.return()   

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()  
```

### try...finally 代码块
正在执行try代码块，那么return()方法会导致立刻进入finally代码块，然后等到finally代码块执行完，再返回return()方法指定的返回值
```js
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers();
g.next() // { value: 1, done: false }
g.next() // { value: 2, done: false }
g.return(7) // { value: 4, done: false }
g.next() // { value: 5, done: false }
g.next() // { value: 7, done: true }
```

# next()、throw()、return() 的共同点
next()、throw()、return()这三个方法本质上是同一件事，可以放在一起理解。它们的作用都是让 Generator 函数恢复执行，并且使用不同的语句替换yield表达式。

- next()是将yield表达式替换成一个值。
- throw()是将yield表达式替换成一个throw语句。
- return()是将yield表达式替换成一个return语句。

# yield* 表达式
**yield表达式后面跟的是一个遍历器对象，需要在yield表达式后面加上星号，表明它返回的是一个遍历器对象。这被称为yield*表达式。**

### 使用场景
Generator 函数嵌套，写起来就非常麻烦。
eg:foo 和 bar 都是generator函数，在bar 里面调用foo,就需要手动遍历foo (使用：for ... of)

```js
function* foo() {
  yield 'a';
  yield 'b';
}

function* bar() {
  yield 'x';
  // 手动遍历 foo()
  for (let i of foo()) {
    console.log(i);
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// x
// a
// b
// y
```

**ES6 提供了yield* 表达式解决Generator 函数嵌套问题**
```js
function* foo() {
  yield 'a';
  yield 'b';
}
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

```
# 作为对象属性的 Generator 函数
```js
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
// 等价于
let obj = {
  * myGeneratorMethod() {
    ···
  }
};

```
myGeneratorMethod属性前面有一个星号，表示这个属性是一个 Generator 函数。
# Generator 函数的this
Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的prototype对象上的方法。

generator 函数不能作为构造函数，也不能使用 new调用,有没有办法让 Generator 函数返回一个正常的对象实例，既可以用next方法，又可以获得正常的this？

```js
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function F() {
  return gen.call(gen.prototype);
}

var f = new F();

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```
# 含义
### Generator 与状态机 
**Generator 是实现状态机的最佳结构**

例子：clock函数一共有两种状态（Tick和Tock），每运行一次，就改变一次状态
```js
var ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}

// 等价于

var clock = function* () {
  while (true) {
    console.log('Tick!');
    yield;
    console.log('Tock!');
    yield;
  }
};
```
### Generator 与协程
协程（coroutine）是一种程序运行的方式，可以理解成"协作的函数"

Generator 函数是 ES6 对协程的实现，但属于不完全实现。Generator 函数被称为“半协程”（semi-coroutine），意思是只有 Generator 函数的调用者，才能将程序的执行权还给 Generator 函数。如果是完全执行的协程，任何函数都可以让暂停的协程继续执行。


如果将 Generator 函数当作协程，完全可以将多个需要互相协作的任务写成 Generator 函数，它们之间使用yield表达式交换控制权。

### Generator 与上下文
Generator 函数执行产生的上下文环境，一旦遇到yield命令，就会暂时退出堆栈，但是并不消失，里面的所有变量和对象会冻结在当前状态。等到对它执行next命令时，这个上下文环境又会重新加入调用栈，冻结的变量和对象恢复执行

# 应用 
Generator 可以暂停函数执行，返回任意表达式的值。这种特点使得 Generator 有多种应用场景。

1. 异步操作的同步化表达
2. 控制流管理
3. 部署 Iterator 接口
4. 作为数据结构

# 使用例子
### 实现斐波那契数列
```js
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
```

### 原生对象添加遍历器接口
```
function* objectEntries(obj) {
  let propKeys = Reflect.ownKeys(obj);

  for (let propKey of propKeys) {
    yield [propKey, obj[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

for (let [key, value] of objectEntries(jane)) {
  console.log(`${key}: ${value}`);
}
```