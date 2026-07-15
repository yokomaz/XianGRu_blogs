---
date: 2026-07-10
categories:
  - tech
tags:
  - 学习
  - 笔记
draft: false
---

# 这是MiniMind项目的学习笔记！

前段时间接触到了一个很有意思的语言模型项目--MiniMind，作者是Jingyao Gong，地址在：https://github.com/jingyaogong/minimind。这个项目的宗旨是完全从0开始，在资源有限的情况下，从零训练一个能够交互的语言模型。

MiniMind项目涵盖了大语言模型的极简结构与完整训练流程。覆盖数据清洗，tokenizer训练，模型预训练（Pretrain），模型监督微调（Supervised fine-tune，SFT），LoRA微调，RLHF，RLAIF，Agent（tools use），自适应思考与模型蒸馏。可以说是完整的大模型项目的小模型版本。这份Blog的目的在于记录我学习这个项目的经历，一步一步拆解我在学习过程中的收获。

我之前也接触过github开源项目，但是我一直没有总结出一套自己的学习方法。这里借鉴了另一位大佬Enping Hu的学习路线，项目地址在：https://github.com/Enping-Hu/minimind-deep-dive。

***用乐高拼出一架飞机，远比坐在头等舱里飞行更让人兴奋！---- Jingyao Gong***

***认识->实践->再认识->再实践，以致无穷。 ---- 毛泽东《实践论》***

# 第一章节：了解项目结构
从接触到这个项目开始，首先了解整个项目的结构，包括有哪些模块，每个模块的作用是什么。MiniMind项目的整体结构如下：

```
MiniMind/
├── dataset/
├── image/
├── model/
├── scripts/
├── trainer/
├── CODE_OF_CONDUCT.md
├── LICENCE
├── README.md
├── eval_llm.md
├── requirements.txt
└── .gitignore
```

## 数据集模块
```
dataset/
├── dataset.md
└── lm_dataset.py
``` 

Dataset模块主要用于根据训练阶段处理数据集，包含以下几个dataset类：

``` PretrainDataset ```：预训练阶段  直接对原始文本进行tokenize，构造[BOS]+Sentence+[EOS]序列，pad到预设文本长度。

``` SFTDataset ```：监督微调阶段  根据对话的JSONL内容，构造prompt和assistant，并且只对assistant的部分进行loss计算。

``` DPODataset ```：模型偏好训练阶段  读取chosen/reject对话对，根据对话对的内容计算loss，使得模型生成更倾向于chosen文本的概率分布。

``` RLAIFDataset ```：强化学习训练阶段  用于生成RL对齐数据集。

## 模型模块
```
model/
├── model_lora.py
├── model_minimind.py
├── tokenizer.json
└── tokenizer_config.json
```

model模块主要用于构建整体项目中的模型结构。

### 具体模型：
``` LoRA ```： LoRA低秩适配网络，包含两种初始化矩阵：1. 高斯初始化。2. 全零初始化。通过A-B旁路实现高效微调。

``` MiniMindConfig ```：用于模型的参数配置，包括：hidden_size，number of heads, MoE参数, ROPE位置参数等。

``` RMSNorm ```：RMS Normalization层。

``` Attention ```：多头注意力机制，支持Flash attention和KV Cache，包含Q/K norm。

``` FeedForward ```：标准SwiGLU FFN: gate_projection + up_proj  -> SiLU -> down proj

``` MOEFeedForward ```: 混合专家FFN，包含路由器gate和多个feedforward网络，支持top-k输出

``` MiniMindBlock ```： 标准Transformer层： RMSNorm -> Attention -> Resudial Connection -> RMSNorm -> FFN/MOE -> Residual Connection

``` MiniMindModel ```：完整的Transformer编码器：embedding -> dropout -> N x MiniMindBlock -> RMSNorm, 包括RoPE buffer和Past key values

``` MiniMindForCausalLM ```： 因果语言模型头部，包含lm_head，自回归生成。

### Tokenizer

用于将文字映射到token id，并在推理之后，将token id对应回文字的词典。从0开始的情况下，tokenzier是需要根据训练语料进行训练，包括词表的构造和切分规则。

同时词表的大小直接应想到了embedding层的维度，因为Embedding层需要对应词表大小，不一致的对应关系将导致训练的不稳定以及输出的幻觉。

作为对64M模型的词表，作者使用了6400长度的词表。


## 数据模块
```
dataset/
├── lm_dataset.py
├── PretrainDataset 预训练数据集
├── SFTDataset  监督微调数据集
├── DPODataset   偏好数据
└── RLAIFDataset(Reinforcement learning, AI Feedback)  强化学习的prompt组织
```

lm_dataset.py, 根据训练的不同阶段，将数据集整理成input与labels的格式。

LLM的训练步骤通常分为三阶段：预训练阶段；监督微调阶段；强化学习阶段。

Pretrian数据集的格式为text -> next token prediction。目的是让模型通过文本语料，学习到不同句子中token之间的统计学关联。

SFT数据集的格式为conversations格式，包含assistant，user，system，包含tools call训练。

RL数据集的格式为good answer和bad answer组合，迫使模型选择good answer。

## 训练策略

```
trainer/
├── train_pretrain.py
├── train_full_stf.py
├── train_dpo.py
├── train_grpo.py
├── train_ppo.py
├── train_distillation.py
├── train_agent.py
├── train_lora.py
├── train_tokenizer.py
└── train_utils.py
```

目前主流的大模型训练阶段包括：预训练（pretrain）->监督微调（supervised fine-tuning）->强化学习微调（RL fine-tuning），因此这里介绍四个主要的训练阶段。作者提供了几乎包括训练大模型期间所有的训练方法，每一个方法会在之后学习到的时候介绍。

``` Pretrain ```: 预训练阶段的目的主要是使得模型从海量的自然语言文本中，学习到各个单词之间的统计规律。例如：在日常生活中，“苹果”的“苹”字在大多数语境下，与“果”字的关联比较密切，或者“苹果”一词一般情况下会出现在讨论水果的语境内，而不会说在宇宙中有一个“苹果”。因此，模型通过海量的文本，学习到了这类词汇出现概率的统计学分布。预训练的方法是：在海量的，没有人工标注的各类句子当中，模型通过预测下一个词的无监督学习方式，理解语言规律与世界知识。

``` SFT ```: 在预训练的过程中，模型学会了如何根据一段话来预测下一个词出现的概率，但是模型并没有学会对话的能力，即模型不会在用户给出指令后进行回复，只会一味的预测用户指令的下一个单词的概率。因此在监督微调的过程里，通过构造高质量的对话文本（指令-回答）对，模型能够学习到如何根据用户的指令，来回答出对应的问题，比如：“用户”：今天的天气如何？ “模型”：今天的天气不错，xx度，紫外线指数xx，......。等。从而使得模型具有基本的交互能力。

``` RL ```: 在通过了监督微调阶段之后，模型已经能够根据用户的指令回答问题了，但是回答问题的质量仍然处于不可控制的阶段，比如模型可能会为了回答用户的问题，凭空捏造一些不存在的事实（模型幻觉），或者回答一些违法犯罪的内容。因此需要一种方法来引导模型的回答朝着更符合人们的价值观，偏好等方向发展。强化学习训练阶段，模型会根据同一个指令，生成多个不同的回答，然后根据对多个不同回答进行打分，使得模型的回答更偏向于分数较高的回答。一般情况下，有人类对模型回答进行打分（Human feedback reinforcement learning, RLHF），也有训练一个奖励模型，奖励模型根据回答进行打分（RLAI）。

# 第二章节：模型

这一章的主要内容包括对于minimind内模型具体结构的学习，包括Attention机制-Group Query Attention（GQA），RMSNorm层（Root Mean Square），SwiGN FeedForward层等。

在minimind（64M）模型中，作者主要采用了decoder-only的架构，在推理阶段采用自回归推理的方式，能够显著提高推理速度

## Attention机制

### Group Query Attention（GQA）
Minimind项目中，作者使用的是分组注意力机制（GQA）。GQA注意力机制的原理在于并非让每个Q使用单独的一个K头，而是让多个Q使用同一个K头，这样在计算上能够节约大量的时间。如果对注意力机制的缺乏了解，可以看我另一篇博客《Attention is all you need学习笔记》。

在经典的《Attention is all you need》论文中，作者给出了Attention的计算思路（scaled dot-Product Attention）：
``` 
               ╔══════════════════════════════════════Scale Dot-Product Attention════════════════════════════════════════════════════╗
input sequence -> positional encoding ─┬─> Query  projection -> Q matrix ┐                                                           ║
               ║                       ├─> Key    projection -> K matrix ┴──> matrix dot -> scale -> mask -> softmax ┐               ║
               ║                       └─> Value  projection -> V matrix --------------------------------------------┴─> matrix dot -> output
               ╚═════════════════════════════════════════════════════════════════════════════════════════════════════════════════════╝
```

这里对比了GQA和MHA的关系：

```
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
在训练时期，我们希望模型能够根据输入的token序列，预测下一个token的概率。然而训练阶段我们一次性给模型整个sequence，得到一个nxn的attention matrix，为了使模型无法观测到未来token的信息，采用一个下三角矩阵，来遮蔽未来token的信息
```

```
