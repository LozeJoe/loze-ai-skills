---
name: loze-multi-agent
description: "刘哲凯多 Agent 并行协作模式 — Hermes delegate_task + Reasonix + Codex 三引擎"
version: 1.0.0
author: Hermes Agent (for Loze)
platforms: [windows]
metadata:
  hermes:
    tags: [多Agent, 并行, 协作, delegate_task, 刘哲凯]
    related_skills: [loze-java-dev, loze-windows-automation, loze-ai-integration, codex]
---

# Loze Multi-Agent — 刘哲凯多 Agent 协作模式

## 可用 Agent 矩阵

| Agent | 角色 | 用途 | 状态 |
|-------|------|------|------|
| **Hermes** | 编排者 | 任务分解、分配、审查、3个子Agent并行 | 主用 |
| **Reasonix** | 编码者 | 独立开发任务、代码生成 | 主用 |
| **Codex CLI** | 编码者 | 自动化编码、PR 生成 | 已装未用 |
| **Claude Code** | 编码者 | 代码审查、重构分析 | 已装未用 |

## 并行开发流水线 (Hermes delegate_task)

### 配置
- max_concurrent_children: 3
- max_spawn_depth: 1
- orchestrator_enabled: true
- child_timeout_seconds: 600

### 标准三路并行模式

```
         ┌─────────────┐
         │   Hermes    │ <- 你下达意图
         │  (编排者)    │
         └──┬───┬───┬──┘
            │   │   │
    ┌───────┘   │   └───────┐
    ▼           ▼           ▼
┌───────┐  ┌───────┐  ┌───────┐
│Agent 1│  │Agent 2│  │Agent 3│
│后端API│  │前端UI │  │测试/DB│
└───────┘  └───────┘  └───────┘
    │           │           │
    └───────────┼───────────┘
                ▼
         ┌─────────────┐
         │   Hermes    │ <- 审查 + 合并
         │  (审查者)    │
         └─────────────┘
```

### 调用方式

```python
# 方式1: 单任务委托
delegate_task(
    goal="为数字乡村管理系统的 VillageController 添加分页和搜索功能",
    context="项目在 /c/Users/Loze/projects/digital-village/..."
)

# 方式2: 并行三路
delegate_task(tasks=[
    {
        "goal": "为 VillageController 添加分页查询",
        "context": "Spring Boot + MyBatisPlus...",
        "toolsets": ["terminal", "file", "web"]
    },
    {
        "goal": "为 Village.vue 添加搜索表单",
        "context": "Vue 3 + Element Plus...",
        "toolsets": ["terminal", "file"]
    },
    {
        "goal": "生成对应的单元测试",
        "context": "JUnit + MockMvc...",
        "toolsets": ["terminal", "file"]
    }
])
```

## Reasonix 独立任务模式

Reasonix 适合从终端独立启动的编码任务：

```bash
# Reasonix 单任务 (在项目目录下)
cd /c/Users/Loze/projects/digital-village
reasonix "优化 VillageController 的异常处理"
```

### Hermes + Reasonix 分工

| 场景 | Hermes | Reasonix |
|------|--------|----------|
| 复杂多步骤任务 | 编排 + 分配 | 执行具体子任务 |
| 代码审查 | 运行 ai_review.py | 审查建议实施 |
| 新功能开发 | 写设计文档 | 写实现代码 |
| Bug 修复 | 诊断根因 | 实施修复 |
| 学习新技术 | 搜索 + 总结 | 写练习代码 |

## Codex CLI (备选)

```bash
# 自动模式
codex exec --full-auto "为数字乡村管理系统添加导出 Excel 功能"

# 后台执行
codex exec "重构 Village 实体类，用 Lombok" &
```

## Reasonix Skill 管理

Reasonix skills 是独立的 `.md` 文件，存放在 `~/.reasonix/skills/`，格式与 Hermes 不同。

### Skill 文件格式

```yaml
---
name: skill-name
description: 技能描述
runAs: subagent         # 可选：作为子代理运行
model: deepseek-v4-pro  # 可选：指定模型
---
# 正文内容（Markdown）
```

### 从 Hermes 同步 Skill 到 Reasonix

当在 Hermes 上安装了新 skill 后，如需同步到 Reasonix：

1. 读取 Hermes skill 内容（`~/.hermes/skills/<name>/SKILL.md`）
2. 转换 frontmatter 为 Reasonix 格式（`name` + `description` + 可选 `runAs`/`model`）
3. 写入 `~/.reasonix/skills/<name>.md`
4. 不需要同步支持文件 —— Reasonix 只识别单个 `.md` 文件

### 当前 Reasonix Skills

| 文件 | 来源 |
|------|------|
| `loze-fullstack.md` | 自建 |
| `gsap-core.md` | Hermes 同步 |
| `gsap-scrolltrigger.md` | Hermes 同步 |
| `gsap-timeline.md` | Hermes 同步 |
| `browser-harness.md` | Hermes 同步（browser-use） |
| `remote-browser.md` | Hermes 同步（browser-use） |

### Hermes Skills 安装速查

```bash
# 搜索 skill
hermes skills search <keyword>

# 安装（非交互）
echo "y" | hermes skills install <identifier>

# 从 GitHub repo 添加 tap 源
hermes skills tap add <owner/repo>

# 安全警告：DANGEROUS 判定的 skill 无法安装（--force 也无效）
```

## Clawd on Desk 多 Agent 可视化

所有 Agent 状态统一显示在 Clawd 桌面宠物上：

| Agent | 连接方式 | 状态 |
|-------|---------|------|
| Hermes | 插件 `clawd-on-desk` (8个生命周期钩子) | 自动 |
| Reasonix | 事件文件桥接 `~/.reasonix/clawd-bridge.py` | cron 保活 |

Clawd 端口从 `~/.clawd/runtime.json` 读取（23333-23337）。详见 `loze-windows-automation` 技能。

1. **Hermes 是总指挥** — 任务分解、最终审查由 Hermes 完成
2. **Agent 之间不互相等待** — 每个 Agent 的任务独立
3. **共享上下文** — 文件路径、API 约定通过 context 参数传递
4. **验证在前** — Agent 返回结果后，Hermes 验证文件是否实际创建/修改
5. **增量开发** — 每次只给一个清晰可验证的子任务

## Agent 间直接通信（agent-hub.js）

除了 delegate_task 和独立启动，Hermes 和 Reasonix 可以直接互发消息：

### 架构

```
Hermes  ←── agent-hub.js ──→  Reasonix
           文件队列: ~/.agent-hub/inbox/
```

### 启动 Reasonix

```bash
npx reasonix code    # 在当前目录启动 Reasonix TUI
```

### 常用命令

```bash
node agent-hub.js send hermes "消息内容"     # Hermes → Reasonix
node agent-hub.js send reasonix "消息内容"   # Reasonix → Hermes
node agent-hub.js read hermes               # 读取发给 Hermes 的消息
node agent-hub.js read reasonix             # 读取发给 Reasonix 的消息
node agent-hub.js list                      # 查看所有消息
node agent-hub.js watch hermes              # 实时监听（长期运行）
node agent-hub.js clear                     # 清空消息
```

### 使用模式

- **日常通信**: 用 `send` + `read`，Agent 间互发消息
- **后台监听**: 用 `watch` 持续运行，新消息秒收
- **脚本位置**: `~/agent-hub.js`，零依赖纯 Node.js
- **消息存储**: `~/.agent-hub/inbox/`，JSON 文件

> 完整脚本和分工建议见 [references/agent-hub-communication.md](references/agent-hub-communication.md)

### Hermes → Reasonix 消息示例

在 Hermes 中对我说"给 Reasonix 发消息" → 我执行：
```bash
node agent-hub.js send hermes "帮我查 CakeShop 的 i18n 问题"
```
Reasonix 执行 `node agent-hub.js read reasonix` 即可收到。

## 常用组合

### 全栈功能开发 (3路并行)
```
Agent 1: 后端 Controller + Service + Mapper
Agent 2: 前端 Vue 组件 + API 调用
Agent 3: 数据库迁移脚本 + 单元测试
```

### 代码审查 + 修复 (3路并行)
```
Agent 1: 运行 ai_review.py + 分析安全问题
Agent 2: 检查代码规范和性能
Agent 3: 审查测试覆盖率
```

### 项目重构 (3路并行)
```
Agent 1: 分析现有代码结构
Agent 2: 生成重构方案
Agent 3: 评估风险点
```
