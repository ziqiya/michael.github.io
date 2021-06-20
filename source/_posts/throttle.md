---
title: 消抖节流
cover: /covers/throttle.jpg
date: 2020-11-22 19:53:37
category: 前端
tags: [前端,javascript]
excerpt: 记录一些前端开发过程中的知识点笔记，消抖节流
---

## 一、节流(throttle)

### 定义

函数节流是指连续触发事件,但是在 n 秒中只执行一次.

### 实现方法

1.定时器实现

```js
// 节流方法 1
function throttle(fn,delay=100) {
  let timer = null;
  return function() {
    if (timer) {
      return;
    }
    timer = setTimeout(()=> {
      fn.apply(this,arguments);
      timer = null;
    },delay)
  }
}
```

1.new Date 实现

```js
// 节流方法 2
function throttle(fn,delay=100) {
  let lastTime = 0;
  return function() {
    const newTime = new Date().getTime();
   if(newTime - lastTime > delay){
     fn.apply(this,arguments);
     lastTime = newTime;
   }
  }
}
```

## 二、消抖(debounce)

### 定义

抖动停止后的时间超过设定的时间时执行一次函数。

### 实现方法

1.定时器实现

```js
// 消抖方法 1
function debounce(fn,delay=100) {
  let timer = null;
  return function() {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(()=> {
      fn.apply(this,arguments);
      timer = null;
    },delay)
  }
}
```

## 三、lodash 包

虽然可以自己实现，但是一般还是选择用 `lodash` 上的包，因为考虑的更全面，还提供了更全面的 API。第三个参数记录如下：

节流：
```js
[options.leading=true] (boolean)
// 指定调用在节流开始前

[options.trailing=true] (boolean)
// 指定调用在节流结束后
```

消抖：
```js
[options.leading=true] (boolean)
// 指定调用在延迟开始前

[options.maxWait] (number)
// 设置 func 允许被延迟的最大值

[options.trailing=true] (boolean)
// 指定调用在延迟结束后
```

## 四、注意点

最近在开发中使用了 `debounce` ，有几个注意点，在这里罗列一下。

1.`debounce` 返回的是一个函数，所以最好用法是在定义函数的时候在外面包一层，而不是在用的时候再去包 `debounce`,事实上这样的话也是没有用的，因为每次调用都重新返回了一个函数，这样最后的效果就是调用多少次最后就会打印多少次。如下：

错误示例：

```js
const yourFunc = () => {
  // your code
  console.log(1);
}

useEffect(() => {
  debounce(yourFunc,1000);
},[dependency])
```

正确示例：

```js
const yourFunc = debounce(() => {
  // your code
  console.log(1);
},1000)

useEffect(() => {
  yourFunc();
},[dependency])
```

2.定义 `debounce` 函数要在钩子 `useEffect` 或其他事件的外层，否则每次都会定义一个 `debounce` 函数，延迟过后还是会多次打印。

错误示例：

```js
useEffect(() => {
  const yourFunc = debounce(() => {
    // your code
    console.log(1);
  },1000)

  yourFunc();
},[dependency])
```

正确示例：

```js
const yourFunc = debounce(() => {
  // your code
  console.log(1);
},1000)

useEffect(() => {
  yourFunc();
},[dependency])
```