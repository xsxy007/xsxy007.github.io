---
layout: post
title: SpringCloud-Eureka
date: 2019-10-15
tags: springcloud
---

**Eureka 是Netflix开源的一款提供服务注册和发现的产品，提供了完整的Service Registery和Service Discovery实现，也是SpringCloud体系中最核心的组件之一**

# Eureka
![eureka-overview](/images/posts/2019/eureka-architecture-overview.png)  
由上边的图可以看出，`Eureka`由客户端和服务端组成，服务用用于服务的注册服务器，客户端用作服务的提供者和发现者

# 案例

[git@github.com:xsxy007/springcloud-demo.git](git@github.com:xsxy007/springcloud-demo.git)

## Eureka Server
> springboot 已经很好的支持了Eureka，只需要在pom中加入Eureka依赖，并在Application启动类中加上相关注解即可

1、pom依赖添加

**本次使用的springboot版本为2.1.9；springcloud版本为Greenwich.RELEASE**

> 以下列出了所有的依赖（ps：一开始因为springcloud和springboot的版本依赖问题，导致项目一直启动报错）

```xml
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
2、在启动类上加注解`@EnableEurekaServer`注解

```java
@SpringBootApplication
@EnableEurekaServer
public class SpringcloudDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudDemoApplication.class, args);
    }
}
```

3、配置文件
> 在默认配置下，该注册中心会将自己作为客户端尝试注册自己，因此在单机的注册中心时，需要将此机制关闭掉
```yml
server:
  port: 8761 # 使用的是默认的端口
spring:
  application:
    name: eureka-server
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:${server.port}/eureka/
```
`eureka.client.register-with-eureka`：表示是否将自己注册的eureka server，默认为true
`eureka.client.fetch-registry`：表示是否从eureka server上拉取注册信息，默认为true
`eureka.client.service-url.defaultZone`：表示注册中心的地址（默认为localhost:8761/eureka/）

4、启动项目

启动项目后访问`localhost:8761`  即可显示如下页面，此时 Instances current Registers里边并没有内容

![1571143069922](/images/posts/2019/eureka-show.png)




# 集群配置
> 简单的集群，只要将各个注册中心相互配置即可
> `eureka.client.serviceUrl.defaultZone=http://域名:/${server.port}/eureka/,http://域名:/${server.port}/eureka/`  请其他注册中心的地址配置的defaultZone中



# 源码分析

## 从启动类注解开始 @EnableEurekaServer
* 可以注意注释中的`EurekaServerAutoConfiguration`，该类就是EurekaServer的自动配置类
```java
/**
 * Annotation to activate Eureka Server related configuration {@link EurekaServerAutoConfiguration}
 * @author Dave Syer
 * @author Biju Kunjummen
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EurekaServerMarkerConfiguration.class)
public @interface EnableEurekaServer {

}
```
* 再看上边注解类上有一个 @Import(EurekaServerMarkerConfiguration.class)，进入该类查看
```java
// 可以看到，该类并没有什么实现，该类的作用：是为了控制 `EurekaServerAutoConfiguration` 类是否加载
@Configuration
public class EurekaServerMarkerConfiguration {
	@Bean
	public Marker eurekaServerMarkerBean() {
		return new Marker();
	}
	class Marker {
	}
}
```
* 查看 `EurekaServerAutoConfiguration`（该类是在通过Springboot的@EnableAutoconfiguration 来加载spring.factories中的类的）
```java
// 省略了部分代码
@Configuration
@Import(EurekaServerInitializerConfiguration.class)
@ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class) // 当存在 EurekaServerMarkerConfiguration 类是才初始化当前文件
@EnableConfigurationProperties({ EurekaDashboardProperties.class,
		InstanceRegistryProperties.class })
@PropertySource("classpath:/eureka/server.properties")
public class EurekaServerAutoConfiguration extends WebMvcConfigurerAdapter {

    // 实例化一个配置类，该配置类在 EurekaServerInitializerConfiguration 中会使用到
	@Configuration
	protected static class EurekaServerConfigBeanConfiguration {
		@Bean
		@ConditionalOnMissingBean
		public EurekaServerConfig eurekaServerConfig(EurekaClientConfig clientConfig) {
			EurekaServerConfigBean server = new EurekaServerConfigBean();
			if (clientConfig.shouldRegisterWithEureka()) {
				// Set a sensible default if we are supposed to replicate
				server.setRegistrySyncRetries(5);
			}
			return server;
		}
	}

    // eurekaServer的上下文
    @Bean
	public EurekaServerContext eurekaServerContext(ServerCodecs serverCodecs,
			PeerAwareInstanceRegistry registry, PeerEurekaNodes peerEurekaNodes) {
		return new DefaultEurekaServerContext(this.eurekaServerConfig, serverCodecs,
				registry, peerEurekaNodes, this.applicationInfoManager);
	}

    // eurekaServer的启动
	@Bean
	public EurekaServerBootstrap eurekaServerBootstrap(PeerAwareInstanceRegistry registry,
			EurekaServerContext serverContext) {
		return new EurekaServerBootstrap(this.applicationInfoManager,
				this.eurekaClientConfig, this.eurekaServerConfig, registry,
				serverContext);
	}
}
```
* 注意 `EurekaServerInitializerConfiguration` 类，该类实现了lifecycle接口，所以会被spring容器毁掉start方法
```java
	@Override
	public void start() {
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					//TODO: is this class even needed now?
					eurekaServerBootstrap.contextInitialized(
            			EurekaServerInitializerConfiguration.this.servletContext);
					log.info("Started Eureka Server");

					publish(new EurekaRegistryAvailableEvent(getEurekaServerConfig()));
					EurekaServerInitializerConfiguration.this.running = true;
					publish(new EurekaServerStartedEvent(getEurekaServerConfig()));
				}
				catch (Exception ex) {
					// Help!
					log.error("Could not initialize Eureka servlet context", ex);
				}
			}
		}).start(); // 调用 start 方法启动线程
	}
```



[参考： https://blog.csdn.net/cuixhao110/article/details/88353714 ]( https://blog.csdn.net/cuixhao110/article/details/88353714 )