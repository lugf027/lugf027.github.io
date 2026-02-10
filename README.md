# lugf027 的博客

这是我的个人博客，使用 [Hugo](https://gohugo.io/) 构建，托管在 [GitHub Pages](https://pages.github.com/)。

## 🚀 快速开始

### 前置要求

- [Hugo](https://gohugo.io/installation/) (Extended 版本)
- [Git](https://git-scm.com/)

### 本地开发

```bash
# 克隆仓库
git clone https://github.com/lugf027/lugf027.github.io.git
cd lugf027.github.io

# 本地预览（包含草稿）
hugo server -D

# 访问 http://localhost:1313
```

### 创建新文章

```bash
hugo new posts/my-new-post.md
```

### 构建网站

```bash
hugo
```

构建产物会输出到 `public/` 目录。

## 📁 目录结构

```
.
├── content/              # 内容目录
│   ├── _index.md        # 首页内容
│   ├── about.md         # 关于页面
│   └── posts/           # 博客文章
├── layouts/             # 布局模板
│   ├── _default/        # 默认布局
│   └── index.html       # 首页模板
├── assets/              # 资源文件
│   └── css/             # 样式文件
├── static/              # 静态文件
├── .github/workflows/   # GitHub Actions
└── hugo.toml            # Hugo 配置文件
```

## 🔧 配置

主要配置在 `hugo.toml` 文件中：

- `baseURL` - 网站地址
- `title` - 网站标题
- `params.description` - 网站描述
- `params.author` - 作者名称

## 📝 部署

推送到 `main` 分支后，GitHub Actions 会自动构建并部署到 GitHub Pages。

### 首次部署步骤

1. 创建 GitHub 仓库 `lugf027.github.io`
2. 在仓库设置中启用 GitHub Pages（Settings → Pages → Source → GitHub Actions）
3. 推送代码到 `main` 分支

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/lugf027/lugf027.github.io.git
git push -u origin main
```

## 📄 License

MIT License
