## SpringBoot

**约定优于配置**

小到配置文件，中间件的默认配置，大到内置容器、生态中的各种 Starters 无不遵循此设计规则

**Starter 机制**

提供自动配置模块及其它有用的依赖。也就意味着当我们项目中引入某个 Starter ，即拥有了此软件的默认使用能力，除非我们需要特定的配置，一般情况下我仅需要少量的配置或者不配置即可使用组件对应的功能。

**Actuator 监控**

Spring Boot 提供的对应用系统的自省和监控的集成功能，可以查看应用配置的详细信息，例如自动化配置信息、创建的 Spring beans 以及一些环境属性等



### Spring Boot 核心注解

Spring Boot 最大的特点是无需 XML 配置文件，能自动扫描包路径装载并注入对象，并能做到根据 classpath 下的 jar 包自动配置。

#### @Configuration

> org.springframework.context.annotation.Configuration

作用：代替 applicationContext.xml 配置文件，所有这个配置文件里面能做到的事情都可以通过这个注解所在类来进行注册。

#### @Bean

作用：代替 XML 配置文件里面的 `<bean ...>` 配置。

#### @ImportResource

作用：有些通过类的注册方式配置不了的，可以通过这个注解引入额外的 XML 配置文件，有些老的配置文件无法通过 `@Configuration` 方式配置的非常管用。

#### [@Import](https://www.carryingcoder.com/2018/10/09/SpringBoot-import%E7%9A%84%E4%BD%BF%E7%94%A8/)

作用：通过查看@Import源码可以发现@Import注解只能注解在类上，以及唯一的参数value上可以配置3种类型的值Configuration，ImportSelector，ImportBeanDefinitionRegistrar

1.直接导入配置类（@Configuration 类）

2.依据条件选择配置类（实现 ImportSelector 接口）

​	如果并不确定引入哪个配置类，需要根据@Import注解所标识的类或者另一个注解(通常是注解)里的定义信息选择配置类的话，用这种方式。

3.动态注册Bean（实现 ImportBeanDefinitionRegistrar 接口）

#### @SpringBootConfiguration

作用：是 `@Configuration` 注解的变体，只是用来修饰是 Spring Boot 配置而已，或者可利于 Spring Boot 后续的扩展。

#### @ComponentScan

> org.springframework.context.annotation.ComponentScan

作用：代替配置文件中的 `component-scan` 配置，开启组件扫描，即自动扫描包路径下的 `@Component` 注解进行注册 bean 实例到 context 中。

#### @EnableAutoConfiguration

> org.springframework.boot.autoconfigure.EnableAutoConfiguration

 作用：Spring Boot 诞生时添加的注解，用来提供自动配置。

#### @SpringBootApplication

作用： 注解就包含了以上 3 个主要注解	 

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

    ...
}
```



### Starter 机制

##### 自动配置原理

[SpringBoot中自动配置原理](https://www.javazhiyin.com/32774.html)

[Spring Boot源码分析——自动配置](https://www.i3geek.com/archives/1871)

[自制一个Spring Boot Starter](https://www.codesheep.cn/2019/01/24/springbt-starter/)



### Actuator 监控

[监控应用](http://www.ityouknow.com/springboot/2018/02/06/spring-boot-actuator.html)



### Spring Boot 启动分析

启动命令：java -jar app-0.0.1-SNAPSHOT.jar，结合源码进行启动分析

```java
@SpringBootApplication
public class AppApplication {
    public static void main(String[] args) {
        SpringApplication.run(AppApplication.class, args);
    }
}
```

#### 主要步骤

1.初始化一个 SpringApplication 对象

2.执行该对象的run方法

#### 步骤具体流程

**初始化一个 SpringApplication 对象**

1. *推断应用类型*
2. *<u>加载初始化构造器 ApplicationContextInitializer</u>*
3. *<u>加载应用监听器 ApplicationListener</u>*
4. *设置应用main()方法所在的类*

流程 2、3 主要通过 SpringFactoriesLoader 找到当前 ClassLoader下的所有 **META-INF/spring.factories** 文件中配置的 ApplicationContextInitializer 和 ApplicationListener 两个接口的实现类并实例化。

> ApplicationContextInitializer 作用：应用程序初始化器，做一些初始化工作
>
> ApplicationListener 作用：Spring 框架对 Java 事件监听机制的一种框架实现



**执行该对象的run方法**

1. *Headless 模式设置*
2. *<u>加载 SpringApplicationRunListeners 监听器</u>*
3. *封装 ApplicationArguments 对象*
4. *<u>配置环境模块</u>*
5. *根据环境信息配置要忽略的 bean 信息*
6. *Banner*
7. *<u>创建 ApplicationContext 应用上下文</u>*
8. *加载 SpringBootExceptionReporter*
9. *<u>ApplicationContext 基本属性配置</u>*
10. *<u>更新应用上下文</u>*
11. *查找是否注册有 CommandLineRunner/ApplicationRunner*



流程 2 主要通过 SpringFactoriesLoader 找到当前 ClassLoader下的所有 **META-INF/spring.factories** 文件中配置的 SpringApplicationRunListener 并实例化，组装成  SpringApplicationRunListeners 。

> SpringApplicationRunListener 作用：在 SpringBoot 应用启动的不同时间点发布不同应用事件类型(ApplicationEvent)

流程 4 创建并配置当前应用将要使用的 Environment

> Environment 作用：描述应用程序当前的运行环境，其抽象了两个方面的内容：配置文件 (profile) 和属性 (properties) 

流程 7 根据用户是否明确设置了applicationContextClass 类型以及 SpringApplication 初始化阶段的推断结果，决定该为当前 SpringBoo t应用创建什么类型的 ApplicationContext

> 类型：AnnotationConfigServletWebServerApplicationContext（SpringMVC）、AnnotationConfigReactiveWebServerApplicationContext（Spring WebFlux）、AnnotationConfigApplicationContext（默认）

流程 9 遍历调用 ApplicationContextInitializer initialize(applicationContext) 方法来对已经创建好的ApplicationContext 进行进一步的处理；设置资源加载器，加载各种 beans 到 ApplicationContext 对象中。

流程 10 完成 IoC 容器可用的最后一道工序：插手容器的启动



## 巨人肩膀

[springboot系列文章之启动原理详解](https://juejin.im/post/5b79a6e651882542aa1b2c22)，掘金，2018

[给你一份Spring Boot知识清单](https://www.jianshu.com/p/83693d3d0a65)，简书，2017

[SpringBoot启动流程解析](https://www.cnblogs.com/trgl/p/7353782.html)，博客园，2017

[史上最简单的 SpringCloud 教程 | 终章](https://blog.csdn.net/forezp/article/details/70148833)，CSDN，2017