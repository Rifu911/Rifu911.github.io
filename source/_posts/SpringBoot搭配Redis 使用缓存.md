---
layout: posts
date: 2019-05-06 09:38:46
title: 快速使用SpringBoot 和Redis 
categories: 学习工具
tags: [SpringBoot, Redis, 缓存]
---



##### 一、步骤

1.开启基于注解的缓存

在Application上标注@EnableCaching

2.在对应的serviceimpl方法上开启

@Cacheable

3.在配置文件里面配置Redis服务器的地址（默认端口为6379）

4.在自定义的CustomRedisConfiguration类里面

添加RedisCacheManager的bean

<!-- more -->

##### 二、命令注解

###### @Cacheable

###### @CachePut

###### @CacheEvict

key:指定要清除的key

allEntries = true :指定清除这个缓存中的所有数据

beforeInvocation = false :

缓存的清除是否在方法之前执行，默认代表缓存清除操作是在方法执行之后执行，出现异常则不会清除缓存

###### @Caching

组建复杂的缓存方式

> @Caching(
>
> 	cacheable = {
>		
> 		@Cacheable(value="user",key="#id")
>		
> 	},
>		
> 	put = {
>		
> 		@CachePut(value="user",key="#result.lastName"),
>		
> 		@CachePut(value="user",key="#result.dId")
>		
> 	}
>
> )

###### @CacheConfig

作用：抽取一部分公共的配置属性

> @CacheConfig(cacheNames="user")



##### 三、自定义KeyGenerator

```java
@Bean
    public KeyGenerator customKeyGenerator(){
        return new KeyGenerator(){

            @Override
            public Object generate(Object o, Method method, Object... objects) {
                return method.getName()+"["+Arrays.asList(objects).toString()+"]";
            }
        };
    }
```



##### 四、自定义序列化器RedisTemplate

```java
	@Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory){
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);					//这行代码可以使得生成的json没有[]
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(objectMapper);

        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(serializer);
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(serializer);
        redisTemplate.afterPropertiesSet();

        return redisTemplate;
    }
```

##### 五、自定义CacheManager

```java
	@Bean
    public RedisCacheManager cacheManager(RedisTemplate redisTemplate){
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisTemplate.getConnectionFactory());
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(redisTemplate.getValueSerializer()));
        return new RedisCacheManager(redisCacheWriter, redisCacheConfiguration);
    }
```





