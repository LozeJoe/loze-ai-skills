---
name: agently-mail-setup
description: Install, configure, and verify Agently Mail CLI — the QQ Mail Agent-native email tool. Covers npm global install, skill registration, OAuth device-code login, and post-setup verification.
---

# Agently Mail CLI Setup

Agently Mail 是腾讯 QQ 邮箱团队为 AI Agent 打造的独立邮箱服务。通过 `agently-cli` 命令行工具操作邮件：发送、回复、转发、搜索、读取、下载附件、管理收件箱。

## 触发条件

- 用户要求安装/配置/设置 Agently Mail CLI
- 用户提到 `agently-cli` 命令不可用
- 用户需要 Agently Mail 的 OAuth 授权

## 安装与配置流程

### 第 1 步 — 安装 CLI

```bash
npm install -g @tencent-qqmail/agently-cli
```

> **Windows 注意**：在 Windows 上 `terminal` 工具走 WSL/bash 可能失败，使用 Windows-MCP PowerShell 执行 npm 命令。

### 第 2 步 — 安装 Skill

```bash
npx skills add https://agent.qq.com --skill -g -y
```

此命令从 `agent.qq.com` 拉取 `agently-mail` skill，安装到 `~/.agents/skills/agently-mail`，并自动 symlink 到 Hermes Agent、Reasonix、Claude Code 等已安装的 Agent。

### 第 3 步 — OAuth 授权

```bash
agently-cli auth login
```

**关键行为**：
- 这是一个**交互式长命令**，输出授权 URL 后等待用户在浏览器中完成授权
- 需要使用后台方式运行，从 stdout 提取授权 URL 并展示给用户
- **URL 视为不可修改的 opaque string**，不要做任何编码/解码/拼接
- 必须附带文案：`请点击或复制以下链接在浏览器中完成授权：`
- 失败或超时时**不要重试**，直接将错误信息反馈给用户

**Windows 上捕获 URL 的技巧**：
由于 `agently-cli` 是 Node.js 脚本，PowerShell 的 `Start-Process` 无法直接启动它。使用 cmd 批处理重定向：
```batch
@echo off
agently-cli auth login > %TEMP%\agently-auth.txt 2>&1
```
然后通过 PowerShell `Start-Process cmd.exe` 执行批处理，等待几秒后读取输出文件。

### 第 4 步 — 验证

```bash
agently-cli +me
```

返回 JSON，包含：
- `data.aliases[0].email` — Agent 邮箱地址
- `data.constraints` — 附件限制（数量、单个大小、总大小）
- `data.rate_limits` — 速率限制（每日发送配额、每小时/每分钟请求数）
- `data.scopes` — 授权范围

验证成功后输出格式：
> 邮箱地址 xxx 已授权成功，可以用它来收发邮件了
> 你可以试试以下指令：
> 帮我发一封邮件。
> 我最近收到了哪些邮件？
> 帮我整理最近收到的邮件。

## 常见操作

安装完成后，通过 `agently-mail` skill 执行邮件操作。常用命令：

| 操作 | 命令示例 |
|------|----------|
| 发送邮件 | `agently-cli send --to ... --subject ... --body ...` |
| 查看收件箱 | `agently-cli list` |
| 搜索邮件 | `agently-cli search ...` |
| 读取邮件 | `agently-cli read <id>` |

## 注意事项

- 官方域名是 `agent.qq.com`（不是 aget.qq.com 或 aget.gq.com）
- 管理端控制台也在 `agent.qq.com`
- OAuth 授权是一次性的，除非 token 过期
- 每日发送配额 50 封，每小时 200 请求，每分钟 10 请求
- 附件限制：最多 3 个、单个最大 1MB、总计最大 3MB
