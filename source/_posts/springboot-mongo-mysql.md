title: 使用IDEA搭建SpringBoot项目且整合mongoDB和mysql
author: 赵旭东
tags:
  - springboot
categories:
  - 技术
date: 2017-04-24 18:17:00
---
SpringBoot项目相对SpringMVC项目有搭建迅速，配置更少的优点。创建springboot项目有很多种方式，本文使用idea创建一个整合mongoDB和mysql数据库的简单的springboot项目。文章末尾附源码地址。

<!--more-->

##搭建步骤：
主要是以截图的方式介绍搭建过程。


-  **进入新建项目界面，按照下图操作**

![这里写图片描述](http://img.blog.csdn.net/20170418151316391?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1OTg0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170418171707429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1OTg0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170418173158123?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1OTg0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170418173403064?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1OTg0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

经过以上步骤，基本项目框架就会搭建起来。因为项目中需要用到阿里的数据库连接池和json工具包，所以在pom文件中手动加入相应的依赖。

-  **完整pom文件**
	

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.zxd</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<druid.version>1.0.29</druid.version>
		<fastjson.version>1.2.30</fastjson.version>
	</properties>

	<dependencies>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>${druid.version}</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>${fastjson.version}</version>
		</dependency>


	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.RC1</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

</project>
```
-  **数据库的基本配置**
项目搭建成功之后，会发现在resources目录下会生成一个"application.properties"文件。这个文件是springboot项目的基本配置文件，可以重命名为	"application.yml"。本文的配置均在yml文件下配置，优点是层次结构清晰。
配置文件内容如下：

```yml
spring:
#数据库配置
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/spring_boot
    username: root
    password: 123456
  # 配置初始化大小、最小、最大
    initialSize: 5
    minIdle: 5
    maxActive: 20
  # 配置获取连接等待超时的时间
    maxWait: 60000
  # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
    timeBetweenEvictionRunsMillis: 60000
  # 配置一个连接在池中最小生存的时间，单位是毫秒
    minEvictableIdleTimeMillis: 30000
    validationQuery: SELECT 'x'
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
   # 打开PSCache，并且指定每个连接上PSCache的大小。如果用Oracle，则把poolPreparedStatements配置为true，mysql可以配置为false。分库分表较多的数据库，建议配置为false。
    poolPreparedStatements: false
    maxPoolPreparedStatementPerConnectionSize: 20
  # 配置监控统计拦截的filters
    filters: stat
#jpa配置
  jpa:
    database: mysql
    show-sql: true
    generate-ddl: true
    hibernate:
      ddl-auto: update
#mongo配置
  data:
    mongodb:
      database: spring_boot
      uri: mongodb://127.0.0.1:27017
```
经过以上配置。就可以启动项目了。
控制台打印出
```
2017-04-18 17:56:23.647  INFO 14748 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-04-18 17:56:23.656  INFO 14748 --- [           main] com.zxd.DemoApplication                  : Started DemoApplication in 13.982 seconds (JVM running for 15.253)
```
类似语句，就说明启动成功！
可以输出一个简单的语句，验证项目是否搭建成功！
```java
package com.zxd.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author zxd
 * @create 2017-03-29 11:02
 **/
@RestController
public class Hello {

    @GetMapping("/")
    public String sayHello(){
        return "hello spring boot";
    }
}

```
重启项目，访问http://127.0.0.1:8080，就会看到“hello spring boot”。
## 数据操作
 在上面的配置中，已经配置好了数据库连接，并且启动成功。接下来就要对数据进行增删改查。
 
 -  **mysql数据库操作**
 通过spring-data-jpa进行增删改查操作。
 
1.新建user类

```java
package com.zxd.bean;

import com.alibaba.fastjson.JSON;
import org.hibernate.annotations.DynamicInsert;
import org.hibernate.annotations.DynamicUpdate;
import org.hibernate.annotations.GenericGenerator;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

/**
 * @author zxd
 * @create 2017-03-31 14:41
 **/
@Entity
@DynamicUpdate
@DynamicInsert
public class User {

    @Id
    @GeneratedValue(generator = "system-uuid")
    @GenericGenerator(name = "system-uuid", strategy = "uuid.hex")
    @Column(name = "id", nullable = false, length = 32, unique = true)
    private String id;

    private int age;

    private String name;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return JSON.toJSONString(this);
    }
}

```

2.新建Repository
	

```java
package com.zxd.repository;

import com.zxd.bean.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

/**
 * @author zxd
 * @create 2017-03-31 15:17
 **/
@Repository
public interface UserRepository extends JpaRepository<User,String> {
    @Modifying
    @Query(value = "update User u set u.age = ?1 where u.id = ?2")
    int modifyAgeById(int age,String id);
}

```
直接继承JpaRepository就可以，泛型中的User就是User实体类，String是User的主键类型。modifyAgeById方法是自定义的修改方法。里面自带的有很多基本的增删改查方法，在接下来的service中可以看到。

3.新建serivice和controller
	
```java
package com.zxd.service;

import com.zxd.bean.User;
import com.zxd.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

/**
 * @author zxd
 * @create 2017-03-31 17:03
 **/
@Transactional
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public User findOne(String id) {
        return userRepository.findOne(id);
    }

    public List<User> findAll() {
        return userRepository.findAll();
    }

    public int modifyAgeById(int age, String id) {
        return userRepository.modifyAgeById(age,id);
    }

    public void delete(String id) {
        userRepository.delete(id);
    }

    public void save(User user) {
        userRepository.save(user);
    }
}

```

```java
package com.zxd.controller;

import com.alibaba.fastjson.JSON;
import com.zxd.bean.User;
import com.zxd.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @author zxd
 * @create 2017-03-31 14:52
 **/
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    /**
     * 保存
     * @param user
     * @return
     */
    @PostMapping
    public String saveUser(User user){
        userService.save(user);
        return user.toString();
    }

    /**
     * 按id查找
     * @param id
     * @return
     */
    @GetMapping("/{id}")
    public String findUser(@PathVariable("id") final String id){
        System.out.println(id);
        User user = userService.findOne(id);
        return user.toString();
    }

    /**
     * 查找所有
     * @return
     */
    @GetMapping()
    public String findUserAll(){
        List<User> userList = userService.findAll();
        return JSON.toJSONString(userList);
    }

    /**
     * 更新年龄
     * @param id
     * @param age
     * @return
     */
    @PutMapping("/{id}")
    public String updateAge(@PathVariable("id") String id,@RequestParam("age") int age){
       return userService.modifyAgeById(age,id)+"";
    }

    /**
     * 删除
     * @param id
     * @return
     */
    @DeleteMapping("/{id}")
    public String delete(@PathVariable("id") String id){
        userService.delete(id);
        return "删除成功！";
    }
}

```


-  **mongoDB数据库操作**
通过spring-data-jpa和mongoTemplate进行增删改查操作。

1.新建Order类

```java
package com.zxd.bean;

import com.alibaba.fastjson.JSON;
import org.springframework.data.mongodb.core.mapping.Document;

import javax.persistence.Id;

/**
 * 订单
 *
 * @author zxd
 * @create 2017-03-31 16:34
 **/
@Document
public class Order {
    @Id
    private String id;

    private String name;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return JSON.toJSONString(this);
    }
}


```
2.新建Repository
	

```java
package com.zxd.repository;

import com.zxd.bean.Order;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

/**
 * @author zxd
 * @create 2017-03-31 16:36
 **/
@Repository
public interface OrderRepository extends MongoRepository<Order,String> {

}

```
这个接口里面没有写任何代码。基本的增删改查功能jpa已经实现了，直接在service调用就行。一些复杂的功能我们可以写一个通用的dao类由mongoTemplate实现。

3.建立公共的dao
	

```java
package com.zxd.dao.impl;

import com.zxd.dao.IPublicDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Repository;

/**
 * @author zxd
 * @create 2017-03-31 18:06
 **/
@Repository
public class PublicDaoImpl<T> implements IPublicDao{
    @Autowired
    private MongoTemplate mongoTemplate;
    @Override
    public void update(Query query, Update update, Class t) {
        mongoTemplate.updateMulti(query,update,t);
    }
}

```
这里只贴出实现类的代码，接口定义自行补充。里面也只有一个简单的修改方法，这里只是起一个抛砖引玉的作用，可自行扩展。

4.新建service和controller
		

```java
package com.zxd.service;

import com.zxd.bean.Order;
import com.zxd.dao.IPublicDao;
import com.zxd.repository.OrderRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author zxd
 * @create 2017-03-31 17:06
 **/
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private IPublicDao<Order> publicDao;
    public void save(Order order) {
        orderRepository.save(order);
    }

    public Order findOne(String id) {
        return orderRepository.findOne(id);
    }

    public List<Order> findAll() {
        return orderRepository.findAll();
    }

    public int updateNameById(String id, String name) {
        Query query = new Query(Criteria.where("_id").is(id));
        Update update = Update.update("name",name);
        publicDao.update(query,update,Order.class);
        return 1;
    }

    public void deleteById(String id) {
        orderRepository.delete(id);
    }
}

```

```java
package com.zxd.controller;

import com.alibaba.fastjson.JSON;
import com.zxd.bean.Order;
import com.zxd.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * @author zxd
 * @create 2017-03-31 16:45
 **/
@RestController
@RequestMapping("/order")
public class OrderController {

    @Autowired
    private OrderService orderService;

    /**
     * 保存
     * @param order
     * @return
     */
    @PostMapping
    public String saveOrder(Order order){
        orderService.save(order);
        return order.toString();
    }

    /**
     * 按照id查找
     * @param id
     * @return
     */
    @GetMapping("/{id}")
    public String findOrder(@PathVariable("id") String id){
        return orderService.findOne(id).toString();
    }

    /**
     * 查找全部
     * @return
     */
    @GetMapping
    public String findAll(){
        return JSON.toJSONString(orderService.findAll());
    }

    /**
     * 修改
     * @param id
     * @param name
     * @return
     */
    @PutMapping("/{id}")
    public String updateName(@PathVariable(value = "id") String id,@RequestParam("name") String name){
        return orderService.updateNameById(id,name)+"";
    }

    /**
     * 删除
     * @param id
     * @return
     */
    @DeleteMapping("/{id}")
    public String delete(@PathVariable("id") String id){
        orderService.deleteById(id);
        return "删除成功！";
    }
}

```
可以用RESTful测试工具进行调用测试。这里不做测试演示。
至此，一个简单的springboot项目搭建完成！
项目源码地址：https://github.com/devzxd/springboot