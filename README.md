# es
## 安装
参考：<https://time.geekbang.org/course/detail/197-102661>

### 单机运行多个实例
```angular2html
bin/elasticsearch -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -d
bin/elasticsearch -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -d
bin/elasticsearch -E node.name=node3 -E cluster.name=geektime -E path.data=node3_data -d
```
-d 在linux下是守护进程
删除进程
```angular2html
ps|grep elasticsearch
kill pid
```

## 基础概念
0. es,，即类似于数据库
1. 索引:可以被视为优化的文档集合，即类似于表格
    - index 索引是文档的容器，是一类文档的结合
        index体现了逻辑空间的概念：每个索引都有自己的Mapping定义，用于定义包含的
        文档字段名和字段类型
    - shard 体现物理空间概念：索引中的数据分撒在shard上
    - 索引的Mapping和Setting 
        + Mapping定义文档字段类型
        + Setting定义不同数据的分布
2. 文档:es是面向文档的，文档是所有可搜索数据的最小单元。如：日志文件中的日志项，
2. 个电影的具体信息。文档会被序列为json，进行储存。每个文档都有ID，可以指定，也可以
3. 由ES生成
3. 字段:是包含数据的键值对，类似于mysql字段
<https://www.cnblogs.com/wwcom123/p/10410124.html>
4. 集群:
5. 节点:您可以将服务器（节点）添加到群集以增加容量，Elasticsearch会自动在所有可用节点上分配数据和查询负载。
6. 分片:实际上是一个自包含的索引，有两种类型的分片:原色和副本。索引中的每个文档都属于一个主分片。
副本分片是主分片的副本。副本提供数据的冗余副本，以防止硬件故障并增加服务读取请求（如搜索或检索文档）的容量。
<https://blog.csdn.net/qq_38486203/article/details/80077844>
说明1:旨在将平均分片大小保持在几GB到几十GB之间。对于具有基于时间的数据的用例，通常会看到20GB到40GB范围内的分片。
避免大量碎片问题。节点可以容纳的分片数量与可用堆空间成比例。作为一般规则，每GB堆空间的分片数应小于20。
确定用例的最佳配置的最佳方法是 使用您自己的数据和查询进行测试。

说明2:主分片和对应副本是被es严格禁止放在同一节点上的，所以只有一个节点是没有副本的
参考:<https://blog.csdn.net/wangmaohong0717/article/details/64906444>

7. 元数据：用于标注文档的相关信息
    - _index 文档所属索引名
    - _type 文档所属类型名
    - _id 文档唯一ID
    - _source 文档的原始json数据
    - _version 文档版本信息
    - _score 相关性打分
8. MYSQL vs ES
```
RDBMS           Elasticsearch
Table           Index(Type)
Row             Document
Column          Field
Schema          Mapping
SQL             DSL
```

9. B树索引和倒排索引
    - 正排索引：文档ID到文档内容和单词的关联，类似于目录页
    - 倒排索引：单词到文档ID的关系

## 安装
参考:<https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html>

### jvm配置
1. 修改JVM- config/jvm.options
    - 7.1下载默认值是1GB
2. 配置建议
    - Xmx和Xms设置成一样
    - Xmx不要超过机器内存的50%
    - 不要超过30GB - https://www.elastic.co/blog/a-heap-of-trouble
### 安装插件
```
bin/elasticsearch-plugin list #检查插件列表，如果是windows，则是elasticsearch-plugin.bat
bin/elasticsearch-plugin install analysis-icu #安装具体插件，如果无法安装，可以尝试换管理员角色安装
```
备注：查看插件安装<http://localhost/_cat/plugins>

### 单机启用多实例
- 启用单节点
```
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data
```
- 启用多实例
```
# 每个实例需要单独开一个客户端才行
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data

bin/elasticsearch -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data

bin/elasticsearch -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data

bin/elasticsearch -E node.name=node3 -E cluster.name=geektime -E path.data=node3_data

#查看集群
GET http://localhost:9200
#查看nodes
GET _cat/nodes
GET _cluster/health
```
备注：如果启动多实例，需要在配置文件中关闭端口信息，具体参考<https://segmentfault.com/q/1010000010798332>，当然也可以采用另外一种方式：<https://blog.csdn.net/ch999999999999999999/article/details/90731742>

## es入门

### 集群健康

1. 查看健康状态
```
GET /_cat/health?v
```
猜测:?v的作用是制表符对齐

每当我们要求群集健康时，我们要么获得绿色，黄色或红色。

绿色 - 一切都很好（集群功能齐全）
黄色 - 所有数据都可用，但尚未分配一些副本（群集功能齐全）
红色 - 某些数据由于某种原因不可用（群集部分功能）
注意:当群集为红色时，它将继续提供来自可用分片的搜索请求，但您可能需要尽快修复它，因为存在未分配的分片。

2. 查看节点列表
```
GET /_cat/nodes?v
```

3. 查看所有指数
```
GET /_cat/indices?v
```

### 创建索引
```
PUT /customer?pretty #使用PUT动词创建名为"customer"的索引。
```
说明:?pretty只是返回信息好看一些，并没有实际作用

### 索引和查询文档

1. 插入文档
让我们在客户索引中加入一些内容。
我们将一个简单的客户文档索引到客户索引中，ID为1，如下所示:
```
PUT /customer/_doc/1?pretty 
{ 
  "name":"John Doe" 
}
```
注意:头信息设置为`Content-Type:application/json`

响应:
```
{ 
  "_ index":"customer"，
  "_ type":"_ doc"，"_ id" 
  :"1"，
  "_ version":1，
  "result":"created"，
  "_ shards":{ 
    "total":2，
    "成功":1，
    "失败":0 
  }，
  "_ seq_no":0，
  "_ primary_term":1 
}
```
注意:Elasticsearch不需要在将文档编入索引之前先显式创建索引

2. 获取文档
```
GET /customer/_doc/1?pretty
```
响应:
```
{ 
  " _ index":"customer"，
  "_ type":"_doc"，"_id" 
  :"1"，
  "_ version":1，
  "_ seq_no":25，
  "_ primary_term":1，
  "found":true，
  " _source":{"name":"John Doe"} 
}
```

3. 删除索引
```
DELETE /customer?pretty
```

### 修改数据
1. 更新数据
备注:es更新数据是采用删除原数据，建立新数据并编制索引
```
POST /customer/_update/1?pretty
{
  "doc":{"name":"Jane Doe"}
}
```
或者添加年龄字段
```
POST /customer/_update/1?pretty
{
  "doc":{"name":"Jane Doe"，"age":20}
}
```

也可以使用简单脚本执行更新。此示例使用脚本将年龄增加5:
```
POST /customer/_update/1?pretty
{
  "script":"ctx._source.age += 5"
}
```
说明:在上面的示例中，ctx._source指的是即将更新的当前源文档。

2. 删除数据
```
DELETE /customer/_doc/1
```

3. 批量操作

- 批量修改/添加
备注：ID不存在的即为添加
```
POST /customer/_bulk?pretty 
{"index":{"_id":"1"}} 
{"name":"John Doe"} 
{"index":{"_id":"2"}} 
{"name ":"Jane Doe"}
```
备注：这个数据在postman中显示错误是正常的，数据每行都需要换行，以\n结束，包括最后一行都需要换行

- 批量删除
```
POST /customer/_bulk?pretty 
{"update":{"_ id":"1"}} 
{"doc":{"name":"John Doe成为Jane Doe"}} 
{"delete":{"_ id" : "2"}}
```
说明：Bulk API不会因其中一个操作失败而失败。
如果单个操作因任何原因失败，它将继续处理其后的其余操作。
批量API返回时，它将为每个操作提供一个状态（按照发送的顺序），
以便您可以检查特定操作是否失败。

- 更多操作示例
```
############Create Document############
#create document. 自动生成 _id
POST users/_doc
{
    "user" : "Mike",
    "post_date" : "2019-04-15T14:12:12",
    "message" : "trying out Kibana"
}

#create document. 指定Id。如果id已经存在，报错
PUT users/_doc/1?op_type=create
{
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

#create document. 指定 ID 如果已经存在，就报错
PUT users/_create/1
{
     "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

### Get Document by ID
#Get the document by ID
GET users/_doc/1


###  Index & Update
#Update 指定 ID  (先删除，在写入)
GET users/_doc/1

PUT users/_doc/1
{
    "user" : "Mike"

}


#GET users/_doc/1
#在原文档上增加字段
POST users/_update/1/
{
    "doc":{
        "post_date" : "2019-05-15T14:12:12",
        "message" : "trying out Elasticsearch"
    }
}



### Delete by Id
# 删除文档
DELETE users/_doc/1


### Bulk 操作
#执行两次，查看每次的结果

#执行第1次
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }


#执行第2次
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }

### mget 操作
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_id" : "2"
        }
    ]
}


#URI中指定index
GET /test/_mget
{
    "docs" : [
        {

            "_id" : "1"
        },
        {

            "_id" : "2"
        }
    ]
}


GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}

### msearch 操作
POST kibana_sample_data_ecommerce/_msearch
{}
{"query" : {"match_all" : {}},"size":1}
{"index" : "kibana_sample_data_flights"}
{"query" : {"match_all" : {}},"size":2}


### 清除测试数据
#清除数据
DELETE users
DELETE test
DELETE test2
```

### analysis 和 analyzer
- analyzer：分词器，在es中由三个部分组成
    1. character filter：针对原始文本处理，例如去除html
    2. tokenizer：按照规则切分为单词
    3. token filter：将切分的单词进行加工，小写，删除stopwords（停词：what,is,are,吗，和之类的），增加同义词

- 使用_analyzer API
```
#Simple Analyzer – 按照非字母切分（符号被过滤），小写处理
#Stop Analyzer – 小写处理，停用词过滤（the，a，is）
#Whitespace Analyzer – 按照空格切分，不转小写
#Keyword Analyzer – 不分词，直接将输入当作输出
#Patter Analyzer – 正则表达式，默认 \W+ (非字符分隔)
#Language – 提供了30多种常见语言的分词器
#2 running Quick brown-foxes leap over lazy dogs in the summer evening

#查看不同的analyzer的效果
#standard
GET _analyze
{
  "analyzer": "standard", #直接指定analyzer进行测试
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

#simpe
GET _analyze
{
  "analyzer": "simple", 
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}


GET _analyze
{
  "analyzer": "stop",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}


#stop
GET _analyze
{
  "analyzer": "whitespace",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

#keyword
GET _analyze
{
  "analyzer": "keyword",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

GET _analyze
{
  "analyzer": "pattern",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}


#english
GET _analyze
{
  "analyzer": "english",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}


POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "他说的确实在理”"
}


POST _analyze
{
  "analyzer": "standard",
  "text": "他说的确实在理”"
}


POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "这个苹果不大好吃"
}

POST _analyze
{
  "field": "title",# 指定索引字段进行测试
  "text": "标题是不是很欠揍"
}

POST _analyze
{
  "analyzer": "standard",
  "filter":["lowercase"],#自定义分词进行测试
  "text": "标题是不是很欠揍"
}   
```

#### 分词器分析
- standard analyzer：默认分词器，按此切分，小写处理，不会去除停词
- simple analyzer：按照非字母切分，非字母的都去除，并且进行小写处理
- whitespace analyzer：按照空格切分
- stop analyzer：在simple analyzer的基础上，去除了停词
- key analyzer：不分词，直接将输入当一个term输出
- pattern analyzer：通过正则表达式分词，默认\W+，非字符的符号进行分割
- language analyzer：
- icu-analyzer：这个是es插件，更适合亚洲语言
- IK：<https://github.com/medcl/elasticsearch-analysis-ik>
- thulac：<https://github.com/microbun/elasticsearch-thulac-plugin>


### SEARCH API

1. uri search：在url中使用参数
```
#URI Query
GET kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie
GET kibana*/_search?q=customer_first_name:Eddie
GET /_all/_search?q=customer_first_name:Eddie
```


```
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s
```
具体解析
- q：指定查询语句，使用query string syntax语法
- df：默认字段，不指定时，会对所有字段进行查询
- sort：排序/from 和 size 用于分页
- profile：可以查看查询如何被执行的





#### 指定查询索引
语法                      范围
/_search                    集群上所有的索引
/index1/_search             index1
/index1,index2/_search      index1和index2
/index*/_search             以index开头的索引

#### 查询语句语法
- 指定字段 vs 泛查询
```
q=title:2012
q=2012
```

- term vs phrase
```
beautiful mind 等效于 beautiful OR mind
"beautiful mind"等效于beautiful AND mind，phrase查询，还要求前后顺序保持一致
```

- 分组与引号
```
title:(beautiful AND mind)
title:"beautiful mind"
```

- 布尔操作
两个词默认是OR的关系，说明：关联词 AND OR NOT 必须大写哦，也可以用&& || ！

- 分组
    a. `+`表示must
    b. `-`表示must_not
    c. `title:(+mastrix -reloaded)`

- 范围查询：区间表示：[]闭区间，{}开区间
```
year:{2019 TO 2018}
year:[* TO 2018]
```

- 运算符号
```
year:>2010
year:(>2010 && <=2018)
year:(+>2010 +<=2018)
```

- 通配符查询：(通配符查询效率低，占用内存大，不建议使用。特别是放在最前面)
```
?代表一个字符，*代表0或者多个字符
title:mi?d
title:be*
```

- 正则表达式
```
title:[bt]oy
```

- 模糊匹配和近似匹配
```
title:befutil~1
title:"lord ring"~2
```

参考：<https://time.geekbang.org/course/detail/197-104956>
```
#基本查询
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s

#带profile
GET /movies/_search?q=2012&df=title
{
    "profile":"true"
}


#泛查询，正对_all,所有字段
GET /movies/_search?q=2012
{
    "profile":"true"
}

#指定字段
GET /movies/_search?q=title:2012&sort=year:desc&from=0&size=10&timeout=1s
{
    "profile":"true"
}


# 查找美丽心灵, Mind为泛查询
GET /movies/_search?q=title:Beautiful Mind
{
    "profile":"true"
}

# 泛查询
GET /movies/_search?q=title:2012
{
    "profile":"true"
}

#使用引号，Phrase查询
GET /movies/_search?q=title:"Beautiful Mind"
{
    "profile":"true"
}

#分组，Bool查询
GET /movies/_search?q=title:(Beautiful Mind)
{
    "profile":"true"
}


#布尔操作符
# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful AND Mind)
{
    "profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful NOT Mind)
{
    "profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful %2BMind)
{
    "profile":"true"
}


#范围查询 ,区间写法
GET /movies/_search?q=title:beautiful AND year:[2002 TO 2018%7D
{
    "profile":"true"
}


#通配符查询
GET /movies/_search?q=title:b*
{
    "profile":"true"
}

//模糊匹配&近似度匹配
GET /movies/_search?q=title:beautifl~1
{
    "profile":"true"
}

GET /movies/_search?q=title:"Lord Rings"~2
{
    "profile":"true"
}

```

### 2. request body 和Query DSL简介 
```
#REQUEST Body
POST kibana_sample_data_ecommerce/_search
{
    "profile": true,
    "query": {
        "match_all": {}
    }
}

POST /kibana_sample_data_ecommerce/_search
{
  "from":10,
  "size":20,
  "query":{
    "match_all": {}
  }
}


#对日期排序
POST kibana_sample_data_ecommerce/_search
{
  "sort":[{"order_date":"desc"}], #进行排序，最好是数字或者日期类型字段
  "query":{
    "match_all": {}
  }

}

#source filtering
POST kibana_sample_data_ecommerce/_search
{
  "_source":["order_date","name*"],# 指定返回字段，也可以采用通配符
  "query":{
    "match_all": {}
  }
}


#脚本字段
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_field": {
      "script": {
        "lang": "painless", #这里使用es中的painless脚本
        "source": "doc['order_date'].value+'hello'"
      }
    }
  },
  "query": {
    "match_all": {}
  }
}


POST movies/_search
{
  "query": {
    "match": {
      "title": "last christmas" # 默认是OR 
    }
  }
}

POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "last christmas",
        "operator": "and"
      }
    }
  }
}

POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love"

      }
    }
  }
}

POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love",
        "slop": 1 # 代表着中间可以有一个词介入

      }
    }
  }
}

```

### simple query string query
1. 类似于query string，但是会忽略错误的语法，同时只支持部分查询语法
2. 不支持AND OR NOT ，会当做字符串处理
3. Term之间默认关系是OR ,可以指定Operator
4. 支持部分逻辑，
    - `+`替代AND
    - `|`替代OR
    - '-'替代NOT
```
PUT /users/_doc/1
{
  "name":"Ruan Yiming",
  "about":"java, golang, node, swift, elasticsearch"
}

PUT /users/_doc/2
{
  "name":"Li Yiming",
  "about":"Hadoop"
}


POST users/_search
{
  "query": {
    "query_string": {
      "default_field": "name",
      "query": "Ruan AND Yiming"
    }
  }
}


POST users/_search
{
  "query": {
    "query_string": {
      "fields":["name","about"],
      "query": "(Ruan AND Yiming) OR (Java AND Elasticsearch)"
    }
  }
}


#Simple Query 默认的operator是 Or
POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Ruan AND Yiming",
      "fields": ["name"]
    }
  }
}


POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Ruan Yiming",
      "fields": ["name"],
      "default_operator": "AND"
    }
  }
}


GET /movies/_search
{
    "profile": true,
    "query":{
        "query_string":{
            "default_field": "title",
            "query": "Beafiful AND Mind"
        }
    }
}


# 多fields
GET /movies/_search
{
    "profile": true,
    "query":{
        "query_string":{
            "fields":[
                "title",
                "year"
            ],
            "query": "2012"
        }
    }
}



GET /movies/_search
{
    "profile":true,
    "query":{
        "simple_query_string":{
            "query":"Beautiful +mind",
            "fields":["title"]
        }
    }
}
```


### Mapping
定义：Mapping类似于数据库中的schema定义，作用如下
- 定义索引中的字段名称
- 定义字段的数据类型，例如字符串，数字，布尔...
- 字段，倒排索引的相关配置

一个Mapping属于一个索引的Type
- 每个文档属于一个Type
- 一个Type都有一个Mapping
- 7.0开始，不需要在Mapping定义中指定type信息

创建Mapping语法
```
PUT movies
{
    "mapping":{
        //具体定义
    }
}
```
建议：不要纯手写，可以先往es里面建立文档，获得自动的mapping信息，然后在这里面更改，
然后删除索引，重新建立


```

#设置 index 为 false
DELETE users
PUT users
{
    "mappings" : {
      "properties" : {
        "firstName" : {
          "type" : "text"
        },
        "lastName" : {
          "type" : "text"
        },
        "mobile" : {
          "type" : "text",
          "index": false #进行该字段建立索引，即不用这个字段进行搜索
        }
      }
    }
}

PUT users/_doc/1
{
  "firstName":"Ruan",
  "lastName": "Yiming",
  "mobile": "12345678"
}

POST /users/_search
{
  "query": {
    "match": {
      "mobile":"12345678"
    }
  }
}




#设定Null_value

DELETE users
PUT users
{
    "mappings" : {
      "properties" : {
        "firstName" : {
          "type" : "text"
        },
        "lastName" : {
          "type" : "text"
        },
        "mobile" : {
          "type" : "keyword",
          "null_value": "NULL" # 这样可以进行null搜索
        }

      }
    }
}

PUT users/_doc/1
{
  "firstName":"Ruan",
  "lastName": "Yiming",
  "mobile": null
}


PUT users/_doc/2
{
  "firstName":"Ruan2",
  "lastName": "Yiming2"

}

GET users/_search
{
  "query": {
    "match": {
      "mobile":"NULL"
    }
  }

}



#设置 Copy to
DELETE users
PUT users
{
  "mappings": {
    "properties": {
      "firstName":{
        "type": "text",
        "copy_to": "fullName"   #类似于创建一个用于搜索的新字段，但是不返回该字段值
      },
      "lastName":{
        "type": "text",
        "copy_to": "fullName"
      }
    }
  }
}
PUT users/_doc/1
{
  "firstName":"Ruan",
  "lastName": "Yiming"
}

GET users/_search?q=fullName:(Ruan Yiming)

POST users/_search
{
  "query": {
    "match": {
       "fullName":{
        "query": "Ruan Yiming",
        "operator": "and"
      }
    }
  }
}


#数组类型
PUT users/_doc/1
{
  "name":"onebird",
  "interests":"reading"
}

PUT users/_doc/1
{
  "name":"twobirds",
  "interests":["reading","music"]
}

POST users/_search
{
  "query": {
        "match_all": {}
    }
}

GET users/_mapping
```
参考：<https://time.geekbang.org/course/detail/197-105685>

#### 四个不同级别的index options配置，可以控制倒排索引记录的内容
1. docs：记录doc id
2. freqs：记录doc id和term frequencies
3. positions:记录doc id和term frequencies，term position
4. offsets：记录doc id和term frequencies，term position，charactor offsets
备注：text默认为positions，其他默认为docs，记录内容越多，占用空间越大


#### 能否更改Mapping字段类型
两种情况：
1. 新增字段
    - dynamic设置为true，一旦有新增字段写入，mapping也同时被更新
    - dynamic设置为false，mapping不会被更新，新增字段的数据无法被索引
    - dynamic设置为strict，文档写入失败
2. 对已有字段，已有数据写入，不再支持修改字段定义
3. 如果希望改变字段类型，必须Reindex API，重建索引
```
                true        false       strict
文档可索引       yes         yes         no
字段可索引       yes         no          no
Mapping可被更新  yes         no          no         
```


### Dynamic Mapping
在写入文档时，如果索引不存在，则自动创建索引

类型自动识别
JSON类型          ES类型
字符串             - 匹配日期格式，设置为Date。
                   - 配置数字，设置为float或者long，该选项默认关闭
                    - 设置为Text，并且增加keyword子字段
布尔值             boolean
浮点数             float
整数               long
对象               Object
数组               由第一个非空值的类型决定
空值               忽略


### 做字段类型
- 厂商名字实现精准匹配，增加一个keyword字段
- 使用不同的analyzer
    + 不同语言
    + pinyin 字段搜索
    + 还支持为搜索和索引指定不同的analyzer

#### Exact Values vs Full Text
Exact Values：包含数字，日期，具体一个不可分割字符串，在es中声明`keyword`
全文本：非结构化的文本数据，在es中声明`text`


#### 自定义分词
如果默认分词器无法满足要求，可以自定义分词器。
- character filter
    在tokenizer之前对文本进行处理。例如增加删除和替换文字。可以配置多个charactor filters。会影响tokenizer 的Position和offset信息
    
    一些自带的character filter
    + HTML strip ：去除html标签
    + Mapping 字符串替换
    + Pattern replace：正则匹配替换
- tokenizer
    按照一定的规则，将文本切分。内置的分词器有：whitespace/standard/uax_url_email/pattern/keyword/path hierarchy
    当然也可以用java自己开发插件
- token filter
-   将tokenizer输出的单词进行增加、修改、删除
-   自带的token filter：Lowercase/stop/synonym(添加近义词)

示例：
```
PUT logs/_doc/1
{"level":"DEBUG"}

GET /logs/_mapping

POST _analyze
{
  "tokenizer":"keyword",
  "char_filter":["html_strip"],#用html_strip去除标签
  "text": "<b>hello world</b>"
}


POST _analyze
{
  "tokenizer":"path_hierarchy",
  "text":"/user/ymruan/a/b/c/d/e"
}



#使用char filter进行替换
POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "mapping",
        "mappings" : [ "- => _"]
      }
    ],
  "text": "123-456, I-test! test-990 650-555-1234"
}

//char filter 替换表情符号
POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "mapping",
        "mappings" : [ ":) => happy", ":( => sad"]
      }
    ],
    "text": ["I am felling :)", "Feeling :( today"]
}

// white space and snowball
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["stop","snowball"],
  "text": ["The gilrs in China are playing this game!"]
}


// whitespace与stop
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["stop","snowball"],
  "text": ["The rain in Spain falls mainly on the plain."]
}


//remove 加入lowercase后，The被当成 stopword删除
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["lowercase","stop","snowball"],
  "text": ["The gilrs in China are playing this game!"]
}

//正则表达式
GET _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "pattern_replace",
        "pattern" : "http://(.*)",
        "replacement" : "$1"
      }
    ],
    "text" : "http://www.elastic.co"
}
```
参考：<https://time.geekbang.org/course/detail/197-105687>


### index template 和 dynamic template
index template：帮助你设定Mapping和Setting，并按照一定规则，自动匹配到新创建的索引上
1. 模板仅在创建时，才会产生作用。修改模板不会影响已创建索引。
2. 你可以设定多个索引模板，这些设定会被"merge"到一起，猜测：merge过程是自动的
3. 你可以指定“order”的数值，控制“merge”的过程


index template的工作方式：
1. 当一个索引被创建时，应用es默认的setting和mapping
2. 应用order数值低的index template中的设定
3. 应用order高的index template设定，之前的会被覆盖
4. 应用创建索引时，用户所指定setting和mapping，并覆盖之前模板中的设定


dynamic template：根据es的数据类型，结合字段名称，来动态设定字段类型
- 可以将所有的字符串类型都设定为keyword，或者关闭keyword字段
- is开头的字段设置为boolean
- long_开头的都设置成long数据
```
#数字字符串被映射成text，日期字符串被映射成日期
PUT ttemplate/_doc/1
{
    "someNumber":"1",
    "someDate":"2019/01/01"
}
GET ttemplate/_mapping


#Create a default template
PUT _template/template_default
{
  "index_patterns": ["*"],
  "order" : 0,
  "version": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas":1
  }
}


PUT /_template/template_test
{
    "index_patterns" : ["test*"],
    "order" : 1,
    "settings" : {
        "number_of_shards": 1,
        "number_of_replicas" : 2
    },
    "mappings" : {
        "date_detection": false,
        "numeric_detection": true
    }
}

#查看template信息
GET /_template/template_default
GET /_template/temp*


#写入新的数据，index以test开头
PUT testtemplate/_doc/1
{
    "someNumber":"1",
    "someDate":"2019/01/01"
}
GET testtemplate/_mapping
get testtemplate/_settings

PUT testmy
{
    "settings":{
        "number_of_replicas":5
    }
}

put testmy/_doc/1
{
  "key":"value"
}

get testmy/_settings
DELETE testmy
DELETE /_template/template_default
DELETE /_template/template_test



#Dynaminc Mapping 根据类型和字段名
DELETE my_index

PUT my_index/_doc/1
{
  "firstName":"Ruan",
  "isVIP":"true"
}

GET my_index/_mapping
DELETE my_index
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
            {
        "strings_as_boolean": {
          "match_mapping_type":   "string",
          "match":"is*",
          "mapping": {
            "type": "boolean"
          }
        }
      },
      {
        "strings_as_keywords": {
          "match_mapping_type":   "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}


DELETE my_index
#结合路径
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match":   "name.*",
          "path_unmatch": "*.middle",
          "mapping": {
            "type":       "text",
            "copy_to":    "full_name"
          }
        }
      }
    ]
  }
}


PUT my_index/_doc/1
{
  "name": {
    "first":  "John",
    "middle": "Winston",
    "last":   "Lennon"
  }
}

GET my_index/_search?q=full_name:John
```


### Aggregation（聚合）
通过聚合，可以得到数据的概览，高性能，实时性。

- Bucket Aggregation：一系列满足特定条件的文档集合，类似于mysql语句中的group by
- Metric Aggregation：数学运算，可以对文档字段进行统计分析，类似于mysql语句中的count()
    + 基于数据集计算结果，除了支持在字段上进行计算，同时也支持脚本(painless script)产生的结果之上进行计算
    + 大多数metric是数学计算，仅输出一个值：min/max/sum/avg/cardinality
    + 部分metric支持输出多个值：stats/percentiles/percentile_ranks
- PipeLine Aggregation：对其他的聚合结果进行二次聚合
- Matric Aggregation：支持对多个字段的操作，并提供一个结果矩阵

```
需要通过Kibana导入Sample Data的飞机航班数据。具体参考“2.2节-Kibana的安装与界面快速浏览”
#按照目的地进行分桶统计
GET kibana_sample_data_flights/_search
{
    "size": 0,
    "aggs":{
        "flight_dest":{
            "terms":{
                "field":"DestCountry"
            }
        }
    }
}



#查看航班目的地的统计信息，增加平均，最高最低价格
GET kibana_sample_data_flights/_search
{
    "size": 0,
    "aggs":{
        "flight_dest":{
            "terms":{
                "field":"DestCountry"
            },
            "aggs":{
                "avg_price":{
                    "avg":{
                        "field":"AvgTicketPrice"
                    }
                },
                "max_price":{
                    "max":{
                        "field":"AvgTicketPrice"
                    }
                },
                "min_price":{
                    "min":{
                        "field":"AvgTicketPrice"
                    }
                }
            }
        }
    }
}



#价格统计信息+天气信息
GET kibana_sample_data_flights/_search
{
    "size": 0,
    "aggs":{
        "flight_dest":{
            "terms":{
                "field":"DestCountry"
            },
            "aggs":{
                "stats_price":{
                    "stats":{
                        "field":"AvgTicketPrice"
                    }
                },
                "wather":{
                  "terms": {
                    "field": "DestWeather",
                    "size": 5
                  }
                }

            }
        }
    }
}
```



### 基于词项(Term)和基于全文的搜索

#### Term查询
Term的重要性，Term是表达语义的最小单位。搜索和利用统计语言模型进行自然语言处理都需要处理Term

特点：
- Term Level Query: Term Query / Range Query / Exists Query / Prefix Query /Wildcard Query
- 在 ES 中，Term 查询，对输⼊不做分词。会将输⼊作为⼀个整体，在倒排索引中查找准确的词项，并
且使⽤相关度算分公式为每个包含该词项的⽂档进⾏相关度算分 – 例如“Apple Store”

- 可以通过 Constant Score 将查询转换成⼀个 Filtering，避免算分，并利⽤缓存，提⾼性能

备注：Term查询不会将搜索条件进行分词（当然也不会转为小写），但是文档的分词器却会进行分词的
```
DELETE products
PUT products
{
  "settings": {
    "number_of_shards": 1
  }
}

# 插入数据，采用默认的standard分词器，即搜索条件词汇变为了小写
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "productID" : "XHDK-A-1293-#fJ3","desc":"iPhone" }
{ "index": { "_id": 2 }}
{ "productID" : "KDKE-B-9947-#kL5","desc":"iPad" }
{ "index": { "_id": 3 }}
{ "productID" : "JODL-X-1937-#pV7","desc":"MBP" }

GET /products

POST /products/_search
{
  "query": {
    "term": {
      "desc": {
        //"value": "iPhone" #无法搜索到数据，因为搜索条件不会变为小写，而待搜索文档分词时变成了小写
        "value":"iphone"
      }
    }
  }
}

POST /products/_search
{
  "query": {
    "term": {
      "desc.keyword": {
        //"value": "iPhone"
        //"value":"iphone"
      }
    }
  }
}


POST /products/_search
{
  "query": {
    "term": {
      "productID": {
        "value": "XHDK-A-1293-#fJ3" #无法搜索出数据，原因同上
      }
    }
  }
}

POST /products/_search
{
  //"explain": true,
  "query": {
    "term": {
      "productID.keyword": {
        "value": "XHDK-A-1293-#fJ3" #采用keyword是可以搜索到的
      }
    }
  }
}




POST /products/_search
{
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "productID.keyword": "XHDK-A-1293-#fJ3"
        }
      }

    }
  }
}


#设置 position_increment_gap
DELETE groups
PUT groups
{
  "mappings": {
    "properties": {
      "names":{
        "type": "text",
        "position_increment_gap": 0
      }
    }
  }
}

GET groups/_mapping

POST groups/_doc
{
  "names": [ "John Water", "Water Smith"]
}

POST groups/_search
{
  "query": {
    "match_phrase": {
      "names": {
        "query": "Water Water",
        "slop": 100
      }
    }
  }
}


POST groups/_search
{
  "query": {
    "match_phrase": {
      "names": "Water Smith"
    }
  }
}
```



#### 复合查询 – Constant Score 转为 Filter
● 将 Query 转成 Filter，忽略 TF-IDF 计算，避免相关性算分的开销
● Filter 可以有效利⽤缓存


#### 基于全⽂的查询
● 基于全⽂本的查找
     Match Query / Match Phrase Query / Query String Query
● 特点
    + 索引和搜索时都会进⾏分词，查询字符串先传递到⼀个合适的分词器，然后⽣成⼀个供查询的词
    项列表
    + 查询时候，先会对输⼊的查询进⾏分词，然后每个词项逐个进⾏底层的查询，最终将结果进⾏合
    并。并为每个⽂档⽣成⼀个算分。- 例如查 “Matrix reloaded”，会查到包括 Matrix 或者 reload
    的所有结果。

Match Query 查询过程

- 基于全⽂本的查找
    + Match Query / Match Phrase Query / Query String
    Query
- 基于全⽂本的查询的特点
    ● 索引和搜索时都会进⾏分词，查询字符串先传递到⼀个合
    适的分词器，然后⽣成⼀个供查询的词项列表
    ● 查询会对每个词项逐个进⾏底层的查询，再将结果进⾏合
    并。并为每个⽂档⽣成⼀个算分

### 结构化搜索

#### 结构化数据
● 结构化搜索（Structured search） 是指对结构化数据的搜索
    ⽇期，布尔类型和数字都是结构化的

● ⽂本也可以是结构化的。
    ○ 如彩⾊笔可以有离散的颜⾊集合： 红（red） 、 绿（green） 、 蓝（blue）
    ○ ⼀个博客可能被标记了标签，例如，分布式（distributed） 和 搜索（search）
    ○ 电商⽹站上的商品都有 UPCs（通⽤产品码 Universal Product Codes）或其他的唯⼀标
    识，它们都需要遵从严格规定的、结构化的格式。
● 布尔，时间，⽇期和数字这类结构化数据：有精确的格式，我们可以对这些格式进⾏逻辑操
作。包括⽐较数字或时间的范围，或判定两个值的⼤⼩。
● 结构化的⽂本可以做精确匹配或者部分匹配
    ○ Term 查询 / Prefix 前缀查询
● 结构化结果只有“是”或“否”两个值
    ○ 根据场景需要，可以决定结构化搜索是否需要打分

```
#结构化搜索，精确匹配
DELETE products
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10,"avaliable":true,"date":"2018-01-01", "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20,"avaliable":true,"date":"2019-01-01", "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30,"avaliable":true, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30,"avaliable":false, "productID" : "QQPX-R-3956-#aD8" }

GET products/_mapping



#对布尔值 match 查询，有算分
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "avaliable": true
    }
  }
}



#对布尔值，通过constant score 转成 filtering，没有算分
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "avaliable": true
        }
      }
    }
  }
}


#数字类型 Term
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "price": 30
    }
  }
}

#数字类型 terms
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "terms": {
          "price": [
            "20",
            "30"
          ]
        }
      }
    }
  }
}

#数字 Range 查询
GET products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "price" : {
                        "gte" : 20,
                        "lte"  : 30
                    }
                }
            }
        }
    }
}


# 日期 range
POST products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "date" : {
                      "gte" : "now-1y"
                    }
                }
            }
        }
    }
}



#exists查询
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "exists": {
          "field": "date"
        }
      }
    }
  }
}

#处理多值字段
POST /movies/_bulk
{ "index": { "_id": 1 }}
{ "title" : "Father of the Bridge Part II","year":1995, "genre":"Comedy"}
{ "index": { "_id": 2 }}
{ "title" : "Dave","year":1993,"genre":["Comedy","Romance"] }


#处理多值字段，term 查询是包含，而不是等于
POST movies/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "genre.keyword": "Comedy"
        }
      }
    }
  }
}


#字符类型 terms
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "terms": {
          "productID.keyword": [
            "QQPX-R-3956-#aD8",
            "JODL-X-1937-#pV7"
          ]
        }
      }
    }
  }
}



POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "match": {
      "price": 30
    }
  }
}


POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "date": "2019-01-01"
    }
  }
}

POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "match": {
      "date": "2019-01-01"
    }
  }
}




POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "productID.keyword": "XHDK-A-1293-#fJ3"
        }
      }
    }
  }
}

POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "productID.keyword": "XHDK-A-1293-#fJ3"
    }
  }
}

#对布尔数值
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "avaliable": "false"
        }
      }
    }
  }
}

POST products/_search
{
  "query": {
    "term": {
      "avaliable": {
        "value": "false"
      }
    }
  }
}

POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "price": {
        "value": "20"
      }
    }
  }
}

POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "match": {
      "price": "20"
    }
    }
  }
}


POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must_not": {
            "exists": {
              "field": "date"
            }
          }
        }
      }
    }
  }
}
```


### 搜索的相关性算分

相关性和相关性算分
+ 相关性 – Relevance
    ● 搜索的相关性算分，描述了⼀个⽂档和查询语句匹配的程度。ES 会对每个匹配查询条件的结
    果进⾏算分 _score
    ● 打分的本质是排序，需要把最符合⽤户需求的⽂档排在前⾯。ES 5 之前，默认的相关性算分
    采⽤ TF-IDF，现在采⽤ BM 25

```
词（Term）         ⽂档（Doc Id）
区块链             1，2，3
的                 2，3，4，5，6，7，8，10，12，13，15，18，19，20
应⽤               2，3，8，9，10，13，15
```

#### 词频 TF
● Term Frequency：检索词在⼀篇⽂档中出现的频率
    检索词出现的次数除以⽂档的总字数
● 度量⼀条查询和结果⽂档相关性的简单⽅法：简单将搜索中每⼀个 词的 TF 进⾏相加
    TF(区块链) + TF(的) + TF(应⽤)
● Stop Word
    “的” 在⽂档中出现了很多次，但是对贡献相关度⼏乎没有⽤处，不应该考虑他们的 TF



#### 逆⽂档频率 IDF
● DF：检索词在所有⽂档中出现的频率
    ● “区块链”在相对⽐较少的⽂档中出现
    ● “应⽤”在相对⽐较多的⽂档中出现
    ● “Stop Word” 在⼤量的⽂档中出现
● Inverse Document Frequency ：简单说 = log(全部⽂档数/检索词出现过的⽂档总数）
● TF-IDF 本质上就是将 TF 求和变成了加权求和
    ● TF(区块链)*IDF(区块链) + TF(的)*IDF(的)+ TF(应⽤)*IDF(应⽤)
```
            出现的⽂档数          总⽂档数        IDF
区块链         200 万               10 亿        log(500) = 8.96
的               10 亿                10 亿        log(1) = 0
应⽤              5 亿             10 亿            Log(2) = 1
```

Lucene 中的 TF-IDF 评分公式，从 ES 5 开始，默认算法改为 BM 25
● 和经典的TF-IDF相⽐，当 TF ⽆限增加时，
BM 25算分会趋于⼀个数值


#### 通过 Explain API 查看 TF-IDF
采用`{"explain":true}`返回算分过程


#### Boosting Relevance
● Boosting 是控制相关度的⼀种⼿段
    ● 索引，字段 或查询⼦条件
● 参数 boost的含义
    ● 当 boost > 1 时，打分的相关度相对性提升
    ● 当 0 < boost < 1 时，打分的权重相对性降低
    ● 当 boost < 0 时，贡献负分
```
PUT testscore
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text"
      }
    }
  }
}


PUT testscore/_bulk
{ "index": { "_id": 1 }}
{ "content":"we use Elasticsearch to power the search" }
{ "index": { "_id": 2 }}
{ "content":"we like elasticsearch" }
{ "index": { "_id": 3 }}
{ "content":"The scoring of documents is caculated by the scoring formula" }
{ "index": { "_id": 4 }}
{ "content":"you know, for search" }



POST /testscore/_search
{
  //"explain": true,
  "query": {
    "match": {
      "content":"you"
      //"content": "elasticsearch"
      //"content":"the"
      //"content": "the elasticsearch"
    }
  }
}

POST testscore/_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "content" : "elasticsearch"
                }
            },
            "negative" : {
                 "term" : {
                     "content" : "like"
                }
            },
            "negative_boost" : 0.2
        }
    }
}


POST tmdb/_search
{
  "_source": ["title","overview"],
  "query": {
    "more_like_this": {
      "fields": [
        "title^10","overview"
      ],
      "like": [{"_id":"14191"}],
      "min_term_freq": 1,
      "max_query_terms": 12
    }
  }
}
```

### Query Context & Filter Context
● ⾼级搜索的功能：⽀持多项⽂本输⼊，针对多个字
段进⾏搜索。
● 搜索引擎⼀般也提供基于时间，价格等条件的过滤
● 在 Elasticsearch 中，有Query 和 Filter 两种不同
的 Context
    ● Query Context：相关性算分
    ● Filter Context：不需要算分（ Yes or No），
    可以利⽤ Cache， 获得更好的性能

#### 条件组合
● 假设要搜索⼀本电影，包含了以下⼀些条件
    ● 评论中包含了 Guitar，⽤户打分⾼于 3 分，同时上映⽇期要在 1993 与 2000 年之间

● 这个搜索其实包含了 3 段逻辑，针对不同的字段
    ● 评论字段中要包含 Guitar / ⽤户评分⼤于 3 / 上映⽇期⽇期需要在给定的范围

● 同时包含这三个逻辑，并且有⽐较好的性能？
    ● 复合查询： bool Query

#### bool 查询
● ⼀个 bool 查询，是⼀个或者多个查询⼦句的组合
    ○ 总共包括 4 种⼦句。其中 2 种会影响算分，2 种不影响算分

● 相关性并不只是全⽂本检索的专利。也适⽤于 yes | no 的⼦句，匹配的⼦句越多，相关性评分
越⾼。如果多条查询⼦句被合并为⼀条复合查询语句 ，⽐如 bool 查询，则每个查询⼦句计算
得出的评分会被合并到总的相关性评分中。

must                必须匹配。 贡献算分
should              选择性匹配。贡献算分
must_not            Filter Context
                    查询字句，必须不能匹配
filter              Filter Context
                    必须匹配，但是不贡献算分

#### bool 查询语法
● ⼦查询可以任意顺序出现
● 可以嵌套多个查询
● 如果你的 bool 查询中，没有 must 条件，
should 中必须⾄少满⾜⼀条查询

如何解决结构化查询 – “包含⽽不是相等”的问题？
解决⽅案：增加⼀个 `genre count` 字段进⾏计数，这个字段插入数据时，可以由php等直接
组装

#### 查询语句的结构，会对相关度算分产⽣影响
● 同⼀层级下的竞争字段，具有有相同的权重
● 通过嵌套 bool 查询，可以改变对算分的影响

#### 控制字段的 Boosting
● Boosting 是控制相关度的⼀种⼿段
    ● 索引，字段 或查询⼦条件

● 参数 boost的含义
    ● 当 boost > 1 时，打分的相关度相对性提升
    ● 当 0 < boost < 1 时，打分的权重相对性降低
    ● 当 boost < 0 时，贡献负分
```
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10,"avaliable":true,"date":"2018-01-01", "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20,"avaliable":true,"date":"2019-01-01", "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30,"avaliable":true, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30,"avaliable":false, "productID" : "QQPX-R-3956-#aD8" }



#基本语法
POST /products/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "price" : "30" }
      },
      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      },
      "should" : [
        { "term" : { "productID.keyword" : "JODL-X-1937-#pV7" } },
        { "term" : { "productID.keyword" : "XHDK-A-1293-#fJ3" } }
      ],
      "minimum_should_match" :1
    }
  }
}

#改变数据模型，增加字段。解决数组包含而不是精确匹配的问题
POST /newmovies/_bulk
{ "index": { "_id": 1 }}
{ "title" : "Father of the Bridge Part II","year":1995, "genre":"Comedy","genre_count":1 }
{ "index": { "_id": 2 }}
{ "title" : "Dave","year":1993,"genre":["Comedy","Romance"],"genre_count":2 }

#must，有算分
POST /newmovies/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}

      ]
    }
  }
}

#Filter。不参与算分，结果的score是0
POST /newmovies/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}
        ]

    }
  }
}


#Filtering Context
POST _search
{
  "query": {
    "bool" : {

      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      }
    }
  }
}


#Query Context
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10,"avaliable":true,"date":"2018-01-01", "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20,"avaliable":true,"date":"2019-01-01", "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30,"avaliable":true, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30,"avaliable":false, "productID" : "QQPX-R-3956-#aD8" }


POST /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "productID.keyword": {
              "value": "JODL-X-1937-#pV7"}}
        },
        {"term": {"avaliable": {"value": true}}
        }
      ]
    }
  }
}


#嵌套，实现了 should not 逻辑
POST /products/_search
{
  "query": {
    "bool": {
      "must": {
        "term": {
          "price": "30"
        }
      },
      "should": [
        {
          "bool": {
            "must_not": {
              "term": {
                "avaliable": "false"
              }
            }
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}


#Controll the Precision
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "price" : "30" }
      },
      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      },
      "should" : [
        { "term" : { "productID.keyword" : "JODL-X-1937-#pV7" } },
        { "term" : { "productID.keyword" : "XHDK-A-1293-#fJ3" } }
      ],
      "minimum_should_match" :2
    }
  }
}



POST /animals/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "brown" }},
        { "term": { "text": "red" }},
        { "term": { "text": "quick"   }},
        { "term": { "text": "dog"   }}
      ]
    }
  }
}

POST /animals/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "quick" }},
        { "term": { "text": "dog"   }},
        {
          "bool":{
            "should":[
               { "term": { "text": "brown" }},
                 { "term": { "text": "brown" }},
            ]
          }

        }
      ]
    }
  }
}


DELETE blogs
POST /blogs/_bulk
{ "index": { "_id": 1 }}
{"title":"Apple iPad", "content":"Apple iPad,Apple iPad" }
{ "index": { "_id": 2 }}
{"title":"Apple iPad,Apple iPad", "content":"Apple iPad" }


POST blogs/_search
{
  "query": {
    "bool": {
      "should": [
        {"match": {
          "title": {
            "query": "apple,ipad",
            "boost": 1.1
          }
        }},

        {"match": {
          "content": {
            "query": "apple,ipad",
            "boost":
          }
        }}
      ]
    }
  }
}

DELETE news
POST /news/_bulk
{ "index": { "_id": 1 }}
{ "content":"Apple Mac" }
{ "index": { "_id": 2 }}
{ "content":"Apple iPad" }
{ "index": { "_id": 3 }}
{ "content":"Apple employee like Apple Pie and Apple Juice" }


POST news/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{"content":"apple"}
      }
    }
  }
}

POST news/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{"content":"apple"}
      },
      "must_not": {
        "match":{"content":"pie"}
      }
    }
  }
}

POST news/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "content": "apple"
        }
      },
      "negative": {
        "match": {
          "content": "pie"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

### 单字符串多字段查询：Dis Max Query
算分过程
● 查询 should 语句中的两个查询
● 加和两个查询的评分
● 乘以匹配语句的总数
● 除以所有语句的总数

Disjunction Max Query 查询
● 上例中，title 和 body 相互竞争
    ● 不应该将分数简单叠加，⽽是应该找到单个最佳匹配的字段的评分
● Disjunction Max Query
    ● 将任何与任⼀查询匹配的⽂档作为结果返回。采⽤字段上最匹配的评分最终评分返回

```
PUT /blogs/_doc/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /blogs/_doc/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}

POST /blogs/_search
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}

POST blogs/_search
{
    "query": {
        # "dis_max"：采用字段中评分最高的分数为排序标准
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ]
        }
    }
}


POST blogs/_search
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            # 将其他匹配语句的评分与 tie_breaker 相乘
            "tie_breaker": 0.2
        }
    }
}
```


### 单字符串多字段查询：Multi Match
三种场景
● 最佳字段 (Best Fields)
    ● 当字段之间相互竞争，⼜相互关联。例如 title 和 body 这样的字段。评分来⾃最匹配字段

● 多数字段 (Most Fields)
    ● 处理英⽂内容时：⼀种常⻅的⼿段是，在主字段( English Analyzer)，抽取词⼲，加⼊同义词，以
匹配更多的⽂档。相同的⽂本，加⼊⼦字段(Standard Analyzer)，以提供更加精确的匹配。其他字
段作为匹配⽂档提⾼相关度的信号。匹配字段越多则越好

● 混合字段 (Cross Field)
    ● 对于某些实体，例如⼈名，地址，图书信息。需要在多个字段中确定信息，单个字段只能作为整体的⼀部分。希望在任何这些列出的字段中找到尽可能多的词

#### Multi Match Query
● Best Fields 是默认类型，可以不⽤指定
● Minimum should match 等参数可以传递到⽣成的 query中
```
POST blogs/_search
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.2 #这个的作用是匹配度最好的分数*1，其他匹配字段分数*0.2
        }
    }
}

POST blogs/_search
{
  "query": {
    "multi_match": {
      "type": "best_fields",
      "query": "Quick pets",
      "fields": ["title","body"],
      "tie_breaker": 0.2,
      "minimum_should_match": "20%"
    }
  }
}



POST books/_search
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}


POST books/_search
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] # ^2，匹配分数*2
    }
}



DELETE /titles
PUT /titles
{
    "settings": { "number_of_shards": 1 },
    "mappings": {
        "my_type": {
            "properties": {
                "title": {
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   {
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}

PUT /titles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}

POST titles/_bulk
{ "index": { "_id": 1 }}
{ "title": "My dog barks" }
{ "index": { "_id": 2 }}
{ "title": "I see a lot of barking dogs on the road " }


GET titles/_search
{
  "query": {
    "match": {
      "title": "barking dogs"
    }
  }
}

DELETE /titles
PUT /titles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english",
        "fields": {"std": {"type": "text","analyzer": "standard"}}
      }
    }
  }
}

POST titles/_bulk
{ "index": { "_id": 1 }}
{ "title": "My dog barks" }
{ "index": { "_id": 2 }}
{ "title": "I see a lot of barking dogs on the road " }

GET /titles/_search
{
   "query": {
        "multi_match": {
            "query":  "barking dogs",
            "type":   "most_fields",
            "fields": [ "title", "title.std" ]
        }
    }
}

GET /titles/_search
{
   "query": {
        "multi_match": {
            "query":  "barking dogs",
            "type":   "most_fields",
            "fields": [ "title^10", "title.std" ]
        }
    }
}
```

### 多语⾔及中⽂分词与检索
⼀些中⽂分词器
● HanLP – ⾯向⽣产环境的⾃然语⾔处理⼯具包
    ● <http://hanlp.com/>
    ● <https://github.com/KennFalcon/elasticsearch-analysis-hanlp>
      <./elasticsearch-plugin install https://github.com/KennFalcon/elasticsearch-analysishanlp/releases/download/v7.1.0/elasticsearch-analysis-hanlp-7.1.0.zip>
● IK 分词器
    </elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysisik/releases/download/v7.1.0/elasticsearch-analysis-ik-7.1.0.zip>
● <https://github.com/medcl/elasticsearch-analysis-ik>

Pinyin Analysis

Pinyin
    ```./elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysispinyin/releases/download/v7.1.0/elasticsearch-analysis-pinyin-7.1.0.zip```


### 全文搜索实例
参考：<https://time.geekbang.org/course/detail/197-109502>

### 使⽤ Search Template 和 Index Alias

#### Search Template – 解耦程序 & 搜索 DSL
工作：创建搜索的模板，便于接口调用

#### Index Alias 实现零停机运维
工作：给接口创建别名，方便前端调用
```
POST _scripts/tmdb
{
  "script": {
    "lang": "mustache",
    "source": {
      "_source": [
        "title","overview"
      ],
      "size": 20,
      "query": {
        "multi_match": {
          "query": "{{q}}",
          "fields": ["title","overview"]
        }
      }
    }
  }
}
DELETE _scripts/tmdb

GET _scripts/tmdb

POST tmdb/_search/template
{
    "id":"tmdb",
    "params": {
        "q": "basketball with cartoon aliens"
    }
}
```

### 综合排序：Function Score Query 优化算分
参考：<https://time.geekbang.org/course/detail/197-111009>
说明：将一些字段加入算分，如点赞数

Function Score Query
● Function Score Query
    ● 可以在查询结束后，对每⼀个匹配的⽂档进⾏⼀系列的重新算分，根据新⽣成的分数进⾏排序。
● 提供了⼏种默认的计算分值的函数
    ● Weight ：为每⼀个⽂档设置⼀个简单⽽不被规范化的权重
    ● Field Value Factor：使⽤该数值来修改 _score，例如将 “热度”和“点赞数”作为算分的参考因素
    ● Random Score：为每⼀个⽤户使⽤⼀个不同的，随机算分结果
    ● 衰减函数： 以某个字段的值为标准，距离某个值越近，得分越⾼
    ● Script Score：⾃定义脚本完全控制所需逻辑


Boost Mode 和 Max Boost
● Boost Mode
    ● Multiply：算分与函数值的乘积
    ● Sum：算分与函数的和
    ● Min / Max：算分与函数取 最⼩/ 最⼤值
    ● Replace：使⽤函数值取代算分
● Max Boost 可以将算分控制在⼀个最⼤值

#### ⼀致性随机函数
● 使⽤场景：⽹站的⼴告需要提⾼展现率
● 具体需求：让每个⽤户能看到不同的随机排名，但是也希望同⼀个⽤户访问时，结果的相对顺序，保持⼀致 (Consistently Random)

```
DELETE blogs
PUT /blogs/_doc/1
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   0
}

PUT /blogs/_doc/2
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   100
}

PUT /blogs/_doc/3
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   1000000
}


POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes"
      }
    }
  }
}

POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p"
      }
    }
  }
}


POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p" ,
        "factor": 0.1
      }
    }
  }
}


POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p" ,
        "factor": 0.1
      },
      "boost_mode": "sum",#表示相加
      "max_boost": 3
    }
  }
}

POST /blogs/_search
{
  "query": {
    "function_score": {
      "random_score": {
        "seed": 911119
      }
    }
  }
}
```

### 搜索建议（Term & Phrase Suggester）
Elasticsearch Suggester API
● 搜索引擎中类似的功能，在 Elasticsearch 中是通过 Suggester API 实现的
● 原理：将输⼊的⽂本分解为 Token，然后在索引的字典⾥查找相似的 Term 并返回
● 根据不同的使⽤场景，Elasticsearch 设计了 4 种类别的 Suggesters
    ● Term & Phrase Suggester
    ● Complete & Context Suggester
参考:<https://time.geekbang.org/course/detail/197-111012>

### ⾃动补全与基于上下⽂的提示
使⽤ Completion Suggester 的⼀些步骤
● 定义 Mapping，使⽤ “completion” type
● 索引数据
● 运⾏ “suggest” 查询，得到搜索建议


什么是 Context Suggester
● Completion Suggester 的扩展
● 可以在搜索中加⼊更多的上下⽂信息，例如，输⼊ “star”
    ● 咖啡相关：建议 “Starbucks”
    ● 电影相关：”star wars”

实现 Context Suggester
● 可以定义两种类型的 Context
    ● Category – 任意的字符串
    ● Geo – 地理位置信息
● 实现 Context Suggester 的具体步骤
    ● 定制⼀个 Mapping
    ● 索引数据，并且为每个⽂档加⼊ Context 信息
    ● 结合 Context 进⾏ Suggestion 查询

参考：<https://time.geekbang.org/course/detail/197-111008>


## kibana

### 安装
上官网下载，基本开箱机用
官网：<https://www.elastic.co>
参考：<https://time.geekbang.org/course/detail/197-102662>

### kibana安装插件
```
bin/kibana-plugin install plugin_location
bin/kibana-plugin list
bin/kibana remove
```

### 启动
```
./kibana.bat
```


## logstash
参考：<https://time.geekbang.org/course/detail/197-102665>
注意：logstash.conf文件中引入数据文件路径写法
```
input {
  file {
    path => "/install/logstash-7.3.0/ml-latest-small/movies.csv" #不用包含C：/
    start_position => "beginning"
    sincedb_path => "null"
  }
}
```
