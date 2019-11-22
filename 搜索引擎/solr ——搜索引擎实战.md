**2019.11.22**

   * [solr ——搜索引擎实战](#solr-搜索引擎实战)
      * [连接solr服务，java的服务封装solrj](#连接solr服务java的服务封装solrj)
      * [从solr端查询数据](#从solr端查询数据)
      * [向Solr更新索引](#向solr更新索引)
      * [向Solr删除索引](#向solr删除索引)


# solr ——搜索引擎实战
上一篇笔记讲述了solr的简单配置和分词设置以及导入数据库。在我的项目结合solr实现了站内搜索引擎后，写这篇笔记记录自己是怎么将solr整合到自己的项目当中的。

## 连接solr服务，java的服务封装solrj
先看对SolrJ的官方介绍：SolrJ是一个使Java应用程序可以轻松与Solr对话的API,SolrJ隐藏了许多连接到Solr的细节,并允许您的应用程序通过简单的高级方法与Solr进行交互.即可以将在solr本地端口网站上的操作封装在api中供程序调用（实际方式是通过http请求来获取一个返回类型为XML或者JSON的字符串）。

去Maven仓库中搜索solr即可找到很多版本，将其配置在maven依赖中即可。
先配置一个和solr源端口交互的客户端，通过以下方式配置即可

```
private  final  String solr_url = "http://127.0.0.1:8983/solr/questioncommunity";
private HttpSolrClient client = new HttpSolrClient.Builder(solr_url).build();
```


## 从solr端查询数据
先生成一条查询语句，是直接用的	solrQuery来先实例化一个query的对象。包含要查找的一个数据。
 

```
  SolrQuery query =new SolrQuery(keyWord);
```
然后给query配置以下参数：
```
  //返回的数据格式
   query.setRows(count);
  //偏移值，从第几个处开始返回
    query.setStart(offset);
  //设置返回高亮（）
    query.setHighlight(true);
   //设置高亮的前缀值
    query.setHighlightSimplePre(hlpre);
   //设置高亮的后缀
    query.setHighlightSimplePost(hlpost);
   //在高亮的返回结果集中选择显示的的字段
    query.set("hl.fl", QUESTION_TITLE_FIELD + "," + QUESTION_CONTENT_FIELD);
    //设置默认的查询字段值，这个很重要（多值查找就用这个字段代替）
    query.set("df",QUERY_FIELD);
```
执行查询语句，并返回一个response的对象，后续找结果是从这个response对象中去查询

```
 QueryResponse response = client.query(query);
```
之后从response中读取经过高亮处理后的数据

```
response.getHighlighting()
```
这里的返回值是一个Map<String,Map<String,List<>>的对象，第一个String一般是ID，第二个map中的Key是查询的字段值，即之前设置的在高亮返回结果中显示的字段，而list即是返回的结果字符串，一般list.get(0)即得到查询后高亮的字段.
之后根据系统逻辑，将得到的字段插入到对象中，即可开始使用。

    for(Map.Entry<String,Map<String,List<String>>> entry:response.getHighlighting().entrySet()){
        Question question =new Question();
        question.setId(Integer.parseInt(entry.getKey()));
        if(entry.getValue().containsKey(QUESTION_TITLE_FIELD)) {
            List<String> titleList = entry.getValue().get(QUESTION_TITLE_FIELD);
            if (titleList.size() > 0) {
                question.setTitle(titleList.get(0));
            }
        }
        if(entry.getValue().containsKey(QUESTION_CONTENT_FIELD)) {
            List<String> contentList = entry.getValue().get(QUESTION_CONTENT_FIELD);
            if (contentList.size() > 0) {
                question.setContent(contentList.get(0));
            }
        }
        reslist.add(question);
    }
然后即可完成服务层，可向上提供接口调用。

## 向Solr更新索引
在实际的项目中，数据库中的数据是时刻变化的，需要solr中的索引字段也要跟着变化，因此需要在服务端添加和删除的时候对solr的索引也进行实时操作。
向solr更新索引的时候，在solrj中进行如下方法：

		//	构建一个新文档，用来向solr端插入
        SolrInputDocument doc =new SolrInputDocument();
        //设置字段值，需要和在solr配置文件中定义的字段保持一致
        doc.setField("id",id);
        doc.setField("question_content",content);
        doc.setField("question_title",title);
        //添加进索引，执行add方法
        UpdateResponse response = client.add(doc,1000);

## 向Solr删除索引
可直接删除一个列表或一个单独的ID，或者更具一个query去删除，更具方法参数不同。
```
client.deleteById()
```

