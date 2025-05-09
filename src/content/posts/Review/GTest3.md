---
# 必填
title: "GTest单元测试(3): 组合测试与PICT工具"
published: 2025-05-09

# 可选
tags:
  - 软件测试
  - GTest

# 进阶，可选
pin: 7
toc: true
abbrlink: 'softwaretest-gtest3'
---

## 碎碎念 

软件的输入域划分测试是一类典型的基于规格的测试方法，输入域代表了软件产品规格定义的输入空间。在前面两篇文章中我们以及接触了等价类划分、边界值分析以及随机测试三种输入域测试方法。组合测试作为我们介绍的最后一种重要的输入域划分测试方法，通过建立被测软件的组合测试输入模型，优化生成组合测试用例，重点检测软件输入域参数及其相互作用可能触发的失效。

---
## 1. 为什么我们需要组合测试

在测试工作中，经常会遇到这样的场景：一个软件功能有多个输入项，每个输入项有多个可选项；一个接口有多个参数，每个参数有多个值。这样的情况在平时非常常见，如果按照排列组合，得到的测试用例数目非常庞大。

举个直观的例子就很容易明白了。有一个接口函数，该函数有3个参数，每个参数又可以取值4个，那如果要验证所有参数传入情况的话则需要测试4×4×4=64种情况。如果参数和取值状态更多话，那将是一个灾难。

有没有一种更好的办法，少做一些测试，同时可以满足测试覆盖率呢？答案是有的，就是我们今天要讲的“因子组合测试覆盖Pairwise”，下面先来简单的介绍一下什么Pairwise。

Pairwise是L.L.Thurstone(29 May1887–30 September 1955)在1927年首先提出来的。他是美国的一位心理统计学家，Pairwise也正是基于数学统计和对传统的正交分析法进行优化后得到的产物。

Pairwise基于如下2个假设：**每一个维度都是正交的，即每一个维度互相都没有交集** 。根据数学统计分析，73%的缺陷（单因子是35%，双因子是38%）是由单因子或2个因子相互作用产生的，19%的缺陷是由3个因子相互作用产生的。因此，Pairwise基于覆盖所有2因子的交互作用产生的用例集合性价比最高而产生的。

软件测试分为黑盒测试和白盒测试，Pairwise算法是针对软件测试中的黑盒测试提出来的一个行之有效的测试方法。

概念往往是晦涩难懂的，下面举一个简单的例子，看完你就明白Pairwise算法是怎么挑选测试case的。

> 现有接口S，有三个输入变量X、Y、Z，取值分别为：D(X) = {x1, x2}; D(Y) = {y1, y2}; D(Z) = {z1,z2}，如下图:

![](https://cdn.ethanzhou.cn/i/2025/05/09/681d9dfe198da.png)

**Pairwise算法过程**：从表的最后一行开始，如果这行的两两组合值能够在上面的行或此表中找到，那么这行就可从用例集中删除。

例如，TC8包含的两两组合值为(X2-Y2,X2-Z2,Y2-Z2)，X2-Y2在TC7中存在，X2-Z2在TC6中存在，Y2-Z2在TC4中存在，则此行删除；

TC7包含的两两组合值为（X2-Y2,X2-Z1,Y2-Z1），X2-Y2在此表中已找不到重复的值，所以保留。依此方法，最后得到的测试用例集如上面的右图。很明显，经过Pairwise过程，测试用例数减少了一半。

那么如果每次都要进行手工计算除了非常浪费时间外，也容易出错，一旦参数多起来的话手工也无能为力了。那是不是可以开发一个自动化程序，让它自动输出结果。这个想法非常好，但是不需要我们在重复造轮子了，市面上已经有了非常好用的工具，那就是PICT工具。

## 2. 任务三

> 使用PICT工具生成一组强度为2的组合测试用例，测试用例存储至一个文本文件CT_test_data.txt，每行存放一条测试用例，测试用例内各个字段之间的分割方式按照csv规范。编写程序读取测试用例文件，使用GTest单元测试框架执行这些测试用例对NextDate函数进行测试。

### 2.1 安装 PICT 工具

首先，我们需要在 PICT 上安装好测试工具。进入PICT项目的Github主页：[microsoft/pict: Pairwise Independent Combinatorial Tool](https://github.com/microsoft/pict)

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9e3e7412c.png)

点击上图红框标注的 Release:

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9e7745a1e.png)

下载 pict.exe。

### 2.2 使用 PICT 工具

使用 PICT 的方法很简单，我们需要先进入安装目录，在目录下新建一个txt文件 demo.txt:

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9eaf25b0f.png)

demo.txt 中的参数如下，我们会从以下的参数里构建我们的组合测试用例：

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9eced0f82.png)

> 注意：冒号、逗号都是英文字符，否则无法识别。

在PICT目录下打开系统Powershell窗口，输入：

```bash

pict demo.txt > CT_test_data.txt 
```

会显示如下结果：

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9eef24bf3.png)

打开PICT为我们创建好的 CT_test_data.txt:

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9f090ca0f.png)

编写我们的测试代码test2.cpp:

```cpp
/* Programme 2.1 test.2.cpp*/
#include <gtest/gtest.h>
#include <fstream>
#include <string>
#include <vector>

bool NextDate(int& year, int& month, int& day);
bool ValidDate(int year, int month, int day);

// 测试数据结构
struct TestData {
    int year;
    int month;
    int day;
    bool expected;
};

// 读取测试数据
std::vector<TestData> readTestData(const std::string& filename) {
    std::vector<TestData> testData;
    std::ifstream file(filename);
    
    if (!file.is_open()) {
        std::cerr << "无法打开文件: " << filename << std::endl;
        return testData;
    }

    // 跳过标题行
    std::string header;
    std::getline(file, header);

    int year, month, day;
    while (file >> year >> month >> day) {
        TestData data;
        data.year = year;
        data.month = month;
        data.day = day;
        data.expected = ValidDate(year, month, day);
        testData.push_back(data);
    }

    return testData;
}

// 测试用例
TEST(NextDateTest, TestCases) {
    auto testData = readTestData("../src/CT_test_data.txt");
    
    for (const auto& data : testData) {
        int inputYear = data.year;
        int inputMonth = data.month;
        int inputDay = data.day;
        
        // 保存原始输入值
        int originalYear = inputYear;
        int originalMonth = inputMonth;
        int originalDay = inputDay;
        
        bool result = NextDate(inputYear, inputMonth, inputDay);
        
        // 使用 ValidDate 生成预期输出
        int expectedYear = originalYear;
        int expectedMonth = originalMonth;
        int expectedDay = originalDay;
        
        if (data.expected) {
            // 尝试所有可能的日期，直到找到有效的下一天
            while (true) {
                expectedDay++;
                if (expectedDay > 31) {
                    expectedDay = 1;
                    expectedMonth++;
                }
                if (expectedMonth > 12) {
                    expectedMonth = 1;
                    expectedYear++;
                }
                if (ValidDate(expectedYear, expectedMonth, expectedDay)) {
                    break;
                }
            }
        }
        
        // 验证结果
        if (data.expected) {
            EXPECT_TRUE(result) << "输入日期: " << originalYear << "-" << originalMonth << "-" << originalDay;
            EXPECT_EQ(inputYear, expectedYear) << "输入日期: " << originalYear << "-" << originalMonth << "-" << originalDay;
            EXPECT_EQ(inputMonth, expectedMonth) << "输入日期: " << originalYear << "-" << originalMonth << "-" << originalDay;
            EXPECT_EQ(inputDay, expectedDay) << "输入日期: " << originalYear << "-" << originalMonth << "-" << originalDay;
        } else {
            EXPECT_FALSE(result) << "输入日期: " << originalYear << "-" << originalMonth << "-" << originalDay;
        }
    }
}

int main(int argc, char **argv) {
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}

```

修改CMakeList后，编译执行我们的项目，并查看我们的测试结果（具体步骤在前两篇博文中已经讲述）：

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9f311225b.png)

至此，任务三圆满完成！

### 2.3 PICT工具的约束条件

在大多数情况下，测试人员往往会更多生成合法的测试用例。而在我们的例子中，PICT工具生成了明显的不合法日期，譬如 2000/2/30等：

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9f63f38fc.png)

PICT工具提供了约束条件让我们可以避免生成这样的测试用例。语法类似于标准 Sql 语句，给出一些例子供读者揣摩：

```cpp
条件约束(LIKE, IN, AND, OR, NOT)，必须分号结束:

[year] <> 2000;  // 年份不能为 2000

[year] < 120 ; // 年份小于 120 

[year] <> 2000 OR [month] < 10; // 年份不为 2000且月份小于 10

[nickname] LIKE “李*”; // 昵称为李开头

IF [username] = “空” THEN NOT [password] = “空”; // 如果用户名为空，那么密码不为空

IF [Cluster size] in {512, 1024, 2048} THEN [Compression] = "Off"; // ...

IF [username] = “空” THEN [password] <> “空” AND [captcha] <>“空”; // ...
```

那么在我们的例子里，我们显然不希望月份为 2 时出现 30/31 日的情况，那么我们可以修改 demo.txt 如下：

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9f94420dd.png)

重新利用 PICT 生成测试用例：

![image.png](https://cdn.ethanzhou.cn/i/2025/05/09/681d9fac0ab73.png)

可以看到生成的用例里没有 2月30/31日了。

