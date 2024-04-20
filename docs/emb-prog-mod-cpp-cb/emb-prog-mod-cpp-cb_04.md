# 第四章：处理中断

嵌入式应用程序的主要任务之一是与外部硬件外设通信。使用输出端口向外设发送数据很容易理解。但是，当涉及到读取时，情况变得更加复杂。

嵌入式开发人员必须知道何时可以读取数据。由于外围设备外部于处理器，这可能发生在任何时刻。

在本章中，我们将学习什么是中断以及如何处理中断。在以 8051 为目标平台的 8 位微控制器上，我们将学习以下主题：

+   如何实现基本中断处理

+   如何使用定时器中断从 MCU 的输出引脚生成信号

+   如何使用中断来计算 MCU 外部引脚上的事件

+   如何使用中断在串行通道上进行通信

通过完成以下示例，我们将学习这些主题：

+   实现中断服务例程

+   使用 8 位自动重装模式生成 5 kHz 方波信号

+   使用定时器 1 作为事件计数器来计算 1 Hz 脉冲

+   串行接收和发送数据

了解如何处理中断的核心概念将帮助您实现响应灵敏且节能的嵌入式应用程序。

然而，在此之前，我们将获取一些背景知识。

# 数据轮询

从外部源等待数据的第一种方法称为**轮询**。应用程序周期性地查询外部设备的输入端口，以检查是否有新数据。这种方法易于实现，但有显著的缺点。

首先，它浪费处理器资源。大多数轮询调用报告数据尚不可用，我们需要继续等待。由于这些调用不会导致某些数据处理，这是对计算资源的浪费。此外，轮询间隔应该足够短，以便快速响应外部事件。开发人员应该在处理器功率的有效利用和响应时间之间寻求折衷。

其次，它使程序的逻辑变得复杂。如果程序应该每 5 毫秒轮询一次事件，例如，那么它的任何子程序都不应该超过 5 毫秒。结果，开发人员人为地将代码分成更小的块，并组织它们之间的复杂切换，以允许轮询。

# 中断服务例程

中断是轮询的一种替代方法。一旦外部设备有新数据，它会在处理器中触发一个称为**中断**的事件。顾名思义，它会中断正常的执行指令流程。处理器保存其当前状态，并开始从不同的地址执行指令，直到遇到从中断返回的指令。然后，它读取保存的状态以继续执行从中断时刻开始的指令流。这种替代的指令序列称为**中断服务例程**（**ISR**）。

每个处理器都定义了自己的一组指令和约定来处理中断；然而，在处理中断时，它们都使用相同的一般方法：

+   中断由数字标识，从 0 开始。这些数字映射到硬件**中断请求线**（**IRQ**），这些线物理上对应于特定的处理器引脚。

+   当 IRQ 线被激活时，处理器使用其编号作为中断向量数组中的偏移量，以定位中断服务例程的地址。中断向量数组存储在内存中的固定地址上。

+   开发人员可以通过更新中断向量数组中的条目来定义或重新定义 ISR。

+   处理器可以被编程以启用或禁用中断，无论是针对特定的 IRQ 线还是一次性禁用所有中断。当中断被禁用时，处理器不会调用相应的 ISR，尽管可以读取 IRQ 线的状态。

+   IRQ 线可以编程触发中断，取决于物理引脚上的信号。这可以是信号的低电平、高电平，或者边沿（即从低到高或从高到低的过渡）。

# ISR 的一般考虑

这种方法不会浪费处理器资源进行轮询，并且由于中断处理是在硬件级别执行的，因此提供了非常短的反应时间。然而，开发人员应该注意其具体情况，以避免未来出现关键或难以检测的问题。

首先，同时处理多个中断，或者在处理前一个中断的同时响应相同的中断，是很难实现的。这就是为什么 ISR 在中断被禁用时执行。这可以防止 ISR 被另一个中断打断，但也意味着待处理中断的反应时间可能会更长。更糟糕的是，如果中断不及时重新启用，这可能会导致数据或事件丢失。

为了避免这种情况，所有 ISR 都被编写为简短的。它们只做最少量的工作，以从设备中读取或确认数据。复杂的数据分析和处理是在 ISR 之外进行的。

# 8051 微控制器中断

8051 微控制器支持六个中断源-复位、两个硬件中断、两个定时器中断和一个串行通信中断：

| **中断号** | **描述** | **字节偏移** |
| --- | --- | --- |
| | 复位 | 0 |
| 0 | 外部中断 INT0 | 3 |
| 1 | 定时器 0（TF0） | 11 |
| 2 | 外部中断 INT1 | 19 |
| 3 | 定时器 1（TF1） | 27 |
| 4 | 串行 | 36 |

中断向量数组位于地址 0 处；除了复位之外，每个条目的大小为 8 字节。虽然最小的 ISR 可以适应 8 字节，但通常，条目包含将执行重定向到实际 ISR 的代码，该 ISR 位于其他地方。

复位入口是特殊的。它由复位信号激活，并立即跳转到主程序所在的地址。

8051 定义了一个称为**中断使能**（**EA**）的特殊寄存器，用于启用和禁用中断。它的 8 位分配如下：

| **位** | **名称** | **含义** |
| --- | --- | --- |
| 0 | EX0 | 外部中断 0 |
| 1 | ET0 | 定时器 0 中断 |
| 2 | EX1 | 外部中断 1 |
| 3 | ET1 | 定时器 1 中断 |
| 4 | ES | 串口中断 |
| 5 | - | 未使用 |
| 6 | - | 未使用 |
| 7 | EA | 全局中断控制 |

将这些位设置为 1 会启用相应的中断，设置为 0 会禁用它们。EA 位启用或禁用所有中断。

# 实现中断服务例程

在这个配方中，我们将学习如何为 8051 微控制器定义中断服务例程。

# 如何做...

按照以下步骤完成这个配方：

1.  切换到我们在第二章中设置的构建系统，*设置环境*。

1.  确保安装了 8051 仿真器：

```cpp
# apt install -y mcu8051ide
```

1.  启动`mcu8051ide`并创建一个名为`Test`的新项目。

1.  创建一个名为`test.c`的新文件，并将以下代码片段放入其中。这会为每个定时器中断增加一个内部`counter`：

```cpp
#include<mcs51reg.h> 

volatile int Counter = 0;
void timer0_ISR (void) __interrupt(1) /*interrupt no. 1 for Timer0 */
{ 

  Counter++;
} 

void main(void) 
{ 
  TMOD = 0x03; 
  TH0 = 0x0; 
  TL0 = 0x0; 
  ET0 = 1; 
  TR0 = 1;
  EA = 1;
  while (1); /* do nothing */ 
} 
```

1.  选择工具|编译来构建代码。消息窗口将显示以下输出：

```cpp
Starting compiler ...

cd "/home/dev"
sdcc -mmcs51 --iram-size 128 --xram-size 0 --code-size 4096 --nooverlay --noinduction --verbose --debug -V --std-sdcc89 --model-small "test.c"
sdcc: Calling preprocessor...
+ /usr/bin/sdcpp -nostdinc -Wall -obj-ext=.rel -D__SDCC_NOOVERLAY -DSDCC_NOOVERLAY -D__SDCC_MODEL_SMALL -DSDCC_MODEL_SMALL -D__SDCC_FLOAT_REENT -DSDCC_FLOAT_REENT -D__SDCC=3_4_0 -DSDCC=340 -D__SDCC_REVISION=8981 -DSDCC_REVISION=8981 -D__SDCC_mcs51 -DSDCC_mcs51 -D__mcs51 -D__STDC_NO_COMPLEX__ -D__STDC_NO_THREADS__ -D__STDC_NO_ATOMICS__ -D__STDC_NO_VLA__ -isystem /usr/bin/../share/sdcc/include/mcs51 -isystem /usr/share/sdcc/include/mcs51 -isystem /usr/bin/../share/sdcc/include -isystem /usr/share/sdcc/include test.c
sdcc: Generating code...
sdcc: Calling assembler...
+ /usr/bin/sdas8051 -plosgffwy test.rel test.asm
sdcc: Calling linker...
sdcc: Calling linker...
+ /usr/bin/sdld -nf test.lk

Compilation successful
```

1.  选择模拟器|启动/关闭菜单项以激活模拟器。

1.  选择模拟器|动画以慢速模式运行程序。

1.  切换到 C 变量面板，并向下滚动，直到显示 Counter 变量。

1.  观察它随时间的增长：

![](img/6bfb07eb-bdc2-4be0-a095-90ce3bda6141.png)

如您所见，`Counter`变量的值字段现在是 74。

# 它是如何工作的...

对于我们的示例应用程序，我们将使用 8051 微控制器的仿真器。有几种可用；但是，我们将使用 MCU8051IDE，因为它在 Ubuntu 存储库中已经准备好了。

我们将其安装为常规的 Ubuntu 软件包，如下所示：

```cpp
# apt install -y mcu8051ide
```

这是一个 GUI IDE，需要 X Window 系统才能运行。如果您使用 Linux 或 Windows 作为工作环境，请考虑直接从[`sourceforge.net/projects/mcu8051ide/files/`](https://sourceforge.net/projects/mcu8051ide/files/)安装和运行它。

我们创建的简单程序定义了一个名为`Counter`的全局变量，如下所示*：*

```cpp
volatile int Counter = 0;
```

这被定义为`volatile`，表示它可以在外部更改，并且编译器不应尝试优化代码以消除它。

接下来，我们定义了一个名为`timer0_ISR`的简单函数*：*

```cpp
void timer0_ISR (void) __interrupt(1)
```

它不接受任何参数，也不返回任何值。它唯一的作用是增加`Counter`变量。它声明了一个重要的属性，称为`__interrupt(1)`，以让编译器知道它是一个中断处理程序，并且它服务于中断号 1。编译器会自动生成代码，自动更新中断向量数组的相应条目。

在定义 ISR 本身之后，我们配置定时器的参数：

```cpp
TMOD = 0x03; 
TH0 = 0x0; 
TL0 = 0x0;
```

然后，我们打开定时器 0，如下所示：

```cpp
TR0 = 1;
```

以下命令启用定时器 0 的中断：

```cpp
ET0 = 1; 
```

以下代码启用所有中断：

```cpp
EA = 1;
```

在这一点上，我们的 ISR 被定时器的中断周期性地激活。我们运行一个无限循环，因为所有的工作都是在 ISR 内完成的：

```cpp
while (1); // do nothing 
```

当我们在模拟器中运行上述代码时，我们会看到`counter`变量的实际值随时间变化，表明我们的 ISR 被定时器激活。

# 使用 8 位自动重装模式生成 5 kHz 方波信号

在前面的示例中，我们学习了如何创建一个简单的 ISR，只进行计数器增量。让我们让中断例程做一些更有用的事情。在这个示例中，我们将学习如何编程 8051 微控制器，以便它生成具有给定频率的信号。

8051 微控制器有两个定时器 - 定时器 0 和定时器 1 - 都使用两个特殊功能寄存器：**定时器模式**（**TMOD**）和**定时器控制**（**TCON**）进行配置。定时器的值存储在 TH0 和 TL0 定时器寄存器中，用于定时器 0，以及 TH1 和 TL1 定时器寄存器用于定时器 1。

TMOD 和 TCON 位具有特殊含义。TMOD 寄存器的位定义如下：

| **位** | **定时器** | **名称** | **目的** |
| --- | --- | --- | --- |
| 0 | 0 | M0 | 定时器模式选择器 - 低位。 |
| 1 | 0 | M1 | 定时器模式选择器 - 高位。 |
| 2 | 0 | CT | 计数器（1）或定时器（0）模式。 |
| 3 | 0 | GATE | 使能定时器 1，但仅当 INT0 的外部中断为高时。 |
| 4 | 1 | M0 | 定时器模式选择器 - 低位。 |
| 5 | 1 | M1 | 定时器模式选择器 - 高位。 |
| 6 | 1 | CT | 计数器（1）或定时器（0）模式。 |
| 7 | 1 | GATE | 使能定时器 1，但仅当 INT1 的外部中断为高时。 |

低 4 位分配给定时器 0，而高 4 位分配给定时器 1。

M0 和 M1 位允许我们以四种模式之一配置定时器：

| **模式** | **M0** | **M1** | **描述** |
| --- | --- | --- | --- |
| 0 | 0 | 0 | 13 位模式。TL0 或 TL1 寄存器包含对应定时器值的低 5 位，TH0 或 TH1 寄存器包含对应定时器值的高 8 位。 |
| 1 | 0 | 1 | 16 位模式。TL0 或 TL1 寄存器包含对应定时器值的低 8 位，TH0 或 TH1 寄存器包含对应定时器值的高 8 位。 |
| 2 | 1 | 0 | 8 位模式自动重装。TL0 或 TL1 包含对应的定时器值，而 TH0 或 TL1 包含重装值。 |
| 3 | 1 | 1 | 定时器 0 的特殊 8 位模式 |

**定时器控制**（**TCON**）寄存器控制定时器中断。其位定义如下：

| **位** | **名称** | **目的** |
| --- | --- | --- |
| 0 | IT0 | 外部中断 0 控制位。 |
| 1 | IE0 | 外部中断 0 边沿标志。当 INT0 接收到高至低边沿信号时设置为 1。 |
| 2 | IT1 | 外部中断 1 控制位。 |
| 3 | IE1 | 外部中断 1 边沿标志。当 INT1 接收到高至低边沿信号时设置为 1。 |
| 4 | TR0 | 定时器 0 的运行控制。设置为 1 以启动，设置为 0 以停止定时器。 |
| 5 | TF0 | 定时器 0 溢出。当定时器达到其最大值时设置为 1。 |
| 6 | TR1 | 定时器 1 的运行控制。设置为 1 以启动，设置为 0 以停止定时器。 |
| 7 | TF1 | 定时器 1 溢出。当定时器达到其最大值时设置为 1。 |

我们将使用称为自动重载的 8051 定时器的特定模式。在这种模式下，TL0（定时器 1 的 TL1）寄存器包含计时器值，而 TH0（定时器 1 的 TH1）包含重载值。一旦 TL0 达到 255 的最大值，它就会生成溢出中断，并自动重置为重载值。

# 如何做...

按照以下步骤完成此操作：

1.  启动*mce8051ide*并创建一个名为`Test`的新项目。

1.  创建一个名为`generator.c`的新文件，并将以下代码片段放入其中。这将在 MCU 的`P0_0`引脚上生成 5 kHz 信号：

```cpp
#include<8051.h> 

void timer0_ISR (void) __interrupt(1) 
{ 
  P0_0 = !P0_0;
} 

void main(void) 
{ 
  TMOD = 0x02;
  TH0 = 0xa3; 
  TL0 = 0x0; 
  TR0 = 1;
  EA = 1; 
  while (1); // do nothing 
}
```

1.  选择工具|编译以构建代码。

1.  选择模拟器|启动/关闭菜单项以激活模拟器。

1.  选择模拟器|动画以以慢速模式运行程序。

# 它是如何工作的...

以下代码定义了定时器 0 的 ISR：

```cpp
void timer0_ISR (void) __interrupt(1) 
```

在每次定时器中断时，我们翻转 P0 的输入输出寄存器的 0 位。这将有效地在 P0 输出引脚上生成方波信号。

现在，我们需要弄清楚如何编程定时器以生成给定频率的中断。要生成 5 kHz 信号，我们需要以 10 kHz 频率翻转位，因为每个波包括一个高相位和一个低相位。

8051 MCU 使用外部振荡器作为时钟源。定时器单元将外部频率除以 12。对于常用作 8051 时间源的 11.0592 MHz 振荡器，定时器每 1/11059200*12 = 1.085 毫秒激活一次。

我们的定时器 ISR 应以 10 kHz 频率激活，或者每 100 毫秒激活一次，或者在每 100/1.085 = 92 个定时器滴答后激活一次。

我们将定时器 0 编程为以第二种模式运行，如下所示：

```cpp
TMOD = 0x02;
```

在这种模式下，我们将定时器的复位值存储在 TH0 寄存器中。ISR 由定时器溢出激活，这发生在定时器计数器达到最大值之后。第二种模式是 8 位模式，意味着最大值是 255。要使 ISR 每 92 个时钟周期激活一次，自动重载值应为 255-92 = 163，或者用十六进制表示为`0xa3`。

我们将自动重载值与初始定时器值一起存储在定时器寄存器中：

```cpp
TH0 = 0xa3; 
TL0 = 0x0;
```

定时器 0 被激活，如下所示：

```cpp
TR0 = 1;
```

然后，我们启用定时器中断：

```cpp
TR0 = 1;
```

最后，所有中断都被激活：

```cpp
EA = 1; 
```

从现在开始，我们的 ISR 每 100 微秒被调用一次，如下面的代码所示：

```cpp
P0_0 = !P0_0;
```

这会翻转`P0`寄存器的`0`位，从而在相应的输出引脚上产生 5 kHz 方波信号。

# 使用定时器 1 作为事件计数器来计算 1 Hz 脉冲

8051 定时器具有双重功能。当它们被时钟振荡器激活时，它们充当定时器。然而，它们也可以被外部引脚上的信号脉冲激活，即 P3.4（定时器 0）和 P3.5（定时器 1），充当计数器。

在这个示例中，我们将学习如何编程定时器 1，以便它计算 8051 处理器的 P3.5 引脚的激活次数。

# 如何做...

按照以下步骤完成此操作：

1.  打开 mcu8051ide。

1.  创建一个名为`Counters`的新项目。

1.  创建一个名为`generator.c`的新文件，并将以下代码片段放入其中。这将在每次定时器中断触发时递增一个计数器变量：

```cpp
#include<8051.h> 

volatile int counter = 0;
void timer1_ISR (void) __interrupt(3) 
{ 
  counter++;
} 

void main(void) 
{ 
  TMOD = 0x60;
  TH1 = 254; 
  TL1 = 254; 
  TR1 = 1;
  ET1 = 1;
  EA = 1; 
  while (1); // do nothing 
}
```

1.  选择工具|编译以构建代码。

1.  打开 Virtual HW 菜单，并选择 Simple Key...条目。将打开一个新窗口。

1.  在 Simple Keypad 窗口中，将端口 3 和位 5 分配给第一个键。然后，单击 ON 或 OFF 按钮以激活它：

![](img/5c45f07b-20cb-4009-93d0-312fa4abe748.png)

1.  选择模拟器|启动/关闭菜单项以激活模拟器。

1.  选择模拟器|动画以以动画模式运行程序，该模式在调试器窗口中显示对特殊寄存器的所有更改。

1.  切换到简单键盘窗口并单击第一个键。

# 工作原理...

在这个过程中，我们利用 8051 定时器的能力，使其作为计数器。我们以与普通定时器完全相同的方式定义中断服务例程。由于我们将定时器 1 用作计数器，我们使用中断线号`3`，如下所示：

```cpp
void timer1_ISR (void) __interrupt(3) 
```

中断例程的主体很简单。我们只递增`counter`变量。

现在，让我们确保 ISR 是由外部源而不是时钟振荡器激活的。为此，我们通过将`TMOD`特殊功能寄存器的 C/T 位设置为 1 来配置定时器 1：

```cpp
TMOD = 0x60;
```

同样的行配置定时器 1 以在 Mode 2 下运行- 8 位模式与自动重载。由于我们的目标是使中断例程在每次外部引脚激活时被调用，我们将自动重载和初始值设置为最大值`254`：

```cpp
TH1 = 254; 
TL1 = 254; 
```

接下来，我们启用定时器 1：

```cpp
 TR1 = 1;
```

然后，激活所有来自定时器 1 的中断，如下所示：

```cpp
 ET1 = 1;
 EA = 1;
```

之后，我们可以进入一个什么也不做的无限循环，因为所有的工作都是在中断服务例程中完成的：

```cpp
 while (1); // do nothing 
```

在这一点上，我们可以在模拟器中运行代码。但是，我们需要配置外部事件的来源。为此，我们利用 MCU8051IDE 支持的虚拟外部硬件组件之一-虚拟键盘。

我们配置其中一个键来激活 8051 的引脚 P3.5。当它在计数模式下使用时，该引脚被用作定时器 1 的源。

现在，我们运行代码。按下虚拟键会激活计数器。一旦计时器值溢出，我们的 ISR 就会被触发，递增`counter`变量。

# 还有更多...

在这个过程中，我们使用定时器 1 作为计数器。同样的方法也可以应用于计数器 0。在这种情况下，引脚 P3.4 应该被用作外部源。

# 串行接收和发送数据

8051 微控制器配备了内置的**通用异步收发器**（**UART**）端口，用于串行数据交换。

串行端口由名为**串行控制**（**SCON**）的**特殊功能寄存器**（**SFR**）控制。其位定义如下：

| **位** | **名称** | **目的** |
| --- | --- | --- |
| 0 | **RI**（**接收** **中断**的缩写） | 当一个字节完全接收时由 UART 设置 |
| 1 | **TI**（**传输** **中断**的缩写） | 当一个字节完全传输时由 UART 设置 |
| 2 | **RB8**（**接收** **位** **8**的缩写） | 在 9 位模式下存储接收数据的第九位。 |
| 3 | **TB8**（**传输位 8**的缩写） | 在 9 位模式下存储要传输的数据的第九位（见下文） |
| 4 | **REN**（**接收使能**的缩写） | 启用（1）或禁用（0）接收操作 |
| 5 | **SM2**（启用多处理器） | 为 9 位模式启用（1）或禁用（0）多处理器通信 |
| 6 | **SM1**（串行模式，高位） | 定义串行通信模式 |
| 7 | **SM0**（串行模式，低位） | 定义串行通信模式 |

8051 UART 支持四种串行通信模式，所有这些模式都由 SM1 和 SM0 位定义：

| **模式** | **SM0** | **SM1** | **描述** |
| --- | --- | --- | --- |
| 0 | 0 | 0 | 移位寄存器，固定波特率 |
| 1 | 0 | 1 | 8 位 UART，波特率由定时器 1 设置 |
| 2 | 1 | 0 | 9 位 UART，固定波特率 |
| 3 | 1 | 1 | 9 位 UART，波特率由定时器 1 设置 |

在这个过程中，我们将学习如何使用中断来实现使用可编程波特率的 8 位 UART 模式进行简单数据交换。

# 如何做...

按照以下步骤完成此过程：

1.  打开 mcu8051ide 并创建一个新项目。

1.  创建一个名为`serial.c`的新文件，并将以下代码片段复制到其中。这段代码将接收到的字节复制到`P0`输出寄存器中。这与 MCU 上的通用输入/输出引脚相关联：

```cpp
#include<8051.h>

void serial_isr() __interrupt(4) { 
    if(RI == 1) {
        P0 = SBUF;
        RI = 0;
    }
 }

void main() {
    SCON = 0x50;
    TMOD = 0x20;
    TH1 = 0xFD;
    TR1 = 1; 
    ES = 1;
    EA = 1;

    while(1);
 }
```

1.  选择工具 | 编译以构建代码。

1.  选择模拟器 | 启动/关闭菜单项以激活模拟器。

# 工作原理...

我们为中断线`4`定义了一个 ISR，用于串行端口事件触发：

```cpp
void serial_isr() __interrupt(4)
```

一旦接收到一个完整的字节并存储在**串行缓冲寄存器**（**SBUF**）中，中断例程就会被调用。我们的中断服务程序的实现只是将接收到的字节复制到输入/输出端口，即`P0`：

```cpp
P0 = SBUF;
```

然后，它重置 RI 标志以启用即将到来的字节的中断。

为了使中断按预期工作，我们需要配置串行端口和定时器。首先，配置串行端口如下：

```cpp
SCON = 0x50;
```

根据上表，这意味着**串行控制寄存器**（**SCON**）的 SM1 和 REN 位仅设置为 1，从而选择通信模式 1。这是一个由定时器 1 定义波特率的 8 位 UARS。然后，它启用接收器。

由于波特率由定时器 1 定义，下一步是配置定时器，如下所示：

```cpp
TMOD = 0x20;
```

上述代码配置定时器 1 使用模式 2，即 8 位自动重载模式。

将 0xFD 加载到 TH1 寄存器中，将波特率设置为 9600 bps。然后，我们启用定时器 1、串行中断和所有中断。

# 还有更多...

数据传输可以以类似的方式实现。如果您向 SBUF 特殊寄存器写入数据，8051 UART 将开始传输。完成后，将调用串行中断并将 TI 标志设置为 1。