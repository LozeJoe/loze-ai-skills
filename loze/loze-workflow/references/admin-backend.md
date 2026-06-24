# Next.js 管理后台建设备忘 — Loze Blog 实践

## 认证系统

- 使用 `jose` 库签发 JWT，存入 httpOnly Cookie (`admin_token`)
- 伴生 Cookie (`admin_logged_in`) 非 httpOnly，供前端导航栏检测登录状态
- middleware.ts 拦截 `/admin/*`（除 login），校验 Cookie 中的 JWT
- 开发环境 (`NODE_ENV === "development"`) 自动放行鉴权

## 统一鉴权

```typescript
// src/lib/admin-auth.ts — 所有 /api/admin/* 共用
export async function checkAdminAuth(request: NextRequest): Promise<boolean> {
  if (process.env.NODE_ENV === "development") return true;
  const token = request.cookies.get("admin_token")?.value;
  if (!token) return false;
  return (await verifyToken(token)) !== null;
}
```

## 密码/账号

- 默认账号: `Loze`，密码: `20060708`
- 环境变量: `ADMIN_USERNAME` / `ADMIN_PASSWORD`
- 修改密码写入 `.env.local`（本地），Vercel 部署需手动更新环境变量

## 站点配置

- 配置文件: `content/site/config.json`
- 读写: `src/lib/site-config.ts`（`import "server-only"`）
- 类型: `src/lib/site-config-types.ts`（客户端/服务端共享）
- 预设主题: 暖棕/墨绿/雾蓝/樱粉/暗金（5套完整亮暗配色）

## 动态 CSS 注入（避免整页刷新）

```typescript
function applyConfigToDOM(config: SiteConfig) {
  const styleEl = document.getElementById("loze-dynamic-theme") || document.createElement("style");
  styleEl.id = "loze-dynamic-theme";
  styleEl.textContent = `${toCSS(light, ":root")} ${toCSS(dark, ".dark")}`;
  document.head.appendChild(styleEl);
}
```

## 视频背景

- BackgroundConfig.type 支持 `"video"`
- VideoBackground 组件：`<video autoPlay loop muted playsInline>` + 半透明遮罩
- 上传 API 已支持 `video/mp4` 和 `video/webm`

## 常见问题

- **404 问题**: 浏览器缓存了旧的 301 重定向 → 清除站点数据或无痕窗口
- **修改密码失败**: password API 和 login API 的默认密码必须一致
- **侧边栏 404**: 创建 `/admin/*` 子页面后确认 layout.tsx 中 href 正确（`/admin/settings` 不是 `/admin/appearance`）
- **服务端模块客户端引用**: `import "server-only"` 的文件不能从客户端组件导入 → 拆分为 types 文件
