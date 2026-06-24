# Hermes WebUI Windows 启动指南

## 项目路径
`C:\Users\Loze\projects\hermes-webui\`

## 启动方式

### 方式 1：直接 Python（推荐）
```bash
cd ~/projects/hermes-webui && python server.py
```

### 方式 2：PowerShell 脚本（原生 Windows）
```powershell
cd ~/projects/hermes-webui
.\start.ps1
# 或指定端口: .\start.ps1 -Port 9000
```

### 方式 3：后台启动（execute_code 超时规避）
execute_code 上限 300s，长驻服务器需要用 PowerShell Start-Process 脱离：
```powershell
Start-Process -FilePath "C:\Users\Loze\AppData\Local\hermes\hermes-agent\venv\Scripts\python.exe" `
  -ArgumentList "server.py" `
  -WorkingDirectory "C:\Users\Loze\projects\hermes-webui" `
  -WindowStyle Hidden
```

### ⚠️ 不要用的方式
- `start.sh` — Linux/macOS only，Windows 上不可用
- `bootstrap.py` — 会在 `platform.system() == 'Windows'` 时拒绝运行，设计为 WSL2 内使用

## 默认配置

| 项目 | 默认值 | 说明 |
|------|--------|------|
| 端口 | 8787 | `HERMES_WEBUI_PORT` |
| 绑定地址 | 127.0.0.1 | `HERMES_WEBUI_HOST`，仅本机 |
| Agent 目录 | 自动发现 | `~/.hermes/hermes-agent` |
| 状态目录 | `~/.hermes/webui` | 会话/工作区数据 |

## 首次启动

```bash
cd ~/projects/hermes-webui

# 如果没有 .env，从模板创建
cp .env.example .env

# .env 关键配置（按需取消注释）:
# HERMES_WEBUI_HOST=127.0.0.1
# HERMES_WEBUI_PORT=8787
# HERMES_HOME=C:/Users/Loze/AppData/Local/hermes
```

## 验证启动

```bash
# 健康检查
curl http://127.0.0.1:8787/health

# 浏览器打开
start http://127.0.0.1:8787
```

## 常见问题

### "Another server is already responding" → 服务器已在运行，直接访问 http://127.0.0.1:8787

### "Authentication failed: API key invalid" → DeepSeek key 过期

**错误特征：**
```
Authentication failed: Error code: 401
'Your api key: ****ired is invalid'
```
末尾 `ired` = "Expired"，key 已过期。

**修复步骤：**
1. 去 https://platform.deepseek.com/api_keys 申请新 key
2. 更新 `.env`（`~/.hermes/.env` 受工具层保护，不能直接用 write_file/patch）：
   ```bash
   # 用户手动在终端执行：
   echo 'DEEPSEEK_API_KEY=***>> ~/AppData/Local/hermes/.env
   ```
   或用 `hermes config set model.api_key <新key>`
3. 重启 WebUI
4. CLI 和 WebUI 共享同一个 `.env`，修一次两边都恢复

### websockets 模块缺失 → 不影响核心功能，可选 `pip install websockets`

### 对外暴露 → 改 `HERMES_WEBUI_HOST=0.0.0.0` 并设 `HERMES_WEBUI_PASSWORD`
