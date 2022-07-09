---
layout: post
title: "@Component @ComponentScan @Import"
tags: ["spring", "java"]
---

spring中声明bean并让其被自动装配.


## 声明bean
### @Component注解

如果一个类中声明了多个@Beanb需要被自动装配,可以在类上添加@Component注解

```java
 @Configuration
 public class AppConfig {
     @Bean
     public MyBean myBean() {
         // instantiate, configure and return bean ...
     }
     
     @Bean
     public AnotherBean anotherBean(){
         // to use
     }
 }
```

## 让声明的bean被spring管理

1、通过AnnotationConfigApplicationContext

@Configuration通常由AnnotationConfigApplicationContext（或者AnnotationConfigWebApplicationContext）引导,通常的使用方式如下：

```java
 AnnotationConfigApplicationContext ctx =
     new AnnotationConfigApplicationContext();
 ctx.register(AppConfig.class);
 ctx.refresh();
 MyBean myBean = ctx.getBean(MyBean.class);
 // use myBean ...
```

2、通过spring <beans> xml 

3、通过component scanning

@Configuration内部包含了@Component注解,因此如果开启了@ComponentScan,@Configuration的类也会被扫描到

4、@Import

如果没有配置@ComponentScan,也可以通过@Import注解来使其生效,通常用于不想使用component scanning,而是想明确标识出需要装配的类.