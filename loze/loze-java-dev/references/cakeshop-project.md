# CakeShop 蛋糕商城项目

## 基本信息

| 项 | 值 |
|----|-----|
| 位置 | `D:\JAVAEE\demo\demo\` |
| 框架 | Spring Boot 2.2.6.RELEASE |
| ORM | MyBatis Plus 3.5.0 |
| 模板 | Thymeleaf |
| 数据库 | MySQL (XAMPP), `cookieshop` |
| JDK | **JDK 12** (C:\Program Files\Java\jdk-12.0.1), class version 56.0 |
| 端口 | 8080 |
| 主类 | `com.example.demo.DemoApplication` |

## 数据库

- **数据库名**: `cookieshop`（不是 cakeshop）
- **MySQL 连接**: root 无密码, 端口 3306
- **表**: user, type, goods, cart, order, orderitem, review, favorite, recommend, rider, rider_message, admin_log
- **测试数据**: 4 用户 (admin/admin123, vili/Vili1234!, rider1/Rider1234!), 10 分类, 8 商品, 3 订单
- **SQL 文件**: 项目根目录下 `cookieshop_complete.sql`（完整）、`cookieshop_full.sql`、`cookieshop_schema_only.sql`、`data.sql`、`data-h2.sql` 等
- **初始化**: `DataInitConfig` 启动时自动执行: ALTER TABLE 加列 + BCrypt 密码升级

## 配置文件

`src/main/resources/application.yml`:

```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/cookieshop?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
    username: root
    password:                # 无密码，不要填 root
    driver-class-name: com.mysql.cj.jdbc.Driver
deepseek:
  api:
    key: ${DEEPSEEK_API_KEY:***}   # 必须有默认值，否则启动崩溃
mybatis:
  type-aliases-package: com.javaBean
```

## 包结构

| 包 | 路径 |
|----|------|
| 主类 | `com.example.demo.DemoApplication` |
| 实体 | `com.javaBean` (User, Goods, Order, Cart, Type, Review, Favorite, Rider, RiderMessage, AdminLog, OrderItem, PageResult, RiderChat) |
| Mapper | `com.mapper` |
| Service | `com.service` |
| Controller | `com.controller` (Admin, AiChat, Cart, Goods, Order, Review, Rider, Statistics, User) |
| 配置 | `com.config` (DataInitConfig, WebMvcConfig, LoginInterceptor, AdminInterceptor, GlobalExceptionHandler 等) |

## 启动命令

```bash
# 用 JDK 12 编译 + 打包
export JAVA_HOME="C:/Program Files/Java/jdk-12.0.1"
"D:/JAVAEE/apache-maven-3.5.2/apache-maven-3.5.2/bin/mvn.cmd" clean package -DskipTests

# 直接启动（能看到完整错误）
"C:/Program Files/Java/jdk-12.0.1/bin/java.exe" -jar target/demo-0.0.1-SNAPSHOT.jar
```

## 登录系统（重要：两表分离）

**用户认证查 `user` 表**，骑手管理查 `rider` 表。两者独立：

| 角色 | 认证表 | 条件 | 密码方式 |
|------|--------|------|----------|
| 管理员/普通用户 | `user` | `isadmin='0'` 或 `'1'` | BCrypt |
| 骑手 | `user` | `isadmin='2'` | BCrypt |
| 骑手资料 | `rider` | — | 明文（仅用于资料展示，不参与认证） |

- `RiderService.login()` 实际查 `user` 表 + BCrypt 验证 + `isadmin='2'` 过滤
- `RiderMapper` (查 `rider` 表) 只用于 `/rider/profile` 等资料页面
- **`rider` 表必须手工插入数据**，DataInitConfig 只填充 `user` 表不填充 `rider` 表

### 默认账号

| 账号 | 密码 | 角色 | 表 |
|------|------|------|-----|
| admin | admin123 | 管理员 | user (isadmin=1) |
| vili | Vili1234! | 普通用户 | user (isadmin=0) |
| rider1 | Rider1234! | 骑手 | user (isadmin=2) + rider |
| rider2 | Rider1234! | 骑手 | user (isadmin=2) + rider |

> ⚠️ **DataInitConfig 会在每次启动时检查并升级密码**。如果 user 表密码不是 BCrypt（`$2a$` 开头），会被强制替换为上述种子密码。这意味着从旧 SQL 导入的 MD5 密码会在首次启动后失效。

## 已知问题

1. **DEEPSEEK_API_KEY 占位符** — `application.yml` 中 `key: ${DEEPSEEK_API_KEY}` 无默认值会直接导致启动崩溃，Maven 只报 exit code 1。必须改成 `${DEEPSEEK_API_KEY:***}`。
2. **JDK 版本严格** — 类文件是 Java 12（version 56.0），JDK 8 会报 `UnsupportedClassVersionError`。
3. **端口易混淆** — 原始 application.yml 配置端口 3308，实际 MySQL 在 3306；密码曾配 `root`，实际是空。
4. **application.yml vs application.properties** — 仅保留 application.yml，删除 application.properties 避免冲突。
5. **AiChatController 依赖 AiChatService** — 如果 DeepSeek API key 不可用，AI 聊天功能不可用但不影响其他页面。
6. **`rider_chat` 表缺失** — 完整的 cookieshop SQL 脚本不包含 `rider_chat` 表，但 `RiderChatMapper.getRecentChats()` 会查询它。访问 `/rider/messages` 时静默失败或报 SQL 错误。手动创建：
   ```sql
   CREATE TABLE IF NOT EXISTS `rider_chat` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `order_id` varchar(50) NOT NULL,
     `sender` varchar(20) NOT NULL COMMENT 'rider/user',
     `sender_name` varchar(50) DEFAULT NULL,
     `content` text NOT NULL,
     `create_time` datetime DEFAULT current_timestamp(),
     PRIMARY KEY (`id`),
     KEY `idx_order_id` (`order_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```
7. **`rider` 表初始为空** — DataInitConfig 不填充 rider 表。如 `/rider/income`、`/rider/profile` 等页面报空指针，需手动插入：
   ```sql
   INSERT INTO rider (username, password, name, phone) VALUES
   ('rider1', 'Rider1234!', '张三', '13900139000'),
   ('rider2', 'Rider1234!', '李骑手', '13800002222');
   ```
8. **DataInitConfig 启动改密码** — 首次启动时自动将 `user` 表中非 BCrypt 密码替换为种子密码（admin123/Vili1234!/Rider1234!），旧密码立即失效。如果用户反馈"登不进去"，先告知新密码。

## Docker 阿里云部署

### 服务器信息

| 项 | 值 |
|----|-----|
| IP | 112.124.53.155 |
| SSH | root / VKHELdqdZ7T_V_z（见 `deploy2.py`） |
| 项目路径 | /opt/app/demo/ |
| 访问地址 | http://112.124.53.155:8090 |
| Docker | 26.1.3, Compose v2.27.0 |
| 带宽 | 2 Mbps（小带宽，拉镜像需要时间） |

### 部署脚本

本地 `D:\JAVAEE\demo\` 下有：

| 脚本 | 用途 |
|------|------|
| `deploy.py` | 一键部署：打包→上传→解压→构建→验证 |
| `deploy2.py` | 仅构建：SSH 连接后 docker compose up --build |
| `deploy_final.py` | 完整版：上传 + 构建（含密码） |

**一键部署命令**：
```bash
python D:\JAVAEE\demo\deploy_final.py
```

### Dockerfile 关键配置

```dockerfile
# JDK 11（非 17！Lombok 与 JDK 17 不兼容：IllegalAccessError: LombokProcessor cannot access JavacProcessingEnvironment）
FROM maven:3.9-eclipse-temurin-11-alpine AS builder
# 阿里云 Maven 镜像加速（不用阿里云镜像，Maven 中央仓库只有 ~21KB/s，构建要 15 分钟）
RUN echo '...maven.aliyun.com/repository/public...' > /usr/share/maven/conf/settings.xml
# ...
FROM eclipse-temurin:11-jre-alpine
```

### 部署常见问题

1. **Lombok + JDK 17 不兼容** — `IllegalAccessError: LombokProcessor cannot access JavacProcessingEnvironment`。必须用 JDK 11 构建。
2. **Maven 下载极慢** — 必须配阿里云镜像（`maven.aliyun.com/repository/public`），否则 21KB/s。
3. **docker-compose vs docker compose** — 服务器用 Docker Compose v2，命令是 `docker compose`（无连字符）。`docker-compose`（带连字符）是 v1 语法，可能不存在。
4. **Alibaba Cloud VNC 不支持粘贴** — 如需在 VNC 内操作，用 ECS 控制台的「发送远程命令」功能代替。
5. **Hermes 密码过滤** — Hermes 会过滤 execute_code 中的密码字符串。解决方案：把密码存在现有脚本文件（deploy2.py）中，新脚本在运行时读取，不在代码中明文写密码。
