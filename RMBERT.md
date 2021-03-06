# RMBERT: News Recommendation via Recurrent Reasoning Memory Network over BERT

基于BERT的循环推理记忆网络新闻推荐

## 论文概况

[https://dl.acm.org/doi/abs/10.1145/3404835.3463234](https://dl.acm.org/doi/abs/10.1145/3404835.3463234)

SIGIR '21: Proceedings of the 44th International ACM SIGIR Conference on Research and Development in Information Retrieval 

July 2021 Pages 1773–1777 

https://doi.org/10.1145/3404835.3463234

## 摘要

​	个性化新闻推荐旨在缓解信息泛滥，帮助用户找到自己感兴趣的新闻。准确匹配候选新闻和用户兴趣是新闻推荐的关键。现有的方法大多是根据新闻内容将每个用户和新闻分别编码为向量，然后对两个向量进行匹配。但是，用户对每个新闻或新闻的每个主题的兴趣可能是不同的。动态地学习用户和新闻向量，并对他们的互动进行建模是必要的。在本研究中，我们提出了基于BERT的循环推理记忆网络(RMBERT)用于新闻推荐。与其他方法相比，我们的方法可以利用BERT的内容建模能力。此外，循环推理记忆网络执行一系列基于注意力的推理步骤，可以动态学习用户和新闻向量，并对每一步的交互进行建模。因此，我们的方法可以更好地模拟用户的兴趣。我们在真实世界的新闻推荐数据集上进行了广泛的实验，结果表明，我们的方法显著优于现有的最先进的方法。

## 1 引言

​	由于互联网的快速发展，人们的阅读习惯逐渐转向网络新闻平台[8]。与报纸、电视等传统媒体相比，MSN 新闻、谷歌新闻等网络新闻平台可以聚合各种来源的新闻，并将新闻文章分发给特定的用户。个性化推荐系统在帮助用户快速发现自己感兴趣的新闻方面发挥着关键作用。

​	一般来说，新闻推荐系统需要对新闻内容进行精确建模。这是因为新闻文章对时间高度敏感，过时的新闻很快就会被更新的新闻所取代，这使得基于id的功能效果不佳。传统的新闻推荐方法依靠人工特征工程来构建新闻和用户表示，如Wide &Deep[2]、DeepFM[6]、DCN[14]等。最近，一些方法探索使用神经网络以端到端方式学习新闻和用户表示，并在训练过程中优化这些表示。DKN [13]， NPA [15]， LSTUR[1]提出使用CNN学习新闻表示，然后将候选新闻与用户点击的新闻进行匹配。DKN将知识图表示引入新闻推荐。NPA通过嵌入用户ID来关注重要词汇和新闻，增强了新闻和用户表示。LSTUR结合了长期和短期用户表示，以更精确地对用户进行建模。NRMS[16]引入多头自我注意学习新闻表示。FIM[12]通过叠加扩张卷积提取每个新闻的多层表示。

​	尽管它们带来了显著的收益，但我们认为这些方法在捕获文本内容中丰富的语义方面仍然不够。现有的方法大多采用cnn或GRU作为新闻编码器，通常使用域内新闻文本进行训练。这不足以获得全面的新闻表示，还会进一步损害推荐性能。受最近关于预训练语言模型BERT[3]的里程碑式工作的启发，我们采用BERT来对新闻文本进行编码。BERT使用深度转换器来提高来自大规模未标记语料库的语言理解。预先训练的BERT模型可以产生更具表现力的新闻嵌入，我们独立地对每条新闻进行编码，允许离线预先计算新闻嵌入，大大减少了在线推理的计算量。

​	此外，在新闻推荐场景中有两个常见的观察结果。首先，不同的用户可能对候选人新闻的不同部分感兴趣。第二，用户的兴趣通常是多样化的，用户历史中不同的新闻代表了该用户的不同兴趣。以前的工作 [9, 13, 16] 使用一个固定的嵌入向量来表示用户和新闻，这可能会限制模型的表达能力。作为一个解决方案，我们需要动态地确定关于候选新闻的用户向量。记忆网络能够捕获两个对象之间的高阶复杂关系，在问题回答[4,7]、情感分析[11]、机器理解[10]等领域都取得了很大的进展。受其成功的激励，我们设计了一个循环结构的迭代推理过程，多个推理记忆单元(RMC)级联执行结构化推理操作，一步一步确定匹配分数。

​	为简洁起见，我们将结果模型称为BERT上的循环推理记忆网络(RMBERT)。我们的方法的体系结构如图1所示。新闻编码器模块通过处理每个新闻的标题并生成新闻嵌入来提取候选新闻和一些用户点击的新闻。在此之后，循环推理记忆网络中包含一些以新闻嵌入为输入的rmc，它们协同工作，逐级执行推理过程。在每一步中，RMC通过查询单元和存储单元进行推理操作。查询单元使用注意机制来选择用户可能感兴趣的候选新闻嵌入的某些方面来更新查询状态。所述存储单元根据查询状态，从用户点击的新闻嵌入中动态检索用户的兴趣。经过几个推理步骤后，RMBERT可以从多个方面捕获用户对候选新闻的偏好。最后，我们根据最后的存储状态来推荐新闻，该状态包含了用户和候选新闻之间的交互信息。

​	这项工作的主要贡献总结如下。

1. 我们使用BERT对候选新闻和用户点击新闻进行独立编码。我们的方法可以利用BERT的表达能力，同时获得离线预计算新闻嵌入的能力。
2. 我们提出了一种循环结构来逐步执行基于注意力的推理操作。 在每个步骤中，RMC 可以推断候选新闻的某些部分与用户兴趣之间的匹配分数。 RMC可以在每一步动态确定新闻和用户向量，并在多方面探索它们的匹配。
3. 从MSN新闻收集的真实数据集上进行的大量实验证实，我们的方法比现有的最先进的方法具有更好的性能。

## 2 提出的方法

### 2.1 新闻编码器

​	我们用n表示候选新闻，$$u=\left[n_{1}, n_{2}, \cdots, n_{M}\right]$$表示用户历史，M是用户历史的长度。$$f_n$$将n编码为一个固定长度的词向量X，$$f_u$$将u中的$$n_i$$编码到另一个向量$$X_i$$中，$$f_n$$和$$f_u$$是预训练的BERT模型并且共享相同的参数。第一，我们将新闻标题n标记为基于BERT的WordPiece令牌$$w_{1}, w_{2}, \cdots, w_{L}$$,这里的L是新闻标题序列的长度。我们遵循BERT的方法:将[CLS]标记前置到输入标记的开头。与[CLS]标记相对应的输出嵌入被用作携带全局信息的聚合序列表示。然后，将这个输入标记序列传递给BERT，由BERT计算每个标记的更符合实际的表示。

​	我们使用[CLS]令牌的最终输出作为候选新闻嵌入$$x_n$$，单词令牌列表的输出是候选新闻的上下文单词嵌入$$X_{c o n}=\left[x_{1}, x_{2}, \cdots, x_{L}\right]$$.为了保留候选新闻的原始单词级信息，我们保留了候选新闻$$X_{c o n}$$的上下文单词嵌入并输入查询单元。类似地，对于用户历史中的每个新闻，我们首先在BERT的开始标记前面加上[CLS]，然后将输入标记序列传递给BERT。我们用$$h_i$$表示u中$$n_i$$的新闻嵌入。用户点击新闻嵌入集可以被表示为$$H=\left[h_{1}, h_{2}, \cdots, h_{M}\right]$$。候选新闻向量$$x_n$$，候选新闻的上下文单词向量$$X_{con}$$和用户点击新闻向量集H被输入到推理记忆网络中来估计n和u的匹配值。

### 2.2 推理存储单元

​	以前的工作是学习固定的候选新闻向量。这将限制模型的表现力。在我们的工作中，我们设计了一个包含递归结构的p RMCs。在每个RMC中，查询单元学习不同的候选新闻向量来表示候选新闻的不同方面，然后存储单元匹配用户的兴趣。这些单元串在一起以执行一系列匹配程序，并共享相同的参数。我们的模型可以从多个方面探索匹配信号。第i个单元接收前一个单元的双隐藏状态，候选新闻单词向量X和用户点击新闻向量集H作为输入并且更新双隐藏状态$$k_i$$和$$m_i$$。初始时，$$k_0$$被$$x_n$$初始化，$$m_0$$被随机初始化。

#### 查询单元

​	查询单元被设计用来确定内存单元在每个步骤中应该考虑的关键信息i。查询单元利用先前的查询状态$$k_{i-1}$$和候选新闻向量X来更新查询状态$$k_{i}$$，它蕴含了候选新闻的关键信息。

​	为了在不同的迭代步骤关注候选新闻的不同方面，我们利用不同的$$W_{q}^{i}$$和$$b_q^i$$来把候选新闻向量线性转换为一个position-aware向量$$q_i$$。此时我们将$$q_i$$和先前的查询状态$$k_{i-1}$$结合，然后通过一个线性变换来产生向量$$kq_i$$。所以单元可以决定要关注哪些方面。
$$
q_{i}=W_{p}^{i} \cdot x_{n}+b_{p}^{i}\tag 1
$$

$$
k q_{i}=W_{k} \cdot\left[q_{i}, k_{i-1}\right]+b_{k} \tag 2
$$

​	在这里$$W_{\alpha} \in \mathbb{R}^{d \times 1}, b_{\alpha} \in \mathbb{R}^{1}, W_{q} \in \mathbb{R}^{d \times d_{q}}, b_{q} \in \mathbb{R}^{d_{q}}$$是投影参数。i是迭代步骤，$$d_q$$是查询单元的维数，d是词向量维数。

​	在推理过程中，我们仍然需要限制有效推理操作的空间，以防止查询单元发生发散。我们应用注意网络来确保查询单元仍然集中在新闻词汇上。最后，根据注意权重对上下文词的嵌入进行求和

$$
k_{i}=W_{q} \cdot \sum_{j=1}^{L} \operatorname{softmax}\left(W_{\alpha} \cdot\left(k q_{i} \odot X_{\text {con }}\right)+b_{\alpha}\right) \cdot X_{\text {con }}+b_{q} \tag 3
$$
​	在这里$$W_{\alpha} \in \mathbb{R}^{d \times 1}, b_{\alpha} \in \mathbb{R}^{1}, W_{q} \in \mathbb{R}^{d \times d_{q}}, b_{q} \in \mathbb{R}^{d_{q}}$$是投影参数,$$\odot$$是矩阵对应元素的乘积（ element-wise product ），$$k_i$$是当前的查询状态。与传统的注意力网络相比，我们抛弃了$$\sum_{j=1}^{L} \alpha_{i}=1$$的约束以保持单词的信息强度。

#### 记忆单元

​	内存单元接收到以前的内存状态$$m_{i-1}$$、当前查询状态$$k_i$$用户点击的新闻嵌入集H作为输入并执行推理操作，以确定用户对候选新闻的兴趣。

​	为了从用户点击的新闻中确定用户的表示，我们对用户点击的新闻执行一个两阶段的过程。首先，我们对$$m_{i-1}$$进行线性变换。对于用户点击的新闻嵌入，我们采用了同样的处理。我们可以通过计算元素乘积来获得它们之间的交互作用$$I_i$$。因为$$m_{i-1}$$包含了用户点击新闻和候选新闻之间的先验提取匹配信号。 记忆单元在进行推理步骤时可以逐步继承这个相关性。 然后，我们通过线性变换结合向量$$I_i$$和𝐻来考虑来自𝐻的新信息。

$$
I_{i}=\left[W_{r} \cdot m_{i-1}+b_{r}\right] \odot\left[W_{H} \cdot H+b_{H}\right]\tag 4
$$

$$
\tilde{I}_{i}=W_{I}\left[I_{i}, H\right]+b_{I}\tag 5
$$

在这里$$W_{r} \in \mathbb{R}^{d_{m} \times d}, b_{r} \in \mathbb{R}^{d}, W_{H} \in \mathbb{R}^{d \times d}, b_{H} \in \mathbb{R}^{d}, W_{I} \in \mathbb{R}^{2 d \times d},b_{I} \in \mathbb{R}^{d}$$是投影参数，$$d_m$$是记忆单元的维度，$$\tilde{I}$$是当前步骤的用户表示。

​	由于用户兴趣的多样性，不同的新闻代表不同的用户兴趣。 我们通过注意力网络计算查询状态$$k_i$$和用户表示 $$\tilde{I_i}$$之间的相似性。用户对候选新闻$$U_i$$的兴趣是用户点击新闻𝐻的表征的总和，由他们的注意力权重加权。

$$
U_{i}=\sum_{j=1}^{M} \operatorname{softmax}\left(W_{\beta} \cdot\left(\tilde{I}_{i} \odot k_{i}\right)+b_{\beta}\right) \cdot H \tag 6
$$
这里$$W_{\beta} \in \mathbb{R}^{d \times 1}, b_{\beta} \in \mathbb{R}^{1}$$是注意力网络的参数。

​	记忆状态保存了用户对候选新闻与用户点击新闻交互获得的候选新闻的偏好。 为了更新记忆状态，我们将先前的记忆状态$$m_{i-1}$$与当前用户的兴趣信息$$U_i$$并通过线性变换相结合。

$$
m_{i}=W_{m}\left[m_{i-1}, U_{i}\right]+b_{m} \tag 7
$$
这里$$W_{m} \in \mathbb{R}^{\left(d_{m}+d\right) \times d_{m}}, b_{m} \in \mathbb{R}^{d_{m}}$$是投影参数。

​	经过𝑝推理步骤，循环推理记忆网络获得了用户历史和候选新闻之间的关系信息。

### 2.3 点击预测模块

​	我们根据最近的记忆状态$$m_p$$预测用户点击候选新闻的概率。我们结合候选新闻向量$$x_n$$h和$$m_p$$然后穿过两个隐藏层。用户点击候选新闻的概率由以下公式预测:
$$
\begin{array}{l}
o=\operatorname{ReLU}\left(W_{1} \cdot\left[x_{n}, m_{p}\right]+b_{1}\right) \\
\hat{y}=\operatorname{Sigmoid}\left(W_{2} \cdot o+b_{2}\right)
\end{array} \tag 8
$$
这里$$W_1,W_2,b_1,b_2$$是密集层的参数。RMBERT使用 BCEWithLogits 损失函数来训练。

## 3 实验

### 3.1 数据集和实验设置

#### 3.1.1 数据集

​	我们使用公开可访问的数据集MIND[17]来评估我们的方法，该数据集来自Microsoft News的用户行为日志。该数据集有两个版本，分别名为MIND-large和MIND-small。MIND-large包含300万用户，MIND-small随机抽样5万用户。印象日志记录用户在特定时间访问新闻平台时显示的新闻和历史点击。印象日志记录了用户在特定时间访问新闻平台时向其展示的新闻以及对这些展示新闻的历史点击行为。 我们使用前 6 天的日志进行训练，最后一天进行测试。 我们进一步随机抽取了 10% 的训练样本进行验证。

#### 3.1.2 实验设置

​	我们将我们的方法与几个竞争基准进行比较，包括 DCN [14]、DeepFM [6]、DKN [13]、NRMS [16]、LSTUR [1]、NPA [15]、FIM [12]。 我们使用手工制作的特征作为 DeepFM 和 DCN 的 ID 特征的补充。 新闻文本的手工特征是从新闻标题中提取的 TF-IDF 特征。 性能通过所有展示的 AUC、MRR 和 nDCG@5 分数来判断。 大多数基线模型都是在 TensorFlow 中实现的，基于 Recommender [5] 的公共代码，遵循它们的参数设置。 我们根据验证集调整其他超参数。 我们使用 Google 的官方预训练模型初始化基于 BERT 的新闻编码器。 循环推理记忆网络的长度为 4。我们使用 Adam 对 RMBERT 进行微调，学习率为$$2×e^{-5}$$，batch size 设置为 50。

### 结果和讨论

​	表1汇总了所有比较方法的总体性能。最好的和第二个结果分别以粗体和下划线突出显示。

​	我们有几个观察结果。 首先，DeepFMand DCN 的性能一直低于其他模型。 这是因为这些手工制作的特征是固定的，无法在训练过程中进行优化。 其次，NRMS 的表现优于 DKN 和 NPA。 这可能是因为 self-attention 可以捕获单词的远程上下文，而 CNN 只能对局部上下文进行建模。 第三，这些模型 NRMS、LSTUR、NPA、FIM 使用注意力机制以端到端的方式学习新闻和用户表示，其性能优于其他基准模型。 这可能是因为注意力机制可以区分新闻标题中的信息词，并且可以选择一些可以反映用户偏好的新闻。

​	总之，我们的方法在所有指标方面始终优于所有比较的基准模型。 我们将RMBERT的优越性归因于它的三个属性：（1）RMBERT使用BERT来捕获单词的大范围上下文。 此外，预训练模型可以利用域外未标记的大规模语料库，从而生成更好的新闻嵌入。 (2)RMBERT在查询单元中使用词级注意，在记忆单元中使用新闻级注意，在每一步中动态确定新闻和用户向量。 (3)循环结构可以迭代地从候选新闻中选择信息，匹配用户点击的新闻，可以增强模型能力。

​	我们进行消融实验（ablation study），以探索我们方法中不同部分的有效性。 我们设计了RMBERT的4个变体来理解RMC和BERT编码器的贡献：（1）忽略查询单元中先前的查询状态（w/o PQ）（2）忽略内存单元中的先前内存状态（w/o PM）（3 ）用平均池化（w/o Att）替换注意力网络（4）用 Bi-LSTM（w/o BERT）替换 BERT。图 2 (a) 显示了上述变体的实验结果。 RBERT 实现了最佳性能。 结果证明RMBERT可以从预训练模型BERT中获益良多。 w/o Att 的表现表明，使用注意力网络突出重要的单词和新闻可以帮助模型理解用户的兴趣。 查询单元一次只能从候选新闻中学习几个方面，需要逐步了解整个候选新闻。 至于 w/o PM，此变体在所有变体中显示出最差的结果。这是因为记忆单元只能在一个步骤中理解两个对象之间的相关性。 这证明了我们的循环结构的有效性，它可以将复杂的推理过程分解为一系列简单的操作。

​	我们进一步研究了RMBERT如何在从1到6的不同数量的RMC下表现。图2（b）显示了不同数量的RMC的auc分数。 我们可以观察到，当 RMC 的数量为 4 时，我们的模型在 AUC 方面取得了最佳性能。AUC 分数通常随着 RMC 数量的增加而增加，因为更多的 RMC 可以执行更多的推理步骤并增强模型 能力。 然而，当 𝑝 = 5 时，由于可能的过度拟合，趋势会发生变化。 更多的 RMC 会增加我们模型的复杂性，并消耗更多的训练时间。 选择一个合适的𝑝来平衡模型的性能和复杂性是很重要的。

## 结论

​	在本文中，我们提出了一种新的新闻推荐模型，该模型在 BERT 上使用循环推理记忆网络。 通过使用 BERT 独立编码候选新闻和用户点击新闻，RMBERT 可以利用 BERT 的表现力。 此外，我们的方法可以通过执行一系列推理操作来动态捕捉用户对候选新闻的偏好。 因此，新闻推荐列表可以得到合理的扩展。 从 MSN 新闻收集的真实数据集上的实验证明了我们模型的有效性。
