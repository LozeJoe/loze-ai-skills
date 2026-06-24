---
name: loze-blog
description: "刘哲凯的 Next.js 静态博客 — MDX/GSAP/管理后台/API路由/文件系统CRUD 全栈模式"
version: 1.0.0
author: Hermes Agent (for Loze)
platforms: [windows]
metadata:
  hermes:
    tags: [博客, Next.js, GSAP, 管理后台, MDX, 刘哲凯]
    related_skills: [loze, gsap-core, gsap-react, gsap-scrolltrigger, gsap-timeline]
---

# Loze Blog — Next.js 静态博客

## 触发条件

用户提到博客、loze-blog、管理后台、admin、文章、日记、相册、友链、说说、GSAP动画、站点外观时加载。

## 项目概览

- **路径**: `~/projects/loze-blog-next/`
- **框架**: Next.js 16.2.7 (App Router, TypeScript, Tailwind CSS v4)
- **端口**: 6708 (`npm run dev -- -p 6708`)
- **设计**: 文艺风 — 暖琥珀配色（法式烘焙），Noto Sans SC 无衬线体，毛玻璃卡片
- **动画**: GSAP + ScrollTrigger (替代 framer-motion)
- **认证**: JWT (`jose`) + httpOnly Cookie，管理后台 `/admin`，账号 `Loze` 密码 `20060708`
- **内容**: MDX 文件存储，API Route 读写本地文件系统

## 架构模式

### 服务端/客户端组件分离
Next.js App Router 中，`fs`/`path` 等 Node.js 模块只能在服务端组件使用。模式：
```
page.tsx (Server) → 读取数据 → ClientPage (Client) → 渲染+交互
```
- 服务端: `getAllPosts()`, `getSiteConfig()` 等文件读取
- 客户端: `useState`, `useGSAP`, `fetch()` 等

### shared 类型文件
当客户端组件需要服务端类型时，抽取共享文件：
```
src/lib/site-config-types.ts   ← 类型+常量（客户端/服务端共享）
src/lib/site-config.ts         ← server-only, fs 操作
```

### CSS 变量体系
项目使用语义 CSS 变量（在 `globals.css` 定义，Tailwind v4 `@theme inline` 映射）：
```
--bg / --text / --accent / --card / --border / --text-muted / --accent-hover / --bg-secondary
→ Tailwind: bg-bg / text-text / bg-card / border-border / text-text-muted / bg-accent
```
暗色模式通过 `.dark` class 切换（非 `@media`），与 ThemeProvider 保持一致。

## 文件结构

```
src/app/           — 页面路由
  admin/           — 管理后台 (/admin)
    login/         — 登录页
    settings/      — 站点外观
    profile/       — 个人信息
    password/      — 修改密码
src/components/    — 共享组件
  admin/           — 管理后台专用组件
  mdx/             — MDX 自定义组件 (Callout, CodeBlock)
src/lib/           — 工具库
  posts.ts         — 文章读写
  mdx.ts           — MDX 编译 (next-mdx-remote + Shiki)
  animations.ts    — GSAP 动画系统
  auth.ts          — API 鉴权中间件
  admin-auth.ts    — JWT 签发/校验
  site-config.ts   — 站点配置 (server-only)
  site-config-types.ts — 配置类型+预设 (shared)
  types.ts         — API 响应类型
src/app/api/       — API 路由
  write/           — 创建文章/日记/说说
  upload/          — 上传图片
  posts|diary|notes|photos|friends/ — CRUD
  admin/           — 管理后台 API
src/middleware.ts  — /admin 路由保护
content/           — 内容存储
  posts/           — 博客文章 (.mdx)
  diary/           — 日记 (.mdx)
  notes/           — 说说 (.mdx)
  friends/links.json — 友链数据
  site/config.json — 站点外观配置
public/photos/     — 照片存储
```

## GSAP 动画规范

全局动画工具: `src/lib/animations.ts`
- 所有组件使用 `useGSAP(() => {...}, { scope: ref })` — 自动清理
- 入场动画: `gsap.from(target, { y, autoAlpha, stagger, ease })`
- 滚动入场: `ScrollTrigger.batch(selector, { onEnter })`
- 弹窗: `gsap.from(dialog, { scale: 0.9, autoAlpha: 0, ease: "back.out" })`
- 关闭动画: `gsap.to(dialog, { scale: 0.95, autoAlpha: 0, duration: 0.25, onComplete: onClose })`
- 不要同时使用 CSS `animate-in` 类和 GSAP — 选其一，优先 GSAP
- 尊重 `prefers-reduced-motion`

## 管理后台

- 入口: `/admin` → Middleware 校验 JWT Cookie → 无 token 则 redirect `/admin/login`
- 默认密码: `admin123` (开发环境，通过 `ADMIN_PASSWORD` 环境变量配置)
- 开发环境自动放行 API 鉴权
- 导航栏检测 `admin_logged_in` Cookie 显示管理入口

## 已知坑点

1. **`server-only` 包**: 使用 `fs`/`path` 等服务端模块的 lib 文件必须 `import "server-only"` 防止被客户端组件引用
2. **类型冲突**: 两个文件定义同名 interface 时，即使字段完全相同 TypeScript 也认为是不同类型。将共享类型抽取到一个文件
3. **`jose` 的 `jwtVerify`**: 需要 `TextEncoder` 编码 secret，Edge Runtime 兼容
4. **Tailwind v4 `@theme inline`**: 变量映射格式为 `--color-bg: var(--bg)`，不是 `--bg: ...`
5. **GSAP + React**: 必须用 `useGSAP` 或 `gsap.context()` + cleanup，否则内存泄漏
6. **MDX 组件注入**: `compileMDX` 时 `components` 参数必须传入 `mdxComponents`（不能是空对象），否则自定义组件不生效
7. **暗色模式 CSS 变量**: 用 `.dark` 选择器（与 ThemeProvider class 一致），不要用 `@media (prefers-color-scheme: dark)`（与手动切换冲突）
8. **草稿过滤**: `getPostBySlug()` 和 `getAllPostSlugs()` 必须检查 `draft: true`，否则草稿可通过 URL 直接访问
9. **Python 脚本路径**: 在 Python 脚本中必须用 `C:\\Users\\Loze\\...` 格式，不能用 `/c/Users/Loze/...`
10. **视频背景上传→生效链路**: 上传到 `/api/upload` → 设置面板更新 `config.background.video` → 保存时 PUT `/api/admin/config` + dispatch `CustomEvent("loze-config-updated")` → `VideoBackground` 客户端组件监听事件实时渲染。同理适用于图片背景和头像（`CustomEvent("loze-profile-updated")`）
11. **管理员密码变更**: 当前是账号 `Loze` / 密码 `20060708`。如果用户说改了，以用户说的为准。API 默认值在 `api/admin/login/route.ts` 和 `api/admin/password/route.ts` 两处都要同步更新
12. **端口默认行为**: `npm run dev` 不加 `-p` 参数时默认启动在 3000 端口（Next.js 默认）。要使用 6708 端口必须显式传 `-- -p 6708`
14. **GSAP barrel export 不完整** — `src/lib/animations.ts` 作为集中导出入口（barrel file），组件从 `@/lib/animations` 导入时，该文件必须显式 `export { xxx } from 'gsap'` 每一个被外部依赖的符号。遗漏会导致页面 500：`does not provide an export named 'scrollBatch'` / `'gsap'` / `'ScrollTrigger'`。排查方法：全局搜索 `from '@/lib/animations'`，核对所有导入项是否都在 `animations.ts` 中有对应的 `export` 语句。常见遗漏：`gsap` 本身、`ScrollTrigger`、`useGSAP`、`scrollBatch`。
15. **Vue 博客双技术栈同步规则** — loze-blog-vue (:5173) 和 loze-blog-next (:3000) 是同一博客，所有参数必须与 Next.js 严格一致。① Glass 透明度：`.glass`=0.65, `.glass-panel`=0.7, `.nav-glass`=0.72，不要擅自提高。② 动画参数：`y:30`, `duration:0.6`, `ease:power2.out`, 使用 `autoAlpha` 而非 `opacity`（与Next.js的`gsap.defaults`一致）。③ 滚动触发：`start:'top 85%'`。④ 页面过渡：`duration:0.6, ease:power2.out`。
16. **Vue GSAP 指令关键坑** — ① 数据异步加载导致子元素初渲染为空：指令必须同时实现 `mounted`+`updated` 双生命周期，`updated` 中用 `__gsapChildCount` 记录子元素数，只在数量变化且>0时重设动画。② esbuild 不兼容 `(binding.value as any)?.delay` 这种 inline type assertion + optional chaining 组合，必须拆成 `const val: any = binding.value` 再 `val.delay`。③ 页面切换后必须 `ScrollTrigger.refresh()`（在 App.vue `onEnter` 的 `onComplete` 中调用）。④ `unmounted` 中 `tween.kill()` 防止内存泄漏。
17. **内容同步** — 两个博客内容必须一致。同步流程：从 Next.js `content/` 读 MDX → 解析 frontmatter → 写入 Vue 的 MySQL `loze_blog` 数据库（表: t_post/t_diary/t_note/t_friend）。注意后端字段映射：slug/title/content/tags/category/description/date，diary 多加 mood/weather。
18. **Shiki 语法高亮冷启动** — 首次加载含大量代码块的文章时 Shiki `createHighlighter` 编译约 10s，后续命中缓存。这是正常行为，不是 bug。
19. **Next.js 自定义 404 页缺失** — Next.js App Router 默认 404 页面显示英文 "This page could not be found"。要显示中文 404，必须在 `src/app/` 下创建 `not-found.tsx`（文件命名必须是 `not-found.tsx`，不是 `404.tsx` 或 `NotFound.tsx`，这是 Next.js 的文件约定）。页面使用 Tailwind class + CSS 变量（如 `var(--text)`、`var(--text-muted)`）保持风格一致，提供"返回首页"链接。
20. **Vue Photos.vue 绕过 API 拦截器直接读 data.json** — `Photos.vue` 在 `onMounted` 中直接 `fetch('/data.json')` 而非通过 axios API 拦截器。因此 `data.json` 中**必须**包含 `photos` 数组，否则相册永远显示"还没有照片"。后端上传新照片后，`data.json` 的 `photos` 数组也需要同步更新。`public/photos/` 目录下有文件但 data.json 缺少 `photos` 键时，页面仍然显示为空。
21. **Vue 所有页面动画必须与 BlogList 一致** — 每个页面必须同时具备：① `v-fade-in` 在标题区（加 `{ delay: 0.1 }` 给次级元素）；② `v-scroll-reveal="{ y: 36, stagger: 0.08 }"` 在列表/网格容器上；③ `onMounted` 数据加载后 `await nextTick(); ScrollTrigger.refresh()`。任何页面缺少 `v-scroll-reveal` 或 `ScrollTrigger.refresh()` 就是动画缺失。详见 `references/vue-blog-patterns.md` 的"页面动画一致性检查清单"。
22. **Vue 所有卡片必须用 glass glass-hover** — PostCard 使用的标准卡片样式是 `glass glass-hover p-5`。任何页面使用 `bg-card border border-border` 替代 `glass` 会破坏毛玻璃效果一致性。Friends.vue、Notes.vue 等列表页的卡片元素必须统一为 `glass glass-hover`。注意：**卡片 class 中不要加 `transition`**（Tailwind 的 `transition` 类会与 GSAP 的 opacity/transform 动画产生竞态，导致动画卡住或闪烁）。详见 `references/vue-blog-patterns.md` 的"玻璃透明度使用规范"。

23. **嵌套 v-fade-in 导致动画失效** — 当父元素和子元素同时使用 `v-fade-in` 指令时，子元素的 `gsap.from({ autoAlpha: 0 })` 会读取到父元素正在动画中的 `opacity: 0`，导致动画从 0→0 无可见效果。**禁止在任何 v-fade-in 元素的直接子元素上再加 v-fade-in**。需要交错淡入时，只在最内层元素上分别设置不同 `delay`（如 `v-fade-in="{ delay: 0.1 }"`），不要在外层包裹 v-fade-in。

24. **data.json 缺少页面需要的 key 会导致 Vue 崩溃 + GSAP 卡死** — 前端后端下线时，axios 响应拦截器 fallback 到 `public/data.json`。如果 data.json 缺少某个页面需要的顶层 key（如 `friends`、`photos`），fallback 返回 `{ success: true, data: undefined }`，模板中 `v-if="list.length"` 会抛出 `TypeError: Cannot read property 'length' of undefined`，导致整个 Vue 页面崩溃，GSAP 动画全部卡在半途。**data.json 必须包含所有页面的 fallback 数据**：`posts`、`entries`、`notes`、`photos`、`friends` 五个顶层 key，每个至少是空数组 `[]`。

25. **terminal 工具无法在 Windows 上启动 npm 前端** — 此 Windows 宿主上 `terminal` 经 bash/MSYS 执行命令时会触发 WSL 拦截，输出乱码且进程立即退出。启动 Vue/Next.js 前端 dev server 必须用 `execute_code` + Python subprocess 或告知用户手动运行 `.bat` 脚本。详见 `references/windows-server-startup.md`。

15. **新文章封面图必须生成 SVG** — MDX frontmatter 中 `cover: "/images/xxx-cover.svg"` 引用的文件必须存在于 `public/images/`。批量创建文章后**必须检查并生成缺失的封面**，否则页面显示裂图。封面 SVG 模板：1200×630 尺寸，渐变色背景 + 主题图标/缩写。生成脚本见 `references/cover-svg-template.md`。

16. **丰富数据时的内容质量标准** — 给博客批量创建文章/日记/说说时：
    - 文章至少 500 字，有实际技术内容（代码示例、个人观点、实践建议）
    - 日记带 mood/weather 字段，有生活细节感
    - 说说 80-200 字，像朋友圈/微博风格
    - Frontmatter 必填：`title`, `date`, `description`, `tags`, `category`

## 双博客同步（Next.js ↔ Vue）

loze-blog-next (:3000) 和 loze-blog-vue (:5173) 是**同一个博客**，只是技术栈不同。内容、设计、功能必须保持一致。修改任一个时需同步另一个。

### 内容同步流程

Next.js MDX → Vue MySQL 的全量同步：

1. 解析 Next.js `content/posts/*.mdx`、`content/diary/*.mdx`、`content/notes/*.mdx` 的 YAML frontmatter + body
2. 清空 Vue 数据库 `t_post`, `t_diary`, `t_note`
3. 通过 `pymysql` 逐条 INSERT（注意 `content` 字段截断 65000 字符）
4. 确保 Vue 后端 (:8088) 在运行 → 重启以刷新缓存

详细脚本见 `references/cross-framework-sync.md`。

### Vue 博客 Glass 卡片规范

**必须与 Next.js 完全一致**。`frontend/src/assets/styles/main.css` 中的规格：

| 类 | 背景 | blur | 用途 |
|------|------|------|------|
| `.glass` | `rgba(255,255,255,0.82)` | `blur(16px)` | 内容卡片 |
| `.glass-panel` | `rgba(255,255,255,0.85)` | `blur(20px)` | 弹窗/面板 |
| `.nav-glass` | `rgba(255,255,255,0.85)` | `blur(20px)` | 导航栏 |
| `.dark .glass` | `rgba(30,30,32,0.82)` | `blur(16px)` | 暗色卡片 |
| `.dark .glass-panel` | `rgba(30,30,32,0.85)` | `blur(20px)` | 暗色面板 |

**不要擅自调高透明度** — 65% 不透明度配合 `backdrop-filter: blur()` 即可保证可读性，用户要求与 Next.js 一致。

### Vue GSAP 动画关键修复

**1. 路由切换后刷新 ScrollTrigger** — `App.vue` 页面过渡的 `onEnter.onComplete` 中必须调用 `ScrollTrigger.refresh()`：
```typescript
import { ScrollTrigger } from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollTrigger)

function onEnter(el: Element, done: () => void) {
  gsap.to(el, { opacity: 1, y: 0, duration: 0.5, ease: 'power3.out',
    onComplete: () => { ScrollTrigger.refresh(); done() }
  })
}
```
不用动态 `import()` — 直接顶部静态导入。

**2. 指令 unmount 清理** — `v-scroll-reveal` 和 `v-fade-in` 必须存 tween 引用并在 `unmounted` 中 `kill()`：
```typescript
mounted(el, binding) {
  (el as any).__gsapTween = gsap.from(el, { ... })
},
unmounted(el) {
  (el as any).__gsapTween?.kill()
}
```
否则组件销毁后 ScrollTrigger 仍持有 DOM 引用导致内存泄漏。

**3. esbuild 兼容性** — 不支持 `(binding.value as any)?.delay`（type assertion + optional chaining 组合）。拆成两行：
```typescript
const val: any = binding.value || {}
const delay = val.delay || 0
```

### Vue 后端须知

- 路径: `~/projects/loze-blog-vue/backend/`, 端口 8088
- 数据库: MySQL `loze_blog`, root 无密码
- 前端通过 axios 调 :8088 API，后端不在时 fallback 到 `public/data.json` — **所以直接改数据库后前端看不到，必须启动后端**
- 启动: `mvn spring-boot:run`（用 JDK 12）

## 快速命令

```bash
cd ~/projects/loze-blog-next
npm run dev -- -p 6708    # 启动开发服务器（指定端口6708）
npm run dev                # 默认端口 3000（不传 -p 时）
npm run build              # 生产构建
```

### terminal 不可用时的启动方案

当 `terminal` 工具返回乱码/失败时（如遇到 WSL 层干扰），用 `execute_code` 启动：

```python
import subprocess
# 创建启动批处理文件
with open('C:\\Users\\Loze\\start-blog.bat', 'w') as f:
    f.write("@echo off\ncd /d C:\\Users\\Loze\\projects\\loze-blog-next\nnpm run dev -- -p 6708 >> C:\\Users\\Loze\\blog-server.log 2>&1\n")
# 用 CREATE_NEW_CONSOLE 在新窗口启动（不受 sandbox 生命周期影响）
subprocess.Popen('C:\\Users\\Loze\\start-blog.bat', creationflags=subprocess.CREATE_NEW_CONSOLE, shell=True)
```

然后用 Python socket 检测端口是否就绪：
```python
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.settimeout(2)
r = sock.connect_ex(('localhost', 6708))
sock.close()
print('OPEN' if r == 0 else 'CLOSED')
```
