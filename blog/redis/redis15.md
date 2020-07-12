#### SpringBoot集成Redis

SpringBoot2.X以后，原来底层使用的Jredis替换成了lettuce，底层是Netty，实例可以在多个县城中共享，不存在线程不安全问题。

查找redis默认配置文件
![redis](../pic/redis/redis2.png)

```

@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    //如果没有自定义redisTemplate，这个redisTemplate才会生效，所以我们要自己定义一个
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    //默认的是Object, Object，使用的时候需要类型强转十分麻烦，需要自定义，redis对象也都是需要序列化的，这里也没序列化，自定要指定序列化规则
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    //因为大部分时候使用的都是string类型，所以默认配置了string类型
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}

```
其中核心配置默认文件 RedisProperties.class
```
@ConfigurationProperties(
    prefix = "spring.redis"
)
public class RedisProperties {
    private int database = 0;
    private String url;
    private String host = "localhost";
    private String password;
    private int port = 6379;
    private boolean ssl;
    private Duration timeout;
    private String clientName;
    private RedisProperties.Sentinel sentinel;
    private RedisProperties.Cluster cluster;
    private final RedisProperties.Jedis jedis = new RedisProperties.Jedis();
    private final RedisProperties.Lettuce lettuce = new RedisProperties.Lettuce();
}
```

Spring Data Redis底层实现有两种方式：Jedis和lettuce，直接看RedisConnectionFactory的实现就能看出：
![redis](../pic/redis/redis3.png)

**Jredis底层实现的很多类都不存在，就算配置也不生效，所以我们配置的时候，一般默认配置lettuce版本的就好，2.x默认都是lettuce版本**

```
#基本类型操作
redisTemplate.opsForValue();
redisTemplate.opsForSet();
redisTemplate.opsForList();
redisTemplate.opsForHash();
redisTemplate.opsForZSet();
redisTemplate.opsForGeo();
redisTemplate.opsForHyperLogLog();


```

RedisTemplate
```
    #序列化配置
    @Nullable
    private RedisSerializer<?> defaultSerializer;
    @Nullable
    private ClassLoader classLoader;
    @Nullable
    private RedisSerializer keySerializer = null;
    @Nullable
    private RedisSerializer valueSerializer = null;
    @Nullable
    private RedisSerializer hashKeySerializer = null;
    @Nullable
    private RedisSerializer hashValueSerializer = null;
    private RedisSerializer<String> stringSerializer = RedisSerializer.string();


    public void afterPropertiesSet() {
        super.afterPropertiesSet();
        boolean defaultUsed = false;
        if (this.defaultSerializer == null) {

            #默认序列化是JDk序列化，会有中文乱码
            this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
        }

        if (this.enableDefaultSerializer) {
            if (this.keySerializer == null) {
                this.keySerializer = this.defaultSerializer;
                defaultUsed = true;
            }
            .......
        }
```
自定义配置类RedisConfig，直接参考RedisAutoConfiguration 默认配置类
```


```
