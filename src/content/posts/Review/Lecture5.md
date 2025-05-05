---
# 必填
title: "ASP.NET期末复习Lecture5 : Minimal APIs 的响应生成"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
toc: true
abbrlink: 'asp-lecture5'
---

第五次课主要围绕 **ASP.NET Core Minimal APIs 的响应生成、路由系统与模型绑定（Model Binding）** 三大核心主题展开。

---
## 一、使用 IResult 生成响应（Generating Responses with IResult）

### 1. 什么是 `IResult`

在 ASP. NET Core 的 Minimal APIs 中，`IResult` 是一个接口，表示一个 HTTP 响应。它是 Minimal APIs 中返回 HTTP 响应的核心抽象，用于封装状态码、响应头、响应体等信息。  

`IResult` 定义了如何将结果写入 HTTP 响应，它由 `Results` 或 `TypedResults` 静态类提供的方法创建。例如：

- `Results.Ok()` → 返回 `200 OK`
- `Results.NotFound()` → 返回 `404 Not Found`
- `TypedResults.Text("Hello")` → 返回纯文本响应
- `TypedResults.Json(obj)` → 返回 JSON 响应


### 2. 为什么使用 `IResult`

ASP.NET Minimal API 的终结点方法（Endpoint Handlers）可以返回多种类型：

- `void` → 返回 200 OK，无内容。
- `string` → 返回文本内容，状态码为 200。
- `T`（任意类型）→ 自动序列化为 JSON。
- `IResult` → 更灵活的响应方式，可自定义状态码、响应体等。

下面的代码展示了不同返回类型的行为：

```csharp
app.MapGet("/fruit/{id}", (string id) =>
    _fruit.TryGetValue(id, out var fruit)
        ? TypedResults.Ok(fruit)
        : Results.NotFound());
```

1. 它注册了一个路由模式为`"/fruit/{id}"`的GET请求处理程序
2. 当这个路由被请求时：
   - 从路由参数中提取`id`字符串
   - 尝试在`_fruit`字典（假设是一个`Dictionary<string, T>`）中查找这个ID
   - 如果找到对应的水果（`TryGetValue`返回true）：
     - 返回200 OK状态码，并将水果对象作为响应体
   - 如果没找到：
     - 返回404 Not Found状态码

使用 `TypedResults.Ok` 和 `Results.NotFound()` 能更优雅地处理资源存在与否。

---

## 二、Routing（路由机制）

### 1. 什么是 Routing？

- Routing 是将 HTTP 请求映射到对应处理方法（Handler）的过程。
- Minimal API 通过 URL 模板（Route Template）来匹配请求路径。
### 2. Endpoint Routing

引入于 ASP.NET Core 3，分为两个中间件：

- `RoutingMiddleware`：选择哪个 Endpoint 被执行。
- `EndpointMiddleware`：调用被选中的 Endpoint。
   
如果没有匹配的路由，将返回 404 Not Found。

### 3. 如何定义 Endpoint 

在 ASP. NET Core（特别是 **Minimal APIs**）中，**Endpoint（终结点）** 由两部分组成：  

1. **路由模板（Route Template）**：定义匹配哪些 URL 请求。  
2. **请求委托（Request Delegate）**：处理请求并返回响应的逻辑代码。  

路由模板是一个 **URL 模式**，用于匹配传入的 HTTP 请求。

路由模板支持：
- **Literal segments**（静态字符串）；
- **Route parameters**（如 `{id}`，可以变化）；
- **Optional parameters**（如 `{id?}`，可选）；
- **Default values**（如 `{name=all}`，有默认值）。

| 路由模板示例 | 匹配的 URL 示例 |
|-------------|----------------|
| `"/"` | `https://example.com/` |
| `"/products"` | `https://example.com/products` |
| `"/users/{id}"` | `https://example.com/users/123` |
| `"/blog/{year}/{month}"` | `https://example.com/blog/2023/10` |

路由中间件将路由模板拆分为段来解析路由模板，段通常由 '/' 分隔。
- 段可以是纯文字 ( literal value )
- 段可以是路由参数
- 请求 URL 必须与文字值完全匹配（忽略大小写）。

请求委托是处理 HTTP 请求并返回响应的代码逻辑，通常是一个 **Lambda 表达式** 或 **方法**。  

```csharp
app.Map[HttpMethod]("路由模板", (参数) => {
    // 处理逻辑
    return 结果;
});
```

- `MapGet` → 处理 `GET` 请求  
- `MapPost` → 处理 `POST` 请求  
- `MapPut` → 处理 `PUT` 请求  
- `MapDelete` → 处理 `DELETE` 请求  



### 4. 练习题讲解

![练习题 1](https://cdn.ethanzhou.cn/i/2025/05/05/681869f716a90.jpg)

### **ASP.NET Core 路由匹配规则**

1. **严格匹配路径**。
2. **不能多段或少段**，必须完全匹配 `/About/Contact` 的结构。
    - ✅ 正确匹配：`/About/Contact`（完全一致）
    - ❌ 不匹配：
        - `/about`（少了一段 `Contact`）
        - `/about-us/contact`（第一段 `about-us` 不匹配 `About`）
        - `/about/contact/email`（多了一段 `email`）
        - `/about/contact-us`（第二段 `contact-us` 不匹配 `Contact`）

当然，配置了默认值或者可选的段允许少段：

![练习题 2](https://cdn.ethanzhou.cn/i/2025/05/05/68186ae0ee181.jpg)

![练习题 3](https://cdn.ethanzhou.cn/i/2025/05/05/68186b943f85d.jpg)

---

## 三、Model Binding（模型绑定）

### 1. 概念

- 模型绑定是将请求中的数据（路由、查询字符串、请求体等）转化为 .NET 对象的过程；
- 这些对象以方法参数的形式传递给 Endpoint Handler。
- 模型绑定发生在端点处理程序在 EndpointMiddleware 中执行之前。
### 2. 数据来源

ASP.NET Core 的 Minimal API 支持从以下六个来源绑定数据。

|来源类型|示例|
|---|---|
|Route Values|`/fruit/{id}` → `string id`|
|Query String|`/fruit?id=apple` → `string id`|
|Header|`Request.Headers["X-Token"]`|
|Body JSON|POST 请求的 JSON 正文|
|Dependency Injection|`[FromServices]` 注入服务|
|Custom Binding|自定义绑定逻辑|

---

## 小结：第五次课重点回顾和课程重点代码

| 模块            | 核心内容                        |
| ------------- | --------------------------- |
| IResult       | 灵活控制返回状态码与内容                |
| Routing       | 利用 URL 模板匹配请求，支持参数、默认值、可选项等 |
| Model Binding | 将请求中的数据自动转为 .NET 对象         |

```csharp
// 全局变量
var _fruit = new ConcurrentDictionary<string, Fruit>();
app.MapGet("/fruit", () => _fruit);

/* 访问 /fruit/{id} 
   如果找到，返回200ok和水果数据v
   如果没有找到，返回404  
*/
app.MapGet("/fruit/{id}", (string id) =>
    _fruit.TryGetValue(id, out var fruit)
        ? TypedResults.Ok(fruit)
        : Results.NotFound());

/* 访问 /fruit/{id} 
   如果ID不存在，添加成功并返回201 Created和新水果数据
   如果ID已存在，返回404 Bad Request和错误信息
*/

app.MapPost("/fruit/{id}", (string id, Fruit fruit) =>
    _fruit.TryAdd(id, fruit)
        ? TypedResults.Created($"/fruit/{id}", fruit)
        : Results.BadRequest(new{ id = "A fruit with this id already exists" }));

/* 访问 /fruit/{id} 
   更新指定ID的水果数据
   直接覆盖旧值，如果id不存在会新增，返回204 No Content
*/

app.MapPut("/fruit/{id}", (string id, Fruit fruit) =>
{
    _fruit[id] = fruit;
    return Results.NoContent();
});


/* 访问 /fruit/{id} 
   删除ID的水果数据
   无论是否存在都返回204 No Content
*/


app.MapDelete("/fruit/{id}", (string id) =>
{
    _fruit.TryRemove(id, out _); //直接丢弃输出值
    return Results.NoContent();
});

```