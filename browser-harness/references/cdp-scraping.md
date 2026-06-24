# CDP WebSocket 直连批量采集

当 `browser-harness` CLI 不足以处理大规模翻页采集任务时，
直接用 Python websocket 连接 CDP 端口，注入 JavaScript 实现高速自动化。

## 前置条件

1. Edge/Chrome 以调试模式启动，**必须加 `--remote-allow-origins=*`**：
```bash
taskkill /F /IM msedge.exe
"/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe" --remote-debugging-port=9222 --remote-allow-origins=* "https://target-url"
```

2. Python 环境需 `websocket-client`：
```bash
pip install websocket-client
```

## 核心模式

### 连接 CDP
```python
import json, urllib.request, websocket

# 获取目标标签页
tabs = json.loads(urllib.request.urlopen("http://localhost:9222/json").read())
target = next(t for t in tabs if '关键词' in t.get('title', ''))
ws = websocket.create_connection(target['webSocketDebuggerUrl'])

# 封装 CDP 调用
def cdp(method, params=None, timeout=2):
    ws.send(json.dumps({"id": 1, "method": method, "params": params or {}}))
    ws.settimeout(timeout)
    results = []
    try:
        while True:
            results.append(json.loads(ws.recv()))
    except:
        pass
    return results
```

### 提取 + 翻页合并调用（关键优化）

将「提取当前页数据」和「点击翻页」合并为**一次 CDP 调用**，
减少网络往返，将每页耗时从 2s+ 降到 0.6s：

```python
extract_and_click = """
(function(){
    // 1. 提取当前页数据
    var text = document.body.innerText;
    // ... 解析逻辑 ...
    var data = {field1: '...', field2: '...'};

    // 2. 点击翻页按钮
    var els = document.querySelectorAll('*');
    for(var i=0; i<els.length; i++){
        if(els[i].textContent && els[i].textContent.trim() === '下一页'){
            els[i].click();
            break;
        }
    }
    return JSON.stringify(data);
})()"""

# 批量循环
for i in range(TOTAL_PAGES):
    ws.send(json.dumps({"id":1, "method":"Runtime.evaluate",
        "params":{"expression": extract_and_click, "returnByValue": True}}))
    time.sleep(0.6)  # 页面渲染等待
    ws.settimeout(0.8)
    try:
        resp = json.loads(ws.recv())
        data = resp['result']['result']['value']
        # 处理 data ...
    except:
        pass
```

## 常用 CDP 方法

| 方法 | 用途 |
|------|------|
| `Runtime.evaluate` | 注入并执行 JavaScript |
| `Network.enable` | 开启网络请求拦截 |
| `Network.getResponseBody` | 获取请求响应体 |
| `Fetch.enable` | 拦截 fetch/XHR 请求 |
| `Input.dispatchKeyEvent` | 模拟键盘输入 |
| `Page.reload` | 刷新页面 |
| `Page.navigate` | 导航到 URL |

## 适用场景

- SPA 页面数据封闭在 JS 模块内，无法从 window 全局访问
- API 需要登录态且 CORS 阻止外部调用
- 需要批量翻页采集（100+ 页）
- `browser-harness` CLI 逐页调用太慢

## Pitfalls

1. **必须 `--remote-allow-origins=*`**：否则 WebSocket 握手返回 403
2. **`execute_code` 有 300s 超时**：大批量采集需分批保存，每 50-100 条写一次文件
3. **提取和翻页必须合并**：分开两次 CDP 调用会让速度翻倍降低
4. **`ws.recv()` 可能收到多条消息**：网络事件会混入，需要用 `while True` + timeout 收集
5. **翻页后等待 0.5-1s**：SPA 需要渲染时间，太快会拿到旧数据
