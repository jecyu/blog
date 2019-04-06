---
author: jecyu
layout: post
title: "地图打印中文乱码"
date: 2018-11-05
comments: true
categories: js 相关
tags: [字体编码, GIS]
toc: true
---

## 导语

利用 arcgis 打印地图，打印地图服务是基于 arcgis server 发布的 GP 服务。本文的情景是，通过调用 arcgis server print 服务来打印地图绘制的标注，出现了中文乱码的情况。

![image_1crhdrdla1531rusoaq13ri1qe39.png-101.5kB][1]

<!-- more -->

## 分析原因

字体乱码问题大多数与字符编码有关。如果用英文字体标注中文会引起乱码，显示方框。
分析步骤：

1. 检查标注的字体，是否全部采用了中文字体标注。或者是服务器本身没有对应的字体库，造成中文无法正常显示。
2. 检查数据库中中文字体存储编码是否和实际数据库字符编码匹配。

通过检查，发现发送给 arcgis 的 JSON 文件时，对应的中文 font-family 字体为 Arial，但是在绘制的时候，是可以正常显示中文在地图上的，打印导出的时候才出现乱码。

## 解决乱码

方案一：在进行绘制标注的时候，把默认的字体设置为 "Microsoft YaHei" 这样发送给打印服务器时，就可以正常打印出中文字体了。

    textSymbol.font.family = "Microsoft YaHei";

方案二：不使用 arcgis 的地图服务，而是借助 html2canvas 实现屏幕截图。这个当然没有方案一好，这个方案的主要解决场景是使用 arcgis 打印服务有一些底图打印不出来。

## 总结

导致打印的地图中文乱码的原因很多，需要一步步排除。像上面提到的，有时候有可能是数据库字符编码问题，通常是数据实际存储中文的是 GBK2312，数据库字符编码设置为 UTF-8，这样导致了中文乱码。

> 正常情况，如果是通过 Arcmap 录入数据，无论客户端编码集和 oracle 数据库编码集是否一致，Oracle 都会做正确的字符集转换，来确保存储的中文字符正确，这样从 Server 或其它客户端读取中文字符也会是正确的结果。这个特殊案例是在数据存储时，字符集没有做正确的转换，导致中文字符编码错误，故而引起 Server 中文标注乱码。

## 参考资料

- [关于字体的设置](https://segmentfault.com/a/1190000006110417)
- [地图打印](https://zhuanlan.zhihu.com/p/35273867)
- [arcgis printTask](https://developers.arcgis.com/javascript/3/jsapi/printtask-amd.html#execute)

[1]: http://static.zybuluo.com/linjiyu/shq54zd2y5ysjfk4bh7ly4dr/image_1crhdrdla1531rusoaq13ri1qe39.png
