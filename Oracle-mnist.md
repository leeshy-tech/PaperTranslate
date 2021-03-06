# Oracle-mnist： a Realistic Image Dataset for Benchmarking Machine Learning Algorithms

一个用于基准机器学习算法的现实图像数据集 

## 摘要

​	我们引入了Oracle-MNIST数据集，包括10个类别的30222个古代字符的28×28灰度图像，用于基准模式分类，特别是图像噪声和失真方面的挑战。训练集共有27222张图像，测试集每个类有300张图像。Oracle-MNIST与原始MNIST数据集共享相同的数据格式，允许与所有现有的分类器和系统直接兼容，但它构成了比MNIST更具挑战性的分类任务。由于三千年的埋藏和老化，古文字的形象受到了极其严重和独特的噪音影响，以及古文字的书写风格的巨大变化，这些都使其具有机器学习研究的现实感。该数据集可以在https://github.com/wm-bupt/oracle-mnist上免费获得。

## 1 引言

​	在过去的几年里，由于专门的数据集作为实验测试平台和当前进展的公共基准的发布，机器学习(ML)取得了快速的进展，从而集中了研究社区的努力。计算机视觉中最广为人知的数据集是MNIST数据集，该数据集于1998年由LeCun等人(1998)首次引入。MNIST是一个10类数字分类数据集，由60,000张用于训练的灰度图像和10,000张用于测试的灰度图像组成。整个数据集相对较小，可以自由访问和使用，并以完全直接的方式进行编码和存储，这几乎肯定有助于它的广泛使用。

​	然而，随着改进学习算法的发现，MNIST的性能已经饱和。例如，卷积神经网络(CNNs) (Krizhevsky等人，2012;He et al.， 2016)可以轻松达到99%以上的准确率。这部分归因于基准测试没有捕获许多真实场景的需求。为了避免饱和性能并为改进的ML算法提供挑战，构建了一些改进的MNIST数据集，例如EMNIST (Cohen等人，2017)和fashionn -MNIST (Xiao等人，2017)。EMNIST通过引入大写和小写字符扩展了类的数量，但额外的类需要改变MNIST使用的深度神经网络的框架。fashion - mnist包含10级时尚产品的7万张灰度图像。这些产品图片来源于Zalando的网站e1，由专业摄影师拍摄，清晰规范。然而，它未能在现实世界中尽可能广泛地捕捉各种变化。

​	本文的目的是提供一个真实且具有挑战性的数据集Oracle-MNIST，以方便ML算法对真实的古代字符图像进行简单快速的评估。oracle - mnist包含属于10个类别的oracle字符的30222个图像。

1. 真实世界的挑战。

   与手写数字不同，甲骨文字符是从真实的甲骨文表面扫描而来。因此，Oracle-MNIST由于数千年的埋藏和老化而产生了极其严重和独特的噪音，并且在每个类别中都包含着各种各样的写作风格，这都使得ML研究更加现实和困难。

2. 易用性。

   与原来的MNIST相同，图像在Oracle-MNIST有28×28灰度像素。它可以立即兼容任何能够与MNIST数据集工作的ML包，因为它共享相同的数据格式。事实上，使用这个数据集需要做的唯一更改就是更改获取MNIST数据集的URL。

## 2 Oracle-MNIST 数据集

### 2.1甲骨文字符发现

​	古代史依赖于对古文字的研究。甲骨文是中国最古老的象形文字(Flad et al.， 2008;Keightley, 1997)，近三千年的历史，为现代文明做出了巨大的贡献，使中华文化得以代代相传，成为唯一延续至今的文明。如图1所示，甲骨文被镌刻在龟甲和兽骨上，记录了商朝(公元前1600-1046年)的生活和历史，包括占卜、征战、狩猎、医疗和生育等。1899年，清朝(1644-1911)，一位名叫王希荣的商人首次发现了它们。20世纪初，中国的研究人员在商朝都城河南安阳的小屯村发掘了大量的甲骨文。此后，甲骨文文字的研究备受关注。它对于中国的词源学和书法，以及学习中国古代乃至世界的文化和历史都具有重要的意义。

​	甲骨文字符大多采用扫描图像存储，扫描图像是将一张纸盖在主体上，然后用卷墨拓印，再现甲骨文表面，如图2(a)所示。识别这些甲骨文字符对专家和机器来说都很困难。到目前为止，已经发现了近4500个不同的甲骨文字符，但只有大约2200个字符被成功破译。原因如下。(1)磨损和噪声。几个世纪以来，许多甲骨铭文遭到破坏，它们的文字现在已经残缺不全。铭文的老化过程也使其不易辨认，因此扫描的文字破损，并含有严重的噪音。(2)大方差。不同的写作风格导致了类内的高度差异。如图2(b)所示，属于同一类别的字符在笔画甚至拓扑结构上都存在较大差异。不同类别的一些字符彼此相似，这给识别带来了很大的困难。例如，图2(c)和图2(d)所示的“木”和“牛”两个类别的字符仅在一些小细节上有所不同。

### 2.2 数据集的细节

​	Oracle-MNIST是基于*殷奇文苑网站*的集合。这些甲骨文字是从真实的甲骨文表面扫描而来的，因此会出现破碎和噪音严重的现象。每个扫描图像的中心都有一个字符。大多数原始图像都有灰色或黑色背景，分辨率也各不相同。

​	我们选择了10个类的30222个常用字符构建Oracle-MNIST。然后将原始图像送入下面的转换管道，如图3所示。我们还试图通过一些图像增强技术，如灰度拉伸和直方图均衡化来处理图像。虽然成功地提高了图像的视觉质量，但识别性能略有下降。因此，Oracle-MNIST并没有采用图像增强技术。我们还提供原始图像，并将数据处理工作留给算法开发人员。

1. 将图像转换为8位灰度像素。
2. 如果前景比背景暗，则取消图像的强度。
3. 使用双三次插值算法将图像的最长边缘调整为28。
4. 将最短的边延长到28，并将图像放到画布的中心。

​	我们利用字符的含义作为它们的类标签。这些标签是由考古学或古生物学专家手工标注的。表2给出了Oracle-MNIST中所有类标签的总结，并给出了每个类的示例。

​	最后，我们将数据集分为训练集和测试集，并确保它们是不相交的。训练集由随机选取的27222张图像组成，测试集每类包含300张图像。图像和标签以与MNIST数据集相同的文件格式存储，MNIST数据集是为存储向量和多维矩阵而设计的。表1列出了结果文件。

## 3 实验

​	我们在Oracle-MNIST上评估了一些不同参数的算法，结果如表3所示。对于每种算法，通过三次重复实验报告了平均分类精度。MNIST和Fashion-MNIST数据集上的基准也包括在并排比较中。

​	从结果中，我们有以下观察结果。首先，经典(浅层)ML算法在MNIST数据集上可以轻松实现97%，这证明了MNIST算法太容易评估。我们的Oracle-MNIST数据集提供了10类古代字符的图像，并进一步捕捉了现实世界中尽可能广泛的变化，这比MNIST数字数据和Fashion-MNIST数据提出了更具有挑战性的分类任务。我们可以看到，所有经典的(浅)ML算法在MNIST上表现最好，其次是fashionmnist，在oracle MNIST上表现最差。例如，随机森林分类器的准确率分别为97.1%、87.1%和64.9%。这是因为上文2.1节所述的类内方差和类间相似度较高，会给分类带来很大的困难。此外，由于模糊、噪声和遮挡等原因，扫描到的甲骨文图像会严重退化，甚至完全失去识别字形信息。

​	其次，CNN优于Oracle-MNIST上所有的经典(浅)ML算法。得益于局部接收场和空间或时间子采样，CNN可以强制提取局部特征，降低输出对位移和失真的敏感性(LeCun et al.， 1995)。因此，现实世界的挑战，如不同的写作风格，噪音和闭塞，可以在一定程度上解决。然而，Oracle-MNIST上的性能还没有饱和。本文使用的CNN在Oracle-MNIST上的错误率为6.2%，还有改进的空间。尽管CNN具有强大的表示能力，但是识别这些古文字的问题还有待完全解决。

## 4 结论

​	在本文中，我们提出了一个现实的和具有挑战性的基准数据集，即Oracle-MNIST。它包含10类30222个古文字的28×28灰度图像，用于对计算机视觉中的图像噪声和失真的鲁棒性进行基准测试。Oracle-MNIST被转换成一种与构建来处理MNIST数据集的分类器直接兼容的格式。由于数千年的埋藏和老化，这些古老的文字噪音大，书写风格各异，给识别带来了很大的困难。基准测试结果表明，古文字分类任务确实更具挑战性。