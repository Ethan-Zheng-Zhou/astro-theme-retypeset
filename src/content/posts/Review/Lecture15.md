---
# 必填
title: "ASP.NET期末复习Lecture15 : 进阶 CRUD 操作"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 13
toc: true
abbrlink: 'asp-lecture14'
---

第十五课主要关注 CRUD 操作的复杂关系

---
## 一、数据写入操作详解

### 1. 核心数据写入方法

EF Core 提供了两种主要方法来实现数据写入：

```csharp
// 添加单个实体
context.Employees.Add(newEmployee); 

// 添加多个实体
context.Employees.AddRange(employeeList);

// 保存变更到数据库
context.SaveChanges();
```

**SaveChanges () 工作原理**：
1. 跟踪所有实体状态变更 (新增、修改、删除)
2. 生成并执行相应的 SQL 语句
3. 返回受影响的行数
4. 提交事务 (默认行为)

### 2. 测试数据生成实现

**仓储接口扩展**：
```csharp
public interface ICountryRepository {
    List<Country> SelectAll();
    void CreateTestData(); // 新增测试数据方法
}

public interface IEmployeeRepository {
    List<Employee> SelectAll();
    void CreateTestData(); // 新增测试数据方法
}
```

**具体实现示例**：

```csharp
public void CreateTestData()
{
    // 清空现有数据
    context.Employees.RemoveRange(context.Employees.ToList());
    context.Countries.RemoveRange(context.Countries.ToList());
    
    // 添加国家数据
    var countries = new List<Country> {
        new Country { CountryName = "USA" },
        new Country { CountryName = "UK" }
    };
    context.Countries.AddRange(countries);
    
    // 添加员工数据
    var employees = new List<Employee> {
        new Employee { EmployeeName = "John", Title = "Manager", Country = "USA" },
        new Employee { EmployeeName = "Alice", Title = "Developer", Country = "UK" }
    };
    context.Employees.AddRange(employees);
    
    context.SaveChanges();
}
```

## 二、员工插入功能完整实现

---
### 1. 仓储层实现

```csharp
public void Insert(Employee emp)
{
    // 数据验证逻辑可在此添加
    context.Employees.Add(emp);
    context.SaveChanges(); // 立即持久化
}
```

### 2. 控制器实现

```csharp
public class EmployeeManagerController : Controller
{
    private readonly IEmployeeRepository empRepo;
    private readonly ICountryRepository countryRepo;
    
    public EmployeeManagerController(IEmployeeRepository eRepo, ICountryRepository cRepo)
    {
        empRepo = eRepo;
        countryRepo = cRepo;
    }
    
    [HttpGet]
    public IActionResult Insert()
    {
        // 准备国家下拉列表数据
        ViewBag.Countries = countryRepo.SelectAll()
            .OrderBy(c => c.CountryName)
            .Select(c => new SelectListItem {
                Text = c.CountryName,
                Value = c.CountryName
            }).ToList();
        return View();
    }
    
    [HttpPost]
    public IActionResult Insert(Employee model)
    {
        if(ModelState.IsValid)
        {
            empRepo.Insert(model);
            return RedirectToAction("List");
        }
        return View(model);
    }
}
```

### 3. 视图实现 (Razor)

```html
@model Employee

<h2>Insert New Employee</h2>
<form asp-controller="EmployeeManager" asp-action="Insert" method="post">
    <table>
        <tr>
            <td><label asp-for="EmployeeName"></label></td>
            <td><input asp-for="EmployeeName" /></td>
            <td><span asp-validation-for="EmployeeName"></span></td>
        </tr>
        <tr>
            <td><label asp-for="Title"></label></td>
            <td><input asp-for="Title" /></td>
            <td><span asp-validation-for="Title"></span></td>
        </tr>
        <tr>
            <td><label asp-for="Country"></label></td>
            <td>
                <select asp-for="Country" asp-items="@(ViewBag.Countries as List<SelectListItem>)">
                    <option value="">Please select</option>
                </select>
            </td>
            <td><span asp-validation-for="Country"></span></td>
        </tr>
    </table>
    <button type="submit">Save</button>
</form>
```

---
## 三、数据更新与删除操作

### 1. 更新实体流程

```csharp
// 1. 查询要修改的实体
var employee = context.Employees.Find(id);

// 2. 修改属性
employee.Title = "Senior " + employee.Title;

// 3. 标记为已修改(自动跟踪时可选)
context.Entry(employee).State = EntityState.Modified;

// 4. 保存更改
context.SaveChanges();
```

### 2. 删除实体流程

```csharp
// 方法1：先查询后删除
var employee = context.Employees.Find(id);
context.Employees.Remove(employee);
context.SaveChanges();

// 方法2：直接附加删除(无需查询)
var employee = new Employee { EmployeeId = id };
context.Employees.Attach(employee);
context.Employees.Remove(employee);
context.SaveChanges();
```

---
## 四、LINQ 解析

### 1. LINQ 核心组件

| 组件 | 说明 | 示例 |
|------|------|------|
| 扩展方法 | 提供查询功能 | Where, OrderBy, Select |
| LINQ 提供程序 | 执行 LINQ 表达式 | LINQ to Objects, LINQ to Entities |
| 查询语法 | SQL 风格语法 | `from x in collection where...` |
| 方法语法 | 链式方法调用 | `collection.Where(x => ...)` |

### 2. LINQ 查询三阶段

**数据源准备**：
```csharp
// 数组(实现IEnumerable<T>)
string[] names = { "Pam", "Jim", "Dwight" };

// EF Core DbSet(实现IQueryable<T>)
var dbContext = new AppDbContext();
var employees = dbContext.Employees;
```

**查询创建**：
```csharp
// 查询语法
var query1 = from e in employees
             where e.Title.Contains("Manager")
             orderby e.EmployeeName
             select e;

// 方法语法
var query2 = employees
    .Where(e => e.Title.Contains("Manager"))
    .OrderBy(e => e.EmployeeName);
```

**查询执行 (延迟执行)**：
```csharp
// 立即执行方式
var result1 = query1.ToList();  // 转换为列表
var result2 = query2.ToArray(); // 转换为数组

// 枚举时执行
foreach(var emp in query1) 
{
    Console.WriteLine(emp.EmployeeName);
}
```

---
## 五、数据关系建模

### 1. 实体关系配置

**改进后的 Employee 类**：
```csharp
public class Employee
{
    public int EmployeeId { get; set; }
    public string EmployeeName { get; set; }
    public string Title { get; set; }
    
    // 外键属性
    public int CountryId { get; set; }
    
    // 导航属性
    public Country Country { get; set; }
}
```

**Country 类**：
```csharp
public class Country
{
    public int CountryId { get; set; }
    public string CountryName { get; set; }
    
    // 导航属性(一对多)
    public ICollection<Employee> Employees { get; set; }
}
```

### 2. 关联数据查询

**使用 Include 加载关联数据**：

```csharp
public List<Employee> SelectAll()
{
    return context.Employees
        .Include(e => e.Country)  // 预加载国家数据
        .ToList();
}
```

**视图中的关联数据展示**：

```html
@foreach (var e in Model)
{
    <tr>
        <td>@e.EmployeeId</td>
        <td>@e.EmployeeName</td>
        <td>@e.Title</td>
        <td>@e.Country?.CountryName</td> <!-- 安全导航操作符 -->
    </tr>
}
```

---
## 六、总结

1. **状态跟踪**：EF Core 自动跟踪实体状态变化 (Added/Modified/Deleted/Unchanged)
2. **工作单元模式**：多个操作通过 SaveChanges () 原子提交
3. **延迟执行**：LINQ 查询在真正需要数据时才执行
4. **显式加载**：通过 Include/ThenInclude 控制关联数据加载
5. **导航属性**：简化关联数据访问，支持懒加载 (需额外配置)
6. **视图数据传递**：ViewBag/ViewData 作为 ViewModel 的补充

