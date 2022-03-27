---
title: 使用SpringBoot的定时任务
date: 2018-10-18 16:03:42
categories: 学习笔记
tags: [SpringBoot , 定时任务]
---





##### 步骤

1.在主类上添加标注

```java
@EnableAsync        //开启异步注解功能
@EnableScheduling     //开启基于注解的定时任务
```

2.在对应的业务层添加标注@Scheduled

```java
	/**
     * second ,minute ,hour ,day of month ,month ,day of week
     */
    @Scheduled(cron = "0 * * * * MON-FRI")  //周一到周五的每分钟
```

