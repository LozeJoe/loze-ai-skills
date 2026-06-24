# Vue 3 + Spring Boot 全栈项目模板

基于 loze-blog-vue 项目（2026-06-12 创建），记录完整的项目结构和关键代码模式。

## 项目规模

- 后端 Java 文件: 45 个
- 前端 Vue 文件: 16 个
- 前端 TS/JS: 10 个
- 数据库表: 7 张 (t_user, t_post, t_diary, t_note, t_friend, t_site_config, t_comment)

## 目录结构速查

```
project/
├── backend/
│   ├── pom.xml                          # Spring Boot 2.7.18 父 POM
│   ├── src/main/java/com/loze/blog/
│   │   ├── BlogApplication.java         # @SpringBootApplication + @MapperScan
│   │   ├── config/WebConfig.java        # CORS + Interceptor + ResourceHandler
│   │   ├── interceptor/AuthInterceptor.java  # JWT Bearer Token 验证
│   │   ├── utils/JwtUtil.java           # HS256 签名/验证/解析
│   │   ├── entity/                      # 7 个 @Data @TableName 实体
│   │   ├── mapper/                      # 7 个 extends BaseMapper<T>
│   │   ├── service/                     # 7 个接口 + 7 个实现
│   │   ├── controller/                  # 9 个 @RestController
│   │   └── dto/                         # ApiResponse, LoginRequest, RegisterRequest, WriteRequest
│   ├── src/main/resources/
│   │   ├── application.yml              # MySQL + MyBatisPlus + JWT + 文件上传配置
│   │   ├── schema.sql                   # 完整建表 + 初始数据
│   │   └── static/                      # photos/, images/, music/ 静态资源
│   └── content/                         # 原博客 MDX 内容文件（备用）
│
├── frontend/
│   ├── src/
│   │   ├── api/index.ts                 # axios 实例 + 拦截器
│   │   ├── router/index.ts              # 12 条路由 + beforeEach 守卫
│   │   ├── stores/                      # auth.ts, theme.ts, site.ts (Pinia)
│   │   ├── views/                       # 8 公开页 + 4 管理页
│   │   ├── components/                  # Header, Footer, PostCard
│   │   ├── assets/styles/main.css       # CSS 变量 + 毛玻璃 + 排版
│   │   ├── App.vue                      # flex-col 布局 + fade 过渡
│   │   └── main.ts                      # createApp + Pinia + Router
│   ├── vite.config.ts                   # @ 别名 + proxy /api -> :8080
│   ├── tailwind.config.js               # 字体配置
│   └── package.json                     # vue3, vue-router, pinia, axios, gsap, markdown-it
└── README.md
```

## 后端关键代码

### pom.xml 核心依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.18</version>  <!-- Java 8 兼容 -->
</parent>
<dependencies>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId></dependency>
    <dependency><groupId>com.baomidou</groupId><artifactId>mybatis-plus-boot-starter</artifactId><version>3.5.5</version></dependency>
    <dependency><groupId>com.mysql</groupId><artifactId>mysql-connector-j</artifactId></dependency>
    <dependency><groupId>io.jsonwebtoken</groupId><artifactId>jjwt</artifactId><version>0.9.1</version></dependency>
    <dependency><groupId>javax.xml.bind</groupId><artifactId>jaxb-api</artifactId><version>2.3.1</version></dependency>
    <dependency><groupId>org.projectlombok</groupId><artifactId>lombok</artifactId></dependency>
</dependencies>
```

### application.yml 模板

```yaml
server:
  port: 8088  # 8080 常被占用，用 8088

spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/loze_blog?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: 20060708
  servlet:
    multipart:
      max-file-size: 100MB

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
  global-config:
    db-config:
      id-type: auto
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0

jwt:
  secret: <至少 32 字符密钥>
  expiration: 86400000  # 24小时
```

### JWT 认证流程

1. POST `/api/admin/login` → 验证用户 → `jwtUtil.generateToken(userId, username, role)` → 返回 `{token, user}`
2. 前端存 `localStorage.setItem('token', token)`
3. axios 拦截器: `config.headers.Authorization = 'Bearer ' + token`
4. AuthInterceptor: 解析 Bearer header → `jwtUtil.validateToken()` → `request.setAttribute("userId", ...)`

### 实体类模式

```java
@Data
@TableName("t_post")
public class Post {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String slug;
    private String title;
    private String content;
    // ...
    @TableLogic
    private Integer deleted;
}
```

### Controller 统一响应

```java
@GetMapping("/posts")
public ApiResponse<Map<String, Object>> getPosts(...) {
    return ApiResponse.ok(data);  // {success: true, data: {...}}
}

@PostMapping("/admin/write")
public ApiResponse<Post> writePost(@Valid @RequestBody WriteRequest req) {
    return ApiResponse.ok(post);  // {success: true, data: {...}}
}
```

## 前端关键代码

### vite.config.ts — 别名 + 代理

```ts
import { fileURLToPath, URL } from 'node:url'
export default defineConfig({
  resolve: { alias: { '@': fileURLToPath(new URL('./src', import.meta.url)) } },
  server: { port: 5173, proxy: { '/api': { target: 'http://localhost:8088', changeOrigin: true } } }
})
```

**关键坑**: 创建 Vite 项目后必须显式添加 `@` 别名，否则所有 import 报 "Failed to resolve import"。

### 路由守卫

```ts
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  if (to.meta.requiresAuth && !token) {
    next('/admin/login')
  } else {
    next()
  }
})
```

### Pinia auth store

```ts
export const useAuthStore = defineStore('auth', () => {
  const token = ref(localStorage.getItem('token') || '')
  const isLoggedIn = computed(() => !!token.value)
  
  async function login(username: string, password: string) {
    const res = await api.post('/admin/login', { username, password })
    if (res.success) {
      token.value = res.data.token
      localStorage.setItem('token', res.data.token)
      return true
    }
    return false
  }
  
  function logout() {
    token.value = ''
    localStorage.removeItem('token')
    router.push('/')
  }
  
  return { token, isLoggedIn, login, logout }
})
```

## 数据库表设计

| 表名 | 用途 | 核心字段 |
|------|------|----------|
| t_user | 用户 | username, password, name, phone, address, role |
| t_post | 文章 | slug, title, content, tags, category, draft, date |
| t_diary | 日记 | slug, title, content, mood, weather, date |
| t_note | 说说 | slug, title, content, date |
| t_friend | 友链 | name, url, avatar, description, sort |
| t_site_config | 配置 | config_key, config_value (KV 存储) |
| t_comment | 评论 | post_id, user_id, username, content |

所有表都有 `create_time` 和 `deleted`（逻辑删除）字段。

## 创建项目时的文件写入策略

由于 `write_file` 和 `terminal` 工具可能在本机失败（WSL 层问题），统一用 `execute_code` + Python `open().write()` 批量创建文件：

```python
import os
base = r'C:\Users\Loze\projects\project-name\backend\src\main\java\com\loze\blog'

# 批量创建目录
for d in ['config', 'controller', 'service', 'entity', 'mapper', 'utils', 'interceptor', 'dto']:
    os.makedirs(os.path.join(base, d), exist_ok=True)

# 批量写入文件
files = {
    'config/WebConfig.java': 'package com.loze.blog.config; ...',
    'entity/Post.java': 'package com.loze.blog.entity; ...',
    # ...
}

for path, content in files.items():
    full = os.path.join(base, path)
    os.makedirs(os.path.dirname(full), exist_ok=True)
    with open(full, 'w', encoding='utf-8') as f:
        f.write(content)
```

## 启动方式

### 后端
用 IDE（IntelliJ IDEA）打开 `backend/`，运行 `BlogApplication.main()`。
或用批处理：设置 JAVA_HOME + MAVEN_HOME PATH 后 `mvn spring-boot:run`。

### 前端
```bash
cd frontend
npm install
npm run dev    # http://localhost:5173
```

使用 `subprocess.Popen(batch_path, creationflags=subprocess.CREATE_NEW_CONSOLE)` 在新窗口中启动，避免进程被杀。
