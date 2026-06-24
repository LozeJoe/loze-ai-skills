# Loze Hexo 博客 — 搭建与维护参考

> 关联技能: loze-workflow
> 项目路径: `C:\Users\Loze\projects\loze-blog`

---

## 技术栈

- **框架**: Hexo 8.0.0
- **主题**: Butterfly (jerryc127/hexo-theme-butterfly)
- **渲染器**: hexo-renderer-pug + hexo-renderer-stylus
- **插件**: hexo-generator-searchdb, hexo-wordcount
- **部署**: 本地预览 → 后续 GitHub Pages / Vercel

## 项目结构

```
loze-blog/
  _config.yml               ← Hexo 主配置
  _config.butterfly.yml      ← 主题配置 (文艺风定制)
  source/
    _posts/                  ← Markdown 博客文章
    about/index.md           ← 关于页面
    css/loze.css             ← 自定义文艺样式
    img/default_cover.svg    ← 默认封面 (暖色调 SVG)
  themes/butterfly/          ← Butterfly 主题 (git clone)
  scaffolds/                 ← 文章模板
```

## 常用命令

```bash
cd ~/projects/loze-blog

# 启动预览 (默认 4000 端口)
hexo server -p 4000

# 新建文章
hexo new "文章标题"
# → 生成 source/_posts/文章标题.md
# 用 Obsidian 直接编辑

# 生成静态文件 → public/
hexo generate

# 清理 + 重建
hexo clean && hexo generate

# 部署 (需先在 _config.yml 配 deploy)
hexo deploy
```

## 文艺风格配置要点

### 配色
- 背景: `#faf8f5` (暖奶油白)
- 文字: `#4a3f35` (暖棕，非纯黑)
- 强调: `#5b7b6a` (墨绿)
- 卡片: `#fffef9` + `border: 1px solid #ede4d3`
- 夜间: `#2c2416` 深褐底

### 字体
- 中文: Noto Serif SC (衬线体)
- 英文: Crimson Text (衬线体)
- 代码: Fira Code / Cascadia Code
- 引入方式: Google Fonts CDN (在 `_config.butterfly.yml` 的 `inject.head` 中)

### 自定义 CSS
- 路径: `source/css/loze.css`
- 覆盖项: 导航栏、文章卡片、代码块、块引用、侧边栏、滚动条
- 暗色模式: `[data-theme="dark"]` 选择器覆盖

### 封面 SVG
- 路径: `source/img/default_cover.svg`
- 规格: 800x400, 暖调渐变, 标题居中, 衬线字体
- 可替换为摄影作品

## 写作工作流

```
1. 在 Obsidian 中写草稿
2. hexo new "标题" → 粘贴内容
3. 补充 frontmatter (tags, categories, cover)
4. hexo server 预览
5. 满意后 hexo generate
6. 部署上线
```

## 文章 Frontmatter 模板

```yaml
---
title: 文章标题
date: 2026-06-03
tags: [标签1, 标签2]
categories: [分类]
description: 文章摘要
cover: /img/default_cover.svg
---
```

## 部署方案

| 平台 | 命令 | 域名 |
|------|------|------|
| GitHub Pages | `hexo deploy` (配 git deployer) | loze.github.io |
| Vercel | git push 自动部署 | loze.vercel.app |
| Netlify | 拖拽 public/ 目录 | loze.netlify.app |

最推荐 GitHub Pages：免费、自动 HTTPS、一行命令部署。

## 已安装插件

```
hexo-renderer-pug        # Pug 模板渲染
hexo-renderer-stylus     # Stylus CSS 渲染
hexo-generator-searchdb  # 本地搜索 (JSON)
hexo-wordcount           # 字数统计 + 阅读时间
```

## 网络限制注意

- hexo init 默认从 GitHub 拉取 starter，在受限网络下可能超时。可手动 npm install 依赖后继续。
- Butterfly 主题通过 gh-proxy.com 镜像 clone：`git clone https://gh-proxy.com/https://github.com/jerryc127/hexo-theme-butterfly themes/butterfly`
- Google Fonts 在国内可能加载慢，后续可改为本地托管字体。
