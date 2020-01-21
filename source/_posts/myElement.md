---
title: myElement
cover: /covers/myElement.jpg
date: 2020-01-21 15:41:38
category: 技术
tags: 油猴
excerpt: 油猴脚本-我的元素
---

## 一、前言

&emsp;&emsp;要说浏览器上的插件哪个最强，那肯定是油猴脚本了，上面有很多好用的油猴脚本，比如获得解析后的 vip 视频脚本，自动滚动的脚本等，而今天要说的是我自己写的一个油猴脚本--myElement，主要用来生成页面的自定义元素，用于页面演示或者样式说明，查看页面样式。在所有的页面都可以使用。

## 二、使用方法

先下载油猴，本脚本已经放到`openuserjs`官网上了： <a href="https://openuserjs.org/scripts/RuanXuSong/%E6%88%91%E7%9A%84%E5%85%83%E7%B4%A0">myElement</a> ,点击链接->点击 Install 下载安装脚本。脚本运行后的效果图如图：

![脚本效果](/images/posts/myElement/element.jpg)

## 三、功能说明

### 1.折叠菜单

- 1.点击右侧圆形收缩按钮可以使侧边栏收缩到最小，不影响正常浏览。
- 2.再次点击圆形收缩按钮可以使菜单栏回到原始形状。

### 2.添加元素

- 1.点击元素出现元素列表
- 2.点击元素 div 或 button 可以在页面中插入对应元素 3.画线:点击 line，再在页面中任意位置点击可以确认直线起点，再点击一次可以确认直线终点，连成一条直线。再点击页面任意位置可以开始下一次画线，可以连续画线。

<span style="color:#F0412C;">注：如果进入画线模式后不退出，执行其他操作将会卡顿，需要点击 line 以后出现的“点击取消画线”按钮，可以退出画线。</span>

### 3.添加属性

- 1.在添加元素以后，可以双击该元素，此时在属性列表下可以看到一个初始化的有一些默认值的属性列表，进行填值，input 失去焦点以后即可给该元素该属性赋值，并将该属性样式存到缓存中。
- 2.添加另外属性:点击属性列表下的加号，会出现新的 input 行，在左边填入 CSS 属性名，右边填入 CSS 属性值，可以添加另外的 CSS 样式，并将该属性存到缓存中。

### 4.增加动画幕布

- 1.在元素 Tab 下，点击 cover 即可一键自动在页面生成动画幕布。

### 5.一键填充表单

- 1.基本模式：在元素 tab 页下，点击下方的“一键填充表单”按钮，出现一对 input 输入框和一个 textarea 文本框，此时可以复制 JSON 格式字符串到 textarea 文本框中，失去焦点即可一键填充页面内的所有对应表单项，免去了测试调试时一个个去填充的烦恼。
- 2.辅助模式：也可以通过上方的 input 输入框来生成 JSON，在左边输入要填充的 input 的 name 值，右边填入要填充的 input 的 value 值，失去焦点以后就会在 textarea 中自动生成对应的 JSON，同时自动填充页面表单值。要添加新的 input 的键值对，只要把原来的 input 内容改为要新增的就可以了，原来的 textarea 中不会被删除或更改。
- 3.退出：点击下方的“退出表单填充模式”按钮即可退出，原有的 textarea 的值不会被清除。

### 6.移动

- 1.生成 div 或 button 元素后，可以用鼠标拖动移动到指定位置。同时样式中的坐标值会改变,同时保存到缓存中。

### 7.下载样式

- 1.若想下载调节样式后的样式表，可以点击元素页下的下载按钮进行下载样式。
- 2.文件格式为 txt，文件名为下载时间，文件下 createdAt 后为元素的创建时间，itemOrder 为元素添加的次序(第几个添加的)，其后面的括号内字符串为 css 格式的字符串，可以直接复制使用。

### 8.删除元素

- 1.点击元素列表，出现删除按钮，点击删除以后，再去点击要删除的线或是元素即可删除。
- 2.退出删除：1.点击“删除”以后，按钮内容变成“退出删除”，此时点击“退出删除”按钮即可退出删除。2.或者点击元素列表下任意元素也可退出删除

### 9.查看页面元素

- 1.点击“查看元素”按钮即可监听鼠标的滑动事件(注：最好先退出画线的状态，否则会卡顿)，鼠标滑过页面上的元素时将显示出滑过的元素的范围和标签，属性，html 元素。效果如下：

![脚本效果](/images/posts/myElement/inspect.jpg)

### 10.禁止跳转

- 1.点击“禁止跳转”按钮可以禁用当前页面所有的链接跳转，防止在操作的时候误点链接导致离开当前页。