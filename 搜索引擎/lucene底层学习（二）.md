**2019.12.6**
# lucene底层学习（二）
上一篇的学习文章简要介绍了lucene的大致内部实现的逻辑结构，在这篇学习笔记中将会从一个具体lucene的代码实例入手，来探究 lucene是怎么一步步实现从分词到建立索引再到进行查询匹配进行选择输出查询结果的过程的。
## 索引的创建
先看一下在index（）这个方法中，创建索引的过程，先是选定了一个分词器，这个分词器是可以后期在配置文件中自由进行配置和选择的。然后将文档进行过分词之后，写入到索引里面去。大致的步骤是分下面的三步：
* 指定分词策略（选分词器Analyzer）;
* 通过添加字段(Field)创建文档(Document)；
* 创建IndexWriter，通过addDocument()方法添加文档(Document)

```
public void index(){
        //使用了标准分词器，可以选择的分词很多
        Analyzer analyzer = new StandardAnalyzer();
        //可以先将索引存储在内存中
        Directory directory = new RAMDirectory();hhIndexWriterConfig config = new IndexWriterConfig(analyzer);
        IndexWriter writer = null;
        try {
            writer = new IndexWriter(directory, config);
            //选择了一些片段
            String[] texts = new String[]{
                    "IThis is a test for 海量数据处理 "
                };

                for (String text : texts) {
                    Document doc = new Document();
                    doc.add(new Field("fieldname", text, TextField.TYPE_STORED));
                    writer.addDocument(doc);
                }
                writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
## 具体类的剖析
### `public class IndexWriter`：
核心组件，通过创建一个新的索引来加入到原有的索引中去。

然后观察其构造器：`public IndexWriter(Directory d, IndexWriterConfig conf)`
第一个参数指定了索引地址，第二个参数就是索引的配置信息了。Lucene使用IndexWriterConfig封装了所有索引时需要的设置的内容，它含有很多内容，定义了许多默认值。具体的配置信息可以通过下面的代码实现：

        Directory directory = FSDirectory.open(Paths.get("路径"));
        IndexWriterConfig config = new IndexWriterConfig();
        writer = new IndexWriter(directory, config);
        System.out.println(writer.getConfig());
在配置好之后，就可以通过下面的方式来完成文档的添加啦

```
public long addDocument(Iterable<? extends IndexableField> doc)

public long addDocuments(Iterable<? extends Iterable<? extends IndexableField>> docs)
```

`public abstract class Directory：`字典类，用来表示索引所在的具体位置

`public abstract class Analyzer：` 具体的分词器类实现是一个接口，经典的模板方法的模式

### `Document 与 Field`
**最重要的两个类**
Document 对应具体的文本文件，而Field则对应相应的文档的属性。在Document这个类里面，通过操作field来完成文档属性的表示。主要有如下两个方法：
`add(Filed field)` 添加一个字段到文档中
`get(String fieldname)` 获取文档中字段的文本
而对全文索引的操作，主要是对这两个对象的表示，来进行完成的。

阅读Document这了类的内部实现，代码如下：
这里面都是往里面添加属性的方法：

```
 /** 为document添加field */
 public final void add(IndexableField field)

/** 删除一个field */
 public final void removeField(String field)

 /** 根据Field名称找出field, 如果多个Field名称一样，返回第一个 */
public final IndexableField getField(String name) 

/** 返回能读懂的document内容 */
public final Sting toString()

/** 把document中的所有field移除*/
public void clear()
```

而对于File这个类的方法如下图所示：（这里面都是其构造器的实现，可以看到其有很多不同的构造）

```
public Field(String name, Reader reader, FieldType type)

public Field(String name, TokenStream tokenStream, FieldType type)

public Field(String name, byte[] value, FieldType type)

public Field(String name, byte[] value, int offset, int length, FieldType type)

public Field(String name, BytesRef bytes, FieldType type)

public Field(String name, String value, FieldType type)
```
## 索引的具体过程
先是下面这个方法：

```
 public long updateDocument(Term term, Iterable<? extends IndexableField> doc){
 long seqNo = docWriter.updateDocument(doc, analyzer, term);
 }
```
继续深入`updateDocument(doc, analyzer, term)`这个方法

```
long updateDocument(final Iterable<? extends IndexableField> doc, final Analyzer analyzer, final Term delTerm)){
    final DocumentsWriterPerThread dwpt = perThread.dwpt;
    seqNo = dwpt.updateDocument(docs, analyzer, delTerm);
}
```
该方法在更新操作时，先删除包含term的doc再添加新的doc。这个操作是原子性的，也就是同一个reader在相同的索引上执行。这里的doc是传入的document，analyzer是在IndexWriterConfig中设置的analyzer，也可以不设置，默认是StandardAnalyzer。

继续深入`updateDocument(docs, analyzer, delTerm)`方法

```
public long updateDocument(Iterable<? extends IndexableField> doc, Analyzer analyzer, Term delTerm){
    docState.doc = doc;
    docState.analyzer = analyzer;
    consumer.processDocument();
}
```
可以看到，其具体的索引构建过程是.`processDocument()`继续深入：

```
private int processField(IndexableField field, long fieldGen, int fieldCount){
String fieldName = field.name();
    IndexableFieldType fieldType = field.fieldType();
    PerField fp = null;
    if (fieldType.indexOptions() == null) {
      throw new NullPointerException("IndexOptions must not be null (field: \"" + field.name() + "\")");
    }
    // Invert indexed fields:
    if (fieldType.indexOptions() != IndexOptions.NONE) {
      // if the field omits norms, the boost cannot be indexed.
      if (fieldType.omitNorms() && field.boost() != 1.0f) {
        throw new UnsupportedOperationException("You cannot set an index-time boost: norms are omitted for field '" + field.name() + "'");
      }
      fp = getOrAddField(fieldName, fieldType, true);
      boolean first = fp.fieldGen != fieldGen;
      fp.invert(field, first);
      if (first) {
        fields[fieldCount++] = fp;
        fp.fieldGen = fieldGen;
      }
    } else {
      verifyUnIndexedFieldType(fieldName, fieldType);
    }
    // Add stored fields:
    if (fieldType.stored()) {
      if (fp == null) {
        fp = getOrAddField(fieldName, fieldType, false);
      }
      if (fieldType.stored()) {
        try {
          storedFieldsWriter.writeField(fp.fieldInfo, field);
        } catch (Throwable th) {
          throw AbortingException.wrap(th);
        }
      }
    }
    DocValuesType dvType = fieldType.docValuesType();
    if (dvType == null) {
      throw new NullPointerException("docValuesType must not be null (field: \"" + fieldName + "\")");
    }
    if (dvType != DocValuesType.NONE) {
      if (fp == null) {
        fp = getOrAddField(fieldName, fieldType, false);
      }
      indexDocValue(fp, dvType, field);
    }
    if (fieldType.pointDimensionCount() != 0) {
      if (fp == null) {
        fp = getOrAddField(fieldName, fieldType, false);
      }
      indexPoint(fp, field);
    }
    return fieldCount;
}
```
这里的IndexableField代表索引时一个的filed。在IndexWriter中，是一个document的内部表示形式。IndexableField是一个接口，它含有几个重要的属性：field-name, field-type, filed-value。
之后是写操作：

```
public void writeField(FieldInfo info, IndexableField field){
    if(long)    bufferedDocs.writeVLong(infoAndBits);
    if(int)     bufferedDocs.writeVInt(bytes.length);
    if(String)  bufferedDocs.writeString(string);
....
}
```
至此为止整个的索引构造就结束啦，之后的笔记还会分析详细的查询过程。
