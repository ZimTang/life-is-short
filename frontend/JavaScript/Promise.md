## Promise是做什么的

* Promise是异步编程的解决方案
* 当网络请求非常复杂时，就会出现回调地狱
<!--more-->
## 一个例子

```js
$.ajax("url1", function (data1) {
        $.ajax(data1["url2"], function (data2) {
          $.ajax(data2["url3"], function (data3) {
            $.ajax(data3["url3"], function (data4) {
              console.log(data4);
            });
          });
        });
      });
```

* 上面这种情况，正常时不会有什么问题的
* 但是这样的代码难看且不易维护
* 我们更加期望的是一种更加优雅的方式来进行异步操作
* Promise可以以一种非常优雅的方式来解决这个问题

## Promise基础

### Promise 的状态

实例对象中的一个属性 [PromiseState]

* pending 未决定的
* resolved / fulfilled 成功
* rejected 失败

### Promise 对象的值

实例对象中的另一个属性 [PromiseResult]

保存着对象【成功/失败】的结果

* resolve
* reject

### Promise的API

#### resolve

* 如果传入的参数为 非Promise类型的对象 则返回的结果为成功的promise对象
* 如果传入的参数为 Promise 对象，则参数的结果决定了 resolve 的结果

```js
let p1 = Promise.resolve(111)
let p2 = Promise.resolve(new Promise((resolve, reject) => {
  // resolve("ok")
  reject("error")
}))
console.log(p2)
```

#### reject

返回的结果永远是失败的(rejected)

```js
let p = Promise.reject(111)
let p2 = Promise.reject("123")
let p3 = Promise.reject(new Promise((resolve, reject) => {
  resolve("ok")
}))
console.log(p3)
```

#### all

接收一个包含promise的数组 只要有一个为rejected 返回的结果就为rejected

```js
let p1 = new Promise((resolve, reject) => {
  resolve("OK")
})
let p2 = Promise.resolve("success")
let p3 = Promise.resolve("oh yeah")
const res = Promise.all([p1, p2, p3])
console.log(res)
```

#### race

接收一个包含promise的数组 返回一个新的promise 第一个完成的promise的结果状态就是最终的结果状态

```js
let p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("OK")
  }, 1000);
})
let p2 = Promise.resolve("success")
const res = Promise.race([p1, p2, p3])
console.log(res)
```

## Promise关键问题

### 状态修改

有三种方法能够修改promise实例对象的状态

1. resolve函数
2. reject函数
3. 抛出错误

```js
let p = new Promise((resolve, reject) => {
   // 1.resolve函数
   // resolve("ok")
   // 2.reject函数
   // reject("error")
   // 3.抛出错误
   throw "error"
})
console.log(p)
```

### 能够执行多个回调

当promise改变为对应状态时对应的回调函数都会调用

```js
 let p = new Promise((resolve, reject) => {
   resolve("ok")
 })
 p.then(value => {
   console.log(value)
 })
 p.then(value => {
   alert(value)
 })
```

### 改变状态与指定回调的顺序执行问题

1. 当执行器里面的任务为同步任务时 先该变promise对应的状态 再指定回调
2. 当执行器里面的任务为异步任务时 先指定回调 再改变状态
3. 指定不是执行

```js
 let p = new Promise((resolve, reject) => {
   setTimeout(() => {
     resolve("ok")
   }, 1000);
 })
 p.then(value => {
   console.log(value)
 }, reason => {
   console.log(reason)
 })
```

```then```是微任务 定时器是宏任务 所以宏任务执行完后先去执行```then```，也就是指定了```promise```改变状态时候的回调函数，然后再执行定时器，执行了```resolve("ok")```，触发```then```里面的回调

### then方法返回由什么决定

1. 抛出错误 返回的为rejected状态
2. 返回结果是非promise类型的对象，返回的为fulfilled状态
3. 返回结果是一个promise对象，返回的状态取决于promise对象的状态

```js
 let p = new Promise((resolve, reject) => {
   resolve("ok")
 })

 // then方法返回的是一个promise对象
 // result是一个promise对象
 let result = p.then(value => {
   // throw "error"
   // return 111;
   return new Promise((resolve, reject) => {
     resolve("ok")
   })
 })
 console.log(result)
```

### Promise如何串联多个任务

使用链式调用

```js
 let p = new Promise((resolve, reject) => {
   setTimeout(() => {
     resolve("ok")
   }, 1000);
 })

 p.then(value => {
   return new Promise((resolve, reject) => {
     resolve("success")
   })
 }).then(value => {
   console.log(value)
 }).then(value => {
   console.log(value)
 })
```

### Promise中的异常传透

当使用```promise```的```then```链式调用时，可以在最后指定失败的回调前面任何操作出了异常，都会传到最后失败的回调中处理

```js
 let p = new Promise((resolve, reject) => {
   setTimeout(() => {
     resolve("ok")
     // reject("error")
   }, 1000);
 })
 p.then(value => {
   // console.log(111)
   throw "error"
 }).then(value => {
   console.log(222)
 }).then(value => {
   console.log(333)
 }).catch(reason => {
   console.log(reason)
 })
```

### 如何中断promise链

有且只有一种方式，返回一个padding状态的promise

```js
 p.then(value => {
   console.log(111)
   return new Promise(() => { })
 }).then(value => {
   console.log(222)
 }).then(value => {
   console.log(333)
 }).catch(reason => {
   console.log(reason)
 })
```

## aysnc与await

### async函数

1. 函数的返回值是一个promise对象
2. promise对象的结果由async函数执行的返回值决定

```js
 const fn = async function () {
   return new Promise((resolve, reject) => {
     resolve()
   })
 }
 console.log(fn())
```

### await表达式

1. await右侧的表达式一般为promise对象，但也可以是其它的值
2. 如果表达式是promise对象，await返回的是promise成功的值
3. 如果await表达式返回的promise是失败的状态，则需要使用```try catch```捕获失败
4. 如果表达式是其它值，直接将此值作为await的返回值

```js
 const fn = async () => {
   try {
     let n = await 123
     console.log(n)//123
     let p = await new Promise((resolve, reject) => {
       // resolve("ok")
       reject("error")
     })
   } catch (error) {
     console.log(error)
   }
 }
 fn()
```

## 手写Promise

```js
// 声明构造函数
function Promise(executor) {
  // 状态
  this.PromiseState = "pending"
  // 结果值
  this.PromiseResult = null
  // 保存this值
  const self = this
  this.callbacks = []
  // resolve函数
  function resolve(data) {
    // promise的状态只能被修改一次
    if (self.PromiseState !== "pending")
      return
    self.PromiseState = "fulfilled"
    self.PromiseResult = data
    // 调用成功的回调
    setTimeout(() => {
      self.callbacks.forEach(item => {
        item.onResolved(data)
      })
    });
  }

  // reject函数
  function reject(data) {
    // promise的状态只能被修改一次
    if (self.PromiseState !== "pending")
      return
    self.PromiseState = "rejected"
    self.PromiseResult = data
    // 调用成功的回调
    setTimeout(() => {
      self.callbacks.forEach(item => {
        item.onRejected(data)
      })
    })
  }

  try {
    // 同步调用执行器函数
    executor(resolve, reject)
  } catch (error) {
    // promise的状态只能被修改一次
    if (this.PromiseState !== "pending")
      return
    this.PromiseState = "rejected"
    this.PromiseResult = error
  }
}

// 添加then方法
Promise.prototype.then = function (onResolved, onRejected) {
  const self = this
  // 判断回调函数的参数
  // 保证异常传透
  if (typeof onRejected !== "function") {
    onRejected = reason => {
      throw reason
    }
  }
  // 保证值传递
  if (typeof onResolved !== "function") {
    onResolved = value => value
  }
  return new Promise((resolve, reject) => {
    // 封装函数
    function callback(type) {
      try {
        let result = type(self.PromiseResult)
        if (result instanceof Promise) {
          result.then(v => {
            resolve(v)
          }, r => {
            reject(r)
          })
        } else {
          resolve(result)
        }
      } catch (error) {
        reject(error)
      }
    }

    // promise实例对象状态为fulfilled
    if (this.PromiseState === "fulfilled") {
      setTimeout(() => {
        callback(onResolved)
      });
    }
    // promise实例对象状态为rejected
    if (this.PromiseState === "rejected") {
      setTimeout(() => {
        callback(onRejected)
      });
    }
    // promise实例对象状态为pending
    if (this.PromiseState === "pending") {
      // 保存then中传入的回调函数
      this.callbacks.push({
        onResolved: function () {
          callback(onResolved)
        },
        onRejected: function () {
          callback(onRejected)
        }
      })
    }
  }
  )
}

// 添加catch方法
Promise.prototype.catch = function (onRejected) {
  return this.then(undefined, onRejected)
}

// 添加Promise.resolve方法
Promise.resolve = function (value) {
  return new Promise((resolve, reject) => {
    if (value instanceof Promise) {
      value.then(v => {
        resolve(v)
      }, r => {
        reject(r)
      })
    } else {
      resolve(value)
    }
  })
}

// 添加Promise.reject方法
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason)
  })
}

// 添加Promise.all方法
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let count = 0
    let arr = []
    promises.forEach((item, index) => {
      item.then(v => {
        count++
        arr[index] = v
        if (count === promises.length) {
          resolve(arr)
        }
      }, r => {
        reject(r)
      })
    })
  })
}

// 添加Promise.race方法
Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    promises.forEach((item) => {
      item.then(v => {
        resolve(v)
      }, r => {
        reject(r)
      })
    })
  })
}

```