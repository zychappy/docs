# Spring Integration
提供的全局锁，支持：  
Gemfire
JDBC
Redis
Zookeeper  

以Redis为例，看Spring Integration里面如何使用分布式锁
添加依赖  
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-integration</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.integration</groupId>
  <artifactId>spring-integration-redis</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
添加配置
```yml
spring:
  redis:
    port: 6379
    host: localhost

```
代码初始化
```java
@Configuration
public class RedisLockConfiguration {
  @Bean
  public RedisLockRegistry redisLockRegistry(RedisConnectionFactory redisConnectionFactory) {
    return new RedisLockRegistry(redisConnectionFactory, "spring-cloud");
  }
}
```
测试
```java
@GetMapping("test")
public void test() throws InterruptedException {
  Lock lock = redisLockRegistry.obtain("lock");
  boolean b1 = lock.tryLock(3, TimeUnit.SECONDS);
  log.info("b1 is : {}", b1);

  TimeUnit.SECONDS.sleep(5);

  boolean b2 = lock.tryLock(3, TimeUnit.SECONDS);
  log.info("b2 is : {}", b2);

  lock.unlock();
  lock.unlock();
}
```

