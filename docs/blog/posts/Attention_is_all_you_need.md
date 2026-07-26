---
date: 2026-07-13
categories:
  - tech
tags:
  - 论文学习笔记
draft: false
---

# 这是学习大模型领域经典论文：Attention is all you need的笔记。

之前已经完整的学习了这篇论文，为了能够成体系的总结所学习到的知识，这里创建了这个博客。

论文的主要内容包括5个部分：
1. Transformer Encoder-Decoder架构
2. Scaled dot-product Attention & Multi-head Attention
3. Position-wise Feed-Forward Networks
4. Embedding and Softmax
5. Positional Encoding

## Transformer Encoder-Decoder
Transformer的结构包括两部分：Encoder与Decoder。结构图如下
<img src="/blog/imgs/AIAYN_encoder_decoder.png">

Encoder部分的功能在于处理输入序列，将输入的句子文本，处理成模型能够理解的连续表示。
Decoder部分的功能在于根据encoder输出的理解内容，逐步输出目标序列。

## Attention
《Attention is all you need》论文的核心贡献在于其对Attention机制的计算。

Attention机制可以理解为，通过计算一段话中不同元素之间的关系，获得句子中每个token与另一个token之间的关联程度，然后根据关联程度，确定句子中哪些部分的token是需要关注的，从而使得计算机能够理解句子的逻辑关系。文中Attention机制分为两点：Scaled dot-product Attention与Multi-head attention。

### Scaled dot-product Attention
文中作者给出了Scaled dot-product Attention的具体计算思路：
``` 
               ╔══════════════════════════════════════Scale Dot-Product Attention════════════════════════════════════════════════════╗
input sequence -> positional encoding ─┬─> Query  projection -> Q matrix ┐                                                           ║
               ║                       ├─> Key    projection -> K matrix ┴──> matrix dot -> scale -> mask -> softmax ┐               ║
               ║                       └─> Value  projection -> V matrix --------------------------------------------┴─> matrix dot -> output
               ╚═════════════════════════════════════════════════════════════════════════════════════════════════════════════════════╝
```
其中，输入序列通过positional encoding，序列中每个token获得一个独一无二的有关其在句中位置的信息，随后通过Query，Key，Value投影矩阵，将其编码为模型中的隐藏向量。随后通过Query查询Key的方式，计算出了序列中，每两个token之间的关系，再根据关系从value向量中提取对应的信息，组合成模型理解后的表示。具体的公式如下：

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

数学原理:

input token shape = (N, 1),

Q, K matrix shape= (1, Dim_K), V matrix shape =(1, Dim_V)

Q_projection, K_projection shape = (N, Dim_K), V_projection shape = (N, Dim_V)

Query-Key $\text{softmax}\left(\frac{QK^T}{\sqrt{Dim_K}}\right)$ shape = (N, N)

Query-Key * V_projection shape = (N, Dim_V)

### Multi-head Attention
事实上，作者他们发现，QKV投影矩阵的维度对于性能的提升有重要作用。之前的QKV Martix shape = (N, Dim)，随后升级为(1, Dim, D_head)。

D_head的作用类似于让模型从不同的角度观察token序列，从而获得不同视角的信息，将不同视角的信息融合到模型理解中，从而提升了模型的理解能力。

$$ MultiHead(Q,K,V) = Concat(head_1, head_2, ..., headn) W^O $$
$$ where head_i = Attention(QW_i^Q, KW_i^K, VW_i^V) $$

## Position-wise Feed-Forward Networks
Attention部分糅合了一段话中每个token和别的token之间的关系，而position-wise feed-forward network则负责对每个token的表示进行非线性，位置独立的特征提取，增强模型的表达能力。与Attention结合，构成了“全剧信息融合+具体特征处理”的结构。

$$ FFN = ReLU(xW_1+b_1)W_2+b_2 $$

## Positional Encoding
在Embedding层将输入token序列转化为具有模型维度的张量之后，模型虽然可以理解每个token处的信息，但是缺少了token之间的位置信息。如模型知道第一个token的隐藏信息和第二个token的隐藏信息，但是模型无法知道第一个token在第二个token之前，因此对每个token按照位置添加信息，能够使得模型获得每个token的上下文信息，从而增强了表达能力。

$$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$

$$PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$

## Embedding & Softmax
Embedding层的核心功能是将离散的符号（如单词索引）映射为连续的稠密向量，让模型能捕捉单词间的语义相似性（比如让“猫”和“狗”在向量空间中距离更近）；在与Softmax结合时，Embedding层通常作为模型输出端的“查表器”：模型计算出的最终特征向量会与Embedding矩阵进行点积运算（相当于在词表中寻找最匹配的词），生成的分数再由Softmax归一化为概率分布，从而告诉我们在当前位置生成每个单词的可能性有多大。