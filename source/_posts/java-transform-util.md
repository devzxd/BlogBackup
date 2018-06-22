title: Java bean相互转换工具
author: 赵旭东
tags:
  - dozer
categories:
  - 技术
date: 2018-06-22 16:14:00
---
# 引入依赖

```XML
<!-- https://mvnrepository.com/artifact/net.sf.dozer/dozer -->
        <dependency>
            <groupId>net.sf.dozer</groupId>
            <artifactId>dozer</artifactId>
            <version>5.5.1</version>
        </dependency>
```

<!--more-->

# 写一个通用的工具类

```Java

package com.github.pig.common.util;

import org.apache.commons.collections.CollectionUtils;
import org.dozer.DozerBeanMapper;
import org.dozer.Mapper;

import java.util.ArrayList;
import java.util.List;

/**
 * Description: 类转换工具
 * User: zhaoxudong
 * Date: 2018-05-14
 * Time: 14:42
 */
public class BeanUtils {
    private final static Mapper mapper = new DozerBeanMapper();

    /**
     * 单个转换
     * @param sourceObject 源数据
     * @param target 目标类
     * @return
     */
    public static <T> T transformBean(Object sourceObject, Class<T> target){
        if(sourceObject==null){
            return null;
        }
        return mapper.map(sourceObject,target);
    }

    /**
     * 批量转换
     * @param sourceList 源数据
     * @param target 目标类
     * @return
     */
    public static <T> List<T> batchTransformBean(List sourceList,Class<T> target){
        if(CollectionUtils.isEmpty(sourceList)){
            return new ArrayList<>();
        }
        List<T> result = new ArrayList<>();
        sourceList.stream().forEach(o -> result.add(transformBean(o,target)));
        return result;
    }
}



```
# 使用案例

```JAVA

package com.github.pig.client.model.entity;

import com.alibaba.fastjson.JSON;
import com.github.pig.client.PigClientApplication;
import com.github.pig.client.model.dto.ClientDTO;
import com.github.pig.common.util.BeanUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.ArrayList;
import java.util.List;


@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = PigClientApplication.class)
public class ClientTest {
    @Test
    public void test(){
        Client client = new Client();
        client.setClientId(1);
        client.setName("test1");
        System.out.println("单个转换："+JSON.toJSONString(BeanUtils.transformBean(client,ClientDTO.class)));
        List<Client> clients = new ArrayList<>();
        clients.add(client);
        clients.add(null);
        Client client1 = new Client();
        client1.setClientId(2);
        client1.setName("test2");
        clients.add(client1);
        Client client2 = new Client();
        client2.setClientId(3);
        client2.setName("test3");
        clients.add(client2);
        System.out.println("批量转换1："+JSON.toJSONString(BeanUtils.batchTransformBean(clients,ClientDTO.class)));
    }
}

```