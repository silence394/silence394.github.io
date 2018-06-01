---
title: define与typedef
categories: C++
---

在C++语言中，define和typedef虽然有相似的地方，但是有着各自的用途，做一下简单的总结。

## 一、define和typedef的用法
### 1、define的基本用法
#### 常量宏定义
``` C++
#define PI 3.1415926f
float pi2 = PI * 2.0f;
```
在编译器进行预处理的时候，进行简单的文本替换，将PI替换成3.1415926f。
#### 类型定义
``` C++
#define uint unsigned int
uint a = 1;
```
在预处理的时候提问宏定义，变成了
``` C++
unsigned int a = 1;
```
#### 函数定义
define定义宏像函数一样接收参数：
``` C++
#define add(a, b) a + b
int c = add(1, 2)
```
这个宏的功能是讲两个参数相加，由于是简单的文本替换，碰到下面的情况就会出问题:
``` C++
int c = 2 * add(1 + 2) * 2;
```
一般我们想要的结果是 2 * 3 * 2 = 12,但是由于define的性质，这条语句被替换成:
``` C++
int c = 2 * 1 + 2 * 2;// 6.
```
得到的结果为7，与我们想要的并不一致。
所以一般情况下我们会对宏的定义加上小括号，保证组合语义的正确:
``` C++
#define add(a, b) (a + b)
```
#### 条件编译
define配合ifdef等条件编译开关避免头文件的重复包含或者代码剔除：
``` C++
#ifndef __HEADERXXX__
#define __HEADERXXX__
// xxxx
void Func()
{
	#ifdef _DEBUG
	std::cout << "DEBUG INFO" << endl;
	#endif
}

#endif
```
在这段代码中，头文件不会被重复包含，如果没有定义_DEBUG，那么"DEBUG INFO"的打印语句就不会被编译到程序中。
### 2、typedef的基本用法
typedef可以给现有类型一个简单明了的新名字，或者简化一些复杂类型的声明。
#### 基本类型别名
``` C++
typedef unsigned int uint;
uint a = 1;
```
#### 数组类型别名
``` C++
int a[100];
int b[100];

// 等价
typedef int INTARR[100];
INTARR arra, arrb;
```
#### 函数指针别名
函数回调需要用到函数指针，例如:
``` C++
void Fun(int i);
void (*pFun)(int); // 声明一个返回值为void，参数为int的函数指针pFun
pFun = &Fun;
pFun(1);
```
这样声明并赋值函数指针比较复杂，可以用typedef简化：
``` C++
typedef void (*PFun)(int); // 定义返回值为void，参数为int的函数指针类型pFun
PFun pFun = &Fun;
pFun(1);
```
这样代码就变得更加简洁易读。
#### 复杂类型别名
当我们用stl的时候，经常会用迭代器，如果定义每个迭代器都要把类型写全的话，代码的可读性会很差，相反，用typedef可以得到很好的简化效果：
``` C++
// 不用typedef
std::vector<int>::iterator it1;
std::vector<int>::iterator it2;

// 使用typedef
typedef std::vector<int>::iterator vecIntIter;
vecIntIter it3, int4;
```
## 二、define和typedef的区别
### 1、作用时间不同
define是宏定义，发生在预处理阶段，编译阶段之前，只进行简单的字符串替换。

typedef在编译阶段起作用，有类型检查。
``` C++
typedef unsigned int uint;
typedef UINT unsigned int;
uint a = "abc"; // 不能将const char* 赋值为 uint类型的变量.
UINT b = "abc"; // 不能将const char* 赋值给 unsigned int类型的变量.
```
从报告的错误信息可以看出二者的区别。
### 2、作用域不同
define没有作用域的限制，只要在之前预定义的宏，之后的程序都可以使用。
而typedef有作用域。
``` C++
void Fun1()
{
	#define UINT unsigned int
	typedef unsigned int uint;
}

void main()
{
	UINT a = 1; // OK.
	uint b = 1; // Error.未定义的标识符uint.
}
```
### 3、修饰指针的不同
注意define和typedef修饰指针的不同之处：
``` C++
typedef int* pint;
#define PINT int*

int a = 1;
int b = 2;
pint p1, p2; // p1, p2均为int*类型
PINT p3, p4; // p3为int*类型，p4为int类型
const pint cp1 = &a;
const PINT cp2 = &b;

*cp1 = 3; // OK.
 cp1 ++;  // Error. const 修饰的是 cp1指针变量, 指针变量不可更改，指向的内容可以更改。

*cp2 = 3; // Error.
cp2 ++;   // ok. const修饰的是int类型，指针指向的内容不可更改，但是指针变量可以改变。
```
容易引起误解的是const pint cp = &a; 因为编译器将pint当成一个类型，所以const pint cp 与 pint const cp是等价的，const均是修饰的cp变量。
写了一个例子加深理解：
``` C++
typedef int* pint;
typedef pint const PINT1;
typedef const pint PINT2;
typedef int* const PINT3;
typedef const int* PINT4;
a = 1;
PINT1 pp1  = &a;
PINT2 pp2 = &a;
PINT3 pp3 = &a;
PINT4 pp4 = &a;

*pp1 = 3;	// OK.
pp1 ++;		// Error.

*pp2 = 3;	// OK.
pp2 ++;		// Error.

*pp3 = 3;	// OK.
pp3 ++;		// Error.

*pp4 = 3;	// Error.
pp4 ++;		// OK.
```
可以得出结论，PINT1, PINT2, PINT3是等价的，const均修饰的是指针变量，PINT4类型的const修饰的是int类型，即指针指向的变量。