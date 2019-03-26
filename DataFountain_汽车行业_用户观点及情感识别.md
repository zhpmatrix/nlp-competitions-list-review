### 比赛背景

数据为用户在汽车论坛中对汽车相关内容的讨论和评价，典型的如汽车之家(该平台是一个大的数据来源地，值得注意)。

### 数据说明

|字段名称|类型|描述|说明|
|------|------|------|------|
|content_id|Int|数据ID|/|
|content|String|文本内容|/|
|subject|String|主题|提取或者依据上下文归纳出来的主题|
|sentiment_value|Int|情感分析|分析出的情感|
|sentiment_word|String|情感词|情感词|

其中subject包括10类：动力，价格，内饰，配置，安全性，外观，操控，油耗，空间，舒适性。

sentiment\_value包括三类：中立(0)，正向(1)和负向(-1)。

每个content\_id可能对应多个subject，每个subject一行记录。

sentiment\_word大部分为空。

测试时的输入字段为content\_id和content，输出字段包括subject，sentiment\_value和sentiment\_word。

### 评估指标

F1得分，只对subject和sentiment\_value进行评估，忽略sentiment\_word。

### 思路分析

已经给定了主题的种类和数目，同时一个content可能对应多个subject。一种典型的建模思路是多标签分类。给定content和subject，接下来就是情感分类了(多分类问题)，可以将content和subject嵌入后直接分类。

### 方案复盘

#### 1.模型

冠军方案的思路和上述思路分析一致，模型上主要采用BERT，以及CNN/RNN等其他模型，借助LR以Stacking的方式融合多个模型。具体方案如下：

![topic](http://wx3.sinaimg.cn/mw690/aba7d18bgy1g1g00d2nbfj20fk0f1tak.jpg)

其中，Multi-Label Multi-Attention Model中的Multi-Attention是指第一：Lable Embedding做一个Attention，目的是建立Label之间的关系；第二，每个Label也有一个Attention过程，目的是学习到Label对应的句子表示。

![sent](http://wx3.sinaimg.cn/mw690/aba7d18bgy1g1fzztadt8j20le0f1jts.jpg)

上图中，AT\_LSTM在自己的2017年的一个比赛中也用到，单模型做到了排名第二的成绩。HEAT和GCAE也分别是两个模型，关于模型细节就不多说了，可以参考具体文献。

#### 2.词向量(多种)

具体包括：[Chinese-Word-Vectors](https://github.com/Embedding/Chinese-Word-Vectors)，[Word vectors for 157 languages](https://fasttext.cc/docs/en/crawl-vectors.html)，[Tencent AI Lab Embedding Corpus for Chinese Words and Phrases](https://ai.tencent.com/ailab/nlp/embedding.html)，[HIT-SCIR，ELMoForManyLangs](https://github.com/HIT-SCIR/ELMoForManyLangs)

上述其实是一个中文预训练词向量的列表，可以用在很多地方，冠军方案中用了这四种。

#### 3.思考

赛题还是一个分类问题。从作者的开源代码Readme中，作者说单模型BERT的评估指标就已经很好了，和融合方案的差距很小，再次证明BERT的强大。冠军用了很多新的工作，要开放心态，做新模型和新方法的尝试。


### 参考

1.[冠军方案，代码](https://github.com/yilirin/BDCI_Car_2018)




