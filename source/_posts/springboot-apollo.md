title: springboot项目接入阿波罗配置中心客户端指南
author: 赵旭东
tags:
  - Spring Boot
  - apollo
  - ''
categories:
  - 技术
date: 2017-11-17 17:24:00
---
## 1. 在项目中引入依赖
 - 引入客户端依赖。例如：
```
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>0.9.0-SNAPSHOT</version>
</dependency> 
```
 > **注意：要根据实际情况引入，不能盲目引入最新版** 
 
 - 引入spring-cloud依赖(配合实时更新)。例如：
 ```
    <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-context</artifactId>
    	<version>1.1.6.RELEASE</version>
    </dependency>
 ```
 
 <!--more-->


## 2. 新建appid
  - 在resources目录下新建MATE-INF目录
  - 在MATE-INF目录下新建app.properties
  - 在properties填入appid,appid要保持唯一性。例如：
  ```
  app.id=10086
  ```
## 3. 引入配置
- apollo配置
```
@Configuration
@EnableApolloConfig(value = "application", order = 10)
public class AppConfig {
}

```

- 应用配置

> 项目中需要经常变更的内容

```
@ConfigurationProperties(prefix = "redis.cache")
@Component
@RefreshScope
public class SampleRedisConfig {

  private static final Logger logger = LoggerFactory.getLogger(SampleRedisConfig.class);

  private int expireSeconds;
  private String clusterNodes;
  private int commandTimeout;

  private Map<String, String> someMap = Maps.newLinkedHashMap();
  private List<String> someList = Lists.newLinkedList();

  //省略getter、setter
```
 配置中心例子
```
redis.cache.expireSeconds = 100
redis.cache.clusterNodes = 1,2
redis.cache.commandTimeout = 100
redis.cache.someMap.key1 = a
redis.cache.someMap.key2 = b
redis.cache.someMap.key3 = c
redis.cache.someList[0] = 54545
redis.cache.someList[1] = 123456

```
- 实时更新配置
> 加上这个类之后，在需要刷新配置的bean上加@RefreshScope。配置中心修改内容，项目无需重启就会实时更新。

```
@Component
public class SpringBootApolloRefreshConfig {
  private static final Logger logger = LoggerFactory.getLogger(SpringBootApolloRefreshConfig.class);

  @Autowired
  private RefreshScope refreshScope;

  @ApolloConfigChangeListener
  public void onChange(ConfigChangeEvent changeEvent) {
    refreshScope.refreshAll();
  }
```
## 4. 环境配置
启动项目时，需要加上启动参数`-Denv=DEV`。`DEV`是环境参数，根据实际情况修改。或者在磁盘上建立文件夹。请参考第五步

## 5. 建立文件夹
- 在C盘（Windows）或根目录（linux）建立opt文件夹，并给予读写权限
- 在opt文件下建立settings文件夹，在settings文件夹下建立server.properties文件，内容为`env=dev`。`dev`是环境参数，根据实际情况修改。效果和第四步操作相同。同时存在时，以第四步配置为准。
- 在opt文件下建立data文件夹，用于数据缓存

**目前，env支持以下几个值（大小写不敏感）：**

- DEV
- FAT
- UAT
- PRO

## 参考资料
https://github.com/ctripcorp/apollo/wiki/Java%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97