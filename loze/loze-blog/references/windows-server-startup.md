# Windows Server Startup Workarounds

この Windows ホストでは `terminal` ツールが bash/MSYS 経由で実行されるため、WSL 層の干渉で `npm run dev` が正常に起動せず、出力も文字化けする。以下は代替の起動方法。

## 方法 1: ユーザーに手動起動を依頼する（最も確実）

```text
ダブルクリックで以下を実行してください：
  C:\Users\Loze\projects\loze-blog-vue\restart-frontend.bat  (Vue :5173)
  C:\Users\Loze\projects\loze-blog-vue\run-frontend.bat      (同上)
```

## 方法 2: execute_code から start コマンドで起動

```python
import subprocess

# Vue フロントエンド起動
subprocess.Popen(
    'start "VueBlog" cmd /c "cd /d C:\\Users\\Loze\\projects\\loze-blog-vue\\frontend && npm run dev"',
    shell=True
)

# Next.js 起動
subprocess.Popen(
    'start "NextBlog" cmd /c "cd /d C:\\Users\\Loze\\projects\\loze-blog-next && npm run dev -- -p 6708"',
    shell=True
)
```

## 方法 3: terminal から background=true で起動（不安定）

```bash
cd "C:/Users/Loze/projects/loze-blog-vue/frontend" && npx vite --host 2>&1
```

⚠ この方法は頻繁に即時終了（exit_code 1）し、WSL 文字化けでエラー内容が読めない。

## ポート確認（Python から）

```python
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.settimeout(2)
r = sock.connect_ex(('localhost', 5173))  # 5173=Vue, 6708=Next.js
sock.close()
print('OPEN' if r == 0 else 'CLOSED')
```
