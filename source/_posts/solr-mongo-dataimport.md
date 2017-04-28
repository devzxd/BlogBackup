title: 浅谈solr-mongo-dataimport
author: 赵旭东
tags:
  - solr
  - mongoDB
categories:
  - 技术
date: 2017-04-24 18:17:00
---
最近项目需要一个全局搜索功能。我们的项目绝大部分数据都是存在mongodb中。搜索引擎准备用solr。
之前因为项目需要，mongodb与solr已经做过一次简单的整合。之所以说简单，是因为只是单表建立索引。用mongo-connector可以很好的将这两者整合。现在这个全局搜索的功能，需要的是多表联合。mongo-connector并不支持此功能。忽然想到mysql与solr整合，建立索引用的是配置文件的方法。可以进行多表联合查询，mongo和solr能用类似的方法吗？于是就进行各种搜索。百度很久，网上一大片都是mongo-connector的教程。GitHub上有SolrMongoImporter的教程，国内也很少见。所以决定写此篇博客与大家分享经验。如有不足之处请多多指教，后面会贴出GitHub地址。

<!--more-->
和mysql-solr整合一样，需要进行一些配置。
本人用的是solr5.5.0，mongo2.4.3。所以以下配置都是基于这两个版本。
## 配置
### jar包管理
- **solr自带jar包**
将solr-5.5.0/dist下的solr-dataimporthandler-5.5.0.jar复制到solr-5.5.0/server/solr-webapp/webapp/WEB-INF/lib下面
- **mongo驱动包(后面提供下载地址)**
将驱动包放入solr-5.5.0/server/lib下面
- **solr-mongo-import.jar（后面提供下载地址）**
将此jar包放到solr-5.5.0/server/solr-webapp/webapp/WEB-INF/lib下面
### xml配置文件
- **修改solrconfig.xml**
在对应core目录中conf文件夹下的solrconfig.xml中加入以下代码

```xml
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
   <lst name="defaults">
      <str name="config">data-config.xml</str>
   </lst>
</requestHandler>

```
- **data-config.xml配置**
在conf文件夹下建立data-config.xml文件，配置如下：
```xml
<?xml version="1.0"?>
<dataConfig>
<!--username:用户名（没有可以不写），password:密码-->
   <dataSource name="kxlist_list" type="MongoDataSource" database="kxlist_list" host="192.168.1.253" port="27017"/>
   <document name="data">
   <!-- if query="" then it imports everything -->
     <entity name="share"
              processor="MongoEntityProcessor"
              query="{'status':'1'}"
              collection="shareResources"
              datasource="kxlist_list"
              transformer="MongoMapperTransformer">
              <field column="id"  name="id" mongoField="_id" />
              <field column="resourceType" name="resourceType" mongoField="resourceType"/>
              <field column="resourceId" name="resourceId" mongoField="resourceId"/>
              <field column="resourceName" name="resourceName" mongoField="resourceName" />
              <field column="shareUserId" name="userId" mongoField="shareId"/>
              <field column="receiveInfo" name="receiveInfo" mongoField="receiveInfo"/>
              <field column="shareType" name="shareType" mongoFiled="shareType"/>
             <!--类似关系型数据库的联合查询-->
            <entity name="commodityList"
                    processor="MongoEntityProcessor"
                    query="{'_id':'${share.resourceId}'}"
                    collection="commoditylist"
                    datasource="kxlist_list"
                    transformer="MongoMapperTransformer">
                    <field column="commodities" name="commodities" mongoField="commodities"/>
                    <field column="listNum" name="listNum" mongoField="num"/>
              </entity>

             <entity name="quotelist"
                    processor="MongoEntityProcessor"
                    query="{'_id':'${share.resourceId}'}"
                    collection="quotelist"
                    datasource="kxlist_list"
                    transformer="MongoMapperTransformer">
                    <field column="commodities" name="commodities" mongoField="commodities"/>
                    <field column="listNum" name="listNum" mongoField="num"/>
              </entity>

              <entity name="demandlist"
                    processor="MongoEntityProcessor"
                    query="{'_id':'${share.resourceId}'}"
                    collection="demandlist"
                    datasource="kxlist_list"
                    transformer="MongoMapperTransformer">
                    <field column="commodities" name="commodities" mongoField="commodities"/>
                    <field column="listNum" name="listNum" mongoField="num"/>
              </entity>

         </entity>
   </document>
 </dataConfig>
```
- **参数说明(只介绍个别参数)**

	dataSource:数据源配置，支持副本集。用“”,“”分割。例如： host="192.168.1.253,192.168.1.254" 			                 port="27017,27017"

	field: 'column'相当于别名。'name'需要与managed-schema中field名字对应。'mongoField'mongodb数据库字段名字。

- **managed-schema配置**

 在此只展示field字段配置，里面也有一些动态字段的配置。
 

```xml
    <field name="_version_" type="long" indexed="true" stored="true"/>
    <field name="_root_" type="string" indexed="true" stored="false"/>
    <field name="_text_" type="text_general" indexed="true" stored="false" multiValued="true"/>
    <copyField source="*" dest="_text_"/>
    <field name="id" type="string" indexed="true" stored="true" />
    <field name="_ts" type="long" indexed="true" stored="true" />
    <field name="resourceType" type="string"  indexed="false" stored="true"/>
    <field name="resourceId" type="string" indexed="false" stored="true"/>
    <field name="resourceName" type="string" indexed="true" stored="true"/>
    <field name="userId" type="string" indexed="true" stored="true"/>
     <field name="listNum" type="string" indexed="true" stored="true"/>
    <field name="shareType" type="string" indexed="false" stored="true"/>
        <!-- Dynamic field definitions allow using convention over configuration
       for fields via the specification of patterns to match field names.
       EXAMPLE:  name="*_i" will match any field ending in _i (like myid_i, z_i)
       RESTRICTION: the glob-like pattern in the name attribute must have
       a "*" only at the start or the end.  -->
<!--在数据库中是以数组形式存在，需要配置动态字段-->
  <dynamicField name="commodities*" type="string" indexed="true" stored="true"/>
  <dynamicField name="receiveInfo*" type="string" indexed="true" stored="true"/>

```
至此，配置已经完毕。可以进行导入数据测试。
![建立索引测试](http://img.blog.csdn.net/20161214114153954?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1OTg0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
导入数据完成之后可以查询测试。
![查询数据](http://img.blog.csdn.net/20161214114425893?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1OTg0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
大功告成！！

## 下面提供各种资源地址

 - [mongodb驱动包](http://download.csdn.net/detail/u013259845/9710920)
 - [solr-mongo-importer jar包 ](http://download.csdn.net/detail/u013259845/9710908)
 - [solrMongoImporter源码](http://download.csdn.net/detail/u013259845/9710898)
## GitHub地址
 
 - [鼻祖地址（github上各种SolrMongoImporter代码都是fork它的）](https://github.com/james75/SolrMongoImporter)
 - [本文主要来源地址（源码和jar包都来源于此地址）](https://github.com/jbonch/SolrMongoImporter)
 
*注：本文是从自己的csdn博客迁移过来的*