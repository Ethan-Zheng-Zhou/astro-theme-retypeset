---
# 必填
title: "ASP.NET期末复习Lecture15 : 进阶 CRUD 操作"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 14
toc: true
abbrlink: 'asp-lecture14'
---

第十六课主要讲解了在 ASP. NET 应用开发中使用 Entity Framework Core 处理数据关系的三种类型：**一对多、一对一、多对多**。

---

### 一、一对多关系（Country ↔ Employee）

**目标**：按国家显示其下属员工列表。  

**实现步骤**：
1. **模型类配置**  
   - **Country 类**：定义反向导航属性 `IEnumerable<Employee> Employees`，表示一个国家对应多个员工。  
   - **Employee 类**：包含外键 `CountryID` 和导航属性 `Country`，表示员工属于一个国家。  
   ```csharp
   public class Country {
       public int CountryID { get; set; }
       public string CountryName { get; set; }
       public IEnumerable<Employee> Employees { get; set; } // 反向属性
   }

   public class Employee {
       public int EmployeeId { get; set; }  // 原讲义中拼写错误，应为 EmployeeId
       public string EmployeeName { get; set; }
       public int CountryID { get; set; }  // 外键
       public Country Country { get; set; } // 导航属性
   }
   ```

2. **仓库层查询**  
   使用 `Include` 方法加载关联数据，确保查询国家时同时获取其员工列表：  
   ```csharp
   public List<Country> SelectAll() {
       return context.Countries.Include(c => c.Employees).ToList();
   }
   ```

3. **视图展示**  
   在 Razor 视图中循环遍历国家及员工，生成嵌套表格：  
   ```html
   <table>
       @foreach (var country in Model) {
           <tr><td>@country.CountryName</td></tr>
           @if (country.Employees != null) {
               foreach (var employee in country.Employees) {
                   <tr style="background-color:lightgray">
                       <td>@employee.EmployeeName</td>
                   </tr>
               }
           }
       }
   </table>
   ```

---

### 二、一对一关系（Employee ↔ Contact）

**目标**：为员工添加联系方式（如电话、地址）。  

**实现步骤**：
1. **模型类配置**  
   - **Employee 类**：添加导航属性 `Contact`。  
   - **Contact 类**：定义外键 `EmployeeId` 和导航属性 `Employee`。  
   ```csharp
   public class Contact {
       public int ContactId { get; set; }
       public string Phone { get; set; }
       public string Address { get; set; }
       public int EmployeeId { get; set; } // 外键
       public Employee Employee { get; set; } // 导航属性
   }
   ```

2. **仓库层查询**  
   使用 `Include` 加载关联的联系方式：  
   ```csharp
   public List<Employee> SelectAll() {
       return context.Employees
           .Include(e => e.Country)
           .Include(e => e.Contact)
           .ToList();
   }
   ```

3. **视图展示**  
   在表格中显示员工及其联系方式：  
   ```html
   <td>@e.Contact.Phone</td>
   <td>@e.Contact.Address</td>
   ```

---

### 三、多对多关系（Employee ↔ Project）

**目标**：通过联结表实现员工与项目的多对多关系。 

**实现步骤**：
1. **定义联结类（Junction）**  
   存储 `EmployeeId` 和 `ProjectId` 作为外键：  
   ```csharp
   public class Junction {
       public int Id { get; set; }
       public int EmployeeId { get; set; }
       public Employee Employee { get; set; }
       public int ProjectId { get; set; }
       public Project Project { get; set; }
   }
   ```

2. **模型类更新**  
   - **Employee 和 Project 类**：添加导航属性指向联结类。  
   ```csharp
   public class Employee {
       // 其他属性...
       public ICollection<Junction> Junctions { get; set; } // 原讲义中拼写错误
   }

   public class Project {
       public int ProjectId { get; set; }
       public string ProjectName { get; set; }
       public ICollection<Junction> Junctions { get; set; }
   }
   ```

3. **仓库层查询**  
   使用 `Include` 和 `ThenInclude` 加载多级关联数据：  
   ```csharp
   public List<Employee> SelectAll() {
       return context.Employees
           .Include(e => e.Junctions)
           .ThenInclude(j => j.Project)
           .ToList();
   }
   ```

4. **视图展示**  
   显示员工参与的项目名称：  
   ```html
   <td>@string.Join(", ", e.Junctions.Select(j => j.Project.ProjectName))</td>
   ```

---

### 四、关键注意事项

1. **拼写修正**  
   - `IfnumerableEmployees` → `IEnumerable<Employee>`  
   - `TimmenshieldFunction` → `ICollection<Junction>`  

2. **EF Core 配置**  
   需在 `DbContext` 中通过 `OnModelCreating` 配置关系：  
   ```csharp
   // 一对多
   modelBuilder.Entity<Country>()
       .HasMany(c => c.Employees)
       .WithOne(e => e.Country)
       .HasForeignKey(e => e.CountryID);

   // 一对一
   modelBuilder.Entity<Employee>()
       .HasOne(e => e.Contact)
       .WithOne(c => c.Employee)
       .HasForeignKey<Contact>(c => c.EmployeeId);

   // 多对多
   modelBuilder.Entity<Junction>()
       .HasKey(j => new { j.EmployeeId, j.ProjectId }); // 建议使用复合主键
   ```

3. **视图层优化**  
   使用 Bootstrap 或 CSS 美化表格，并确保 Razor 语法正确（如闭合标签、循环嵌套）。

---

### 五、运行效果

- **国家-员工列表**：以树形结构展示国家及其下属员工。  
- **员工详细信息**：包含职位、国家名称、联系方式。  
- **项目信息**：显示员工参与的所有项目名称。

