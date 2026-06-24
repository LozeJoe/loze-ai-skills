# GSAP 全站动画集成模式 — Loze Blog 实践

> 来源: loze-blog-next 项目 (Next.js 16 + GSAP + React)

## 架构

```
src/lib/animations.ts          ← 共享动画系统（唯一入口）
  ├── gsap + ScrollTrigger + useGSAP 注册（一次性）
  ├── gsap.defaults({ duration: 0.6, ease: "power2.out" })
  ├── 可复用函数: fadeInUp, slideInLeft, titleReveal, scaleIn
  ├── ScrollTrigger 辅助: scrollFadeInUp, scrollBatch
  └── 无障碍: prefers-reduced-motion 检测
```

## 组件中使用

```tsx
"use client";
import { useGSAP, gsap, fadeInUp, scrollBatch } from "@/lib/animations";

export default function MyComponent() {
  const containerRef = useRef(null);

  useGSAP(() => {
    // 入场动画
    fadeInUp(".card", 0.08);
    // 滚动动画
    scrollBatch(".post-card");
  }, { scope: containerRef });

  return <div ref={containerRef}>...</div>;
}
```

## 关键规则

1. **统一入口**: 所有组件从 `@/lib/animations` 导入，不从 `gsap`/`@gsap/react` 直接导入
2. **useGSAP hook**: 自动注册 ScrollTrigger，自动清理（替代 useEffect + gsap.context）
3. **scope ref**: 始终传 scope 限制选择器范围
4. **transform 优先**: 只动画 `y`/`x`/`scale`/`autoAlpha`，避免 `width`/`height`/`top`/`left`
5. **ScrollTrigger.batch**: 批量 scroll 入场比逐个创建 ScrollTrigger 高效
6. **暗色过渡**: `html { transition: background-color 0.3s, color 0.2s; }`

## 不要做的事

- ❌ 不要在同一个元素上同时用 GSAP hover 和 CSS `group-hover:scale-105`（叠加导致双倍缩放）
- ❌ 不要从 `@gsap/react` 直接导入（绕过全局 defaults 和 ScrollTrigger 注册）
- ❌ 不要对带 `!important` 的 Tailwind class（如 `hidden`）用 JS 改 style

## 安装

```bash
npm install gsap @gsap/react
```
