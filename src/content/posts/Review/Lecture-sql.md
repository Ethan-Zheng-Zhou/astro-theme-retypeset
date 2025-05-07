---
# 必填
title: "ASP.NET开发: 使用 Entity Framwork Core 操作数据库"
published: 2025-05-07

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 15
toc: true
abbrlink: 'asp-lecture-sqlserver'
---

## 碎碎念

ASP. NET 开发在现如今已经算是"过时"的技术了，在网上搜索也很难找到高质量的博客/视频教程。 Ethan 因为选课失误不得不去学习这门技术，考点里重点之一就是使用 EF Core 操作数据库，苦苦搜寻找不到一个明确的文档教程，所以写下这篇文章以供南邮的学弟学妹们进行期末复习，文章里使用的例子均来自于《ASP. NET 应用开发 (双语) 》课程提供的课件与实验，请放心食用。

---
## 1. How to use EF Core?
### 1.1 EF Core 简介与添加流程

Entity Framework Core (EF Core) 是一个轻量级、可扩展、开源的 ORM 框架，用于. NET 应用程序与数据库的交互。将 EF Core 添加到 ASP. NET Core 应用程序中是一个多步骤的过程：

1. **选择数据库提供程序**：如 Postgres、SQLite 或 MS SQL Server
2. **安装 EF Core NuGet 包**：根据选择的数据库安装相应包
3. **设计 DbContext 和数据模型**：定义实体类和上下文
4. **注册 DbContext 到 DI 容器**：通过依赖注入使用
5. **生成和应用迁移**：更新数据库架构

### 1.2 安装 EF Core NuGet 包

EF Core 通过提供程序模型支持多种数据库，其模块化特性允许开发者使用相同的高级 API 操作不同数据库：

- PostgreSQL: Npgsql. EntityFrameworkCore. PostgreSQL
- Microsoft SQL Server: Microsoft. EntityFrameworkCore. SqlServer
- MySQL: MySql. Data. EntityFrameworkCore
- SQLite: Microsoft. EntityFrameworkCore. SQLite

《ASP. NET 应用开发 (双语) 》课程项目使用 Visual Studio 提供的 LocalDB (SQL Server 开发者版本)，需安装以下包：

```
Microsoft.EntityFrameworkCore 
Microsoft.EntityFrameworkCore.Design Microsoft.EntityFrameworkCore.Relational Microsoft.EntityFrameworkCore.SqlServer 
Microsoft.EntityFrameworkCore.Tools
```

你可以在 Visual Studio 2022 中通过如下路径找到 Nuget 包安装工具：

```
工具 → Nuget包管理器 → 管理解决方案的 Nuget 程序包
```

并在工具的搜索栏中搜索安装我们需要的五个包。

![Nuget 包安装工具](https://cdn.ethanzhou.cn/i/2025/05/07/681b17a221b0f.jpg)

### 1.3 设计 DbContext 和数据模型

EF Core 通过 DbContext（可以理解为你数据库的抽象类）和实体模型构建其内部模型。EF Core 在将实体定义为 POCO 类时，非常倾向于采用约定而非配置的方法。举个例子： EF Core 将 id 后缀的属性标识为表的主键，而不需要你手动配置。

> **什么是 POCO 类？**
>
> 在.NET环境中，一个POCO类通常不会继承自特定的基类也不实现特定的接口，它完全由你自己定义。简单来说：**它就是一个纯净的实体类。**

我们以跨国公司员工管理系统为背景，定义其需要的数据模型（POCO 类）：

```C#
/Model/Country.cs

public class Country
{ 
	public int CountryId { get; set; } // 自动识别为表的主键(Id后缀)
	public string CountryName { get; set; }
}

```

```C#
/Model/Employee.cs

public class Employee 
{ 
	public int EmployeeId { get; set; } // 员工ID 
	public string EmployeeName { get; set; } // 姓名 
	public string Title { get; set; } // 职位 
	public string Country { get; set; } // 国籍 
}
```

除了定义实体之外，我们还需要定义应用的 DbContext 类，DbContext 是应用程序中 EF Core 的核心，用于所有数据库调用，有两个知识点：

- 构造函数接收一个 `DbContextOptions<T>` 对象，其中 T 是 Context 类。
- 构造函数参数将为 Entity Framework Core 提供连接到数据库服务器所需的配置信息。

```C#
/Model/AppContext.cs

using Microsoft.EntityFrameworkCore; 

public class AppDbContext : DbContext 
{ 
	public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
	
	public DbSet<Employee> Employees { get; set; } // 员工表 
	public DbSet<Country> Countries { get; set; } // 国家表
}
```

### 1.4 数据库的配置

与 ASP 中的任何其他服务一样，.Net Core在依赖注入（DI）容器中注册 AppDbContext。EF Core 为此提供了一个通用的 `AddDbContext<T>` 扩展方法。

当然，在注册 DbContext 前，我们需要先进行数据库的配置，分两步完成：

- 在 `appsetting.json` 中设置连接字符串
- 在 `Program.cs` 配置应用程序

在你的 ` appsetting. json` 中添加如下的行

```json
"ConnectionStrings": 
{ 
"DefaultConnection": "Server=(localdb)\\MSSQLLocalDB; 			 Database=EmployeeManager; 
Trusted_Connection=True;
MultipleActiveResultSets=true" 
}
```

- `(localdb)\\MSSQLLocalDB` 指定要连接的服务器实例
- `Database=EmployeeManager` 指定要连接的数据库名称
- `Trusted_Connection=True` 指定使用 Windows 身份验证
- `MultipleActiveResultSets=True` 指定启用多活动结果集(MARS)

在你的 `Program.cs` 启用数据库服务

```C#
using Microsoft.EntityFrameworkCore;  

var builder = WebApplication.CreateBuilder(args); 

// 启用数据库服务
builder.Services.AddDbContext<AppDbContext>( options => options.UseSqlServer (builder.Configuration["ConnectionStrings:DefaultConnection"]));  

var app = builder.Build();  
app.UseStaticFiles(); 
app.MapDefaultControllerRoute();  
app.Run();
```

`AddDbContext` 方法为实体框架核心上下文类创建服务，`UseSqlServer` 接收一个选项对象用于选择数据库提供程序，` IConfiguration ` 服务用于获取数据库的连接字符串。

现在，我们有了一个名为 AppDbContext 的 DbContext，以及与数据库对应的数据模型。很显然，我们想要连接的数据库 EmployeeManager 中还没有与数据模型对应的模式。我们需要使用迁移生成 SQL 语句，使得数据库模式与应用程序的数据模型保持同步。

迁移是应用程序中的一种 `C#` 代码文件，它定义了数据模型的更改方式——添加了哪些列、新实体等，提供了随着时间的推移数据库模式如何作为应用程序的一部分演变的记录，因此模式始终与应用程序的数据模型同步。

### 1.5 如何创建迁移

我们使用命令行工具从迁移中创建新数据库，或通过向现有数据库应用新的迁移来更新现有数据库。我们安装的 EF 工具包用于从命令行管理数据库，并管理提供数据访问的项目包。

安装和卸载 EF 工具包的方式：

```shell
dotnet tool uninstall ‐‐global dotnet‐ef 
dotnet tool install ‐‐global dotnet‐ef ‐‐version x.x.x
```

创建迁移的方式：

```
dotnet ef migrations add InitialSchema
```

![创建结果](https://cdn.ethanzhou.cn/i/2025/05/07/681b2441b1852.jpg)

此命令在项目的 Migrations 文件夹中创建三个文件：

-  Migration file
-  Migration designer. cs file
-  AppDbContextModelSnapshot. cs

![](https://cdn.ethanzhou.cn/i/2025/05/07/681b29d85b744.jpg)

`Timestamp_MigrationName. cs ` 格式的迁移文件（Migration file）描述了对数据库采取的操作，例如创建表或添加列。

`Migration designer.cs ` 文件描述了生成迁移时 EF Core 的内部数据模型。

`AppDbContextModelSnapshot. cs` 文件描述了 EF Core 当前的内部模型,，添加另一个迁移时会更新此文件，因此它应始终与当前（最新）迁移相同。EF Core 可以在创建新迁移时使用 `AppDbContextModelSnapshot. cs`  来确定数据库的先前状态，而无需直接与数据库交互。


特别要注意的是，添加迁移不会更新数据库本身中的任何内容，必须运行其他命令才能将迁移应用于数据库。

```shell
dotnet ef database update
```

数据库更新命令执行四个步骤：
- 构建应用程序；
- 加载在应用程序的 Program. Cs 中配置的服务，包括 AppDbContext；
- 检查 AppDbContext 中的数据库；连接字符串存在，如果不存在，则创建它；
- 通过应用任何未应用的迁移来更新数据库

![](https://cdn.ethanzhou.cn/i/2025/05/07/681b26c44feab.jpg)

看到类似的输出说明更新成功。

当然还有一些其他有用的指令可以使用，例如显示迁移将在数据库中执行的 SQL 命令:

```shell
dotnet ef migrations script
```

删除添加到项目中的最新迁移：

```shell
dotnet ef migrations remove
```

完全删除数据库：

```shell
dotnet ef database drop ‐‐force
```

### 1.6 在 VS 中查看数据库

通过如下路径打开数据库连接工具：

```
工具 → 连接到数据库
```

![](https://cdn.ethanzhou.cn/i/2025/05/07/681b275174a1d.jpg)

选择数据源为 Microsoft SQL Server：

![](https://cdn.ethanzhou.cn/i/2025/05/07/681b279b9b9fb.jpg)

输入您在连接字符串的服务器名称和表名称：

![](https://cdn.ethanzhou.cn/i/2025/05/07/681b28e2d062a.jpg)

连接成功后你会看到我们迁移成功的数据库：

![](https://cdn.ethanzhou.cn/i/2025/05/07/681b2bc47b5d6.jpg)

---
## 2. Data Operation

数据操作的关键是 `DbSet<T>` 类。

```C#
public class AppDbContext: DbContext 
{  
	...... 
	public DbSet<Employee> Employees { get; set; } 
	public DbSet<Country> Countries { get; set; } 
}
```

- 2.1 节 ~ 2.3 节主要介绍查找数据的方法
- 2.4 节 ~ 2.7 节主要介绍增加数据的方法
- 2.8 节 ~ 2.9 节主要介绍更改数据和删除数据的方法
### 2.1 通过键值读取对象

`Find（key）` 方法读取表中具有指定键的行，并返回表示该行的对象。

```C#
dbContext.Employees.Find(id)
```

### 2.2 查询所有的对象

要检索数据库中存储的所有数据对象，我们需要读取上下文类的 Employees 属性的值，该属性返回一个 `DbSet<Employee>`  对象。EF Core 在枚举 `DbSet<t>`  属性之前不会从数据库读取数据。

想象你有一个快递仓库（数据库），里面有一堆包裹（`Employee` 数据）。  `DbContext` 就像你的仓库管理员，而 `DbSet<Employee>` 是管理员手里的一个 **未拆封的包裹清单**（还没真正去仓库里拿东西）。当你真正用代码遍历清单（比如 `. ToList ()` 或 `foreach`），管理员才会跑去仓库，把包裹一个个搬出来（从数据库查询数据）。   

```c#
Var employees = dbContext.Employees.ToList (); // 这时才查数据库！
```

### 2.3 在应用里查询对象并以表格形式输出

继续我们在第一节提到的跨国公司员工管理系统的例子来讲解数据操作，我们首先创建两个仓储接口 (repository interface) 用于定义对数据对象的操作：

```C#
/Model/IEmployeeRepository.cs

public interface IEmployeeRepository
{
	List<Employee> SelectAll();
}
```

```C#
/Model/ICountryRepository.cs

public interface IEmployeeRepository
{
	List<Country> SelectAll();
}
```

两个接口的具体实现如下：

```c#
/Model/EmployeeRepository.cs

public class EmployeeRepository: IEmployeeRepository
{  
	private AppDbContext context; 
	public EmployeeRepository (AppDbContext ctx) 
	{ 
		context = ctx; 
	} 
	public List<Employee> SelectAll () 
	{ 
		return context.Employees.ToList ();
	} 
}

```

```c#
/Model/EmployeeRepository.cs

public class CountryRepository: ICountryRepository
{  
	private AppDbContext context; 
	public CountryRepository (AppDbContext ctx) 
	{ 
		context = ctx; 
	} 
	public List<Country> SelectAll () 
	{ 
		return context.Countries.ToList ();
	} 
}
```

当然我们要进行依赖注入的配置。`AddTransient` 方法确保每次解析对 ` IRepository` 的依赖关系时都会创建一个新的 ` Repository` 对象。

```C#
services.AddTransient<IEmployeeRepository,EmployeeRepository>(); services.AddTransient<ICountryRepository, CountryRepository>();
```

我们想使用取出来的所有员工数据对象，控制器的创建就可以如下：

```C#
/Controller/EmployeeController.cs

Public class EmployeeManagerController : Controller 
{  
	private IEmployeeRepository repo; 
	public EmployeeManagerController (IEmployeeRepository r) 
	{  
		Repo = r; 
	}  
	public IActionResult List () 
	{  
		return View (repo.SelectAll ());
	} 
}

```

控制器对应的 Razor 视图如下：

```html
@model IEnumerable<Employee>  
<h2>Employee Info List</h2> 
<table> 
	<tr> 
		<td>Employee ID</td> 
		<td>Employee Name</td> 
		<td>Employee Title</td>
		<td>Employee Country</td>
	</tr> 
	@foreach (Employee e in Model)
	 {  
	 	<tr> 
	 		<td>@e.EmployeeId</td> 
		 	<td>@e.EmployeeName</td> 
		 	<td>@e.Title</td> 
	 		<td>@e.Country</td>
	  	</tr> 
	  } 
</table>
```

### 2.4 在数据库中存储数据

通过 ` DbSet<T>`  调用的 `Add` 和 `AddRange`方法让 EF Core 存储数据的更改。`SaveChanges ` 方法将对 EF Core 管理的未完成更改保存到数据库中。是不是有点像 git 里的 `add` 和 `push` ?

我们在 2.2 节给出了这个例子：

> 想象你有一个快递仓库（数据库），里面有一堆包裹（`Employee` 数据）。  `DbContext` 就像你的仓库管理员，而 `DbSet<Employee>` 是管理员手里的一个 **未拆封的包裹清单**（还没真正去仓库里拿东西）。当你真正用代码遍历清单（比如 `. ToList ()` 或 `foreach`），管理员才会跑去仓库，把包裹一个个搬出来（从数据库查询数据）

`Add` 和 `AddRange` 方法就好像管理员往清单上加了包裹数据，`SaveChanges` 方法才是管理员把包裹往仓库里面放。

```C#
context.Employees.Add(newEmployee); 
context.SaveChanges();
```

### 2.5 在应用中添加测试数据

我们更新一下 `ICountryRepository`：

```C#
/Model/ICountryRepository.cs

public interface IEmployeeRepository
{
	List<Country> SelectAll();
	// 添加增加测试数据的方法
	void CreateTestData();
}
```

方法实现如下：

```C#
public void CreateTestData()
{
	//删除数据表所有数据
	context.Countries.RemoveRange(context.Countries):
	//添加测试数据
	context.Countries.AddRange(
		new Country[]
		{
			new Country(CountryName="USA"]
   			new Country(CountryName=~UK3.
   			new Country(CountryName="CN”]
		}
	)
	//保存到数据库
	context.SaveChanges();
}
```

`IEmployeeRepository` 的更新与之类似，这里不过多赘述。

更新 `EmployeeController.cs`:

```C#
/Controller/EmployeeController.cs

Public class EmployeeManagerController : Controller 
{  
	private IEmployeeRepository repo; 
	private ICountryRepository crepo;
	public EmployeeManagerController (IEmployeeRepository r, ICountryRepository cr) 
	{  
		repo = r; 
		crepo = cr;
	}  
	public IActionResult List () 
	{  
		return View (repo.SelectAll ());
	} 
	public IActionResult CreateTestData()
	{
		repo.CreateTestData();
		crepo.CreateTestData();
		return RedirectToAction("list");
	}
}

```

更新该控制器对应 Razor 视图：

```html
<form>
	<button asp-action="CreateTestData">Create Test Data</button>
</form>
```

此时运行项目，您应该可以看到测试数据的显示。

### 2.6 模拟用户通过表单输入的场景

为了更好的模拟用户通过表单输入登记的场景，我们可以往 Repository Interface 里添加如下方法：

```C#
void Insert(Employee emp);
```

具体实现如下：

```C#
public void Insert(Employee emp) 
{  
	context.Employees.Add(emp); 
	context.SaveChanges(); 
}
```

增加控制器方法以使用 Repository Interface 的新增功能：

```C#
public IActionResult Insert(Employee model)
{
	repo.Insert(Employee emp);
}
```

增加实现插入功能的 Razor 视图：

```html
@model Employee
<h2>Insert New Employee</h2>

<form asp-controller="EmployeeManager" asp-action="Insert" method="post">
	<table>
	<tr>
		<td><label>Name :</label></td>
		<td><input type="text" asp-for="EmployeeName" /></td>
	</tr>
	<tr>
		<td><label>Title:</label></td>
		<td><input type="text" asp-for="Title" />
	</tr>
	<tr>
		<td><label>Country:</label></td>
		<td><input type="text" asp-for="Country" /></td>
	</tr>
	</table>
	<button type="submit">Save</button>
</form>
```

### 2.7 在列表中选择输入项

想像一种场景，用户想在下拉框里选择国家，如图所示:

![](https://cdn.ethanzhou.cn/i/2025/05/07/681b557ccb0b0.jpg)

我们可以在控制器里增加如下的方法：

```C#
public IActionResult Insert()
{
	List<SelectListItem> countries =(from c in crepo.SelectAll()orderby c.CountryName ascending select new SelectListItem()
	{
		Text = c .CountryName.
		Value = c.CountryName
	}).ToList();
	ViewBag.Countries = countries;
	return View();
}
```

在对应的 Razor 视图中添加如下的复选框：

```C#
<td> 
	<select asp‐for="Country" asp‐items="@ViewBag.Countries"> 
		<option value="">Please select</option> 
	</select> 
</td>
```

大功告成！

### 2.8 更新数据

更新数据可以使用 `Update` 方法：

```C#
context.Employees.Update(changedEmployee); 
context.SaveChanges();
```

### 2.9 删除数据

删除数据可以使用 `Remove` 方法：

```C#
context.Employees.Remove(e); 
context.SaveChanges();
```