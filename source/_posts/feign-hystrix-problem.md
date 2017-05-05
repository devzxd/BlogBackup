title: Feign使用Hystrix无效原因及解决方法
author: 赵旭东
tags:
  - Spring Cloud
  - feign
  - hystrix
  - ''
categories:
  - 技术
date: 2017-05-05 14:26:00
---
最近项目重构使用了Spring Boot和Spring Cloud。这两者结合确实给项目带来了方便，同时也遇到了一些问题。其中使用feign作为服务消费，但是断路器hystrix一直不起作用让人很费解。最终经过重重查找终于找到原因，以及解决方法。

<!--more-->

<h2> 问题产生原因</h2>

首先，使用spring-cloud搭建微服务的过程大部分是根据网上的教程来的，由于网上教程的时间较早，而spring-cloud更新迭代较快,会造成依赖上的一些问题。教程中的spring-cloud的依赖是

```xml
<dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-dependencies</artifactId>
	    <version>Brixton.RELEASE</version>
	    <type>pom</type>
	    <scope>import</scope>
	</dependency>
    
```

而我自己使用idea搭建项目使用的是较新的依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-dependencies</artifactId>
	<version>Dalston.RELEASE</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```
发现两者的区别了吗？对！就是依赖版本不同。教程中的版本是 <code> Brixton.RELEASE</code>  而我使用的版本是<code>Dalston.RELEASE</code> 。


<h2>探究过程(此过程不想看的话请直接跳转至[解决方案](#Mark))</h2> 

根据这个关系顺藤摸瓜找到了Netflix的依赖版本

![netflix依赖版本](http://ooqkdlcps.bkt.clouddn.com/blog/20170505/145052760.png?imageslim)

接着，去了[官网](https://spring.io/docs/reference)找到对应的版本，查看文档和API

![mark](http://ooqkdlcps.bkt.clouddn.com/blog/20170505/145346802.png?imageslim)

在文档中会看到
![文档说明](http://ooqkdlcps.bkt.clouddn.com/blog/20170505/145600600.png?imageslim)

这个意思就说feign默认是启用hystrix的，如果要禁用的话需要加配置语句。但是种种迹象表明，feign中并没有有启用hystrix，看到这里当时我就很疑惑，但是发现了hystrix在feign中的开关，还是有所收获的。我抱着试一试的心态照着上面的描述在配置文件中加上了配置，将<code>false</code>改为了<code>true</code>，结果神奇般的起了作用！

虽然问题解决了，为什么官方文档还是有错误的？在这里吐槽一句:TMD（挺萌的）~~~。

抱着追根求源的心态，查看了netflix的源码，看看什么时候修改了默认配置。点击上图中的API就可以看到源github上的源码了。里面这两段代码，就是管理默认配置的。

![HystrixSecurityAutoConfiguration.java](http://ooqkdlcps.bkt.clouddn.com/blog/20170505/151356402.png?imageslim)

![FeignClientsConfiguration.java](http://ooqkdlcps.bkt.clouddn.com/blog/20170505/151244619.png?imageslim)

为什么要默认关闭hystrix呢？请看这里：https://github.com/spring-cloud/spring-cloud-netflix/issues/1277

至此，终于知道了产生错误的原因，以及为什么要默认关闭hystrix。

<h2> 解决方案</h2> 

<div id="Mark"></div>
如果是yml文件，请在文件中加入：

```yaml
feign:
  hystrix:
    enabled: true
```
如果是properties文件，请在文件中加入：

```
feign.hystrix.enabled=true
```
重启服务，大功告成！