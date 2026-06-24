# CakeShop 项目模式

## 启动条件

- 路径: `D:/JAVAEE/demo/demo/`（注意双重目录）
- 命令: `D:/JAVAEE/apache-maven-3.5.2/apache-maven-3.5.2/bin/mvn.cmd spring-boot:run`（Maven 路径也是双重目录）
- JDK: 必须用 JDK 12 (`C:/Program Files/Java/jdk-12.0.1`)，JDK 8 会报 `UnsupportedClassVersionError`
- 数据库: MySQL `cookieshop` (XAMPP, D:/xampp/mysql/bin/mysql.exe, root 无密码)
- 端口: 8080 (在 application.yml 中配置，不是 properties)

## 启动前必查清单

1. ✅ MySQL XAMPP 在运行（端口 3306）
2. ✅ `cookieshop` 数据库存在且有表
3. ✅ `rider_chat` 表已创建（schema.sql 可能不包含此表）
4. ✅ `rider` 表有数据（不是空的）
5. ✅ `application.yml` 中 MySQL 端口 = 3306，密码为空
6. ✅ `application.yml` 中 `DEEPSEEK_API_KEY` 有默认值或已设置

## 常见启动失败

### UnsupportedClassVersionError
类文件是 Java 12 编译的 (class version 56)，JDK 8 无法运行。用 JDK 12 启动。

### DEEPSEEK_API_KEY 缺失
`application.yml` 中 `key: ${DEEPSEEK_API_KEY}` 无默认值导致 `AiChatService` 注入失败。改为 `key: ${DEEPSEEK_API_KEY:***}`。

### rider_chat 表不存在
数据库里没有此表，但 `RiderChatMapper` 会查询。手动创建：
```sql
CREATE TABLE IF NOT EXISTS rider_chat (
  id INT AUTO_INCREMENT PRIMARY KEY,
  order_id VARCHAR(50) NOT NULL,
  sender VARCHAR(20) NOT NULL,
  sender_name VARCHAR(50),
  content TEXT NOT NULL,
  create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
  KEY idx_order_id (order_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### rider 表为空
骑手数据需要手动插入（DataInitConfig 不会自动创建 rider 记录）：
```sql
INSERT INTO rider (username, password, name, phone) VALUES
('rider1', 'Rider1234!', '张三', '13900139000'),
('rider2', 'Rider1234!', '李骑手', '13800002222');
```

## 密码系统

DataInitConfig 在启动时自动升级密码：
- 检测非 BCrypt 密码（不以 `$2a$` 开头）
- 自动替换为 seed 密码的 BCrypt 哈希

**升级后的有效密码**：
- admin / admin123
- vili / Vili1234!
- rider1 / Rider1234!

## 数据库表注意

- `order` 是保留字，SQL 中必须用反引号：`` `order` ``
- `orderitem` 表只有 4 列：`id`, `order_id`, `goods_id`, `price`, `amount`
- `cart` 表用 `user_name`（不是 `user_id`）和 `good_id`（varchar）
- `review` 表用 `goods_id`（不是 `good_id`）
