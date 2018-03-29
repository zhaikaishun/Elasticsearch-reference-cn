## 搜索API  
现在让我们开始几个简单的搜索操作。Elasticsearch有两个基本的方法去进行搜索：一个是通过[REST request URI](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-uri-request.html)发送搜索的参数，另一个是通过[REST request body](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-request-body.html)。这种请求主体方法允许你更具表现力，并以更具可读性的JSON格式定义您的搜索。我们将尝试请求URI方法的一个示例，但在本教程的其余部分中，我们将专门使用请求主体方法。   
用于搜索的REST API可以从_search端点访问。本示例返回银行索引中的所有文档： 
```
GET /bank/_search?q=*&sort=account_number:asc&pretty
```
让我们先解析这个搜索请求：  
_search 前面有个bank，代表搜索bank索引下的数据
q=\*表示Elasticsearch匹配索引中的所有文档  
sort=account_number:asc表示搜索的结果按照account_num升序排序  
&pretty表示将JSON结果返回  

响应如下（部分展示）  
```
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```
在响应中，我们可以看到如下部分  
- **took**  Elasticsearch执行搜索的时间（以毫秒为单位）
- **timed_out**  告诉我们搜索是否超时
- **_shards**  告诉我们搜索了多少shards，以及搜索shards成功/失败的次数
- **hits**  搜索结果
- **hits.total** 符合我们搜索条件的文件总数
- **hits.hits** 实际的搜索结果数组（默认为前10个文档）
- **hits.sort** 为结果排序键（按分数排序时缺失）
- **hits._score并且max_score**  现在忽略这些字段  

下面是另外一种替代上面搜索的方法  
```
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```
这里的区别在于，不是在URI中传递q=*，而是将JSON样式的查询请求主体发布到_search API。我们将在下一节讨论这个JSON查询。
重要的是要明白，一旦你得到你的搜索结果，Elasticsearch完全完成了请求，并没有维护任何种类的服务器端资源或打开游标到你的结果。这与许多其他平台（例如SQL）形成鲜明对比，在其他平台中，你可能最初会先获取查询结果的部分数据，然后如果要得到其余部分的数据，则必须不断返回服务器的结果使用某种有状态的服务器端游标（例如数据库中的使用迭代的hashnext的方法）。  
## 查询语言简介  
Elasticsearch提供了JSON样式的查询语言，这被称为 [DSL查询](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl.html)。查询语言非常全面，乍一看可能会让人感到恐慌，但实际学习它的最好方法是从几个基本示例开始。 回到我们的最后一个例子，我们执行了这个查询：
```
GET /bank/_search
{
  "query": { "match_all": {} }
}
```
分析上面的语句  
**query**告诉我们查询的定义  
**match_all**部分仅仅是我们想要运行的查询类型。**match_all**就是说查询这个index下的所有文档  
另外，这个**query**部分，我们也可以传入一些其他的参数来影响搜索的结果。比如这里我们传入一个size，希望只查一条  
```
GET /bank/_search
{
  "query": { "match_all": {} },
  "size": 1
}
```
**注意**，如果没有设置size，大小默认是10  
下面这个例子是match_all并且返回第10到第19条：  
```
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```
from一般在分页的时候很有用，若没有指定，from默认是从0开始

下面这个自理是查询了所有，并且按照balance进行降序排序  
```
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```
## 执行搜索操作  
目前为止我们已经看懂啊了一些基本的搜索参数，现在让我们深入学习一下DSL查询语句。首先来看看返回的文档字段，默认情况下，返回的是所有的字段，如果我们不希望搜索的字段返回，我们可以在_source中添加想要返回的字段。  
```

```
这和SQL 的select field1，field2 from 有点类似  

此示例搜索account_num=20的账户 
```
GET /bank/_search
{
  "query": { "match": { "account_number": 20 } }
}
```

此示例返回address中包含术语"mill"的所有帐户：
```
GET /bank/_search
{
  "query": { "match": { "address": "mill" } }
}
```
本示例返回address中包含术语""mill或"lane"的所有帐户：
```
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```
本示例是**match (match_phrase)**的一个辩题，返回地址包含"mill lane"的账户:  
```
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```
现在我们来介绍一下这个**bool quere**。该bool quere允许我们将几个较小的查询，进行布尔逻辑的操作，合并成一个更大额查询。
本示例组成两个match查询并返回地址中包含"mill"和"lane"的所有帐户。  
```
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```
上述案例中"bool must" 指定了所有match必须都为true才能返回。  
  
相反，下面这个例子使用**should**查询，将返回地址中包含mill或者lane的账户  
```
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```
上述例子中，所有match中只要有一个为true即可返回  

下面这个例子组成的两个match操作将返回地址中既不包含"mill"，也不包含"lane"的操作。  
```
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```
在上面的例子中，**bool must not**指定了查询必须都不满足才能返回  

我们可以在一个bool查询中组合**must**, "should"和**must not** 。  
此外，我们可以在bool中，再写bool来实现更多复杂的布尔逻辑操作。  

下面这个例子返回40岁但是居住地（state 州）不在ID（爱达荷州）的所有账户。  
```
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

## 执行Filters操作  

在上一节中，我们跳过了一个称为文档分数（_score搜索结果中的字段）的细节。分数是一个数值，它是文档与我们指定的搜索查询匹配度的相对度量。分数越高，文档越相关，分数越低，文档的相关性越低。

但查询并不总是需要生成分数，特别是当它们仅用于“过滤”文档集时。Elasticsearch检测这些情况并自动优化查询执行，以便不计算无用分数  
我们在前一节中介绍的bool查询还支持filter使用查询来限制将被其他子句匹配的文档的子句，而不会更改计算分数的方式。作为一个例子，我们来介绍一下range查询，它允许我们通过一系列值来过滤文档。这通常用于数字或日期过滤。

此示例使用bool查询返回余额介于20000和30000之间的所有帐户。换句话说，我们希望查找余额大于或等于20000且小于等于30000的帐户。  
```
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```
解析上述内容，bool查询包含match_all查询（查询部分）和range查询（过滤器部分）。我们可以将任何其他查询替换为查询和过滤器部分。在上述情况下，范围查询非常有意义，因为落入该范围的文档全部匹配“平等”，即没有文档比另一个更相关。

除了match_all，match，bool，和range查询，有很多可用的其他查询类型的，我们不会进入他们在这里。由于我们已经对其工作原理有了基本的了解，因此将这些知识应用于学习和试用其他查询类型应该不会太困难。   


## 执行聚合操作
聚合提供了从数据中分组和提取统计数据的功能。考虑聚合的最简单方法是将其大致等同于SQL GROUP BY和SQL聚合函数。在Elasticsearch中，您可以执行返回匹配的搜索，同时还可以在一个响应中返回与匹配不同的聚合结果。这是非常强大和高效的，因为您可以运行查询和多个聚合，并使用简洁和简化的API避免网络往返，从而一次性获得两种（或两种）操作的结果。  

此示例按州的缩写（美国的地级划分单位）对所有帐户进行分组，然后按照默认的降序排序输出默认的10个：  
```
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```  
上面这个聚合操作，类似于sql中的  
```
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
```
响应如下（部分操作）  
```
{
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped" : 0,
    "failed": 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets" : [ {
        "key" : "ID",
        "doc_count" : 27
      }, {
        "key" : "TX",
        "doc_count" : 27
      }, {
        "key" : "AL",
        "doc_count" : 25
      }, {
        "key" : "MD",
        "doc_count" : 25
      }, {
        "key" : "TN",
        "doc_count" : 23
      }, {
        "key" : "MA",
        "doc_count" : 21
      }, {
        "key" : "NC",
        "doc_count" : 21
      }, {
        "key" : "ND",
        "doc_count" : 21
      }, {
        "key" : "ME",
        "doc_count" : 20
      }, {
        "key" : "MO",
        "doc_count" : 20
      } ]
    }
  }
}
```
我们可以看到有27个账户在ID（爱达荷州）这个州，其次是TX（德克萨斯州）的27个账户，其次是AL（阿拉巴马州）的25个账户，等等。
注意，我们将size=0设置为不显示搜索匹配，因为我们只想查看响应中的聚合结果。

下面的示例，我们将按照state.keyword进行分组，然后计算每组中的平均金额:  
```
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```
这个要注意我们是如何将"average_balance"聚合到按州分组后（group_by_state）的每个组里面去的。这就是嵌套聚合的模式，你可以嵌套任意的聚合操作，一边从数据中提取你想要的结果。  

基于前面的汇总，我们再加上一个降序排序的功能，示例如下  
```
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```
这个例子演示了我们如何按年龄段（20-29岁，30-39岁和40-49岁）进行分组，然后按性别进行分组，然后最终得出每个性别的年龄段平均账户余额：
```
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}

```

还有其他许多聚合功能，我们在这里不会详细介绍。进一步的学习请参考**[聚合指南](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-aggregations.html)**  


# 结论  
Elasticsearch既是一个简单又复杂的产品。现在，我们已经了解了它的基本原理，以及如何使用一些REST API进行处理的操作。希望本教程能够让你更好地理解Elasticsearch是什么，更重要的是，激发你去尝试更多的功能。  

































