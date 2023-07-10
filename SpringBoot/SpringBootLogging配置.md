## 配置与使用

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
    <version>2.7.9</version>
    <scope>compile</scope>
</dependency>
```

> 一般不需要单独导入依赖，因为SpringBoot项目中默认集成有该依赖，如 `spring-boot-starter-web`



### 配置yml文件

```yml
# application.yml / application-xxx.yml
logging:
    level:
        root: warn
        # 包名: 日志级别
        cn.tangshh: info
        cn.tangshh.common.util: debug
    # 以下为非必要参数
    # 在需要的环境文件配置即可，logback会读取这些配置
    file:
        path:
        name:
    pattern:
        console:
        file:
    charset:
        console:
        file:
```

> 这里配置了demo项目包路径，级别是 `info`，此时 `cn.tangshh` 下所有目录的日志都会被限定为 `info` ，同样也可以继续细化里层的日志级别。当非 `cn.tangshh` 目录输出日志是会取 `root` 配置为默认级别

### 配置xml文件

> `resource` 下创建文件 `logback-spring.xml` ，文件名不能用 `logback.xml` 因为加载顺序在yml之前，会导致获取不到属性
> 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="30 seconds" debug="false">
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!-- 获取YML日志文件路径 -->
    <springProperty scope="context" name="logFilePath" source="logging.file.path" defaultValue="logs/"/>
    <!-- 获取YML日志文件名称 -->
    <springProperty scope="context" name="logFileName" source="logging.file.name" defaultValue="spring.log"/>
    <!-- 获取YML控制台日志输出格式配置 -->
    <springProperty scope="context" name="consolePattern" source="logging.pattern.console"
                    defaultValue="%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>
    <springProperty scope="context" name="consoleCharset" source="logging.charset.console" defaultValue="utf-8"/>
    <!-- 获取YML日志文件输出格式配置 -->
    <springProperty scope="context" name="filePattern" source="logging.pattern.file"
                    defaultValue="%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>
    <springProperty scope="context" name="fileCharset" source="logging.charset.file" defaultValue="utf-8"/>

    <!--输出到控制台配置-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- 日志格式 -->
            <pattern>${consolePattern}</pattern>
            <!-- 日志编码集 -->
            <charset>${consoleCharset}</charset>
        </encoder>
    </appender>

    <!-- 输出到文件配置 -->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logFilePath}/${logFileName}</file>
        <encoder>
            <!-- 日志格式 -->
            <pattern>${filePattern}</pattern>
            <!-- 日志编码集 -->
            <charset>${fileCharset}</charset>
        </encoder>
        <!-- 日志文件压缩配置 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 文件名格式配置 -->
            <fileNamePattern>${logFilePath}/%d{yyyy-MM}/%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <!-- 日志最大大小 -->
            <maxFileSize>50MB</maxFileSize>
            <!-- 历史最大保留 -->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
    </appender>

    <!-- 日志级别放高点，避免出现yml配置了，日志却输出不了 -->
    <root level="trace">
        <!-- dev 环境输出到控制台 -->
        <springProfile name="dev">
            <appender-ref ref="console"/>
        </springProfile>
        <!-- test、prod 环境输出到文件 -->
        <springProfile name="test,prod">
            <appender-ref ref="file"/>
        </springProfile>
    </root>
</configuration>
```

### 使用

> 完成上述配置后就可以使用，只需要在代码中编写相关日志打印即可

#### 通过@Slf4j注解

```xml
<!-- Lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>xxx</version>
</dependency>
```


```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
class RedisUtil {
    public static String get(String key) {
        try {
            String cacheValue = null;
            // ...
            log.debug("{} 缓存值：{}", key, cacheValue);
            // log.info("{} 缓存值：{}", key, cacheValue);
            return cacheValue;
        } catch (Exception e) {
            log.warn("{} 获取缓存值失败 原因：{}", key, e.getMessage());
            // log.error("{} 获取缓存值失败", key, e);
        }
        return null;
    }
}
```
