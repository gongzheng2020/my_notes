
---

## 基本语法

C++ 程序可以定义为**对象的集合**，这些对象通过**调用彼此的方法进行交互**。现在让我们简要地看一下什么是类、对象，方法、即时变量。

- **对象 -** 对象**具有状态和行为**。例如：一只狗的状态 - 颜色、名称、品种，行为 - 摇动、叫唤、吃。对象是类的实例。
- **类 -** 类可以定义为描述**对象行为/状态的模板/蓝图**。
- **方法 -** 从基本上说，**一个方法表示一种行为。一个类可以包含多个方法**。可以在方法中写入逻辑、操作数据以及执行所有的动作。
- **即时变量 -** 每个对象**都有其独特的即时变量**。对象的**状态是由这些即时变量的值创建的**。

``` c
#include <iostream>
using namespace std;
// main() 是程序开始执行的地方
int main()
{
   cout << "Hello World"; // 输出 Hello World
   return 0;
}
```

---

## Vector容器

`vector` 本质上是一个**可自动扩容的动态数组**，与传统数组相比，**vector 不需要手动管理内存，可以根据元素数量自动扩容**。

- 使用 Vector，包含其头文件`#include <vector>`即可。

- 创建 Vector  
	- 创建一个空 vector：`std::vector<int>vec;`
	- 指定初始大小：`std::vector<int>vec(5);` (创建 5 个元素，默认值为 0。)
	- 指定初始值：`std::vector<int>vec(5,10)`，`std::vector<int>vec = {0, 1, 2, 3}`

- 添加元素：
	- 使用 `push_back()` 向尾部添加元素：`vec.push_back(100);`
	- 使用`emplace_back()`：`vec.emplace_back("Tom", 20);`（ Vector 内部原地构造）

- 访问元素
	- 使用下标访问：`int x = vec[0];` （不进行越界检查，速度快）
	- 使用 at 访问：`int y = vec.at(1);` （进行越界检查）

- 获取大小：`vec.size();`
	>**capacity 通常大于等于 size**。这是因为 vector 会提前申请更多空间，以**减少频繁扩容**。
    >- **size：**当前元素数量。
    >- **capacity：**当前已分配的内存容量。

- 预分配空间：为了避免频繁扩容，可以提前分配内存。
	``` cpp
	std::vector<int>vec;
	vec.reserve(1000000);
	```

- 遍历 Vector
	- 使用下标：`vec[i]`
	- 使用迭代器：`for (auto it = vec.begin(); it != vec.end(); ++it)`
	- 范围 for 循环：
		``` cpp
		for (int element : vec) {
		    std::cout << element << " ";
		}
		```

- 删除元素：`vec.erase(vec.begin() + 2);`

- 清空元素：`vec.clear();`
>`clear()`只会让size变为0，但capacity 可能仍然保留
>释放内存使用：
>- `std::vector<int>().swap(vec);`
>- `vec.shrink_to_fit();`

- Vector 与数组的区别

	| 特性 | 数组 | vector |
	| :---: | :---: | :---: |
	|大小固定|是|否|
	|自动扩容|否|是|
	|连续内存|是|是|
	|随机访问|快|快|
	|STL 支持|较少|完整|
	|安全访问|无|at()|

---

## 数据结构

1. 数组
数组是最基础的数据结构，用于**存储一组相同类型的数据**。
	``` cpp
	int arr[5] = {1, 2, 3, 4, 5};
cout << arr[0]; // 输出第一个元素
	```

2. 结构体
结构体**允许将不同类型的数据组合在一起**，形成一种自定义的数据。
	``` cpp
	struct Person {
	    string name;
	    int age;
	};
	Person p = {"Alice", 25};
	cout << p.name << endl; // 输出 Alice
	```

3. 类
类是 C++ 中用于面向对象编程的核心结构，**允许定义成员变量和成员函数**。
	``` cpp
	class Person {
	private:
	    string name;
	    int age;
	public:
	    Person(string n, int a) : name(n), age(a) {}
	    void printInfo() {
	        cout << "Name: " << name << ", Age: " << age << endl;
	    }
	};
	Person p("Bob", 30);
	p.printInfo(); // 输出: Name: Bob, Age: 30
	```

4. 链表
链表是一种动态数据结构，由一系列节点组成，**每个节点包含数据和指向下一个节点的指针**。
	>不需要提前定义容量、插入删除复杂度为O(1)、查找复杂度为O(n)。
	``` cpp
	struct Node {
	    int data;
	    Node* next;
	};
	Node* head = nullptr;
	Node* newNode = new Node{10, nullptr};
	head = newNode; // 插入新节点
	```

5. 栈（Stack）
栈是一种**后进先出**（LIFO, Last In First Out）的数据结构。
	``` cpp
	stack<int> s;
	s.push(1);
	s.push(2);
	cout << s.top(); // 输出 2
	s.pop();
	```

6. 队列（Queue）
队列是一种**先进先出**（FIFO, First In First Out）的数据结构。
	``` cpp
	queue<int> q;
	q.push(1);
	q.push(2);
	cout << q.front(); // 输出 1
	q.pop();
	```

7. 双端队列（Deque）
双端队列**允许在两端进行插入和删除操作**，是栈和队列的结合体。
	``` cpp
	deque<int> dq;
	dq.push_back(1);
	dq.push_front(2);
	cout << dq.front(); // 输出 2
	dq.pop_front();
	```

8. 哈希表（Hash Table）
哈希表是一种**通过键值对存储数据的数据结构**，支持快速查找、插入和删除操作。
	``` cpp
	unordered_map<string, int> hashTable;
	hashTable["apple"] = 10;
	cout << hashTable["apple"]; // 输出 10
	```

9. 映射（Map）
map 是**一种有序的键值对容器**，底层实现是红黑树。
	``` cpp
	map<string, int> myMap;
	myMap["apple"] = 10;
	cout << myMap["apple"]; // 输出 10
	```

10. 集合（Set）
set 是一种**用于存储唯一元素的有序集合**，底层同样使用红黑树实现。它**保证元素不重复且有序**。
	>元素**唯一**且**自动按照升序排列**。
	``` cpp
	set<int> s;
	s.insert(1);
	s.insert(2);
	cout << *s.begin(); // 输出 1
	```

11. [动态数组（Vector）](#Vector容器)