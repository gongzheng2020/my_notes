
---

## 动态内存

C++ 程序中的内存分为两个部分：
- **栈：** 由编译器自动管理，遵循LIFO，分配极快但容量小，用于函数调用上下文与局部变量。
- **堆：** 由开发者手动管理，容量大但分配慢、易碎片化，用于运行时动态数据。
![[Pasted image 20260609104323.png]]

C++中通过`new`申请内存，使用`delete`释放内存。通用语法如下：
``` cpp
new data-type;
```
一个例子如下：
``` cpp
#include <iostream>
using namespace std;
int main ()
{
   double* pvalue  = NULL; // 初始化为 null 的指针
   pvalue  = new double;   // 为变量请求内存
   *pvalue = 29494.99;     // 在分配的地址存储值
   cout << "Value of pvalue : " << *pvalue << endl;
   delete pvalue;         // 释放内存
   return 0;
}
```

### 数组的动态内存分配
``` cpp
char* pvalue  = NULL;   // 初始化为 null 的指针
pvalue  = new char[20]; // 为变量请求内存
delete [] pvalue;        // 删除 pvalue 所指向的数组
```

### 对象的动态内存分配
``` cpp
#include <iostream>
using namespace std;
class Box
{
   public:
      Box() { 
         cout << "调用构造函数！" <<endl; 
      }
      ~Box() { 
         cout << "调用析构函数！" <<endl; 
      }
};
int main( )
{
   Box* myBoxArray = new Box[4];
   delete [] myBoxArray; // 删除数组
   return 0;
}
```

---

## 命名空间

**命名空间**作为附加信息来区分不同库中相同名称的函数、类、变量等。使用了命名空间即定义了上下文。本质上，命名空间就是定义了一个范围。

### 定义命名空间

使用关键字 **namespace**，后跟命名空间的名称，如下所示
``` cpp
namespace namespace_name {
   // 代码声明
}
```
为了调用带有命名空间的函数或变量，需要在**前面加上命名空间的名称**，如下所示：
``` cpp
namespace::code;  // code 可以是变量或函数
```
一个例子如下：
``` cpp
#include <iostream>
using namespace std;
// 第一个命名空间
namespace first_space{
   void func(){
      cout << "Inside first_space" << endl;
   }
}
// 第二个命名空间
namespace second_space{
   void func(){
      cout << "Inside second_space" << endl;
   }
}
int main ()
{
   // 调用第一个命名空间中的函数
   first_space::func();
   // 调用第二个命名空间中的函数
   second_space::func(); 
   return 0;
}
```
>一个命名空间的各个组成部分**可以分散在多个文件中**。

### using指令

使用 **using namespace** 指令，这样**在使用命名空间时就可以不用在前面加上命名空间的名称**。这个指令会告诉编译器，**后续的代码将使用指定的命名空间中的名称**。一个例子如下：
``` cpp
#include <iostream>
using namespace std;
// 第一个命名空间
namespace first_space{
   void func(){
      cout << "Inside first_space" << endl;
   }
}
// 第二个命名空间
namespace second_space{
   void func(){
      cout << "Inside second_space" << endl;
   }
}
using namespace first_space;
int main ()
{
   // 调用第一个命名空间中的函数
   func();
   return 0;
}
```
using 指令也可以用来**指定命名空间中的特定项目**，例如`using std::cout;`，一个例子如下：
``` cpp
#include <iostream>
using std::cout;
int main ()
{
   cout << "std::endl is used with std!" << std::endl;
   return 0;
}
```

### 命名空间的嵌套

在一个已有的命名空间中可以嵌套另外的命名空间，并使用`::`运算符来访问嵌套的命名空间中的成员，示例如下：
``` cpp
namespace namespace_name1 {
   // 代码声明
   namespace namespace_name2 {
      // 代码声明
   }
}
using namespace namespace_name1::namespace_name2; //访问空间1中的空间2成员
```

---

## 模板

模板是**泛型编程**的基础，泛型编程即**以一种独立于任何特定类型的方式编写代码**。

### 函数模板

模板函数的一般形式如下：
``` cpp
template <typename type> ret-type func-name(parameter list)
{
   // 函数的主体
}
```
一个例子如下：
``` cpp
#include <iostream>
#include <string>
using namespace std;
template <typename T>
inline T const& Max (T const& a, T const& b) 
{ 
    return a < b ? b:a; 
} 
int main ()
{
    int i = 39;
    int j = 20;
    cout << "Max(i, j): " << Max(i, j) << endl; 
    double f1 = 13.5; 
    double f2 = 20.7; 
    cout << "Max(f1, f2): " << Max(f1, f2) << endl; 
    string s1 = "Hello"; 
    string s2 = "World"; 
    cout << "Max(s1, s2): " << Max(s1, s2) << endl; 
    return 0;
}
```

### 类模板[](https://www.runoob.com/cplusplus/cpp-templates.html)

泛型类声明的一般形式如下所示：
``` cpp
template <class type> class class-name {
. . . 
}
```
一个例子如下：
``` cpp
#include <iostream>
#include <vector>
#include <cstdlib>
#include <string>
#include <stdexcept>
using namespace std;
template <class T>
class Stack { 
  private: 
    vector<T> elems;     // 元素 
  public: 
    void push(T const&);  // 入栈
    void pop();               // 出栈
    T top() const;            // 返回栈顶元素
    bool empty() const{       // 如果为空则返回真。
        return elems.empty(); 
    } 
}; 
template <class T>
void Stack<T>::push (T const& elem) 
{ 
    // 追加传入元素的副本
    elems.push_back(elem);    
} 
template <class T>
void Stack<T>::pop () 
{ 
    if (elems.empty()) { 
        throw out_of_range("Stack<>::pop(): empty stack"); 
    }
    // 删除最后一个元素
    elems.pop_back();         
} 
template <class T>
T Stack<T>::top () const 
{ 
    if (elems.empty()) { 
        throw out_of_range("Stack<>::top(): empty stack"); 
    }
    // 返回最后一个元素的副本 
    return elems.back();      
} 
int main() 
{ 
    try { 
        Stack<int>         intStack;  // int 类型的栈 
        Stack<string> stringStack;    // string 类型的栈 
        // 操作 int 类型的栈 
        intStack.push(7); 
        cout << intStack.top() <<endl; 
        // 操作 string 类型的栈 
        stringStack.push("hello"); 
        cout << stringStack.top() << std::endl; 
        stringStack.pop(); 
        stringStack.pop(); 
    } 
    catch (exception const& ex) { 
        cerr << "Exception: " << ex.what() <<endl; 
        return -1;
    } 
}
```

### 标准模板库 STL

C++ 标准模板库（Standard Template Library，STL）是一套功能强大的 C++ 模板类和函数的集合，它提供了**一系列通用的、可复用的算法和数据结构**。
使用STL主要有以下好处：
- **代码复用**：STL 提供了大量的通用数据结构和算法，可以减少重复编写代码的工作。
- **性能优化**：STL 中的算法和数据结构都经过了优化，以提供最佳的性能。
- **泛型编程**：使用模板，STL 支持泛型编程，使得算法和数据结构可以适用于任何数据类型。
- **易于维护**：STL 的设计使得代码更加模块化，易于阅读和维护。
STL主要包含以下几个组件：

|组件|描述|
|---|---|
|容器（Containers）|容器是 STL 中最基本的组件之一，**提供了各种数据结构**，包括向量（vector）、链表（list）、队列（queue）、栈（stack）、集合（set）、映射（map）等。这些容器具有不同的特性和用途，可以根据实际需求选择合适的容器。|
|算法（Algorithms）|STL 提供了大量的算法，**用于对容器中的元素进行各种操作**，包括排序、搜索、复制、移动、变换等。这些算法在使用时不需要关心容器的具体类型，只需要指定要操作的范围即可。|
|迭代器（iterators）|迭代器用于遍历容器中的元素，**允许以统一的方式访问容器中的元素，而不用关心容器的内部实现细节**。STL 提供了多种类型的迭代器，包括随机访问迭代器、双向迭代器、前向迭代器和输入输出迭代器等。|
|函数对象（Function Objects）|函数对象是**可以像函数一样调用的对象，可以用于算法中的各种操作**。STL 提供了多种函数对象，包括一元函数对象、二元函数对象、谓词等，可以满足不同的需求。|
|适配器（Adapters）|适配器**用于将一种容器或迭代器适配成另一种容器或迭代器**，以满足特定的需求。STL 提供了多种适配器，包括栈适配器（stack adapter）、队列适配器（queue adapter）和优先队列适配器（priority queue adapter）等。|

---

## 多线程

多线程是多任务处理的一种特殊形式，**多任务处理允许让电脑同时运行两个或两个以上的程序**。
一般情况下，两种类型的多任务处理：**基于进程和基于线程**。
- 基于进程的多任务处理**是程序的并发执行**。
- 基于线程的多任务处理是**同一程序的片段的并发执行**。

### 基本概念

1. 线程 (Thread)
	- 多个线程可以**在同一个进程中独立运行**。
	- 线程**共享进程的地址空间、文件描述符、堆和全局变量等资源**，但每个线程**有自己的栈、寄存器和程序计数器**。
2. 并发 (Concurrency) 与并行 (Parallelism)
	- **并发**：多个任务**在时间片段内交替执行，表现出同时进行的效果**。
	- **并行**：多个任务**在多个处理器或处理器核上同时执行**。

### 创建线程

C++ 11 之后添加了新的标准线程库 std::thread，std::thread 在 `<thread>` 头文件中声明，因此使用 std::thread 时需要包含 在 `<thread>`头文件。
``` cpp
#include<thread>
std::thread thread_object(callable, args...);
```
主要有以下几种创建方法：
1. 使用函数指针创建线程
	``` cpp
	#include <iostream>
	#include <thread>
	void printMessage(int count) {
		for (int i = 0; i < count; ++i) {
			std::cout << "Hello from thread (function pointer)!\n";
		}
	}
	int main() {
		std::thread t1(printMessage, 5); // 创建线程，传递函数指针和参数
		t1.join(); // 等待线程完成
		return 0;
	}
	```
2. 使用函数对象
通过类中的[[CPP——面向对象#函数调用运算符|operator()方法]]方法定义函数对象来创建线程：
	``` cpp
	#include <iostream>
	#include <thread>
	class PrintTask {
	public:
		void operator()(int count) const {
			for (int i = 0; i < count; ++i) {
				std::cout << "Hello from thread (function object)!\n";
			}
		}
	};
	int main() {
		std::thread t2(PrintTask(), 5); // 创建线程，传递函数对象和参数
		t2.join(); // 等待线程完成
		return 0;
	}
	```
3. 使用 [[CPP——基础#Lambda 函数与表达式|Lambda]] 表达式
Lambda 表达式可以直接内联定义线程执行的代码：
	``` cpp
	#include <iostream>
	#include <thread>
	int main() {
	    std::thread t3([](int count) {
	        for (int i = 0; i < count; ++i) {
	            std::cout << "Hello from thread (lambda)!\n";
	        }
	    }, 5); // 创建线程，传递 Lambda 表达式和参数
	    t3.join(); // 等待线程完成
	    return 0;
	}
	```

### 线程管理

1. `t.join()` 用于**等待线程完成执行**。如果不调用 join() 或 detach() 而直接销毁线程对象，会导致程序崩溃。
2. `t.detach()` 将线程与主线程分离，**线程在后台独立运行，主线程不再等待它**。

### 线程的传参

1. 参数可以通过值传递给线程 `std::thread t(func, arg1, arg2);`
2. 引用传递:
	``` cpp
	#include <iostream>
	#include <thread>
	void increment(int& x) {
	    ++x;
	}
	int main() {
	    int num = 0;
	    std::thread t(increment, std::ref(num)); // 使用 std::ref 传递引用
	    t.join();
	    std::cout << "Value after increment: " << num << std::endl;
	    return 0;
	}
	```

### 线程综合案例

``` cpp
#include <iostream>
#include <thread>
using namespace std;
// 一个简单的函数，作为线程的入口函数
void foo(int Z) {
    for (int i = 0; i < Z; i++) {
        cout << "线程使用函数指针作为可调用参数\n";
    }
}
// 可调用对象的类定义
class ThreadObj {
public:
    void operator()(int x) const {
        for (int i = 0; i < x; i++) {
            cout << "线程使用函数对象作为可调用参数\n";
        }
    }
};
int main() {
    cout << "线程 1 、2 、3 独立运行" << endl;
    // 使用函数指针创建线程
    thread th1(foo, 3);
    // 使用函数对象创建线程
    thread th2(ThreadObj(), 3);
    // 使用 Lambda 表达式创建线程
    thread th3([](int x) {
        for (int i = 0; i < x; i++) {
            cout << "线程使用 lambda 表达式作为可调用参数\n";
        }
    }, 3);
    // 等待所有线程完成
    th1.join(); // 等待线程 th1 完成
    th2.join(); // 等待线程 th2 完成
    th3.join(); // 等待线程 th3 完成
    return 0;
}
```

### 线程同步与互斥

1. 互斥量
互斥量是一种同步原语，**用于防止多个线程同时访问共享资源**。当一个线程需要访问共享资源时，它**首先需要锁定（lock）互斥量**。如果互斥量已经被其他线程锁定，那么**请求锁定的线程将被阻塞，直到互斥量被解锁（unlock）**。
一个例子如下：
	``` cpp
	#include <mutex>
	#include <thread>
	std::mutex mtx; // 全局互斥量
	void safeFunction() {
	    mtx.lock(); // 请求锁定互斥量
	    // 访问或修改共享资源
	    mtx.unlock(); // 释放互斥量
	}
	int main() {
	    std::thread t1(safeFunction);
	    std::thread t2(safeFunction);
	    t1.join();
	    t2.join();
	    return 0;
	}
	```

2. 锁（Locks）
C++提供了多种锁类型，用于**简化互斥量的使用和管理**。常见的锁类型如下：
	- std::lock_guard：作用域锁，当**构造时自动锁定互斥量，当析构时自动解锁**。
	- std::unique_lock：与std::lock_guard类似，但提供了更多的灵活性，例如**可以转移所有权和手动解锁**。

	一个实例如下：
	``` cpp
	#include <mutex>
	std::mutex mtx;
	void safeFunctionWithLockGuard() {
		std::lock_guard<std::mutex> lk(mtx);
		// 访问或修改共享资源
	}
	void safeFunctionWithUniqueLock() {
		std::unique_lock<std::mutex> ul(mtx);
		// 访问或修改共享资源
		// ul.unlock(); // 可选：手动解锁
		// ...
	}
	```

3. 条件变量（Condition Variable）
条件变量**用于线程间的协调**，**允许一个或多个线程等待某个条件的发生**。它通常与互斥量一起使用，以实现线程间的同步。`std::condition_variable`用于实现线程间的等待和通知机制。
一个实例如下：
	``` cpp
	#include <mutex>
	#include <condition_variable>
	std::mutex mtx;
	std::condition_variable cv;
	bool ready = false;
	void workerThread() {
	    std::unique_lock<std::mutex> lk(mtx);
	    cv.wait(lk, []{ return ready; }); // 等待条件
	    // 当条件满足时执行工作
	}
	void mainThread() {
	    {
	        std::lock_guard<std::mutex> lk(mtx);
	        // 准备数据
	        ready = true;
	    } // 离开作用域时解锁
	    cv.notify_one(); // 通知一个等待的线程
	}
	```

4. 原子操作（Atomic Operations）
原子操作确保对共享数据的访问是不可分割的，即在多线程环境下，**原子操作要么完全执行，要么完全不执行，不会出现中间状态**。
	``` cpp
	#include <atomic>
	#include <thread>
	std::atomic<int> count(0);
	void increment() {
	    count.fetch_add(1, std::memory_order_relaxed);
	}
	int main() {
	    std::thread t1(increment);
	    std::thread t2(increment);
	    t1.join();
	    t2.join();
	    return count; // 应返回2
	}
	```

5. 线程局部存储（Thread Local Storage, TLS）
线程局部存储**允许每个线程拥有自己的数据副本**。这可以通过thread_local关键字实现，**避免了对共享资源的争用**。
	``` cpp
	#include <iostream>
	#include <thread>
	thread_local int threadData = 0;
	void threadFunction() {
	    threadData = 42; // 每个线程都有自己的threadData副本
	    std::cout << "Thread data: " << threadData << std::endl;
	}
	int main() {
	    std::thread t1(threadFunction);
	    std::thread t2(threadFunction);
	    t1.join();
	    t2.join();
	    return 0;
	}
	```

6. 死锁（Deadlock）和避免策略
死锁发生在**多个线程互相等待对方释放资源，但没有一个线程能够继续执行**。避免死锁的策略包括：
	- 总是以相同的顺序请求资源。
	- 使用超时来尝试获取资源。
	- 使用死锁检测算法。

### 线程间通信

std::future 和 std::promise：实现线程间的值传递。

``` cpp
std::promise<int> p;
std::future<int> f = p.get_future();
std::thread t([&p] {
	p.set_value(10); // 设置值，触发 future
});
int result = f.get(); // 获取值
```

---


