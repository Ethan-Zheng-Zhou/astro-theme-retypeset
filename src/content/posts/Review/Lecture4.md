---
# 必填
title: "ASP.NET期末复习Lecture4 : Minimal APIs 的基础与应用"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 2
toc: true
abbrlink: 'asp-lecture4'
---
第四次课讲义的主题是 **Minimal APIs 的基础与应用**，主要围绕 ASP.NET Core 中如何快速定义 HTTP API（Web API）展开。介绍了 Minimal API 的概念、适用场景、路由设计、动词映射、以及处理函数的编写方式。

---

##  一、什么是 HTTP API（Web API）

### 概念解释：

- HTTP API 就是通过 HTTP 协议暴露多个 URL 接口，用于获取（GET）、修改（POST/PUT）、删除（DELETE）服务器上的数据。
- 返回数据的格式通常是 **JSON**，适合前后端分离的架构。
---

##  二、什么情况下使用 HTTP API？

讲义列举了三种典型场景：

### 1. SPA（单页应用 Single Page Application）：

- 前端使用如 Angular、React、Vue 等框架构建。
- 浏览器加载 SPA 后，前后端通过 HTTP 请求交换 JSON 数据。
- 服务器不直接返回 HTML，而是返回结构化数据（如 JSON）。

### 2. 移动应用：

- 移动端（iOS/Android）通过 HTTP API 获取或提交数据，与 SPA 的模式类似。
- 服务端仍以 JSON 响应数据。

### 3. 后台服务（Other back-end services）：

- 如邮件服务提供 API，供其他后端系统调用，传入 email 和内容即可发送邮件。
- 各种语言（如 Python、Java、Node.js）都能轻松通过 HTTP 访问你的 API。

---

##  三、定义 Minimal API 的路由端点

### 1. 简单端点：

```csharp
app.MapGet("/", () => "Hello World!");
app.MapGet("/person", () => new Person("Lee"));
```

### 2. 动态路由（Parameterized Routes）：

- 使用 `{parameter}` 占位符获取路径中的动态值：

```csharp
app.MapGet("/person/{name}", (string name) => $"Hello, {name}!");
```

- 访问 `/person/Lee` 时，`name` 参数值为 `"Lee"`。

---

## 四、HTTP 动词与端点映射

### HTTP 动词及其功能：

|动词|作用|
|---|---|
|GET|获取数据|
|POST|添加数据|
|PUT|更新数据|
|DELETE|删除数据|

### 使用 Map 系列函数映射：

```csharp
app.MapGet("/fruit", () => Fruit.All);
app.MapPost("/fruit/{id}", Handlers.AddFruit);
app.MapDelete("/fruit/{id}", DeleteFruit);
```

---

## 五、端点处理函数的两种写法

### 方法一：Lambda 表达式

```csharp
app.MapGet("/fruit", () => Fruit.All);
```

### 方法二：单独定义函数或静态方法

```csharp
void DeleteFruit(string id)
{ Fruit.All.Remove(id); }

app.MapDelete("/fruit/{id}", DeleteFruit);

class Handlers {
    public static void AddFruit(string id, Fruit fruit)
    { Fruit.All.Add(id, fruit); }
}
app.MapPost("/fruit/{id}", Handlers.AddFruit);
```

---

## 示例中的数据模型（record 类型）

```csharp
record Fruit(string Name, int Stock)
{
    public static readonly Dictionary<string, Fruit> All = new();
}
```

- `record` 是 C# 9 引入的简洁数据结构声明方式，适合定义不可变对象。
- `Fruit.All` 是一个静态字段，用于模拟内存数据库。

---

## 总结

|核心点|说明|
|---|---|
|Minimal API|轻量级、快速创建 Web API 的方式，省略了 MVC 控制器等结构。|
|动态路由|使用 `/resource/{parameter}` 格式，自动解析参数。|
|动词映射|MapGet、MapPost、MapDelete 等可清晰对应 HTTP 动作。|
|数据处理|可以使用 Lambda 或普通方法来作为处理器。|
