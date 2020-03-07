---
title: React Hooks
cover: /covers/hooks.jpg
date: 2020-03-07 15:12:56
category: 技术
tags: [react,hooks]
excerpt: React的新特性hooks的介绍及常用函数介绍
---

## 一、前言

&emsp;&emsp;以前开发React的时候组件都是分为无状态组件和有状态组件的，无状态组件一般用function优化性能，有状态组件就用class，然后在组件里加一些生命周期钩子函数，比如：`componentWillMount,componentWillUpdate,componentDidMount,componentWillUnmount`。React Hooks只能用于函数组件,但是在hooks的函数组件中也可以使用状态，那就可以统一用hooks function来写有状态组件了。下面是摘自官网对React Hooks的介绍：

> Hooks let you use state and other React features without writing a class.

## 二、具体使用对比

### 1.useState VS setState

&emsp;&emsp;之前我们在项目里面经常在`constructor`构造器里用`this.state = {}`来进行`state`状态的初始化，再在函数体里用`this.setState({})`来进行异步设置状态值。如下：

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

&emsp;&emsp;但在使用hooks以后`state`的初始化就变成了`const [count, setCount] = useState(0)`，在括号里是对应变量`count`的初值,`setCount`是修改`count`变量的方法，对应原来的`setState`，不同是`setCount`是针对单个值修改，而`setState`是针对于整个`state`中的状态对象。<b>这里的`count`和`setCount`都是自定义的名字，推荐用这种命名格式将变量与对应set方法对应。</b>。hooks修改后函数如下：

```js
import React, { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

&emsp;&emsp;显而易见，相比`class`的的写法要简洁得多。

### 2.useEffect VS componentDidMount,componentDidUpdate,componentWillUnmount

&emsp;&emsp;先来介绍一下`useEffect`函数，`useEffect`接受两个参数，第一个是函数，当组件第一次被挂载到DOM以后，`useEffect`就必然会执行第一个函数，第二个参数是一个数组，表示依赖项，React会监听`useEffect`第二个参数中数组中每一个变量的值，当其中某一个或多个发生变化时，`useEffect`就会再执行一次`useEffect`中的第一个函数。

&emsp;&emsp;之前我们在`componentDidMount`的钩子函数里执行组件被挂载到DOM后的操作，且只会调用一次。这个可以用`useEffect`第二个参数传`[]`的方法代替。类似的，如果想要在每次组件重新渲染的时候就调用就不传第二个参数,这样就等效于同时用了`componentDidMount`和`componentDidUpdate`。

&emsp;&emsp;再如`componentWillUnmount`,我们用在卸载组件的时候执行，用`useEffect`可以选择`return`一个函数回去，效果是一样的，具体原代码如下：

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    document.title = `卸载组件`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

以上代码跟下面的代码是等效的：

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
    // 这里执行的是卸载组件要执行的函数
    return () => {
      document.title = `卸载组件`;
    }
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### 3.useReducer VS redux

`useReducer`是hooks里类似`Redux`功能的一个hook函数，一般用在监听变量比较多，`useState`应付不过来的时候。首先来复习一下`Redux`的常见写法:

- 1.`"react-redux"`中的`connect`可以接受两个括号传参，第一个括号里接受两个函数A,B，第二个括号接受当前组件Component。
- 2.函数A一般叫`mapStateToProps`，用来把当前的`state`关联到`this.props`中可以使用。
- 3.函数B一般叫`mapDispatchToProps`，结合`'redux'`中的`bindActionCreators`方法把函数C(用于执行操作并调用`dispatch`)也关联到`this.props`中可以使用。
- 4.函数C中的`dispatch`调用以后则会调用`reducer`中的处理方法，判断`dispatch`的`type`类型返回对应的`state`。

看着饶了一大圈其实简单的来说就是：调用函数C->dispatch->调用`reducer`方法返回更新的state->在`this.props`里获得更新后的state里的变量.

`useReducer`相对来说简单好理解一些，如下是`useReducer`的实例:

```js
import React,{ useReducer } from 'react'

// 初始化的state
const initialState = {
  count: 0,
}

export default function ReducerDemo() {
    const reducer = (state,action) => {
      if(action.type === 'add'){
        return {
          ...state,
          count: count + 1
        }
      }
      return state;
    };
    const [state, dispatch] = useReducer(reducer, initialState);
    const { count } = state;

    // 新增count
    const handleIncreaseCount = () => {
      dispatch({ type: 'add' });
    }
    return (
        <div onClick={}>
            <h1 className="title">{count}</h1>
        </div>
    )
}
```

&emsp;&emsp;`useReducer`的用法是：接受一个 `reducer` 函数作为参数，函数`reducer` 接受两个参数一个是 `state`(prev状态) 另一个是 `action`(调用`dispatch`的传参)，函数`reducer`返回值是更新后的`state` 。`useReducer`返回值是一个数组，第一个值是状态 `state`(当前状态) 和 `dispath`， `dispatch` 是一个可以发布事件调用`reducer`函数来更新 state 的方法。

&emsp;&emsp;可以从上面的代码看出，`useReducer`的步骤就是：调用函数`handleIncreaseCount`->dispatch->调用`reducer`方法更新`state`->在`useReducer`返回值`state`中获取更新后的`state`。

&emsp;&emsp;以上步骤一对比发现其实`useReducer`和`reducer`控制state的思想是一样的。区别就是`useReducer`更简单，便于使用。
