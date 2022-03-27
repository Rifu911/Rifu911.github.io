---
layout: posts
title: RabbitMQ 快速入门
date: 2018-10-17 15:08:45
categories: 学习笔记
tags: [RabbitMQ, 消息队列]
---

ActiveMQ是JMS的实现，RabbitMQ是AMQP的实现

<!-- more -->

### 一、简介



![](clipboard.png)

![](微信截图_20181017144332.png)

![](微信截图_20181017144512.png)

### 二、环境搭建

1.在docker下面

docker pull registry.docker-cn.com/library/rabbitmq:3-management

2.运行该镜像

docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq01 IMAGE-ID

3.在浏览器输入:http://ip:15672/	即可看到登录界面

默认的账号和密码都是guest

> 如果是在云主机上面安装的RabbitMQ,会无法访问，首先需要开启对应的端口
>
> 5672是用于tcp连接的,
>
> 15672是AMQP协议的
>
> 25672是Clurting的
>
> 接下来就是在对应的rabbitmq文件修改
>
> 1.查看rabbitmq的配置文件在哪里
>
> whereis rabbit.app
>
> 最好先进行备份一下，不然后果无法想象
>
> 2.修改有一个属性是*-users:[<<guest>>] 的，改为*_users:[]







### 三、SpringBoot环境搭建RabbitMQ使用

##### 1.步骤：

a).引入rabbitmq的依赖

```java
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

添加以上依赖之后，springboot会对rabbitmq进行自动配置

注入rabbitmq的连接工厂

注入RabbitProperties 映射配置文件中RabbitMQ的属性

注入RabbitTemplate:给RabbitMQ发送和接受消息

注入了AmqpAdmin:RabbitMQ系统管理功能组件

> 创建和删除queue，exchange, binding

@EnableRabbit  +  @RabbitListener  监听消息队列的内容

b).在配置文件中添加rabbitmq的属性

> spring.rabbitmq.host=your.ip
>
> spring.rabbitmq.username=guest
>
> spring.rabbitmq.password=guest
>
> spring.rabbitmq.port=5672(这个可以不写，默认就是5672)
>
> spring.rabbitmq.virtual-host=/(这个可以不写，默认就是/)

c).在Application标注@EnableRabbit开启基于注解的RabbitMQ







### 四、拓展

##### 1）rabbitmq传输数据默认使用的是jdk的序列化器，我们传输数据一般使用的是json的方式

在自定义的rabbit的配置类里面

```java
	/**
     * 替换掉原来默认使用的jdk序列化器，改成支持json传输数据的方式
     * @return
     */
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
```

##### 2) 开启监听消息队列,前提是开启了基于注解的RabbitMQ

在对应的service方法上添加注解

@RabbitListener(queues = "your.msgQueue")

```java
	@RabbitListener(queues = "zava.news")
    public void receiveCustomMsg(CustomMsg msg){
        System.out.println(msg);
    }
```

##### 3) 使用AmqpAdmin动态创建exchange,queue,binding

```java
	/**
     * 使用amqpAdmin创建exchange
     */
    @Test
    public void testCreateExchange(){
        amqpAdmin.declareExchange(new DirectExchange("amqpAdmin.directExchange"));
    }

    /**
     * 使用amqpAdmin创建queue
     */
    @Test
    public void testCreateQueue(){
        amqpAdmin.declareQueue(new Queue("amqpAdmin.queue",true));
    }

    /**
     * 使用amqpAdmin创建binding规则
     */
    @Test
    public void testCreateBinding(){
        amqpAdmin.declareBinding(new Binding("amqpAdmin.queue",
                Binding.DestinationType.QUEUE,
                "amqpAdmin.directExchange",
                "amqpAdmin.abc",null
                ));
    }
```

