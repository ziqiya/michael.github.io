---
title: 闭包
cover: /covers/closure.jpg
date: 2020-10-22 19:53:37
category: 前端
excerpt: 记录一些前端开发过程中的知识点笔记，闭包
---

## 详细

### 一、闭包

#### 定义

闭包是指有权访问另外一个函数作用域中的变量的函数。内部函数总是可以访问其所在的外部函数中声明的参数和变量，即使在其外部函数被返回了之后。

#### 产生本质

当前环境中存在指向父级作用域的引用

#### 特性

- 让外部访问函数内部变量成为可能。
- 闭包找到的是同一地址中父级函数中对应变量最终的值。
- 可以避免使用全局变量，防止全局变量污染。
- 会造成内存泄漏（有一块内存空间被长期占用，而不被释放）
- 闭包就是可以创建一个独立的环境，每个闭包里面的环境都是独立的，互不干扰。

```js
// 例子1
function outerFn(){
var i = 0;
  function innnerFn(){
      i++;
      console.log(i);
  }
  return innnerFn;
}
var inner1 = outerFn();
var inner2 = outerFn();
inner1();
inner2();
inner1();
inner2();    //1 1 2 2
```

例1说明了：闭包就是可以创建一个独立的环境，每个闭包里面的环境都是独立的，互不干扰。

```js
// 例子2
(function() { 
  var m = 0; 
  function getM() { return m; } 
  function seta(val) { m = val; } 
  window.g = getM; 
  window.f = seta; 
})(); 
f(100);
console.info(g());   //100
```

例2说明了：闭包找到的是同一地址中父级函数中对应变量最终的值

```js
// 例子3
function fn(){
   var arr = [];
   for(var i = 0;i < 5;i ++){
	 arr[i] = function(){
		 return i;
	 }
   }
   return arr;
}
var list = fn();
for(var i = 0,len = list.length;i < len ; i ++){
   console.log(list[i]());
}  //5 5 5 5 5
```

```js
// 例子4
function fn(){
  var arr = [];
  for(var i = 0;i < 5;i ++){
	arr[i] = (function(i){
		return function (){
			return i;
		};
	})(i);
  }
  return arr;
}
var list = fn();
for(var i = 0,len = list.length;i < len ; i ++){
  console.log(list[i]());
}  //0 1 2 3 4
```

例子3和例子4是因为立即执行函数的区别，立即执行并返回一个函数，这样就防止了闭包的返回最终值的特性。