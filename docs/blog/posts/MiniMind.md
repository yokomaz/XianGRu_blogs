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

'PretrainDataset'类：预训练阶段  直接对原始文本进行tokenize，构造[BOS]+Sentence+[EOS]序列，pad到预设文本长度。

'SFTDataset'类：监督微调阶段  根据对话的JSONL内容，构造prompt和assistant，并且只对assistant的部分进行loss计算。

'DPODataset'类：模型偏好训练阶段  读取chosen/reject对话对，根据对话对的内容计算loss，使得模型生成更倾向于chosen文本的概率分布。

'RLAIFDataset'类：强化学习训练阶段  用于生成RL对齐数据集。

## 模型模块
```
model/
├── model_lora.py
├── model_minimind.py
├── tokenizer.json
└── tokenizer_config.json
```

model模块主要用于构建整体项目中的模型结构。

### 类：
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

用于将文字映射到数字的工具箱。这里作者使用了自己训练的6400个词表的tokenzier，对应于64M模型的能力。


## 数据模块
```
dataset/
├── lm_dataset.py
├── PretrainDataset 预训练数据集
├── SFTDataset  监督微调数据集
├── DPODataset   偏好数据
└── RLAIFDataset(Reinforcement learning, AI Feedback)  强化学习的prompt组织

lm_dataset.py, 根据训练的不同阶段，将数据集整理成input与labels的格式。