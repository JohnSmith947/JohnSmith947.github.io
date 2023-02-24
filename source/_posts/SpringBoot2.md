---

title: SpringBoot

date: 2023-2-24 22:40:36

tags:
  - Spring
---
# SpringBoot2

### 前言：

​	主程序所在的包及其下面所有的子包里面的组件都会被默认扫描进来。如果要自定义扫描指定的包，可以在@SpringBootApplication 注解后指定 scanBasePackages 的路径即可。

​	在主程序中，SpringBootApplication.run 的出参即是 IOC 容器，可以使用 getBeanDefinitionNames 方法获取所有 Spring 管理的 Bean。 



### 依赖管理：

​	springboot默认引入了许多常用的依赖，可以在 pom 文件中的 parent 标签中点进去查看。

​	如果需要更改springboot自带的依赖的版本，可以在pom文件里加入properties标签，在里面输入需要修改的依赖的版本号，再重新引入依赖即可。如：

```xml
<properties>
	<mysql.version>5.1.43</mysql.version>
</properties>
```



#### .properties 配置：

​	默认配置都是映射到 MultipartProperties。

​	配置文件的值最终会绑定到某个类上，这个类会在容器中创建对象。

按需加载所有的自动配置项：

​	引入了什么场景这个场景的自动配置才会生效，比如常见的 web 开发，只需要引入 spring-boot-starter-web，那么 web 场景的配置才会开始生效。



### 常见注解：

#### @Configuration

```java
@Configuration // 告诉Spring容器这是一个配置类，等同于配置文件
public class MyConfig {
  
  /**
   * 给容器中添加 bean，使用 @Bean 注解。
   * 以方法名作为 bean 的 ID，返回类型作为 bean 的类型，返回值就是该 bean 在容器中的实例。
   * 如果在 @Bean 注解后给了值，如下所示的 “Smith”，那么该 bean 在容器中的名称则为 Smith，而不是 	    * user01。
   */
  @Bean("Smith")
  public User user01(){
    return new User("John", 18);
  }
  
}
```

注册的 bean 默认为单实例，不管获取多少次，都是同一个对象。

**@Configuration(proxyBeanMethods = true)：**

- Full 模式：这个属性如果为 true 的话，从容器中获取的配置类实例为被 spring 增强过的代理对象。每次获取该配置类中的对象的时候，都会去容器中检查该对象是不是已经存在，会让该对象保持单例，则不管获取多少次都是同一个。
- Lite 模式：如果 proxyBeanMethods 为 false，每次从容器中获取该配置类的对象的时候，创建的都是新的对象，而不是先在容器中去检查有没有该对象。这样做的优点是 springboot 启动起来会很快。

实际情况下：

- 配置类组件之间无依赖关系的时候使用 Lite 模式，减少判断。
- 配置类组件之间有依赖关系的时候，则用 Full 模式。

#### 

#### @Import：

​	给容器中导入组件，注解后面接收的是 Class 类的数组，会使用其中的类无参构造器构建一个对象导入到容器里面。

```java
@Import({User.class, Enterprise.class})
```



#### @Conditional：

​	条件装配，意思是满足 Conditional 指定的条件时才会进行注入。在 @Conditional 注解下面派生出了许多不同的注解，根据特定情况使用。

​	如下，@ConditionalOnBean(name = "user01") 的意思是，容器中如果有 user01 这个属性的时候，则将 Pet 类注入到容器中。很明显下面并没有将 user01 注入到容器中，所以容器里面也不会有 cat 属性。

​	这个注解同样可以用在类上，表示，如果容器中没有某某 bean，则下面所有打了 @Bean 注解的都不生效，都不会被注入到容器中。

```java
@Configuration
public class MyConfig {
  
  @Bean
  @ConditionalOnBean(name = "user01")
  public Pet cat(){
    return new Pet("tomcat");
  }
  
  public User user01(){
    return new User("John", 18);
  }
  
}
```



#### @ImportResource：

​	让 xml 文件生效。

​	使用 springboot 的时候，如果还有一些 bean 是在 xml 中指定注入的，此时 xml 中指定的 bean 是无法注入到容器中的，需要让 springboot 读取这个配置文件，使其生效。用法如下：

```java
@ImportResource("classpath:beans.xml") // 指定该配置文件的路径即可
```



#### @ConfigurationProperties：

​	配置绑定。

​	可以将 .properties 文件中配置的值赋给某个类。如下面代码块，通过 @Component + @ConfigurationProperties 注解可以将 .properties 文件配置的值注入到 Car 类的属性中。@ConfigurationProperties 注解需要指定 prefix，也就是 .properties 文件中的前缀。

​	需要注意的是：Car 类必须先要被注册到容器中（加 @Component 注解），不然配置绑定无法生效。

```properties
mycar.brand=BYD
```

```java
@Data
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
}
```

**第二种方式：**

​	不需要在 Car 类上加 @Component 注解了。在某些情况下，第三方包是无法自己加 @Component 注解的，那么使用这种方式就可以很好的使用配置绑定了。

​	在配置类上加 @EnableConfigurationProperties(Car.class) 注解，有两个功能：开启 Car.class 的属性配置，且将 Car 注册到容器中。

```java
@Data
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
}
```

```java
@Configuration
@EnableConfigurationProperties(Car.class)
public class MyConfig {

}
```



### 源码分析：

@SpringBootApplication

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {}
```

在 @SpringBootApplication 注解中可见，它是以下三个注解的合成注解：

1、@SpringBootConfiguration：包含@Configuration，代表当前是一个配置类

2、@ComponentScan：指定扫描哪些类

3、@EnableAutoConfiguration：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {}
```

@EnableAutoConfiguration 注解主要包含以下两个注解：

- @AutoConfigurationPackage

  ```java
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @Import({AutoConfigurationPackages.Registrar.class})
  public @interface AutoConfigurationPackage {}
  ```

  - 利用 Register 给容器中导入一系列组件。

    ```java
    static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
            Registrar() {
            }

            public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
                AutoConfigurationPackages.register(registry, (String[])(new PackageImports(metadata)).getPackageNames().toArray(new String[0]));
            }

            public Set<Object> determineImports(AnnotationMetadata metadata) {
                return Collections.singleton(new PackageImports(metadata));
            }
        }
    ```

  - 将指定的包下的所有组件导入进来，这里通常为启动类 main 方法所在的包下。

- @Import({AutoConfigurationImportSelector.class})

  - 利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件

  - 调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类

  - 利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件

  - 从META-INF/spring.factories位置来加载一个文件。

     默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
     	spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories

  - SpringBoot 启动时会自动加载很多个类，约127个左右， 会自动按需加载。




### 修改 SpringBoot 默认配置：

```java
@Bean
@ConditionalOnBean(MultipartResolver.class)  //容器中有这个类型组件
@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) //容器中没有这个名字 multipartResolver 的组件
public MultipartResolver multipartResolver(MultipartResolver resolver) {
  //给@Bean标注的方法传入了对象参数，这个参数的值就会从容器中找。
  //SpringMVC multipartResolver。防止有些用户配置的文件上传解析器不符合规范
  // Detect if the user has created a MultipartResolver but named it incorrectly
  return resolver;
}
给容器中加入了文件上传解析器；
```

> SpringBoot 默认会在底层装配好所有的组件，但是如果用户自己配置了，则以用户的优先。

**总结：**

- SpringBoot 先加载所有的自动配置类，命名规则一般为：xxxxxAutoConfiguration；
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值——从 xxxx.Properties 里面取值，xxx.Properties 和配置类进行了绑定；
- 生效的配置类就会给容器中装配很多组件；
- 只要容器中有这些组件，相当于这些功能就有了；
- `定制化配置`


- - 用户直接自己@Bean替换底层的组件
  - 用户去看这个组件是获取的配置文件什么值就去修改（在官方文档中也可以查找到该配置的写法）




### SpringBoot 使用最佳实践（大致流程）：

- 引入场景依赖


- - <https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter>


- 查看自动配置了哪些（选做）


- - 自己分析，引入场景对应的自动配置一般都生效了
  - 配置文件中 debug=true 开启自动配置报告。Negative（不生效），Positive（生效）


- 是否需要修改配置


- - 参照文档修改配置项


- - - <https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties>
    - 自己分析，找到对应 xxxx.Properties 绑定了配置文件的哪些


- - 自定义加入或者替换组件


- - - @Bean、@Component...


- - 自定义器 ：**XXXXXCustomizer**；
  - ......




### yml配置文件的用法：

```java
@Data
public class Person {
	
	private String userName;
	private Boolean boss;
	private Date birth;
	private Integer age;
	private Pet pet;
	private String[] interests;
	private List<String> animal;
	private Map<String, Object> score;
	private Set<Double> salarys;
	private Map<String, List<Pet>> allPets;
}

@Data
public class Pet {
	private String name;
	private Double weight;
}
```

```yaml
# yaml表示以上对象
# value 加双引号的时候，里面的转义字符会生效，加单引号则不会
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
```

直接在 yaml 文件中配置类的属性的时候没有提示，而 Spring 原生的配置项都会带提示的，自定义的类需要提示的时候需要引入如下依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```

在打包的时候是不需要这个提示功能的依赖的，为了避免影响性能，所以在 maven 打包插件中将该依赖进行排除：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <excludes>
          <exclude>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
          </exclude>
        </excludes>
      </configuration>
    </plugin>
  </plugins>
</build>
```



### web 开发：

#### SpringBoot 中的 SpringMVC 自动配置：

> Spring Boot provides auto-configuration for Spring MVC that **works well with most applications.(大多场景我们都无需自定义配置)**
>
> The auto-configuration adds the following features on top of Spring’s defaults:
>
> - Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
>
>
> - - 内容协商视图解析器和BeanName视图解析器
>
>
> - Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).
>
>
> - - 静态资源（包括webjars）
>
>
> - Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
>
>
> - - 自动注册 `Converter，GenericConverter，Formatter `
>
>
> - Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).
>
>
> - - 支持 `HttpMessageConverters` （后来我们配合内容协商理解原理）
>
>
> - Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-message-codes)).
>
>
> - - 自动注册 `MessageCodesResolver` （国际化用）
>
>
> - Static `index.html` support.
>
>
> - - 静态index.html 页支持
>
>
> - Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)).
>
>
> - - 自定义 `Favicon`  
>
>
> - Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).
>
>
> - - 自动使用 `ConfigurableWebBindingInitializer` ，（DataBinder负责将请求数据绑定到JavaBean上）
>
> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.9.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.
>
> **不用@EnableWebMvc注解。使用 **`@Configuration` + `WebMvcConfigurer`**自定义规则**
>
> If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.
>
> **声明 **`WebMvcRegistrations`**改变默认底层组件**
>
> If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.
>
> **使用 **`@EnableWebMvc+@Configuration+DelegatingWebMvcConfiguration 全面接管SpringMVC`



#### 静态资源访问（了解）：

​	不常用，现在一般都是前后端分离了。

> ​	By default, Spring Boot serves static content from a directory called `/static` (or `/public` or `/resources` or `/META-INF/resources`) in the classpath or from the root of the `ServletContext`. It uses the `ResourceHttpRequestHandler` from Spring MVC so that you can modify that behavior by adding your own `WebMvcConfigurer` and overriding the `addResourceHandlers` method.

​	**原理：**在请求路径和静态资源名称一致的时候，会优先找 Controller 看是否能够处理，不能处理则丢给静态资源管理器，如果都不能够处理则响应 404.

```yaml
spring:
  mvc:
  	# 修改默认静态资源的访问前缀
    static-path-pattern: /res/**

  resources:
  	# 改变默认静态资源的访问路径
    static-locations: [classpath:/haha/]
    # 禁用所有静态资源规则
    add-mappings: false   
```

- 欢迎页支持

  静态资源路径下放：index.html

- Favicon 

  > As with other static resources, Spring Boot checks for a `favicon.ico` in the configured static content locations. If such a file is present, it is automatically used as the favicon of the application.

  直接放在静态资源路径下。

- **静态资源配置原理：**

  ​	主要看 `WebMvcAutoConfiguration` 类，其中包含内部类 `WebMvcAutoConfigurationAdapter`，可在其中看到静态资源的相关配置原理。

  ​	当配置类只有一个有参构造器时：有参构造器的所有参数的值都会从容器中去确定，如 `WebMvcAutoConfigurationAdapter`。

  ```java
  public WebMvcAutoConfigurationAdapter(WebProperties webProperties, WebMvcProperties mvcProperties, ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider, ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider, ObjectProvider<DispatcherServletPath> dispatcherServletPath, ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
    this.resourceProperties = webProperties.getResources();
    this.mvcProperties = mvcProperties;
    this.beanFactory = beanFactory;
    this.messageConvertersProvider = messageConvertersProvider;
    this.resourceHandlerRegistrationCustomizer = (ResourceHandlerRegistrationCustomizer)resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
    this.dispatcherServletPath = dispatcherServletPath;
    this.servletRegistrations = servletRegistrations;
    this.mvcProperties.checkConfiguration();
  }
  ```



#### REST风格请求（了解）：

​	REST风格请求原理。

​	这里是基于表单提交的（html 页面）REST风格请求，现在用处不大。如果用客户端工具（如：POSTMAN）发送的话，是不走下面的过滤器的。

​	首先需要在配置文件中开启：

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true   #开启页面表单的Rest功能
```

​	参照：`org.springframework.web.filter.HiddenHttpMethodFilter#doFilterInternal`

```java
public class HiddenHttpMethodFilter extends OncePerRequestFilter {
    private static final List<String> ALLOWED_METHODS;
    public static final String DEFAULT_METHOD_PARAM = "_method";
    private String methodParam = "_method";

    public HiddenHttpMethodFilter() {
    }

  	// 可以自己编写配置类重新定义 _method 不叫这个名字
    public void setMethodParam(String methodParam) {
        Assert.hasText(methodParam, "'methodParam' must not be empty");
        this.methodParam = methodParam;
    }

    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        HttpServletRequest requestToUse = request;
      	// 默认请求方法必须是 POST
        if ("POST".equals(request.getMethod()) && request.getAttribute("javax.servlet.error.exception") == null) {
          	// 获取表单传入的 _method 属性（表单中是 hidden 的）
            String paramValue = request.getParameter(this.methodParam);
            if (StringUtils.hasLength(paramValue)) {
                String method = paramValue.toUpperCase(Locale.ENGLISH);
                if (ALLOWED_METHODS.contains(method)) {
                  	// HttpMethodRequestWrapper 重写了 getMethod() 方法，返回的是这里传入的值。
                  	// 相当于作了一层包装，将我们传入的请求方式传了进去。
                    requestToUse = new HttpMethodRequestWrapper(request, method);
                }
            }
        }

        filterChain.doFilter((ServletRequest)requestToUse, response);
    }

    static {
        ALLOWED_METHODS = Collections.unmodifiableList(Arrays.asList(HttpMethod.PUT.name(), HttpMethod.DELETE.name(), HttpMethod.PATCH.name()));
    }

    private static class HttpMethodRequestWrapper extends HttpServletRequestWrapper {
        private final String method;

        public HttpMethodRequestWrapper(HttpServletRequest request, String method) {
            super(request);
            this.method = method;
        }

        public String getMethod() {
            return this.method;
        }
    }
}
```



#### 请求映射：

​	当我们发送请求，SpringBoot 如何根据我们传入的路径知道我们要调用的是什么方法？

​	参照：`org.springframework.web.servlet.DispatcherServlet#doDispatch`

```java
@SuppressWarnings("deprecation")
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
  HttpServletRequest processedRequest = request;
  HandlerExecutionChain mappedHandler = null;
  boolean multipartRequestParsed = false;

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

  try {
    ModelAndView mv = null;
    Exception dispatchException = null;

    try {
      processedRequest = checkMultipart(request);
      multipartRequestParsed = (processedRequest != request);

      // Determine handler for the current request.
      // 确定当前请求的处理器，见下方的方法
      mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
      }
		// ...
   
      
// 通过这个方法得到处理器
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
  // this.handlerMappings 里面装的是所有 @RequestMapping 和 handler 的映射规则。
  if (this.handlerMappings != null) {
    // 循环所有的映射处理器，找到能够处理的
    for (HandlerMapping mapping : this.handlerMappings) {
      HandlerExecutionChain handler = mapping.getHandler(request);
      if (handler != null) {
        return handler;
      }
    }
  }
  return null;
}
```

> ​	默认的 `RequestMappingHandlerMapping`  实现会扫描项目目录下的所有带有  `@Controller`  和  `@RequestMapping`  类进行处理。
>
> ​	项目中可能会有需要自定义映射的方式，这时我们可以写一个类继承 `RequestMappingHandlerMapping`，重写其中的方法，再将其放到 SpringBoot 的配置类中即可。
>
> ​	如：可以重写下面的 isHandler() 方法，来改变 `RequestMappingHandlerMapping` 的扫描规则。  

```java
// org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping#isHandler
@Override
protected boolean isHandler(Class<?> beanType) {
  return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
          AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}
```



### 请求处理—常用参数注解

- @PathVariable

  ​	除了常规在路径变量上使用大括号的方式动态获取值，还可以用 Map 直接获取值，但是要注意 Map 的泛型必须是 <String, String>。

  ```java
  @GetMapping("/car/{id}/owner/{username}")
  public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                   @PathVariable("username") String name,
                                   @PathVariable Map<String,String> pv){

  	// 这里的 pv 这个 map 里存放的就是所有的路径动态变量。
  }
  ```

- @RequestHeader

  ​	获取请求头，详见下面注释。

  ```java
  @GetMapping("/car/{id}/owner/{username}")
  public Map<String,Object> getCar(@RequestHeader("User-Agent") String userAgent,
                                   @RequestHeader Map<String,String> header,
                                   @RequestHeader HttpHeaders header){

  	/**
           * 三种获取请求头的方式：
           * 1.通过 Map 获取，泛型要求 Map<String, String>;
           * 2.通过注解后填写请求头里的属性名获取单个的请求头信息：@RequestHeader("User-Agent") String userAgent;
           * 3.通过 RequestHeaders 对象获取，获取到的是整个请求头的信息;
           */
  }
  ```

- @RequestParam

  ​	和上面俩注解一样，也是可以通过 Map<String, String> 的方式获取到所有传入的变量。

- @CookieValue

  ​	获取 Cookie 里的值。通过注解获取某个 Cookie 的值或者使用 Cookie 对象来获取。用法不多赘述，可以参照源码里的注释。

- @RequestBody

  ​	获取请求体对象，常用于 POST 请求，不多赘述。

- @MatrixVariable（了解）

  ​	矩阵变量，必须放在请求路径中，在 `;` 后面。一种获取到请求传的值的方式，不太常见。




### 参数处理原理（源码分析）

​	每一个请求进来，都会从 DispatcherServlet 进入，主要的方法是 doDispatch 方法。

> 大致步骤：
>
> - 首先从 HandlerMapping 中找到能够处理请求的 Handler，也就是 Controller 里的方法；
> - 再为当前 Handler 找到一个适配器 HandlerAdapter（RequestMappingHandlerAdapter）；
> - 适配器执行目标方法，并确定方法参数的每一个值。

1、确定处理器

```java
// Determine handler for the current request.
// 确定一个 handler
mappedHandler = getHandler(processedRequest);
if (mappedHandler == null) {
  noHandlerFound(processedRequest, response);
  return;
}
```

2、确定适配器


```java
// Determine handler adapter for the current request.
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
  if (this.handlerAdapters != null) {
    for (HandlerAdapter adapter : this.handlerAdapters) {
      if (adapter.supports(handler)) {
        return adapter;
      }
    }
  }
  throw new ServletException("No adapter for handler [" + handler +
                             "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

3、执行目标方法

```java
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

最终会走到：org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#invokeHandlerMethod

--->

org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod#invokeAndHandle

```java
public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
  //这里获取参数的值
  Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
  setResponseStatus(webRequest);

  if (returnValue == null) {
    if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
      disableContentCachingIfNecessary(webRequest);
      mavContainer.setRequestHandled(true);
      return;
    }
  }
  else if (StringUtils.hasText(getResponseStatusReason())) {
    mavContainer.setRequestHandled(true);
    return;
  }

  mavContainer.setRequestHandled(false);
  Assert.state(this.returnValueHandlers != null, "No return value handlers");
  try {
    // 这里处理返回值，this.returnValueHandlers 里面存放了所有的返回值处理器
    this.returnValueHandlers.handleReturnValue(
      returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
  }
  catch (Exception ex) {
    if (logger.isTraceEnabled()) {
      logger.trace(formatErrorForReturnValue(returnValue), ex);
    }
    throw ex;
  }
}
```

--->

org.springframework.web.method.support.InvocableHandlerMethod#invokeForRequest

```java
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
  // 获取到方法参数的值
  Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
  if (logger.isTraceEnabled()) {
    logger.trace("Arguments: " + Arrays.toString(args));
  }
  return doInvoke(args);
}
```

```java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
  MethodParameter[] parameters = getMethodParameters();
  if (ObjectUtils.isEmpty(parameters)) {
    return EMPTY_ARGS;
  }

  Object[] args = new Object[parameters.length];
  for (int i = 0; i < parameters.length; i++) {
    MethodParameter parameter = parameters[i];
    parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
    args[i] = findProvidedArgument(parameter, providedArgs);
    if (args[i] != null) {
      continue;
    }
    // 这里的 resolvers 是个接口，里面包含了两个方法
    // 其一是 supportsParameter，判断是否支持该参数
    // 其二是 resolveArgument，解析参数
    if (!this.resolvers.supportsParameter(parameter)) {
      throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
    }
    try {
    // 步骤很明了，先 supportsParameter 判断是否能处理，能的话就执行 resolveArgument 进行处理
      args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
    }
    catch (Exception ex) {
      // Leave stack trace for later, exception may actually be resolved and handled...
      if (logger.isDebugEnabled()) {
        String exMsg = ex.getMessage();
        if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
          logger.debug(formatArgumentError(parameter, exMsg));
        }
      }
      throw ex;
    }
  }
  return args;
}
```

​	确定将要执行的方法的每一个参数的值。

​	SpringMVC 目标方法能写多少种参数类型取决于参数解析器能够支持哪些。

```java
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
	// 这里获取到参数解析器
  HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
  if (resolver == null) {
    throw new IllegalArgumentException("Unsupported parameter type [" +
                                       parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
  }
  return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
}
```

```java
// 遍历所有的参数解析器，找到能够处理该参数的，然后各自 HandlerMethodArgumentResolver 的 resolveArgument 方法即可 
private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
  HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
  if (result == null) {
    for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
      if (resolver.supportsParameter(parameter)) {
        result = resolver;
        this.argumentResolverCache.put(parameter, result);
        break;
      }
    }
  }
  return result;
}
```



#### 复杂参数：

​	Map、Model 作为请求参数的时候，原理是将其中的值放入了 request 请求域中，就是 request.setAttribute。

​	**Map、Model类型的参数**，会返回 mavContainer.getModel（）；---> BindingAwareModelMap 是Model 也是Map，在底层都是同一个对象。 

​	参数处理的过程还是跟上方描述的差不多，找到合适的适配器处理器等。下面是他最终被放到请求域中的源码：

**processDispatchResult**(processedRequest, response, mappedHandler, mv, dispatchException);

```java
InternalResourceView：
@Override
protected void renderMergedOutputModel(
  Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

  // Expose the model object as request attributes.
  exposeModelAsRequestAttributes(model, request);

  // Expose helpers as request attributes, if any.
  exposeHelpers(request);

  // Determine the path for the request dispatcher.
  String dispatcherPath = prepareForRendering(request, response);

  // Obtain a RequestDispatcher for the target resource (typically a JSP).
  RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
  if (rd == null) {
    throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
                               "]: Check that the corresponding file exists within your web application archive!");
  }

  // If already included or response already committed, perform include, else forward.
  if (useInclude(request, response)) {
    response.setContentType(getContentType());
    if (logger.isDebugEnabled()) {
      logger.debug("Including [" + getUrl() + "]");
    }
    rd.include(request, response);
  }

  else {
    // Note: The forwarded resource is supposed to determine the content type itself.
    if (logger.isDebugEnabled()) {
      logger.debug("Forwarding to [" + getUrl() + "]");
    }
    rd.forward(request, response);
  }
}
```

```java
protected void exposeModelAsRequestAttributes(Map<String, Object> model,
                                              HttpServletRequest request) throws Exception {

  //model中的所有数据遍历挨个放在请求域中
  model.forEach((name, value) -> {
    if (value != null) {
      request.setAttribute(name, value);
    }
    else {
      request.removeAttribute(name);
    }
  });
}
```



#### 自定义对象参数解析

​	**ServletModelAttributeMethodProcessor  这个参数处理器支持。**

- 是否为简单类型

  ```java
  public static boolean isSimpleValueType(Class<?> type) {
    return (Void.class != type && void.class != type &&
            (ClassUtils.isPrimitiveOrWrapper(type) ||
             Enum.class.isAssignableFrom(type) ||
             CharSequence.class.isAssignableFrom(type) ||
             Number.class.isAssignableFrom(type) ||
             Date.class.isAssignableFrom(type) ||
             Temporal.class.isAssignableFrom(type) ||
             URI.class == type ||
             URL.class == type ||
             Locale.class == type ||
             Class.class == type));
  }
  ```

- 解析参数

  ```JAVA
  @Override
  @Nullable
  public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                                      NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

    Assert.state(mavContainer != null, "ModelAttributeMethodProcessor requires ModelAndViewContainer");
    Assert.state(binderFactory != null, "ModelAttributeMethodProcessor requires WebDataBinderFactory");

    String name = ModelFactory.getNameForParameter(parameter);
    ModelAttribute ann = parameter.getParameterAnnotation(ModelAttribute.class);
    if (ann != null) {
      mavContainer.setBinding(name, ann.binding());
    }

    Object attribute = null;
    BindingResult bindingResult = null;

    if (mavContainer.containsAttribute(name)) {
      attribute = mavContainer.getModel().get(name);
    }
    else {
      // Create attribute instance
      try {
        attribute = createAttribute(name, parameter, binderFactory, webRequest);
      }
      catch (BindException ex) {
        if (isBindExceptionRequired(parameter)) {
          // No BindingResult parameter -> fail with BindException
          throw ex;
        }
        // Otherwise, expose null/empty value and associated BindingResult
        if (parameter.getParameterType() == Optional.class) {
          attribute = Optional.empty();
        }
        bindingResult = ex.getBindingResult();
      }
    }

    if (bindingResult == null) {
      // Bean property binding and validation;
      // skipped in case of binding failure on construction.
      WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
      if (binder.getTarget() != null) {
        if (!mavContainer.isBindingDisabled(name)) {
          bindRequestParameters(binder, webRequest);
        }
        validateIfApplicable(binder, parameter);
        if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
          throw new BindException(binder.getBindingResult());
        }
      }
      // Value type adaptation, also covering java.util.Optional
      if (!parameter.getParameterType().isInstance(attribute)) {
        attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);
      }
      bindingResult = binder.getBindingResult();
    }

    // Add resolved attribute and BindingResult at the end of the model
    Map<String, Object> bindingResultModel = bindingResult.getModel();
    mavContainer.removeAttributes(bindingResultModel);
    mavContainer.addAllAttributes(bindingResultModel);

    return attribute;
  }
  ```

  WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);

  WebDataBinder : web数据绑定器，将请求参数的值绑定到指定的JavaBean里面

  WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中。

  ​

  **GenericConversionService**：

  ​	在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（JavaBean -- Integer）



#### 自定义 Converter

​	给 WebDataBinder 里面放自己定义的 Converter。

示例：

```java
//1、WebMvcConfigurer定制化SpringMVC的功能
@Bean
public WebMvcConfigurer webMvcConfigurer(){
  return new WebMvcConfigurer() {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
      UrlPathHelper urlPathHelper = new UrlPathHelper();
      // 不移除；后面的内容。矩阵变量功能就可以生效
      urlPathHelper.setRemoveSemicolonContent(false);
      configurer.setUrlPathHelper(urlPathHelper);
    }

    @Override
    public void addFormatters(FormatterRegistry registry) {
      registry.addConverter(new Converter<String, Pet>() {

        @Override
        public Pet convert(String source) {
          // 啊猫,3
          if(!StringUtils.isEmpty(source)){
            Pet pet = new Pet();
            String[] split = source.split(",");
            pet.setName(split[0]);
            pet.setAge(Integer.parseInt(split[1]));
            return pet;
          }
          return null;
        }
      });
    }
  };
}
```



####  如何处理返回值？

​	标注了 @ResponseBody 注解的方法，或者是类上有 @RestController，在响应时会去遍历所有的返回值处理器（returnValueHandler），直到找到能够处理的处理器，然后去处理结果返回。

​	那么返回 JSON 数据的处理器最终是 RequestResponseBodyMethodProcessor；

​	不同的返回值会调用不同的处理器，RequestResponseBodyMethodProcessor 用于处理 JSON 格式的返回值。

​	RequestResponseBodyMethodProcessor  会用 HttpMessageConvertor 进行处理将数据写为 JSON；

- 内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）
- 服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据
- SpringMVC 会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？
  - 得到 MappingJackson2HttpMessageConverter 可以将对象写为 json
  - 利用 MappingJackson2HttpMessageConverter 将对象转为 json 再写出去。




#### 内容协商

```java
//org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor#writeWithMessageConverters(T, org.springframework.core.MethodParameter, org.springframework.http.server.ServletServerHttpRequest, org.springframework.http.server.ServletServerHttpResponse)
MediaType selectedMediaType = null;
MediaType contentType = outputMessage.getHeaders().getContentType();
boolean isContentTypePreset = contentType != null && contentType.isConcrete();
if (isContentTypePreset) {
  if (logger.isDebugEnabled()) {
    logger.debug("Found 'Content-Type:" + contentType + "' in response");
  }
  selectedMediaType = contentType;
}
else {
  HttpServletRequest request = inputMessage.getServletRequest();
  List<MediaType> acceptableTypes;
  try {
    acceptableTypes = getAcceptableMediaTypes(request);
  }
  catch (HttpMediaTypeNotAcceptableException ex) {
    int series = outputMessage.getServletResponse().getStatus() / 100;
    if (body == null || series == 4 || series == 5) {
      if (logger.isDebugEnabled()) {
        logger.debug("Ignoring error response content (if any). " + ex);
      }
      return;
    }
    throw ex;
  }
  List<MediaType> producibleTypes = getProducibleMediaTypes(request, valueType, targetType);

  if (body != null && producibleTypes.isEmpty()) {
    throw new HttpMessageNotWritableException(
      "No converter found for return value of type: " + valueType);
  }
  List<MediaType> mediaTypesToUse = new ArrayList<>();
  for (MediaType requestedType : acceptableTypes) {
    for (MediaType producibleType : producibleTypes) {
      if (requestedType.isCompatibleWith(producibleType)) {
        mediaTypesToUse.add(getMostSpecificMediaType(requestedType, producibleType));
      }
    }
  }
  if (mediaTypesToUse.isEmpty()) {
    if (logger.isDebugEnabled()) {
      logger.debug("No match for " + acceptableTypes + ", supported: " + producibleTypes);
    }
    if (body != null) {
      throw new HttpMediaTypeNotAcceptableException(producibleTypes);
    }
    return;
  }

  MediaType.sortBySpecificityAndQuality(mediaTypesToUse);

  for (MediaType mediaType : mediaTypesToUse) {
    if (mediaType.isConcrete()) {
      selectedMediaType = mediaType;
      break;
    }
    else if (mediaType.isPresentIn(ALL_APPLICATION_MEDIA_TYPES)) {
      selectedMediaType = MediaType.APPLICATION_OCTET_STREAM;
      break;
    }
  }

  if (logger.isDebugEnabled()) {
    logger.debug("Using '" + selectedMediaType + "', given " +
                 acceptableTypes + " and supported " + producibleTypes);
  }
}

if (selectedMediaType != null) {
  selectedMediaType = selectedMediaType.removeQualityValue();
  for (HttpMessageConverter<?> converter : this.messageConverters) {
    GenericHttpMessageConverter genericConverter = (converter instanceof GenericHttpMessageConverter ?
                                                    (GenericHttpMessageConverter<?>) converter : null);
    if (genericConverter != null ?
        ((GenericHttpMessageConverter) converter).canWrite(targetType, valueType, selectedMediaType) :
        converter.canWrite(valueType, selectedMediaType)) {
      body = getAdvice().beforeBodyWrite(body, returnType, selectedMediaType,
                                         (Class<? extends HttpMessageConverter<?>>) converter.getClass(),
                                         inputMessage, outputMessage);
      if (body != null) {
        Object theBody = body;
        LogFormatUtils.traceDebug(logger, traceOn ->
                                  "Writing [" + LogFormatUtils.formatValue(theBody, !traceOn) + "]");
        addContentDispositionHeader(inputMessage, outputMessage);
        if (genericConverter != null) {
          genericConverter.write(body, targetType, selectedMediaType, outputMessage);
        }
        else {
          ((HttpMessageConverter) converter).write(body, selectedMediaType, outputMessage);
        }
      }
      else {
        if (logger.isDebugEnabled()) {
          logger.debug("Nothing to write: null body");
        }
      }
      return;
    }
  }
}
```



​	根据客户端接收能力的不同，返回不同媒体类型的数据。

​	根据请求头中的 Accept 属性来获取客户端能够接收的数据类型（q-0.9 表示的是权重）。

原理：

- 1、判断当前响应头中是否已经有确定的媒体类型；
- 2、获取客户端支持接收的内容类型；
  - contentNegotiationManager 内容协商管理器，默认使用基于请求头的策略；
  - HeaderContentNegotiationStrategy 确定客户端可以接收的内容类型；
- 3、遍历循环当前系统的 MessageConvertor，看哪一个支持操作这个对象；
- 4、找到支持当前要返回的对象的 convertor，把 convertor 支持的媒体类型统计出来；
- 5、一般来说，客户端需要的格式为 application/json，服务端能力一般为 10 种（在此不列举，可 debug 查看）
- 6、进行内容协商的最佳匹配媒体类型；
- 7、用支持的对象转为最佳匹配媒体类型的 convertor，调用它进行转化。




在配置文件中可以开启基于请求参数的内容协商功能：

```yaml
spring:
    contentnegotiation:
      favor-parameter: true  #开启请求参数内容协商模式
```

开启后，可以在请求 url 的后面带上 format 参数，值只能是 json 或者 xml，来获取指定格式的数据。

如：http://localhost:8080/test/person?format=json

开启了这个配置后，会优先以请求参数中的 format 的值为准来返回数据。



#### 自定义 MessageConverter 以及 基于请求参数的自定义内容协商

​	实现多协议数据兼容。json、xml 或是自定义格式的数据。

**0、**@ResponseBody 响应数据出去 调用 **RequestResponseBodyMethodProcessor **处理

1、Processor 处理方法返回值。通过 **MessageConverter **处理

2、所有 **MessageConverter **合起来可以支持各种媒体类型数据的操作（读、写）

3、内容协商找到最终的 **messageConverter**；

**如何配置？**

```java
@Bean
public WebMvcConfigurer myConfiguration() {
  return new WebMvcConfigurer() {
    /**
     * 自定义的消息转换器
     * @param converters the list of configured converters to be extended
     */
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
      converters.add(new JohnMessageConverter());
    }

    /**
     * 自定的基于请求参数的内容协商
     * @param configurer
     */
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
      Map<String, MediaType> mediaTypes = new HashMap<>();
      mediaTypes.put("john",MediaType.parseMediaType("application/x-john"));
      // 让请求头方式的内容协商生效
      HeaderContentNegotiationStrategy headerContentNegotiationStrategy = new HeaderContentNegotiationStrategy();
      // url参数方式的内容协商
      ParameterContentNegotiationStrategy parameterContentNegotiationStrategy = new ParameterContentNegotiationStrategy(mediaTypes);
      configurer.strategies(Arrays.asList(headerContentNegotiationStrategy, parameterContentNegotiationStrategy));
    }
  };
}
```



### 试图解析与模板引擎

#### 视图解析原理流程

1、目标方法处理的过程中，所有数据都会被放在 ModelAndViewContainer 里面。包括数据和视图地址

2、方法的参数是一个自定义类型对象（从请求参数中确定的），把他重新放在ModelAndViewContainer

3、任何目标方法执行完成以后都会返回 ModelAndView（数据和视图地址）。

4、processDispatchResult  处理派发结果（页面改如何响应）

- render(mv, request, response); 进行页面渲染逻辑

  - 根据方法的String返回值得到 View 对象【定义了页面的渲染逻辑】
    - 所有的视图解析器尝试是否能根据当前返回值得到View对象
    - 得到了redirect:/main.html --> Thymeleaf new RedirectView()
    - ContentNegotiationViewResolver 里面包含了下面所有的视图解析器，内部还是利用下面所有视图解析器得到视图对象。
    - view.render(mv.getModelInternal(), request, response);   视图对象调用自定义的render进行页面渲染工作
      - RedirectView 如何渲染【重定向到一个页面】
      - 获取目标url地址
      - response.sendRedirect(encodedURL);


**视图解析：**

- 返回值以 forward: 开始： new InternalResourceView(forwardUrl); -->  转发request.getRequestDispatcher(path).forward(request, response); 
- 返回值以 redirect: 开始：new RedirectView() --》 render就是重定向 
- 返回值是普通字符串： new ThymeleafView（）---> 




#### 拦截器

​	定义一个拦截器，实现 HandlerInterceptor 接口，一共三个方法实现，看下面的注释。

```java
@Log4j2
public class JohnInterceptor implements HandlerInterceptor {

  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    String requestURI = request.getRequestURI();
    log.info("拦截到的uri：{}", requestURI);
    return HandlerInterceptor.super.preHandle(request, response, handler);
  }

  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    log.info("方法执行后：{}", request.toString());
    HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
  }

  @Override
  public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    log.info("页面渲染完后：{}", response.toString());
    HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
  }
}
```

​	把自定义的拦截器注册到容器里面去，可以指定拦截的路径和放行的路径。

```java
@Configuration
public class MyConfig implements WebMvcConfigurer{
    /**
     * 添加拦截器
      */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new JohnInterceptor())
                .addPathPatterns("/**");
        WebMvcConfigurer.super.addInterceptors(registry);
    }
}
```



#### 拦截器原理

1、根据当前请求，找到 HandlerExecutionChain【可以处理请求的handler以及handler的所有拦截器】

2、顺序执行所有拦截器的 preHandle方法

- 如果当前拦截器 preHandler 返回为 true，则执行下一个拦截器的 preHandle
- 如果当前拦截器返回为 false，直接倒序执行所有已经执行了的拦截器的  afterCompletion；

3、如果任何一个拦截器返回 false。直接跳出不执行目标方法

4、所有拦截器都返回 true，则执行目标方法

5、倒序执行所有拦截器的 postHandle 方法

6、前面的步骤有任何异常都会直接倒序触发 afterCompletion

7、页面成功渲染完成以后，也会倒序触发 afterCompletion

> 流程：
>
> 拦截器A.preHandle -> 拦截器B.preHandle -> 拦截器C.preHandle
>
> -> 拦截器C.postHandle -> 拦截器B.postHandle -> 拦截器A.postHandle
>
> -> 拦截器C.afterCompletion-> 拦截器B.afterCompletion-> 拦截器A.afterCompletion
>
> 注意：在 preHandle 和 postHandle 的过程中，如果发生了异常，则直接执行当前拦截器的 afterCompletion 方法，并倒序往前执行前面拦截器的 afterCompletion 方法。



### 异常处理

1、默认规则

- 默认情况下，SpringBoot 提供了 `/error` 处理所有错误的映射；
- 对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于浏览器客户端，响应一个”whitelabel“错误视图，以HTML格式呈现相同的数据
- 要完全替换默认行为，可以实现 `ErrorController` 并注册该类型的Bean定义，或添加 `ErrorAttributes` 类型的组件以使用现有机制但替换其内容。



2、定制错误处理逻辑

​	这里的三种方式，默认对应了第 4 点中 HandlerExceptionResolverComposite 里面的三个解析器。

- 通过 @ControllerAdvice + @ExceptionHandler 注解来处理全局异常（在SpringMVC篇中已经写过，在此不赘述）。由 ExceptionHandlerExceptionResolver 处理。

- 通过自定义异常类 + @ResponseStatus 注解来实现异常的处理。由 ResponseStatusExceptionResolver 处理。

  ```java
  @ResponseStatus(value = HttpStatus.CONFLICT, reason = "随便定义的异常")
  public class JohnException extends RuntimeException{
    private static final long serialVersionUID = -8913972407058563347L;

    public JohnException() {
    }

    public JohnException(String message) {
      super(message);
    }
  }
  ```

- Spring 底层的异常，如：参数类型转换异常。由 DefaultHandlerExceptionResolver 处理。

> 自定义异常处理解析器：
>
> ​	实现 HandlerExceptionResolver 类后放入容器中即可。
>
> ​	可以作为默认的全局异常处理，不过需要调高优先级，不然的话轮不到他。
>
> ```java
> @Component
> @Order(Ordered.HIGHEST_PRECEDENCE)
> public class JohnHandlerExceptionResolver implements HandlerExceptionResolver {
>   @Override
>   public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
>     try {
>       response.sendError(666);
>     } catch (IOException e) {
>       throw new RuntimeException(e);
>     }
>     return new ModelAndView();
>   }
> }
> ```



3、异常处理自动配置原理

- ErrorMvcAutoConfiguraiton 自动配置异常处理规则

  - 容器中的组件：DefaultErrorAttributes：定义了错误页面可以包含哪些数据

    ```java
    /**
     * ErrorAttributes的默认实现。在可能的情况下提供以下属性: 
     * timestamp -错误被提取的时间 
     * status -状态码 
     * error -错误原因 
     * exception -根异常的类名(如果配置) 
     * message—异常消息(如果配置了) 
     * errors -来自BindingResult异常的任何ObjectErrors(如果配置) 
     * trace——异常堆栈跟踪(如果配置了) 
     * path—引发异常时的URL路径 
     */
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver, Ordered {
      ...
    }
    ```

  - 容器中的组件：BasicErrorController：JSON+白页适配响应

    - 处理默认 `/error` 路径的请求，页面响应 new ModelAndView("error", model);
    - 容器中有组件 View，ID 是 error，默认响应错误页；
    - 容器中有组件 BeanNameResolver（视图解析器），按照返回的视图名作为组件的 ID 去容器中找 View 对象；

  - 容器中的组件：DefaultErrorViewResolver

    - 如果发生错误，会以 HTTP 状态码作为视图页地址（viewName），找到真正的页面（这里可能指的是有模板引擎存在的情况下定制的错误页面）



4、异常处理步骤流程

- 执行目标方法，目标方法运行期间有任何异常都会被 catch，而且代表着当前的请求已经结束了，并且用 dispatchException 封装；

- 进入视图解析流程（页面渲染）

  ```java
  //org.springframework.web.servlet.DispatcherServlet#processDispatchResult
  if (exception != null) {
    if (exception instanceof ModelAndViewDefiningException) {
      logger.debug("ModelAndViewDefiningException encountered", exception);
      mv = ((ModelAndViewDefiningException) exception).getModelAndView();
    }
    else {
      Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
      // 这里进入下一个步骤
      mv = processHandlerException(request, response, handler, exception);
      errorView = (mv != null);
    }
  }
  ```

- 开始处理异常，在下面这个方法里会遍历所有的处理异常的解析器，里面包含了：

  ```java
  //org.springframework.web.servlet.DispatcherServlet#processHandlerException
  这里最后还是会将异常抛出
  ```

  - DefaultErrorAttributes：把异常信息保存到请求域后直接就返回 null 了

    ```java
    //org.springframework.boot.web.servlet.error.DefaultErrorAttributes#resolveException
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,
                                         Exception ex) {
      storeErrorAttributes(request, ex);
      return null;
    }
    ```

  - HandlerExceptionResolverComposite：处理程序异常解析器组合，里面包含了三个异常解析器，默认是没有解析器可以处理这个异常（如果有全局异常处理的话，会走标记了@ExceptionHandler注解的方法，看是否能处理当前异常），所以还是返回了 null

    ```java
    //org.springframework.web.servlet.handler.HandlerExceptionResolverComposite#resolveException
    @Override
    @Nullable
    public ModelAndView resolveException(
      HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

      if (this.resolvers != null) {
        for (HandlerExceptionResolver handlerExceptionResolver : this.resolvers) {
          ModelAndView mav = handlerExceptionResolver.resolveException(request, response, handler, ex);
          if (mav != null) {
            return mav;
          }
        }
      }
      return null;
    }
    ```

- 由于没有处理器可以处理这个异常，底层最后会重新发送一个请求，也就是 `/error`。

  - 请求将被底层的 BasicErrorController 处理；
  - 解析错误视图，遍历所有的 ErrorViewResolver 看谁能够解析；
  - 默认的 DefaultErrorViewResolver，作用是把响应状态码作为错误页的地址返回出去；
  - 模板引擎最终响应这个页面。




### 定制化原理

1、定制化的常见方式

- 修改配置文件
- Web 应用编写一个配置类实现 WebMvcConfigurer 即可实现定制化 web 功能，+@Bean 给容器中再拓展一些组件
- @EnableWebMvc + 实现 WebMvcConfigurer + @Bean 可实现全面接管 SpringMVC，所有规则全部都由自己配置，实现功能的拓展
  - 一旦使用了 @EnableWebMvc，会@Import(DelegatingWebMvcConfiguration.class)，这样只保证了 SpringMvc 的最基本的使用，那些自动配置的类都会失效，所以需要自己手动配置。
  - @WebMvcAutoConfiguration 上有一个前提条件：@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)，如果配置了 WebMvcConfigurationSupport 相关的类进去了，那么默认配置就不会生效了。




### 数据访问（MySQL、连接池等参数配置）

### NoSQL（Redis）

### 单元测试（Junit5）



### 指标监控

#### SpringBootActuator

​	未来每一个微服务在云上部署以后，我们都需要对其进行监控、追踪、审计、控制等。SpringBoot就抽取了Actuator场景，使得我们每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

```xml
<!-- 引入如下依赖 -->
 <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



**如何使用？**

直接访问：<http://localhost:8080/actuator/>**

配置文件：

```yaml
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露所有端点
```

可视化：https://github.com/codecentric/spring-boot-admin



#### Endpoint

| ID                 | 描述                                       |
| ------------------ | ---------------------------------------- |
| `auditevents`      | 暴露当前应用程序的审核事件信息。需要一个`AuditEventRepository组件`。 |
| `beans`            | 显示应用程序中所有Spring Bean的完整列表。               |
| `caches`           | 暴露可用的缓存。                                 |
| `conditions`       | 显示自动配置的所有条件信息，包括匹配或不匹配的原因。               |
| `configprops`      | 显示所有`@ConfigurationProperties`。          |
| `env`              | 暴露Spring的属性`ConfigurableEnvironment`     |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。需要一个或多个`Flyway`组件。   |
| `health`           | 显示应用程序运行状况信息。                            |
| `httptrace`        | 显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应）。需要一个`HttpTraceRepository`组件。 |
| `info`             | 显示应用程序信息。                                |
| `integrationgraph` | 显示Spring `integrationgraph` 。需要依赖`spring-integration-core`。 |
| `loggers`          | 显示和修改应用程序中日志的配置。                         |
| `liquibase`        | 显示已应用的所有Liquibase数据库迁移。需要一个或多个`Liquibase`组件。 |
| `metrics`          | 显示当前应用程序的“指标”信息。                         |
| `mappings`         | 显示所有`@RequestMapping`路径列表。               |
| `scheduledtasks`   | 显示应用程序中的计划任务。                            |
| `sessions`         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。 |
| `shutdown`         | 使应用程序正常关闭。默认禁用。                          |
| `startup`          | 显示由`ApplicationStartup`收集的启动步骤数据。需要使用`SpringApplication`进行配置`BufferingApplicationStartup`。 |
| `threaddump`       | 执行线程转储。                                  |

如果您的应用程序是Web应用程序（Spring MVC，Spring WebFlux或Jersey），则可以使用以下附加端点：

| ID           | 描述                                       |
| ------------ | ---------------------------------------- |
| `heapdump`   | 返回`hprof`堆转储文件。                          |
| `jolokia`    | 通过HTTP暴露JMX bean（需要引入Jolokia，不适用于WebFlux）。需要引入依赖`jolokia-core`。 |
| `logfile`    | 返回日志文件的内容（如果已设置`logging.file.name`或`logging.file.path`属性）。支持使用HTTP`Range`标头来检索部分日志文件的内容。 |
| `prometheus` | 以Prometheus服务器可以抓取的格式公开指标。需要依赖`micrometer-registry-prometheus`。 |

##### 最常用的 Endpoint：

- Health：健康状况
- Metrics：运行时指标
- Loggers：日志记录



##### 管理Endpoint

​	默认所有的 Endpoint 都是开启的。

> Endpoint 暴露的方式：
>
> - HTTP：默认只暴露 health 和 info
> - JMX：默认暴露所有 Endpoint

```yaml
# 开启beans endpoint
management:
  endpoint:
    beans:
      enabled: true

# 关闭所有 endpoint，只打开自己想要的
management:
  endpoints:
    enabled-by-default: false
  endpoint:
    beans:
      enabled: true
    health:
      enabled: true
```



#### 定制 Endpoint

##### 定制 Health 信息

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}

// 构建Health
Health build = Health.down()
                .withDetail("msg", "error service")
                .withDetail("code", "500")
                .withException(new RuntimeException())
                .build();
```

```yaml
management:
    health:
      enabled: true
      show-details: always #总是显示详细信息。可显示每个模块的状态信息
```

```java
@Component
public class MyComHealthIndicator extends AbstractHealthIndicator {

    /**
     * 真实的检查方法
     * @param builder
     * @throws Exception
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //mongodb。  获取连接进行测试
        Map<String,Object> map = new HashMap<>();
        // 检查完成
        if(1 == 2){
//            builder.up(); //健康
            builder.status(Status.UP);
            map.put("count",1);
            map.put("ms",100);
        }else {
//            builder.down();
            builder.status(Status.OUT_OF_SERVICE);
            map.put("err","连接超时");
            map.put("ms",3000);
        }


        builder.withDetail("code",100)
                .withDetails(map);

    }
}
```



##### 定制 info 信息

配置文件方式

```yaml
info:
  appName: boot-admin
  version: 2.0.1
  mavenProjectName: @project.artifactId@  #使用@@可以获取maven的pom文件值
  mavenProjectVersion: @project.version@
```

编写 InfoContributor

```java
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("example",
                Collections.singletonMap("key", "value"));
    }

}
```



##### 定制 Metrics 信息

示例：

```java
class MyService{
    Counter counter;
    public MyService(MeterRegistry meterRegistry){
         counter = meterRegistry.counter("myservice.method.running.counter");
    }

    public void hello() {
        counter.increment();
    }
}


//也可以使用下面的方式
@Bean
MeterBinder queueSize(Queue queue) {
    return (registry) -> Gauge.builder("queueSize", queue::size).register(registry);
}
```



##### 定制 Endpoint

```java
@Component
@Endpoint(id = "container")
public class DockerEndpoint {


    @ReadOperation
    public Map getDockerInfo(){
        return Collections.singletonMap("info","docker started...");
    }

    @WriteOperation
    private void restartDocker(){
        System.out.println("docker restarted....");
    }

}
```



### 原理解析

#### Profile 功能

- 默认配置文件：application.yaml：任何时候都会加载

- 指定环境配置文件：application-{env}.yaml

- 激活指定环境

  - 配置文件激活

  ```yaml
  spring:
    profiles:
      active: dev
  ```

  - 命令行激活

  ```shell
  java -jar xxx.jar --spring.profiles.active=prod  --person.name=haha
  ```

- 默认配置与环境配置同时生效

- 同名配置项，profile 配置优先



#### @Profile

```java
@Data
@Component
@ConfigurationProperties("person")//在配置文件中配置
public class Person{
    private String name;
    private Integer age;
}
```

application.yaml

```yaml
person: 
  name: lun
  age: 8
```

```java
public interface Person {

   String getName();
   Integer getAge();

}

@Profile("test")//加载application-test.yaml里的
@Component
@ConfigurationProperties("person")
@Data
public class Worker implements Person {

    private String name;
    private Integer age;
}

@Profile(value = {"prod","default"})//加载application-prod.yaml里的
@Component
@ConfigurationProperties("person")
@Data
public class Boss implements Person {

    private String name;
    private Integer age;
}
```

application-test.yaml

```yaml
person:
  name: test-张三

server:
  port: 7000
```

application-prod.yaml

```yaml
person:
  name: prod-张三

server:
  port: 8000
```

@Profile还可以修饰在方法上：

```java
class Color {
}

@Configuration
public class MyConfig {

    @Profile("prod")
    @Bean
    public Color red(){
        return new Color();
    }

    @Profile("test")
    @Bean
    public Color green(){
        return new Color();
    }
}
```



#### 配置文件加载顺序

​	从上至下加载，下面覆盖上面的。

- 当前 jar 包内部的 application.properties 和 application.yaml
- 当前 jar 包内部的 application-{env}.properties 和 application-{env}.yaml
- 引用的外部 jar 包的 application.properties 和 application.yaml
- 当前 jar 包内部的 application-{env}.properties 和 application-{env}.yaml



#### 自定义 starter

##### starter启动原理

- starter的pom.xml引入autoconfigure依赖

  ​			starter -> autoconfigure -> spring-boot-starter

- autoconfigure包中配置使用`META-INF/spring.factories`中`EnableAutoConfiguration`的值，使得项目启动加载指定的自动配置类



##### 自定义 starter

- 目标：创建`HelloService`的自定义starter。
- 创建两个工程，分别命名为`hello-spring-boot-starter`（普通Maven工程），`hello-spring-boot-starter-autoconfigure`（需用用到Spring Initializr创建的Maven工程）。
- `hello-spring-boot-starter`无需编写什么代码，只需让该工程引入`hello-spring-boot-starter-autoconfigure`依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lun</groupId>
    <artifactId>hello-spring-boot-starter</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.lun</groupId>
            <artifactId>hello-spring-boot-starter-autoconfigure</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

- `hello-spring-boot-starter-autoconfigure`的pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.2</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.lun</groupId>
	<artifactId>hello-spring-boot-starter-autoconfigure</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<name>hello-spring-boot-starter-autoconfigure</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
	</dependencies>
</project>
```

- 创建4个文件
  - `com/lun/hello/auto/HelloServiceAutoConfiguration`
  - `com/lun/hello/bean/HelloProperties`
  - `com/lun/hello/service/HelloService`
  - `src/main/resources/META-INF/spring.factories`

```java
@Configuration
@ConditionalOnMissingBean(HelloService.class)
@EnableConfigurationProperties(HelloProperties.class)//默认HelloProperties放在容器中
public class HelloServiceAutoConfiguration {

    @Bean
    public HelloService helloService(){
        return new HelloService();
    }

}
```

```java
@ConfigurationProperties("hello")
public class HelloProperties {
    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

```java
/**
 * 默认不要放在容器中
 */
public class HelloService {

    @Autowired
    private HelloProperties helloProperties;

    public String sayHello(String userName){
        return helloProperties.getPrefix() + ": " + userName + " > " + helloProperties.getSuffix();
    }
}
```

```
# Auto Configure -> spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.lun.hello.auto.HelloServiceAutoConfiguration
```

- 用maven插件，将两工程install到本地。
- 接下来，测试使用自定义starter，创建springboot工程，引入`hello-spring-boot-starter`依赖
- 添加配置文件`application.properties`：

```properties
hello.prefix=hello
hello.suffix=666
```

- 添加单元测试类：

```java
@SpringBootTest
class HelloSpringBootStarterTestApplicationTests {

    @Autowired
    private HelloService helloService;

    @Test
    void contextLoads() {
        // System.out.println(helloService.sayHello("lun"));
        Assertions.assertEquals("hello: lun > 666", helloService.sayHello("lun"));
    }

}
```



### SpringApplication 创建初始化流程

#### SpringBoot 启动过程

- 启动类：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloSpringBootStarterTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloSpringBootStarterTestApplication.class, args);
    }

}
```

```java
public class SpringApplication {
    
    ...
    
	public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
		return run(new Class<?>[] { primarySource }, args);
	}
    
    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
    
    //先看看new SpringApplication(primarySources)，下一节再看看run()
	public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
	}
    
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        //WebApplicationType是枚举类，有NONE,SERVLET,REACTIVE,下行webApplicationType是SERVLET
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
        
        //初始启动引导器，去spring.factories文件中找org.springframework.boot.Bootstrapper，但我找不到实现Bootstrapper接口的类
		this.bootstrappers = new ArrayList<>(getSpringFactoriesInstances(Bootstrapper.class));
		
        //去spring.factories找 ApplicationContextInitializer
        setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		
        //去spring.factories找 ApplicationListener
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));

        this.mainApplicationClass = deduceMainApplicationClass();
	}
 	
    private Class<?> deduceMainApplicationClass() {
		try {
			StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
			for (StackTraceElement stackTraceElement : stackTrace) {
				if ("main".equals(stackTraceElement.getMethodName())) {
					return Class.forName(stackTraceElement.getClassName());
				}
			}
		}
		catch (ClassNotFoundException ex) {
			// Swallow and continue
		}
		return null;
	}
    
    ...
    
}
```

- return new SpringApplication(primarySources).run(args);

```java
public class SpringApplication {
    
    ...
    
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();//开始计时器
		stopWatch.start();//开始计时
        
        //1.
        //创建引导上下文（Context环境）createBootstrapContext()
        //获取到所有之前的 bootstrappers 挨个执行 intitialize() 来完成对引导启动器上下文环境设置
		DefaultBootstrapContext bootstrapContext = createBootstrapContext();
		
        //2.到最后该方法会返回这context
        ConfigurableApplicationContext context = null;
		
        //3.让当前应用进入headless模式
        configureHeadlessProperty();
        
        //4.获取所有 RunListener（运行监听器）,为了方便所有Listener进行事件感知
		SpringApplicationRunListeners listeners = getRunListeners(args);
		
        //5. 遍历 SpringApplicationRunListener 调用 starting 方法；
		// 相当于通知所有感兴趣系统正在启动过程的人，项目正在 starting。
        listeners.starting(bootstrapContext, this.mainApplicationClass);
		try {
            //6.保存命令行参数 ApplicationArguments
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			
            //7.准备环境
            ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
			configureIgnoreBeanInfo(environment);
			
            /*打印标志
              .   ____          _            __ _ _
             /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
            ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
             \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
              '  |____| .__|_| |_|_| |_\__, | / / / /
             =========|_|==============|___/=/_/_/_/
             :: Spring Boot ::                (v2.4.2)
            */
            Banner printedBanner = printBanner(environment);
            
            // 创建IOC容器（createApplicationContext（））
			// 根据项目类型webApplicationType（NONE,SERVLET,REACTIVE）创建容器，
			// 当前会创建 AnnotationConfigServletWebServerApplicationContext
			context = createApplicationContext();
			context.setApplicationStartup(this.applicationStartup);
            
            //8.准备ApplicationContext IOC容器的基本信息
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
			//9.刷新IOC容器,创建容器中的所有组件,Spring框架的内容
            refreshContext(context);
			//该方法没内容，大概为将来填入
			afterRefresh(context, applicationArguments);
			stopWatch.stop();//停止计时
			if (this.logStartupInfo) {//this.logStartupInfo默认是true
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
            //10.
			listeners.started(context);
            
            //11.调用所有runners
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
            //13.
			handleRunFailure(context, ex, listeners);
			throw new IllegalStateException(ex);
		}

		try {
            //12.
			listeners.running(context);
		}
		catch (Throwable ex) {
            //13.
			handleRunFailure(context, ex, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
 
    //1. 
    private DefaultBootstrapContext createBootstrapContext() {
		DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
		this.bootstrappers.forEach((initializer) -> initializer.intitialize(bootstrapContext));
		return bootstrapContext;
	}
    
    //3.
   	private void configureHeadlessProperty() {
        //this.headless默认为true
		System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS,
				System.getProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
	}
    
    private static final String SYSTEM_PROPERTY_JAVA_AWT_HEADLESS = "java.awt.headless";
    
    //4.
    private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		//getSpringFactoriesInstances 去 spring.factories 找 SpringApplicationRunListener
        return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args),
				this.applicationStartup);
	}
    
    //7.准备环境
    private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
			DefaultBootstrapContext bootstrapContext, ApplicationArguments applicationArguments) {
		// Create and configure the environment
        //返回或者创建基础环境信息对象，如：StandardServletEnvironment, StandardReactiveWebEnvironment
		ConfigurableEnvironment environment = getOrCreateEnvironment();
        //配置环境信息对象,读取所有的配置源的配置属性值。
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		//绑定环境信息
        ConfigurationPropertySources.attach(environment);
        //7.1 通知所有的监听器当前环境准备完成
		listeners.environmentPrepared(bootstrapContext, environment);
		DefaultPropertiesPropertySource.moveToEnd(environment);
		configureAdditionalProfiles(environment);
		bindToSpringApplication(environment);
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
					deduceEnvironmentClass());
		}
		ConfigurationPropertySources.attach(environment);
		return environment;
	}
    
    //8.
    private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
		//保存环境信息
        context.setEnvironment(environment);
        //IOC容器的后置处理流程
		postProcessApplicationContext(context);
        //应用初始化器
		applyInitializers(context);
        //8.1 遍历所有的 listener 调用 contextPrepared。
        //EventPublishRunListenr通知所有的监听器contextPrepared
		listeners.contextPrepared(context);
		bootstrapContext.close(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		if (this.lazyInitialization) {
			context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[0]));
        //8.2
		listeners.contextLoaded(context);
	}

    //11.调用所有runners
    private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<>();
        
        //获取容器中的 ApplicationRunner
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
		//获取容器中的  CommandLineRunner
        runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
        //合并所有runner并且按照@Order进行排序
		AnnotationAwareOrderComparator.sort(runners);
        //遍历所有的runner。调用 run 方法
		for (Object runner : new LinkedHashSet<>(runners)) {
			if (runner instanceof ApplicationRunner) {
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}
    
    //13.
    private void handleRunFailure(ConfigurableApplicationContext context, Throwable exception,
			SpringApplicationRunListeners listeners) {
		try {
			try {
				handleExitCode(context, exception);
				if (listeners != null) {
                    //14.
					listeners.failed(context, exception);
				}
			}
			finally {
				reportFailure(getExceptionReporters(context), exception);
				if (context != null) {
					context.close();
				}
			}
		}
		catch (Exception ex) {
			logger.warn("Unable to close ApplicationContext", ex);
		}
		ReflectionUtils.rethrowRuntimeException(exception);
	}
    
    ...
}
```

```java
//2. new SpringApplication(primarySources).run(args) 最后返回的接口类型
public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {
    String CONFIG_LOCATION_DELIMITERS = ",; \t\n";
    String CONVERSION_SERVICE_BEAN_NAME = "conversionService";
    String LOAD_TIME_WEAVER_BEAN_NAME = "loadTimeWeaver";
    String ENVIRONMENT_BEAN_NAME = "environment";
    String SYSTEM_PROPERTIES_BEAN_NAME = "systemProperties";
    String SYSTEM_ENVIRONMENT_BEAN_NAME = "systemEnvironment";
    String APPLICATION_STARTUP_BEAN_NAME = "applicationStartup";
    String SHUTDOWN_HOOK_THREAD_NAME = "SpringContextShutdownHook";

    void setId(String var1);

    void setParent(@Nullable ApplicationContext var1);

    void setEnvironment(ConfigurableEnvironment var1);

    ConfigurableEnvironment getEnvironment();

    void setApplicationStartup(ApplicationStartup var1);

    ApplicationStartup getApplicationStartup();

    void addBeanFactoryPostProcessor(BeanFactoryPostProcessor var1);

    void addApplicationListener(ApplicationListener<?> var1);

    void setClassLoader(ClassLoader var1);

    void addProtocolResolver(ProtocolResolver var1);

    void refresh() throws BeansException, IllegalStateException;

    void registerShutdownHook();

    void close();

    boolean isActive();

    ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
}

```

```java
//2. new SpringApplication(primarySources).run(args) 最后返回的接口类型
public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {
    String CONFIG_LOCATION_DELIMITERS = ",; \t\n";
    String CONVERSION_SERVICE_BEAN_NAME = "conversionService";
    String LOAD_TIME_WEAVER_BEAN_NAME = "loadTimeWeaver";
    String ENVIRONMENT_BEAN_NAME = "environment";
    String SYSTEM_PROPERTIES_BEAN_NAME = "systemProperties";
    String SYSTEM_ENVIRONMENT_BEAN_NAME = "systemEnvironment";
    String APPLICATION_STARTUP_BEAN_NAME = "applicationStartup";
    String SHUTDOWN_HOOK_THREAD_NAME = "SpringContextShutdownHook";

    void setId(String var1);

    void setParent(@Nullable ApplicationContext var1);

    void setEnvironment(ConfigurableEnvironment var1);

    ConfigurableEnvironment getEnvironment();

    void setApplicationStartup(ApplicationStartup var1);

    ApplicationStartup getApplicationStartup();

    void addBeanFactoryPostProcessor(BeanFactoryPostProcessor var1);

    void addApplicationListener(ApplicationListener<?> var1);

    void setClassLoader(ClassLoader var1);

    void addProtocolResolver(ProtocolResolver var1);

    void refresh() throws BeansException, IllegalStateException;

    void registerShutdownHook();

    void close();

    boolean isActive();

    ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
}

```

```
#4.
#spring.factories
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener

```

```java
class SpringApplicationRunListeners {

	private final Log log;

	private final List<SpringApplicationRunListener> listeners;

	private final ApplicationStartup applicationStartup;

	SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners,
			ApplicationStartup applicationStartup) {
		this.log = log;
		this.listeners = new ArrayList<>(listeners);
		this.applicationStartup = applicationStartup;
	}

    //5.遍历 SpringApplicationRunListener 调用 starting 方法；
	//相当于通知所有感兴趣系统正在启动过程的人，项目正在 starting。
	void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
		doWithListeners("spring.boot.application.starting", (listener) -> listener.starting(bootstrapContext),
				(step) -> {
					if (mainApplicationClass != null) {
						step.tag("mainApplicationClass", mainApplicationClass.getName());
					}
				});
	}
    
    //7.1
    void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
		doWithListeners("spring.boot.application.environment-prepared",
				(listener) -> listener.environmentPrepared(bootstrapContext, environment));
	}
    
    //8.1
    void contextPrepared(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.context-prepared", (listener) -> listener.contextPrepared(context));
	}
    
    //8.2
    void contextLoaded(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.context-loaded", (listener) -> listener.contextLoaded(context));
	}
    
    //10.
    void started(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.started", (listener) -> listener.started(context));
	}
    
    //12.
    void running(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.running", (listener) -> listener.running(context));
	}
    
    //14.
    void failed(ConfigurableApplicationContext context, Throwable exception) {
		doWithListeners("spring.boot.application.failed",
				(listener) -> callFailedListener(listener, context, exception), (step) -> {
					step.tag("exception", exception.getClass().toString());
					step.tag("message", exception.getMessage());
				});
	}
    
    private void doWithListeners(String stepName, Consumer<SpringApplicationRunListener> listenerAction,
			Consumer<StartupStep> stepAction) {
		StartupStep step = this.applicationStartup.start(stepName);
		this.listeners.forEach(listenerAction);
		if (stepAction != null) {
			stepAction.accept(step);
		}
		step.end();
	}
    
    ...
    
}
```

