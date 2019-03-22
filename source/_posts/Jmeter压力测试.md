---
title: Jmeter压力测试
date: 2018-12-14 21:58:11
categories: 性能
tags: [Jmeter]
---


## jmeter压力测试

> 注意：需在安装jdk的环境下操作

Apache JMeter是Apache组织开发的基于Java的压力测试工具。用于对软件做压力测试，它最初被设计用于Web应用测试，但后来扩展到其他测试领域。 

<!-- more --> 

### mac

* 安装
```
brew install jmeter

路径：/usr/local/Cellar/jmeter/5.0
```

* 启动
```
open /usr/local/bin/jmeter
```

### windows

* 下载地址：https://mirrors.tuna.tsinghua.edu.cn/apache/jmeter/binaries

![5QGLby.jpg](https://s1.ax2x.com/2018/12/12/5QGLby.jpg)

* 安装：直接将下载好的zip压缩包进行解压即可。
* 启动：进入bin目录，找到jmeter.bat文件，双机打开即可启动。


### 使用
#### 1、修改主题和语言

默认的主题是黑色风格的主题并且语言是英语，这样不太方便使用，所以需要修改下主题和中文语言。

 ![5QGTJK.png](https://s1.ax2x.com/2018/12/12/5QGTJK.png)

 ![5QGz5n.png](https://s1.ax2x.com/2018/12/12/5QGz5n.png)

![5QGVOE.png](https://s1.ax2x.com/2018/12/12/5QGVOE.png)

主题修改完成。

接下来设置语言为简体中文。

![5QG0g2.png](https://s1.ax2x.com/2018/12/12/5QG0g2.png)

语言修改完成。 ![5QGdcz.png](https://s1.ax2x.com/2018/12/12/5QGdcz.png)

#### 2、创建首页的测试用例

第一步：保存测试用例 ![5QGOZH.png](https://s1.ax2x.com/2018/12/12/5QGOZH.png)

第二步：添加线程组，使用线程模拟用户的并发

 ![5QGub9.png](https://s1.ax2x.com/2018/12/12/5QGub9.png)

 ![5QG4UA.png](https://s1.ax2x.com/2018/12/12/5QG4UA.png)

1000个线程，每个线程循环10次，也就是tomcat会接收到10000个请求。

第三步：添加http请求

 ![5QGJnO.png](https://s1.ax2x.com/2018/12/12/5QGJnO.png)

 ![5QGhQq.png](https://s1.ax2x.com/2018/12/12/5QGhQq.png)

第四步：添加请求监控

 ![5QGvce.png](https://s1.ax2x.com/2018/12/12/5QGvce.png)

 ![5QGyid.png](https://s1.ax2x.com/2018/12/12/5QGyid.png)

#### 3、启动、进行测试 

 ![5QGaXR.png](https://s1.ax2x.com/2018/12/12/5QGaXR.png)

#### 4、聚合报告

在聚合报告中，重点看吞吐量。

 ![5QGcZr.png](https://s1.ax2x.com/2018/12/12/5QGcZr.png)