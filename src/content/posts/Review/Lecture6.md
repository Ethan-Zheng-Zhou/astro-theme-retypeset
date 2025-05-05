---
# 必填
title: "ASP.NET期末复习Lecture6 : Minimal APIs 参数绑定与 MVC基础"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 4
toc: true
abbrlink: 'asp-lecture6'
---

第六次课主要内容涉及 **ASP.NET Core 中的参数绑定（Model Binding）机制**，以及 **MVC 框架结构的基础配置**。：

---
## 一、参数绑定机制（Model Binding）

### 1. 路由参数优先 (Route Parameter)

当参数名与路由模板中定义的参数名称匹配时，系统会优先从路由值中获取参数值。

```csharp
app.MapGet("/products/{id?}", (int id) => $"Received {id}");
```

- **示例 1**: 请求 `/products/123`
  - 路由模板 `{id?}` 匹配到值 "123"
  - 参数 `id` 成功绑定为 123
  - 输出: "Received 123"

### 2. 查询字符串次之 (Query String)

如果路由中没有匹配的参数，系统会尝试从查询字符串中获取值。

```csharp
app.MapGet("/products/{id?}", (int id) => $"Received {id}");
```

- **示例 2**: 请求 `/products?id=456`
  - 路由中没有提供 id 值（可选参数未使用）
  - 从查询字符串 `?id=456` 中获取值
  - 参数 `id` 成功绑定为 456
  - 输出: "Received 456"
### 3.显式绑定

使用绑定特性可以覆盖默认绑定行为，明确指定参数来源：

- `[FromRoute]` - 强制从路由获取值
- `[FromQuery]` - 强制从查询字符串获取值
- `[FromBody]` - 从请求体获取值 (常用于 POST/PUT)
- `[FromHeader]` - 从请求头获取值
- `[FromServices]` - 从依赖注入容器获取服务

```csharp
// 强制从查询字符串获取，即使路由中有同名参数
app.MapGet("/products/{id}", ([FromQuery] int id) => $"Received {id}");

// 混合绑定示例
app.MapGet("/search/{category}", ([FromRoute] string category, [FromQuery] string keyword) => 
    $"Searching {category} for '{keyword}'");

// 从不同来源绑定
app.MapPost("/users", ([FromBody] User user, [FromHeader] string authorization) => 
    $"User {user.Name} created with auth {authorization}");
```

### 4.绑定复杂类型（如类对象）

Minimal API 可以自动将 JSON 请求体反序列化成对象。

```csharp
app.MapPost("/product", (Product product) => $"Received {product}");

record Product(int Id, string Name, int Stock);
```

```json
{
  "id": 1,
  "name": "Shoes",
  "stock": 12
}
```

ASP.NET Core 会自动构建 `Product` 对象传入函数中。

---
## 二、MVC 架构模式

### 1. MVC 是什么？

MVC（Model-View-Controller）是一种**用户界面设计模式**，将应用程序分为三个核心部分：

|角色|功能|
|---|---|
|Model|数据和业务逻辑|
|View|用户界面展示模板|
|Controller|接收用户请求并协调 Model 和 View|

### 2. 请求处理流程：

1. 用户发起请求；
2. 控制器（Controller）处理请求；
3. 控制器向模型（Model）请求数据（内存/文件/数据库）；
4. 控制器把数据传给视图（View）；
5. 视图渲染 HTML 页面返回给用户。

---

## 三、配置 ASP.NET Core MVC 应用

### 1. 启动入口 Program.cs

- 配置 ASP.NET Core 平台；
- 加载中间件、服务、路由等。
### 2. Services（服务）

- 提供功能支持的对象（如配置、日志等）；
- 由 ASP.NET Core 管理，可全局共享。
### 3. 配置服务（Configuration）

- 配置来源：
	- `appsettings.json`：常规配置项；
	- `launchSettings.json`：启动环境配置（端口号、运行环境等）；
- 支持多个配置文件，如：
    - `appsettings.Development.json`
    - `appsettings.Production.json`
- 配置内容类型：
	- **Settings**：影响程序行为的普通配置值；
	- **Secrets**：如密码等敏感数据。
