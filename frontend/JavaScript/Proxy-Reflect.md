# Proxy与Reflect

## 监听对象

我们在ES6之前，其实是可以通过 `Object.defineProperty()` 实现对象的监听操作。

```js
Object.keys(obj).forEach(key => {
  let value = obj[key]
  Object.defineProperty(obj,key,{
    set(newValue){
      console.log(`监听到${key}的变化`)
      value = newValue
    },
    get(){
      return value
    }
  })
})
```

**但是这样做是有缺点的**：因为 `Object.defineProperty()` 的设计初衷就不是为了监听对象来设计，其次如果我们想要监听更加丰富的操作：新增、删除等，这个方法是无法实现的。

## Proxy

在ES6中，新增了一个 `Proxy` 类，这个类从名字就可以看出来，是用于帮助我们创建一个代理的：

也就是说，如果我们希望 **监听一个对象** 的相关操作，那么我们可以先创建一个代理对象（ `Proxy` 对象）。

之后对该对象的所有操作，都通过代理对象来完成，代理对象可以监听我们想要对原对象进行的操作。

### Proxy实现监听

我们将上面的案例用 `Proxy` 来实现：

```js
const obj = {
  name: 'curry',
  age: 33,
}

// 创建代理对象
// target指的是目标对象，handler指的是包含捕获器的处理器对象
// new Proxy(target,handler)
const objProxy = new Proxy(obj, {})
```

之后我们要操作对象的话，直接操作创建出来的代理对象即可。

```js
const obj = {
  name: 'curry',
  age: 33,
}

const objProxy = new Proxy(obj, {
  get(target, key, recevicer) {
    return target[key]
  },
  set(target, key, newValue, recevicer) {
    target[key] = newValue
  },
  has(target, key, recevicer) {
    return key in target
  },
  deleteProperty(target, key) {
    delete target[key]
  },
})
```

### Proxy的捕获器

`Proxy` 的捕获器一共有13种，常用的捕获器大概有以下几种：

`handler.has()` ：`in` 操作符的捕捉器。
`handler.get()` ：属性读取操作的捕捉器。
`handler.set()` ：属性设置操作的捕捉器。
`handler.deleteProperty()` ：`delete` 操作符的捕捉器。
`handler.apply()` ：函数调用操作的捕捉器。
`handler.construct()` ：`new` 操作符的捕捉器。

## Reflect

`Reflect` 也是ES6新增的一个 `API` ，它是一个对象，字面的意思是反射。

### Reflect的作用

`Reflect` 提供了很多操作对象的方法，这些方法与 `Object` 中操作对象的方法类似。

既然有了 `Object` 中的方法可以做这些操作，**为什么需要新增 `Reflect` 对象呢？**

这是因为在早期的 `ECMA规范` 没有考虑到这种对 **对象的操作** 如何设计会更加规范，所以将这些 `API` 放在了 `Object` 上。

但是 `Object` 作为构造函数，使用这些操作并不合适。

因此才新增了 `Reflect` 对象。

### Reflect中的方法

`Reflect` 中的很多方法都与 `Object` 中的方法类似，而且它的所有方法与 `Proxy` 的13种捕获器一一对应。

### Reflect的使用

我们可以将 `Proxy` 中的捕获器中对对象的操作从 `Object` 中的方法修改为 `Reflect` 中的方法。

```js
const obj = {
  name: 'curry',
  age: 33,
}

const objProxy = new Proxy(obj, {
  get(target, key) {
    // return target[key]
    return Reflect.get(target,key)
  },
  set(target, key, newValue) {
    // target[key] = newValue
    Reflect.set(target,key,newValue)
  },
  has(target, key) {
    // return key in target
    return Reflect.has(target,key)
  },
  deleteProperty(target, key) {
    // delete target[key]
    Reflect.deleteProperty(target,key)
  },
})
```

### Proxy与Reflect中的Receiver的作用

如果我们的源对象有setter、getter的访问器属性，那么可以通过 `receiver` 来改变里面的 `this` ，将 `this` 指向了 `proxy` 对象。

```js
const obj = {
  _name: 'curry',
  age: 33,
  get name(){
    console.log(this)
    return this._name
  },
  set name(newValue){
    console.log(this)
    this._name = newValue
  }
}

const objProxy = new Proxy(obj, {
  // receiver是创建出来的代理对象
  get(target, key, receiver) {
    // return target[key]
    console.log('getter')
    // 此时 obj里面的getter中的this指向的是 objProxy对象
    return Reflect.get(target, key, receiver)
  },
  set(target, key, newValue, receiver) {
    // target[key] = newValue
    console.log('setter')
    Reflect.set(target, key, newValue, receiver)
  },
  has(target, key) {
    // return key in target
    return Reflect.has(target, key)
  },
  deleteProperty(target, key) {
    // delete target[key]
    Reflect.deleteProperty(target, key)
  },
})

objProxy.name = 'james'
```

### Reflect中的construct

利用 `Reflect.construct(target, argumentsList, newTarget)` 方法，可以实现通过一个函数作为构造函数，创建出另外一个类型的操作。

```js
function Person(name,age){
  this.name = name
  this.age = age
}

function Coder(){

}

const obj = Reflect.construct(Person,['curry',33], Coder)
console.log(obj.__proto__ == Coder.prototype) // true
```

## 总结

`Proxy` 与 `Reflect` 搭配实现的响应式相比以前的 `Object.defineProperty()` 来说更为强大。

在 `Vue3` 中实现响应式是通过 `Proxy` 与 `Reflect` 来实现的。后面的章节中我们也会自己模拟响应式的实现。
