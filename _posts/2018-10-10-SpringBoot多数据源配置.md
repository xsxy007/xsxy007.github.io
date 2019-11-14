---
layout: post
title: springboot多数据源配置
date: 2018-10-10
tags: springboot
---

# 多数据源

使用Spring Boot时，默认情况下，配置DataSource非常容易。Spring Boot会自动为我们配置好一个DataSource。

如果在application.yml中指定了spring.datasource的相关配置，Spring Boot就会使用该配置创建一个DataSource。如果在application.yml中没有指定任何spring.datasource的相关配置，Spring Boot会在classpath中搜索H2、hsqldb等内存数据库的jar包，如果找到了，就会自动配置一个内存数据库的DataSource，所以，我们只要引入jar包即可。例如，配置一个hsqldb数据源：

```xml
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
```

但是，在某些情况下，如果我们需要配置多个数据源，应该如何在Spring Boot中配置呢？

我们以JDBC为例，演示如何在Spring Boot中配置两个DataSource。对应的，我们会创建两个JdbcTemplate的Bean，分别使用这两个数据源。

首先，我们必须在application.yml中声明两个数据源的配置，一个使用spring.datasource，另一个使用spring.second-datasource：

```yml
spring:
  application:
    name: data-multidatasource
  datasource:
    driver-class-name: org.hsqldb.jdbc.JDBCDriver
    url: jdbc:hsqldb:mem:db1
    username: sa
    password:
  second-datasource:
    driver-class-name: org.hsqldb.jdbc.JDBCDriver
    url: jdbc:hsqldb:mem:db2
    username: sa
    password:
```

这两个DataSource都使用hsqldb，但是数据库是不同的。此外，在使用多数据源的时候，所有必要配置都不能省略。

其次，我们需要自己创建两个DataSource的Bean，其中一个标记为@Primary，另一个命名为secondDatasource：

```java
@Configuration
public class SomeConfiguration {
    @Bean
    @Primary
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondDatasource")
    @ConfigurationProperties(prefix = "spring.second-datasource")
    public DataSource secondDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

对于每一个`DataSource`，我们都必须通过`@ConfigurationProperties(prefix = "xxx")`指定配置项的前缀。

紧接着，我们创建`两个``JdbcTemplate`的Bean，其中一个标记为`@Primary`，另一个命名为`secondJdbcTemplate`，分别使用对应的DataSource：

```java
@Bean
@Primary
public JdbcTemplate primaryJdbcTemplate(DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}

@Bean(name = "secondJdbcTemplate")
public JdbcTemplate secondJdbcTemplate(@Qualifier("secondDatasource") DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}
```

注意到secondJdbcTemplate在创建时，传入的DataSource必须用`@Qualifier("secondDatasource")`声明，这样，才能使用第二个DataSource。

现在，我们就创建了两个JdbcTemplate的Bean。在需要使用第一个JdbcTemplate的地方，我们直接注入：

```java
@Component
public class SomeService {
    @Autowired
    JdbcTemplate jdbcTemplate;
}
```

在需要使用第二个JdbcTemplate的地方，我们注入时需要用@Qualifier("secondJdbcTemplate")标识：

```java
@Component
public class AnotherService {
    @Autowired
    @Qualifier("secondJdbcTemplate")
    JdbcTemplate secondJdbcTemplate;
}
```

这样，我们就可以针对不同的数据源，用不同的JdbcTemplate进行操作。

注意事项
当存在多个相同类型的Bean，例如，多个DataSource，多个JdbcTemplate时，强烈建议总是使用@Primary把其中某一个Bean标识为“主要的”，使用@Autowired注入时会首先使用被标记为@Primary的Bean。

相同类型的其他Bean，每一个都需要用@Bean(name="xxx")标识名字，并且，在使用@Autowired注入时配合@Qualifier("xxx")指定注入的Bean的名字。

完整的示例工程源码请参考：

[https://github.com/michaelliao/springcloud/tree/master/data-multidatasource](https://github.com/michaelliao/springcloud/tree/master/data-multidatasource)


# 动态数据源
在大型应用程序中，配置主从数据库并使用读写分离是常见的设计模式。在Spring应用程序中，要实现读写分离，最好不要对现有代码进行改动，而是在底层透明地支持。

Spring内置了一个AbstractRoutingDataSource，它可以把多个数据源配置成一个Map，然后，根据不同的key返回不同的数据源。因为AbstractRoutingDataSource也是一个DataSource接口，因此，应用程序可以先设置好key， 访问数据库的代码就可以从AbstractRoutingDataSource拿到对应的一个真实的数据源，从而访问指定的数据库。它的结构看起来像这样：
![multipartDatasource](/images/posts/2019/multipartDatasource1.png)

## 第一步：配置多数据源

首先，我们在SpringBoot中配置两个数据源，其中第二个数据源是ro-datasource：

```yml
spring:
  datasource:
    jdbc-url: jdbc:mysql://localhost/test
    username: rw
    password: rw_password
    driver-class-name: com.mysql.jdbc.Driver
    hikari:
      pool-name: HikariCP
      auto-commit: false
      ...
  ro-datasource:
    jdbc-url: jdbc:mysql://localhost/test
    username: ro
    password: ro_password
    driver-class-name: com.mysql.jdbc.Driver
    hikari:
      pool-name: HikariCP
      auto-commit: false
      ...
```

在开发环境下，没有必要配置主从数据库。只需要给数据库设置两个用户，一个rw具有读写权限，一个ro只有SELECT权限，这样就模拟了生产环境下对主从数据库的读写分离。

在SpringBoot的配置代码中，我们初始化两个数据源：

```java
@SpringBootApplication
public class MySpringBootApplication {
    /**
     * Master data source.
     */
    @Bean("masterDataSource")
    @ConfigurationProperties(prefix = "spring.datasource")
    DataSource masterDataSource() {
       logger.info("create master datasource...");
        return DataSourceBuilder.create().build();
    }

    /**
     * Slave (read only) data source.
     */
    @Bean("slaveDataSource")
    @ConfigurationProperties(prefix = "spring.ro-datasource")
    DataSource slaveDataSource() {
        logger.info("create slave datasource...");
        return DataSourceBuilder.create().build();
    }

    ...
}
```

## 第二步：编写RoutingDataSource

然后，我们用Spring内置的RoutingDataSource，把两个真实的数据源代理为一个动态数据源：

```java
public class RoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return "masterDataSource";
    }
}
```

对这个`RoutingDataSource`，需要在SpringBoot中配置好并设置为主数据源：

```java
@SpringBootApplication
public class MySpringBootApplication {
    @Bean
    @Primary
    DataSource primaryDataSource(
            @Autowired @Qualifier("masterDataSource") DataSource masterDataSource,
            @Autowired @Qualifier("slaveDataSource") DataSource slaveDataSource
    ) {
        logger.info("create routing datasource...");
        Map<Object, Object> map = new HashMap<>();
        map.put("masterDataSource", masterDataSource);
        map.put("slaveDataSource", slaveDataSource);
        RoutingDataSource routing = new RoutingDataSource();
        routing.setTargetDataSources(map);
        routing.setDefaultTargetDataSource(masterDataSource);
        return routing;
    }
    ...
}
```

现在，`RoutingDataSource`配置好了，但是，路由的选择是写死的，即永远返回"masterDataSource"，

* 现在问题来了：如何存储动态选择的key以及在哪设置key？

在Servlet的线程模型中，使用ThreadLocal存储key最合适，因此，我们编写一个`RoutingDataSourceContext`，来设置并动态存储key：

```java
public class RoutingDataSourceContext implements AutoCloseable {

    // holds data source key in thread local:
    static final ThreadLocal<String> threadLocalDataSourceKey = new ThreadLocal<>();

    public static String getDataSourceRoutingKey() {
        String key = threadLocalDataSourceKey.get();
        return key == null ? "masterDataSource" : key;
    }

    public RoutingDataSourceContext(String key) {
        threadLocalDataSourceKey.set(key);
    }

    public void close() {
        threadLocalDataSourceKey.remove();
    }
}
```

然后，修改`RoutingDataSource`，获取key的代码如下：

```java
public class RoutingDataSource extends AbstractRoutingDataSource {
    protected Object determineCurrentLookupKey() {
        return RoutingDataSourceContext.getDataSourceRoutingKey();
    }
}
```

这样，在某个地方，例如一个Controller的方法内部，就可以动态设置DataSource的Key：

```java
@Controller
public class MyController {
    @Get("/")
    public String index() {
        String key = "slaveDataSource";
        try (RoutingDataSourceContext ctx = new RoutingDataSourceContext(key)) {
            // TODO:
            return "html... www.liaoxuefeng.com";
        }
    }
}
```

到此为止，我们已经成功实现了数据库的动态路由访问。

这个方法是可行的，但是，需要读从数据库的地方，就需要加上一大段try (RoutingDataSourceContext ctx = ...) {}代码，使用起来十分不便。有没有方法可以简化呢？

有！

我们仔细想想，Spring提供的声明式事务管理，就只需要一个`@Transactional()`注解，放在某个Java方法上，这个方法就自动具有了事务。

我们也可以编写一个类似的`@RoutingWith("slaveDataSource")`注解，放到某个Controller的方法上，这个方法内部就自动选择了对应的数据源。代码看起来应该像这样：

```java
@Controller
public class MyController {
    @Get("/")
    @RoutingWith("slaveDataSource")
    public String index() {
        return "html... www.liaoxuefeng.com";
    }
}
```

这样，完全不修改应用程序的逻辑，只在必要的地方加上注解，自动实现动态数据源切换，这个方法是最简单的。

想要在应用程序中少写代码，我们就得多做一点底层工作：必须使用类似Spring实现声明式事务的机制，即用AOP实现动态数据源切换。

实现这个功能也非常简单，编写一个`RoutingAspect`，利用AspectJ实现一个`Around`拦截：

```java
@Aspect
@Component
public class RoutingAspect {
    @Around("@annotation(routingWith)")
    public Object routingWithDataSource(ProceedingJoinPoint joinPoint, RoutingWith routingWith) throws Throwable {
        String key = routingWith.value();
        try (RoutingDataSourceContext ctx = new RoutingDataSourceContext(key)) {
            return joinPoint.proceed();
		}
	}
}
```

注意方法的第二个参数RoutingWith是Spring传入的注解实例，我们根据注解的value()获取配置的key。编译前需要添加一个Maven依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

到此为止，我们就实现了用注解动态选择数据源的功能。最后一步重构是用字符串常量替换散落在各处的"masterDataSource"和"slaveDataSource"。

> 使用限制  
受Servlet线程模型的局限，动态数据源不能在一个请求内设定后再修改，也就是@RoutingWith不能嵌套。此外，@RoutingWith和@Transactional混用时，**要设定AOP的优先级**。

本文代码需要SpringBoot支持，JDK 1.8编译并打开-parameters编译参数。