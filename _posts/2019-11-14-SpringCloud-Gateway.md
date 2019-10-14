---
layout: post
title: SpingCloud Gateway 网关
date: 2019-11-14
tags: spingcloud
---

# 前言

在看SpringCloud之前，看下SpingBoot的启动流程

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
## 进入SpringApplication的源码中：
```java
// resourceLoader 默认为null，primarySources默认为启动类（SpringcloudDemoApplication.class）
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        // 决定web的类型
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
        // 加载初始化器
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        // 设置监听器
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
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




----
# Spring Cloud Gateway api网关
 *
