---
layout: post
title: Hessian轻量级RPC框架
date: 2019-10-25
tags: RPC
---

## Hessian轻量级RPC框架

### 一、什么是RPC

    RPC全称Remote Procedure Call，中文名叫远程过程调用。RPC是一种远程调用技术，用于不同系统之间的远程相互调用。其在分布式系统中应用十分广泛。

### 二、什么是Hessian

    Hessian是一个轻量级的RPC框架。 相比WebService，Hessian更简单、快捷。采用的是二进制RPC协议，因为采用的是二进制协议，所以它很适合于发送二进制数据。

### 三、Hessian的使用

　　1、引入jar包

```xml
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.38</version>
</dependency>
```

　　2.编写业务代码（和普通的业务代码一样）

```java
public interface UserService {
    String getUserInfoById (Integer id);
}
```

```java
@Component("uservice")
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;

    public String getUserInfoById(Integer id) {
        User user = userMapper.selectByPrimaryKey(id);
        return user.toString();
    }
}
```

　　3.创建并加载hessian-servlet.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
    <!-- 使用Spring的HessianServie做代理 -->
    <bean name="/userServiceImpl" class="org.springframework.remoting.caucho.HessianServiceExporter">
        <!-- service引用具体的实现实体Bean-->
        <property name="service" ref="uservice" />
        <property name="serviceInterface" value="com.myproject.hessian.UserService" />
    </bean>
 
</beans>
```

```java
public class HessianServiceProxyExporter extends HessianServiceExporter {
    private static final Base64 base64 = new Base64();

    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        String authorization = request.getHeader("Authorization");
        if(authorization != null && authorization.length() > 0){
            String userAndPwd = new String(base64.decode(authorization.split(" ")[1]));
            String user = userAndPwd.split(":")[0];
            String password = userAndPwd.split(":")[1];
            if("user".equalsIgnoreCase(user) && "123456".equalsIgnoreCase(password)) {
                super.handleRequest(request, response);
            } else {
                System.out.println("您无权调用");
            }
        }
    }
}
```

```xml
<beans>
    <!-- 自己定义代理类来继承org.springframework.remoting.caucho.HessianServiceExporter类，然后进行权限等一系列操作-->
    <bean name="/userServiceImpl" class="com.myproject.hessian.exporter.HessianServiceProxyExporter">
        <!-- service引用具体的实现实体Bean-->
        <property name="service" ref="uservice" />
        <property name="serviceInterface" value="com.myproject.hessian.UserService" />
    </bean>
 
</beans>
```

```xml
<!-- web.xml中进行拦截，并加载配置文件hessian -->
    <servlet>
        <servlet-name>hessian</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:hessian-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>hessian</servlet-name>
        <url-pattern>/hessian/*</url-pattern>
    </servlet-mapping>
```

　　5.客户端编写

　　　　①普通方式调用。同样需要引入hessian的jar包

```java
//先将服务端的服务接口搬过来，包名和类名方法名最好是要一模一样
public interface UserService {
    String getUserInfoById (Integer id);
}
```

```java
public class Test {
    public static void main(String[] args) throws MalformedURLException {
        String url = "http://localhost:8080/zmyproject/hessian/userServiceImpl";
        HessianProxyFactory factory = new HessianProxyFactory();
　　　　 factory.setUser("user");
　　　　 factory.setPassword("123456")；
　　　　 UserService userService = (UserService)factory.create(UserService.class, url);
        System.out.println(userService.getUserInfoById(2));
    }
}
```

　　　　②spring框架调用

 　　　　　　Ⅰ、先引入jar包，注意jar包的版本我使用的hession-3.1.5.jar，启动会找不到一个factory类，所以用了4.0.38版本的。

　　　　　　 Ⅱ、配置hession-client.xml，并加载该文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:task="http://www.springframework.org/schema/task"
    xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
                        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd  
                        http://www.springframework.org/schema/context  
                        http://www.springframework.org/schema/context/spring-context-4.0.xsd  
                        http://www.springframework.org/schema/mvc  
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
                        http://www.springframework.org/schema/aop
                        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
                        http://www.springframework.org/schema/task
                        http://www.springframework.org/schema/task/spring-task-3.1.xsd">
    
    <!--客户端Hessian代理工厂Bean-->
    <bean id="userService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
        <!--这是因为接口中出现方法重载，在调用时，服务器端会跑出异常 。在整合spring中，在客户端的配置里面加上如下代码可以解决：-->
        <property name="overloadEnabled" value="true" />
        <!--请求代理Servlet路径：-->
        <property name="serviceUrl" value="http://localhost:8080/zmyproject/hessian/userServiceImpl" />
        <!--接口定义：-->
        <property name="serviceInterface" value="com.myproject.hessian.UserService" />
        <property name="username" value="user" />
        <property name="password" value="123456" />
    </bean> 
</beans>
```

```xml
<!-- web.xml中配置 -->
  <listener>
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>  
  <param-name>contextConfigLocation</param-name>  
     <param-value>classpath:hession-servlet.xml</param-value>  
  </context-param> 
``` 

```java
//和hessian服务端一样的接口
public interface UserService {

    String getUserInfoById (Integer id);
}

@Controller//将该类标注为处理器，并且加入spring容器中
@RequestMapping("/hessian")
public class HessianController{
    
    @Autowired
    private UserService userService;
    
    @RequestMapping(value="/getInfo",method=RequestMethod.GET)
    @ResponseBody
    public String getInfo(HttpServletRequest request,HttpServletResponse response){
        String userInfo = userService.getUserInfoById(1);
       return userInfo;
    }
}
```
