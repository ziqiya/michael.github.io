---
title: docker
cover: /covers/docker.jpg
date: 2020-04-26 10:27:31
category: 技术
tags: [项目管理]
excerpt: 关于docker的基础指令及用法
---

## 一、前言

&emsp;&emsp;Docker，后端同学应该很熟悉了，但有些前端同学可能还没有接触过。Docker，英文直译是“码头工人”。类似的，他在项目中的作用也是如此，他可以帮你把打包好的货物（应用/项目）运到船上（容器），再送到目的地（线上环境）交付。官方的解释是：Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上。docker的本质是一个附加系统，类似于虚拟机，我们可以在虚拟机上运行其他系统的程序，docker上经过系统管理以及配置以后也可以用虚拟的操作系统运行应用，而且docker非常轻量。

&emsp;&emsp;这篇博客主要记录一下对线上已存在镜像的docker拉到本地的指令及方法。

## 二、用法步骤

### 1.镜像加速器

&emsp;&emsp;因为为了更快地拉取线上容器，推荐加个镜像加速器，需要在阿里云的控制台里搜索“容器镜像服务”，进入“镜像中心”下的“镜像加速器”,如下图
![镜像加速器](/images/posts/docker/mirror-accelerate.jpg)

&emsp;&emsp;如果是最新版docker，有两种添加方法：

 * 法1：用上图中推荐的方法。
 * 法2：复制加速器地址，进入docker目录下的preference，进入Docker Engine，在JSON的配置项中加`"registry-mirrors"`项，并把镜像加速器地址设为数组中的值，应用并重启Docker，拉取速度就可以大大提升。

配置如图:
![镜像加速配置](/images/posts/docker/dockerEngine.jpg)

### 2.拉取并挂载docker
 * 1.拉线上docker镜像 `docker pull 镜像名:v版本号`
 * 2.获得本地地址:`pwd`
 * 3.复制docker到本地test文件夹
  
  ``` bash
  docker cp 容器名:应用地址 eMacBook-Pro:~ michael$ cd test/
  ```
 * 4.docker新开运行并挂载到本地，挂载以后修改本地就会同步修改Docker容器文件：
  
  ``` bash
  docker run -d --name 容器名 -p 运行端口号:80 -p 8000:8000 -p 9000:9000 -p 9001:9001 -p 9002:9002 -v 本地挂载地址:对应线上地址 镜像名:v版本号
  ```
  * 如果只是docker新开运行（不挂载到本地）：
  ``` bash
  docker run -d --name 容器名 -p 运行端口号:80 -p 8000:8000 -p 9000:9000 -p 9001:9001 -p 9002:9002 镜像名:v版本号
  ```

### 3.运行docker
 * 1.进入docker命令行运行镜像 docker exec -it 容器名 bash
 * 2.`docker ps`查看docker运行状态
 * 3.运行要执行的指令

到这里docker服务以及启动了，可以在`http://localhost:8999`地址看到运行的内容了。

## 三、其他常用指令

* 停止docker
`docker stop 容器名`

* 删除容器
`docker rm 容器名`

* 运行容器,容器停止后可以用此命令开启
`docker start 容器名`

* 重启容器，注：重启以后所有数据状态都会丢失
`docker restart 容器名`

* 查看容器日志
`docker logs -f 容器名`