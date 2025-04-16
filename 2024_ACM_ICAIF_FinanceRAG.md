#### 比赛介绍

https://www.kaggle.com/competitions/icaif-24-finance-rag-challenge/overview

#### 数据集

https://huggingface.co/datasets/Linq-AI-Research/FinanceRAG

#### 解决方案

（1）[Multi-Reranker: Maximizing performance of retrieval-augmented generation in the FinanceRAG challenge](https://github.com/cv-lee/FinanceRAG)

![image](https://github.com/user-attachments/assets/ff0ec43e-539c-4cf6-a6f7-ab84d991e38c)


主要工作：

+ query侧做了扩展，包含3类。原始query，query中抽取的关键词，hyde

+ 二次rerank（jina+bge）

+ generation时按照token长度进行路由


(2)[Contextual RAG System with Hybrid Search and Reranking](https://github.com/chatterjeesaurabh/Contextual-RAG-System-with-Hybrid-Search-and-Reranking)

![image](https://github.com/user-attachments/assets/ecf3cf49-077a-4c30-aaf9-e116057ed810)

采用标准的混合搜索方案，其中基于RRF进行融合的策略如下：

**从每种方法中获取排名前5的结果；将两种方法的分数归一化到一个共同的范围（0-1）；使用权重参数对两种结果集中都出现的文档进行加权组合，并聚合分数。**

这样的逻辑是局部最优的。

