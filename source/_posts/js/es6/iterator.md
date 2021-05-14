---
title: Iterator
tags: ['ES6']
toc: true
date: 2021-05-13 08:35:38
categories: js
---
# 概念
JavaScript 有四种数据集合：Array、Object、Map、Set ,我们可以组合使用它们定义数据结构，这样就需要一种统一的接口机制来处理不同的数据结构

**<span style="background-color:yellow">遍历器（Iterator）：它是一个接口，为各种不同的数据结构提供统一的访问机制，任何数据结构只要部署了Iterator 接口，就可以完成遍历操作</span>**

###  Iterator 作用
1. 为不同数据结构提供统一的访问接口
2. 使数据结构的成员能够按某种次序排序
3. ES6 新的遍历方式for...of，Iterator 接口主要提供for .. of 消费

###  Iterator的遍历过程
1. 创建一个指针对象，指向当前数据结构的起始位置 （<span style="background-color:yellow">遍历器的本质就是一个指针对象</span>)
3. 第二次调用next方法，指针指向数据结构的第二个成员
4. 不断调用指针对的next 方法，直到指向数据结构的结束位置

###  模拟遍历器生成函数

```js
function makeIterator(array){
    var nextIndex=0;
    return {
        next:function(){
            return nextIndex<array.length?
            {value:array[nextIndex++]} :
            {done:true}
        }
    }
}
```
**遍历器与数据结构是分离的**

# 默认 Iterator 接口
- **Iterator 接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即for...of循环。当使用for...of循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口。**

- **一种数据结构只要部署了 Iterator 接口，我们就称这种数据结构是“可遍历的”**

- **ES6 规定，默认的 Iterator 接口部署在数据结构的Symbol.iterator属性,也就是说一个数据结构只要有Symbol.iterator 数据就可以认为是可遍历的**

<span style="background-color:yellow">**Symbol.iterator属性本身是一个函数，就是当前数据结构默认的【遍历器生成函数】。执行这个函数，就会返回一个遍历器**</span>

### 原生具备 Iterator 接口的数据结构如
- Array
- Map
- Set
- String
- arguments 对象
- NodeList 对象

### 调用Iterator 接口的场合
1. 解构赋值  会默认调用Symbol.iterator 方法
2. 扩展运算符 也会默认调用Iterator 接口
3. yield* 后面跟的是一个可遍历结构，它也会调用该结构的遍历器接口。
4. for...of 、Array.from()、Map() 、Set()、WeakMap()、WeakSet()、Promise.all()、Promise.race()

### 字符串的Iterator 接口
字符串也原生具有Iterator 接口
```js
var someStr="hi";
typeof someStr[Symbol.iterator]
var iterator =someStr[Symbol.iterator]()
iterator.next(); // {value:"h",done:false}
iterator.next(); // {value:"i",done:false}
iterator.next(); // {value:undefined,done:true}
```
Symbol.iterator方法返回一个遍历器对象，在遍历器对象上可以调用next方法，实现对于字符串的遍历。也可以覆盖Symbol.iterator 方法，达到修改遍历器行为的目的

### 遍历器对象的return(),throw()
遍历器对象除了具有next()方法，还具有return()、throw()方法。自定义的遍历器对象必须部署next()方法，return()和throw()方法是否部署是可选择的

**return()方法:break 或者 出错  会触发执行return()方法**
```js
function readLinesSync(file) {
  return {
    [Symbol.iterator]() {
      return {
        next() {
          return { done: false };
        },
        return() {
          file.close();
          return { done: true };
        }
      };
    },
  };
}
```
**throw()方法主要配合Generator 函数使用**

### for...of 循环
**一个数据结构只要部署了Symbol.iterator 属性，就被是为具有iterator 接口，就可以用  for...of 循环遍历它的成员。<span style="background-color:yellow">也就是说for...of 循环内部调用的是数据结构的Symbol.iterator 方法</span>**

**for...in循环读取键名，for...of循环读取键值。**
```js
var arr = ['a', 'b', 'c', 'd'];

for (let a in arr) {
  console.log(a); // 0 1 2 3
}

for (let a of arr) {
  console.log(a); // a b c d
}
```
**计算生成的数据结构**
有些数据结构是在现有数据结构的基础上，计算生成的。eg:数组、Set、Map 都部署了以下三个方法，调用后都返回遍历器对象
- entries() 返回一个遍历器对象，用来遍历[键名，键值]组成的数组，数组的键名就是索引，Set 的键名，键值相同，Map结构的Iterator 结构默认就是调用entries方法

- keys() 返回一个遍历器对象，用来遍历所有的键名
- values() 返回一个遍历器对象，用来遍历所有的键值

**与其他遍历语法的比较**
1. for循环:写法麻烦 
```js
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```

2. 数组内置方法forEach:无法中途跳出forEach循环，break或者return 命令不能奏效
```js
myArray.forEach(function (value) {
  console.log(value);
});
```

3. for...in循环：遍历键名。
缺点
- 数组的键名是数字，但是for...in 循环是以字符串作为键名
- 会遍历原型链上的键。
- 以任意顺序遍历
总之，for...in循环主要是为遍历对象而设计的，不适用于遍历数组。

- for ...of 循环
优点
- 有着同for...in一样的简洁语法，但是没有for...in那些缺点
- 它可以与break、continue和return配合使用
- 提供了遍历所有数据结构的统一操作接口。

# 相关链接
[阮一峰ES6](https://es6.ruanyifeng.com/#docs/iterator)

