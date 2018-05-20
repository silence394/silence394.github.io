---
title: C++内存泄漏分析工具（VLD）
categories: C++
---

C++的内存管理对于C++程序员来说是一个必须要掌握的技能，稍微不慎就会出现内存泄漏。那么我们可以通过内存泄漏分析工具检测我们的程序，检查出程序是否有内存泄漏以及分析内存泄漏的原因。

## VLD
VLD（Visual Leak Detector）是一款用于C++内存泄漏的检测工具。具有以下的功能：
- 可以得到内存泄漏位置的调用栈，以及所在文件和行号
- 可以得到泄漏内存的数据
- 轻量，使用简单，源码易读

VLD主页：https://kinddragon.github.io/vld/

文档：https://github.com/KindDragon/vld/wiki

在Windows下，下载其安装包安装，会默认将include 和 库目录添加到系统变量中。
如果需要自行配置工程，把安装目录下的头文件、Lib和Dll拷贝到自己的工程下，配置好C++头文件附加包含目录和链接附加库目录。

使用非常简单，如下：
``` C++
#include <iostream>
#include "vld.h"
int main( )
{
	int* test = new int[1024];
	return 0;
}
```

运行之后出现如下内存泄漏警告：
![image](https://i.loli.net/2018/05/20/5b017c0cee538.png)

可以看到调用栈，所在文件以行号，内存数据信息。

VLD有开关供用户调用，可以通过合理运用开关来分析一段时间内存的增长情况。
``` C++
VLDDisable( );
VLDEnable( );
// ...
VLDReportLeaks( );
```
通过这种使用可以获得从VLDEnable( ) 到 VLDReportLeaks( )之前的内存泄漏情况。对不断增长的内存，但是却没有泄漏，可以用这种方法查。

## 实现原理

VLD中通过重载operator new 和operator delete方法，记录 new 调用时候的行号，文件，内存申请大小等信息，delete的时候把对应的信息删除掉。那么在Report泄漏信息的时候，余下的记录即为泄漏的部分。

通过这种思路，实现了一个VLD的简易版本：
``` C++
#include <iostream>
#include <stdlib.h>
using namespace std;

struct Block
{
	Block*		next;
	void*		ptr;
	const char*	file;
	int			line;
	size_t		size;
};

Block*	gBlockHead = nullptr;

void Insert( Block* block )
{
	if ( block == nullptr )
		return;

	gBlockHead->next = block;
}

void Delete( void* ptr )
{
	Block* it = gBlockHead;
	Block* pre = gBlockHead;

	while( it->next != nullptr )
	{
		if ( it->next->ptr == ptr )
		{
			Block* t = it->next;
			it->next = t->next;
			free( t );

			return;
		}

		pre = it;
		it = it->next;
	}
}

void* operator new( size_t size, const char *file, int line)
{
	void* ptr = malloc( size );

	Block* block = (Block*) malloc(sizeof(Block));
	block->ptr = ptr;
	block->file = file;
	block->line = line;
	block->size = size;
	block->next = nullptr;

	Insert( block );

	return ptr;
}

void* operator new []( size_t size, const char *file, int line )
{
	void* ptr = malloc( size );

	Block* block = (Block*) malloc(sizeof(Block));
	block->ptr = ptr;
	block->file = file;
	block->line = line;
	block->size = size;
	block->next = nullptr;

	Insert( block );

	return ptr;
}

void operator delete ( void* ptr )
{
	Delete( ptr );
}

void operator delete[] ( void* ptr )
{
	Delete( ptr );
}

void Report( )
{
	Block* it = gBlockHead;

	if ( it->next == nullptr )
		cout << "Congratulation !!! no leak! " << endl;

	while ( it->next != nullptr )
	{
		Block* next = it->next;
		cout << "LeakInfo :" << endl;
		cout << "-- ptr : " << next->ptr << ", size = " << next->size << " bytes." << endl;
		cout << "-- file : " << next->file << endl;
		cout << "-- line : " << next->line << endl;
		cout << "------------------------" << endl;

		it = it -> next;
	}
}

#define new new(__FILE__, __LINE__)

int main( )
{
	Block* block = (Block*) malloc(sizeof(Block));
	block->next = nullptr;
	gBlockHead = block;

	int* test = new int;
	delete test;

	int* testarray = new int[10];

	Report( );

	system( "pause" );
	return 0;
}
```

例子打印如下：

![image](https://i.loli.net/2018/05/20/5b0181398fd12.png)