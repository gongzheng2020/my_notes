
---

## 类定义

定义一个类，本质上是定义一个数据类型的蓝图，它定义了**类的对象包括了什么**，以及可以在这个**对象上执行哪些操作**。

![[Pasted image 20260524161945.png]]

---

## 类成员函数

类的成员函数是指那些**把定义和原型写在类定义内部的函数**，就像类定义中的其他变量一样。类成员函数是类的一个成员，它**可以操作类的任意对象，可以访问对象中的所有成员**。
``` cpp
class Box
{
   public:
      double length;         // 长度
      double breadth;        // 宽度
      double height;         // 高度
      double getVolume(void);// 返回体积
};
```
针对上述类中声明的`getVolume`函数，可使用的定义方式如下：
- 定义在类定义内部
	``` cpp
	class Box
	{
	   public:
		...
	      double getVolume(void)
	      {
	         return length * breadth * height;
	      }
	};
	```
- 使用**范围解析运算符 ::**
	``` cpp
	double Box::getVolume(void)
	{
	    return length * breadth * height;
	}
	```
>在 :: 运算符之前必须使用类名

---

## 类访问修饰符

在 C++ 面向对象编程（OOP）中，数据封装（Encapsulation） 是核心概念之一。简单来说，**就是把数据"藏"起来，只开放必要的接口给外界使用，以保证数据的安全**。

|修饰符|现实类比|谁可以访问？|典型用途|
|---|---|---|---|
|**public**|**客厅/大门**|**所有人**（类内部、子类、外部代码）|**对外提供**的接口函数（API）。|
|**protected**|**卧室**|**你和你的孩子**（类内部、子类）|**仅限家族内部**（继承体系）使用的数据。|
|**private**|**保险箱**|**只有你自己**（仅限本类内部）|核心数据、**不想被随意修改的变量**。|

1. 公有成员
在程序中**任何地方都能访问**，无需通过成员函数读写。

2. 私有成员
只有**类自身的成员函数**与**被授予友元权限的实体**能够操作这些内容。
> **若未使用任何访问说明符，成员默认为私有**

3. protected（受保护）成员
没有继承，它和 `private` 一样；有继承，**子类（派生类）可以访问父类的 `protected` 成员**，但不能访问 `private` 成员

4. 派生类中的继承权变化
当你定义一个子类时（如 `class B : public A`），冒号后面的 `public` 也是一种访问修饰符，它决定了**父类的成员在子类中"表现"成什么样**。
>**继承方式决定了父类成员在子类中的最高权限。**  
>如果继承方式比成员原本的权限更严格，成员的权限就会被"降级"到继承方式的级别。

---

## 构造与析构函数

### 构造函数

类的**构造函数**是类的一种特殊的成员函数，它会在**每次创建类的新对象时自动执行**。
构造函数的**名称与类的名称完全相同，且没有返回类型**（连 `void` 也不能有）。构造函数通常用于**为成员变量设置初始值**。

1. 无参数构造函数
``` cpp
class Line { 
public: 
	Line(); // 构造函数声明 
	void setLength(double len); 
	double getLength() const; 
private: 
	double length; 
};
Line::Line() { 
	cout << "Object is being created" << endl; 
	length = 0.0; // 显式初始化成员变量 
}
```

2. 带参数的构造函数
``` cpp
Line::Line(double len) {
	cout << "Object is being created, length = " << len << endl;
	length = len; 
}
```

3. 初始化列表来初始化字段
``` cpp
C::C(double a, double b, double c) : X(a), Y(b), Z(c) { 
	// 其他初始化逻辑（若有）
}
```

### 析构函数

类的**析构函数**是类的一种特殊的成员函数，在对象的生命周期结束时（离开作用域或被 `delete`）自动执行，用于**释放对象占用的资源（如动态内存、文件句柄等）**。
析构函数的名称与类的名称完全相同，**只是在前面加了个波浪号（`~`）作为前缀**。它**不会返回任何值，也不能带有任何参数，每个类只能有一个析构函数**。

### 拷贝构造函数

拷贝构造函数是一种特殊的构造函数，它在创建对象时，是使用**同一类中之前创建的对象来初始化新创建的对象**。常用于：
- 使用另一个同类型的对象来**初始化新创建的对象**。
- 复制对象把它**作为参数传递给函数**。
- 复制对象，并从**函数返回这个对象**。

常见拷贝构造函数如下：
``` cpp
classname (const classname &obj) {
   // 构造函数的主体
}
```
一个例子如下：
``` cpp
#include <iostream>

using namespace std;
 
class Line
{
   public:
      int getLength( void );
      Line( int len );             // 简单的构造函数
      Line( const Line &obj);      // 拷贝构造函数
      ~Line();                     // 析构函数
   private:
      int *ptr;
};
// 成员函数定义，包括构造函数
Line::Line(int len)
{
    cout << "调用构造函数" << endl;
    // 为指针分配内存
    ptr = new int;
    *ptr = len;
}
Line::Line(const Line &obj)
{
    cout << "调用拷贝构造函数并为指针 ptr 分配内存" << endl;
    ptr = new int;
    *ptr = *obj.ptr; // 拷贝值
}
void display(Line obj)
{
   cout << "line 大小 : " << obj.getLength() <<endl;
}
// 程序的主函数
int main( )
{
   Line line1(10);
   Line line2 = line1; // 这里也调用了拷贝构造函数
   display(line1);
   display(line2);
   return 0;
}
```
>参数加上`const`表示无法对传入对象进行修改。

---

## 友元函数

类的友元函数是**定义在类外部**，但**有权访问类的所有私有（private）成员和保护（protected）成员**。尽管友元函数的原型有在类的定义中出现过，但是**友元函数并不是成员函数**。
如果要声明函数为一个类的友元，需要在类定义中该函数原型前使用关键字 **friend**：
``` cpp
class Box
{
   double width;
public:
   double length;
   friend void printWidth( Box box );
   void setWidth( double wid );
};
```
实例程序如下：
``` cpp
#include <iostream>
using namespace std;
class Box
{
   double width;
public:
   friend void printWidth( Box box );
   void setWidth( double wid );
};
...
// 请注意：printWidth() 不是任何类的成员函数
void printWidth( Box box )
{
   /* 因为 printWidth() 是 Box 的友元，它可以直接访问该类的任何成员 */
   cout << "Width of box : " << box.width <<endl;
}
// 程序的主函数
int main( )
{
   Box box;
   // 使用成员函数设置宽度
   box.setWidth(10.0);
   // 使用友元函数输出宽度
   printWidth( box );
   return 0;
}
```

---

## 内联函数

在函数前加 `inline`，**建议**编译器在调用点**直接展开函数体，替代传统函数调用**。消除短小高频函数的**调用开销**（压栈、跳转、返回）。
一个例子如下：
``` cpp
inline double square(double x) { return x * x; }
```

---

## 指向类的指针

访问**指向类的指针**的成员，需要使用**成员访问运算符 ->**，就像访问指向结构的指针一样。
> `->` 等于`(*ptr).member`
``` cpp
#include <iostream>
class MyClass {
public:
    int data;
    void display() {
        std::cout << "Data: " << data << std::endl;
    }
};
int main() {
    // 创建类对象
    MyClass obj;
    obj.data = 42;
    // 声明和初始化指向类的指针
    MyClass *ptr = &obj;
    // 通过指针访问成员变量
    std::cout << "Data via pointer: " << ptr->data << std::endl;
    // 通过指针调用成员函数
    ptr->display();
    return 0;
}
```

---

## 静态成员

使用 **static** 关键字来把类成员定义为静态的。当我们声明类的成员为静态时，这意味着**无论创建多少个类的对象，静态成员都只有一个副本**。
>使用**范围解析运算符 ::** 对类中的静态变量或函数进行访问；
>**静态成员函数与普通成员函数的区别：**
>- 静态成员函数**没有 this 指针，只能访问静态成员**（包括静态成员变量和静态成员函数）。
>- 普通成员函数**有 this 指针，可以访问类中的任意成员**。

``` cpp
#include <iostream>
using namespace std;
class Box
{
   public:
      static int objectCount;
      // 构造函数定义
      Box(double l=2.0, double b=2.0, double h=2.0)
      {
         cout <<"Constructor called." << endl;
         length = l;
         breadth = b;
         height = h;
         // 每次创建对象时增加 1
         objectCount++;
      }
      double Volume()
      {
         return length * breadth * height;
      }
      static int getCount()
      {
         return objectCount;
      }
   private:
      double length;     // 长度
      double breadth;    // 宽度
      double height;     // 高度
};
// 初始化类 Box 的静态成员
int Box::objectCount = 0;
int main(void)
{
   // 在创建对象之前输出对象的总数
   cout << "Inital Stage Count: " << Box::getCount() << endl;
   Box Box1(3.3, 1.2, 1.5);    // 声明 box1
   Box Box2(8.5, 6.0, 2.0);    // 声明 box2
   // 在创建对象之后输出对象的总数
   cout << "Final Stage Count: " << Box::getCount() << endl;
   return 0;
}
```

---

## 函数调用运算符

`operator()` 是 C++ 中的函数调用运算符，**重载它后，类的对象可以像函数一样被调用**。
这样的对象称为**函数对象**（Function Object）或**函子**（Functor）。
一个例子如下：
``` cpp
#include <iostream>
class MyFunctor {
public:
    int multiplier;
    // 构造函数
    MyFunctor(int m) : multiplier(m) {}
    // 重载 operator()，使对象可像函数一样调用
    int operator()(int x) {
        return x * multiplier;
    }
};
int main() {
    // 创建函数对象
    MyFunctor obj(3);
    // 像函数一样调用对象
    int result = obj(5);  // 等价于 obj.operator()(5)    
    std::cout << "Result: " << result << std::endl;  // 输出 15 (5 * 3)
    // 可以多次调用，携带状态
    std::cout << "obj(10): " << obj(10) << std::endl;  // 输出 30
    std::cout << "obj(7): " << obj(7) << std::endl;    // 输出 21
    return 0;
}
```

---

## 继承

当创建一个与已有类有继承关系的类时，您不需要重新编写新的数据成员和成员函数，只需指定新建的类继承了一个已有的类的成员即可。这个已有的类称为**基类**，新建的类称为**派生类**。
![[Pasted image 20260531213049.png]]
``` cpp
// 基类
class Animal {
    // eat() 函数
    // sleep() 函数
};
//派生类
class Dog : public Animal {
    // bark() 函数
};
```
类派生列表以一个或多个基类命名，形式如下：
``` cpp
class derived-class: access-specifier base-class
```
访问修饰符 access-specifier 是 **public、protected** 或 **private** 其中的一个，base-class 是之前定义过的某个类的名称。如果未使用访问修饰符 access-specifier，则默认为 private。
>一个派生类继承了所有的基类方法，但下列情况除外：
>- 基类的**构造函数、析构函数和拷贝构造函数**。
>- 基类的**重载运算符**。
>- 基类的**友元函数**。

---

## 重载运算符和重载函数

C++ 允许在同一作用域中的某个**函数**和**运算符**指定多个定义，分别称为**函数重载**和**运算符重载**。
重载声明是指一个与之前已经在该作用域内声明过的函数或方法具有相同名称的声明，但是**它们的参数列表和定义（实现）不相同**。
当您调用一个**重载函数**或**重载运算符**时，编译器通过把您所使用的参数类型与定义中的参数类型进行比较，决定选用最合适的定义。选择最合适的重载函数或重载运算符的过程，称为**重载决策**。

### 函数重载

同一个作用域内，可以声明几个功能类似的同名函数，但是**这些同名函数的形式参数（指参数的个数、类型或者顺序）必须不同**。您**不能仅通过返回类型的不同来重载函数**。
一个函数重载的例子如下：
``` cpp
#include <iostream>
using namespace std;
class printData
{
   public:
      void print(int i) {
        cout << "整数为: " << i << endl;
      }
      void print(double  f) {
        cout << "浮点数为: " << f << endl;
      }
      void print(char c[]) {
        cout << "字符串为: " << c << endl;
      }
};
int main(void)
{
   printData pd;
   // 输出整数
   pd.print(5);
   // 输出浮点数
   pd.print(500.263);
   // 输出字符串
   char c[] = "Hello C++";
   pd.print(c);
   return 0;
}
```

### 运算符重载

重载的运算符是**带有特殊名称的函数**，函数名是由关键字 operator 和其后要重载的运算符符号构成的。与其他函数一样，**重载运算符有一个返回类型和一个参数列表**。
>可以被定义为非成员或类成员函数

``` cpp
Box operator+(const Box&); //类成员函数
Box operator+(const Box&, const Box&); //非成员函数（需要传递两个参数）
```
一个例子如下（对象作为参数进行传递，对象的属性使用 **this** 运算符进行访问）：
``` cpp
#include <iostream>
using namespace std;
class Box
{
   public:
      double getVolume(void)
      {
         return length * breadth * height;
      }
      void setLength( double len )
      {
          length = len;
      }
      void setBreadth( double bre )
      {
          breadth = bre;
      }
      void setHeight( double hei )
      {
          height = hei;
      }
      // 重载 + 运算符，用于把两个 Box 对象相加
      Box operator+(const Box& b)
      {
         Box box;
         box.length = this->length + b.length;
         box.breadth = this->breadth + b.breadth;
         box.height = this->height + b.height;
         return box;
      }
   private:
      double length;      // 长度
      double breadth;     // 宽度
      double height;      // 高度
};
// 程序的主函数
int main( )
{
   Box Box1;                // 声明 Box1，类型为 Box
   Box Box2;                // 声明 Box2，类型为 Box
   Box Box3;                // 声明 Box3，类型为 Box
   double volume = 0.0;     // 把体积存储在该变量中
   // Box1 详述
   Box1.setLength(6.0); 
   Box1.setBreadth(7.0); 
   Box1.setHeight(5.0);
   // Box2 详述
   Box2.setLength(12.0); 
   Box2.setBreadth(13.0); 
   Box2.setHeight(10.0);
   // Box1 的体积
   volume = Box1.getVolume();
   cout << "Volume of Box1 : " << volume <<endl;
   // Box2 的体积
   volume = Box2.getVolume();
   cout << "Volume of Box2 : " << volume <<endl;
   // 把两个对象相加，得到 Box3
   Box3 = Box1 + Box2;
   // Box3 的体积
   volume = Box3.getVolume();
   cout << "Volume of Box3 : " << volume <<endl;
   return 0;
}
```

---

## 多态

C++ 多态**允许使用基类指针或引用来调用子类的重写方法**，从而使得**同一接口可以表现不同的行为**。
下面是几个重要概念：
1. **虚函数(virtual)允许派生类重写**，通过基类指针/引用调用时触发动态绑定（运行时根据实际类型决策）；
2. 纯虚函数(=0)定义抽象类并**强制子类实现**；
3. 底层通过对象内的**V-Ptr指向存储函数指针的V-Table实现多态**。

### 虚函数

**虚函数**是在基类中使用关键字 **virtual** 声明的函数。
**虚函数**允许子类重写它，从而**在运行时通过基类指针或引用调用子类的重写版本，实现动态绑定**。
下面是一个例子：
``` cpp
#include <iostream>
using namespace std;
class Animal {
public:
    virtual void sound() {  // 虚函数
        cout << "Animal makes a sound" << endl;
    }
};
class Dog : public Animal {
public:
    void sound() override {  // 重写虚函数
        cout << "Dog barks" << endl;
    }
};
int main() {
    Animal *animal = new Dog();
    animal->sound();  // 输出: Dog barks
    delete animal;
}
```

### 纯虚函数

纯虚函数是没有实现的虚函数，**在基类中用 = 0 来声明**，子类**必须重写**，且必须通过**派生类实现所有纯虚函数才能创建对象**
``` cpp
#include <iostream>
using namespace std;
class Shape {
public:
    virtual int area() = 0;  // 纯虚函数，强制子类实现此方法
};
class Rectangle : public Shape {
private:
    int width, height;
public:
    Rectangle(int w, int h) : width(w), height(h) { }
    int area() override {  // 实现纯虚函数
        return width * height;
    }
}; 
int main() {
    Shape *shape = new Rectangle(10, 5);
    cout << "Rectangle Area: " << shape->area() << endl;  // 输出: Rectangle Area: 50
    delete shape;
}
```

---

## 数据抽象[](https://www.runoob.com/cplusplus/cpp-data-abstraction.html)

C++ 类为**数据抽象**提供了可能。它们向外界提供了大量用于操作对象数据的公共方法，也就是说，外界实际上并不清楚类的内部实现。
例如，您的程序可以调用 **sort()** 函数，而不需要知道函数中排序数据所用到的算法。实际上，函数排序的底层实现会因库的版本不同而有所差异，只要接口不变，函数调用就可以照常工作。
>**设计策略**：抽象把代码分离为接口和实现。所以在设计组件时，必须保持接口独立于实现，这样，如果改变底层实现，接口也将保持不变。在这种情况下，不管任何程序使用接口，接口都不会受到影响，只需要将最新的实现重新编译即可。

---

## 数据封装[](https://www.runoob.com/cplusplus/cpp-data-encapsulation.html)

数据封装（Data Encapsulation）是面向对象编程（OOP）的一个基本概念，它通过将数据和操作数据的函数封装在一个类中来实现。这种封装确保了数据的私有性和完整性，防止了外部代码对其直接访问和修改。
>**设计策略**：通常情况下，我们都会设置类成员状态为私有（private），除非我们真的需要将其暴露，这样才能保证良好的**封装性**。这通常应用于数据成员，但它同样适用于所有成员，包括虚函数。

---

## 接口（抽象类）[](https://www.runoob.com/cplusplus/cpp-interfaces.html)

接口描述了类的行为和功能，而不需要完成类的特定实现。**如果类中至少有一个函数被声明为纯虚函数，则这个类就是抽象类。**
>**设计策略**：面向对象的系统可能会**使用一个抽象基类为所有的外部应用程序提供一个适当的、通用的、标准化的接口**。然后，派生类通过继承抽象基类，就把所有类似的操作都继承下来。外部应用程序提供的功能（即公有函数）在抽象基类中是以纯虚函数的形式存在的。这些纯虚函数在相应的派生类中被实现。这个架构也使得新的应用程序可以很容易地被添加到系统中，即使是在系统被定义之后依然可以如此。

---




