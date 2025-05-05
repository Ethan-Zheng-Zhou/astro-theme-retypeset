---
# 必填
title: "ASP.NET期末复习Lecture8 :  Repository 模式和 Razor视图"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 6
toc: true
abbrlink: 'asp-lecture8'
---

 第八次课聚焦于 ASP.NET MVC 中的 **Repository 模式** 完整实现，以及 **视图（View）与 Razor 语法** 的基本使用：

---
## 一、Repository 模式简介

### 1. 为什么使用 Repository 模式？

- **中介作用**：在数据访问层与控制器等上层模块之间起桥梁作用；
- **封装数据访问逻辑**：Controller 不直接操作数据库，提高代码复用性和可维护性；
- **模拟数据源**：初期使用内存列表（List），后期可替换为数据库访问。

---

## 二、实现 Repository 模式的步骤

### 1️. 定义 Repository 接口

![Repository接口](https://cdn.ethanzhou.cn/i/2025/05/05/68187e5d8dba1.jpg)

```csharp
public interface IEmployeeRepository
{
    List<Employee> SelectAll();  // 查询所有员工
}
```

### 2️. 实现 Repository 类

```csharp
public class EmployeeRepository : IEmployeeRepository
{
	// 将员工数据保存在一个列表中
    private List<Employee> data = new List<Employee>();
	// 构造函数，添加初始数据
    public EmployeeRepository()
    {
        data.Add(new Employee { EmployeeId = 1, EmployeeName = "Adam", Title = "manager", Country = "USA" });
        data.Add(new Employee { EmployeeId = 2, EmployeeName = "Bob", Title = "president", Country = "UK" });
        data.Add(new Employee { EmployeeId = 3, EmployeeName = "Cathy", Title = "sale", Country = "USA" });
    }

    public List<Employee> SelectAll()
    {
        return data;
    }
}
```

### 3️. 控制器中使用 Repository

```csharp
using SampleApplication.Models;

public class EmployeeManagerController : Controller
{
    private IEmployeeRepository repo;

    public EmployeeManagerController(IEmployeeRepository r)
    {
    	// 获取接口实例
        repo = r;
    }

    public IActionResult List()
    {
        return View(repo.SelectAll());
    }
}
```

控制器中不再关心数据从哪儿来，只调用接口，符合 **依赖倒置原则（DIP）**。当然，出现了一个问题，我们为什么能“直接调用未实现的接口”？
### 4. 依赖注入容器的工作机制

在 ASP.NET Core 的 `Program.cs` 或 `Startup.cs` 中，一定有类似这样的注册代码：

```csharp
// 注册接口与实现的映射关系
builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();
```

这意味着：
- 当控制器需要 `IEmployeeRepository` 时
- 容器会自动提供 `EmployeeRepository` 的实例

### 5. 控制器的构造函数注入

```csharp
public EmployeeManagerController(IEmployeeRepository r)
{
    repo = r;  // 这里实际接收到的是 EmployeeRepository 实例
}
```

虽然参数类型是接口，但运行时注入的是具体实现类的实例。

---

## 三、视图（View）基础

### 1. View 文件夹结构约定：

- 对应控制器的视图保存在 `Views/控制器名/`
- 通用视图保存在 `Views/Shared/`

---

## 四、Razor View 引擎

### 1. Razor 是 ASP.NET 默认的视图引擎，特点包括：

- 使用 `@` 标识符嵌入 C# 表达式；
- HTML 和 C# 代码混合；
- 生成 `.cshtml` 文件。

---

## 五、视图调用方式

### 1. 方式一：默认名称匹配

- 当一个操作方法调用 View 方法时，它会创建一个 ViewResult，“告诉”MVC 框架使用默认约定来定位视图
- Razor 视图引擎查找与动作方法同名的视图，并添加 cshtml 文件扩展名。

```csharp
// EmployerController.cs
public IActionResult List()
{
    return View(repo.SelectAll()); // 匹配Views/EmployeeManager/List.cshtml
}
```

### 2. 方式二：手动指定视图名

```csharp
return View("Watersports", prod);
```

---

## 六、Razor 语法说明

### 1️. 表达式

```html
<h1>Listing @items.Length items.</h1>
```

### 2️. C# 代码块

```csharp
@{
    string s = "Hello!";
    ViewBag.Title = "Employee List";
}
```

### 3️. 循环语法

```csharp
<ul>
@foreach (var auction in auctions)
{
    <li><a href="@auction.Href">@auction.Title</a></li>
}
</ul>
```

### 4️. 条件语句

```csharp
@if (DateTime.Today.Year == 2025) {
    <span>Current year</span>
}
else {
    <span>Past year</span>
}
```

---
## 七、总结

|内容|作用|
|---|---|
|`IRepository` 接口|解耦数据访问逻辑|
|`Razor`|构建动态视图的模板语言|
|`View()`|呈现页面，传递模型|
|控制器中调用 `repo.SelectAll()`|获取数据并传给视图|

