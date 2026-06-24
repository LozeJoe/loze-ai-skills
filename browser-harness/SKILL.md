---
name: browser-harness
description: 浏览器自动化控制。通过 CDP 连接用户本机 Edge/Chrome，实现网页截图、内容提取、自动化操作。当用户需要：打开网页并截图、提取网页动态内容、看 JS 渲染的页面、自动化操作浏览器时触发。
---

# Browser Harness 技能

通过 `browser-harness` CLI 控制用户本机浏览器（Edge 或 Chrome）。

## 前置条件

1. **本机已安装 browser-harness**：
   ```bash
   git clone https://github.com/browser-use/browser-harness
   cd browser-harness
   uv tool install -e .
   ```

2. **浏览器调试模式启动**：

   **⚠️ 关键：如果 Edge 已经在运行（有普通窗口），直接用 `--remote-debugging-port` 启动只会打开新窗口，不会启用调试端口！**

   **必须完全关闭 Edge 再重启**：
   ```bash
   # 先彻底关闭所有 Edge 进程
   taskkill /F /IM msedge.exe 2>/dev/null
   sleep 2
   
   # 再以调试模式启动
   "/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe" --remote-debugging-port=9222 --no-first-run &
   ```

   或使用 Chrome：
   ```bash
   taskkill /F /IM chrome.exe 2>/dev/null
   sleep 2
   "/c/Program Files/Google/Chrome/Application/chrome.exe" --remote-debugging-port=9222 &
   ```

3. **验证调试端口已开启**（bash 语法）：
   ```bash
   # 检查端口是否在监听
   curl -s http://localhost:9222/json/version | head -5
   # 有 JSON 返回说明端口正常
   ```
   
   或用 Python 快速检查：
   ```bash
   python -c "import socket; s=socket.socket(); s.settimeout(2); r=s.connect_ex(('localhost',9222)); s.close(); print('OPEN' if r==0 else 'CLOSED')"
   ```

4. **设置环境变量**（bash/MSYS 语法）：
   ```bash
   export BU_CDP_URL="http://127.0.0.1:9222"
   ```

## 常用命令

### 截图
```bash
browser-harness -c "new_tab('https://example.com'); wait_for_load(); capture_screenshot()"
```
截图保存到 `$TEMP/shot.png`（Windows 通常在 `C:\Users\<用户名>\AppData\Local\Temp\shot.png`），用完后复制到工作目录分析。

### 打开网页
```bash
browser-harness -c "new_tab('https://example.com')"           # 新标签页打开
browser-harness -c "goto_url('https://example.com')"        # 在当前标签页打开
```

### 等待页面加载
```bash
browser-harness -c "new_tab('https://example.com'); wait_for_load()"
```

### 获取页面信息
```bash
browser-harness -c "print(page_info())"  # URL、标题、尺寸
browser-harness -c "js('document.body.innerText')"  # 页面文字内容
```

### 提取页面内容
```bash
browser-harness -c "js('document.title')"  # 标题
browser-harness -c "js('document.querySelectorAll(\"a\").length')"  # 链接数量
```

### 关闭标签页
```bash
browser-harness -c "close_tab()"
```

## 注意事项

- **新标签用 `new_tab()`**，不要用 `goto_url()`（会覆盖用户当前页面）
- **截图后** 从 `C:\Users\<用户名>\AppData\Local\Temp\shot.png` 复制出来分析（用 `echo $TEMP` 查看具体路径）
- **等待加载**：`wait_for_load()` 在导航后必须调用
- **JS 渲染页面**：`web_fetch` 抓不到的页面可以用这个
- **视频内容**：只能看到封面和文字，看不到实际视频内容
- **云端浏览器**：设置 `BROWSER_USE_API_KEY` 和 `start_remote_daemon()` 可用远程浏览器

## 故障排除

### 连接失败
```bash
# 检查调试端口是否开启（bash）
curl -s http://localhost:9222/json/version | head -3
# 或用 Python
python -c "import urllib.request; print(urllib.request.urlopen('http://localhost:9222/json/version').read()[:200])"
```

### Edge 已在运行导致调试端口不生效
这是最常见的问题。Edge 已运行时，`--remote-debugging-port` 参数被忽略（新窗口加入现有进程）。**必须完全关闭再重启**：
```bash
taskkill /F /IM msedge.exe 2>/dev/null
sleep 2
"/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe" --remote-debugging-port=9222 &
```

### 重启浏览器调试
```bash
# Edge
taskkill /F /IM msedge.exe 2>/dev/null && sleep 2
"/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe" --remote-debugging-port=9222 &

# Chrome
taskkill /F /IM chrome.exe 2>/dev/null && sleep 2
"/c/Program Files/Google/Chrome/Application/chrome.exe" --remote-debugging-port=9222 &
```

### CDP WebSocket 直连批量采集

当需要大规模翻页采集（100+ 页 SPA 内容）时，`browser-harness` CLI 逐页调用太慢。直接用 Python websocket 连接 CDP 端口，合并「提取+翻页」为一次调用，可达 0.6s/页。

详见：`references/cdp-scraping.md`

### terminal 不可用时的降级方案

当 `terminal` 工具无法正常工作时（如返回 WSL 乱码），**不要放弃测试**。改用 `execute_code` + Python `urllib` 进行全面的 HTTP 层面测试：

```python
import urllib.request, urllib.parse, re, http.cookiejar

base = 'http://localhost:8090'

# 1. 路由遍历测试 — 检查所有端点的 HTTP 状态码
# 2. 表单 POST 测试 — 模拟登录/注册/支付
# 3. 安全审计 — 检查 CSRF、安全响应头、XSS/SQLi 注入
# 4. Cookie/Session 管理 — 用 http.cookiejar 跟踪登录态
# 5. HTML 结构审查 — 正则提取链接、表单、脚本，检查缺失导航栏等

cj = http.cookiejar.CookieJar()
opener = urllib.request.build_opener(urllib.request.HTTPCookieProcessor(cj))
```

这种方式可以覆盖 80% 的 Web 测试需求（路由、表单、权限、安全），只是缺少截图和 JS 渲染后的 DOM 状态。
