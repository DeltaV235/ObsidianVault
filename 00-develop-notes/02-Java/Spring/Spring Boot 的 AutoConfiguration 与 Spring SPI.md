---

title: Spring Boot 的 AutoConfiguration 与 Spring SPI
created: 2025-03-16
tags:
    - SpringBoot
    - AutoConfiguration
    - SPI

---

## 一、SPI 简介（Service Provider Interface）

SPI 是一种服务发现机制，允许框架定义接口，第三方实现接口，并通过外部配置文件指定实现类，框架动态加载这些实现类。

Spring Boot 基于 Spring Framework 的 `SpringFactoriesLoader` 类实现 SPI 机制，用于加载自动配置候选类。

---

## 二、Spring Boot SPI 机制实现过程

### （1）扫描 SPI 配置文件

Spring Boot 启动时会扫描 classpath 下所有 jar 包中的 SPI 配置文件：

- Spring Boot 2.x 版本：

```properties
META-INF/spring.factories
```

- Spring Boot 3.x 版本：

```properties
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

---

### （2）解析 SPI 配置文件内容

Spring Boot 2.x 中的 `spring.factories` 文件是 Java Properties 格式：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration
```

Spring Boot 3.x 中的文件更加简洁，直接列出自动配置类：

```properties
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration
```

### （3）加载自动配置类到 JVM

Spring Boot 使用类加载器动态加载 SPI 文件中定义的类：

```java
Class<?> clazz = ClassUtils.forName(className, classLoader);
```

### （4）条件化加载自动配置类

```java
@Configuration
@ConditionalOnClass({DispatcherServlet.class, WebMvcConfigurer.class})
public class WebMvcAutoConfiguration {
    // 配置内容
}
```

只有满足注解条件时，配置类才会加载到 Spring 容器。

## 三、Spring Boot 3.x SPI 文件变化说明

|Spring Boot 版本|SPI 配置文件路径|文件格式|
|---|---|---|
|Spring Boot 2.x|`META-INF/spring.factories`|Properties 文件|
|Spring Boot 3.x|`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`|简单类名列表，每行一个类|

---

## 四、自定义自动配置类示例

### Step 1：编写自动配置类

```java
package com.example.autoconfig;

@Configuration
public class MyCustomAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService() {
        return new MyService();
    }
}
```

### Step 2：定义 SPI 配置文件

创建文件：`src/main/resources/META-INF/spring.factories`

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfig.MyCustomAutoConfiguration
```

这样，Spring Boot 启动时会自动加载你的自定义自动配置。

## 五、SPI 机制优势总结

- **松耦合**：无需硬编码配置类，便于扩展。
- **灵活性**：第三方库可轻松加入自定义自动配置。
- **动态性**：按需加载配置类，避免冗余加载。

## Reference

[Spring高手之路14——深入浅出：SPI机制在JDK与Spring Boot中的应用](https://developer.aliyun.com/article/1327091)
