---
layout: posts
date: 2018-10-16 16:03:04
title: Idea 常用的插件 
categories: 教程
tags: [Idea, 插件]
---



记录一下本人在使用Idea 时用到的十分好用的插件

[官方插件库](http://plugins.jetbrains.com/idea)

<!-- more -->

##### 一、ApiDebugger

###### 效果预览



![img](https:////upload-images.jianshu.io/upload_images/3335021-1d6eecf42c0bba00.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/498/format/webp)

apidebugger.gif

###### 支持的请求方式





![img](https:////upload-images.jianshu.io/upload_images/3335021-bd8655d3d1da1144.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/121/format/webp)

method

 目前ApiDebugger支持 

```
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
TRACE
```



###### Header支持

1. 发送Header
    可在请求发送区的Header内填写需要发送的Header键值对。



   ![img](https:////upload-images.jianshu.io/upload_images/3335021-a0a5cca6cab004e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/471/format/webp)

   request-header

2. 接收Header
    如果请求中携带Header数据，可在响应区Headers中查看到获取到的Header键值对。



   ![img](https:////upload-images.jianshu.io/upload_images/3335021-11605e3907721302.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/465/format/webp)

   response-header

###### Cookie支持

如果请求中存在cookie，可在响应区中的`Cookies`查看所有cookie。




![img](https:////upload-images.jianshu.io/upload_images/3335021-e42931c1eab718f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/467/format/webp)

cookie



###### 多类型的Content-Type支持

支持多种Content-Type，包含`from-data` `x-www-urlencoded` `raw` `binary`。




![img](https:////upload-images.jianshu.io/upload_images/3335021-5dea4df0b8f65476.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/470/format/webp)

form-data





![img](https:////upload-images.jianshu.io/upload_images/3335021-a6a1b21013bdef14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/467/format/webp)

binary

###### 响应支持高亮、格式化支持

在response区域中，body可支持响应内容的高亮，对html、json、xml等数据的格式化显示以及自动换行等。



![img](https:////upload-images.jianshu.io/upload_images/3335021-a07369f1919012dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/465/format/webp)

json



![img](https:////upload-images.jianshu.io/upload_images/3335021-6b15837d5317aba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/467/format/webp)

html

###### 多tab支持

这个过tab的支持可帮助你同时测多个API。



![img](https:////upload-images.jianshu.io/upload_images/3335021-91d63c2c032d1160.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/507/format/webp)

tabs

###### 总结

这个插件能直接在idea的右侧打开就可进行网络请求，真的超级方便，做了沉浸式的开发。



##### 二、MybatisX

mapper接口 与 mapper配置文件相互跳转