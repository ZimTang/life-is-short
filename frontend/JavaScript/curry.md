
## 什么是函数柯里化

函数柯里化是函数式编程的一个重要概念，它可以提高函数使用的灵活度，增加参数的复用

函数柯里化的定义是一种将使用多个参数的一个函数转换成一系列使用一个参数的函数的技术。

举个例子

```js
function sum(a,b,c){
  console.log(a + b + c)
}
```

sum是一个简单的累加函数，将所有形参加起来。在未实现函数柯里化前，我们只能通过固定的传三个参数来调用sum函数，而实现函数柯里化后我们就可以这样:```sum(1,2)(3)```不再是固定的传参数来调用函数

## 函数柯里化的实现

通过```sum(1,2)(3)```这种调用方式我们能够想到，柯里化函数一定是有递归调用的，且返回值一定是个函数，因此我们可以通过判断实参与形参的个数来判断是否继续递归调用

```js
function curry(fn,curryArgs){
  return function(){
    // arguments是类数组 通过Array.prototype.slice.call方法将arguments转换为数组
    let args = Array.prototype.slice.call(arguments);
    // 判断是否提供参数，若没有提供，就不用进行数组拼接
    if(curryArgs !== undefined){
      args = args.concat(curryArgs)
    }
    // 如果参数的个数小于源函数fn的形参个数，递归调用
    if(args.length < fn.length){
      return curry(fn,args)
    }
    // 递归出口
    return fn.apply(null,args);
  }
}
```

实现柯里化函数后，我们来看看结果

```js
function sum(a,b,c){
  console.log(a + b + c)
}

sum(1,2)(3); // 6
sum(1)(2,3); // 6
sum(1,2,3) // 6
```

但是此时如果我们还要实现不限制参数的个数的话，就需要延迟执行

```js
function sum() {
  // 进行数组转换
  let args = Array.prototype.slice.call(arguments);
  let inner = function () {
    args.push(...arguments)
    // 重写此函数的toString方法
    inner.toString = function () {
      return args.reduce((pre, cur) => {
        return pre + cur
      }, 0)
    }
    return inner;
  }
  return inner;
}

sum(1,2)(3)(4) // 10
```

## 函数柯里化的用途

1. 参数复用

```js
function url(protocol, hostname, pathname) {
  return `${protocol}${hostname}${pathname}`
}
```

在输入网址时，我们需要一直输入protocol也就是协议，通过函数柯里化省去这个过程，此时也就是实现了参数的复用

```js
function url_curry(protocol) {
  return function (hostname, pathname) {
    return `${protocol}${hostname}${pathname}`
  }
}

const url_http = url_curry("http://");
const url1 = url_http("www.baidu.com", "/hello")
const url2 = url_http("www.baidu.com", "/world")
```

2. 提前确认/提前返回
  
在进行兼容性检测的时候，需要判断addEventListener和attachEvent方法

```js
const whichEvent = (function () {
  if (window.addEventListener) {
    // element是绑定的元素，type是判定的事件类型，listener是绑定的事件的回调，useCapture是事件捕获还是事件冒泡
    return function (element, type, listener, useCapture) {
      element.addEventListener(type, function (e) {
        listener.call(element, e);
      }, useCapture)
    }
  } else if (window.attachEvent) {
    return function (element, type, handler) {
      element.attachEvent("on" + type, function (e) {
        listener.call(element, e)
      }, useCapture)
    }
  }
})()
const btn = document.querySelector("button")

whichEvent(btn, "click", function () { 
  console.log("hello") 
})
```