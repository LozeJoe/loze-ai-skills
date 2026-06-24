# 博客封面 SVG 模板

## 使用时机

在 `content/posts/` 创建新 MDX 文章后，如果 frontmatter 中指定了 `cover: "/images/xxx.svg"`，必须在 `public/images/` 下生成对应的 SVG 文件，否则页面显示裂图。

## 模板 (1200×630, 渐变色 + 图标)

```python
import os

covers = {
    "article-slug-cover.svg": ("#HEX1", "#HEX2", "ICON", "LABEL"),
}

for filename, (c1, c2, icon, label) in covers.items():
    svg = f'''<svg xmlns="http://www.w3.org/2000/svg" width="1200" height="630" viewBox="0 0 1200 630">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:{c1}"/>
      <stop offset="100%" style="stop-color:{c2}"/>
    </linearGradient>
    <filter id="blur"><feGaussianBlur stdDeviation="60"/></filter>
  </defs>
  <rect width="1200" height="630" fill="url(#bg)"/>
  <circle cx="900" cy="200" r="300" fill="white" opacity="0.08" filter="url(#blur)"/>
  <circle cx="300" cy="500" r="250" fill="white" opacity="0.06" filter="url(#blur)"/>
  <text x="600" y="280" text-anchor="middle" font-size="120" font-family="system-ui,sans-serif" fill="white" opacity="0.95">{icon}</text>
  <text x="600" y="420" text-anchor="middle" font-size="36" font-family="system-ui,sans-serif" fill="white" opacity="0.7" font-weight="300" letter-spacing="6">{label}</text>
  <rect x="450" y="450" width="300" height="3" rx="2" fill="white" opacity="0.3"/>
</svg>'''
    path = os.path.join("public/images", filename)
    with open(path, 'w', encoding='utf-8') as f:
        f.write(svg)
```

## 推荐配色

| 主题 | 主色 | 深色 |
|------|------|------|
| TypeScript | `#3178C6` | `#235A97` |
| Rust | `#DEA584` | `#C07D5A` |
| Docker | `#2496ED` | `#1A6FB5` |
| Git | `#F05032` | `#C13D27` |
| CSS | `#1572B6` | `#0D4F7C` |
| AI | `#10B981` | `#059669` |
| 工具 | `#F59E0B` | `#D97706` |
| 阅读/生活 | `#8B5CF6` | `#6D28D9` |
