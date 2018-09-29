---
title: ElasticSearch查询报错fielddata=true
date: 2018-08-17 17:01:18
categories: 日常记录
tags: [日常记录, ElasticSearch]
---

记录下ElasticSearch查询报错的解决方案，今天在java中查询ElasticSearch的数据发生报错。

<!-- more -->
```
java.lang.IllegalArgumentException: Fielddata is disabled on text fields by default. Set fielddata=true on [pkgts_date] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory.
```

百度了一下。特汇总解决方法。
首先，看到这个报错。很自然的就去将pkgts_date字段设置  fielddata=true.但是这样会占用比较多的内存。导致性能下降。因此找到了另外的解决方法。
遇到这个错误是因为你尝试对一个text类型的字段做排序，而text类型的字段是要分词的。 一来词典很大，性能会很差；二来排序结果是词典里的词，而并非整个text的内容。 出于这2点原因，ES5.x以后对于text类型默认禁用了fielddata，防止对text字段一些错误的操作（排序，聚合，script)而给heap造成很大的压力。
如果一定有对该字段按照文本字母序做排序的需求，可以将该字段定义为multi-filed，例如:
```
    PUT my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "city": {
              "type": "text",
              "fields": {
                "raw": { 
                  "type":  "keyword"
                }
              }
            }
          }
        }
      }
    }
```

上面的city是text类型，适合做全文搜索，然后排序的时候可以用其keyword类型即city.raw。  这样排序结果是正确的，并且keyword字段是通过doc values排序的，内存消耗远小于fielddata。
