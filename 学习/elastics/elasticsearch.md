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

# 批量

就像 mget 允许我们一次性检索多个文档一样， bulk API允许我们使用单一请求来实现多个文档





# 别名

所以请做好准备：在应用中使用别名而不是索引。然后你就可以在任何时候重建索引。别名的开销很小，应当广泛使用。