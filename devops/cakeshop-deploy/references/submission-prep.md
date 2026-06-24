# CakeShop 考核提交准备

## 提交路径
`C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop\`

## 目录结构（最终提交版）

```
CakeShop/
├── src/                    源码 (含32个测试 + AI模块)
├── ai-traces/              AI参与痕迹 (31个文件)
│   ├── AI-DEVELOPMENT.md   AI协作总记录 (含技能清单)
│   ├── deploy*.py ×12      部署脚本迭代全记录
│   ├── 178*.txt ×6         Reasonix CLI 会话日志
│   ├── 01~10-*.png ×10     项目截图
│   └── 截图替换指南.md + config.properties
├── sql/                    数据库
│   └── cookieshop_完整.sql 合并版 (表结构+升级+测试数据)
├── .mvn/                   Maven Wrapper
├── demo/                   路演素材
├── pom.xml + Dockerfile + docker-compose.yml + .env + settings.xml
└── README.md + geocode_snippet.txt
```

## 纳入清单（项目运行 + 考核必须）

| 类别 | 内容 | 说明 |
|------|------|------|
| 源码 | `src/` | 含 main + test，AiChatController等AI模块和32个测试均已包含 |
| AI痕迹 | `ai-traces/` | 31个文件：协作记录、部署脚本迭代、Reasonix日志、项目截图 |
| 数据库 | `sql/cookieshop_完整.sql` | 合并版：表结构 + ALTER升级 + 测试数据 |
| 构建 | `pom.xml`, `mvnw`, `mvnw.cmd`, `.mvn/`, `settings.xml` | Maven 构建体系 |
| 部署 | `Dockerfile`, `docker-compose.yml`, `.env` | Docker 部署 |
| 演示 | `demo/` | init.sql, logo.jpg, cakeshop-roadshow.html, config.properties |
| 文档 | `README.md`, `geocode_snippet.txt` | 项目说明 + 代码片段 |

## 排除清单（无关文件）

| 类别 | 排除项 | 原因 |
|------|--------|------|
| IDE配置 | `.idea/`, `demo.iml` | 个人IDE配置，与运行无关 |
| 编译产物 | `.gradle/`, `build/`, `target/` | Maven/Gradle 编译输出，可重新生成 |
| Gradle体系 | `gradle/`, `build.gradle`, `settings.gradle`, `gradlew`, `gradlew.bat` | 项目主构建工具是 Maven |
| 开发笔记 | `BUG_REPORT.md`, `FIX_GUIDE.md`, `design-summary.md`, `read.md` | 开发过程文档，非考核要求 |
| Git配置 | `.gitattributes`, `.gitignore` | 版本控制配置 |

## 验证方法

提交前在目标目录执行：
```powershell
# 确认核心文件存在
Test-Path "C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop\pom.xml"
Test-Path "C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop\src\main"
Test-Path "C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop\src\test"
Test-Path "C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop\ai-traces\AI-DEVELOPMENT.md"
Test-Path "C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop\sql\cookieshop_完整.sql"

# 确认无编译产物污染
Get-ChildItem -Recurse -Directory | Where-Object { $_.Name -in @('.gradle','build','target','.idea') }
# 应返回空

# 统计（参考值: ~280 files, ~15.5 MB）
(Get-ChildItem -Recurse -File).Count
```
