#### 比赛地址

[点击这里](https://www.kaggle.com/competitions/WattBot2025)

#### 比赛介绍

WattBot 2025挑战赛聚焦**AI能耗领域的文档-grounded技术问答**，要求参赛系统基于指定参考文档回答问题，同时满足严苛的精度要求。该比赛由Kaggle平台举办，是RAG技术在垂直领域应用的重要评测基准。

#### 任务类型

文档问答（Document-Grounded QA）+ RAG（检索增强生成）

#### 比赛背景

随着AI模型规模不断增大，AI能耗问题（包括电力消耗、水耗、碳足迹等）成为研究热点。该比赛旨在评估AI系统在技术文档理解和问答方面的能力，特别强调**答案准确性、引用精确性和幻觉避免**三个方面。

#### 数据分析

##### 参考文档
- **文档数量**：32份AI能耗相关技术文档
- **文档类型**：研究论文、技术报告、演示幻灯片
- **数据规模**：约500K tokens
- **涵盖主题**：AI模型的电力消耗、水耗、碳足迹、性能特性等

##### 测试问题
- **训练集**：41道题
- **测试集**：282道题（公共分区66%、私有分区34%）
- **问题类型**：涵盖功耗、水耗、碳足迹等AI能耗指标
- **答案要求**：需从参考文档中提取答案或推断结论

##### 输出格式
系统需针对每个问题输出三元组**(a, R, b)**：

|字段|描述|要求|
|------|------|------|
|a（答案）|数值型或分类型答案|数值答案误差需控制在±0.1%以内，分类答案需精确匹配|
|R（引用集）|答案对应的源文档ID|从32份文档中准确选取，需与真实引用集保持高相似度|
|b（弃权标记）|显式弃权标记|b=1时表示弃权（仅当无足够证据时使用），b=0时表示有效答案|

#### 评估指标

最终分数按三部分加权计算：

|指标|权重|计算方式|
|------|------|------|
|答案准确率|75%|数值答案误差≤0.1%得满分，分类答案精确匹配得满分|
|引用精度|15%|用杰卡德相似度（|R∩R*|/|R∪R*|）衡量预测引用与真实引用的重合度|
|幻觉避免|10%|证据不足时弃权得满分，无证据作答或引用错误将受惩罚|

#### 方案复盘：KohakuRAG

##### 核心思路

KohakuRAG是一个**基于四层树形层级文档索引（文档→章节→段落→句子）的轻量级RAG框架**，通过自底向上的嵌入聚合保留文档结构、LLM驱动的查询规划+跨查询重排序提升检索覆盖率、感知弃权的集成推理投票稳定答案输出。

最终在**WattBot 2025挑战赛中以0.861的最终分数获公共和私有排行榜第一**。

##### 技术架构

![KohakuRAG架构图](https://mmbiz.qpic.cn/mmbiz_png/wUtXN4IVhDHWFlEz1EZ1U3q7F5KZBr47ZWkvtyK0Gq51hgt5hicvRCCQCic9jbsmrkv6Sia9mjTZVAz24wibf33SkJwZs7W4Lk0WmSicVhwrOia0/640?wx_fmt=png&from=appmsg)

##### 1. 离线层级文档索引

**文档解析方式**：
- 将文档构建为**四层树形结构（文档→章节→段落→句子）**
- 图像/表格作为特殊段落节点
- 由**Qwen3-VL生成标题**（使用PyMuPDF提取图像，通过Qwen3-VL生成标题，聚焦技术内容、数据值、坐标轴标签等）

**解析结果**：
- 639个章节
- 944个段落
- 24,565个句子
- 280张图像

**解析技术特点**：
通过**规则化启发式方法**实现，依赖PyMuPDF工具做基础文本/位置信息提取，再通过字体、间距、格式等规则划分层级：

- **章节检测规则**：字体大小≥中位数1.2倍的文本块视为潜在章节标题，带编号模式（如1.2、A.3）的进一步确认章节边界
- **段落检测规则**：垂直间距<1.5倍行高的连续文本块合并为段落，大间距为段落边界
- **句子分割规则**：基于NLTK的sent_tokenize，结合技术文本定制规则（处理Fig.、etal.、i.e.等缩写）

**嵌入计算方式**：
采用**自底向上的长度加权嵌入聚合**策略：
- 叶子节点（句子）直接编码
- 内部节点为子节点嵌入的**token数加权平均值**
- 嵌入模型采用**Jina v3**（纯文本，768维，支持8K上下文）和**Jina v4**（多模态，128-2048维Matryoshka表示，支持文本+直接图像嵌入）
- 向量存储基于**sqlite-vec**的轻量级存储

##### 2. 在线多查询检索

**LLM查询规划**：
- 将原问题扩展为n个语义相关查询
- 覆盖替代表述、缩写展开、子查询分解等

**稠密检索**：
- 对每个规划查询做余弦相似度检索
- 仅针对句子/段落节点

**混合检索**：
- 可选BM25稀疏检索增强
- 仅对段落索引，结果直接拼接在稠密检索后

**跨查询重排序**：
用频率、分数、组合三种策略，优先检索被多个查询命中的节点：

![跨查询重排序](https://mmbiz.qpic.cn/sz_mmbiz_png/wUtXN4IVhDE08fUMCVLuKlmiciar3DKiaerhcsmluH1syuBgl9JrT67PXYNuxbiat8H3d3NyB0f7SfC5MdLWovVtZQuwSd8dYDVGNHtAFD19LpE/640?wx_fmt=png)

**层级上下文扩展**：
为命中节点补充父节点和兄弟节点，同时做树形去重减少冗余：

![层级上下文扩展](https://mmbiz.qpic.cn/mmbiz_png/wUtXN4IVhDHfiaqBGHzQNhJk8Kq3ZxrrWWUaFrWbkSJMIGFbic1JqicoN7ZSRk61fsraW6Q4iaiapmbEofTobI7ZncBx2kYRJ3VMmAkPrSNbZ6T4/640?wx_fmt=png)

##### 3. 答案生成Pipeline

**提示词构建**：
- 将上下文置于问题前（重排序），解决"迷失在中间"问题
- 格式化引用标记便于LLM归因

![提示词构建](https://mmbiz.qpic.cn/mmbiz_png/wUtXN4IVhDFF3nXic1sxmeUSnJ4Lz5umo60TcrjJEv6Rs6tqhNf8vvl5Vpxj14Mu1sItB00FVLXUdXxz6dYGQx0Ds3yXcGrTSolw3zj3vSc0/640?wx_fmt=png)

**结构化输出**：
- 让LLM输出含答案、提取值、引用ID、解释、是否空白的JSON
- 证据不足时显式弃权

**重试机制**：
- 当LLM输出空白时，增大检索k值重新检索
- 直至得到有效答案或达到重试上限

![重试机制](https://mmbiz.qpic.cn/sz_mmbiz_png/wUtXN4IVhDHuc8912sgjZmZictBocqRljT2SQ3rFD1XDGb4icfAd2Xu5lYaNDuZGAiaIGWDPsqVZOd3kpAMw0iamhRf8TxAKspwF0mdSaz3cPsQ/640?wx_fmt=png)

**集成推理**：
- m次独立推理（温度>0）
- 过滤空白响应后进行多策略投票
- 支持AnswerPriority/Independent/Union等策略，其中AnswerPriority表现最优

![集成推理](https://mmbiz.qpic.cn/sz_mmbiz_png/wUtXN4IVhDGZ4AJaeGeicAdwBqU0akmoRAUocjazdJ9M63XTJhVQGFtoTO5sfHIBu2IdI6Oe9GibGzD2Lx44Jyr9gUERfdZHZ7xwrlBsc1UWY/640?wx_fmt=png)

**数值归一化**：
- 对数值答案解析科学计数法、统一单位
- ±0.1%内视为等效
- 因为有准确性要求，标准化是最好的方式

#### 技术亮点

1. **层级索引结构**：通过四层树形结构保留文档的语义层级关系，相比扁平索引能更好地捕捉上下文信息

2. **自底向上聚合**：采用token数加权平均的嵌入聚合策略，使得上层节点能够继承下层节点的语义信息

3. **多查询扩展**：通过LLM驱动的查询规划，从多个角度检索相关内容，提高召回率

4. **集成推理**：通过多次独立推理和投票机制，稳定答案输出，降低幻觉风险

5. **感知弃权**：在证据不足时主动弃权，避免无证据作答，这在实际应用中非常重要

#### 技术限制

1. **依赖规则化解析**：通过规则化启发式方法实现文档解析，对非标准布局文档鲁棒性较差

2. **通用性限制**：由于依赖特定格式的文档结构，在不同类型文档上的泛化能力有限

3. **计算复杂度**：多层级的检索和重排序机制增加了计算开销

#### 思考与总结

这个方案是典型的**打榜思路**，体现了以下几个重要特点：

1. **系统集成**：通过精巧的系统设计，将文档解析、检索、生成、集成等多个环节有机结合

2. **细节优化**：在每个环节都做了细致的优化，如提示词构建、重试机制、集成推理等

3. **评估对齐**：方案设计紧密对齐评估指标，特别是对引用精度和幻觉避免的针对性优化

4. **工程实践**：体现了RAG系统在实战中的各种工程技巧和trick

与之前的相关工作（如RAPTOR、BookRAG、PageIndexRAG等）相比，KohakuRAG在**集成推理**和**感知弃权**方面有独特创新，这也是其能够在竞赛中获得第一名的重要原因。

#### 参考文献

1. [KohakuRAG: A simple RAG framework with hierarchical document indexing](https://arxiv.org/pdf/2603.07612)
2. [KohakuRAG代码实现](https://github.com/KohakuBlueleaf/KohakuRAG)
3. [WattBot 2025挑战赛](https://www.kaggle.com/competitions/WattBot2025)