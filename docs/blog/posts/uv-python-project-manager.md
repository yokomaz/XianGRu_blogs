---
date: 2026-07-07
categories:
  - tech
tags:
  - python
  - uv
  - 工具
draft: false
---

# 用 uv 管理 Python 项目

## 什么是 uv

[uv](https://github.com/astral-sh/uv) 是用 Rust 写的一个极速 Python 包管理器，由 Ruff 的作者 [Charlie Marsh](https://github.com/charliermarsh) 开发。它可以替代 `pip`、`virtualenv`、`pip-tools` 甚至 `poetry`。

## 为什么我喜欢它

### 1. 快，真的很快

```bash
# pip install 一个包通常要等几秒到几十秒
# uv 几乎秒装完
$ time uv add mkdocs
# 0.8s — 比 pip 快 10-100 倍
```

### 2. 一个工具搞定一切

- `uv init` — 创建项目
- `uv add <pkg>` — 安装依赖
- `uv run <cmd>` — 在虚拟环境中运行命令
- `uv lock` — 锁定依赖版本

不再需要在 `pip`、`virtualenv`、`pip-tools` 之间来回切换。

### 3. 兼容性好

生成的 `pyproject.toml` 完全标准，兼容任何 Python 工具链。就算以后不用 uv 了，项目照样跑。

## 小结

对于新项目，我已经全面转向 uv 了。老项目迁移也很简单，`uv init` 然后 `uv add -r requirements.txt` 就搞定了。

强烈推荐给还在用 pip + virtualenv 的朋友。
