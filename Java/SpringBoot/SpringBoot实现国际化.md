## 什么是国际化

国际化（Internationalization 简称 I18n）是指软件开发时应该具备支持多种语言和地区的功能。换句话说就是，开发的软件需要能同时应对不同国家和地区的用户访问，并根据用户地区和语言习惯，提供相应的、符合用具阅读习惯的页面和数据，例如，为中国用户提供汉语界面显示，为国外用户提供提供外语界面显示

## 国际化配置

### 导入依赖

需要用到的依赖项

```xml
<!-- 用于配置cookie和拦截器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 创建国际化文件

添加国际化属性文件

```text
resources
├─ ...
└─ i18n
    ├─ messages.properties
    ├─ messages_en_US.properties
    ├─ messages_zh_CN.properties
    ├─ ....
    └─ messages_xx_XX.properties
```

```properties
# messages.properties & messages_zh_CN.properties
name.blank=名称不能为空
age.value=年龄不能小于 {0}
```

```properties
# messages_en_US.properties
name.blank=name cannot be empty
age.value=age cannot be less than {0}
```

### 配置文件路径

添加yml配置

```yml
# application.yml
spring:
  messages:
    basename: i18n.messages # 扫描 resources 下的资源
```

!> 路径根据自己的配置，支持 `a/b` `a.b` 的写法，b 为文件名（不需要后缀），且支持配置多个 `a.b,a.c`，但不支持配置文件夹路径 `i18n/*` `i18n.*`，下面会提供一种 [解决思路](/Java/SpringBoot/SpringBoot实现国际化.md?id=优化配置) ，另外 messages_xx_XX 的文件不需要都配上，国际化会匹配对应前缀的语言配置文件

### 添加国际化配置类

添加配置项

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.i18n.CookieLocaleResolver;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;

import java.util.Locale;

/**
 * 国际化配置类(类名任意)
 */
@Configuration
public class I18nMessageConfig implements WebMvcConfigurer {

    /**
     * 配置国际化Cookie
     */
    @Bean
    public LocaleResolver localeResolver() {
        CookieLocaleResolver localeResolver = new CookieLocaleResolver();
        localeResolver.setCookieName("localeCookie");
        // 设置默认区域
        localeResolver.setDefaultLocale(Locale.CHINA);
        // 设置cookie有效期.
        localeResolver.setCookieMaxAge(3600);
        return localeResolver;
    }

    /**
     * 配置国际化参数 /xxx?lang=zh_CN
     */
    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor interceptor = new LocaleChangeInterceptor();
        interceptor.setParamName("lang");
        return interceptor;
    }

    /**
     * 注册拦截器
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }
}
```

### 使用方法

只需要在任意接口后面添加国际化参数即可切换语言

**示例：**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.context.i18n.LocaleContextHolder;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 测试接口
 */
@RestController
@RequestMapping("/test")
public class TestController {
    @Autowired
    private MessageSource message;

    /**
     * 国际化测试
     */
    @GetMapping("/i18n/{key}")
    public String testLocale(@PathVariable String key, String... params) {
        return message.getMessage(key, params, LocaleContextHolder.getLocale());
    }
}
```

访问接口 `http://127.0.0.1:port/test/i18n/age.value?params=18` 与 `http://127.0.0.1:port/test/i18n/age.value?params=18&lang=en_us` 就可以获得不同语言的信息。如果想在 **Thymeleaf** 里使用只需要在 `th:xx='#{xxx.xxx}'`，并且通过 `CookieLocaleResolver` 实现了cookie 记录，所以当请求携带一次后其他接口不带也会采用同样的语言，持续时间取决与 `CookieMaxAge` 设置的时间

!> `lang=xx_xx` 并不是必传项，当无该值时会取设置的默认语言，如 `localeResolver.setDefaultLocale(Locale.CHINA)` ，配置了默认语言为中文

## 优化配置

默认的国际化配置需要一个个配，如像下列这样分模块/功能管理就会十分的麻烦，而且每次新增都要去配置中添加文件路径

```text
resources
├─ ...
└─ i18n
    ├─ messages.properties
    ├─ messages_xx_XX.properties
    ├─ common.properties
    ├─ common_xx_XX.properties
    └─ ....
```

```yml
# application.yml
spring:
  messages:
    basename: i18n.messages,i18n.common,...
```

下面对该问题进行优化

### 添加依赖

这个主要是用下工具类，导不导都可以取决于个人开发习惯

```xml
<!-- 工具包 -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <!-- 版本自己选 -->
    <version>5.8.15</version>
</dependency>
```

### 修改文件路径配置

```yml
# application.yml
spring:
  messages:
    basename: i18n.* #或i18n/*
```

### 修改配置类

```java

import cn.hutool.core.util.StrUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.context.MessageSourceProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.i18n.CookieLocaleResolver;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.Locale;
import java.util.stream.Collectors;
/**
 * 国际化配置类(类名任意)
 */
@Configuration
public class I18nMessageConfig implements WebMvcConfigurer, InitializingBean {
    @Autowired
    private ResourceBundleMessageSource messageSource;
    @Autowired
    private MessageSourceProperties properties;

    @Bean
    public LocaleResolver localeResolver() {
        // ...
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        // ...
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // ...
    }

    /**
     * 新增实例化后操作
     */
    @Override
    public void afterPropertiesSet() {
        String path = properties.getBasename();
        if (StrUtil.isNotBlank(path) && path.matches("^\\w*([./](\\*|\\w*))+$")) {
            try {
                // 以下写法是为了跨模块打包时也能加载到，如文件定义在common模块下在server模块打jar包，一样可以正常使用
                PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
                Resource[] resources = resolver.getResources(ResourceLoader.CLASSPATH_URL_PREFIX + path.replace(".", "/"));
                List<String> baseNames = Arrays.stream(resources).filter(e -> {
                    String filename = e.getFilename();
                    return filename != null && !filename.matches("\\w*[-_][a-zA-z]{2}_[a-zA-Z]{2}\\..*");
                }).map(resource -> {
                    String[] split = resource.getFilename().split("\\.");
                    return path.replace("*", split[0]);
                }).collect(Collectors.toList());
                messageSource.setBasenames(baseNames.toArray(new String[]{}));
            } catch (IOException e) {
                log.error("i18n file path {} not found file/directory", path, e);
            }
        }
    }
}

```

到这里优化包扫描的工作就完成了，以上仅仅只是提供一个思路，具体操作上取决于实际路径/实际情况
