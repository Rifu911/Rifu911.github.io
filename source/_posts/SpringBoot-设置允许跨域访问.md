---
title: SpringBoot 设置允许跨域访问
date: 2018-10-31 13:40:51
categories: 学习笔记
tags: [SpringBoot, 跨域访问]
---

在SpringBoot项目中设置允许跨域访问

<!-- more -->

```java
/**
 * @Author Rifu
 * @Date 2018/10/31  13:37
 */
@Configuration
public class CorsConfig {

    private CorsConfiguration buildConfig(){
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 允许任何域名使用
        corsConfiguration.addAllowedHeader("*"); // 允许任何头
        corsConfiguration.addAllowedMethod("*"); // 允许任何方法（post、get等）
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**",buildConfig());
        return new CorsFilter(source);
    }
}

```

