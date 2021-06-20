---
title: cordova
cover: /covers/cordova.jpg
date: 2020-02-03 19:03:28
category: 前端
tags: cordova
excerpt: cordova环境搭建的问题总结与解决方法
---

## 一、前言

&emsp;&emsp;近期因为接手一个基于 cordova 和 taro 的项目，在安卓和苹果端的打包过程中遇到了一些坑，这里给出基本流程指南及常见问题的解决方法。

## 二、打包步骤

### 1、环境安装

#### 1.全局安装`cordova`:

```bash
npm install -g cordova
```

#### 2.验证`cordova`是否安装成功:

```bash
cordova -v
```

#### 3.预下载包，进入`cordova`项目根目录下执行

```bash
cordova prepare
```

这一步可以帮你把一些插件包都安装好，否则直接执行`build`的话会在安装某些包的时候卡死。

第一次执行`cordova`时可能会遇到以下问题:

- Q：提示报错 `no such file or directory open xxx\.config\cordova-config.json` 。
  A：需要在对应的目录下手动新建一个名为`cordova-config.json`的文件，让里面的内容为空对象。

- Q：提示报错 `Current working directory is not a Cordova-based project.`。  
   A：这并不是因为这不是一个`cordova`项目，事实上，你只要在项目根目录下手动新建一个`www`的空文件夹就可以解决了。

#### 4.打包项目：

- 安卓端：

```bash
cordova build android -release
```

&emsp;&emsp;这里第一次执行的时候会特别慢，毕竟服务器在国外，这里可以参考<a href="https://blog.csdn.net/yanzisu_congcong/article/details/78020056?utm_source=blogxgwz2">这个博客</a>的步骤将 gradle 包先下载下来或者翻墙，这一步可以把安卓的各个手机类型的包打包出来，这里的`-release`参数加上才可以得到`release`的 apk 包，可用于安装使用，否则只会获得`debug`版本的，只能用于调试。

这里可能会遇到的问题：

```bash
A Problem occurred configuring root project 'android'.
> Could not resolve all dependencies for configuration ':classpath'
```

这是因为`gradle`的版本在安装时不一致，需要清一下缓存:

```bash
  cd platforms
  cd android
```

进入 android 文件夹下，再清缓存即可解决：

```bash
./gradlew clean
```

- 苹果端：

**_1.执行以下代码来更新 ios 工程下的文件：_**

```bash
cordova build ios
```

可能会遇到以下报错：

```bash
Cordova build fails with 'toLowerCase' of undefined
```

这是因为`xcode11`跟`cordova`有些冲突,需要执行以下指令：

```bash
cordova platform rm ios
cordova platform add ios@latest
```

安装最新版的 ios 并使你的`cordova-ios` 版本 >= 5.0.0。

2.打开`Xcode`进行打包发布，在 xcode 中打开第一步中更新的`platforms/ios/xxx.xcworkspace`文件，打开 iOS 工程，选为`Generic iOS Device`。

3.在菜单中选择 Product -> Archive 进行打包。

4.打包成功后点击`Distribute App`,选择发布方式，生产环境选择 iOS App Store，测试环境选择 Ad Hoc。再点击`Next`，再选择合适的协议文件。

5.最后，测试的点击`export`,生产的点击`upload`，打包就完成了。

## 三、其他问题

- Q：如果在执行 taro 项目打包的时候遇到这个问题怎么办：
  ![bug-img](/images/posts/cordova/taro-bug.jpg)  
  A：这个问题可能是 taro 版本的问题，因为低版本 taro 不支持箭头函数写组件内的函数，可以把`package.json`中更改`taro`版本到`1.3.12`，再删除`package-lock.json`,重新`npm install`装包。如果还没有解决，那可能是因为本地的一些 node 依赖与`taro`不兼容，需要撤销本地的修改后再重装依赖。

## 四、补充注意

### 补充注意

1、在安装插件或其他的时候遇到以下问题：
```
Could not resolve all files for configuration ':classpath'.
Connect to maven.google.com:443 [maven.google.com/172.217.160.110] failed: Operation timed out
```
可以修改操作如下：
![bug-img](/images/posts/cordova/md_28.jpg)  
```
// 替换代码如下
 maven { url 'https://maven.aliyun.com/repository/google' }
 maven { url 'https://maven.aliyun.com/repository/jcenter' }
```

2、如果出现 ios 支付无法使用的问题，需要把 ios 的 platform remove 掉，再 add，add的时候如果报错需要重装 cordova 版本为7.1.0。这是因为插件 plugins 需要更新，remove 再 add platform 可以自动重装插件。

3、如果遇到插件重装报错问题，可以先把 node_modules 里后面带箭头的文件都删掉再重装插件否则会报错。

4、如果遇到以下问题：
```
Error: Unable to graft xml at selector "/manifest/application/meta-data[@android:name='JPUSH_CHANNEL']" from "/Users/michael/Work/leishu/nengxin/ebc-mobile/platforms/android/AndroidManifest.xml" during config install
```
解决方法:把 config.xml 里的	JPUSH_CHANNEL 对应项改掉，参考：<a href="https://tracker.moodle.org/browse/MOBILE-3283">解决方法</a> 

5、打包的安卓如果无法请求数据和图片，platforms 下的 android 文件夹直接用已有线上仓库的。