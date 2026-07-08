# XianGRu's Blog

个人博客，记录生活、读书和技术学习。

## 技术栈

- [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- Python 依赖管理：[uv](https://github.com/astral-sh/uv)
- 托管：[GitHub Pages](https://pages.github.com/)

## 快速开始

```bash
# 1. 克隆项目
git clone git@github.com:xiangruchen/XianGRu_blogs.git
cd XianGRu_blogs

# 2. 安装依赖（需要先装 uv）
uv sync

# 3. 本地预览
uv run mkdocs serve
# 打开 http://localhost:8000

# 4. 写文章
# 在 docs/blog/posts/ 下创建新 .md 文件，格式参考已有文章

# 5. 构建
uv run mkdocs build
# 输出在 site/ 目录
```

## 文章分类

| 分类 | 目录 | 说明 |
|------|------|------|
| `life` | `docs/blog/posts/` | 生活随想 |
| `books` | `docs/blog/posts/` | 读书笔记 |
| `tech` | `docs/blog/posts/` | 技术学习 |

文章通过 frontmatter 中的 `categories` 字段进行分类。

## 文章模板

```markdown
---
date: 2026-07-08
categories:
  - tech          # life / books / tech
tags:
  - python
  - 标签2
draft: false      # true = 草稿，不会发布
---

# 标题

正文写在这里，支持标准 Markdown 语法。
```

## 部署到 GitHub Pages

推荐使用 GitHub Actions 自动部署，创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync
      - run: uv run mkdocs build
      - uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
```

然后在 GitHub 仓库 Settings → Pages 中，Source 选 "GitHub Actions"。
