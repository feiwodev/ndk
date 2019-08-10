### 前言
在学Java的时候 ， 我们会接触到面向对象的三大特征，封装，继承，多态 。而Java是从C++里面脱胎出来的，所以也具有这些特征 ， 在上篇中 ， 介绍了继承，友元函数等 ， 今天来学习一下多态 。我相信大家对多态并不陌生 ， C++中的多态和Java类似，只是实现起来有些差别 。说明一下：基础篇只是对语言语法的简单介绍， 并不会深入语言机制，将C++与Java的不同之处点出 ， 所以相对比较枯燥 ， 各位看官也可仅仅有个印象即可，再在用到的时候，就有迹可寻。

### 多态
C++的多态用法和Java类似 ， 只是需要虚函数的支撑。
> Meat.h

```c++ 
// 父类 .h
#pragma once

/*肉类*/

class Meat {
	
public:
	// 使用虚函数 ， 实现多态
    virtual void getName();
};

```
> Meat.cpp

```c++
#include "Meat.h"
#include <iostream>

using namespace std;

void Meat::getName() {

	cout << "我是肉类 -- super class" << endl;

}
```
父类将要被复写的函数需要`virtual`表示 ， 虚函数才具有之类复写的特性。

>  Fish.h

```c++
#pragma once

/*
	鱼肉 继承 肉类
*/

#include "Meat.h"

class Fish : public Meat {

public:
	virtual void getName();
};
```

> Fish.cpp

```c++

#include "Fish.h"
#include <iostream>

using namespace std;

void Fish::getName() {

	cout << "我是鱼肉。。。" << endl;
}
```

> Chicken.h

```c++
#pragma once

/*
	鸡肉 继承 肉类
*/

#include "Meat.h"

class Chicken : public Meat{

public:
	virtual void getName();

};
```

>Chicken.cpp

```c++
#include "Chicken.h"
#include <iostream>

using namespace std;

void Chicken::getName() {

	cout << "我是鸡肉。。。" << endl;

}
```

> C++多态作用与用法和Java差不多

```c++
void printMeatInfo(Meat &m) {
	m.getName();
}

/*使用多态*/
void usePolymorphism() {

	Chicken c;
	Fish f;

	printMeatInfo(c);
	printMeatInfo(f);
}
```
函数的对象参数使用引用传递，可以减少对象的的临时生成，加大对内存空间的利用率。多态要在用的时候才会真正体会到他的便捷 ， 在C++中配合纯抽象类使用更佳 。

### 模板
C++中有模板函数还有模板类，类似于Java的泛型，在逻辑计算中，无需关心数据的具体类型 。在Java中，我们最常用的泛型救数`List`， 例如：`List<String> cities = new ArrayList();`下面，我们来看看C++的模板函数。

```c++
/*
	模板函数 根据实际类型，自动推导
*/
template<typename T>
void anySwap(T &t1,T &t2) {
	T tmp = NULL;
	tmp = t1; 
	t1 = t2;
	t2 = tmp;
}

/*使用模板函数*/
void useTemplateFunc() {

	int i = 10; 
	int j = 30;

	anySwap(i, j);

	cout << "i = " << i << "   j = " << j << endl;

	cout << " ------- 字符交换 ---------" << endl;

	char* c1 = "思念在蔓延";
	char* c2 = "一通电话在世界两边";

	anySwap(c1, c2);

	cout << "c1 = " << c1 << "   c2 = " << c2 << endl;
}
```
模板函数，在函数之前使用template<typename T>来声明模板函数，T为自动推导类型。下面，我们用模板，来模拟一下Java的ArrayList类。

```c++
/*
	CPP  template class 

	模板类 , 相当于Java的泛型类 , 模拟Java集合类
*/

#include <iostream>

using namespace std;

/*
	使用模板类加上纯虚函数模拟Java的Collection接口
*/
template <typename T>
class Collection {

public:
	virtual void add(T t) = 0;
	virtual void del(T t) = 0;
	virtual void change(int index, T t) = 0;
	virtual T*  query() = 0;
};

/*
	模拟Java的ArrayList集合
*/
template <typename T>
class ArrayList : public Collection<T> {
	
public:
	void add(T t) {

	}

	void del(T t) {

	}

	void change(int index, T t) {

	}
	T*  query() {
		return NULL;
	}
};

class templateClassDemo {

public:
	void useTemplateClass() {
		// 在使用的时候指定模板类的类型
		ArrayList<int> a;
		a.add(1);
		cout << "使用CPP模板类, 模拟Java集合类" << endl;
	}

};

```
C++的模板特性和Java的泛型及其类似，只是写法上有所区别，万物都是相通的，变的是外在，不变的是算法核心。

### 异常
C++的异常比Java的异常处理自由，可以通过随意的定义数字字符等抛出，然后通过catch函数参数类型去匹配。

```c++
    /*CPP 可以抛出任意类型的Exception*/
	void throwStringException() {
		throw "抛出字符类型Exception";
	}

	void throwIntException() {
		throw 1;
	}

    /*捕获异常*/
	void catchException() {
		try
		{
			throwStringException();
			throwIntException();
		
		}
		catch (char* e)
		{
			cout <<"捕获String类型的异常信息 ： "<< e << endl;
		}
		catch (int e1) {
			cout << "捕获int类型的异常信息：" << e1 << endl;
		}
	}
```

> 自定义异常

```c++
/*自定义异常类*/
class CustomException {

public:
	CustomException(char* msg) {
		this->msg = msg;
	}

	char* showMsg() {
		return this->msg;
	}

private:
	char* msg;
};
```

> 使用

```c++
// use
void throwObjectException() {
		// 不要抛异常指针 ， new了对象指针之后必须销毁 ， 并且还会产生对象副本，占内存
		// 还有标准异常处理 ， 查看API
		throw CustomException("我是自定义对象异常");
	}

      /*捕获异常*/
	void catchException() {
		try
		{
			
			throwObjectException();
		}
		catch (CustomException &ex) {
			cout << "捕获自定义异常：" << ex.showMsg() << endl;
		}
	}
```

### 结语
C++基础系列基本上到此结束了，因其基本语法与C比较像，就没做过多介绍，基本上只做了面向对象部分的介绍。

C++基础系列，断断续续，今天总算是可以告一段落了 ， 上个月由于一些事情停更了一段时间，日子也没以前那么悠闲了 ， 要学的东西如排山倒海般涌来，IT行业的发展是越来越快，要学的也越来越多，大到各种语言学习C/C++ ， Java ， Python ， JavaScript 。小到各种奇淫巧技 ， 无所不包 。

语言不是限定 ， 没有哪一个项目只用一种语言可以搞定的，学习语言，不只是学习那门语言的语法，更重要的是那门语言思想 。

### 路漫漫其修远兮 ， 吾将上下而求索 。
