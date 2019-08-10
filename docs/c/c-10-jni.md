在上篇中我们了解了 ， 多类型集合的联合体 ， 固定值集合的枚举 ， 内容相对比较简单 ， 今天我们谈谈预编译 ， 也是本系列最后一个知识点 ， C语言基础系列就要告一段落了 ， 要开始我们的jni系列了 ， JNI（Java Native Interface） 是java与C/C++进行通信的一种技术 ， 使用JNI技术，可以java调用C/C++的函数对象等等，Android中的Framework层与Native层就是采用的JNI技术 。

### 预编译
> 预编译（预处理，宏定义，宏替换）这种叫法 ， 关键字`#define` ， 其本质是替换文本。

首先我们了解一下C语言的执行过程：
1. 编译  --> 生成目标代码（.obj）
2. 连接 --> 将目标代码与C函数库合并 ， 生成最终可执行文件
3. 执行

> 预编译 ， 主要在编译时期完成文本替换工作 ， 常见的预编译指令有： `#include`,`ifndef`,`#endif`,`define`,`#pragma once`等等 。

我们在jni.h头文件中 ， 可以看到较多的预编译指令 ， 例如:
```c
#if defined(__cplusplus)
typedef _JNIEnv JNIEnv;
typedef _JavaVM JavaVM;
#else
typedef const struct JNINativeInterface* JNIEnv;
typedef const struct JNIInvokeInterface* JavaVM;
#endif
```
如果编译环境是C++， 则使用：
```c
typedef _JNIEnv JNIEnv;
typedef _JavaVM JavaVM;
```
C语言编译环境 ， 则使用：
```c
typedef const struct JNINativeInterface* JNIEnv;
typedef const struct JNIInvokeInterface* JavaVM;
```
这里的定义就是我们后需要结束的`JNIEnv`指针 ， 在C++环境中`JNIEnv`是一个一级指针 ， 但是在C语言环境中 ， 他是一个二级指针 ，这个我们将在jni系列中 ， 再详细说明 。

### 预编译示例
```c
// 定义一个常数
#define MAX 100 

void main() {

	int i = 99;
	if (i < MAX) // 在编译时期， 会将MAX替换成100
	{
		printf("i 小于 MAX\n");
	}
}
```
定义一个宏常量`MAX`他的代表`100` ， 宏常数没有类型 ， 只做替换。

### 宏函数
>  `#define` 预编译指令 ， 不光可以定义常量 ， 还可以定义函数 ， 因为其本质是替换 ， 所有可以简化很多很长的函数名称 。

在jni.h中 ， 也可以看到宏函数 ， 如下：
```c
#define CALL_TYPE_METHODA(_jtype, _jname)                                   \
    __NDK_FPABI__                                                           \
    _jtype Call##_jname##MethodA(jobject obj, jmethodID methodID,           \
        jvalue* args)                                                       \
    { return functions->Call##_jname##MethodA(this, obj, methodID, args); }
```
其中`(_jtype, _jname) `里面的`_jtype`  , `_jname`都是替换标识 ， 替换`_jtype`,`##_jname##` 。

### 宏函数示例
```c
// 普通函数
void _jni_define_func_read() {
	printf("read\n");
}

void _jni_define_func_write() {
	printf("write\n");
}

// 定义宏函数
#define jni(NAME) _jni_define_func_##NAME() ;

void main() {

	// 宏函数
	//jni(read); 可以简化函数名称
    jni(write) ;

	system("pause");
}
```
宏函数的核心就是替换 ， 只要记住这一点就够了 。下面我们就来做另一个实例 ，  打印类似android Log日志的形式的函数。
```c
// 模拟Android日志输出 , 核心就是替换
#define LOG(LEVE,FORMAT,...) printf(##LEVE); printf(##FORMAT,__VA_ARGS__) ;
#define LOGI(FORMAT,...) LOG("INFO:",##FORMAT,__VA_ARGS__) ;
#define LOGE(FORMAT,...) LOG("ERROR:",##FORMAT,__VA_ARGS__) ;
#define LOGW(FORMAT,...) LOG("WARN:",##FORMAT,__VA_ARGS__) ;

void main() {

	LOGI("%s", "自定义日志。。。。\n");
	LOGE("%s", "我是错误日志...\n");

	system("pause");
}
```
输出：
```c
INFO:自定义日志。。。。
ERROR:我是错误日志...
```
我们可以看到输出信息前面带有类型标识 。在这里做几点代码说明：
> 1. 上述代码中`LEVE`,`FORMAT`都是自定义的 ， 可以是任意名称 ， 只有后面替换的名称一致即可。
2. `LOG(LEVE,FORMAT,...)`中的`...`表示可变参数 ， 替换则是使用`__VA_ARGS__`这种固定写法 。
3. 宏函数的核心就是——替换

预编译指令就介绍到这里 ， 其中心点就是`替换`  。 学到这里 ， C语言的基础也介绍得七七八八了 ， 下面一起来分析分析 ， 我们编写JNI代码的一个比较重要的头文件jni.h 。

### 简要分析jni.h
jni.h我们编写NDK的时候需要引入的一个头文件 ， 它位于`android-ndk-r10e\platforms\android-21\arch-arm\usr\include\jni.h`不同的平台都有相应的jni.h 。

在文件开始部分 ， 定义了很多类型别名 ， 区分C++与C的编译环境 ， 如下：
```c
#ifdef __cplusplus
/*
 * Reference types, in C++
 */
class _jobject {};
class _jclass : public _jobject {};
class _jstring : public _jobject {};
class _jarray : public _jobject {};
class _jobjectArray : public _jarray {};

#else /* not __cplusplus */

/*
 * Reference types, in C.
 */
typedef void*           jobject;
typedef jobject         jclass;
typedef jobject         jstring;
typedef jobject         jarray;
typedef jarray          jobjectArray;
```
接着定义了对象引用的枚举 ， 和方法签名的结构体：
```c
typedef enum jobjectRefType {
    JNIInvalidRefType = 0,
    JNILocalRefType = 1,
    JNIGlobalRefType = 2,
    JNIWeakGlobalRefType = 3
} jobjectRefType;

typedef struct {
    const char* name;
    const char* signature;
    void*       fnPtr;
} JNINativeMethod;
```
接下来就是最重要的JNIEnv了 ， 区分了C和C++环境 
```c
struct _JNIEnv;
struct _JavaVM;
typedef const struct JNINativeInterface* C_JNIEnv;

#if defined(__cplusplus)
typedef _JNIEnv JNIEnv;
typedef _JavaVM JavaVM;
#else
typedef const struct JNINativeInterface* JNIEnv;
typedef const struct JNIInvokeInterface* JavaVM;
#endif
```
JNIEnv是一个结构体指针 ， 里面定义了很多函数 ， 有调用java方法的函数 ， 转换java类型的函数 ， 我可以看到 ， C编译环境中 ， JNIEnv是`struct JNINativeInterface*`的指针别名，我们来看看`JNINativeInterface`结构体是怎样的：
```c
/*
 * Table of interface function pointers.
 */
struct JNINativeInterface {
    void*       reserved0;
    void*       reserved1;
    void*       reserved2;
    void*       reserved3;

    jint        (*GetVersion)(JNIEnv *);

    jclass      (*DefineClass)(JNIEnv*, const char*, jobject, const jbyte*,
                        jsize);
    jclass      (*FindClass)(JNIEnv*, const char*);

    jmethodID   (*FromReflectedMethod)(JNIEnv*, jobject);
    jfieldID    (*FromReflectedField)(JNIEnv*, jobject);
    /* spec doesn't show jboolean parameter */
    jobject     (*ToReflectedMethod)(JNIEnv*, jclass, jmethodID, jboolean);

    jclass      (*GetSuperclass)(JNIEnv*, jclass);
    jboolean    (*IsAssignableFrom)(JNIEnv*, jclass, jclass);

```
大部分的操作都是在`JNINativeInterface`这个结构体里面，定义了很多操作函数 ， 比较常见的字符处理函数：
```c
jstring     (*NewStringUTF)(JNIEnv*, const char*);
jsize       (*GetStringUTFLength)(JNIEnv*, jstring);
/* JNI spec says this returns const jbyte*, but that's inconsistent */
const char* (*GetStringUTFChars)(JNIEnv*, jstring, jboolean*);
```
> 1. 将字符指针转换成java的String类型
2. 得到java的String类型长度
3. 将java的String类型转换成C的字符指针

值得注意的是 ， C++的JNIEnv结构体指针 ， 并没有重新实现 ， 而生将C中的`JNINativeInterface`结构体 ， 重新打了一次包 ， 如下：
```c
/*
 * C++ object wrapper.
 *
 * This is usually overlaid on a C struct whose first element is a
 * JNINativeInterface*.  We rely somewhat on compiler behavior.
 */
struct _JNIEnv {
    /* do not rename this; it does not seem to be entirely opaque */
    const struct JNINativeInterface* functions;

#if defined(__cplusplus)

    jint GetVersion()
    { return functions->GetVersion(this); }

    jclass DefineClass(const char *name, jobject loader, const jbyte* buf,
        jsize bufLen)
    { return functions->DefineClass(this, name, loader, buf, bufLen); }

    jclass FindClass(const char* name)
    { return functions->FindClass(this, name); }
```
因为C++是面向对象语言 ， 所有有上下文环境变量`this` ， 简化了C语言中调用函数需要传递本身结构体指针的操作 。

分析了部分jni.h的源码 ， 后续的源码 ， 我相信看完C语言系列， 也可以看得懂 ， 这里就不再做分析了 ， 读者可自行查看源码分析一遍 ，对写jni代码还是有好处的 ， 至少对JNIEnv结构体指针中的函数有一个大致的印象 。

### 结语
C语言基础系列 ， 就宣告完结了 ， 还有很多知识点没讲到 ，  还有很多需要学习 ， 但我们不是要把C语言学透了学精了 ， 再去学JNI 和NDK， 这样 ， 不知要到何年何月了 。 所以 ， 先学基础了解概貌 ， 将java与C结合起来 ， 着手做些东西出来 ， 然后再继续深入 。

> 语言都是相通的 ， 关键是解决问题的思路 。


