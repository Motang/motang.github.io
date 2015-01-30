---
layout: post
title: SpringMVC异常处理
category: springmvc
tags: [springmvc, SpringMVC异常处理]
comments: true
share: true
---
## 目录 ##

* Table of Contents
{:toc}

## 1. SpringMVC异常处理笔记 ##
SpringMVC异常处理核心接口 - HandlerExceptionResolver接口
三个实现

    org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver
    org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver
    org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

ExceptionHandlerExceptionResolver：该类主要处理Controller中用@ExceptionHandler注解定义的方法。该类也是<annotation-driven/>配置中定义的HandlerExceptionResolver实现类之一，大多数异常处理都是由该类操作。

DefaultHandlerExceptionResolver：<annotation-driven/>配置中定义的HandlerExceptionResolver实现类之一。该类的doResolveException方法中主要对一些特殊的异常进行处理，比如NoSuchRequestHandlingMethodException、HttpRequestMethodNotSupportedException、HttpMediaTypeNotSupportedException、HttpMediaTypeNotAcceptableException等

ResponseStatusExceptionResolver：<annotation-driven/>配置中定义的HandlerExceptionResolver实现类之一。该类的doResolveException方法主要在异常及异常父类中找到@ResponseStatus注解，然后使用这个注解的属性进行处理。

为什么ExceptionHandlerExceptionResolver、DefaultHandlerExceptionResolver、ResponseStatusExceptionResolver是<annotation-driven/>配置中定义的HandlerExceptionResolver实现类。我们看下<annotation-driven/>配置解析类AnnotationDrivenBeanDefinitionParser中部分代码片段 line247：

    RootBeanDefinition exceptionHandlerExceptionResolver = new RootBeanDefinition(ExceptionHandlerExceptionResolver.class);
    exceptionHandlerExceptionResolver.setSource(source);
    exceptionHandlerExceptionResolver.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    exceptionHandlerExceptionResolver.getPropertyValues().add("contentNegotiationManager", contentNegotiationManager);
    exceptionHandlerExceptionResolver.getPropertyValues().add("messageConverters", messageConverters);
    exceptionHandlerExceptionResolver.getPropertyValues().add("order", 0);
    String methodExceptionResolverName = parserContext.getReaderContext().registerWithGeneratedName(exceptionHandlerExceptionResolver);

    RootBeanDefinition responseStatusExceptionResolver = new RootBeanDefinition(ResponseStatusExceptionResolver.class);
    responseStatusExceptionResolver.setSource(source);
    responseStatusExceptionResolver.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    responseStatusExceptionResolver.getPropertyValues().add("order", 1);
    String responseStatusExceptionResolverName = parserContext.getReaderContext().registerWithGeneratedName(responseStatusExceptionResolver);

    RootBeanDefinition defaultExceptionResolver = new RootBeanDefinition(DefaultHandlerExceptionResolver.class);
    defaultExceptionResolver.setSource(source);
    defaultExceptionResolver.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    defaultExceptionResolver.getPropertyValues().add("order", 2);
    String defaultExceptionResolverName = parserContext.getReaderContext().registerWithGeneratedName(defaultExceptionResolver);

## 2. 总结 ##
ExceptionHandlerExceptionResolver处理过程总结一下：根据用户调用Controller中相应的方法得到HandlerMethod，之后构造ExceptionHandlerMethodResolver，构造ExceptionHandlerMethodResolver有2种选择，1.通过HandlerMethod拿到Controller，找出Controller中带有@ExceptionHandler注解的方法(局部) 2.找到@ControllerAdvice注解配置的类中的@ExceptionHandler注解的方法(全局)。这2种方式构造的ExceptionHandlerMethodResolver中都有1个key为Throwable，value为Method的缓存。之后通过发生的异常找出对应的Method，然后调用这个方法进行处理。这里异常还有个优先级的问题，比如发生的是NullPointerException，但是声明的异常有Throwable和Exception，这时候ExceptionHandlerMethodResolver找Method的时候会根据异常的最近继承关系找到继承深度最浅的那个异常，即Exception

ResponseStatusExceptionResolver 主要在异常及异常父类中找到@ResponseStatus注解。@ResponseStatus注解  让1个方法或异常有状态码(status)和理由(reason)返回。这个状态码是http响应的状态码。

DefaultHandlerExceptionResolver 如果有些异常比如NoSuchRequestHandlingMethodException会发生404错误。没配置@ExceptionHandler以及该异常没有@ResponseStatus注解，最终由DefaultHandlerExceptionResolver解析