---

title: Spring IOC

date: 2022-12-11 14:51:00

tags:

---
# SpringIOC 基本

## 一、Spring的作用

1、轻量级：体积小，无侵入性；

2、主要作用：解耦；

3、控制反转：把创建对象的工作交由spring容器完成，创建对象即可赋值（DI依赖注入）；

4、面向切面：在不改变原有业务逻辑的前提下，对业务进行曾强。



## 二、Spring架构体系

![img](https://img-blog.csdnimg.cn/img_convert/95500e85e2ab9ab98e4d684252c963d2.gif)



## 三、IOC流程

### （一）通过set方法注入属性

**1、创建普通maven工程**

**2、导入相关依赖：**

​        只需要导入spring-context包即可（等于导入了spring-beans和spring-core），本次使用的版本为5.2.10 RELEASE。

```xml
    <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>5.2.10.RELEASE</version>
    </dependency>
```

**3、在resource中创建springContext.xml：**

​        按需导入spring相关配置，在文件头中可定义。

![img](https://img-blog.csdnimg.cn/a4df3a08c0a44174b3d36d08a30e37d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_20,color_FFFFFF,t_70,g_se,x_16)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

**4、创建实体类：**

​        添加get、set和toString方法（为方便演示不使用lombok）。

```java
package org.example.spring.bean;


/**
 * 实体类
 *
 * @Author: John Smith
 * @Date: 2022/4/19
 */
public class Student {

    private String name;

    private String gender;

    private Integer age;
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", gender='" + gender + '\'' +
                ", age=" + age +
                '}';
    }
}

```

**5、将Student类交给spring容器管理并注入属性：**

​        在xml文件中如下定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--通过bean标签，将student类交给spring容器管理-->
    <bean id="student" class="org.example.spring.bean.Student">
        <!--在bean标签中使用property标签对bean的属性进行注入-->
        <property name="age" value="20"/>
        <!--property的value属性会自动转换基本类型，如int和string等-->
        <property name="name" value="张三"/>
        <property name="gender" value="男"/>
    </bean>

</beans>
```

 **6、创建测试类：**

​        测试student属性是否注入成功：

```java
package org.example.spring;

import org.example.spring.bean.Student;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * 测试类
 *
 * @Author: John Smith
 * @Date: 2022/4/19
 */
public class TestMain {
    public static void main(String[] args) {
        // 通过ClassPathXmlApplicationContext类读取xml配置
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("springContext.xml");
        // 通过context.getBean()方法得到spring容器内的对象，入参为在xml中定义的bean标签的id
        Student student = (Student)context.getBean("student");
        // 输出对象信息
        System.out.println(student);
    }
}

```

​        输出内容：

![img](https://img-blog.csdnimg.cn/41f8aedd01f646bbbb55c14852032d9a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_20,color_FFFFFF,t_70,g_se,x_16)

> 注意：
>
> ​	实体类中必须要存在set方法，不然属性无法注入，会报错。
>
> 原理：
>
> ​	通过反射调用Student对象的set方法进行注入。

**7、在对象中注入非基本类型的对象：**

​        在Student对象中增加如下字段并给set、get方法，并重新生成toString方法。

```java
    private Date birth;

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", gender='" + gender + '\'' +
                ", age=" + age +
                ", birth=" + birth +
                '}';
    }
```

**8、在springContext.xml文件中进行如下更改：**

​        详情见注释：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--通过bean标签，将student类交给spring容器管理-->
    <bean id="student" class="org.example.spring.bean.Student">
        <!--在bean标签中使用property标签对bean的属性进行注入-->
        <property name="age" value="20"/>
        <!--property的value属性会自动转换基本类型，如int和string等-->
        <property name="name" value="张三"/>
        <property name="gender" value="男"/>
        <!--此处不再使用property标签的value属性，而是用ref来引用下面注入的Date类型，值为注入了Date的bean标签的id-->
        <property name="birth" ref="date"/>
    </bean>
    <!--将非基本类型的对象纳入到spring容器中-->
    <bean id="date" class="java.util.Date"/>

</beans>
```

或者

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--通过bean标签，将student类交给spring容器管理-->
    <bean id="student" class="org.example.spring.bean.Student">
        <!--在bean标签中使用property标签对bean的属性进行注入-->
        <property name="age" value="20"/>
        <!--property的value属性会自动转换基本类型，如int和string等-->
        <property name="name" value="张三"/>
        <property name="gender" value="男"/>
        <!--此处不再使用property标签的value属性，而是用ref来引用下面注入的Date类型，值为注入了Date的bean标签的id-->
        <property name="birth">
            <!--直接放在property标签内-->
            <bean class="java.util.Date"/>
        </property>
    </bean>

</beans>
```

**9、自定义对象赋值与Date类赋值方式类似， 不再赘述，如下：**

​        - 定义Clazz对象：

![img](https://img-blog.csdnimg.cn/e855c822b0fc4efdb098b6ce5d58098a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_20,color_FFFFFF,t_70,g_se,x_16)

​        - Student持有Clazz对象： 

![img](https://img-blog.csdnimg.cn/b0f3a1c7bbf54ea6932cea9c87a76162.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_20,color_FFFFFF,t_70,g_se,x_16)

​         - 修改xml文件（可提前赋值）：

![img](https://img-blog.csdnimg.cn/521ffbc937914fd7a7fd568d5c049645.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_20,color_FFFFFF,t_70,g_se,x_16)

​         - 输出：

![img](https://img-blog.csdnimg.cn/187de1f860534bd3adc6654b1c6284a4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_20,color_FFFFFF,t_70,g_se,x_16)

**10、对象注入集合类属性：**

​        - 实体类增加字段，set、get、toString等；

![img](https://img-blog.csdnimg.cn/df730db8312c48e9ba1a40d65c85e242.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_13,color_FFFFFF,t_70,g_se,x_16)

​        - 修改xml文件如下：

```xml
<!--通过bean标签，将student类交给spring容器管理-->
    <bean id="student" class="org.example.spring.bean.Student">
        <!--在bean标签中使用property标签对bean的属性进行注入-->
        <property name="age" value="20"/>
        <!--property的value属性会自动转换基本类型，如int和string等-->
        <property name="name" value="张三"/>
        <property name="gender" value="男"/>
        <!--此处不再使用property标签的value属性，而是用ref来引用下面注入的Date类型，值为注入了Date的bean标签的id-->
        <property name="birth" ref="date"/>
        <property name="clazz" ref="clazz"/>
        <property name="hobbies">
            <list>
                <value>唱</value>
                <value>跳</value>
                <value>rap</value>
                <value>篮球</value>
            </list>
        </property>
    </bean>
    <!--将非基本类型的对象纳入到spring容器中-->
    <bean id="date" class="java.util.Date"/>

    <bean id="clazz" class="org.example.spring.bean.Clazz">
        <property name="no" value="1"/>
    </bean>
```

**11、如果集合中不是存放的String等基本类型属性，那么xml文件如下：**

​        - 新建了Hobby类，Student持有List<Hobby>属性：

```xml
        <property name="hobbies">
            <list>
                <bean class="org.example.spring.bean.Hobby"/>
                <bean class="org.example.spring.bean.Hobby"/>
                <bean class="org.example.spring.bean.Hobby">
                    <!--可在此处设置该list中单个对象的属性-->
                </bean>
            </list>
        </property>
```

或者

```xml
        <property name="hobbies">
            <list>
                <!--引用已经纳入到spring中的bean-->
                <ref bean="hobby"/>
                <ref bean="hobby"/>
                <ref bean="hobby"/>
            </list>
        </property>
```

 **12、Map类属性注入：**

​        - 在Student中持有Map<String, Hobby>属性；

​        - xml中写法较多，可自行选择合适的写法，不完全展示如下：

```xml
        <property name="hobbyMap">
            <map>
                <entry key="h1">
                    <bean class="org.example.spring.bean.Hobby"/>
                </entry>
                <entry>
                    <key>
                        <value>h2</value>
                    </key>
                    <ref bean="hobby"/>
                </entry>
            </map>
        </property>
```

​        - 输出：

![img](https://img-blog.csdnimg.cn/9d8666385e594385a6c9611e3dac1a18.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_20,color_FFFFFF,t_70,g_se,x_16)



### （二）通过构造方法注入

通过（一）后，后面不再详细描述细节问题。

**1、新建Book类，并添加全参构造方法。**

```java
package org.example.spring.bean;

/**
 *
 *
 * @Author: John Smith
 * @Date: 2022/4/19
 */
public class Book {

    private String no;

    private Integer price;

    private Hobby hobby;

    public Book(String no, Integer price, Hobby hobby) {
        this.no = no;
        this.price = price;
        this.hobby = hobby;
    }

    @Override
    public String toString() {
        return "Book{" +
                "no='" + no + '\'' +
                ", price=" + price +
                ", hobby=" + hobby +
                '}';
    }
}

```

 **2、xml文件中，通过bean标签的constructor-args标签，完成属性注入，index属性需要按照构造方法入参的顺序从0开始。**

​        - 自定义对象的注入，可以去掉ref，将constructor-args变双标签，在里面加入bean标签，而后注入其属性。

```xml
    <bean id="book" class="org.example.spring.bean.Book">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="50"/>
        <constructor-arg index="2" ref="hobby"/>
    </bean>
```

**3、输出**

![img](https://img-blog.csdnimg.cn/6a9f192a2c154af698c654040177ccc9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASm9oblNtaXRoOTQ3,size_19,color_FFFFFF,t_70,g_se,x_16)



### （三） Bean的作用域

​        在获取bean的时候如果用contex.getBean的方式来获取对象，那么获取的对象均为同一个（默认单例模式），同时，spring容器在初始化的时候便会创建对象，即为饿汉模式（默认）。

​        那么在xml中的bean标签内，可以用scope属性来定义，每次获取对象的时候是单例模式还是多例模式。

​        - 单例模式（默认）：singleton，每次获取对象实例均为同一个，可在bean标签内设置lazy-init = true属性来将饿汉模式变为懒汉模式，即用到该对象的时候才创建。

​        - 多例模式：prototype，每次创建的实例不同，从而lazy-init变得没意义了，因为每次创建的对象都不同。

```xml
    <!--设置创建实例模式为多例模式-->
    <bean id="book" class="org.example.spring.bean.Book" scope="prototype">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="50"/>
        <constructor-arg index="2" ref="hobby"/>
    </bean>

    <!--设置加载模式为懒汉模式-->
    <bean id="book" class="org.example.spring.bean.Book" lazy-init="true">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="50"/>
        <constructor-arg index="2" ref="hobby"/>
    </bean>
```



### （四）Bean的生命周期方法

​        在bean标签中的init-method属性可以指定一个方法作为该bean的初始化方法。该方法会在构造方法执行后执行。

​        也可以再bean标签内设置destroy-method属性，指定一个方法作为bean的销毁方法，该方法会在bean销毁前执行。

```xml
    <!--这里再Book类里新增了start和end两个方法，分别指定为初始化方法和销毁方法-->
    <bean id="book" class="org.example.spring.bean.Book" init-method="start" destroy-method="end">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="50"/>
        <constructor-arg index="2" ref="hobby"/>
    </bean>
```



### （五）自动装配

​        在bean标签中可以通过autowire属性，指定在bean实例化时通过何种方式来找到相应的实例来赋值到当前对象中，有以下两种方式：

​        - byName：通过名称，假如下面的Book类中有一个属性为Hobby，那么在bean标签中定义autowire=byName后，相当于开启根据名称自动装配，在spring初始化时，会在容器中查找是否有名称相同的bean，如果有则注入到Book类的Hobby类中，使其实例化。（注意：开启自动装配，必须要提供对应属性的set方法，否则无法进行注入），如果名称对应的bean的类型和Book中的属性不是同一个类，无法注入，会报错。

​        - byType：通过类型注入，综上所说，现在不根据名称在spring容器内查找了，会根据对应类型来查找并注入，但如果spring容器内同时存在两个相同类型的bean，会报错（因为它不知道要注入哪一个）。

```xml
    <bean id="hobby" class="org.example.spring.bean.Hobby"/>

    <bean id="book" class="org.example.spring.bean.Book" autowire="byName">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="50"/>
        <constructor-arg index="2" ref="hobby"/>
    </bean>
```



## 四、spring IOC 注解配置

### （一）定义xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <!--开启注解配置-->
    <context:annotation-config></context:annotation-config>
    <!--指定扫bean的范围-->
    <context:component-scan base-package="org.example.spring.bean"/>

</beans>
```

注意：开启ioc注解配置需要在xml的头中增加几个context相关的schema，否则无法开启。



### （二）注解

**@Component：**

​        将类交给spring管理，可在注解后指定bean的名称，不指定名称默认类名首字母小写。

```java
@Component(value = "book")
```

**@Controller、@Service、@Repository：**

​        以上三个的作用与Component的作用无异，区别在于语义上，好区分层级。这里不过多解释，Component则是声明除了这三个以外的类。

**@Scope：**

​	加在类上，声明类是单例还是多例模式。在注解后括号中指定，值为singleton和prototype。

```java
@Scope(value = "prototype")
```

**@Lazy：**

​        开启懒汉模式，只用于单例模式，在注解后的括号中指定true或false。

```java
@Lazy(value = true)
```

**@PostConstruct：**

​        加在方法上，指定类的初始化方法。

```java
    @PostConstruct
    public void start(){
        // 在构造方法执行后执行
    }
```

**@PreDestroy：**

​        加在方法上，指定类的销毁方法。

```java
    @PreDestroy
    public void end(){
        // 在类销毁前执行
    }
```

**@AutoWired：**

​        加在类的属性上——自动装配，注入属性。默认根据类型（byType）注入。

​        可在后面加上required = false，在匹配不到类型时，允许属性为null，不然无法注入则会npe异常。

​        该注解也可放在set方法上，且必须是set方法。

```java
    @Autowired(required = false)
    private BookService service;
```

**@Qualifier("xxx")**

​        需要结合@Autowired一起使用，在后面可以写上bean的名字，指根据名称来注入（byName）。

```java
    @Autowired(required = false)
    @Qualifier("bookService")
    private BookService service;
```

**@Resource**

​        用法和作用同@Autowired。默认byName注入，如果根据名称找不到，则自动byType，如果两个都找不到，则会报错。

```java
    @Resource
    private BookService service;
```



## 五、总结

​        自此spring ioc的基本概念和用法总结完毕，没有过多讲原理概念类的东西。仅针对于用法。肯定还有遗漏的地方，后期发现再补。