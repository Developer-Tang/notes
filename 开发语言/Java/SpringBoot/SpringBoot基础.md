## 启动类注解

**@SpringBootApplication**

- **@SpringBootConfiguration** ：组合了 @Configuration 注解，实现配置文件的功能
- **@EnableAutoConfiguration** ：打开自动配置的功能，也可以关闭某个自动配置的选项。 @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
- **@ComponentScan** ：Spring组件扫描

## 支持配置文件格式

<!-- tabs:start -->
<!-- tab:properties -->

```properties
database.username=root
database.password=123456
```

<!-- tab:yml/yaml -->

```yaml
database:
  username: root
  password: 123456
```

<!-- tabs:end -->

## 自动配置原理

`@EnableAutoConfiguration` (开启自动配置) 该注解引入了 `AutoConfigurationImportSelector` ，该类中的方法会扫描所有存在 `META-INF/spring.factories` 的jar包

## 配置文件加载顺序

`bootstrap.properties` > `bootstrap.yml` > `application.properties`> `application.yml` > `application.yaml`

## 读取配置的方式

!> 需要注意的是这些类都是通过 spring 注册的 bean，如果是普通类就需要通过实现 `ApplicationContextAware` 来获取bean对象

<!-- tabs:start -->
<!-- tab:@Value -->

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;

@Slf4j
@Configuration
public class ServerInfo {
    @Value("${server.port}")
    private Integer port;

    public void printPort() {
        log.info("server port: {}", port);
    }
}
```

<!-- tab:@ConfigurationProperties -->

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.boot.context.properties.ConfigurationProperties;

@Slf4j
@Configuration
@ConfigurationProperties(prefix = "server")
public class ServerInfo {
    private Integer port;

    public void setPort(Integer port) {
        this.port = port;
    }

    public void printPort() {
        log.info("server port: {}", port);
    }
}
```

<!-- tab:Environment -->

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;

@Slf4j
@Configuration
public class ServerInfo {
    @Resource
    private Environment env;

    public void printPort() {
        // env.getProperty("xx") 默认返回的是String，可以通过 env.getProperty("xx",Xx.Class) 指定类型，但类型兼容会报错
        log.info("server port: {}", env.getProperty("server.port"));
    }
}
```

<!-- tab:@PropertySource -->

`@PropertySource` 主要用于指定使用的属性文件

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

@Slf4j
@Configuration
@PropertySource("classpath:config/application.properties")
public class ServerInfo {
    @Resource
    private Environment env;

    public void printPort() {
        log.info("server port: {}", env.getProperty("server.port"));
    }
}
```

<!-- tabs:end -->
