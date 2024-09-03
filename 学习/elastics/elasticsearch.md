[TOC]



# 1、是什么

```
分布式搜索引擎
```



# 2、倒排索引

```
正向索引：根据文档找词（mysql)
倒排索引：根据词找文档


通过分词器，将文档按照语义分成不同的词条（term）,统计不同文档中是否存在该词条。

词条         文档id
小米         1
手机         1,2
苹果         2
```

![image-20221113185328818](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221113185328818.png)

# 3、概念

![image-20221113185943454](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221113185943454.png)

```
最重要的是，在同一索引中存储具有很少或没有共同字段的不同实体会导致数据稀疏并干扰 Lucene 有效压缩文档的能力。
由于这些原因，官方决定从 Elasticsearch 中删除_type的概念

es 8 已经没有type这个概念了，现在一个index相当于一个表而不是库。
```



# 4、mapping属性

```
mapping是对索引库中文档的约束
```

![image-20221113201727532](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221113201727532.png)

创建索引时需要考虑的问题：

​	字段名、数据类型、是否参与搜索、是否分词、如果分词，分词器是什么？

https://www.bilibili.com/video/BV1LQ4y127n4?p=91&vd_source=ad1816bd115387168ec5dfd6660ef759

```
#参考下
PUT /alldata
{
  "mappings": {
    "properties": {
      "ID": {
        "type": "keyword"
      },
      "seno": {
        "type": "keyword",
        "index": false
      },
      "code": {
        "type": "keyword",
        "copy_to": "all_index"
      },
      "name": {
        "type": "keyword",
        "copy_to": "all_index"
      },
      "desc": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "xianjia": {
        "type": "float",
        "index": false
      },
      "zhangdiefu": {
        "type": "float",
        "index": false
      },
      "date": {
        "type": "date",
        "copy_to": "all_index"
      },
      "location": {
        "type": "geo_point"
      },
      "all_index": {
        "type": "text"
      }
    }
  }
}
```



# 5、索引库操作

## 1、创建

索引名。这个名字必须是全部小写，不能以下划线开头，不能包含逗号

```

{
	"email":"372532055@qq.com",
	"info": "周杰伦的代表作是青花瓷",
	"name":{
	  "first_name":"云",
	  "last_name":""赵
	}
}

#创建索引库
PUT /my_data
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "type": "object",
        "properties": {
          "first_name": {
            "type": "keyword"
          },
          "last_name": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

## 2、查看

```
#查看索引库
GET /my_data
```

## 3、删除

```
#删除索引库
DELETE /my_data
```

## 4、修改

```
#修改索引库，索引库和mapping一旦创建无法修改，但是可以新增字段
PUT /my_data/_mapping
{
	"properties":{
	  "age":{
	    "type":"text"
	  }
	}
}
```

![image-20221113210807008](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221113210807008.png)

# 6、文档操作

_id:

```
id仅仅是一个字符串，它与 _index 和 _type 组合时，就可以在ELasticsearch中唯一标识一个文档。当创建一个文档，你可以 自定义 _id ，也可以让Elasticsearch帮你自动生成。
```



## 1、新增

```
#插入文档
POST /my_data/_doc/1
{
	"email":"372532055@qq.com",
	"info": "周杰伦的代表作是青花瓷",
	"name":{
	  "first_name":"云",
	  "last_name":"赵"
	}
}
```

## 2、查询

```
GET /my_data/_doc/1
```



## 3、删除

```
DELETE /my_data/_doc/1
```



## 4、修改

```
#1、全量修改，会删除旧的文档、新增新文档  [和新增一样，put、post都可以]
PUT /my_data/_doc/1
{
	"email":"372532055@qq.com",
	"info": "周杰伦的代表作是青花瓷",
	"name":{
	  "first_name":"云",
	  "last_name":"赵"
	}
}

#2、增量修改
POST /my_data/_update/12  (只能post)
{
  "doc": {
    "email": "zhoujielun@qq.com"
  }
}
```



![image-20221113210711645](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221113210711645.png)



# 7、 DSL查询语法

## 1、查询所有

```
GET /my_data/_search
{
  "query": {
    "match_all": {}
  }
}
```

## 2、全文检索

match查询：全文检索查询的一种，会对用户输入内容分词，然后到倒排索引检索。

multi_match查询：与match类似，只不过multi_match支持多个字段查询。（参与字段越多、性能越差建议copy_to方式+match性能会好点）

```
#单字段匹配 match
GET /my_data/_search
{
  "query": {
    "match": {
      "info": "青花瓷"
    }
  }
}

#多字段匹配
GET /my_data/_search
{
  "query": {
    "multi_match": {
      "query": "青花瓷",
      "fields": [
        "name.first_name","info"
      ]
    }
  }
}
```

## 3、精确查询

除了text外的其他类型字段，不会对查询字段进行分词，需要完全匹配。

term：根据词条精确值查询

range：根据值的范围查询。

```
#精确匹配
GET /my_data/_search
{
  "query": {
    "term": {
      "email": {
        "value": "372532055@qq.com"
      }
    }
  }
}

#range范围匹配
GET /my_data/_search
{
  "query": {
    
    "range": {
      "age": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```

## 4、地理查询

根据地理坐标查找

Geo_bounding_box：根据两个点间对角形成的矩阵查找范围 （地区找房）

Geo_distance: 根据一个点和距离画圆查询 （我的附件）

![image-20221115234829142](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221115234829142.png)

![image-20221115235012396](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221115235012396.png)

```
# distance 距离和中心点
GET /alldata/_search
{
  "query": {
    "geo_distance": {
      "distance": 100,
      "location": {
        "lat": 40.73,
        "lon": -74.1
      }
    }
  }
}
```

## 5、复合查询

复合查询可以将其他简单查询组合起来，实现更复杂的搜索逻辑，如：

### <1> Function_scope:

算分函数查询：可以控制文档相关性算分，干预文档排名，如百度竞价。

​	三要素：

- ​		 filter过滤条件：哪些文档要加分

- ​		 算分函数：如何计算function_scope

-  		加权方式：function_scope 与 query_scope如何运算

![image-20221116235644068](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221116235644068.png)

```
#Function_scope
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "customer_last_name": "Padilla"
        }
      },
      "functions": [
        {
          "filter": {
            "term": {
              "email": "yasmine@padilla-family.zzz"
            }
          },
          "weight": 10
        }
      ],
      "boost_mode": "sum"  
    }
  }
}
```

### <2>Boolean Query

![image-20221117000652244](/Users/thebug4j/workspace/github/Documents/学习/elastics/assets/image-20221117000652244.png)



# 8、排序

排序不会再计算相关性分数

```
#排序
GET /my_data/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "email": {
        "order": "asc"
      }
    }
  ]
}
#地理位置排序
GET /my_data/_search
{
  "query": {
    "match_all": {}
  },
 "sort": [
   {
     "_geo_distance": {
      "location": {
        "lat": 40.73,
        "lon": -74.1
      },
       "order": "asc",
       "unit": "km"
     }
   }
 ]
}
```

# 9、分页

```
#分页
GET /kibana_sample_data_ecommerce/_search
{
  "query": {"match_all": {}},
  "from": 0,
  "size": 2
}
```

# 10、聚合

聚合（aggregations）可以实现对文档数据的统计、分析、运算。聚合常见的有三类：

- 桶（Bucket）聚合：用来对文档做分组TermAggregation：按照文档字段值分组Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组
- 度量（Metric）聚合：用以计算一些值，比如：最大值、最小值、平均值等Avg：求平均值Max：求最大值Min：求最小值Stats：同时求max、min、avg、sum等
- 管道（pipeline）聚合：其它聚合的结果为基础做聚合

```
#Bucket聚合（统计200元以下的酒店品牌有几种，此时可以根据酒店品牌的名称做聚合）
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "lte": 200 // 只对200元以下的文档聚合
      }
    }
  }, 
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "order": {
          "_count": "asc" // 按照_count升序排列
        },
        "size": 20
      }
    }
  }
}
#Metrics 聚合 获取每个品牌的用户评分的min、max、avg等值
GET /hotel/_search
{
  "size": 0, 
  "aggs": {
    "brandAgg": { 
      "terms": { 
        "field": "brand", 
        "size": 20
      },
      "aggs": { // 是brands聚合的子聚合，也就是分组后对每组分别计算
        "score_stats": { // 聚合名称
          "stats": { // 聚合类型，这里stats可以计算min、max、avg等
            "field": "score" // 聚合字段，这里是score
          }
        }
      }
    }
  }
}
```



# 11、自动补全

当用户在搜索框输入字符时，我们应该提示出与该字符有关的搜索项

```
#因此字段在创建倒排索引时应该用my_analyzer分词器；字段在搜索时应该使用ik_smart分词器;避免同音字
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "ik_max_word", 
          "filter": "py"
        }
      },
      "filter": {
        "py": { // 过滤器名称
          "type": "pinyin", // 过滤器类型，这里是pinyin
					"keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer",
        "search_analyzer": "ik_smart"
      }
    }
  }
}

```



# 12、ES Repository和ESTemplate的区别

JPA中有个ElasticsearchRepository可以做Elasticsearch的相关增删改查，用法和普通的CRUDRepository是一样的，这样就能统一ElasticSearch和普通的JPA操作，获得和操作mysql一样的代码体验。但是同时可以看到ElasticsearchRepository的功能是比较少的，简单查询够用，但复杂查询就不够了。而ElasticsearchTemplate则提供了更多的方法来完成更多的功能，也包括分页之类的，他其实就是一个封装好的ElasticSearch Util功能类，通过直接连接client来完成数据的操作。



# 批量

就像 mget 允许我们一次性检索多个文档一样， bulk API允许我们使用单一请求来实现多个文档





# 别名

所以请做好准备：在应用中使用别名而不是索引。然后你就可以在任何时候重建索引。别名的开销很小，应当广泛使用。



# 集群

故障转移：网络分区时，根据候选节点超过半数，重新选举出主节点，可以解决脑裂问题。但此时，集群处于亚健康状态，此时主节点会拷贝正常节点中故障节点的切片副本，让集群回复到正常状态。当网络分区恢复后，再重新将切片迁移回去。

