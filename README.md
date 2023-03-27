# opensearch-analysis-hanlp
HanLP Analyzer for Opensearch

[![GitHub release](https://img.shields.io/github/release/Anytinz/opensearch-analysis-hanlp.svg)](https://github.com/Anytinz/opensearch-analysis-hanlp/releases)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)

Forked from KennFalcon/elasticsearch-analysis-hanlp (https://github.com/KennFalcon/elasticsearch-analysis-hanlp)

此分词器基于[HanLP](http://www.hankcs.com/nlp)，提供了HanLP中大部分的分词方式。

KennFalcon: 开源不易，有空还是会跟进改动。

安装步骤
----------

### 1. 下载安装Opensearch对应Plugin Release版本

安装方式：

方式一

a. 下载对应的release安装包

b. 执行如下命令安装，其中PATH为插件包绝对路径：

`./bin/opensearch-plugin install file://${PATH}`

方式二

a. 使用opensearch插件脚本安装，command如下：

`./bin/opensearch-plugin install https://github.com/Anytinz/opensearch-analysis-hanlp/releases/download/v2.6.0/opensearch-analysis-hanlp-2.6.0.zip`

### 2. 安装数据包

release包中存放的为HanLP源码中默认的分词数据，若要下载完整版数据包，请查看[HanLP Release](https://github.com/hankcs/HanLP/releases)。

数据包目录：*OPENSEARCH_HOME*/plugins/analysis-hanlp

**注：因原版数据包自定义词典部分文件名为中文，这里的hanlp.properties中已修改为英文，请对应修改文件名**

### 3. 重启Opensearch

**注：上述说明中的OPENSEARCH_HOME为自己的Opensearch安装路径，需要绝对路径**

### 4. 热更新

在本版本中，增加了词典热更新，修改步骤如下：

a. 在*OPENSEARCH_HOME*/plugins/analysis-hanlp/data/dictionary/custom目录中新增自定义词典

b. 修改hanlp.properties，修改CustomDictionaryPath，增加自定义词典配置

c. 等待1分钟后，词典自动加载

**注：每个节点都需要做上述更改**

提供的分词方式说明
----------

hanlp: hanlp默认分词

hanlp_standard: 标准分词

hanlp_index: 索引分词

hanlp_nlp: NLP分词

hanlp_crf: CRF分词

hanlp_n_short: N-最短路分词

hanlp_dijkstra: 最短路分词

hanlp_speed: 极速词典分词

样例
----------

```text
POST http://localhost:9200/twitter2/_analyze
{
  "text": "美国阿拉斯加州发生8.0级地震",
  "tokenizer": "hanlp"
}
```

```json
{
  "tokens" : [
    {
      "token" : "美国",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "nsf",
      "position" : 0
    },
    {
      "token" : "阿拉斯加州",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "nsf",
      "position" : 1
    },
    {
      "token" : "发生",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "v",
      "position" : 2
    },
    {
      "token" : "8.0",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "m",
      "position" : 3
    },
    {
      "token" : "级",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "q",
      "position" : 4
    },
    {
      "token" : "地震",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "n",
      "position" : 5
    }
  ]
}
```

远程词典配置
----------

配置文件为*OPENSEARCH_HOME*/config/analysis-hanlp/hanlp-remote.xml

```xml
<properties>
    <comment>HanLP Analyzer 扩展配置</comment>

    <!--用户可以在这里配置远程扩展字典 -->
    <entry key="remote_ext_dict">words_location</entry>

    <!--用户可以在这里配置远程扩展停止词字典-->
    <entry key="remote_ext_stopwords">stop_words_location</entry>
</properties>
```

### 1. 远程扩展字典

其中words_location为URL或者URL+" "+词性，如：

    1. http://localhost:8080/mydic
    
    2. http://localhost:8080/mydic nt

第一个样例，是直接配置URL，词典内部每一行代表一个单词，格式遵从[单词] [词性A] [A的频次] [词性B] [B的频次] ... 如果不填词性则表示采用词典的默认词性n。

第二个样例，配置词典URL，同时配置该词典的默认词性nt，当然词典内部同样遵循[单词] [词性A] [A的频次] [词性B] [B的频次] ... 如果不配置词性，则采用默认词性nt。

### 2. 远程扩展停止词字典

其中stop_words_location为URL，如：

    1. http://localhost:8080/mystopdic

样例直接配置URL，词典内部每一行代表一个单词，不需要配置词性和频次，换行符用 \n 即可。


**注意，所有的词典URL是需要满足条件即可完成分词热更新：**

- 该 http 请求需要返回两个头部(header)，一个是 Last-Modified，一个是 ETag，这两者都是字符串类型，只要有一个发生变化，该插件就会去抓取新的分词进而更新词库。

- 可以配置多个字典路径，中间用英文分号;间隔

- URL每隔1分钟访问一次

- 保证词典编码UTF-8

自定义分词配置
----------

HanLP在提供了各类分词方式的基础上，也提供了一系列的分词配置，分词插件也提供了相关的分词配置，我们可以在通过如下配置来自定义自己的分词器：

| Config                               | Opensearch version  |
| :----------------------------------- | :------------------ |
| enable_custom_config                 | 是否开启自定义配置    |
| enable_index_mode                    | 是否是索引分词        |
| enable_number_quantifier_recognize   | 是否识别数字和量词    |
| enable_custom_dictionary             | 是否加载用户词典      |
| enable_translated_name_recognize     | 是否识别音译人名      |
| enable_japanese_name_recognize       | 是否识别日本人名      |
| enable_organization_recognize        | 是否识别机构         |
| enable_place_recognize               | 是否识别地名         |
| enable_name_recognize                | 是否识别中国人名      | 
| enable_traditional_chinese_mode      | 是否开启繁体中文      |
| enable_stop_dictionary               | 是否启用停用词        |
| enable_part_of_speech_tagging        | 是否开启词性标注      |
| enable_remote_dict                   | 是否开启远程词典      |
| enable_normalization                 | 是否执行字符正规化    |
| enable_offset                        | 是否计算偏移量        |

注意： 如果要采用如上配置配置自定义分词，需要设置enable_custom_config为true

例如：
```text
PUT test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_hanlp_analyzer": {
          "tokenizer": "my_hanlp"
        }
      },
      "tokenizer": {
        "my_hanlp": {
          "type": "hanlp",
          "enable_stop_dictionary": true,
          "enable_custom_config": true
        }
      }
    }
  }
}
```

```text
POST test/_analyze
{
  "text": "美国,|=阿拉斯加州发生8.0级地震",
  "analyzer": "my_hanlp_analyzer"
}
```

结果：
```text
{
  "tokens" : [
    {
      "token" : "美国",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "nsf",
      "position" : 0
    },
    {
      "token" : ",|=",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "w",
      "position" : 1
    },
    {
      "token" : "阿拉斯加州",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "nsf",
      "position" : 2
    },
    {
      "token" : "发生",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "v",
      "position" : 3
    },
    {
      "token" : "8.0",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "m",
      "position" : 4
    },
    {
      "token" : "级",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "q",
      "position" : 5
    },
    {
      "token" : "地震",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "n",
      "position" : 6
    }
  ]
}

```
