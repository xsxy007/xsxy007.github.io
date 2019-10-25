---
layout: post
title: PropertyPlaceholderConfigurer读取属性文件使用详解
date: 2019-10-25
tags: spring
---

## PropertyPlaceholderConfigurer读取配置文件

### PropertyPlaceholderConfigurer

    是一个bean工厂后置处理器的实现，也就是BeanFactoryPostProcessor接口的实现

### 作用

      在Spring中，使用`PropertyPlaceholderConfigurer`可以在XML配置文件中加入外部属性文件，当然也可以指定外部文件的编码。`PropertyPlaceholderConfigurer`可以将上下文（配置文 件）中的属性值放在另一个单独的标准java Properties文件中去。在XML文件中用`${key}`替换指定的`properties`文件中的值。这样的话，只需要对`properties`文件进 行修改，而不用对xml配置文件进行修改

### 使用

* 编写`properties`文件  

```properties
    /jdbc.properties 文件
    jdbc.url=jdbc:mysql://66.59.208.106:3306/ds?useUnicode=true&amp;characterEncoding=utf-8&allowMultiQueries=true
    jdbc.username=root
    jdbc.password=root
```

* 在xml中引入外部文件（即`properties`文件）

引入单个文件

```xml
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
 <property name="locations">
  <value>jdbc.properties</value>
 </property>
 <property name="fileEncoding">
    <value>UTF-8</value>
    </property>
</bean>
```

引入多个文件

```xml
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	<property name="locations">
		 <list>  
			<value>classpath:jdbc.properties</value>
			<value>classpath:inter.properties</value>
		 	<value>classpath:email.properties</value>
		 </list>
	</property>
</bean>
```

* 引入文件后，就可以在xml中用`${key}`来引用`properties`中的值（例如jdbc的配置）

```xml
<!-- 配置dbcp数据源 -->
<bean id="dataSourceDefault" class="org.apache.commons.dbcp.BasicDataSource">
	<property name="driverClassName" value="${jdbc.driverClassName}" />
	<property name="url" value="${jdbc.url}" />
	<property name="username" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
</bean>
```


----

## spring2.5之后提供的简化方式

> 为简化PropertyPlaceholderConfigurer的使用，Spring提供了<context:property-placeholder/>元素,启用它后，开发者便不用配置PropertyPlaceholderConfigurer对象了

* 编写`properties`文件

```properties
    /jdbc.properties 文件
    jdbc.url=jdbc:mysql://66.59.208.106:3306/ds?useUnicode=true&amp;characterEncoding=utf-8&allowMultiQueries=true
    jdbc.username=root
    jdbc.password=root
```

* 配置属性文件的位置

```xml
<!-- 数据库配置文件位置 -->
<context:property-placeholder location="classpath:jdbc.properties" />
```

>PropertyPlaceholderConfigurer内置的功能非常丰富，如果它未找到${xxx}中定义的xxx键，它还会去JVM系统属性（System.getProperty()）和环境变量（System.getenv()）中寻找。通过启用systemPropertiesMode和searchSystemEnvironment属性，开发者能够控制这一行为。context:property-placeholder大大的方便了我们数据库的配置。这样就可以为spring配置的bean的属性设置值了

* 配置`bean`的属性

```xml
<!-- 配置dbcp数据源 -->
<bean id="dataSourceDefault" class="org.apache.commons.dbcp.BasicDataSource">
	<property name="driverClassName" value="${jdbc.driverClassName}" />
	<property name="url" value="${jdbc.url}" />
	<property name="username" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
</bean>
```

----

### 自定义PropertyPlaceholderConfigurer

1、继承PropertyPlaceholderConfigurer 类，重写processProperties（）方法，之所以自定义，很有可能是因为java代码中使用了属性值，那些经常变得值，但又是常量，但是这样配置的话，如果属性文件中的属性值改变了，必须重启容器，因为容器启动时，已经将属性值初始化进了代码中就不变了

```java
public class PropertyPlaceholder extends PropertyPlaceholderConfigurer {

    private static Map<String,String> propertyMap;

    @Override
    protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess, Properties props) throws BeansException {
        super.processProperties(beanFactoryToProcess, props);
        propertyMap = new HashMap<String, String>();
        for (Object key : props.keySet()) {
            String keyStr = key.toString();
            String value = props.getProperty(keyStr);
            propertyMap.put(keyStr, value);
        }
    }

    //自定义一个方法，即根据key拿属性值,方便java代码中取属性值
    public static String getProperty(String name) {
        return propertyMap.get(name);
    }
}
```

在创建bean时就会调用processProperties（）方法，属性文件中设置的键值对都放在了Properties中

2、继承PropertyPlaceholderConfigurer 类，重写postProcessBeanFactory方法

```java
	@Override
	public void postProcessBeanFactory(
			ConfigurableListableBeanFactory beanFactory) throws BeansException {
		Properties[] propertiesArray = ConfigUtils.getPropertiesArray();
        // 这里调用父类的方法，其实设置propertiesarray后，后边同样会走上边的processProperties方法
		setPropertiesArray(propertiesArray);

		super.postProcessBeanFactory(beanFactory);
	}
```
