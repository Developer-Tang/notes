## 什么时RedisTemplate

RedisTemplate 是简化 Redis 数据访问代码的辅助类

## 导入依赖

<!-- tabs:start -->
<!-- tab:Maven -->

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>

```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'org.springframework.boot:spring-boot-starter-data-redis-reactive'
```

<!-- tabs:end -->

## 添加配置

<!-- tabs:start -->
<!-- tab:yml/yaml -->

```yaml
spring:
  data:
    redis:
      database: 0
      host: 127.0.0.1
      password: 
```

<!-- tabs:end -->

## 添加代码

### 自定义模板类

这里我使用了自定义的模板类，采用类 SpringBoot 框架自身的 Jackson 序列化器去处理要缓存的数据，这是个人喜好，也是为了方便抽取静态操作工具类

```java
/**
 * 自定义Redis操作模板
 *
 * @author Tang
 * @version v1.0
 */
public class IRedisTemplate extends RedisTemplate<String, Object> {

    public IRedisTemplate(ObjectMapper mapper) {
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer(mapper);
        setKeySerializer(stringRedisSerializer);
        setHashKeySerializer(stringRedisSerializer);
        setValueSerializer(jsonRedisSerializer);
        setHashValueSerializer(jsonRedisSerializer);
    }

    public IRedisTemplate(RedisConnectionFactory connectionFactory) {
        this(new ObjectMapper());
        setConnectionFactory(connectionFactory);
        afterPropertiesSet();
    }

    public IRedisTemplate(RedisConnectionFactory connectionFactory, ObjectMapper mapper) {
        this(mapper);
        setConnectionFactory(connectionFactory);
        afterPropertiesSet();
    }
}
```

### 添加配置类

这里可以看到我从框架中直接注入了 ObjectMapper 这个对象，这个是 SpringBoot 框架中为了 mvc 创建的 json 序列化器，这里 copy 了一个新对象出来，并设置了将对象的类名写入到缓存数据中`{"@class":"cn.tangshh.xx",...}`，这样在查询时就能通过 Redis 框架自动去反序列化拿到对应的类对象。如果不加这个会出现反序列化的对象类型为 LinkedHashMap，这时候要使用这个对象还需要考虑转换类型和取值就很麻烦

```java
/**
 * Redis 配置
 *
 * @author Tang
 * @version v1.0
 */
@Slf4j
@Configuration
public class RedisConfig {

    @Bean
    public IRedisTemplate iRedisTemplate(RedisConnectionFactory connectionFactory, ObjectMapper mapper) {
        ObjectMapper copy = mapper.copy();
        // 设置类型到值中，实现查询结果为对象是可以直接解析，但并不支持自动的跨类型转换
        copy.activateDefaultTyping(copy.getPolymorphicTypeValidator(), ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        return new IRedisTemplate(connectionFactory, copy);
    }
}
```

### 添加工具类

这里就提供个简单的示例，感兴趣的可以自行扩展

<!-- tabs:start -->
<!-- tab:Redis公共操作工具 -->

```java
import cn.hutool.core.util.StrUtil;
import cn.hutool.extra.spring.SpringUtil;
import cn.tangshh.universal.cache.handler.IRedisTemplate;
import cn.tangshh.universal.core.exception.FunctionException;
import jakarta.annotation.Nullable;
import jakarta.validation.constraints.NotNull;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.connection.DataType;
import org.springframework.data.redis.core.RedisCallback;

import java.util.Collection;
import java.util.Date;
import java.util.Properties;
import java.util.Set;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;


/**
 * 基于Redis公共方法的缓存操作工具
 *
 * @author Tang
 * @version v1.0
 */
@Slf4j
public class RedisUtil {
    protected final static double VERSION;
    protected final static IRedisTemplate TEMPLATE;

    static {
        TEMPLATE = SpringUtil.getBean(IRedisTemplate.class);
        String version = TEMPLATE.execute((RedisCallback<String>) connection -> {
            Properties server = connection.serverCommands().info("server");
            return server != null ? server.getProperty("redis_version") : null;
        });
        VERSION = StrUtil.isNotBlank(version) ? Double.parseDouble(StrUtil.replaceLast(version, ".", "0")) : -1;
    }

    protected RedisUtil() {
    }

    protected static void verifyCompatible(double needVersion) {
        if (VERSION < needVersion) {
            throw new FunctionException("当前Redis版本不兼容该命令");
        }
    }

    /**
     * 查询键是否存在
     *
     * @param key key
     * @return boolean
     */
    public static boolean hasKey(@NotNull String key) {
        return Boolean.TRUE.equals(TEMPLATE.hasKey(key));
    }

    /**
     * 删除键
     *
     * @param key key
     * @return boolean
     */
    public static boolean del(@NotNull String key) {
        return del(key, false);
    }

    /**
     * 删除键
     *
     * @param key            key
     * @param asyncDoubleDel 异步双删
     * @return boolean
     */
    public static boolean del(@NotNull String key, boolean asyncDoubleDel) {
        boolean equals = Boolean.TRUE.equals(TEMPLATE.delete(key));
        if (asyncDoubleDel) {
            CompletableFuture.runAsync(() -> TEMPLATE.delete(key), CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS));
        }
        return equals;
    }

    /**
     * 批量删除键
     *
     * @param keys keys
     * @return long
     */
    public static long del(Collection<String> keys) {
        return del(keys, false);
    }


    /**
     * 批量删除键
     *
     * @param keys           keys
     * @param asyncDoubleDel 异步双删
     * @return long
     */
    public static long del(Collection<String> keys, boolean asyncDoubleDel) {
        Long delNum = TEMPLATE.delete(keys);
        long l = delNum == null ? 0 : delNum;
        if (asyncDoubleDel) {
            CompletableFuture.runAsync(() -> TEMPLATE.delete(keys), CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS));
        }
        return l;
    }

    /**
     * 设置键的有效时间
     *
     * @param key       key
     * @param validTime 有效时间
     * @return boolean
     */
    public static boolean expire(@NotNull String key, long validTime) {
        return expire(key, validTime, TimeUnit.SECONDS);
    }

    /**
     * 设置键的有效时间
     *
     * @param key    key
     * @param expire valid time
     * @param unit   unit
     * @return boolean
     */
    public static boolean expire(@NotNull String key, long expire, @NotNull TimeUnit unit) {
        if (expire > 0) {
            return Boolean.TRUE.equals(TEMPLATE.expire(key, expire, unit));
        }
        return false;
    }

    /**
     * 设置键的有效时间
     *
     * @param key  key
     * @param date valid time
     * @return boolean
     */
    public static boolean expire(@NotNull String key, @NotNull Date date) {
        return Boolean.TRUE.equals(TEMPLATE.expireAt(key, date));
    }

    /**
     * 键重命名
     *
     * @param oldKey 旧key
     * @param newKey 新key
     */
    public static void rename(@NotNull String oldKey, @NotNull String newKey) {
        TEMPLATE.rename(oldKey, newKey);
    }

    /**
     * 通过表达式查询键（*匹配任意内容）
     * example: user:*,2023*,*09
     *
     * @param keyExpr key expression
     * @return {@link Set}<{@link String}>
     */
    @Nullable
    public static Set<String> keys(@NotNull String keyExpr) {
        return TEMPLATE.keys(keyExpr);
    }

    /**
     * 查询键剩余有效时间
     *
     * @param key key
     * @return {@link Long}
     */
    @Nullable
    public static Long ttl(@NotNull String key) {
        return ttl(key, TimeUnit.SECONDS);
    }

    /**
     * 查询键剩余有效时间
     *
     * @param key  key
     * @param unit unit
     * @return {@link Long}
     */
    @Nullable
    public static Long ttl(@NotNull String key, @NotNull TimeUnit unit) {
        return TEMPLATE.getExpire(key, unit);
    }

    /**
     * 查询键的存储类型
     *
     * @param key key
     * @return {@link DataType}
     */
    @Nullable
    public static DataType type(@NotNull String key) {
        return TEMPLATE.type(key);
    }
}
```

<!-- tab:Redis string类型工具 -->

```java
import jakarta.annotation.Nullable;
import jakarta.validation.constraints.NotNull;
import org.springframework.data.redis.core.ValueOperations;

import java.time.Duration;
import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeUnit;

/**
 * 基于Redis String类型的缓存操作工具<br/>
 * 部分返回值通过泛型方式返回，需要注意的是如果序列化时的类型不一样则会报错
 *
 * @author Tang
 * @version v1.0
 */
@SuppressWarnings("unchecked")
public final class RedisStrUtil extends RedisUtil {
    private final static ValueOperations<String, Object> OPERATIONS;

    static {
        OPERATIONS = TEMPLATE.opsForValue();
    }

    private RedisStrUtil() {
    }

    /**
     * 追加值
     *
     * @param key   key
     * @param value value
     * @return {@link Integer}
     */
    @Nullable
    public static Integer append(@NotNull String key, String value) {
        return OPERATIONS.append(key, value);
    }

    /**
     * 值递增 键不存在则新增并从零开始计算
     *
     * @param key       key
     * @param increment increment value
     * @return {@link Double}
     */
    @Nullable
    public static Double incr(@NotNull String key, double increment) {
        return OPERATIONS.increment(key, increment);
    }

    /**
     * 值递增 键不存在则新增并从零开始计算
     *
     * @param key       key
     * @param increment increment value
     * @return {@link Long}
     */
    @Nullable
    public static Long incr(@NotNull String key, long increment) {
        return OPERATIONS.increment(key, increment);
    }

    /**
     * 值递增1 键不存在则新增并从零开始计算
     *
     * @param key key
     * @return {@link Long} increment value
     */
    @Nullable
    public static Long incr(@NotNull String key) {
        return incr(key, 1);
    }

    /**
     * 值递减
     *
     * @param key       key
     * @param decrement decrement value
     * @return {@link Long}
     */
    @Nullable
    public static Long decr(@NotNull String key, long decrement) {
        return OPERATIONS.decrement(key, decrement);
    }

    /**
     * 值递减1
     *
     * @param key key
     */
    @Nullable
    public static Long decr(@NotNull String key) {
        return decr(key, 1);
    }

    /**
     * 设置值
     *
     * @param key   key
     * @param value value
     */
    public static void set(@NotNull String key, Object value) {
        OPERATIONS.set(key, value);
    }

    /**
     * 设置值和有效时间
     *
     * @param key       key
     * @param value     value
     * @param validTime valid time (sec)
     */
    public static void setEx(@NotNull String key, Object value, long validTime) {
        if (validTime > 0) {
            OPERATIONS.set(key, value, validTime);
        } else if (validTime == -1) {
            OPERATIONS.set(key, value);
        }
    }

    /**
     * 设置值和有效时间
     *
     * @param key       key
     * @param value     value
     * @param validTime valid time
     * @param unit      unit
     */
    public static void setEx(@NotNull String key, Object value, long validTime, @NotNull TimeUnit unit) {
        if (validTime > 0) {
            OPERATIONS.set(key, value, validTime, unit);
        } else if (validTime == -1) {
            OPERATIONS.set(key, value);
        }
    }

    /**
     * 设置值和有效时间
     *
     * @param key     key
     * @param value   value
     * @param timeout valid time
     */
    public static void setEx(@NotNull String key, Object value, @NotNull Duration timeout) {
        OPERATIONS.set(key, value, timeout);
    }

    /**
     * 如果键不存在则设置值
     *
     * @param key   key
     * @param value value
     * @return boolean
     */
    public static boolean setNx(@NotNull String key, Object value) {
        return Boolean.TRUE.equals(OPERATIONS.setIfAbsent(key, value));
    }

    /**
     * 如果键不存在则设置值和有效时间
     *
     * @param key       key
     * @param value     value
     * @param validTime valid time (sec)
     * @return boolean
     */
    public static boolean setNx(@NotNull String key, Object value, long validTime) {
        return setNx(key, value, validTime, TimeUnit.SECONDS);
    }

    /**
     * 如果键不存在则设置值和有效时间
     *
     * @param key       key
     * @param value     value
     * @param validTime valid time
     * @param unit      unit
     * @return boolean
     */
    public static boolean setNx(@NotNull String key, Object value, long validTime, @NotNull TimeUnit unit) {
        return Boolean.TRUE.equals(OPERATIONS.setIfAbsent(key, value, validTime, unit));
    }

    /**
     * 设置多个键值
     *
     * @param map {key:value,key:value,...}
     */
    public static void batchSet(@NotNull Map<String, Object> map) {
        OPERATIONS.multiSet(map);
    }

    /**
     * 如果键都不存在，则设置多个键值
     *
     * @param map {key:value,key:value,...}
     * @return boolean
     */
    public static boolean batchSetNx(@NotNull Map<String, Object> map) {
        return Boolean.TRUE.equals(OPERATIONS.multiSetIfAbsent(map));
    }

    /**
     * 获取值
     *
     * @param key key
     * @return {@link String}
     */
    @Nullable
    public static <T> T get(@NotNull String key) {
        return (T) OPERATIONS.get(key);
    }

    /**
     * 获取多个值
     *
     * @param keys keys
     * @return {@link List}<{@link String}>
     */
    @Nullable
    public static <T> List<T> batchGet(@NotNull Collection<String> keys) {
        return (List<T>) OPERATIONS.multiGet(keys);
    }

    /**
     * 获取并设置新值
     *
     * @param key      key
     * @param newValue new value
     * @return {@link T}
     */
    @Nullable
    public static <T> T getAndSet(@NotNull String key, @NotNull T newValue) {
        return (T) OPERATIONS.getAndSet(key, newValue);
    }

    /**
     * 获取并删除值
     *
     * @param key key
     * @return {@link String}
     */
    @Nullable
    public static <T> T getAndDel(@NotNull String key) {
        return (T) OPERATIONS.getAndDelete(key);
    }
}
```

<!-- tabs:end -->

### 测试

```java
/**
 * 演示实体类
 */
@Data
@AllArgsConstructor
class IUser {
    private Long id;
    private String name;
    private String mobile;
    private Integer sex;
}

void test() {
    IUser user = new IUser(1001, "Tang", "17888888888", 1);
    RedisStrUtil.setEx("user1001", user, 30);

    IUser cache = RedisStrUtil.get("user1001");
}
```
