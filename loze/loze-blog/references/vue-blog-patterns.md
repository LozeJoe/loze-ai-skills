# Vue 博客模式参考

## 双项目同步规则

loze-blog-next (:3000, Next.js) 和 loze-blog-vue (:5173, Vue 3 + Spring Boot :8088) 是**同一个博客**，只是技术栈不同。以下必须保持一致：

- **内容**：文章/日记/说说数量、标题、正文
- **设计**：CSS 变量、玻璃透明度、配色方案
- **动画**：入场/页面过渡参数

## GSAP 指令 (v-scroll-reveal / v-fade-in)

### 核心坑：异步数据 + 子元素

`v-scroll-reveal` 配合 `v-for` 渲染异步数据时，`mounted` 时子元素为空。必须加 `updated` 生命周期，且只在子元素数量变化时重设动画：

```typescript
const scrollReveal = {
  mounted(el, binding) { setupScrollReveal(el, binding) },
  updated(el, binding) {
    const oldCount = el.__gsapChildCount || 0
    const newCount = el.children.length
    if (newCount !== oldCount && newCount > 0) {
      setupScrollReveal(el, binding)  // 杀掉旧 tween 重建
    }
  },
  unmounted(el) { el.__gsapTween?.kill() },
}
```

### 路由切换后刷新

Vue Router 替换 DOM 后 ScrollTrigger 不知道。在每个页面数据加载完成后调：

```typescript
import { ScrollTrigger } from 'gsap/ScrollTrigger'
await nextTick()
ScrollTrigger.refresh()
```

App.vue 页面过渡的 `onComplete` 里也要调。

### esbuild 兼容性

esbuild 不支持 `(binding.value as any)?.delay`（type assertion + optional chaining）。必须拆开：

```typescript
// ❌ 会报 Invalid assignment target
const delay = (binding.value as any)?.delay || 0

// ✅ 拆成两行
const val: any = binding.value || {}
const delay = val.delay || 0
```

### 默认参数对齐 Next.js

```typescript
gsap.defaults({ duration: 0.6, ease: 'power2.out' })
// v-scroll-reveal 默认: y: 30, start: 'top 85%', ease: 'power3.out'
// v-fade-in 默认: y: 16, duration: 0.6, ease: 'power2.out'
```

## API 响应格式

Spring Boot 后端返回 `{ success: true, data: { posts: [...], total: N } }`。

**坑**：data 是嵌套对象，不是数组。组件取值时必须：

```typescript
// ❌ 当成数组直接用
posts.value = res.data

// ✅ 取嵌套的 posts 数组
posts.value = Array.isArray(res.data) ? res.data : (res.data.posts || [])
```

## 玻璃 CSS 规格（与 Next.js 一致）

| 类 | 背景 | blur |
|------|------|------|
| `.glass` | `rgba(255,255,255,0.65)` | `blur(16px)` |
| `.glass-panel` | `rgba(255,255,255,0.7)` | `blur(20px)` |
| `.nav-glass` | `rgba(255,255,255,0.72)` | `blur(20px)` |
| `.dark .glass` | `rgba(30,30,32,0.65)` | `blur(16px)` |
| `.dark .glass-panel` | `rgba(30,30,32,0.7)` | `blur(20px)` |

## 启动命令

```bash
# 前端 (Vite dev)
cd ~/projects/loze-blog-vue/frontend && npm run dev

# 后端 (Spring Boot, JDK 12)
cd ~/projects/loze-blog-vue/backend
D:/JAVAEE/apache-maven-3.5.2/apache-maven-3.5.2/bin/mvn.cmd spring-boot:run
# 端口: 8088, 数据库: loze_blog (MySQL root 无密码)
```

## 页面动画一致性检查清单

**所有 Vue 页面必须与 BlogList.vue 保持一致的动画模式：**

| 元素 | 动画 | 参数 |
|------|------|------|
| 页面标题区（h1 + 描述） | `v-fade-in` | 默认（y:16, delay:0） |
| 次级元素（标签云、按钮组） | `v-fade-in` | `{ delay: 0.1 }` |
| 列表/网格容器（v-for 的父级） | `v-scroll-reveal` | `{ y: 36, stagger: 0.08 }` |
| 底部元素（分页等） | `v-scroll-reveal` | `{ y: 16 }` |

**每个页面的 `<script setup>` 必须：**

```typescript
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { nextTick } from 'vue'

onMounted(async () => {
  // ... 加载数据 ...
  await nextTick()
  ScrollTrigger.refresh()
})
```

**检查要点**：任何页面只有 `v-fade-in` 没有 `v-scroll-reveal` 就是缺失了滚动入场动画。任何页面加载数据后不调 `ScrollTrigger.refresh()` 会导致滚动动画不触发。

### ⚠ 嵌套 v-fade-in 禁止

`v-fade-in` 指示詞を親要素と子要素の両方に適用すると、子要素の `gsap.from({ autoAlpha: 0 })` が親のアニメーション中の `opacity: 0` を検出し、0→0 の不可視アニメーションになる。

```html
<!-- ❌ 嵌套 v-fade-in → 子要素のアニメが効かない -->
<div v-fade-in>
  <div v-fade-in>  <!-- 親 opacity:0 の影響で 0→0 -->
    <h1 v-fade-in="{ delay: 0.1 }">タイトル</h1>  <!-- 同上 -->
  </div>
</div>

<!-- ✅ 外層は v-fade-in なし、内層のみ遅延付き v-fade-in -->
<div>
  <div>
    <div v-fade-in>アバター</div>
    <h1 v-fade-in="{ delay: 0.1 }">タイトル</h1>
    <p v-fade-in="{ delay: 0.2 }">説明文</p>
  </div>
</div>
```

### ⚠ data.json の完全性（backend ダウン時のクラッシュ防止）

バックエンド (:8088) がダウンしている時、axios インターセプターは `public/data.json` をフォールバックデータとして使用する。このファイルに**どのページからもアクセスされる可能性のあるキーが欠けている**と、テンプレート内の `v-if="list.length"` が `TypeError: Cannot read property 'length' of undefined` をスローし、Vue ページ全体がクラッシュ、GSAP アニメーションが停止する。

**data.json に必須のトップレベルキー：**
- `posts` — BlogList, BlogDetail, Home
- `entries` — Diary
- `notes` — Notes
- `photos` — Photos（`fetch('/data.json')` で直接アクセス）
- `friends` — Friends

各キーは最低限空配列 `[]` にすること。

## 玻璃透明度 (glass) 使用规范

### 所有卡片必须用 `glass glass-hover`

`PostCard` 组件使用 `glass glass-hover p-5` 作为标准卡片样式。其他页面的卡片必须保持一致：

```html
<!-- ✅ 正确 — 使用 glass + glass-hover（不要加 transition！） -->
<div class="glass glass-hover p-5 rounded-xl">

<!-- ❌ 错误 — 使用 bg-card + border-border，玻璃效果缺失 -->
<div class="bg-card border border-border p-5 rounded-xl">

<!-- ❌ 错误 — glass-hover + transition 会与 GSAP 动画冲突 -->
<div class="glass glass-hover p-5 rounded-xl transition">
```

**⚠ `transition` 类と GSAP の競合**：Tailwind の `transition` クラスは `transition-property: color, background-color, border-color, ...` を設定し、GSAP が同じ要素の opacity/transform をアニメーションする際に競合してカクつきやアニメーション停止を引き起こす。GSAP アニメーション対象の要素には `transition` クラスを付けないこと。

**影响范围**：Friends.vue 的友链卡片、Notes.vue 的说说卡片、以及任何 v-for 渲染的列表项卡片。统一使用 `glass glass-hover` 确保所有页面的卡片透明度一致。

`Photos.vue` **不走 axios API 拦截器**，直接 `fetch('/data.json')` 读取 `photos` 数组。这意味着：

- `public/data.json` 必须包含 `"photos": [...]` 数组，否则相册永远为空
- 后端上传新照片后，不会自动出现在前端相册（因为 Photos.vue 不调后端 API）
- `public/photos/` 下有文件 ≠ 相册有内容

**解决方案 A**（当前采用）：后端不在时 data.json 作为静态数据源，维护 `photos` 数组与 `public/photos/` 目录同步。

**解决方案 B**（如需后端支持）：改造 Photos.vue 通过 axios 调 `/api/photos`，并在 `api/index.ts` 拦截器中添加 `/photos` 的 fallback 处理。
