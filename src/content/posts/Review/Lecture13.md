---
# 必填
title: "ASP.NET期末复习Lecture13 : Entity Framework Core 简单使用"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 11
toc: true
abbrlink: 'asp-lecture13'
---

第十三次课简要讲解了如何将 EF Core 添加到 ASP. NET Core 应用程序中。

---
## 一、EF Core 简介与添加流程

Entity Framework Core (EF Core) 是一个轻量级、可扩展、开源的 ORM 框架，用于. NET 应用程序与数据库的交互。将 EF Core 添加到 ASP. NET Core 应用程序中是一个多步骤的过程：

1. **选择数据库提供程序**：如 Postgres、SQLite 或 MS SQL Server
2. **安装 EF Core NuGet 包**：根据选择的数据库安装相应包
3. **设计 DbContext 和数据模型**：定义实体类和上下文
4. **注册 DbContext 到 DI 容器**：通过依赖注入使用
5. **生成和应用迁移**：更新数据库架构

---
## 二、详细步骤解析

### 1. 选择数据库提供程序

EF Core 通过提供程序模型支持多种数据库，其模块化特性允许开发者使用相同的高级 API 操作不同数据库：

- PostgreSQL: Npgsql. EntityFrameworkCore. PostgreSQL
- Microsoft SQL Server: Microsoft. EntityFrameworkCore. SqlServer
- MySQL: MySql. Data. EntityFrameworkCore
- SQLite: Microsoft. EntityFrameworkCore. SQLite

本课程项目使用 Visual Studio 提供的 LocalDB (SQL Server 开发者版本)，需安装以下包：
```
Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.Design
Microsoft.EntityFrameworkCore.Relational
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Tools
```

### 2. 构建数据模型

EF Core 通过 DbContext 和实体类构建内部数据库模型，遵循"约定优于配置"原则：

```csharp
// 实体类示例
public class Country
{
    public int CountryId { get; set; }  // 自动识别为表的主键(Id后缀)
    public string CountryName { get; set; }
}

public class Employee
{
    public int EmployeeId { get; set; }  // 员工ID
    public string EmployeeName { get; set; }  // 姓名
    public string Title { get; set; }  // 职位
    public string Country { get; set; }  // 国籍
}
```

### 3. 定义 DbContext

DbContext 是 EF Core 的核心，用于所有数据库操作：

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    
    public DbSet<Employee> Employees { get; set; }  // 员工表
    public DbSet<Country> Countries { get; set; }  // 国家表
}
```

### 4. 注册数据上下文

将 AppDbContext 注册到依赖注入容器：

```csharp
// 配置连接字符串(appsettings.json)
"ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;
    Database=EmployeeManager;
    Trusted_Connection=True;
    MultipleActiveResultSets=true"
}

// 在Program.cs中注册
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options => 
    options.UseSqlServer(builder.Configuration["ConnectionStrings:DefaultConnection"]));

var app = builder.Build();
app.UseStaticFiles();
app.MapDefaultControllerRoute();
app.Run();
```

### 5. 连接字符串详解

连接字符串提供数据库连接所需信息：
- `Server`: 服务器主机名 (本课使用 LocalDB)
- `Database`: 数据库名称
- `Trusted_Connection`: 使用 Windows 身份验证
- `MultipleActiveResultSets`: 允许单个连接上执行多个活动 SQL 语句

---
## 三、总结

1. **数据库提供程序**：EF Core 通过不同提供程序支持多种数据库
2. **约定优于配置**：默认规则简化配置 (如 Id 后缀自动成为主键)
3. **DbContext**：数据库操作的核心入口点
4. **依赖注入**：通过 DI 容器管理 DbContext 生命周期
5. **连接字符串**：集中配置在 appsettings. Json 中
