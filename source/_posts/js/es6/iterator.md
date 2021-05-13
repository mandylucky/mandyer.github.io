---
title: Iterator
tags: ['ES6']
toc: true
date: 2021-05-13 08:35:38
categories: js
---
# 概念
JavaScript 有四种数据集合：Array、Object、Map、Set ,我们可以组合使用它们定义数据结构，这样就需要一种统一的接口机制来处理不同的数据结构

**遍历器（Iterator）：它是一个接口，为各种不同的数据结构提供统一的访问机制，任何数据结构只要部署了Iterator 接口，就可以完成遍历操作**

###  Iterator 作用
1. 为不同数据结构提供统一的访问接口
2. 使数据结构的成员能够按某种次序排序
3. ES6 新的遍历方式for...of，Iterator 接口主要提供for .. of 消费

###  Iterator的遍历过程
1. 创建一个指针对象，指向当前数据结构的起始位置 （遍历器的本质就是一个指针对象）
2. 第一次调用next方法，指针指向数据结构的第一个成员
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

ES6有些数据结构原生具备Iterator 接口（这些数据结构原生部署了Symbol.iterator 属性），*凡是部署了Symbol.iterator属性的数据结构，就称为部署了遍历器接口*

### 原生具备 Iterator 接口的数据结构如
- Array
- Map
- Set
- String
- arguments 对象
- NodeList 对象

### 调用Iterator 接口的场合 
1. 解构赋值 
2. 扩展运算符
3. yield* 
4. for...of 、Array.from()、Map() 、Set()、WeakMap()、WeakSet()、Promise.all()、Promise.race()

### 字符串的Iterator 接口