---
layout: posts
title: 正则校验
date: 2020-01-13 14:26:05
cover: /covers/regs.jpg
category: 技术
tags: [javascript,正则]
excerpt: 正则校验字符串格式化的问题
---

## 问题

&emsp;&emsp;在 vscode 的编辑器的字符串里如果输入`\p`或者`\.`等带斜杠的字符串，编辑器就会自动过滤前面的反斜杠，保证正确编辑，但是再某些情况下，这种格式的字符串却是需要的，比如在字符串拼接复杂正则的时候，这个时候应该怎么处理呢？

## 解决方法

&emsp;&emsp;其实也简单，那就是不用字符串来保存正则片段，保存的时候，先用正则储存，等用到的时候再转换成字符串拼接使用,`getRegString`的函数可以把 RegExp 格式的正则转化为去掉两边`/`的正则字符串。

```typeScript
export const getRegString = (reg: RegExp) => ('' + reg).substring(1, ('' + reg).length - 1);

```

从正则里获得 reg 的字符串再用`new RegExp()`从 string 中获得最终的正则表达式。

## 举例

比如想获得英文和空格组成的字符串（这里只是简单举例），那就可以这样子保存：

```typeScript
export default {
  英文: /a-z|A-Z/,
  空格: /\s/
}
```

在用的地方可以这样用

```typeScript
import regs, { getRegString } from '@/utils/regex-utils';

const getRegsExp = () => {
  const rulesArr = ['英文','空格'];
  const regString = `^[${rulesArr.map(item => getRegString(regs[item])).join('|')}]*$`;
  const regsExp = new RegExp(regString);
  return regsExp;
}
```

这样就可以获得校验英文和空格的正则了，并且不用担心正则字符串被编译器自动转化。
