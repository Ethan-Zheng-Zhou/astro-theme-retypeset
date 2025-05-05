---
# 必填
title: "ASP.NET期末复习Lecture9 :  视图的高级用法"
published: 2025-05-05

# 可选
tags:
  - ASP.NET 
  - 期末复习

# 进阶，可选
toc: true
abbrlink: 'asp-lecture8'
---


Lecture 9主要围绕 **ASP.NET MVC 中视图的高级用法** 展开，重点讲解了以下几个核心内容：

---
### 一、设置视图模型类型（Setting the View Model Type）

#### 1. Razor 不知道控制器传入视图的数据类型

- 需要用 `@model` 指定模型类型。
- 用 `@Model` 表达式来访问模型对象的属性。

```csharp
@model List<Employee>

@foreach (var item in Model) {
    <p>@item.EmployeeName</p>
}
```

---

### 二、使用共享视图（Using Shared Views）

在多个控制器或视图中共享公共内容（如页面布局）可以避免重复代码。

#### 1. Layout（布局）

- 将 HTML 文档结构、`<head>` 部分、CSS 引入等公共内容抽取到 `_Layout.cshtml`。
- `_Layout.cshtml` 放在 `/Views/Shared` 文件夹下
- 每个具体视图只需关注自己的“独特”部分。
- 使用 `@RenderBody()` 将子视图的内容插入到布局模板的指定位置。

```html
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Employee Management</title>
</head>
<body>
    <div>
        @RenderBody()
    </div>
</body>
</html>
```

#### 2. 在视图中使用布局

在视图文件头部设定布局文件：

```csharp
@model Product
@{
    Layout = "_Layout";
}
```

---

### 三、视图开始文件（View Start File）

`_ViewStart.cshtml` 是 ASP.NET Core MVC/Razor Pages 中一个特殊的全局配置文件，它会在**所有视图渲染之前自动执行**，用于设置视图的公共布局和默认配置。

#### 1. 核心特性

1. **自动执行**：位于 `Views` 文件夹根目录下，会影响该文件夹及其子文件夹中的所有视图
2. **执行顺序**：在所有视图渲染之前执行
3. **常见用途**：设置默认布局页面
4. **文件位置**：通常放在 `/Views/_ViewStart.cshtml`

![视图开始文件位置](https://cdn.ethanzhou.cn/i/2025/05/05/6818ab03d838f.jpg)

#### 2. 示例

```csharp
@{
    Layout = "_Layout";
}
```

---

### 四、视图导入文件（View Imports File）

- Razor 默认需要用完整命名空间引用类。
- 可以通过 `_ViewImports.cshtml` 引入公共命名空间。

**示例：**

```csharp
@using SampleApplication.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

---

### 五、项目实践：实现列表视图（List View）

**目标**：展示所有员工信息。

```csharp
@model IEnumerable<Employee>

<h2>Employee Info List</h2>
<table>
<tr>
    <td>Employee ID</td>
    <td>Employee Name</td>
    <td>Employee Title</td>
    <td>Employee Country</td>
</tr>

@foreach(Employee e in Model) {
<tr>
    <td>@e.EmployeeId</td>
    <td>@e.EmployeeName</td>
    <td>@e.Title</td>
    <td>@e.Country</td>
</tr>
}
</table>
```

---

### 六、总结

这一讲重点介绍了如何通过 `@model`、布局、视图开始文件和导入文件来提升视图的可维护性与复用性，同时也动手实现一个完整的员工列表展示页面。
