---
layout: posts
title: SpringBoot 发送邮件
date: 2018-10-18 15:58:31
categories: 学习笔记
tags: [SpringBoot, 邮件]
---



##### 步骤

1.添加依赖包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

<!-- more -->

2.配置信息

```properties
spring.mail.username=528428122@qq.com
spring.mail.password=***
spring.mail.host=smtp.qq.com
spring.mail.properties.mail.smtp.ssl.enable=true
```

== 其中密码=授权码去自己的个人邮箱设置里面找 ==

3.发送

```java
	@Autowired
    JavaMailSenderImpl mailSender;

    @Test
    public void sendMail(){
        SimpleMailMessage mail = new SimpleMailMessage();
        mail.setSubject("通知");
        mail.setText("今晚加班哦");

        mail.setTo("13071679487@163.com");
        mail.setFrom("528428122@qq.com");

        mailSender.send(mail);
    }

    @Test
    public void sendMailWithHTMLAndFile() throws Exception {
        MimeMessage mail = mailSender.createMimeMessage();

        MimeMessageHelper helper = new MimeMessageHelper(mail, true);
        helper.setSubject("通知，这个是可以发送html和文件的");

        helper.setText("<b style='color:blue'> 带颜色的文本 </b>",true);   //第二个参数为true,支持html
        helper.addAttachment("test.png",new File("C:\\Users\\Lifu\\Desktop\\照片\\zava.png"));
        helper.addAttachment("test.png",new File("C:\\Users\\Lifu\\Desktop\\照片\\rifu.png"));


        helper.setTo("13071679487@163.com");
        helper.setFrom("528428122@qq.com");

        mailSender.send(mail);
    }
```

