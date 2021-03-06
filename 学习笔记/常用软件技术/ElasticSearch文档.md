[TOC]
# Lucene学习笔记

## 一. Lucene简介

`Lucene`是一个封装了全文检索算法的一个快速查找算法类库，专注于企业级搜索、查询、多维统计等功能模块的实现；

## 二. 常见查询、搜索算法

### 2.1 顺序扫描法

> - 算法描述：从头到尾对文件进行查找，直到扫描完整个文件为止；
> - 优势：算法查找准确率高；
> - 缺点：随着数据量的增大，算法效率变低；

### 2.2 倒排索引

> - 算法描述：查询时会先将内容进行提起出来组装成文档，对文档进行切分词，组装成索引目录，索引和文档有关联关系，可通过查询索引，找到索引对应的文档所在目录即可；
>   - 切分词：使用分词器对文档进行分词，将一个文档进行单词切分即可；
> - 优势：切分后的词组成索引目录，实现了重复索引的丢弃，保证索引查询的简洁高效性能；



## 三.全文检索

> `Lucene`是全文检索的一个开源工具引擎，提供了一个全文检索架构，提供了一个完整的查询引擎和索引引擎以及部分文档分析引擎；

### 3.1 Lucene索引流程图

![1595079580417](assets/1595079580417.png)

### 3.2 Lucene文档-Document

#### 3.2.1 创建文档

> - `Lucene`中使用的是Document对象作为文档，通常情况下类比于一条数据库中的数据；
> - `Document`队对象存在着一个唯一的索引id，该ID是`Lucene`自定义实现的，无法进行控制；
> - `Field`是`Document`对象中的属性集合，是一个或者多个属性组装成的集合对象，类比数据库中一条数据里面的多个字段；
> - `Field`中的作用域是属于独立的`Document`对象中的，每一Document对象中的Field可不一致，并且一个Document对象中可以存在多个`Field-key`与`Field-value`;
>
> ![1595082249402](assets/1595082249402.png)

#### 3.2.2 创建索引案例

```java
    @Test
    public void createIndexWriter() throws IOException {
        List<Food> data = Utils.getList();
        //创建文档集合
        List<Document> documents = new ArrayList<Document>();
        data.stream().forEach(food -> {
            Document document = new Document();
            documents.add(document);
            document.add(new IntPoint("id", food.getId()));
            document.add(new TextField("food_name", food.getFood_name(), Field.Store.YES));
            document.add(new TextField("food_desc", food.getFood_desc(), Field.Store.YES));
            document.add(new TextField("food_type", food.getFood_type(), Field.Store.YES));
            document.add(new TextField("food_address", food.getFood_address(), Field.Store.YES));
            document.add(new TextField("food_info", food.getFood_info(), Field.Store.YES));
            document.add(new TextField("food_material", food.getFood_material(), Field.Store.YES));
            document.add(new TextField("food_user_id", food.getFood_user_id(), Field.Store.YES));
            document.add(new TextField("food_user_name", food.getFood_user_name(), Field.Store.YES));
            document.add(new TextField("food_score", String.valueOf(food.getFood_score()), Field.Store.YES));
            document.add(new TextField("food_score", String.valueOf(food.getFood_score() + Math.random()), Field.Store.YES));
        });

        //创建分词器
        IKAnalyzer analyzer = new IKAnalyzer();
        FSDirectory directory = FSDirectory.open(Paths.get("F:\\APP\\Lucene"));
        IndexWriterConfig writerConfig = new IndexWriterConfig(analyzer);
        IndexWriter indexWriter = new IndexWriter(directory, writerConfig);
        indexWriter.addDocuments(documents);
        indexWriter.close();
    }

```

#### 3.2.3 搜索案例

```java
  public void indexSearchTest() throws IOException, ParseException {
        IKAnalyzer analyzer = new IKAnalyzer();
        FSDirectory directory = FSDirectory.open(Paths.get("F:\\APP\\Lucene"));
        IndexReader indexReader = DirectoryReader.open(directory);
        IndexSearcher indexSearcher = new IndexSearcher(indexReader);
        QueryParser queryParser = new QueryParser("food_name", analyzer);
        Query query = queryParser.parse("芝麻");
        TopDocs search = indexSearcher.search(query, 10);
        for (ScoreDoc scoreDoc : search.scoreDocs) {
            int doc = scoreDoc.doc;
            Document document = indexReader.document(doc);
            String str = document.toString();
            System.out.println(str);
        }
    }
```



### 4. Lucene涉及组件

> - `Document`：文档对象，作用于文档的管理；
> - `Field`：属性对象，包含于`Document`对象中，key值与value值均可重复；
>
> | Field类                                        | 数据类型               | Analyzer是否分词 | Indexed是否创建索引 | Store是否存储 | 说明                                                         |
> | ---------------------------------------------- | ---------------------- | ---------------- | ------------------- | ------------- | ------------------------------------------------------------ |
> | `StringField(fieldName, fieldValue,Store.YES)` | String                 | 否               | 是                  | 是/否         | 用于构建一个字符串Field，**整个字符串存储在索引中**，**不进行分词**，通常使用Store来控制是否存储在文档中； |
> | `FloatPoint(fieldName, fieldValue)`            | float                  | 是               | 是                  | 否            | 用于构建一个Float型的Field，**进行分词和构建索引，不存储在文档中** |
> | `DoublePoint(fieldName, fieldValue)`           | double                 | 是               | 是                  | 否            | 用于构建一个double型的Field，**进行分词和构建索引，不存储在文档中** |
> | `IntPoint(fieldName, fieldValue)`              | int                    | 是               | 是                  | 否            | 用于构建一个int型的Field，**进行分词和构建索引，不存储在文档中** |
> | `LongPoint(fieldName, fieldValue)`             | long                   | 是               | 是                  | 否            | 用于构建一个long型的Field，**进行分词和构建索引，不存储在文档中** |
> | `StoreField(fieldName, fieldValue)`            | 重载方法，支持多种类型 | 否               | 否                  | 是            | 用于构建一个不同类型的Field，**不进行分词和构建索引，存储在文档中** |
> | `TextField(fieldName, fieldValue,Store.YES)`   | 字符串 或者 流         | 是               | 是                  | 是/否         | 用于构建一个double型的Field，**进行分词和构建索引，不存储在文档中** |
> | `NumberDocValuesField(fieldName, fieldValue)`  | 数值类型               |                  |                     |               | **配合其他field进行排序使用**                                |
>
> 
>
> - `IndexWriter`：写入索引库的操作对象；
> - `IkAnalyzer`：中文分词器，主要作用于写入的document进行分词、搜索条件分词；
> - `IndexWriterConfig`：配置写入`IndexWriter`对象的信息；
> - `Directory`：文件系统对象，主要是操作Lucene的索引库；
>   - `FSDirectory`：通常使用的文件系统对象实现类；
> - `IndexReader`：文件系统的读取类对象；
> - `IndexSearcher`：文档索引系统的搜索类对象；
> - `QueryParser`：将输入的查询条件封装为Lucene需要的查询条件对象；
> - `Query`：封装的查询条件对象；
> - `TopDocs`：搜索的检索数据集合，包含文档的条数、分数、文档唯一索引等；

### 4.1 使用案例

> - 当使用`DoublePoint`、`IntPoint`等数据结构时，无法将数据保存到`document`中，所以可以使用`StoredField`将数据保存到document中进行存储；
> - 使用范围查询的字段，均需要设置索引的field；

```java
    public void createIndexWriter() throws IOException {
        List<Food> data = Utils.getList();
        //创建文档集合
        List<Document> documents = new ArrayList<Document>();
        data.stream().forEach(food -> {
            Document document = new Document();
            documents.add(document);
            /**
             * 通常情况：
             *      id 关联的是数据库的唯一主键
             *      是否分词： 否，因为ID一般是不需要分词的
             *      是否索引： 是，可根据id查找到对应的document
             *      是否存储： 是，通常情况下，id是需要关联数据库的，所以在document需要存储id
             *  StringField符合上述需求
             */
            document.add(new StringField("id", String.valueOf(food.getId()), Field.Store.YES));

            /**
             * 通常情况：
             *      name是数据document的名称，相当于标题，适用于各个场景
             *      是否分词：是，因为名称是字符串组合而成
             *      是否索引：是，因为名称需要进行索引
             *      是否存储：是，因为需要在document中获取id值
             *  TextField符合上述需求
             */
            document.add(new TextField("food_name", food.getFood_name(), Field.Store.YES));
            document.add(new TextField("food_desc", food.getFood_desc(), Field.Store.YES));
            document.add(new TextField("food_address", food.getFood_address(), Field.Store.YES));
            document.add(new TextField("food_info", food.getFood_info(), Field.Store.YES));
            document.add(new TextField("food_material", food.getFood_material(), Field.Store.YES));
            document.add(new TextField("food_user_name", food.getFood_user_name(), Field.Store.YES));

            /**
             * 是否分词：否
             * 是否索引：是
             * 是否存储：是
             */
            document.add(new StringField("food_user_id", food.getFood_user_id(), Field.Store.YES));
            document.add(new StringField("food_type", food.getFood_type(), Field.Store.YES));


            /**
             * 是否分词：通常情况下，是需要进行分词的    Lucene使用范围查询，是必须进行分词的
             * 是否索引：是，因为会涉及到范围查询
             * 是否存储：是，这个需要在界面进行展示
             * 由于IntPoint无法再文档中进行存储数据，所以需要使用StoreField将分数进行存储
             *
             */
            document.add(new IntPoint("food_score", food.getFood_score()));
            document.add(new StoredField("food_score",food.getFood_score()));

            /**
             * 图片路径是不需要进行查询，只需要保存
             * 是否分词：否
             * 是否索引：否
             * 是否存储：是
             * StoredField符合上诉情况
             */
            document.add(new StoredField("food_iamge", food.getFood_image()));
        });

        //创建分词器
        IKAnalyzer analyzer = new IKAnalyzer();
        FSDirectory directory = FSDirectory.open(Paths.get("F:\\APP\\Lucene"));
        IndexWriterConfig writerConfig = new IndexWriterConfig(analyzer);
        IndexWriter indexWriter = new IndexWriter(directory, writerConfig);
        indexWriter.addDocuments(documents);
        indexWriter.close();
    }
```

### 4.2 索引操作

```java
 @Test
    public void indexUpdate() throws IOException {
        Document document = new Document();
        document.add(new StringField("id", "723", Field.Store.YES));
        document.add(new TextField("food_name", "奶香柠檬茶", Field.Store.YES));
        document.add(new TextField("food_desc", "奶香柠檬茶", Field.Store.YES));
        document.add(new TextField("food_address", "重庆市局", Field.Store.YES));
        IKAnalyzer analyzer = new IKAnalyzer();
        FSDirectory directory = FSDirectory.open(Paths.get("F:\\APP\\Lucene"));
        IndexWriterConfig writerConfig = new IndexWriterConfig(analyzer);
        IndexWriter indexWriter = new IndexWriter(directory, writerConfig);
        indexWriter.updateDocument(new Term("id", "723"), document);
        indexWriter.close();
    }

    /**
     * 索引删除
     */
    @Test
    public void testDeleteIndex() throws IOException {
        IKAnalyzer analyzer = new IKAnalyzer();
        FSDirectory directory = FSDirectory.open(Paths.get("F:\\APP\\Lucene"));
        IndexWriterConfig writerConfig = new IndexWriterConfig(analyzer);
        IndexWriter indexWriter = new IndexWriter(directory, writerConfig);

        indexWriter.deleteDocuments(new Term("id", "723"));
        indexWriter.close();
    }
```

## 5. 分词器

### 5.1 标准分词器-StandardAnalyzer

> `Lcuene`自带的标准分词器，作用于英文分词器分词；针对中文分词是不够友好的，采用的是一个汉字一个词语；

### 5.2 WhitespaceAnalyzer

> `Lucene`中自带的分词器，主要是作用于空格的去除；不支持中文；

## 5.3 SimpleAnalyzer

> `Lucene`自带的分词器，主要是将除了字母以外的所有字符去除，并将所有字母从小写改为大写，并且会将数字进行剔除，不支持中文；

## 5.4 CJKAnalyzer

> 支持中日韩三种语言分词，中文是二分法分词，去掉空格，去掉标点符号，中文支持不友好；

## 6. Query查询

> `Query`查询时，需要创建一个Query查询对象；
>
> - `QueryParser`：主要是用于创建Query对象，封装数据；
>
>   - 该对象中可以使用连接符，直接在多个筛选条件中使用 ` AND `即可；
>     - `AND`：多维度之间使用AND进行关联；
>     - `OR`：多维度之间使用OR进行关联；
>   - 使用方式：
>
>   ```java
>   IKAnalyzer analyzer = new IKAnalyzer();
>   FSDirectory directory = FSDirectory.open(Paths.get("F:\\APP\\Lucene"));
>   IndexReader indexReader = DirectoryReader.open(directory);
>   IndexSearcher indexSearcher = new IndexSearcher(indexReader);
>   QueryParser queryParser = new QueryParser("food_name", analyzer);
>   Query query = queryParser.parse("芝麻 OR 早餐");
>   TopDocs search = indexSearcher.search(query, 10);
>   ```
>
>   



## 7. Lucene索引文件

> `Lucene`创建的索引，是需要存储在索引文件中进行保存的，索引文件主要分为：
>
> | 文件名              | 文件扩展名   | 描述                                                        |
> | ------------------- | ------------ | ----------------------------------------------------------- |
> | Segments File       | segment_N    | 保存一个提交点的信息                                        |
> | Lock File           | write.lock   | 上锁，放置索引文件被多线程操作                              |
> | Segment Info        | .si          | 保存了索引段的元数据信息                                    |
> | Compound File       | .cfs    .cfe | 一个可选的虚拟文件，把所有的索引信息都存入到复合索引文件中  |
> | Fields              | .fnm         | 保存fields的相关信息                                        |
> | Field Index         | .fdx         | 保存指向field data的指针                                    |
> | Field Data          | .fdt         | 文档存储的字段的值                                          |
> | Term Index          | .tip         | 到Term Dictionary的索引                                     |
> | Term Dictionary     | .tim         | term 词典，存储term信息                                     |
> | Frequncies          | .doc         | 由包含每个term以及频率的docs的列表组成                      |
> | Positions           | .pos         | 存储出现在索引中每个term的位置信息                          |
> | Payloads            | .pay         | 存储额外的term信息                                          |
> | Norms               | .nvd    .nvm | .nvm保存索引字段加权因子的元数据  . nvd保存索引字段加权数据 |
> | Per-Document-Values | .dvd    .dvm | .dvm保存索引文档评分因子元数据， .dvd保存索引文档评分数据   |
>
> 

### 7.1 索引文件词典数据结构

> 倒排索引词典位于内存当中，其结构由其重要，词典结构很多，但是需要从时间与空间上找到一个快速查找的平衡点，所以需要使用不同的数据结构；
>
> | 数据结构                 | 优缺点                                           |
> | ------------------------ | ------------------------------------------------ |
> | 跳跃表                   | 占用内存小，可调节，对模糊查询支持不友好         |
> | 排序列表                 | 使用二分法进行查找，可能导致二叉树不平衡         |
> | 字典表                   | 查询效率与字符串长度有关，只适合英文词典         |
> | 哈希表                   | 性能高，内存消耗大                               |
> | 双数组字典数             | 适合中文字典，内存占用小，常见分词工具采用该算法 |
> | Finite State Transducers | 一种有限状态机，Lucene4开源实现，并大量使用      |
> | B树                      | 磁盘索引，方便、效率慢，适用于数据库6            |
>
> 

### 7.2 跳跃表

> ​	跳跃表结构简单，内存占用小，`Lucene3.0`之前使用的是跳跃表数据结构，现阶段在 `Lucene`中的倒排表合并、文档号索引等地方还在使用；
>
> ​	





# ES学习笔记

​	ES是一个分布式的实时搜索分析引擎，它能够提欧共快速的去规模、探索对应的数据。主要包括：

- 全文检索。
- 结构化搜索。
- 结构化分析。

## 一、基本介绍

### 1、Lucence库

​	ElasticSearch使用的是Lucence做索引与搜索引擎的，能够将复杂的 **Lucence库**精简为一套简介的**RESTFul API**的操作方式，更加高效的使用Lucence库。

### 2、基本描述

​	ElasticSearch是一个：

- 分布式的实时文档存储，每一个字段都能够进行索引与搜索。

- 一个分布式实时搜索引擎。
- 能够支持多节点的扩展，支持高数据量的规模的数据操作。

​	ES是将所有的功能打包成为一个单独的服务，对外提供的是 简单的 **RESTFUL API** 的方式进行通信。能够使用web客户端或者其他请求功能（postman等）来进行ElasticSearch的操作。

### 3、基本使用

- 首先，先启动ElasticSearch。

- 直接在浏览器中发送请求： http://localhost:9200/?pretty  能够查看ElasticSearch的基本信息。	得到的结果：

  ```json
  {
      name: "Glenn Talbot",
      //集群节点名称，能够在 elasticsearch.yml中修改集群名称
      cluster_name: "elasticsearch",
      cluster_uuid: "6dqa0tIqR7ev15vVZvcnjA",
      version: {
      	number: "2.4.6",
      	build_hash: "5376dca9f70f3abef96a77f4bb22720ace8240fd",
      	build_timestamp: "2017-07-18T12:17:44Z",
      	build_snapshot: false,
      	lucene_version: "5.5.4"
      },
      tagline: "You Know, for Search"
  }
  ```

- 修改集群名称： 在 ElasticSearch.yml中添加：

  ```yml
  node.name: my_elasticsearch
  ```

- 修改集群插件、日志、数据目录：也是在 elasticsearch.yml中进行修改：

  ```yaml
  path.data: /path/to/data1,/path/to/data2 
  
  # Path to log files:
  path.logs: /path/to/logs
  
  # Path to where plugins are installed:
  path.plugins: /path/to/plugins
  ```

- 设置最小节点数：

  ```yaml
  discovery.zen.minimum_master_nodes: 2
  ```

### 4、ES交互

​	ElasticSearch使用的RESTFUL API方式进行交互操作，请求组成：

```http
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

- <VERB> : 选择的HTTP请求类型，GET、POST、PUT、DELETE等。
- <PROTOCOL> : 使用的是什么请求（http...）
- <HOST> : ElasticSearch集群中任意节点的主机名（主机地址）。
- <PORT> : 运行ElasticSearch的端口号，默认为9200。
- <PATH> : API的终端路径（也就是请求地址，例如 ： _cluster/status等）。
- <QUERY_STRING> : 任意可选的查询字符串参数(例如： ?pretty 表示的是格式化的输出)。
- <BODY> : 一个JSON格式的请求体。

**格式化的获取到文档的数量：**

```http
http://localhost:9200/_count?pretty
```

## 二、面向文档

​	ElasticSearch的基本数据类型是面向文档的，它的最小存储单位是 **对象 或者 文档**，而不是基本的key-value等方式的键值对数据。

​	ElasticSearch中 能**够对文档进**行 **索引、排序、检索和过滤** 的操作，而不是仅仅针对的是行等。

### 1、索引

​	文档数据是存储在索引当中的。**一个ElasticSearch集群包含多个索引，一个索引包含多个类型，一个类型包含多个文档。**

- **索引（动词）：** 索引意为 传统关系型数据库中的 数据库。是一个用于存储文档的地方。
- **索引（名词）：** 索引一个文档就是存储一个文档到索引中，以便以后进行查找。
- **倒排索引：** 类似于 关系型数据库中的 索引，主要是用于提高数据的检索效率（文档中的每一一个属性都是被索引了的）。

### 2、新增文档 -- PUT 

​	ES中新增数据，使用的是 **PUT** 请求方式。例如：

```json
/* put 索引名称/索引类型/索引id号  */
PUT /learn/goods/1
{
	"id": 1779,
	"goodsId": "113966985958589",
	"goodsMerchantsId": 97,
	"goodsName": "福尔摩斯探案全集：巴斯克维尔的猎犬",
	"goodsNowAmount": 10,
	"goodsDefaultAmount": 16,
	"goodsProdPlace": "重庆",
	"goodsProdTime": "Jan 16, 2019 4:44:57 PM",
	"goodsShelfLife": "十二个月",
	"goodsTypeId": 6,
	"goodsState": "selling_now",
	"goodsSellTime": "Jan 16, 2019 4:44:57 PM",
	"goodsInventory": 40,
	"rawAddTime": "Jan 16, 2019 4:44:57 PM"
}
```

### 3、检索文档 -- GET

​	检索ElasticSearch中的数据，使用的是 **GET** 发送请求。

- 查询到ES中 id 为 2 的数据: 返回的是索引、类型、版本以及数据等信息。

```http
GET /learn/goods/2
```

- 查询到所有的数据：返回的是所有的数据的 索引、、、、、

```http
GET /learn/goods/_search
```

- 查询到 商家的 id 为 89 的数据：
  - 将查询条件和条件的值 赋值给 变量 q。

```http
GET /learn/goods/_search?q=goodsMerchantsId:89
```

### 4、领域特定语言（DSL）

​	指定了使用一个JSON，来封装需要查询的条件。

- 全文检索：

```http
GET /_search?q=34
```



- 基础条件查询：

```json
GET /learn/goods/_search
{
  "query": {
    "match": {
      "goodsState": "selling_now"
    }
  }
}
```

- **过滤搜索**，使用过滤条件查询：查询出正在售卖的商品，并查找到商品在售价格大于 70的商品

```json
GET learn/goods/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "goodsState": "selling_now"
          }
        }
      ],
      "filter": {
        "range": {
          "goodsNowAmount": {
            "gt": 70
            }
          }
        }
      }
    }
}
```

- **全文搜索**，能够对文档进行全文的搜索（实现的是匹配所有文档中的字段的一些字、词）
  - **ES全文检索出的数据，能够根据相关性得分来进行排序。也就是查询到的数据与查询条件的匹配相似程度。**
  - **模糊短语匹配** ： 使用 **match** 作为条件。
  - **精确短语匹配** ： 使用 **match_phrase**作为查询条件。

```json
GET learn/goods/_search
{
  "query": {
    "match": {
      "goodsName": "洁云"
    }
  }
}
```

```json
GET learn/goods/_search
{
  "query": {
    "match_phrase": {
      "goodsName": "纸巾"
    }
  }
}
```

- **高亮搜索**，对查询到的文档的 对应的属性的值，能够匹配上查询条件的数据添加高亮效果。

```json
GET learn/goods/_search
{
  "query": {
    "match": {
      "goodsName": "纸巾"
    }
  },
  "highlight": {
    "fields": {
      "goodsName": {}
    }
  }
}
```

- 获取集群健康信息
  - green : 主分片、副分片 都正常运行。
  - yellow : 主分片正常运行，但不是所有的副分片都能够进行正常的运行。
  - red : 有主分片没有能够正常运行。

```json
GET /_cluster/health?pretty

{
  "cluster_name": "elasticsearch",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 16,
  "active_shards": 16,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 16,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50
}
```





### 3、更新文档

#### 1） PUT 更新 --- 全文档更新

​	使用 PUT方式进行数据的更新操作，会将之前的文档删除，然后将数据进行更新为PUT里面的数据。例如：

```json
PUT /search_goods/goods/1807
{
  "goodsName": "这个是商品 ",
  "goodsType": 8
}
```

​	**注意：**更新文档，是先将之前的文档标记为 ==清除状态==，然后更换id指向新增的文档。

#### 2）update 更新 --- 文档部分更新

​	使用 update方式的更新，能够保证的是覆盖现有字段的值，以及新增新的字段。例如：

```http
POST /learn/goods/12/_update
{
  "doc": {
    "birth": "2018-11-12",
    "args": [
      1,2,3
      ]
  }
}
```

#### 3）update更新 --- 更新失败重试

​	当多个进程进行修改操作以后，则可能会导致更新失败（重建索引的时候，版本错误了）。则需要设置更新失败的重试次数。

​	设置参数 `retry_on_conflict` 来自动完成，这个参数规定了失败之前 `update` 应该重试的次数，它的默认值为 `0` 。	

```http
POST /learn/goods/12/_update?retry_on_conflict=5
{
  "doc": {
    "birth": "2018-11-12",
    "args": [
      1,2,3
      ]
  }
}
```

### 4、创建文档

#### 1）POST--创建文档

​	使用 POST请求创建文档，能够自动的调用ElasticSearch生成 _id 的方法，保证文档创建出来的id唯一性。例如：

```json
POST /learn/goods
{
  "name": "asfsad",
  "age": 50
}
```

#### 2）PUT --创建文档

​	使用PUT方式进行文档的创建，需要手动指定文档的 _id，以及指定操作类型为 create。例如：

```json
PUT /learn/goods/12?op_type=create
{
  "name": "aaa",
  "age": 21
}
```

- 创建成功，会返回的状态码为 201 create，以及携带的元数据信息。
- 创建失败，会返回 409 Conflict响应码，以及携带者错误信息。

### 5、删除文档 -- DELETE

​	删除文档，使用的是 DELETE标签。例如：

```http
DELETE /learn/goods/12
```

- **查找到了对应的文档**：ElasticSearch如果在库中查找到了对应的文档，则进行删除的操作，并返回 200 状态码。返回的结构体中，version版本增加了1。

- **未查找到对应的文档**：ElasticSearch如果未查找到对应的文档，则返回 404 Not Found 状态码。返回的结构体中，version的版本增加1。

  **注意：** 在ElasticSearch中，对文档的操作，无论是正常还是异常操作， _version的版本都会记录依次增加，这是实现跨多节点时保障正确执行的顺序操作。

  **注意：**在ElasticSearch中，删除文档，也不会立即将文档从磁盘中进行删除，而是先将其标记为 **删除状态**。待到不断的索引更多的数据，ElasticSearch才会在后台清除标记为已删除的文档。

### 6、数据上锁

-  ***悲观并发控制*** 

   这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。 

-  ***乐观并发控制***

   Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。 

  - 在ElasticSearch中，==使用的是 **_version** 来实现乐观锁==。使用方式：

    - 第一步，查看文档对应的版本，此处版本为1

    ```json
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "1",
      "_version" : 1,
      "found" :    true,
      "_source" :  {
          "title": "My first blog entry",
          "text":  "Just trying this out..."
      }
    }
    ```

    - 第二步，使用当前版本，来进行文档的更新操作

    ```json
    PUT /website/blog/1?version=1 
    {
      "title": "My first blog entry",
      "text":  "Starting to get the hang of this..."
    }
    ```

    - 第三步，版本变成了2。下次在更改，则需要将版本提升为 2。否则会更新失败。

  - 手动指定创建、更新文档以后，文档的版本号。例如：

  ```json
  <!--首先先创建一个版本号为5的文档-->
  PUT /website/blog/2?version=5&version_type=external
  {
    "title": "My first external blog entry",
    "text":  "Starting to get the hang of this..."
  }
  
  <!--更改这个文档，并将其版本指定为10-->
  PUT /website/blog/2?version=10&version_type=external
  {
    "title": "My first external blog entry",
    "text":  "This is a piece of cake..."
  }
  ```

### 7、获取多个文档

​	在ElasticSearch中，能够使用 **`multi_get`、`mget API`**来进行将多个请求组合在一起进行查询，避免了但条查询的网络延迟、传输延迟。

#### 1）mget API  批量操作 ---只能查询

​	**使用 mget API，需要一个dos数组作为参数，每个元素包含需要检索的元数据(\_index 、\_type  、_id)。如果需要检索多个特定字段，则可以将其加入到 `\_source` 标签中。**

实例一：查询到对应的商品的数据，使用 _source 指定对应的属性名称。

```http
GET /_mget
{
  "docs": [
    {
      "_index" : "search_goods",
      "_type" : "goods",
      "_id" : 1903
    },
    {
      "_index" : "search_goods",
      "_type" : "goods",
      "_id" : 1905,
      "_source" : ["goodsImage3","goodsId","goodsNowAmount"]
    }
  ]
}
```

实例二：简洁写法

```http
GET /search_goods/goods/_mget
{
  "ids":["1903","1904"]
}
```

#### 2）bulk  批量操作 -- 增删改查

​	**bulk能够允许在单个步骤中进行多次 create、index、update、delete请求。**

​	bulk的请求体格式为：

```http
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```

​	例如一：

```http
POST /_bulk
{"create" : {"_index": "learn", "_type": "goods", "_id": "5002"}}
{"title" : "asdfasdf", "age" : 25}
{"delete": {"_index": "search_goods", "_type": "goods", "_id": "1907"}}
```

​	例如二：

```http
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } 
```



## 三、搜索

### 1、基本概念

- **映射（Mapping）** ：描述数据在每一个字段内如何存储。
- **分析（Analysis）** ： 全文是如何处理，使之可以被搜索。
- **领域特定查询语言（Query SQL）** ： ElasticSearch中强大的查询语言。

### 2、多索引、多类型

​	通常情况下，在多个索引中进行查询数据是常有的，可以通过URL中指定特殊的索引和类型达到想要的效果。例如：

> /\_search
>        在所有的索引中搜索所有的类型 
>/gb/\_search
>       在 gb 索引中搜索所有的类型 
>/gb,us/\_search
>        在 gb 和 us 索引中搜索所有的文档 
>/g\*,u\*/\_search
>        在任何以 g 或者 u 开头的索引中搜索所有的类型 
>/gb/user/\_search
>        在 gb 索引中搜索 user 类型 
>/gb,us/user,tweet/\_search
>        在 gb 和 us 索引中搜索 user 和 tweet 类型 
>/\_all/user,tweet/\_search
>        在所有的索引中搜索 user 和 tweet 类型 

### 3、查询分页

​	通常情况下，查询分页是根据  `size` 和 `from` 两个关键参数来设定的。

- size ： 显示应该返回的结果数量。

- from ： 显示应该跳过的厨师结果数量。

  例如：

  ```http
  GET /_all/goods/_search?size=3&from=7
  ```

### 4、倒排索引

​	ElasticSearch中使用的是都==**倒排索引**== 的数据结构来进行快速的全文检索。一个倒排索引由文档汇总所有的不重复词的列表组成。

​	将文档中的数据进行拆分成单独的词，可以称它为 **词条** 或者 **tokens**，创建一个包含所有不重复词条的排序列表，然后累出每一个词条出现在哪一个文档中即可。

### 5、分析与分析器

#### 1）词条分析

​	将需要进行拆分的词条，调用 ==**recall分析器**== 进行词条的分析操作，主要分为三部分：

- **字符过滤器** ，将字符串按照顺序通过每一个字符过滤器，他们的任务是在分此前整理字符串。（字符过滤器主要是能够用来去掉不必要的属性，例如： HTML标签、将&改为and）。
- **分词器** ，字符串使用 分词器 将其分解为单个的词条。（一个简单的分词器遇到空格或者标点的时候，就可能将其进行拆分）。

- **token过滤器** ，词条按照顺序**通过 token过滤器**，这个过程可能会改变词条（例如：小写化Quick）、删除词条（删除像 a、and、or等无用词条）、增加词条（jump、leap同义词）。

#### 2）词条分析器

- **标准分析器** ，标准分析器是ElasticSearch中默认的分析器。
  - 使用Unicode联盟定义的 单词边界 进行文本的划分。
  - 删除绝大多数标点，将词条小写。
- **简单分析器** ，简单分析器在任何不是字母的地方分割文本，将词条小写。

- **空格分析器** ，在有空格的地方进行文本的划分。

- **语言分析器** ，根据语言的特性进行不同的文本的划分操作。

#### 3）分析器使用场景

- 索引一个文档，需要将其进行拆分为词条来创建倒排索引。

- 当需要进行全文搜索，也需要将搜索条件进行分析成词条，来与倒排索引进行匹配的操作。

### 6、词条映射

#### 1）简单域映射类型

​	ElasticSearch支持的简单域类型：

- 字符串 ： string。
- 整数 ： byte、short、integer、long
- 浮点数 ： float、double
- 布尔类型 ： boolean
- 日期 ： date

​	映射主要是针对不同的数据类型，映射到ES中的不同的数据类型。

#### 2）自动映射关系

​	在ElasticSearch中，会使用动态映射操作，通过JSON中的基本数据类型，尝试猜测域类型，使用规则如下：

```
布尔类型： true、false        域类型：boolean
整数： 123				   	  域类型：long
浮点数：123.5			     域类型：double
字符串，有效日期 2014-11-20    域类型：date
字符串 for update			  域类型：string
```

#### 3）查看文档映射的类型：

```http
GET /learn/_mapping
```

#### 4）字段索引方式--index

​	index属性使用与控制在ElasticSearch中，**控制字段的索引方式**，主要包含三种：

- **analyzed** : 全文索引，首先先分析字符串，然后进行索引。
- **not_analyzed** ：索引这个域，索引精确值。
- **no** ： 不索引这个域，这个域不会被搜索到。

#### 5）删除索引

```http
DELETE /learn
```

#### 6）创建新索引

```http
PUT /user_my
{
  "mappings": {
    "tweet" : {
      "properties": {
        "tweet" : {
          "type": "string",
          "analyzer": "standard"
        },
        "date" : {
          "type": "date"
        },
        "name" : {
          "type": "string"
        }
      }
    }
  }
}

```

#### 7）复杂核心域类型

​	复杂核心域类型主要是包含 null、数组、对象等。

- **多值域** ： 使用一个标签包含多个数据，比如数组：{"tag": ["zsl", "fsp"]}
  - 数组封装在ElasticSearch中以后，会变得无序。

### 7、请求体查询

#### 1）GET请求携带参数查询

​	使用GET请求，也能够携带请求体进行请求的查询操作。例如：

- 空请求体： 

  ```http
  GET  /_search
  ```

- 携带索引，进行查询：

  ```http
  GET /learn/_search
  ```

- 携带分页信息：

  ```http
  GET /search_goods/_search
  {
    "from": 30,
    "size": 20
  }
  ```

#### 2）POST携带请求参数查询

​	在大多数情况下，为了能够更好的进行请求参数的携带，使用POST请求进行查询。例如：

```http
POST /search_goods/_search
{
  "from": 0,
  "size": 2
}
```

#### 3）Query DSL查询表达式

​	ElasticSearch中，能够使用一种非常灵活又富有表现能力的Query语言，使用简单的JSON接口来展现大多数的功能。**在ElasticSearch中，只需要将查询条件传递给 query 参数中。**例如：

```http
GET /_search
{
  "query": {
    "match_all": {}
  }
}
```

- **查询语句的结构为：**

```http
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

#### 4）合并查询语句--bool查询

**接收以下参数：**

> - must：
>   文档 必须匹配这些条件才能被包含进来。 

> - must_not：
>   文档 必须不匹配这些条件才能被包含进来。 

> - should：
>   如果满足这些语句中的任意语句，将增加 _score，否则，无任何影响。它们主要用于修正每个文档的相关性得分。 

> - filter：
>   必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

```json
{
    "bool": {
        "must": { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

### 8、查询与过滤

​	在ElasticSearch中，拥有两套对应的查询方式：

- 条件查询： Query
- 过滤查询： filter

​        条件查询 query主要是用于评分查询的操作，用于计算每一个文档的 **相关程度**，并将其分配给 "_score" 字段进行记录。**过滤查询要从文档中查询对应的匹配的文档，并计算相关的得分，主要是用于全文检索的操作。**

​        **过滤查询 filter 主要是能够不执行评分的操作，在不需要全文检索的情况下，通常能够提高效率。**

### 9、重要查询

- **==match_all查询==** ：match_all 查询简单的 **匹配所有文档**。例如：

  ```http
  GET search_goods/_search
  {
    "query": {
      "match": {
      "goodsName" : "阿斯蒂芬"
      }
    }
  }
  ```

- **==match查询==** ： match 查询是进行任何查询中的**标准查询**操作。

  - match查询如果是针对的 not_analyzed 类型的字段，则是进行精确查询。
  - match进行精确查询的时候，能够使用 fliter 来代替query。

- **==multi_match查询==** ： multi_match **能够在多个字段上进行match操作**。例如：

  ```http
  GET search_goods/_search
  {
    "query": {
      "multi_match": {
      "query" : "阿斯蒂芬",
      "fields" : ["goodsName","goodsDetail1"]
      }
    }
  }
  ```

- ==**range查询**== ： 查询找出那些落**在指定区间内的数字或者时间**。

  - **gt** ： 大于
  - **gte** ： 大于等于
  - **lt** ： 小于
  - **lte** ： 小于等于
  - 例如：

  ```http
  GET /search_goods/_search
  {
    "query": {
      "range": {
      "goodsDefaultAmount": {
        	"gte": 30,
        	"lt": 50
        }
    	}
    }
  }
  ```

- **==trem查询==** ： trem查询被用于精确值的匹配操作，这些精确值可能是数字、时间、布尔值等 not_analyzed类型的字符串。例如：

  ```http
  GET /search_goods/_search
  {
    "query": {
      "term": {
        "goodsDefaultAmount": {
          "value": "50"
        }
      }
    }
  }
  ```

- **==trems查询==** ： terms查询与term相比，是能够进行多指匹配操作

```http
GET /search_goods/_search
{
  "query": {
    "terms": {
      "goodsDetail1": [
        "书",
        "本"
      ]
    }
  }
}
```

- **==exists查询==** ：exists 查询适用于指定那些字段中有值（exists == 存在）的文档。例如：
- **==missing查询==** ： missing 查询适用于指定字段中无值（missing == 不存在）的文档。

### 10、多组合查询

​	多个字段上查询各种各样的文本，并根据条件进行过滤操作。类似于构建高级查询的操作。

- **==bool查询==** ： bool查询将多个查询组合在一起，成为一个用户自己需求的bool查询。参数：

  - must ： 文档必须匹配这些条件才能够被包含进来。
  - must_not ： 文档必须不匹配这些条件才能够被包含进来。
  - should ： 文档中瞒住这些语句中的任意语句，将增加 `_score` 属性，否则，不进行计分的操作。
  - filter ： 必须匹配，但是不以评分、过滤的模式来进行文档的查询和过滤操作.

  **注意**：*如果不存在must语句，则至少需要能够匹配其中的一条should语句。如果存在must语句，则能够不需要should语句 。*

  **例如：**

  ```http
  GET /search_goods/_search
  {
    "query": {
      "bool": {
        "must": [
          {"match": {
            "goodsDefaultAmount": 55
          }}
        ],
        "must_not": [
          {"match": {
            "goodsDetail1": "书"
          }}
        ],
        "should": [
          {"range": {
            "goodsDefaultAmount": {
              "gte": 30,
              "lte": 70
            }
          }}
        ],
        "filter": {
          "range": {
            "goodsTypeId": {
              "gte": 7,
              "lt": 8
            }
          }
        }
      }
    }
  }
  ```

### 11、查询验证

-  **==/_validate/query==** 关键字来进行查询条件的验证操作。例如:

```http
GET /search_goods/_validate/query
{
  "query": {
    "match": {
      "goodsDesc": "TEXT"
    }
  }
}
```

返回的结果为：

```http
{
   /*表示的是对应的查询的语句是否正常*/
  "valid": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  }
}
```

- 使用 **==/_validate/query?explain==** 来对查询语句进行可读的描述。例如：

```http
GET /search_goods/_validate/query?explain
{
  "query": {
    "match": {
      "goodsDesc": "TEXT"
    }
  }
}
```

​	结果为：

```http
{
  "valid": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "explanations": [
    {
      "index": "search_goods",
      "valid": true,
      /*表示的是这个字段对应的字段类型*/
      "explanation": "goodsDesc:text"
    }
  ]
}
```

### 12、排序

​	ElasticSearch中，是将 **相关性抽象成为了一个浮点数**。相关性的得分是由一个浮点数进行表示的，并在搜索结果中的 `_score` 参数中返回。默认是使用 `_score` 进行**降序排序**。

- 使用多个字段进行排序：

```http
GET /search_goods/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "goodsDetail1": "作者"
        }
      }
    }
  },
  "sort": [
    {
      "rawAddTime": {
        "order": "desc"
      },
      "goodsDefaultAmount": {
        "order": "desc"
      }
    }
  ]
}
```

## 四、索引

### 1、创建索引

​	ElasticSearch中，通常情况下是需要创建索引来实现自定义索引属性的。手动创建索引的代码：

```http
PUT /my_index
{
  "settings": {
    ...
  },
  "mappings": {
    "type_one": {...},
    "type_two": {...},
    ....
  }
}
```

### 2、删除索引

- 删除一条索引：

```http
DELETE /learn
```

- 删除指定的多条索引：

```http
DELETE /learn,search_goods
```

- 删除全部索引：

```http
DELETE /*
```

### 3、索引设置

- **==number_of_shards==** ：在ElasticSearch中，默认的索引的主分片数为 5，且索引创建以后就不能够进行修改。

- **==number_of_replicas==** ：每一个主分片的数目，默认值都是1。这个备份分片数目能够在后期进行更改。

  例如：创建索引，设置对应的分片数据：

```http
PUT /my_index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

​	修改创建的索引的备份分片数目：

```http
PUT /my_index/_settings
{
  "number_of_replicas": 2
}
```

### 4、分析器

​	分析器主要是适用于分割索引中的属性的数据。ElasticSearch中自定义的分析器主要有：

- **==standard 分词器==** ： 通过单词边界分割输入的文本。

#### 1）分析器解析

​	分析器中包含了三种函数，主要是：

- **==字符过滤器==** ：字符过滤器  用来 `整理` 一个尚未被分词的字符串。例如，如果我们的文本是HTML格式的，它会包含像 `<p>` 或者 `<div>` 这样的HTML标签，这些标签是我们不想索引的。我们可以使用 [`html清除` 字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-htmlstrip-charfilter.html) 来移除掉所有的HTML标签，并且像把 `&Aacute;` 转换为相对应的Unicode字符 `Á`  这样，转换HTML实体。

- **==分词器==** ： 一个分析器 *必须* 有一个唯一的分词器。分词器把字符串分解成单个词条或者词汇单元。 `标准` 分析器里使用的 [`标准` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-standard-tokenizer.html)   把一个字符串根据单词边界分解成单个词条，并且移除掉大部分的标点符号，然而还有其他不同行为的分词器存在。

- **==词单元过滤器==** ：词单元过滤器可以修改、添加或者移除词单元。我们已经提到过 [ `lowercase` ](http://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenizer.html) 和  [ `stop` 词过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-stop-tokenfilter.html) ，但是在 Elasticsearch 里面还有很多可供选择的词单元过滤器。 [词干过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-stemmer-tokenfilter.html) 把单词 `遏制` 为  词干。 [ `ascii_folding` 过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-asciifolding-tokenfilter.html)移除变音符，把一个像 `"très"` 这样的词转换为 `"tres"` 。 [`ngram`](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-ngram-tokenfilter.html) 和 [ `edge_ngram` 词单元过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-edgengram-tokenfilter.html) 可以产生  适合用于部分匹配或者自动补全的词单元。

### 5、属性

​	属性对应着也就是对象中的字段，属性具有三个重要的设置：

- ==**type**== ： 字段的数据类型。
- ==**index**== ： 字段的索引类型。
  - 字段是否被当做全文检索（analyzed）；
  - 字段是否被当成一个精确的值（not_analyzed）；
  - 字段完全不可被检索（no）。
- ==**analyzer**== ： 确定在索引和搜索的时候使用的是 **全文检索**。

### 6、元数据 

#### 1） _source 字段

​	==**_source**== 字段存储代表文档体的JSON字符串。和所有存储的字段一致， _source 字段在被写入磁盘之前会先被压缩。

​	**==_source==** 存储的数据的作用为：

-  搜索结果包括了整个可用的文档——不需要额外的从另一个的数据仓库来取文档。 

-  如果没有 `_source` 字段，部分 `update` 请求不会生效。 

-  当你的映射改变时，你需要重新索引你的数据，有了_source字段你可以直接从Elasticsearch这样做，而不必从另一个（通常是速度更慢的）数据仓库取回你的所有文档。 

-  当你不需要看到整个文档时，单个字段可以从 `_source` 字段提取和通过 `get` 或者 `search` 请求返回。 

-  调试查询语句更加简单，因为你可以直接看到每个文档包括什么，而不是从一列id猜测它们的内容。 

  **在搜索的请求中，可以通过指定 `_source` 的参数，来返回特定的字段**。例如：

  ```http
  GET /_search
  {
    "query": {
      "match_all": {}
    },
    "_source": [ "goodsDetail1", "goodsName" ]
  }
  ```

#### 2）**==_all==** 字段

​	在默认情况下，如果需要进行查询的操作的话，可以使用全字段匹配的操作。例如：

```http
GET /_search
{
  "query": {
    "match": {
      "_all": "小书童"
    }
  }
}
```

#### 3）文档标示

​	文档标识主要是包含了四个元数据字段：

- _id ：文档的id值
- _type ： 文档的类型
- _index ：文档的索引
- _uid ：\_type 和 \_id 链接在一起构造成的 typeid。















# 第一章 ElasticSearch入门篇

## 第一节 ElasticSearch概述
### 1.1ElasticSearch是一个基于Lucene的搜索服务器。
它提供了一个分布式多用户能力的全文搜索引擎，基于RESTfulweb接口。ElasticSearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。构建在全文检索开源软件Lucene之上的Elasticsearch，不仅能对海量规模的数据完成分布式索引与检索，还能提供数据聚合分析。据国际权威的数据库产品评测机构DBEngines的统计，在2016年1月，Elasticsearch已超过Solr等，成为排名第一的搜索引擎类应用
概括：基于Restful标准的高扩展高可用的实时数据分析的全文搜索工具

### 1.2ElasticSearch的基本概念
Index

　　类似于mysql数据库中的database
　　

Type

　　类似于mysql数据库中的table表，es中可以在Index中建立type（table），通过mapping进行映射。
　　

Document

　　由于es存储的数据是文档型的，一条数据对应一篇文档即相当于mysql数据库中的一行数据row，一个文档中可以有多个字段也就是mysql数据库一行可以有多列。
　　
Field
　　es中一个文档中对应的多个列与mysql数据库中每一列对应
　　

Mapping

　　可以理解为mysql或者solr中对应的schema，只不过有些时候es中的mapping增加了动态识别功能，感觉很强大的样子，其实实际生产环境上不建议使用，最好还是开始制定好了对应的schema为主。
　　

indexed

　　就是名义上的建立索引。mysql中一般会对经常使用的列增加相应的索引用于提高查询速度，而在es中默认都是会加上索引的，除非你特殊制定不建立索引只是进行存储用于展示，这个需要看你具体的需求和业务进行设定了。

Query DSL

　　类似于mysql的sql语句，只不过在es中是使用的json格式的查询语句，专业术语就叫：QueryDSL

GET/PUT/POST/DELETE

　　分别类似与mysql中的select/update/delete......
　　
　　
### 1.3Elasticsearch的架构
![image](https://images2015.cnblogs.com/blog/980882/201702/980882-20170208171207963-1711795457.png)

Gateway层

es用来存储索引文件的一个文件系统且它支持很多类型，例如：本地磁盘、共享存储（做snapshot的时候需要用到）、hadoop的hdfs分布式存储、亚马逊的S3。它的主要职责是用来对数据进行长持久化以及整个集群重启之后可以通过gateway重新恢复数据。

Distributed Lucene Directory

Gateway上层就是一个lucene的分布式框架，lucene是做检索的，但是它是一个单机的搜索引擎，像这种es分布式搜索引擎系统，虽然底层用lucene，但是需要在每个节点上都运行lucene进行相应的索引、查询以及更新，所以需要做成一个分布式的运行框架来满足业务的需要。

四大模块组件

districted lucene directory之上就是一些es的模块，Index Module是索引模块，就是对数据建立索引也就是通常所说的建立一些倒排索引等；Search Module是搜索模块，就是对数据进行查询搜索；Mapping模块是数据映射与解析模块，就是你的数据的每个字段可以根据你建立的表结构通过mapping进行映射解析，如果你没有建立表结构，es就会根据你的数据类型推测你的数据结构之后自己生成一个mapping，然后都是根据这个mapping进行解析你的数据；River模块在es2.0之后应该是被取消了，它的意思表示是第三方插件，例如可以通过一些自定义的脚本将传统的数据库（mysql）等数据源通过格式化转换后直接同步到es集群里，这个river大部分是自己写的，写出来的东西质量参差不齐，将这些东西集成到es中会引发很多内部bug，严重影响了es的正常应用，所以在es2.0之后考虑将其去掉。

Discovery、Script

es4大模块组件之上有 Discovery模块：es是一个集群包含很多节点，很多节点需要互相发现对方，然后组成一个集群包括选主的，这些es都是用的discovery模块，默认使用的是 Zen，也可是使用EC2；es查询还可以支撑多种script即脚本语言，包括mvel、js、python等等。

 Transport协议层

再上一层就是es的通讯接口Transport，支持的也比较多：Thrift、Memcached以及Http，默认的是http，JMX就是java的一个远程监控管理框架，因为es是通过java实现的。

RESTful接口层

最上层就是es暴露给我们的访问接口，官方推荐的方案就是这种Restful接口，直接发送http请求，方便后续使用nginx做代理、分发包括可能后续会做权限的管理，通过http很容易做这方面的管理。如果使用java客户端它是直接调用api，在做负载均衡以及权限管理还是不太好做。
### 1.4RESTfull API
一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。在目前主流的三种Web服务交互方案中，REST相比于SOAP（Simple Object Access protocol，简单对象访问协议）以及XML-RPC更加简单明了

(Representational State Transfer

意思是：表述性状态传递)

它使用典型的HTTP方法，诸如GET,POST.DELETE,PUT来实现资源的获取，添加，修改，删除等操作。即通过HTTP动词来实现资源的状态扭转
复制代码

GET 用来获取资源

POST 用来新建资源（也可以用于更新资源）、获取资源

PUT 用来更新资源

DELETE 用来删除资源

### 1.5CRUL命令
以命令的方式执行HTTP协议的请求
GET/POST/PUT/DELETE

示例：
访问一个网页

curl www.baidu.com

curl -o tt.html www.baidu.com

显示响应的头信息

curl -i www.baidu.com

显示一次HTTP请求的通信过程

curl -v www.baidu.com

执行GET/POST/PUT/DELETE操作

curl -X GET/POST/PUT/DELETE url

### 1.6CentOS7下安装ElasticSearch6.2.4
(1)配置JDK环境

 配置环境变量

export JAVA_HOME="/opt/jdk1.8.0_144"

export PATH="$JAVA_HOME/bin:$PATH"

export CLASSPATH=".:$JAVA_HOME/lib"

(2)安装ElasticSearch6.2.4

下载地址：https://www.elastic.co/cn/downloads/elasticsearch

启动报错：
![image](https://note.youdao.com/yws/api/personal/file/F967846635974B32A8D490508E781F00?method=download&shareKey=2ad37f2dc1cc016e1afe3a0849046cef)

解决方式：
bin/elasticsearch -Des.insecure.allow.root=true

或者修改bin/elasticsearch，加上ES_JAVA_OPTS属性：
ES_JAVA_OPTS="-Des.insecure.allow.root=true"

再次启动：
![image](https://note.youdao.com/yws/api/personal/file/F432E6405D5C4D5599A80F3F2F0FEB83?method=download&shareKey=242de0ee6034de7f0e46c6c120d88e68)

这是出于系统安全考虑设置的条件。由于ElasticSearch可以接收用户输入的脚本并且执行，为了系统安全考   虑，建议创建一个单独的用户用来运行ElasticSearch。

创建用户组和用户：

groupadd esgroup

useradd esuser -g esgroup -p espassword

更改elasticsearch文件夹及内部文件的所属用户及组：

cd /opt

chown -R esuser:esgroup elasticsearch-6.2.4

切换用户并运行：

su esuser

./bin/elasticsearch

再次启动显示已杀死：
![image](https://note.youdao.com/yws/api/personal/file/A03FC0640DD043EBBAFF66A34CB4B225?method=download&shareKey=073d77cddf7efa2810059f5b591b3548)

需要调整JVM的内存大小：

vi bin/elasticsearch

ES_JAVA_OPTS="-Xms512m -Xmx512m"

再次启动：启动成功

如果显示如下类似信息：

[INFO ][o.e.c.r.a.DiskThresholdMonitor] [ZAds5FP] low disk watermark [85%] exceeded on     [ZAds5FPeTY-ZUKjXd7HJKA][ZAds5FP][/opt/elasticsearch-6.2.4/data/nodes/0] free: 1.2gb[14.2%],     replicas will not be assigned to this node

需要清理磁盘空间。

后台运行：./bin/elasticsearch -d

测试连接：curl 127.0.0.1:9200

会看到一下JSON数据：
 [root@localhost ~]# curl 127.0.0.1:9200
  {
  "name" : "rBrMTNx",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "-noR5DxFRsyvAFvAzxl07g",
  "version" : {
​    "number" : "5.1.1",
​    "build_hash" : "5395e21",
​    "build_date" : "2016-12-06T12:36:15.409Z",
​    "build_snapshot" : false,
​    "lucene_version" : "6.3.0"
  },
  "tagline" : "You Know, for Search"
 }


实现远程访问：
需要对config/elasticsearch.yml进行   配置：
​    network.host: 192.168.25.131
​    
再次启动报错：
![image](https://note.youdao.com/yws/api/personal/file/EA3ED55EB0ED40C683112AC6ED8716AE?method=download&shareKey=7517e79986e6585de886c59966057d9c)

处理第一个错误：

vim /etc/security/limits.conf       //文件最后加入

esuser soft nofile 65536

esuser hard nofile 65536

esuser soft nproc 4096

esuser hard nproc 4096

处理第二个错误：

进入limits.d目录下修改配置文件。

vim /etc/security/limits.d/20-nproc.conf 
修改为 esuser soft nproc 4096

处理第三个错误：
​    
vim /etc/sysctl.conf

vm.max_map_count=655360

执行以下命令生效：
sysctl -p

关闭防火墙：systemctl stop firewalld.service

再次启动成功！


### 1.7安装Head插件
Head是elasticsearch的集群管理工具，可以用于数据的浏览和查询

(1)elasticsearch-head是一款开源软件，被托管在github上面，所以如果我们要使用它，必须先安装git，通过git获取elasticsearch-head

(2)运行elasticsearch-head会用到grunt，而grunt需要npm包管理器，所以nodejs是必须要安装的

(3)elasticsearch5.0之后，elasticsearch-head不做为插件放在其plugins目录下了。
使用git拷贝elasticsearch-head到本地

cd /usr/local/

 git clone git://github.com/mobz/elasticsearch-head.git

(4)安装elasticsearch-head依赖包

[root@localhost local]# npm install -g grunt-cli

[root@localhost _site]# cd /usr/local/elasticsearch-head/

[root@localhost elasticsearch-head]# cnpm install

(5)修改Gruntfile.js

[root@localhost _site]# cd /usr/local/elasticsearch-head/

[root@localhost elasticsearch-head]# vi Gruntfile.js

在connect-->server-->options下面添加：hostname:’*’，允许所有IP可以访问

(6)修改elasticsearch-head默认连接地址
[root@localhost elasticsearch-head]# cd /usr/local/elasticsearch-head/_site/

[root@localhost _site]# vi app.js

将this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";中的localhost修改成你es的服务器地址

(7)配置elasticsearch允许跨域访问

打开elasticsearch的配置文件elasticsearch.yml，在文件末尾追加下面两行代码即可：

http.cors.enabled: true

http.cors.allow-origin: "*"

(8)打开9100端口

[root@localhost elasticsearch-head]# firewall-cmd --zone=public --add-port=9100/tcp --permanent

重启防火墙

[root@localhost elasticsearch-head]# firewall-cmd --reload

(9)启动elasticsearch

(10)启动elasticsearch-head

[root@localhost _site]# cd /usr/local/elasticsearch-head/

[root@localhost elasticsearch-head]# node_modules/grunt/bin/grunt server

(11)访问elasticsearch-head

关闭防火墙：systemctl stop firewalld.service

浏览器输入网址：http://192.168.25.131:9100/

### 1.8安装Kibana
Kibana是一个针对Elasticsearch的开源分析及可视化平台，使用Kibana可以查询、查看并与存储在ES索引的数据进行交互操作，使用Kibana能执行高级的数据分析，并能以图表、表格和地图的形式查看数据

(1)下载Kibana
https://www.elastic.co/downloads/kibana

(2)把下载好的压缩包拷贝到/soft目录下

(3)解压缩，并把解压后的目录移动到/user/local/kibana

(4)编辑kibana配置文件

[root@localhost /]# vi /usr/local/kibana/config/kibana.yml

![image](https://images2017.cnblogs.com/blog/210978/201708/210978-20170805113725272-708617928.png)

将server.host,elasticsearch.url修改成所在服务器的ip地址

(5)开启5601端口

Kibana的默认端口是5601

开启防火墙:systemctl start firewalld.service

开启5601端口:firewall-cmd --permanent --zone=public --add-port=5601/tcp

重启防火墙：firewall-cmd –reload

(6)启动Kibana

[root@localhost /]# /usr/local/kibana/bin/kibana

浏览器访问：http://192.168.25.131:5601

### 1.9安装中文分词器
方法一：

(1)下载中文分词器
https://github.com/medcl/elasticsearch-analysis-ik

    下载elasticsearch-analysis-ik-master.zip

(2)解压elasticsearch-analysis-ik-master.zip

   unzip elasticsearch-analysis-ik-master.zip

(3)进入elasticsearch-analysis-ik-master，编译源码

mvn clean install -Dmaven.test.skip=true 

(4)在es的plugins文件夹下创建目录ik

(5)将编译后生成的elasticsearch-analysis-ik-版本.zip移动到ik下，并解压

(6)解压后的内容移动到ik目录下  

方法二（5.5.1版本以后才能够使用）：
只需在es根目录下执行  
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.1/elasticsearch-analysis-ik-6.2.1.zip

## 第二节 ElasticSearch基本操作

### 2.1倒排索引
​	**Elasticsearch 使用一种称为 倒排索引 的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。**

#### 2.1.1 分词器介绍及内置分词器

分词器：从一串文本中切分出一个一个的词条，并对每个词条进行标准化，主要包括三部分：

- character filter：分词之前的预处理，过滤掉HTML标签，特殊符号转换等

- tokenizer：分词

- token filter：标准化

**内置分词器：**

- standard 分词器：(默认的)他会将词汇单元转换成小写形式，并去除停用词和标点符号，支持中文采用的方法为单字切分

- simple 分词器：首先会通过非字母字符来分割文本信息，然后将词汇单元统一为小写形式。该分析器会去掉数字类型的字符。

- Whitespace 分词器：仅仅是去除空格，对字符没有lowcase化,不支持中文；
  并且不对生成的词汇单元进行其他的标准化处理。

- language 分词器：特定语言的分词器，不支持中文



### 2.2文档CRUD

- **添加索引：**这里进行了索引的创建，并设置其主分片与从分片的个数.

```http
PUT /lib/
{
  "settings":{
      "index":{
        "number_of_shards": 5,
        "number_of_replicas": 1
      }
   }
}
```



- **查看索引信息:**

>- 查看单个索引结构：GET /lib/_settings
>- 查看所有索引结构：GET _all/_settings



- **添加文档:**   
  - 方法一：自己指定id值，可使用 PUT 方法  

```http
PUT /lib/user/1
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
```
​	- 方法二：自己不指定id值，可以使用POST生成id  

```http
POST /lib/user/
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         23,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```



- **查看文档:**  

>- GET /lib/user/1 ： 查看那个索引下的user类型的id为1的文档。
>- GET /lib/user/ ： 查看lib索引下的user类型的所有文档。
>- GET /lib/user/1?_source=age,interests ： 只查询age与interests两个字段的值。



- **更新文档:**
  - 方法一：直接覆盖。

```http
PUT /lib/user/1
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         36,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
```
​	- 方法二：根据文档的查询目录来进行修改：
```http
POST /lib/user/1/_update
{
  "doc":{
      "age":33
      }
}
```



- **删除一个文档**

>- DELETE /lib/user/1



- **删除一个索引**

>- DELETE /lib



### 2.3批量获取文档:Multi Get API

​	使用Multi Get API可以通过索引名、类型名、文档id一次得到一个文档集合，文档可以来自同一个索引库，也可以来自不同索引库。

- **简单使用：**

```http
GET /_mget
{
    "docs":[
       {
           "_index": "lib",
           "_type": "user",
           "_id": 1
       },
       {
           "_index": "lib",
           "_type": "user",
           "_id": 2
       },
       {
           "_index": "lib",
           "_type": "user",
           "_id": 3
       }
     ]
}
```

- **可以指定具体的字段：**

```http
GET /_mget
{ 
    "docs":[
        {
            "_index": "lib",
            "_type": "user",
            "_id": 1,
            "_source": "interests"
        },
        {
            "_index": "lib",
            "_type": "user",
            "_id": 2,
            "_source": ["age","interests"]
        }
    ]
}
```

- **获取同索引同类型下的不同文档：**  
  方法一：在mget目录填写索引与类型：

```http
GET /lib/user/_mget
{
    "docs":[
        {
        	"_id": 1
        },
        {
        	"_type": "user",
        	"_id": 2,
        } 
    ]
}
```
方法二：将需要获取的id聚合在一起：
```http
GET /lib/user/_mget
{
   
   "ids": ["1","2"]
   
}
```


### 2.4使用Bulk API 实现批量操作

**bulk的包含属性：**  

>- action:(行为)
>   - create：文档不存在时创建
>   - update:更新文档
>   - index:创建新文档或替换已有文档
>   - delete:删除一个文档

>- metadata：
>   - _index
>   - _type
>   - _id

**create 和index的区别 ：**  如果数据存在，使用create操作失败，会提示文档已经存在，使用index则可以成功执行。

**删除操作**
```http
{"delete":{"_index":"lib","_type":"user","_id":"1"}}
```

**批量添加:**
```http
POST /lib2/books/_bulk
{"index":{"_id":1}}
{"title":"Java","price":55}
{"index":{"_id":2}}
{"title":"Html5","price":45}
{"index":{"_id":3}}
{"title":"Php","price":35}
{"index":{"_id":4}}
{"title":"Python","price":50}
```

**批量获取:**
```http
GET /lib2/books/_mget
{
"ids": ["1","2","3","4"]
}
```

**删除：没有请求体**
```http
POST /lib2/books/_bulk
{"delete":{"_index":"lib2","_type":"books","_id":4}}
{"create":{"_index":"tt","_type":"ttt","_id":"100"}}
{"name":"lisi"}
{"index":{"_index":"tt","_type":"ttt"}}
{"name":"zhaosi"}
{"update":{"_index":"lib2","_type":"books","_id":"2"}}
{"doc":{"price":58}}
```

**bulk一次最大处理多少数据量:**

>  bulk会把将要处理的数据载入内存中，所以数据量是有限制的，最佳的数据量不是一个确定的数值，它取决于你的硬件，你的文档大小以及复杂性，你的索引以及搜索的负载.  

>  一般建议是1000-5000个文档，大小建议是5-15MB，默认不能超过100M，可以在es的配置文件（即$ES_HOME下的config下的elasticsearch.yml）中。


### 2.5版本控制

ElasticSearch采用了乐观锁来保证数据的一致性，也就是说，当用户对document进行操作时，并不需要对该document作加锁和解锁的操作，只需要指定要操作的版本即可。当版本号一致时，ElasticSearch会允许该操作顺利执行，而当版本号存在冲突时，ElasticSearch会提示冲突并抛出异常（VersionConflictEngineException异常）。

ElasticSearch的版本号的取值范围为1到2^63-1。

内部版本控制：使用的是_version


外部版本控制：elasticsearch在处理外部版本号时会与对内部版本号的处理有些不同。它不再是检查_version是否与请求中指定的数值_相同_,而是检查当前的_version是否比指定的数值小。如果请求成功，那么外部的版本号就会被存储到文档中的_version中。

为了保持_version与外部版本控制的数据一致
使用version_type=external

### 2.6 Mapping映射

​	es自动创建了index，type，以及type对应的mapping(dynamic mapping)。

- **查看es的mapping**

```http
GET /myindex/article/_mapping
```

- **映射：**  
   mapping定义了type中的每个字段的数据类型以及这些字段如何分词等相关属性

```http
{
  "myindex": {
    "mappings": {
      "article": {
        "properties": {
          "author_id": {
            "type": "long"
          },
          "content": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "post_date": {
            "type": "date"
          },
          "title": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

​	**创建索引的时候,可以预先定义字段的类型以及相关属性，这样就能够把日期字段处理成日期，把数字字段处理成数字，把字符串字段处理字符串值等。**

**动态映射：**  
	当ES在文档中碰到一个以前没见过的字段时，它会利用动态映射来决定该字段的类型，并自动地对该字段添加映射。  
可以通过dynamic设置来控制这一行为，它能够接受以下的选项：

```http
true：默认值。动态添加字段
false：忽略新字段
strict：如果碰到陌生字段，抛出异常
```

**创建索引：**

```http
POST /lib2
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "books":{
        "properties":{
            "title":{"type":"text"},
            "name":{"type":"text","index":false},
            "publish_date":{"type":"date","index":false},
            "price":{"type":"double"},
            "number":{"type":"integer"}
        }
      }
     }
}
```



### 2.7 数据类型

- **核心数据类型**（Core datatypes）

  - 数字型： long, integer, short, byte, double, float  

  - 日期型： date(默认不会执行分词)  
  - 布尔型： boolean  
  - 二进制型：binary  
  - 字符型： string，string类型包括 text(默认可以执行分词) 和 keyword  

 > ​	text类型被用来索引长文本，在建立索引前会将这些文本进行分词，转化为词的组合，建立索引。允许es来检索这些词语。text类型不能用来排序和聚合。  
> ​	Keyword类型不需要进行分词，可以被用来检索过滤、排序和聚合。keyword 类型字段只能用本身来进行检索。

- **复杂数据类型**（Complex datatypes）

    数组类型（Array datatype）：数组类型不需要专门指定数组元素的type，例如：
        字符型数组: [ "one", "two" ]
        整型数组：[ 1, 2 ]
        数组型数组：[ 1, [ 2, 3 ]] 等价于[ 1, 2, 3 ]
        对象数组：[ { "name": "Mary", "age": 12 }, { "name": "John", "age": 10 }]
    对象类型（Object datatype）：_ object _ 用于单个JSON对象；
    嵌套类型（Nested datatype）：_ nested _ 用于JSON数组；

- **地理位置类型**（Geo datatypes）

    地理坐标类型（Geo-point datatype）：_ geo_point _ 用于经纬度坐标；
    地理形状类型（Geo-Shape datatype）：_ geo_shape _ 用于类似于多边形的复杂形状；

- **特定类型**（Specialised datatypes）

    IPv4 类型（IPv4 datatype）：_ ip _ 用于IPv4 地址；
    Completion 类型（Completion datatype）：_ completion _提供自动补全建议；
    Token count 类型（Token count datatype）：_ token_count _ 用于统计做了标记的字段的index数目，该值会一直增加，不会因为过滤条件而减少。
    mapper-murmur3
    类型：通过插件，可以通过 _ murmur3 _ 来计算 index 的 hash 值；
    附加类型（Attachment datatype）：采用 mapper-attachments
    插件，可支持_ attachments _ 索引，例如 Microsoft Office 格式，Open Document 格式，ePub, HTML 等。

**mapping支持的属性有：**

- "store":false 
  - 是否单独设置此字段的是否存储而从_source字段中分离，默认是false，只能搜索，不能获取值。

- **"index": true** 
  - 分词，不分词是：false 。设置成false，字段将不会被索引。

- **"analyzer":"ik"** 
  - 指定分词器,默认分词器为standard analyzer.
- **"search_analyzer":"ik"** 
  - 设置搜索时的分词器，默认跟ananlyzer是一致的，比如index时用standard+ngram，搜索时用standard用来完成自动提示功能。

- "boost":1.23 
  - 字段级别的分数加权，默认值是1.0.

- "doc_values":false 
  - 对not_analyzed字段，默认都是开启，分词字段不能使用，对排序和聚合能提升较大性能，节约内存.

- "fielddata":{"format":"disabled"} 
  - 针对分词字段，参与排序或聚合时能提高性能，不分词字段统一建议使用doc_value。

- "fields":{"raw":{"type":"string","index":"not_analyzed"}} 
  - 可以对一个字段提供多种索引模式，同一个字段的值，一个分词，一个不分词。

- "include_in_all":ture 
  - 设置是否此字段包含在_all字段中，默认是true，除非index设置成no选项。

- "index_options":"docs"
  - 4个可选参数：
    - docs（索引文档号）--- 其他默认选择（除了分词的字段以外）
    - freqs（文档号+词频），
    - positions（文档号+词频+位置，通常用来距离查询）--- 分词默认选择
    - offsets（文档号+词频+位置+偏移量，通常被使用在高亮字段）

- "norms":{"enable":true,"loading":"lazy"} 
  - 分词字段默认配置，不分词字段：默认{"enable":false}，存储长度因子和索引时boost，建议对需要参与评分字段使用 ，会额外增加内存消耗量。

- "null_value":"NULL" 
  - 设置一些缺失字段的初始化值，只有string可以使用，分词字段的null值也会被分词

- "position_increament_gap":0 
  - 影响距离查询或近似查询，可以设置在多值字段的数据上火分词字段上，查询时可指定slop间隔，默认值是100。



### 2.7基本查询(Query查询)

#### 2.7.1数据准备
```http
PUT /lib3
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text"},
            "address": {"type":"text"},
            "age": {"type":"integer"},
            "interests": {"type":"text"},
            "birthday": {"type":"date"}
        }
      }
     }
}
```
```http
# 正常条件查询
GET /lib3/user/_search?q=name:lisi
# 正常条件查询，且结果按序排序。
GET /lib3/user/_search?q=name:zhaoliu&sort=age:desc
```

#### 2.7.2 term查询和terms查询

**注意：term query会去倒排索引中寻找确切的term，它并不知道分词器的存在(所以对查询条件不会进行分词查询)。这种查询适合keyword 、numeric、date。**

- **term**:查询某个字段里含有某个关键词的文档

```http
GET /lib3/user/_search/
{
  "query": {
      "term": {"interests": "changge"}
  }
}
```
- **terms**:查询某个字段里含有多个关键词的文档

```http
GET /lib3/user/_search
{
    "query":{
        "terms":{
            "interests": ["hejiu","changge"]
        }
    }
}
```

#### 2.7.3 控制查询返回的数量

>- from：从哪一个文档开始  
>- size：需要的个数

```http
GET /lib3/user/_search
{
    "from":0,
    "size":2,
    "query":{
        "terms":{
            "interests": ["hejiu","changge"]
        }
    }
}
```

#### 2.7.4 返回版本号: "version":true,

```http
GET /lib3/user/_search
{
    "version":true,
    "query":{
        "terms":{
            "interests": ["hejiu","changge"]
        }
    }
}
```

#### 2.7.5 match查询

**注意：match query知道分词器的存在(所以对查询条件会先进行分词，再进行查询)，会对filed进行分词操作，然后再查询。**

```http
GET /lib3/user/_search
{
    "query":{
        "match":{
            "name": "zhaoliu"
        }
    }
}
```

```http
GET /lib3/user/_search
{
    "query":{
        "match":{
            "age": 20
        }
    }
}
```

**match_all:查询所有文档**
```http
GET /lib3/user/_search
{
  "query": {
    "match_all": {}
  }
}
```

**multi_match:可以指定多个字段**
```http
GET /lib3/user/_search
{
    "query":{
        "multi_match": {
            "query": "changge",
            "fields": ["interests","name"]
         }
    }
}
```

**match_phrase:短语匹配查询**

> ​	ElasticSearch引擎首先分析（analyze）查询字符串，从分析后的文本中构建短语查询，这意味着**必须匹配短语中的所有分词，并且保证各个分词的相对位置不变，这样才能够查出数据**：  
>​	例如：必须 "interests": "duanlian，shuoxiangsheng" 这里面的 duanlian，shuoxiangsheng 完全一样才行。

```http
GET lib3/user/_search
{
  "query":{  
      "match_phrase":{  
         "interests": "duanlian，shuoxiangsheng"
      }
   }
}
```

#### 2.7.6 指定返回的字段
```http
GET /lib3/user/_search
{
    "_source": ["address","name"],
    "query": {
        "match": {
            "interests": "changge"
        }
    }
}
```

#### 2.7.7控制加载的字段

##### 1）基本字段设置

```http
GET /lib3/user/_search
{
    "query": {
        "match_all": {}
    },
    
    "_source": {
          "includes": ["name","address"],
          "excludes": ["age","birthday"]
      }
}
```

##### 2）使用通配符设置字段
```http
GET /lib3/user/_search
{
    "_source": {
          "includes": "addr*",
          "excludes": ["name","bir*"] 
    },
    "query": {
        "match_all": {}
    }
}
```

#### 2.7.8 排序

> 使用sort实现排序： **desc:降序，asc升序**

```http
GET /lib3/user/_search
{
    "query": {
        "match_all": {}
    },
    "sort": [
        {
           "age": {
               "order":"asc"
           }
        }
    ]
}
```

```http
GET /lib3/user/_search
{
    "query": {
        "match_all": {}
    },
    "sort": [
        {
           "age": {
               "order":"desc"
           }
        }
    ]
        
}
```

#### 2.7.9 前缀匹配查询

```http
GET /lib3/user/_search
{
  "query": {
    "match_phrase_prefix": {
        "name": {
            "query": "zhao"
        }
    }
  }
}
```

#### 2.7.10 范围查询---range

**range:实现范围查询**  

参数包含：
> from , to , include_lower , include_upper , boost  
>
> include_lower:是否包含范围的左边界，默认是true  
>
> include_upper:是否包含范围的右边界，默认是true  

```http
GET /lib3/user/_search
{
    "query": {
        "range": {
            "birthday": {
                "from": "1990-10-10",
                "to": "2018-05-01"
            }
        }
    }
}
```

```http
GET /lib3/user/_search
{
    "query": {
        "range": {
            "age": {
                "from": 20,
                "to": 25,
                "include_lower": true,
                "include_upper": false
            }
        }
    }
}
```

#### 2.7.11 wildcard查询---允许使用通配符* 和 ?来进行查询

- ***代表0个或多个字符**

- **？代表任意一个字符**

```http
GET /lib3/user/_search
{
    "query": {
        "wildcard": {
             "name": "zhao*"
        }
    }
}
```

```http
GET /lib3/user/_search
{
    "query": {
        "wildcard": {
             "name": "li?i"
        }
    }
}
```


#### 2.7.12 模糊查询---fuzzy

**value：查询的关键字**

**boost：查询的权值，默认值是1.0**

>- min_similarity:设置匹配的最小相似度，默认值为0.5，对于字符串，取值为0-1(包括0和1);对于数值，取值可能大于1;对于日期型取值为1d,1m等，1d就代表1天
>
>- prefix_length:指明区分词项的共同前缀长度，默认是0
>
>- max_expansions:查询中的词项可以扩展的数目，默认可以无限大

```http
GET /lib3/user/_search
{
    "query": {
        "fuzzy": {
             "interests": "chagge"
        }
    }
}
```

```http
GET /lib3/user/_search
{
    "query": {
        "fuzzy": {
             "interests": {
                 "value": "chagge"
             }
        }
    }
}
```

#### 2.7.13 高亮搜索结果---指定查询结果的某个字段高亮

```http
GET /lib3/user/_search
{
    "query":{
        "match":{
            "interests": "changge"
        }
    },
    "highlight": {
        "fields": {
             "interests": {}
        }
    }
}
```

### 2.8 Filter查询
**特点：**
	filter是不计算相关性的，同时可以cache。因此，filter速度要快于query。

添加数据：
```http
POST /lib4/items/_bulk
{"index": {"_id": 1}}
{"price": 40,"itemID": "ID100123"}
{"index": {"_id": 2}}
{"price": 50,"itemID": "ID100124"}
{"index": {"_id": 3}}
{"price": 25,"itemID": "ID100124"}
{"index": {"_id": 4}}
{"price": 30,"itemID": "ID100125"}
{"index": {"_id": 5}}
{"price": null,"itemID": "ID100127"}
```

####2.8.1 简单的过滤查询
```http
GET /lib4/items/_search
{ 
       "post_filter": {
             "term": {
                 "price": 40
             }
       }
}
```

```http
GET /lib4/items/_search
{
      "post_filter": {
          "terms": {
                 "price": [25,40]
              }
        }
}
```
```http
# itemID被默认设置为text类型，被分词了，所以查询不到
GET /lib4/items/_search
{
    "post_filter": {
        "term": {
            "itemID": "ID100123"
          }
      }
}
```

#### 2.8.2 过滤查询---bool

**特点：**可以实现组合过滤查询

**参数：** 

- must:必须满足的条件---and

- should：可以满足也可以不满足的条件--or

- must_not:不需要满足的条件--not

**格式：**

```http
{
    "bool": {
        "must": [],
        "should": [],
        "must_not": []
    }
}
```

**例如**：

```http
GET /lib4/items/_search
{
    "post_filter": {
          "bool": {
               "should": [
                    {"term": {"price":25}},
                    {"term": {"itemID": "id100123"}}
                  ],
                "must_not": {
                    "term":{"price": 30}
                   }
                }
             }
}
```

##### 嵌套使用bool：

```http
GET /lib4/items/_search
{
    "post_filter": {
          "bool": {
                "should": [
                    {"term": {"itemID": "id100123"}},
                    {
                      "bool": {
                          "must": [
                              {"term": {"itemID": "id100124"}},
                              {"term": {"price": 40}}
                            ]
                          }
                    }
                  ]
                }
            }
}
```

#### 2.8.3 范围过滤
单词简介：

>- gt: >
>
>- lt: <
>
>- gte: >=
>
>- lte: <=

例如：
```http
GET /lib4/items/_search
{
     "post_filter": {
          "range": {
              "price": {
                   "gt": 25,
                   "lt": 50
                }
            }
      }
}
```

#### 2.8.5 过滤非空---exists

例如：
```http
# 查询出价格不为空的所有数据。--exists 关键字
GET /lib4/items/_search
{
  "query": {
    "bool": {
      "filter": {
          "exists":{
             "field":"price"
         }
      }
    }
  }
}
```

```http
GET /lib4/items/_search
{
    "query" : {
        "constant_score" : {
            "filter": {
                "exists" : { "field" : "price" }
            }
        }
    }
}
```

#### 2.8.6 过滤器缓存---filter cache

​	ElasticSearch提供了一种特殊的缓存，即过滤器缓存（filter cache），用来存储过滤器的结果，被缓存的过滤器并不需要消耗过多的内存（因为它们只存储了哪些文档能与过滤器相匹配的相关信息），而且可供后续所有与之相关的查询重复使用，从而极大地提高了查询性能。

- **以下过滤器默认不缓存**：
  - numeric_range
  - script
  - geo_bbox
  - geo_distance
  - geo_distance_range
  - geo_polygon
  - geo_shape
  - and
  - or
  - not

- **默认是开启缓存的:**
  - exists
  - missing
  - range
  - term
  - terms

##### 开启缓存

​	在filter查询语句后边加上      **=="_catch":true==**

### 2.9 聚合查询

#### 2.9.1 sum---求价格的总和

```http
GET /lib4/items/_search
{
  "size":0,
  "aggs": {
     "price_of_sum": {
         "sum": {
           "field": "price"
         }
     }
  }
}
```

#### 2.9.2 min---求商品最小值

```http
GET /lib4/items/_search
{
  "size": 0, 
  "aggs": {
     "price_of_min": {
         "min": {
           "field": "price"
         }
     }
  }
}
```

#### 2.9.3 max---求商品最大值

```http
GET /lib4/items/_search
{
  "size": 0, 
  "aggs": {
     "price_of_max": {
         "max": {
           "field": "price"
         }
     }
  }
}
```

#### 2.9.4 avg---求商品平均值

```http
GET /lib4/items/_search
{
  "size":0,
  "aggs": {
     "price_of_avg": {
         "avg": {
           "field": "price"
         }
     }
  }
}
```

#### 2.9.5 cardinality:求基数

```http
GET /lib4/items/_search
{
  "size":0,
  "aggs": {
     "price_of_cardi": {
         "cardinality": {
           "field": "price"
         }
     }
  }
}
```

#### 2.9.6 terms:分组

- 基本分组

```http
GET /lib4/items/_search
{
  "size":0,
  "aggs": {
     "price_group_by": {
         "terms": {
           "field": "price"
         }
     }
  }
}
```

- 复杂分组

```http
GET /lib3/user/_search
{
  "query": {
      "match": {
        "interests": "changge"
      }
   },
   "size": 0, 
   "aggs":{
       "age_group_by":{
           "terms": {
             "field": "age",
             "order": {
               "avg_of_age": "desc"
             }
           },
           "aggs": {
             "avg_of_age": {
               "avg": {
                 "field": "age"
               }
             }
           }
       }
   }
}
```

### 2.10 复合查询
​	将多个基本查询组合成单一查询的查询。

#### 2.10.1 使用bool查询

**接收以下参数：**

>- must：
>    文档 必须匹配这些条件才能被包含进来。 

>- must_not：
    文档 必须不匹配这些条件才能被包含进来。 

>- should：
    如果满足这些语句中的任意语句，将增加 _score，否则，无任何影响。它们主要用于修正每个文档的相关性得分。 

>- filter：
    必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

**注意 ：**   相关性得分是如何组合的。每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， bool 查询就将这些得分进行合并并且返回一个代表整个bool操作的得分。

例如：  
下面的查询用于查找 title 字段匹配 how to make millions 并且不被标识为 spam 的文档。那些被标识为 starred 或在2014之后的文档，将比另外那些文档拥有更高的排名。如果 _两者_ 都满足，那么它排名将更高：

```http
{
    "bool": {
        "must": { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

​	**如果没有 must 语句，那么至少需要能够匹配其中的一条 should 语句。但，如果存在至少一条 must 语句，则对 should 语句的匹配没有要求。** 
如果我们不想因为文档的时间而影响得分，可以用 filter 语句来重写前面的例子：

```http
{
    "bool": {
        "must": { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }} 
        }
    }
}
```

通过将 range 查询移到 filter 语句中，我们将它转成不评分的查询，将不再影响文档的相关性排名。由于它现在是一个不评分的查询，可以使用各种对 filter 查询有效的优化手段来提升性能。

bool 查询本身也可以被用做不评分的查询。简单地将它放置到 filter 语句中并在内部构建布尔逻辑：

```http
{
    "bool": {
        "must": { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": { 
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```

#### 2.10.2 constant_score查询

它将一个不变的常量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 filter 而没有其它查询（例如，评分查询）的情况下。
```http
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } 
        }
    }
}
```

term 查询被放置在 constant_score 中，转成不评分的filter。这种方式可以用来取代只有 filter 语句的 bool 查询。 

## 第三节 ElasticSearch原理

### 3.1 分片和副本机制

- index包含多个shard(分片) 。

- 每个shard都是一个最小工作单元，承载部分数据；每个shard都是一个lucene实例，有完整的建立索引和处理请求的能力

- 增减节点时，shard会自动在nodes中负载均衡

- primary shard(主分片)和replica shard(副分片)，每个document肯定只存在于某一个primary shard以及其对应的replica shard中，不可能存在于多个primary shard

- replica shard是primary shard的副本，负责容错，以及承担读请求负载，，用于作为主分片的副节点，当主分片宕机，副分片自动接替当前工作。一个主分片至少有一个副分片。

- primary shard的数量在创建索引的时候就固定了，replica shard的数量可以随时修改

- primary shard的默认数量是5，replica默认是1，默认有10个shard，5个primary shard，5个replica shard

- primary shard不能和自己的replica shard放在同一个节点上（否则节点宕机，primary shard和副本都丢失，起不到容错的作用），但是可以和其他primary shard的replica shard放在同一个节点上


### 3.2 单节点环境下创建索引分析
```http
PUT /myindex
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```
**注意：**   
>- 这个时候，只会将3个primary shard分配到仅有的一个node上去。  
>
>- 另外3个replica shard是无法分配的（一个shard的副本replica，他们两个是不能在同一个节点的）。
>
>- 集群可以正常工作，但是一旦出现节点宕机，数据全部丢失，而且集群不可用，无法接收任何请求。


### 3.3 两个节点环境下创建索引分析

>- 将3个primary shard分配到一个node上去，另外3个replica shard分配到另一个节点上。
>
>- primary shard 和replica shard 保持同步。
>
>- primary shard 和replica shard 都可以处理客户端的读请求。


### 3.4 水平扩容的过程

1.  扩容后primary shard和replica shard会自动的负载均衡。

2.  扩容后每个节点上的shard会减少，那么分配给每个shard的CPU，内存，IO资源会更多，性能提高。

3.  扩容的极限，如果有6个shard，扩容的极限就是6个节点，每个节点上一个shard，如果想超出扩容的极限，比如说扩容到9个节点，那么可以增加replica shard的个数。

4.  6个shard，3个节点，最多能承受几个节点所在的服务器宕机？(容错性)
任何一台服务器宕机都会丢失部分数据。

为了提高容错性，增加shard的个数：
9个shard，(3个primary shard，6个replicashard)，这样就能容忍最多两台服务器宕机了。

总结：扩容是为了提高系统的吞吐量，同时也要考虑容错性，也就是让尽可能多的服务器宕机还能保证数据不丢失。


### 3.5 ElasticSearch的容错机制

**以9个shard，3个节点(一主两从)为例：**  
如果master node 宕机，此时不是所有的primary shard都是Active status，所以此时的集群状态是red。

**容错处理的第一步:** 是选举一台服务器作为master   

**容错处理的第二步:** 新选举出的master会把挂掉的primary shard的某个replica shard 提升为primary   shard,此时集群的状态为yellow，因为少了一个replica shard，并不是所有的replica shard都是active status

**容错处理的第三步：** 重启故障机，新master会把所有的副本都复制一份到该节点上，（同步一下宕机后发生的修改），此时集群的状态为green，因为所有的primary shard和replica shard都是Active status

### 3.6 文档的核心元数据

#### 1._index:

说明了一个文档存储在哪个索引中

同一个索引下存放的是相似的文档(文档的field多数是相同的)

索引名必须是小写的，不能以下划线开头，不能包括逗号

#### 2._type:

表示文档属于索引中的哪个类型

一个索引下只能有一个type

类型名可以是大写也可以是小写的，不能以下划线开头，不能包括逗号

#### 3._id:

文档的唯一标识，和索引，类型组合在一起唯一标识了一个文档

可以手动指定值，也可以由es来生成这个值

### 3.7 文档id生成方式

#### 1.手动指定

  put /index/type/66

  通常是把其它系统的已有数据导入到es时

#### 2.由es生成id值

  post /index/type

 es生成的id长度为20个字符，使用的是base64编码，URL安全，使用的是GUID算法，分布式下并发生成id值时不会冲突


### 3.8 _source元数据分析

**简介：** 就是我们在添加文档时request body中的内容(数据字段)

**例如：** 用于指定返回的结果中含有哪些字段：

```
get /index/type/1?_source=name
```

### 3.9 改变文档内容---原理解析

##### 替换方式：

```http
PUT /lib/user/4
{ 	"first_name" : "Jane",
	"last_name" :   "Lucy",
	"age" :         24,
	"about" :       "I like to collect rock albums",
	"interests":  [ "music" ]
}
```

##### 修改方式(partial update)：

```http
POST /lib/user/2/_update
{
    "doc":{
       "age":26
     }
}
```

#### 删除文档：
标记为deleted，随着数据量的增加，es会选择合适的时间删除掉。


### 3.10 groovy执行partial update

​	es有内置的脚本支持，可以基于groovy脚本实现复杂的操作  。

**例如：**  

- 修改年龄

```http
POST /lib/user/4/_update
{
  "script": "ctx._source.age+=1"
}
```

- 修改名字

```http
POST /lib/user/4/_update
{
  "script": "ctx._source.last_name+='hehe'"
}
```

- 添加爱好

```http
POST /lib/user/4/_update
{
  "script": {
    "source": "ctx._source.interests.add(params.tag)",
    "params": {
      "tag":"picture"
    }
  }
}
```

- 删除爱好

```http
POST /lib/user/4/_update
{
  "script": {
    "source": "ctx._source.interests.remove(ctx._source.interests.indexOf(params.tag))",
    "params": {
      "tag":"picture"
    }
  }
}
```

- 删除文档

```http
POST /lib/user/4/_update
{
  "script": {
    "source": "ctx.op=ctx._source.age==params.count?'delete':'none'",
    "params": {
        "count":29
    }
  }
}
```

- upsert：当文档不存在，则创建文档，存在的话，就执行 "ctx._source.age += 1",

```http
POST /lib/user/4/_update
{
  "script": "ctx._source.age += 1",
  
  "upsert": {
     "first_name" : "Jane",
     "last_name" :   "Lucy",
     "age" :  20,
     "about" :       "I like to collect rock albums",
     "interests":  [ "music" ]
  }
}
```

### 3.11 并发处理--version

**方法：** 使用乐观锁:_version

**参数：** retry_on_conflict --- 当发生冲突以后，重新获取文档数据和版本信息进行更新，不断的操作，最多操作的次数就是retry_on_conflict的值。
```
POST /lib/user/4/_update?retry_on_conflict=3&version=5
```


### 3.12 文档数据路由原理解析

#### 1.文档路由到分片

 	一个索引由多个分片构成，当添加(删除，修改)一个文档时，es就需要决定这个文档存储在哪个分片上，这个过程就称为数据路由(routing)

#### 2.路由算法：

```java
 shard=hash(routing) % number_of_pirmary_shards
```

**示例：**一个索引，3个primary shard(也就是一个索引分了三个分片)。

- 第一步：每次增删改查时，都有一个routing值，默认是文档的_id的值。

- 第二步：对这个routing值使用哈希函数进行计算。

- 第三步：计算出的值再和主分片个数取余数。余数肯定在0 ---（分片个数-1）之间，文档就在对应的shard上。

  **routing值默认是文档的_id的值**，也可以手动指定一个值，手动指定对于负载均衡以及提高批量读取的性能都有帮助。 

3. **primary shard(分片个数)个数一旦确定就不能修改。**


### 3.13 文档增删改内部原理

- 第一步：发送增删改请求时，可以选择任意一个节点，该节点就成了**协调节点**(coordinating node)。  

- 第二步：协调节点使用路由算法进行路由，然后将请求转到primary shard所在节点，该节点处理请求，并把数据同步到它的replica shard。  

- 第三步：最后由协调节点对客户端做出响应。  


### 3.14 一致性原理---quorum

#### **1、consistency**参数 

##### 1）基本介绍：

​	用于在任何增删改的操作中。参数作用于：

- one: (primary shard)只要有一个primary shard是活跃的就可以执行
- all: (all shard)所有的primary shard和replica shard都是活跃的才能执行
- quorum: (default) 默认值，大部分shard是活跃的才能执行（例如共有6个shard，至少有3个shard是活跃的才能执行写操作）  

##### 2）使用方式：  

```http
PUT/lib/user/3?consistency=one
```

#### **2.quorum** 机制：

##### 1）基本介绍：

必须多数shard都是活跃的才能执行。

##### 2）使用公式：

```java
int((primary+number_of_replica)/2)+1
```

​	例如：3个primary shard，1个replica。
```
int((3+1)/2)+1=3
```
所以，至少3个shard是活跃的。


> 注意：可能出现shard不能分配齐全的情况。

> 比如：1个primary shard,1个replica
> int((1+1)/2)+1=2
> 但是如果只有一个节点，因为primary shard和replica shard不能在同一个节点上，所以仍然不能执行写操作

> 再举例：1个primary shard,3个replica,2个节点
> int((1+3)/2)+1=3

> 最后:当活跃的shard的个数没有达到要求时，  
> es默认会等待一分钟，如果在等待的期间活跃的shard的个数没有增加，则显示timeout

> 修改超时等待时间：  
> put /index/type/id?timeout=60s


### 3.15 文档查询内部原理

- 第一步：查询请求发给任意一个节点，该节点就成了coordinating node，该节点使用路由算法算出文档所在的primary shard

- 第二步：协调节点把请求转发给primary shard也可以转发给replica shard(使用轮询调度算法(Round-Robin Scheduling，把请求平均分配至primary shard 和replica shard)

- 第三步：处理请求的节点把结果返回给协调节点，协调节点再返回给应用程序

**特殊情况：** 请求的文档还在建立索引的过程中，primary shard上存在，但replica shar上不存在，但是请求被转发到了replica shard上，这时就会提示找不到文档。

### 3.16 查询结果分析
```http
{
  "took": 419,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "lib3",
        "_type": "user",
        "_id": "3",
        "_score": 0.6931472,
        "_source": {
          "address": "bei jing hai dian qu qing he zhen",
          "name": "lisi"
        }
      },
      {
        "_index": "lib3",
        "_type": "user",
        "_id": "2",
        "_score": 0.47000363,
        "_source": {
          "address": "bei jing hai dian qu qing he zhen",
          "name": "zhaoming"
        }
      }
```

- took：查询耗费的时间，单位是毫秒

- _shards：共请求了多少个shard

- total：查询出的文档总个数

- max_score： 本次查询中，相关度分数的最大值，文档和此次查询的匹配度越高，_score的值越大，排位越靠前

- hits：默认查询前10个文档

- timed_out：

```http
GET /lib3/user/_search?timeout=10ms
{
    "_source": ["address","name"],
    "query": {
        "match": {
            "interests": "changge"
        }
    }
}
```

### 3.19 多index，多type查询模式

- GET _search  --- 查看所有文档

- GET /lib/_search --- 查询lib索引下的所有文档

- GET /lib,lib3/_search --- 查询lib、lib3索引下的所有文档

- GET /*3,*4/_search

- GET /lib/user/_search

- GET /lib,lib4/user,items/_search

- GET /_all/_search  ---查看所有索引

- GET /_all/user,items/_search

### 3.20 分页查询中的deep paging问题


```http
GET /lib3/user/_search
{
    "from":0,
    "size":2,
    "query":{
        "terms":{
            "interests": ["hejiu","changge"]
        }
    }
}
```


GET /_search?from=0&size=3

**deep paging:** 意思是--查询的很深，比如一个索引有三个primary shard， 分别存储了6000条数据，我们要得到第100页的数据(每页10条)，类似这种情况就叫deep paging。

**比如：如何得到第100页的10条数据？**
> 在每个shard中搜索990到999这10条数据，然后用这30条数据排序，排序之后取10条数据就是要搜索的数据，这种做法是错的，因为3个shard中的数据的_score分数不一样，可能这某一个shard中第一条数据的_score分数比另一个shard中第1000条都要高，所以在每个shard中搜索990到999这10条数据然后排序的做法是不正确的。  
>
> 正确的做法是每个shard把0到999条数据全部搜索出来（按排序顺序），然后全部返回给协调节点(coordinate node)，由coordinate node按_score分数排序后，取出第100页的10条数据，然后返回给客户端。


**deep paging性能问题**

- 耗费网络带宽，因为搜索过深的话，各shard要把数据传送给coordinate node，这个过程是有大量数据传递的，消耗网络，

- 消耗内存，各shard要把数据传送给coordinate node，这个传递回来的数据，是被coordinate node保存在内存中的，这样会大量消耗内存。

- 消耗cpu coordinate node要把传回来的数据进行排序，这个排序过程很消耗cpu.

**鉴于deep paging的性能问题，所以应尽量减少使用。**


### 3.21 query string查询及copy_to解析

GET /lib3/user/_search?q=interests:changge

GET /lib3/user/_search?q=+interests:changge

GET /lib3/user/_search?q=-interests:changge

GET /lib3/user/_search?q=-name:zsl  //查找所有文档中name为zsl的文档。

GET /lib3/user/_search?q=changge  //当没有指定field时，就会从所有字段中或者copy_to字段中查询

**copy_to字段：** 是把其它字段中的值，以空格为分隔符组成一个大字符串，然后被分析和索引，但是不存储，也就是说它能被查询，但不能被取回显示。

**注意:** copy_to指向的字段字段类型要为：text


### 3.22字符串排序问题

**问题：** 对一个字符串类型的字段进行排序通常不准确，因为已经被分词成多个词条了。

**解决方式：** 对字段索引两次，一次索引分词（用于搜索），一次索引不分词(用于排序)。

```http
GET /lib3/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "interests": {
        "order": "desc"
      }
    }
  ]
}
```
```http
GET /lib3/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "interests.raw": {
        "order": "asc"
      }
    }
  ]
}
```
```http
DELETE lib3
```

```http
PUT /lib3
{
    "settings":{
        "number_of_shards" : 3,
        "number_of_replicas" : 0
      },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text"},
            "address": {"type":"text"},
            "age": {"type":"integer"},
            "birthday": {"type":"date"},
            "interests": {
                "type":"text",
                "fields": {
                  "raw":{
                     "type": "keyword"
                   }
                },
                "fielddata": true
             }
          }
        }
     }
}
```

### 3.23 计算相关度分数
**使用方法：**
使用的是TF/IDF算法(Term Frequency&Inverse Document Frequency)

**特点一、** Term Frequency:我们查询的文本中的词条在document本中出现了多少次，出现次数越多，相关度越高

> 搜索内容： hello world
> 
> Hello，I love china.
> 
> Hello world,how are you!

**特点二、** Inverse Document Frequency：我们查询的文本中的词条在索引的所有文档中出现了多少次，出现的次数越多，相关度越低

> 搜索内容：hello world
> 
> hello，what are you doing?
> 
> I like the world.

hello 在索引的所有文档中出现了500次，world出现了100次

**特点三、** Field-length(字段长度归约) norm:field越长，相关度越低

> 搜索内容：hello world
> 
> {"title":"hello,what's your name?","content":{"owieurowieuolsdjflk"}}
> 
> {"title":"hi,good morning","content":{"lkjkljkj.......world"}}


**查看分数是如何计算的：**  
```http
GET /lib3/user/_search?explain=true
{
    "query":{
        "match":{
            "interests": "duanlian,changge"
        }
    }
}
```

**_explain ：** 查看一个文档能否匹配上某个查询：
```http
GET /lib3/user/2/_explain
{
    "query":{
        "match":{
            "interests": "duanlian,changge"
        }
    }
}
```

### 3.24 Doc Values 解析

DocValues其实是Lucene在构建倒排索引时，会额外建立一个有序的正排索引(基于document => field value的映射列表)


```
{"birthday":"1985-11-11",age:23}
{"birthday":"1989-11-11",age:29}

document     age       birthday

doc1         23         1985-11-11
doc2         29         1989-11-11
```

**特点：**
> 存储在磁盘上，节省内存 
>
> 对排序，分组和一些聚合操作能够大大提升性能 

**注意：** 默认对不分词的字段是开启的，对分词字段无效（需要把fielddata设置为true）
```
PUT /lib3
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text"},
            "address": {"type":"text"},
            "age": {
              "type":"integer",
              "doc_values":false
            },
            "interests": {"type":"text"},
            "birthday": {"type":"date"}
        }
      }
     }
}
```

### 3.25 滚动搜索大量数据---scroll
**简介：**
如果一次性要查出来比如10万条数据，那么性能会很差，此时一般会采取用scoll滚动查询，一批一批的查，直到所有数据都查询完为止。

**特点：**

>- 1.scoll搜索会在第一次搜索的时候，保存一个当时的视图快照，之后只会基于该旧的视图快照提供数据搜索，如果这个期间数据变更，是不会让用户看到的。
>
>- 2.采用基于_doc(不使用_score)进行排序的方式，性能较高。
>
>- 3.每次发送scroll请求，我们还需要指定一个scoll参数，指定一个时间窗口，每次搜索请求只要在这个时间窗口内能完成就可以了。

```http
# 查询完以后会保存快照
GET /lib3/user/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort":["_doc"],
  "size":3
}
```

```http
# 接着上面的快照后面继续查询
GET /_search/scroll
{
   "scroll": "1m",
   "scroll_id": "DnF1ZXJ5VGhlbkZldGNoAwAAAAAAAAAdFkEwRENOVTdnUUJPWVZUd1p2WE5hV2cAAAAAAAAAHhZBMERDTlU3Z1FCT1lWVHdadlhOYVdnAAAAAAAAAB8WQTBEQ05VN2dRQk9ZVlR3WnZYTmFXZw=="
}
```

### 3.26 dynamic mapping策略

#### 1）dynamic：的取值范围：

- true:遇到陌生字段就 dynamic mapping
- false:遇到陌生字段就忽略

- strict:约到陌生字段就报错

```http
PUT /lib8
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "dynamic":strict,
        "properties":{
            "name": {"type":"text"},
            "address":{
                "type":"object",
                "dynamic":true
            },
        }
      }
     }
}
```

#### 2）日期格式--date_detection:  

​	默认会按照一定格式识别date，比如yyyy-MM-dd。

#### 3）关闭date_detection:

使用方式 ： "date_detection": false,

```http
PUT /lib8
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "date_detection": false,
        }
    }
}
```

**定制 dynamic mapping template(type)**
```http
PUT /my_index
{ 
  "mappings": { 
    "my_type": { 
      "dynamic_templates": [ 
        { 
          "en": { 
            "match": "*_en", 
            "match_mapping_type": "string", 
            "mapping": { 
              "type": "text", 
              "analyzer": "english" 
            } 
          } 
        } 
      ] 
     } 
  } 
}
```

### 3.27重建索引
**需求：**
一个field的设置是不能修改的，如果要修改一个field，那么应该重新按照新的mapping，建立一个index，然后将数据批量查询出来，重新用bulk api写入到index中。

**解决办法：**
批量查询的时候，建议采用scroll api，并且采用多线程并发的方式来reindex数据，每次scroll就查询指定日期的一段数据，交给一个线程即可。

```http
PUT /index1/type1/4
{
   "content":"1990-12-12"
}
```
```http
GET /index1/type1/_search

GET /index1/type1/_mapping
```


> **错误的操作：数据类型不一致，会导致报错。例如：**
```
PUT /index1/type1/4
{
   "content":"I am very happy."
}
```

> **修改content的类型为string类型,报错，不允许修改。例如：**
```
PUT /index1/_mapping/type1
{
  "properties": {
    "content":{
      "type": "text"
    }
  }
}
```

##### **正确修改索引类型：**  

- **第一步：** 创建一个新的索引，把index1索引中的数据查询出来导入到新的索引中。  
  但是应用程序使用的是之前的索引，为了不用重启应用程序，给index1这个索引起个#别名。

```http
PUT /index1/_alias/index2
```

- **第二步：** 创建新的索引，把content的类型改为字符串。

```http
PUT /newindex
{
  "mappings": {
    "type1":{
      "properties": {
        "content":{
          "type": "text"
        }
      }
    }
  }
}
```

- **第三步：** 使用scroll批量查询

```http
GET /index1/type1/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": ["_doc"],
  "size": 2
}
```

- **第四步：** 使用bulk批量写入新的索引

```http
POST /_bulk
{"index":{"_index":"newindex","_type":"type1","_id":1}}
{"content":"1982-12-12"}
```

- **第五步：** 将别名index2和新的索引关联，应用程序不用重启

```http
POST /_aliases
{
  "actions": [
    {"remove": {"index":"index1","alias":"index2"}},
    {"add": {"index": "newindex","alias": "index2"}}
]
}
```
- **第六步：** 查看对应的索引

```
GET index2/type1/_search
```

### 3.28 索引不可变的原因

**倒排索引包括：**
文档的列表，文档的数量，词条在每个文档中出现的次数，出现的位置，每个文档的长度，所有文档的平均长度



## 第四节 在Java应用中访问ElasticSearch

### 4.1在Java应用中实现查询文档

pom中加入ElasticSearch6.2.4的依赖：

<dependencies>
​    <dependency>
​      <groupId>org.elasticsearch.client</groupId>
​      <artifactId>transport</artifactId>
​      <version>6.2.4</version>
​    </dependency>    
​    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>

  </dependencies> 

  <build>
​      <plugins>
​			<!-- java编译插件 -->
​			<plugin>
​				<groupId>org.apache.maven.plugins</groupId>
​				<artifactId>maven-compiler-plugin</artifactId>
​				<version>3.2</version>
​				<configuration>
​					<source>1.8</source>
​					<target>1.8</target>
​					<encoding>UTF-8</encoding>
​				</configuration>
​			</plugin>
​		</plugins>
  </build>  

### 4.2 在Java应用中实现添加文档

需要添加的数据为：  
```
"{" +
    "\"id\":\"1\"," +
    "\"title\":\"Java设计模式之装饰模式\"," +
    "\"content\":\"在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能。\"," +
    "\"postdate\":\"2018-05-20\"," +
    "\"url\":\"csdn.net/79239072\"" +
"}"
```
Java操作的代码为：  

第一步：在kibana中创建这种类型的mapping。
```
PUT /index2
{
  "settings":{
    "number_of_shards": 3,
    "number_of_replicas": 0
  },
  "mappings": {
        "blog": {
            "properties": {
                "id": {
                  "type": "long"
                },
                "title":{
                  "type": "text",
                  "analyzer": "ik_max_word"
                },
                "content":{
                  "type": "text",
                  "analyzer": "ik_max_word"
                },
                "postdate":{
                  "type": "date"
                },
                "url":{
                  "type": "text"
                }
            }
      }
  }
}
```

第二步：添加数据：
```
 XContentBuilder doc1 = XContentFactory.jsonBuilder()
                    .startObject()
                    .field("id","3")
                    .field("title","Java设计模式之单例模式")
                    .field("content","枚举单例模式可以防反射攻击。")
                    .field("postdate","2018-02-03")
                    .field("url","csdn.net/79247746")
                    .endObject();
        
IndexResponse response = client.prepareIndex("index1", "blog", null)
        .setSource(doc1)
        .get();
        
System.out.println(response.status());
```

### 4.3在Java应用中实现删除文档
```
DeleteResponse response=client.prepareDelete("index1","blog","SzYJjWMBjSAutsuLRP_P").get();

//删除成功返回OK，否则返回NOT_FOUND

System.out.println(response.status());
```

### 4.4在Java应用中实现更新文档

```
 UpdateRequest request=new UpdateRequest();
        request.index("index1")
                .type("blog")
                .id("2")
                .doc(
                		XContentFactory.jsonBuilder().startObject()
                        .field("title","单例模式解读")
                        .endObject()
                );
UpdateResponse response=client.update(request).get();

//更新成功返回OK，否则返回NOT_FOUND

System.out.println(response.status());
```

upsert方式：
```
IndexRequest request1 =new IndexRequest("index1","blog","3")
                .source(
                		XContentFactory.jsonBuilder().startObject()
                                .field("id","3")
                                .field("title","装饰模式")
                                .field("content","动态地扩展一个对象的功能")
                                .field("postdate","2018-05-23")
                                .field("url","csdn.net/79239072")
                                .endObject()
                );
        UpdateRequest request2=new UpdateRequest("index1","blog","3")
                .doc(
                		XContentFactory.jsonBuilder().startObject()
                        .field("title","装饰模式解读")
                        .endObject()
                ).upsert(request1);
        
UpdateResponse response=client.update(request2).get();
        
//upsert操作成功返回OK，否则返回NOT_FOUND

System.out.println(response.status());
```

### 4.5在Java应用中实现批量操作
```java
 MultiGetResponse mgResponse = client.prepareMultiGet()
	                .add("index1","blog","3","2")
	                .add("lib3","user","1","2","3")
	                .get();
		    
for(MultiGetItemResponse response:mgResponse){
	            GetResponse rp=response.getResponse();
	            if(rp!=null && rp.isExists()){
	                System.out.println(rp.getSourceAsString());
	            }
	        }
```

bulk批量执行：
```java
BulkRequestBuilder bulkRequest = client.prepareBulk();

bulkRequest.add(client.prepareIndex("lib2", "books", "4")
                .setSource(XContentFactory.jsonBuilder()
                        .startObject()
                        .field("title", "python")
                        .field("price", 68)
                        .endObject()
                )
        );
bulkRequest.add(client.prepareIndex("lib2", "books", "5")
                .setSource(XContentFactory.jsonBuilder()
                        .startObject()
                        .field("title", "VR")
                        .field("price", 38)
                        .endObject()
                )
        );
        //批量执行
BulkResponse bulkResponse = bulkRequest.get();
        
System.out.println(bulkResponse.status());
if (bulkResponse.hasFailures()) {
            
            System.out.println("存在失败操作");
        }
```



## 五、Spring Boot整合Elastic Search

### 1、Repository方式整合

#### 1）QueryBuilder查询条件封装

- **PrefixQueryBuilder**  ：前缀查询条件封装。

```java
	/**
	 * 根据字段，进行前缀查询
	 * @param field : 字段名称
	 * @param value : 字段查询的值
	 * @return
	 */
	public Iterable<Goods> searchPrefixQueryBuilder(String field, String value) {
		PrefixQueryBuilder prefixQueryBuilder = new PrefixQueryBuilder(field, value);
		Iterable<Goods> search = goodsRepository.search(prefixQueryBuilder);
	}
```

- **TermsQueryBuilder** ： 字段多 value 查询。

```java
	/**
	 * 单属性 多value查询
	 * @param field : 属性名称
	 * @param values : 属性需要匹配的value，可变参数能够传入数组
	 * @return
	 */
	public Iterable<Goods> searchTermsQueryBuilder(String field, String... values) {
		TermsQueryBuilder termsQueryBuilder = new TermsQueryBuilder(field, values);
		Iterable<Goods> search = goodsRepository.search(termsQueryBuilder);
		return search;
	}
```








































​              

































​    



























