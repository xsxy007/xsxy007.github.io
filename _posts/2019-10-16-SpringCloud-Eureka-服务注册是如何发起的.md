---
layout: post
title: SpringCloud-Eureka-服务注册是如何发起的
date: 2019-10-16
tags: springcloud
---

​	Spring Cloud环境下，服务提供者和消费者启动后都会将自身注册到Eureka



一、将服务注册到Eureka
	一个SpringBoot应用如果要注册到Spring Cloud环境(Greenwich.SR3版本)，步骤很简单：

* pom.xml中添加启动器：spring-cloud-starter-netflix-eureka-client；

* 增加配置：eureka.client.serviceUrl.defaultZone: http://localhost:8100/eureka/；

* 启动应用；

* 如果注册中心正常，此时就能在注册中心发现这个应用了，如下图红框所示：

![eureka-web](/images/posts/2019/2019-10-16-00001.jpg)

* 按照spring.factories中的配置，EurekaClientAutoConfiguration中的配置都会生效，包括下面这段代码返回的bean：

* ```java
  @Bean
  public DiscoveryClient discoveryClient(EurekaInstanceConfig config, EurekaClient client) {
      return new EurekaDiscoveryClient(config, client);
  }
  ```

* spring容器初始化时会实例化所有单例bean，就会执行EurekaClientAutoConfiguration的discoveryClient方法获取这个bean实例，于是就构造了一个EurekaDiscoveryClient对象；

* 注意EurekaDiscoveryClient的构造方法，第二个入参是com.netflix.discovery.EurekaClient类型，此对象同样来自EurekaClientAutoConfiguration类，如下方法：

* ```java
  @Bean(destroyMethod = "shutdown")
  @ConditionalOnMissingBean(value = EurekaClient.class, search = SearchStrategy.CURRENT)
  @org.springframework.cloud.context.config.annotation.RefreshScope
  @Lazy
  public EurekaClient eurekaClient(ApplicationInfoManager manager, EurekaClientConfig config, EurekaInstanceConfig instance) {
      manager.getInfo(); // force initialization
      return new CloudEurekaClient(manager, config, this.optionalArgs,this.context);
  }
  ```

* 
  CloudEurekaClient的父类com.netflix.discovery.DiscoveryClient来自netflix发布的eureka-client包中，所以可以这么理解：EurekaDiscoveryClient类是个代理身份，真正的服务注册发现是委托给netflix的开源包来完成的，我们可以专心的使用SpringCloud提供的服务注册发现功能，只需要知道EurekaDiscoveryClient即可，真正的服务是eureka-client来完成的；

* 接下来需要关注com.netflix.discovery.DiscoveryClient的构造方法，因为这里面有服务注册的逻辑，整个构造方法内容太多，无需都细看，只看关键代码即可；

* DiscoveryClient的构造方法中，最熟悉的应该是下图红框中这段日志输出的了：

  对应的应用启动日志中就有这段日志输出，如下图红框：	

  ![](/images/posts/2019/2019-10-16-00002.jpg)

  红框中的”us-east-1”，是默认的region，来自配置类EurekaClientConfigBean，这里面有各种eureka相关的配置信息，以及默认配置，如下图：

  ![](/images/posts/2019/2019-10-16-00003.jpg)

* 继续看DiscoveryClient的构造方法，服务注册相关的initScheduledTasks方法在此被调用，如下图：

  ![](/images/posts/2019/2019-10-16-00004.jpg)

* initScheduledTasks方法的内容如下，请注意中文注释：

  ```java
      private void initScheduledTasks() {
          //获取服务注册列表信息
          if (clientConfig.shouldFetchRegistry()) {
              //服务注册列表更新的周期时间
              int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();
              int expBackOffBound = clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
              //定时更新服务注册列表
              scheduler.schedule(
                      new TimedSupervisorTask(
                              "cacheRefresh",
                              scheduler,
                              cacheRefreshExecutor,
                              registryFetchIntervalSeconds,
                              TimeUnit.SECONDS,
                              expBackOffBound,
                              new CacheRefreshThread() //该线程执行更新的具体逻辑
                      ),
                      registryFetchIntervalSeconds, TimeUnit.SECONDS);
          }
          if (clientConfig.shouldRegisterWithEureka()) {
              //服务续约的周期时间
              int renewalIntervalInSecs = instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
              int expBackOffBound = clientConfig.getHeartbeatExecutorExponentialBackOffBound();
              //应用启动可见此日志，内容是：Starting heartbeat executor: renew interval is: 30
              logger.info("Starting heartbeat executor: " + "renew interval is: " + renewalIntervalInSecs);
              // 定时续约
              scheduler.schedule(
                      new TimedSupervisorTask(
                              "heartbeat",
                              scheduler,
                              heartbeatExecutor,
                              renewalIntervalInSecs,
                              TimeUnit.SECONDS,
                              expBackOffBound,
                              new HeartbeatThread() //该线程执行续约的具体逻辑
                      ),
                      renewalIntervalInSecs, TimeUnit.SECONDS);
  
              //这个Runable中含有服务注册的逻辑
              instanceInfoReplicator = new InstanceInfoReplicator(
                      this,
                      instanceInfo,
                      clientConfig.getInstanceInfoReplicationIntervalSeconds(),
                      2); // burstSize
  
              statusChangeListener = new ApplicationInfoManager.StatusChangeListener() {
                  @Override
                  public String getId() {
                      return "statusChangeListener";
                  }
  
                  @Override
                  public void notify(StatusChangeEvent statusChangeEvent) {
                      if (InstanceStatus.DOWN == statusChangeEvent.getStatus() ||
                              InstanceStatus.DOWN == statusChangeEvent.getPreviousStatus()) {
                          // log at warn level if DOWN was involved
                          logger.warn("Saw local status change event {}", statusChangeEvent);
                      } else {
                          logger.info("Saw local status change event {}", statusChangeEvent);
                      }
                      instanceInfoReplicator.onDemandUpdate();
                  }
              };
  
              if (clientConfig.shouldOnDemandUpdateStatusChange()) {
                  applicationInfoManager.registerStatusChangeListener(statusChangeListener);
              }
              //服务注册
              instanceInfoReplicator.start(clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
          } else {
              logger.info("Not registering with Eureka server per configuration");
          }
      }
  ```

  上述代码中有几处需要注意，这些关键点在后面的章节将继续展开：
  a. 周期性更新服务列表；
  b. 周期性服务续约；
  c. 服务注册逻辑被放入Runnable实现类InstanceInfoReplicator之中，在新线程中执行；

