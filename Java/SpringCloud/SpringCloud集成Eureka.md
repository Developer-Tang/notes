## 搭建注册中心

### 导入依赖

<!-- tabs:start -->
<!-- tab:Maven -->

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
```

<!-- tabs:end -->

### 添加配置

<!-- tabs:start -->
<!-- tab:yml/yaml -->

```yaml
# application.yaml
server:
  port: 8099 #自定
spring:
  application:
    name: eureka-server
eureka:
  instance:
    hostname: <ip> # 注册中心的地址
  client:
    fetch-registry: true # 是否拉取服务清单
    register-with-eureka: true # 是否注册到eureka
    service-url:
      defaultZone: http://<ip>:<port>/eureka,http://<ip>:<port>/eureka # 可以多个注册中心之间相互注册提高可用性

```

<!-- tab:properties -->

```properties
# application.properties
spring.application.name=eureka-server
server.port=8099
# 注册中心的地址
eureka.instance.hostname=<ip>
# 是否拉取服务清单
eureka.client.fetch-registry=true
# 是否注册到eureka
eureka.client.register-with-eureka=true
# 定义注册中心的服务地址
eureka.client.serviceUrl.defaultZone=http://<ip>:<port>/eureka,http://<ip>:<port>/eureka
```

<!-- tabs:end -->

### 修改启动类

```java
// 启动类
@EnableEurekaServer
@SpringBootApplication
public class EurekaServer { // 类名参考自己的
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer.class, args);
    }
}
```

## 实现服务注册

### 导入依赖

<!-- tabs:start -->
<!-- tab:Maven -->

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
```

<!-- tabs:end -->

### 添加配置

<!-- tabs:start -->
<!-- tab:yml/yaml -->

```yaml
# application.yaml
spring:
  application:
    name: xxx-server
eureka:
  client:
    fetch-registry: true # 是否拉取服务清单
    register-with-eureka: true # 是否注册到eureka
    service-url:
      defaultZone: http://<ip>:<port>/eureka,http://<ip>:<port>/eureka # 定义注册中心的服务地址
```

<!-- tab:properties -->

```properties
# application.properties
spring.application.name=xxx-server
# 是否拉取服务清单
eureka.client.fetch-registry=true
# 是否注册到eureka
eureka.client.register-with-eureka=true
# 定义注册中心的服务地址
eureka.client.serviceUrl.defaultZone=http://<ip>:<port>/eureka,http://<ip>:<port>/eureka
```

<!-- tabs:end -->

### 修改启动类

```java
// 启动类
@EnableEurekaServer
@SpringBootApplication
public class XxxServer { // 按自己的来
    public static void main(String[] args) {
        SpringApplication.run(XxxServer.class, args);
    }
}
```
