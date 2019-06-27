# redis-auth-cache
Spring Redis Authentication Store And Caching

* Running redis on Docker

`docker run -p 6379:6379 --name some-redis -d redis`

* Redis with persistence

`docker run -p 6379:6379 --name some-redis -d redis redis-server --appendonly yes`


**RedisConnection and RedisConnectionFactory**

Jedis is used as Redis java client. Lettuce is another alternative that is supported by Spring. 
Connection to redis is provided by RedisConnection, and active RedisConnection objects are created through RedisConnectionFactory.


* Configure Jedis Connector with default settings 

```java
@Configuration
class AppConfig {

  @Bean
  public JedisConnectionFactory redisConnectionFactory() {
    return new JedisConnectionFactory();
  }
}
```

* Configure Jedis Connector with custom config

```java
@Configuration
class AppConfig {

  @Bean
      JedisConnectionFactory jedisConnectionFactory() {
  
          RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
  
          redisStandaloneConfiguration.setHostName("localhost");
          redisStandaloneConfiguration.setPort(6379);
          redisStandaloneConfiguration.setDatabase(0);
          redisStandaloneConfiguration.setPassword(RedisPassword.none());
  
          return new JedisConnectionFactory(redisStandaloneConfiguration);
      }

}
```
**RedisTemplate and working with objects**

RedisTemplate offers a high-level abstraction for Redis interactions. 
It provides interfaces as HashOperations, ListOperations, SetOperations, ValueOperations etc. to manage Redis operations.

* Configure RedisTemplate

```java
@Configuration
class AppConfig {

  @Bean
      public RedisTemplate<String, Object> redisTemplate() {
          RedisTemplate<String, Object> template = new RedisTemplate<>();
          template.setConnectionFactory(jedisConnectionFactory());
          return template;
      }

}
```

* Redis commands via RedisTemplate

After accessing redisTemplate (autowire), it is possible to use its operations that matches redis commands.

`redisTemplate.opsForValue().set("myKey", "myValue");`

One thing to note here is that by default **RedisTemplate** uses java serialization, you may see keys look a little weird if you reach them via **redis-cli**, for example:

`"\xac\xed\x00\x05t\x00\x03myKey"`

When we are retrieving via redisTemplate it will deserialize it in same manner so there wont be problem. But if you like to change this check out changing default serialization:

https://stackoverflow.com/questions/13215024/why-is-data-getting-stored-with-weird-keys-in-redis-when-using-jedis-with-spring?lq=1


**Redis as Spring Session Storage**

Adding the following dependency handles auto-configuration of spring session storage as redis.

```
<dependency>
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session-data-redis</artifactId>
</dependency>
```

Having this dependency and a working redis client (JedisConnection) that manages connections to server 
will be enough to automatically add session filter to security chain with redis configurations. Example session stored:
```
127.0.0.1:6379> KEYS *
1) "spring:session:index:org.springframework.session.FindByIndexNameSessionRepository.PRINCIPAL_NAME_INDEX_NAME:user"
2) "spring:session:sessions:6c39c36c-25e2-4412-8ae8-770ea6d39fac"
3) "spring:session:expirations:1561624560000"
4) "spring:session:sessions:expires:6c39c36c-25e2-4412-8ae8-770ea6d39fac"
```

Note here that we created our redis connector as bean with the given configuration above. 
Another way to create redis connection instance can be with auto configuration. we could say spring-data-redis 
to configure and start redis connection for us with adding following line in application.properties file:

`spring.session.store-type=redis`

So just having dependency and this parameter in application.properties is enough to generate redis instance and store session data in redis.

Additional possible configuration for application.properties

```
spring.redis.host=localhost # Redis server host.
spring.redis.password= # Login password of the redis server.
spring.redis.port=6379 # Redis server port.
```

