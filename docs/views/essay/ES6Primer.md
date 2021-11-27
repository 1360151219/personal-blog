---
title: 《ES6标准入门》阮一峰---读书笔记
date: 2021-11-24
categories:
  - 随笔日记
author: 盐焗乳鸽还要砂锅
tags:
  - JavaScript
---

## 第 15 章 Iterator 遍历器接口

在使用`...`、`for...of`等运算的时候都会默认调用对象的 Iterator 接口。但是 JavaScript 中普通对象是没有这个接口的。下面这个例子是手动给对象添加上这个 Iterator 遍历器。

```js
let obj = {
  count: 0,
  [Symbol.iterator]() {
    return {
      next: () => {
        if (obj.count < 10) {
          return {
            value: obj.count++,
            done: false,
          };
        } else
          return {
            value: obj.value,
            done: true,
          };
      },
    };
  },
};
for (let i of obj) {
  console.log(i); // 0~9
}
```

## 第 16 章 Generator 函数的语法

Generator 函数是 ES6 提供的一个异步编程解决方案，它的语法行为跟正常函数完全不一样。可以理解为一个状态机。

### 状态机

```js
function* clock() {
  while (1) {
    console.log("Tick!");
    yield;
    console.log("Tock!");
    yield;
  }
}
let c = clock();
c.next(); // Tick!
c.next(); // Tock!
```

上述例子就有`Tick`、`Tock`两种状态。

> yield 表达式只能用在 Generator 函数里面。如果要用在另一表达式中，除了用在函数实参以及赋值表达式右边不需要加括号以外，都需要加括号。

### 遍历器对象 Iterator

Generator 函数会返回一个遍历器对象。

```js
let obj = {
  a: 1,
  b: 2,
  [Symbol.iterator]: function* () {
    for (let [key, val] of Object.entries(obj)) {
      yield [key, val];
    }
  },
};
for (let i of obj) {
  console.log(i); // ['a', 1],['b', 2]
}
```

这种方案可以最简单的为对象添加上遍历器属性。

### next 方法的参数

`next()`可以传入一个参数，这个参数会作为上一个`yield`表达式的返回值。`yield`本身是没有返回值的，即`undefined`

```js
function* foo() {
  for (let i = 0; i < 100; i++) {
    let res = yield i;
    if (res) i = 0;
  }
}
for (let i of foo()) {
  console.log(i);
}
```

上述例子会打印出 1~100 的数字，因为`res`总是`undefined`。因此我们可以做以下尝试：

```js
let g = foo();
console.log(g.next());
console.log(g.next());
console.log(g.next());
console.log(g.next(8));
// { value: 0, done: false }
// { value: 1, done: false }
// { value: 2, done: false }
// { value: 1, done: false }
```

因此第 4 次传入一个参数 8，因此触发 if 条件，i=0，此后 `i++` 变为 1 输出

> 因此 next 的参数表示上一条 yield 语句的返回值，所以第一次 next 传参是无效的。V8 引擎直接忽略第一次 next 的参数。从语义上说，第一次 next 是用于启动遍历器对象的。

### Generator.prototype.throw()

Generator 函数返回的遍历器对象有一个 throw()方法，在函数体外抛出错误，可以在函数体内捕获错误，并且不会影响后续的遍历器操作。

> throw()执行后，会附带执行下一次 yield 语句。

### Generator.prototype.return()

可以返回给定的数，并且终结 Generator 函数的遍历。

### yield\* 表达式

如果想要在 Generator 函数中使用另一个 Generator 函数，那么就得使用`yield*` 表达式。

```js
function* foo() {
  yield 1;
  yield* bar();
  yield 4;
}
function* bar() {
  yield 2;
  yield 3;
}

for (let i of foo()) {
  console.log(i);
}
```

有了这个特性，我们可以利用 Generator 函数来实现一个扁平化数组的算法：

```js
function* flat(arr) {
  if (Array.isArray(arr)) {
    for (let i of arr) {
      yield* flat(i);
    }
  } else yield arr;
}

let arr = [1, [2, 3, [4, 5], 6], 7];
let g = flat(arr);
let res = [];
for (let i of g) {
  res.push(i);
}
console.log(res);
```

### 异步编程

这部分可以直接使用`async..await`了