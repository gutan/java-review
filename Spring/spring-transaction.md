## Spring 事务

> 1. 跨不同事务 API 的统一的编程模型，无论你使用的是 jdbc、jta、jpa、hibernate。
>
>    Spring 并不直接管理事务，而是提供了多种事务管理器，他们将事务管理委托给 Hibernate 或者 JTA 等持久化机制所提供的相关平台框架的事务来实现。通过事务管理器 PlatformTransactionManager，Spring 为各个平台如JDBC、Hibernate 等提供了对应的事务管理器，但是具体的实现依赖各自平台的事务实现。
>
> 2. 支持声明式事务
> 3. 简单的事务管理 API
> 4. 能与 Spring 的数据访问抽象层完美集成

### Spring 事务基本属性

##### *隔离规则* [参考](../Database/transaction.md)

- ISOLATION_DEFAULT：使用后端数据库默认的隔离级别

- ISOLATION_READ_UNCOMMITTED：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读

- ISOLATION_READ_COMMITTED：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

- ISOLATION_REPEATABLE_READ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生

- ISOLATION_SERIALIZABLE：最高的隔离级别，完全服从 ACID 的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的

##### *回滚规则*

默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚 。但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

##### *传播行为*

- **PROPAGATION_REQUIRED** ：required , 必须。默认值，A如果有事务，B将使用该事务；如果A没有事务，B将创建一个新的事务。
- PROPAGATION_SUPPORTS：supports ，支持。A如果有事务，B将使用该事务；如果A没有事务，B将以非事务执行。

- PROPAGATION_MANDATORY：mandatory ，强制。A如果有事务，B将使用该事务；如果A没有事务，B将抛异常。

- **PROPAGATION_REQUIRES_NEW** ：requires_new，必须新的。如果A有事务，将A的事务挂起，B创建一个新的事务；如果A没有事务，B创建一个新的事务。
- PROPAGATION_NOT_SUPPORTED ：not_supported ,不支持。如果A有事务，将A的事务挂起，B将以非事务执行；如果A没有事务，B将以非事务执行。

- PROPAGATION_NEVER ：never，从不。如果A有事务，B将抛异常；如果A没有事务，B将以非事务执行。

- **PROPAGATION_NESTED** ：nested ，嵌套。A和B底层采用保存点机制，形成嵌套事务。

##### *是否只读*

事务只对后端的数据库进行读操作，数据库可以利用事务的只读特性来进行一些特定的优化。

##### *事务超时*

事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。

### Spring 事务管理

事务管理流程包括：配置事务管理器、事务配置

##### *1. 配置事务管理器*

```xml
 <!-- 配置事务管理器 -->
 <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
 		<property name="dataSource" ref="dataSource"/>
 </bean>
```

##### *2. 事务配置*

- 注解方式

  1. (2种)开启注解方式支持

  ```xml
  <!-- 开启注解方式的事务配置支持-->
  <tx:annotation-driven transaction-manager="txManager"/>
  
  或
  
  //Java 代码注解 @EnableTransactionManagement 开启注解方式支持
  @EnableTransactionManagement
  public class TxMain{
  ...
  }
  ```

  2. 事务管理的类和方法注解 @Transactional

  3. @Transactional 属性配置：事务管理器名、传播行为、隔离级别、超时、只读、x 异常回滚 / y 异常不会滚，参考 Spring 事务基本属性。

     ***rollbackFor 默认情况下是对 RuntimeException 进行回滚***

- 声明式事务配置

  1. 事务配置

     ```xml
     <!-- ********************** 声明式事务配置 begin   ************** -->
         <!-- 配置事务增强的 advice -->
         <tx:advice id="txAdvice" transaction-manager="txManager">
             <tx:attributes>
                 <!-- all methods starting with 'get' are read-only -->
                 <tx:method name="get*" read-only="true" />
                 <!-- other methods use the default transaction settings (see below) -->
                 <tx:method name="*" />
             </tx:attributes>
         </tx:advice>
     
         <!-- 配置事务的 AOP 切面 --> 
         <aop:config>
             <aop:pointcut id="allService" expression="execution(* com.study.leesmall.spring.sample.tx.service.*Service.*(..)))"/>
             <aop:advisor advice-ref="txAdvice" pointcut-ref="allService"/>
         </aop:config>
      <!-- ********************** 声明式事务配置 end ************** -->
     ```

  2. \<tx:method\> 属性配置：方法名、传播行为、隔离级别、超时、只读、x 异常回滚 / y 异常不会滚，参考 Spring 事务基本属性

- 编码式事务管理

  ```java
  @Autowired
  private PlatformTransactionManager txManager;
  
  public User insertUser(User u) {
          // 1、创建事务定义
          TransactionDefinition definition = new DefaultTransactionDefinition();
          // 2、根据定义开启事务
          TransactionStatus status = txManager.getTransaction(definition);
          try {
              this.userDao.insert(u);
              // 3、提交事务
              txManager.commit(status);
              return this.userDao.find(u.getId());
          } catch (Exception e) {
              // 4、异常了，回滚事务
              txManager.rollback(status);
              throw e;
          }
  }
  ```

### Spring 事务原理

todo

### 名词解释

##### *JPA*

> 全称 Java Persistence API，是 Java EE 5 规范中提出的 Java 持久化接口，旨在规范、简化 Java 对象的持久化工作。而 JPA 相关的技术就是 ORM 技术，ORM 是 Object-Relation-Mapping，即对象关系映射技术，是对象持久化的核心。ORM 是对 JDBC 的封装。
>
> 简单理解，Hibernate 是持久化具体实现技术，而 JPA 是持久化的标准协议，Spring Data JPA 是在Hibernate 的基础上更上层的封装实现。
>
> JPA 框架主要包括：Jboss 的 Hibernate EntityManager、Oracle 捐献给 Eclipse 社区的 EclipseLink、Apache 的 OpenJPA 

##### *JTA*

> 即 Java Transaction API，译为 Java 事务 API。
>
> JTA 允许应用程序执行分布式事务处理——在两个或多个网络计算机资源上访问并且更新数据。JDBC 驱动程序的 JTA 支持极大地增强了数据访问能力。
>
> 与 JDBC 事务的区别：JDBC 事务的一个缺点是事务的范围局限于一个数据库连接。一个 JDBC 事务不能跨越多个数据库。也无法在通过 RPC 的方式调用中保证事务。
>
> 而 JTA 提供了跨数据库连接（或其他 JTA 资源）的事务管理能力。JTA 事务管理则由 JTA 容器实现。一个 JTA事务可以有多个参与者（不同的数据库 或 MQ），而一个 JDBC 事务则被限定在一个单一的数据库连接。

## 巨人肩膀

[可能是最漂亮的Spring事务管理详解](https://juejin.im/post/5b00c52ef265da0b95276091)，掘金，2018

[实战Spring事务传播性与隔离性](https://www.jianshu.com/p/249f2cd42692)，简书，2017

[框架源码系列十一：事务管理](https://www.cnblogs.com/leeSmall/p/10306672.html)，博客园，2019

[Spring（八）：事务管理](https://zhuanlan.zhihu.com/p/37108469)，知乎，2018

[Hibernatee 与 JPA 关系](https://www.zhihu.com/question/30691648)，知乎，2018

[Java中的事务——JDBC事务和JTA事务](https://www.hollischuang.com/archives/1658)，技术博客，2-16

