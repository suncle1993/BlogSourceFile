---
abbrlink: 3045060095
alias: 2019/11/26/Huffman-encode-the-principle-and-the-reality/index.html
categories:
- 随笔
date: '2019-11-26T10:36:51'
description: ''
tags:
- 哈夫曼
- 编码
- 投资
title: 从哈夫曼编码再出发：原理和现实
---









对于计算机科班出身的人来说，在大学阶段几乎都学过信息论和算法这两门课，信息论都会讲到香农三大定理以及哈夫曼编码，算法课上会学习二叉树，甚至哈弗曼树。在介绍哈夫曼编码之前，先介绍一下什么是有效编码，以及香农第一定理的内容。

一个好的有效编码需要遵循两个基本原则：

- 易辨识
- 有效性

那么怎样才能做到有效编码呢？下面有一个问题：

**用10根手指头，能表达多少个数字？**

<!--more-->

常见的回答有以下两种：

1. 能表达10个数字，因为小孩子数数的时候就是掰着指头数的。
2. 能表达100个数字，因为我们平时能用一只手就能做出10个形状，也就是能数10个数，将两只手组合起来，一个表示十位，一个表示个位，就能表示从0到99共100个数字

第一个回答最直观，第二个回答其实就利用了编码的知识。

但是这依然不是最有效的编码，如果我们考虑采用二进制，而不是十进制进行编码，则能表示1024个不同的数字。

具体的做法是这样的，把10个手指并排在一起，从左到右依次给手指编号，编码为0~9。每一个手指头都有伸出和收起两种状态。每一种状态对应于一位二进制，十个手指头就能表示10位二进制，也就是2的10次方，也就是1024种数字。

当然也有人觉得可以让每个手指具有伸开、半伸开、收缩三个状态，表示3的10次方也就是59049中数字。虽然这种想法也是正确的，但是过分强调有效性，而忽视了易辨识这个原则，凡事过犹不及。

常见的比较有效的编码有阿拉伯数字，莫尔斯电码以及计算机中根据电路状态演化的二进制编码。

一个有效的编码是否就是最优编码呢，答案当然是不一定。香农第一定理告诉我们编码长度是有理论最小值的，摘录信息论这本书中的公式如下：

**编码长度 ≥ 信息熵(信息量) / 每一个码的信息量**

香农对此做出了严格的数学证明，同时还证明，只要编码系统设计得足够巧妙，上面的等号是成立的。

我们以二进制编码为例来说明这个公式，为了预测世界杯冠军，我们先对世界杯的32只球队编码，那如何编码才能使得编码长度最短呢？对于这样的n选1的问题，根据香农第一定理，32选1的信息熵为log32=5比特（以2为底的对数），每个编码的信息量为1比特，根据公式最短编码长度为5。如果编码长度小于5，那么传递出去的信息就一定包含不确定性，也就是有损信息、失真信息。

至于信息熵的计算为什么是以2为底的对数，可以参考分治思想。

如果我们对经常出现的字母采用较短的编码，对不常见的字母采用较长的编码，根据常识，这样是能够降低编码的整体长度的。在莫尔斯电码中，我们会发现26个英文字母中的5个元音字母aeiou的编码长度是最短的。如果对英文26个字母采用等长度的编码，比如进行二进制编码，需要log26，就是5比特信息。而采用莫尔斯的编码方式，平均只需要3比特，这样效率就提升了很多。

在中国，北京和上海等重要城市的长途电话区位码就是两位，小城市就使用3位，比如北京是010，上海是021，而江苏常州是0519（所有都忽略掉前面的0），这样做的目的就是为了减少平均的编码长度。

那怎样才能找到最有效的二进制编码呢？哈夫曼在*《A Method for the Construction of Minimum-Redundancy Codes》*这篇论文中发表了基于自底向上的有序频率二叉树的编码方法，并很快证明了这个方法是最有效的。

关于哈夫曼树的构建过程可以参加文末的参考中的Wikipedia链接，此处只做一个简单描述：

假设我们要给一个英文单字**"F O R G E T"**进行哈夫曼编码，而每个英文字母出现的频率分别列在下图中。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/Huffman-code-and-investment/Huffman-fig-1.jpg)

进行霍夫曼编码前，我们先创建一个霍夫曼树。

1. 将每个英文字母依照出现频率由小排到大，最小在左，如上图。
2. 每个字母都代表一个终端节点（叶节点），比较**F.O.R.G.E.T**六个字母中每个字母的出现频率，将最小的两个字母频率相加合成一个新的节点。
3. 比较**5.R.G.E.T**，发现**R**与**G**的频率最小，故相加4+4=8。
4. 比较**5.8.E.T**，发现**5**与**E**的频率最小，故相加5+5=10。
5. 比较**8.10.T**，发现**8**与**T**的频率最小，故相加8+7=15。
6. 最后剩**10.15**，没有可以比较的对象，相加10+15=25。

最后产生的树状图就是霍夫曼树，如下图。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/Huffman-code-and-investment/Huffman_algorithm.gif)

给霍夫曼树的所有左节点'0'与右节点'1'，从树根至树叶依序记录所有字母的编码，如下图。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/Huffman-code-and-investment/Huffman-fig-3.jpg)

哈夫曼编码从本质上讲，是将最宝贵的资源（最短的编码）给出现概率最大的信息。我们可以在任何需要分配资源的工作中利用哈夫曼编码的思想。

在风险投资领域，利用哈夫曼编码原理投资就是一套比较有效的系统方法。假定你有一大笔钱可以用于风险投资，怎样投资效果最好？下面有三种做法：

1. 平均的投入到100个初创公司
2. 利用自己的眼光投入到一家最可能的公司中
3. 利用哈夫曼编码进行投资

第一种方法，过于平均，基本上只能得到一个市场的平均回报。第二种方法，只投一家，其实这就是赌博，我的一些朋友购买股票时，会只买单只股票并且重仓，这种情况如果碰到了会有几倍收入，但是大多数情况下都是血本无归，这是极为糟糕的投资方式。第三种方法是利用哈夫曼编码的原理，可以先把钱逐级投下去，每一次投资的公司呈指数减少，而金额倍增，这样通常不会错失上市的那家。大部分资金都集中到了最后的上市或被收购的企业中，这种投资回报要远远高于前两种。

对于个人而言，利用哈夫曼编码进行投资也是适用的。美国有名的私立学校哈克学校的前校长尼克诺夫博士说过，在孩子小时候，要让他们尝试各种不同的爱好，但是最终他们要在一个点上实现突破。他将这比作圆规画圆，一方面有一个扎得很深的中心，另一方面有足够广的很浅的覆盖面。

在工作中，一方面需要成为某个方面的专家，做到足够的深入，比如在DevOps方面，另一方面也需要有足够的覆盖面，了解各个细分领域的设计思想，基本原理和简单实用。

对于我而言，我会尝试很多新的事情，不会去排斥，是因为不想失去机会，虽然结果是绝大部分失败了，但是至少也尝试过了，毕竟谋事在人成事在天。另一方面对于我花了一些精力，但是看样子也成不了的事情，我是坚决做减法退场止损。这条同样也适用于感情。

---

参考：

1. wiki：[https://zh.wikipedia.org/wiki/%E9%9C%8D%E5%A4%AB%E6%9B%BC%E7%BC%96%E7%A0%81](https://zh.wikipedia.org/wiki/霍夫曼编码)
2. dahuffman：https://github.com/soxofaan/dahuffman
3. 哈夫曼树的调整：https://blog.csdn.net/fx677588/article/details/70767446