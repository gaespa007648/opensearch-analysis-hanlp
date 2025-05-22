# opensearch-analysis-hanlp

HanLP Analyzer for Opensearch

[![GitHub release](https://img.shields.io/github/release/Anytinz/opensearch-analysis-hanlp.svg)](https://github.com/Anytinz/opensearch-analysis-hanlp/releases)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)

Forked from KennFalcon/elasticsearch-analysis-hanlp (https://github.com/KennFalcon/elasticsearch-analysis-hanlp)

此分词器基于[HanLP](http://www.hankcs.com/nlp)，提供了HanLP中大部分的分词方式。

KennFalcon: 开源不易，有空还是会跟进改动。

## 打包步驟

### 建立打包檔案

請執行以下指令打包plugin的zip檔案

./gradlew build

成功之後, 會在build/distributions底下產生"opensearch-analysis-hanlp-{distribution_version}-{opensearch_version}"的檔案。

### 更新Opensearch Plugin

這邊提供本地端更新docker建立的Opensearch plugin的流程, 此打包檔案是用來在opensearch新增tokenizer "hanlp"。

1. 複製檔案到docker-node:

   docker cp build/distributions/opensearch-analysis-hanlp-1.0.0-2.17.0.zip opensearch-node1:/tmp/

   請將上述zip檔案和docker-node的名稱換成自己環境下的名稱。
2. 進入指定的docker-node安裝plugin:

   docker exec -it opensearch-node1 /bin/bash

   ./bin/opensearch-plugin install file:///tmp/opensearch-analysis-hanlp-1.0.0-2.17.0.zip

## 更新版本

以下提供如何更新打包檔對應的Opensearch版本

### 安裝環境

1. 安裝sdkman: 你可以參考[官方安裝](https://sdkman.io/install/)或直接執行以下指令

   [安裝]

   curl -s "https://get.sdkman.io" | bash

   source "$HOME/.sdkman/bin/sdkman-init.sh"

   [測試]

   在終端機裡面測試以下指令是否成功

   sdk version
2. 安裝gradle: 使用sdkman安裝gradle

   [安裝]

   sdk install gradle 8.10.2

   sdk use gradle 8.10.2

   [測試]
   gradle -v

### 打包測試

首先, 你需要在gradle.properties設定要打包的opensearch版本

opensearchVersion = 2.17.0

接下來, 在主目錄下執行以下指令開始打包

gradle build

如果成功, 會在build/distributions底下出現相應的打包檔, 再來需要wrapper執行環境為gradlew檔案

先使用以下指令查看目前gradle的版本是什麼, 為GRADLE_VERSION

gradle -v

將GRADLE_VERSION帶入以下指令執行, 成功後會產生新的gradlew為統一環境下的安裝檔

gradle wrapper --gradle-version {GRADLE_VERSION} --distribution-type bin

---

Q: 如果打包失敗的話怎麼辦？

首先要先找出打包失敗的原因, 目前的經驗會是有以下可能性

1. gradle版本過舊, 需要參考Opensearch的gradle-wrapper的指定版本進行更新

   在我們的build.gradle當中, 可以看到我們會使用opensearch對應的build-tools來進行打包

   ```
   buildscript {
       repositories {
           gradlePluginPortal()
           mavenCentral()
           jcenter()
       }
       dependencies {
           classpath "org.opensearch.gradle:build-tools:${opensearchVersion}"
       }
   }
   ```

   需要debug的內容是要先檢查build-tool {opensearchVersion}對應的gradle版本, 這部分可以上github查看gradle wrapper。

   以下以2.17.0為例, 查看[opensearch-project/OpenSearch/gradle/wrapper](https://github.com/opensearch-project/OpenSearch/blob/2.17.0/gradle/wrapper/gradle-wrapper.properties)後發現distributionUrl裡面指定8.10的gradle版本。

   再來需要檢查repositories, 這個是抓套件的來源路徑, 需要確保相關套件皆可以在指定的repositories找到。前版2.6.0便缺少了gradlePluginPortal的路徑所以會有部分的套件找不到安裝來源。
2. 使用到deprecated的套件

   在此次升版的過程中, 有發現舊版本部分的套件在新版的bool-tool已經不支援。例如以下套件:

   import org.opensearch.core.internal.io.IOUtils;

   在後續版本中已不支援, 需全部替換成

   import org.opensearch.common.util.io.IOUtils;

   以上的訊息可在gradle build時加上-info的參數, 便可以印出來在打包時哪個流程會失敗來進行debug。

   gradle build -v

---

安装步骤
--------

### 1. 下载安装Opensearch对应Plugin Release版本

安装方式：

方式一

a. 下载对应的release安装包

b. 执行如下命令安装，其中PATH为插件包绝对路径：

`./bin/opensearch-plugin install file://${PATH}`

方式二

a. 使用opensearch插件脚本安装，command如下：

`./bin/opensearch-plugin install https://github.com/Anytinz/opensearch-analysis-hanlp/releases/download/v1.0.0/opensearch-analysis-hanlp-1.0.0-2.6.0.zip`

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
------------------

hanlp: hanlp默认分词

hanlp_standard: 标准分词

hanlp_index: 索引分词

hanlp_nlp: NLP分词

hanlp_crf: CRF分词

hanlp_n_short: N-最短路分词

hanlp_dijkstra: 最短路分词

hanlp_speed: 极速词典分词

样例
----

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
------------

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
--------------

HanLP在提供了各类分词方式的基础上，也提供了一系列的分词配置，分词插件也提供了相关的分词配置，我们可以在通过如下配置来自定义自己的分词器：

| Config                             | Opensearch version |
| :--------------------------------- | :----------------- |
| enable_custom_config               | 是否开启自定义配置 |
| enable_index_mode                  | 是否是索引分词     |
| enable_number_quantifier_recognize | 是否识别数字和量词 |
| enable_custom_dictionary           | 是否加载用户词典   |
| enable_translated_name_recognize   | 是否识别音译人名   |
| enable_japanese_name_recognize     | 是否识别日本人名   |
| enable_organization_recognize      | 是否识别机构       |
| enable_place_recognize             | 是否识别地名       |
| enable_name_recognize              | 是否识别中国人名   |
| enable_traditional_chinese_mode    | 是否开启繁体中文   |
| enable_stop_dictionary             | 是否启用停用词     |
| enable_part_of_speech_tagging      | 是否开启词性标注   |
| enable_remote_dict                 | 是否开启远程词典   |
| enable_normalization               | 是否执行字符正规化 |
| enable_offset                      | 是否计算偏移量     |

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
