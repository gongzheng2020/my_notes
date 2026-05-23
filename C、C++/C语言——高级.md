
---

## 数组

存储一个固定大小的相同类型元素的顺序集合

动态数组是在运行时通过动态内存分配函数（如 **malloc** 和 **calloc**）手动分配内存的数组：
- 内存分配：动态数组的内存空间在运行时**通过动态内存分配函数手动分配**，并存储在堆上。需要使用 `malloc`、`calloc` 等函数来申请内存，并使用 `free` 函数来释放内存。
- 大小可变：动态数组的大小**在运行时可以根据需要进行调整**。可以使用 `realloc` 函数来重新分配内存，并改变数组的大小。
- 生命周期：动态数组的生命周期由程序员控制。需要***在使用完数组后手动释放内存**，以避免内存泄漏。

``` c
int size = 5;
int *dynamicArray = (int *)malloc(size * sizeof(int)); // 动态数组内存分配
// 使用动态数组
free(dynamicArray); // 动态数组内存释放
```

---

## 枚举

用于定义一组具有离散值的常量，格式为`enum　枚举名　{枚举元素1,枚举元素2,……};`，例如：

``` c
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
enum season {spring, summer=3, autumn, winter}; //spring = 0
```

---

## 指针

每一个变量都有一个内存位置，每一个内存位置都定义了可使用 & 运算符访问的地址，它表示了在内存中的一个地址。

``` c
int    *ip;    /* 一个整型的指针 */
double *dp;    /* 一个 double 型的指针 */
float  *fp;    /* 一个浮点型的指针 */
char   *ch;    /* 一个字符型的指针 */
```

- **指针数组**存储了一组指针，每个指针可以指向不同的数据对象。

``` c
#include <stdio.h>
const int MAX = 4;
int main ()
{
   const char *names[] = {
                   "Zara Ali",
                   "Hina Ali",
                   "Nuha Ali",
                   "Sara Ali",
   };
   int i = 0;
 
   for ( i = 0; i < MAX; i++)
   {
      printf("Value of names[%d] = %s\n", i, names[i] );
   }
   return 0;
}
```

- **指向指针的指针**是一种多级间接寻址的形式，或者说是一个指针链。通常，一个指针包含一个变量的地址。当我们定义一个指向指针的指针时，**第一个指针包含了第二个指针的地址，第二个指针指向包含实际值的位置**。

``` c
#include <stdio.h>
int main ()
{
   int  V;
   int  *Pt1;
   int  **Pt2;
   V = 100;
   /* 获取 V 的地址 */
   Pt1 = &V;
   /* 使用运算符 & 获取 Pt1 的地址 */
   Pt2 = &Pt1;
   /* 使用 pptr 获取值 */
   printf("var = %d\n", V );
   printf("Pt1 = %p\n", Pt1 );
   printf("*Pt1 = %d\n", *Pt1 );
   printf("Pt2 = %p\n", Pt2 );
   printf("**Pt2 = %d\n", **Pt2);
   return 0;
}
```

产生的结果为：

``` c
var = 100
Pt1 = 0x7ffee2d5e8d8
*Pt1 = 100
Pt2 = 0x7ffee2d5e8d0
**Pt2 = 100
```

- **函数指针**是指向函数的指针变量，可以像一般函数一样，用于调用函数、传递参数。

``` c
#include <stdio.h>
int max(int x, int y)
{
    return x > y ? x : y;
}
int main(void)
{
    /* p 是函数指针 */
    int (* p)(int, int) = & max; // &可以省略
    int a, b, c, d;
    printf("请输入三个数字:");
    scanf("%d %d %d", & a, & b, & c);
    /* 与直接调用函数等价，d = max(max(a, b), c) */
    d = p(p(a, b), c); 
    printf("最大的数字是: %d\n", d);
    return 0;
}
```

- **函数指针作为某个函数的参数**，回调函数是由别人的函数执行时调用你实现的函数。

> 以下是来自知乎作者常溪玲的解说：
你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。在这个例子里，你的**电话号码就叫回调函数**，你把**电话留给店员就叫登记回调函数**，店里后来**有货了叫做触发了回调关联的事件**，店员给你**打电话叫做调用回调函数**，你到店里去**取货叫做响应回调事件**。

``` c
#include <stdlib.h>  
#include <stdio.h>
void populate_array(int *array, size_t arraySize, int (*getNextValue)(void))
{
    for (size_t i=0; i<arraySize; i++)
        array[i] = getNextValue();
}
// 获取随机值
int getNextRandomValue(void)
{
    return rand();
}
int main(void)
{
    int myarray[10];
    /* getNextRandomValue 不能加括号，否则无法编译，因为加上括号之后相当于传入此参数时传入了 int , 而不是函数指针*/
    populate_array(myarray, 10, getNextRandomValue);
    for(int i = 0; i < 10; i++) {
        printf("%d ", myarray[i]);
    }
    printf("\n");
    return 0;
}
```

---

## 字符串

**字符串**实际上是使用空字符 `\0` 结尾的一维字符数组。

``` c
char site[7] = {'R', 'U', 'N', 'O', 'O', 'B', '\0'};
char site[] = "RUNOOB";
```

---

## 结构体

**结构**是 C 编程中另一种用户自定义的可用的数据类型，它允许您存储不同类型的数据项

>https://www.runoob.com/cprogramming/c-structures.html

``` c
struct tag { 
    member-list
    member-list 
    member-list  
    ...
} variable-list ;
```

``` c
struct 
{
    int a;
    char b;
    double c;
} s1;
struct SIMPLE
{
    int a;
    char b;
    double c;
};
struct SIMPLE t1, t2[20], *t3;
typedef struct
{
    int a;
    char b;
    double c; 
} Simple2;
Simple2 u1, u2[20], *u3;
```

---

## 共用体

**共用体**是一种特殊的数据类型，允许您**在相同的内存位置存储不同的数据类型**

``` c
union [union tag]
{
   member definition;
   member definition;
   ...
   member definition;
} [one or more union variables];

union Data
{
   int i;
   float f;
   char  str[20];
} data;
```
