---
title: C++函数重载解析
categories: C++
---

函数重载解析是函数调用处理过程的一部分，但是并不是每个调用都会用到重载解析。如果通过函数指针或者成员函数指针来调用，调用的函数是在运行期由指针来决定的，没有设计到重载解析。

一个函数调用，通常会进行以下的处理流程：
- 查找名称，生成一个重载集合
- 从重载集中删除任何与调用不匹配的候选函数，得到可行的候选函数集合
- 执行重载解析来寻找一个最佳的候选函数。如果找到，则选择这个最佳候选函数；否则，调用具有二义性
- 检查被选定的最佳候选函数，如不能访问私有成员等

下面对重载解析的规则进行简化剖析。

## 一、简化的重载解析
重载解析通过比较调用实参与候选函数参数的匹配程度，对所有的候选函数进行分级。对匹配级别高的候选函数，每个参数的匹配程度都不能低于匹配级别低的候选函数的相应参数的匹配程度。如例：
``` C++
void Add(int, double)；
void Add(long, int)
Add(1, 2);	// 产生二义性
```

第一个Add最佳匹配函数调用的第一个实参，第二个Add最佳匹配函数调用的第二个参数。

下面给出匹配分级的规则（从最佳匹配到最差匹配）：
- 完美匹配。参数的类型与调用实参相同，或者是实参类型的引用。也可以增加const或volatile限定符。
- 有细微调整的匹配。数组decay为指向数组第一个元素的指针，或添加const，让类型为int**的实参匹配为int const* const *的参数。
- 类型提升的匹配。提升是一种隐式转换，它包含把占位少的整数类型（如bool、char、short等）转换为占位多的类型（如int、unsigned int、 long等），还包括从float到double的类型转换。
- 标准类型转换的匹配。包含任何种类的标准转型（如int到float），但不包含隐式调用的类型转换和单参数的构造函数。
- 用户自定义类型转换的匹配。允许任何类型的隐式转换。
- 和省略号的匹配。省略号可以匹配任何类型。

如例子：
``` C++
void Fun(int);
void Fun(const int&);
Fun(1); // 错误，二义性

void Fun(int);
void Fun(double);
Fun(1.0f); // 调用Fun(double)，发生类型提升的匹配

void Fun(int);
void Fun(char);
Fun(true); // 调用Fun(int)，发生类型提升的匹配

void Fun(float)；
void Fun(char)；
Fun(1); // 错误，二义性，有两个标准类型转换的匹配

class Test
{
public:
	Test(int);
};

void Fun(Test){}
void Fun(...){}
Fun(1); // 调用Fun(Test);
```

对于大多数函数都符合以上的解析规则，还存在一些不能用这些原则解析的情况，下面给出这些原则的重要细节。
### 1、成员函数的隐含实参
我们知道对于类的成员函数隐藏着一个this的指针，对下面的例子，自定义类型转换函数隐藏着参数int：
``` C ++
class BadString
{
public:
	BadString(char* str)
	{ mStrPtr = str;}

	char& operator[](unsigned int index){ return mStrPtr[index]; }	// (1)

	operator char*(){}						// (2)

private:
	char* mStrPtr;
};

BadString str("abcccccc");
str[2] = 'd'; // 在VS2017上产生二义性
```
对于实参2，完美匹配类型是int，匹配类型(1)则需要发生int到unsigned int的隐式类型转换。但是仍然存在一个可选的函数(2)，其具有一个隐式的int类型的参数，str发生自定义类型转换为char*，但参数5与隐式的int类型的参数完美匹配，所以就产生了二义性。

我们可以通过将(1)的参数改为int类型，或者将用户自定义类型转换改为显示类型转换。
### 2、完美匹配的细化
对于int类型的参数，有3种参数类型可以与它完美匹配：int, int&，const int&。

如果实参是左值，优先考虑没有const的版本；如果实参是右值，则优先考虑const的版本。成员函数的隐式实参同样符合这种规则。
``` C++
void Fun(int&);
void Fun(const int&);
int i = 1;
Fun(i); // 调用Fun(int&)
Fun(1); // 调用Fun(const int&)

class Test
{
public:
	void Fun1();		// (1)
	void Fun1() const;	// (2)
	void Fun2() const;	// (3)
};

Test test1;
test1.Fun1(); // (1)
test1.Fun2(); // (3)
const Test test2;
test2.Fun1(); // (2)
test2.Fun2(); // (3)
```
如果再添加一个Fun(int&)的方法，上面Fun()的例子就会出现二义性。
``` C++
int i = 1;
Fun(i); // 二义性，Fun(int&)与Fun(int&)匹配程度一样

Fun(1); // 二义性，Fun(int&)与Fun(const int&)匹配程度一样
```

## 二、重载的细节
### 1、非模板优先
### 2、转型序列
### 3、指针的转型
### 4、仿方函数和代理函数
### 5、获取函数地址