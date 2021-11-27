## call 与 apply 共同点

* 都能够改变函数执行时的上下文，将一个对象的方法交给另一个对象来执行，并且是立即执行的
* 比如，A 对象有一个方法，而 B 对象因为某种原因，也需要用到同样的方法，那么这时候我们可以借用一下 A 对象的方法，既完成了需求，又减少了内存的占用。
* 调用 call 和 apply 的对象，必须是一个函数 Function
* call 和 apply 的第一个参数是一个对象。Function 的调用者，将会指向这个对象，如果不传，默认情况下是window

## call 与 apply 的区别

### call 的写法

```js
Function.call(obj,[param1[,param2[,…[,paramN]]]])
```

**注意：**

* 调用call的对象，必须是给函数 Function
* 第二个参数开始，可以接收多个参数。每个参数会映射到相应位置的 Function 的参数上

### apply 的写法

```js
Function.apply(obj[,argArray])
```

**注意：**

* 第二个参数必须是数组或者是类数组，它们会被转换成类数组，传入 Function 中，并且会映射到相应位置的 Function 的参数上，这也是 call 和 apply 之间，很重要的区别

```js
fn.apply(obj, [1,2,3])
// fn 接收到的参数是 1,2,3

fn.apply(obj, {
    0: 1,
    1: 2,
    2: 3,
    length: 3
})
// fn 接收到的参数是 1,2,3
```

## call 与 apply的使用

### call 的使用

1. 对象的继承

```js
function Animal(){
  this.species = "动物"
}

function Cat(name,color){
  Animal.call(this);
  this.name = name;
  this.age = age;
}
```

> Cat 通过 call 方法，继承了 Animal 的 species 属性

2. 借用方法

```js
let args = Array.prototype.slice.call(arguments);
```

> 将 arguments 对象转换成数组

### apply的使用

1. 获取数组最大最小值

```js
let max = Math.max.call(null,arr);
let min = Math.min.call(null,arr);
```

2. ES5 前实现数组合并

```js
let arr1 = [1,2,3];
let arr2 = [4,5,6];
Array.prototype.push(arr1,arr2);
```

> ES6后可以使用 spread 运算符实现数组合并

## bind 的使用

### bind 的写法

```js
Function.bind(thisArg[, arg1[, arg2[, ...]]])
```

bind 方法的返回值是函数，需要调用才会执行，而 call 和 apply 是立即调用

```js
function add (a, b) {
    return a + b;
}

add.bind(null, 5, 3); // 这时，并不会返回 8
add.bind(null, 5, 3)(); // 调用后，返回 8
```

## 总结

call 和 apply 的主要作用，是改变对象的执行上下文，并且是立即执行的。它们在参数上的写法略有区别。

bind 也能改变对象的执行上下文，它与 call 和 apply 不同的是，返回值是一个函数，并且需要稍后再调用一下，才会执行。