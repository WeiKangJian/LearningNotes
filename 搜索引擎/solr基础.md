**2019.12.3**

# solr基础
由于在自己的项目中要使用到搜索引擎，所以开始了此方面的学习。由于项目面向的群体较小，只采用了单机版的solr配置，也没有选择使用可伸缩的ES。只采用Solr完成了搜索引擎的建立，最终实现的搜索效果在我的上线项目中[https://www.bewithu.net/](https://www.bewithu.net/)

***而要更加深入的理解搜索引擎还是要去阅读lucene的源码，后面会去一点一点做这件事。***

## solr中文分词器和字段的配置
好像solr的使用界面建立在一个web服务上，类似tomcat的形式，在官网上下载好solr的安装包之后，在 bin目录下，运行`slor start`（windows环境）即可。默认solr运行的端口是8393。即可出现以下界面。创建自己的solr项目使用`solr.cmd create -c projectName`即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121114049789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
但是solr默认是不支持中文分词的，需要自己在solr的配置文件中配置分词的依赖相关包，在solr目录的项目目录下的managed-schema中配置以下字段来指定分词器。需要注意的是，为了能使得jar包被搜索的到，需要把ik的jar包放入`solr\server\solr-webapp\webapp\WEB-INF\lib`这个目录下

```
<schema name="default-config" version="1.6">

<fieldType name="text_ik" class="solr.TextField">  
        <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>  
</fieldType>  
```
同时还要配置搜索的字段，并在type中指定使用ik的中文分词器，这两个字段也是后面我们需要查询的字段。

```
<field name="question_title"  type="text_ik" indexed="true"  stored="true"  multiValued="true" />
<field name="question_content"  type="text_ik" indexed="true"  stored="true"  multiValued="true" />
```
## solr多字段索引
在搜索引擎真正的使用当中，肯定是不能只针对某一个字段进行索引的，为了实现全文索引的效果，需要配置多字段索引：

```
<field name="keyvalue"  type="text_ik" indexed="true"  stored="true"  multiValued="true" />
<copyField source="question_title" dest="keyvalue"/>
<copyField source="question_content" dest="keyvalue"/>
```
新建一个源字段，并将其他字段的最终端导入该字段，即只需搜索该字段就相当于搜索多个字段了。

## solr从数据库导入数据
一般我们都是从项目的数据库中向solr导入数据，来进行查询，在数据库更新的时候还要注意更新索引的库，将mysql数据库中的数据导入solr的方法如下：
在`solr\server\solr\questioncommunity\conf`中新增加一个db-data-config.xml的文件，在这个文件中配置以下内容：

```
<dataConfig>
    <dataSource 
        type="JdbcDataSource" 
        driver="com.mysql.jdbc.Driver" 
        url="jdbc:mysql://localhost/questioncommunity" 
        user="" 
        password="" />
    <document>
        <entity 
            name="question"
            query="select id, title, content from question">
            <field column="id" name="id"/>
            <field column="title" name="question_title"/>
            <field column="content" name="question_content"/>
        </entity>
    </document>
</dataConfig>
```
其中将字段值和数据库中的值进行了一一对应。还需要在`solrconfig.xml`中配置以下内容：

```
<!-- Solr data import handler -->
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
  <lst name="defaults">
    <str name="config">db-data-config.xml</str>
  </lst>
</requestHandler>
```
注意为了使得相关的mysql包和 importdata包能被找到，还需要复制solr-7.3.1\dist下的solr-dataimporthandler-7.3.1.jar和solr-dataimporthandler-extras-7.3.1.jar至`solr\server\solr-webapp\webapp\WEB-INF\lib`目录下，并在这个目录下添加一个mysql的连接包。

## Solr高亮查询
在我们真正的搜索中往往要把结果设置成高亮的形式，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121121345196.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
需要设置solr的高亮显示，即在搜索结果字段前后加入前后缀，在图形界面中选中如下字段：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121121741406.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
在高亮的结果中即可查看效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121121822335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
在封装solr服务的solrj中，都是对这些功能的调用和展现，在后面的笔记中会介绍solrj的使用。
