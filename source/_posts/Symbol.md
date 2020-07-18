---
title: Symbol的应用
date: 2020-01-13 17:13:22
category: 前端
cover: /covers/Symbol.jpg
tags: [ES6,javascript]
excerpt: Symbol的应用和解释
---

## 前言

&emsp;&emsp;在 ES6 里又新加了一个 `Symbol` 的基础类型，有空去学习了一下其基础的使用方法和用处。在这里整理了一下比较常用的属性和方法。

## 解释

&emsp;&emsp;`Symbol`，英文里的意思是象征，标识的意思，表示独一无二的值。这样在使用的时候就可以避免与其他值重复。`Symbol` 函数不能使用`new`命令，因为 `Symbol` 可以理解为跟 string 类似的原始类型的值。所以 `Symbol` 也没有对象中的属性。

&emsp;&emsp;`Symbol` 函数可以接受一个字符串作为参数，相当于一个标记，也就是对 `Symbol` 的描述，当然每一个 `Symbol` 的值都是不同的。就算标记一样，`Symbol` 也不会相等，这一点有点像 NaN，就算两个 `Symbol` 完全一样也不会相等。

```bash
console.log(Symbol('name') === Symbol('name'));
```

&emsp;&emsp;在控制台里运行上面的代码也自然会是 `false`。

## 用法

### 对象中使用

&emsp;&emsp;以 `Symbol` 的这个特性，可以用来在对象里面获得唯一的属性名。防止公用的对象属性被改写或覆盖。可以这么写：

```typescript
let s = Symbol();

let obj = {
  [s]: "Hello"
};

const word = obj[s]; // "Hello"
```

&emsp;&emsp;但是在使用 `Symbol` 的时候就不能直接用`.`运算符了，因为对象的`.`运算符都是直接引用的 string，跟 `Symbol` 不是一个类型，要引用必须用`[]`包括起来。

&emsp;&emsp;但是原来我们用的`Object.keys`获得所有 object 的属性数组的方法也是无法获得 `Symbol` 类型的属性的，需要获得 `Symbol` 类型属性要用新的方法：`Object.getOwnPropertySymbols()`，同时，如果想获得所有 `Symbol` 和`string`类型的属性就要用`Reflect.ownKeys()`了。这些是需要注意的点。

### Symbol.for()

&emsp;&emsp;我们如果想使用同一个`Symbol`值的话，就可以`Symbol.for()`,每次调用`Symbol()`都会返回不一样的值，但是`Symbol.for()`不一样，每次`Symbol.for()`的时候都会检查全局是否有登记过的`Symbol.for`的标签。如果有就会拿之前的值，否则会自动登记一个新的`Symbol.for`标签。前后的值是一样的，即:

```typescript
Symbol.for("bar") === Symbol.for("bar");
// true
```

&emsp;&emsp;`Symbol.keyFor()`接收一个`Symbol`类型的参数，可以获得登记过的`Symbol`的标签名。
