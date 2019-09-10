### SpringBoot 启动分析

启动命令：java -jar app-0.0.1-SNAPSHOT.jar

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

#### 步骤具体流程(结合源码走流程)

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