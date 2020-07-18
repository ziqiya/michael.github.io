---
title: AntV
cover: /covers/g2plot.jpg
date: 2020-05-27 20:02:52
category: 前端
tags: [图表, 可视化, AntV]
excerpt: 使用AntV的G2Plot和G2图表开发charts
---

## 一、前言

&emsp;&emsp;之前开发图表用的都是 echarts，组件也很多，但是在每次 UI 图出来以后，少不了一些新的图，每次都要自己重新配置，虽然 echarts 文档很全面，但是也很麻烦。于是尝试使用 AntV 系列进行图表的开发，配合`ChartCube`工具让 UI 根据上面的配置自己选配置出图，更方便。最近在写搭建公司的图表组件库，文档地址在下面。

&emsp;&emsp;官方解释：AntV 是蚂蚁金服全新一代数据可视化解决方案，致力于提供一套简单方便、专业可靠、无限可能的数据可视化最佳实践。因为去年才沉淀完全，所以在一些方面还有些坑需要摸索。

## 二、模块

&emsp;&emsp;AntV 的图表实现主要分为两大块：G2Plot 和 G2 ,其中 G2 比较底层，可配置项也更多，G2Plot 则比较轻量，而且配置更加方便清晰。 g2 即意指图形语法 (the Gramma of Graphics)。

## 三、在线文档

&emsp;&emsp;这个是我写的 `@td-design/charts` 图表组件库文档，记录了一些坑和 G2 图表的用法，大家可以参考一下。[在线文档](https://thundersdata-frontend.github.io/td-doc/#/charts)
