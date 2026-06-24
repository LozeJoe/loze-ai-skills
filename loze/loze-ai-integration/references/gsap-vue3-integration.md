# GSAP Vue 3 集成模式

> 2026-06-20 | 用于 loze-blog-vue 等 Vue 3 项目

## 架构

```
src/plugins/gsap.ts          ← GSAP 插件（注册 ScrollTrigger + 全局指令）
src/main.ts                   ← app.use(gsapPlugin)
组件中使用 v-scroll-reveal / v-fade-in 指令
```

## 插件代码 (`src/plugins/gsap.ts`)

```ts
import { App, DirectiveBinding } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

// v-scroll-reveal: 滚动揭示
const scrollReveal = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const { y=32, duration=0.7, delay=0, stagger=0, ease='power3.out', start='top 88%', once=true } = binding.value || {}
    
    if (stagger > 0 && el.children.length > 0) {
      gsap.from(el.children, { y, opacity: 0, duration, stagger, ease, delay,
        scrollTrigger: { trigger: el, start, once } })
      return
    }
    gsap.from(el, { y, opacity: 0, duration, delay, ease,
      scrollTrigger: { trigger: el, start, once } })
  },
}

// v-fade-in: 简单淡入
const fadeIn = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const delay = binding.value?.delay || 0
    gsap.from(el, { opacity: 0, y: 16, duration: 0.6, delay, ease: 'power2.out' })
  },
}

export default {
  install(app: App) {
    app.directive('scroll-reveal', scrollReveal)
    app.directive('fade-in', fadeIn)
  },
}
```

## 组件中使用

```html
<!-- 滚动揭示 -->
<section v-scroll-reveal="{ y: 40 }">
  <h2>标题</h2>
  <p>内容...</p>
</section>

<!-- 子元素交错进入 -->
<div v-scroll-reveal="{ stagger: 0.1 }">
  <PostCard v-for="post in posts" ... />
</div>

<!-- 简单淡入 -->
<div v-fade-in="{ delay: 0.2 }">
  ...
</div>
```

## GSAP 页面过渡（App.vue）

```html
<router-view v-slot="{ Component }">
  <transition @before-enter="onBeforeEnter" @enter="onEnter" @leave="onLeave">
    <component :is="Component" />
  </transition>
</router-view>
```

```ts
function onBeforeEnter(el: Element) {
  gsap.set(el as HTMLElement, { opacity: 0, y: 24 })
}
function onEnter(el: Element, done: () => void) {
  gsap.to(el as HTMLElement, { opacity: 1, y: 0, duration: 0.5, ease: 'power3.out', onComplete: done })
}
function onLeave(el: Element, done: () => void) {
  gsap.to(el as HTMLElement, { opacity: 0, y: -12, duration: 0.25, ease: 'power2.in', onComplete: done })
}
```

## 阅读进度条

```ts
const progress = ref(0)
function onScroll() {
  const val = (window.scrollY / (document.documentElement.scrollHeight - window.innerHeight)) * 100
  gsap.to('.reading-progress-bar', { width: val + '%', duration: 0.15, ease: 'power1.out', overwrite: 'auto' })
}
```

## 关键依赖

```json
"gsap": "^3.12.5"
```

无需 `@gsap/react`（那是 React 用的），Vue 直接用 gsap core + ScrollTrigger。
