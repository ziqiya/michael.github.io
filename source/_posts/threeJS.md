---
title: 关于 threeJS
date: 2019-12-29 21:40:15
cover: /covers/three.jpg
category: 前端
tags: [three.js, javascript, 可视化]
excerpt: three.js 中遇到的一些问题的解决方法
---

## 前言

&emsp;&emsp;今天把之前项目里做的 three.js 的照片墙重新用 html 的方法写了一遍，其中遇到了一些问题跟大家分享一下哈，最终效果在左侧照片墙的链接。

## 配置

### hexo 中的配置

&emsp;&emsp;首先，是 hexo 博客中要使用 three 的配置，不用 hexo 的小伙伴可以跳过这一点。为了在 hexo 里面显示出原页面的 html，需要先在 hexo 中配置一下`skip_render`用来跳过渲染，最好在文件目录下新建一个文件,假设是`photoWall`然后在`_config.yml`中设置:

```bash
skip_render: photoWall/*
```

&emsp;&emsp;表示跳过 photoWall 文件下所有文件的渲染,也可以用\*.html 等进行更进一步的匹配跳过渲染，其他文章里也有提到这里不再赘述。接下来就可以在 photoWall 的文件夹下新建一个 index 文件写 原样的 html 文件了。

### 引入 js 脚本

&emsp;&emsp;如果要在 html 中使用 three.js 的话，需要先把 three.js 给下载到本地进行引入，同时如果想要动画效果的话也可以把`tween.js` 引入:

```javascript
 <script src="js/three.js"></script>
 <script src="js/tween.js"></script>
```

&emsp;&emsp;我这里可以提供现成的下载链接，可以直接右键另存为保存到本地<br/>

- three.js：<a href="/js/three/Three.js">three.js</a>
- tween.js：<a href="/js/three/tween.js">tween.js</a>

&emsp;&emsp;`tween.js` 是一个很好用的平滑动画转换的 js 插件，可以让值从一个状态平滑的过渡到另一个状态。以后在我的博客里会继续介绍。

## 常见问题

## xxx is not a constructor

&emsp;&emsp;因为 three.js 的更新，我最近经常遇到如下问题：

```bash
THREE.FirstPersonControls is not a constructor
```


这个问题意思是，THREE 下的 FirstPersonControls 不是一个构造函数，这就是因为没有引入 script 脚本，因为 three 里面有些插件是需要额外引入的，这个问题有一个解决方法：那就是把`FirstPersonControls.js`的代码在本地引入这样就没问题了，存在同样问题的有`MTLLoader`和`OBJLoader`，也可以用同样的方式解决。这里提供给有一样问题的小伙伴下载链接，可以直接右键另存到本地。

- FirstPersonControls：<a href="/js/three/FirstPersonControls.js">FirstPersonControls：</a>
- MTLLoader：<a href="/js/three/MTLLoader.js">MTLLoader</a>
- OBJLoader：<a href="/js/three/OBJLoader.js">OBJLoader</a>

在本地就可以直接引用`FirstPersonControls：`不需要引用`THREE.`的前缀啦。

### Failed to execute 'texImage2D'

这个问题全部报错信息如下：

```bash
ThreeJS DOMException: Failed to execute 'texImage2D' on 'WebGLRenderingContext'
```

&emsp;&emsp;这个原因是在贴图的时候，由于浏览器或者本地环境 localhost 的安全问题，可以试着部署到线上或者换成谷歌浏览器就可以了。

### glTfloader 的问题

glTFloader 可以直接引入模型和材质,很方便,但是在`three.js`包里的 glTF 文件下的 glTFloader 存在很多问题，比如要分别引入`glTFShaders.js`,`glTFLoaderUtils.js`,`glTF-parser.js`和`glTFLoader.js`的插件，也会报很多 undefined 的错,这里推荐用`three-gltf-loader`的插件,可以直接引入 GLTFLoader，而且用法也是完全一样的，下面给出`three-gltf-loader`的下载地址:
<a href="/js/three/three-gltf-loader.js">three-gltf-loader</a>
