# Loze Blog — 常见问题与修复

## 暗色模式 CSS 变量与 class 不一致

**问题**: `@media (prefers-color-scheme: dark)` 定义的变量不受 ThemeProvider 的 `.dark` class 控制。
**修复**: 将暗色变量从媒体查询改为 `.dark` 选择器。

```css
/* ❌ */
@media (prefers-color-scheme: dark) {
  :root { --bg: #2c2416; }
}

/* ✅ */
.dark {
  --bg: #2c2416;
}
```

## Tailwind `hidden` 与 JS `style.display` 冲突

**问题**: `hidden` 使用 `display: none !important`，JS 设置 `style.display = "flex"` 被覆盖。
**修复**: 用 `useState` 条件渲染替代 hidden + style 切换。

```tsx
// ❌
<img onError={(e) => {
  e.target.style.display = "none";
  nextSibling.style.display = "flex";
}} />
<span className="hidden">...</span>  // !important 覆盖了 flex

// ✅
const [imgError, setImgError] = useState(false);
{imgError ? <Fallback /> : <img onError={() => setImgError(true)} />}
```

## 服务端组件不能用 `"use client"` + `fs` 同时存在

**问题**: `"use client"` 组件中不能用 `fs`，服务端组件中不能用 `onError` 等事件处理器。
**修复**: 拆分为服务端 page.tsx (读数据) + 客户端组件 (渲染+交互)，通过 props 传递数据。

## 嵌套三元表达式导致 JSX 解析错误

**问题**: `{a ? null : b ? (...长JSX...) : (...长JSX...)}` 容易出错。
**修复**: 改为 `{a && (...)}` + `{b && (...)}` 的独立条件块。

```tsx
// ❌
{isNote ? null : isDiary ? (
  <MoodSelector />
) : null}

// ✅
{isDiary && <MoodSelector />}
{!isDiary && !isNote && <TagInput />}
```

## MSYS2 路径 vs Windows 路径

**问题**: Python 脚本中使用 `/c/Users/Loze/...` 路径，`Path.exists()` 返回 false。
**修复**: Python 脚本中使用 `r"C:\Users\Loze\..."` 原生 Windows 路径。

## Next.js 路由重定向导致 404

**问题**: `redirects()` 中 `/blog/:slug*` → `/posts/:slug*` 把博客路由全部重定向到不存在的位置。
**修复**: 删除错误的重定向规则。除非确实有旧路由需要迁移，否则不要加 redirect。

## 草稿文章 URL 泄漏

**问题**: `getPostBySlug()` 和 `getAllPostSlugs()` 不过滤 `draft: true`，草稿可通过 URL 直接访问。
**修复**: 在这两个函数中增加草稿过滤。

```typescript
export function getPostBySlug(slug: string): Post | null {
  const post = parsePost(...);
  if (post && post.frontmatter.draft) return null;  // ← 加这行
  return post;
}
```

## 标签云点击无筛选效果

**问题**: TagCloud 生成 `/blog?tag=Vue` 链接，但 blog/page.tsx 不读取 `tag` 参数。
**修复**: 在 blog/page.tsx 中增加 `tag` 参数处理，调用 `getPostsByTag()` 筛选。

## 友链头像回退失效 (Tailwind hidden 冲突的另一个案例)

参见上方 "Tailwind hidden 与 JS style.display 冲突" — 同一个根因，同样的修复方式。

## Next.js 自定义 404 页缺失 → 英文提示

**问题**: Next.js App Router 默认 404 页面是英文 "This page could not be found"。项目没有自定义 `not-found.tsx` 时，中文用户看到英文错误页。

**修复**: 在 `src/app/` 下创建 `not-found.tsx`：

```tsx
import Link from 'next/link'

export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-[60vh] px-4 text-center">
      <h1 className="text-6xl font-bold text-amber-600 mb-4">404</h1>
      <h2 className="text-2xl font-semibold mb-2" style={{ color: 'var(--text)' }}>
        页面未找到
      </h2>
      <p className="mb-8" style={{ color: 'var(--text-muted)' }}>
        抱歉，您访问的页面不存在或已被移除。
      </p>
      <Link href="/" className="px-6 py-3 rounded-full bg-amber-600 text-white hover:bg-amber-700 transition-colors">
        返回首页
      </Link>
    </div>
  )
}
```

**关键**: 文件名必须是 `not-found.tsx`（Next.js 约定），不是 `404.tsx` 或 `NotFound.tsx`。使用 CSS 变量（`var(--text)` 等）保持与站点主题一致。

## Vue Photos.vue 绕过 API 拦截器 → 相册永远为空

**问题**: `Photos.vue` 在 `onMounted` 中直接 `fetch('/data.json')`，不经过 axios API 拦截器。如果 `public/data.json` 缺少 `photos` 数组，即使 `public/photos/` 目录下有文件、后端运行正常，相册页面仍显示"还没有照片"。

**修复**: 在 `data.json` 中添加 `photos` 数组，包含相对于 public 目录的图片路径：

```json
{
  "posts": [...],
  "entries": [...],
  "notes": [...],
  "photos": [
    "/photos/example1.jpg",
    "/photos/example2.jpg"
  ]
}
```

**影响面**: 后端运行时的 `/api/photos` 和 `data.json` 的 `photos` 字段是**两套独立数据**。Photos.vue 只读 data.json，所以：
- 后端上传新照片后，必须同步更新 data.json
- 或改造 Photos.vue 通过 axios 调 API（需同时在 `api/index.ts` 添加 `/photos` 的 fallback 处理）
