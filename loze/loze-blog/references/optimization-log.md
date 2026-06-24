# Loze Blog 优化日志

> 基于三轮流代码审查的 bug 发现与修复记录
> 项目: loze-blog-next (Next.js 16.2.7)

---

## 已修复问题清单

### 致命 Bug (5个)

| # | 问题 | 根因 | 修复 |
|---|------|------|------|
| 1 | 博客路由全部 404 | `/blog/:slug*` → `/posts/:slug*` 错误重定向 | 删除重定向 |
| 2 | API 零安全防护 | 7 个 API route 无任何鉴权 | 创建 auth.ts 中间件，dev 免检/prod 需 Bearer token |
| 3 | 暗色模式颜色不一致 | CSS 变量用 `@media` 但 ThemeProvider 用 `class="dark"` | `@media` → `.dark` 选择器 |
| 4 | MDX 自定义组件不渲染 | `compileMDX({ components: {} })` 空对象 | 传入 `mdxComponents` |
| 5 | 草稿可通过 URL 公开访问 | `getPostBySlug`/`getAllPostSlugs` 不过滤 draft | 增加 draft 检查，返回 null 时跳过 |

### 严重 Bug (5个)

| # | 问题 | 根因 | 修复 |
|---|------|------|------|
| 6 | 导航栏 /gallery 404 | 路由实际是 /photos | 改为 /photos，新增 /notes 入口 |
| 7 | 文章阅读时间刷新后变为 1 | API 不返回 content，客户端硬编码 | API 返回 content 前 200 字，客户端计算 |
| 8 | 日记内容刷新后丢失 | API 不返回 content，客户端写死 `""` | API 返回完整 content |
| 9 | 标签云点击无筛选 | blog/page.tsx 不读 `tag` query 参数 | 增加 activeTag 状态 + getPostsByTag |
| 10 | 友链头像回退失效 | Tailwind `hidden` 的 `!important` 覆盖 JS inline style | useState(imgError) 条件渲染 |

### 代码质量 (6项)

- 删除 8 处未使用导入 (Image/Search/Clock/ReactNode/SITE_URL/tagStr/ext/useSearchParams)
- 友链头像 `<img>` → `next/image`
- 15 个 `any` → PostItem/DiaryItem/NoteItem/PhotoItem/FriendItem 接口
- 创建 `src/lib/types.ts` 共享类型
- 创建 `src/lib/auth.ts` 鉴权中间件
- 修复 useEffect 依赖数组 + setState in effect 模式

## 架构经验

1. **服务端+客户端分离模式**最适合文件系统博客：page.tsx 读文件+渲染静态部分，*-client-page.tsx 处理所有交互
2. **内容管理 CRUD** 用 Node.js fs API route 可以在本地和 Vercel 都工作（Vercel 上文件系统是临时的）
3. **状态刷新**统一模式：`const [refreshKey, setRefreshKey] = useState(0)` → `useEffect(() => { fetch().then(setData) }, [refreshKey])` → 操作成功后 `setRefreshKey(k => k+1)`
