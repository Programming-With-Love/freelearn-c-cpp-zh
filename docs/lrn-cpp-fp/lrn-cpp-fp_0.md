# 前言

函数式编程是一种通过组合纯函数构建计算机程序的元素和结构的风格，避免共享状态、可变数据和副作用，就像我们通常在数学中看到的那样。代码函数中的变量表示函数参数的值，并且类似于数学函数。这个想法是程序员定义包含表达式、定义和可以用变量表示的参数的函数来解决问题。

函数式编程是声明式的而不是命令式的，这意味着编程是通过表达式或声明而不是语句完成的。函数式编程的应用状态通过纯函数流动，因此它避免了副作用。与命令式编程相比，应用状态通常与对象中的方法共享和共存。在命令式编程中，表达式被评估，并且结果值被赋给变量。例如，当我们将一系列表达式组合成一个函数时，结果值取决于该时刻变量的状态。由于状态不断变化，评估的顺序很重要。在函数式编程中，禁止破坏性赋值，每次赋值发生时，都会引入一个新变量。最重要的是，函数式代码往往更简洁、可预测，比命令式或面向对象的代码更容易测试。

尽管有一些专门设计用于函数式编程的语言，比如 Haskell 和 Scala，我们也可以使用 C++来完成函数式编程的设计，正如我们将在本书中讨论的那样。

# 本书内容

《第一章》《现代 C++的深入探讨》概述了现代 C++，包括现代 C++中几个新特性的实现，比如 auto 关键字、decltype 关键字、空指针、基于范围的 for 循环、标准模板库、Lambda 表达式、智能指针和元组。

《第二章》《在函数式编程中操作函数》介绍了函数式编程中操作函数的基本技术；它们是第一类函数技术、纯函数和柯里化技术。通过应用第一类函数，我们可以将函数视为数据，这意味着它可以分配给任何变量，而不仅仅是作为函数调用。我们还将应用纯函数技术，使函数不再产生副作用。此外，为了简化函数，我们可以应用柯里化技术，通过在每个函数中评估一系列带有单个参数的函数来减少多参数函数。

《第三章》《将不可变状态应用于函数》解释了我们如何为可变对象实现不可变对象。我们还将深入研究第一类函数和纯函数，这些内容在上一章中讨论过，以产生一个不可变对象。

《第四章》《使用递归算法重复方法调用》讨论了迭代和递归的区别，以及为什么递归技术对函数式编程更好。我们还将列举三种递归：函数式、过程式和回溯递归。

《第五章》《使用惰性求值延迟执行过程》解释了如何延迟执行过程以获得更高效的代码。我们还将实现缓存和记忆化技术，使我们的代码运行更快。

第六章，*使用元编程优化代码*，讨论了使用元编程在编译时执行代码以优化代码。我们还将讨论如何将流程控制重构为模板元编程。

第七章，*使用并发运行并行执行*，向我们展示了如何在 C++编程中运行多个线程，以及如何同步线程以避免死锁。我们还将在 Windows 操作系统中应用线程处理。

第八章，*使用函数式方法创建和调试应用程序*，详细介绍了我们在前几章中讨论的所有技术，以设计函数式编程。此外，我们将尝试调试代码，以找到解决方案，如果出现意外结果或程序在执行中崩溃。

# 您需要什么来阅读本书

要阅读本书并成功编译所有源代码示例，您需要一台运行 Microsoft Windows 8.1（或更高版本）的个人电脑，并包含以下软件：

+   GCC 的最新版本，支持 C++11、C++14 和 C++17（在撰写本书时，最新版本是 GCC v7.1.0）

+   Microsoft Visual Studio 2017 提供的 Microsoft C++编译器，支持 C++11、C++14 和 C++17（适用于第七章，*使用并发运行并行执行*）

+   Code::Blocks v16.01（所有示例代码均使用 Code::Blocks IDE 编写；但是，使用此 IDE 是可选的）

# 这本书适合谁

本书适用于熟悉面向对象编程的 C++开发人员，他们有兴趣学习如何应用函数式范式来创建健壮且可测试的应用程序。

# 约定

在本书中，您将找到一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL 和用户输入显示如下："`auto`关键字也可以应用于函数，以自动推断函数的返回类型。"

代码块设置如下：

```cpp
    int add(int i, int j)
    {
      return i + j;
    }

```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```cpp
    // Initializing a string variable
 Name n = {"Frankie Kaur"};
       cout << "Initial name = " << n.str;
       cout << endl; 

```

**新术语**和**重要单词**以粗体显示。

警告或重要提示会出现在这样的形式。

提示和技巧会出现在这样的形式。