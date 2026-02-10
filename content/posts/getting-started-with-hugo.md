---
title: "Hugo 快速入门指南"
date: 2026-02-10
draft: false
tags: ["Hugo", "博客", "教程"]
categories: ["技术"]
---

## Hugo 是什么？

Hugo 是一个用 Go 语言编写的静态网站生成器，以其极快的构建速度而闻名。

<!--more-->

### 为什么选择 Hugo？

1. **速度快** - 毫秒级构建时间
2. **安装简单** - 单一二进制文件，无依赖
3. **主题丰富** - 大量开源主题可选
4. **部署方便** - 生成静态文件，可部署到任何地方

### 基本概念

#### 内容组织

Hugo 的内容放在 `content` 目录下：

```
content/
├── _index.md        # 首页内容
├── about.md         # 关于页面
└── posts/           # 文章目录
    ├── _index.md
    └── my-post.md
```

#### Front Matter

每篇文章开头的元数据：

```yaml
---
title: "文章标题"
date: 2026-02-10
draft: false
tags: ["标签1", "标签2"]
categories: ["分类"]
---
```

### 常用命令

```bash
# 创建新文章
hugo new posts/my-new-post.md

# 本地预览
hugo server -D

# 构建网站
hugo

# 构建到指定目录
hugo -d public
```

### 总结

Hugo 是一个非常适合个人博客的工具，学习曲线平缓，功能强大。推荐给所有想要搭建个人博客的朋友！

---

*如有问题，欢迎留言讨论！*
