---
layout: post
title: SpringBoot
date: 2019-11-14
tags: SpringBoot
---

# 前言

SpingBoot的启动流程

## SpingBoot 启动流程
### 启动类
```java
@SpringBootApplication
public class SpringcloudDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudDemoApplication.class, args);
    }
}
```

## 调用SpringApplication的静态run方法：
```java
	// 最终找到的是该run方法
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}



```
## 进入SpringApplication的构造函数中（ new SpringApplication(primarySources) ）：
```java
// resourceLoader 默认为null，primarySources默认为启动（SpringcloudDemoApplication.class）
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	// 决定web的类型
	this.webApplicationType = WebApplicationType.deduceFromClasspath();
	// 加载初始化器
	this.setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
	// 设置监听器
	this.setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
	this.mainApplicationClass = deduceMainApplicationClass();
}
```
#### 设置webAapplication的类型，进入WebApplicationType.deduceFromClasspath() 方法中：
```java
static WebApplicationType deduceFromClasspath() {
	// 判断根路径下是否有org.springframework.web.reactive.DispatcherHandler.class，如果存在的话，则返回 **reactive web server**
	if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
			&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
		return WebApplicationType.REACTIVE;
	}
	for (String className : SERVLET_INDICATOR_CLASSES) {
		if (!ClassUtils.isPresent(className, null)) {
			return WebApplicationType.NONE;
		}
	}
	// 否则返回**servlet web server**   默认返回为SERVLET
	return WebApplicationType.SERVLET;
}
```
##### 加载初始化器setInitializers()方法，在之前先看getSpringFactoriesInstances(ApplicationContextInitializer.class);返回什么数据

```java
// type:ApplicationContextInitializer.class  paramerTypes:默认为空数组
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
	ClassLoader classLoader = getClassLoader();
	// Use names and ensure unique to protect against duplicates
	// 使用Set是为了防止重复加载
	Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
	List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
	// 根据类上的@Order进行排序
	AnnotationAwareOrderComparator.sort(instances);
	return instances;
}

// 进入SpringFactoriesLoader.loadFactoryNames()方法中，该方法是加载项目中所有的META-INF/spring.factories中的所有实现ApplicationContextInitilizer接口的类
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
	String factoryClassName = factoryClass.getName();
	// loadSpringFactories() 方法是加载spring.factories中所有的数据 ,方法返回的是一个ConcurrentReferenceHashMap()  
	// getOrdefault()方法返回key为ApplicationContextInilializer权限定名的value（也就是各种inilitializer字符串）
	return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}


// 然后进入createSpringFactoriesInstances()方法中
/**
	* type: ApplicationContextInitilizer.class
	* parameterTypes: 空数组
	* classLoader: 类加载器
	* names: 为之前定义的空的Set
	*/
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
		ClassLoader classLoader, Object[] args, Set<String> names) {
	List<T> instances = new ArrayList<>(names.size());
	// 通过反射，根据字符串权限定名创建出instance实例
	for (String name : names) {
		try {
			Class<?> instanceClass = ClassUtils.forName(name, classLoader);
			Assert.isAssignable(type, instanceClass);
			Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
			T instance = (T) BeanUtils.instantiateClass(constructor, args);
			instances.add(instance);
		}
		catch (Throwable ex) {
			throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
		}
	}
	return instances;
}

```
##### 进入setInitializers() 方法
```java
// 该方法就是将之前查到的instance实例对象设置到SpringApplication对象的属性中
public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {
	this.initializers = new ArrayList<>();
	this.initializers.addAll(initializers);
}

```

## 而setListener()方法
**该方法和setInitializers()方法类似**，只是加载的是各种Listener

## deduceMainApplicationClass()方法：
```java
// 
private Class<?> deduceMainApplicationClass() {
	try {
		// getStackTrace()方法：提供了访问堆栈信息的入口，返回的是一个堆栈跟踪元素数组，数组的第0个元素表示位于栈定的最后一个被调用的方法；数组的最后一个元素，表示位于堆栈的底部，即最先被调用的方法
		StackTraceElement[] stackTrace = (new RuntimeException()).getStackTrace();
		StackTraceElement[] var2 = stackTrace;
		int var3 = stackTrace.length;

		// 循环处理堆栈数组，直到查询到栈帧中包含main方法的，返回该方法所属的className
		for(int var4 = 0; var4 < var3; ++var4) {
			StackTraceElement stackTraceElement = var2[var4];
			if ("main".equals(stackTraceElement.getMethodName())) {
				return Class.forName(stackTraceElement.getClassName());
			}
		}
	} catch (ClassNotFoundException var6) {
		;
	}
	return null;
}
```

## 接下来看SpringApplication对象的run方法（new SpringApplication(**).run(args); ）：
```java
// 启动spring application，创建并刷新上下文
public ConfigurableApplicationContext run(String... args) {
	// StopWatch：一个简单的秒表，可以显示每个任务的运行时间和总的运行时间
	StopWatch stopWatch = new StopWatch();
	stopWatch.start();
	// 上下文对象
	ConfigurableApplicationContext context = null;
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	configureHeadlessProperty();
	// 这里的这个getRunListeners()和SpringApplication构造函数中的setInitializers()方法类似，都是从spring.factories中获取类的全路径名，并实例化
	SpringApplicationRunListeners listeners = getRunListeners(args);
	// 将获取到RunListener启动（默认就一个 EventPublishingRunListener）
	listeners.starting();
	try {
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		// 准备环境
		ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
		configureIgnoreBeanInfo(environment);
		Banner printedBanner = printBanner(environment);
		// 创建应用上下文
		context = createApplicationContext();
		// 还是从spring.factories中获取实例
		exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
		// 准备 上下文
		prepareContext(context, environment, listeners, applicationArguments, printedBanner);
		// ** 刷新上下文 **
		refreshContext(context);
		afterRefresh(context, applicationArguments);
		stopWatch.stop();
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
		}
		listeners.started(context);
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
		listeners.running(context);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	return context;
}

```

#### 进入prepareEnvironment(listeners, applicationArguments)方法
```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
		ApplicationArguments applicationArguments) {
	// 在 getOrCreateEnvironment() 方法中，如果是第一次创建，则会根剧最开始设置的webApplicationType来创建不同的web环境，默认为 StandardServletEnviroment
	ConfigurableEnvironment environment = getOrCreateEnvironment();
	// 如果有参数的话 设置参数
	configureEnvironment(environment, applicationArguments.getSourceArgs());
	ConfigurationPropertySources.attach(environment);
	// 通知监听器，环境已经准备好了
	listeners.environmentPrepared(environment);
	bindToSpringApplication(environment);
	if (!this.isCustomEnvironment) {
		environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
				deduceEnvironmentClass());
	}
	ConfigurationPropertySources.attach(environment);
	return environment;
}
```

#### 创建应用上下文createApplicationContext();
```java
protected ConfigurableApplicationContext createApplicationContext() {
	Class<?> contextClass = this.applicationContextClass;
	if (contextClass == null) {
		try {
			switch (this.webApplicationType) {
			case SERVLET:
				// 默认创建的 org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext
				contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
				break;
			case REACTIVE:
				contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
				break;
			default:
				contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
			}
		}
		catch (ClassNotFoundException ex) {
			//....
		}
	}
	return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

#### 准备上下文prepareContext(**);
```java
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
		SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
	// 设置应用环境
	context.setEnvironment(environment);
	// 向容器中先行注入 BeanNameGenerator 和 ResourceLoader 的实例
	postProcessApplicationContext(context);
	// 应用初始化器，调用之前设置的ApplicationContextInitalizer的inilitalize()方法
	applyInitializers(context);
	// 通知监听器，上下文已经准备好了
	listeners.contextPrepared(context);
	// 输出一写启动日志，log默认为Apache的common.logging中的log
	if (this.logStartupInfo) {
		logStartupInfo(context.getParent() == null);
		// 输出profiles active的具体文件
		logStartupProfileInfo(context);
	}
	// Add boot specific singleton beans
	// 获取创建Bean的工厂 默认为DefaultListableBeanFactory
	ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
	// 注册单实例的bean 将之前得到的的应用参数解析器注入到Spring容器中
	beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
	if (printedBanner != null) {
		// 将之前得到的的springBootBanner注入到Spring容器中
		beanFactory.registerSingleton("springBootBanner", printedBanner);
	}
	if (beanFactory instanceof DefaultListableBeanFactory) {
		// 设置单实例的bean不允许覆盖
		((DefaultListableBeanFactory) beanFactory)
				.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
	}
	// Load the sources 加载sources 这里是指我们的启动类
	Set<Object> sources = getAllSources();
	Assert.notEmpty(sources, "Sources must not be empty");
	// 将bean加载到上下文中
	load(context, sources.toArray(new Object[0]));
	// 通知监听器，上下文已经加载
	listeners.contextLoaded(context);
}
```

#### 注册单实例的bean:registerSingleton(**);
```java
// 注册完成之后，会将单实例的bean方法一个concurrentHashmap中，即sigletonObjects；
public void registerSingleton(String beanName, Object singletonObject) throws IllegalStateException {
	Assert.notNull(beanName, "Bean name must not be null");
	Assert.notNull(singletonObject, "Singleton object must not be null");
	synchronized (this.singletonObjects) {
		Object oldObject = this.singletonObjects.get(beanName);
		if (oldObject != null) {
			throw new IllegalStateException("Could not register object [" + singletonObject +
					"] under bean name '" + beanName + "': there is already object [" + oldObject + "] bound");
		}
		addSingleton(beanName, singletonObject);
	}
}

```
#### 加载一些bean：load(context, sources.toArray(new Object[0]));
```java
protected void load(ApplicationContext context, Object[] sources) {
	// 创建BeanDefinition的加载器 直接new BeanDefinitionLoader
	BeanDefinitionLoader loader = createBeanDefinitionLoader(
	getBeanDefinitionRegistry(context), sources);
	// 如果beanNameGenerator 对象不为空的话，则在BeanDefinitionLoader中set beanNameGenerator 
	if (this.beanNameGenerator != null) {
		loader.setBeanNameGenerator(this.beanNameGenerator);
	}
	// 如果resourceLoader 对象不为空的话，则在BeanDefinitionLoader中set resourceLoader 
	if (this.resourceLoader != null) {
		loader.setResourceLoader(this.resourceLoader);
	}
	// 如果environment 对象不为空的话，则在BeanDefinitionLoader中set environment
	if (this.environment != null) {
		loader.setEnvironment(this.environment);
	}
	// 用BeanDefinitionLoader进行加载
	loader.load();
}
```

## 刷新上下文： refreshContext(**);  
** 重要的步骤 **
```java
public void refresh() throws BeansException, IllegalStateException {
		// 线程安全的方法
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				// 创建web容器的地方（ tomcat 或 jetty） 默认走的是：ServletWebServerApplicationContext
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
}
```
