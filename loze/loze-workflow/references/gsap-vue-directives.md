# GSAP Vue 3 指令模式

可复用的 Vue 3 GSAP 动画指令，任何 Vue 项目可直接复制使用。

## 安装

```bash
npm install gsap
```

## 插件代码 (`src/plugins/gsap.ts`)

```ts
import { App, DirectiveBinding } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

/** v-scroll-reveal — 滚动触发入场 */
const scrollReveal = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const { y = 32, duration = 0.7, delay = 0, stagger = 0, start = 'top 88%', once = true } = binding.value || {}
    
    if (stagger > 0 && el.children.length > 0) {
      gsap.from(el.children, { y, opacity: 0, duration, stagger, ease: 'power3.out', delay,
        scrollTrigger: { trigger: el, start, once } })
      return
    }
    gsap.from(el, { y, opacity: 0, duration, delay, ease: 'power3.out',
      scrollTrigger: { trigger: el, start, once } })
  },
}

/** v-fade-in — 页面加载淡入 */
const fadeIn = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const delay = (binding.value as any)?.delay || 0
    gsap.from(el, { opacity: 0, y: 16, duration: 0.6, delay, ease: 'power2.out' })
  },
}

export default { install(app: App) { app.directive('scroll-reveal', scrollReveal); app.directive('fade-in', fadeIn) } }
```

## 注册 (`main.ts`)

```ts
import gsapPlugin from './plugins/gsap'
app.use(gsapPlugin)
```

## 使用

```html
<!-- 简单淡入 -->
<div v-fade-in>内容</div>

<!-- 滚动揭示 -->
<div v-scroll-reveal>滚动到才显示</div>

<!-- 子元素交错 -->
<div v-scroll-reveal="{ y: 36, stagger: 0.08 }">
  <PostCard v-for="..." />
</div>

<!-- 延迟入场 -->
<div v-fade-in="{ delay: 0.3 }">先等一会</div>
```
