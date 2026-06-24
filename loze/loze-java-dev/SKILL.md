---
name: loze-java-dev
description: "刘哲凯的 Java/JavaEE 全栈开发工作流 — Spring, SpringMVC, MyBatisPlus, Vue.js, Servlet"
version: 1.1.0
author: Hermes Agent (for Loze)
platforms: [windows]
metadata:
  hermes:
    tags: [Java, Spring, MyBatis, Vue, JavaEE, 刘哲凯]
    related_skills: [loze-ai-integration, loze-windows-automation]
---

# Loze Java Dev — 刘哲凯 Java 全栈开发技能

## 触发条件

用户提到 Java、Spring、MyBatis、Vue、Servlet、JSP、JavaEE、实验报告、项目开发、IDEA 相关内容时加载。

## 环境

| 工具 | 路径/版本 |
|------|----------|
| Java | JDK 1.8.0_112 (C:\Program Files\Java\jdk1.8.0_112) / JDK 12.0.1 (C:\Program Files\Java\jdk-12.0.1) — 优先用 JDK 12 |
| IntelliJ IDEA | C:\Users\Loze\Desktop\IntelliJ IDEA (快捷方式) |
| Eclipse | C:\Users\Loze\Desktop\Eclipse (快捷方式) |
| Maven | 3.5.2 (D:\JAVAEE\apache-maven-3.5.2\apache-maven-3.5.2\bin\mvn.cmd), ~/.m2/settings.xml 已配阿里云镜像 |
| Node.js | 24.14.0 (C:\Program Files\nodejs) |
| Tomcat | 通过 IDEA/Eclipse 管理 |
| MySQL | MariaDB 10.3.16 (Navicat 管理), root 无密码或 20060708, 端口 3306 |

## 项目结构

```
IdeaProjects/
  Demo02/                    # Java Web 项目 (Servlet + JSP)
    src/                     # Java 源码
    web/                     # Web 资源
      WEB-INF/web.xml        # Servlet 配置

Desktop/
  JAVAEE/                    # JavaEE 实验和项目文件
    实验一~十.doc             # Spring/SpringMVC/MyBatis 实验
    蛋糕商城项目实训二.doc     # 电商实战项目
    springAOPXML/            # Spring AOP 练习
    springTest/              # Spring 入门
    springmvc1/              # SpringMVC 入门
    springMVCTest/           # SpringMVC 练习
    mybatisPlus/             # MyBatisPlus 练习
    mybatisTest(4)/          # MyBatis 练习
    vue.3.2.27.js / vue2.6.14.js  # Vue 前端
    数据库及API接口/          # 数据库设计项目
```

## 技术栈速查

| 层 | 技术 | 说明 |
|----|------|------|
| 核心 | Spring Framework | IOC/DI, AOP |
| Web | SpringMVC | RESTful API, 拦截器 |
| ORM | MyBatis / MyBatisPlus | CRUD, 分页, 条件查询 |
| 前端 | Vue.js 2.x/3.x | 渐进式框架 |
| 数据库 | MySQL | Navicat 管理 |
| 构建 | Maven (IDEA内置) | 依赖管理 |
| 容器 | Tomcat | Web 部署 |

## 常用操作

### 快速创建 Spring Boot 项目
```bash
# 使用 Spring Initializr (IDEA 内)
# File -> New -> Project -> Spring Initializr
# 选择: Spring Web, MyBatis Framework, MySQL Driver
```

### Maven 依赖管理
```bash
# 在 IDEA Terminal 中执行
mvn clean install
mvn dependency:resolve
```

### Vue 前端集成
```html
<!-- 直接引入 Vue CDN (无需构建工具) -->
<script src="vue.2.6.14.js"></script>
```

### MyBatisPlus 代码生成器快速模板
```java
// 典型实体类
@Data
@TableName("t_user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String username;
    private String password;
}
```

## 自动化脚本

项目配套脚本在 `~/scripts/`：

| 脚本 | 用途 |
|------|------|
| `scaffold.py` | Java 项目脚手架：`python ~/scripts/scaffold.py 项目名 com.example.pkg --type spring\|servlet\|mybatis` |
| `sqlgen.py` | 实体类→SQL：`python ~/scripts/sqlgen.py User.java` 或 `--dir src/entity/` 批量生成 |

## Vue 3 + Spring Boot 全栈项目模板

当用户需要创建 Vue 3 前端 + Spring Boot 后端的全栈项目时，使用以下标准结构。

### 项目结构

```
项目名/
├── backend/                     Spring Boot 2.7.x (兼容 Java 8)
│   ├── pom.xml                  Spring Boot 2.7.18 + MyBatisPlus 3.5.5 + jjwt 0.9.1
│   ├── src/main/java/com/loze/blog/
│   │   ├── BlogApplication.java
│   │   ├── config/WebConfig.java      CORS + 拦截器 + 静态资源
│   │   ├── interceptor/AuthInterceptor.java  JWT 验证
│   │   ├── utils/JwtUtil.java
│   │   ├── entity/                    @Data + @TableName
│   │   ├── mapper/                    extends BaseMapper<T>
│   │   ├── service/                   接口
│   │   └── controller/               @RestController
│   └── src/main/resources/
│       ├── application.yml
│       └── schema.sql
├── frontend/                    Vue 3 + Vite + Pinia + Vue Router + Tailwind
│   ├── src/
│   │   ├── router/index.ts          路由 + beforeEach 守卫
│   │   ├── stores/auth.ts           Pinia (token/localStorage)
│   │   ├── stores/theme.ts          暗色模式切换
│   │   ├── api/index.ts             axios + 拦截器
│   │   ├── views/                   页面组件
│   │   └── components/              通用组件
│   ├── vite.config.ts               proxy /api -> localhost:8080
│   └── tailwind.config.js
└── README.md
```

### Java 8 兼容性约束

用户环境是 Java 1.8（Oracle JDK 1.8.0_112），必须使用兼容版本：
- Spring Boot: 2.7.18（3.x 需要 Java 17+）
- MyBatisPlus: 3.5.5
- jjwt: 0.9.1（需要额外 jaxb-api 依赖）
- 不能使用 records、text blocks、var（均是 Java 14+ 特性）
- 用 Lombok `@Data` 替代手动 getter/setter

### MySQL 连接

用户 MySQL 运行在 `localhost:3306`，root 密码 `20060708`，数据库 `loze_blog`：
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/loze_blog?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: 20060708
```

### 关键代码模式

**JWT 登录流程**：POST /api/admin/login -> 验证用户名密码 -> 生成 Token -> 返回给前端 -> 前端存 localStorage -> axios 拦截器附加 Authorization header -> 后端 AuthInterceptor 验证。

**MyBatisPlus 逻辑删除**：所有表加 `deleted TINYINT DEFAULT 0`，实体加 `@TableLogic`。

**Vue 路由守卫**：`router.beforeEach` 检查 `localStorage.getItem('token')`，无 token 跳转登录页。

### 后端 API 路由约定

| 路径模式 | 说明 |
|----------|------|
| GET /api/posts | 公开接口（分页 + 筛选） |
| GET /api/admin/posts | 管理接口（需 JWT） |
| POST /api/admin/write | 创建文章/日记/笔记（统一入口，body.type 区分） |

### 已知可实现的项目

`C:\\Users\\Loze\\projects\\loze-blog-vue` — 已完成的 Vue 3 + Spring Boot 博客项目（45 Java 文件 + 16 Vue 文件），可作为模板参考。

### 相关参考文档

- `references/vue-glass-morphism.md` — 液态玻璃多层 z-index 实现、Tailwind CSS 变量桥接、Vite public/ 目录视频处理、axios 后端不可用回退模式。

## 已知坑点

1. **本机文件工具走 WSL 层** — `write_file` 和 `terminal` 工具可能返回 WSL 更新提示乱码而失败。此时使用 `execute_code` 中的 Python `open()` / `subprocess` 作为替代。创建大量文件时直接用 execute_code 批量 `open().write()` 更可靠。
2. **npm/mvn 在 execute_code 中找不到** — `subprocess.run(['npm', ...])` 会报 FileNotFoundError。需要使用 `['cmd', '/c', 'npm', ...]` 或完整路径如 `C:\Program Files\nodejs\npm.cmd`。
3. **启动持久进程（dev server）** — 使用 `subprocess.Popen(..., creationflags=subprocess.CREATE_NEW_CONSOLE)` 配合批处理文件启动，避免进程随脚本结束被杀。
4. **Java 1.8 版本限制** — 不能使用 records、text blocks、var、switch expressions 等 Java 14+ 特性。Spring Boot 锁定 2.7.18，MyBatisPlus 锁定 3.5.5。用 Lombok `@Data` 替代手动 getter/setter。
5. **JAVAEE 项目嵌套目录** — `Desktop/JAVAEE/` 下的项目有两层嵌套（如 `mybatisPlus/mybatisPlus/src/…`）。git init 在项目根目录，但源码路径需要穿透内层。
6. **老式 Java 字段风格** — 部分项目使用 package-private 字段（无 `private` 关键字）+ Tab 缩进，与标准 Lombok + `private` 风格不同。`sqlgen.py` 已适配两种写法。
7. **项目混合 Eclipse + IDEA 格式** — 同时存在 `.classpath`/`.project`（Eclipse）和 `.idea/`/`.iml`（IDEA），`.gitignore` 已覆盖两者。
8. **Vite 项目 `@` 别名** — 创建 Vue 3 + Vite 项目后，必须在 `vite.config.ts` 显式配置别名，否则所有 `import xxx from '@/views/...'` 报 "Failed to resolve import"：
```ts
import { fileURLToPath, URL } from 'node:url'
export default defineConfig({
  resolve: { alias: { '@': fileURLToPath(new URL('./src', import.meta.url)) } },
  // ...
})
```

9. **Vue SPA 页面 HTML 检查无用** — Vue 客户端渲染的初始 HTML 只有 `<div id="app"></div>`，所有内容由 JS 动态生成。用 `urllib` 抓取首页看不到任何渲染结果。调试应检查 Vite 终端输出、浏览器 Console (F12)、或 Vite 编译错误 overlay。  
10. **前端独立运行时需要后端回退** — 当 Spring Boot 未启动时，Vue 前端应能展示静态内容。方案：axios 响应拦截器捕获网络错误 → 读取 `public/data.json` → 返回模拟 API 响应。数据文件格式 `{"posts": [...], "entries": [...], "notes": [...], "photos": [...]}`。详见 `references/vue-glass-morphism.md`。

11. **Vite public/ 大文件 500 错误** — 照片/视频（>1MB）在 public/ 目录下返回 HTTP 500。原因是 Vite 默认将静态文件当作 JS 模块处理。修复：`vite.config.ts` 中设置 `build: { assetsInlineLimit: 0 }`，且 Vue 模板中媒体文件必须用 `:src="variable"` 绑定（而非 `src="/path/file.mp4"` 直接字符串），避免被 Vue 编译器转为模块导入。

12. **Tailwind JIT 清除自定义 CSS 类** — `bg-bg`、`text-text`、`glass`、`nav-glass` 等自定义类名不在 Tailwind content 扫描范围内，构建时会被 purged。修复：在 `tailwind.config.js` 添加 `safelist: ['bg-bg', 'text-text', 'bg-card', 'border-border', 'glass', 'glass-hover', 'nav-glass', ...]` 或使用 `style` 内联绑定绕过 Tailwind。

13. **Vue SPA 初始 HTML 无内容** — 用 HTTP 抓取页面只能看到 `<div id="app"></div>` + `<script>` 标签。所有内容由 JS 动态渲染。确认页面是否工作的唯一方式是：
    - 浏览器 F12 Console 查看 JS 错误
    - Vite 终端编译错误
    - 临时创建纯 inline style 的最小测试组件验证 Vue 本身工作（排除 CSS/Tailwind 问题）

14. **backdrop-filter 液态玻璃需要正确的 z-index 层级** — `backdrop-filter: blur()` 只模糊元素**后面**的内容。如果父容器有纯色背景（如 `bg-bg`），玻璃卡片看不到视频/纹理，blur 无效果。正确架构：
    - z-0: 视频/纹理背景
    - z-1: 半透明颜色叠加层（确保文字可读）
    - z-10: 内容区（透明背景）+ 玻璃卡片
    - 玻璃卡片 `.glass { background: rgba(255,255,255,0.72); backdrop-filter: blur(16px); }`
    - 根容器移除 `bg-bg`，改为在叠加层上控制背景色

15. **API 双重回退模式** — axios 响应拦截器回退 + Vue 组件内 `fetch('/data.json')` 直接回退。axios 拦截器只在网络错误（`!err.response`）时触发，如果后端返回 404/500 则不触发。因此在关键页面（BlogDetail, BlogList, Home）必须在 `catch` 块中添加 `fetch('/data.json')` 作为第二层保障。

16. **MDX 内容迁移到 JSON** — 从原 Next.js 博客的 MDX 文件提取内容时，frontmatter 中的 `tags: [xxx, yyy]` 可能被错误解析为单个字符串 `'[\"xxx\", \"yyy\"]'`。需要对 tags 做二次清洗：`JSON.parse` 尝试解析，失败则手动 `strip('[]').split(',')`。

17. **Inline style 绑定优先于 Tailwind 自定义类** — 当 Tailwind JIT 反复清除自定义 CSS 变量类（`bg-bg`、`text-text` 等），最可靠的方案是在 Vue 模板中直接使用 `style="color: var(--text)"` 内联绑定。这绕过 Tailwind 的 purge 机制，CSS 变量由 `:root` 定义保证生效。

18. **Java 字符串中的中文标点导致编译错误** — `response.getWriter().write(\"{\\\"success\\\":false,\\\"error\\\":\\\"未授权，请先登录\\\"}\");` 中如果误用全角逗号 `，`(U+FF0C) 或引号嵌套未转义，Java 编译器报 \"不是语句\"/\"需要';'\"。JSON 字符串内必须使用 ASCII 标点：逗号 `,` 冒号 `:` 引号必须 `\\\"` 转义。Maven 编译失败的典型信号：`mvn compile` 后 target/classes/ 目录没有 .class 文件。

19. **Maven 编译必须设 JAVA_HOME 指向 JDK** — `mvn` 命令在 execute_code 沙箱中可能报 `No compiler is provided in this environment`。原因是 JAVA_HOME 未设置或指向 JRE 而非 JDK。修复：启动 Maven 前在 subprocess 的 `env` 中传入 `JAVA_HOME=r'C:\\Program Files\\Java\\jdk-12.0.1'`。JDK 8 不支持 `--release` 标志，优先用 JDK 12。

20. **Maven 路径嵌套** — 实际 `mvn.cmd` 路径是 `D:\\JAVAEE\\apache-maven-3.5.2\\apache-maven-3.5.2\\bin\\mvn.cmd`（两层 `apache-maven-3.5.2` 嵌套）。另有 `D:\\JAVAEE\\apache-maven-3.8.9\\apache-maven-3.8.9\\bin\\mvn.cmd` 备选。

21. **Spring Boot 启动秒退诊断流程** — 编译成功但 `spring-boot:run` 或 `java -jar` 后进程秒退不给响应。按优先级排查：

    **(a) `${ENV_VAR}` 占位符无默认值** — `application.yml` 或 `@Value` 中引用 `${DEEPSEEK_API_KEY}` 等环境变量但未定义默认值时，Spring Boot 直接崩溃。Maven 只报 "Application finished with exit code: 1"，看不到真实原因。**必做**：先用 `java -jar target/*.jar` 直接启动（绕过 Maven），会看到完整错误 "Could not resolve placeholder 'XXX'"。修复：给占位符加默认值 `${VAR:***}` 或 `${VAR:}`。

    **(b) JDK 版本不匹配** — 报 `UnsupportedClassVersionError: class file version 56.0 … only recognizes up to 52.0`。版本号对照：52=Java8, 55=Java11, 56=Java12, 61=Java17。用 `mvn clean compile` 以正确 JDK 重编，或切换到匹配的 JRE 运行。

    **(c) `application.yml` 和 `application.properties` 共存冲突** — Spring Boot 会合并两者。如果 properties 写了一套数据库配置、yml 写了另一套（不同端口/密码），最终行为不可预测。**检查 src/main/resources/ 下是否两个文件同时存在**，选择保留一个。

    **(d) 数据库连接参数错误** — 端口（3306 vs 3308）、密码（空 vs root）、数据库名不一致。用 Python `socket` 验证端口 + `mysql -u root -e 'SHOW DATABASES'` 确认。

    **(e) H2 + MySQL 驱动共存冲突** — pom.xml 同时有 h2 和 mysql-connector-java 时，Spring Boot 自动配置可能错误选择 H2。需显式指定 `spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver`。

22. **CakeShop 蛋糕商城项目** — 源码位置 `D:\\JAVAEE\\demo\\CakeShop\\`（已从 demo/demo 改名，artifactId=CakeShop）；考核提交版 `C:\\Users\\Loze\\Desktop\\2400130326刘哲凯\\CakeShop\\`（已排除 IDE/构建产物/Gradle/开发笔记）。Spring Boot 2.2.6 + MyBatis Plus 3.5.0 + Thymeleaf + MySQL。数据库名 `cookieshop`（非 cakeshop），MySQL root 无密码，端口 3306。**必须 JDK 12 编译运行**（class version 56.0），JDK 8 无法启动。`application.yml` 位于 `src/main/resources/`，包含 deepseek API 集成（需要 `${DEEPSEEK_API_KEY}` 占位符）。DataInitConfig 启动时自动执行 DB 迁移（ALTER TABLE 添加列）和密码 BCrypt 升级。SQL 文件在项目根目录（`cookieshop_complete.sql` 等）。详见 `references/cakeshop-project.md`。

    **考核提交打包规则**（适用于所有学校项目）：
    - **必须包含的 AI 痕迹**：`.trae/AI-DEVELOPMENT.md`、`.reasonix/truncated-results/*.txt`（Reasonix 会话日志）、`deploy*.py`（AI 辅助部署迭代全记录，保留所有版本）、`screenshots/`（项目截图）
    - **扁平结构**：AI 痕迹文件直接放在项目根目录，不要嵌套 `ai-traces/` 等子文件夹——用户偏好一切在一个文件夹里
    - **排除项**：`.gradle/`、`build/`、`target/`（编译产物）、`.idea/`、`.iml`（IDE 配置）、`gradle/`、`build.gradle`、`gradlew*`（Gradle 冗余）、`BUG_REPORT.md`、`FIX_GUIDE.md`、`design-summary.md`（开发笔记）、`.gitattributes`、`.gitignore`（Git 配置）、`docs/`、`sql/`（空目录）
    - **保留项**：`src/`（含 test）、`pom.xml`、`mvnw*`、`.mvn/`、`Dockerfile`、`docker-compose.yml`、`.env`、`settings.xml`、`*.sql`、`README.md`、`demo/`（路演素材）、`geocode_snippet.txt`
    - 详见 `references/cakeshop-submission.md`

23. **CakeShop Docker 部署四坑（详见 cakeshop-deploy 技能）** —
    **(a) JDK 版本必须是 11** — Dockerfile 用 `maven:3.9-eclipse-temurin-11-alpine` + `eclipse-temurin:11-jre-alpine`。JDK 17 会报 `LombokProcessor cannot access JavacProcessingEnvironment`，JDK 8 alpine 镜像已废弃。pom.xml 的 java.version、maven.compiler.source、maven.compiler.target、release 全部改为 11。
    **(b) Maven 必须配阿里云镜像** — ECS 访问 Maven Central 只有 ~21KB/s，构建需 15 分钟。Dockerfile builder 阶段内联写入：`RUN echo '...<url>https://maven.aliyun.com/repository/public</url>...' > /usr/share/maven/conf/settings.xml`。不要用单独的 settings.xml 文件 COPY（多一个依赖更容易出错）。
    **(c) init.sql 挂载路径** — docker-compose 中必须挂载到 `/docker-entrypoint-initdb.d/init.sql`（不是 `/init.sql`），否则 MySQL 容器不会自动执行建表。如果已启动失败，手动执行：`docker exec -i cakeshop-mysql mysql -uroot -proot cookieshop < sql/init.sql` + `docker restart cakeshop-app`。
    **(d) Docker Compose 版本** — 服务器用 v2，命令是 `docker compose`（无连字符），脚本中不能用 `docker-compose`（v1 已废弃）。
    **(e) 用户偏好简单** — 用户要的是一键部署（`python deploy_final.py`），不要拆多步。阿里云 VNC 不支持粘贴，优先用 ECS「发送远程命令」功能。
    **(f) 部署后验证** — HTTP 200 不代表完全正常，必须检查 `docker logs` 确认无 `Table 'xxx' doesn't exist` 错误。完整验证清单：首页、登录、后台、骑手页面。
    **(g) Docker 缓存导致代码不更新** — `docker compose up --build` 使用缓存层，源码改了也跳过重编。必须 `docker compose build --no-cache cakeshop && docker compose up -d cakeshop`。辅助：`touch` 改动的 .java 文件破时间戳。验证：`docker exec cakeshop-app unzip -p /app/app.jar BOOT-INF/classes/com/javaBean/User.class | strings | grep 字段名`。
    **(h) MyBatis @Results 覆盖自动映射** — `@Results({@Result(property="x", column="x")})` 只映射指定列，其余全部丢失变 `LinkedHashMap`，触发 `No converter for LinkedHashMap` 错误。正确：只用 `map-underscore-to-camel-case: true`（application.yml 已配），不写 @Results。自动映射失效时用 SQL 别名：`SELECT *, rider_apply as riderApply FROM user`。
    **(i) Thymeleaf 共用模板变量缺失** — 多个控制器方法共用同一模板时，所有方法都必须设齐模板变量。如 `/admin/userRiderApply` 用 `admin/userList` 模板，漏设 `unverified` 变量报模板错误但 HTTP 仍是 200。

24. **MySQL subprocess Python 3.13 注意事项** — 在 execute_code 中用 `subprocess` 连接 MySQL 时有两个坑：
    - **`input=` 必须是 bytes** — Python 3.13 的 `subprocess.run(input=...)` 不接受 str，报 `TypeError: a bytes-like object is required`。用 `input=sql.encode()` 或改用 `-e "SQL"` 命令行参数。
    - **`--default-character-set=utf8` 可能导致 stdin 管道超时** — 不加此标志、依赖默认编码通常更稳定。查询结果用 `r.stdout.decode('utf-8', errors='replace')` 解码。
    - **`SELECT` 返回 None** — 如果 stdout 解码后为空的 `''`，检查是否用了错误的 `-D` 参数或数据库名拼错。用 `-e 'SELECT 1'` 做连通性测试。

25. **Spring Boot 启动时自动修改数据库数据** — 有 `@PostConstruct` 或 `ApplicationRunner` 的配置类（如 `DataInitConfig`）会在**每次**启动时执行数据迁移和修复。常见影响：
    - **BCrypt 密码升级** — 检测到非 BCrypt 密码（不以 `$2a$` 开头）时自动替换为种子密码。用户用旧密码登录失败 → 先查配置类确认种子密码值。
    - **ALTER TABLE 加列** — 每次启动尝试 ALTER TABLE，不影响现有数据但会产生 SQL 日志噪音。
    - **种子数据补全** — 缺失的种子用户会被重新插入。
    排查「为什么数据变了」时，**先 grep `@PostConstruct` 和 `ApplicationRunner`** 找自动初始化逻辑。

26. **Vue 博客液态玻璃透明度默认值** — `frontend/src/assets/styles/main.css` 中 `.glass` / `.glass-panel` 类默认 `rgba(255,255,255,0.65)` 和 `0.7` 太低，内容文字透过卡片看不清。如果用户反馈「字看不清」「透明度太高」：
    - `.glass` → `rgba(255,255,255,0.88)`
    - `.glass-panel` → `rgba(255,255,255,0.9)`
    - `.dark .glass` → `rgba(30,30,32,0.85)`
    - `.dark .glass-panel` → `rgba(30,30,32,0.88)`
    同时增强边框（0.06→0.08）和阴影（0.03→0.04 / 0.04→0.06）。

27. **GSAP ScrollTrigger 在 Vue Router 切换后失效** — Vue Router 的 `<router-view>` 切换组件时，旧 DOM 销毁新 DOM 挂载，但 `ScrollTrigger` 的内部缓存仍指向旧元素。症状：滚动动画只播一秒就卡住，或无动画。修复两步：
    **(a) App.vue 页面过渡 onEnter 回调中刷新**：
    ```ts
    function onEnter(el: Element, done: () => void) {
      gsap.to(el, {
        opacity: 1, y: 0, duration: 0.5, ease: 'power3.out',
        onComplete: () => {
          import('gsap/ScrollTrigger').then(({ ScrollTrigger }) => {
            ScrollTrigger.refresh()
          })
          done()
        },
      })
    }
    ```
    **(b) 自定义指令加 unmounted 清理** — `v-scroll-reveal` 和 `v-fade-in` 必须在 `unmounted` 钩子中 `tween.kill()`，防止内存泄漏和旧动画干扰新页面。

28. **CakeShop 启动前检查清单** — 启动此项目必须逐项确认：
    - MySQL `cookieshop` 数据库存在且有数据（`SHOW DATABASES; USE cookieshop; SELECT COUNT(*) FROM user`）
    - `rider_chat` 表存在（代码依赖但 schema.sql 可能遗漏），不存在则 `CREATE TABLE`
    - `rider` 表有数据（骑手登录依赖），空则插入 seed riders
    - `application.yml` 端口用 3306（非 3308），密码为空（非 root）
    - `${DEEPSEEK_API_KEY}` 有默认值 `${DEEPSEEK_API_KEY:***}`（否则启动崩溃）
    - JDK 12 编译运行（class version 56，JDK 8 报 UnsupportedClassVersionError）
    - 无冲突的 `application.properties`（删除它，只保留 yml）
2. 项目使用 IDEA 的 .iml 格式，非标准 Maven 目录结构
3. 优先在 IDEA 内操作，不要用命令行代替 IDE 功能
4. 数据库连接通过 Navicat 管理，避免直接用命令行操作 MySQL
5. Tomcat 通过 IDE 集成部署，不做独立安装
