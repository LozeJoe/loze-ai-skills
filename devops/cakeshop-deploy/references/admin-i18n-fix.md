# 管理后台 i18n 修复手册

## 问题现象
管理后台切换中文/EN后界面不完全翻译，仍显示部分中文。

## 三个根因

### 1. Controller 硬编码中文
AdminController 中 pageTitle 和 headerExtra 直接写死中文字符串：
- `pageTitle = "🛵 骑手管理"` → 英文环境不会变
- `modelAndView.addObject("headerExtra", "<a href='/admin/userEdit'>添加用户</a>")` → 同样硬编码
**修复**: commons.html header 改用 `th:text="#{...}"` 根据 filterType/unverified 选 key；按钮移到模板用 i18n

### 2. CookieLocaleResolver locale 简写不匹配
侧栏 `?lang=zh` → cookie=`zh` → 找 `messages_zh.properties`（不存在，只有 `messages_zh_CN.properties`）
**修复**: 创建 `messages_zh.properties`（复制自 messages_zh_CN.properties）

### 3. 两套 i18n 文件不同步
admin 模板从 `i18n/messages_*.properties` 子目录加载，与根目录是独立的
**修复**: 同步补全缺失 key：user.rider, user.roleRider, user.roleAdmin, user.roleUser, user.statusFrozen, user.statusNormal, user.confirmDelete, user.riderTitle, user.pendingTitle, user.allTitle
