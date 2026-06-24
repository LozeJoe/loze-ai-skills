# Next.js ↔ Vue 博客内容同步脚本

## 触发条件

向任一博客添加内容后，必须同步到另一个。

## 同步方向

通常：Next.js (:3000 MDX) → Vue (:5173 MySQL)。因为 Next.js 是"源"（Git 管理 MDX 文件），Vue 是"镜像"（数据库存储）。

## Python 同步脚本

```python
import os, re, subprocess, json
import pymysql

mysql_bin = "D:/xampp/mysql/bin/mysql.exe"
next_dir = "C:/Users/Loze/projects/loze-blog-next/content"

def parse_mdx(path):
    """解析 MDX frontmatter + body"""
    with open(path, 'r', encoding='utf-8') as f:
        content = f.read()
    fm_match = re.match(r'^---\s*\n(.*?)\n---', content, re.DOTALL)
    meta = {}
    if fm_match:
        for line in fm_match.group(1).split('\n'):
            if ':' in line:
                key, val = line.split(':', 1)
                meta[key.strip()] = val.strip().strip('"').strip("'")
    body = re.sub(r'^---\s*\n.*?\n---\s*\n', '', content, flags=re.DOTALL).strip()
    return meta, body

# 1. 读取 Next.js 内容
posts = []; diaries = []; notes = []
for sub, lst in [('posts', posts), ('diary', diaries), ('notes', notes)]:
    d = os.path.join(next_dir, sub)
    if not os.path.exists(d): continue
    for f in os.listdir(d):
        if f.endswith('.mdx'):
            meta, body = parse_mdx(os.path.join(d, f))
            meta['slug'] = f.replace('.mdx', '')
            meta['body'] = body
            lst.append(meta)

# 2. 清空 Vue 数据库
for tbl in ['t_post', 't_diary', 't_note']:
    subprocess.run([mysql_bin, '-u', 'root', 'loze_blog', '-e', f'DELETE FROM {tbl}'])

# 3. 插入
conn = pymysql.connect(host='localhost', user='root', password='', database='loze_blog', charset='utf8mb4')
cur = conn.cursor()

for p in posts:
    tags = p.get('tags', '').replace('[', '').replace(']', '').replace('"', '')
    cur.execute("""INSERT INTO t_post (slug, title, content, tags, category, description, date, author_id, draft)
        VALUES (%s, %s, %s, %s, %s, %s, %s, 1, 0)""",
        [p['slug'], p.get('title'), p['body'][:65000], tags,
         p.get('category', '未分类'), p.get('description', '')[:500],
         p.get('date', '2026-06-01') + ' 00:00:00'])

for d in diaries:
    cur.execute("""INSERT INTO t_diary (slug, title, content, mood, weather, date, user_id)
        VALUES (%s, %s, %s, %s, %s, %s, 1)""",
        [d['slug'], d.get('title'), d['body'][:65000],
         d.get('mood', ''), d.get('weather', ''), d.get('date', '') + ' 00:00:00'])

for n in notes:
    cur.execute("""INSERT INTO t_note (slug, title, content, date, user_id)
        VALUES (%s, %s, %s, %s, 1)""",
        [n['slug'], n.get('title'), n['body'][:65000], n.get('date', '') + ' 00:00:00'])

conn.commit(); conn.close()
print(f"Done: {len(posts)} posts, {len(diaries)} diaries, {len(notes)} notes")
```

## 同步后验证

```python
import urllib.request, json

# 确认 API 可访问
r = urllib.request.Request("http://localhost:8088/api/posts?size=20")
data = json.loads(urllib.request.urlopen(r).read())
print(f"API: {len(data['data']['posts'])} posts")

# 更新 data.json 备用
fallback = {"posts": data['data']['posts'], "entries": [], "notes": []}
with open("C:/Users/Loze/projects/loze-blog-vue/frontend/public/data.json", 'w') as f:
    json.dump(fallback, f, ensure_ascii=False)
```

## 注意事项

- MySQL `content` 字段为 `LONGTEXT`，但 Python 插入时建议截断 65000 字符
- 同步后必须确保 Vue 后端 :8088 在运行，否则前端走 `data.json` fallback
- `data.json` 也要同步更新（作为后端不可用时的备用数据源）
