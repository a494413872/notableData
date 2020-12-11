---
tags: [java/中间件]
title: Elasticsearch(ES)
created: '2020-10-13T06:50:13.489Z'
modified: '2020-10-28T07:26:39.254Z'
---

# Elasticsearch(ES)

#### 数据分类
- 结构化数据
  > 有固定长度或者格式的数据，比如关系型数据库
- 非结构化数据
  > 长度不定，结构也不定的数据，比如word文档。对于非结构化的数据查找一般有两种方法。

#### ES的核心概念
传统关系型数据库，使用的流程如下：新建数据库->建表->插入数据->查询。而ES和传统关系型数据库的概念不太一样。
- 索引index
  - 一个索引可以理解为一个关系型数据库
- 类型type
  - 一个type相当于一个表
  - 但是ES 5.x中一个index可以有多个type，6.x中一个index只能有一个type，7.x已经移除了type的概念
- 映射mapping集群
  - mapping定义了每个字段的类型等信息。相当于关系型数据库中的表结构。
- 文档document
  - 相当于关系型数据库的一行记录。
- 字段
  - 相当于关系型数据库中表的字段
- 集群
  - 集群是由一个多个节点主城，一个集群有一个默认名称elasticsearch
- 节点
  - 一台机器或者一个进程。
- 分片和副本
  - 副本是分片的副本，分片有主分片和副分片之分
  - 一个Index数据在物理上被分布在多个主分片，每个主分片只存放部分数据
  - 每个主分片可以有多个副本，叫副本分片，是主分片的复制。

#### RESTful风格
RESTful风格，指的是通过HTTP动词来实现资源状态扭转，所谓的HTTP动词就是HEAD，POST，GET，PUT，DELET。
POST和PUT都是创建和更新资源，一般情况下是通过POST来更新，通过PUT来新增。

GET /user 列出所有的用户
POST /user 新建或者更新一个用户
PUT /user 新建或者更新一个用户
DELETE /user 删除一个用户

#### 下面通过postman操作索引
```
POST 方式提交http://localhost:9200/xdclass/_doc/1
body 选择raw，类型选JSON，内容{"user":"sdfs","message":"aaaaaaaaaaaaaaa"}
插入成功返回值
{
    "_index": "xdclass",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}

通过GET方式访问http://localhost:9200/xdclass/_doc/1
{"_index":"xdclass","_type":"_doc","_id":"1","_version":1,"_seq_no":2,"_primary_term":1,"found":true,"_source":{"user":"sdfs","message":"aaaaaaaaaaaaaaa"}}
```
删除一个index
```
DELETE 方式提交
删除成功返回值
{
    "_index": "xdclass",
    "_type": "_doc",
    "_id": "1",
    "_version": 2,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
通过GET方式访问http://localhost:9200/xdclass/_doc/1
{"_index":"xdclass","_type":"_doc","_id":"1","found":false}
```
下面是一些命令

- 查看所有索引
  - http://localhost:9200/_all
  - http://localhost:9200/_cat/indices?v
- 关闭和开启索引
  - http://localhost:9200/nba/_close
  - http://localhost:9200/nba/_open

#### 映射的介绍和使用
maaping相当于mysql中的表的字段定义
```
GET 方式提交http://localhost:9200/nba/_mapping获取mapping信息
{
    "nba": {
        "mappings": {}
    }
}
POST 方式提交http://localhost:9200/nba/_mapping
body 选择raw，类型选JSON，内容
{
  "properties":{
    "name":{"type":"keyword"},
    "desc":{"type":"text"}
  }
}
插入成功返回值
{
    "acknowledged": true
}
通过GET方式访问http://localhost:9200/nba/_mapping
{
    "nba": {
        "mappings": {
            "properties": {
                "desc": {
                    "type": "keyword"
                },
                "name": {
                    "type": "text"
                }
            }
        }
    }
}
text 表示使用全文索引，可以进行分词的。
keyword 表示关键字，不会被分词。
```
##### mapping只能加字段，不能改字段。比如不可以把一个字段从text改为keyword

#### 文档的增删改查
比如上面的这个例子：POST 方式提交http://localhost:9200/xdclass/_doc/1
_doc其实就是ES的默认类型，7以后就只能用这个类型了，如果添加别的类型的话，会报错。而_doc后面的/1表示的是id=1。不带id的话，ES支持自己生成ID
```
{
    "error": {
        "root_cause": [
            {
                "type": "illegal_argument_exception",
                "reason": "Rejecting mapping update to [nba] as the final mapping would have more than 1 type: [_doc, lld]"
            }
        ],
        "type": "illegal_argument_exception",
        "reason": "Rejecting mapping update to [nba] as the final mapping would have more than 1 type: [_doc, lld]"
    },
    "status": 400
}
```
- 自动创建索引
  - 查看auto_create_index开关状态。可以通过http://localhost:9200/settings
  - 当索引不存在，并且auto_create_index为true，新增文档自动创建索引
  - 为false不会自动创建索引
  - 修改状态PUT 如下数据到http://localhost:9200/settings
  ```
  {
    "persistent"：{
      "action.auto_create_index":"false"
    }
  }
  ```
  因为POST同时支持更新和新增。所以可以通过添加?op_type=create。
  http://localhost:9200/xdclass/_doc/1?op_type=create

#### 查看文档
查看的话有几种方式
1. GET方式，根据ID单独查
```
http://localhost:9200/nba/_doc/1
```
2. POST方式通过_wget，可以根据传到后台的JSON进行查询，需要指定id。有多重形式
```
  http://localhost:9200/_wget
  {
    "docs":[
    {
    	"_index":"nba",
    	"_type":"_doc",
    	"_id":"1"
    },
    {
    	"_index":"nba",
    	"_type":"_doc",
    	"_id":"2"
    }
    ]
}
```

```
  http://localhost:9200/nba/_wget
  {
    "docs":[
    {
    	"_type":"_doc",
    	"_id":"1"
    },
    {
    	"_type":"_doc",
    	"_id":"2"
    }
    ]
}
```
```
  http://localhost:9200/nba/_doc/_wget
{
    "docs":[
    {
    	"_id":"1"
    },
    {
    	"_id":"2"
    }
    ]
}
或者
{
    "ids":["1","2"]
}
```


#### 修改文档
POST  http://localhost:9200/nba/_update/1
修改字段的值
```
{
   "doc":{
   	"name":"1111"
   }
}
```
添加字段
```
{
   "script":"ctx._source.age=18"
}
```
删除字段
```
{
   "script":"ctx._source.remove(\"age\")"
}
```
#### 删除文档
直接采用DELETE 请求POST  http://localhost:9200/nba/_doc/1

#### 搜索的简单使用
- term词条查询
  >不会分词完全匹配才能搜索到
- full text全文查询
  > 先分析字符串，拆分为多个分词，已分析字段总包含词条的任意一个，或者全部包含，就匹配，返回文档。

##### term
POST http://localhost:9200/nba/_search
```
{
  "query":{
    "term":{
      "name":"123"
    }
  }
}
```
##### full text
POST http://localhost:9200/nba/_search
```
查询所有
{
  "query":{
    "match_all":{
    }
  }
}

根据名称查询
from 和 size 是从0开始的100条
match只能搜索text类型的数据
{
  "query":{
    "match":{
    	"name":"sdfs"
    }
  },
  "from":0,
  "size":100
}


通过多个字段查询
{
  "query":{
    "multi_match":{
      "name":"sdfs",
      "desc":"111111"
    }
  }
}

如果通过sdfs sss查询，可能查询出来 名称为sdfs的数据的。所以可以通过match_phrase来实现比较精确的查询，这时候就只能查到名称为sdfs sss的记录了。
{
  "query":{
    "match_phrase":{
    	"name":"sdfs sss"
    }
  }
}

match_phrase_prefix  前缀查询，查询以指定开头查询
如果不加_prefix下面查询是查不到数据的。
{
  "query":{
    "match_phrase_prefix":{
    	"name":"sdfs s"
    }
  }
}

```
#### 分词器
- 什么是分词器
  - 分词器是将用户输入的文本根据一定逻辑，分析称多个词语的工具
- ES常用的内置分词器
  - standard 标准分词器，不指定的话默认使用
  - simple 只要不是字母的字符，就将文本解析文term，所有term都是小写的
  - withespace 遇到空白，就解析成term
  - stop 会自动删除停止词，停止词比如{the,a,is,that,of,at}等等
  - language 根据语言分词
  - pattern 根据正则分词，默认是\W+(非单词字符)
- 分词器可以在创建缩印的时候进行指定

- 常见中文分词器（需要安装插件）
  - smartCN 简单的中文或者中英文混合文本分词器
  - IK分词器 更智能更友好的中文分词器

#### 核心数据类型
- 字符串
  - text 全文索引，通过分词器分词
  - keyword 部分词，只能搜索完整的值。
-  数值类型
  - long，integer，short，byte，double，float，half_float，scaled_float
- 布尔类型
- 二进制 binary
  - 这个类型把字段值当做base64编码的字符串，默认不存储，且不可搜索。
- 范围类型
  - 范围类型存储的不是一个值，而是一个范围。{"gte":20,"lte":40} ,搜索的时候只要符合条件都能搜索到，比如搜21，22，23都行。
- 日期 -date
  - 由于Json没有date类型，所以ES通过识别字符串是否符合format来判断是否为日期类型。
  - format格式为：strict_date_optional_time||epoch_millis
    - "2020-01-01","2020/01/01 12:30:21"
    - linxu时间戳
    - 毫秒数
- 数组 array
  -  ES没有专门数组类型，直接通过[]定义就行，数组中值必须是同一个类型
- 对象类型Object
  - 对象类型可能有内部对象
- 专用数据类型
  - IP类型

#### 精确查询
nba/_search
```
精确查询
{
  "query":{
    "term":{
      "name":"123"
    }
  }
}
```


```
查询是否存在
{
  "query":{
    "exists":{
      "name":"123"
    }
  }
}
```

```
前缀查询
{
  "query":{
    "prefix":{
      "name":"123"
    }
  }
}
```

```
通配符查询，*表示任意字符串？表示单个字符。
{
  "query":{
    "wildcard":{
      "name":"s*s"
    }
  }
}
```

```
IDS 查询
{
  "query":{
    "ids":{
      "values":[1,2]
    }
  }
}
```
```
范围查询
{
  "query":{
    "range":{
      "age":{"get":2,"lte":10}
    }
  }
}
根据日期的话，要去format
```

#### 布尔查询
type|description
:-|:-
must|必须出现在文档中
filter|必须出现在文档中，但是不打分
must_not|不能出现在文档中
should|应该出现在文档中

单独使用should，是表示符合一个条件就行。
但是如果搭配must或者filter使用，should的条件就算都不满足也会查出来。这时候should里面加上"minimum_should_match":1,表示should里面至少有一个条件要符合。

POST nba/_search
```
名称中包含sss的
{
  "query":{
    "bool":{
      "must":[
      	{
        "match":{"name":"sss"}
    	}
      ]
    }
  }
}
```

#### query_string 查询
query_sting查询，如果熟悉lucene的查询语法，可以直接使用Lucene查询语法写一个查询串进行查询。ES接受到请求之后，通过查询解析器解析查询串生成对应的查询。
POST /nba/_search
```
通过名称查询
{
  "query":{
    "query_string":{
       "default_field":"name",
       "query":"sss OR sdfs"
    }
  }
}
{
  "query":{
    "query_string":{
       "default_field":"name",
       "query":"sss AND aaa"
    }
  }
}
多个字段的话
{
  "query":{
    "query_string":{
       "fields":["name","desc"],
       "query":"sss OR aaaaaaaaaaaaaaa"
    }
  }
}
```

#### ES 的刷新操作
ES中新添加的是不能被直接搜索到的，需要被刷新才能被搜索到
修改默认更新时间，默认是1s
手动刷新的话是使用/nba/_doc/1?refresh
```
PUT /nba/_settings
{
  "index":{
    "refresh_interval":"5s"
  }
}
refresh_interval 设置为-1就不会自动刷新了。
```

#### 实际操作
1. 类似的数据放在一个索引，非类似的数据放不同索引：product index（包含了所有的商品），sales index（包含了所有的商品销售数据），inventory index（包含了所有库存相关的数据）。如果你把比如product，sales，human resource（employee），全都放在一个大的index里面，比如说company index，不合适的。
2. index中包含了很多类似的document：类似是什么意思，其实指的就是说，这些document的fields很大一部分是相同的，你说你放了3个document，每个document的 fields都完全不一样，这就不是类似了，就不太适合放到一个index里面去了。
3. 索引名称必须是小写的，不能用下划线开头，不能包含逗号：product，website，blog
新建一个索引带mapping
POST  http://10.200.3.92:9200/code
```
{}
----------
{
    "acknowledged": true
}
```
查看索引
GET http://10.200.3.92:9200/code
```
{
    "code": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "creation_date": "1603146409066",
                "uuid": "pERq7XWRRyGYw4ll2HPIHA",
                "number_of_replicas": "1",
                "number_of_shards": "5",
                "version": {
                    "created": "1070199"
                }
            }
        },
        "warmers": {}
    }
}
```
创建mapping，创建mapping的时候必须有type
POST http://10.200.3.92:9200/code/data/_mapping
```
{
  "properties":{
    "code_code_id":{"type":"String","index":"not_analyzed"},
    "code_serialno":{"type":"String","index":"not_analyzed"},
    "code_create_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    "code_status":{"type":"String","index":"not_analyzed"},
    "code_code":{"type":"String"},
    "code_add_code":{"type":"String","index":"not_analyzed"},
    "code_add_code_md5":{"type":"String"},
    "code_mobile":{"type":"String","index":"not_analyzed"},
    "code_sms_content":{"type":"String"},
    "code_ext_id":{"type":"String","index":"not_analyzed"},
    "code_url":{"type":"String"},
    "code_object_id":{"type":"String","index":"not_analyzed"},
    "code_object_type":{"type":"String"},
    "code_status_no":{"type":"String"},
    "code_status_explanation":{"type":"String"},
    "code_terminal_content":{"type":"String"},
    "code_code_image":{"type":"String"},
    "code_send_sms":{"type":"String"},
    "code_update_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    "code_send_orderid":{"type":"String"},
    "code_order_id":{"type":"String"},
    "code_reapply_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    "code_reapply_count":{"type":"String"},
    "code_code_total":{"type":"String"},
    "code_failed_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    "code_disabled":{"type":"String"},
    "code_create_datetime":{"type":"String","index":"not_analyzed"},
    "code_provider_id":{"type":"String","index":"not_analyzed"},
    "code_provider_name":{"type":"String"},
    "code_distributor_id":{"type":"String"},
    "code_file_id":{"type":"String"},
    "code_lvurl_index":{"type":"String"},
    "code_refund_item_info":{"type":"String"},
    "code_code_image_flag":{"type":"String"},
    "code_hoard_flag":{"type":"String","index":"not_analyzed"},
    "code_hoard_code_batch":{"type":"String"},
    "code_export_code_batch":{"type":"String"},
    "code_hoard_code_status":{"type":"String","index":"not_analyzed"},
    "code_hoard_mobile":{"type":"String"},
    "code_pass_code_return_type":{"type":"String"},
    "code_code_max_process_times":{"type":"String"},
    "code_code_processed_times":{"type":"String"},
    "code_code_detail_status":{"type":"String","index":"not_analyzed"},
    "code_batch_no":{"type":"String"},
    "code_has_new_list":{"type":"String"},
    "provider_is_auto_perform":{"type":"String"},
    "detail_pass_code_detail_id":{"type":"String"},
    "detail_pass_code_id":{"type":"String"},
    "detail_visite_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    "detail_user_id":{"type":"String","index":"not_analyzed"},
    "detail_suppgoods_id":{"type":"String"},
    "detail_provider_goods_code":{"type":"String"},
    "detail_prod_manager":{"type":"String","index":"not_analyzed"},
    "detail_supplier_id":{"type":"String","index":"not_analyzed"},
    "detail_supplier_name":{"type":"String","index":"not_analyzed"},
    "detail_distributor_code":{"type":"String","index":"not_analyzed"},
    "detail_distributor_name":{"type":"String","index":"not_analyzed"},
    "detail_content":{"type":"String"},
    "detail_redestroy_count":{"type":"String"},
    "detail_distribution_channel":{"type":"String"},
    "detail_hoard_voucher":{"type":"String"},
    "detail_hoard_hashid":{"type":"String"},
    "detail_hoard_full_name":{"type":"String"},
    "detail_hoard_act_id":{"type":"String"},
    "detail_create_date":{"type":"String"},
    "detail_update_date":{"type":"String"},
    "pass_port_code":{"type":"nested",
    	"properties":{
    		"pass_point_id":{"type":"String"},
    		"code_id":{"type":"String"},
    		"status":{"type":"String","index":"not_analyzed"},
    		"used_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    		"terminal_content":{"type":"String"},
    		"valid_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    		"invalid_time":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
    		"ext_id":{"type":"String"},
    		"status_no":{"type":"String"},
    		"status_explanation":{"type":"String"},
    		"invalid_date":{"type":"String"},
    		"invalid_date_memo":{"type":"String"},
    		"disabled":{"type":"String"},
    		"create_date":{"type":"String"},
    		"update_date":{"type":"String"},
    		"code_content":{"type":"String"}
    	}
    }
    
  }
}
---------------------------------------------
{
    "acknowledged": true
}
```
#### ES 查询类型
es 在查询时， 可以指定搜索类型为下面四种：
ES 天生就是为分布式而生， 但分布式有分布式的缺点。 比如要搜索某个单词， 但是数据却分别在 5 个分片（Shard)上面， 这 5 个分片可能在 5 台主机上面。 因为全文搜索天生就要排序（ 按照匹配度进行排名） ,但数据却在 5 个分片上， 如何得到最后正确的排序呢？ ES是这样做的， 大概分两步：
　　step1、 ES 客户端将会同时向 5 个分片发起搜索请求。
　　step2、 这 5 个分片基于本分片的内容独立完成搜索， 然后将符合条件的结果全部返回。

客户端将返回的结果进行重新排序和排名，最后返回给用户。也就是说，ES的一次搜索，是一次scatter/gather过程（这个跟mapreduce也很类似）
- QUERY_THEN_FETCH
  - 向索引的所有分片 （ shard）都发出查询请求， 各分片返回的时候把元素文档 （ document）和计算后的排名信息一起返回。
- QUERY_AND_FEATCH
  - 第一步， 先向所有的 shard 发出请求， 各分片只返回文档 id(注意， 不包括文档 document)和排名相关的信息(也就是文档对应的分值)， 然后按照各分片返回的文档的分数进行重新排序和排名， 取前 size 个文档。
　- 第二步， 根据文档 id 去相关的 shard 取 document。 这种方式返回的 document 数量与用户要求的大小是相等的。
- DFS_QUERY_THEN_FEATCH
  - 这种方式比第一种方式多了一个 DFS 步骤，有这一步，可以更精确控制搜索打分和排名。也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作
- DFS_QUERY_AND_FEATCH
  - 比第 2 种方式多了一个 DFS 步骤。
　　也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作、

#### DFS
DFS 其实就是在进行真正的查询之前， 先把各个分片的词频率和文档频率收集一下， 然后进行词搜索的时候， 各分片依据全局的词频率和文档频率进行搜索和排名

