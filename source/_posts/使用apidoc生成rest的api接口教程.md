---
layout: posts
title: 使用apidoc生成接口文档 
date: 2018-10-31 13:40:51
categories: 学习教程
tags: [apidoc, 接口]
---

### 一、简介

现在大家普遍使用的是swagger结合springmvc来生成api接口文档，对比apidoc，swagger有一个明显的劣势，便是返回的响应，无法生成文档描述，即无法描述响应体的数据结构，这对前后端对接，或者是与移动端/其他端对接来说，需要耗费更多的交流成本，沟通成本，即不可能每个接口都通过实际调用后，看返回实体获悉响应参数。

<!-- more -->

### 二、环境准备

1.安装node.js，官网：https://nodejs.org/en/点击打开链接；windows64位下载地址https://nodejs.org/dist/v8.9.4/node-v8.9.4-x64.msi下载；

2.安装apidoc，命令行下，输入npm install apidoc -g，参考官网：http://apidocjs.com/#install 点击打开链接

npm install apidoc -g
安装完毕，可在命令下使用apidoc -h测试是否安装成功
apidoc -h
3.apidoc指令能成功识别，apidoc环境便已经安装好了，这时可在项目中使用，所有的代码基于注释即可。

### 三、整合项目使用

##### 1.项目根路径下建立apidoc.json文件，配置好基本的文档信息。

{
  "name": "API文档",
  "version": "1.0.0",
  "description": "开发技术接口文档",
  "title": "API文档",
  "url" : "http://localhost:8080/test",
  "sampleUrl":"http://localhost:8080/test"
}

---------------------
##### 2.抽象一些通用的返回信息，自定义一些tag

```java
public class BaseApi {
    /**
     * @apiDefine error_msg 全局配置失败响应信息
     * @apiError 1001 保存失败
     * @apiError 1002 修改失败
     * @apiError 1003 删除失败
     * @apiError 1004 上传失败
     * @apiError 1005 注册失败
     * @apiError 1101 输入参数格式不正确
     * @apiError 1102 用户名或者密码错误
     * @apiError 1103 用户名不存在
     * @apiError 1201 发送手机注册验证码失败
     * @apiError 1202 用户注册失败
     * @apiError 1203 机构不存在
     * @apiError 1204 注册验证码输入错误
     * @apiError 1205 手机号码已存在
     * @apiError 1206 用户名已存在
     * @apiError 1207 机构不存在
     * @apiError 1208 手机或者用户名已存在
     * @apiError 4101 token过期
     * @apiError 4102 token签名错误
     * @apiError 4103 无效token
     * @apiError 4104 token格式错误
     * @apiError 5000 接口内部错误
     * @apiErrorExample 错误响应例子:
     *     {
     *       "code": 1101,
     *       "msg": "输入参数格式不正确",
     *       "res": "",
     *       "timestamp": 1489110927975
     *     }
     *
     */
 
    /**
     * @apiDefine success_msg 全局配置成功响应信息
     * @apiSuccess (success 2000) {Date}  timestamp     时间戳
     * @apiSuccess (success 2000) {Integer} code        响应码
     * @apiSuccess (success 2000) {String}  msg       响应信息
     * @apiSuccess (success 2000) {Object}  res   响应实体
     */
 
    /**
     * @apiDefine token_msg 全局配置token鉴权请求头
     * @apiError 4101 token过期
     * @apiError 4102 token签名错误
     * @apiError 4103 无效token
     * @apiError 4104 token格式错误
     * @apiHeader {String}  Authorization 鉴权信息：为Bearer + "空格" +  {token}
     * @apiHeaderExample {json} 请求头例子:
     *     {
     *       "Authorization": "Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxNDg5NjAiLCJpYXQiOjE0OTUxNjYyMzgsImV4cCI6MTQ5Nzc1ODIzOH0.Mv8BfTIGxGZ6AGkYqHFTRhp40x5xHV6k7Hpwo6OdgiA"
     *     }
     */
}
抽象一些返回的错误代码

public enum AttendRestEnum implements RestEnum{
    /**
     * @apiDefine ATTEND_EMPTY_ID
     * @apiError 5001 规则不能为空
     */
    ATTEND_EMPTY_ID(5001,"规则不能为空"),
    /**
     * @apiDefine ATTEND_EMPTY_VALUE
     * @apiError 5002 值不能为空
     */
    ATTEND_EMPTY_VALUE(5002,"值不能为空"),
    /**
     * @apiDefine ATTEND_ERROR_EQUAL_VALUE
     * @apiError 5003 设置参数的个数不一致
     */
    ATTEND_ERROR_EQUAL_VALUE(5003,"设置参数的个数不一致"),
    /**
     * @apiDefine ATTEND_EMPTY_LONGITUDE
     * @apiError 5004 经度不能为空
     */
    ATTEND_EMPTY_LONGITUDE(5004, "经度不能为空"),
    /**
     * @apiDefine ATTEND_EMPTY_LATITUDE
     * @apiError 5005 纬度不能为空
     */
    ATTEND_EMPTY_LATITUDE(5005,"纬度不能为空" ),
    /**
     * @apiDefine ATTEND_EMPTY_DEVICE_SN
     * @apiError 5006 设备不能为空
     */
    ATTEND_EMPTY_DEVICE_SN(5006,"设备不能为空" ),
    
    /**
     * @apiDefine ATTEND_EMPTY_ORG
     * @apiError 5007 机构不能为空
     */
    ATTEND_EMPTY_ORG(5007,"机构不能为空"),
    
    /**
     * @apiDefine ATTEND_NOT_FIND_ORG
     * @apiError 5008 机构没有找到
     */
    ATTEND_NOT_FIND_ORG(5008,"机构没有找到"),
    
    /**
     * @apiDefine ATTEND_EMPTY_MINUTES
     * @apiError 5009 使用时长不能为空
     */
    ATTEND_EMPTY_MINUTES(5009,"使用时长不能为空"),
    
    /**
     * @apiDefine ATTEND_ERROR_MINUTES
     * @apiError 5010 使用时长不能为负数
     */
    ATTEND_ERROR_MINUTES(5010,"使用时长不能为负数"),
    
    /**
     * @apiDefine ATTEND_ERROR2_MINUTES
     * @apiError 5011 当天使用时长不能大于24小时
     */
    ATTEND_ERROR2_MINUTES(5011,"当天使用时长不能大于24小时")
    
    ;
 
    private  final int code;
    private final String msg;
 
    private AttendRestEnum(int code,String msg){
        this.code = code;
        this.msg = msg;
    }
    @Override
    public int getCode() {
        return this.code;
    }
 
    @Override
    public String getMsg() {
        return this.msg;
    }
}
```

##### 3.在接口中使用自定义标签

```java

	/**
     * @api {get} /rest/area/getAreasByCode 行政区域查询
     * @apiDescription 根据行政编码获取行政区域,0获取省级行政区域
     * @apiName getAreasByCode
     * @apiGroup area
     * @apiVersion 1.0.0
     *
     * @apiParam {String} code 行政编码
     *
     * @apiSampleRequest /rest/area/getAreasByCode
     * @apiUse token_msg
     * @apiUse success_msg
     * @apiSuccess (success 2000) {String}   res.id    标识码
     * @apiSuccess (success 2000) {String}   res.name    行政地区名称
     * @apiSuccess (success 2000) {String}   res.code    行政编码
     * @apiSuccess (success 2000) {String}   res.prevCode    上级行政编码
     * @apiSuccess (success 2000) {String}   res.allName    全称
     *

```



##### 4.生成接口文档

使用命令：apidoc -i ./ -o ./apidoc

然后就可以在apidoc文档的index.html看到生成的接口文档啦





参考博客：[Spring项目集成apidoc生成api接口文档](https://blog.csdn.net/chemphone/article/details/79193466)







