## 启动类注解

> - **`@SpringBootApplication`**
>   - **`@SpringBootConfiguration`**: 组合了 @Configuration 注解，实现配置文件的功能。
>   - **`@EnableAutoConfiguration`**: 打开自动配置的功能，也可以关闭某个自动配置的选项。 @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
>   - **`@ComponentScan`**: Spring组件扫描

## 支持配置文件格式

### properties

```properties
username=root
password=123456
```

### yml

```yaml
database:
  username: root
  password: 123456
```

## 自动配置原理

> `@EnableAutoConfiguration` (开启自动配置) 该注解引入了 `AutoConfigurationImportSelector` ，该类中的方法会扫描所有存在 `META-INF/spring.factories` 的jar包

## 配置文件加载顺序

> `bootstrap.properties` -> `bootstrap.yml` -> `application.properties` -> `application.yml`

## 读取配置的方式

> - **注解**
>   - `@PropertySource`
>   - `@Value`
>   - `@Environment`
>   - `@ConfigurationProperties`
> - **xx类**