# 测试方法论

## HTTP 测试要点

### 会话隔离
不同角色（用户/骑手/管理员）必须用独立 CookieJar：
```python
def new_session():
    return urllib.request.build_opener(
        urllib.request.HTTPCookieProcessor(http.cookiejar.CookieJar())
    )
user = new_session()
admin = new_session()
```
共享 CookieJar 会导致 403、跳转错误等误判。

### 中文 URL 编码
Python `urllib` 不自动编码中文参数，必须手动：
```python
import urllib.parse
kw = urllib.parse.quote('蛋糕')
url = f"{BASE}/goods/search?keyword={kw}"
```

### HTTP 200 不等于成功
Spring Boot/Thymeleaf 错误页面也返回 200。必须检查：
- 页面是否有 `error` CSS 类或错误文本
- `docker logs` 确认无异常
- 关键内容如用户名、按钮文本是否存在

## 登录路由速查

| 角色 | 路径 | 参数 |
|------|------|------|
| 普通用户 | POST /user/login | userName, userPassword |
| 骑手 | POST /rider/doLogin | username, password |
| 管理员 | POST /user/login | userName, userPassword（用 user 表 isadmin='1' 账户） |

注意：骑手登录参数名是 `username`/`password`（不是 `userName`/`userPassword`）。

## Maven 测试运行

### 前置条件
- MySQL 必须运行，数据库 `cookieshop` 有数据
- `goods` 表必须有 `status` 列（MariaDB 10.3 需手动 `ALTER TABLE goods ADD COLUMN status INT(1) DEFAULT 1`）
- `application-test.yml` 必须覆盖 `ark.api.key`（主配置用 `${ARK_API_KEY}` 占位符无默认值）
- 设置 `ARK_API_KEY` 环境变量作兜底

### 运行命令
```bash
set JAVA_HOME=C:\Program Files\Java\jdk-12.0.1
set ARK_API_KEY=test-key
mvn clean test
```

### Maven 输出解码
Windows Maven 输出为 GBK 编码，AI 执行时需：
```python
result = subprocess.run(['cmd', '/c', mvn_cmd, 'test'], capture_output=True, env=env)
stdout = result.stdout.decode('gbk', errors='replace')
```

### 结果分层
| 层级 | 类数 | 特征 |
|------|------|------|
| unit.service.* | 10 | Mockito 纯单元测试，不依赖 Spring 上下文 |
| integration.* | 6 | 需 Spring Boot + 真实 MySQL |
| controller.* | 8 | MockMvc + Spring 上下文 |
| e2e.* | 5 | 完整 HTTP 请求链路 |

### 常见失败模式
1. **全部 integration/controller/e2e 报同一 Error** → Spring 容器启动失败，看 `surefire-reports/*.txt` 第一条错误
2. **Unknown column 'g.status'** → MariaDB 不支持 `ADD COLUMN IF NOT EXISTS`（坑B）
3. **Could not resolve placeholder 'ARK_API_KEY'** → `application-test.yml` 缺 `ark.api.key`（坑D）
1. 首页加载 → 有商品、登录/注册/购物车链接
2. 用户登录 → 用已知账号
3. 后台管理 → admin 登录 → 商品/订单/用户管理
4. 骑手登录 → /rider/doLogin → /rider/index
5. 商品浏览 → 详情/分类/搜索
6. 购物车 → 加购/查看
7. 静态资源 → CSS/图片 200
8. 错误处理 → 404 页面有内容
