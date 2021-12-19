# 迭代器与生成器

## 什么是迭代器

**迭代器**是指确使用户可在容器对象（container，例如链表或数组）上遍访的对象，使用该接口无需关心对象的内部实现细节。

在很多编程语言里，都有自己的迭代器。JavaScript也不例外。在JavaScript中，迭代器是一个对象，它需要符合一种协议，这种协议叫做**迭代协议**。

这个对象中需要有一个叫做 `next` 的函数，这个函数**必须有以下要求**：

1. 它的参数是一个或者没有。
2. 它的返回值是一个包含 `done` 属性 和 `value` 属性的对象。
3. `done` 属性是一个布尔值，如果迭代器可以产生序列中的下一个值，则为 `false` 。如果迭代器已将序列迭代完毕，则为 `true` 。这种情况下，`value` 是可选的，如果它依然存在，即为迭代结束之后的默认返回值。
4. `value` 属性表示迭代器返回的值。

## 编写一个迭代器

下面这个例子，我们来编写一个自己的迭代器。

```js
const names = ['curry', 'james', 'harden']

function createNamesIterator(names){
  let index = 0
  return {
    next:() => {
      if (index < names.length) return { done: false, value: names[index++] }
      else return { done: true, value: undefined }
    }
  }
}

const namesIterator = createNamesIterator(names)

console.log(namesIterator.next())
console.log(namesIterator.next())
console.log(namesIterator.next())
console.log(namesIterator.next())
```

## 可迭代对象

**可迭代对象和迭代器是不同的概念**，当一个对象实现了可迭代协议的时候，那么它就是一个可迭代对象。

可迭代对象要求必须实现 `@@iterator` 方法，这意味着对象（或者它原型链上的某个对象）必须有一个键为 `@@iterator` 的属性，可通过常量 `Symbol.iterator` 访问该属性。

> 当一个对象变成可迭代对象的时候，可以进行某些迭代操作。比如 `for of` 操作，就是调用 `@@iterator` 方法。

```js
const iterableObj = {
  names: ['curry', 'james', 'harden'],
  [Symbol.iterator]: function () {
    let index = 0
    return {
      next: () => {
        if (index < this.names.length)
          return { done: false, value: this.names[index++] }
        else return { done: true, value: undefined }
      },
    }
  },
}

for (const item of iterableObj) {
    console.log(item)
}
```

## 内置的可迭代对象

JavaScript的很多内置对象都实现了可迭代协议，会生成一个迭代器对象：

包括： `String` 、`Array` 、`Map` 、`Set` 、`arguments对象` 、`NodeList集合` 。

## 可迭代对象的应用场景

可迭代对象在很多地方都被使用：

1. `for of`

```js
// 上一个例子
for (const item of iterableObj) {
    console.log(item)
}
```

2. 展开语法

```js
const arr1 = [1, 2, 3]
const arr2 = [4, 5, 6]
const arr3 = [...arr1, ...arr2]
```

> 注意：对象的展开运算符并不是使用的迭代器，而是ES9新增的特性。

3. 解构语法

```js
const [name1, name2] = names
```

> 注意：对象的解构并不是使用的迭代器，而是ES9新增的特性。

4. 创建其他对象

例如 `Map` 、 `Set` 、`Promise.all` 中，我们传入的参数必须是一个可迭代对象。

```js
const set1 = new Set(iterableObj)
const set2 = new Set(names)

const arr1 = Array.from(iterableObj)

// 5.Promise.all
Promise.all(iterableObj).then(res => {
  console.log(res)
})
```

## 自定义类的迭代

如果我们要设计一个类，并且默认这个类创建处理的是可迭代对象，那么我们在原型上需要加上 `@@iterator` 方法。

```js
class Team {
  constructor(name, players) {
    this.name = name
    this.players = players
  }

  [Symbol.iterator]() {
    let index = 0
    return {
      next: () => {
        if (index < this.players.length) {
          return {
            done: false,
            value: this.players[index++],
          }
        } else {
          return {
            done: true,
            value: undefined,
          }
        }
      },
    }
  }
}

const team1 = new Team('gsw', ['curry', 'green', 'klay'])

const team2 = new Team('lakers', ['ad', 'james', 'kobe'])

for (const player of team1) {
  console.log(player)
}

for (const player of team2) {
  console.log(player)
}
```

## 迭代器的中断

在某些情况下，迭代器会在没有完全迭代完的时候被中断掉。

如果我们需要监听中断，可以实现 `return` 方法。

```js
const iterableObj = {
  names: ['curry', 'james', 'harden'],
  [Symbol.iterator]() {
    let index = 0
    return {
      next: () => {
        if (index < this.names.length)
          return { done: false, value: this.names[index++] }
        else return { done: true, value: undefined }
      },
      return() {
        console.log('被中断了')
        return { done: true }
      },
    }
  },
}

for (const item of iterableObj) {
  console.log(item)
  if (item === 'james') break
}
```
