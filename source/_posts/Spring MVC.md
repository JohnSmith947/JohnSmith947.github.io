---
title: Spring MVC
date: 2022-12-21 22:46:00
# img: /source/images/xxx.jpg,featureImages 中的某个值
summary: Spring MVC 笔记
tags:
  - Spring
---
# Spring MVC

### SpringMVC的请求处理流程

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec34d936d3484a6d9f91ca1aed7171b4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

① 前端发送请求被前端控制器DispatcherServlet拦截
② 前端控制器调用处理器映射器HandlerMapping对请求URL进行解析，解析之后返回调用链给前端控制器
③ 前端控制器调用处理器适配器处理调用链
④ 处理器适配器基于反射通过适配器设计模式完成处理器(控制器)的调用处理用户请求
⑤ 处理器适配器将控制器返回的视图和数据信息封装成ModelAndView响应给前端控制器
⑥ 前端控制器调用视图解析器ViewResolver对ModelAndView进行解析，将解析结果（视图资源和数据）响应给前端控制器
⑦ 前端控制器调用视图view组件将数据进行渲染，将渲染结果（静态视图）响应给前端控制器
⑧ 前端控制器响应用户请求



### SpringMVC的核心组件

- `DispatcherServlet` 前端控制器、总控制器
  - 作用：接收请求，协同各组件工作、响应请求
- `HandlerMapping` 处理器映射
  - 作用：负责根据用户请求的URL找到对应的Handler
  - **可配置** SpringMVC提供多个处理器映射的实现，可以根据需要进行配置
- `HandlerAdapter` 处理器适配器
  - 作用：按照处理器映射器解析的用户请求的调用链，通过适配器模式完成Handler的调用
- `Handler` 处理器/控制器
  - 由工程师根据业务的需求进行开发
  - 作用：处理请求
- `ModelAndView` 视图模型
  - 作用：用于封装处理器返回的数据以及相应的视图
  - ModelAndView = Model + View
- `ViewResolver` 视图解析器
  - 作用：对ModelAndView进行解析
  - **可配置** SpringMVC提供多个视图解析器的实现，可以根据需要进行配置
- `View` 视图
  - 作用：完成数据渲染





### Spring MVC 中可配置的两个组件：

#### 处理器映射器

> 不同的处理器映射器对URL处理的方式也不相同，使用对应的处理器映射器之后我们的前端请求规则也需要发生相应的变化。主要涉及的是 `@RequestMapping` 注解。
>
> SpringMVC提供的处理器映射器：
>
> - BeanNameUrlHandlerMapping  根据控制器的类名（首字母需要小写，如 UserController 则为 userController）访问控制器，可以把 Controller 类上的 `@RequestMapping` 注解去掉。
> - SimpleUrlHandlerMapping 根据控制器配置的URL访问（默认），在 xml 文件中配置的时候可以去掉 `@RequestMapping` 注解，默认是使用注解然后在后面注明路径的方式（如：@RequeMapping("/user")）。

配置处理器映射器：

- 在SpringMVC的配置文件中通过bean标签声明处理器映射器

- 配置BeanNameUrlHandlerMapping，交给 Spring 容器管理就可以了。

  ```xml
  <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>
  ```

- 配置SimpleUrlHandlerMapping

  ```xml
  <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
      <property name="mappings">
          <props>
              <prop key="/aaa">bookController</prop>
              <prop key="/bbb">studentController</prop>
          </props>
      </property>
  </bean>
  ```



#### 视图解析器

> Spring提供了多个视图解析器：
>
> - UrlBasedViewResolver
> - InternalResourceViewResolver

- UrlBasedViewResolver 需要依赖jstl

  - 添加JSTL的依赖

  ```xml
  <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
  </dependency>
  ```

  - 配置视图解析器

  ```xml
  <bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
      <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
      <property name="prefix" value="/"/>
      <property name="suffix" value=".jsp"/>
  </bean>
  ```

- InternalResourceViewResolver

  ```xml
  <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name="prefix" value="/"/>
      <property name="suffix" value=".jsp"/>
  </bean>
  ```



### 日期格式处理

> 如果前端需要输入日期数据，在控制器中转换成Date对象，SpringMVC要求前端输入的日期格式必须为`yyyy/MM/dd`
>
> 如果甲方要求日期格式必须为指定的格式，而这个指定格式SpringMVC不接受，该如何处理呢？
>
> - 自定义日期转换器

#### 创建自定义日期转换器

```java
/***
 * 1.创建一个类实现Converter接口，泛型指定从什么类型转换为什么类型
 * 2.实现convert转换方法
 */
public class MyDateConverter implements Converter<String, Date> {

    SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日");

    public Date convert(String s) {
        Date date = null;
        try {
            date = sdf.parse(s);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }

}
```

#### 配置自定义转换器

```xml
<!--  声明MVC使用注解驱动  -->
<mvc:annotation-driven conversion-service="converterFactory"/>

<bean id="converterFactory" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.qfedu.utils.MyDateConverter"/>
        </set>
    </property>
</bean>
```



### 控制器接收数据和文件

> SpringMVC处理上传文件需要借助于CommonsMultipartResolver文件解析器

- 添加依赖：commons-io  commons-fileupload

  ```xml
  <dependency>
       <groupId>commons-io</groupId>
       <artifactId>commons-io</artifactId>
       <version>2.4</version>
  </dependency>
  <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.4</version>
  </dependency>
  ```

- 在spring-servlet.xml中配置文件解析器

  ```xml
  <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
      <property name="maxUploadSize" value="10240000"/>
      <property name="maxInMemorySize" value="102400"/>
      <property name="defaultEncoding" value="utf-8"/>
  </bean>
  ```

- 控制器接收文件

  - 在处理文件上传的方法中定义一个MultiPartFile类型的对象，就可以接受图片了

  ```java
  @Controller
  @RequestMapping("/book")
  public class BookController {


      @RequestMapping("/add")
      public String addBook(Book book, MultipartFile imgFile, HttpServletRequest request) throws IOException {
          System.out.println("--------------add");

          //imgFile就表示上传的图片
          //1.截取上传文件的后缀名,生成新的文件名
          String originalFilename = imgFile.getOriginalFilename();
          String ext = originalFilename.substring( originalFilename.lastIndexOf(".") ); 
          String fileName = System.currentTimeMillis()+ext;

          //2.获取imgs目录在服务器的路径
          String dir = request.getServletContext().getRealPath("imgs");
          String savePath = dir+"/"+fileName; 

          //3.保存文件
          imgFile.transferTo( new File(savePath));

          //4.将图片的访问路径设置到book对象
          book.setBookImg("imgs/"+fileName);

          //5.调用service保存book到数据库
          return "/tips.jsp";
      }

  }
  ```



### 自定义拦截器

#### 创建拦截器

```java
public class MyInterceptor1 implements HandlerInterceptor {

    //预处理方法
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("--------------预处理");
        Enumeration<String> keys = request.getParameterNames();
        while (keys.hasMoreElements()){
            String key = keys.nextElement();
            if("bookId".equals(key)){
                return true;
            }
        }
        response.setStatus(400);
        return false;
    }

    //后处理方法
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        modelAndView.addObject("tips","这是通过拦截器的后处理添加的数据");
        System.out.println("--------------后处理");
    }
}
```

#### XML方式配置拦截器

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/book/query"/>
        <mvc:mapping path="/book/add"/>
        <mvc:mapping path="/student/**"/>
        <mvc:exclude-mapping path="/student/add"/>
        <bean class="com.qfedu.utils.MyInterceptor1"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 拦截器链

> 将多个拦截器按照一定的顺序构成一个执行链，先配置的先执行，如下图，执行链路为预处理1 -> 预处理2 -> 后处理2 -> 后处理1。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b59cadb3c9c4dd4a9da96e0c5b10828~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### SpringMVC统一异常处理

- 使用异常处理类进行统一处理

  ```java
  @ControllerAdvice
  public class MyExceptionHandler {

      @ExceptionHandler(NullPointerException.class)
      public String nullHandler(){
          return "/err1.jsp";
      }

      @ExceptionHandler(NumberFormatException.class)
      public String formatHandler(){
          return "/err2.jsp";
      }

  }
  ```

