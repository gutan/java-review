

## Mybatis

### 基础部件

> **Configuration**        MyBatis 所有的配置信息都维持在 Configuration 对象之中。
>
> **SqlSession**              作为 MyBatis 工作的主要顶层 API ，表示和数据库交互的会话，完成必要数据库增删改查功能。
>
> **SqlSessionFactory**  是 MyBatis 中最为核心的组件，每个基于 MyBatis 的应用都是以一个SqlSessionFactory 实例为中心的。

### 使用

##### 基于 Mybatis API 接口

1. *注解映射器*

   > **注意：** 在使用 xml 配置文件方式构建 SqlSessionFactory 实例时，既可以配置 xml 映射器，也可以配置注解映射器；但是直接通过 Configuration 对象构建 SqlSessionFactory 实例时，只能配置注解映射器。

   ```java
   public interface UserDAO {
       @Select("select * from user")
       List<User> getAllUsers();
   }
   ```

2. *Configuration 构建 SqlSessionFactory*

```java
public static void main(String[] args) throws IOException {
  // 通过Java API构建SqlSessionFactory
  // 数据源
  PooledDataSource dataSource = new PooledDataSource();
  dataSource.setDriver("com.mysql.cj.jdbc.Driver");
  dataSource.setUrl(
    "jdbc:mysql://127.0.0.1:3306/demo?characterEncoding=utf8&serverTimezone=UTC");
  dataSource.setUsername("root");
  dataSource.setPassword("root");
  // 事务管理器
  TransactionFactory transactionFactory = new JdbcTransactionFactory();
  Environment environment = new Environment("dev", transactionFactory, dataSource);
  Configuration configuration = new Configuration(environment);

  // 注册指定映射器
  configuration.addMapper(UserDAO.class);
  // 注册映射器类所在包名下的所有映射器
  //configuration.addMappers("com.fanshutou.hellp.plus.mybatis.mapper");
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);

  SqlSession sqlSession = sqlSessionFactory.openSession();

  UserDAO userDAO = sqlSession.getMapper(UserDAO.class);
  List<User> list = userDAO.getAllUsers();
  System.out.println("list.size = [" + ((List) list).size() + "]");
}
```

##### 基于 Mybatis xml 配置文件（结合 Spring）构建SqlSessionFactory

1. Mybatis xml 配置文件

   ```xml
   <mappers>
       <!-- xml映射器：将SQL语句写在xml文件中 -->
       <mapper resource="xxx/xx/x.xml" />
   
       <!-- 注解映射器：将SQL语句写在Java代码中, 这里有2种方式:  -->
       <!-- 方式一: 注册每一个映射器接口,需要明确注册每一个接口 -->
       <!-- 方式二: 指定映射器接口所在包,则该包下的所有映射器接口都会被注册 -->
       <!-- <mapper class="TestMapper" /> -->
       <package name="org.chench.test.mybatis.mapper.impl"/>
   </mappers>
   ```

2. *Xml 映射器*

   ```xml
   <mapper namespace="com.fanshutou.hellp.plus.mybatis.dao.UserDao">
     <resultMap id="BaseResultMap" type="com.fanshutou.hellp.plus.mybatis.bean.User">
       <id property="id" column="id"></id>
       <result property="name" column="name"></result>
     </resultMap>
     <select id="getAllUsers" resultMap="BaseResultMap">
       select
         id,
         name
       from user
     </select>
   </mapper>
   ```

3. Spring 和 MyBatis 整合 spring xml 配置文件构建 SqlSessionFactory

```xml
<!-- spring 和 MyBatis 整合，不需要 mybatis 配置映射文件 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
<property name="dataSource" ref="dataSource" />
<!-- 自动扫描 mapping.xml 文件 -->
<property name="mapperLocations" value="classpath:mapping/*.xml"></property>
<!-- mybatis 配置文件-->
<property name="configLocation" value="classpath:mybatis-config.xml"/>
</bean>

<!-- mapper 接口所在包名，Spring 会自动查找其下的类 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
<property name="basePackage" value="com.xx.xxx" />
<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
</bean>

<!-- (事务管理) transaction manager, use JtaTransactionManager for global tx -->
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource" />
</bean>
```

## 巨人肩膀

[深入理解Mybatis](https://blog.csdn.net/u010349169/article/category/9263284)，CSDN，2014

[深入浅出mybatis之启动详解](https://www.cnblogs.com/nuccch/p/8466957.html)，博客园，2018