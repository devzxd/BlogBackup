title: Spring Boot多数据源配置（二）MongoDB
author: 赵旭东
tags:
  - Spring Boot
  - mongoDB
categories:
  - 技术
date: 2017-06-06 19:57:00
---
在[Spring Boot多数据源配置（一）durid、mysql、jpa 整合](/2017/06/06/springboot-datasource-mysql.html)中已经讲过了Spring Boot如何配置mysql多数据源。本篇文章讲一下Spring Boot如何配置mongoDB多数据源。

<!--more-->
## 配置文件

```yaml
spring:
#mongo配置
  data:
    mongodb:
      statis:
        database: kxlist_statis
        uri: 192.168.1.115:27017
      list:
        database: kxlist_list
        uri: 192.168.1.115:27017

```
## JAVA文件：
* 总的配置：

```java
@Configuration
public class MultipleMongoProperties {
    @Bean(name="statisMongoProperties")
    @Primary
    @ConfigurationProperties(prefix="spring.data.mongodb.statis")
    public MongoProperties statisMongoProperties() {
        System.out.println("-------------------- statisMongoProperties init ---------------------");
        return new MongoProperties();
    }

    @Bean(name="listMongoProperties")
    @ConfigurationProperties(prefix="spring.data.mongodb.list")
    public MongoProperties listMongoProperties() {
        System.out.println("-------------------- listMongoProperties init ---------------------");
        return new MongoProperties();
    }

}
```
* statis数据源配置

```java
@Configuration
@EnableMongoRepositories(basePackages = "com.kxlist.statistics.domain", mongoTemplateRef = "statisMongo")
public class StatisMongoMongoTemplate {
    @Autowired
    @Qualifier("statisMongoProperties")
    private MongoProperties mongoProperties;

    @Primary
    @Bean(name = "statisMongo")
    public MongoTemplate statisMongoTemplate() throws Exception {
        return new MongoTemplate(statisFactory(this.mongoProperties));
    }

    @Bean
    @Primary
    public MongoDbFactory statisFactory(MongoProperties mongoProperties) throws Exception {

        ServerAddress serverAdress = new ServerAddress(mongoProperties.getUri());

        return new SimpleMongoDbFactory(new MongoClient(serverAdress), mongoProperties.getDatabase());

    }
}
```
* list数据源配置
和statis的配置很相似
```java
@Configuration
@EnableMongoRepositories(basePackages = "com.kxlist.statistics.domain.list", mongoTemplateRef = "listMongo")
public class ListMongoTemplate {
    @Autowired
    @Qualifier("listMongoProperties")
    private MongoProperties mongoProperties;

    @Bean(name = "listMongo")
    public MongoTemplate listTemplate() throws Exception {
        return new MongoTemplate(listFactory(this.mongoProperties));
    }

    @Bean
    public MongoDbFactory listFactory(@SuppressWarnings("SpringJavaAutowiringInspection") MongoProperties mongoProperties) throws Exception {

        ServerAddress serverAdress = new ServerAddress(mongoProperties.getUri());

        return new SimpleMongoDbFactory(new MongoClient(serverAdress), mongoProperties.getDatabase());

    }
}
```
在相应的包下建立实体类和Repository就可以了。
