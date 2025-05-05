---
# 必填
title: "ASP.NET期末复习Lecture7 : 简单的 MVC 应用"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
toc: true
abbrlink: 'asp-lecture7'
---

第七次课主要围绕一个简单的 **ASP.NET MVC 应用**展开，通过开发“员工信息管理系统”项目，讲解 **Model、Controller** 的实现和 **Repository 模式** 的引入。

---
## 一、MVC 项目实战：员工信息管理系统

目标：开发一个 MVC 应用，实现对员工信息的完整管理（增删改查）。
### 📋 员工信息包括：

- 工号（EmployeeId）
- 姓名（EmployeeName）
- 职务（Title）
- 国籍（Country）

---

## 二、模型层（Model）设计

###  1.创建员工模型类（Employee）

```csharp
// Employee.cs
public class Employee
{
    public int EmployeeId { get; set; }      // 工号
    public string EmployeeName { get; set; } // 姓名
    public string Title { get; set; }        // 职务
    public string Country { get; set; }      // 国籍
}
```

**目的**：用于在应用中传递和存储员工的结构化数据。

---

## 三、控制器层（Controller）设计

### 1. 控制器定义

- 所有控制器类需继承自 `Controller` 基类；
- 控制器放置在 `Controllers` 文件夹中；
- 命名规范：以 `Controller` 结尾，如 `EmployeeManagerController`。

### 2. 控制器方法（Action）= 可处理的请求动作

```csharp
public class EmployeeManagerController : Controller
{
    // 罗列员工信息
    public IActionResult List() { ... }

    // 新增员工信息
    public IActionResult Insert(Employee model) { ... }

    // 更新员工信息
    public IActionResult Update(Employee model) { ... }

    // 删除员工信息
    public IActionResult Delete(int EmployeeId) { ... }
}
```

---

## 四、企业级应用架构与 Repository 模式

### 1. 企业级应用（Enterprise Application）特点：

- 以数据驱动为核心；
- 多用户并发使用；
- 使用持久化存储（数据库）；
- 项目复杂、模块分层、逻辑清晰。
    
### 2. 问题：是否应该在 Controller 中直接写数据操作逻辑？

不推荐！更合理的方式是使用三层架构（Layered Architecture）：

|层|功能|示例|
|---|---|---|
|表现层（UI）|展示界面|Razor 页面、HTML|
|业务逻辑层（Business）|控制业务流程|Controller|
|数据访问层（Data）|操作数据库|Repository|

---

## 五、Repository 模式介绍

### 1. Repository 模式作用：

- 将数据访问代码封装起来，**解耦控制器与数据库逻辑**；
- 提供统一接口操作数据实体；
- 更易于测试和维护。
### 2. 数据层常见设计模式：

|模式|功能|
|---|---|
|Repository|映射数据库实体对象|
|Unit of Work|管理事务提交|
|Lazy Load|延迟加载数据，提升性能|

---

## 六、总结

-  掌握 MVC 模式下 Model 和 Controller 的基本写法；
-  理解 Controller 不应直接承担数据访问职责；
-  学会引入 Repository 模式分离关注点，提升项目可维护性；
-  为后续的数据库集成和服务逻辑封装打好基础。
