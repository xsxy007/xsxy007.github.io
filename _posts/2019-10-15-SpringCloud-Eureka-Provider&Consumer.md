---
layout: post
title: SpringCloud-Eureka-Provider&Consumer
date: 2019-10-15
tags: springcloud
---

# Eureka-Provider 服务的提供者

### 新建一个服务提供者项目

1、导入pom文件

```xml
<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

```

2、在启动类上加注解

```java
@SpringBootApplication
@EnableDiscoveryClient // 这个注解加不加都可以，因为Eureka
public class EurekaProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaProviderApplication.class, args);
    }
}

```
> 上边那个@EnableDiscoverClient 注解加不加都行的原因会在后边表名

3、在Eureka-Provider项目中添加一个简单的接口

```java
@RestController
public class EurekaProviderController {
    @GetMapping("/provider")
    public String provider(@RequestParam String aaa){
        return "eureka-provider-return" + aaa;
    }
}
```

4、以上配置完成之后启动Eureka-Provider
	启动后会在控制台输出 
DiscoveryClient_EUREKA-PROVIDER/192.168.1.4:eureka-provider:8000: registering service...
	

​	同时看localhost:8761 页面的Instance currently registered 会多出一条信息
​	Application				  AMIs	Availability Zones	Status
​	EUREKA-PROVIDER	n/a (1)	(1)							UP (1) - 192.168.1.4:eureka-provider:8000



# Eureka-Consumer 服务的调用者

**调用者的配置和上边提供者类似，applicatioin.yml配置修改`server.port=8100`**

1、启动类修改

```java
@SpringBootApplication   // 另一个发现服务的注解可以不用谢（SpringCloud版本要在Edgware之后）
public class EurekaConsumerApplication {
    @Bean
    @LoadBalanced
    RestTemplate restTemplate(){
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class, args);
    }
}

```



2、编写一个调用接口的类

```java
@RestController
public class EurekaConsumerController {
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping("/consumer")
    @GetMapping("/consumer")
    public String consumer(@RequestParam String aaa){
        return restTemplate.getForObject("http://EUREKA-PROVIDER/provider?aaa=" + aaa,String.class );
    }
}
```

3、启动服务调用者

​	启动后，调用调用者的触发地址`localhost:8100/consumer?aaa=consumerSemdParamter`

​	浏览器显示内容：

>  eureka-provider-returnconsumerSemdParamter 





### 上边遗留的一个问题，`eureka-client`不加`@EnableDiscoveryClient`以将自己注册到注册中心

* 先看`EurekaClientAutoConfiguration`类中

  ```java
  @Configuration
  @EnableConfigurationProperties
  @ConditionalOnClass(EurekaClientConfig.class)
  @Import(DiscoveryClientOptionalArgsConfiguration.class)
  @ConditionalOnBean(EurekaDiscoveryClientConfiguration.Marker.class)
  @ConditionalOnProperty(value = "eureka.client.enabled", matchIfMissing = true)
  @ConditionalOnDiscoveryEnabled
  @AutoConfigureBefore({ NoopDiscoveryClientAutoConfiguration.class,
  		CommonsClientAutoConfiguration.class, ServiceRegistryAutoConfiguration.class })
  @AutoConfigureAfter(name = {
  		"org.springframework.cloud.autoconfigure.RefreshAutoConfiguration",
  		"org.springframework.cloud.netflix.eureka.EurekaDiscoveryClientConfiguration",
  		"org.springframework.cloud.client.serviceregistry.AutoServiceRegistrationAutoConfiguration" })
  public class EurekaClientAutoConfiguration {
  }
  ```

>​	可以看到该类加载的条件是需要有`EurekaDiscoveryClientConfiguration.Maker.class`的Bean存在，并且`eureka.client.enabled`为true，因为该值默认为true，所以不需要关注，因此重要的就是`EurekaDiscoveryClientConfiguration.Maker.class`这个，而这个类在` Dalston `之前的旧版本是不会自动加载的，而在` Edgware `之后，该类就配置到`spring.factories`文件中了，改文件中所配置的bean在springboot启动的时候就会被加载（所以不需要手动配上注册服务的注解了，springboot会自动配置）