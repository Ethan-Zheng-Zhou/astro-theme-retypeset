---
# 必填
title: "白盒测试入门系列(2) : 结构覆盖-语句覆盖与分支覆盖"
published: 2025-05-09

# 可选
tags:
  - 软件测试
  - 白盒测试
# 进阶，可选
pin: 9
toc: true
abbrlink: 'softwaretest-codetest2'
---

## 碎碎念

想象一下，你的程序就像一座藏宝图，而**控制流图**就是画在藏宝图上的路线。走通这张图，就等于把程序里所有犄角旮旯的代码都溜达了一遍，连程序员自己都可能忘记的边边角角都不放过！这就是白盒测试的奥义——拿着放大镜，把代码翻个底朝天。

---

## 1. 从控制流图看测试覆盖

这一篇我们学习一种基于代码的测试方法，结构覆盖。当我们能够使用控制流图来对复杂的程序进行抽象，我们也就能够对复杂的程序开展白盒测试或者是基于代码的测试。我们知道在控制流图当中，它的节点对应程序当中的语句，而节点与节点之间的有向边，对应程序当中语句与语句之间的执行序列，**那么我们对程序进行测试，就相当于对控制流图来进行遍历**，所以如何更好地对程序进行测试，这样的问题就等价于如何更好的对控制流图进行遍历的问题。

> ✅ 结论：**测试程序 = 遍历控制流图**，因此优化测试 = 优化图的遍历

---
## 2. 语句覆盖

我们首先来关注组成控制流图的结构，也就是点和边，要对一个控制流图进行遍历，我们首先需要考虑控制流图当中的节点，那么由于控制流图当中的节点与程序当中语句的对应关系，对控制流图当中节点的遍历，就相当于对程序当中语句的访问。

### 2.1 语句测试与语句覆盖率

这里我们首先有**语句测试**的概念，它是指**我们设计测试用例时，应该去确保执行所有可被执行的语句**。当一组测试用例执行完毕后，我们可以针对这组测试用例来计算语句覆盖率，**它的分子是所有已经被执行过的语句的数量，而分母是所有可被执行语句的数量**。

> ✅ 结论：**语句覆盖率** = 已经被执行过的语句数量 / 所有可被执行语句的数量

### 2.2 语句覆盖的示例

我们继续三角形程序的例子，这个程序接受 `a b c ` 三个输入，它们分别表示三角形三条边的长度，程序会根据三条边的长度来判断这个三角形是等边三角形还是等腰三角形还是普通三角形。

![三角形程序示例](https://cdn.ethanzhou.cn/i/2025/05/09/681dd974b9ae3.jpg)

在这样一段程序当中，我们共标记了 5 行关键代码，5 行关键代码在控制流图当中分别用 `12345 ` 这 5 个节点来表示，在控制流图当中，另外会有 6 和 7 这两个节点，它们是**虚拟**的节点，不指代程序当中的任何代码。

运行第一条测试用例 ` a = 2，b=2，c=2` 后，程序当中共有两条语句被执行，它们分别是 1 和 2，所以此时的语句覆盖率是 2/5。

当第二条测试用例 `a = 2，b=2，c = 3` ，运行后，程序当中又有两条语句被执行，分别是语句 3 和语句 4，所以此时的语句覆盖率又**增加**了 2/5，变成了 4/5。

当第三条测试用例 ` a = 2，b=3，c = 4`，运行后，程序当中编号为 5 的语句也被执行。那么此时语句覆盖率达到了 100 %。此时，**控制流图中所有的点都被我们遍历过了** 。

---

## 3. 分支覆盖

要对一个控制流图进行遍历，我们还需要考虑控制流图当中的边。控制流图当中的边体现了程序当中语句与语句之间可能的执行顺序，**所以如果我们能遍历控制流图当中所有的边就意味着我们执行了程序当中所有可能的分支**。

### 3.1 分支测试与分支覆盖率

那么所谓的**分支测试就是指我们在设计测试用例时，需要保证测试用例能够执行所有可能被执行到的分支**。

那么当一组测试用例执行完毕后，我们可以针对这组测试用例来计算分支覆盖率，它的分子是已经被执行的分支的数量，而它的分母是所有可被执行的分支的数量。

> ✅ 结论：**分支覆盖率** = 已经被执行过的分支数量 / 所有可被执行分支的数量

### 3.2 分支覆盖的示例

我们还是考虑这样三角形程序，这段程序当中共有两个 if else 的双向分支结构，所以它们共有 4 个分支，体现为控制流图当中的 1 → 2 、1 → 3 、 3 → 4， 3 → 5 的边。

![三角形程序示例](https://cdn.ethanzhou.cn/i/2025/05/09/681de926cb0e0.jpg)

当我们执行第一条测试用例 `a = 2，b = 2，c = 2 ` 之后，控制流图当中从 1 → 2 的边被执行，此时分支覆盖率是 1/4。

当我们执行第二条测试用例 `a = 2，b = 2，c = 3 ` 时，控制流图当中的从 1 → 3  以及从 3 → 4 这两条边被执行，那么此时累加的分支覆盖率变成了 3/4。

当我们运行最后一条测试用例 `a = 2，b = 3，c = 4 ` 之后，控制流图当中的另外一条边，也就是  3 → 5 的这条边也被执行。那么此时累计的分支覆盖率就是 100%。

---

## 4. 可被执行语句/分支（关于 **executable** 的解释）

在学习了语句覆盖和分支覆盖的概念之后，我们来考虑这样一个问题，为什么在语句测试和分支测试的定义当中都会有 executable(可被执行的) 这样一个形容词的存在。考虑这样一个案例，这个案例可能并没有什么实际的含义，我们主要去关注它的结构：

![示例代码](https://cdn.ethanzhou.cn/i/2025/05/09/681dea7fa71c3.jpg)


我们来考虑这样的问题。`code 2 ` 的这样一段代码是否有机会被执行？我们可以看到如果在 `code 1` 这样一段代码当中没有对 x 的值和 y 的值做任何改变的话，那么 `code 2` 这段代码是永远不会有机会被执行的。体现在控制流图上，我们就会发现节点 3 是永远不会有机会被执行的。同样从节点 2 到节点 3 以及从节点 3 到节点 4 的这样两条边，也是永远不会有机会被执行的。

这也就是说在有些情况下，在程序当中可能会存在一些永远不能被执行的语句或者分支，我们称之为**不可达语句或者不可达分支**。所以在之前的文章里介绍 PIE 模型时，PIE 模型当中的第一个条件，其实它包含了两个条件，一个是可达，另外一个才是执行。

> ✅ 结论：语句/分支覆盖率的分母必须是**可达语句/可达分支**。

---

## 5. 语句覆盖 VS 分支覆盖，谁更强？

我们再来考虑一个问题，语句覆盖和分支覆盖，他们的目标都是为了对控制流图当中的某种元素来进行遍历，那么语句覆盖主要用于对控制流图当中的节点进行遍历，分支覆盖主要用于对控制流图当中的边进行遍历，那么这两种方法它们是否等价？

我们还是看一个简单的例子，那么这个例子里面我们会比较 x 和 y 这两个数的大小，然后找出其中最大的，那么程序当中会有一个单向的分支结构：

![示例代码](https://cdn.ethanzhou.cn/i/2025/05/09/681debddd44bb.jpg)

假如我们执行一条测试用例，也就是 `x 等于 2，y 等于 1 ` 这条测试用例，那么程序的执行会是从控制流图当中的节点 1 到节点 2 再到节点 3，那么这个时候程序当中所有的代码都被执行，语句覆盖率达到了 100 %，但是控制流图当中的三条边，那么只执行了从 1 → 2 以及从 2 → 3 这两条边， 1 → 3 的这条边并没有被执行，所以分支覆盖率就没有达到 100 %。

通过这样一个例子，我们可以看出语句覆盖和分支覆盖这两种覆盖准则，**分支覆盖比语句覆盖会更强**，也就是说当一组测试用例，它的语句覆盖率达到 100 %之后，并不能保证它的分支覆盖率也达到 100 %。

> ✅ 结论：分支覆盖条件更强。


