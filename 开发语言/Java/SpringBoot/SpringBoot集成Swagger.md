## 什么是Swagger

Swagger 是一个开源的 API 设计和文档工具，它可以帮助开发人员更快、更简单地设计、构建、文档化和测试 Restful API。Swagger 可以自动生成交互式 API 文档、客户端 SDK、服务器 stub 代码等，从而使开发人员更加容易地开发、测试和部署 API --Apifox

## 集成Swagger

这里采用较新的 SpringBoot 版本（2.6.+）与 Swagger3.0 进行集成，如果使用其他版本可能存在兼容性问题，如遇到需要自行摸索调整

### 导入依赖

```xml
<!-- 所需依赖 -->
<dependencys>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Swagger所需依赖 -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-boot-starter</artifactId>
        <version>3.0.0</version>
    </dependency>
    <!-- 工具包 -->
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.8.15</version>
    </dependency>
    <!-- 这个全凭个人喜好 -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencys>
```

### 配置文件

这里添加了一些自定义属性，可以按需自行调整，与下面的配置类中的前缀/字段保持一致即可

```yaml
# application
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher # 因为 Swagger3.0 的路径匹配规则与 SpringBoot2.6.+ 不一致所以需要添加此配置
swagger:
  enabled: true # 定义是否开启接口文档，生产环境一般为 false 关闭
  title: 脚手架 # swagger文档上显示的标题
  author: Tang # 作者
  authorUrl: https://gitee.com/Tangshh # 作者网站 
  email: tang_0416@126.com # 联系邮箱
  version: V1.0 # 版本
  authHeader: Authorization # token的请求头
```

!> 这里需要注意 `spring.mvc.pathmatch.matching-strategy:ant_path_matcher` 可以解决 `Swagger3.0` 与 `SpringBoot2.6+` 带来的问题，但也会造成与 `spring-boot-starter-actuator` 依赖的冲突

### 添加配置类

```java
import cn.hutool.core.util.StrUtil;
import cn.tangshh.common.util.RegexUtil;
import io.swagger.annotations.ApiOperation;
import io.swagger.models.auth.In;
import io.swagger.v3.oas.models.OpenAPI;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.StringUtils;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.oas.web.OpenApiTransformationContext;
import springfox.documentation.oas.web.WebMvcOpenApiTransformationFilter;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spi.service.contexts.SecurityContext;
import springfox.documentation.spring.web.plugins.Docket;

import javax.servlet.http.HttpServletRequest;
import java.util.ArrayList;
import java.util.List;

/**
 * Swagger配置
 *
 * @author Tang
 * @version v1.0
 */
@Data
@Slf4j
@EnableOpenApi
@Configuration
@ConfigurationProperties(prefix = "swagger")
public class SwaggerConfig implements WebMvcOpenApiTransformationFilter {

    private Boolean enabled = false;
    private String title = "Open Api Document";
    private String author = "";
    private String authorUrl = "";
    private String email = "";
    private String version = "1.0";
    private String authHeader = "Authorization";

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30)
                // 是否启用Swagger
                .enable(enabled)
                // 用来创建该API的基本信息，展示在文档的页面中（自定义展示的信息）
                .apiInfo(apiInfo())
                .select()
                // 扫描所有有注解的api，用这种方式更灵活
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                // 扫描所有
                .paths(PathSelectors.any())
                .build()
                // 设置安全模式，swagger可以设置访问token
                .securitySchemes(securitySchemes())
                .securityContexts(securityContexts());
    }

    /**
     * 安全模式，这里指定token通过Authorization头请求头传递
     */
    private List<SecurityScheme> securitySchemes() {
        List<SecurityScheme> apiKeyList = new ArrayList<>();
        apiKeyList.add(new ApiKey(authHeader, authHeader, In.HEADER.toValue()));
        return apiKeyList;
    }

    /**
     * 安全上下文
     */
    private List<SecurityContext> securityContexts() {
        List<SecurityContext> securityContexts = new ArrayList<>();
        securityContexts.add(
                SecurityContext.builder()
                        .securityReferences(defaultAuth())
                        .operationSelector(o -> o.requestMappingPattern().matches("/.*"))
                        .build());
        return securityContexts;
    }

    /**
     * 默认的安全上引用
     */
    private List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        List<SecurityReference> securityReferences = new ArrayList<>();
        securityReferences.add(new SecurityReference(authHeader, authorizationScopes));
        return securityReferences;
    }

    /**
     * 添加摘要信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(title)
                .description("description：...")
                .contact(new Contact(author, authorUrl, email))
                .version("version:" + version)
                .build();
    }

    /**
     * 设置文档的服务请求地址
     */
    @Override
    public OpenAPI transform(OpenApiTransformationContext<HttpServletRequest> context) {
        OpenAPI swagger = context.getSpecification();
        context.request().ifPresent(req -> {
            String scheme = "http";
            String referer = req.getHeader("Referer");
            String prefix = req.getHeader("X-Forwarded-Prefix"); // 解决 nginx 代理后请求路径不对的问题
            if (StringUtils.hasLength(referer)) {
                scheme = referer.split(":")[0];
            }
            String finalScheme = scheme + ":";
            // 重新组装servers信息
            swagger.getServers().forEach(item -> {
                String url = RegexUtil.regexReplace(item.getUrl(), RegexUtil.HTTP_STARTS_WITH, finalScheme);
                if (StrUtil.endWithAny(url, ":80", ":443")) {
                    url = url.replaceAll("(:80|:443)", StrUtil.EMPTY);
                    item.setUrl(url + StrUtil.emptyToDefault(prefix, StrUtil.EMPTY));
                }
            });
        });
        return swagger;
    }

    @Override
    public boolean supports(DocumentationType documentationType) {
        return documentationType.equals(DocumentationType.OAS_30);
    }
}
```

如上述户代码中重写了 `transform` 方法，这里主要是为了去除接口文档中 `Servers` 中端口为 `80|443` 时没有省略的问题，以及当使用 `nginx` 做反向代理/负载均衡时配置如 `location /api` 等规则时程序中获取不到对应请求前缀问题添加了 `X-Forwarded-Prefix` 请求头来解决这个问题

如果使用了 nginx 代理可以参考如下配置

```nginx
# nginx配置示例
http {
    # ...
    
    server {
        listen       80;
        # listen       443;
        server_name  0.0.0.0;
        # server_name  xxx.xxx.xxx;
        
        # ...
        
        location /api/ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            # X-Forwarded-Prefix 与规则保持一致即可
            proxy_set_header X-Forwarded-Prefix /api;
            proxy_pass http://localhost:9001/;
        }
    }
}
```

### 使用示例

在所需的类上加上对应的注解即可，以下给出示例及使用场景

```java
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.*;

@RestController
@AllArgsConstructor
@Api(tags = "用户控制器")
@RequestMapping("/user")
public class UserController {
    // ...

    @ApiOperation("登录")
    @PostMapping("/login")
    public String login(@RequestBody AuthParam param) {
        // ...
        return "token";
    }
}
```

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

@Data
@ApiModel("授权参数")
public class AuthParam {
    @ApiModelProperty(value = "用户名", required = true)
    public String username;
    @ApiModelProperty(value = "用户名", required = true)
    public String password;
}
```

启动程序后，访问 `http://127.0.0.1:{port}/swagger-ui/index.html`，这样就可以看到生成的接口文档了

### 关于鉴权拦截

因为一般程序都不会完全开放访问，会用到一些鉴权框架或者自定义鉴权拦截器，这里访问 Swagger 文档就需要去设置对应的访问权限

**自定义拦截器示例**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.*;

@EnableWebMvc
@Configuration
public class WebbAdapter implements WebMvcConfigurer {
    @Autowired
    AuthInterceptor authInterceptor; // 这个改成自己的拦截器实现类

    /**
     * ！！！重点
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor)
                // 按需设置不需要拦截的路径
                .excludePathPatterns("/")
                .excludePathPatterns("/csrf")
                .excludePathPatterns("/error/**")
                .excludePathPatterns("/webjars/**")
                .excludePathPatterns("/swagger/**")
                .excludePathPatterns("/v3/api-docs")
                .excludePathPatterns("/swagger-ui.html")
                .excludePathPatterns("/swagger-ui/**")
                .excludePathPatterns("/swagger-ui/index.html")
                .excludePathPatterns("/swagger-resources/**");
    }
}
```

**spring security 框架配置示例**

```java
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;

@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // ...

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity
                // CSRF禁用，因为不使用session
                .csrf().disable()
                // 基于token，所以不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                // 过滤请求
                .authorizeRequests()
                // 对于登录login ... 允许匿名访问
                .antMatchers(
                        HttpMethod.GET,
                        "/",
                        "/*.html",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/profile/**"
                ).permitAll() // permitAll:用户可以任意访问
                // .antMatchers("/login").anonymous() // 项目中不需要登录就能调用的接口 例如登录、注册、首页数据等
                .antMatchers("/api-docs").anonymous() // anonymous:匿名可以访问
                .antMatchers("/swagger-ui/**").anonymous()
                .antMatchers("/swagger-ui.html").anonymous()
                .antMatchers("/swagger-resources/**").anonymous()
                .antMatchers("/webjars/**").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated() // anyRequest:匹配所有请求路径 authenticated:用户登录后可访问
                .and()
                .headers().frameOptions().disable();
        // 配置登出处理
        // 添加JWT filter
        // 添加CORS filter
    }
}
```

!> shiro 的暂时没用过，需要的自己去找下，上面的示例根据自己的项目去调整适配补全

## 优化使用

### 自定义枚举提示

在使用 Swagger 时偶尔会遇到枚举类型提示不友好的情况，这里通过翻看依赖文件源码，在 `springfox-core-3.x.jar` 中找到了一个处理枚举类允许值的类 `springfox.documentation.schema.Enums` ，下面是源码中的局部代码

```java
package springfox.documentation.schema;

// import ...;

public class Enums {
    // ...

    static List<String> getEnumValues(final Class<?> subject) {
        return transformUnique(subject.getEnumConstants(), (Function<Object, String>) input -> {
            Optional<String> jsonValue = findJsonValueAnnotatedMethod(input)
                    .map(evaluateJsonValue(input));
            if (jsonValue.isPresent() && !isEmpty(jsonValue.get())) {
                return jsonValue.get();
            }
            return ((Enum) input).name();
        });
    }

    private static <E> List<String> transformUnique(E[] values, Function<E, String> mapper) {
        return Stream.of(values)
                .map(mapper)
                .distinct()
                .collect(toList());
    }

    private static Optional<Method> findJsonValueAnnotatedMethod(Object enumConstant) {
        for (Method each : enumConstant.getClass().getMethods()) {
            JsonValue jsonValue = AnnotationUtils.findAnnotation(each, JsonValue.class);
            if (jsonValue != null && jsonValue.value()) {
                return of(each);
            }
        }
        return empty();
    }

    // ...
}
```

观察上面的 `static List<String> getEnumValues(final Class<?> subject)` 方法可以看出，默认提供的枚举提示是通过 `@JsonValue` 标记的值或直接获取枚举值 `name` ，实现很单调

这里我们可以自定义一个结构相同的类去覆写这个块的实现逻辑，在需要的模块或公共模块的 `src/main/java` 目录下添加目录 `springfox.documentation.schema` ，在该目录下将 `Enums` 整个代码拷贝过来，并对 `getEnumValues` 方法进行扩展，完整代码如下

```java
package springfox.documentation.schema;

import cn.hutool.core.util.ReflectUtil;
import cn.hutool.core.util.StrUtil;
import com.fasterxml.jackson.annotation.JsonValue;
import org.springframework.core.annotation.AnnotationUtils;
import springfox.documentation.service.AllowableListValues;
import springfox.documentation.service.AllowableValues;

import java.lang.reflect.Method;
import java.util.List;
import java.util.Optional;
import java.util.function.Function;
import java.util.stream.Stream;

import static java.util.Optional.empty;
import static java.util.Optional.of;
import static java.util.stream.Collectors.toList;

/**
 * <p>重写Swagger枚举处理</p>
 *
 * @author Tang
 * @version v1.0
 */
public class Enums {
    private Enums() {
        throw new UnsupportedOperationException();
    }

    public static AllowableValues allowableValues(Class<?> type) {
        if (type.isEnum()) {
            List<String> enumValues = getEnumValues(type);
            return new AllowableListValues(enumValues, "LIST");
        }
        return null;
    }

    static List<String> getEnumValues(final Class<?> subject) {
        return transformUnique(subject.getEnumConstants(), (Function<Object, String>) input -> {
            Optional<String> jsonValue = findJsonValueAnnotatedMethod(input)
                    .map(evaluateJsonValue(input));
            if (jsonValue.isPresent() && StrUtil.isNotBlank(jsonValue.get())) {
                return jsonValue.get();
            }
            // 这里判断了枚举类存不存在 remark 字段
            if (ReflectUtil.hasField(subject, "remark")) {
                // 存在就获取这个字段
                Object remark = ReflectUtil.getFieldValue(input, "remark");
                Enum<?> ienum = (Enum<?>) input;
                // 处理最终返回的显示结果 enumName(ordinal)=remark 例：A(0)=类型A
                return StrUtil.format("{}({})={}", ienum.name(), ienum.ordinal(), remark);
            }
            return ((Enum<?>) input).name();
        });
    }

    @SuppressWarnings("PMD")
    private static Function<Method, String> evaluateJsonValue(final Object enumConstant) {
        return input -> {
            try {
                return input.invoke(enumConstant).toString();
            } catch (Exception ignored) {
            }
            return "";
        };
    }

    private static <E> List<String> transformUnique(E[] values, Function<E, String> mapper) {
        return Stream.of(values)
                .map(mapper)
                .distinct()
                .collect(toList());
    }

    private static Optional<Method> findJsonValueAnnotatedMethod(Object enumConstant) {
        for (Method each : enumConstant.getClass().getMethods()) {
            JsonValue jsonValue = AnnotationUtils.findAnnotation(each, JsonValue.class);
            if (jsonValue != null && jsonValue.value()) {
                return of(each);
            }
        }
        return empty();
    }

    public static AllowableValues emptyListValuesToNull(AllowableListValues values) {
        if (!values.getValues().isEmpty()) {
            return values;
        }
        return null;
    }
}
```

需要注意的是这里只是在原有的基础上添加了通过最简单的判断是否存在说明字段去处理提示，实际上可以通过自定义注解去实现，这样回显字段的灵活性更高

PS： `ordinal()` 方法获取的是枚举值在枚举类中声明时的顺序（或称索引，从 0 开始）而非某个字段值，例如： `enum{A(1,"类型A"),B(2,"类型B")}` 取值： `A.ordinal()=0 B.ordinal()=1`，至于这里为什么要返回这个序号呢，因为较新的 `web` 依赖中的 `jackson` 序列化是可以通过这个进行序列化的，也就是用来接收的枚举字段可以传字符串 "A" 也可以传数字 0

### String类型示例值

在使用 Swagger 时偶尔会遇到 `String` 类型字段没设置示例值的话会出现 Swagger 提供了一个 `xxx: 'stirng'` 示例值没啥用，比如入参有个 `String keyword`，默认给个 string 测试还得删，麻烦得很，不过找了半天大概捋出来的处理逻辑是 String 类型没示例值就省略了 `example` 字段，然后 ui 文件中给了个默认值 string ，不想改前端代码所以找到另个方法设置了个示例值，方法如下

在需要的模块或公共模块的 `src/main/java` 目录下添加目录 `io.swagger.v3.oas.models.media` ，在该目录下将 `StringSchema` 整个代码拷贝过来，加个 `example` 字段

```java
package io.swagger.v3.oas.models.media;

import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.Getter;

import java.util.List;
import java.util.Objects;

/**
 * StringSchema
 */
public class StringSchema extends Schema<String> {
    // ...
    @Override
    public Object getExample() {
        return StringUtils.defaultString(example); // 重写获取示例值给个默认值就好
    }
}
```

### Date类型示例值

在使用 Swagger 时偶尔会遇到 `Date` 类型字段没设置示例值的话会出现 Swagger 提供了一个示例值，但格式并不是我们比较常用的 `yyyy-MM-dd HH:mm:ss` 的格式，这个的相关处理类在写这部分是暂时只在 `swagger-models-2.x.jar` 翻到 `io.swagger.v3.oas.models.media.DateTimeSchema`

在需要的模块或公共模块的 `src/main/java` 目录下添加目录 `io.swagger.v3.oas.models.media` ，在该目录下将 `DateTimeSchema` 整个代码拷贝过来，并对 `cast` 方法与其他重写的方法进行调整，完整代码如下

```java
package io.swagger.v3.oas.models.media;

import java.time.OffsetDateTime;
import java.time.ZoneOffset;
import java.util.Date;

/**
 * DateTimeSchema
 */

public class DateTimeSchema extends Schema<OffsetDateTime> {
    // ...

    @Override
    public Object getExample() {
        if (example == null) {
            return DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss");
        }
        if (example == null) {
            // 调用工具类给个时间就好字符串，这里我从文档注解中获取了format配合格式转换
            return DateFormatUtils.format(new Date(), StringUtils.defaultString(getFormat(), "yyyy-MM-dd HH:mm:ss"));
        }
        return example;
    }
}
```