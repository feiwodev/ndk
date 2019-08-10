### 前言
C++是一门面向对象的编程语言 ， 创建用以创建对象 ， 则不在话下 ， 我们知道，Java在创建对象的时候需要使用关键字`new` (忽略反射)， 而在C++中创建对象可以使用`new`也可以不使用 ， 那么使用`new`和不使用 ， 有什么区别呢 ？ 答案是：对象存储的内存空间不一样 ， 不使用`new`关键字创建的对象，对象数据存储在栈内存空间里面 ， 使用`new`关键字 ， 则会将对象数据存储在堆内存空间 。

我们知道 ， 栈内存空间是固定有限的 ， 所有对象数据很大很多的话 ， 就需要存储在堆内存空间 ， 不然可能会造成栈溢出 。

### new 一个对象
使用`new`关键字创建对象 ， 写法和Java一样 ， `Music* m = new Music("决口不提！爱你","郑中基");` ， 就连对象存储的内存空间都是一样的 ， Java对象与C++ new出来的对象 ， 都是存储在堆内存空间 。有点区别的是 ， 使用new创建的C++对象 ， 需要使用`delete`来回收对象开辟的内存空间 ， 如果使用`free`函数回收开辟的内存空间，则对象执行完不会执行析构函数。

```c++
using namespace std;

class Songstr {
private:
	char* name;

public:
	Songstr(char* name) {
		this->name = name;
		cout << "Songstr构造函数" << endl;
	}

	char* getName() {
		return this->name;
	}

	~Songstr()
	{
		cout << "Songstr析构函数" << endl;
	}
};

class Music {
private:
	char* musicName;
	// 如果类中存在对象字段，而这个对象需要传入构造函数参数，则需要在当前类构造函数)后面：进行赋值初始化
	// Music(char* musicName,char* songstrName) : s(songstrName)
	Songstr s;

public:
	Music(char* musicName,char* songstrName) : s(songstrName) {
		this->musicName = musicName;
		cout << "Music构造函数" << endl;
	}

	void printInfo() {
		cout << "歌曲： " << this->musicName << endl;
		cout << "歌手： " << this->s.getName() << endl;
	}

	~Music()
	{
		cout << "Music析构函数" << endl;
	}
};


/*使用new关键字创建对象*/
void useNewKeyCreateObject() {
	// 使用new关键创建对象 ， 则会在堆内存中创建一个空间来存储对象数据
	// 需要使用delete关键字来回收堆内存中的内存空间 ， 也可以使用free
	// 使用free则不会出发析构函数
	Music* m = new Music("决口不提！爱你","郑中基");
	m->printInfo();

	// 回收为对象开辟的空间
	delete m;

	//// new一个数组
	//int* arr = new int[];

	//// 回收数组空间
	//delete[] arr;
}
```

### 继承
C++的基础方式和Java不同， C++可以多继承， 一个类可以继承多个类 ， 而对于父类构造函数的赋值也不一样 。C++的基础符号是 `:`以冒号作为继承符号`class Rose : public Flower` ， 在继承的时候最好明确继承权限(可不写继承权限，默认public) 

> 继承权限列表：

|基类|继承权限|子类|
|:----:|:----:|:---:|
|public|public继承|public|
|public|protected继承|protected|
|public|private继承|private|
|protected|public继承|protected|
|protected|protected继承|protected|
|protected|private继承|private|
|private|public继承|子类无权访问|
|private|protected继承|子类无权访问|

```c++
using namespace std;

class Flower {

protected:
	int id;
	char* name;

public:
	Flower(int id, char* name) {
		this->id = id;
		this->name = name;
	}

	void printInfo() {
		cout << "id = " << this->id << "    name = " << this->name << endl;
	}
};


/*
	CPP 可以进行多继承 ， 继承父类的的构造函数按照如下编写 ， 也可直接传入默认值
*/
class Rose : public Flower {

public:
	/*子类继承父类，父类有构造函数，需要子类传入，则在构造函数后面使用:父类名(传入参数)*/
	Rose(int id, char*name) : Flower(id, name) {

	}

	void printInfo() {
		cout << "id = " << this->id << "    name = " << name << " -- "<< endl;
	}
};
```
因为C++是多继承的特性 ， 所有就会造成继承的二义性，所以我们需要使用虚继承来解决同名成员不明确的问题 。

```c++
//继承的二义性
//虚继承，不同路径继承来的同名成员只有一份拷贝，解决不明确的问题
/*
class A{
public:
	char* name;
};

class A1 : virtual public A{
	
};

class A2 : virtual public A{

};

class B : public A1, public A2{

};

void main(){
	B b;	
	b.name = "zeno";
	//指定父类显示调用
	//b.A1::name = "zeno";
	//b.A2::name = "zeno";
	system("pause");
}
```
Tips：虚继承 ， 在继承父类的时候使用`virtual`关键字修饰 ， 这个关键字， 后续还会遇到很多次

### 友元函数
如果在Java中，子类想访问父类的私有成员，除非使用反射机制，不然那是不可能的 。但在C++中 ，想要访问父类的私有成员 ， 那就轻松很多了 。我们只需要在父类中，声明子类是父类的友元类 ， 那么就可以在之类访问修改父类的私有成员了。

```c++
/*
	friend function demo

	// 友元函数可以访问一个类的任何字段和函数
*/

#include <iostream>

using namespace std;

class FriendFunction {

	// 创建一个友元函数 ， 用来访问私有字段 ， 需要在外部实现函数
	friend void accessFriendFunctionFeild(FriendFunction *p, int id, char* name);

	// 友元类 ， 友元类可以访问此类任意字段和函数
	friend class FriendClass;

// 类私有字段 , 私有字段，常规访问需要创建public get的访问函数 , 不能进行直接访问
private:
	int id;
	char* name;


public:
	FriendFunction(int id, char* name) {
		this->id = id;
		this->name = name;
	}

	void printfInfo() {
		cout << "id = " << this->id << "   name = " << this->name << endl;
	}

};

/*作为类的友元类，可以修改类中任意字段与函数*/
class FriendClass {

private:
	FriendFunction *f;

public:
	FriendClass(FriendFunction *f) {
		this->f = f;
	}

	void accessFriendFeild() {
		f->id = 100;
		f->name = "友元类修改";
	}

};
```
不管是友元类还是友元函数 ， 只要在类中声明 ， 那么此类相当于他们就没有秘密了，友元类和友元函数，可以任意访问此类的私有成员。需要注意的是，友元函数需要在类的外部实现 。

> 使用友元函数，友元类

```c++
/*友元函数实现 用来访问私有字段*/
void accessFriendFunctionFeild(FriendFunction *p, int id, char* name) {
	p->id = id;
	p->name = name;
}

/*友元函数使用*/
void useFriendFunction() {

	FriendFunction *f = new FriendFunction(1, "找不到你的方向");
	f->printfInfo();

	accessFriendFunctionFeild(f, 3, "我是你的眼");
	f->printfInfo();

	/*java中的Class对象，可以访问任意对象的任意字段，本质是Class类每个类的友元类*/
	FriendClass fc(f);
	fc.accessFriendFeild();
	f->printfInfo();
}
```

### 函数可变参数
函数的可变参数，在Android很常见，比如我们经常使用的`AsyncTask` 。在C++也有函数的可变参数 ，常见于Android LOG函数。 

```C++
/*
	CPP 可变参数
*/

#include <stdarg.h>
#include <iostream>

using namespace std;

class VariableParameter
{
public:

	/*可变参数*/
	void VariableParameter::variableParams(int i, ...)
	{
		// 可变参数列表
		va_list ap_list;
		// 从...之前的第一个参数开始，后面就是可变参数
		va_start(ap_list, i);
		// 得到可变参数 ， 参数①可变参数列表  ， 参数②参数类型。
		int age = va_arg(ap_list, int);
		double weight = va_arg(ap_list, double);
		char* registry = va_arg(ap_list, char*);
		va_end(ap_list);

		cout << "编号: " << i << " 年龄： " << age << " 体重： " << weight << "  籍贯： " << registry << endl;
	}

};
```

> 使用可变参数

```c++
/*函数可变参数*/
void useVariableParameter() {
	VariableParameter v;
	v.variableParams(1, 20,100.2,"湖南");
}
```
Tips：C++的可变参数，必须含有一个固定参数，可变参数类型不固定，可以是任意数据类型，通过`va_arg()`函数指定数据类型 。


### 结语
C++与Java有很多相似的地方，学习起来 ， 应该没那么晦涩难懂， 只是有些语法上的差别，和一些特殊的用法 。作为Android程序员学C++，不一定要去成天写C++，但至少要能看懂，要能修改 ， 我们的主要目的就是用成熟的C/C++库 ， 编写小模块的C/C++程序 。

###源码
[Hello_CPP](https://github.com/zhuyongit/Hello_CPP)
