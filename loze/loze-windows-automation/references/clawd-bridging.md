# Clawd on Desk — Agent 桥接参考

## 架构

Clawd 运行在本地 `127.0.0.1:23333-23337`，通过 HTTP API 接收 Agent 状态。

Hermes 通过内置插件直连；Reasonix (Codex CLI) 通过事件文件监控 + HTTP POST 桥接。

## 端口发现

Clawd 启动时写入 `~/.clawd/runtime.json`：
```json
{"app": "clawd-on-desk", "port": 23333}
```

备用端口（23334-23337）在 23333 被占用时使用。

## 桥接脚本

**主桥接：** `~/.reasonix/clawd-bridge.py`
- 轮询 `~/.reasonix/sessions/` 下最新 `.events.jsonl` 文件
- 解析 JSONL 事件，映射到 Clawd 状态
- POST 到 `http://127.0.0.1:{port}/state`

**看门狗：** `~/AppData/Local/hermes/scripts/reasonix-clawd-watchdog.py`
- WMIC 检查 clawd-bridge.py 是否在运行
- 不在则 subprocess.Popen 拉起
- Hermes cron 每 5 分钟触发

## 事件映射

| Reasonix 事件类型 | Clawd state | Clawd event |
|------------------|------------|-------------|
| session.opened | idle | SessionStart |
| model.turn.started | thinking | UserPromptSubmit |
| model.reasoning | thinking | UserPromptSubmit |
| tool.preparing | working | PreToolUse:{tool_name} |
| tool.dispatched | working | ToolCall:{tool_name} |
| tool.call | working | ToolCall:{tool_name} |
| tool.result | working | PostToolUse |
| model.final | attention | Stop |

## Clawd /state API

```
POST /state
Content-Type: application/json

{
  "agent_id": "hermes|reasonix",
  "event": "AgentState",
  "state": "idle|thinking|working|attention|sleeping",
  "session_id": "可选"
}
```

响应包含 `x-clawd-server: clawd-on-desk` 头。

## 故障排查

1. **Clawd 无响应：** `cat ~/.clawd/runtime.json` 确认端口，`tasklist | grep "Clawd on Desk"` 确认进程
2. **桥接未运行：** `python ~/.reasonix/clawd-bridge.py` 手动启动测试
3. **状态不更新：** 检查 Reasonix 是否在活跃会话中（`~/.reasonix/sessions/*.events.jsonl` 文件大小是否增长）
