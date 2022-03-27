---
layout: posts
title: Springboot集成Mybatis下PageHelper不起作用
date: 2018-10-25 16:37:28
categories: 问题解决本
tags: [SpringBoot, Mybatis, PageHelper, 分页]
---





最近在项目中使用SpringBoot集成Mybatis，但是Pagehelper不去分页效果的作用



在依赖文件中，去掉原本的pagehelper，改用集成的pagehelper：

```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper-spring-boot-starter -->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.5</version>
        </dependency>
```

