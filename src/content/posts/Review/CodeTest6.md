---
# 必填
title: "白盒测试入门系列(6) : 数据流覆盖"
published: 2025-05-10

# 可选
tags:
  - 软件测试
  - 白盒测试
# 进阶，可选
pin: 13
toc: true
abbrlink: 'softwaretest-codetest6'
---

## 碎碎念

本文会从电子计算机的起源讲起，回顾它最初在军事弹道计算中的关键作用。为了清晰地阐释数据流测试的核心概念，我们会通过直观的图示，揭示程序节点中变量的定义（DEF）和使用（USE）之间的逻辑关系。  

随后，会系统地介绍三种数据流覆盖准则：  

- **定义覆盖（All-Defs）** ——追踪每个变量的定义是否被覆盖  
- **引用覆盖（All-Uses）** ——确保每个变量的使用点都被执行  
- **定义-引用路径覆盖（All-DU-Paths）** ——严格检查变量从定义到使用的完整路径  

通过代码示例对比三种准则的严格性和适用场景，理解它们的区别与优劣。

---
## 1. 数据流测试

### 1.1 数据流测试的背景和动机

最早的电子计算机是用于**战斗中的弹道计算**，核心任务是**数据处理**。软件运行过程中，**数据的流动与处理是核心**，因此软件测试中产生了“**数据流测试**”的概念。传统的结构化测试只关注程序结构，如路径覆盖、条件覆盖等，而**数据流测试专注于变量的定义与使用是否正确**。

> ✅结论：数据流测试专注于变量的定义和使用是否正确

### 1.2 变量的两类操作

在前面的文章中，我们只强调了程序结构性的覆盖，我们并没有关心数据流图中“结点”上到底是什么东西，而数据流我们是关注关于这些“节点”中变量的操作是否正确。

对变量的操作可以分成定义和使用两种：

- **定义（Definition）**：
    - 向变量赋值、初始化。
    - 表示把值“塞”进内存中。
    - 例：`x = 42`（将值42赋给变量x，是对x的定义）。
- **使用（Use）**：
    - 读取变量的值或以其参与运算、判断、控制流等。
    - 包括赋值右值、条件语句、循环条件等。
    - 例：`z = x * 2`（x为使用，z为定义）；`if (x > 0)`（x被使用）。

我们来看这个结构：

![](https://cdn.ethanzhou.cn/i/2025/05/10/681f29db2f8f6.png)

在节点 1 有 `x = 42` 这是关于变量 `X` 的定义。在节点 5 `Z = X * 2` 这是关于变量 `X` 的使用，也是关于变量 `Z` 的定义，节点 6 也是一样。

从这张图我们可以了解，变量定义了，当然以后就要用它。**用的地方可能一个也可能有多个**。**变量也可以在不同的地方定义，而它使用的地方也有可能会交叉**，因此构建了定义和使用之间比较复杂的一些关系。

> ✅结论：变量可以在不同的地方被定义和使用，这会导致复杂的关系

### 1.3 DU 对（Definison-Use 对）

为了理清变量定义和使用之间的关系，我们首先引入以下几个定义：

- **def(n)**：在节点 `n` 上定义的所有变量集合（definition set）。
- **use(n)**：在节点 `n` 上使用的所有变量集合（use set）。
- **def(e)**：在边 `e` 上定义的变量。
- **use(e)**：在边 `e` 上使用的变量。

我们定义 **(li, lj) 是变量 v 的 DU 对** ，表示变量 v 在节点 `li` 上被**定义**，在节点 `lj` 上被**使用**，要求它们之间的路径是**定义清晰的路径（Definition-Clear Path）** ，即从 `li ` 到 `lj ` 的路径上，变量 ` v ` 没有被重新定义。

> 注：若中间某个节点重新定义了变量 v，则前面 `li` 的定义就被“屏蔽”，无法传到 `lj`。

这一点大家很容易理解。假如我们定义被再重新定义以后，前面的定义就没有用了，所以数据流的可达是指有**一条定义清晰的路径**，可以从 `li` 到 `lj` 。

### 1.4 DU 路径（Definition-Use Path）

DU 路径是指定义清晰且简单的一条子路径，下面这张图阐释 DU 路径与路径集合的具体定义和符号表示：

![](https://cdn.ethanzhou.cn/i/2025/05/10/681f2bc6aaac3.png)

- **DU路径（DU path）**：
    - 是一条简单路径（无重复节点）；
    - 是定义清晰路径；
    - 从`ni`定义点到`nj`使用点。
    
- **DU路径集合记法**：
    - `du(ni, nj, v)`：从 `ni` 定义 v 到 `nj` 使用 v 的所有DU路径集合；
    - `du(ni, v)`：从`ni`定义v到**任意使用v的地方**的所有路径集合。

---
## 2. 数据流覆盖准则

### 2.1 三种数据流覆盖的概念

- 定义覆盖（All-Defs Coverage）
	- 要求每一个变量的每一个定义点**至少参与一次**的DU对测试；
	- 即每一个定义过的变量，都要至少使用一次。

- 引用覆盖（All-Uses Coverage）
	- 要求每个变量的每个使用点**至少通过一次清晰路径到达**；
	- **强调所有使用点都被覆盖**；
	- 因为有使用就必须有对应的定义，否则程序运行就会出错。

- 所有定义-引用路径覆盖（All DU-Paths Coverage）
	- 最强的准则；
	- 要求**每一个定义-引用对之间的所有清晰路径都被覆盖**；
	- 对变量v的每一个定义点，必须考虑其到所有使用点的**所有清晰路径**。

### 2.2 数据流覆盖的示例

我们接着第一节的例子，针对变量 `x` 进行数据流覆盖：

![](https://cdn.ethanzhou.cn/i/2025/05/10/681f2e8fe51e7.png)

对于定义覆盖，我们只需要保证变量 `x` 的每一个定义点**至少参与一次** DU 对测试。所以我们只需要一条测试用例，并保证测试用例通过定义点 1 即可。

对于引用覆盖，我们需要保证变量 `x` 的每个使用点**至少通过一次清晰路径到达**。所以我们只需要两条测试用例，并保证测试用例通过使用点 5 和 6 即可。

对于定义引用覆盖，我们需要保证变量 `x` 的**每一个定义-引用对之间的所有清晰路径都被覆盖**。所以我们只需要四条条测试用例，覆盖所有 1 → 5 和 1 → 6 的情况。

