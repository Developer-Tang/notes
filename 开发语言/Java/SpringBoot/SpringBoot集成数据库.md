?> 这里只写集成不同依赖的的配置，具体使用就不写了，因为操作上通过 mybatis 或其他 orm 框架其实没有太大区别

## 数据库

### 集成MySQL

#### 导入MySQL驱动

一般不需要特地指定依赖版本，因为 `spring-boot-dependencies` 中有对 mysql 依赖版本进行管理 `mysql.version`，当然如果版本不符合自己的也可以指定版本去替换掉

<!-- tabs:start -->
<!-- tab:Maven -->

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <!-- <version>${mysql.version}</version> -->
</dependency>
```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'com.mysql:mysql-connector-j'
```

<!-- tabs:end -->

#### 配置MySQL数据库信息

<!-- tabs:start -->
<!-- tab:yml/yaml -->

```yaml
spring:
  datasource:
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://ip:port/db_name? #写自己的链接和参数
    username:
    password: 
```

<!-- tabs:end -->

### 集成TiDB

与集成 MySQL 是一样的，直接套用就可以了

### 集成达梦

#### 导入达梦驱动

<!-- tabs:start -->
<!-- tab:Maven -->

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.dameng</groupId>
    <artifactId>DmJdbcDriver18</artifactId>
    <version>${dm-jdbc.version}</version> <!-- 版本自己选 -->
</dependency>
```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'com.dameng:DmJdbcDriver18:$dmJdbcVersion' //版本自己选
```

<!-- tabs:end -->

#### 配置达梦数据库信息

<!-- tabs:start -->
<!-- tab:yml/yaml -->

```yaml
spring:
  datasource:
    datasource:
    driver-class-name: dm.jdbc.driver.DmDriver
    url: jdbc:dm://ip:port/db_name? #写自己的链接和参数
    username:
    password:

```

<!-- tabs:end -->

## 连接池

### HikariCP(SpringBoot推荐)

#### 导入HikariCP

一般不需要特地指定依赖版本，因为 `spring-boot-dependencies` 中有对 hikaricp 依赖版本进行管理 `hikaricp.version`，当然如果版本不符合自己的也可以指定版本去替换掉

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <!-- <version>${hikaricp.version}</version> -->
</dependency>
```

<!-- tab:Gradle -->

```gradle
// build.gradle
implementation 'com.zaxxer:HikariCP'
```

<!-- tabs:end -->

#### 配置HikariCP连接池信息

这里只写几个常用的，其他配置参考 `com.zaxxer.hikari.HikariConfig` 中的配置属性

<!-- tabs:start -->
<!-- tab:yml/yaml -->

```yaml
spring:
  datasource:
    hikari:
      connection-timeout: 30000 # 设置客户端等待池中连接的最大毫秒数
      maximum-pool-size: 20 # 此属性控制池允许达到的最大大小，包括空闲和正在使用的连接
      minimum-idle: 10 # 此属性控制 HikariCP 尝试在池中维护的最小空闲连接数，包括空闲连接和正在使用的连接
      idle-timeout: 600000 # 此属性控制允许连接在池中闲置的最大时间（以毫秒为单位）
      max-lifetime: 1800000 # 此属性控制池中连接的最大生命周期
      connection-test-query: SELECT 1 # 设置要执行的 SQL 查询以测试连接的有效性
```

<!-- tabs:end -->
