# 骑手注册功能

## 概述
- 用户在登录页的「骑手登录」面板下方点击「注册成为骑手」链接
- 进入 rider/register.html 填写注册表单
- 注册成功后自动审核通过，可直接登录骑手端

## 涉及文件

### 后端
- **RiderController.java**: GET/POST `/rider/register`
  - 注入了 UserMapper 和 UserService
  - 注册时 isadmin="2", isvalidate="1"(自动审核)
- **UserMapper.java**: `register()` 方法，7参数（含address）
- **UserService.java / UserServiceImpl.java**: `encodePassword()` BCrypt加密

### 前端
- **login.html**: 骑手面板底部链接 `<a th:href="@{/rider/register}">注册成为骑手</a>`
- **rider/register.html**: 注册表单（账号/密码/确认密码/姓名/手机/邮箱）

### 管理员管理骑手
- **admin/userEdit.html**: 角色选择增加「🛵 骑手」radio（value="2"），新建时 `param.role == 'rider'` 预选
- **admin/userList.html**: 「设为骑手」按钮（isadmin='0' 的普通用户可见）→ `/admin/userSetAdmin?id=X&isadmin=2`
- 不需要单独的「添加骑手」按钮 — 统一走 userEdit 入口即可

## 表单字段
| 字段 | 验证规则 |
|------|----------|
| username | 2-50字符，不可重复 |
| password | ≥8字符 |
| confirmPassword | 与password一致 |
| name | 必填 |
| phone | 11位数字，1开头 |
| email | 可选 |

## 关键调用
```java
// 注册骑手
userMapper.register(username, encodedPassword, name, phone, "", email, "2");
// 自动审核
userMapper.verifyUser(newUser.getId());
```

## 已废弃：骑手申请审核系统
此前实现的「用户申请→管理员审核」模式已全部移除，文件改动：
- User.java: 移除 riderApply 字段
- UserMapper.java: 移除 updateRiderApply / getRiderApplications / countRiderApplications
- UserService/UserServiceImpl: 移除申请/审核相关方法
- UserController.java: 移除 applyRider 方法
- AdminController.java: 移除 userRiderApply / userApproveRider / userRejectRider
- DataInitConfig.java: 移除 rider_apply 列 ALTER TABLE
- admin/userList.html: 移除「骑手申请」tab
- myOrder.html: 移除申请按钮/审核状态
