在上篇中 ， 我们初步了解了C语言的IO操作 ， 编写IO操作的大致流程：
### 文件IO编写步骤：
1.使用`fopen`函数 ， 得到文件指针
2.指定`fopen`的操作模式`r`,`w` （指定输入输出流）
3.创建缓冲区 ， 缓存读写数据（将流数据读入到内存或写入到磁盘）
3.关闭流 （关闭文件流）

接着介绍了文件的加密解密 ， 文件的存储都是以二进制保存在磁盘上的 ， 所以我们可以通过二进制运算来进行文件的加密解密操作 。在C语言的IO函数中 ， 没有直接获取文件大小的函数 ， 所以我们需要自己通过文件指针来获取文件的大小：
```c
/*获取文件大小*/
void getFileSize() {

	char* path = "E:\\android_pdf\\研磨设计模式.pdf";
	// 打开文件
	FILE* fp = fopen(path, "r");
	if (fp == NULL) {
		printf("打开文件失败\n");
		return;
	}

	// 重新定位文件指针 , SEEK_END文件末尾，0是文件指针的偏移量
	fseek(fp, 0l, SEEK_END);
	// 返回当前的文件指针，相对于文件开头的位移量
	long fileSize = ftell(fp);

	printf("文件大小：%ld M\n", fileSize / 1024 / 1024);

}
```

在jni.h中 ， 我们可以看到这样一段代码：
```c
typedef union jvalue {
    jboolean    z;
    jbyte       b;
    jchar       c;
    jshort      s;
    jint        i;
    jlong       j;
    jfloat      f;
    jdouble     d;
    jobject     l;
} jvalue;
```
这样C语言中的联合体 ， 那么什么是联合体呢 ？
>  联合体：不同类型的变量共用一段内存（相互覆盖） ， 始终只有一个成员存在 ， 最后赋值的那个 ， 有利于节省内存。
联合体大小：成员中最大的成员所占的字节数

联合体 ， 将不同的类型联合起来 ， 组成一个新的聚合类型 ， 这个类型可以是联合体中的任意类型 。
```c
/*联合体*/
/*
	不同类型的变量共用一段内存（相互覆盖） ， 始终只有一个成员存在 ， 最后赋值的那个 ， 有利于节省内存
	联合体大小：成员中最大的成员所占的字节数
*/
union mValue
{
	int		i;
	short	s;
	long	l;
	float	f;

};

/*联合体示例*/
void useUnion() {
	union mValue m;
	m.f = 23.4f;
	m.i = 100;  // 最后一次赋值有效

	printf("联合体：\n%f - %d\n", m.f, m.i);
}
```
在java中 ， 当我们需要一个类列举几个状态时 ， 因为状态是固定的 ， 所以我们通常的做法是使用枚举 ， 将状态列举出来 ， 在C语言也有枚举这个类型：
```c
/*
	枚举（列举所有的情况）
	限定值，保证取值的安全性
*/
enum NetStatus {
	NET_SUCEESS,
	NET_ERROR,
	NOT_NET,
	NET_FAILURE
};
```
下面我们来看一个示例 ， 来大致了解一下枚举的使用场景：
```c
/*枚举*/
/*
	枚举（列举所有的情况）
	限定值，保证取值的安全性
*/
enum NetStatus {
	NET_SUCEESS,
	NET_ERROR,
	NOT_NET,
	NET_FAILURE
};

/*模拟网络请求*/
void requestHttp(char* url, void(*callBack)(enum NetStatus status,char* res)) {
	printf("请求地址：%s\n", url);
	printf("请求网络....\n");
	Sleep(2000);
	enum NetStatus status = NET_SUCEESS;
	char* res = "如果 爱情是一场花火 ,一闪即逝的花火,我也要去追求\n";
	callBack(status, res);
}

/*网络回调函数*/
void callBackHttp(enum NetStatus status, char* res) {
	switch (status)
	{
	case NET_SUCEESS:
		printf("网络数据：\n%s", res);
		break;
	case NET_ERROR:
		printf("请求网络错误\n");
		break;
	case NOT_NET:
		printf("没有网络\n");
		break;
	case NET_FAILURE:
		printf("请求网络失败\n");
		break;
	default:
		printf("未知错误\n");
		break;
	}
}

/*枚举示例*/
void useEnum() {
	enum NetStatus status = NET_FAILURE;

	printf("枚举中元素的值：%d\n", status);

	char* url = "http://www.zhuyongit.com";
	requestHttp(url, callBackHttp);
}
```
我们模拟了网络请求的几种情况 ， 成功 ， 失败 ， 错误等等 ， 通过枚举将这些情况列举出来 ，然后通过网络回调函数传递到网络处理函数 ， 通过`switch`分支语句来进行不同网络状态的判断 。

在jni.h头文件中 ， 我们也可以见到类似的应用：
```c
typedef enum jobjectRefType {
    JNIInvalidRefType = 0,
    JNILocalRefType = 1,
    JNIGlobalRefType = 2,
    JNIWeakGlobalRefType = 3
} jobjectRefType;
```
使用枚举来标识对象引用的类型 ， 本地引用，全局引用，弱引用，等等 ，因为我们在使用jni调取java对象的时候 ， java虚拟机并没有将引用数加1 ， 所以我们需要用过加强引用类型 ， 来保证jni引用的java对象，不被虚拟机回收掉 。 
