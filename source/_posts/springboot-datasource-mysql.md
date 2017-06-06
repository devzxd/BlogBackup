title: Spring Boot多数据源配置（一）durid、mysql、jpa整合
author: 赵旭东
tags:
  - Spring Boot
  - mysql
categories:
  - 技术
date: 2017-06-06 19:21:00
---
目前在做一个统计项目。需要多数据源整合，其中包括mysql和mongo。本节先讲mysql、durid、jpa与spring-boot的整合。

<!--more-->
## 引入Durid包
```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.0.29</version>
</dependency>
```
## 配置文件
```yaml
spring:
  #mysql配置
  datasource:
    user:
      url: jdbc:mysql://192.168.1.252/kxlist_user?characterEncoding=utf-8&useSSL=false
      username: root
      password: 123456
      driver-class-name: com.mysql.jdbc.Driver
    product:
      url: jdbc:mysql://192.168.1.252/kxlist_product?characterEncoding=utf-8&useSSL=false
      username: root
      password: 123456
      driver-class-name: com.mysql.jdbc.Driver
  #jpa配置
  jpa:
    database: mysql
    database-platform: org.hibernate.dialect.MySQL5Dialect
    show-sql: true
    hibernate:
      ddl-auto: update
```
## JAVA文件
- 总的配置：
通过`@Primary`表示主数据源。
```java
@Configuration
public class DruidDataSourceConfig {
    @Bean(name="userDataSource")
    @Primary
    @ConfigurationProperties(prefix="spring.datasource.user")
    public DataSource primaryDataSource() {
        System.out.println("-------------------- userDataSource init ---------------------");
        return new DruidDataSource();
    }

    @Bean(name="productDataSource")
    @ConfigurationProperties(prefix="spring.datasource.product")
    public DataSource secondaryDataSource() {
        System.out.println("-------------------- productDataSource init ---------------------");
        return new DruidDataSource();
    }
}
```
- user数据源的配置：

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="userEntityManagerFactory",
        transactionManagerRef="userTransactionManager",
        basePackages= { "com.kxlist.statistics.domain.user" }) //设置Repository所在位置
public class UserDataSourceConfig {

    @Autowired
    private JpaProperties jpaProperties;


    @Autowired
    @Qualifier("userDataSource")
    private DataSource userDataSource;
    /**
     * 我们通过LocalContainerEntityManagerFactoryBean来获取EntityManagerFactory实例
     * @return
     */
    @Bean(name = "userEntityManagerFactoryBean")
	//@Primary
    public LocalContainerEntityManagerFactoryBean userEntityManagerFactoryBean(EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(userDataSource)
                .properties(getVendorProperties(userDataSource))
                .packages("com.kxlist.statistics.domain.user") //设置实体类所在位置
                .persistenceUnit("userPersistenceUnit")
                .build();
        //.getObject();//不要在这里直接获取EntityManagerFactory
    }

    private Map<String, String> getVendorProperties(DataSource dataSource) {
        return jpaProperties.getHibernateProperties(dataSource);
    }
    /**
     * EntityManagerFactory类似于Hibernate的SessionFactory,mybatis的SqlSessionFactory
     * 总之,在执行操作之前,我们总要获取一个EntityManager,这就类似于Hibernate的Session,
     * mybatis的sqlSession.
     * @param builder
     * @return
     */
    @Bean(name = "userEntityManagerFactory")
    @Primary
    public EntityManagerFactory userEntityManagerFactory(EntityManagerFactoryBuilder builder) {
        return this.userEntityManagerFactoryBean(builder).getObject();
    }

    /**
     * 配置事物管理器
     * @return
     */
    @Bean(name = "userTransactionManager")
    @Primary
    public PlatformTransactionManager writeTransactionManager(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(userEntityManagerFactory(builder));
    }
}
```

*注意：`LocalContainerEntityManagerFactoryBean`和`userEntityManagerFactory`方法其中一个注解`@Primary`即可，不然启动会报错。*

* product数据源的配置

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="productEntityManagerFactory",
        transactionManagerRef="productTransactionManager",
        basePackages= { "com.kxlist.statistics.domain.product" }) //设置Repository所在位置
public class ProductDataSourceConfig {

    @Autowired
    private JpaProperties jpaProperties;


    @Autowired
    @Qualifier("productDataSource")
    private DataSource productDataSource;
    /**
     * 我们通过LocalContainerEntityManagerFactoryBean来获取EntityManagerFactory实例
     * @return
     */
    @Bean(name = "productEntityManagerFactoryBean")
    public LocalContainerEntityManagerFactoryBean productEntityManagerFactoryBean(EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(productDataSource)
                .properties(getVendorProperties(productDataSource))
                .packages("com.kxlist.statistics.domain.product") //设置实体类所在位置
                .persistenceUnit("productPersistenceUnit")
                .build();
        //.getObject();//不要在这里直接获取EntityManagerFactory
    }

    private Map<String, String> getVendorProperties(DataSource dataSource) {
        return jpaProperties.getHibernateProperties(dataSource);
    }
    /**
     * EntityManagerFactory类似于Hibernate的SessionFactory,mybatis的SqlSessionFactory
     * 总之,在执行操作之前,我们总要获取一个EntityManager,这就类似于Hibernate的Session,
     * mybatis的sqlSession.
     * @param builder
     * @return
     */
    @Bean(name = "productEntityManagerFactory")
    public EntityManagerFactory productEntityManagerFactory(EntityManagerFactoryBuilder builder) {
        return this.productEntityManagerFactoryBean(builder).getObject();
    }

    /**
     * 配置事物管理器
     * @return
     */
    @Bean(name = "productTransactionManager")
    public PlatformTransactionManager writeTransactionManager(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(productEntityManagerFactory(builder));
    }
}    
```
依照代码在相对应的包下建实体类和Repository即可。

![目录结构图](http://ooqkdlcps.bkt.clouddn.com/blog/20170606/194515474.png?imageslim)

至此，spring-boot与mysql多数据源的整合已经结束。

### 参考文章
https://my.oschina.net/lengchuan/blog/882391