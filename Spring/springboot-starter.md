## SpringBoot Starter 机制

> 当开发者完成依赖添加（即 pom 文件的依赖添加），功能会自动创建和注入到上下文，不需要编写麻烦的配置，只需要提供参数属性。

#### 关键概念

1. 业务类（待检测类）SqlSessionFactory.class, SqlSessionFactoryBean.class：传统方式时，通常使用 Maven 引入依赖，或 Jar 包等方式引入类。
2. 自动配置类 MybatisAutoConfiguration：判断业务类是否存在，若存在对业务类进行初始化、装载，比如完成读取参数、初始化等操作。
3. 扫描加载自动配置类 AutoConfigurationImportSelector：调用自动配置类。

### SpringBoot 自动配置原理分析

#### 主要步骤

> 自动配置时机：在 SpringApplication 执行 run 函数中 **流程10：初始化上下文** 。见 [SpringBoot 启动分析](springboot-starter.md) 
>
> 1. 主函数通过 @EnableAutoConfiguration 注解，利用其内 @Import 方法导入
> 2. 利用 AutoConfigurationImportSelector 类完成 spring.factories 文件的扫描，从而加载配置。

#### 步骤具体流程

**业务类：**

> 自动配置的触发点，被检测的核心业务类，若该类存在才会进行自动配置；
>

**自动配置类**：


从 application.properties 中读取参数；

> 在属性参数文 件application.properties 中，读取符合定义参数，如：
>
> mybatis.type-aliases-package=com.fanshulao.iot.app.bean
> mybatis.configuration.mapUnderscoreToCamelCase=true

完成对业务类 Bean 的初始化、装载；

> 1. 首先利用 @Conditional 等注解判断业务类是否存在。若存在则继续。
> 2. @EnableConfigurationProperties 注解来加载配置参数对象
> 3. 编写Resolver方法，利用参数初始化装载业务 Bean

配置 spring.factories

>  spring.factories 添加上刚刚定义的自动配置类。用于运行时扫描自动加载类时使用，否则将无法加载该配置





## 巨人肩膀

[Spring Boot源码分析——自动配置](https://www.i3geek.com/archives/1871)，爱上极客，2018

[自制一个Spring Boot Starter](https://www.codesheep.cn/2019/01/24/springbt-starter/)，程序羊博客，2018

