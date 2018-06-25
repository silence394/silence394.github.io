---
title: Lua类的实现思路
categories: C+=
---

好多年没有用到lua的元表，元方法了，之前掌握的不牢，已经忘记了怎么使用，复习一下。

目标是用lua实现以下C++例子的功能。
``` C++
class Anima
{
private:
	std::string	mName;

public:
	Anima(){}
	Anima(const std::string& str) : mName(str) {}

	void SetName(const std::string& str);
	std::string GetName();
};

class Dog : public Anima
{
private:
	;

public:
	Dog(){}
	Dog(const std::string& str) : Anima(str) {}
	void PlayBall();
}

Dog dog("wangcai");
dog:PlayBall();
```

## 一、 元表、元方法介绍
## 二、实现