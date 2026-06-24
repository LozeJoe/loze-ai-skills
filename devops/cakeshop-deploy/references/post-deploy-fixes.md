# CakeShop 部署后代码修复

两个必须应用到源码并重新部署的修复。

## 修复1：RiderController — /rider 根路径重定向

**问题**: 访问 `/rider` 返回 404，骑手不知道入口在哪。

**文件**: `src/main/java/com/controller/RiderController.java`

在类体内添加（第一个方法之前）：

```java
// 重定向 /rider -> /rider/index
@RequestMapping("")
public String root() {
    return "redirect:/rider/index";
}
```

## 修复2：UserController — 骑手登录后跳转错误

**问题**: 所有用户通过公共 `/user/login` 登录后统一跳转到 `/goods/goodList`。
骑手登录后应跳到骑手首页 `/rider/index`。

**文件**: `src/main/java/com/controller/UserController.java`

在 `login()` 方法中找到：
```java
session.setAttribute("user", user);
modelAndView.setViewName("redirect:/goods/goodList");
```

替换为：
```java
session.setAttribute("user", user);
// Role-based redirect
String role = request.getParameter("role");
if ("rider".equals(role)) {
    modelAndView.setViewName("redirect:/rider/index");
} else if ("admin".equals(role)) {
    modelAndView.setViewName("redirect:/admin/index");
} else {
    modelAndView.setViewName("redirect:/goods/goodList");
}
```

## 完整功能验证清单

部署后逐项测试（20项）：

```
用户端:
  ✅ GET  /                         首页商品列表
  ✅ GET  /user/login               用户登录页
  ✅ GET  /user/register            用户注册页
  ✅ POST /user/login               vili/Vili1234! 登录
  ✅ GET  /goods/detail?id=1        商品详情
  ✅ GET  /goods/goodList?typeid=1  分类列表
  ✅ GET  /goods/goodList           全部商品
  ✅ GET  /goods/newGoods           新品推荐
  ✅ GET  /goods/topSell            热销排行
  ✅ GET  /goods/search?keyword=蛋糕 商品搜索
  ✅ GET  /cart/cartList            购物车

后台:
  ✅ GET  /admin                    后台登录页
  ✅ POST /admin/login              admin/admin123
  ✅ GET  /admin/index              后台首页
  ✅ GET  /admin/goods              商品管理
  ✅ GET  /admin/orders             订单管理
  ✅ GET  /admin/users              用户管理
  ✅ GET  /admin/types              分类管理

骑手端:
  ✅ GET  /rider                    → 骑手登录页（未登录）/ 骑手首页（已登录）
  ✅ POST /user/login?role=rider    rider1/Rider1234! → /rider/index
  ✅ GET  /rider/index              骑手首页
  ✅ GET  /rider/accept             接单
  ✅ GET  /rider/profile            个人资料

服务器:
  ✅ docker logs cakeshop-app | grep -i error  → 空（无异常日志）
```
