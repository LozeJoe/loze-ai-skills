# Agently Mail CLI 安装配置详解

## 概述

Agently Mail 是腾讯 QQ 邮箱团队为 AI Agent 打造的独立邮箱服务，与个人邮箱完全隔离。
- 纯 OAuth 令牌授权，无传统密码
- 通过 `agently-cli` 命令行操作，Agent 代劳
- 不适合：绑定第三方邮件客户端（需要 SMTP 密码）

## 安装步骤

### 第 1 步 — 安装 CLI

```bash
npm install -g @tencent-qqmail/agently-cli
```

### 第 2 步 — 安装 Skill

```bash
npx skills add https://agent.qq.com --skill -g -y
```

安装到: `~/.agents/skills/agently-mail`，自动 symlink 到 Hermes/Reasonix/Claude Code/Trae CN。

### 第 3 步 — OAuth 授权

```bash
agently-cli auth login
```

命令会输出授权 URL，用户需在浏览器中打开并微信扫码授权。

**Windows PowerShell 陷阱：**
- `Start-Process` 无法启动 Node.js 脚本（`不是有效的 Win32 应用程序`）
- 需要创建 `.bat` 批处理文件并重定向输出：

```batch
@echo off
agently-cli auth login > %TEMP%\agently-auth.txt 2>&1
```

然后从 `%TEMP%\agently-auth.txt` 提取授权 URL。

- 授权完成后命令自动退出，无需手动终止

### 第 4 步 — 验证

```bash
agently-cli +me
```

返回格式:
```json
{
  "ok": true,
  "data": {
    "aliases": [{"email": "xxx@agent.qq.com", ...}],
    "constraints": {"max_attachment_count": 3, "max_attachment_size_bytes": "1048576"},
    "rate_limits": {"daily_send_quota": 50, "requests_per_hour": 200}
  }
}
```

## 命令参考

```bash
# 身份
agently-cli +me

# 邮件操作
agently-cli message +list --limit 10
agently-cli message +read --id msg_xxx
agently-cli message +search --q "关键词"
agently-cli message +send --to user@example.com --subject "标题" --body "内容"
agently-cli message +reply --id msg_xxx --body "回复内容"

# 附件
agently-cli attachment +download --msg msg_xxx --att att_xxx
agently-cli attachment +upload --file ./report.pdf

# 认证
agently-cli auth login       # 首次授权
agently-cli auth refresh     # 刷新 token
```

## 限制

| 项目 | 限制 |
|------|------|
| 每日发送 | 50 封 |
| 每小时请求 | 200 次 |
| 每分钟请求 | 10 次 |
| 附件数量 | ≤ 3 个 |
| 单个附件 | ≤ 1 MB |
| 附件总大小 | ≤ 3 MB |

## 适用/不适用场景

| 场景 | 能否用 |
|------|--------|
| Agent 代发邮件 | ✅ |
| 注册需填邮箱+验证码 | ✅（Agent 帮你读验证码）|
| Newsletter 订阅 | ✅ |
| 注册需 SMTP 密码 | ❌ 无传统密码 |
| 绑定第三方客户端 | ❌ 纯 OAuth |

## 相关文件

- CLI: `@tencent-qqmail/agently-cli` (全局 npm)
- Skill: `~/.agents/skills/agently-mail/`
- 官方文档: `https://agent.qq.com/doc/cli-setup.md`
