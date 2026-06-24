# 视频/图片背景实时生效模式

> 适用于: 管理后台修改外观后无需刷新页面即可看到效果的场景

## 问题

Next.js 服务端组件在首次渲染后不会更新。管理后台保存配置到文件后，页面不刷新就无法看到新的背景。

## 解决方案: CustomEvent 桥接

```
┌─────────────────────┐     CustomEvent      ┌─────────────────────┐
│  settings-appearance │ ──── "loze-config- ──→ │  VideoBackground    │
│  (保存配置)           │      updated"        │  (客户端组件)         │
│                      │                       │  setState → re-render│
│  applyConfigToDOM()  │ ←── dispatchEvent ── │                       │
│  + localStorage      │                       │                       │
└─────────────────────┘                       └─────────────────────┘
```

## 实现步骤

### 1. 在保存端 dispatch 事件

```typescript
// settings-appearance.tsx
window.dispatchEvent(new CustomEvent("loze-config-updated", { detail: config }));
localStorage.setItem("loze_site_config", JSON.stringify(config));
```

### 2. 在渲染端监听事件

```typescript
// video-background.tsx (client component)
useEffect(() => {
  // 初始加载从 API 读取
  fetch("/api/admin/config").then(r => r.json()).then(setBg);
  
  // 监听保存事件
  const handler = (e: CustomEvent) => setBg(e.detail.background);
  window.addEventListener("loze-config-updated", handler);
  return () => window.removeEventListener("loze-config-updated", handler);
}, []);
```

### 3. localStorage 缓存

首次加载时从 localStorage 读取，避免 API 调用延迟导致的闪烁：

```typescript
const cached = localStorage.getItem("loze_site_config");
if (cached) setBg(JSON.parse(cached).background);
```

## 同理: 头像实时更新

```typescript
// profile-editor.tsx (保存端)
window.dispatchEvent(new CustomEvent("loze-profile-updated", { detail: payload }));

// about/page.tsx (渲染端)
useEffect(() => {
  fetch("/api/admin/profile").then(r => r.json()).then(setProfile);
  window.addEventListener("loze-profile-updated", handler);
}, []);
```

## 上传流程

```
用户选择文件 → /api/upload → 返回 {url}
  → setConfig({ background: { type, video/image: url } })
  → 用户点击「保存」
  → PUT /api/admin/config → 写入 config.json
  → applyConfigToDOM() → CSS变量 + dispatch事件
  → VideoBackground 收到事件 → setState → 实时渲染
```
