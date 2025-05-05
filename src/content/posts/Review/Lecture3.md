---
# 必填
title: "ASP.NET期末复习Lecture3 : 中间件与管道机制"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 1
toc: true
abbrlink: 'asp-lecture3'
---

第3次课，主要介绍了 ASP.NET Core 中 **Middleware（中间件）管道机制** 的基础知识及应用。

---
## 1. Middleware 是什么？

### 中间件的定义：

Middleware（中间件）是 ASP.NET Core 中处理 HTTP 请求和响应的基本构建块。每个中间件都可以：

- 处理传入请求（Request）
- 对请求做出响应（Response）
- 决定是否将请求传递给下一个中间件

它们串联在一起组成了“**Middleware Pipeline（中间件管道）**”。

---
## 2. 中间件的核心用途与设计模式

### 中间件常用于处理跨领域关注点（Cross-cutting concerns）：

也就是对所有请求都需要处理的事情，例如：

- **记录日志（Logging）**
- **添加安全响应头（Security headers）**
- **关联用户身份（Authentication）**
- **设置请求语言（Localization）**

### 采用的设计模式：

> **Chain-of-Responsibility（责任链）设计模式**

请求通过一个个中间件传递，就像一条链。

---

## 3. 管道是双向的（Bidirectional）

- 请求从上往下穿过每个中间件；
- 响应则从最后一个中间件向前逐层返回。
### Terminal Middleware（终结中间件）：

有些中间件可以**短路**整个管道流程（如直接返回响应），后面的中间件将不会被执行。

---
## 4. 如何组合中间件？

### 添加中间件的方式：

使用 `.UseXXX()` 方法将中间件添加到 ASP.NET Core 的 `WebApplication` 中：

```csharp
app.UseStaticFiles();     // 添加静态文件中间件
app.UseRouting();         // 添加路由中间件
```

---
## 5. 示例讲解

### 示例 1：Holding Page

- 所有请求都返回一个简单的 HTML 页面。
- 用于调试环境下测试是否能正常响应请求。
- 使用 `WelcomePageMiddleware` 来实现。
    
```csharp
//WelcomePageMiddleware处理所有请求“/”路径，并返回一个示例的HTML响应。
app.UseWelcomePage("/")
```
### 示例 2：处理静态文件

- 用 `StaticFileMiddleware` 从 `wwwroot` 目录中提供静态资源（图片、JS、CSS 等）。
- `wwwroot` 是 ASP.NET Core 默认可访问的公开目录。
    
```csharp
app.UseStaticFiles(); // 启用静态文件支持
```

---

## 6. 构建 Minimal API 应用（最简 API 应用）

这个部分介绍了如何通过添加中间件构建一个最基础的 Web API 项目：
### 中间件顺序：

1. **异常处理中间件（Exception Handling）**
2. **静态文件中间件**
3. **路由中间件（Routing）**
4. **终结点中间件（Endpoint Middleware）**

### 路由与终结点：

使用 `MapGet()` 方法定义路由终结点：

```csharp
app.MapGet("/", () => "Hello World!");
```

这是终结点，不是中间件，但由路由和终结点中间件一起执行。

---

## 7. 中间件顺序的重要性

- 中间件的执行顺序 **非常关键**。
- 早注册的中间件会先运行，也可以决定是否继续传递。
- 例如，错误处理中间件应该放在最前面，这样能捕获后面所有中间件中抛出的异常。

---

## 8. 总结

- Middleware 是 ASP.NET Core 的核心。
- 每个中间件负责一部分功能，比如：日志、安全、错误处理、路由等。
- 它们像积木一样串联在一起，形成一个请求处理的完整管道。
- 你需要理解：**如何添加、如何组合、顺序如何影响结果。**
