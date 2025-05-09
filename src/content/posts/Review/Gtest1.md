---
# 必填
title: "GTest单元测试(1): 边界值分析与GTest简单用法"
published: 2025-05-09

# 可选
tags:
  - 软件测试
  - GTest

# 进阶，可选
pin: 5
toc: true
abbrlink: 'softwaretest-gtest1'
---


## 碎碎念

单元测试是项目中非常重要的一个环节。在开发的过程中，应该养成每实现一个小模块就相应地做一系列单元测试的习惯，以保证系统中每个单独模块的正确性。系列文章将以某品牌拖地机器人时间戳被错误设置为背景，您需要通关三个任务，以入门 Gtest 框架使用方法和单元测试。

本篇文章主要介绍输入域测试技术——边界值分析与GTest简单用法。

## 0. 我们需要安装

---

- Windows
- gcc
- make
- cmake
- vscode插件
    - cmake
    - C++ TestMate（作用为可视化文字的Gtest测试结果，也可以选择不安装）

## 1. 背景

---

2020年12月31日，某公司生产拖地机器人由于时间戳被错误地置为13月1日，导致当日无法正常使用。

在此背景下，我们给定该公司工程师为拖地机器人系统设计的基于 C++ 语言编写的 `NextDate` 函数，其功能为输入一个合法的时期后，计算该日期下一天的年、月、日并直接通过参数返回。

```cpp
/* Programme1_1  NextDate.cpp */

// 判断是否为闰年
bool LeapYear(int year) {
    if (year % 4 != 0) {
        return false;
    } else if (year % 100 != 0) {
        return true;
    } else {
        return (year % 400 == 0);
    }
}

// 判断日期是否合法
bool ValidDate(int year, int month, int day) {
    // 检查年份是否合理
    if (year < 1) {
        return false;
    }
    
    // 检查月份是否合理
    if (month < 1 || month > 12) {
        return false;
    }
    
    // 检查天数是否合理
    if (day < 1) {
        return false;
    }
    
    // 根据月份检查天数上限
    if (month == 2) {
        // 二月特殊处理
        int maxDay = LeapYear(year) ? 29 : 28;
        if (day > maxDay) {
            return false;
        }
    } else if (month == 4 || month == 6 || month == 9 || month == 11) {
        // 4月、6月、9月、11月有30天
        if (day > 30) {
            return false;
        }
    } else {
        // 其他月份有31天
        if (day > 31) {
            return false;
        }
    }
    
    return true;
}

bool NextDate(int& year, int& month, int& day) {
    if (!ValidDate(year, month, day)) {
        return false;
    }
    else {
        if (month == 1 || month == 3 || month == 5 || month == 7 || month == 8 || month == 10) {
            if (day == 31) {
                day = 1;
                month++;
            }
            else {
                day++;
            }
        }
        else if (month == 4 || month == 6 || month == 9 || month == 11) {
            if (day == 30) {
                day = 1;
                month++;
            }
            else {
                day++;
            }
        }
        else if (month == 12) {
            if (day == 31) {
                day = 1;
                month = 1;
                year++;
            }
            else {
                day++;
            }
        }
        else {
            if ((LeapYear(year) && day == 29) || (!LeapYear(year) && day == 28)) {
                day = 1;
                month++;
            }
            else {
                day++;
            }
        }
        return true;
    }
}
```

针对这个函数，我们利用三个任务简单入门并实战 Gtest 单元测试，帮助工程师上线前解决潜在问题，开始第一个任务吧！

---
## 2. 任务一


> 使用 GTest 单元测试框架对补全后的 NextDate 函数进行测试。根据PIE模型的原理需要确保 NextDate 函数中所有的代码行均被执行过至少一次（整个 if 语句看作一行，不考虑短路计算）。

不过，在正式介绍利用 GTest 解决任务之前，不妨先花几分钟了解一下 GTest 框架以外的一种原始C测试方式。

### 2.1 原始的测试方式——assert语句

通常在现代的软件测试中，我们会使用一些测试框架辅助我们进行测试。不过也存在一些原始的方法可以进行简单测试，譬如在代码中使用 `assert()` 语句来判断结果的正确性，就像下面这样：

```cpp
/* Programme 2_1 test.cpp */
#include <cassert>
int main(){
	assert(2 * 3 == 6) // true
	assert(2 * 3 == 5) // faluse
	assert(/* Expression */)
	return 0;
}
```

使用`assert()`的目的是当测试结果不正确时，测试程序会终止，从而能被检测到该测试程序是否正常结束。至于为什么引入`assert()`的例子，其实它与 GTest 提供的 `ASSERT_*()` 和 `EXPECT_*()` 宏定义在用法上几乎是没有区别，便于对后面的学习建立起直观的印象。

### 2.2 GTest——Part Of GoogleTest

Google Test（简称 GTest）是 Google 开发的一个 C++ 单元测试框架，用于编写和运行自动化测试。它提供了丰富的断言宏、测试夹具（Fixtures）、参数化测试等功能，帮助开发者编写可维护的测试代码。GoogleTest主要有 GTest 和 GMock 两个功能模块。第三节会简单介绍 GTest 模块的基本用法，不涉及 GMock 模块的内容。

### 2.3 集成 GTest 到项目中

在 GTest 项目的 [README](https://github.com/google/googletest/blob/main/googletest/README.md) 文件中提供了四种方法将 GTest 集成到我们的项目：

- Embed the GoogleTest source code as a direct copy in the main project's source tree. This is often the simplest approach, but is also the hardest to keep up to date. Some organizations may not permit this method.
- Add GoogleTest as a git submodule or equivalent. This may not always be possible or appropriate. Git submodules, for example, have their own set of advantages and drawbacks.
- Download the GoogleTest source code manually and place it at a known location. This is the least flexible approach and can make it more difficult to use with continuous integration systems, etc.
- Use **CMake** to download GoogleTest as part of the build's configure step. This approach doesn't have the limitations of the other methods.

我们采用文档中最为详细介绍的方案，即利用 `CMake` 脚本获取 GTest 项目到本地。该脚本在官方 README 文档中已经给出，针对我们的任务，略微修改后如下：

```csharp
# Programme 2_2 CMakeLists.txt

# 选择适合你项目的版本
cmake_minimum_required(VERSION 3.16)  

# 启用测试
enable_testing()

include(FetchContent)
FetchContent_Declare(
  googletest
  # 请指定依赖的特定提交版本，并建议定期更新
  URL <https://github.com/google/googletest/archive/5376968f6948923e2411081fd9372e71a59d8e77.zip>
)
# 针对Windows系统：防止覆盖父项目的编译器/链接器设置
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

# 待测试的功能代码
add_library(myLib src/nextDate.cpp)
# 测试用例
add_executable(testBin test/test.cpp)
# 链接
target_link_libraries(testBin
    myLib
    gtest 
    gtest_main
)

# 添加测试
add_test(NAME MyTests COMMAND testBin)
```

`CMake` 脚本的编写并不是我们学习的重点，大家对照注释懂得什么意思即可。

### 2.4 在 VSCODE 中建立起项目结构

```cpp
sorrow.../
├── .vscode/            # VS Code 配置文件目录
├── build/              # 构建输出目录（编译生成的文件）
├── src/
│   └── nextDate.cpp    # 主功能实现代码（C++）
├── test/
│   └── test.cpp        # 测试用例代码
└── CMakeLists.txt      # CMake 构建配置文件
```

如下图红框所示：

![](https://cdn.ethanzhou.cn/i/2025/05/09/681d967e3bf7c.png)
### 2.5 编写测试用例

采用 **等价类划分** 与 **边界值分析（Boundary Value Analysis, BVA）**方法，构建任务一测试用例,具体步骤如下：

1. **划分有效类与无效类**
2. **选取边界点**：针对输入变量的最小值、略高于最小值、正常值、略低于最大值、最大值进行测试。
    - 天数：1（最小）、28/29/30/31（根据月份的最大值）
    - 月份：1（最小）、2（特殊）、6（正常）、11（略低于最大）、12（最大）
    - 年份：闰年（如2020）和非闰年（如2023）

所以我们容易设计出如下的测试用例：

|测试用例|测试的边界|覆盖场景|边界点|
|---|---|---|---|
|`InvaliDate1`|2月30日（闰月非法日期）|无效输入验证|无效值|
|`InvaliDate2`|1月32日（大月非法日期）|无效输入验证|无效值|
|`InvaliDate3`|4月31日（小月非法日期）|无效输入验证|无效值|
|`InvaliMonth`|13月1日（非法月份）|无效输入验证|无效值|
|`InvaliNegD`|1月-1日（负数日）|无效输入验证|无效值|
|`InvaliNegM`|-1月10日（负数月）|无效输入验证|无效值|
|`CommonMonth`|6月10日（月中普通日期）|仅天数+1|正常值|
|`CommonDay`|4月30日（非年末月的最后一天）|月份+1，天数归1|最大值|
|`CrossYear`|12月31日（年末最后一天）|年份+1，月份和天数归1|最大值|
|`LeapYear`|2月29日（闰年最后一天）|闰年2月的最后一天，月份 +1，天数归 1|最大值|
|`NonLeapYear`|2月28日（非闰年2月最后一天）|非闰年2月最后一天|最大值|
|..|..|..|..|

由于我们的目的是计算下一天，所以不存在作日期的减法，最小值和略高于最小值可以不用做测试（你也可以理解为它们都属于正常值）。以上给出的测试用例表没有涉及到略低于最大值（即给出非闰年2月27日/闰年2月28日/小月29日/大月30日），留给读者们自己编写。

给出未补充完全的测试用例 `text.cpp` 的代码如下：

```bash
/* Programme 2_2 test.cpp */
#include "gtest/gtest.h"
 
// 引入 NextDate 函数声明
bool NextDate(int& year, int& month, int& day);
 
// 测试用例：ValidDate 为 false 的情况
TEST(NextDateTest, InvalidDate1) {
    int year = 2023;
    int month = 2;
    int day = 30;
    EXPECT_FALSE(NextDate(year, month, day));
}

TEST(NextDateTest, InvalidDate2) {
    int year = 2023;
    int month = 1;
    int day = 32;
    EXPECT_FALSE(NextDate(year, month, day));
}

TEST(NextDateTest, InvalidDate3) {
    int year = 2023;
    int month = 4;
    int day = 31;
    EXPECT_FALSE(NextDate(year, month, day));
}

TEST(NextDateTest, InvalidMonth) {
    int year = 2023;
    int month = 13;
    int day = 1;
    EXPECT_FALSE(NextDate(year, month, day));
}

TEST(NextDateTest, InvalidNagM) {
    int year = 2023;
    int month = -1;
    int day = 30;
    EXPECT_FALSE(NextDate(year, month, day));
}

TEST(NextDateTest, InvalidNagD) {
    int year = 2023;
    int month = 1;
    int day = -1;
    EXPECT_FALSE(NextDate(year, month, day));
}

// 测试用例：年份未跨年，月份未跨月，并且天数为该月最大天数的情况（最大值）
TEST(NextDateTest, CommonDay) {
    int year = 2023;
    int month = 4;
    int day = 30;
    EXPECT_TRUE(NextDate(year, month, day));
    EXPECT_EQ(year, 2023);
    EXPECT_EQ(month, 5);
    EXPECT_EQ(day, 1);
}
 
// 测试用例：年份未跨年，月份未跨月，并且天数不为该月最大天数的情况（正常值）
TEST(NextDateTest, CommonMonth) {
    int year = 2023;
    int month = 6;
    int day = 10;
    EXPECT_TRUE(NextDate(year, month, day));
    EXPECT_EQ(year, 2023);
    EXPECT_EQ(month, 6);
    EXPECT_EQ(day, 11);
}
 

// 测试用例：年份跨了一次的情况（最大值）
TEST(NextDateTest, CrossYear) {
    int year = 2023;
    int month = 12;
    int day = 31;
    EXPECT_TRUE(NextDate(year, month, day));
    EXPECT_EQ(year, 2024);
    EXPECT_EQ(month, 1);
    EXPECT_EQ(day, 1);
}
 
// 测试用例：闰年 2 月 29 日的情况（最大值）
TEST(NextDateTest, LeapYear) {
    int year = 2020;
    int month = 2;
    int day = 29;
    EXPECT_TRUE(NextDate(year, month, day));
    EXPECT_EQ(year, 2020);
    EXPECT_EQ(month, 3);
    EXPECT_EQ(day, 1);
}

// 测试用例：非闰年 2 月 28 日的情况（最大值）
TEST(NextDateTest, NonLeapYear) {
    int year = 2022;
    int month = 2;
    int day = 28;
    EXPECT_TRUE(NextDate(year, month, day));
    EXPECT_EQ(year, 2022);
    EXPECT_EQ(month, 3);
    EXPECT_EQ(day, 1);
}

```

至此，任务一所需代码已经全部写完，我们介绍 GTest 的基本用法，解释一下我们测试文件中所用的语句。

---
## 3.GTest的用法

### 3.1 在CMake中链接GTest库

GTest提供了四个库文件：`gtest`、`gtest_main`、`gmock`、`gmock_main`。目前我们不讨论GMock，因此我们只需要用到前两个库。

我们通过阅读GTest的官方文档可以知道，GTest测试框架是提供了`main()`函数的。因此我们需要将待测试的功能代码添加为库文件，并把使用GTest框架的测试代码添加为可执行文件，最后将我们的库和`gtest`、`gtest_main`两个库链接到该可执行文件中去即可。

类似下面这样：

```cpp
# 待测试的功能代码
add_library(myLib src/nextDate.cpp)
# 测试用例
add_executable(testBin test/test.cpp)
# 链接
target_link_libraries(testBin
    myLib
    gtest 
    gtest_main
)
```

### **3.2 `TEST()` 宏**

首先，大部分情况下我们不需要自己写`main()`函数。我们利用GTest提供的`TEST()`宏来完成我们的测试代码，用法如下：

```cpp
TEST(TestSuiteName, TestName) {
  ... test body ...
}
```

这里的两个输入参数由左至右分别是该项测试的分类名称和测试名称。这样我们在输出结果中就能根据测试的分类和名称方便地找到某一项测试的结果，我们来看一看任务一的测试结果是什么样子的。

在VSCODE中按住 `CTRL+SHIFT+P` 打开命令，键入：

```bash
CMAKE:BUILD
```

以编译我们任务一的代码（我已经补全了测试用例）：

![](https://cdn.ethanzhou.cn/i/2025/05/09/681d96f4e1880.png)

我们可以得到任务一的单元测试结果文件 `testBin`，如果没有显示结果，可以在终端中键入:

```bash
cd build
./testBin
```

![](https://cdn.ethanzhou.cn/i/2025/05/09/681d972565c9b.png)

可以看到 15 条测试用例全部 PASS ！显示的也是：

```bash
测试的分类名称.测试名称
```

与我们 `TEST()` 的输入参数对应。

> Tips: GTest的测试结果只有文本输出，需要引入第三方软件才能有比较好看的可视化结果。官方推荐的是GTest Runner和GoogleTest UI.
> 
> 这里推荐一个VSCode里的可视化插件：[C++ TestMate](https://marketplace.visualstudio.com/items?itemName=matepek.vscode-catch2-test-adapter)，使用该插件运行测试样例，可以得到下面的可视化结果：
> ![](https://lumiowo.github.io/2021/01/17/CMake-%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E5%92%8CGoogleTest%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8/image-20210117205156858.png)

### **3.3 断言宏 `ASSERT_*()`和`EXPECT_*()`**

GTest提供了一系列用于测试执行结果是否正确的宏定义，主要有两类：`ASSERT_*()`和`EXPECT_*()` (回旋镖，大家想到了前文介绍的 `assert()`了嘛，理解起来一样一样的)。这两个宏的区别在于，当断言失败时，`ASSERT_*()`产生一个fatal error并从当前执行的位置返回，而`EXPECT_*()`产生一个nonfatal error并运行函数继续执行。

如果你想在断言失败的时候输出一些额外的信息，使用`<<`操作符即可，例如：

```bash
ASSERT_EQ(x.size(), y.size()) << "Vectors x and y are of unequal length";
```

下面给出若干常用的宏定义用法。

- 布尔量

|Fatal|Nonfatal|说明|
|---|---|---|
|ASSERT_TRUE(condition);|EXPECT_TRUE(condition);|条件为真|
|ASSERT_FALSE(condition);|EXPECT_FALSE(condition);|条件为假|

- 二元比较

|Fatal|Nonfatal|说明|
|---|---|---|
|ASSERT_EQ(val1, val2);|EXPECT_EQ(val1, val2);|val1 == val2|
|ASSERT_NE(val1, val2);|EXPECT_NE(val1, val2);|val1 != val2|
|ASSERT_LT(val1, val2);|EXPECT_LT(val1, val2);|val1 < val2|
|ASSERT_LE(val1, val2);|EXPECT_LE(val1, val2);|val1 <= val2|
|ASSERT_GT(val1, val2);|EXPECT_GT(val1, val2);|val1 > val2|
|ASSERT_GE(val1, val2);|EXPECT_GE(val1, val2);|val1 >= val2|

- 字符串比较

参数为C字符串，即以`\\0`字符结尾的一维字符数组。

|Fatal|Nonfatal|说明|
|---|---|---|
|ASSERT_STREQ(str1,str2);|EXPECT_STREQ(str1,str2);|内容相同|
|ASSERT_STRNE(str1,str2);|EXPECT_STRNE(str1,str2);|内容不同|
|ASSERT_STRCASEEQ(str1,str2);|EXPECT_STRCASEEQ(str1,str2);|无视大小写，内容相同|
|ASSERT_STRCASENE(str1,str2);|EXPECT_STRCASENE(str1,str2);|无视大小写，内容不同|

### **3.4 测试夹具 `TEST_F()`（选读）**

在构建测试样例时，我们需要在`TEST()`函数体内构建待测的数据对象。但很多时候，我们可能需要对同一组数据在多个不同的`TEST()`里进行不同的测试。这时，如果每次都手动构造同一份数据，代码会显得重复且繁杂。

GTest提供了`TEST_F()`宏和一个基类来为我们解决这个问题。为构造重复使用的数据，我们需要继承`::testing::Test`类，然后在这个新的类中构造我们的数据。例子如下：

```
class QueueTest : public ::testing::Test {
protected:
    // 成员变量
    Queue<int> q0_;
    Queue<int> q1_;
    Queue<int> q2_;

    // 可以看做构造函数
    void SetUp() override {
        q1_.Enqueue(1);
        q2_.Enqueue(2);
        q2_.Enqueue(3);
    }

    // 可以看做析构函数，这里不需要
    // void TearDown() override {}
};
```

首先，我们需要把访问属性设为`protected`，然后继承父类的`SetUp()`和`TearDown()`函数。这两个函数相当于这个类的构造函数和析构函数。我们在`SetUp()`函数里初始化我们的数据，之后我们就可以用`TEST_F()`函数来访问这些数据了。

> Tips: SetUp()函数很容易拼成Setup()，导致没有覆盖父类的对应函数。因此建议使用C++11标准的override关键字，让编译器帮你检查拼写问题。

`TEST_F()`的使用如下所示：

```bash
TEST_F(TestFixtureName, TestName) {
  ... test body ...
}
```

它的参数和`TEST()`类似，但在第一个参数上有所区别。`TestFixtureName`必须是我们新继承的测试类的类名。测试部分的例子如下：

```
TEST_F(QueueTest, IsEmptyInitially) {
    EXPECT_EQ(q0_.size(), 0);
}

TEST_F(QueueTest, DequeueWorks) {
    int* n = q0_.Dequeue();
    EXPECT_EQ(n, nullptr);

    n = q1_.Dequeue();
    ASSERT_NE(n, nullptr);
    EXPECT_EQ(*n, 1);
    EXPECT_EQ(q1_.size(), 0);
    delete n;

    n = q2_.Dequeue();
    ASSERT_NE(n, nullptr);
    EXPECT_EQ(*n, 2);
    EXPECT_EQ(q2_.size(), 1);
    delete n;
}
```

上面这段测试代码运行时的流程是这样的：

1. GTest构建一个`QueueTest`对象，不妨称为`t1`；
2. `t1.SetUp()`申请内存并初始化数据；
3. `IsEmptyInitially`在`t1`的数据上运行；
4. 测试结束后，`t1.TearDown()`释放之前申请的内存空间；
5. `t1`对象被析构；
6. 回到第1步并运行`DequeueWorks`测试。

> 参考资料：
> 
> 1. GoogleTest - Generic Build Instructions，[https://github.com/google/googletest/blob/master/googletest/README.md](https://github.com/google/googletest/blob/master/googletest/README.md)
> 2. Googletest Primer，[https://github.com/google/googletest/blob/master/docs/primer.md](https://github.com/google/googletest/blob/master/docs/primer.md)
> 3. LumiOwo的博客,[CMake - 单元测试和 GoogleTest 简单使用 | LumiOwO 的博客](https://lumiowo.github.io/2021/01/17/CMake-%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E5%92%8CGoogleTest%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8/)

