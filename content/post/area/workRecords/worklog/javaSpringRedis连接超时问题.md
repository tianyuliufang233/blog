---
title: "JavaSpringRedis连接超时问题"
date: 2024-09-20T10:35:56+08:00
categories:
- 收件箱
- 记录
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
## java 偶然遇到redis连接超时,并且持续时间刚好15分钟
```
2024-09-19 17:34:59.237 ERROR 8 --- [  XNIO-1 task-5] c.p.s.s.e.h.core.GlobalExceptionHandler  : 全局异常处理:Redis command timed out; nested exception is io.lettuce.core.RedisCommandTimeoutException: Command timed out after 1 minute(s)
org.springframework.dao.QueryTimeoutException: Redis command timed out; nested exception is io.lettuce.core.RedisCommandTimeoutException: Command timed out after 1 minute(s)
	at org.springframework.data.redis.connection.lettuce.LettuceExceptionConverter.convert(LettuceExceptionConverter.java:70)
	at org.springframework.data.redis.connection.lettuce.LettuceExceptionConverter.convert(LettuceExceptionConverter.java:41)
	at org.springframework.data.redis.PassThroughExceptionTranslationStrategy.translate(PassThroughExceptionTranslationStrategy.java:44)
	at org.springframework.data.redis.FallbackExceptionTranslationStrategy.translate(FallbackExceptionTranslationStrategy.java:42)
	at org.springframework.data.redis.connection.lettuce.LettuceConnection.convertLettuceAccessException(LettuceConnection.java:272)
	at org.springframework.data.redis.connection.lettuce.LettuceKeyCommands.convertLettuceAccessException(LettuceKeyCommands.java:810)
	at org.springframework.data.redis.connection.lettuce.LettuceKeyCommands.keys(LettuceKeyCommands.java:227)
	at org.springframework.data.redis.connection.DefaultedRedisConnection.keys(DefaultedRedisConnection.java:111)
	at org.springframework.data.redis.cache.DefaultRedisCacheWriter.lambda$clean$4(DefaultRedisCacheWriter.java:216)
	at org.springframework.data.redis.cache.DefaultRedisCacheWriter.execute(DefaultRedisCacheWriter.java:304)
	at org.springframework.data.redis.cache.DefaultRedisCacheWriter.clean(DefaultRedisCacheWriter.java:205)
	at org.springframework.data.redis.cache.RedisCache.clear(RedisCache.java:193)
	at org.springframework.cache.interceptor.AbstractCacheInvoker.doClear(AbstractCacheInvoker.java:122)
	at org.springframework.cache.interceptor.CacheAspectSupport.performCacheEvict(CacheAspectSupport.java:505)
	at org.springframework.cache.interceptor.CacheAspectSupport.processCacheEvicts(CacheAspectSupport.java:493)
	at org.springframework.cache.interceptor.CacheAspectSupport.execute(CacheAspectSupport.java:434)
	at org.springframework.cache.interceptor.CacheAspectSupport.execute(CacheAspectSupport.java:345)
	at org.springframework.cache.interceptor.CacheInterceptor.invoke(CacheInterceptor.java:64)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:750)
	at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:123)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:388)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:750)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:750)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:692)
```

## 参考
[lettuce 间歇性发生 RedisCommandTimeoutException 的深层原理及解决方案](https://www.cnblogs.com/wingcode/p/14527107.html)
[tcp_retries2 tcp出方向重传](https://pracucci.com/linux-tcp-rto-min-max-and-tcp-retries2.html)
[TCP 连接异常的问题](https://zhuanlan.zhihu.com/p/390939380)
[Redis Lettuce长时间超时问题](https://www.cnblogs.com/songjiyang/p/16809694.html)
