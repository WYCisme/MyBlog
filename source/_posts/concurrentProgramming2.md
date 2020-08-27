---
title: C++并发编程学习（二）
reward: false
copyright: false
tags:
  - C++
  - 并发编程
categories:
  - 学习
date: 2020-03-24 12:09:42
---

C++11 新标准中引入了几个头文件来支持多线程编程，他们分别是：

- `<atomic>`该头文主要声明了两个类, std::atomic 和 std::atomic_flag，另外还声明了一套 C 风格的原子类型和与 C 兼容的原子操作的函数。
- `<thread>`该头文件主要声明了 std::thread 类，另外 std::this_thread 命名空间也在该头文件中。
- `<mutex>`该头文件主要声明了与互斥量(mutex)相关的类，包括 std::mutex 系列类，std::lock_guard, std::unique_lock, 以及其他的类型和函数。
- `<condition_variable>`该头文件主要声明了与条件变量相关的类，包括 std::condition_variable 和 std::condition_variable_any。
- `<future>`该头文件主要声明了 std::promise, std::package_task 两个 Provider 类，以及 std::future 和 std::shared_future 两个 Future 类，另外还有一些与之相关的类型和函数，std::async() 函数就声明在此头文件中。

------

##  std::thread

构造函数

| default (1)        | `thread() noexcept; `                                   |
| ------------------ | ------------------------------------------------------- |
| initialization (2) | `template  explicit thread (Fn&& fn, Args&&... args); ` |
| copy [deleted] (3) | `thread (const thread&) = delete; `                     |
| move (4)           | `thread (thread&& x) noexcept;`                         |

- (1). 默认构造函数，创建一个空的 thread 执行对象。
- (2). 初始化构造函数，创建一个 thread对象，该 thread对象可被 joinable，新产生的线程会调用 fn 函数，该函数的参数由 args 给出。
- (3). 拷贝构造函数(被禁用)，意味着 thread 不可被拷贝构造。
- (4). move 构造函数，move 构造函数，调用成功之后 x 不代表任何 thread 执行对象。
- 注意：可被 joinable 的 thread 对象必须在他们销毁之前被主线程 join 或者将其设置为 detached.
- [拷贝构造函数，移动构造函数](https://www.jianshu.com/p/d19fc8447eaa)

 

### thread使用例子

```c++
#include <iostream>
#include <utility>
#include <thread>
#include <chrono>
#include <functional>
#include <atomic>
 
void f1(int n)
{
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread " << n << " executing\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
 
void f2(int& n)
{
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread 2 executing\n";
        ++n;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
 
int main()
{
    int n = 0;
    std::thread t1; // t1 is not a thread
    std::thread t2(f1, n + 1); // pass by value
    std::thread t3(f2, std::ref(n)); // pass by reference
    std::thread t4(std::move(t3)); // t4 is now running f2(). t3 is no longer a thread
    t2.join();
    t4.join();
    std::cout << "Final value of n is " << n << '\n';
}
```

------

## std::mutex

#### Mutex 类

- std::mutex，最基本的 Mutex 类，如果当前线程对同一个mutex多次加锁，会产生死锁（dead lock）；
- std::recursive_mutex，递归锁，允许同一个线程对互斥量多次上锁（即递归上锁），来获得对互斥量对象的多层所有权，recursive_mutex 释放互斥量时需要调用与该锁层次深度相同次数的 unlock()，可理解为 lock() 次数和 unlock() 次数相同，。
- std::time_mutex，定时 Mutex 类。
- std::recursive_timed_mutex，定时递归 Mutex 类。

#### Lock 类

- std::lock_guard，与 Mutex RAII 相关，方便线程对互斥量上锁。
- std::unique_lock，与 Mutex RAII 相关，方便线程对互斥量上锁，但提供了更好的上锁和解锁控制。

#### 其他类型

- std::once_flag，配合std:call_once使用；
- std::adopt_lock_t，通常作为参数传入给 unique_lock 或 lock_guard 的构造函数；
- std::defer_lock_t，通常作为参数传入给 unique_lock 的构造函数；
- std::try_to_lock_t，通常作为参数传入给 unique_lock 的构造函数；

#### 函数

- std::try_lock，调用时没有获得锁，则直接返回 false
- std::lock，调用线程将阻塞等待该互斥量。
- std::try_lock_for，对定时锁可用，接受一个时间范围，表示在这一段时间范围之内线程如果没有获得锁则被阻塞住，如果在此期间其他线程释放了锁，则该线程可以获得对互斥量的锁，如果超时则返回 false；
- std::try_lock_util，对定时锁可用，接受一个时间点作为参数，在指定时间点未到来之前线程如果没有获得锁则被阻塞住，如果在此期间其他线程释放了锁，则该线程可以获得对互斥量的锁，如果超时则返回 false；
- std::call_once，如果多个线程需要同时调用某个函数，call_once 可以保证多个线程对该函数只调用一次。

**定时锁例子：**

```c++
#include <iostream>       // std::cout
#include <chrono>         // std::chrono::milliseconds
#include <thread>         // std::thread
#include <mutex>          // std::timed_mutex

std::timed_mutex mtx;

void fireworks() {
  // waiting to get a lock: each thread prints "-" every 200ms:
  while (!mtx.try_lock_for(std::chrono::milliseconds(200))) {
    std::cout << "-";
  }
  // got a lock! - wait for 1s, then this thread prints "*"
  std::this_thread::sleep_for(std::chrono::milliseconds(1000));
  std::cout << "*\n";
  mtx.unlock();
}

int main ()
{
  std::thread threads[10];
  // spawn 10 threads:
  for (int i=0; i<10; ++i)
    threads[i] = std::thread(fireworks);

  for (auto& th : threads) th.join();

  return 0;
}	
```

### lock_guard例子

[用RAII思想管理锁，不用手动释放](https://www.cnblogs.com/chenny7/p/11990105.html)

在 lock_guard 对象构造时，传入的 Mutex 对象会被当前线程锁住。在lock_guard 对象被析构时，它所管理的 Mutex 对象会自动解锁，由于不需要程序员手动调用 lock 和 unlock 对 Mutex 进行上锁和解锁操作，因此这也是最简单安全的上锁和解锁方式，尤其是在程序抛出异常后先前已被上锁的 Mutex 对象可以正确进行解锁操作，极大地简化了程序员编写与 Mutex 相关的异常处理代码。

值得注意的是，lock_guard 对象并不负责管理 Mutex 对象的生命周期，lock_guard 对象只是简化了 Mutex 对象的上锁和解锁操作，方便线程对互斥量上锁，即在某个 lock_guard 对象的声明周期内，它所管理的锁对象会一直保持上锁状态；而 lock_guard 的生命周期结束之后，它所管理的锁对象会被解锁。

一个简单例子:

```c++
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock_guard
#include <stdexcept>      // std::logic_error

std::mutex mtx;

void print_even (int x) {
    if (x%2==0) std::cout << x << " is even\n";
    else throw (std::logic_error("not even"));
}

void print_thread_id (int id) {
    try {
        // using a local lock_guard to lock mtx guarantees unlocking on destruction / exception:
        std::lock_guard<std::mutex> lck (mtx);
        print_even(id);
    }
    catch (std::logic_error&) {
        std::cout << "[exception caught]\n";
    }
}

int main ()
{
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(print_thread_id,i+1);

    for (auto& th : threads) th.join();

    return 0;
}
```

adopting初始化，对一个已经加锁的mutex使用lock_guard，由其负责解锁:

```c++
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock_guard, std::adopt_lock

std::mutex mtx;           // mutex for critical section

void print_thread_id (int id) {
  mtx.lock();
  std::lock_guard<std::mutex> lck(mtx, std::adopt_lock);
  std::cout << "thread #" << id << '\n';
}

int main ()
{
  std::thread threads[10];
  // spawn 10 threads:
  for (int i=0; i<10; ++i)
    threads[i] = std::thread(print_thread_id,i+1);

  for (auto& th : threads) th.join();

  return 0;
}
```

lock_guard只能保证在析构的时候执行解锁操作，本身并没有提供加锁和解锁的接口，看下面的例子:

```c++
class LogFile {
    std::mutex _mu;
    ofstream f;
public:
    LogFile() {
        f.open("log.txt");
    }
    ~LogFile() {
        f.close();
    }
    void shared_print(string msg, int id) {
        {
            std::lock_guard<std::mutex> guard(_mu);
            //do something 1
        }
        //do something 2
        {
            std::lock_guard<std::mutex> guard(_mu);
            // do something 3
            f << msg << id << endl;
            cout << msg << id << endl;
        }
    }

};
```

上面的代码中，一个函数内部有两段代码需要进行保护，这个时候使用`lock_guard`就需要创建两个局部对象来管理同一个互斥锁（其实也可以只创建一个，但是锁的力度太大，效率不行），修改方法是使用`unique_lock`。

### unique_lock例子

它提供了`lock()`和`unlock()`接口，能记录现在处于上锁还是没上锁状态，在析构的时候，会根据当前状态来决定是否要进行解锁（`lock_guard`就一定会解锁）。 

```c++
class LogFile {
    std::mutex _mu;
    ofstream f;
public:
    LogFile() {
        f.open("log.txt");
    }
    ~LogFile() {
        f.close();
    }
    void shared_print(string msg, int id) {

        std::unique_lock<std::mutex> guard(_mu);
        //do something 1
        guard.unlock(); //临时解锁

        //do something 2

        guard.lock(); //继续上锁
        // do something 3
        f << msg << id << endl;
        cout << msg << id << endl;
        // 结束时析构guard会临时解锁
        // 这句话可要可不要，不写，析构的时候也会自动执行
        // guard.ulock();
    }

};
```

上面的代码可以看到，在无需加锁的操作时，可以先临时释放锁，然后需要继续保护的时候，可以继续上锁，这样就无需重复的实例化`lock_guard`对象，还能减少锁的区域。



adopting 初始化，新创建的 unique_lock 对象管理 Mutex 对象 m， m 应该是一个已经被当前线程锁住的 Mutex 对象。(并且当前新创建的 unique_lock 对象拥有对锁(Lock)的所有权)

deferred 初始化，新创建的 unique_lock 对象管理 Mutex 对象 m，但是在初始化的时候并不锁住 Mutex 对象。 m 应该是一个没有当前线程锁住的 Mutex 对象

try-locking 初始化，新创建的 unique_lock 对象管理 Mutex 对象 m，并尝试调用 m.try_lock() 对 Mutex 对象进行上锁，但如果上锁不成功，并不会阻塞当前线程

```c++
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock, std::unique_lock
                          // std::adopt_lock, std::defer_lock
std::mutex foo,bar;

void task_a () {
  std::lock (foo,bar);         // simultaneous lock (prevents deadlock)
  std::unique_lock<std::mutex> lck1 (foo,std::adopt_lock);
  std::unique_lock<std::mutex> lck2 (bar,std::adopt_lock);
  std::cout << "task a\n";
  // (unlocked automatically on destruction of lck1 and lck2)
}

void task_b () {
  // foo.lock(); bar.lock(); // replaced by:
  std::unique_lock<std::mutex> lck1, lck2;
  lck1 = std::unique_lock<std::mutex>(bar,std::defer_lock);
  lck2 = std::unique_lock<std::mutex>(foo,std::defer_lock);
  std::lock (lck1,lck2);       // simultaneous lock (prevents deadlock)
  std::cout << "task b\n";
  // (unlocked automatically on destruction of lck1 and lck2)
}


int main ()
{
  std::thread th1 (task_a);
  std::thread th2 (task_b);

  th1.join();
  th2.join();

  return 0;
}
```

后面在学习条件变量的时候，还会有`unique_lock`的用武之地。

 

### call_once例子

若调用发生异常，不会翻转flag，以令其它调用得到尝试

```c++
#include <iostream>
#include <thread>
#include <mutex>
 
std::once_flag flag1, flag2;
 
void simple_do_once()
{
    std::call_once(flag1, [](){ std::cout << "Simple example: called once\n"; });
}
 
void may_throw_function(bool do_throw)
{
  if (do_throw) {
    std::cout << "throw: call_once will retry\n"; // 这会出现多于一次
    throw std::exception();
  }
  std::cout << "Didn't throw, call_once will not attempt again\n"; // 保证一次
}
 
void do_once(bool do_throw)
{
  try {
    std::call_once(flag2, may_throw_function, do_throw);
  }
  catch (...) {
  }
}
 
int main()
{
    std::thread st1(simple_do_once);
    std::thread st2(simple_do_once);
    std::thread st3(simple_do_once);
    std::thread st4(simple_do_once);
    st1.join();
    st2.join();
    st3.join();
    st4.join();
 
    std::thread t1(do_once, true);
    std::thread t2(do_once, true);
    std::thread t3(do_once, false);
    std::thread t4(do_once, true);
    t1.join();
    t2.join();
    t3.join();
    t4.join();
}
```



------

##  std::future

<future> 头文件中包含了以下几个类和函数：

- Providers 类：std::promise, std::package_task
- Futures 类：std::future, shared_future
- Providers 函数：std::async()
- 其他类型：std::future_error, std::future_errc, std::future_status, std::launch

 

`future`和`promise`的作用是在不同线程之间传递数据，大概流程如下

<img src="https://gitee.com/wycisme/imageBed/raw/master/img/20200325133626.png"/>

流程：

1. 线程1初始化一个promise对象和一个future对象，并将promise传递给线程2，相当于线程2对线程1的一个承诺；future相当于一个接受一个承诺，用来获取未来线程2传递的值；
2. 线程2获取到promise后，需要对这个promise传递有关的数据，之后线程1的future就可以获取数据了；
3. 如果线程1想要获取数据，而线程2未给出数据，则线程1阻塞，直到线程2的数据到达

 

一个例子：

```c++
#include <iostream>
#include <functional>
#include <future>
#include <thread>
#include <chrono>
#include <cstdlib>

void thread_set_promise(std::promise<int>& promiseObj) {
    std::cout << "In a thread, making data...\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    promiseObj.set_value(35);
    std::cout << "Finished\n";
}

int main() {
    std::promise<int> promiseObj;
    std::future<int> futureObj = promiseObj.get_future();
    std::thread t(&thread_set_promise, std::ref(promiseObj));
    std::cout << futureObj.get() << std::endl;
    t.join();

    return 0;
}
```

### promise例子

在 promise 对象构造时可以和一个共享状态（通常是std::future）相关联，并可以在相关联的共享状态(std::future)上保存一个类型为 T 的值。

**future对象的成员函数：**

- std::promise::get_future，返回一个与 promise 共享状态相关联的 future ，返回的 future 对象可以访问由 promise 对象设置在共享状态上的值或者某个异常对象，如果不设置值或者异常，promise 对象在析构时会自动地设置一个 future_error 异常；
- std::promise::set_value，设置共享状态的值，此后 promise 的共享状态标志变为 ready；
- std::promise::set_exception，为 promise 设置异常，此后 promise 的共享状态变标志变为 ready；
- std::promise::set_value_at_thread_exit，设置共享状态的值，但是不将共享状态的标志设置为 ready，当线程退出时该 promise 对象会自动设置为 ready；

 

设置异常的例子：

```c++
#include <iostream>       // std::cin, std::cout, std::ios
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future
#include <exception>      // std::exception, std::current_exception

void get_int(std::promise<int>& prom) {
    int x;
    std::cout << "Please, enter an integer value: ";
    std::cin.exceptions (std::ios::failbit);   // throw on failbit
    try {
        std::cin >> x;                         // sets failbit if input is not int
        prom.set_value(x);
    } catch (std::exception&) {
        prom.set_exception(std::current_exception());
    }
}

void print_int(std::future<int>& fut) {
    try {
        int x = fut.get();
        std::cout << "value: " << x << '\n';
    } catch (std::exception& e) {
        std::cout << "[exception caught: " << e.what() << "]\n";
    }
}

int main ()
{
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread th1(get_int, std::ref(prom));
    std::thread th2(print_int, std::ref(fut));

    th1.join();
    th2.join();
    return 0;
}
```

 

### package_task例子

std::packaged_task 包装一个可调用的对象，并且允许异步获取该可调用对象产生的结果，

std::packaged_task 对象内部包含了两个最基本元素，一、被包装的任务(stored task)，任务(task)是一个可调用的对象，如函数指针、成员函数指针或者函数对象，二、共享状态(shared state)，用于保存任务的返回值，可以通过 std::future 对象来达到异步访问共享状态的效果。

**packaged_task对象的成员函数：**

- std::packaged_task::valid，检查当前 packaged_task 是否和一个有效的共享状态相关联；
- std::packaged_task::get_future，来获取与共享状态相关联的 std::future 对象；
- std::packaged_task::make_ready_at_thread_exit，
- std::packaged_task::reset()，重置 packaged_task 的共享状态，但是保留之前的被包装的任务

 

package_task使用例子:

```c++
#include <iostream>     // std::cout
#include <future>       // std::packaged_task, std::future
#include <chrono>       // std::chrono::seconds
#include <thread>       // std::thread, std::this_thread::sleep_for

// count down taking a second for each value:
int countdown (int from, int to) {
    for (int i=from; i!=to; --i) {
        std::cout << i << '\n';
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "Finished!\n";
    return from - to;
}

int main ()
{
    std::packaged_task<int(int,int)> task(countdown); // 设置 packaged_task
    std::future<int> ret = task.get_future(); // 获得与 packaged_task 共享状态相关联的 future 对象.

    std::thread(std::move(task), 10, 0).detach();   //创建一个新线程完成计数任务.

    int value = ret.get();                    // 等待任务完成并获取结果.

    std::cout << "The countdown lasted for " << value << " seconds.\n";

    return 0;
}
```

reset 的例子:

```c++
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

// a simple task:
int triple (int x) { return x*3; }

int main ()
{
    std::packaged_task<int(int)> tsk (triple); // package task


    std::future<int> fut = tsk.get_future();
    std::thread (std::move(tsk), 100).detach();
    std::cout << "The triple of 100 is " << fut.get() << ".\n";


    // re-use same task object:
    tsk.reset();
    fut = tsk.get_future();
    std::thread(std::move(tsk), 200).detach();
    std::cout << "Thre triple of 200 is " << fut.get() << ".\n";

    return 0;
}
```

###  future例子

std::future 可以用来获取异步任务的结果。

std::future 通常由某个 Provider 创建，你可以把 Provider 想象成一个异步任务的提供者，Provider 在某个线程中设置共享状态的值，与该共享状态相关联的 std::future 对象调用 get（通常在另外一个线程中） 获取该值，如果共享状态的标志不为 ready，则调用 std::future::get 会阻塞当前的调用者，直到 Provider 设置了共享状态的值（此时共享状态的标志变为 ready），std::future::get 返回异步任务的值或异常（如果发生了异常）。

一个有效(valid)的 std::future 对象通常由以下三种 Provider 创建，并和某个共享状态相关联。Provider 可以是函数或者类，其实我们前面都已经提到了，他们分别是：

- std::async 函数；
- std::promise::get_future，get_future 为 promise 类的成员函数；
- std::packaged_task::get_future，此时 get_future为 packaged_task 的成员函数；

 

async 函数使用例子:

```c++
// future example
#include <iostream>             // std::cout
#include <future>               // std::async, std::future
#include <chrono>               // std::chrono::milliseconds

// a non-optimized way of checking for prime numbers:
bool
is_prime(int x)
{
    for (int i = 2; i < x; ++i)
        if (x % i == 0)
            return false;
    return true;
}

int
main()
{
    // call function asynchronously:
    std::future < bool > fut = std::async(is_prime, 444444443);

    // do something while waiting for function to set future:
    std::cout << "checking, please wait";
    std::chrono::milliseconds span(100);
    while (fut.wait_for(span) == std::future_status::timeout)
        std::cout << '.';

    bool x = fut.get();         // retrieve return value

    std::cout << "\n444444443 " << (x ? "is" : "is not") << " prime.\n";

    return 0;
}
```

std::async() 返回一个 std::future 对象，通过该对象可以获取异步任务的值或异常（如果异步任务抛出了异常）

另外，async 函数可以指定启动策略 std::launch ，该枚举参数可以是launch::async，launch::deferred，以及两者的按位或( | )；

```c++
#include <stdio.h>
#include <stdlib.h>

#include <cmath>
#include <chrono>
#include <future>
#include <iostream>

double ThreadTask(int n) {
    std::cout << std::this_thread::get_id()
        << " start computing..." << std::endl;

    double ret = 0;
    for (int i = 0; i <= n; i++) {
        ret += std::sin(i);
    }

    std::cout << std::this_thread::get_id()
        << " finished computing..." << std::endl;
    return ret;
}

int main(int argc, const char *argv[])
{
    std::future<double> f(std::async(std::launch::async, ThreadTask, 100000000));

#if 0
    while(f.wait_until(std::chrono::system_clock::now() + std::chrono::seconds(1))
            != std::future_status::ready) {
        std::cout << "task is running...\n";
    }
#else
    while(f.wait_for(std::chrono::seconds(1))
            != std::future_status::ready) {
        std::cout << "task is running...\n";
    }
#endif

    std::cout << f.get() << std::endl;

    return EXIT_SUCCESS;
}
```

 std::launch枚举类型主要是在调用 std::async 设置异步任务的启动策略的。

| 类型                                                | 描述                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| [launch::async](http://www.cplusplus.com/launch)    | **Asynchronous:** 异步任务会在另外一个线程中调用，并通过共享状态返回异步任务的结果（一般是调用 std::future::get() 获取异步任务的结果）。 |
| [launch::deferred](http://www.cplusplus.com/launch) | **Deferred:** 异步任务将会在共享状态被访问时调用，相当与按需调用（即延迟(deferred)调用）。 |

 

**future对象的成员函数：**

- std::future::valid()，检查当前的 std::future 对象是否有效；
- std::future::get()，调用该函数会阻塞当前的调用者，而此后一旦共享状态的标志变为 ready，get 返回 Provider 所设置的共享状态的值或者异常（如果抛出了异常）；
- std::future::share()，返回一个 std::shared_future 对象，调用该函数之后，该 std::future 对象本身已经不和任何共享状态相关联，因此该 std::future 的状态不再是 valid 的了；
- std::future::wait()，等待与该 std::future 对象相关联的共享状态的标志变为 ready，但是 wait() 并不读取共享状态的值或者异常；
- std::future::wait_for()，可以设置一个时间段 rel_time，如果共享状态的标志在该时间段结束之前没有被 Provider 设置为 ready，则调用 wait_for 的线程被阻塞，在等待了 rel_time 的时间长度后 wait_until() 返回；
- std::future::wait_until()，可以设置一个系统绝对时间点 abs_time，如果共享状态的标志在该时间点到来之前没有被 Provider 设置为 ready，则调用 wait_until 的线程被阻塞，在 abs_time 这一时刻到来之后 wait_for() 返回；

valid 使用例子:

```c++
#include <iostream>       // std::cout
#include <future>         // std::async, std::future
#include <utility>        // std::move

int do_get_value() { return 11; }

int main ()
{
    // 由默认构造函数创建的 std::future 对象,
    // 初始化时该 std::future 对象处于为 invalid 状态.
    std::future<int> foo, bar;
    foo = std::async(do_get_value); // move 赋值, foo 变为 valid.
    bar = std::move(foo); // move 赋值, bar 变为 valid, 而 move 赋值以后 foo 变为 invalid.

    if (foo.valid())
        std::cout << "foo's value: " << foo.get() << '\n';
    else
        std::cout << "foo is not valid\n";

    if (bar.valid())
        std::cout << "bar's value: " << bar.get() << '\n';
    else
        std::cout << "bar is not valid\n";

    return 0;
}
```

wait_for 使用例子:

```c++
#include <iostream>                // std::cout
#include <future>                // std::async, std::future
#include <chrono>                // std::chrono::milliseconds

// a non-optimized way of checking for prime numbers:
bool do_check_prime(int x) // 为了体现效果, 该函数故意没有优化.
{
    for (int i = 2; i < x; ++i)
        if (x % i == 0)
            return false;
    return true;
}

int main()
{
    // call function asynchronously:
    std::future < bool > fut = std::async(do_check_prime, 194232491);

    std::cout << "Checking...\n";
    std::chrono::milliseconds span(1000); // 设置超时间隔.

    // 如果超时，则输出"."，继续等待
    while (fut.wait_for(span) == std::future_status::timeout)
        std::cout << '.';

    std::cout << "\n194232491 ";
    if (fut.get()) // guaranteed to be ready (and not block) after wait returns
        std::cout << "is prime.\n";
    else
        std::cout << "is not prime.\n";

    return 0;
}
```

 

### shared_future例子

shared_future支持拷贝，多个 std::shared_future 可以共享某个共享状态的最终结果(即共享状态的某个值或者异常)。；

------

##  std::condition_variable

 先看一个例子:

```c++
#include <iostream>                // std::cout
#include <thread>                // std::thread
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx; // 全局互斥锁.
std::condition_variable cv; // 全局条件变量.
bool ready = false; // 全局标志位.

void do_print_id(int id)
{
    std::unique_lock <std::mutex> lck(mtx);
    while (!ready) // 如果标志位不为 true, 则等待...
        cv.wait(lck); // 当前线程被阻塞, 当全局标志位变为 true 之后,
    // 线程被唤醒, 继续往下执行打印线程编号id.
    std::cout << "thread " << id << '\n';
}

void go()
{
    std::unique_lock <std::mutex> lck(mtx);
    ready = true; // 设置全局标志位为 true.
    cv.notify_all(); // 唤醒所有线程.
}

int main()
{
    std::thread threads[10];
    // spawn 10 threads:
    for (int i = 0; i < 10; ++i)
        threads[i] = std::thread(do_print_id, i);

    std::cout << "10 threads ready to race...\n";
    go(); // go!

  for (auto & th:threads)
        th.join();

    return 0;
}
```

wait函数执行的步骤：

- *unlock mutex，wait调用要和mutex配合，调用wait前要先获取mutex的锁，调用wait时会先自动解锁，使得其他被阻塞在锁竞争上的线程得以继续执行。*
- *waiting for notify*，阻塞等待唤醒；
- waked by notify，被唤醒；
- lock mutex，自动重新加锁，使得mutex状态和wait被调用时相同；

另外，上面的代码中，

```c
while (!ready) // 如果标志位不为 true, 则等待...
        cv.wait(lck); // 当前线程被阻塞, 当全局标志位变为 true 之后,  线程被唤醒, 继续往下执行打印线程编号id.
```

可以用下面的语句替换：

```c
cv.wait(lck, isReady);


// isReady的实现
bool isReady() {
    return ready;
}
```



### 更多wait函数

```c
// wait
void wait (unique_lock<mutex>& lck);
template <class Predicate>
  void wait (unique_lock<mutex>& lck, Predicate pred);

// wait_for
template <class Rep, class Period>
  cv_status wait_for (unique_lock<mutex>& lck,
                      const chrono::duration<Rep,Period>& rel_time);
template <class Rep, class Period, class Predicate>
       bool wait_for (unique_lock<mutex>& lck,
                      const chrono::duration<Rep,Period>& rel_time, Predicate pred);

// wait_until
template <class Clock, class Duration>
  cv_status wait_until (unique_lock<mutex>& lck,
                        const chrono::time_point<Clock,Duration>& abs_time);
template <class Clock, class Duration, class Predicate>
       bool wait_until (unique_lock<mutex>& lck,
                        const chrono::time_point<Clock,Duration>& abs_time,
                        Predicate pred);
```

*wait_for 可以指定一个时间段，在当前线程收到通知或者指定的时间 rel_time 超时之前，该线程都会处于阻塞状态；*

*wait_until 可以指定一个时间点，在当前线程收到通知或者指定的时间点 abs_time 超时之前，该线程都会处于阻塞状态；*

 wait_for 例子:

```c++
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <chrono>             // std::chrono::seconds
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable, std::cv_status

std::condition_variable cv;

int value;

void do_read_value()
{
    std::cin >> value;
    cv.notify_one();
}

int main ()
{
    std::cout << "Please, enter an integer (I'll be printing dots): \n";
    std::thread th(do_read_value);

    std::mutex mtx;
    std::unique_lock<std::mutex> lck(mtx);
    while (cv.wait_for(lck,std::chrono::seconds(1)) == std::cv_status::timeout) {
        std::cout << '.';
        std::cout.flush();
    }

    std::cout << "You entered: " << value << '\n';

    th.join();
    return 0;
}
```

*上面的例子使用了std::cv_status枚举类型：*

| cv_status::no_timeout | wait_for 或者 wait_until 没有超时，即在规定的时间段内线程收到了通知。 |
| --------------------- | ------------------------------------------------------------ |
| cv_status::timeout    | wait_for 或者 wait_until 超时。                              |

### notify函数

- notify_one，唤醒某个等待(wait)线程。如果当前没有等待线程，则该函数什么也不做，如果同时存在多个等待线程，则唤醒某个线程是不确定的(unspecified)
- notify_all，唤醒所有的等待(wait)线程。如果当前没有等待线程，则该函数什么也不做。

 

### notify_all_at_thread_exit

当调用该函数的线程退出时，所有在 cond 条件变量上等待的线程都会收到通知。

例子：

```c++
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void print_id (int id) {
  std::unique_lock<std::mutex> lck(mtx);
  while (!ready) cv.wait(lck);
  // ...
  std::cout << "thread " << id << '\n';
}

void go() {
  std::unique_lock<std::mutex> lck(mtx);
  std::notify_all_at_thread_exit(cv,std::move(lck));
  ready = true;
}

int main ()
{
  std::thread threads[10];
  // spawn 10 threads:
  for (int i=0; i<10; ++i)
    threads[i] = std::thread(print_id,i);
  std::cout << "10 threads ready to race...\n";

  std::thread(go).detach();   // go!

  for (auto& th : threads) th.join();

  return 0;
}
```

 

### condition_variable_any

与 std::condition_variable 类似，只不过 std::condition_variable_any 的 wait 函数可以接受任何 lockable 参数，而 std::condition_variable 只能接受 std::unique_lock<std::mutex> 类型的参数，除此以外，和 std::condition_variable 几乎完全一样。

------

## std::atomic

原子操作是可以lock-free的算法和数据结构。 

### std::atomic_flag:

```c
#include <iostream>              // std::cout
#include <atomic>                // std::atomic, std::atomic_flag, ATOMIC_FLAG_INIT
#include <thread>                // std::thread, std::this_thread::yield
#include <vector>                // std::vector

std::atomic<bool> ready(false);    // can be checked without being set
std::atomic_flag winner = ATOMIC_FLAG_INIT;    // always set when checked

void count1m(int id)
{
    while (!ready) {
        std::this_thread::yield();
    } // 等待主线程中设置 ready 为 true.

    for (int i = 0; i < 1000000; ++i) {
    } // 计数.

    // 如果某个线程率先执行完上面的计数过程，则输出自己的 ID.
    // 此后其他线程执行 test_and_set 是 if 语句判断为 false，
    // 因此不会输出自身 ID.
    if (!winner.test_and_set()) {
        std::cout << "thread #" << id << " won!\n";
    }
};

int main()
{
    std::vector<std::thread> threads;
    std::cout << "spawning 10 threads that count to 1 million...\n";
    for (int i = 1; i <= 10; ++i)
        threads.push_back(std::thread(count1m, i));
    ready = true;

    for (auto & th:threads)
        th.join();

    return 0;
}
```

std::atomic_flag 的 test_and_set 函数是原子的：

test_and_set() 函数检查 std::atomic_flag 标志，如果 std::atomic_flag 之前没有被设置过，则设置 std::atomic_flag 的标志，并返回先前该 std::atomic_flag 对象是否被设置过，如果之前 std::atomic_flag 对象已被设置，则返回 true，否则返回 false。 

 std::atomic_flag 的 clear 函数，清除 std::atomic_flag 标志使得下一次调用 std::atomic_flag::test_and_set 返回 false。



> From：<https://www.cnblogs.com/chenny7/p/11996237.html>