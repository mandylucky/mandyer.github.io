---
title: promise
tags: ['ES6']
toc: true
date: 2021-05-21 07:43:56
categories: js
---
使用例子
```js 
const promise = new Promise(function(resolve, reject) {
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});

promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

```js
const PENDING='PENDING'
const REJECTED='REJECTED'
const FULFILLED='FULFILLED'
class MyPromise{
    // constructor:初始化数据，调用传入函数
    constructor(handler){
        if(typeof handler !=='function'){
            throw new Error('handler not a func ')
        }
        this._status=PENDING;
        this._value=undefined;
        this._fullfillQueue=[];
        this._rejectQueue=[];
        try{
            handler(this._resolve.bind(this),this._reject.bind(this))
        }catch(err){
            this._reject(err)
        }
    }
    // _resolve:只有状态为pending 时才执行，修改状态、修改值、执行并清空_fullfillQueue、_rejectQueue中函数。 另外要判断value 是不是promise 实例做不同处理
    _resolve(value){
        if(this._status !==PENDING) return;
        const runFulfilled=(value)=>{
            let cb;
            while(this._fullfillQueue.length){
                cb=this._fullfillQueue.shift()
                cb(value)
            }

        }
        const runRejected=(value)=>{
             let cb;
            while(this._rejectQueue.length){
                cb=this._rejectQueue.shift()
                cb(value)
            }
        }
        if(value instanceof MyPromise){
            value.then(res=>{
                this.value=res;
                this._status=FULFILLED;
                runFulfilled(res)
            },err=>{
                this.value=err;
                this._status=REJECTED;
                runRejected(err)
            })
        }else{
            this.value=value;
            this._status=FULFILLED;
            runFulfilled(value)
        }
    }
    _reject(err){
        if(this._status !==PENDING) return
        this._status
    }
}
```