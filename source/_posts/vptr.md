---
title: 虚函数的实现原理
categories: C++
---
C++向我们提供了多态的机制。将基类类型的指针，指向子类的实例，通过基类的指针访问子类的函数。本文对虚函数的实现原理及相应的内存布局做了的深入了解。

## 一、虚函数表的引入

多态的示例如下，语法不再赘述。

``` c++
class Base
{
public:
	virtual void FunA() { cout << "Base::FunA()" << endl; }
};

class Derive : public Base
{
public:
	virtual void FunA() { cout << "Derive::FunA()" << endl; }
};

int main()
{
	Base* base = new Derive;
	base->FunA();

	return 0;
}
```
得到的输出是：Derive::FunA()。

C++编译器为了实现多态机制，为每个拥有虚函数的类建立了一个虚函数表（virtual table）。虚函数表中存放的是虚函数的地址，解决了继承、覆盖、新虚函数的添加的问题，保证其真实反映实际的函数。

编辑器为每个对象实例都创建了一个虚函数表的指针，这个指针指向虚函数表。当发生虚函数调用的时候，通过虚函数指针来找对应虚函数表的函数指针地址，来保证函数的正确调用。

在vs编译器中，虚函数表的指针存在于对象实例中最前面的位置。这意味着我们通过对象实例的地址得到这张虚函数表，然后就可以遍历其中函数指针，并调用相应的函数。
<!-- more --> 
``` C++
	Derive derive;

	typedef void(*Fun) ();
	Fun pFun;

	int** vptr = (int**) &derive;
	pFun = (Fun)(*vptr)[0]; // 等价于 (Fun)(*((int*)(*vptr + 0) + 0))
	pFun(); // Derive::FunA()
```
由于虚函数表的指针放在实例中的最前面，将Derive对象的base指针强制转换成int\*\*可以取得虚函数表指针的地址，那么通过(\*vptr)[]可以很方便的取到虚函表里的虚函数地址。

我们通过VS编译器提供的cl命令来查看某个类的内存布局。使用方法如下：

- 开始  -> Microsoft Visual Studio 2017  -> x64 Native Tools Command Prompt for VS 2017；
- cl D:\MyGit\c++\Test\Test.cpp /d1reportSingleClassLayoutDerive

执行完命令,可以看到内存布局为：
```
class Derive    size(8):
        +---
 0      | +--- (base class Base)
 0      | | {vfptr}
        | +---
        +---

Derive::$vftable@:
        | &Derive_meta
        |  0
 0      | &Derive::FunA
```
可以注意到Derive的size显示的是8个字节，这是编译器做了8字节的内存对齐。

画成图的形式：

![image](https://i.loli.net/2018/03/29/5abcf4c16841e.bmp)

下面的内存模型的以打印形式展示，不再以图片形式给出，二者是等价的。

## 二、单继承的虚函数表

### 1、无虚函数覆盖

示例如下：
``` C++
class Base
{
public:
	virtual void FunA() { cout << "Base::FunA()" << endl; }
	virtual void FunB() { cout << "Base::FunB()" << endl; }
};

class Derive : public Base
{
public:
	virtual void FunC() { cout << "Derive::FunC()" << endl; }
	virtual void FunD() { cout << "Derive::FunD()" << endl; }
};

```
在这个继承中，子类没有重写父类的方法，而是加入了两个新的虚函数，内存模型如下：
```
class Derive    size(8):
        +---
 0      | +--- (base class Base)
 0      | | {vfptr}
        | +---
        +---

Derive::$vftable@:
        | &Derive_meta
        |  0
 0      | &Base::FunA
 1      | &Base::FunB
 2      | &Derive::FunC
 3      | &Derive::FunD
 ```
可以看到子类新增的虚函数按照声明顺序，依次存放在虚函数表中。

写代码测试证明：
``` C++
	Derive derive;
	Fun pFun;
	int** vptr = (int**) &derive;
	pFun = (Fun)(*vptr)[0]; // 等价于 (Fun)(*((int*)(*vptr + 0) + 0))
	pFun(); // Output: Base::FunA()

	pFun = (Fun)(*vptr)[1];
	pFun(); // Output: Base::FunB()

	pFun = (Fun)(*vptr)[2];
	pFun(); // Output: Derive::FunC()

	pFun = (Fun)(*vptr)[3];
	pFun(); // Output: Derive::FunD()
```

### 2、有虚函数覆盖

示例如下：
``` C++
class Base
{
public:
	virtual void FunA() { cout << "Base::FunA()" << endl; }
	virtual void FunB() { cout << "Base::FunB()" << endl; }
};

class Derive : public Base
{
public:
	virtual void FunA() { cout << "Derive::FunA()" << endl; }
	virtual void FunC() { cout << "Derive::FunC()" << endl; }
};

```
子类重写了父类的FunA方法，内存模型如下：
```
class Derive    size(8):
        +---
 0      | +--- (base class Base)
 0      | | {vfptr}
        | +---
        +---

Derive::$vftable@:
        | &Derive_meta
        |  0
 0      | &Derive::FunA
 1      | &Base::FunB
 2      | &Derive::FunC
```
在继承的时候，子类重写的虚函数将在虚函数表中替代父类的虚函数，新增的虚函数按照声明顺序以此存放在虚函数表中。

测试证明：
``` C++
	Derive derive;
	Fun pFun;
	int** vptr = (int**) &derive;
	pFun = (Fun)(*vptr)[0]; // 等价于 (Fun)(*((int*)(*vptr + 0) + 0))
	pFun(); // Output: Derive::FunA()

	pFun = (Fun)(*vptr)[1];
	pFun(); // Output: Base::FunB()

	pFun = (Fun)(*vptr)[2];
	pFun(); // Output: Derive::FunC()
```

## 三、多继承的虚函数表
### 1、无虚函数覆盖
子类继承父类，并且子类无函数重载，示例如下：
```
class Base
{
public:
	virtual void FunA() { cout << "Base::FunA()" << endl; }
	virtual void FunB() { cout << "Base::FunB()" << endl; }
};

class Base1
{
public:
	virtual void FunA() { cout << "Base1::FunA()" << endl; }
	virtual void FunB() { cout << "Base1::FunB()" << endl; }
};

class Base2
{
public:
	virtual void FunA() { cout << "Base2::FunA()" << endl; }
	virtual void FunB() { cout << "Base2::FunB()" << endl; }
};

class Derive : public Base, public Base1, public Base2
{
public:
	virtual void FunC() { cout << "Derive::FunC()" << endl; }
	virtual void FunD() { cout << "Derive::FunD()" << endl; }
};
```
打印内存模型：
```
class Derive    size(24):
        +---
 0      | +--- (base class Base)
 0      | | {vfptr}
        | +---
 8      | +--- (base class Base1)
 8      | | {vfptr}
        | +---
16      | +--- (base class Base2)
16      | | {vfptr}
        | +---
        +---

Derive::$vftable@Base@:
        | &Derive_meta
        |  0
 0      | &Base::FunA
 1      | &Base::FunB
 2      | &Derive::FunC
 3      | &Derive::FunD

Derive::$vftable@Base1@:
        | -8
 0      | &Base1::FunA
 1      | &Base1::FunB

Derive::$vftable@Base2@:
        | -16
 0      | &Base2::FunA
 1      | &Base2::FunB
```
多重继承下，虚函数表指针的数量与父类的数量一致，此例中，有三个虚函数表的指针，在实例中依次存放。子类中新加的虚函数会以声明顺序添加到第一个虚函数表中。
写例子测试结果如下：
``` C++
	Derive derive;
	Fun pFun;
	int** vptr = (int**) &derive;
	pFun = (Fun)vptr[0][0]; // 等价于 (Fun)(*((int*)(*vptr + 0) + 0))
	pFun(); // Output: Base::FunA()

	pFun = (Fun)vptr[0][1];
	pFun(); // Output: Base::FunB()

	pFun = (Fun)vptr[0][2];
	pFun(); // Output: Derive::FunC()

	pFun = (Fun)vptr[0][3];
	pFun(); // Output: Derive::FunD()

	pFun = (Fun)vptr[1][0];
	pFun(); // Output: Base1::FunA()

	pFun = (Fun)vptr[1][1];
	pFun(); // Output: Base1::FunB()

	pFun = (Fun)vptr[2][0];
	pFun(); // Output: Base2::FunA()

	pFun = (Fun)vptr[2][1];
	pFun(); // Output: Base2::FunB()
```
另外通过不同的指针来访问，调用对应的虚函数。
``` C++
	Base* base = &derive;
	Base1* base1 = &derive;
	Base2* base2 = &derive;

	base->FunA(); // Output: Base::FunA()
	base1->FunA(); // Output: Base1::FunA()
	base2->FunA(); // Output: Base2::FunA()
```
在Debug的时候会发现三个指针并不相同并且差了四个字节，编译器为我们做了偏移操作使虚函数表的指针指向对应的虚函数表。在第四部分再进行简单的介绍原理。
### 2、有虚函数覆盖

有虚函数覆盖的多重继承示例如下：
``` C++
class Base
{
public:
	virtual void FunA() { cout << "Base::FunA()" << endl; }
	virtual void FunB() { cout << "Base::FunB()" << endl; }
};

class Base1
{
public:
	virtual void FunA() { cout << "Base1::FunA()" << endl; }
	virtual void FunB() { cout << "Base1::FunB()" << endl; }
};

class Base2
{
public:
	virtual void FunA() { cout << "Base2::FunA()" << endl; }
	virtual void FunB() { cout << "Base2::FunB()" << endl; }
};

class Derive : public Base, public Base1, public Base2
{
public:
	virtual void FunA() { cout << "Derive::FunA()" << endl; }
	virtual void FunC() { cout << "Derive::FunC()" << endl; }
};
```
打印内存模型如下：
```
class Derive    size(24):
        +---
 0      | +--- (base class Base)
 0      | | {vfptr}
        | +---
 8      | +--- (base class Base1)
 8      | | {vfptr}
        | +---
16      | +--- (base class Base2)
16      | | {vfptr}
        | +---
        +---

Derive::$vftable@Base@:
        | &Derive_meta
        |  0
 0      | &Derive::FunA
 1      | &Base::FunB
 2      | &Derive::FunC

Derive::$vftable@Base1@:
        | -8
 0      | &thunk: this-=8; goto Derive::FunA
 1      | &Base1::FunB

Derive::$vftable@Base2@:
        | -16
 0      | &thunk: this-=16; goto Derive::FunA
 1      | &Base2::FunB
```
子类重载的虚函数会替换所有虚函数表里父类的虚函数，新加的虚函数会加到第一个虚函数表中。验证测试就不再贴出。
### 3、带有成员变量和虚函数的多重继承内存模型
以上例子的类中都不具有成员变量，虚函数表的指针在类示例中依次存放。在这个部分，在上面例子中基础上给类加上成员变量，更加清楚地了解类示例的内存模型。
``` C++
class Base
{
public:
	virtual void FunA() { cout << "Base::FunA()" << endl; }
	virtual void FunB() { cout << "Base::FunB()" << endl; }
	int a;
	int b;
};

class Base1
{
public:
	virtual void FunA() { cout << "Base1::FunA()" << endl; }
	virtual void FunB() { cout << "Base1::FunB()" << endl; }
	int a;
	int b;
};

class Base2
{
public:
	virtual void FunA() { cout << "Base2::FunA()" << endl; }
	virtual void FunB() { cout << "Base2::FunB()" << endl; }
	int a;
	int b;
};

class Derive : public Base, public Base1, public Base2
{
public:
	virtual void FunA() { cout << "Derive::FunA()" << endl; }
	virtual void FunC() { cout << "Derive::FunC()" << endl; }
	int a;
	int b;
};
```
打印内存模型：
```
class Derive    size(56):
        +---
 0      | +--- (base class Base)
 0      | | {vfptr}
 8      | | a
12      | | b
        | +---
16      | +--- (base class Base1)
16      | | {vfptr}
24      | | a
28      | | b
        | +---
32      | +--- (base class Base2)
32      | | {vfptr}
40      | | a
44      | | b
        | +---
48      | a
52      | b
        +---

Derive::$vftable@Base@:
        | &Derive_meta
        |  0
 0      | &Derive::FunA
 1      | &Base::FunB
 2      | &Derive::FunC

Derive::$vftable@Base1@:
        | -16
 0      | &thunk: this-=16; goto Derive::FunA
 1      | &Base1::FunB

Derive::$vftable@Base2@:
        | -32
 0      | &thunk: this-=32; goto Derive::FunA
 1      | &Base2::FunB
```
可以清晰的看到，示例的虚函数表指针不再是连续存放，父类按顺序存放，此时要把成员变量的内存占用考虑进去。写例子验证如下：
``` C++
	Derive derive;
	Fun pFun;
	int** vptr = (int**) &derive;
	pFun = (Fun)(*vptr)[0]; // 等价于 (Fun)(*((int*)(*vptr + 0) + 0))
	pFun(); // Output: Derive::FunA()

	pFun = (Fun)(*vptr)[1];
	pFun(); // Output: Base::FunB()

	pFun = (Fun)(*vptr)[2];
	pFun(); // Output: Derive::FunC()

	vptr += sizeof(Base) / sizeof(int);

	pFun = (Fun)(*vptr)[0];
	pFun(); // Output: Derive::FunA()

	pFun = (Fun)(*vptr)[1];
	pFun(); // Output: Base1::FunB()

	vptr += sizeof(Base1) / sizeof(int);

	pFun = (Fun)(*vptr)[0];
	pFun(); // Output: Derive::FunA()

	pFun = (Fun)(*vptr)[1];
	pFun(); // Output: Base2::FunB()
```
## 四、指针的表现
以上节带有成员变量和虚函数的多重继承为例，使用不同的父类指针指向子类：
```
	Base* pbase = &derive;
	Base1* pbase1 = &derive;
	Base2* pbase2 = &derive;
```
Base是第一父类，所以pbase指向的是derive对象的地址，编译器不需要做指针的偏移。
Base1是第二个父类，Base1的大小是12个字节，所以pbase1的地址是在基址的基础上加上12字节的便宜，即指向Base2的位置。
同理pbase2在基指上加上24个字节，指向Base3的位置。

但是当发生指针强制转换时，指针的地址没有发生变化。成员变量/虚函数与成员函数不同，成员变量/虚函数的访问会根据指针实际的地址来决定，而成员函数的调用会根据指针的类型来调用。

