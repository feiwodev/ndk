在上一篇中 ， 我们学习了函数与二级指针 ， 函数和java中的方法类似 ， 只是缺少了访问控制符 ， 二级指针也就是指针的指针 ， 指针里面存储的是指针的地址 ， 可以通过`*`操作符不断往上追溯 ， 然后通过内存地址操作内存空间 。

### 函数指针
当我们定义一个函数的时候 ， 这个函数也会像变量一样 ， 会有一个内存地址 ，  我们也可以将函数定义成为一个函数指针 ， 但函数不同于变量 ， 变量存储的是固定的值 ， 而函数指针存储的是函数的内存地址 。下面我就用一个示例来说明：

```c
// windows 弹出框头文件
#include <Windows.h>

/*函数*/
void message() {
	MessageBox(NULL, "我是弹出框", "消息", NULL);
}

void main() {

	// 函数指针定义 , 返回值(函数指针名称)(函数参数) = 函数名称
	void(*func_p)() = &message;

	// 调用函数指针
	func_p();

	printf("函数指针地址：%#x\n", func_p);

      getchar();
}
```
输出：
```c
函数指针地址：0xe6d31073
```
我们可以通过函数指针地址 ， 反汇编我们的程序员 ， 查看他的跳转

![函数指针汇编1.png](http://upload-images.jianshu.io/upload_images/643851-54683d30e1dcf48b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由上图我们可以看出 ， 我们的函数指针 ， 通过jmp指令跳转到另一个地址上 ， 下面我来看看message的地址里面是什么：

![函数指针汇编2.png](http://upload-images.jianshu.io/upload_images/643851-1653c46da9f17dc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们看到通过函数指针里面存储的是一个函数的地址 ，然后通过一个jmp指令调到我们的函数定义执行 。

有了我们的函数指针 ， 我们可以做很多事情 ， 下面我们来看一个简单的示例：
```c
  
int add(int num1, int num2) {
	
	return num1 + num2;
}

int minus(int num1, int num2) {
	
	return num1 - num2;
}

// 将函数指针直接定义到函数形参中 ， 类似java中的多态
// 我们可以函数指针作为函数参数传入
void showMsg(int(*c)(int num1, int num2), int a, int b) {
		int r = c(a, b);
		printf("计算完成=%d\n", r);
}

void main() {

	showMsg(add, 10, 10);

	showMsg(minus, 30, 2);
}

getchar();


```
输出：
```c
计算完成=20
计算完成=28
```
通过传入函数地址 ， 就可以调用函数进行运算 ， 我们只要按照传入的参数和返回值 ， 写我们自己的函数 ， 通过这个方法 ， 我们就可以统一的实现我们函数的功能 。下面我们来写一个回调函数：
```c
/*模拟网络请求回调*/
void requestNet(char* url , void(*callBack)(char *)) {
	printf("请求地址：%s , 正在请求网络....\n",url);
	// 模拟网络请求耗时
	Sleep(2000);
	char* str = "我是请求的网络数据 ， 落花有意随流水 ， 流水无情恋落花\n";
	callBack(str);
}

/*回调函数*/
void netCallBack(char *str) {
	printf("网络请求回调\n");
	printf("请求到的数据：%s" ,str);
}

void main() {
    char* url = "www.zhuyongit.com";
	requestNet(url, netCallBack);
}
```
输出：
```c
请求地址：www.zhuyongit.com , 正在请求网络....
网络请求回调
请求到的数据：我是请求的网络数据 ， 落花有意随流水 ， 流水无情恋落花
```
我们模拟了网络请求的常见封装 ， 使用一个回调函数来接收我们请求回来的数据 。函数指针很强大 ， 我们可以直接传入函数名称 ， 再另一个函数里面执行我们传入的函数 ， 如果是在java里面 ，我还需要传入一个对象 ， 再通过对象来调用方法 ， 在C语言里面 ， 直接通过函数指针就可以搞定 。
