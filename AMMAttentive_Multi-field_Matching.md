# AMM: Attentive Multi-field Matching for News Recommendation

AMM：面向新闻推荐的注意力多字段匹配

## 摘要

​	个性化新闻推荐是帮助用户找到感兴趣的新闻的关键技术，如何精准匹配用户的兴趣和候选新闻是新闻推荐的核心。现有研究通常通过聚合用户浏览过的新闻来学习用户的兴趣向量，然后将其与候选新闻向量进行匹配，这可能会丢失用于推荐的文本语义匹配信号。在本文中，我们提出了一种用于新闻推荐的注意力多字段匹配（AMM）框架，该框架捕获每个浏览的新闻和候选新闻之间的语义匹配表示，然后将它们聚合为最终的用户新闻匹配信号。此外，我们的方法结合了多字段信息并设计了字段内和跨字段匹配机制，该机制利用来自不同字段（例如标题、摘要和正文）的互补信息并获得多字段匹配表示。为了实现全面的语义理解，我们使用最流行的语言模型 BERT 来学习每个浏览候选新闻对的匹配表示，并在聚合过程中结合注意力机制来表征每个匹配表示对最终用户新闻的重要性匹配信号。在真实世界数据集上的实验验证了 AMM 的有效性。

## 引言

​	Google News1 和 MSN News2 等在线新闻平台旨在提供个性化的新闻服务，缓解用户的信息过载 [3, 7]。 每天产生的大量新闻使得用户很难找到他们感兴趣的新闻。 为了减轻信息过载并改善用户的在线阅读体验，推荐系统成为这些平台不可或缺的一部分。

​	用户兴趣与候选新闻的准确匹配是准确的个性化新闻推荐的前提。现有的方法主要是通过顺序或注意力模型聚合用户之前浏览过的新闻来学习用户的兴趣向量，然后将其与候选新闻向量进行匹配，取得了相当大的进展。例如，NPA [12] 根据候选新闻和之前点击的新闻之间的相似性来学习用户表示。 LSTUR [1] 使用 GRU 网络从点击的新闻中模拟短期和长期的用户兴趣。 NAML [11] 和 NRMS [13] 应用注意力网络从点击的新闻中学习用户表示。 FIM [9] 为每个新闻提取多级表示，并通过卷积执行细粒度匹配。然而，这些方法中的大多数将每个用户和新闻表示为单个向量，这可能会丢失文本匹配信号（例如，词级关系）。因此，我们的方法不是简单地将用户浏览的新闻建模为一个整体，而是关注每个浏览的候选新闻对之间的关系，以捕获细粒度的语义匹配表示。此外，这种对设计是在线服务友好的，匹配的表示可以离线存储，以避免重复计算。在这里，我们首先使用 BERT 来学习每个浏览候选新闻对在不同语义级别的匹配表示，然后将它们聚合到最终的用户新闻匹配信号。

​	此外，上述方法基于单视图新闻信息[1,9,13]或多视图信息[11]学习新闻之间的语义相关性，它们仅利用字段内语义相关性（例如，标题-标题，正文-正文）。 但有时，跨领域信息（例如标题-摘要、标题-正文）可能有利于新闻匹配。 比如新闻𝑎的标题信息丰富，而新闻𝑏的标题很吸引人，新闻𝑎的标题和新闻𝑏的正文匹配可能更好。 因此，与每条新闻相关的多个字段可能包含互补信息，这些信息促使我们学习多字段（字段内和跨字段）匹配表示。

​	在本文中，我们提出了一种注意力多字段匹配（AMM）框架，该框架捕获每个浏览候选新闻对的多字段语义匹配表示，然后将这些表示聚合为最终的用户新闻匹配信号，用于新闻推荐。 这项工作的主要贡献总结如下：

- 我们提出了一种新的方法，专注于分离的浏览候选新闻对的匹配，以捕获文本语义匹配信号，这对在线服务很友好。
- 我们设计了 AMM 框架，以在领域内和跨领域的方式中提取多领域匹配表示，以深入探索用户的兴趣。
- 我们对两个公共基准数据集进行了实验，以证明我们方法的有效性。

## 我们的方法

​	图 1 显示了 AMM 的整体架构。 首先，我们为每个浏览的候选新闻构建多字段对输入。 然后，匹配编码器用于提取每对输入的匹配表示。 最后，我们将所有对的匹配表示聚合到用户新闻信号以估计点击概率。 我们将在以下小节中介绍每个组件。

### 2.1 多头部匹配

​	不同的新闻字段通常可以相互补充。 为了更好地编码浏览-候选匹配，我们利用字段内和跨字段匹配作为多字段信息来增强匹配表示。

​	给定用户浏览的新闻$\left[N_{1}, N_{2}, \ldots, N_{i}\right]$和候选新闻$N_x$，对于每个浏览-候选对$\left(N_{i}, N_{c}\right)$，我们建立多头部对输入$\left[N_{i}^{m}, N_{c}^{n}\right]$，其中$m \in\{t, a, b\},n \in\{t, a, b\}$，t、a、b分别是新闻的标题、摘要、主体。我们将新闻标题$N^t$表示为词向量$\left[w_{1}^{t}, w_{2}^{t}, \ldots, w_{\left|N^{t}\right|}^{t}\right]$，新闻摘要$N^a$表示为$\left[w_{1}^{a}, w_{2}^{a}, \ldots, w_{|N a|}^{a}\right]$，新闻主体$N^b$表示为$\left[w_{1}^{b}, w_{2}^{b}, \ldots, w_{\left|N^{\prime}\right|}^{b}\right]$，其中$\left|N^{t}\right|\left|N^{a}\right|$和$\left|N^{b}\right|$分别表示标题、摘要、主体的长度。

​	在这项研究中，如图 1 所示，不仅像$\left[N_{i}^{t}, N_{c}^{t}\right]$（标题和标题）这样的字段内对输入，而且像$\left[N_{i}^{t}, N_{c}^{b}\right]$（标题和正文）这样的跨字段对输入都被获得 ，这些输入对中的每一个都被馈送到匹配编码器以获得其匹配表示。 我们最终将这些表示串接为浏览候选新闻对的多字段匹配表示。

### 2.2 匹配编码器

​	该模块用于学习每个浏览过的新闻和候选新闻之间的语义匹配。 我们将来自多字段匹配的浏览-候选多字段对作为输入，然后将其输入到来自 Transformers (BERT) [4] 的双向编码器表示中，以进行匹配表示学习。

​	给定多字段对$\left[N_{i}^{m}, N_{c}^{n}\right]$，其中$m \in\{t, a, b\},n \in\{t, a, b\}$，并且对应的词序列$N_{i}^{m}=\left[w_{1}^{i}, w_{2}^{i}, \ldots, w_{\left|N^{m}\right|}^{i}\right]$并且$N_{c}^{n}=\left[w_{1}^{c}, w_{2}^{c}, \ldots, w_{\left|N^{n}\right|}^{c}\right]$。我们将这两个序列串接成对作为输入：
$$
\operatorname{inp}=\left[[\mathrm{CLS}], w_{1}^{i}, \ldots, w_{\left|N^{m}\right|}^{i},[\mathrm{SEP}], w_{1}^{c}, \ldots, w_{\left|N^{n}\right|}^{c}\right]
$$
添加特殊标记[CLS]作为语义匹配的聚合序列的表示并且[SEP]表示分离序列，我们以𝑖𝑛𝑝为例来详细说明编码过程。

#### 2.2.1 多头部自注意层

​	该子层旨在捕获句子的上下文感知语义匹配。 使用多头，来自不同位置的不同语义子空间的信息是可联合学习的，这对于捕获不同标记之间的匹配信号非常有帮助。

​	给定三个输入矩阵$Q \in \mathbb{R}^{L_{q} \times d}, K \in \mathbb{R}^{L_{k} \times d}$和$V \in \mathbb{R}^{L_{k} \times d}$，则自注意力函数定义为：
$$
\operatorname{Attn}(Q, K, V)=\operatorname{softmax}\left(\frac{Q K^{T}}{\sqrt{d_{k}}}\right) V \tag 1
$$
​	多头自注意力层（MH）将并行应用ℎ注意力函数来产生连接的输出表示并从多个视图中捕获交互信息。
$$
\mathrm{MH}(\mathrm{Q}, \mathrm{K}, \mathrm{V})=\left[\text { head }_{1} ; \ldots ; \text { head }_{h}\right] \mathrm{W}^{O} \tag 2
$$

$$
\operatorname{head}_{i}=\operatorname{Attn}\left(\mathrm{QW}_{i}^{Q}, \mathrm{KW}_{i}^{K}, \mathrm{VW}_{i}^{V}\right) \tag 3
$$

​	其中$W_{i}^{Q}, W_{i}^{K}, W_{i}^{V} \in \mathbb{R}^{d \times d / h} \text { and } W^{O} \in \mathbb{R}^{d \times d}$是可学习参数，𝑑 是匹配嵌入的隐藏大小，并且和𝑄、𝐾和𝑉是𝑖𝑛𝑝的嵌入矩阵.

#### 2.2.2 位置智慧型前馈网络

​	这个子层的目的是赋予模型非线性和不同维度之间的相互作用，这是一个完全连接的前馈网络(FFN)，它由两个线性变换组成，中间有一个ReLU激活：
$$
\operatorname{FFN}(x)=\operatorname{ReLU}\left(x W_{1}+b_{1}\right) W_{2}+b_{2} \tag 4
$$
其中$W_{1}, W_{2} \in \mathbb{R}^{d \times d}$和$b_{1}, b_{2} \in \mathbb{R}^{d}$是可学习的参数，并在所有位置共享。

​	另外，对每个子层进行残差连接和层归一化处理。最后，我们可以通过该匹配编码器得到每个浏览-候选新闻的多字段对的匹配表示。

### 2.3 匹配聚合器

​	匹配聚合器用于聚合每个浏览新闻和候选新闻的匹配表示，以获得最终的用户新闻匹配信号。

​	将匹配表示矩阵对表示为$M\in\mathbb{R}^{n \times d^{\prime}}$，这里n是浏览-候选新闻对的数量，$d^{\prime}$代表表示多字段对匹配表示的维数。在这里，我们使用以下三种类型的聚合器获得最终的用户新闻匹配表示$\tilde{M}$：

- 最大/平均聚合器：对的匹配嵌入直接取最大或平均作为用户-新闻匹配嵌入。
  $$
  \tilde{M}=\text { Aggregate }(M)=\max / \operatorname{mean}(M)\tag 5
  $$

- 注意力聚合器：使用门限注意网络 [2] 来学习 𝑀 的组合权重，它应用具有$W_h$和$b_h$的全连接神经网络，使用 tanh 作为第一层激活函数，然后使用$W_o$和$b_o$构造第二个全连接神经网络来学习在等式（6）中定义的组合权重 𝑟 ，最后在等式（7）中计算词向量的线性组合。
  $$
  \tilde{M}=\text { Aggregate }(M)=\mathbf{r}^{\top} M\tag 6
  $$

  $$
  r=\operatorname{softmax}\left(\tanh \left(\tilde{E} W_{h}+b_{h}\right) W_{o}+b_{o}\right)\tag 7
  $$

### 2.4 点击预测

​	最后，我们介绍点击预测模块。 将**用户-新闻匹配表示**表示为$\tilde{M}$，点击概率 𝑦 通过应用全连接层计算：
$$
y=\operatorname{softmax}\left(\tilde{M} W^{o}+b^{o}\right)\tag 8
$$
其中$W^{o} \in \mathbb{R}^{d^{\prime} \times 1}$和$b^{o} \in \mathbb{R}^{1}$是可学习参数。

## 3 实验

### 3.1 实验设置

#### 数据集

​	我们在两个著名的可用数据集上评估我们的方法：MIND 和 Adressa。

​	MIND：[14]发布的大型公共英语新闻推荐数据集3，收集自微软新闻网站的匿名行为日志。 我们通过从 MIND 数据集中随机抽取 50,000 个用户及其行为日志来使用 MIND 的小型版本。 每条新闻都由标题、摘要和正文组成。

​	Adressa：来自挪威新闻门户网站的真实世界在线新闻数据集 [5]，我们的实验中使用了 Adressa-1week。 我们将数据拆分为训练对的前 6 天数据和测试对的最后一天数据。 用户一天浏览的新闻是这一天之前的点击行为，每条新闻由一个标题组成。 由于 Adressa 中只有点击数据，我们应用比例为 4 的负采样来构建包含点击和非点击行为的印象日志。

#### 评价指标

​	我们使用 AUC、MRR、nDCG@𝐾，其中𝐾 = 5, 10 用于 MIND，𝐾 = 1, 3 用于 Adressa，作为我们的评估指标。 性能是所有展示日志中这些指标的平均值。

#### 实验设置

​	我们使用比率为 4 的负采样进行模型训练。 预训练的 BERT 用于初始化 AMM 的匹配编码器，用于 MIND 的 bert-mini4 和用于 Adressa 的 bert-base,multilingual4。 新闻标题的最大长度设置为 20。用户点击新闻的最大数量设置为 25。模型以 64 的 mini-batch 大小进行训练，学习率为 2𝑒−5 的 Adam 优化器用于 模型优化。 我们以 0.1 的速率应用 dropout 以避免过度拟合。 在验证集上调整超参数，我们将每个实验重复 5 次并报告平均结果。

### 3.2 性能评估

​	我们通过将 AMM 与几种基准方法进行比较来评估 AMM 的性能，包括：（1）LibFM [8]，分解机（FM）； (2) DeepFM[6]：一种结合了FM和神经网络的深度分解机； (3) DKN[10]，一种基于知识感知CNN的深度新闻推荐方法； (4) NPA[12]，引入注意力机制来选择重要的词和新闻； (5) NAML [11]，一种具有注意力多视图学习的神经新闻推荐方法； (6) LSTUR [1]，它使用 GRU 从点击历史中对短期和长期兴趣进行建模； (7) NRMS [13]，它使用多头自注意力来学习用户和新闻表示； (8) FIM[9]，一种用于神经新闻推荐的细粒度兴趣匹配方法； (9)AMM，我们的方法。对于 DKN、NPA、LSTUR、NAML 和 NRMS，我们使用官方代码和设置5。对于其他基线，我们重新实现它们并根据他们论文报告的实验设置策略设置它们的参数。

​	所有方法的实验结果总结在表1中，我们从结果中做出以下观察。首先，使用深度神经网络提取新闻语义表示的方法（3-8）比基于特征的方法（1-2）表现更好。这种性能改进应归功于更好的新闻表示方法。在方法（3-8）中，NAML 在 MIND 中表现最好，NRMS 在 Adressa 中表现最好，这是因为 NAML 使用了不同种类的新闻信息，而 NRMS 应用了多头自注意力来进一步捕捉单词和点击新闻互动。 FIM 在所有基线中的 Adressa 中表现最好，因为它可以捕获更细粒度的兴趣匹配信号。第三，我们提出的 AMM 通过考虑分离的浏览候选新闻对和跨字段匹配来捕获多字段的更好的文本语义匹配，在所有指标方面在两个数据集上实现了最佳性能。

### 3.3 消融实验

​	为了验证每个字段的有效性，我们比较了AMM与三个单一字段(标题、摘要、正文)在MIND上的性能。从图2(a)中我们可以观察到，带有标题和正文的AMM的表现优于带有摘要的AMM，这表明标题和正文比摘要更有利于新闻表征。此外，单个标题或正文的AMM的性能优于最佳基准，表明了分离策略引入文本语义匹配的必要性。

​	接下来，我们通过实验验证了多场匹配的有效性。我们比较了多场(场内+跨场)和仅场内的AMM。我们观察到，如图2(b)所示，AMM多字段比AMM内字段性能更好，这可能是因为通过跨字段匹配可以融合更多的互补信息，字段内和多字段信息的集成可以提高用户新闻匹配，捕捉细粒度用户的兴趣。

​	此外，我们探讨了不同匹配聚合器的有效性。 如图 2(c) 所示，我们可以看到带有注意力聚合器的 AMM 的性能略好于带有最大+平均池化的 AMM。 而且，最大+平均池化的表现也比 最佳基准方法好很多，验证了多字段语义匹配的有效性，这些简单的操作让在线服务更有可能。

## 结论

​	在本文中，我们提出了一种新颖的注意力多领域匹配（AMM）新闻推荐框架。 我们建议以新闻分离的方式学习用户-新闻匹配表示，捕获每对浏览候选新闻的匹配表示，然后将这些表示聚合为最终匹配信号，从而获得细粒度的语义匹配信息以及它是在线服务友好的。 此外，我们的方法同时考虑了字段内和跨字段作为多字段匹配，利用了字段间的互补信息，提高了新闻对的匹配表示。 两个公共基准数据集的实验结果证明了 AMM 的最新性能。