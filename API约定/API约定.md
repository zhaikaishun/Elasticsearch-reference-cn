# API约定
ES的REST API都是通过JSON和HTTP来发送和接受数据
除非另有说明，否则本章中列出的约定可以应用于整个REST API。  

//TODO 链接地址变化一下
- [多个索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-index.html)
- [索引名称中的日期数学支持](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-math-index-names.html)
- [常用选项](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html)
- [基于URL的访问控制](https://www.elastic.co/guide/en/elasticsearch/reference/current/url-access-control.html)

## 多个索引  

大多数引用index参数的API 支持跨多个索引执行，使用简单的test1,test2,test3符号（或_all所有索引）。它还支持通配符，例如：test*或*test或te*t或*test*，以及“排除”（-）的能力，例如：test*,-test3。  

所有多索引API都支持以下url查询字符串参数：  

**ignore_unavailable**    
控制是否忽略任何指定的索引不可用，这包括不存在的索引或闭合的索引。无论是true或false 可以指定。   
**allow_no_indices**   
控制如果通配符索引表达式不生成具体索引，是否失败。无论是true或false可以指定。例如，如果foo*指定了通配符表达式，并且没有可用的索引，foo然后根据此设置开始，则请求将失败。此设置也适用时_all，*或没有指数已经指定。此设置也适用于别名，以防别名指向封闭索引。  

**expand_wildcards**  
控制什么样的具体指数通配符指数表达式扩展到。如果open被指定，那么通配符表达式被扩展为仅打开索引，如果closed被指定，则通配符表达式仅扩展到闭合索引。也open,closed可以指定两个值（）以扩展到所有索引。

如果none指定，则通配符扩展将被禁用，如果all 指定，通配符表达式将扩展到所有索引（这相当于指定open,closed）。

上述参数的默认设置取决于正在使用的api。

## 索引名称对日期的数学运算的支持  
日期数学索引名称解析使您可以搜索一系列时间序列索引，而不是搜索所有时间序列索引并过滤结果或保留别名。限制搜索索引的数量会减少集群的负载并提高执行性能。例如，如果您要在日常日志中搜索错误，则可以使用日期数学名称模板将搜索限制在过去两天。  

几乎所有具有index参数的API都支持index参数值中的数学运算。  

日期数学索引名称采用以下形式：  
```
<static_name{date_math_expr{date_format|time_zone}}>
```
其中：  
- **static_name**: 是名称的静态文本部分
- **date_math_expr**: 是动态计算日期的动态日期数学表达式
- **date_format**: 是应该呈现计算日期的可选格式。默认为YYYY.MM.dd。
- **time_zone**: 是可选时区。默认为utc。
你必须将日期数学索引名称表达式包含在尖括号中，并且所有特殊字符都应该使用URI编码。例如：
```
# GET /<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```
**注意**如果是一些特殊符号，需要进行如下编码转换  
```
<	%3C
>	%3E
/	%2F
{	%7B
}	%7D
|	%7C
+	%2B
:	%3A
,	%2C
```
以下示例显示了日期数学索引名称的不同形式，以及在当前时间为2024年3月22日中午utc时解析的最终索引名称。  
```
表达	解决了
<logstash-{now/d}>    					logstash-2024.03.22

<logstash-{now/M}>    					logstash-2024.03.01

<logstash-{now/M{YYYY.MM}}>     		logstash-2024.03

<logstash-{now/M-1M{YYYY.MM}}>  		logstash-2024.02

<logstash-{now/d{YYYY.MM.dd|+12:00}}>   logstash-2024.03.23

```
特殊的**{**和**}**, 可以用反斜杠来转义  
```
<elastic\\{ON\\}-{now/M}> resolves to elastic{ON}-2024.03.01
```
以下示例显示了一个搜索请求，假设Logstash索引使用默认的名称格式 logstash-YYYY.MM.dd。我们想要搜索Logstash索引三天内的数据  
```
# GET /<logstash-{now/d-2d}>,<logstash-{now/d-1d}>,<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd-2d%7D%3E%2C%3Clogstash-%7Bnow%2Fd-1d%7D%3E%2C%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```  

## 常用选项  
以下选项可以应用于所有REST API。  

### Pretty Results  
当添加 **?pretty=true** 到任何的request时，返回的JSON格式将变成格式化（仅用于调试！）。另一个选择是设置**?format=yaml**，返回的结果将是更易读的yaml格式。  
### 人类可读的输出  
统计数据以适合人类（例如"exists_time": "1h"或"size": "1kb"）和计算机（例如"exists_time_in_millis": 3600000或"size_in_bytes": 1024）的格式返回。可以通过添加?human=false 来关闭可读取的值。当统计结果被监测工具消耗而不是用于人类消耗时，这是有意义的。human标志的默认值是 false。
### 日期数学  
对于日期这种格式，可以使用大于和小于，例如gt和lt。还可以使用范围的查询，例如from和to 你可以在[daterange聚集](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-daterange-aggregation.html)  中了解更多。  
简单的例子  
- **+1h** 加一小时
- **-1d** 减去一天
- **/d** 一直到最近的一天
下面是支持的一些简写，更多简写请参考 [简写](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#time-units)  

```
y	years

M	months

w	weeks

d	days

h	hours

H	hours

m	minutes

s   seconds
```
假设now是2001-01-01 12:00:00，一些例子如下    
now+1h: 变成：2001-01-01 13:00:00  
now-1h:	变成：2001-01-01 11:00:00  
now-1h/d： now以毫秒减1小时，向下舍入到UTC 00:00 变成: 2001-01-01 00:00:00    
2001.02.01\|\|+1M/d: 2001-02-01以毫秒加一个月。变成：2001-03-01 00:00:00  

###  响应设置(Response Filtering)
所有的REST API都接受filter_path这个参数，这个参数可以减少Elasticsearch返回的信息。使用例子如下  
```
GET /_search?q=elasticsearch&filter_path=took,hits.hits._id,hits.hits._score
```
响应  
```
{
  "took" : 3,
  "hits" : {
    "hits" : [
      {
        "_id" : "0",
        "_score" : 1.6375021
      }
    ]
  }
}
```
它还提供通配符*的功能  
```
GET /_cluster/state?filter_path=metadata.indices.*.stat*
```
响应如下  
```
{
  "metadata" : {
    "indices" : {
      "twitter": {"state": "open"}
    }
  }
}
```  
### 列表显示设置  
flat_settings可以设置返回的列表的格式，默认是false，下面是两个例子  
```
GET twitter/_settings?flat_settings=true
```
返回  
```
{
  "twitter" : {
    "settings": {
      "index.number_of_replicas": "1",
      "index.number_of_shards": "1",
      "index.creation_date": "1474389951325",
      "index.uuid": "n6gzFZTgS664GUfx0Xrpjw",
      "index.version.created": ...,
      "index.provided_name" : "twitter"
    }
  }
}
```
若设置为false  
```
GET twitter/_settings?flat_settings=false
```
返回  
```
{
  "twitter" : {
    "settings" : {
      "index" : {
        "number_of_replicas": "1",
        "number_of_shards": "1",
        "creation_date": "1474389951325",
        "uuid": "n6gzFZTgS664GUfx0Xrpjw",
        "version": {
          "created": ...
        },
        "provided_name" : "twitter"
      }
    }
  }
}
```  
**还有一些常用选项，直接看官方文档吧:https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#time-units **  

## URL访问控制  
许多用户使用具有基于URL的访问控制的代理来保护对Elasticsearch索引的访问。对于多重搜索， 多重获取和批量请求，用户可以选择在URL中指定一个索引，也可以在请求正文中指定每个请求。这可以使基于URL的访问控制具有挑战性。  

为了防止用户覆盖已在URL中指定的索引，请将此设置添加到elasticsearch.yml文件中：  
```
rest.action.multi.allow_explicit_index：false
```
默认值是true，但设置false为时，Elasticsearch将拒绝在请求正文中指定具有明确索引的请求。