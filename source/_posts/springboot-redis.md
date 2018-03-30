title: springboot下使用Redis
date: 2018-03-30 19:51:53
---
# 引入依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
            <version>1.4.1.RELEASE</version>
        </dependency>
    </dependencies>
    
```
<!--more-->
# application.properties配置

> 正常配置即可

```
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379  
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8  
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1  
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8  
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0  
# 连接超时时间（毫秒）
spring.redis.timeout=0
```
# RedisService 示例

```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Set;
import java.util.concurrent.TimeUnit;


@Service
public class RedisService<T>  {

    protected RedisTemplate redisTemplate;
    @Autowired
    protected StringRedisTemplate stringRedisTemplate;

    @Autowired
    public void setRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        // 使用Jackson2JsonRedisSerialize 替换默认序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 设置value的序列化规则和 key的序列化规则
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer(new ObjectMapper()));
        redisTemplate.afterPropertiesSet();
        this.redisTemplate = redisTemplate;
    }

    public List<?> getClients(){
        return redisTemplate.getClientList();
    }

    /**
     * 保存数据
     *
     * @param key   键
     * @param value 值
     * @throws Exception
     */
    public void set(String key, T value) throws Exception {
        redisTemplate.opsForValue().set(key, value);
    }

    /**
     * 保存数据并设置有效时间
     *
     * @param key      键
     * @param value    值
     * @param liveTime 有效时间(秒)
     * @throws Exception
     */
    public void set(String key, T value, Long liveTime) throws Exception {
        redisTemplate.opsForValue().set(key, value, liveTime, TimeUnit.SECONDS);
    }

    /**
     * 根据key获取对象信息
     *
     * @param key 键
     * @return 值
     * @throws Exception
     */
    public Object get(String key) {
        return redisTemplate.opsForValue().get(key);
    }

    /**
     * 删除 key（支持多个key）
     *
     * @param keys 键数组
     * @throws Exception
     */
    public void del(final String... keys) {
        redisTemplate.delete(Arrays.asList(keys));
    }

    /**
     * 设置 key 生效时间
     *
     * @param key      键
     * @param liveTime 秒
     * @return true: 成功, false: 失败
     * @throws Exception
     */
    public boolean setExpire(String key, Long liveTime) {
        if (StringUtils.isEmpty(key) || liveTime <= 0) {
            return false;
        }
        return redisTemplate.expire(key, liveTime, TimeUnit.SECONDS);
    }

    /**
     * 设置 key 生效时间
     *
     * @param key      键
     * @param liveDate 日期
     * @return true: 成功, false: 失败
     * @throws Exception
     */
    public boolean setExpire(String key, Date liveDate) {
        if (StringUtils.isEmpty(key) || liveDate == null) {
            return false;
        }
        return redisTemplate.expireAt(key, liveDate);
    }

    /**
     * 是否存在 key
     *
     * @param key 键
     * @return true: 存在, false: 不存在
     */
    public boolean isExist(String key) {
        return redisTemplate.hasKey(key);
    }

    /**
     * 往区域(map)里添加数据
     *
     * @param region 区域key
     * @param key    数据key
     * @param value  值
     * @throws Exception
     */
    public void addRegion(String region, String key, Object value) throws Exception {
        if (!StringUtils.hasText(region)) {
            throw new Exception("region is not null");
        }
        redisTemplate.opsForHash().put(region, key, value);
    }

    /**
     * 从区域里面获取指定key的数据
     *
     * @param region 区域key
     * @param key    数据key
     * @return 值
     * @throws Exception
     */
    public Object getRegion(String region, String key) throws Exception {
        if (!StringUtils.hasText(region)) {
            throw new Exception("region is not null");
        }
        return redisTemplate.opsForHash().get(region, key);
    }

    /**
     * 从区域里面删除指定key数据
     *
     * @param region 区域key
     * @param key    数据key
     * @return true: 成功, false: 失败
     */
    public boolean delByRegion(String region, String key) {
        try {
            redisTemplate.opsForHash().delete(region, key);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 新增set类型数据
     * @param key set的key
     * @param value 数据值
     */
    public Long addSet(String key,Set value){
       return redisTemplate.opsForSet().add(key,value.toArray(new Object[value.size()]));
    }

    /**
     * 根据key值获取set数据
     * @param key
     * @return
     */
    public Set getSet(String key){
        return redisTemplate.opsForSet().members(key);
    }

    /**
     * 移除set里面的值
     * @param key set的key值
     * @param value 需要移除的值
     */
    public Long removeSetValue(String key,Object... value){
        return redisTemplate.opsForSet().remove(key,value);
    }
    /**
     * 获取redis数据库大小
     *
     * @return 数据库大小
     */
    public long dbSize() {
        return (long) redisTemplate.execute((RedisCallback<Long>) connection -> connection.dbSize());
    }

    /**
     * 发送消息
     *
     * @param channel 管道名称
     * @param message 发送消息(字符串)
     */
    public void sendMessage(String channel, String message) {
        stringRedisTemplate.convertAndSend(channel, message);
    }

    /**
     * 发送消息
     *
     * @param channel 管道名称
     * @param message 发送消息(对象)
     */
    public void sendMessage(String channel, Object message) {
        redisTemplate.convertAndSend(channel, message);
    }


    public Long increment(String key) {
        return redisTemplate.opsForValue().increment(key, 1);
    }


}
```


*业务代码直接注入RedisService即可使用*