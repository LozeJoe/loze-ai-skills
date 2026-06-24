# delegate_task 实战经验

> 从本次博客项目多轮建设中学到的 delegate_task 使用技巧

## 有效模式

### 按功能域拆分（推荐）
将大型任务按文件/功能域拆分为 3 路并行 Agent：
```
Agent 1: 基础设施 (lib/auth, lib/types, API routes)
Agent 2: 页面组件 (layout, header, pages)
Agent 3: 内容+SEO (markdown, sitemap, RSS)
```

### 按页面/组件拆分
```
Agent 1: 核心基础设施 (API, 认证, 配置)
Agent 2: 前端展示 (首页, 列表页, 动画)
Agent 3: 功能页面 (弹窗, 管理, 编辑器)
```

## Context 编写要点

每个 Agent 的 context 必须包含：
1. **项目路径** — 绝对路径，Agent 不知道你在哪个目录
2. **已安装的依赖** — Agent 无法 `npm ls`，需要明确告诉
3. **共享的工具/类型** — 如果 Agent B 需要用到 Agent A 创建的文件，必须告知文件路径和导出
4. **文件结构约定** — 项目已有的命名规范、目录结构
5. **预期输出格式** — "把修改写到对应文件"、"用中文输出结果" 等

## 验证规则

Agent 返回后不要直接信任结果，必须验证：
- `agent said "wrote file X"` → `stat X` 确认文件存在且内容正确
- `agent said "fixed bug Y"` → `read_file` 确认修改正确
- 重要: Agent 的 summary 是自述报告，不是真理

## 常见失败模式

| 失败 | 原因 | 预防 |
|------|------|------|
| 文件未修改 | patch 匹配失败（行号/空格差异） | 用 write_file 而非 patch 创建新文件 |
| 模块未找到 | Agent 不知道其他 Agent 创建了模块 | context 中列出所有依赖文件路径 |
| 超时 | 任务太大或 API 调用慢 | 每个 Agent 任务不超过 10 个文件 |
| 类型冲突 | 多个 Agent 各自定义同名 interface | 先建共享 types 文件，告知所有 Agent |
| 构建失败 | Agent 删除了仍在使用的 import | 告知 "保持所有原有功能，只增加..." |
