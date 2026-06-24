# B站内容提取 — Windows 实操备忘

> 配合 `bili-note` skill 使用的实战记录。bili-note 是 hub-installed skill，路径在 `C:\Users\Loze\AppData\Local\hermes\skills\bili-note\`。

## 环境速查

| 项目 | 值 |
|------|-----|
| bili-note 安装路径 | `C:\Users\Loze\AppData\Local\hermes\skills\bili-note\` |
| Python 解释器 | `C:\Users\Loze\AppData\Local\hermes\hermes-agent\venv\Scripts\python.exe` |
| 核心功能 | ✅ 无需第三方包（stdlib only） |
| 浏览器 AI 字幕 | ❌ 需要 `web-access`（localhost:3456），当前未配置 |
| 音频 ASR | ❌ 需要 ffmpeg + faster-whisper/funasr/openai-whisper |

## 字幕可用性判断

**不要假设登录就能拿到字幕。** B站很多课程即使登录后 API 也返回空字幕。

判断流程：
1. 先跑 `check_environment.py` 看 `public_subtitles_comments_archive` 状态
2. 探测结果 `need_login_subtitle: true` + `subtitles: []` → 需要登录态 API
3. 用登录态调 `api.bilibili.com/x/player/wbi/v2` → 看 `data.subtitle.subtitles` 数组
4. 如果 `allow_submit: false` + `subtitles: []` → **该课程根本没有字幕**，UP主未开启 AI 字幕
5. 此种情况下唯一路线是音频 ASR（见下方），或生成纯结构大纲

## Chrome CDP 踩坑

`--remote-debugging-port=9222` 在 Windows 上**极不可靠**：

- 如果 Chrome 已有后台进程在跑，新进程会复用现有 profile，**忽略**调试端口参数
- 杀进程后重启，Chrome 的 background apps 常驻可能导致端口仍不监听
- 即使 Chrome 进程正常启动，`localhost:9222` 也可能持续拒绝连接（Connection Refused）

**可靠替代方案**：
- `web-access` 代理（bili-note 脚本的预期路线，localhost:3456）
- 或直接提取 Cookie 调 API（见下方）

## Cookie 提取路线

Chrome cookies 在 `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Network\Cookies`。

**问题**：Chrome 锁着该文件，且现代 Chrome 用 AES-256-GCM 加密（v10/v20 格式），不是纯 DPAPI。

**可行流程**：
1. `taskkill /F /IM chrome.exe` 短暂杀进程
2. 立即 `shutil.copy2()` 复制 Cookies 文件到 TEMP
3. 重启 Chrome（`Start-Process chrome.exe`）
4. 从 `%LOCALAPPDATA%\Google\Chrome\User Data\Local State` 读取 `os_crypt.encrypted_key`
5. DPAPI 解密该 key → 得到 AES 密钥
6. 用 AES-256-GCM 解密每个 cookie 值
7. 构建 Cookie header 直接调 B站 API

**注意**：杀 Chrome 会打断用户的浏览会话，原则上只应在用户明确许可时使用。优先让用户自己准备登录态。

## 与 Obsidian 的集成

用户偏好：**不要自动存到 Obsidian**。用户会自己指定目标路径。

正确流程：
1. 提取完成后把笔记放在临时目录或用户指定的路径
2. 告知用户文件位置
3. 用户决定是否导入 Obsidian 以及放到哪个 Vault/路径

用户的 Obsidian 路径：`C:\Users\Loze\Obsidian\`（默认 vault），也可能用 `D:\Loze\Documents\Obsidian Vault\`。
