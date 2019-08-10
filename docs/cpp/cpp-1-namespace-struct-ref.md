### 前言
在进行NDK开发的时候 ， 我们使用的很多第三方库 ， 大多数都是使用的C/C++编写的 ， 有些可能是C和C++混编的 。如，我在NDK系列中提到的[增量更新](http://www.jianshu.com/p/47d6c115f7ca)使用的两个库 `bsdiff`和`bzip2` 。那么 ， 我们在学完C语言之后 ， 有必要研究一下C++，因为C++是C的拓展 ， 加入了面向对象和模板技术 ，那么基本语法就不用介绍了。本系列假定你具有一定的编程经验，对面向对象有一定的了解 。

### 命名空间
在Java中 ， 我们使用包来区分文件的所在路径和类来自哪个包 ，使用`package`来声明文件在哪个文件目录下 ， 进而在使用的时候可以区分 ， 来自不同的包的相同类名 。在C++里面没有`package`关键字 ， 而是使用`namespace`来作为区分 ， 在PHP中也是如此 ， 在PHP5.0的时候引入的了`namespace`来做为区分不同路径下的同名函数和类 。

> 定义namespace

```c++
  /*
	cpp namespace demo
*/

#include <iostream>

/*
	自定义命名空间 ， 相当于Java中的包 。
	命名空间可以嵌套
*/
namespace NSP_A
{
	class A {
	public:
		void sayHello() {
			std::cout << "say Hello " << std::endl;
		}
	};

	// 嵌套namespace
	namespace NSP_A_1
	{
		class A {
		public:
			void sayHello() {
				std::cout << "Say Hello 1" << std::endl;
			}
		};
	}
}
```

> 使用namespace

```c++
// 标准命名空间 （包含很多标准的定义）
using namespace std;

// use namespace
using namespace NSP_A;


/*
	使用自定义命名空间
*/
void useNameSpace() {

	A a; 
	a.sayHello();

	// 嵌套命名空间使用
	NSP_A_1::A a2;
	a2.sayHello();

}
```
在C++中也定义了一些标准命名空间 ， 如`std` ，C++中也兼容C语言语法 ， 可以引入C语言头文件，使用C标准函数 。

### C++ 类 与 结构体

C++是面向对象语言 ， 面向对象语言的一大特征就是可以将类型整合起来 ， 可以创建类型实例 。

> 创建C++类

```c++
/*
	cpp class type
*/

#include <iostream>

using namespace std;

namespace CPP_CLASS
{
	class Animal {
	// C++ 共用权限访问修饰符
	private:
		char* name;
		int age;
	public:
		void setName(char* name) {
			this->name = name;
		}
		void setAge(int age) {
			this->age = age;
		}

		void showInfo() {
			cout << "名称：" << this->name << " 年岁：" << this->age << endl;
		}
	};
}
```
写法都是类似的 ， C++字段和函数都是采用的共享权限修饰符 ， 值得注意的是 ， 在C++中 ， 结构体里面的属性和函数也具有权限访问修饰符 。

```c++
/*
	CPP Struct
*/
namespace CPP_STRUCT
{
	// C++结构体与C结构不同之处在于 ， 在C++中结构体内字段函数可以有权限修饰符，用法和类用法一致
	// 和类不同的是 ， struct 不能继承
	struct Person
	{
	private:
		char* name;
		int age;
	public:
		void setName(char* name) {
			this->name = name;
		}
		void setAge(int age) {
			this->age = age;
		}

		void showPersonInfo() {
			cout << "姓名： " << this->name << "  年龄： " << this->age << endl;
		}
	};
}
```
结构体和类最大的不同是 ， 结构体不能继承 ， 不能进行抽象化。

> 使用C++类 与 结构体

```c++

/* CPP Class */
using namespace CPP_CLASS;

/* CPP Struct */
using namespace CPP_STRUCT;

/*
	Simple CPP Class
*/
void useCppClass() {

	Animal animal;
	animal.setName("dog");
	animal.setAge(2);

	animal.showInfo();

}

/*
	C and C++ 结构体的区别
*/
void useCppStruct() {

	Person p;
	p.setName("zeno");
	p.setAge(20);

	p.showPersonInfo();

}
```
在C++中 ， 使用类与使用结构体 ， 用法并无二致 ， 只是结构不能使用`new`而类可以使用 ， `new`出来的是一个对象指针 。

### 引用
相信大家对引用并不陌生 ， 在Java中 ， 我们常常将对象变量叫做对象引用 ， 在C++中也不例外 ， C++中的引用也可以作为对象变量 ， 但C++中的引用不会开辟新的空间 ， 去存储指向对象内存空间的地址值 ， 而是作为对象引用的一个别名 。

> 简单示意图

![引用](http://upload-images.jianshu.io/upload_images/643851-072bd8dc42f60e77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 引用示例

```c++
/*
	CPP 引用 
*/

#include <iostream>

namespace CPP_QUOTE {

	class Quote {
	public:
		int x = 10;

		// 引用 ，就是传入变量的别名，引用不会开辟新的内存空间，如同指针一样，指向的是传入变量的内存空间
		// 一般作为函数参数或返回值
		// 引用使用方便
		void swip(int &a, int &b) {
			int c = 0;
			c = a;
			a = b;
			b = c;
		}

		// 指针交换
		void swip_p(int* a , int* b) {
			int c = 0; 
			c = *a;
			*a = *b;
			*b = c;
		}
	};

}
```
引用在函数传值的时候， 可以当指针来使用 ， 引用最广泛的用途就在于 ， 在函数参数中 ， 充当指针用 。

> 引用使用

```c++

/* C++ 引用的使用 */
/* CPP Quote */
using namespace CPP_QUOTE;

void useCppQuote() {

	Quote q;
	// Quote q 的别名
	Quote &q1 = q;

	q1.x = 100;

	printf("q的内存地址 ： %#x , q1的内存地址：%#x\n", &q, &q1);

	cout << " q == " << q.x << endl;

	int x = 20, y = 40;

	q1.swip(x, y);

	cout << " q swipe  x = " << x << "  y = " << y << endl;

	// 指针值交换
	q1.swip_p(&x, &y);

	cout << " 指针值交换 ：   x = " << x << "  y = " << y << endl;
}
```
打印对象的地址与引用变量的地址 ， 我们会发现内存地址是一致 ， 因为引用是变量的别名 ， 不会创建新的内存空间。

### 结语
有了C语言基础 ， 学C++ ， 只要掌握语法的差异性和一些C++的特性 ， C++还是很容易上手的 ， 基础语法 ， 需要动手练 ， 多做实验 ， 慢慢就会熟能生巧 。
