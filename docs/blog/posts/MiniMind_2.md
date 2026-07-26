---
date: 2026-07-15
categories:
  - tech
tags:
  - 工程学习笔记
draft: true
---

# 这是MiniMind项目的学习笔记！（二）

#  第二章节：tokenizer

这一章的主要内容包括对于minimind内模型具体结构的学习，包括Attention机制-Group Query Attention（GQA），RMSNorm层（Root Mean Square），SwiGN FeedForward层等。

在minimind（64M）模型中，作者主要采用了decoder-only的架构，在推理阶段采用自回归推理的方式，能够显著提高推理速度

## Attention机制

### Group Query Attention（GQA）
Minimind项目中，作者使用的是分组注意力机制（GQA）。GQA注意力机制的原理在于并非让每个Q使用单独的一个K头，而是让多个Q使用同一个K头，这样在计算上能够节约大量的时间。如果对注意力机制的缺乏了解，可以看我另一篇博客《Attention is all you need学习笔记》。

在经典的《Attention is all you need》论文中，作者给出了Attention的计算思路（scaled dot-Product Attention）：

这里对比了GQA和MHA的关系：

```
《Attention is all you need》
  MHA (标准多头注意力)          GQA (分组查询注意力)          MQA (多查询注意力)
  Q: ■■■■■■■■  (8个头)         Q: ■■■■■■■■  (8个头)         Q: ■■■■■■■■  (8个头)
  K: ■■■■■■■■  (8个头)         K: ■■■■□□□□  (4个头)         K: ■□□□□□□□  (1个头)
  V: ■■■■■■■■  (8个头)         V: ■■■■□□□□  (4个头)         V: ■□□□□□□□  (1个头)
     Q、K、V一一对应              每2个Q共享1组K/V               所有Q共享1组K/V
```
在需要QK计算attention matrix的时候，通过复制拼接K，V矩阵，获得与Q相同shape的矩阵，从而使得矩阵乘法计算能够正确进行。

### KV cache
在训练阶段，模型接受的是整个sequence，包含了n个token的序列，因此可以直接计算整个token的注意力矩阵。然而，在推理阶段，模型的自回归推理，每次循环，模型预测下一个token，所获得的sequence都在逐步增长，因此每次循环计算的时候都会计算一个QKV projection表示。

因此在推理阶段，保留过去的token sequence的KV计算结果至关重要，否则每次迭代都需要完整计算当前句子的所有token的KV矩阵，会大大消耗计算资源，降低推理速度。

### Attention mask
在训练时期，我们希望模型能够根据输入的token序列，预测下一个token的概率。然而训练阶段我们一次性给模型整个sequence，得到一个nxn的attention matrix，为了使模型无法观测到未来token的信息，采用一个下三角矩阵，从而实现对于未来token信息的遮蔽。

## RMSNorm层

