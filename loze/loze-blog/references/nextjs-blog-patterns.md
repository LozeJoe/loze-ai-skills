# Next.js App Router 博客架构模式

> 从 loze-blog-next 项目中提炼的可复用模式
> 适用: Next.js 16 + App Router + TypeScript + Tailwind v4

## 1. 服务端/客户端组件分离

**核心原则**: `fs`/`path`/`process.cwd()` 只在服务端可用。

### 标准模式
```
src/app/blog/page.tsx          ← Server Component (async, 读取数据)
src/components/blog-client-page.tsx  ← Client Component ("use client", 交互+动画)
```

```typescript
// blog/page.tsx (Server)
export default async function BlogPage({ searchParams }) {
  const posts = getAllPosts();  // 服务端 fs 操作
  const tags = getAllTags();
  return <BlogClientPage initialPosts={posts} initialTags={tags} />;
}

// blog-client-page.tsx (Client)
"use client";
export default function BlogClientPage({ initialPosts, initialTags }) {
  const [posts, setPosts] = useState(initialPosts);
  // ... 交互逻辑、GSAP 动画、fetch API
}
```

## 2. API Route 文件系统 CRUD

```typescript
// src/app/api/posts/route.ts
import { NextRequest, NextResponse } from "next/server";
import * as fs from "fs";
import * as path from "path";

export async function GET() { /* 列出 */ }
export async function POST(request: NextRequest) { /* 创建 */ }
export async function DELETE(request: NextRequest) {
  const slug = request.nextUrl.searchParams.get("slug");
  fs.unlinkSync(path.join(postsDir, `${slug}.mdx`));
  return NextResponse.json({ success: true });
}
```

## 3. JWT 认证中间件

```typescript
// src/middleware.ts
import { jwtVerify } from "jose";

export async function middleware(request: NextRequest) {
  if (!pathname.startsWith("/admin")) return NextResponse.next();
  if (pathname.startsWith("/admin/login")) return NextResponse.next();
  
  const token = request.cookies.get("admin_token")?.value;
  if (!token) return NextResponse.redirect(new URL("/admin/login", request.url));
  
  try {
    await jwtVerify(token, new TextEncoder().encode(secret));
    return NextResponse.next();
  } catch {
    return NextResponse.redirect(new URL("/admin/login", request.url));
  }
}

export const config = { matcher: ["/admin/:path*"] };
```

## 4. server-only 类型共享

当客户端组件需要服务端定义的类型/常量时，抽取共享文件：

```
src/lib/site-config-types.ts   ← 纯类型+常量 (无 fs 操作, 客户端安全)
src/lib/site-config.ts         ← server-only (import fs, 服务端专属)
```

```typescript
// site-config-types.ts
export interface SiteConfig { ... }
export const PRESET_THEMES = [...];
export const DEFAULT_CONFIG = { ... };

// site-config.ts
import "server-only";
import { DEFAULT_CONFIG } from "./site-config-types";
export function getSiteConfig(): SiteConfig { /* fs 操作 */ }
```

## 5. GSAP React 集成

```typescript
// src/lib/animations.ts
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { useGSAP } from "@gsap/react";
gsap.registerPlugin(ScrollTrigger, useGSAP);

// 组件中
"use client";
const containerRef = useRef(null);
useGSAP(() => {
  gsap.from(".item", { y: 30, autoAlpha: 0, stagger: 0.08, ease: "power3.out" });
}, { scope: containerRef });
```

**关键规则**:
- 所有 GSAP 代码必须在 `useGSAP` 或 `gsap.context()` 内
- 必须用 `scope` ref 限制选择器范围
- `ScrollTrigger` 放在 timeline/top-level tween 上，不放在子 tween 里
- 弹窗关闭: `gsap.to(el, { scale: 0.95, autoAlpha: 0, onComplete: callback })`

## 6. MDX 自定义组件注入

```typescript
// blog/[slug]/page.tsx
import mdxComponents from "@/mdx-components"; // ← 关键：必须导入

const { content } = await compileMDXSource({
  source: post.content,
  components: mdxComponents,  // ← 不能是空对象
});
```

`mdx-components.tsx` 注册全局组件映射: `Callout`, `CodeBlock`, 自定义 `p`/`img`/`a`/`pre` 等。

## 7. 暗色模式 CSS 变量

```css
/* globals.css */
:root {
  --bg: #faf8f5;
  --text: #4a3f35;
  --accent: #5b7b6a;
  /* ... */
}
.dark {
  --bg: #2c2416;
  --text: #e8dcc8;
  /* ... */
}
/* Tailwind v4 映射 */
@theme inline {
  --color-bg: var(--bg);
  --color-text: var(--text);
  /* ... */
}
```

**不要用** `@media (prefers-color-scheme: dark)` — 与 ThemeProvider 的 `class="dark"` 切换不一致。
