# GSAP 在 Next.js/Vue 项目中的集成模式

## Next.js 中转导出模块模式

Next.js 博客项目中 `src/lib/animations.ts` 不仅定义动画工具函数，还被多个组件当作 GSAP 的**中转导出模块**。组件不直接 `import from "gsap"`，而是 `import { useGSAP, gsap, ScrollTrigger } from "@/lib/animations"`。

### 规则

修改 `animations.ts` 时，**必须保留原始重新导出**：

```typescript
// 保留这些导出 — 其他组件依赖它们
export { default as gsap } from "gsap";
export { ScrollTrigger } from "gsap/ScrollTrigger";
export { useGSAP } from "@gsap/react";

// 你自己的增强函数
export function heroReveal(...) { }
export function cardHover(...) { }
```

**删除这些重新导出会导致全站 500**，因为 `home-animated.tsx`、`featured-posts.tsx`、`search-dialog.tsx` 等都从 `@/lib/animations` 导入 GSAP 原语。

### 报错信号

```
Export ScrollTrigger doesn't exist in target module
Export gsap doesn't exist in target module
Export useGSAP doesn't exist in target module
```

全部来自同一个原因：`animations.ts` 不再重新导出这些符号。

## Vue 3 GSAP 插件模式

Vue 3 中使用 GSAP 最佳实践：创建插件，用 `app.use()` 全局注册自定义指令。

```typescript
// src/plugins/gsap.ts
import { App, DirectiveBinding } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

export default {
  install(app: App) {
    app.directive('scroll-reveal', {
      mounted(el, binding) {
        const opts = binding.value || {}
        gsap.from(el.children.length ? el.children : el, {
          y: opts.y ?? 32, opacity: 0,
          duration: opts.duration ?? 0.7,
          stagger: opts.stagger ?? 0,
          ease: 'power3.out',
          scrollTrigger: { trigger: el, start: 'top 88%', once: true }
        })
      }
    })
    app.directive('fade-in', {
      mounted(el, binding) {
        gsap.from(el, { opacity: 0, y: 16, duration: 0.6, delay: binding.value?.delay ?? 0 })
      }
    })
  }
}
```

注册：`main.ts` 中 `app.use(gsapPlugin)`。

## Windows 项目启动

### 清理僵尸 Node 进程

多次启动/停止 dev server 会留下僵尸进程占用端口：
```python
subprocess.run(['taskkill', '/F', '/IM', 'node.exe'], capture_output=True, timeout=5)
```
注意：这会杀掉**所有** Node 进程，包括其他正在运行的项目。更精确的做法是只杀特定端口的进程。

### 清理 Next.js 缓存

构建缓存损坏也会导致 500：
```python
import shutil
shutil.rmtree(r'C:\Users\Loze\projects\loze-blog-next\.next', ignore_errors=True)
```

### 用完整路径启动 npm

`execute_code` 沙箱中 npm 不在 PATH，必须用完整路径：
```python
npm = r'C:\Program Files\nodejs\npm.cmd'
subprocess.Popen([npm, 'run', 'dev'], cwd=project_dir, creationflags=0x00000008)
```

`creationflags=0x00000008`（`DETACHED_PROCESS`）让进程脱离沙箱独立运行。

### Spring Boot / Maven 启动

Maven 路径是嵌套的，且必须设置 JAVA_HOME：
```python
mvn = r'D:\JAVAEE\apache-maven-3.5.2\apache-maven-3.5.2\bin\mvn.cmd'
jdk_home = r'C:\Program Files\Java\jdk-12.0.1'
env = os.environ.copy()
env['JAVA_HOME'] = jdk_home
subprocess.Popen([mvn, 'spring-boot:run'], cwd=demo_dir, env=env, creationflags=0x00000008)
```

`spring-boot:run` 启动可能需要 1-3 分钟（首次编译 + Spring 初始化），用轮询检测端口可用性。
