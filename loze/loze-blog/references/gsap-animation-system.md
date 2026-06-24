# GSAP 动画系统 — Loze Blog

> 2026-06-04 添加 · 参考 gsap-skills 官方技能库

## 安装

```bash
npm install gsap @gsap/react
```

## 架构

```
src/lib/animations.ts        ← 共享动画系统
  ├── 注册: gsap.registerPlugin(ScrollTrigger, useGSAP)
  ├── 全局 gsap.defaults({ duration: 0.6, ease: "power2.out" })
  ├── 导出: fadeInUp, slideInLeft, titleReveal, scaleIn
  ├── 导出: scrollFadeInUp, scrollBatch (ScrollTrigger)
  └── 导出: useGSAP, gsap, ScrollTrigger
```

## 核心模式: 服务端/客户端组件拆分

动画必须在客户端执行。模式：

```typescript
// page.tsx (服务端) — 只获取数据
export default async function Page() {
  const data = await fetchData(); // fs, MDX 解析
  return <ClientPage initialData={data} />; // 传给客户端
}

// client-page.tsx (客户端) — 交互 + 动画
"use client";
import { useGSAP, gsap } from "@/lib/animations";

export default function ClientPage({ initialData }) {
  const containerRef = useRef(null);
  const [data, setData] = useState(initialData);

  useGSAP(() => {
    // 入场动画
    gsap.from(".item", { y: 30, autoAlpha: 0, stagger: 0.1, ease: "power3.out" });
    
    // 滚动触发
    ScrollTrigger.batch(".card", {
      onEnter: (els) => gsap.from(els, { y: 30, autoAlpha: 0, stagger: 0.08 }),
      start: "top 90%",
      once: true,
    });
  }, { scope: containerRef, dependencies: [data.length] });

  return <div ref={containerRef}>...</div>;
}
```

## 已实现的动画清单

| 组件 | 动画 | GSAP 方法 |
|------|------|-----------|
| Header | 导航栏从上方滑入 + 链接 stagger | `gsap.from` (y: -20, autoAlpha: 0) |
| Footer | ScrollTrigger 底部淡入 | `gsap.from` + `scrollTrigger` |
| 首页 Hero | 标题弹入/副标题/按钮缩放 | `gsap.from` (y: 60, ease: power3.out) |
| 首页精选 | 磁贴 scroll stagger | `ScrollTrigger.batch` |
| 首页时间线 | 条目 scroll 淡入 | `ScrollTrigger.batch` |
| 博客列表 | 卡片 grid scroll 入场 | `scrollBatch` |
| 文章详情 | 标题淡入 + 段落逐段渐现 + TOC滑入 | `gsap.from` + `ScrollTrigger.batch` |
| 日记 | 条目左滑入 + 圆点缩放 | `slideInLeft` + `back.out` |
| 相册 | 照片 stagger | `fadeInUp` |
| 说说 | 卡片 stagger | `fadeInUp` |
| 友链 | 卡片 stagger | `fadeInUp` |
| PostCard | hover 浮起 (GSAP 替代 framer-motion) | `gsap.to` + mouseenter/leave |
| 弹窗 | 缩放弹入 / 缩放关闭 | `scaleIn` / `gsap.to` (scale: 0.95) |
| QuickActions | 按钮缩放弹入 | `scaleIn` |

## 关键规则

1. **useGSAP 替代 useEffect** — 自动 cleanup，避免泄漏
2. **scope ref** — 限制选择器范围，防止匹配到组件外元素
3. **dependencies** — 当数据量变化时（写新文章后）重新运行动画
4. **prefers-reduced-motion** — `animations.ts` 内置 matchMedia 检查
5. **不要同时用 CSS animate-in + GSAP** — GSAP 替代 Tailwind `animate-in` 类

## 无障碍

```typescript
// animations.ts 内置
const reducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)");
// 检测到 reduced motion 时动画 duration 自动设为 0
```

## 参考

- 官方技能: `~/.hermes/skills/gsap/` (8 个 SKILL.md)
  - gsap-core, gsap-timeline, gsap-scrolltrigger, gsap-react
  - gsap-plugins, gsap-frameworks, gsap-performance, gsap-utils
