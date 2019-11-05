---
layout: post
title: SpringMVC-ModelAndView
date: 2019-10-28
tags: spring
---


### ModelAndView

> spring MVC  向前台传值  
> 该对象中包含了一个model属性和一个view属性  

```java
    private Object view;
    private ModelMap model;
```

1、model

model其实就是一个ModelMap类型，该类型是`LinkedHashMap`的自雷

2、view

包含了一些视图信息

> 当视图解析器是ModelAndView时，其中model本身就是一个Map。视图解析器讲model中的每个元素都通过`request.setAttribute(name,value);`添加到`request`请求域中。这样就可以在jsp页面中通过SpEl表达式获取对应的值

3、向ModelAndView中添加数据

方法一：通过`addObject`方法
```java
    public ModelAndView addObject(String attributeName, Object attributeValue) {
        this.getModelMap().addAttribute(attributeName, attributeValue);
        return this;
    }
```

具体代码：

```java
ModelAndView mv = new ModelAndView("mmmm");
mv.addObject("time",new Date());
```

方法二：由于ModelAndView中的model就是`Map`的实现类，那么可以通过Map的方法来添加

```java
mv.getModel.put("name","caocao");
```

完整代码：

```java
@RequestMapping("/test")
public ModelAndView test(){
    ModelAndView mv = new ModelAndView("mmmm");
    mv.addObject("time",new Date());
    mv.getModel.put("name","caocao");
    return mv;
}
```

在实例化ModelAndView时，其中的参数为视图的名称（这个也可以通过`mv.setViewName("mmmm")`来实现）

jsp页面：
```jsp
time: ${time}
<br/>
name:${name}
```

