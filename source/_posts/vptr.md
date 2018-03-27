---
title: 虚函数的实现原理探究
---
C++向我们提供了多态的机制。将基类类型的指针，指向子类的示例，通过基类的指针实际子类的函数。本文对虚函数的实现原理及相应的内存布局做了一定的探究。

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

在vs编译器中，虚函数表的指针存在于对象实例中最前面的位置（这是为了保证正确取到虚函数的偏移量）。这意味着我们通过对象实例的地址得到这张虚函数表，然后就可以遍历其中函数指针，并调用相应的函数。

``` C++
	Base* base = new Derive;

	typedef void(*Fun) ();
	Fun pFun;

	int** vptr = (int**) base;
	pFun = (Fun)(*vptr)[0]; // 等价于 (Fun)(*((int*)(*vptr + 0) + 0))
	pFun(); // Derive::FunA()
```

由于虚函数表的指针放在实力中的最前面，将Derive对象的base指针强制转换成int**可以取得虚函数表指针的地址，那么通过(*vptr)[]可以很方便的取到虚函表里的虚函数地址。

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)
