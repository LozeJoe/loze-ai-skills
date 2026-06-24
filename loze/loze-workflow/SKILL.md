---
name: loze-workflow
description: "刘哲凯的人机合一全天候工作流 — 从意图到交付的完整 AI 驱动开发流水线，含闲鱼副业交付模式"
version: 1.0.0
author: Hermes Agent (for Loze)
platforms: [windows]
metadata:
  hermes:
    tags: [人机合一, 工作流, 全流程, AI驱动, 刘哲凯]
    related_skills: [loze, loze-multi-agent, loze-java-dev, loze-windows-automation, loze-ai-integration, design-md, claude-design, popular-web-designs, baoyu-infographic, gsap-core, gsap-timeline, gsap-scrolltrigger, gsap-plugins, p5js, pretext, sketch]
---

# Loze Workflow — 人机合一全天候工作流

## 核心理念

```
早晨 (意图输入)   →   白天 (AI自主执行)   →   晚上 (审查验收)
    5min                   持续运行                  5min
```

你只做决策和审查，所有机械执行由 AI 完成。

---

## 日循环

### 早晨 5 分钟

```
打开 Hermes，说:

"今天我要完成:
1. [具体任务1]
2. [具体任务2]
3. [具体任务3]

帮我预加载环境，分解任务，准备开工。"
```

AI 会:
- 加载 loze 技能包
- 分解任务为子任务
- 对可并行任务用 delegate_task
- 打开需要的项目 (通过 LozeOpenProject)

### 白天持续

```
每次需要写代码时:
  "帮我实现 X 功能"   → AI 写代码 → 你审查 → 通过

每次遇到 Bug:
  "这段代码报错 X"   → AI 诊断 → 你确认 → AI 修复

每次完成一个小功能:
  "review 我刚才的改动" → AI review → 修复问题 → commit

写博客时:
  cd ~/projects/loze-blog-next && npm run dev
  在 content/posts/ 下创建 .mdx → 浏览器预览
  详见 loze-blog 技能
```

### 晚上 5 分钟

```
"总结今天做了什么，明天应该做什么"

AI 会:
- 列出今天完成的任务
- 推荐明日优先级
- 更新 Obsidian 日总结
- 标记未完成项

每晚 21:00 Cron 自动执行总结
```

---

## 周循环

```
周一 09:00  → 项目健康检查 (自动 Cron)
周一 10:00  → 学习建议推荐 (自动 Cron)
周五 17:00  → 周回顾 (手动触发)
周日 20:00  → 下周计划 (手动触发)
```

---

## 开发流水线 (标准模式)

### 新功能开发

```
Step 1: 你描述需求
  "为数字乡村系统添加数据导出 Excel 功能"

Step 2: Hermes 并行分解
  Agent 1: 后端 Controller + Service + 导出逻辑
  Agent 2: 前端下载按钮 + API 调用
  Agent 3: 单元测试 + 集成测试

Step 3: 你审查结果
  检查文件是否正确创建、代码质量、逻辑正确性

Step 4: 运行测试
  python ~/scripts/ai_review.py --dir .
  
Step 5: 提交
  git add -A && git commit -m "feat: 添加 Excel 导出"
```

### Bug 修复

```
Step 1: 你描述 Bug
  "村民档案列表分页不工作"

Step 2: Hermes 诊断
  读代码 → 定位根因 → 提出修复方案

Step 3: 你确认 + AI 修复

Step 4: 回归测试
  Agent: "运行相关测试用例，确认修复"
```

### 代码审查

```
# 自动化审查
python ~/scripts/ai_review.py --dir .

# 深度审查 (多Agent并行)
Agent 1: 安全漏洞
Agent 2: 代码质量 + 规范
Agent 3: 架构设计 + 性能
```

---

## 自动化 Cron 一览

| 时间 | 任务 | 作用 |
|------|------|------|
| 每天 21:00 | 日总结 | 回顾今天对话，输出总结 |
| 每周一 09:00 | 项目健康检查 | 未提交代码、陈旧项目提醒 |
| 每周一 10:00 | 学习建议 | 知识图谱分析，推荐学习方向 |

---

## 闲鱼副业交付流水线（Loze + Hermes 协作）

用户通过闲鱼接编程/AI服务单，Hermes 负责所有技术产出。

### 分工

| Loze | Hermes |
|------|--------|
| 闲鱼沟通、报价、收款、交付文件 | 写代码、生成内容、设计图、配音 |

### 标准流程

1. 买家咨询 → Loze 问清需求+预算+截止
2. Loze 把需求发我 → 我评估可行性+建议报价
3. Loze 报价 → 买家下单
4. 我干活 → 发给 Loze
5. Loze 交付 → 确认收货

### 沟通格式

```
新单：
类型：Python/Vue/Java/AI设计/配音/OCR/文案
需求：xxx
预算：¥xxx
截止：x天内
```

### 红线

- ✅ 走闲鱼担保交易，不私下转账
- ❌ 不代写毕设/论文/课程设计（A6灰产）
- ❌ 不刷量/灰产/采集隐私
- ❌ 不先交付完整代码再等付款

### 已上架品类（9个）

| 品类 | 起步价 | Hermes 能力 |
|------|--------|------------|
| Python脚本/Excel | ¥50 | 写代码 |
| AI工具安装配置 | ¥30 | 写教程/远程指导 |
| Vue前端/网页 | ¥100 | 写前端代码 |
| 数据采集 | ¥80 | 写爬虫脚本 |
| Java后端/Bug修复 | ¥100 | 写Java代码 |
| AI设计/海报 | ¥15 | ComfyUI生图 |
| AI配音 | ¥30 | TTS语音合成 |
| OCR文档识别 | ¥20 | EasyOCR处理 |
| 内容创作/文案 | ¥30 | 大模型写作 |

### 定价哲学

新号走保守起步价（¥15-100），先攒好评。5 单后全线涨价 30%。AI 设计/配音/OCR 低价引流 → 转化高单价编程单。

### 文件位置

- 画像: `~/.money-state.json`
- 方法论: `~/projects/cheat-on-money/`
- 商品清单: `~/projects/cheat-on-money/闲鱼商品清单.md`
- 交付SOP: `~/projects/cheat-on-money/接单交付SOP.md`
- Obsidian: `D:\Loze\Documents\Obsidian Vault\副业接单记录.md`

用户偏好先写方案文档再执行。这是最高效的协作模式：

```
方案文档(.md)   →   AI 执行   →   本地审查   →   优化文档(.md)   →   AI 修复迭代
     ↑                                                              ↓
     └──────────────────  循环直到满意  ─────────────────────────────┘
```

### 方案文档结构

用户通常提供包含以下内容的 Markdown 文件：
- 技术栈选型和理由
- 功能列表和实现方式
- 项目结构设计
- 时间估算

### AI 执行阶段

1. 读取方案文档，理解全部需求
2. 用 `delegate_task` 并行执行可拆分的子任务
3. 完成后构建验证 → 本地预览

### 优化迭代阶段

用户审查后提供「优化方案.md」或类似文档，包含：
- Bug 清单（按严重程度分级：🔴严重 🟡中等 🟢轻微）
- 构建测试结果
- 功能改进建议
- 动画/体验优化方案
- 工程优化建议

接到优化文档后：
1. 按优先级排序（严重Bug→中等Bug→轻微→功能增强）
2. 批量修复 Bug（多个 patch 并行）
3. 重建验证 → 用户确认

### 博客项目速查

| 项目 | 路径 | 端口 | 启动 | 技术栈 |
|------|------|------|------|--------|
| Next.js 博客 (主用) | `~/projects/loze-blog-next/` | 6708 | `npm run dev -- -p 6708` | Next.js 16 + MDX + GSAP |
| Vue博客 (全栈) | `~/projects/loze-blog-vue/` | 5173/8088 | `cd frontend && npm run dev` | Vue3 + SpringBoot + GSAP |

博客功能：文章(MDX/Shiki高亮)、日记(时间线+心情)、相册(瀑布流+灯箱)、友链(卡片+回退)、说说(Twitter风)、暗色模式、标签筛选、RSS、SEO、GSAP全站动画。

### 创意设计技能栈

用于网页/前端项目的美化和动画：

| 技能 | 用途 |
|------|------|
| `gsap-core` | 动画引擎 |
| `gsap-timeline` | 时间线编排 |
| `gsap-scrolltrigger` | 滚动驱动动画 |
| `gsap-plugins` | SplitText/MorphSVG/Flip等 |
| `popular-web-designs` | 54套设计系统参考(Stripe/Linear/Apple…) |
| `claude-design` | 设计品味+避坑 |
| `p5js` | 生成艺术/创意编程 |
| `pretext` | 文字即几何创意效果 |
| `sketch` | HTML变体快速对比 |
| `design-md` | Google DESIGN.md令牌规范 |
| `baoyu-infographic` | 信息图生成 |
| `comfyui` | AI图像/视频生成 |

---

## 工具索引

| 命令 | 用途 |
|------|------|
| `npm run dev` | 启动博客开发服务器 (loze-blog-next) |
| `npm run build` | 构建博客生产版本 |
| `python ~/scripts/scaffold.py` | Java 项目脚手架 |
| `python ~/scripts/sqlgen.py` | 实体类 → SQL |
| `python ~/scripts/batch_ocr.py` | 批量 .docx OCR |
| `python ~/scripts/ai_review.py` | AI 代码审查 |
| `python ~/scripts/knowledge_migrate.py` | 知识迁移 |
| `python ~/scripts/knowledge_graph.py` | 知识图谱 |
| `python ~/scripts/project_health.py` | 项目健康检查 |
| `python ~/scripts/ollama_finetune_prep.py` | 微调数据准备 |
| `bash ~/scripts/env_check.sh` | 环境检测 |
| `delegate_task` | 多 Agent 并行 |
| `hexo` | 博客管理 (新建/预览/生成/部署) — 详见 loze-blog |

---

## 相关技能

- `loze` — 总入口（人物速写、环境、偏好）
- `loze-multi-agent` — 多 Agent 并行协作
- `loze-java-dev` — Java 全栈开发
- `gsap-*` — GSAP 动画引擎（core/timeline/scrolltrigger/react/plugins/performance/utils/frameworks）

## 参考文件

- `references/gsap-pattern.md` — GSAP 全站动画集成模式（架构、组件用法、关键规则、坑点）
- `references/gsap-vue-directives.md` — GSAP Vue 3 指令模式（v-scroll-reveal + v-fade-in，可直接复制使用）
- `references/admin-backend.md` — 管理后台建设备忘（认证、鉴权、配置、视频背景、常见问题）

## Reasonix 并行使用注意事项

用户同时使用 Hermes 和 Reasonix 两个 AI Agent。两者可能修改同一项目的文件，需注意：

- **修改文件前先 `git status`** 检查是否有 Reasonix 留下的未提交改动
- **冲突信号**: globals.css、layout.tsx、page.tsx 被外部修改 → Reasonix 做了主题或结构调整
- **不要覆盖 Reasonix 的改动**: 如果发现有未预期的文件变更，先 `git diff` 查看内容再决定是否保留
- **用户可能切换 Agent**: 用户说「Reasonix 进行了优化」时，先去 `git diff` 看改了什么，再在此基础上继续工作
- **密码等配置可能被改**: Reasonix 可能修改了管理员密码等配置，操作管理后台相关功能前先确认当前有效凭据

## 关键原则

1. **你永远做最终决策** — AI 的建议你可以接受、修改、拒绝
2. **增量交付** — 每次只做一个小任务，快速审查
3. **测试先行** — AI 写代码的同时生成测试
4. **版本记录** — 每个有意义的变化都 commit
5. **日清日结** — 当天的对话当天总结

## Next.js 博客开发模式 (loze-blog-next)

### 项目速查
- 路径: `C:\Users\Loze\projects\loze-blog-next`
- 端口: 6708 (3000 被其他项目占用)
- 启动: `npm run dev -- -p 6708`
- 管理后台: `/admin` (账号 Loze / 密码 20060708)

### 常用 Next.js 开发模式

**服务端/客户端拆分**: 需要 fs 读写 → 服务端组件；需要交互(onClick/onError) → 客户端组件。两者都需要时：
- `page.tsx`: 读数据(服务端) → 传给 `<ClientPage initialData={data} />`
- `client-page.tsx`: "use client" → 接收 props → 渲染交互

**API Route 模式**: 
- GET 公开, POST/PUT/DELETE 加鉴权
- 开发环境 `process.env.NODE_ENV === "development"` 自动放行
- 生产环境读 Cookie 中的 JWT Token

**CSS 变量暗色模式**: 必须用 `.dark` 选择器而非 `@media (prefers-color-scheme)`，否则 JS 主题切换器的 `class="dark"` 无法覆盖 CSS 变量

**服务端专用模块**: `import "server-only"` 防止客户端组件意外导入 Node.js 模块(fs/path)

### 方案文档驱动迭代

用户通过方案文档(.md)驱动开发，是最核心的工作流：

1. 用户写方案文档(如 `优化方案.md`)列出 Bug/功能/优先级
2. AI 用 `delegate_task` 并行修复（3路 Agent 同时工作）
3. 构建验证 → 本地 http://localhost:6708 预览
4. 用户审查后更新方案文档 → 下一轮迭代

方案文档结构惯例：
- Bug 分级: 🔴致命 > 🟡中等 > 🟢轻微
- 功能改进: 高优先级 > 中优先级 > 低优先级
- 每个问题标注: 位置(文件:行号) + 问题 + 建议修复
- 末尾附修复优先级路线图

### 管理后台建设模式

用 `管理方案.md` 定义需求 → 分 Phase 实现：
- Phase 1: 认证(JWT+Cookie) + 布局 + Dashboard
- Phase 2: 配置系统(数据模型+API+UI)
- Phase 3: 用户管理(个人信息+密码)
- Phase 4: 集成(导航入口+复用现有组件)

每个 Phase 用 `delegate_task` 三路并行（认证系统、配置系统、用户管理），效率提升 3x。
