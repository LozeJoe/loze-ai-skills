# 刘哲凯 Loze Blog 设计系统

> 应用于 Next.js 博客的完整设计规范。后续博客相关修改必须遵循。
> 最后同步: 2026-06-21，来源 `src/app/globals.css`

## 配色 — 暖琥珀·法式烘焙

### 亮色模式
| 变量 | 色值 | 用途 |
|------|------|------|
| `--bg` | `#faf9f6` | 页面背景（奶油纸） |
| `--text` | `#1c1917` | 正文文字（深炭） |
| `--accent` | `#b7610a` | 链接、按钮、强调色（暖琥珀） |
| `--card` | `#ffffff` | 卡片/面板背景 |
| `--border` | `#e7e5e2` | 边框分隔线 |
| `--text-muted` | `#78716c` | 次要文字 |
| `--accent-hover` | `#9a4f08` | 悬停加深 |
| `--bg-secondary` | `#f5f3f0` | 次级背景 |
| `--nav-bg` | `rgba(250,249,246,0.78)` | 导航毛玻璃 |

### 暗色模式
| 变量 | 色值 |
|------|------|
| `--bg` | `#1c1917` |
| `--text` | `#e7e5e2` |
| `--accent` | `#f0a050` |
| `--card` | `#262320` |
| `--border` | `#3d3833` |
| `--text-muted` | `#a8a29e` |
| `--accent-hover` | `#f4b870` |
| `--bg-secondary` | `#23201d` |
| `--nav-bg` | `rgba(28,25,23,0.82)` |

## 字体

- 正文字体: `Noto Sans SC`, `-apple-system`, `BlinkMacSystemFont`, sans-serif
- 代码字体: `JetBrains Mono`, `SF Mono`, `Fira Code`, monospace
- 字号: 16px 基准，行高 1.7
- 注意: 已从衬线体(Noto Serif SC)切换为无衬线体(Noto Sans SC)

## 组件规则

- 卡片圆角: `rounded-xl` (12px)
- 卡片 hover: 轻微浮起 (GSAP `y: -4, scale: 1.02`) + 阴影增加
- 按钮 CTA: `bg-accent text-white` 实色背景
- 次要按钮: `border border-border text-text-muted` 线框
- 空状态: 大号图标 (`text-text-muted/30`) + 引导文字 + 行动链接
- 弹窗: GSAP `scaleIn` (`back.out` 缓动) 入场，缩小+淡出退场

## 每个页面的操作栏

所有内容页面必须在顶部有「添加」和「管理」两个按钮：
```
[+ 写文章]  [⚙ 管理]
```
- 「添加」按钮: 实色 bg-accent，打开 WriteDialog/上传控件
- 「管理」按钮: 线框 border，打开 ContentManager
- **不要做全局统一点的 FAB** — 每页独立按钮

## 中文 UI 规范

- 所有按钮文案、弹窗标题、提示文字使用中文
- 提交按钮文案按场景区分: post→「发布」, diary→「写日记」, note→「发说说」
- 关闭确认: 「放弃编辑？未保存的内容将会丢失。」
- 路径提示: 底部栏显示实际保存路径 (content/posts/, content/notes/ 等)
