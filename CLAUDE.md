# CLAUDE.md

## 项目概述

基于 MkDocs + Material for MkDocs 主题的个人博客，部署在 GitHub Pages（`yokomaz.github.io/XianGRu_blogs/`）。内容涵盖技术学习笔记、读书笔记和生活记录。

## 博客写作约定

- **技术类博客分两类：**
  - **项目笔记**（如 MiniMind 系列）：聚焦具体项目的实现细节和学习过程
  - **独立概念博客**（如 Attention 论文笔记、LLM 基础概念）：深入讲解通用的技术原理
- **项目笔记中的基础概念**：用链接指向独立概念博客，避免重复介绍
- **写作流程**：边做边记草稿，阶段性整理成博客发布

## 当前博客体系

| 文件 | 内容 |
|------|------|
| `docs/blog/posts/MiniMind_1.md` | MiniMind 项目学习笔记（一）— 项目结构与 Tokenizer |
| `docs/blog/posts/MiniMind_2.md` | MiniMind 项目学习笔记（二）— 模型结构与训练 |
| `docs/blog/posts/Attention_is_all_you_need.md` | 经典论文《Attention is all you need》笔记 |
| `docs/blog/posts/basic_in_LLM.md` | LLM 基础概念合集（GQA、KV Cache、RMSNorm 等） |
| `docs/blog/posts/logic_of_the_world.md` | 读书/思考笔记 |
| `docs/blog/posts/OrangePI_project.md` | OrangePI 项目笔记 |
| `docs/blog/posts/utils.md` | Markdown 工具集 |

## 发布流程

1. 在 `docs/blog/posts/` 下编写 `.md` 文件，设置 `draft: false`
2. 提交并推送到 `main` 分支
3. GitHub Actions 自动构建并部署到 GitHub Pages
