---
name: loze
description: "刘哲凯的总入口技能 — 自动加载 Java 开发、Windows 自动化、AI 工具链三大子技能"
version: 1.0.0
author: Hermes Agent (for Loze)
platforms: [windows]
metadata:
  hermes:
    tags: [刘哲凯, 总入口, auto-load]
    related_skills: [loze-java-dev, loze-windows-automation, loze-ai-integration, loze-multi-agent, loze-workflow, loze-blog, loze-side-hustle]
    auto_load: true
---

# Loze — 刘哲凯专属总技能

## 触发条件

任何与刘哲凯相关的对话开始时，自动加载此技能。此技能作为总入口，指向三个子技能：

- **loze-java-dev** — Java/JavaEE 全栈开发 (Spring, MyBatis, Vue)
- **loze-windows-automation** — Windows AI 桌面自动化 (Windows-MCP, Clawd)
- **loze-ai-integration** — 本地 AI 工具链 (Ollama, EasyOCR, Codex)
- **loze-multi-agent** — 多 Agent 并行协作 (Hermes + Reasonix + Codex)
- **loze-workflow** — 全天候工作流 (日/周循环、开发流水线)
- **loze-side-hustle** — 编程副业搞钱 (闲鱼接单、反诈过滤、Loze+Hermes协作交付)
- **gsap-\\*** — GSAP 动画引擎 (core/timeline/scrolltrigger/react/plugins/performance/utils/frameworks)

## 人物速写

- 姓名：刘哲凯 · 学号：2400130326 · 软件技术3班
- OS：Windows 11 · Shell：Git Bash/MSYS2
- 主模型：DeepSeek v4 Pro
- 语言偏好：中文（简体）
- 网络环境：GitHub 被封锁，使用 gh-proxy.com 镜像
- **双 Agent 并行**：同时使用 Hermes 和 Reasonix。Hermes 负责编排/审查/复杂任务分解；Reasonix 负责独立编码任务和主题美化
- **设计偏好**：默认文艺暖棕风(#5b7b6a墨绿 accent)，同时接受 Reasonix 的主题改动(如复古黑金 Art Deco)

## AI 协作风格偏好

- **方案驱动**：先写方案文档(.md) → AI 执行 → 用户本地审查 → 提供优化文档 → 分轮迭代修复
- **分轮执行**：不要一次塞入过多修改。每轮聚焦一个方案文档，修完→验证→下一轮
- **功能页面全覆盖**：每个页面都要有对应的操作按钮(添加/管理)，不做全局统一点的按钮
- **本地优先**：博客部署在本地(localhost:6708)用于个人使用，暂不考虑上线
- **中文界面**：所有 UI 组件、弹窗、提示文字使用中文
- **关键规则**: 方案文档中的内容具有最高优先级，AI 必须严格按方案执行。方案更新后立即对照新版本修复
- **简洁输出**：当用户询问技术指令/安装步骤/命令时，只给步骤和指令，不要展开解释。用户说"只给我步骤和指令就行"即为明确信号
- **任务聚焦**：用户明确在做一个项目时，不要跳到其他项目的错误。用户说"我们不是在干X吗"是明确的停止信号

2. **日常任务** — 遇到对应领域问题时，加载对应子技能
3. **跨领域任务** — 同时加载多个子技能
4. **新任务** — 先判断属于哪个领域，再加载对应技能

## 知识管理规则

- **Obsidian 知识库**：`C:/Users/Loze/Obsidian/`，只记录用户自己需要掌握的知识
- **AI 不记录**：AI agent 能自行完成的知识（如工具安装升级、AI 配置等）不写入 Obsidian
- **已有笔记**：AI技能使用手册.md、每日学习日志模板.md、毕设AI辅助框架.md

## 参考文档

- `references/bilibili-extraction.md` — B站内容提取实战（字幕判断、Chrome CDP、Cookie 提取、Obsidian 集成偏好）

## 已知坑点

### Python 脚本中的 Windows 路径
在 Python 脚本中使用 `Path.exists()` 或 `os.path.exists()` 时，**必须用 Windows 原生路径格式**（`C:\Users\Loze\...` 或 `r"C:\Users\Loze\..."`），不能用 MSYS 风格的 `/c/Users/Loze/...`。后者在 Git Bash 的 shell 中可用，但 Python 的 `Path` 对象不识别。

```python
# ✓ 正确
path = Path(r"C:\Users\Loze\projects")
# ✗ 错误 — Path.exists() 返回 False
path = Path("/c/Users/Loze/projects")
```

### GitHub 网络限制
- 直接从 GitHub clone/pull 会超时
- 用 `gh-proxy.com` 镜像：`git clone https://gh-proxy.com/https://github.com/owner/repo`
- Ollama 模型注册表 (`registry.ollama.ai`) 可直接访问，无需镜像
- npm 注册表通常可直接访问
- **降级方案**：当 git clone 超时时，用 Python `urllib` 下载 zip 包 → 解压（`execute_code` 沙箱内 urllib 可通 GitHub）

### 终端工具 WSL 乱码
当 `terminal` 工具输出乱码（显示 WSL 安装提示等二进制字符），**不要反复重试**。立即切换到 `execute_code`，用 `subprocess.run()` 执行命令。`execute_code` 直连 Windows 可执行文件，绕过 WSL 层干扰。

## 已配置的基础设施

| 资源 | 说明 |
|------|------|
| `~/scripts/` | 12 个脚本：Java 脚手架、SQL 生成器、OCR 批处理、环境检测、微信/云音乐自动化 |
| `~/projects/loze-blog-next/` | Next.js 静态博客 (MDX, 日记/相册/友链/说说, 端口6708) — 详见 loze-blog |
| `~/projects/loze-blog-vue/` | Vue3+SpringBoot 博客 (端口5173前端/8088后端) — 详见 loze-blog |
| `~/projects/Windows-MCP/` | AI 桌面自动化 MCP 服务器 |
| `~/projects/hermes-webui/` | Hermes Web 界面 (端口8787, `python server.py` 启动) |
| `~/projects/gbrain/` | Garry Tan 知识大脑 (Bun+PostgreSQL, `bun run src/cli.ts`) |
| `~/projects/hermes-agent-self-evolution/` | DSPy+GEPA 技能自进化 (pip -e 已装) |
| `~/Documents/comfy/ComfyUI/` | ComfyUI 本地生图 (端口8188, SDXL 模型, RTX 4060 8GB) |
| `ai-image-prompts-skill` | 1万+精选生图提示词库 (Hermes skill 已装) |
| `awesome-hermes-agent` | Hermes 生态导航 (skill tap 已添加) |
| Cron `c994f126e3ae` | 每日 21:00 自动生成工作总结并投递 |
| Cron `e7a3493e7a27` | 每周一 09:00 项目健康检查 |
| Cron `25b2e6063772` | 每周一 10:00 学习建议推荐 |
| Windows-MCP `loze_tools.py` | 3 个自定义 MCP 工具（桌面整理、临时清理、项目启动） |
| 微信 Bot | WeixinClawBot 已连接 Gateway (token 过期时 `hermes gateway setup` → 12→y→Y→扫码) |

## 快速命令

```bash
# 博客 (端口 6708)
cd ~/projects/loze-blog-next && npm run dev -- -p 6708

# Java 开发
cd /c/Users/Loze/IdeaProjects/Demo02

# Windows-MCP
cd /c/Users/Loze/Windows-MCP && uv run windows-mcp

# OCR 文档
python /c/Users/Loze/ocr_docx.py <文件路径>

# Codex 自动编程
codex exec "任务描述"

# Ollama 本地模型
ollama list && ollama run <模型名>

# 查看定时任务
hermes cron list

# ComfyUI 生图 (端口 8188)
cd ~/Documents/comfy/ComfyUI && python main.py

# Hermes WebUI (端口 8787)
cd ~/projects/hermes-webui && python server.py

# 微信 Bot 重连
hermes gateway setup  # → 12 → y → Y → 扫码

# 项目脚手架
python ~/scripts/scaffold.py 项目名 com.example.pkg --type spring

# 实体类→SQL
python ~/scripts/sqlgen.py User.java
```
