---
# 必填
title: "ASP.NET期末复习Lecture14 : 数据库迁移和基础 CRUD 操作"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 12
toc: true
abbrlink: 'asp-lecture14'
---

第十四课主要讲解数据库迁移和基础 CRUD 操作

---
## 一、迁移 (Migrations) 概念与作用

迁移是 EF Core 中管理数据库架构变更的核心机制，它解决了应用程序数据模型与数据库架构同步的问题。迁移本质上是一组 `C#` 代码文件 ，记录了数据模型的变更历史 (如新增表、添加列等)。

**迁移的核心价值**：
1. **版本控制**：记录数据库架构的完整演变历史
2. **团队协作**：确保团队成员使用相同的数据库结构
3. **环境一致性**：使开发、测试和生产环境的数据库保持同步
4. **回滚能力**：可以撤销不成功的数据库变更
---
## 二、EF Core 工具链详解

### 1. 安装与配置 EF 工具

在使用迁移前，需要安装全局 EF Core 命令行工具：

```bash
# 卸载旧版本(如有需要)
dotnet tool uninstall --global dotnet-ef

# 安装指定版本(需与项目EF Core版本匹配)
dotnet tool install --global dotnet-ef --version x.x.x
```

### 2. 迁移相关命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `migrations add` | 创建新迁移 | `dotnet ef migrations add InitialSchema` |
| `database update` | 应用迁移到数据库 | `dotnet ef database update` |
| `migrations script` | 生成迁移 SQL 脚本 | `dotnet ef migrations script` |
| `migrations remove` | 删除最新迁移 | `dotnet ef migrations remove` |
| `database drop` | 删除数据库 | `dotnet ef database drop --force` |

---
## 三、迁移工作流程详解

### 1. 创建初始迁移

```bash
dotnet ef migrations add Initial
```

**生成的文件结构**：
```
Migrations/
├── 20200815033848_Initial.cs       # 主迁移文件
├── 20200815033848_Initial.Designer.cs # 设计器文件
└── AppDbContextModelSnapshot.cs    # 当前模型快照
```

**文件作用**：
- **时间戳_迁移名. Cs**：包含 `Up()` (应用迁移) 和 `Down()` (回滚迁移)方法
- **设计器文件**：EF Core 内部使用的元数据
- **快照文件**：记录当前数据模型的完整状态

### 2. 应用迁移到数据库

```bash
dotnet ef database update
```

**执行过程**：
1. 构建应用程序
2. 加载 Program.cs 中配置的服务 (包括 AppDbContext)
3. 检查数据库是否存在，不存在则创建
4. 应用所有未执行的迁移

### 3. 验证数据库

在 Visual Studio 中使用"SQL Server 对象资源管理器"：
1. 连接 LocalDB 实例：`(localdb)\MSSQLLocalDB`
2. 查看生成的数据库结构
3. 检查 `__EFMigrationsHistory` 表 (记录已应用的迁移)

---
## 四、数据操作基础

### 1. 核心查询方法

数据操作的关键是 `DbSet<T>` 类。它被用作数据库上下文类定义的属性的结果:

```csharp
public class AppDbContext:DbContext 
{ 
	public DbSet<Employee> Employees { get; set; } 
	public DbSet<Country> Countries { get; set; } 
}
```

```csharp
// 按主键查询
var employee = context.Employees.Find(id);

// 查询所有记录
var allEmployees = context.Employees.ToList();

// 延迟执行示例
var query = context.Employees.Where(e => e.Country == "USA"); // 未立即执行
var results = query.ToList(); // 实际执行查询
```

### 2. 仓储模式实现

**接口定义**：
```csharp
public interface IEmployeeRepository
{
    List<Employee> SelectAll();
}

public interface ICountryRepository
{
    List<Country> SelectAll();
}
```

**具体实现**：
```csharp
public class EmployeeRepository : IEmployeeRepository
{
    private AppDbContext context;
    
    public EmployeeRepository(AppDbContext ctx)
    {
        context = ctx;
    }
    
    public List<Employee> SelectAll()
    {
        return context.Employees.ToList();
    }
}
```

### 3. 依赖注入配置

```csharp
// 在Program.cs中注册服务
builder.Services.AddTransient<IEmployeeRepository, EmployeeRepository>();
builder.Services.AddTransient<ICountryRepository, CountryRepository>();
```

**AddTransient 特点**：
- 每次请求都会创建新实例
- 适合轻量级、无状态的服务

---
## 五、控制器与视图集成

### 1. 控制器实现

```csharp
public class EmployeeManagerController : Controller
{
    private IEmployeeRepository repo;
    
    public EmployeeManagerController(IEmployeeRepository r)
    {
        repo = r;
    }
    
    public IActionResult List()
    {
        return View(repo.SelectAll());
    }
}
```

### 2. 视图实现 (Razor 语法)

```html
@model List<Employee>

<table>
    <tr>
        <th>Employee ID</th>
        <th>Employee Name</th>
        <th>Title</th>
        <th>Country</th>
    </tr>
    @foreach (var emp in Model)
    {
        <tr>
            <td>@emp.EmployeeId</td>
            <td>@emp.EmployeeName</td>
            <td>@emp.Title</td>
            <td>@emp.Country</td>
        </tr>
    }
</table>
```

---
## 六、新增迁移的典型场景

当应用程序演进时，常见的模型变更包括：
1. 添加新属性到现有实体
2. 添加新实体类
3. 删除废弃的类
4. 修改关系配置

**新增迁移流程**：
1. 修改实体类
2. 创建新迁移：`dotnet ef migrations add AddNewProperty`
3. 应用迁移：`dotnet ef database update`

---
## 七、总结

1. **迁移即代码**：数据库变更通过 C #代码管理 ，而非直接操作数据库
2. **两阶段提交**：先 `add` 生成迁移，再 `update` 应用变更
3. **DbContext 生命周期**：默认作用域 (Scoped)，每个请求一个实例
4. **延迟执行**：LINQ 查询在枚举时才实际执行
5. **仓储模式**：抽象数据访问层，提高可测试性和可维护性
