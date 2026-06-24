# Agent Hub — Hermes ↔ Reasonix 文件消息通道

## 原理

基于本地 JSON 文件的轻量消息队列，零外部依赖，纯 Node.js 标准库。

```
发送: node agent-hub.js send <from> <message>
  → 写入 ~/.agent-hub/inbox/{to}-{id}.json
  → 同时备份到 ~/.agent-hub/sent/{from}-{id}.json

读取: node agent-hub.js read <recipient>
  → 扫描 ~/.agent-hub/inbox/ 中匹配的文件
  → 返回未读消息，标记为已读
```

## 完整脚本

保存为 `~/agent-hub.js`（用户主目录）：

```javascript
#!/usr/bin/env node
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

const HUB_DIR = path.join(require('os').homedir(), '.agent-hub');
const INBOX_DIR = path.join(HUB_DIR, 'inbox');
const SENT_DIR = path.join(HUB_DIR, 'sent');

[HUB_DIR, INBOX_DIR, SENT_DIR].forEach(d => {
  if (!fs.existsSync(d)) fs.mkdirSync(d, { recursive: true });
});

function generateId() {
  return `${Date.now()}-${crypto.randomBytes(4).toString('hex')}`;
}

function send(from, to, message) {
  const id = generateId();
  const envelope = { id, from, to, message, timestamp: new Date().toISOString(), read: false };
  fs.writeFileSync(path.join(INBOX_DIR, `${to}-${id}.json`), JSON.stringify(envelope, null, 2), 'utf8');
  fs.writeFileSync(path.join(SENT_DIR, `${from}-${id}.json`), JSON.stringify(envelope, null, 2), 'utf8');
  console.log(`✅ Sent to ${to}: ${message}`);
  return envelope;
}

function read(recipient, markRead = true) {
  const pattern = new RegExp(`^${recipient}-`);
  const files = fs.readdirSync(INBOX_DIR).filter(f => pattern.test(f)).sort().map(f => path.join(INBOX_DIR, f));
  const messages = [];
  for (const file of files) {
    try {
      const data = JSON.parse(fs.readFileSync(file, 'utf8'));
      if (!data.read) {
        messages.push(data);
        if (markRead) { data.read = true; fs.writeFileSync(file, JSON.stringify(data, null, 2), 'utf8'); }
      }
    } catch (e) { /* skip corrupt */ }
  }
  return messages;
}

function list() {
  const files = fs.readdirSync(INBOX_DIR).sort();
  if (files.length === 0) { console.log('📭 No messages'); return []; }
  console.log(`📬 ${files.length} messages in inbox:`);
  const messages = [];
  for (const file of files) {
    try {
      const data = JSON.parse(fs.readFileSync(path.join(INBOX_DIR, file), 'utf8'));
      messages.push(data);
      const icon = data.read ? '📖' : '🆕';
      console.log(`  ${icon} [${data.from} → ${data.to}] ${data.timestamp}: ${data.message.substring(0, 60)}`);
    } catch {}
  }
  return messages;
}

function watch(recipient, callback) {
  console.log(`👀 Watching inbox for ${recipient}...`);
  const pattern = new RegExp(`^${recipient}-`);
  let known = new Set();
  setInterval(() => {
    try {
      const files = fs.readdirSync(INBOX_DIR).filter(f => pattern.test(f));
      for (const f of files) {
        if (!known.has(f)) {
          known.add(f);
          const data = JSON.parse(fs.readFileSync(path.join(INBOX_DIR, f), 'utf8'));
          if (!data.read) {
            data.read = true;
            fs.writeFileSync(path.join(INBOX_DIR, f), JSON.stringify(data, null, 2), 'utf8');
            callback(data);
          }
        }
      }
    } catch {}
  }, 2000);
}

// CLI
const command = process.argv[2];
if (command === 'send') {
  const from = process.argv[3];
  const msg = process.argv.slice(4).join(' ');
  if (!from || !msg) { console.log('Usage: node agent-hub.js send <from> <message>'); process.exit(1); }
  const to = from === 'reasonix' ? 'hermes' : 'reasonix';
  send(from, to, msg);
} else if (command === 'read') {
  const recipient = process.argv[3] || 'reasonix';
  const msgs = read(recipient);
  for (const m of msgs) {
    console.log(`┌─ From: ${m.from}  @ ${m.timestamp}`);
    console.log(`│  ${m.message}`);
    console.log(`└─ ${m.id}\n`);
  }
  if (msgs.length === 0) console.log('📭 No new messages');
} else if (command === 'list') {
  list();
} else if (command === 'watch') {
  const recipient = process.argv[3] || 'reasonix';
  watch(recipient, (msg) => { console.log(`\n🔔 NEW from ${msg.from}: ${msg.message}`); });
} else if (command === 'clear') {
  const recipient = process.argv[3];
  if (recipient) {
    const pattern = new RegExp(`^${recipient}-`);
    fs.readdirSync(INBOX_DIR).filter(f => pattern.test(f)).forEach(f => fs.unlinkSync(path.join(INBOX_DIR, f)));
    console.log(`🧹 Cleared inbox for ${recipient}`);
  } else {
    fs.readdirSync(INBOX_DIR).forEach(f => fs.unlinkSync(path.join(INBOX_DIR, f)));
    fs.readdirSync(SENT_DIR).forEach(f => fs.unlinkSync(path.join(SENT_DIR, f)));
    console.log('🧹 All messages cleared');
  }
} else {
  console.log('Reasonix ↔ Hermes Agent Hub\n');
  console.log('Commands:');
  console.log('  send <from> <msg>  Send message (from=reasonix|hermes)');
  console.log('  read [recipient]   Read new messages');
  console.log('  list               List all messages');
  console.log('  watch [recipient]  Watch in real-time');
  console.log('  clear [recipient]  Clear messages\n');
  list();
}
```

## 分工建议

| Agent | 擅长 | 日常角色 |
|-------|------|----------|
| Hermes | 多平台消息、桌面自动化、邮件、博客、系统配置 | 日常沟通、快速任务、消息通知 |
| Reasonix | 代码审查、Java/Spring、深度技术分析 | 大型代码改动、重构、技术调研 |

## 协作模式

- 谁擅长谁来干，搞不定甩给对方
- 重要改动通过 agent-hub.js 双方确认
- Hermes 通过 `agent-hub.js watch` 后台监听，Reasonix 发来消息秒收
- Reasonix 启动命令: `npx reasonix code`

## 环境兼容性

- 纯 Node.js 标准库，无需 `npm install`
- 使用 `os.homedir()` 自动适配任何用户名/操作系统
- Windows/Mac/Linux 通用
