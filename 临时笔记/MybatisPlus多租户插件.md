?> [官方文档](https://baomidou.com/pages/aef2f2/)

## 多租户插件原理

### 实现TenantLineHandler

```java
public interface TenantLineHandler {

    /**
     * 获取租户 ID 值表达式，只支持单个 ID 值
     * <p>
     *
     * @return 租户 ID 值表达式
     */
    Expression getTenantId();

    /**
     * 获取租户字段名
     * <p>
     * 默认字段名叫: tenant_id
     *
     * @return 租户字段名
     */
    default String getTenantIdColumn() {
        return "tenant_id";
    }

    /**
     * 根据表名判断是否忽略拼接多租户条件
     * <p>
     * 默认都要进行解析并拼接多租户条件
     *
     * @param tableName 表名
     * @return 是否忽略, true:表示忽略，false:需要解析并拼接多租户条件
     */
    default boolean ignoreTable(String tableName) {
        return false;
    }
}
```

## 多租户功能实现

### 导入依赖

#### Maven配置

```xml
<!-- version版本按需选择 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3</version>
</dependency>
```

#### Gradle配置

```groovy

```

### 配置

```java
import cn.hutool.core.util.StrUtil;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.handler.TenantLineHandler;
import com.baomidou.mybatisplus.extension.plugins.inner.TenantLineInnerInterceptor;
import net.sf.jsqlparser.expression.Expression;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * MP配置类
 */
@Configuration
@MapperScan({"cn.**.mapper"})
public class MybatisPlusConfig {
    /**
     * 配置mybatis-plus拦截器
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new TenantLineInnerInterceptor(new TenantLineHandler() {
            /**
             * 获取租户id
             */
            @Override
            public Expression getTenantId() {
                return null;
            }

            /**
             * 过滤不需要多租户隔离的表
             * @param tableName 表名
             * @return boolean
             */
            @Override
            public boolean ignoreTable(String tableName) {
                return StrUtil.startWithAny(tableName, "sys_");
            }
        }));
        return interceptor;
    }
}
```
