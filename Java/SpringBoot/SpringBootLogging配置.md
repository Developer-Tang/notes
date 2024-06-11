## 配置与使用

### 导入依赖

一般不需要单独导入该依赖，因为SpringBoot项目的其他依赖包中默认集成有该依赖，如 `spring-boot-starter-web`

<!-- tabs:start -->
<!-- tab:Maven -->

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'org.springframework.boot:spring-boot-starter-logging'
```

<!-- tabs:end -->

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

这里配置了demo项目包路径，级别是 `info`，此时 `cn.tangshh` 下所有目录的日志都会被限定为 `info` ，同样也可以继续细化里层的日志级别。当非 `cn.tangshh` 目录输出日志是会取 `root` 配置为默认级别

### 配置xml文件

`resource` 下创建文件 `logback-spring.xml` ，文件名不能用 `logback.xml` 因为加载顺序在yml之前，会导致获取不到属性

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="30 seconds" debug="false">
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!-- 日志文件路径 -->
    <springProperty scope="context" name="logFilePath" source="logging.file.path" defaultValue="logs/"/>
    <!-- 日志文件名称 -->
    <springProperty scope="context" name="logFileName" source="logging.file.name" defaultValue="spring.log"/>
    <!-- 控制台日志输出样式配置 -->
    <springProperty scope="context" name="consolePattern" source="logging.pattern.console"
                    defaultValue="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%0.15t]){faint} %clr(%-45.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>
    <springProperty scope="context" name="consoleCharset" source="logging.charset.console" defaultValue="utf-8"/>
    <!-- 日志文件输出样式配置 -->
    <springProperty scope="context" name="filePattern" source="logging.pattern.file"
                    defaultValue="%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>
    <springProperty scope="context" name="fileCharset" source="logging.charset.file" defaultValue="utf-8"/>

    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${consolePattern}</pattern>
            <charset>${consoleCharset}</charset>
        </encoder>
    </appender>

    <!-- springProfile避免本地开发环境生产log文件 -->
    <springProfile name="test,prod">
        <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${logFilePath}/${logFileName}</file>
            <encoder>
                <pattern>${filePattern}</pattern>
                <charset>${fileCharset}</charset>
            </encoder>
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <fileNamePattern>${logFilePath}/%d{yyyy-MM}/%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
                <MaxHistory>30</MaxHistory>
                <maxFileSize>10MB</maxFileSize>
            </rollingPolicy>
        </appender>
    </springProfile>

    <root level="trace">
        <appender-ref ref="console"/>
    </root>

    <springProfile name="!dev">
        <root level="trace">
            <appender-ref ref="console"/>
            <appender-ref ref="file"/>
        </root>
    </springProfile>
</configuration>
```

### 使用

完成上述配置后就可以使用，只需要在代码中编写相关日志打印即可

#### 通过@Slf4j注解

<!-- tabs:start -->
<!-- tab:Maven -->

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'org.projectlombok:lombok'
```

<!-- tabs:end -->

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
class XxxUtil {
    public static String get(String key) {
        log.debug("entering param:{}", key);
        return null;
    }
}
```

#### 通过内部变量

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

class XxxUtil {
    private final static Logger logger = LoggerFactory.getLogger(XxxUtil.class);

    public static String get(String key) {
        logger.debug("entering param:{}", key);
        return null;
    }
}
```
