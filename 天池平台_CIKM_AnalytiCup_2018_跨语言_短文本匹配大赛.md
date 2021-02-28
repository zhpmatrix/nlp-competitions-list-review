**阿里小蜜机器人跨语言短文本匹配算法竞赛**

**任务**

基于聊天机器人中最常见的文本匹配算法，利用语言适应技术构建跨语言的短文本匹配模型

**数据**

源语言为英语，目标语言为西班牙语

提供训练数据集包含两种语言。20000个标注好的英语问句对作为源数据，1400个标注好的西班牙语问句对，以及55669个未标注的西班牙语问句。

数据格式：

英语问句对，有匹配标注：

英语问句1，西班牙语翻译1，英语问句2，西班牙语翻译2，匹配标注。

标注为1表示两个问句语义相同，0表示不同。

西班牙问句对，有匹配标注：

西班牙语问句1，英语翻译1，西班牙语问句2，英语翻译2，匹配标注。

标注为1表示两个问句语义相同，0表示不同。

无标注西班牙语问句对：

西班牙语问句1，英语翻译1，西班牙语问句2，英语翻译2

不同字段以”\t”符号分隔。

**评测指标**

使用 $\log loss$ 来评估性能。

假设 $y_*{i}$ 是标注答案，$p_*{i}$ 是样本 $x_{i}$ 的预测概率，那么可以定义 $\log loss$ 为：

![img](D:\duominuo\weixinobU7VjlxVTDz6HO47W1i7HUDaN7A\51288d68cbec4609a0f5178bd742638a\a.tfsprivate.png)

**top2方案思路**

1.特征工程：

最终使用三种特征：距离特征，主题特征和文本特征

距离特征，用三种方式构建:

Bag-of-words模型

带有TF/IDF的词袋模型

基于TF/IDF的加权平均词嵌入

主题特征：使用LDA和LSI对句子进行了矢量化，并用余弦距离比较两者的差异来捕捉

文本特征：文本特征都与句子的性质有关，例如:

句子的长度:句子越长，概率越低。

停止词和唯一词数:显示冗余程度。

词汇的多样性:它通常反映了一个诉求的复杂性。

2.模型构建：

使用三种模型：Decomposable attention、CPRNN、DACNN

Decomposable attention两个缺点：

抛弃了单词的顺序，意味着丢失了词组或单词之间的依赖关系等信息。因为比赛禁止使用任何其他外部数据。依赖关系信息一旦消失，就再也找不回来了

冷启动。发现谷歌译者倾向于将一些相似的概念概括成一个单一的概念，这导致训练数据缺乏词汇的多样性。然而，测试数据是原始的西班牙语句子，并不是那么简单。

因此，提出了另外两个模型CPRNN和DACNN。RNN模型用于重建依赖关系，CNN模型用于捕获区域语义表示

Compare-Propagare Recurrent Neural Network：

用MLP替换FM层，并将所有交互特性混合为一个向量。

由于对称的体系结构是泛化的关键，但在创建交互特征时，总是应用连接操作破坏了对称性，而连接特征对模型的性能起着至关重要的作用。因此， 为了同时掌握对称和连接特征，作者做了两次连接操作。第一次是[Sent1, Sent2]，第二次是[Sent2, Sent1]。最后，求这两个结果的平均值。

Densely Augmented Convolutional Neural Network：

设计了一个comparison loop，并将结果与原始嵌入重复连接，就像DenseNet一样。这是一个很好的特征重用和正则化的想法，使DACNN的运行速度比CPRNN快3倍，性能几乎相同。

CPRNN和DACNN模型输入：

Word level input、Character level input、Word level input with meta-features

实验发现bagging以上三种输入，使log loss显著有提升

![img](D:\duominuo\weixinobU7VjlxVTDz6HO47W1i7HUDaN7A\556175b7406f4ed28048fceb38f923e2\clipboard.png)

**参考**

第二名方案详解[https://tianchi.aliyun.com/forum/postDetailspm=5176.12586969.1002.3.316a7097TgUs3y&postId=11157](https://tianchi.aliyun.com/forum/postDetail?spm=5176.12586969.1002.3.316a7097TgUs3y&postId=11157)
