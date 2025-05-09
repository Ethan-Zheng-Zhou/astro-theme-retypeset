---
# 必填
title: "GTest单元测试(2): 浅谈随机测试"
published: 2025-05-09

# 可选
tags:
  - 软件测试
  - GTest

# 进阶，可选
pin: 6
toc: true
abbrlink: 'softwaretest-gtest2'
---


## 碎碎念

相信你读完上一篇文章后，已经对GTest基本用法以及输入域测试技术中的等价类划分、边界值分析有了简单的了解了，这篇文章我们通过任务二来学习随机测试的相关知识，并实战GTest框架，开始吧！

---
## 1.随机测试

### 1.1 什么是随机测试（Random/Monkey Testing）？

随机测试是一种使用随机、相互独立的程序输入来对计算机程序进行测试的输入域测试技术。在处理完随机且独立的程序输入后，程序输出的结果将会和软件规格说明（software specifications）中所描述的软件行为进行比对来判断该测试是否通过。在这一篇文章中，我们并没有一个可以用来进行对比的软件规格说明，所以如何自动获得测试用例的预期结果是一个问题。不过我们可以按照互联网公司常用的灰度发布AB测试的方式，可以使用一个不同版本`NextDate`函数来产生预期结果，这个后文再具体操作。

随机测试的核心思想接近于[无限猴子定理](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E6%97%A0%E9%99%90%E7%8C%B4%E5%AD%90%E5%AE%9A%E7%90%86&zhida_source=entity)（The Infinite Monkey Theorem），所以随机测试有时也被称为Monkey Testing。无限猴子定理是来自Émile Borel的一本出版于1909年的概率相关的书籍，其中描述了这样一个场景：当一只猴子在打字机键盘上不限时间地随机敲击，它可能输出任何给定文章，例如莎士比亚的完整作品，并且这种可能性随着时间的增长会不断地接近100% 。这个定理在数学上是基于柯尔莫哥洛夫的零一律的一个例子，它强调了即便一个事件发生的概率极小，只要时间足够长，就几乎必然会发生。

换用测试的语言来描述则是**测试程序通过不断地生成测试输入，其能根据PIE模型的原理，完整测试所有程序分支并寻找到程序中异常的可能性会不断增大**。

### 1.2 早期的随机测试

最早对随机测试的使用可以追溯到上个世纪五十年代，那时的数据还存储在打孔卡片（Punched Card）上。程序员会将扔进垃圾桶中的卡片或者标记有随机数字的卡组作为计算机程序的输入来进行随机测试。

当随机测试被广泛认定为测试程序的最差方式时，**Duran和Ntafos两人在1981年正式地对使用随机测试技术对程序进行测试的有效性进行了调研，结果表明相对于系统化的测试技术，随机测试是一个成本低收益高的替代品。**

后来在1983年，苹果公司的Steve Capps开发了一款名为“The Monkey”的随机输入生成工具用来对传统的MacOS应用进行测试。他对工具的命名就是化用了无限猴子定理。

1991年，一款名叫 “crashme” 的工具发布。这款工具意在通过随机执行带有随机参数的[系统调用](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8&zhida_source=entity)来测试Unix以及类Unix操作系统的[鲁棒性](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E9%B2%81%E6%A3%92%E6%80%A7&zhida_source=entity)。

### 1.3 随机测试的优缺点

- **优点**
    - **容易实现和使用**：随机测试并不需要知道程序细节，而且输入也通过随机生成。
    - **对程序不存在偏见**：由于随机测试的输入都是随机生成，不存在人为因素影响，也不会因为对程序的某一部分信任而忽略潜在的漏洞。
    - **能快速查找漏洞**：随机测试的测试速度快，通过快速和大量的测试，能够在短时间找到大量的候选漏洞。
- **缺点**
    - **寻找漏洞的精度不高**：由于随机测试的完全随机性，寻找到的漏洞很可能是一些无关紧要的错误。
    - **过于随机导致对程序的代码覆盖率不高**：大部分人认为对程序的测试过于依赖随机，不如通过人工白盒测试的方法更精确地测试程序。

### 1.4 随机测试进阶——模糊测试（Fuzz Testing）

在1988年的一个风雨交加的夜晚，威斯康星大学的Barton Miller教授在自己的公寓中通过一条电话线连接他在学校中的计算机。暴风雨引发了电话线中的信号错乱，以至于所连接的Unix终端不断接收到糟糕的命令输入，最终导致了系统崩溃。频发的崩溃使这位讲授操作系统课程的教授感到惊讶，因此他脑海中浮现了一个对Unix系统进行[鲁棒性测试](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E9%B2%81%E6%A3%92%E6%80%A7%E6%B5%8B%E8%AF%95&zhida_source=entity)的念头。于是他在给学生的课程作业中写道：

_The goal of this project is to evaluate the robustness of various UNIX utility programs, given an unpredictable input stream. This project has two parts. First, you will build a fuzz generator... Second, you will take the fuzz generator and use it to attack as many UNIX utilities as possible, with the goal of trying to break them..._

教授在作业中要求学生开发一个模糊生成器（fuzz generator），这个生成器可以产生不可预测的输入流，然后将这些杂乱的输入给到Unix系统设施，然后试图攻陷这些设施并找到和分析引发错误的随机输入和原因。这就是模糊测试的诞生，教授在作业中使用的fuzz一词也就被用来命名这一技术。

|特性|随机测试|模糊测试|
|---|---|---|
|输入生成|完全随机|半随机，基于规则或模型|
|数据有效性|可能包含有效和无效数据|特意生成某些边界值、畸形数据和无效数据|
|生成策略|**无特定策略**|**有特定策略**，可能使用语法模型、变异测试或生成式方法|

从上文可以看到模糊测试在诞生之初和随机测试十分相似，都是通过随机的输入来对计算机程序进行功能行为测试，但是模糊测试会通过某些特定策略生成一些特定的结果。下面的Python代码片段给出了一个最简单的模糊器（fuzzer）例子。这个模糊测试器接收三个参数：最大长度（max_length）、字符的[ASCII码](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=ASCII%E7%A0%81&zhida_source=entity)起始值（char_start）以及字符的ASCII码从起始到结束的范围（char_range）。该模糊器将生成一个长度为max_length的包含ASCII码在[char_start, char_start + char_range)范围内的字符串用于测试。

```python
# Programme 1_1 fuzeer.py
import random

def fuzzer(max_length: int = 100, char_start: int = 32, char_range: int = 32) -> str:
    string_length = random.randrange(0, max_length + 1)
    out = ""
    for i in range(0, string_length):
        out += chr(random.randrange(char_start, char_start + char_range))
    return out
    
# 生成了符合特定策略的字符串
```

所以最原始的模糊测试和随机测试拥有相同的优缺点。其中最显著的问题就是由于其随机性和策略，寻找到的程序错误过于刁钻。可能找到的程序错误只是因为错误的输入导致而和程序本身实现无关，或者程序使用人员根本就不会使用像模糊测试器生成的输入，所以不会在软件设计的考虑范畴之内，甚至模糊测试根本就没有测试到程序的主要功能。

无论如何模糊测试还是有其可取之处的（尤其是测试速度快、易于实现），所以后来的研究者们不断想方设法地，尤其是针对如何更好地生成测试输入方面去提高模糊测试的精度（找到的错误确实是和软件规范说明或预期设想行为不符）和深度（能更多地去测试程序的主要功能）来完善模糊测试的实用性。实现这些目标的主要方法就是通过一些静态（基于覆盖率、基于变异）或动态（基于搜索、基于语法）的方式给模糊器提供额外的辅助信息来帮助模糊器更高效地生成、更有效地测试输入，而不再是完全随机，这也促使模糊测试从[黑盒测试](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E9%BB%91%E7%9B%92%E6%B5%8B%E8%AF%95&zhida_source=entity)向灰盒测试进行转变。下文会进行具体介绍。

---
## 2.如何改进模糊测试？

### 2.1 现代模糊测试技术的划分方式

**·** 一种模糊测试技术可以是基于生成（generation-based）的或者基于变异（mutation-based）改进的，取决于它是在没有任何参考的情况下生成随机输入，还是在现有输入的基础上进行修改，也就是所谓的变异；

**·** 一种模糊测试技术可以是愚笨的（dumb）或者聪明的（smart），取决于它是否对测试输入的结构敏感；

**·** 一种测试技术可以是白盒、灰盒或者黑盒（基本上不再使用），取决于它是否对程序结构信息或者程序运行信息敏感。

目前最流行的模糊测试工具主要有AFL以及AFL的扩展系列，如AFLFast、AFL++、AFLGo等。AFL系列就是基于突变的、覆盖率引导的fuzz generator。它改变一组测试用例以达到程序中以前未探索的代码。覆盖率发生变化时，触发新覆盖率的测试用例将保存为测试用例队列的一部分。

### 2.2 基于覆盖率的模糊测试（Converage-Based Fuzzing）

对于如何提高模糊测试精度和深度的探索，其中一种是利用了[代码覆盖率](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=2&q=%E4%BB%A3%E7%A0%81%E8%A6%86%E7%9B%96%E7%8E%87&zhida_source=entity)（code coverage）这一概念。**代码覆盖率包括函数覆盖率（function coverage）、[语句覆盖率](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E8%AF%AD%E5%8F%A5%E8%A6%86%E7%9B%96%E7%8E%87&zhida_source=entity)（statement coverage）、边覆盖率（edge coverage）以及[条件覆盖](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E6%9D%A1%E4%BB%B6%E8%A6%86%E7%9B%96&zhida_source=entity)（condition coverage）率等多种覆盖准则，这些覆盖率都在一定程度上反映了测试用例对于程序代码的测试程度，即代码覆盖率越高，测试用例对代码的测试程度越高。**近期的一项工作也观察到模糊测试技术的代码覆盖率和能找到程序中的错误具有强相关性。

下面的Python代码片段将给出简单的例子。当输入`inp`为一个正数的时候，程序执行过程中将会执行第 2 行和第 3 行的代码语句，则语句覆盖率为 2 / 6 = 33.3%。另外对于条件覆盖率来说，整个函数中函数有三个 `if` 条件语句，每一个 if 条件语句都有真假两种情况，则根据组合原理该函数共有 6 个条件分支。在上述情况下，程序执行过程中只有第一个 `if` 条件语句为真的分支被执行，则条件覆盖为 1 / 6 = 16.7%。对于输入`inp`为零和负数的情况，语句覆盖为 3 / 6 = 50% 和 4 / 6 = 66.7%，条件覆盖率为 2 / 6 = 33.3% 和 3 / 6 = 50%。

```python
# Programme 1_2 fun1.py
def fun1(inp: int) -> str:
if inp > 0:
return "positive"
if inp == 0:
return "zero"
if inp < 0:
return "negative"
```

在模糊测试中对于覆盖率的利用，主要是在测试用例的生成过程中。模糊器首先生成一定个数的随机输入，这些输入被称为种子（seed）。接着模糊器将这些种子输入程序，回收程序执行的结果和预先选定的覆盖率指标。接下来模糊器会根据覆盖率高低，将覆盖率低的种子丢弃，将覆盖率高的种子保留并对这些种子进行操作生成一批新种子再作为输入运行程序。重复上述过程到覆盖率不能够更进一步提高时，终止测试。

### 2.3 **基于变异的模糊测试（Mutation-Based Fuzzing）**

由于最初的模糊器完全靠随机生成程序输入，模糊测试很难生成出符合程序要求的合法输入，以至于很难测试到程序的核心功能，同时这也是代码覆盖率极低的一个原因。于是基于变异的模糊测试被提出来解决这个问题。

变异的核心思想是对现有的种子（种子可以合法也可以不合法，但大多数情况下会使用合法的种子，这样通过变异得到的种子后代质量较高）以及通过变异得到种子后代进行操作来生成测试输入。具体的变异操作需要测试人员执行制定。

假如针对一个计算机程序的一个合法的程序输入为“3+0”，那么在指定变异操作为随机增加、删除和替换字符串中的某一个字符这三种操作时，经过变异之后的输入就有多种可能，如“3 + - 0”、“3 +”、“3 / 0”。将这些通过变异得到的种子后代扔入程序中运行，这就是基于变异的模糊测试。

变异操作可以被设计为更加复杂精细的过程，例如结合其他种子质量评估指标进行变异或者进行某种针对性变异操作。上一小节末尾描述的将变异操作和覆盖率指标进行结合来反复对程序进行测试，这样可以很好地利用这两种方法的优势，从而增强模糊测试的有效性。著名的模糊测试工具AFL就是基于这样的思路开发的。

### 2.4 **基于搜索的模糊测试（Search-Based Fuzzing）**

基于搜索的模糊测试主要依赖于搜索算法。搜索算法也被称为[启发式算法](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E5%90%AF%E5%8F%91%E5%BC%8F%E7%AE%97%E6%B3%95&zhida_source=entity)（heuristic algorithm），其核心思想是通过某些程序信息来启发和引导算法执行。这些算法的灵感大多是来自自然界的现象，如[模拟退火算法](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E6%A8%A1%E6%8B%9F%E9%80%80%E7%81%AB%E7%AE%97%E6%B3%95&zhida_source=entity)和本小节会介绍的遗传算法。而它们之所以被称为搜索算法，是因为执行这些算法可以在较大的搜索空间中比[随机算法](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E9%9A%8F%E6%9C%BA%E7%AE%97%E6%B3%95&zhida_source=entity)或遍历算法更高效。同样以上一小节的计算机程序输入为例，模糊器对该程序进行模糊测试本质上就是在不断地探索其输入空间，该输入空间中包含了任意长度的合法或不合法的表示四则运算表达式的字符串。想要通过测试来挖掘出程序中隐藏的缺陷，就是在输入空间中搜索到特定的输入使得该程序在运行时出现异常行为；想要满足更广的程序覆盖率，就是在输入空间中搜索到特定的输入使得让该程序运行时执行更多的程序语句。

因此搜索算法也就自然而然地与模糊测试结合在了一起，来使模糊器生成更优质的[程序测试](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E7%A8%8B%E5%BA%8F%E6%B5%8B%E8%AF%95&zhida_source=entity)输入。下面将对基于结合代码覆盖率的遗传算法的模糊测试进行简单的介绍。

我们构建一个简单函数 `fun2`，该函数将判断特定字符是否存在于输入字符串中，然后对应分支返回一个整型数字。

```python
# Programme 2_3 fun2.py
def fun2(inp: str) -> int:
    if "a" in inp:
        if "b" in inp:
            if "c" in inp:
                return 3
            return 2
        return 1
    return 0
```


在这个例子中我们给定模糊器的初始种子，为“axxx”，“xxxb”、“xxxc”和“xxxd”。我们首先将这些种子作为输入来运行这个函数，得到每个种子的语句覆盖数量，分别是 3 、2 、2、2 。输入“axxx”是可以让程序执行第 2、3 和 8 行语句，而后三个输入只能执行第 2 和 8 行语句。第一轮测试已经结束，这时我们根据种子的语句覆盖数量来生成下一次测试输入集合，具体过程为从种子中选取语句覆盖数量最多的两个作为亲代来进行交叉变异（模拟自然界的[遗传现象](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E9%81%97%E4%BC%A0%E7%8E%B0%E8%B1%A1&zhida_source=entity)）。由于后三个种子的数据覆盖数量都为 2 ，则在选择时会随机选取。假设“axxx”和“xxxb”得到了选择，则继续对这两个种子进行交叉变异，**此处交叉变异定义为交换两个种子的右半部分子串，即得到的子代为“axxb”和“xxxx”。重复这样的选择并交叉变异的过程直到子代数量等于初始种子数量**。假定经过上述过程我们最后得到了“axxb”、“xxxx”、“axxc”和“xxxx”这四个子代，然后就继续重复整个执行过程，也就是将这些子代输入程序继续运行遗传算法来生成更加优质的测试输入，最终提高代码覆盖率和[异常检测](https://zhida.zhihu.com/search?content_id=218026041&content_type=Article&match_order=1&q=%E5%BC%82%E5%B8%B8%E6%A3%80%E6%B5%8B&zhida_source=entity)能力。

从上述例子中可以看到在子代中的“axxb”不仅将原本“axxx”的语句覆盖数量从 3 提升到了 4，并且能更深入地探索整个函数中的条件分支。但可以看到对于上述例子中假设得到子代整个算法已经无法再有进步了，因为这些子代不论怎么交叉变异也无法得到一个包含abc三个字母的字符串。在实际工业生产环境中算法本身会更加复杂，测试种子会经过精心生成和挑选，同时纳入更多指标进行算法引导，对交叉变异的过程也不会只是简单地字符串交换。

---
## 3. 任务二

> 编写一个程序，随机生成NextDate函数的100条测试用例，测试用例存储至一个文本文件RT_test_data.csv，每行存放一条测试用例，测试用例内各个字段之间的分割方式按照csv规范。编写程序读取测试用例文件，使用GTest单元测试框架执行这100条测试用例对NextDate函数进行测试。

学习过随机测试的知识，我们尝试利用 2.3 节的**基于变异的模糊测试方法**来写，具体思路：

首先定义一个包含年月日和描述信息的`DateSeed`结构体，然后通过`mutateDate`函数对种子日期进行随机变异，该函数会随机选择年、月或日字段进行小幅度增减。`generateSBFTTestCases`函数负责生成测试用例，它使用一组精心挑选的边界值作为初始种子（如闰年2月29日、年末日期等），前几个测试用例直接使用这些种子值，后续用例则通过对随机选择的种子进行变异生成。生成的测试用例会被写入`CSV`文件，包含年、月、日和预期验证结果。测试执行部分使用Google Test框架，读取`CSV`文件中的测试数据，调用另外一种实现的`nextDate`函数`ValidDate`进行验证，并将实际结果与预期结果进行比对，如果发现不一致则会输出详细的错误信息

```cpp
/* Programme 3_1 test1.cpp */
#include <iostream>
#include <fstream>
#include <random>
#include <vector>
#include <algorithm>
#include <gtest/gtest.h>

using namespace std;

bool NextDate(int& year, int& month, int& day);
bool ValidDate(int year, int month, int day);

// 种子日期结构体
struct DateSeed {
    int year;
    int month;
    int day;
    int expect_year;
    int expect_month;
    int expect_day;
    string desc; // 种子描述
};

// 生成变异日期
DateSeed mutateDate(const DateSeed& seed) {
    static random_device rd;
    static mt19937 gen(rd());
    
    DateSeed mutated = seed;
    
    // 随机选择要变异的字段
    int field = uniform_int_distribution<>(0, 2)(gen);
    
    // 小幅度变异(-5到+5)
    int delta = uniform_int_distribution<>(-5, 5)(gen);
    
    switch(field) {
        case 0: mutated.year += delta; break;
        case 1: mutated.month += delta; break;
        case 2: mutated.day += delta; break;
    }
    
    // 确保月份和日不为负
    mutated.month = max(1, mutated.month);
    mutated.day = max(1, mutated.day);
    
    // 使用 ValidDate 生成预期结果
    mutated.expect_year = mutated.year;
    mutated.expect_month = mutated.month;
    mutated.expect_day = mutated.day;
    ValidDate(mutated.expect_year, mutated.expect_month, mutated.expect_day);
    
    return mutated;
}

// 生成测试用例
void generateSBFTTestCases(const string& filename, int count = 100) {
    static random_device rd;
    static mt19937 gen(rd());
    
    ofstream outfile(filename);
    outfile << "year,month,day,expect_year,expect_month,expect_day" << endl;
    
    // 初始种子集 (关键边界值)
    vector<DateSeed> seeds = {
        {2020, 2, 28, 0, 0, 0, "非闰年2月末"},
        {2020, 2, 29, 0, 0, 0, "闰年2月29"},
        {2020, 12, 31, 0, 0, 0, "年末"},
        {2020, 1, 1, 0, 0, 0, "年初"},
        {2020, 4, 30, 0, 0, 0, "小月末"},
        {2020, 3, 31, 0, 0, 0, "大月末"}
    };
    
    // 为种子生成预期结果
    for (auto& seed : seeds) {
        seed.expect_year = seed.year;
        seed.expect_month = seed.month;
        seed.expect_day = seed.day;
        ValidDate(seed.expect_year, seed.expect_month, seed.expect_day);
    }
    
    // 生成测试用例
    for (int i = 0; i < count; ++i) {
        // 50%概率从种子变异，50%概率随机生成
        DateSeed testCase;
        if (i < seeds.size()) {
            testCase = seeds[i]; // 先用完所有种子
        } else {
            // 从随机种子开始变异
            int seedIdx = uniform_int_distribution<>(0, seeds.size()-1)(gen);
            testCase = mutateDate(seeds[seedIdx]);
        }
        
        // 写入文件
        outfile << testCase.year << "," 
                << testCase.month << "," 
                << testCase.day << ","
                << testCase.expect_year << ","
                << testCase.expect_month << ","
                << testCase.expect_day << endl;
    }
}

// GTest测试用例
TEST(NextDateTest, GeneratedCases) {
    ifstream infile("RT_test_data.csv");
    string line;
    
    // 跳过标题行
    getline(infile, line);
    
    int test_case_num = 0;
    while (getline(infile, line)) {
        test_case_num++;
        int year, month, day, expect_year, expect_month, expect_day;
        
        // 解析CSV行
        size_t pos1 = line.find(',');
        size_t pos2 = line.find(',', pos1+1);
        size_t pos3 = line.find(',', pos2+1);
        size_t pos4 = line.find(',', pos3+1);
        size_t pos5 = line.find(',', pos4+1);
        
        year = stoi(line.substr(0, pos1));
        month = stoi(line.substr(pos1+1, pos2-pos1-1));
        day = stoi(line.substr(pos2+1, pos3-pos2-1));
        expect_year = stoi(line.substr(pos3+1, pos4-pos3-1));
        expect_month = stoi(line.substr(pos4+1, pos5-pos4-1));
        expect_day = stoi(line.substr(pos5+1));
        
        // 执行测试
        bool result = NextDate(year, month, day);
        
        // 为每个测试用例添加独立标识
        SCOPED_TRACE("Test case #" + to_string(test_case_num) + 
                    ": " + line);
        
        EXPECT_TRUE(result);
        EXPECT_EQ(expect_year, year);
        EXPECT_EQ(expect_month, month);
        EXPECT_EQ(expect_day, day);
    }
}
```

修改Cmake文件，编译执行后：

![](https://cdn.ethanzhou.cn/i/2025/05/09/681d9af3c58c8.png)

由于我们只有一个TEST()宏，所以只会显示一条聚合输出结果，显示所有测试用例通过。如果某一行有错误GTest会单独汇报错误的行。至于如何单独显示每一条结果，则可以使用参数化`Test_P（）宏`改写，留给读者探索。

