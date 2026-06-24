# Vue 3 液态玻璃 + API 回退参考

## 多层 z-index 架构

液态玻璃 (`backdrop-filter: blur()`) 需要正确的层级结构才能生效：

```
z-0:  <video> 视频/纹理背景 (opacity 0.2~0.3)
z-1:  半透明颜色叠加层 (bg-color opacity 0.5~0.7, 确保文字可读)
z-10: 主内容区 (透明背景) + .glass 卡片 (blur 16~20px)
```

关键：根元素和 main 不能有纯色背景，否则玻璃看不到视频。

## Tailwind CSS 变量桥接

Tailwind JIT 不认识自定义类名 `bg-bg`, `text-text` 等，需要：
- `tailwind.config.js` 添加 `safelist: [...]`
- 或在 Vue 模板中用 `style="color: var(--text)"` 内联绑定（最可靠）

```css
:root {
  --bg: #faf9f6;
  --text: #1c1917;
  --accent: #b7610a;
  --card: #ffffff;
  --border: #e7e5e2;
  --text-muted: #78716c;
}
.dark {
  --bg: #1c1917;
  --text: #e7e5e2;
  --accent: #f0a050;
}
```

## Vite public/ 大文件 500 错误

照片/视频（>1MB）在 public/ 目录被 Vite 当作 JS 模块处理 → HTTP 500。

修复：
```ts
// vite.config.ts
export default defineConfig({
  build: { assetsInlineLimit: 0 },
  // ...
})
```

且 Vue 模板中媒体文件用 `:src="variable"` 绑定，避免 `<source src="/path.mp4">` 被编译器转为模块导入。

## API 后端不可用双重回退

axios 拦截器只在网络错误 (`!err.response`) 时触发回退。页面内需要第二层保障：

```ts
// api/index.ts - 拦截器层
api.interceptors.response.use(
  res => res.data,
  async err => {
    if (!err.response) { /* 回退到 /data.json */ }
    return Promise.reject(err)
  }
)

// BlogDetail.vue - 组件层
try {
  await api.get('/posts/' + slug)
} catch {
  const r = await fetch('/data.json')  // 第二层
  const data = await r.json()
  post.value = data.posts.find(p => p.slug === slug)
}
```

## data.json 格式

```json
{
  "posts": [{
    "slug": "hello-world", "title": "...", "content": "...",
    "date": "2024-01-01", "tags": ["tag1", "tag2"],
    "category": "技术", "description": "...",
    "coverImage": "/images/cover.svg", "draft": false
  }],
  "entries": [{
    "slug": "diary-1", "title": "...", "content": "...",
    "date": "2024-01-15", "mood": "开心", "weather": "晴"
  }],
  "notes": [{ "slug": "n1", "title": "", "content": "...", "date": "..." }],
  "photos": ["/photos/img1.jpg"],
  "friends": [{ "name": "...", "url": "...", "avatar": "...", "description": "..." }]
}
```

## MDX 标签迁移陷阱

从 MDX frontmatter `tags: [博客, Next.js]` 迁移到 JSON 时，简单 `split(',')` 可能产生 `['["博客"', ' "Next.js"']']` 嵌套字符串。

正确清洗：
```python
def clean_tags(tags):
    if isinstance(tags, str):
        return [t.strip().strip('"\'[]') for t in tags.split(',') if t.strip()]
    return tags
```
