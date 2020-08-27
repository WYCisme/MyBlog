---
title: C++并发编程学习（一）
reward: false
copyright: false
tags:
  - C++
  - 并发编程
categories:
  - 学习
date: 2020-03-23 22:52:50
---

> C++11标准在标准库中为多线程提供了组件，这意味着使用C++编写与平台无关的多线程程序成为可能，而C++程序的可移植性也得到了有力的保证。

# 1. 何为并发

**并发**指的是两个或多个独立的活动在**同一时段**内发生。生活中并发的例子并不少，例如在跑步的时候你可能同时在听音乐；在看电脑显示器的同时你的手指在敲击键盘。这时我们称我们大脑并发地处理这些事件，只不过我们大脑的处理是有次重点的：有时候你会更关注你呼吸的频率，而有时候你更多地被美妙的音乐旋律所吸引。这时我们可以说**大脑是一种并发设计的结构。这种次重点在计算机程序设计中，体现为某一个时刻只能处理一个操作**。

与并发相近的另一个概念是**并行**。它们两者存在很大的差别。并行就是**同时执行**，计算机在**同一时刻**，在某个时间点上处理两个或以上的操作。判断一个程序是否并行执行，只需要看某个时刻上是否多两个或以上的工作单位在运行。一个程序如果是单线程的，那么它无法并行地运行。利用多线程与多进程可以使得计算机并行地处理程序（当然 ，前提是该计算机有多个处理核心）。

- 并发：同一时间段内可以交替处理多个操作：

<img src="https://gitee.com/wycisme/imageBed/raw/master/img/20200323233645.png"/>

图中整个安检系统是一个**并发**设计的结构。两个安检队列队首的人竞争这一个安检窗口，两个队列可能约定交替着进行安检，也可能是大家同时竞争安检窗口（通信）。后一种方式可能引起冲突：因为无法同时进行两个安检操作。在**逻辑**上看来，这个安检窗口是同时处理这两个队列。

- 并行：同一时刻内同时处理多个操作：

<img src="https://gitee.com/wycisme/imageBed/raw/master/img/20200323233743.png"/>

图中整个安检系统是一个**并行**的系统。在这里，每个队列都有自己的安检窗口，两个队列中间没有竞争关系，队列中的某个排队者只需等待队列前面的人安检完成，然后再轮到自己安检。在**物理**上，安检窗口同时处理这两个队列。

**并发的程序设计，提供了一种方式让我们能够设计出一种方案将问题（非必须地）并行地解决。如果我们将程序的结构设计为可以并发执行的，那么在支持并行的机器上，我们可以将程序并行地执行。因此，并发重点指的是程序的设计结构，而并行指的是程序运行的状态。并发编程，是一种将一个程序分解成小片段独立执行的程序设计方法。**

# 2.并发的基本方式途径

**多线程与多进程是并发的两种途径。**
想象两个场景：

- 场景一：你和小伙伴要开发一个项目，但小伙伴们放寒假都回家了，你们只能通过QQ聊天、手机通话、发送思维导图等方式来进行交流，总之你们无法很方便地进行沟通。好处是你们各自工作时可以互不打扰。
- 场景二：你和小伙伴放假都呆在学校实验室中开发项目，你们可以聚在一起使用头脑风暴，可以使用白板进行观点的阐述，总之你们沟通变得更方便有效了。有点遗憾的是你在思考时可能有小伙伴过来问你问题，你受到了打扰。

这两个场景描绘了并发的两种基本途径。每个小伙伴代表一个线程，工作地点代表一个处理器。场景一中每个小伙伴是一个单线程的进程，他们拥有独立的处理器，多个进程同时执行；场景二中只有一个处理器，所有小伙伴都是属于同一进程的线程。



## 2.1 多进程并发

多个进程独立地运行，它们之间通过进程间常规的通信渠道传递讯息（信号，套接字，文件，管道等），这种进程间通信不是设置复杂就是速度慢，这是因为为了避免一个进程去修改另一个进程，操作系统在进程间提供了一定的保护措施，当然，这也使得编写安全的并发代码更容易。
运行多个进程也需要固定的开销：进程的启动时间，进程管理的资源消耗。



## 2.2 多线程并发

在当个进程中运行多个线程也可以并发。线程就像轻量级的进程，每个线程相互独立运行，但它们共享地址空间，所有线程访问到的大部分数据如指针、对象引用或其他数据可以在线程之间进行传递，它们都可以访问全局变量。进程之间通常共享内存，但这种共享通常难以建立且难以管理，缺少线程间数据的保护。因此，在多线程编程中，我们必须确保每个线程锁访问到的数据是一致的。

# 3. C++中的并发与多线程

C++标准并没有提供对多进程并发的原生支持，所以C++的多进程并发要靠其他API——这需要依赖相关平台。
C++11 标准提供了一个新的线程库，内容包括了管理线程、保护共享数据、线程间的同步操作、低级原子操作等各种类。标准极大地提高了程序的可移植性，以前的多线程依赖于具体的平台，而现在有了统一的接口进行实现。

C++11 新标准中引入了几个头文件来支持多线程编程：

- **< thread >** :包含std::thread类以及std::this_thread命名空间。管理线程的函数和类在 中声明.
- **< atomic > **:包含std::atomic和std::atomic_flag类，以及一套C风格的原子类型和与C兼容的原子操作的函数。
- **< mutex > **:包含了与互斥量相关的类以及其他类型和函数
- **< future > **:包含两个Provider类（std::promise和std::package_task）和两个Future类（std::future和std::shared_future）以及相关的类型和函数。
- **< condition_variable > **:包含与条件变量相关的类，包括std::condition_variable和std::condition_variable_any。



## 3.1 初试多线程

我们从一个hello开始。在单线程时：

```cpp
# include<iostream>
using namespace std;
int main()
{
    cout<<"hello world"<<endl;
}
```

在这里，进行由一个线程组成，该线程的初始函数是main。我们启动第二个线程来打印hello world：

```cpp
# include<iostream>
# include<thread>
using namespace std;
void hello()
{
    cout<<"hello world"<<endl;
}
int main()
{
    thread t (hello);
    t.join();
}
```

在这里，我们将打印hello world的语句放在函数hello中。每个线程都必须有一个初始函数，新线程的执行开始于初始函数。对于第一段程序来说，它的初始函数是main，对于我们新创建的线程，可以在std::thread()对象的构造函数中指定。
在第二段程序里，程序由两个线程组成：初始线程始于main，新线程始于hello。这里将新线程t的初始函数指定为hello。
新线程启动之后会与初始进程一并运行，初始线程可以等待或不等待新进程的运行结束——如果需要等待线程，则新线程实例需要使用join(),否则可以使用detach()。如果不等待新线程，则初始线程自顾自地运行到main()结束。
关于< thread > 我们将在下一篇中进行详解。
由于我们的初始线程并没有做什么事情，启动新线程后，新线程将打印出hello world。

这就是我们编写出的第一个多线程的程序，一般来说并不值得为了如此简单的任务而使用多线程，尤其是在这期间初始线程并没做什么。

在下一篇文章里，我们将继续探索**< thread >头文件**的内容，编写更复杂的并发程序。

来源文章：http://www.cnblogs.com/QG-whz/p/5186243.html