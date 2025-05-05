---
# 必填
title: "ASP.NET期末复习Lecture10 :  依赖注入、路由与日志"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
toc: true
abbrlink: 'asp-lecture10'
---

第十次课主要讲解了 **依赖注入（Dependency Injection, DI）**、**URL 路由（Routing）** 和 **日志服务（Logging Service）** 三个重要概念，是 ASP.NET Core 中支撑可扩展性与可维护性的关键部分。

---
## 一、依赖注入（Dependency Injection）

### 1. 什么是 DI？

依赖注入是一种实现**控制反转（IoC）**的技术，它将对象的创建和绑定从对象内部转移到外部容器管理。简单说就是：

- **不是**在类内部 `new` 依赖对象
- **而是**通过构造函数、方法或属性接收依赖项
### 2. 为什么需要 DI？

在没有 DI 的情况下，一个类（例如 RegisterUser）如果要发送邮件，就必须：
- 创建邮件内容
- 配置邮件服务器
- 手动创建 `EmailSender` 类实例并调用发送逻辑

这样做的问题包括：
- **违反单一职责原则（SRP）**
- **高耦合，难以测试**
- **依赖关系隐式存在，需查源码才能发现**

### 3. 有 DI 后的改进

- 将 `EmailSender` 作为构造函数参数注入（构造函数注入）
- 由 **DI 容器（Container）** 创建所需的对象，并注入到使用它们的类中
- 依赖关系**显式化**

```csharp
public class RegisterUser {
    private readonly IEmailSender _emailSender;

    public RegisterUser(IEmailSender emailSender) {
        _emailSender = emailSender;
    }

    public void Register(...) {
        _emailSender.SendEmail(...);
    }
}
```

### 3. 在 ASP.NET Core 中使用 DI

ASP.NET Core 内置了一个轻量级的IoC容器，包含三个核心部分：

**服务注册**：在 `Program.cs` 中配置

```csharp
builder.Services.AddScoped<IUserService, UserService>();
```

**服务解析**：通过构造函数自动注入

```csharp
public class HomeController : Controller
{
    private readonly IUserService _userService;
    
    public HomeController(IUserService userService)
    {
        _userService = userService;
    }
}
```

**服务生命周期**：

- **Transient**：每次请求都创建新实例
- **Scoped**：每个HTTP请求一个实例
- **Singleton**：整个应用生命周期一个实例

| 生命周期      | 示例场景     | 创建时机       | 适用情况    |
| --------- | -------- | ---------- | ------- |
| Transient | 轻量级无状态服务 | 每次请求依赖时    | 日志服务    |
| Scoped    | 数据库上下文   | 每个HTTP请求一次 | EF Core |
| Singleton | 配置服务/缓存  | 应用启动时创建一次  | 全局配置    |

---

## 二、URL 路由（Routing）

URL 路由是 ASP.NET Core MVC 中一个核心机制，它负责将传入的 HTTP 请求映射到对应的控制器和动作方法。
### 1. 路由方式：约定路由（Convention Routing）

约定路由（也称为传统路由）是基于命名约定而非特性的路由方式。它不依赖 `[Route]` 特性，而是通过控制器和动作方法的名称自动匹配 URL。

默认的约定路由模式是：

```csharp
{controller}/{action}/{id?}
```

假设有以下控制器：

```csharp
public class ProductsController : Controller
{
    public IActionResult Index() { /*...*/ }
    public IActionResult Details(int id) { /*...*/ }
}
```

自动生成的URL路由：

- `/Products/Index` → `ProductsController.Index()`
- `/Products/Details/5` → `ProductsController.Details(5)`
### 2. 默认路由设置

```csharp
app.MapDefaultControllerRoute();
```

等同于：

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);
```

也可以自定义：

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=EmployeeManager}/{action=List}/{id?}"
);
```

这个配置表示：

1. **默认控制器**：`EmployeeManagerController`
2. **默认动作**：`List`
3. **可选ID参数**：`{id?}`

### 3. 高级路由配置

可以注册多个路由规则，按顺序匹配：

```csharp
// 特定路由（优先匹配）
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{action}/{id?}",
    defaults: new { controller = "Blog" }
);

// 默认路由
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);
```

**注意**：路由匹配是顺序敏感的，更具体的规则应该放在前面

---

## 三、日志服务（Logging Service）

### 1. 日志的作用

- 调试错误（Debug Errors）
- 追踪操作（Trace Operations）
- 分析使用行为（Analyze Usage）
### 2. 日志的结构

- **Log Entry**：日志条目，每条信息都有一个“等级（LogLevel）”
- ASP.NET Core定义了6种日志级别，按严重程度递增：

| 级别            | 数值  | 使用场景                      |
| ------------- | --- | ------------------------- |
| `Trace`       | 0   | 最详细的调试信息，可能包含敏感数据，仅开发环境使用 |
| `Debug`       | 1   | 开发调试信息，用于诊断问题             |
| `Information` | 2   | 常规操作信息，记录应用流程             |
| `Warning`     | 3   | 异常或意外事件，但不会导致应用失败         |
| `Error`       | 4   | 导致当前操作失败的错误，但应用仍可运行       |
| `Critical`    | 5   | 导致应用崩溃或灾难性失败的严重错误         |

![一条日志](https://cdn.ethanzhou.cn/i/2025/05/05/6818b660ae6cd.jpg)
### 3. ASP.NET Core 日志体系结构

- `ILogger`：用于写日志，有一个 `Log()` 方法。
- `ILoggerProvider`：用于创建日志实例（如 Console、File 等）
- `ILoggerFactory`：用于管理和协调 Provider 和 Logger

### 4. 添加 Console 日志

在 `Program.cs` 中配置：

```csharp
builder.Logging.AddConsole();
```

当然，你要注意它的位置：

![添加位置](https://cdn.ethanzhou.cn/i/2025/05/05/6818b71ea3cf5.jpg)
### 5. 设置日志级别过滤（Filtering）

在 `appsettings.json` 中设置日志输出级别：

```json
"Logging": {
  "LogLevel": {
    "Default": "Information",
    "Microsoft": "Warning",
    "Microsoft.Hosting.Lifetime": "Information"
  }
}
```

- 低于指定等级的日志不会被记录。
- 可针对命名空间和组件分别设定。

### 6. 在控制器中注入使用日志

```csharp
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index()
    {
        _logger.LogInformation("访问首页");
        
        try
        {
            // 业务逻辑
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "处理首页请求时出错");
        }
        
        return View();
    }
}
```

