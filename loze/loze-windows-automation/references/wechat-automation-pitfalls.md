# WeChat 自动化实战坑点

> 记录于 2026-06-22，通过 Windows-MCP 操控微信桌面端的实战教训。

## 1. App 启动创建重复登录窗口

`mcp_windows_mcp_App(mode='launch', name='微信')` 会在已有微信登录态下额外弹出一个登录选择窗口（"进入微信"/"切换账号"/"仅传输文件"），挡在聊天界面上方，导致所有对聊天列表、输入框等元素的点击被拦截。

**症状**：
- UI Tree 底部出现独立的 `window "微信"` 含 `进入微信` / `切换账号` / `仅传输文件` 按钮
- 点击聊天列表项（如 (1383, 493) 白骑士）无反应
- `Focused Window` 始终显示登录窗而非聊天窗口

**解决（按优先级）**：
1. **PowerShell SendKeys** — 激活窗口并发送 Enter（"进入微信" 默认聚焦）：
   ```powershell
   $wshell = New-Object -ComObject wscript.shell
   $wshell.AppActivate('微信')
   Start-Sleep 1
   $wshell.SendKeys('{ENTER}')
   ```
2. 点击登录窗的关闭按钮（坐标约 1456, 544，具体看 Snapshot）
3. **避免用 App 启动已有的微信**——如果微信已在运行且有登录态，不要用 `App(launch)`，用窗口切换方式

## 2. Pillow 截图后端无法检测前台窗口

Screenshot（Pillow 后端）的 `Focused Window: No active window found` 和 `Opened Windows: No windows found` 是预期行为，**不等于窗口不存在**。UI Tree（来自 UIAutomation）能正确识别所有窗口和 UI 元素。

**规则**：Screenshot 用于快速看图，Snapshot 用于获取 UI 元素坐标和交互能力。

## 3. Type 工具空字符串报错

`Type` 的 `text` 参数不能为空字符串，会报 `string index out of range`。

**正确做法**：只想按 Enter 时用 `Shortcut(shortcut='enter')`。

## 4. Type 工具需配合 Click 先聚焦

在输入框打字前必须先点击输入框获取焦点：
1. `Click(loc=[input_x, input_y])`
2. `Wait(duration=1)`
3. `Type(loc=[input_x, input_y], text="...", clear=true)`
4. `Shortcut(shortcut='enter')` 发送

## 5. 联系人列表点击不可靠 → 用搜索

直接通过坐标点击联系人列表项（如白骑士）可能不触发聊天切换，原因不明（窗口层级/焦点问题）。

**✅ 可靠方案**：
1. `Click(loc=[1374, 246])` — 点击搜索框
2. `Type(loc=[1374, 246], text='联系人名称', press_enter=true)` — 搜索并回车

## 6. ⚠️ 朋友圈点赞：死胡同

WeChat PC 朋友圈的「赞」「评论」按钮是**自定义绘制元素**。Windows UIAutomation 无法识别，右键菜单也不含点赞选项，双击帖子会打开图片查看器而非互动菜单。

**规则**：**不要尝试自动化朋友圈点赞**。无解。告诉用户用手机微信操作。

## 7. 消息撤回：2分钟窗口

微信消息撤回仅限发送后 **2 分钟内**。超时后右键菜单不显示「撤回」选项，只有复制/删除/转发等。

**规则**：误发后立即尝试撤回，超时则只能手动处理。

## 8. 主聊天窗窗口名称

微信登录后的主聊天界面在 UI Tree 中标题为 `Weixin`，登录/选择界面标题为 `微信`。两个是不同的窗口。

## 9. ⚠️ 致命坑：Windows-MCP 并发冲突

**问题**：Windows-MCP 是**单实例桌面操控服务器**。当多个 Hermes 会话（如当前微信会话 + 另一个 CLI/Tmux 会话）同时通过 Windows-MCP 执行操作时，双方的截图、鼠标移动、点击会相互干扰——一个会话的 Snapshot 移动了鼠标，另一个会话的 Click 就会点到错误位置。

**症状**：
- 正在进行的自动化操作（如考试答题）突然跳回首页
- 点击经常命中错误目标
- 鼠标光标在不操作时频繁跳动

**检测方法**：
```
# 查看是否有多余的 windows-mcp serve 进程
mcp_windows_mcp_Process(mode='list', name='python')
```

**规则**：
- 在执行任何 Windows-MCP 操作前，**先确认没有其他 Hermes 会话正在操控桌面**
- 如果用户提到「另一个程序在用浏览器」或「电脑在跑别的东西」，**立即停止所有 Windows-MCP 操作**
- 等待用户明确说「可以了」再继续

## 10. execute_code 中文标点编码陷阱

在 `execute_code` 的 Python 字符串中嵌入中文标点（如 `""`（全角引号）、`，`、`；`）可能触发 `SyntaxError`，因为 Hermes 代码生成层可能破坏字符串边界。

**症状**：
```
SyntaxError: invalid character '，' (U+FF0C)
```

**解决**：
- 中文内容先 `write_file` 写到磁盘，再用 `execute_code` 读取后处理
- 或使用 Windows-MCP `FileSystem` 工具写文件
- Python 字符串中用半角标点 + 「」替代全角引号
