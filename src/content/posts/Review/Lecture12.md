---
# 必填
title: "ASP.NET期末复习Lecture12 : Entity Framework Core 简介"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
pin: 10
toc: true
abbrlink: 'asp-lecture12'
---

第十二课简要介绍了 Entity Framework Core

---
## 1. 数据库访问在 Web 应用中的重要性

大多数应用程序都需要存储和加载某种形式的数据，这些数据通常存储在数据库中。然而，直接与数据库交互是一个复杂的过程，包括：
- 管理数据库连接
- 将应用程序数据转换为数据库能理解的格式
- 处理各种潜在问题
---
## 2. 什么是 Entity Framework Core (EF Core)

EF Core 是一个提供面向对象方式访问数据库的库，它作为对象关系映射器 (ORM)，与数据库通信并将数据库响应映射到. NET 类和对象。

---
## 3. ORM (对象关系映射) 的概念

ORM 是一组技术，允许开发者从面向对象编程语言访问关系型数据库管理系统 (RDBMS) 数据。其核心思想是创建一个转换 (映射) 机制，让开发者能够使用熟悉的面向对象概念来操作关系数据库中的表、视图、存储过程等。

---
## 4. 为什么使用 ORM

ORM 工具将数据库表中的数据表示为对象，具有以下优势：
- 使用原生. NET 类型暴露数据
- 使用简单的. NET 属性暴露关系
- 提供编译时检查
- 开发者不必使用 SQL，而是使用 LINQ 查询数据
---
## 5. ORM 的结果表现

EF Core 将数据库暴露为一组对象：
- `DbContext`：数据库的抽象表示
- `DbSet<TRowType>`：DbContext 的集合属性，表示表的抽象
- `Entity`：集合中的每个对象，对应表中的一行数据
- `TRowType` 类的属性：表示表中列的抽象
---
## 6. ORM 示例

典型的 EF Core 使用模式如下：

```csharp
// 定义实体类
Public class Product
{
    Public int Id { get; set; }
    Public string Name { get; set; }
    Public decimal Price { get; set; }
}

// 定义 DbContext
Public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    
    Protected override void OnConfiguring (DbContextOptionsBuilder optionsBuilder)
    {
        OptionsBuilder.UseSqlServer ("连接字符串");
    }
}

// 使用示例
Using (var context = new AppDbContext ())
{
    // 添加数据
    Var product = new Product { Name = "笔记本电脑", Price = 5999 };
    Context.Products.Add (product);
    Context.SaveChanges ();
    
    // 查询数据
    Var expensiveProducts = context. Products
                                  .Where (p => p.Price > 5000)
                                  .ToList ();
}
```

---
## 7. EF Core 的核心优势

1. **生产力提升**：减少大量重复的数据访问代码
2. **可维护性**：使用强类型 LINQ 查询而非字符串 SQL
3. **数据库无关性**：支持多种数据库，可轻松切换
4. **迁移支持**：提供代码优先的数据库迁移工具


