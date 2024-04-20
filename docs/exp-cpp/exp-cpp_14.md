# 第十四章：网络和安全

网络编程变得越来越受欢迎。大多数计算机都连接到互联网，越来越多的应用程序现在依赖于它。从可能需要互联网连接的简单程序更新到依赖稳定互联网连接的应用程序，网络编程已经成为应用程序开发的必要部分。

直到最近的标准更新，C++语言才开始支持网络。网络支持已经推迟到了后续的标准，很可能要等到 C++23。然而，我们可以通过处理网络应用程序来为发布做好准备。我们还将讨论网络的标准扩展，并看看语言中支持网络会是什么样子。本章将集中讨论网络的主要原则和驱动设备之间通信的协议。设计网络应用程序是作为程序员技能的重要补充。

开发人员经常面临的一个主要问题是应用程序的安全性。无论是与正在处理的输入数据相关还是使用经过验证的模式和实践进行编码，应用程序的安全性必须是首要任务。对于网络应用程序来说尤为重要。在本章中，我们还将深入探讨 C++中安全编程的技术和最佳实践。

本章将涵盖以下主题：

+   计算机网络简介

+   C++中的套接字和套接字编程

+   设计网络应用程序

+   了解 C++程序中的安全问题

+   利用安全编程技术进行项目开发

# 技术要求

在本章的示例中，将使用 g++编译器以`-std=c++2a`选项进行编译。

您可以在[`github.com/PacktPublishing/Expert-CPP`](https://github.com/PacktPublishing/Expert-CPP)找到本章的源文件。

# 在 C++中发现网络编程

两台计算机通过网络进行交互。计算机使用特殊的硬件组件称为**网络适配器**或**网络接口控制器**连接到互联网。安装在计算机上的操作系统提供驱动程序以与网络适配器一起工作；也就是说，为了支持网络通信，计算机必须安装有支持网络堆栈的操作系统。通过堆栈，我们指的是数据在从一台计算机传输到另一台计算机时经历的一系列修改层。例如，在浏览器上打开网站会呈现通过网络收集的数据。该数据以一系列零和一接收，然后转换为对 Web 浏览器更易理解的形式。分层在网络中是至关重要的。如今的网络通信由符合我们将在此讨论的 OSI 模型的几个层组成。网络接口控制器是支持**开放系统互连**（**OSI**）模型的物理和数据链路层的硬件组件。

OSI 模型旨在标准化各种设备之间的通信功能。设备在结构和组织上有所不同。这涉及硬件和软件。例如，使用英特尔 CPU 运行 Android OS 的智能手机与运行 macOS Catalina 的 MacBook 电脑是不同的。不同之处不在于上述产品背后的名称和公司，而在于硬件和软件的结构和组织。为了消除网络通信中的差异，OSI 模型提出了一套标准化的协议和互联功能。我们之前提到的层如下：

+   应用层

+   表示层

+   会话层

+   传输层

+   网络层

+   数据链路层

+   物理层

更简化的模型包括以下四个层：

+   **应用程序**：处理特定应用程序的详细信息。

+   **传输**：这提供了两个主机之间的数据传输。

+   **网络**：这处理网络中数据包的传输。

+   **链路**：这包括操作系统中的设备驱动程序，以及计算机内的网络适配器。

链路（或数据链路）层包括操作系统中的设备驱动程序，以及计算机中的网络适配器。

为了理解这些层，让我们假设您正在使用桌面应用程序进行消息传递，比如*Skype*或*Telegram*。当您输入一条消息并点击发送按钮时，消息会通过网络传输到其目的地。在这种情况下，假设您正在向安装了相同应用程序的朋友发送文本消息。从高层次的角度来看，这可能看起来很简单，但这个过程是复杂的，即使是最简单的消息在到达目的地之前也经历了许多转换。首先，当您点击发送按钮时，文本消息会被转换为二进制形式。网络适配器使用二进制。它的基本功能是通过介质发送和接收二进制数据。除了实际发送到网络上的数据之外，网络适配器还应该知道数据的目的地地址。目的地地址是附加到用户数据的许多属性之一。通过用户数据，我们指的是您输入并发送给朋友的文本。目的地地址是您朋友计算机的唯一地址。输入的文本与目的地地址和其他必要信息一起打包，以便发送到目标位置。您朋友的计算机（包括网络适配器、操作系统和消息应用程序）接收并解包数据。然后消息应用程序会在屏幕上显示该数据包中的文本。

几乎在本章开头提到的每个 OSI 层都会向通过网络发送的数据添加其特定的标头。以下图表描述了应用层数据在移动到目的地之前如何叠加标头：

![](img/a757ca62-f6cd-41ca-a522-d59b4bb81220.png)

OSI 模型

看一下前面图表中的第一行（**应用层**）。**数据**是您在消息应用程序中输入的文本，以便将其发送给您的朋友。在每一层，一直到**物理层**，数据都会被打包，并附加有 OSI 模型每一层特定的标头。另一边的计算机接收并检索打包的数据。在每一层，它会移除该层特定的标头，并将其余的数据包移动到下一层。最终，数据到达您朋友的消息应用程序。

作为程序员，我们主要关注编写能够在网络上发送和接收数据的应用程序，而不深入了解各层的细节。然而，我们需要对如何在更高层次上使用标头增强数据有一定的了解。让我们学习一下网络应用程序在实践中是如何工作的。

# 网络应用程序的内部工作

安装在设备上的网络应用程序通过网络与其他设备上安装的应用程序进行通信。在本章中，我们将讨论通过互联网一起工作的应用程序。可以在以下图表中看到这种通信的高层概述：

![](img/3e256446-4904-4374-be81-67b37eae5d24.png)

在通信的最低层是物理层，它通过介质传输数据位。在这种情况下，介质是网络电缆（也考虑 Wi-Fi 通信）。用户应用程序抽象了网络通信的较低层。程序员所需的一切都由操作系统提供。操作系统实现了网络通信的低级细节，比如**传输控制协议**/**互联网协议**（**TCP**/**IP**）套件。

每当应用程序需要访问网络，无论是局域网还是互联网，它都会请求操作系统提供一个访问点。操作系统通过利用网络适配器和特定软件与硬件通信来管理提供网络的网关。

这更详细的说明如下：

![](img/000a23f6-9ba9-4102-8fe3-88292e377065.png)

操作系统提供了一个用于处理其网络子系统的 API。程序员应该关心的主要抽象是套接字。我们可以将套接字视为通过网络适配器发送其内容的文件。套接字是连接两台计算机的访问点，如下图所示：

![](img/26dbd171-672b-494d-b565-81f297597189.png)

从程序员的角度来看，套接字是一个允许我们在应用程序中通过网络实现数据传输的结构。套接字是一个连接点，可以发送或接收数据；也就是说，应用程序也可以通过套接字接收数据。操作系统在请求时为应用程序提供套接字。一个应用程序可以拥有多个套接字。客户端应用程序在客户端-服务器架构中通常使用单个套接字。现在，让我们详细了解套接字编程。

# 使用套接字编程网络应用

正如我们之前提到的，套接字是对网络通信的抽象。我们将它们视为常规文件 - 所有写入套接字的内容都由操作系统通过网络发送到目的地。通过网络接收到的所有内容都会被操作系统写入套接字。这样，操作系统为网络应用程序提供了双向通信。

假设我们运行两个不同的与网络相关的应用程序。例如，我们打开一个网页浏览器来浏览网页，并使用一个消息应用（如 Skype）与朋友聊天。网页浏览器代表了客户端-服务器网络架构中的客户端应用程序。在这种情况下，服务器是响应所请求数据的计算机。例如，我们在网页浏览器的地址栏中输入一个地址，然后在屏幕上看到生成的网页。每当我们访问一个网站时，网页浏览器都会从操作系统请求一个套接字。在编码方面，网页浏览器使用操作系统提供的 API 创建一个套接字。我们可以用更具体的前缀来描述套接字：客户端套接字。为了让服务器处理客户端请求，运行 Web 服务器的计算机必须监听传入的连接；也就是说，服务器应用程序创建一个用于监听连接的服务器套接字。

每当客户端和服务器之间建立连接时，数据通信就可以进行。下图描述了网页浏览器对**facebook.com**的请求：

![](img/025167ab-52c6-40b9-a864-ee63f96cdb67.png)

请注意前图中的数字组。这被称为**Internet Protocol**（**IP**）**地址**。IP 地址是我们需要的位置，以便将数据传输到设备。有数十亿台设备连接到互联网。为了对它们进行唯一区分，每个设备都会暴露一个代表其地址的唯一数字值。使用 IP 协议建立连接，这就是为什么我们称其为 IP 地址。IP 地址由四组 1 字节长度的数字组成。它的点分十进制表示形式为 X.X.X.X，其中 X 是 1 字节数字。每个位置的值范围从 0 到 255。更具体地说，这是一个版本 4 的 IP 地址。现代系统使用版本 6 地址，这是数字和字母的组合，提供了更广泛的可用地址值范围。

创建套接字时，我们将本地计算机的 IP 地址分配给它；也就是说，我们将套接字绑定到该地址。当使用套接字向网络中的另一设备发送数据时，我们应该设置其目标地址。目标地址由该设备上的另一个套接字持有。为了在两个设备之间创建连接，我们使用两个套接字。可能会出现一个合理的问题——如果设备上运行了多个应用程序怎么办？如果我们运行了多个应用程序，每个应用程序都为自己创建了一个套接字怎么办？哪一个应该接收传入的数据？

要回答这些问题，请仔细查看前面的图表。您应该在 IP 地址末尾的冒号后看到一个数字。这被称为**端口号**。端口号是一个 2 字节长度的数字，由操作系统分配给套接字。由于 2 字节长度限制，操作系统无法为套接字分配超过 65,536 个唯一的端口号；也就是说，您不能有超过 65,536 个同时运行的进程或线程通过网络进行通信（但是有方法可以重用套接字）。除此之外，还有一些端口号专门为特定应用程序保留。这些端口称为众所周知的端口，范围从 0 到 1023。它们保留用于特权服务。例如，HTTP 服务器的端口号是 80。这并不意味着它不能使用其他端口。

让我们学习如何在 C++中创建套接字。我们将设计一个封装**便携操作系统接口**（**POSIX**）套接字的包装类，也称为**伯克利**或**BSD**套接字。它具有用于套接字编程的标准函数集。网络编程的 C++扩展将是语言的巨大补充。工作草案包含有关网络接口的信息。我们将在本章后面讨论这一点。在那之前，让我们尝试为现有和低级库创建我们自己的网络包装器。当我们使用 POSIX 套接字时，我们依赖于操作系统的 API。操作系统提供了一个 API，表示用于创建套接字、发送和接收数据等的函数和对象。

POSIX 将套接字表示为文件描述符。我们几乎可以像处理常规文件一样使用它。文件描述符遵循 UNIX 哲学，提供了一个通用的数据输入/输出接口。以下代码使用`socket()`函数（在`<sys/socket.h>`头文件中定义）创建套接字：

```cpp
int s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
```

`socket()`函数的声明如下：

```cpp
int socket(int domain, int type, int protocol);
```

因此，`AF_INET`、`SOCK_STREAM`和`IPPROTO_TCP`都是数值。域参数指定套接字的协议族。我们使用`AF_INET`来指定 IPv4 协议。对于 IPv6，我们使用`AF_INET6`。第二个参数指定套接字的类型，即它是面向流的还是数据报的套接字。对于每种特定类型，最后一个参数应相应地指定。在前面的示例中，我们使用`IPPROTO_TCP`指定了`SOCK_STREAM`。**传输控制协议**（**TCP**）代表可靠的面向流的协议。这就是为什么我们将类型参数设置为`SOCK_STREAM`的原因。在实现简单的套接字应用程序之前，让我们更多地了解网络协议。

# 网络协议

网络协议是一组规则和数据格式，用于定义应用程序之间的互联。例如，Web 浏览器和 Web 服务器通过**超文本传输协议**（**HTTP**）进行通信。HTTP 更像是一组规则，而不是传输协议。传输协议是每个网络通信的基础。传输协议的一个例子是 TCP。当我们提到 TCP/IP 套件时，我们指的是 TCP 在 IP 上的实现。我们可以将**互联网协议**（**IP**）视为互联网通信的核心。

它提供主机到主机的路由和寻址。我们通过互联网发送或接收的所有内容都被打包成*IP 数据包*。以下是 IPv4 数据包的外观：

![](img/947b4431-3e7b-4324-a92c-42a6140bcb04.png)

IP 头部重量为 20 字节。它结合了从源地址到目的地址传递数据包所需的标志和选项。在 IP 协议领域，我们通常称数据包为数据报。每个层都有其特定的数据包术语。更加细心的专家会谈论将 TCP 段封装到 IP 数据报中。将它们称为数据包是完全可以的*.*

每个更高级别的协议都会向通过网络发送和接收的数据附加元信息；例如，TCP 数据封装在 IP 数据报中。除了这些元信息，协议还定义了应该执行的底层规则和操作，以完成两个或多个设备之间的数据传输。

您可以在称为**请求评论**（**RFCs**）的特定文档中找到更详细的信息。例如，RFC 791 描述了互联网协议，而 RFC 793 描述了传输控制协议。

许多流行的应用程序 - 文件传输、电子邮件、网络等 - 使用 TCP 作为它们的主要传输协议。例如，HTTP 协议定义了从客户端到服务器和反之亦然传输的消息格式。实际的传输是使用传输协议进行的 - 在这种情况下是 TCP。但是，HTTP 标准并不限制 TCP 成为唯一的传输协议。

下图说明了在将数据传递到较低级别之前，TCP 头被附加到数据中：

![](img/84ddead5-264a-4848-8552-5bc1e8c1986f.png)

注意源端口号和目标端口号。这些是在操作系统中区分运行进程的唯一标识符。还要看一下序列号和确认号。它们是 TCP 特有的，用于传输可靠性。

在实践中，TCP 由于以下特点而被使用：

+   丢失数据的重传

+   按顺序传递

+   数据完整性

+   拥塞控制和避免

**IP**（即互联网协议）是不可靠的。它不关心丢失的数据包，这就是为什么 TCP 处理丢失数据包的重传。它使用唯一标识符标记每个数据包，应该由传输的另一端确认。如果发送方没有收到数据包的**确认码**（**ACK**），协议将重新发送数据包（有限次数）。正确接收数据包也非常重要。TCP 重新排序接收到的数据包以正确表示排序信息。这就是为什么在线听音乐时，我们不会在歌曲的开头听到结尾。

数据包的重传可能会导致另一个问题，即**网络拥塞**。当节点无法快速发送数据包时，就会发生这种情况。数据包会被卡住一段时间，不必要的重传会增加它们的数量。TCP 的各种实现采用了拥塞避免算法。

它维护一个拥塞窗口 - 一个确定可以发送的数据量的因素。使用慢启动机制，TCP 在初始化连接后缓慢增加拥塞窗口。尽管该协议在相应的**请求评论**（**RFC**）中有描述，但在操作系统中实现的机制有很多不同。

在另一边是**用户数据报协议**（**UDP**）。这两者之间的主要区别是 TCP 是可靠的。这意味着在丢失网络数据包的情况下，它会重新发送相同的数据包，直到它到达指定的目的地。由于其可靠性，通过 TCP 进行的数据传输被认为比使用 UDP 需要更长的时间。UDP 不能保证我们可以正确地传递数据包而且没有丢失。相反，开发人员应该负责重新发送、检查和验证数据传输。需要快速通信的应用程序倾向于依赖 UDP。例如，视频通话应用程序或在线游戏使用 UDP 因为它的速度。即使在传输过程中丢失了几个数据包，也不会影响用户体验。在玩游戏或进行视频聊天时，最好出现小故障，而不是等待下一帧游戏或视频。

TCP 比 UDP 慢的主要原因之一是 TCP 连接初始化过程中步骤较多。下图显示了 TCP 连接建立的过程，也称为三次握手：

![](img/297eb191-d103-4058-b238-d114d4404673.png)

客户端在向服务器发送`SYN`数据包时选择一个随机数。服务器将该随机数加一，选择另一个随机数，并回复一个`SYN-ACK`数据包。客户端将从服务器接收的两个数字都加一，并通过向服务器发送最后一个`ACK`完成握手。成功完成三次握手后，客户端和服务器可以相互传输数据包。这种连接建立过程适用于每个 TCP 连接。握手的细节对网络应用程序的开发者是隐藏的。我们创建套接字并开始监听传入的连接。

注意两种端点之间的区别。其中之一是客户端。在实现网络应用程序时，我们应该明确区分客户端和服务器，因为它们有不同的实现。这也与套接字的类型有关。创建服务器套接字时，我们使其监听传入的连接，而客户端不监听 - 它发出请求。下图描述了客户端和服务器的某些函数及其调用顺序：

![](img/abc95252-99b1-40e7-8518-218dab92e194.png)

在代码中创建套接字时，我们指定协议和套接字的类型。当我们需要两个端点之间的可靠连接时，我们选择 TCP。有趣的是，我们可以使用 TCP 等传输协议来构建自己的协议。假设我们定义了一种特殊的文档格式来发送和接收以使通信有效。例如，每个文档应该以单词 PACKT 开头。HTTP 也是这样工作的。它使用 TCP 进行传输，并定义了其上的通信格式。在 UDP 的情况下，我们还应该为通信设计和实现可靠性策略。前面的图表显示了 TCP 如何在两个端点之间建立连接。客户端向服务器发送`SYN`请求。服务器用`SYN-ACK`响应回答，让客户端知道可以继续握手。最后，客户端向服务器发送`ACK`，表示连接已正式建立。他们可以随意进行通信。

**同步**（**SYN**）和**确认**（ACK）是协议定义的术语，在网络编程中变得常见。

UDP 不是这样工作的。它将数据发送到目的地，而不必担心是否建立了连接。如果您使用 UDP 但需要一些可靠性，您应该自己来实现；例如，通过检查一部分数据是否到达了目的地。为了检查它，您可以等待目的地用自定义定义的`ACK`数据包进行回复。大多数可靠性导向的实现可能会重复已经存在的协议，如 TCP。然而，有许多情况下您不需要它们；例如，您不需要拥塞避免，因为您不需要发送相同的数据包两次。

在上一章中，我们设计了一个策略游戏。假设游戏是在线的，你正在与一个真正的对手而不是一个自动化的敌对玩家进行游戏。游戏的每一帧都是基于通过网络接收的数据进行渲染的。如果我们在使数据传输可靠、增加数据完整性以及确保没有任何数据包丢失方面付出了一些努力，可能会因为玩家的不同步而影响用户体验。这种情况适合使用 UDP。我们可以实现数据传输而不需要重传策略，以便提高游戏的速度。当然，使用 UDP 并不强迫我们避免可靠性。在同样的情况下，我们可能需要确保数据包被玩家成功接收。例如，当玩家投降时，我们应该确保对手收到消息。因此，我们可以根据数据包的优先级进行有条件的可靠性。UDP 在网络应用程序中提供了灵活性和速度。

让我们来看一个 TCP 服务器应用程序的实现。

# 设计网络应用程序

使用一个需要网络连接的小子系统来设计应用程序的方法与完全与网络相关的应用程序不同。后者的一个例子可能是用于文件存储和同步的客户端-服务器应用程序（如 Dropbox）。它由服务器和客户端组成，其中客户端安装为桌面或移动应用程序，也可以用作文件资源管理器。由 Dropbox 控制的系统中文件的每次更新都将立即与服务器同步。这样，您将始终在云中拥有您的文件，并可以在任何地方通过互联网连接访问它们。

我们将设计一个类似的简化的服务器应用程序，用于文件存储和操作。服务器的主要任务如下：

+   从客户端应用程序接收文件

+   在指定的位置存储文件

+   根据请求向客户端发送文件

参考第十章，*设计面向世界的应用程序*，我们可以继续进行以下应用程序的顶层设计：

![](img/45a77bf5-8c18-40ec-9537-129401650f37.png)

在上图中的每个矩形代表一个类或一组类，涉及特定的任务。例如，**存储管理器**处理与存储和检索文件相关的所有事务。在这一点上，它使用文件、位置、数据库等类并不那么关心。

**客户端管理器**是一个类或一组类，用于处理与客户端（指客户端应用程序）相关的所有事务，包括认证或授权客户端，与客户端保持稳定的连接，从客户端接收文件，向客户端发送文件等。

在本章中，我们特别强调了**网络**作为一个感兴趣的实体。所有与网络连接相关的事情，以及与客户端的数据传输，都是通过**网络**来处理的。现在，让我们看看我们可以使用什么功能来设计网络类（我们将称之为网络管理器以方便起见）。

# 使用 POSIX 套接字

正如我们之前提到的，诸如`socket()`、`bind()`和`accept()`之类的函数在大多数 Unix 系统中默认支持。之前，我们包含了`<sys/socket.h>`文件。除此之外，我们还需要几个其他头文件。让我们实现经典的 TCP 服务器示例，并将其封装在 Networking 模块中，用于我们的文件传输应用服务器。

正如我们之前提到的，服务器端开发在套接字的类型和行为方面与客户端开发不同。虽然两边都使用套接字，但服务器端套接字不断监听传入的连接，而客户端套接字则与服务器建立连接。为了使服务器套接字等待连接，我们创建一个套接字并将其绑定到服务器 IP 地址和客户端将尝试连接的端口号。以下 C 代码表示了 TCP 服务器套接字的创建和绑定：

```cpp
int s = socket(AF_INET, SOCK_STREAM, 0);

struct sockaddr_in server;
server.sin_family = AF_INET;
server.sin_port = htons(port);
server.sin_addr.s_addr = INADDR_ANY;

bind(s, (struct sockaddr*)&server, sizeof(server));
```

第一个调用创建了一个套接字。第三个参数设置为 0，这意味着将根据套接字的类型选择默认协议。类型作为第二个参数传递，`SOCK_STREAM`，这将使协议值默认等于`IPPROTO_TCP`。`bind()`函数将套接字绑定到指定的 IP 地址和端口号。我们在`sockaddr_in`结构中指定了它们，该结构将网络地址相关的细节组合在一起。

虽然我们在前面的代码中跳过了这一点，但你应该考虑检查对`socket()`和`bind()`函数（以及 POSIX 套接字中的其他函数）的调用是否出现错误。几乎所有这些函数在出现错误时都会返回`-1`。

另外，注意`htons()`函数。它负责将其参数转换为网络字节顺序。问题隐藏在计算机设计的方式中。一些机器（例如 Intel 微处理器）使用**小端**字节顺序，而其他一些使用**大端**顺序。**小端**顺序将最不重要的字节放在最前面。**大端**顺序将最重要的字节放在最前面。以下图表显示了两者之间的区别：

![](img/16478fe6-bab4-4ef2-9d55-9473870f4425.png)

网络字节顺序是与特定机器架构无关的约定。`htons()`函数将提供的端口号从主机字节顺序（**小端**或**大端**）转换为网络字节顺序（与机器无关）。

就是这样——套接字已经准备好了。现在，我们应该指定它准备好接收传入的连接。为了指定这一点，我们使用`listen()`函数：

```cpp
listen(s, 5);
```

顾名思义，它用于监听传入的连接。传递给`listen()`函数的第二个参数指定了服务器在丢弃新的传入请求之前将排队的连接数。在前面的代码中，我们指定了`5`作为最大数。在高负载环境中，我们会增加这个数字。最大数由`<sys/socket.h>`头文件中定义的`SOMAXCONN`常量指定。

backlog 数（`listen()`函数的第二个参数）的选择基于以下因素：

+   如果连接请求的速率在短时间内很高，那么 backlog 数应该有一个较大的值。

+   服务器处理传入连接的持续时间。时间越短，backlog 值就越小。

当连接初始化发生时，我们可以选择放弃它或接受它并继续处理连接。这就是为什么我们在下面的代码段中使用`accept()`函数：

```cpp
struct sockaddr_in client;
int addrlen;
int new_socket = accept(s, (struct sockaddr_in*)&client, &addrlen);
// use the new_socket
```

在前面的代码中需要考虑的两件事如下：

+   首先，接受的套接字连接信息被写入客户端的`sockaddr_in`结构中。我们可以从该结构中收集关于客户端的所有必要信息。

+   接下来，要注意`accept()`函数的返回值。它是一个新的套接字，用于处理来自特定客户端的请求。下一次调用`accept()`函数将返回另一个值，代表另一个具有独立连接的客户端。我们应该正确处理这一点，因为`accept()`调用是阻塞的；也就是说，它等待新的连接请求。我们将修改前面的代码，以便在单独的线程中处理多个连接。

在前面的代码中带有注释的最后一行说明`new_socket`可以用于接收或发送数据给客户端。让我们看看如何实现这一点，然后开始设计我们的`Networking`类。要读取套接字接收的数据，我们需要使用`recv()`函数，如下所示：

```cpp
char buffer[BUFFER_MAX_SIZE]; // define BUFFER_MAX_SIZE based on the specifics of the server
recv(new_socket, buffer, sizeof(buffer), 0);
// now the buffer contains received data
```

`recv()`函数接受一个`char*`缓冲区来写入数据。它在`sizeof(buffer)`处停止写入。函数的最后一个参数是我们可以设置用于读取的附加标志。您应该考虑多次调用该函数以读取大于`BUFFER_MAX_SIZE`的数据。

最后，要通过套接字发送数据，我们调用`send()`函数，如下所示：

```cpp
char msg[] = "From server with love";
send(new_socket, msg, sizeof(msg), 0);
```

通过这样，我们几乎涵盖了实现服务器应用程序所需的所有函数。现在，让我们将它们封装在一个 C++类中，并加入多线程，以便我们可以并发处理客户端请求。

# 实现一个 POSIX 套接字包装类

让我们设计和实现一个类，它将作为基于网络的应用程序的起点。该类的主要接口如下所示：

```cpp
class Networking
{
public:
  void start_server();

public:
  std::shared_ptr<Networking> get_instance();
  void remove_instance();

private:
  Networking();
  ~Networking();

private:
  int socket_;
  sockaddr_in server_;
  std::vector<sockaddr_in> clients_;

private:
  static std::shared_ptr<Networking> instance_ = nullptr;
  static int MAX_QUEUED_CONNECTIONS = 1;
};
```

`Networking`类作为单例是很自然的，因为我们希望有一个单一的实例来监听传入的连接。同时，拥有多个对象，每个对象代表与客户端的单独连接，也是很重要的。让我们逐渐改进类的设计。之前，我们看到在服务器套接字监听并接受连接请求之后，将创建每个新的客户端套接字。

在那之后，我们可以通过新的客户端套接字发送或接收数据。服务器的操作方式与下图中所示的类似：

![](img/1d1c32ac-afd0-44be-a927-e1e391ec20c8.png)

也就是说，在接受每个传入的连接之后，我们将有一个单独的套接字用于连接。我们将它们存储在`Networking`类的`clients_`向量中。因此，我们可以在一个函数中编写创建服务器套接字、监听和接受新连接的主要逻辑，如果需要的话，可以并发工作。`start_server()`函数作为服务器监听传入连接的起点。以下代码块说明了这一点：

```cpp
void Networking::start_server()
{
  socket_ = socket(AF_INET, SOCK_STREAM, 0);
  // the following check is the only one in this code snippet
  // we skipped checking results of other functions for brevity, 
  // you shouldn't omit them in your code
  if (socket_ < 0) { 
    throw std::exception("Cannot create a socket");
  }

  struct sockaddr_in server;
  server.sin_family = AF_INET;
  server.sin_port = htons(port);
  server.sin_addr.s_addr = INADDR_ANY;

  bind(s, (struct sockaddr*)&server, sizeof(server));
  listen(s, MAX_QUEUED_CONNECTIONS);
 // the accept() should be here
}
```

现在，我们停在了应该接受传入连接的地方（请参阅前面的代码片段中的注释）。我们在这里有两种选择（实际上，不止两种选择，但我们只讨论其中的两种）。我们可以直接将`accept()`调用放入`start_server()`函数中，或者我们可以实现一个单独的函数，`Networking`类用户在适当时将调用它。

为项目中的每个错误情况拥有特定的异常类并不是一个坏的做法。在考虑自定义异常时，前面的代码可能会被重写。您可以将其作为一个作业项目来完成。

其中一个选择在`start_server()`函数中有`accept()`函数，它将每个新连接推送到`clients_`向量中，如下所示：

```cpp
void Networking::start_server()
{
  // code omitted for brevity (see in the previous snippet)
  while (true) {
    sockaddr_in client;
    int addrlen;
    int new_socket = accept(socket_, (sockaddr_in*)&client, &addrlen);
    clients_.push_back(client);
  }
}
```

是的，我们使用了一个无限循环。这听起来可能很糟糕，但只要服务器在运行，它就必须接受新的连接。然而，我们都知道无限循环会阻塞代码的执行；也就是说，它永远不会离开`start_server()`函数。我们将我们的网络应用程序介绍为一个至少有三个组件的项目：客户端管理器、存储管理器，以及我们正在设计的`Networking`类。

一个组件的执行不应以不好的方式影响其他组件；也就是说，我们可以使用线程使一些组件在后台运行。在线程的上下文中运行的`start_server()`函数是一个不错的解决方案，尽管我们现在应该关心我们在第八章中讨论的同步问题，即并发和多线程。

还要注意前面循环的不完整性。在接受连接后，它将客户端数据推送到`clients_`向量中。我们应该考虑使用另一个结构，因为我们还需要存储套接字描述符，以及客户端。我们可以使用`std::undordered_map`将套接字描述符映射到客户端连接信息，但简单的`std::pair`或`std::tuple`也可以。

然而，让我们更进一步，创建一个表示客户端连接的自定义对象，如下所示：

```cpp
class Client
{
public:
  // public accessors

private:
  int socket_;
  sockaddr_in connection_info_;
};
```

我们将修改`Networking`类，使其存储`Client`对象的向量：

```cpp
std::vector<Client> clients_;
```

现在，我们可以改变设计方法，使`Client`对象负责发送和接收数据：

```cpp
class Client
{
public:
  void send(const std::string& data) {
    // wraps the call to POSIX send() 
  }
  std::string receive() {
    // wraps the call to POSIX recv()
  }

  // code omitted for brevity 
};
```

更好的是，我们可以将`std::thread`对象附加到`Client`类，这样每个对象都可以在单独的线程中处理数据传输。然而，你应该小心不要使系统陷入饥饿状态。传入连接的数量可能会急剧增加，服务器应用程序将会变得卡住。在下一节中，当我们讨论安全问题时，我们将讨论这种情况。建议您利用线程池，这将帮助我们重用线程并控制程序中运行的线程数量。

类的最终设计取决于我们接收和发送给客户端的数据类型。至少有两种不同的方法。其中一种是连接到客户端，接收必要的数据，然后关闭连接。第二种方法是实现客户端和服务器之间通信的协议。虽然听起来复杂，但协议可能很简单。

这也是可扩展的，使应用程序更加健壮，因为您可以在项目发展过程中支持更多功能。在下一节中，当我们讨论如何保护网络服务器应用程序时，我们将回到设计用于验证客户端请求的协议。

# 保护 C++代码

与许多其他语言相比，C++在安全编码方面稍微难以掌握。有许多指南提供了关于如何避免 C++程序中的安全风险的建议。我们在第一章中讨论的最受欢迎的问题之一是使用预处理器宏。我们使用的例子有以下宏：

```cpp
#define DOUBLE_IT(arg) (arg * arg)
```

不正确使用这个宏会导致难以发现的逻辑错误。在下面的代码中，程序员期望在屏幕上打印`16`：

```cpp
int res = DOUBLE_IT(3 + 1);
std::cout << res << std::endl;
```

输出是`7`。这里的问题在于`arg`参数周围缺少括号；也就是说，前面的宏应该重写如下：

```cpp
#define DOUBLE_IT(arg) ((arg) * (arg))
```

尽管这个例子很受欢迎，我们强烈建议尽量避免使用宏。C++提供了许多可以在编译时处理的构造，比如`constexpr`、`consteval`和`constinit` - 即使语句也有`constexpr`的替代方案。如果您需要在代码中进行编译时处理，请使用它们。当然，还有模块，这是语言中期待已久的补充。您应该更喜欢使用模块，而不是使用`#include`和无处不在的包含保护：

```cpp
module my_module;
export int test;

// instead of

#ifndef MY_HEADER_H
#define MY_HEADER_H
int test
#endif 
```

这不仅更安全，而且更高效，因为模块只处理一次（我们可以将它们视为预编译头）。

虽然我们不希望您对安全问题变得偏执，但您几乎应该在任何地方小心。通过学习语言的怪癖和奇特之处，您将避免大部分这些问题。此外，一个好的做法是使用替换或修复以前版本的缺点的最新功能。例如，考虑以下`create_array()`函数：

```cpp
// Don't return pointers or references to local variables
double* create_array()
{
  double arr[10] = {0.0};
  return arr;
}
```

`create_array()` 函数的调用者因为`arr`具有自动存储期而留下了指向不存在数组的指针。如果需要，我们可以用更好的替代方案来替换前面的代码：

```cpp
#include <array>

std::array<double> create_array()
{
  std::array<double> arr;
  return arr;
}
```

字符串被视为字符数组，是许多缓冲区溢出问题的原因。其中最常见的问题之一是在忽略其大小的情况下向字符串缓冲区写入数据。在这方面，`std::string`类是 C 字符串的一个更安全的替代方案。然而，在支持旧代码时，您在使用`strcpy()`等函数时应该小心，就像以下示例中所示：

```cpp
#include <cstdio>
#include <cstring>

int main()
{
  char small_buffer[4];
  const char* long_text = "This text is long enough to overflow small buffers!";
 strcpy(small_buffer, long_text);
}
```

鉴于法律上，`small_buffer`应该在末尾有一个空终结符，它只能处理`long_text`字符串的前三个字符。然而，在调用`strcpy()`后发生了以下情况：

![](img/2e5109ce-bee4-4c01-b755-126f953aacc8.png)

在实现网络应用程序时，您应该更加小心。大部分来自客户端连接的数据应该得到适当处理，缓冲区溢出并不罕见。让我们学习如何使网络应用程序更加安全。

# 保护网络应用程序

在本书的前一节中，我们设计了一个使用套接字连接接收客户端数据的网络应用程序。除了大部分渗入系统的病毒来自外部世界这一事实之外，网络应用程序有这种自然倾向，即向互联网上的各种威胁打开计算机。首先，每当您运行一个网络应用程序时，系统中就存在一个开放的端口。知道您的应用程序正在监听的确切端口的人可以通过伪造协议数据侵入。我们将主要讨论网络应用程序的服务器端；然而，这里的一些主题也适用于客户端应用程序。

你应该做的第一件事之一是加入客户端授权和认证。这两个术语很容易混淆。小心不要将它们互换使用；它们是不同的：

+   **认证**是验证客户端访问的过程。这意味着并非每个传入的连接请求都会立即得到服务。在与客户端传输数据之前，服务器应用程序必须确保客户端是已知的客户端。几乎与我们通过输入电子邮件和密码访问社交网络平台的方式相同，客户端的认证定义了客户端是否有权访问系统。

+   **授权**，另一方面，定义了客户端在系统中可以做什么。这是一组权限，提供给特定的客户端。例如，我们在前一节讨论的客户端应用程序能够上传文件到系统中。迟早，您可能希望加入付费订阅，并为付费客户提供更广泛的功能；例如，允许他们创建文件夹来组织他们的文件。因此，当客户端请求创建文件夹时，我们可能希望授权请求以发现客户端是否有权这样做。

当客户端应用程序与服务器建立连接时，服务器获得的只是连接详细信息（IP 地址，端口号）。为了让服务器知道客户端应用程序背后的是谁（实际用户），客户端应用程序发送用户的凭据。通常，这个过程涉及向用户发送一个唯一标识符（如用户名或电子邮件地址）和密码以访问系统。然后，服务器会检查这些凭据与其数据库，并验证是否应该允许客户端访问。客户端和服务器之间的这种通信形式可能是简单的文本传输或格式化对象传输。

例如，服务器定义的协议可能要求客户端以以下形式发送**JavaScript 对象表示**（**JSON**）文档：

```cpp
{
  "email": "myemail@example.org",
  "password": "notSoSIMPLEp4s8"
}
```

服务器的响应允许客户端进一步进行，或者更新其用户界面以让用户知道操作的结果。在使用任何网络应用程序或网络应用程序时，您可能遇到了几种情况。例如，错误输入的密码可能导致服务器返回“无效的用户名或密码”错误。

除了这一必要的第一步之外，验证来自客户端应用程序的每一条数据都是明智的。如果检查电子邮件字段的大小，就可以很容易地避免缓冲区溢出。例如，当客户端应用程序故意试图破坏系统时，可能会发送一个 JSON 对象，其中的字段具有非常大的值。这个检查是服务器的责任。预防安全漏洞始于数据验证。

另一种安全攻击形式是从单个或多个客户端每秒发出过多的请求。例如，一个客户端应用程序在 1 秒内发出数百个身份验证请求，导致服务器密集处理这些请求，并浪费资源试图为它们提供服务。最好检查客户端请求的速率，例如，将其限制为每秒一个请求。

这些形式的攻击（有意或无意的）被称为**拒绝服务**（**DOS**）攻击。DOS 攻击的更高级版本采取了从多个客户端向服务器发出大量请求的形式。这种形式被称为**分布式 DOS**（**DDOS**）攻击。一个简单的方法可能是黑名单 IP 地址，这些 IP 地址试图通过每秒发出多个请求来使系统崩溃。作为网络应用程序的程序员，在开发应用程序时，您应该考虑本书范围之外的所有这些问题以及其他许多问题。

# 总结

在本章中，我们介绍了在 C++中设计网络应用程序。从其第一个版本开始，C++一直缺乏对网络的内置支持。C++23 标准计划最终在语言中引入对网络的支持。

我们首先介绍了网络的基础知识。完全理解网络需要很长时间，但在实现与网络有关的任何应用程序之前，每个程序员都必须了解一些基本概念。这些基本概念包括 OSI 模型中的分层和不同类型的传输协议，如 TCP 和 UDP。了解 TCP 和 UDP 之间的区别对于任何程序员都是必要的。正如我们所学到的，TCP 在套接字之间建立可靠的连接，而套接字是开发网络应用程序时程序员遇到的下一个东西。这些是两个应用程序实例的连接点。每当我们需要通过网络发送或接收数据时，我们应该定义一个套接字，并且几乎可以像处理常规文件一样处理它。

我们在应用程序开发中使用的所有抽象和概念都由操作系统处理，并最终由网络适配器处理。这是一种能够通过网络介质发送数据的设备。从介质接收数据并不能保证安全。网络适配器接收来自介质的任何东西。为了确保我们正确处理传入数据，我们还应该注意应用程序安全性。本章的最后一节是关于编写安全代码和验证输入，以确保程序不会受到伤害。保护程序是确保程序质量的良好步骤。开发程序的最佳方法之一是彻底测试它们。您可能还记得，在第十章中，*设计面向世界的应用程序*，我们讨论了软件开发步骤，并解释了一旦编码阶段完成，测试程序是最重要的步骤之一。测试后，您很可能会发现许多错误。其中一些错误很难重现和修复，这就是调试发挥作用的地方。

下一章是关于以正确的方式测试和调试您的程序。

# 问题

1.  列出 OSI 模型的所有七层。

1.  端口号的意义是什么？

1.  为什么应该在网络应用程序中使用套接字？

1.  描述在服务器端使用 TCP 套接字接收数据时应执行的操作顺序。

1.  TCP 和 UDP 之间有什么区别？

1.  为什么不应该在代码中使用宏定义？

1.  在实现服务器应用程序时，如何区分不同的客户端应用程序？

# 进一步阅读

+   *R. Stevens 的《TCP/IP Illustrated，Volume 1: The Protocols》：[`www.amazon.com/TCP-Illustrated-Protocols-Addison-Wesley-Professional/dp/0321336313/`](https://www.amazon.com/TCP-Illustrated-Protocols-Addison-Wesley-Professional/dp/0321336313/)

+   *Gordon Davies 的《网络基础知识》：[`www.packtpub.com/cloud-networking/networking-fundamentals`](https://www.packtpub.com/cloud-networking/networking-fundamentals)