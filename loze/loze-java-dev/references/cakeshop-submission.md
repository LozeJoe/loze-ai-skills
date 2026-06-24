# CakeShop 考核提交打包规范

> 适用场景：将 JavaEE 项目从开发目录打包到桌面提交文件夹，供学校考核使用。

## 提交目标路径

```
C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop\
```

## 源码路径

```
D:\JAVAEE\demo\CakeShop\
```

## 核心规则

1. **扁平结构**：所有文件直接放在项目根目录，不创建 `ai-traces/` 等嵌套子文件夹
2. **AI 痕迹必须包含**：不只是项目内的 AI 功能代码，还要有外部 AI 参与开发的证据
3. **排除无关文件**：编译产物、IDE 配置、冗余构建系统、开发笔记

## 打包清单

### ✅ 必须保留

| 类别 | 文件/目录 | 说明 |
|------|----------|------|
| 源码 | `src/` | 含 main + test（32 测试文件 + 6 AI 模块） |
| 构建 | `pom.xml`, `mvnw`, `mvnw.cmd`, `.mvn/`, `settings.xml` | Maven 全套 |
| 部署 | `Dockerfile`, `docker-compose.yml`, `.env` | Docker 部署 |
| 数据库 | `cookieshop*.sql` (5个), `test_order_data.sql` | 建表 + 测试数据 |
| 路演 | `demo/` | init.sql, logo, roadshow HTML, config.properties |
| 文档 | `README.md`, `geocode_snippet.txt` | 项目说明 |

### 🔥 AI 痕迹（必须包含）

| 类别 | 文件 | 来源 |
|------|------|------|
| AI 协作记录 | `AI-DEVELOPMENT.md` | `D:\JAVAEE\demo\.trae\` |
| Reasonix 日志 | `178*.txt` (6个) | `D:\JAVAEE\demo\.reasonix\truncated-results\` |
| 部署迭代 | `deploy*.py` (12个) | `D:\JAVAEE\demo\` |
| 项目截图 | `01~10-*.png` (10张) + `截图替换指南.md` | `D:\JAVAEE\demo\screenshots\` |

### ❌ 必须排除

| 类别 | 文件/目录 | 原因 |
|------|----------|------|
| 编译产物 | `.gradle/`, `build/`, `target/` | 可重新生成 |
| IDE 配置 | `.idea/`, `*.iml` | 个人环境相关 |
| Gradle 冗余 | `gradle/`, `build.gradle`, `settings.gradle`, `gradlew*` | 项目用 Maven |
| 开发笔记 | `BUG_REPORT.md`, `FIX_GUIDE.md`, `design-summary.md`, `read.md` | 非考核材料 |
| Git 配置 | `.gitattributes`, `.gitignore` | 非运行相关 |
| 空目录 | `docs/`, `sql/` (如为空) | 无实际内容 |

## 执行命令（PowerShell）

```powershell
$dest = "C:\Users\Loze\Desktop\2400130326刘哲凯\CakeShop"
$src = "D:\JAVAEE\demo\CakeShop"

# 1. 核心源码 + 构建
robocopy "$src\src" "$dest\src" /E
robocopy "$src\.mvn" "$dest\.mvn" /E
robocopy "$src\demo" "$dest\demo" /E
Copy-Item "$src\pom.xml", "$src\mvnw", "$src\mvnw.cmd", "$src\Dockerfile", "$src\docker-compose.yml", "$src\.env", "$src\settings.xml", "$src\README.md", "$src\geocode_snippet.txt", "$src\cookieshop*.sql", "$src\test_order_data.sql" "$dest\" -Force

# 2. AI 痕迹（扁平放入根目录）
Copy-Item "D:\JAVAEE\demo\.trae\AI-DEVELOPMENT.md" "$dest\" -Force
Copy-Item "D:\JAVAEE\demo\deploy*.py" "$dest\" -Force
Copy-Item "D:\JAVAEE\demo\.reasonix\truncated-results\*" "$dest\" -Force
Copy-Item "D:\JAVAEE\demo\screenshots\*" "$dest\" -Force
```

## 最终结构

```
CakeShop/
├── src/                     # 源码（含 32 测试 + 6 AI 模块）
├── .mvn/                    # Maven Wrapper
├── demo/                    # 路演素材
├── pom.xml                  # Maven 构建
├── Dockerfile               # Docker 镜像
├── docker-compose.yml       # 容器编排
├── cookieshop*.sql ×5       # 数据库脚本
├── deploy*.py ×12           # AI 辅助部署迭代全记录
├── 178*.txt ×6              # Reasonix CLI 会话日志
├── 01~10-*.png ×10          # 项目截图
├── AI-DEVELOPMENT.md        # AI 协作记录
├── 截图替换指南.md
└── README.md
```

## 关键坑点

- **不要创建 `ai-traces/` 子目录**：用户偏好所有文件扁平放在一个文件夹
- **SSH 密码不要泄露**：`deploy*.py` 中的 `PASSWORD="***"` 已脱敏处理
- **Reasonix 日志文件名**是 Unix 时间戳格式（`1781679512929-983f37d0-run_command.txt`），不要改名
- **`Move-Item` 在 PowerShell 中可能仅复制不移动**（目标已存在时），用 `Copy-Item` + 手动清理更可靠
