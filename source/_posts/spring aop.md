---
title: Spring AOP
date: 2022-12-13 00:11:00
# img: /source/images/xxx.jpg,featureImages 中的某个值
summary: 简单的AOP示例
tags:
  - Spring
---
# Spring AOP

### 前言：

​	大部分的说明其实都可以在代码里找到。

​	示例代码：https://gitee.com/john0947smith/spring-aop-test



### 一、动态代理

#### 使用代理的好处：

1. 被代理类只需腰关注核心业务的实现，将通用的逻辑和业务逻辑分离；
2. 通用代码放在代理类中实现，提高代码的复用性；
3. 通过在代理类中增加代码，实现对原有业务逻辑的增强（扩展）。



#### 1、JDK动态代理

​	`通过被代理类实现的接口`来创建代理对象，因此JDK动态代理只能代理实现了接口的类。



#### 2、cglib动态代理

​	非JDK自带的，所以需要导入依赖。

​	`通过创建被代理类的子类`来创建代理对象的，因此即使没有实现任何接口的类，都可以被代理。

但是不能为final类创建代理对象。



### 二、AOP

​	利用“横切“的技术，底层逻辑就是动态代理。

​	基于动态代理，在不改变原有业务逻辑的情况下对其进行增强。



#### 概念

- 连接点：程序中的方法（可以被增强的方法）
- 切入点：被 spring 横切的方法
- 通知/增强：配置新增的业务的配置方式（配置新增的逻辑是在切入点的前面还是后面）
- 切点：织入到切入点上的方法
- 切面：定义切点方法的类



#### 定义切入点时的表达式

```xml
<!--
	根据括号里面的表达式
	第一个*：代表方法的返回值，如果为void，则只取返回值为void的方法作为切入点
	全限定名：可以为*，类可以为*，方法可以为*()
	方法为*(..)：表示类中的有无入参的方法都作为切入点，不加..则只取无入参的
	* *(..)：表示所有的方法
-->
"execution(* org.example.dao.impl.StudentDAOImpl.add())"
```



#### AOP的通知策略

​	就是声明将切面类中的切点方法如何织入到切入点。

- before：前置通知，在目标方法执行前执行。
- after：后置通知。
- after-throwing：抛出异常时执行。
- after-returning：方法返回后执行。对一个方法而言 return 也是方法的一部分，所以跟 after 的时间点基本上相同，在 5.2.10 版本的 spring 中，测试得出的结论是哪个策略先配置，则先执行哪个通知的代码。
- around：环绕通知，必须遵循定义规则，如下：

```java
/**
 * 1.返回值必须为 Object
 * 2.入参必须为 ProceedingJoinPoint
 * 3.joinPoint.proceed()这一行代码则是真正执行方法，所以在这一行的前后实现的代码则为前置/后置
 * 4.最后返回的必须是joinPoint.proceed()的返回值
 */
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
  System.out.println("----环绕前----");
  Object proceed = joinPoint.proceed();
  System.out.println("----环绕后----");
  return proceed;
}
```



#### 开发步骤

1. 创建切面类，在切面类中定义切点方法；
2. 将切面类交给 spring 容器管理；
3. 声明切入点；
4. 配置 AOP 的通知策略。



`注意：`

1. 如果要使用 AOP，则不可以自己 new 对象，必须从 spring 容器里获取。
2. 如果一个类没有被声明为切入点，通过 spring 获取的对象就是真实创建的对象。如果被声明为切入点，则获取到的是动态代理的对象。

​	

#### 基于注解的AOP配置

​	注意：如果是第三方的类需要增强则无法使用注解的方式，必须用 xml 进行配置。

```java
@Component
@Aspect
public class LogManager {

  @Pointcut("execution(* org.example.dao.impl.*.*(..))")
  public void temp(){
    // 随便定义一个无参无返回值的类，把表达式写在上面，然后其他的切点引用这个即可
  }

  @Before("temp()")
  public void begin(){
    System.out.println("---开始--");
  }

  @After("temp()")
  public void print() {
    System.out.println("---打印--");
  }

  @Around("temp()")
  public Object around(ProceedingJoinPoint point) throws Throwable {
    System.out.println("---环绕前---");
    Object proceed = point.proceed();
    System.out.println("---环绕后---");
    return proceed;
  }

}
```



