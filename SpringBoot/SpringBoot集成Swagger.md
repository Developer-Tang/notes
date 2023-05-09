## 什么是Swagger

> Swagger 是一个开源的 API 设计和文档工具，它可以帮助开发人员更快、更简单地设计、构建、文档化和测试 RESTful API。Swagger 可以自动生成交互式 API 文档、客户端 SDK、服务器 stub 代码等，从而使开发人员更加容易地开发、测试和部署 API --Apifox

## 集成Swagger

> 这里采用较新的 SpringBoot 版本（2.6.+）与 Swagger3.0 进行集成，如果使用其他版本可能存在兼容性问题，如遇到需要自行摸索调整

### 导入依赖

```xml
<!-- Swagger所需依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

### 配置文件

> 这里添加了一些自定义属性，可以按需自行调整，与下面的配置类中的前缀/字段保持一致即可

```yaml
# application
swagger:
  enabled: true # 定义是否开启接口文档，生产环境一般为 false 关闭
  title: 脚手架 # swagger文档上显示的标题
  author: Tang # 作者
  authorUrl: https://gitee.com/Developer-Tang # 作者网站 
  email: tang_0416@126.com # 联系邮箱
  version: V1.0 # 版本
  authHeader: Authorization # token的请求头
```

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

> 如上述户代码中重写了 `transform` 方法，这里主要是为了去除接口文档中 `Servers` 中端口为 `80|443` 时没有省略的问题，以及当使用 `nginx` 做反向代理/负载均衡时配置如 `location /api` 等规则时程序中获取不到对应请求前缀问题添加了 `X-Forwarded-Prefix` 请求头来解决这个问题

> 如果使用了 nginx 代理可以参考如下配置

```nginx
# nginx配置示例
http {
    # ...
    
    server {
        listen       80;
        # listen       443;
        server_name  localhost;
        # server_name  xxx.xxx.xxx;
        
        # ...
        
        location /api/ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Prefix /api;
            proxy_pass http://localhost:9001/;
        }
    }
}

```
