# 博客搭建记录

> 日期：2026-07-08 ~ 2026-07-09

## 项目信息

| | |
|---|---|
| 本地路径 | `/home/xiangru-chen/projects/XianGRu_blogs` |
| GitHub 仓库 | `github.com/yokomaz/XianGRu_blogs` |
| 博客地址 | `yokomaz.github.io/XianGRu_blogs/` |
| GitHub 用户名 | yokomaz |

## 技术栈

- **静态站点生成器**：MkDocs + Material for MkDocs (v9.7.6)
- **依赖管理**：uv (Python 3.12)
- **托管**：GitHub Pages（GitHub Actions 自动部署）
- **写作**：Markdown

## 项目结构

```
XianGRu_blogs/
├── pyproject.toml          # uv 依赖管理
├── mkdocs.yml              # MkDocs 配置（核心文件）
├── .github/workflows/
│   └── deploy.yml          # 自动部署到 GitHub Pages
├── .gitignore
├── README.md
└── docs/
    ├── index.md            # 首页
    ├── about.md            # 关于页
    ├── tags.md             # 标签页
    └── blog/
        ├── index.md        # 博客列表（插件自动管理）
        └── posts/          # 所有文章放在这里
            ├── hello-world.md                (分类: life)
            ├── my-coffee-corner.md           (分类: life)
            ├── pragmatic-programmer-notes.md (分类: books)
            └── uv-python-project-manager.md  (分类: tech)
```

## 三个文章分类

| 分类 | 用途 | 访问地址 |
|------|------|---------|
| `life` | 生活随想 | `/blog/category/life/` |
| `books` | 读书笔记 | `/blog/category/books/` |
| `tech` | 技术学习 | `/blog/category/tech/` |

## 写新文章

在 `docs/blog/posts/` 下创建 `.md` 文件，模板：

```markdown
---
date: 2026-07-09
categories:
  - tech          # life / books / tech
tags:
  - python
  - 标签名
draft: false      # true = 草稿，不发布
---

# 文章标题

正文写在这里，支持标准 Markdown 语法。

<!-- more -->

折叠部分（首页预览不会显示，点进文章才能看到）
```

## 常用命令

```bash
cd /home/xiangru-chen/projects/XianGRu_blogs

# 本地预览（打开 http://localhost:8000）
uv run mkdocs serve

# 构建静态文件（输出到 site/ 目录）
uv run mkdocs build

# 推送到 GitHub
git add -A && git commit -m "描述" && git push
```

## 部署流程

1. Push 到 `main` 分支后，GitHub Actions 自动执行 `.github/workflows/deploy.yml`
2. 构建完成后自动部署到 GitHub Pages
3. 博客地址：`https://yokomaz.github.io/XianGRu_blogs/`

## 踩过的坑

### 1. post_dir 配置错误导致博客列表不显示文章

- **现象**：博客列表页只显示标题，没有任何文章
- **原因**：将 `post_dir` 设为 `posts`，但默认值是 `{blog}/posts`。设为 `posts` 后插件在 `docs/posts/` 找文章，但文章实际在 `docs/blog/posts/`
- **解决**：删除 `post_dir` 配置，使用默认值

### 2. GitHub 用户名修正

- 项目初期使用了 `xiangruchen`，但实际 SSH key 对应的用户名为 `yokomaz`
- 修改了 `mkdocs.yml`（site_url、social link）、`docs/about.md`、`README.md` 中的所有引用

### 3. Git 远程地址

- 远程地址：`git@github.com:yokomaz/XianGRu_blogs.git`
- SSH 验证通过的用户是 yokomaz

## 后续可扩展

- 自定义域名
- 评论系统（如 Giscus）
- 自定义 CSS 样式
- 图床配置
- RSS 订阅
