---
title: Elasticsearch学习笔记1-分词
date: 2018-05-30 14:24:00
categories:
  - 数据库
  - Elasticsearch
tags:
  - elasticsearch
  - analysis
---

分词是将文本转换成一系列单词(```terms```或```tokens```)的过程，也叫做文本分析，在```Elasticsearch```里面称为```Analysis```。

<!--more-->

## 分词器```Analyzer```

分词器是```Elasticsearch```中专门处理分词的组件，它的组成如下，

### Character Filters

针对原始文本进行处理，比如去除html标签等。可以有一个或者多个，分词器会按照顺序处理。

### Tokenizer

讲原始文本按照一定的规则切分为单词，<u>```tokenizer```只能有一个</u>。

### Token Filters

针对tokenizer处理的单词进行再加工，比如转小写、删除或者新增等处理。可以有一个或者多个，分词器会按照顺序处理。

示例，

使用```Elasticsearch```自带的[Whitespace Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-whitespace-analyzer.html)，

```http
POST _analyze
{
    "analyzer": "whitespace",
    "text": "the quick brown fox."
}
```

使用组件自己组一个Analyzer，

```http
POST _analyze
{
    "tokenizer": "standard",
    "filter": ["lowercase","asciifolding"],
    "text": "Is this déja vu?"
}
```

[参考文档：Anatomy of an analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html#analyzer-anatomy)

```Elasticsearch```默认有以下几种自带的分词器，[Standard Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html), [Simple Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-simple-analyzer.html), [Whitespace Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-whitespace-analyzer.html), [Stop Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-analyzer.html), [Keyword Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-analyzer.html), [Pattern Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-analyzer.html), [Language Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html), [Fingerprint Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-fingerprint-analyzer.html)

## 自定义分词

当自带的分词无法满足需求时，可以通过自定义```Character Filters```、```Tokenizer```和```Token Filter```组件来实现自定义分词。

这三个组件```Elasticsearch```都有提供一些自带的组件，可以参考下面文档：

[Character Filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html)

[Tokenizers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)

[Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html)

自定义一个分词器，

```http
PUT test_index
{
    "settings": {
        "analysis": {
            "analyzer": {
                "my_custom_analyzer": {
                    "type": "custom",
                    "tokenizer": "standard",
                    "char_filter": [
                        "html_strip"
                    ],
                    "filter": [
                        "lowercase",
                        "asciifolding"
                    ]
                }
            }
        }
    }
}
```

## 中文分词

### 常用分词系统

#### IK

- 实现中英文单词切分，支持```ik_smart```,```ik_maxword```等模式；
- 可自定义词库，支持热更新分词词典；

[https://github.com/medcl/elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)

#### jieba

- ```python```中最流行的分词系统，支持分词和词性标注；
- 支持繁体分词、自定义词典、并行分词等；

[https://github.com/sing1ee/elasticsearch-jieba-plugin](https://github.com/sing1ee/elasticsearch-jieba-plugin)

### 基于自然语言处理的分词系统

#### Hanlp

https://github.com/hankcs/HanLP

#### THULAC

https://github.com/microbun/elasticsearch-thulac-plugin